# Detection Tuning Policy

[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)]() [![Last%20Updated](https://img.shields.io/badge/Last%20Updated-2026--06--05-blue?style=flat-square)]()

---

## Overview

Detection tuning is the process of deliberately suppressing known-good,
low-value alerts so that genuine threats are visible and actionable. This
document records every suppression and exclusion decision made in this lab,
along with the reasoning behind each one.

The guiding principle: **suppress noise, never suppress risk.** Every entry
below represents a deliberate decision, not a blanket silence.

---

## Discord Alert Suppression (Rule IDs)

These rules are suppressed from Discord real-time notifications only. All
events are still indexed in Wazuh and visible in the dashboard for periodic
review.

Suppression is configured in `/opt/wazuh-discord/alert_forwarder.py` on the
Wazuh SIEM VM.

| Rule ID | Description | Reason for Suppression |
| ------- | ----------- | ---------------------- |
| 510 | Host-based anomaly detection (rootcheck) | Fires in bursts every few hours as part of the scheduled rootcheck scan cycle across all agents. Low signal-to-noise ratio for real-time paging. Review rootcheck findings periodically in the dashboard instead. |
| 2902 | dpkg package installed | Fires during nightly unattended-upgrades on Ubuntu/Debian agents. Expected behavior. |
| 2903 | dpkg package purged | Fires during package removal as part of upgrade cycle. Expected behavior. |
| 2904 | dpkg package half-configured | Mid-install state during dpkg operations. Always resolves immediately. |
| 2905 | dpkg package half-installed | Mid-install state during dpkg operations. Always resolves immediately. |
| 5715 | PAM auth session opened | High volume on all Linux agents from routine system processes. |
| 31530 | Wazuh agent started | Informational only. Fires on every agent restart. |

**Rules intentionally NOT suppressed:**
- Rule 550 (FIM integrity checksum changed) — kept active for unexpected file changes
- Rule 533 (File deleted) — kept active, unexpected deletions are high value
- Any rule 10+ severity — always paged regardless of category

---

## FIM Exclusions (ossec.conf)

These paths are excluded from File Integrity Monitoring in the Wazuh manager
config at `~/wazuh-docker/single-node/config/wazuh_cluster/wazuh_manager.conf`.

FIM scans every 12 hours by default. Excluded paths would otherwise generate
hundreds of 550 alerts per scan cycle with no actionable signal.

### Proxmox (Agent: pve)

| Path | Reason |
| ---- | ------ |
| `/etc/pve` | Entire Proxmox cluster state directory. Contains RRD performance metrics (`.rrd`), cluster version counter (`.version`), cluster event log (`.clusterlog`), node resource manager heartbeat (`lrm_status`), and auth key pairs that rotate periodically. All managed exclusively by Proxmox — no external process should modify these. |

### Pihole (Agent: pihole)

| Path | Reason |
| ---- | ------ |
| `/etc/pihole/pihole-FTL.db` | SQLite database storing DNS query history. Updated continuously on every DNS query processed. |
| `/etc/pihole/pihole-FTL.db-wal` | SQLite Write-Ahead Log. Written alongside every database transaction. |
| `/etc/pihole/pihole-FTL.db-shm` | SQLite shared memory file. Updated with every WAL write. |
| `/etc/pihole/versions` | Tracks installed vs available pihole component versions. Updated nightly by the pihole update checker. |

### All Agents

| Path | Reason |
| ---- | ------ |
| `/etc/resolv.conf` | DNS resolver configuration. Rewritten by systemd-resolved and DHCP client on network changes. |
| `/etc/apt/apt.conf.d` | APT configuration directory. Modified during package upgrades (e.g. Proxmox injects proxy config files here). |
| `/usr/bin` | System binary directory. Updated during package installs and upgrades. The nightly unattended-upgrades process installs Python package binaries here, generating dozens of 550 alerts per cycle. |

---

## Review Schedule

| Item | Frequency | Notes |
| ---- | --------- | ----- |
| Rootcheck findings (rule 510) | Weekly | Review in Wazuh dashboard — Security Configuration Assessment module |
| FIM alerts in dashboard | Weekly | Review all 550s in dashboard even if suppressed from Discord |
| Suppression list audit | Monthly | Verify each suppressed rule is still justified |
| New rule additions | As needed | Document here before suppressing |

---

## What Gets Alerted (Never Suppressed)

For reference, the following categories always generate Discord notifications
regardless of volume:

- Any alert level 10+ (High / Critical)
- FIM changes to `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`
- Authentication failures and brute force attempts
- New user creation or privilege escalation
- Malware or rootkit detection
- MITRE ATT&CK technique matches

---

*Security Operations Lab | Anthony Ciresi*
