# Discord Alerting Integration

![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)

---

## Overview

Custom Discord webhook integration that forwards Wazuh alerts to a dedicated
Discord channel in real time, simulating on-call SOC analyst paging.

---

## Architecture
Wazuh alerts.json → discord-alert.sh → Discord Webhook → #wazuh-alerts channel

---

---

## Implementation

A bash script runs as a systemd service on the Wazuh VM, tailing the Wazuh
alerts JSON file and forwarding any alerts meeting the severity threshold to
Discord via webhook.

| Component | Details |
|---|---|
| **Script location** | /home/wazuh-siem/discord-alert.sh |
| **Service name** | wazuh-discord.service |
| **Alert threshold** | Level 7+ |
| **Alerts file** | /var/lib/docker/volumes/single-node_wazuh_logs/_data/alerts/alerts.json |

---

## Why Not Native Wazuh Integration?

Wazuh's built in integratord runs inside the Docker container which uses a
different network identity that Discord's API rejects with 403 Forbidden.
The host-based script approach bypasses this by making webhook requests
directly from the VM where outbound requests are permitted.

---

## Service Management

```bash
# Check status
sudo systemctl status wazuh-discord

# Restart
sudo systemctl restart wazuh-discord

# View logs
sudo journalctl -u wazuh-discord -n 50
```

---

*Deployed on Wazuh SIEM VM (192.168.88.90) | Security Operations Lab*
