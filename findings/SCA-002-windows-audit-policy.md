# SCA-002 — Insufficient Windows Audit Policy Configuration

![Severity](https://img.shields.io/badge/Severity-Medium-orange?style=flat-square)
![Status](https://img.shields.io/badge/Status-Open-orange?style=flat-square)
![System](https://img.shields.io/badge/System-windows--breakfix-blue?style=flat-square)

---

## Finding Details

| Field | Details |
|---|---|
| **Finding ID** | SCA-002 |
| **Date Identified** | 2026-06-01 |
| **System** | windows-breakfix |
| **Assessment Type** | Behavioral Analysis / SIEM Observation |
| **Severity** | Medium |
| **Status** | Open |

---

## Summary

During SIEM monitoring of endpoint `windows-breakfix`, high severity Windows
security events including local user account creation and privilege escalation
were observed generating only Level 3-5 Wazuh alerts instead of the expected
Level 8-12. This indicates that Windows Security Audit Policy is not fully
configured, resulting in insufficient logging fidelity and potential gaps in
threat detection capability.

Specifically, the following actions were performed as part of controlled
testing and generated unexpectedly low severity alerts:

- Local user account creation (`net user /add`)
- Adding user to local administrators group (`net localgroup administrators`)
- Local user account deletion (`net user /delete`)

These actions map directly to high severity MITRE ATT&CK techniques and
should trigger high severity alerts in a properly configured environment.

---

## Evidence

| Action Performed | Expected Alert Level | Actual Alert Level |
|---|---|---|
| Local user account created | 8-12 | 3-5 |
| User added to Administrators | 8-12 | 3-5 |
| Local user account deleted | 8-12 | 3-5 |

---

## Risk Analysis

**Insufficient Visibility**
Without proper audit policy configuration, malicious actions such as
backdoor account creation, privilege escalation, and lateral movement
may generate low severity alerts that get filtered out or deprioritized
by analysts, allowing attackers to operate undetected.

**Alert Threshold Bypass**
SOC alert thresholds are typically set to focus analyst attention on
level 7+ events. Actions generating only level 3-5 alerts may never
reach the analyst's attention even when they represent active attacks.

**MITRE ATT&CK Mapping**

| Technique | ID | Tactic |
|---|---|---|
| Create Account: Local Account | T1136.001 | Persistence |
| Valid Accounts: Local Accounts | T1078.003 | Privilege Escalation |
| Account Manipulation | T1098 | Persistence |

---

## Remediation Recommendations

Configure Windows Advanced Audit Policy via Local Security Policy
(`secpol.msc`) or Group Policy under:

**Advanced Audit Policy Configuration → Account Management:**
- Audit User Account Management: Success and Failure
- Audit Security Group Management: Success and Failure
- Audit Computer Account Management: Success and Failure

**Advanced Audit Policy Configuration → Privilege Use:**
- Audit Sensitive Privilege Use: Success and Failure

**Advanced Audit Policy Configuration → Policy Change:**
- Audit Authentication Policy Change: Success and Failure

These settings ensure that account creation, privilege escalation, and
policy changes generate detailed Windows Security Event Log entries that
Wazuh can correlate to high severity alerts.

---

## References

- [MITRE ATT&CK T1136.001 - Create Account: Local Account](https://attack.mitre.org/techniques/T1136/001/)
- [MITRE ATT&CK T1078.003 - Valid Accounts: Local Accounts](https://attack.mitre.org/techniques/T1078/003/)
- [CIS Windows 11 Benchmark - Audit Policy](https://www.cisecurity.org/benchmark/microsoft_windows_desktop)
- [Microsoft Advanced Audit Policy Configuration](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/advanced-security-audit-policy-settings)

---

*Finding identified through controlled attack simulation and SIEM behavioral analysis*
*Analyst: Anthony Ciresi | Security Operations Lab*
