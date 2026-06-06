# Lab Architecture

[![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)]() [![VLAN%20Redesign](https://img.shields.io/badge/VLAN%20Redesign-Planned-orange?style=flat-square)]()

---

## Overview

A self-hosted security operations lab built on a Proxmox hypervisor with a
full Wazuh SIEM stack, network-wide DNS filtering, and real-time Discord
alerting. The lab simulates a small enterprise SOC environment for threat
detection, incident response practice, and security tooling development.

---

## Physical Infrastructure

| Device | Model | Role |
| ------ | ----- | ---- |
| **Hypervisor** | Custom Build — AMD Ryzen 5 3600, 48GB RAM, 1TB NVMe + 4TB HDD | Proxmox VE host running all lab VMs |
| **Router** | MikroTik RB5009UG+S+IN | Edge router, DHCP, firewall, future VLAN routing |
| **Switch** | MikroTik CSS610-8G-2S+ | Managed switch, future VLAN segmentation |
| **Upstream** | AT&T Gateway (bridge/bypass mode) | ISP modem — IP passthrough to RB5009 |

**Current Network:** `192.168.88.0/24` (flat, single subnet)

> **Planned:** VLAN segmentation into MAIN (10), LAB (20), IOT (30), and MGMT (99)
> subnets. See the [VLAN runbook](../runbooks/vlan-cutover.md) for the implementation plan.

---

## Virtual Machines

| VM Name | OS | IP | Role |
| ------- | -- | -- | ---- |
| wazuh-siem | Ubuntu 24.04 LTS | 192.168.88.90 | Wazuh SIEM stack (Docker Compose) |
| proxmox-host | Proxmox VE / Debian | 192.168.88.244 | Hypervisor host — also a monitored endpoint |
| pihole | Ubuntu 24.04 LTS | 192.168.88.250 | Network-wide DNS filtering and ad blocking |
| monitoring | Debian 13 (Trixie) | 192.168.88.246 | Grafana + InfluxDB + Telegraf metrics stack |
| windows-breakfix | Windows 11 | 192.168.88.91 | Windows endpoint for SCA findings and attack simulation |

---

## Security Monitoring Stack

```
┌─────────────────────────────────────────────────────┐
│                  Monitored Endpoints                 │
│  windows-breakfix  │  pve  │  pihole  │  monitoring  │
└────────────────────┬────────────────────────────────┘
                     │ Wazuh Agents (port 1514)
                     ▼
┌─────────────────────────────────────────────────────┐
│              Wazuh SIEM (192.168.88.90)              │
│  ┌──────────────┐  ┌───────────┐  ┌──────────────┐  │
│  │ wazuh.manager│→ │wazuh.index│→ │wazuh.dashboard│  │
│  │ (analysis)   │  │(OpenSearch│  │  (web UI)    │  │
│  └──────┬───────┘  └───────────┘  └──────────────┘  │
│         │ alerts.json                                │
└─────────┼───────────────────────────────────────────┘
          │ Python forwarder (systemd)
          ▼
┌─────────────────────┐
│   Discord Webhook   │
│  #wazuh-alerts      │
└─────────────────────┘
```

---

## Metrics and Observability Stack

```
┌──────────────────────────────────────┐
│  monitoring VM (192.168.88.246)       │
│                                      │
│  Telegraf → InfluxDB → Grafana       │
│  (collect)   (store)   (visualize)   │
└──────────────────────────────────────┘
```

Grafana dashboard displays system metrics (CPU, memory, disk, network) for
the Proxmox host and VMs via Telegraf agents reporting to InfluxDB.

**Planned additions:**
- MikroTik SNMP metrics (bandwidth per port, firewall hit counters)
- Per-VLAN traffic visibility post-segmentation

---

## Network Diagram (Current — Flat)

```
Internet
    │
AT&T Gateway (bridge/passthrough)
    │
RB5009 Router (192.168.88.1)
    │ ether2
CSS610 Switch
    ├── Port 3 → Main PC
    ├── Port 4 → Proxmox (192.168.88.244)
    ├── Port 5 → Extra Laptop
    └── Port 6 → Hue Bridge
```

---

## Network Diagram (Planned — VLAN Segmented)

```
Internet
    │
AT&T Gateway (bridge/passthrough)
    │
RB5009 Router
    ├── VLAN 10 (MAIN)  192.168.10.0/24 → Main PC, Laptop
    ├── VLAN 20 (LAB)   192.168.20.0/24 → Proxmox + all VMs
    ├── VLAN 30 (IOT)   192.168.30.0/24 → Hue Bridge, IoT devices
    └── VLAN 99 (MGMT)  192.168.99.0/24 → Router, Switch management
```

**Firewall policy:**
- MAIN → LAB: allow (access lab from main PC)
- MAIN → IOT: allow outbound only
- LAB → MAIN: block
- IOT → LAN: block (internet only)
- All → internet: allow

---

## Planned Additions

| Component | Purpose | Status |
| --------- | ------- | ------ |
| VLAN segmentation | Network isolation between MAIN, LAB, IOT | Planned |
| WireGuard VPN | Remote access to lab via RB5009 native WireGuard | Planned |
| Suricata IDS | Network-layer intrusion detection on Proxmox host | Planned |
| Velociraptor | Endpoint forensics and artifact collection | Planned |
| Atomic Red Team | MITRE ATT&CK detection testing on windows-breakfix | Planned |
| MikroTik Grafana | Network metrics dashboard via SNMP | Planned |

---

*Security Operations Lab | Anthony Ciresi*
