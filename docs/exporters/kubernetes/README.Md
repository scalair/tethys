# kube-prometheus-stack

`kube-prometheus-stack` is a suit of tools to install into your Kubernetes cluster in order to monitor it.

`kube-prometheus-stack` is an helm chart that is maintained by [prometheus-community/helm-charts project](https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack).

## Connect to your Kubernetes cluster

First, connect to you Kubernetes cluster to install `kube-prometheus-stack` within.

## Configure the `kube-prometheus-stack`

Create a `kube-prometheus-stack.yml` file with the following content :

```yaml
nodeExporter:
  enabled: true
kubeStateMetrics:
  enabled: true
prometheus:
  enabled: true
alertmanager:
  enabled: false
grafana:
  enabled: false
```

**Ensure to have at least** : `nodeExporter.enabled: true`, `kubeStateMetrics.enabled: true` and `prometheus.enabled: true` to have the minimum required tools to monitor your Kubernetes cluster.

> Be free to configure any other parameters with the value you want.

## Deploy the `kube-prometheus-stack`

Run the following commands to deploy what we just configured before :

```yaml
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install --create-namespace -n monitoring -f kube-prometheus-stack.yml kube-prometheus-stack prometheus-community/kube-prometheus-stack --version 14.9.0
```
