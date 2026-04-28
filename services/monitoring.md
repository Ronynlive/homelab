# Monitoring Stack

## Overview

Full observability stack for the homelab. Collects metrics from the Proxmox host, all LXC containers, and virtual machines. Visualised through Grafana dashboards.

## Container

| Setting | Value |
|---------|-------|
| Type | LXC |
| OS | Debian 12 |
| CPU | 2 cores |
| RAM | 2048 MB |
| Disk | 10 GB |

## Components

| Component | Version | Purpose |
|-----------|---------|---------|
| Prometheus | 2.42.0 | Metrics collection and time-series storage |
| Prometheus Node Exporter | latest | Host-level system metrics (CPU, RAM, disk, network) |
| Prometheus PVE Exporter | latest | Proxmox-specific metrics via the PVE API |
| Grafana | 13.0.1 | Dashboard and visualisation frontend |

All components run as native systemd services — no Docker involved in this container.

## Ports

| Port | Service |
|------|---------|
| 9090 | Prometheus UI |
| 3000 | Grafana web UI (proxied via NPM) |
| 9100 | Node Exporter (scraped by Prometheus) |

## Prometheus Scrape Configuration

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'pve'
    static_configs:
      - targets: ['<proxmox-host>']

  - job_name: 'node_lxc'
    static_configs:
      - targets: [<per-container node exporters>]
```

## What is Monitored

- Proxmox host: CPU, memory, storage pool usage, VM/CT status
- Per-container: CPU, memory, disk I/O, network throughput
- Prometheus itself (self-monitoring)

## Design Decisions

**Native systemd instead of Docker.** Running Prometheus and Grafana as systemd services avoids the overhead of a container-in-container setup and simplifies log management (`journalctl`).

**PVE Exporter for Proxmox metrics.** The `prometheus-pve-exporter` uses the Proxmox API to expose per-VM and per-CT metrics. This gives full visibility into the hypervisor layer without needing agents inside each guest.

**Grafana 13.** Installed from the official Grafana APT repository to stay on a current version rather than the Debian-packaged version, which lags significantly.

**Separate from Uptime Kuma.** The monitoring stack focuses on metrics and dashboards. Uptime Kuma handles alerting on service availability. The two are intentionally decoupled.
