# Uptime Kuma

## Overview

Self-hosted uptime monitoring and status page. Monitors all homelab services and sends alerts when something goes down.

## Container

| Setting | Value |
|---------|-------|
| Type | LXC |
| OS | Ubuntu |
| CPU | 1 core |
| RAM | 512 MB |
| Disk | 4 GB |

## Stack

| Component | Image |
|-----------|-------|
| Uptime Kuma | louislam/uptime-kuma:1 |
| Runtime | Docker |

## Ports

| Port | Purpose |
|------|---------|
| 3001 | Web UI (proxied via NPM) |

## What is Monitored

- Nginx Proxy Manager availability
- Immich web UI
- Grafana dashboard
- Fitness app
- All other proxied services

## Alerting

Kuma supports multiple notification channels. Alerts are configured to fire when a service fails to respond within the defined interval.

## Design Decisions

**Separate container from monitoring stack.** Uptime Kuma runs independently from Grafana and Prometheus. This means if the monitoring stack goes down, Kuma still functions and can alert on it.

**Minimal resource footprint.** 1 core and 512 MB RAM is sufficient. Kuma is a lightweight Node.js application with a built-in SQLite database — no external database required.
