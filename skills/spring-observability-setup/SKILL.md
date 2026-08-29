---
name: 'spring-observability-setup'
description: 'Skill for setting up full 360-degree observability in Spring Boot: incoming controller metrics, outgoing third-party API metrics, HikariCP database pool, JVM/GC/Threads, structured JSON logging for Grafana Loki with Promtail, and MDC correlation tracing.'
---

# Spring Observability Setup Skill (Full 360° Observability)

Use this skill when asked to:
- *"Set up full application metrics in Spring Boot for Prometheus & Grafana"*
- *"Track incoming controller requests (latency, status codes, percentiles, error rates)"*
- *"Track outgoing third-party HTTP calls with Micrometer and cardinality protection"*
- *"Monitor HikariCP database connection pools, JVM heap/GC, and async thread executors"*
- *"Configure structured JSON logging for Loki with Promtail"*
- *"Add MDC correlation tracing (traceId) to Spring Boot requests"*
- *"Refactor messy log statements to production JSON format"*

---

## 🛠️ Step-by-Step Skill Workflow

### Step 1: Detect Spring Boot Stack & Endpoints
Inspect `pom.xml` / `build.gradle`, `@RestController` endpoints, and outbound clients (`RestTemplate`, `RestClient`, `WebClient`, `FeignClient`).

### Step 2: Add Dependencies & 360° Metrics Configuration
Add `spring-boot-starter-actuator`, `micrometer-registry-prometheus`, `spring-boot-starter-aop`, and `logstash-logback-encoder`.
Configure `application.yml` with distribution percentiles (p50, p90, p95, p99), SLAs, and HikariCP pool metrics.

### Step 3: Register Observable Client Interceptor & TimedAspect
- Register `ObservableClientHttpRequestInterceptor` to capture `http.client.requests`.
- Register `TimedAspect` to enable `@Timed` on custom business methods.

### Step 4: Configure `logback-spring.xml` for Loki
Set up rolling asynchronous JSON logs in `logs/app-json.log` with full MDC context (`traceId`, `userId`, `clientIp`, `uri`, `httpMethod`).

### Step 5: Configure Promtail
Deploy `promtail-config.yml` to ship `logs/app-json.log` to Grafana Loki.
