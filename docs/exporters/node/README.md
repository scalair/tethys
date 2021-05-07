# node exporter

Prometheus exporter for hardware and OS metrics exposed by *nix kernels.

The exporter is maintained by [prometheus/node_exporter project](https://github.com/prometheus/node_exporter).

We provides two ways to install the node exporter :

- on a standard instance (on-prem, cloud, etc)
- on Kubernetes cluster nodes

## Install on a standard instance

For a standard instance, we use an ansible role maintained by [cloudalchemy/ansible-node-exporter](https://github.com/cloudalchemy/ansible-node-exporter), to deploy the exporter into an instance.

### Create the ansible playbook file

Create a `node-exporter.yml` file with the following content :

```yaml
- hosts: <host>
  roles:
    - role: cloudalchemy.node-exporter
```

> Replace _\<host\>_ with the host (or list of hosts) where to install the exporter.

### Deploy it

```bash
ansible-galaxy install cloudalchemy.node-exporter
ansible-playbook -vv node-exporter.yml
```

## Install on Kubernetes cluster nodes

If you've already installed the [monitoring for Kubernetes](../kubernetes/README.md) in your Kubernetes cluster, so your nodes are probably already monitored.
