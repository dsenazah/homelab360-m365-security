# Phase 2 — Microsoft Intune & Endpoint Management

<div align="center">

![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Intune](https://img.shields.io/badge/Microsoft_Intune-MDM%2FMAM-0078D4?style=for-the-badge&logo=microsoft)
![Devices](https://img.shields.io/badge/Devices-Enrolled-00B294?style=for-the-badge)
![Compliance](https://img.shields.io/badge/Compliance-Policy_Active-brightgreen?style=for-the-badge)

</div>

**Tenant:** homelab360.com
**License:** Microsoft 365 E5 / Intune P2
**Prerequisite:** Phase 1 complete

---

## Objectives

Deploy Microsoft Intune as the Mobile Device Management (MDM) and Mobile
Application Management (MAM) solution for homelab360.com. Ensure all corporate
devices meet security compliance requirements before accessing Microsoft 365
resources. Establish the endpoint security foundation that Phase 3 Defender
XDR builds on.

---

## Environment

| Property | Detail |
|---|---|
| Primary device | Windows 11 Desktop (DESKTOP-2F6RRO8) |
| OS Version | Windows 11 24H2 (10.0.26100) |
| Enrolment type | Azure AD Join + automatic MDM enrolment |
| Ownership | Corporate |
| Primary user | *****@homelab360.com |
| Additional devices | 3 laptops available for future enrolment |

---

## What Was Configured

### 1. Device Enrolment

Windows 11 desktop enrolled into Intune via Azure AD Join.
Automatic MDM enrolment triggered upon Azure AD join — no manual
enrolment steps required.

```
Enrolment path:
Settings → Accounts → Access work or school
→ Connect → sign in as dsenah@homelab360.com
→ Azure AD Join triggers automatic Intune MDM enrolment
→ Device appears in Intune within minutes
→ Policies begin applying on first sync
```

---

### 2. WIN-001 — Windows 11 Compliance Baseline

Core compliance policy defining the minimum security requirements
for all Windows devices in the tenant.

**Device Health:**
| Setting | Value |
|---|---|
| BitLocker | Require |
| Secure Boot | Require |
| Code Integrity | Require |

**Device Properties:**
| Setting | Value |
|---|---|
| Minimum OS version | 10.0.22000 (Windows 11) |

**System Security:**
| Setting | Value |
|---|---|
| Password required | Require |
| Simple passwords | Block |
| Password type | Numeric |
| Minimum length | 6 characters |
| Inactivity timeout | 15 minutes |
| Password expiration | 90 days |
| Password history | 5 previous passwords |
| Encryption | Require |
| Firewall | Require |
| TPM | Require |
| Antivirus | Require |
| Antispyware | Require |

**Defender:**
| Setting | Value |
|---|---|
| Microsoft Defender Antimalware | Require |
| Real-time protection | Require |
| Security intelligence up-to-date | Require |
| Machine risk score | Medium or below |

**Actions for noncompliance:**
| Action | Schedule |
|---|---|
| Mark device noncompliant | Immediately |
| Send email notification | 1 day |
| Retire device | 30 days |

**Assignment:** All-Employees group

**Compliance status after deployment:**
```
BitLocker              → Compliant   ✅
Secure Boot            → Compliant   ✅
TPM                    → Compliant   ✅
Firewall               → Compliant   ✅
Antivirus              → Compliant   ✅
Real-time protection   → Compliant   ✅ (fixed during deployment)
Machine risk score     → Noncompliant ⚠️ (resolves after Phase 3 MDE onboarding)
```

---

### 3. Compliance Settings — Tenant Wide

| Setting | Value | Rationale |
|---|---|---|
| Devices with no policy → | Not compliant | Unmanaged devices not trusted by default |
| Compliance validity period | 1 day | Devices must check in daily |

**Why "no policy = not compliant"?**
Any device that enrols but has no compliance policy assigned is treated
as untrusted. This prevents new devices from accessing corporate resources
before policies are applied. Security by default, not convenience by default.

**Why 1 day validity?**
A device that goes offline for more than 24 hours automatically becomes
noncompliant. This ensures compliance status is always current and
a stolen or abandoned device loses access quickly.

---

### 4. WIN-002 — MDM Security Baseline

Microsoft's recommended security configuration baseline for Windows 11
deployed via Endpoint security → Security baselines.

| Setting | Value |
|---|---|
| Baseline version | 25H2 (latest) |
| Assignment | All-Employees group |
| Settings | Microsoft recommended defaults (200+ settings) |

Covers: Above Lock, App Runtime, Application Management, Auto Play,
BitLocker, Browser, Connectivity, Credentials, Device Guard,
DMA Guard, Event Log, Firewall, Internet Explorer, Local Policies,
MS Security Guide, MSS Legacy, Power, Remote Assistance,
Remote Desktop, Remote Management, Remote Procedure Call,
Search, Smart Screen, System, Wi-Fi, Windows Defender, Windows Ink,
Windows PowerShell, and more.

**Note on conflicts:**
Initial deployment showed a conflict on MIN Device Password Length
due to legacy policies from previous tenant configuration. Resolved
by deleting all legacy configuration policies and resyncing the device.

---

### 5. APP-001 — iOS App Protection Policy (MAM)

Mobile Application Management policy controlling corporate data
on personal iOS devices. No device enrolment required.

| Setting | Value |
|---|---|
| Platform | iOS/iPadOS |
| Target apps | All Microsoft Apps |
| Backup to iCloud | Block |
| Send org data to other apps | Policy managed apps only |
| Save copies of org data | Block |
| Receive data from other apps | Policy managed apps only |
| Cut/copy/paste | Policy managed apps with paste in |
| Third party keyboards | Block |
| Encrypt org data | Require |
| PIN for access | Require (numeric, 6 digits) |
| Biometrics | Allow |
| Recheck access after | 30 minutes |
| Assignment | All-Employees group |

**What this prevents:**
```
→ User cannot copy email from Outlook and paste into WhatsApp
→ User cannot save corporate OneDrive files to personal iCloud
→ User cannot back up corporate app data to personal iCloud
→ Corporate data encrypted at rest on device
→ Third party keyboards blocked (keylogger risk)
```

---

### 6. App Deployments

| App | Type | Assignment | Install |
|---|---|---|---|
| Microsoft 365 Apps | M365 Apps suite | All-Employees (Required) | Silent |
| Company Portal | Microsoft Store (new) | All Devices (Required) | System |

**Microsoft 365 Apps includes:**
Word, Excel, PowerPoint, Outlook, Teams, OneNote, Edge (Chromium)

**Why Company Portal on All Devices vs All Users?**
Company Portal is a device-level app needed before any user signs in.
Assigning to All Devices ensures it's present on every enrolled device
immediately after enrolment, regardless of which user signs in first.

**Why Microsoft Defender not deployed separately?**
Microsoft Defender is built into Windows 11 natively. No separate
deployment needed. Onboarding to Defender for Endpoint (the cloud
security portal) is handled in Phase 3.

---

### 7. Endpoint Analytics — Baseline Score

| Metric | Score | Industry Median |
|---|---|---|
| Overall score | 50 | 33 |
| Startup performance | 50 | — |
| Application reliability | 0 | — (insufficient data) |
| Work from anywhere | 0 | — (insufficient data) |

Tenant score of 50 is above the industry median of 33.
Application reliability and Work from anywhere scores will populate
as more devices enrol and more data is collected over time.

---

## Lessons Learned

**Policy conflicts from legacy configuration:**
Old configuration policies (Win-OIB-ES-ASR, Win-OIB-TP-Health,
Win-10 MDE) from previous tenant setup conflicted with WIN-002.
All legacy policies deleted and device resynced to resolve.
Lesson: always audit existing policies before deploying new baselines.

**MDE machine risk score requires Phase 3:**
WIN-001 requires device to be at or under Medium risk score. This
setting cannot be evaluated until Defender for Endpoint is onboarded
and reporting risk scores. Device remains noncompliant on this setting
until Phase 3 MDE onboarding is complete.

**Real-time protection was disabled:**
Desktop had real-time protection disabled — discovered during compliance
evaluation. Enabled manually via Windows Security. In production this
would have been caught earlier by monitoring.

**User vs Device app assignment:**
Productivity apps (M365 suite) assigned to users — apps follow the
person across devices. Security/management apps (Company Portal)
assigned to devices — present before any user signs in.

**CA003 enforcement deferred:**
CA003 (require compliant device) kept in Report-only until MDE
onboarding resolves the machine risk score noncompliance. Enabling
CA003 before full compliance would lock out the primary admin account.

---

## Security Posture — Phase 2 Summary

```
Device enrolment        ACTIVE — desktop enrolled and managed
Compliance policy       ACTIVE — WIN-001 evaluating all settings
Security baseline       ACTIVE — WIN-002 applying 200+ security settings
App protection (MAM)    ACTIVE — APP-001 protecting iOS corporate data
App deployment          ACTIVE — M365 Apps and Company Portal deployed
CA003 enforcement       DEFERRED — enable after Phase 3 MDE onboarding
CA008 enforcement       DEFERRED — enable after device shows fully compliant
Endpoint analytics      ACTIVE — baseline score 50 (above median of 33)
```

---

## Next Phase

**[Phase 3 — Microsoft Defender XDR](../Phase-3/README.md)**

- Onboard desktop to Defender for Endpoint
- Run EICAR test to verify detection
- Configure Attack Surface Reduction rules
- Enable Safe Links and Safe Attachments
- Configure Anti-phishing policy
- Run Attack Simulator phishing campaign
- Connect Defender for Cloud Apps
- Enable CA003 and CA008 once device is fully compliant
