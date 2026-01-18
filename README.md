# Observability
- Used for understanding state of an:
  -  application
  -  infra
  -  network
  Example: latency, https traffic and so on.

***What? ==> Metrics (Historical data of the event)**

  What is the disk utilization of a node in last 24 hours?
  Is it under utilized or contrainted by allocated capacity?
  What is the CPU usage? Memory and so on.
  Out of 1000 http requests, the percentage of failure and sucess?

***Why  ==> Logging (info, error, debug)**

  Why the 5 Http requests failed?
  Why there is a memory leak?

***How  ==> Traces ( helps for troubleshooting)**

  How to fix the issues?
  Complete http request tracing from client to backend pod.
  Can we get the complete traces of the request.

***Monitoring (Sub part of observability)**

Metrics -> alerts -> dashboards

***Observability : It provides continious feedback.**

***Who immplements observability?**

- Collective effort of developers and devops engineers
- Developers ensure the applications emits metrics, logs and traces
- Devops use Prometheus, ELK to scrap the logs, metrics and traces

```
alertmanager-monitoring-kube-prometheus-alertmanager-0
alertmanager-monitoring-kube-prometheus-alertmanager-1
monitoring-grafana-5f75f76f9b-lgrrq
monitoring-kube-prometheus-operator-7767f5fbb4-8vz72
monitoring-kube-state-metrics-7896ff97b9-pzd96
monitoring-prometheus-node-exporter-9chgp
monitoring-prometheus-node-exporter-vxdnl
prometheus-monitoring-kube-prometheus-prometheus-0
```
```
Nodes → node-exporter ┐
                      ├─> Prometheus ──> Alertmanager ──> Slack/Email
K8s API → kube-state ┘
                           │
                           └─> Grafana dashboards
```

- ***alertmanager-monitoring-kube-prometheus-alertmanager-0:** Statefulset to ensure no alerts are lost (2 pods, peristent storage and deos deduplication, grouping and send notifications)
- ***monitoring-grafana-5f75f76f9b-lgrrq:** Deployment for visualization, if down dashboard wont be available, promethus continues to collect metrics.
- ***monitoring-kube-prometheus-operator:** Manages the whole stack. Changes to any configuration will not apply if its down, and will sync once its up.
- ***monitoring-kube-state-metrics:** Collects the kube state events and converts them to metrics and exposes under /metrics.
- ***monitoring-prometheus-node-exporter:** Collects the events from nodes, its a daemonset runs on each node.
- ***prometheus-monitoring-kube-prometheus-prometheus:** Statefull set continoulsy scrape metrics (ensures no metrics are lost) and stores it in TSDB (Time services database).

Testing the metrics
==
kubectl apply -f crash.yaml
```
apiVersion: v1
kind: Pod
metadata:
  name: crash
  labels:
    run: crash
spec:
  restartPolicy: Never
  containers:
    - name: crash
      image: busybox
      command: ["/bin/sh", "-c", "exit 1"]
```
- Query promethus
  - kube_pod_container_status_restarts_total{namespace="monitoring",pod="crash"}
  - Set time to 1 min



| Function   | Metric type | Purpose |
|-----------|------------|---------|
| rate()    | Counter    | Per-second speed |
| increase()| Counter    | Total events over time |
| sum()     | Any        | Aggregate multiple series |
| avg()     | Gauge      | Typical (average) value |
| Buckets   | Histogram  | Percentiles (p95, p99) |


1. rate → how fast
2. increase → how many
3. sum → all together
4. avg → typical
5. buckets → tail latency


Instrument the metrics
==
- Expose application metrics
- **Metrics types:**
   - Counter -> Always increase like http requests, total pod restarts 
   - Guage -> Increase and also decrease (CPU, Memory usage)
   - Histogram -> Uses buckets to store information. (Find latency, API response more then 10m, 20m ..)
- How prometheus scrape the application metrics?
   - Uses service discovery yaml, tells prom to scrap metrics from this service.

Types of exporters
==
 - Kube-state-metrics: Kubernetes object state, reports actual vs desired state.
 - node-exporter: Node CPU/Memory/Disk.
 - cAdvisor: Container resource usage.
 - Prometheus: Scrap and store metrics in TSDB (/prometheus persistent storage).

Important metrics
==
1. Pod restarts
2. Pod phase
3. Node rediness
4. Resource limit/requests
5. Replica status
6. PVC status
7. HPA metrics

Top 10 prom alerts
==
1. kube_node_status_ready_condition{condition="Ready",status="true"} == 0
```
    Why -> Node is unhealthy, pod may be evicted
    Metric returns 0 or 1 per node.
    1 -> Node is Ready
    2 -> Node is NotReady
```
2. increase(kube_pod_container_status_restart_total[5m]) > 3
```
   3 restarts indicates real issue, less then 3 could be blip or noise.
   Kubelet increases delay between restarts upto 5 minutes (immediate, 5s,10s,20s,40s..upto 5mis).
   If pod is stable for 10mins, it resets the backoff.
   Reduce load on dependencies (DB) and on CPU and memory.
   Pod crashloopbackoff
   Why: App crashes, instability
        OOM/BadConfig/Dependency failure
```
3.  kube_pod_status_ready{condition="true"} == 0  (per pod)
```
   Pod not ready (Warnning -> Critical)
   why?: startup or probe failure.
```
4. kube_deoloyment_spec_replicas!=kube_deployment_spec_replicas_available
```
Why: rollout failure, bad config, image pull error
```
5. kube_node_status_condition{condition="DiskPressure",state="true"} == 1
```
Disk full
kubectl may evict pods
```
6. avg(rate(node_cpu_seconds_total{node!="idle"}[5m])) by (instance) > 0.85
```
High node CPU usage (Warning)
HPA may not scale
```
7. node_memory_MemAvailable_bytes/node_MemTotalBytes) < 0.10
```
High node memory usage
OOM killed
Node instability
```
8. kube_persistentVolumeClam_status_phases{pahse="Pending"} == 1
```
Per volume, returns one if its in pending state
storage class issue
capacity exhausted.
```
9. rate(apiserver_request_total(code=

```
```
10. etcd
| Metric                                      | What it measures                            | Histogram / Percentile | PromQL Example                                                                                            | Explanation / Why it matters                                                    | Suggested Alert Threshold    |
| ------------------------------------------- | ------------------------------------------- | ---------------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------- | ---------------------------- |
| `etcd_disk_wal_fsync_duration_seconds`      | Time to fsync WAL (write-ahead log) to disk | p95                    | `histogram_quantile(0.95, sum(rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])) by (le))`            | Measures how long it takes to persist WAL updates; high latency → slower writes | p95 > 0.5s                   |
| `etcd_disk_backend_commit_duration_seconds` | Time to commit a transaction to backend DB  | p99                    | `histogram_quantile(0.99, sum(rate(etcd_disk_backend_commit_duration_seconds_bucket[5m])) by (le))`       | Critical for cluster responsiveness; high commit latency slows Kubernetes API   | p99 > 1s                     |
| `etcd_server_watch_duration_seconds`        | Duration of watch RPC calls                 | p90                    | `histogram_quantile(0.9, sum(rate(etcd_server_watch_duration_seconds_bucket[5m])) by (le))`               | Watches are used by controllers and kubelets; high latency → delayed events     | p90 > 1s                     |
| `etcd_network_peer_round_trip_time_seconds` | Network RTT between etcd peers              | p95                    | `histogram_quantile(0.95, sum(rate(etcd_network_peer_round_trip_time_seconds_bucket[5m])) by (le, peer))` | Ensures cluster communication is fast; high RTT → slow quorum                   | p95 > 0.1s                   |
| `etcd_server_proposals_committed_total`     | Number of proposals committed               | Counter / rate         | `rate(etcd_server_proposals_committed_total[5m])`                                                         | Measures throughput of etcd; sudden drop → possible failure                     | Sudden drop to 0             |
| `etcd_server_has_leader`                    | Whether node has leader                     | Gauge                  | `etcd_server_has_leader == 0`                                                                             | Node without leader cannot commit writes                                        | == 0 triggers critical alert |

```
# High backend commit latency
alert: EtcdCommitHighLatency
expr: histogram_quantile(0.99, sum(rate(etcd_disk_backend_commit_duration_seconds_bucket[5m])) by (le)) > 1
for: 5m
labels:
  severity: warning
annotations:
  summary: "High etcd commit latency (p99 > 1s)"

# WAL fsync too slow
alert: EtcdWALFsyncHighLatency
expr: histogram_quantile(0.95, sum(rate(etcd_disk_wal_fsync_duration_seconds_bucket[5m])) by (le)) > 0.5
for: 5m
labels:
  severity: warning

# Node lost leader
alert: EtcdNoLeader
expr: etcd_server_has_leader == 0
for: 1m
labels:
  severity: critical
```
11. kube_node_status_condition{condition="MemoryPressure",state="true"} == 1
```
We can also find from kubectl describe node
```
12. kube_pod_container_status_waiting_reason(reason="imagePullBackoff")} == 1
```
why? -  image pull failure
```

Avoid these mistakes
==
- Alerting on raw CPU usage without duration
- Alerting per container instead of aggregation
- No severity level
- No SLA based alert

What is p95?
- You measure backend commit times for 1000 requests.
- 950 requests complete under 0.2s.
- 50 requests are slower (0.3–0.5s).
- Then p95 = 0.2s → 95% of requests are ≤ 0.2s.
