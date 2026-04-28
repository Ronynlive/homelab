# Fitness App

## Overview

Self-hosted web application for personal health and fitness tracking. Built as a development project to practice full-stack development with a modern stack. All data stays local.

## Container

| Setting | Value |
|---------|-------|
| Type | LXC |
| OS | Ubuntu |
| CPU | 2 cores |
| RAM | 2048 MB |
| Disk | 20 GB |

## Stack

| Component | Image / Version | Purpose |
|-----------|----------------|---------|
| App | hashimoto-app:latest | Next.js frontend and API routes |
| Supabase Auth | supabase/gotrue:v2.132.3 | JWT-based authentication |
| PostgreSQL | supabase/postgres:15.1.0.117 | Primary database |
| Runtime | Docker Compose | Orchestration |

## Ports

| Port | Purpose |
|------|---------|
| 3000 | Web UI (proxied via NPM) |
| 9999 | GoTrue auth API (internal) |
| 5432 | PostgreSQL (internal) |

## Design Decisions

**Supabase as self-hosted backend.** Instead of using the Supabase cloud service, the individual Supabase components (GoTrue for auth, Postgres with extensions) are run locally via Docker Compose. This avoids any data leaving the local network.

**Next.js for frontend and API.** API routes handle server-side logic within the same Next.js application, keeping the stack simple without a separate backend service.

**LXC over KVM.** Unlike Immich, this application does not require ML workloads or a custom kernel. LXC is sufficient and more resource-efficient for a standard Docker Compose application.

## Notes

- The application is only accessible through the reverse proxy
- Authentication is handled by GoTrue (Supabase's auth service), not the application itself
- The PostgreSQL instance is not exposed outside the container network
