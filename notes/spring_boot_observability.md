# Spring Boot Microservices Observability Ecosystem

## 1. Application-Level Build Dependencies
These libraries must be included in your `pom.xml` (Maven) or `build.gradle` (Gradle) to generate and format telemetry data natively using **Spring Boot 3.x** and **Micrometer**.

### 📊 Metrics Generation
* **Spring Boot Actuator** (`spring-boot-starter-actuator`): Exposes built-in production-ready endpoints (like `/actuator/metrics` and `/actuator/health`).
* **Micrometer Prometheus Registry** (`micrometer-registry-prometheus`): Formats, manages, and translates Actuator metrics into a time-series format readable by Prometheus.

### 🗺️ Distributed Tracing
* **Micrometer Tracing Bridge OTel** (`micrometer-tracing-bridge-otel`): Bridges Spring’s internal observability API to the standard OpenTelemetry (OTel) specification.
* **OpenTelemetry Exporter OTLP** (`opentelemetry-exporter-otlp`): The native transport layer mechanism used to push trace spans using the OpenTelemetry Protocol (OTLP).

### 📝 Structured Logging
* **Logstash Logback Encoder** (`logstash-logback-encoder`): Converts classic Spring Boot log outputs into structured JSON format. This automatically injects active `trace_id` and `span_id` contexts into your log patterns.

---

## 2. Telemetry Collection & Routing Agents
These helper components run alongside or outside your application containers to scrape, aggregate, and forward telemetry.

* **OpenTelemetry Collector (OTel Collector):** A high-performance proxy daemon that acts as a central hub. It receives traces or metrics from your services, processes/batches them, and exports them to diverse backends.
* **Prometheus Server:** A scraper engine configured to poll the `/actuator/prometheus` endpoint of each microservice at regular intervals.
* **Grafana Promtail / Fluentbit:** Lightweight log collection agents that tail system text files, capture raw stdout streams, and ship them to log backends.

---

## 3. Storage Backends & Visualization Platforms
The centralized infrastructure where telemetry data is preserved and indexed for active querying.

```
[ Spring Boot Microservice ]
        │
        ├─► Metrics ──► Prometheus ──────┐
        ├─► Logs ─────► Grafana Loki ────┼──► Integrated Grafana Dashboards
        └─► Traces ───► Tempo / Jaeger ──┘
```

### 📈 Metrics
* **Prometheus / VictoriaMetrics:** High-performance, time-series databases optimized for running PromQL metrics evaluations.
* **Grafana:** The single visual platform used to aggregate and dashboard all your metrics, logs, and traces.

### 🔍 Traces
* **Grafana Tempo:** A high-scale, cost-effective distributed tracing backend designed for object storage.
* **Jaeger / Zipkin:** Alternative tracing systems used to build UI timelines of service-to-service call latencies and failures.

### 🗄️ Logs
* **Grafana Loki:** A cost-efficient log aggregation platform that indexes metadata labels instead of entire log line contents.
* **OpenSearch / Elasticsearch:** Heavyweight document databases utilized when deep full-text indexing is required across highly verbose system logs.

---

## 4. Production Configuration Template (`application.yml`)
To activate automatic tracking and route tracing information across all microservice communication components, use this configuration:

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,info,metrics,prometheus" # Expose observability endpoints
  metrics:
    tags:
      application: ${spring.application.name} # Inject application context label
  tracing:
    sampling:
      probability: 1.0 # Capture 100% of traces (scale down in high-traffic production)
  otlp:
    tracing:
      endpoint: http://localhost:4318/v1/traces # Endpoint for OTel Collector or Tempo
```
