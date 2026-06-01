# Wazuh SIEM

![Version](https://img.shields.io/badge/Wazuh-v4.7.3-blue?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)
![Deployment](https://img.shields.io/badge/Deployment-Docker%20Compose-blue?style=flat-square)

---

## Overview

Wazuh is an open source Security Information and Event Management (SIEM) 
platform deployed as the core security monitoring solution in this lab. 
It provides real time threat detection, security configuration assessment, 
file integrity monitoring, and compliance reporting across all monitored 
endpoints.

---

## Architecture

| Component | Role |
|---|---|
| **Wazuh Agent** | Installed on each monitored endpoint. Collects logs, monitors files, and ships data to the manager |
| **Wazuh Manager** | The analysis engine. Receives agent data, runs detection rules, and triggers alerts |
| **Wazuh Indexer** | OpenSearch database that stores all events and makes them searchable |
| **Wazuh Dashboard** | Web UI for viewing alerts, dashboards, and investigation tools |

**Data Flow:** `Endpoints → Agents → Manager → Indexer → Dashboard`
---

## Deployment Details

| Component | Details |
|---|---|
| **Host VM** | Ubuntu 24.04 LTS (wazuh-siem) |
| **VM IP** | 192.168.88.90 |
| **Deployment Method** | Docker Compose (single-node) |
| **Wazuh Version** | 4.7.3 |
| **Dashboard URL** | https://192.168.88.90 |

**Docker Compose Stack:**
| Container | Purpose |
|---|---|
| wazuh.manager | Event analysis and rule processing |
| wazuh.indexer | Log storage and search (OpenSearch) |
| wazuh.dashboard | Web UI and visualization |

---

## Monitored Endpoints

| Agent ID | Agent Name | OS | IP | Status |
|---|---|---|---|---|
| 001 | windows-breakfix | Windows 11 | 192.168.88.91 | ✅ Active |
| 002 | proxmox-host | Debian (Proxmox VE) | 192.168.88.244 | ✅ Active |
| 003 | pihole | Ubuntu 24.04 LTS | 192.168.88.250 | ✅ Active |
| 004 | monitoring | Debian 13 (Trixie) | 192.168.88.246 | ✅ Active |

---

## Capabilities Enabled

| Capability | Description |
|---|---|
| **Security Configuration Assessment** | CIS Benchmark compliance scanning on all agents |
| **File Integrity Monitoring** | Detects unauthorized file changes across endpoints |
| **Log Analysis** | Collects and correlates Windows Event Logs, syslog, and application logs |
| **Rootcheck** | Scans for rootkits and suspicious system configurations |
| **Vulnerability Detection** | Identifies known CVEs on monitored systems |
| **MITRE ATT&CK Mapping** | Automatically maps detected events to ATT&CK techniques |

---

## Key Configuration Files

| File | Location | Purpose |
|---|---|---|
| Docker Compose | ~/wazuh-docker/single-node/docker-compose.yml | Stack definition |
| Agent Config | /var/ossec/etc/ossec.conf | Per-agent configuration |
| Manager Rules | /var/ossec/ruleset/rules/ | Detection rule definitions |

---

## Installation Notes

- Requires `vm.max_map_count=262144` set on host for OpenSearch to function
- SSL certificates generated via `generate-indexer-certs.yml` during initial setup
- Proxmox host required `lsb-release` package installed before agent deployment
- All Linux agents installed via `.deb` package (DEB amd64)
- Windows agent installed via PowerShell MSI deployment

---

## Related Findings

| ID | Title | Date |
|---|---|---|
| [SCA-001](../findings/SCA-001-windows-password-policy.md) | Insufficient Windows Password Policy Configuration | 2026-06-01 |

---

*Deployed and maintained by Anthony Ciresi | Security Operations Lab*
