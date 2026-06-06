# Wazuh SIEM

[![Version](https://img.shields.io/badge/Wazuh-v4.7.3-blue?style=flat-square)](https://documentation.wazuh.com/4.7/) [![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)]() [![Deployment](https://img.shields.io/badge/Deployment-Docker%20Compose-blue?style=flat-square)]()

---

## Overview

Wazuh is an open source Security Information and Event Management (SIEM)
platform deployed as the core security monitoring solution in this lab.
It provides real time threat detection, security configuration assessment,
file integrity monitoring, and compliance reporting across all monitored
endpoints.

---

## Architecture

| Component           | Role                                                                                               |
| ------------------- | -------------------------------------------------------------------------------------------------- |
| **Wazuh Agent**     | Installed on each monitored endpoint. Collects logs, monitors files, and ships data to the manager |
| **Wazuh Manager**   | The analysis engine. Receives agent data, runs detection rules, and triggers alerts                |
| **Wazuh Indexer**   | OpenSearch database that stores all events and makes them searchable                               |
| **Wazuh Dashboard** | Web UI for viewing alerts, dashboards, and investigation tools                                     |

**Data Flow:** `Endpoints → Agents → Manager → Indexer → Dashboard → Discord`

---

## Deployment Details

| Component             | Details                       |
| --------------------- | ----------------------------- |
| **Host VM**           | Ubuntu 24.04 LTS (wazuh-siem) |
| **VM IP**             | 192.168.88.90                 |
| **Deployment Method** | Docker Compose (single-node)  |
| **Wazuh Version**     | 4.7.3                         |
| **Dashboard URL**     | https://192.168.88.90         |

**Docker Compose Stack:**

| Container       | Purpose                             |
| --------------- | ----------------------------------- |
| wazuh.manager   | Event analysis and rule processing  |
| wazuh.indexer   | Log storage and search (OpenSearch) |
| wazuh.dashboard | Web UI and visualization            |

**Docker Compose files:** `~/wazuh-docker/single-node/`

---

## Monitored Endpoints

| Agent ID | Agent Name       | OS                  | IP             | Status    |
| -------- | ---------------- | ------------------- | -------------- | --------- |
| 001      | monitoring       | Debian 13 (Trixie)  | 192.168.88.246 | ✅ Active |
| 002      | windows-breakfix | Windows 11          | 192.168.88.91  | ✅ Active |
| 003      | pve              | Debian (Proxmox VE) | 192.168.88.244 | ✅ Active |
| 004      | pihole           | Ubuntu 24.04 LTS    | 192.168.88.250 | ✅ Active |

> **Note:** Agent 003 re-registered under its system hostname `pve` following a manager reset on 2026-06-02.

---

## Capabilities Enabled

| Capability                            | Description                                                              |
| ------------------------------------- | ------------------------------------------------------------------------ |
| **Security Configuration Assessment** | CIS Benchmark compliance scanning on all agents                          |
| **File Integrity Monitoring**         | Detects unauthorized file changes across endpoints                       |
| **Log Analysis**                      | Collects and correlates Windows Event Logs, syslog, and application logs |
| **Rootcheck**                         | Scans for rootkits and suspicious system configurations                  |
| **Vulnerability Detection**           | Identifies known CVEs on monitored systems                               |
| **MITRE ATT&CK Mapping**              | Automatically maps detected events to ATT&CK techniques                  |
| **Discord Alerting**                  | Real-time alert forwarding to Discord via Python systemd service         |

---

## Key Configuration Files

| File              | Location                                                                 | Purpose                         |
| ----------------- | ------------------------------------------------------------------------ | ------------------------------- |
| Docker Compose    | ~/wazuh-docker/single-node/docker-compose.yml                            | Stack definition                |
| Manager Config    | ~/wazuh-docker/single-node/config/wazuh_cluster/wazuh_manager.conf      | Manager ossec.conf (bind mount) |
| Agent Config      | /var/ossec/etc/ossec.conf                                                | Per-agent configuration         |
| Manager Rules     | /var/ossec/ruleset/rules/                                                | Detection rule definitions      |
| Alerts Log (host) | /var/lib/docker/volumes/single-node_wazuh_logs/_data/alerts/alerts.json | Raw alert output                |

---

## FIM Exclusions

The following paths are excluded from File Integrity Monitoring to reduce
noise from high-frequency system writes. All exclusions are documented with
justification — see [detection-tuning.md](../docs/detection-tuning.md) for
the full policy.

| Path | Agent | Reason |
| ---- | ----- | ------ |
| `/etc/pve` | pve | Entire Proxmox cluster state directory — RRD metrics, auth keys, cluster logs update constantly |
| `/etc/pihole/pihole-FTL.db` | pihole | SQLite query database, updated on every DNS query |
| `/etc/pihole/pihole-FTL.db-wal` | pihole | SQLite WAL journal file |
| `/etc/pihole/pihole-FTL.db-shm` | pihole | SQLite shared memory file |
| `/etc/pihole/versions` | pihole | Version tracking file, updated nightly by auto-upgrade |
| `/etc/resolv.conf` | all | Rewritten by systemd-resolved and DHCP regularly |
| `/etc/apt/apt.conf.d` | all | APT configuration directory, modified during package upgrades |
| `/usr/bin` | all | Binary directory, updated during package upgrades |

---

## Installation Notes

- Requires `vm.max_map_count=262144` set on host for OpenSearch to function
- SSL certificates generated via `generate-indexer-certs.yml` during initial setup
- Proxmox host required `lsb-release` package installed before agent deployment
- All Linux agents installed via `.deb` package (DEB amd64)
- Windows agent installed via PowerShell MSI deployment

---

## Troubleshooting

### wazuh-db / queue/db/wdb Socket Failure

**Symptoms:**
- Wazuh API shows as down in dashboard
- `docker compose logs` shows `ERROR: Cannot find 'queue/db/wdb'`
- `wazuh-db: ERROR: (1226): Error reading XML file 'etc/ossec.conf': (line 0)`
- All three containers show `Up` but alerts stop flowing

**Root Cause:** Corrupted or stale `.db` files in the wazuh_queue volume combined
with a malformed `ossec.conf` bind mount prevent wazuh-db from initializing its
socket, causing all dependent daemons to fail.

**Resolution — Full Stack Reset:**

```bash
cd ~/wazuh-docker/single-node

docker compose down

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

git checkout -- .

docker compose up -d
docker compose logs -f wazuh.manager
```

**Post-Reset:** Agents automatically re-enroll within 1-2 minutes. Agent IDs
may be reassigned during re-enrollment.

**Verify fix:**
```bash
docker compose exec wazuh.manager ls -la /var/ossec/queue/db/
curl -k -u admin:SecretPassword https://localhost:55000/ 2>/dev/null | python3 -m json.tool
docker compose exec wazuh.manager /var/ossec/bin/agent_control -l
```

---

## Related Findings

| ID | Title | Date |
| -- | ----- | ---- |
| [SCA-001](../findings/SCA-001-windows-password-policy.md) | Insufficient Windows Password Policy Configuration | 2026-06-01 |
| [SCA-002](../findings/SCA-002-windows-audit-policy.md) | Insufficient Windows Audit Policy Configuration | 2026-06-01 |

---

*Deployed and maintained by Anthony Ciresi | Security Operations Lab*
