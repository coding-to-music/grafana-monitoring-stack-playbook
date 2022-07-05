# grafana-monitoring-stack-playbook

# 🚀 Javascript full-stack 🚀

https://github.com/coding-to-music/grafana-monitoring-stack-playbook

From / By https://github.com/45Drives/monitoring-stack

https://github.com/45Drives/monitoring-stack

Previous articles: https://github.com/coding-to-music/monitoring-stack

[Setting up an example Grafana dashboard which queries Prometheus for data. Set up a Prometheus data source and create a Prometheus Graph in a new Grafana Dashboard #211](https://github.com/coding-to-music/coding-to-music.github.io/issues/211)

[Cloud Home Base Setup - Data ingestion, processing, storage, display #213](https://github.com/coding-to-music/coding-to-music.github.io/issues/213)

[Installation, Starting and Stopping Grafana via systemd and init.d for Debian and Ubuntu #224](https://github.com/coding-to-music/coding-to-music.github.io/issues/224)

[Monitoring Stack: Grafana, Alertmanager, Prometheus, Node Exporter, ZnapZend Exporter, Ansible #279](https://github.com/coding-to-music/coding-to-music.github.io/issues/279)

[Telegraf, InfluxDB, and Grafana topology monitoring Cloudflare #300](https://github.com/coding-to-music/coding-to-music.github.io/issues/300)

[![Grafana screenshot](https://prometheus.io/assets/grafana_prometheus.png)](https://prometheus.io/assets/grafana_prometheus.png)

https://grafana.com/grafana/dashboards/

## Restart grafana-server:

```
sudo systemctl restart grafana-server
```

## Environment variables:

```java

```

## GitHub

```java
git init
git add .
git remote remove origin
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:coding-to-music/grafana-monitoring-stack-playbook.git
git push -u origin main
```

## Add redis integration for Grafana cloud

https://codingtomusic.grafana.net/a/grafana-easystart-app/redis

```
sudo ARCH=amd64 GCLOUD_STACK_ID="389534" GCLOUD_API_KEY="secret-goes-here" GCLOUD_API_URL="https://integrations-api-us-central.grafana.net" /bin/sh -c "$(curl -fsSL https://raw.githubusercontent.com/grafana/agent/release/production/grafanacloud-install.sh)"
```

Optional configurations

We strongly recommend that you configure a separate user for the Agent, and give it only the strictly mandatory security privileges necessary for monitoring your node, as per the official documentation.

Making sure to change the `redis_addr` to the address of the Redis server you want to monitor in the agent config:

```
integrations:
  redis_exporter:
    enabled: true
    redis_addr: "redis:6379"
```

For a full description of configuration options, including guidance on monitoring more than one Redis server, see `redis_exporter_config` in the Agent documentation.

After installation, the Agent config is stored in /etc/grafana-agent.yaml. Restart the agent for any changes to take effect:

```
sudo systemctl restart grafana-agent.service
```

# Monitoring Stack

| Service Name      | Description                                                     |
| ----------------- | --------------------------------------------------------------- |
| Grafana           | Display Statistics & Metrics from database                      |
| Alertmanager      | Query db of metrics and send alerts based on user defined rules |
| Prometheus        | Collect and store metrics scraped from exporters in database    |
| Node Exporter     | Export hardware and OS metrics via http endpoint                |
| ZnapZend Exporter | Export state information on zfs snapshots and replication tasks |

The services outlined above are deployed as containers using either podman or docker depending on Host OS.
Containers are managed via systemd services and/or cockpit-podman module

# Installation

- Clone git repo to "/usr/share"

```sh
cd /usr/share/
git clone https://github.com/45drives/monitoring-stack.git

```

- Included inventory file "hosts" has two groups "metrics" and "exporters"

  - All hosts in the "metrics" group will have prometheus, alertmanager and grafana installed
  - All hosts in the "exporters" group will have node_exporter and znapzend_exporter installed
  - By default "metrics" and "exporters" is populated by localhost. This is sufficient for a single server deployment.
    - To add multiple servers add new hosts in the "exporters" group
    - It is possible to have the metric stack not run on the same server as the exporter services.

- Configure email send/recieve setting for alertmanager in "group_vars/metrics.yml"

- Default ports are defined in the table below, they can be changed in metrics.yml or exporters.yml

| Default Setting          | Value |
| ------------------------ | ----- |
| Prometheus Port          | 9091  |
| Alertmanager Port        | 9093  |
| Grafana Port             | 3456  |
| Grafana Default User     | admin |
| Grafana Default Password | admin |
| Node Exporter Port       | 9100  |
| Znapzend Port            | 9101  |

- Run metrics playbook

```sh
cd /usr/share/monitoring-stack
sudo ansible-playbook -i hosts deploy-monitoring.yml
```

- To uninstall monitoring stack

```sh
ansible-playbook -i hosts purge-monitoring.yml
```

# Verification

To ensure monitoring stack is working as expected, simulate failure condition and you will recieve an email notification

- Offline a disk in your zpool
- Set disk as "Offline" in Houston UI, "ZFS + File Sharing"
- Or in cli: zpool offline tank 1-1
- After ~30 seconds you should see email with subject line "[FIRING:1] ZpoolDegradedState ($HOSTNAME node warning degraded $POOL_NAME)"
