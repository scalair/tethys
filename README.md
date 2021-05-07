# tethys

![Apache 2.0 Licence](https://img.shields.io/hexpm/l/plug.svg) ![Ansible](https://img.shields.io/badge/ansible-2.10.x-green.svg)

**tethys** aims to **facilitate the integration of monitoring stack** using the well-known [Prometheus](https://prometheus.io/), [Grafana](https://grafana.com/) and [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/), in order to **monitor several products** (node, kubernetes, apache, elasticsearch, etc).

We also provide a **[collection of Prometheus exporters](docs/exporters/README.md)** that collects metrics for several products.

Here is an overview :

![overview](docs/medias/overview.svg)

## Why tethys

Because we are used to integrating monitoring stacks for our clients, we have concluded several things :

- there are a ton of Prometheus exporters in the community, even for the same product. So, it is hard an boring to choose the good one. **We provide a list of Prometheus exporters for each product you want to monitor, that just works with our stack.**
- there are a ton of Grafana dashboars in the community, even for the same product. It is hard an boring to choose the good one. **We provide Grafana dashboards for each product you want to monitor, that just works with our stack.**
- glue between Prometheus exporters, Prometheus servers, Prometheus federation server, Grafana and Alertmanager is a complex task and requires a certain expertise to be maintained. **We've decided to add an abstraction layer to facilitate the integration and the glue.**
- because we had a monitoring stack for each customer which operated independently, it was becoming complicated on a daily basis to consult and maintain the stack of each customer. **tethys provides a simplified customer-centric view, and simplify multi-tenant configuration of the stack.**

For all these reasons, we decided to create tethys.

## Prerequisites

- ![ansible](https://img.shields.io/badge/ansible-2.10.x-green.svg)

## Usage

### `clients`

A list of clients to federate. Each client in `clients` has the following variables :

- `name` : define the client name that will be used accross all the stack to identify the client, and to add filtering on queries.
- `prometheus_federation`: a list of prometheuses servers to federate for this client. It allows to retrieve data metrics from specified prometheuses servers. Each federated prometheus in `prometheus_federation` has the following variables :
  - `name`: specify a name for this federated prometheus. It also adds a label `clusterID: <name>` for all metrics data retrieved from this prometheus server.
  - `endpoint`: specify the prometheus endpoint to federate. The endpoint must be available from the tethys deployed prometheus that federates (e.g. http(s)://\<url\>:\<port\>).
  - `username` : specify the username for basic_auth to request the endpoint.
  - `password` : specify the password for basic_auth to request the endpoint.
  - `kubernetes_hosted`: specify if the prometheus endpoint is hosted on Kubernetes. If `true`, then it will also retrieve metrics data and create dashboards for this Kubernetes cluster.
- `products`: a list of products the client has. It means the specified federated prometheus actually has metrics data for these products. For now, it only allows `node`, `kubernetes`, `elasticsearch` and `apache`.
- `prometheus_rules`: a list of custom prometheus rules based on [Prometheus 2.0 documentation](https://prometheus.io/docs/prometheus/latest/configuration/template_examples/), to create for this client. Please note to use `!unsafe` keyword as prefix for every string that uses dollar sign `$` to avoid templating. By default, `prometheus_rules` is empty and [standard rules](https://github.com/scalair/tethys/blob/dev/templates/prometheus/client.rules.j2) will be automatically applied for each defined `products`.

_Example usage:_

```yaml
clients:
- name: c1111
  prometheus_federation:
  - name: preprod
    endpoint: prometheus-preprod-c1111.eu-west-1.elb.amazonaws.com:80
    kubernetes_hosted: true
  - name: prod
    endpoint: https://prometheus-prod-c1111.eu-west-1.elb.amazonaws.com:80
    username: admin
    password: p4ssw0rd
    kubernetes_hosted: true
  products:
  - node
  - kubernetes
- name: c1337
  prometheus_federation:
  - name: prod
    endpoint: prometheus-preprod.eu-west-1.elb.amazonaws.com:80
    kubernetes_hosted: false
  products:
  - node
  prometheus_rules:
  - name: "c1337-node"
    rules:
    - alert: InstanceDown
      annotations:
        description: !unsafe '{{ $labels.instance }} of job {{ $labels.job }} has been down for more than 5 minutes.'
        summary: !unsafe "Instance {{ $labels.instance }} down"
      expr: 'up{clientID="c1111"} == 0'
      for: 5m
      labels:
        severity: critical
```

### `alertmanager`

For now, `alertmanager` only has `enabled` attribute. It specifies if alertmanager must be installed or not.

_Example usage :_

```yaml
alertmanager:
  enabled: true
```

### `grafana`

You can manage Grafana's `user` (default: `admin`) and `password` (default: `tethys`) by setting them.

_Example usage :_

```yaml
grafana:
  username: admin
  password: 4dm!npwd
```

### `reverse_proxy`

`reverse_proxy` has several variables :

- `enabled` (default: false): if true, then install Apache as reverse proxy in front of Prometheus, Grafana and Alertmanager applications.
- `certbot` (default: false) : if true, then install certbot in order to handle automatic SSL certificates for vhosts defined in `vhosts` variable (see `vhosts` below)
- `email` : specify the email address that will be used as contact for Apache vhosts, and Let's Encrypt notifications.
- `vhosts` is a list of vhosts to be installed into Apache reverse proxy. If `certbot` is true, then it also manage SSL certificates for the specified vhost domains. Each vhost in `vhosts` variable has the following attributes :
  - `name`: the vhost name
  - `domain`: the vhost domain helps to create the reverse proxy vhost, then manage SSL certificate for this domain regarding `certbot` value.
  - `backend_endpoint`: where to forward the request for this domain.
- `allow_list` (default: []): define a list of IP addresses or subnets that are allowed to access the entire vhosts. (Even with a `allow_list` defined, the `/.well-known/` URL is never restricted in order to allow Lets Encrypt to validate the SSL certificates).

_Example usage :_

```yaml
reverse_proxy:
  enabled: true
  certbot: true
  email: jdoe@foo.bar
  vhosts:
  - name: prometheus
    domain: prometheus.foo.bar
    backend_endpoint: localhost:9090
  - name: grafana
    domain: grafana.foo.bar
    backend_endpoint: localhost:3000
  - name: alertmanager
    domain: alertmanager.foo.bar
    backend_endpoint: localhost:9093
  allow_list:
  - 10.10.0.0/16
```

## Versioning

For the versions available, see the [tags on this repository](https://github.com/scalair/tethys/tags).

Additionaly you can see what change in each version in the [CHANGELOG.md](CHANGELOG.md) file.

## Authors

- **xat** - [xat](https://github.com/Xat59)
- **scalair** - [scalair](https://github.com/scalair)

## Contributing

Please read [CONTRIBUTING.md](.github/CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.
