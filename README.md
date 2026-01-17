# Observability
- Used for understanding state of an:
  -  application
  -  infra
  -  network
  Example: latency, https traffic and so on.

What? ==> Metrics (Historical data of the event)
==
  What is the disk utilization of a node in last 24 hours?
  Is it under utilized or contrainted by allocated capacity?
  What is the CPU usage? Memory and so on.
  Out of 1000 http requests, the percentage of failure and sucess?

Why  ==> Logging (info, error, debug)
==
  Why the 5 Http requests failed?
  Why there is a memory leak?

How  ==> Traces ( helps for troubleshooting)
==
  How to fix the issues?
  Complete http request tracing from client to backend pod.
  Can we get the complete traces of the request.

Monitoring (Sub part of observability)
==
Metrics -> alerts -> dashboards

Observability : It provides continious feedback.
==

Who immplements observability?
==
- Collective effort of developers and devops engineers
- Developers ensure the applications emits metrics, logs and traces
- Devops use Prometheus, ELK to scrap the logs, metrics and traces
- 

  
