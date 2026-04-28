# Discord Bots

## Overview

Three self-developed Discord bots, each with a distinct function. Currently inactive but infrastructure is retained for reactivation.

## Containers

Each bot runs in its own LXC container for isolation. All containers are currently stopped.

| Container | Bot | Status |
|-----------|-----|--------|
| LXC (moderator) | Moderator bot | stopped |
| LXC (musicbot) | Music bot | stopped |
| LXC (bot) | Analytics bot | stopped |

## Bots

### Moderator Bot

Automated moderation for Discord servers.

- Content filtering with configurable keyword and pattern rules
- Automatic ban enforcement based on rule violations
- User management commands for manual moderator actions
- Audit logging for all moderation actions

### Music Bot

Audio playback in Discord voice channels.

- Queue-based playback
- Standard controls: play, pause, skip, stop, queue management
- Runs as a persistent process inside the container

### Analytics Bot

Server traffic analysis and reporting.

- Tracks message activity, channel usage, and member engagement over time
- Generates periodic reports posted to a designated channel
- Stores aggregated statistics locally — no data is sent to external services

## Design Decisions

**One container per bot.** Each bot is isolated in its own LXC container. This allows individual bots to be started, stopped, or updated without affecting the others.

**Self-developed, not third-party.** All bots are written and maintained independently. This gives full control over functionality, data handling, and update cadence.

**Currently inactive.** The containers are stopped but preserved. Reactivation requires starting the LXC containers — no reinstallation needed.
