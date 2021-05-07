# node exporter

Prometheus exporter for various metrics about ElasticSearch.

The exporter is maintained by [justwatchcom/elasticsearch_exporter project](https://github.com/justwatchcom/elasticsearch_exporter).

We provides two ways to install the elasticsearch exporter :

- on a standard instance (on-prem, cloud, etc)
- on Kubernetes cluster nodes

## Install on a standard instance

For a standard instance, we use an ansible role maintained by [Lyr/ansible-elasticsearch-exporter](https://github.com/Lyr/ansible-elasticsearch-exporter), to deploy the exporter into an instance.

### Create the ansible playbook file

Create a `elasticsearch-exporter.yml` file with the following content :

```yaml
- hosts: <host>
  roles:
    - role: lyr.elasticsearch_exporter
      elasticsearch_exporter_es_uri: "<scheme>://<username>:<password>@<url>:<port>"
      elasticsearch_exporter_es_all: true
      elasticsearch_exporter_es_indices: true
      elasticsearch_exporter_es_shards: true
```

Of course, the elasticsearch endpoint must be available from the exporter.

> Replace _\<host\>_ with the host (or list of hosts) where to install the exporter.
Plus, replace _\<scheme\>_, _\<username\>_, _\<password\>_, _\<url\>_ and _\<port\>_ with your elasticsearch cluster information.

### Deploy it into the instance

```bash
ansible-galaxy install lyr.elasticsearch_exporter
ansible-playbook -vv elasticsearch-exporter.yml
```

## Install on Kubernetes cluster nodes

To install the exporter in a Kubernetes cluster, do the following.

### Connect to your Kubernetes cluster

First, connect to your Kubernetes cluster you want to deploy the exporter in.

### Configure the exporter

Create the `ekasticsearch-exporter.yml` file with the following content :

```yaml
es:
  all: true
  cluster_settings: true
  indices: true
  indices_settings: true
  shards: true
  snapshots: true
  uri: <scheme>://<username>:<password>@<url>:<port>
 
# If you are using prometheus-operator, add the following :
service:
  annotations: {}
  httpPort: 9108
  labels:
    job: elasticsearch-exporter
  metricsPort:
    name: http
  type: ClusterIP
serviceMonitor:
  enabled: true
  interval: 60s
  labels:
    prometheus-scrape: "true"
  metricRelabelings: []
  relabelings:
    - sourceLabels: [ "job" ]
      regex: ^.*$
      action: replace
      replacement: elasticsearch-exporter
      targetLabel: job
  sampleLimit: 0
  scheme: http
  scrapeTimeout: 10s
  targetLabels:
    - job
```

> Replace _\<scheme\>_, _\<username\>_, _\<password\>_, _\<url\>_ and _\<port\>_ with your elasticsearch cluster information.
Even if _\<scheme\>_ is _http_, it is mandatory to specify it.

### Deploy it into your Kubernetes cluster

```yaml
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm upgrade --install --create-namespace -f elasticsearch-exporter.yml -n monitoring elasticsearch-exporter prometheus-community/prometheus-elasticsearch-exporter --version 4.4.0
```
