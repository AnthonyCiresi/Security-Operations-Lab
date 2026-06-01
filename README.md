# 🔐 Security Operations Lab

![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat-square)
![SIEM](https://img.shields.io/badge/SIEM-Wazuh-blue?style=flat-square)
![Hypervisor](https://img.shields.io/badge/Hypervisor-Proxmox-orange?style=flat-square)
![OS](https://img.shields.io/badge/OS-Ubuntu%2024.04-purple?style=flat-square)

A personal homelab environment designed to simulate real-world SOC and NOC 
analyst workflows including threat detection, security monitoring, incident 
response, and technical documentation.

---

## 🖥️ Hardware

| Component | Spec |
|---|---|
| CPU | AMD Ryzen 5 3600 (6c/12t) |
| RAM | 48GB DDR4 |
| Storage | 1TB NVMe SSD + 4TB HDD |
| GPU | NVIDIA GTX 1650 |
| Motherboard | MSI B550 Tomahawk |

---

## 🏗️ Core Infrastructure

| Service | Platform | Purpose |
|---|---|---|
| Proxmox VE | Bare metal hypervisor | VM and container management |
| Wazuh SIEM | Docker (Ubuntu 24.04 VM) | Security monitoring and alerting |
| Pi-hole | Docker | DNS filtering and network monitoring |
| Grafana + Prometheus | Docker | Infrastructure metrics and dashboards |
| Plex Media Server | Docker | Media services |
| Windows 11 VM | Proxmox VM | Endpoint monitoring and break/fix testing |

---

## 🛡️ SOC/NOC Capabilities

- **SIEM Monitoring** — Wazuh deployed with agents across endpoints,
  collecting and correlating security events in real time
- **Compliance Assessment** — CIS Benchmark scanning via Wazuh SCA module
- **Threat Detection** — Custom alert rules and thresholds for suspicious
  activity
- **Incident Documentation** — All findings documented in analyst report
  format (see /findings)
- **Runbooks** — Standard operating procedures for common alert types
  (see /runbooks)

---

## 📁 Repository Structure

| Folder | Contents |
|---|---|
| /findings | Security assessment findings and incident reports |
| /runbooks | Alert response procedures and SOPs |
| /architecture | Network diagrams and infrastructure documentation |
| /tools | Notes and configs for each deployed tool |

---

## 🎓 Certifications & Training

![Security+](https://img.shields.io/badge/CompTIA-Security%2B-red?style=flat-square)
![CySA+](https://img.shields.io/badge/CompTIA-CySA%2B%20In%20Progress-yellow?style=flat-square)
![OSU](https://img.shields.io/badge/Ohio%20State-Cybersecurity%20Bootcamp-grey?style=flat-square)

- ✅ CompTIA Security+
- 🔄 CompTIA CySA+ *(Expected 2026)*
- ✅ Ohio State University Cybersecurity Bootcamp

---

## 🔗 Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Anthony%20Ciresi-0077B5?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/anthony-ciresi-60bb28316/)
