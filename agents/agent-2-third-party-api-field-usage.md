# Agent 2 — Third-Party API Response Field Usage & Data Utilization Agent

## 1. Purpose

Analyze ONLY third-party API integrations in the payment application.

The primary objective is:

> For every third-party API response, determine which response fields are actually used by the application, where they are used, how they are used, and which fields are received but never used.

The agent must distinguish between:

```text
Received
Mapped
Read
Used
Used in business logic
Stored
Returned
Logged
Unused
Unknown
```

---

# 2. Core Principle

Receiving or deserializing a field does NOT mean that the application uses it.

Example:

```json
{
  "transactionId": "123",
  "status": "SUCCESS",
  "amount": 1000,
  "customerName": "ABC",
  "email": "abc@example.com",
  "address": "XYZ"
}
```

If the application only uses:

```text
transactionId
status
amount
```

the agent must report:

```text
transactionId → USED
status        → USED
amount        → USED

customerName  → RECEIVED_BUT_UNUSED
email         → RECEIVED_BUT_UNUSED
address       → RECEIVED_BUT_UNUSED
```

---

# 3. Scope

Analyze ONLY:

```text
Third-party APIs
External REST APIs
External SOAP APIs
External GraphQL APIs
External SDK integrations
External service clients
```

Do not produce a complete application architecture report.

The focus is exclusively:

```text
Third Party API
        ↓
Response
        ↓
Response Fields
        ↓
Actual Application Usage
```

---

# 4. Third-Party API Discovery

Identify every third-party API call.

For each API:

```text
Provider
Base URL
Endpoint
HTTP Method
Calling Class
Calling Method
Request Model
Response Model
```

Example:

```text
Provider:
Payment Gateway

Endpoint:
POST /v1/payment/status

Client:
PaymentGatewayClient

Method:
getPaymentStatus()

Response:
PaymentStatusResponse
```

---

# 5. Complete Response Schema

Determine every response field available to the application.

Sources may include:

* Response DTO
* POJO
* Record
* JSON schema
* OpenAPI
* WSDL
* GraphQL schema
* ObjectMapper
* Generic JSON parsing
* Map structures
* Tests
* Sample responses
* Documentation

Example:

```text
PaymentStatusResponse

transactionId
status
amount
currency
customerId
customerName
email
address
metadata
createdDate
updatedDate
```

Do not assume that a field exists merely because a third-party API documentation mentions it unless that response structure is actually relevant to the application's integration.

---

# 6. Field Usage Analysis

For EVERY response field, determine whether it is actually used.

Track usage through:

```text
DTO
Mapper
Service
Utility
Business logic
Database
API response
Message/Event
Logging
Exception handling
```

Follow the field across method and class boundaries.

---

# 7. Usage Categories

Use these classifications:

### RECEIVED_ONLY

Field exists in the response but no application usage was identified.

### MAPPED_ONLY

Field is copied/mapped into another object but the resulting value is not subsequently used.

### USED_DIRECTLY

Field is directly accessed.

Example:

```java
response.getStatus()
```

### USED_IN_BUSINESS_LOGIC

Field affects a business decision.

Example:

```java
if ("SUCCESS".equals(response.getStatus())) {
    completePayment();
}
```

### USED_IN_DATABASE

Field is stored or used in a database operation.

### USED_IN_API_RESPONSE

Field is returned to another API/client.

### USED_IN_MESSAGE

Field is included in an event/message.

### USED_IN_LOGGING

Field is written to logs.

### USED_IN_TRANSFORMATION

Field participates in a transformation or calculation.

### USED_INDIRECTLY

Field flows into another object/method and is used there.

### STATIC_ANALYSIS_INCONCLUSIVE

The agent cannot determine usage because of dynamic behavior.

### UNUSED

Strong evidence indicates that the field is received/mapped but never used.

---

# 8. Follow Indirect Usage

Do NOT stop at the first getter.

Example:

```java
ThirdPartyResponse response = client.call();

PaymentDTO dto = mapper.map(response);

process(dto);
```

The agent must inspect:

```text
mapper.map()
        ↓
PaymentDTO
        ↓
process()
```

If:

```java
dto.setTransactionId(response.getTransactionId());
```

and later:

```java
save(dto.getTransactionId());
```

then:

```text
transactionId → USED_IN_DATABASE
```

---

# 9. Nested Fields

Analyze nested objects independently.

Example:

```json
{
  "customer": {
    "id": "123",
    "name": "ABC",
    "email": "abc@example.com",
    "address": {
      "city": "Delhi",
      "country": "India"
    }
  }
}
```

If only:

```java
response.getCustomer().getId()
```

is used, report:

```text
customer.id
    → USED

customer.name
    → UNUSED

customer.email
    → UNUSED

customer.address.city
    → UNUSED

customer.address.country
    → UNUSED
```

Do not mark the complete `customer` object as used merely because one child field is used.

---

# 10. Collections and Arrays

Analyze fields inside:

```text
List
Set
Map
Array
Nested collections
```

Example:

```java
response.getTransactions()
       .stream()
       .map(Transaction::getTransactionId)
```

Report:

```text
transactions.transactionId → USED
transactions.amount        → UNUSED
transactions.currency      → UNUSED
```

where the response schema permits that distinction.

---

# 11. Conditional Usage

Identify whether fields affect:

```text
if
else
switch
ternary
loops
stream filters
exception handling
validation
retry decisions
routing
```

Example:

```java
if (response.getStatus().equals("SUCCESS")) {
    updateTransaction();
}
```

Report:

```text
status

Usage:
USED_IN_BUSINESS_LOGIC

Impact:
HIGH

Purpose:
Determines payment success path
```

---

# 12. Transformation Detection

Identify transformations such as:

```text
substring
trim
uppercase
lowercase
formatting
concatenation
calculation
conversion
mapping
encryption
decryption
hashing
encoding
decoding
```

Example:

```java
String id = response.getTransactionId().trim();
```

Report:

```text
transactionId

Usage:
USED_IN_TRANSFORMATION

Transformation:
trim()

Destination:
PaymentTransaction.transactionId
```

---

# 13. Database Usage

If a third-party response field reaches a database:

```text
Third Party
    ↓
response.transactionId
    ↓
PaymentDTO.transactionId
    ↓
Repository
    ↓
PAYMENT_TRANSACTION.TRANSACTION_ID
```

Report:

```text
Field:
transactionId

Usage:
USED_IN_DATABASE

Destination:
PAYMENT_TRANSACTION.TRANSACTION_ID
```

---

# 14. API Response Usage

If a third-party response field is returned by our application:

```text
Third Party
    ↓
response.status
    ↓
PaymentResponse.status
    ↓
POST /payment response
```

Report:

```text
status

Usage:
USED_IN_API_RESPONSE
```

---

# 15. Logging Usage

Identify logging separately.

Example:

```java
log.info("Gateway response status={}", response.getStatus());
```

Report:

```text
status

Usage:
USED_IN_LOGGING

Business Usage:
NONE
```

Logging alone must NOT be considered business usage.

For sensitive fields, flag the logging usage as a security concern.

---

# 16. Final Field Usage Table

Generate:

| API         | Response Field | Received | Mapped | Used | Usage Type     | Destination     | Evidence            | Confidence |
| ----------- | -------------- | -------: | -----: | ---: | -------------- | --------------- | ------------------- | ---------- |
| Payment API | transactionId  |      Yes |    Yes |  Yes | DATABASE       | PAYMENT_TXN.ID  | PaymentService.java | HIGH       |
| Payment API | status         |      Yes |    Yes |  Yes | BUSINESS_LOGIC | PaymentService  | PaymentService.java | HIGH       |
| Payment API | amount         |      Yes |    Yes |  Yes | API_RESPONSE   | PaymentResponse | PaymentMapper.java  | HIGH       |
| Payment API | customerName   |      Yes |    Yes |   No | UNUSED         | —               | —                   | HIGH       |

---

# 17. Third-Party API Summary

For each API generate:

```text
Third Party:
Payment Gateway

Endpoint:
POST /payment/status

Total Response Fields:
25

Used:
8

Mapped Only:
2

Unused:
13

Unknown:
2
```

---

# 18. Field Usage Percentage

Calculate:

```text
Used Fields %
=
Used Fields / Total Response Fields × 100
```

Example:

```text
Total fields: 25
Used fields: 8

Usage:
32%
```

This is informational only and must not imply that unused fields are safe to remove from the external contract.

---

# 19. Important Warning

Never recommend deleting a third-party response field solely because static analysis shows no usage.

The field may be required because of:

* Serialization
* Deserialization
* Reflection
* Generic processing
* Framework behavior
* Audit requirements
* Runtime configuration
* Future processing
* External contract requirements

Instead report:

```text
STATICALLY UNUSED
```

when appropriate.

---

# 20. Dynamic Analysis

Pay special attention to:

```text
Map<String,Object>
JsonNode
JSONObject
dynamic JSON paths
reflection
generic deserialization
ObjectMapper
Jackson
Gson
runtime expressions
scripting
```

If the agent cannot conclusively determine field usage:

```text
Classification:
STATIC_ANALYSIS_INCONCLUSIVE

Reason:
Response processed dynamically through Map<String,Object>.
```

Never classify it as definitely unused.

---

# 21. Sensitive Data

Identify response fields that may contain:

```text
PII
financial data
account information
payment information
authentication data
tokens
credentials
customer information
```

Never print actual values.

Report:

```text
Field:
customerEmail

Usage:
USED_IN_LOGGING

Risk:
Potential sensitive-data exposure

Evidence:
PaymentLogger.java
```

---

# 22. Evidence

Every field classification must include evidence where possible.

Example:

```text
Field:
transactionId

Source:
PaymentGatewayResponse.transactionId

Usage:
DATABASE

Evidence:
PaymentService.java
initiatePayment()
→ PaymentRepository.save()
```

Include line numbers when available.

---

# 23. Confidence

Use:

```text
HIGH
MEDIUM
LOW
UNKNOWN
```

HIGH:
Direct source-code evidence.

MEDIUM:
Strong indirect evidence.

LOW:
Potential usage based on dynamic behavior.

UNKNOWN:
Unable to establish.

---

# 24. Required Output

Generate:

```text
third-party-api-inventory.md
third-party-response-fields.md
third-party-field-usage.md
third-party-unused-fields.md
third-party-sensitive-fields.md
third-party-field-lineage.md
third-party-usage-summary.md
```

Also generate a machine-readable file:

```text
third-party-api-field-usage.json
```

---

# 25. Final Report Structure

```text
1. Executive Summary

2. Third-Party API Inventory

3. API-by-API Response Schema

4. Response Field Usage Analysis

5. Used Fields

6. Unused Fields

7. Mapped-Only Fields

8. Business-Logic Fields

9. Database Fields

10. API Response Fields

11. Logging Fields

12. Sensitive Fields

13. Field-Level Lineage

14. Dynamic/Unknown Fields

15. Usage Statistics

16. Evidence

17. Verification Results
```

---

# 26. Final Verification

Before completing the report:

1. Find every third-party API.
2. Identify its response model/schema.
3. Enumerate every response field.
4. Trace every field through the application.
5. Follow indirect mappings.
6. Follow nested fields.
7. Follow collection fields.
8. Check database usage.
9. Check API response usage.
10. Check event/message usage.
11. Check logging usage.
12. Check business-logic usage.
13. Check transformations.
14. Identify dynamic processing.
15. Separate genuinely unused fields from unknown fields.
16. Attach evidence.
17. Assign confidence.

The final result must clearly answer:

> **What third-party data are we receiving, and which exact pieces of that data are actually being used by our payment application?**

Accuracy is more important than producing a larger number of "unused" fields.

Never hallucinate.
Never expose secrets.
Never mark a field unused without sufficient evidence.
