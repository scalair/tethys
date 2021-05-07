# apache exporter

Prometheus exporter that exposes Apache mod_status statistics.

The exporter is maintained by [Lusitaniae/apache_exporter project](https://github.com/Lusitaniae/apache_exporter).

## Install on a standard instance

For a standard instance, we use an ansible role maintained by [Xat59/ansible-role-apache_exporter](https://github.com/Xat59/ansible-role-apache_exporter/), to deploy the exporter into an instance.

### Create the ansible playbook file

Create a `apache-exporter.yml` file with the following content :

```yaml
- hosts: <host>
  roles:
    - role: xat59.apache_exporter
      apache_exporter_option_scrape_uri: http://<server>/server-status?auto
```

> Replace _\<host\>_ with the host (or list of hosts) where to install the exporter.
Plus, replace _\<server\>_ with the DNS or IP of the Apache you want to monitor.

### Deploy it

```bash
ansible-galaxy install xat59.apache_exporter
ansible-playbook -vv apache-exporter.yml
```
