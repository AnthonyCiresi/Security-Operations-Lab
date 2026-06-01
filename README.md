# Security-Operations-Lab

A personal homelab environment designed to sumulate real-world SOC and NOC analyst workflows including threat detection, security monitoring, incident response, and technical documentation.

---

## Lab Overview

This lab is built on a dedicated homelab server running Proxmox as the hypervisor, hosting multiple virtual machines and Docker containers that collectively simulate a small enterprise environment.

**Hardware:**
- CPU: AMD Ryzen 5 3600 (6c/12t)
- RAM: 48 GB DDR4
- Storgae: 1 TB NVMe SSD + 4 TB HDD
- GPU: NVIDIA GTX 1650
- Motherboard: MSB B550 Tomahawk

**Core Infrastructure:**
| Service | Platform | Purpose
|---|---|---|
| Proxmox VE | Bare metal hypervisor | VM and container management |
| Wazuh SIEM | Docker (Ubuntu VM) | Security monitoring and alerting | 
| Pi-hole | Docker | DNS filtering and network monitoring |
| Grafana + Prometheus | Docker | Infrastructure metrics and dashboard |
| Plex Media Server | Docker | Media Services |
| Windows 11 VVM | Proxmox VM | Endpoint monitoring and break/fix testing |

---

## SOC/NOC Capabilities

- **SIEM Monitoring** - Wazuh deployed with agents across endpoints collecting and correlataing security events in real time
- **Compliance Assessment** - CIS Benchmark Scanning via Wazuh SCA module
- **Threat Detection** - Custom alert rules and thresholds for suspicious activity
- **Incident Documentation** - All findings documented in analyst report format (see /findings)
- **Runbooks** - Standard operating procedures for common alert types (see /runbooks)

---

## Respoitory Structure

|Folder | Contents |
|---|---|
| /findings | Security assessment findings and incident reports |
| /runbooks | Alert response procedures and SOPs |
| /architecture | Network diagrams and infrastructure documentation |
| /tools | Notes and configs for each deployed tool |

---

## Certification & Training
- CompTUA Security+
- CySA+ (In progress, expected 2026)
- Ohio State University Cybersecurity Bootcamp (2024-2025)

---

## Connect
- [LinkedIn](https://www.linkedin.com/in/anthony-ciresi-60bb28316/)
