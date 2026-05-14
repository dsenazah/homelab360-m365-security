# Phase 1 — Entra ID & Identity Security

<div align="center">

![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Entra ID](https://img.shields.io/badge/Entra_ID-P2-0078D4?style=for-the-badge&logo=microsoft-azure)
![CA Policies](https://img.shields.io/badge/CA_Policies-11_Deployed-00B294?style=for-the-badge)
![PIM](https://img.shields.io/badge/PIM-Configured-0078D4?style=for-the-badge)

</div>

**Tenant:** homelab360.com  
**License:** Microsoft 365 E5 / Entra ID P2  
**Prerequisite:** Phase 0 complete

---

## Objectives

Build a Zero Trust identity security foundation using Microsoft Entra ID P2.
Every access decision is verified — no implicit trust based on network location
or previous authentication. Identity is the new perimeter.

---

## What Was Configured

### 1. Named Locations

Named locations define geographic and IP-based boundaries used by CA policies.

| ID | Name | Type | Purpose |
|---|---|---|---|
| NL-001 | Trusted Office Network | IP ranges | Trusted location — add at project end |
| NL-002 | Allowed Countries — US, GB, CA | Countries | Permitted sign-in regions |
| NL-003 | High-Risk Countries — Block | Countries | CN, RU, KP, IR, SY |

NL-001 deferred until Intune device enrolment is complete to avoid lockout
during policy enforcement. This is the correct sequencing approach.

---

### 2. Conditional Access Policies

11 policies deployed following a structured rollout approach.
Policies that could cause lockout deployed in Report-only mode first.
Policies with no lockout risk enabled immediately.

#### Rollout States

| Policy | State | Reason |
|---|---|---|
| CA002 — Block legacy auth | On | No legitimate use case — safe immediately |
| CA005 — Block high-risk countries | On | Geographic block — safe immediately |
| CA008 — Privileged roles | Report-only | Requires Intune enrolment first |
| CA010 — Guest MFA | On | External users only — safe immediately |
| CA011 — No persistent browser sessions | On | Session control only |
| All others | Report-only | Monitor 2 weeks before enforcing |

#### Full Policy List

**CA001 — Require MFA for all users**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  All resources
Grant:      Require MFA
State:      Report-only
```

**CA002 — Block legacy authentication**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  All resources
Conditions: Client apps — Exchange ActiveSync + Other clients
Grant:      Block access
State:      On (enabled immediately)
```
Legacy authentication protocols cannot perform MFA. Any account using
legacy auth is completely unprotected. Blocking immediately with no
report-only period is correct — there is no legitimate modern use case.

**CA003 — Require compliant device — Windows and macOS**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  Office 365
Conditions: Device platforms — Windows, macOS
Grant:      Compliant device OR Hybrid Entra joined (OR)
State:      Report-only (enable after Phase 2 Intune enrolment)
```

**CA004 — Require approved app for iOS and Android**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  Office 365
Conditions: Device platforms — iOS, Android
Grant:      Approved client app OR App protection policy (OR)
State:      Report-only
```
MAM without enrolment — users can access M365 on personal devices
only through Microsoft-approved apps with protection policies applied.

**CA005 — Block sign-in from high-risk countries**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  All resources
Conditions: Locations — High-Risk Countries (NL-003)
Grant:      Block access
State:      On (enabled immediately)
```

**CA006 — Sign-in risk HIGH or MEDIUM — Require MFA**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  All resources
Conditions: Sign-in risk — High, Medium
Grant:      MFA + compliant device (AND)
Session:    Sign-in frequency 1 hour
State:      Report-only
```
Uses Entra ID Protection real-time ML risk signals backed by Microsoft
global threat intelligence. Requires Entra ID P2.

**CA007 — User risk HIGH — Force password change**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  All resources
Conditions: User risk — High
Grant:      MFA + password change (AND)
State:      Report-only
```
User risk HIGH indicates Microsoft has high confidence credentials are
compromised (e.g. found in breach database). Forces immediate reset.

**CA008 — Privileged roles — MFA and compliant device always**
```
Users:      Directory roles (GA, Security Admin, Helpdesk Admin,
            Password Admin, CA Admin, Priv Auth Admin, Intune Admin)
            (exclude Break-Glass Accounts group)
Resources:  All resources
Grant:      MFA + compliant device (AND)
Session:    Sign-in frequency 4 hours, Persistent browser Never
State:      Report-only (enable after Phase 2 Intune enrolment)
```
No exceptions for location or trusted networks. Admin accounts always
require strong authentication and a managed device.

**CA009 — MFA registration from trusted location only**
```
State:      Not yet created
Reason:     Requires NL-001 (trusted office IP) — deferred to end of project
```

**CA010 — Guest and B2B users — Require MFA always**
```
Users:      All guest and external user types
Resources:  All resources
Grant:      MFA
Session:    Sign-in frequency 1 hour, Persistent browser Never
State:      On
```
MFA evaluated at homelab360.com tenant regardless of guest home tenant.

**CA011 — No persistent browser sessions on unmanaged devices**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  All resources
Conditions: Client apps — Browser only
Grant:      None (session control only)
Session:    Sign-in frequency 1 hour, Persistent browser Never
State:      On
```

**CA012 — CASB session control via Defender for Cloud Apps**
```
Users:      All users (exclude Break-Glass Accounts group)
Resources:  Office 365
Conditions: Client apps — Browser, unmanaged devices
Grant:      MFA
Session:    Conditional Access App Control — Monitor only
State:      Report-only (enable after MDCA connector configured in Phase 3)
```

---

### 3. Privileged Identity Management (PIM)

Just-in-time privileged access — no permanent admin roles for regular accounts.

**Configuration:**

| Setting | Value |
|---|---|
| Account | ******@homelab360.com |
| Role | Security Reader |
| Assignment type | Eligible (not active) |
| Duration | Permanently eligible |
| Max activation | 1 hour |

**Activation workflow tested:**
1. Sign in as it-admin
2. Navigate to PIM → My roles
3. Click Activate on Security Reader
4. Enter justification
5. Role activates for 1 hour
6. Auto-expires — no permanent access

**PIM Access Reviews configured:**

| Setting | Value |
|---|---|
| Review name | PIM Security Reader Review — 90 Days |
| Frequency | Quarterly |
| Duration | 14 days |
| Role reviewed | Security Reader |
| Reviewer | Primary admin account |
| End | Never (runs indefinitely) |

Every 90 days the reviewer confirms it-admin still requires Security Reader.
If not confirmed the assignment is automatically removed.

---

### 4. Authentication Methods Hardened

Only phishing-resistant methods permitted tenant-wide.

| Method | Status | Reason |
|---|---|---|
| Microsoft Authenticator | Enabled — All users | Strongest push MFA, passwordless capable |
| Temporary Access Pass | Enabled — All users | Secure onboarding for new users |
| SMS | Disabled | SIM-swappable — weak MFA |
| Software OATH tokens | Disabled | Less secure than Authenticator |
| Email OTP | Disabled | Email compromise = MFA bypass |
| Voice call | Disabled | Social engineering risk |
| Passkey (FIDO2) | Disabled | Can enable as future enhancement |
| Hardware OATH tokens | Disabled | Not applicable for this environment |

**Why disable SMS?**
SMS MFA is vulnerable to SIM-swapping attacks where an attacker convinces
a mobile carrier to transfer a victim's phone number to a new SIM.
Once done they receive all SMS codes. Removing SMS forces users to
Microsoft Authenticator which is phishing-resistant and device-bound.

**Number matching enabled:**
Microsoft Authenticator number matching is enabled for all users.
Users must match a number displayed on screen during authentication
rather than just tapping Approve. This defeats MFA fatigue attacks
where attackers spam push notifications hoping for an accidental approval.

---

### 5. Self-Service Password Reset (SSPR)

| Setting | Value |
|---|---|
| Enabled | All users |
| Methods required | 2 |
| Available methods | Mobile app notification, Mobile app code, Email, Mobile phone |

Two methods required prevents single-factor account takeover via SSPR.
An attacker who only knows a user's email address cannot reset their password.

---

### 6. Entra ID Protection

Risk policies reviewed and confirmed. Legacy Identity Protection policies
(User risk policy, Sign-in risk policy) are read-only and being retired by
Microsoft on October 1, 2026. All risk-based access control is handled by
CA006 and CA007 using the current Conditional Access framework — the correct
modern approach that Microsoft is migrating all customers to.

```
Risk signal handling:
Sign-in risk HIGH/MEDIUM → CA006 → MFA + compliant device required
User risk HIGH           → CA007 → MFA + forced password change
```

---

## Lessons Learned

**CA008 lockout incident:**
Enabling CA008 (privileged roles — require compliant device) before completing
Intune device enrolment caused a lockout on the primary admin account.
The break-glass account was used to recover access and CA008 was moved back
to Report-only. Lesson: never enable device compliance requirements before
confirming all admin devices are enrolled and marked compliant in Intune.

**Lockout diagnosis process:**
Sign-in logs → Conditional Access tab → identified failing policy →
fixed configuration → verified recovery. This is the correct enterprise
troubleshooting workflow for CA-related access issues.

**Microsoft built-in CA policies conflict:**
Microsoft's built-in "Multifactor authentication for admins" and
"Multifactor authentication for all users" policies were On and conflicting
with our custom CA policies. Both moved to Report-only since CA001 and CA008
provide equivalent and more granular coverage.

**Identity Protection policy retirement:**
Microsoft is retiring legacy Identity Protection risk policies in favour of
Conditional Access. Building CA006 and CA007 using Conditional Access from
the start means no migration required — already on the modern approach.

---

## Security Posture — Phase 1 Summary

```
Legacy authentication        BLOCKED (CA002 — On)
High-risk country access     BLOCKED (CA005 — On)
Guest/B2B access             MFA REQUIRED (CA010 — On)
Browser session persistence  CONTROLLED (CA011 — On)
MFA for all users            MONITORED (CA001 — Report-only, enforce after 2 weeks)
Device compliance            MONITORED (CA003 — Report-only, enforce after Phase 2)
Privileged role access       MONITORED (CA008 — Report-only, enforce after Phase 2)
Risk-based access            MONITORED (CA006/007 — Report-only)
Admin roles                  JIT via PIM (Security Reader tested)
Authentication methods       Authenticator + TAP only (SMS/Email OTP disabled)
SSPR                         Enabled, 2 methods required
```

---

## Files in This Phase

Screenshots and configuration files will be added upon project completion.
---

## Next Phase

**[Phase 2 — Microsoft Intune & Endpoint Management](../Phase-2/README.md)**

- Enrol Windows 11 device into Intune
- Enrol mobile device (MAM)
- Create device compliance policies
- Deploy MDM Security Baseline
- Create App Protection Policies
- Enable CA003 and CA008 once devices are compliant
