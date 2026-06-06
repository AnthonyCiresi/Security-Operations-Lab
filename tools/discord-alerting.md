# Discord Alerting Integration

[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)]() [![Language](https://img.shields.io/badge/Language-Python%203-blue?style=flat-square)]() [![Service](https://img.shields.io/badge/Managed-systemd-blue?style=flat-square)]()

---

## Overview

Custom Discord webhook integration that forwards Wazuh alerts to a dedicated
Discord channel in real time, simulating on-call SOC analyst paging. Implemented
as a Python script running as a systemd service on the Wazuh VM.

---

## Architecture

```
Wazuh alerts.json → alert_forwarder.py (systemd) → Discord Webhook → #wazuh-alerts channel
```

---

## Implementation

A Python script runs as a systemd service on the Wazuh VM, tailing the Wazuh
alerts JSON log and forwarding alerts meeting the severity threshold to Discord
via webhook embed. State is persisted between restarts to prevent alert replay.

| Component            | Details                                                                    |
| -------------------- | -------------------------------------------------------------------------- |
| **Script location**  | /opt/wazuh-discord/alert_forwarder.py                                      |
| **Service name**     | wazuh-discord.service                                                      |
| **Environment file** | /etc/wazuh-discord.env (mode 600)                                          |
| **State file**       | /opt/wazuh-discord/last_position                                           |
| **Alert threshold**  | Level 7+ (Medium, High, Critical)                                          |
| **Alerts file**      | /var/ossec/logs/alerts/alerts.json (symlink to Docker volume)              |
| **Log file**         | /var/log/wazuh-discord.log                                                 |

**Alerts symlink target:**
`/var/lib/docker/volumes/single-node_wazuh_logs/_data/alerts/alerts.json`

---

## Alert Format

Alerts are sent as Discord embeds with color-coded severity:

| Level | Severity    | Color  |
| ----- | ----------- | ------ |
| 13+   | 🔴 Critical | Red    |
| 10–12 | 🟠 High     | Orange |
| 7–9   | 🟡 Medium   | Yellow |

Each embed includes agent name, agent IP, rule ID, severity, MITRE ATT&CK
techniques (if mapped), source IP (if available), and affected file path
for FIM events.

---

## Rule Suppression

Low-value noisy rules are suppressed from Discord notifications. Suppressed
rules are still visible in the Wazuh dashboard — Discord suppression only
affects real-time paging. See [detection-tuning.md](../docs/detection-tuning.md)
for the full suppression policy and reasoning.

| Rule ID | Description |
| ------- | ----------- |
| 510     | Rootcheck anomaly detection (scheduled scan noise) |
| 2902    | dpkg package installed |
| 2903    | dpkg package purged |
| 2904    | dpkg package half-configured |
| 2905    | dpkg package half-installed |
| 5715    | PAM auth session opened |
| 31530   | Wazuh agent started |

---

## Why Not Native Wazuh Integration?

Wazuh's built-in integratord runs inside the Docker container, which uses a
different network identity that Discord's API rejects with 403 Forbidden.
The host-based Python script bypasses this by making webhook requests directly
from the VM where outbound HTTPS is permitted.

---

## Key Features

- **State persistence** — tracks file position across restarts, no duplicate alerts on service restart
- **Log rotation handling** — automatically resets position if alerts.json is rotated
- **Discord rate limit handling** — respects 429 responses and retries after the specified delay
- **Rule suppression list** — configurable set of noisy rule IDs to filter out
- **Graceful shutdown** — handles SIGTERM/SIGINT cleanly for systemd stop/restart
- **Resource limits** — capped at 128MB RAM and 10% CPU via systemd unit

---

## Service Management

```bash
# Check status
sudo systemctl status wazuh-discord

# Restart
sudo systemctl restart wazuh-discord

# View live logs
sudo journalctl -u wazuh-discord -f

# View last 50 log lines
sudo journalctl -u wazuh-discord -n 50
```

---

## Backlog Flood Prevention

After a Wazuh manager reset, historical SCA alerts will replay and flood
Discord. Fast-forward the state file to skip the backlog:

```bash
sudo systemctl stop wazuh-discord

sudo python3 -c "
import os
path = '/var/ossec/logs/alerts/alerts.json'
size = os.path.getsize(path)
open('/opt/wazuh-discord/last_position', 'w').write(str(size))
print(f'Position set to {size}')
"

sudo systemctl start wazuh-discord
```

---

## Environment Variables

| Variable              | Description                                 |
| --------------------- | ------------------------------------------- |
| `DISCORD_WEBHOOK_URL` | Full Discord webhook URL                    |
| `WAZUH_MIN_LEVEL`     | Minimum alert level to forward (default: 7) |

---

*Deployed on Wazuh SIEM VM (192.168.88.90) | Security Operations Lab*
