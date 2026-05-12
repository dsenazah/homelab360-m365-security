# Phase 0 — Tenant Setup & Foundations

<div align="center">

![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=for-the-badge)
![Tenant](https://img.shields.io/badge/Tenant-homelab360.com-0078D4?style=for-the-badge&logo=microsoft)

</div>

**Duration:** Session 1  
**Tenant:** homelab360.com  
**Prerequisite for:** All subsequent phases

---

## Overview

Phase 0 establishes the secure foundation that every other phase depends on.
No security policies are worth deploying on a poorly configured tenant — 
this phase ensures the base is correct before anything is built on top of it.

---

## What Was Configured

### 1. Domain Verification
Custom domain `homelab360.com` verified in Microsoft Entra ID.
All user accounts created with `@homelab360.com` UPN suffix.

---

### 2. Break-Glass Emergency Account

A dedicated emergency access account configured to survive any failure scenario.

| Property | Detail |
|---|---|
| Domain | onmicrosoft.com (not the custom domain — intentional) |
| Role | Global Administrator (permanent) |
| MFA | None registered (intentional) |
| Group | Break-Glass Accounts security group |
| Credentials | Stored offline — not in any cloud-dependent system |

**Why onmicrosoft.com domain?**

If `homelab360.com` experiences a DNS failure or federation misconfiguration,
every account on the custom domain could fail to authenticate simultaneously.
The onmicrosoft.com domain is Microsoft-managed and always available regardless
of what happens to the custom domain. This account is the last line of defence.

**Why no MFA on break-glass?**

If the MFA infrastructure itself fails (Authenticator service outage, lost
devices, misconfigured policy), the break-glass account must still be able
to sign in. MFA on break-glass defeats its purpose.

**Why a group exclusion instead of individual account?**

All CA policies exclude the *Break-Glass Accounts group*, not the account
directly. When rotating or adding emergency accounts, only group membership
changes — not every CA policy. Scales cleanly as the environment grows.

---

### 3. Administrative Account Structure

| Account Type | Roles | License | Purpose |
|---|---|---|---|
| Primary admin | Global Administrator | E5 | Day-to-day administration |
| PIM practice account | None (permanent) | None | Phase 1 PIM demonstration |

Note: In a production environment the primary admin account would carry no
permanent roles and would elevate exclusively through PIM. This lab retains
permanent Global Admin on the working account for continuity while demonstrating
PIM via a dedicated practice account in Phase 1.

---

### 4. Email Authentication — SPF

```
Type:   TXT
Host:   @
Value:  v=spf1 include:spf.protection.outlook.com -all
```

`-all` (hard fail) — any mail server not listed in SPF is rejected outright.
This is stricter than `~all` (soft fail) which only marks suspicious email.
Hard fail is the correct enterprise setting for a domain that only sends
through Microsoft 365.

---

### 5. Email Authentication — DKIM

| Property | Value |
|---|---|
| Status | Valid — Enabled |
| Domain | homelab360.com |
| Selectors | selector1._domainkey · selector2._domainkey |
| Verified in | Microsoft Defender — Email Authentication Settings |

Two CNAME records added to GoDaddy DNS pointing to Microsoft's DKIM signing
infrastructure. Signing keys are managed and rotated by Microsoft.

---

### 6. Email Authentication — DMARC

```
Type:   TXT
Host:   _dmarc
Value:  v=DMARC1; p=quarantine; pct=100; fo=1
        rua=mailto:[internal mailbox]
        ruf=mailto:[internal mailbox]
```

| Tag | Value | Meaning |
|---|---|---|
| p | quarantine | Failing emails go to spam — not rejected yet |
| pct | 100 | Applied to 100% of messages |
| rua | internal mailbox | Aggregate reports destination |
| ruf | internal mailbox | Forensic failure reports destination |
| fo | 1 | Generate report if SPF or DKIM fails |

**Why quarantine and not reject?**

`p=reject` on day one risks dropping legitimate email from any sender with
a misconfigured mail path. The correct enterprise approach is:

```
Week 1-2   p=none      Monitor only, no action
Week 3-4   p=quarantine  Failing mail goes to junk
Month 2+   p=reject    Failing mail is dropped entirely
```

This tenant started directly at `p=quarantine` after confirming SPF and DKIM
were valid. Migration to `p=reject` planned after monitoring aggregate reports.

**Verification:** All checks passed on MXToolbox DMARC analyser.

---

### 7. Test Users

Four test users created across key departments to enable realistic policy
testing throughout the project:

| Department | Count | Used For |
|---|---|---|
| Finance | 1 | DLP testing, CA policy targeting |
| HR | 1 | Insider risk simulation, policy scoping |
| IT | 1 | Endpoint testing, PIM practice |
| Executives | 1 | Privileged user policy testing |

**No licenses assigned by default.** Licenses will be assigned per workload
as each phase requires — a real enterprise cost optimisation practice.
Assigning E5 to every test account by default is wasteful and unrealistic.

---

### 8. Security Groups

| Group | Members | Purpose |
|---|---|---|
| Finance-Users | Finance department | Department-scoped policy targeting |
| IT-Admins | IT department + PIM account | Admin policy targeting |
| Executives | Executive team | Privileged user policy targeting |
| All-Employees | All test users | Broad policy targeting |
| Break-Glass Accounts | Emergency account | CA policy exclusion |

**Why groups instead of individual user targeting in policies?**

Targeting groups in CA policies, Intune and Defender means:
- Adding a new user to the company = add them to the right group = policies apply automatically
- No need to edit individual policies every time the org changes
- Mirrors how enterprise environments actually operate at scale

---

### 9. Audit Logging

| Property | Detail |
|---|---|
| Status | Enabled |
| Portal | Microsoft Purview |
| Retention | 1 year (E5 entitlement) |
| Scope | All user and admin activity |

**Why enable audit logging before anything else?**

Every configuration change made during this project is recorded in the audit
log with a timestamp and the identity of who made the change. This is the
evidence trail for the entire project — it proves that every policy was
intentionally configured, when, and by whom. In a real environment this is
your defence when an auditor or regulator asks what happened.

---

### 10. Security Defaults

| Property | Detail |
|---|---|
| Status | Disabled |
| Detected by | Microsoft Entra (automatic) |
| Reason | Conditional Access policies in use |
| Replacement | 12 Conditional Access policies (Phase 1) |

Security Defaults is Microsoft's basic pre-configured protection for tenants
with no dedicated IT security. It cannot run alongside Conditional Access —
they are mutually exclusive. Entra automatically disabled Security Defaults
upon detecting existing CA policies, confirming the tenant is ready for
granular policy control.

---

## Email Security Stack — Final State

```
homelab360.com email authentication

SPF   ✅  v=spf1 include:spf.protection.outlook.com -all
          Hard fail enforcement — unauthorised senders rejected

DKIM  ✅  Valid and Enabled in Microsoft Defender
          Digital signatures on all outbound mail
          Keys managed by Microsoft, rotated automatically

DMARC ✅  p=quarantine, pct=100
          100% coverage, reports to internal mailbox
          All checks verified via MXToolbox
```

---

## Lessons Learned

**Security Defaults vs Conditional Access**  
These cannot coexist. Attempting to create CA policies on a tenant with
Security Defaults active will either fail or produce unpredictable results.
Always confirm Security Defaults status before building CA policies.

**Break-glass account domain matters**  
Using a custom domain for break-glass accounts is a common mistake. The account
must be on a domain that survives any infrastructure failure — onmicrosoft.com
is the only reliable choice for this purpose.

**Audit logging is not automatic on new tenants**  
Many engineers skip this step assuming it is on by default. It is not.
Enable it before touching anything else — you cannot retroactively log
activity that happened before it was turned on.

**DMARC rollout is a process, not a one-time config**  
p=quarantine is a milestone, not the destination. Monitor aggregate reports,
confirm all legitimate mail sources are passing authentication, then move
to p=reject. Rushing to reject without monitoring causes legitimate mail loss.

---

## Next Phase

**[Phase 1 — Entra ID & Identity Security](../Phase-1/README.md)**

- Create 3 Named Locations (trusted office IP, allowed countries, high-risk country block)
- Deploy all 12 Conditional Access policies (report-only → enforce)
- Configure PIM just-in-time access for admin account
- Enable passwordless authentication (Microsoft Authenticator)
- Configure Self-Service Password Reset (SSPR)
- Configure Entra ID Protection sign-in and user risk policies

---

*All configurations implemented hands-on in a live Microsoft 365 E5 tenant.*
