# 📊 Prometheus, PromQL & Grafana Observability Cheatsheet

A concise, practical reference guide for **Prometheus metrics, PromQL query syntax, Grafana alerting, 4 Golden Signals, RED/USE monitoring methods, and production best practices**.

---

## 1. 🏗️ Observability Architecture & Core Concepts

| Concept | Description |
|---|---|
| **Pull Model** | Prometheus scrapes HTTP `/metrics` endpoints at configurable scrape intervals (e.g. `15s`). |
| **Labels & Cardinality** | Key-value pairs attached to time series. *Warning:* High cardinality (e.g. user IDs in labels) explodes memory usage. |
| **Instant Vector** | A single value per time series at a specific timestamp (e.g., `http_requests_total`). |
| **Range Vector** | A set of time series containing a range of data points over time (e.g., `http_requests_total[5m]`). |

---

## 2. 📈 Prometheus Metric Types

| Type | Description | Best Functions | Example Metric |
|---|---|---|---|
| **Counter** | Cumulative metric that only increases or resets to 0 on restart. | `rate()`, `irate()`, `increase()` | `http_requests_total`, `process_cpu_seconds_total` |
| **Gauge** | Value that can go up or down arbitrary amounts. | `avg_over_time()`, `max_over_time()`, `delta()` | `node_memory_active_bytes`, `active_connections` |
| **Histogram** | Samples observations (durations/sizes) into configurable buckets. | `histogram_quantile()` | `http_request_duration_seconds_bucket` |
| **Summary** | Calculates client-side quantiles over a sliding time window. | Direct quantile label access | `rpc_duration_seconds{quantile="0.95"}` |

---

## 3. 🔍 PromQL Query Cheat Sheet

### Basic Selectors & Filtering

```promql
# Exact label match
http_requests_total{status="200", method="POST"}

# Regex label matching (=~ match, !~ not match)
http_requests_total{status=~"5.."}
http_requests_total{environment=~"prod|staging"}

# Exclude specific label value
http_requests_total{handler!="/healthz"}
```

### Rates & Increases

```promql
# Per-second average rate over 5 minutes (handles counter resets)
rate(http_requests_total[5m])

# Instantaneous per-second rate (best for spiky/volatile counters)
irate(http_requests_total[2m])

# Total absolute count increase over 1 hour
increase(http_requests_total[1h])
```

### Aggregations & Grouping

```promql
# Sum rate across all instances grouped by status code
sum(rate(http_requests_total[5m])) by (status)

# Average memory utilization % by cluster
avg(node_memory_MemFree_bytes / node_memory_MemTotal_bytes) by (cluster) * 100

# Top 5 endpoints by request volume
topk(5, sum(rate(http_requests_total[5m])) by (path))
```

---

## 4. 🧮 Advanced PromQL & Percentiles

### Calculating Latency Percentiles (p95 / p99)

```promql
# p95 request duration across all job targets
histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))

# p99 request duration grouped by service and handler
histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le, service, handler))
```

### Error Rates & Percentages

```promql
# HTTP 5xx error percentage rate over total requests
sum(rate(http_requests_total{status=~"5.."}[5m]))
/
sum(rate(http_requests_total[5m])) * 100
```

### Absence & Liveness Detection

```promql
# Alert if a service target stops reporting metrics
absent(up{job="api-service"} == 1)
```

---

## 5. 🎯 Monitoring Frameworks: RED & USE & 4 Golden Signals

### The 4 Golden Signals (Google SRE Handbook)

1. **Latency**: Time taken to service a request (differentiate successful vs failed requests).
2. **Traffic**: Demand on system (e.g. HTTP requests per sec, active socket connections).
3. **Errors**: Rate of failed requests (explicit HTTP 5xx or implicit timeouts).
4. **Saturation**: How full the service is (CPU, Memory, Disk I/O queues, Database connection pool).

### The RED Method (APIs & Microservices)

- **R**ate: `sum(rate(http_requests_total[5m]))`
- **E**rrors: `sum(rate(http_requests_total{status=~"5.."}[5m]))`
- **D**uration: `histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))`

### The USE Method (Infrastructure & Server Resources)

- **U**tilization: % of time resource was busy (e.g. CPU `100 - (avg(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)`).
- **S**aturation: Amount of extra queued work (e.g. System load average per CPU core).
- **E**rrors: Count of hardware/driver error events (e.g. Network drop packets `rate(node_network_receive_drop_total[5m])`).

---

## 6. 🚨 Prometheus Alert Rules & Grafana Setup

### Sample Prometheus Alert Rule (`alerts.yml`)

```yaml
groups:
  - name: ProductionAlerts
    rules:
      - alert: HighHTTPErrorRate
        expr: (sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m]))) * 100 > 5
        for: 2m
        labels:
          severity: critical
          team: devops
        annotations:
          summary: "High 5xx error rate on {{ $labels.instance }}"
          description: "Error rate is {{ $value | printf \"%.2f\" }}% over the last 5 minutes."

      - alert: HighMemoryUsage
        expr: (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / node_memory_MemTotal_bytes * 100 > 90
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Host node memory usage exceeds 90%"
```

---

## 7. 💡 Production Best Practices & Anti-Patterns

- ❌ **Avoid High-Cardinality Labels**: Do NOT add `user_id`, `email`, `uuid`, or raw timestamp values as Prometheus label keys.
- ✅ **Use `rate()` before `sum()`**: Always calculate rate per time series first, then aggregate: `sum(rate(x[5m]))` (NOT `rate(sum(x)[5m])`).
- ✅ **Histogram Bucket Granularity**: Set buckets that tightly envelope SLA boundaries (e.g., `[0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5, 10]`).
- ✅ **Alert on Symptoms over Causes**: Prefer alerting on high user-facing latency or error rate breaches over high CPU spikes.
