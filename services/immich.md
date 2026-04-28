# Immich

## Overview

Self-hosted photo and video management platform. Used as a full replacement for cloud-based photo services. Photos and videos are stored locally and never leave the network.

## Virtual Machine

| Setting | Value |
|---------|-------|
| Type | KVM (QEMU) |
| OS | Linux (l26 kernel) |
| CPU | 2 cores |
| RAM | 5722 MB (~5.6 GB) |
| Disk | 200 GB (LVM-thin, iothread enabled) |

A full VM was chosen over LXC here because Immich runs multiple Docker containers including a machine learning service that benefits from full kernel isolation.

## Stack

| Component | Image | Purpose |
|-----------|-------|---------|
| immich-server | ghcr.io/immich-app/immich-server:v2 | Main API and web UI |
| immich-machine-learning | ghcr.io/immich-app/immich-machine-learning:v2 | Face and object recognition |
| immich-postgres | ghcr.io/immich-app/postgres:14-vectorchord | Database with pgvector extension |
| immich-redis | valkey/valkey:9 | Cache and job queue |

## Ports

| Port | Purpose |
|------|---------|
| 2283 | Web UI and API (proxied via NPM) |

## Features in Use

- Photo and video upload via mobile app and web UI
- Automatic face recognition and person grouping (runs locally via ML container)
- Object and scene detection
- Timeline and album organisation
- Original file preservation — no re-encoding or quality loss

## Design Decisions

**pgvectors / vectorchord extension.** Immich uses a custom Postgres image with the `pgvectors` and `vectorchord` extensions for storing and querying ML embeddings (face vectors, CLIP embeddings). This enables fast similarity search directly in the database.

**Valkey instead of Redis.** Immich ships with Valkey (a Redis fork) as the default cache and job queue backend. No changes were made to the default compose configuration here.

**iothread enabled.** The VM disk uses `iothread=1` in the Proxmox config, which gives the disk its own I/O thread and improves throughput for the large media library.

**Backup priority.** This VM is the only one with automated daily vzdump backups, as it holds the largest and least reproducible dataset (the photo library).

## Backup

Backed up daily via Proxmox vzdump to the USB backup pool. Three daily snapshots are retained before rotation.
