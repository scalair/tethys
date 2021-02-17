# tethys

Ansible role that creates monitoring stack based on the well-known following products :

- prometheus
- grafana
- alertmanager

The resulting Prometheus server is planned to federate others Prometheuses servers. [More information about Prometheus federation](https://prometheus.io/docs/prometheus/latest/federation/).

## Why tethys

- simplify Prometheuses servers federation
- add glue between Prometheus, Grafana and AlertManager in order to get a stack that just work from scratch
- provides custom dashboards for well-known products, such as :
  - node : monitor any VM/instances (cloud-based or on-prem)
  - elasticsearch : monitor elasticsearch clusters
  - kubernetes : monitor k8s clusters, and its workloads
- simplify multi-tenant configuration
