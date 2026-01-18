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

