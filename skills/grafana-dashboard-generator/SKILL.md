---
name: 'grafana-dashboard-generator'
description: 'Skill for generating complete, modern, dark-themed Grafana Dashboard JSON models visualizing full 360-degree Spring Boot Prometheus metrics and Loki log streams.'
---

# Grafana Dashboard Generator Skill

Use this skill when asked to:
- *"Generate a Grafana dashboard JSON for our Spring Boot app"*
- *"Create Grafana visualizations for Prometheus and Loki logs"*
- *"Show incoming controller traffic, outbound third-party latency, HikariCP, and JVM memory in Grafana"*
- *"Set up Prometheus alerting rules for Spring Boot"*

---

## 🛠️ Step-by-Step Skill Workflow

### Step 1: Collect Metadata
Confirm target Prometheus datasource name (`Prometheus`), Loki datasource name (`Loki`), and service name (`spring.application.name`).

### Step 2: Generate Dashboard JSON
Generate `dashboards/spring-boot-360-overview.json` configured with:
- Top KPI stat cards (RPS, p95 latency, 5xx error rate %, outgoing API errors, Heap %, Hikari pending).
- Incoming Controller APIs panels (request rates, p50/p90/p95/p99 latency time series, status distribution, slowest endpoints table).
- Outgoing Third-Party Integrations panels (request rates by client, p95 latency by service, error/timeout counters, SLA table).
- Database & Connection Pools panels (Hikari active vs idle vs pending vs max, acquisition latency).
- JVM & System Resources panels (Heap memory by generation, GC pause duration, thread states, CPU usage).
- Async Task Executor panels (active threads, queue depth, throughput).
- Live Loki Logs panel (log volume by severity bar chart, real-time error log stream with traceId search).

### Step 3: Provide Alerting Rules
Output `prometheus-alerts.yml` with rules for high error rates, connection pool exhaustion, and memory limits.
