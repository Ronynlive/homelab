# Homelab

Personal homelab running on Proxmox VE. This repository documents the architecture, services, and infrastructure decisions behind my self-hosted setup.

---

## Hardware

| Component | Details |
|-----------|---------|
| Device | Lenovo ThinkCentre M720q |
| CPU | Intel Core i5-8400T (6 cores, no hyperthreading) |
| RAM | 32 GB DDR4 |
| Primary disk | 477 GB SSD |
| Backup disk | 1.8 TB USB HDD |
| Form factor | Tiny PC, fanless-class, low power draw |

The ThinkCentre M720q was chosen for its low idle power consumption, small footprint, and sufficient single-thread performance for a home server workload.

---

## Hypervisor

**Proxmox VE 8.4** on Debian 12 (Bookworm), kernel 6.8.12-pve.

Proxmox manages all workloads as either QEMU/KVM virtual machines or LXC containers depending on the use case:

- **LXC** for lightweight, Linux-native services (proxies, monitoring, bots)
- **KVM** for services that benefit from full isolation or need a custom kernel (e.g. Docker-heavy workloads)

### Storage layout

| Pool | Type | Size | Purpose |
|------|------|------|---------|
| local | Directory | ~94 GB | ISO images, container templates, backups |
| local-lvm | LVM-thin | ~357 GB | VM and container disks |
| usb-backup | Directory (USB) | 1.8 TB | Automated off-pool backups via vzdump |

No ZFS is in use. The LVM-thin pool provides snapshot support and thin provisioning without the RAM overhead of ZFS.

---

## Network

Two Linux bridges are configured on the host:

- **vmbr0** — main LAN bridge, all production VMs and containers
- **vmbr1** — secondary bridge, isolated segment for internal-only traffic

All external access is routed through the reverse proxy. No ports are directly forwarded to individual services.

---

## Services

### Nginx Proxy Manager (LXC)
Central reverse proxy for all self-hosted services. Handles SSL termination via Let's Encrypt and routes traffic to internal services by hostname. No service is directly reachable from outside this proxy.

**Stack:** Docker, jc21/nginx-proxy-manager

---

### Immich (KVM)
Self-hosted photo and video management platform, replacing cloud-based photo services. Includes machine-learning-based face recognition and object detection running locally.

**Stack:** Docker Compose, immich-server, PostgreSQL (pgvectors), Redis (Valkey), immich-machine-learning

**Disk:** 200 GB dedicated VM disk

---

### Uptime Kuma (LXC)
Internal uptime monitoring and status page for all homelab services. Sends alerts on downtime.

**Stack:** Docker, louislam/uptime-kuma

---

### Monitoring Stack (LXC)
Full metrics pipeline for host and container observability.

- **Prometheus** — metrics collection and storage
- **Prometheus PVE Exporter** — scrapes Proxmox host and guest metrics
- **Prometheus Node Exporter** — per-host system metrics
- **Grafana** — dashboards for CPU, memory, disk, network per VM/CT

**Stack:** Native systemd services on Debian

---

### Fitness App (LXC)
Self-hosted web application for personal health tracking. Built and maintained as a development project.

**Stack:** Docker Compose, Next.js, Supabase (GoTrue auth), PostgreSQL 15

---

### Discord Bots (LXC, currently inactive)
Three self-developed Discord bots:

- **Moderator bot** — automated content filtering, user management, and ban enforcement
- **Music bot** — audio playback in voice channels
- **Analytics bot** — server traffic analysis and activity reporting

Currently stopped; infrastructure remains in place for reactivation.

---

## Backup

Automated daily backups via Proxmox `vzdump` to the USB backup pool.

- Format: `.vma.zst` (compressed Proxmox backup format)
- Retention: 3 daily snapshots per VM
- Currently covering: VM 106 (Immich) — largest and most critical data store

---

## Architecture Overview

```
Internet
    |
    v
Nginx Proxy Manager (LXC)
    |
    +-- Immich            (KVM,  Docker Compose)
    +-- Uptime Kuma       (LXC,  Docker)
    +-- Fitness App       (LXC,  Docker Compose)
    +-- Grafana           (LXC,  systemd)

Monitoring (LXC)
    |
    +-- Proxmox PVE Exporter  -> scrapes Proxmox API
    +-- Node Exporter         -> scrapes host metrics
    +-- Prometheus            -> stores all metrics
    +-- Grafana               -> visualises metrics
```

---

## Design Principles

**Everything behind a reverse proxy.** No service exposes ports directly. All routing goes through Nginx Proxy Manager with valid SSL certificates.

**Separation by isolation level.** Lightweight stateless services run as LXC containers. Docker-heavy or storage-intensive workloads run as full VMs.

**Backups on separate physical media.** The USB backup drive is physically separate from the SSD pool. A failure of the primary disk does not affect backups.

**Low footprint hardware.** The ThinkCentre M720q draws roughly 10–15W at idle. Running 24/7 this keeps electricity costs manageable while providing enough compute for all current workloads.

---

## To Do

- [ ] Extend backup coverage to all running containers
- [ ] Add off-site backup target (e.g. rclone to encrypted cloud storage)
- [ ] Make Uptime Kuma status page publicly accessible
- [ ] Document individual service configurations in `/services`
