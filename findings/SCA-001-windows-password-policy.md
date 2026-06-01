# SCA-001 — Insufficient Windows Password Policy Configuration

![Severity](https://img.shields.io/badge/Severity-High-red?style=flat-square)
![Status](https://img.shields.io/badge/Status-Open-orange?style=flat-square)
![System](https://img.shields.io/badge/System-windows--breakfix-blue?style=flat-square)

---

## Finding Details

| Field | Details |
|---|---|
| **Finding ID** | SCA-001 |
| **Date Identified** | 2026-06-01 |
| **System** | windows-breakfix |
| **Assessment Type** | Security Configuration Assessment (SCA) |
| **Benchmark** | CIS Microsoft Windows 11 Enterprise Benchmark v1.0.0 |
| **Overall Score** | 32% (126 passed / 261 failed / 8 not applicable) |
| **Severity** | High |
| **Status** | Open |

---

## Summary

During a Security Configuration Assessment of endpoint `windows-breakfix` 
using the CIS Microsoft Windows 11 Enterprise Benchmark v1.0.0, a compliance 
score of 32% was identified. The most significant cluster of failures relates 
to password and account lockout policy configuration — foundational security 
controls that directly reduce the risk of brute force and credential-based 
attacks.

A default Windows 11 installation typically scores in the 30-40% range 
against this benchmark as hardening is not applied out of the box. However 
the password policy failures represent a High severity finding due to their 
direct exposure to credential based attack techniques.

---

## Affected Controls

| Control ID | Title | Result |
|---|---|---|
| 26000 | Enforce password history set to 24 or more | ❌ Failed |
| 26002 | Minimum password age set to 1 or more days | ❌ Failed |
| 26003 | Minimum password length set to 14 or more characters | ❌ Failed |
| 26004 | Password complexity requirements enabled | ❌ Failed |
| 26006 | Account lockout duration set to 15 or more minutes | ❌ Failed |
| 26007 | Account lockout threshold set to 5 or fewer attempts | ❌ Failed |
| 26008 | Reset lockout counter after 15 or more minutes | ❌ Failed |
| 26009 | Administrator account status set to Disabled | ✅ Passed |

---

## Risk Analysis

Without these controls in place the following attack scenarios are possible:

**Brute Force Attack**
An attacker with network access could attempt unlimited password guesses 
against any local account with no lockout enforced. Combined with no minimum 
password length requirement, weak passwords could be cracked rapidly.

**Password Spraying**
Without an account lockout threshold, automated tools could spray common 
passwords across multiple accounts without triggering any defensive response.

**Credential Reuse**
Without password history enforcement, users could immediately revert to 
previously compromised passwords after a forced reset, negating the 
security benefit of the password change.

**MITRE ATT&CK Mapping**
| Technique | ID | Tactic |
|---|---|---|
| Brute Force | T1110 | Credential Access |
| Password Spraying | T1110.003 | Credential Access |

---

## Remediation Recommendations

The following settings should be configured via Local Security Policy 
(`secpol.msc`) or Group Policy:

**Account Policies → Password Policy:**
- Password history: 24 passwords remembered
- Maximum password age: 365 days
- Minimum password age: 1 day
- Minimum password length: 14 characters
- Password complexity requirements: Enabled

**Account Policies → Account Lockout Policy:**
- Account lockout threshold: 5 invalid attempts
- Account lockout duration: 15 minutes
- Reset lockout counter after: 15 minutes

---

## References

- [CIS Microsoft Windows 11 Benchmark](https://www.cisecurity.org/benchmark/microsoft_windows_desktop)
- [MITRE ATT&CK T1110 — Brute Force](https://attack.mitre.org/techniques/T1110/)
- [MITRE ATT&CK T1110.003 — Password Spraying](https://attack.mitre.org/techniques/T1110/003/)

---

*Finding identified using Wazuh SIEM Security Configuration Assessment module*
*Analyst: Anthony Ciresi | Security Operations Lab*
