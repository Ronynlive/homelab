# Nginx Proxy Manager

## Overview

Central reverse proxy for all homelab services. Every service is accessed exclusively through this proxy — no ports are forwarded directly from the router to individual containers or VMs.

## Container

| Setting | Value |
|---------|-------|
| Type | LXC |
| OS | Debian 12 |
| CPU | 2 cores |
| RAM | 512 MB |
| Disk | 8 GB |

## Stack

| Component | Image |
|-----------|-------|
| Proxy | jc21/nginx-proxy-manager:latest |
| Runtime | Docker |

## Ports

| Port | Purpose |
|------|---------|
| 80 | HTTP (redirects to HTTPS) |
| 443 | HTTPS (all proxied services) |
| 81 | Admin web UI (internal only) |

## Design Decisions

**Single entry point.** All inbound traffic hits this container first. Individual services have no direct external exposure, which reduces the attack surface and simplifies firewall rules.

**Let's Encrypt via NPM.** SSL certificates are issued and renewed automatically through the NPM admin UI. Each proxied host gets its own certificate.

**Admin UI on port 81.** The management interface is not proxied externally — it is only accessible from within the local network.

## Notes

- Runs as a Docker container inside an LXC container (nested containerisation)
- A restart policy ensures NPM starts automatically after host reboots
- Proxy hosts are configured through the web UI, not config files
