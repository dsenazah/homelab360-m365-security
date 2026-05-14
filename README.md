#  Microsoft 365 E5 — Enterprise Security Project

<div align="center">

![Microsoft 365](https://img.shields.io/badge/Microsoft_365-E5-0078D4?style=for-the-badge&logo=microsoft&logoColor=white)
![Entra ID](https://img.shields.io/badge/Entra_ID-P2-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Defender](https://img.shields.io/badge/Defender_XDR-Active-00B294?style=for-the-badge&logo=microsoft&logoColor=white)
![Sentinel](https://img.shields.io/badge/Sentinel-SIEM-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Zero Trust](https://img.shields.io/badge/Zero_Trust-Architecture-FF6B35?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Actively_Building-brightgreen?style=for-the-badge)

**A hands-on, production-grade Microsoft 365 E5 security implementation**  
**built on a real tenant — homelab360.com**

[View Live Portfolio](https://homelab360.com) · [Phase 0 — Complete](./Phase-0/README.md) · [Phase 1 — In Progress](./Phase-1/README.md)

</div>

---

##  About This Project

This is not a course walkthrough or a simulated lab environment.

Every policy, every configuration and every security decision in this repository was deployed on a **live Microsoft 365 E5 tenant** with a verified custom domain. The goal is to build, document and demonstrate the full Microsoft cloud security stack — from identity and endpoint management through to threat detection, compliance and automation — the same way it is done in enterprise environments.

**Why I built this:**  
I am a Technology Specialist with 3+ years of hands-on Microsoft 365 experience transitioning into a Microsoft Cloud Security Engineer role. This project documents that transition in real time — every phase completed, every decision justified, every tool mastered.

---

## 🏢 Environment

| Property | Detail |
|---|---|
| **Tenant Domain** | homelab360.com |
| **License** | Microsoft 365 E5 |
| **Azure AD** | Entra ID P2 |
| **Departments** | IT · Finance · HR · Executives |
| **Devices** | Windows 11 · iOS · Android · macOS |
| **Location** | Columbus, Ohio, USA |
| **Architecture** | Microsoft Zero Trust |
| **DNS Provider** | GoDaddy |

---

## 🗺️ Project Roadmap

| # | Phase | Focus Area | Status |
|---|---|---|---|
| 0 | [Tenant Foundations](./Phase-0/README.md) | Domain · Email auth · Users · Groups · Audit | ✅ **Complete** |
| 1 | [Entra ID & Identity](./Phase-1/README.md) | Conditional Access · PIM · Passwordless · ID Protection | ✅ **Complete** |
| 2 | [Intune & Endpoints](./Phase-2/README.md) | MDM · MAM · Autopilot · Compliance policies · ASR | 🔄 **In Progress**|
| 3 | [Defender XDR](./Phase-3/README.md) | MDE · MDO P2 · MDI · MDCA · Attack simulation | ⬜ Upcoming |
| 3b | [Microsoft Sentinel](./Phase-3b/README.md) | SIEM · KQL detection rules · SOAR playbooks | ⬜ Upcoming |
| 4 | [Microsoft Purview](./Phase-4/README.md) | DLP · Sensitivity labels · Retention · Insider Risk | ⬜ Upcoming |
| 5 | [SharePoint Security](./Phase-5/README.md) | Site security · External sharing · Permissions | ⬜ Upcoming |
| 6 | [Power Automate](./Phase-6/README.md) | Security workflows · Automated responses · Integration | ⬜ Upcoming |

---

##  Full Security Stack

### Identity & Access
```
Microsoft Entra ID P2
├── Conditional Access          12 policies covering MFA, device compliance,
│                               risk-based access, guest users, CASB routing
├── Privileged Identity Mgmt    Just-in-time admin access, activation workflow,
│                               access reviews, audit trail
├── Entra ID Protection         Real-time sign-in risk + user risk policies
│                               backed by Microsoft threat intelligence
├── Passwordless Auth           Microsoft Authenticator, Windows Hello for Business
└── SSPR                        Self-service password reset, 2-method verification
```

### Endpoint Security
```
Microsoft Intune
├── Windows 11 (MDM)            Autopilot enrolment, compliance policies,
│                               BitLocker, TPM 2.0, Secure Boot
├── iOS / Android (MAM)         App protection policies, no copy/paste to
│                               personal apps, selective wipe
└── macOS                       FileVault encryption, Gatekeeper, Defender

Microsoft Defender for Endpoint P2
├── EDR                         Endpoint detection and response
├── ASR Rules                   Attack surface reduction (audit → block)
├── TVM                         Threat and vulnerability management
└── AIR                         Automated investigation and remediation
```

### Threat Protection
```
Microsoft Defender XDR (unified portal)
├── Defender for Office 365 P2  Safe Links, Safe Attachments, anti-phishing,
│                               ZAP, Attack Simulator campaigns
├── Defender for Identity       On-prem AD lateral movement detection,
│                               pass-the-hash, honey token accounts
└── Defender for Cloud Apps     Shadow IT discovery, OAuth app governance,
                                session policies, anomaly detection
```

### SIEM & SOAR
```
Microsoft Sentinel
├── Data Connectors             Defender XDR, Entra ID, Office 365,
│                               Azure Activity, Purview DLP
├── KQL Detection Rules         Custom analytics rules mapped to MITRE ATT&CK:
│                               MFA fatigue, impossible travel, password spray,
│                               mass download, new GA outside PIM
├── SOAR Playbooks              Logic Apps: auto-disable compromised user,
│                               Teams SOC alerts, Jira ticket creation
└── Workbooks                   Sign-in map, failed MFA over time, top risky users
```

### Compliance & Data
```
Microsoft Purview
├── Sensitivity Labels          4-tier taxonomy: Public / Internal /
│                               Confidential / Highly Confidential
├── DLP Policies                Block financial data exfiltration via email,
│                               Teams, and USB on endpoints
├── Retention Policies          7-year financial records retention
├── Insider Risk Management     Departing employee data theft policy
└── Compliance Manager          Improvement actions and compliance score tracking
```

### Collaboration & Automation
```
SharePoint Online               Site security, external sharing controls,
                                permissions management, sensitivity labels

Power Automate                  Security workflow automation, approval flows,
                                automated incident notifications

Microsoft Graph API             PowerShell automation for policy deployment,
                                reporting and tenant management
```

---

## 📂 Repository Contents

### Scripts & Configs
| File | Description |
|---|---|
| [`Phase-1/conditional-access/homelab360-ca-policies.json`](./Phase-1/conditional-access/) | Full JSON export of all 12 CA policies with named locations |
| [`Phase-1/conditional-access/Deploy-CAPolicies.ps1`](./Phase-1/conditional-access/) | PowerShell Graph API deployment script with WhatIf / ReportOnly / Enabled modes |
| [`Phase-3b/kql-detection-rules/`](./Phase-3b/kql-detection-rules/) | Custom Sentinel KQL analytics rules mapped to MITRE ATT&CK |
| [`Phase-3b/soar-playbooks/`](./Phase-3b/soar-playbooks/) | Logic Apps SOAR playbook templates |
| [`Phase-3/incident-response/`](./Phase-3/incident-response/) | IR reports from Attack Simulator campaigns |

### Documentation
Each phase folder contains a detailed README covering:
- What was configured and why
- Security decisions and rationale
- Step-by-step implementation notes
- Screenshots from the live portal
- Lessons learned and enterprise considerations

---

##  MITRE ATT&CK Coverage

| Technique | ID | Control |
|---|---|---|
| Valid Accounts | T1078 | CA001 MFA · CA006 Sign-in risk · CA008 Privileged roles |
| Legacy Authentication | T1078.004 | CA002 Block legacy auth |
| Brute Force / Password Spray | T1110 | Sentinel KQL rule · ID Protection |
| MFA Fatigue | T1621 | CA009 Registration restriction · Sentinel alert |
| Credential Stuffing | T1110.004 | CA007 User risk HIGH · force password change |
| Web Session Cookie | T1550.004 | CA011 No persistent browser sessions |
| Cloud Account Manipulation | T1098 | PIM · CA008 Privileged roles hardening |
| Data from Cloud Storage | T1530 | CA003 Compliant device · CA012 CASB session |

---

##  Certifications

| Badge | Certification | Status |
|---|---|---|
| SC-900 | Microsoft Security, Compliance & Identity Fundamentals | ✅ Complete |
| Security+ | CompTIA Security+ | ✅ Complete |
| SC-300 | Microsoft Identity and Access Administrator | 🔄 Studying |
| SC-200 | Microsoft Security Operations Analyst | ⬜ Upcoming |
| SC-100 | Microsoft Cybersecurity Architect Expert | ⬜ Upcoming |

---

## 💡 Key Engineering Decisions

> **Why group-based CA exclusions instead of individual accounts?**  
> Excluding the break-glass *group* from CA policies means rotating or adding
> emergency accounts requires only a group membership change — not editing
> every CA policy individually. Scales cleanly in large environments.

> **Why report-only mode before enforcing CA policies?**  
> Enables 2-week baselining of would-be blocks without impacting users.
> Prevents accidental lockouts and builds confidence before enforcement.
> This is standard enterprise rollout practice.

> **Why DMARC quarantine before reject?**  
> Hard reject (p=reject) on day one risks dropping legitimate email from
> misconfigured senders. Quarantine lets you monitor aggregate reports first,
> identify any gaps, then escalate to reject with confidence.

> **Why PowerShell + Graph API instead of portal-only?**  
> Portal clicks are not auditable, not repeatable and do not scale.
> Code-based deployment means every change is version controlled,
> peer-reviewable and can be re-run identically across any tenant.

---

## 📊 Project Stats

```
Phases planned          8
Phases complete         1
CA policies deployed    12
KQL detection rules     5+
SOAR playbooks          3
Certifications earned   2
Certifications in prog  1
```

---

##  About the Author

**Daniel Senazah**  
Technology Specialist → Microsoft Cloud Security Engineer  
3+ years enterprise Microsoft 365 administration (500+ user environments)  
📍 Columbus, Ohio  
🌐 [homelab360.com](https://homelab360.com)

*Open to Microsoft cloud security engineer opportunities.*

---

<div align="center">

*Built hands-on. Documented in real time. Every config has a reason.*

⭐ Star this repo if you find it useful

</div>
