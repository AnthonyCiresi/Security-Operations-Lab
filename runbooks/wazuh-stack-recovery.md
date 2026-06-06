# Runbook: Wazuh Stack Recovery

[![Type](https://img.shields.io/badge/Type-Runbook-blue?style=flat-square)]() [![Severity](https://img.shields.io/badge/Severity-High-red?style=flat-square)]()

---

## Purpose

Step-by-step recovery procedure for the Wazuh Docker Compose stack when the
wazuh-db daemon fails to initialize, causing the API and alert pipeline to go
down. Encountered and resolved on 2026-06-02.

---

## Symptoms

- Wazuh dashboard shows API status as down (red indicator)
- Alerts stop flowing — Discord notifications cease
- `docker compose ps` shows all three containers as `Up` but functionality is broken
- Logs show one or more of the following errors:

```
wazuh-db: ERROR: (1226): Error reading XML file 'etc/ossec.conf': (line 0)
wazuh-authd: INFO: Cannot find 'queue/db/wdb'. Waiting N seconds to reconnect.
wazuh-authd: ERROR: Unable to connect to socket 'queue/db/wdb'.
```

---

## Root Cause

The `wazuh-db` daemon reads `ossec.conf` on startup and initializes a Unix
socket at `/var/ossec/queue/db/wdb`. If either the config file is malformed
or the queue volume contains stale/corrupted `.db` files from a previous
unclean shutdown, wazuh-db fails silently and the socket is never created.
All daemons that depend on the socket (wazuh-authd, wazuh-remoted,
wazuh-analysisd) then fail to connect.

The `ossec.conf` in this deployment is a bind mount from:
`~/wazuh-docker/single-node/config/wazuh_cluster/wazuh_manager.conf`

The queue volume is: `single-node_wazuh_queue`

---

## Pre-Recovery Checklist

- [ ] Open Winbox or have console access to the network in case of connectivity issues
- [ ] Note the current time — agents will briefly show Disconnected during recovery
- [ ] Ensure you are SSH'd into the Wazuh VM (192.168.88.90) or have Proxmox console access

---

## Diagnosis Commands

Run these before attempting a fix to confirm the root cause:

```bash
cd ~/wazuh-docker/single-node

# Check container status
docker compose ps

# Check for the specific wdb errors
docker compose logs wazuh.manager 2>&1 | grep -E "wdb|ossec.conf|ERROR|CRITICAL" | tail -30

# Check if the socket exists (should show 'wdb' if healthy)
docker compose exec wazuh.manager ls -la /var/ossec/queue/db/

# Check ossec.conf bind mount source is not empty
wc -l ~/wazuh-docker/single-node/config/wazuh_cluster/wazuh_manager.conf
head -5 ~/wazuh-docker/single-node/config/wazuh_cluster/wazuh_manager.conf
```

---

## Recovery Procedure

> **Warning:** This is a full stack reset. All Docker volumes are removed and
> recreated from scratch. Agent keys are wiped from the manager — agents will
> automatically re-enroll within 1-2 minutes after the stack comes back up.
> No changes are needed on the agent machines.

### Step 1 — Stop the stack

```bash
cd ~/wazuh-docker/single-node
docker compose down
```

### Step 2 — Remove all Wazuh volumes

```bash
docker volume rm \
  single-node_wazuh_etc \
  single-node_wazuh_var \
  single-node_wazuh_queue \
  single-node_wazuh_logs \
  single-node_wazuh_agentless \
  single-node_wazuh_active_response \
  single-node_wazuh_integrations \
  single-node_wazuh_var_multigroups \
  single-node_wazuh_wodles \
  single-node_wazuh_api_configuration \
  single-node_wazuh-indexer-data \
  single-node_filebeat_etc \
  single-node_filebeat_var \
  single-node_wazuh-dashboard-config \
  single-node_wazuh-dashboard-custom
```

### Step 3 — Reset config files to stock

```bash
git checkout -- .
```

### Step 4 — Re-apply custom config changes

After a git reset the following customizations must be re-applied manually
to `~/wazuh-docker/single-node/config/wazuh_cluster/wazuh_manager.conf`:

- FIM exclusions in the `<syscheck>` block (see [wazuh.md](../tools/wazuh.md#fim-exclusions))
- Syslog forwarding config if MikroTik integration is active

### Step 5 — Bring the stack up

```bash
docker compose up -d
```

Watch initialization (takes 2-3 minutes):

```bash
docker compose logs -f wazuh.manager
```

Healthy startup looks like:
```
Started wazuh-apid...
Started wazuh-dbd...
Started wazuh-remoted...
Started wazuh-analysisd...
```

No `ERROR` lines following the started messages = healthy.

### Step 6 — Verify recovery

```bash
# wdb socket must exist
docker compose exec wazuh.manager ls -la /var/ossec/queue/db/ | grep wdb

# API must respond (Unauthorized = API is up, credentials just need JWT)
curl -k -u admin:SecretPassword https://localhost:55000/ 2>/dev/null | python3 -m json.tool

# All agents must show Active (allow 2-3 minutes for re-enrollment)
docker compose exec wazuh.manager /var/ossec/bin/agent_control -l
```

### Step 7 — Flush Discord alert backlog

After a manager reset, historical SCA alerts will replay and flood Discord.
Fast-forward the forwarder state file:

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
sudo systemctl status wazuh-discord
```

---

## Post-Recovery Checklist

- [ ] wdb socket exists at `/var/ossec/queue/db/wdb`
- [ ] Wazuh API returns a valid response
- [ ] All 4 agents show Active in agent_control
- [ ] Wazuh dashboard accessible at https://192.168.88.90
- [ ] Discord forwarder running and position fast-forwarded
- [ ] FIM exclusions re-applied to wazuh_manager.conf
- [ ] Test alert received in Discord

---

## Time Estimate

| Phase | Duration |
| ----- | -------- |
| Diagnosis | 2-5 minutes |
| Volume removal and reset | 2 minutes |
| Stack initialization | 2-3 minutes |
| Agent re-enrollment | 1-2 minutes |
| **Total** | **~10 minutes** |

---

*Security Operations Lab | Anthony Ciresi*
