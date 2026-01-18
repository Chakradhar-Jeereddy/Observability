# 🧠 PromQL One-Page Cheat Sheet (Kubernetes Focused)

---

## 🔹 Metric Types

| Type       | Meaning                | Example                  |
|------------|----------------------|-------------------------|
| Counter    | Always increases      | `*_total`               |
| Gauge      | Goes up & down        | memory, pods            |
| Histogram  | Buckets for latency   | `*_bucket`              |
| Summary    | Client-side percentiles | `*_quantile`          |

---

## 🔹 Core Functions (When to Use)

| Function           | Use When                     | Example                                      |
|-------------------|------------------------------|---------------------------------------------|
| `rate()`           | Speed per second             | CPU, RPS                                    |
| `increase()`       | Total events                 | Pod restarts                                |
| `sum()`            | Cluster / namespace view     | Total CPU                                   |
| `avg()`            | Typical (average) value      | Average node CPU                             |
| `topk()`           | Find worst offenders         | Top pods                                    |
| `histogram_quantile()` | p95 / p99 latency        | API latency                                 |

---

## 🔹 Common Kubernetes Queries

### Pod restart alert
```promql
increase(kube_pod_container_status_restarts_total[5m]) > 3
CPU usage per pod
```
promql
Copy code
sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)
High node memory usage
```
promql
Copy code
1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) > 0.9
Pod not ready
```
promql
Copy code
kube_pod_status_ready{condition="true"} == 0
🔹 Histogram / Latency Example
```
promql
Copy code
histogram_quantile(
  0.95,
  sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
)
Real user experience (p95 latency)
```
🔹 Label Rules (VERY IMPORTANT)

- Rule	Why
- Always aggregate (sum by)	Avoid alert storms
- Alert on pods/apps, not containers	Reduce noise
- Use time windows	Avoid flapping
- Avoid raw counters	Raw _total values are meaningless

🔹 Common Mistakes ❌
- Using rate() on gauges
- Alerting only on averages
- Missing by() clause → label explosion
- Using _bucket without histogram_quantile()
- Alerting on raw _total counters instead of increase()

🎯 Interview-ready Summary
“Counters use rate() or increase(), gauges use avg() or raw values, histograms use buckets with percentiles, and aggregation is mandatory to avoid alert noise.”

# ⚡ PromQL Ultra-Compact Flashcard

| Function           | When to Use                  | Quick Example                                      |
|-------------------|-----------------------------|---------------------------------------------------|
| `rate()`           | Counter, per-second speed    | `rate(container_cpu_usage_seconds_total[5m])`    |
| `increase()`       | Counter, total events       | `increase(kube_pod_container_status_restarts_total[5m])` |
| `sum()`            | Aggregate multiple series   | `sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)` |
| `avg()`            | Typical value, gauge        | `avg(container_memory_working_set_bytes) by (pod)` |
| `topk()`           | Find worst offenders        | `topk(5, sum(rate(container_cpu_usage_seconds_total[5m])) by (pod))` |
| `histogram_quantile()` | Percentiles / latency      | `histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))` |

---

### Quick Tips:
- **Always aggregate** (`sum by`) to reduce noise  
- **Use `increase()` for alerts** on counters  
- **Percentiles > average** for latency  
- **Include labels** (pod, namespace, app)  
- **Time windows** are mandatory for stability  

---

### Memory Trick:
> **rate = speed, increase = count, sum = all together, avg = typical, buckets = tail (p95/p99)**
