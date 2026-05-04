# SX-04a: Incident Response Playbook (Phishing Attack)

---

**Project Title:** Incident Response Playbook: Phishing Attack on ClearBridge HMS
**System Reference:** ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD
**Framework / Standard:** NIST SP 800-61 Rev 2 | CompTIA Security+ SY0-701 Domain 4 (Security Operations)
**Author:** Ina Grace Kamara, ISSO, ClearBridge Technologies
**Date:** May 2024
**Status:** Complete

---

## Purpose

An Incident Response Playbook is a documented, step-by-step procedure for responding to a specific type of security incident. This playbook covers the full lifecycle of a phishing attack targeting ClearBridge HMS users, from initial detection through post-incident review.

Phishing is the most common initial access vector for threat actors targeting federal systems. For ClearBridge HMS, a successful phishing attack could result in credential compromise, unauthorized access to PHI, or deployment of ransomware. This playbook follows the NIST SP 800-61 Rev 2 incident response lifecycle: Preparation, Detection and Analysis, Containment, Eradication and Recovery, and Post-Incident Activity.

---

## Incident Classification

| Field | Detail |
|---|---|
| Incident Type | Phishing (credential harvesting or malware delivery) |
| Affected System | ClearBridge HMS user accounts; potentially EC2 or RDS if credentials are used |
| NIST SP 800-61 Category | Phishing / Social Engineering |
| FedRAMP Severity Threshold | Moderate (credential phishing) to High (if credentials used for data access) |
| Reporting Requirement | Moderate: Report to HHS CISO within 1 hour; US-CERT within 1 hour per NIST SP 800-61 |

---

## Phase 1: Preparation

### Pre-Incident Controls

Before any incident occurs, the following controls reduce the likelihood and impact of a phishing attack:

| Control | Description | Purpose |
|---|---|---|
| Phishing simulation training | Quarterly simulated phishing campaigns via KnowBe4 | Measures staff susceptibility; reinforces recognition skills |
| MFA enforcement | All ClearBridge HMS users require MFA via HHS IdP (Okta Verify or PIV card) | Even if credentials are phished, MFA prevents immediate account takeover |
| Email filtering | Microsoft 365 Defender ATP with Safe Links and Safe Attachments | Blocks known phishing URLs and malicious attachments before delivery |
| SPF/DKIM/DMARC | DNS records configured for clearbridge.com and hhs.gov domains | Reduces spoofed email from ClearBridge and HHS domains |
| SIEM alerting | Splunk alerts on: impossible travel logins, logins from new geographies, failed MFA pushes | Detects credential abuse after successful phishing |

### Incident Response Team

| Role | Name | Contact | Responsibility |
|---|---|---|---|
| Incident Commander | Ina Grace Kamara, ISSO | ikamara@clearbridge.com | Overall incident coordination; AO communication |
| Technical Lead | Kevin D. Ashworth, Cloud Ops | kashworth@clearbridge.com | Technical investigation; containment actions |
| HHS Liaison | Walter J. Drummond, SAISO | wdrummond@hhs.gov | HHS CISO notification; US-CERT coordination |
| Legal/Privacy | Angela N. Weiss, Privacy Officer | aweiss@hhs.gov | PHI breach assessment; legal obligations |
| Communications | Brian A. Holloway, VP Engineering | bholloway@clearbridge.com | Executive notification; media response if required |

---

## Phase 2: Detection and Analysis

### Detection Sources

Phishing attacks may be detected through several signals:

| Signal | Source | Example |
|---|---|---|
| User report | Help desk ticket or direct email to ISSO | "I clicked a link that asked for my HHS password" |
| MFA push anomaly | Splunk SIEM alert | User receiving unexpected MFA push requests; push fatigue attack |
| Impossible travel alert | Splunk / HHS IdP | Login from Virginia followed 5 minutes later by login from Eastern Europe |
| Failed login spike | SIEM | 15 failed login attempts on a single account in 10 minutes |
| Email security alert | Microsoft 365 Defender | Malicious URL detected in email; user clicked before quarantine |
| Endpoint alert | CrowdStrike Falcon | Suspicious process launched from Outlook.exe (macro-based malware) |

### Initial Analysis Steps

When a potential phishing incident is detected:

1. **Verify the signal.** Confirm the alert is real. Review the SIEM event details, email headers, or user report.
2. **Identify the scope.** How many users received the email? How many clicked? Use Microsoft 365 admin to pull message trace.
3. **Check for credential use.** Review HHS IdP logs: did the potentially compromised account log in to ClearBridge HMS after the phishing email? From where?
4. **Assess MFA status.** Did MFA block the login attempt? Or was MFA bypassed (adversary-in-the-middle phishing kit)?
5. **Check for data access.** If the attacker got a session token, review CloudTrail and application logs for what data was accessed.
6. **Classify the incident.** Use the table below to determine initial severity.

### Incident Severity Classification

| Scenario | Severity | Response Time |
|---|---|---|
| User received phishing email; did not click | Low | 24 hours |
| User clicked link; entered credentials; MFA blocked login | Moderate | 1 hour |
| User clicked link; credentials and MFA session token stolen | High | Immediate (15 minutes) |
| Attacker used stolen credentials to access ClearBridge HMS data | High/Critical | Immediate (15 minutes) |
| Malware deployed on user workstation with HMS access | High/Critical | Immediate (15 minutes) |

**Severity 5 (Critical) Declaration:** If PHI has been accessed by an unauthorized party, this is a potential HIPAA breach. Declare Critical, notify Privacy Officer within 15 minutes, and initiate breach assessment procedure immediately.

---

## Phase 3: Containment

### Immediate Containment Actions (within 15 minutes of High/Critical declaration)

| Step | Action | Who | Tool |
|---|---|---|---|
| 1 | Disable the affected user account in HHS IdP | HHS IAM team (ISSO coordinates) | HHS Active Directory / Okta admin |
| 2 | Revoke all active sessions for the affected account | HHS IAM team | Okta session revocation |
| 3 | Rotate the user's IAM credentials if they have AWS IAM access | Kevin D. Ashworth | AWS IAM console |
| 4 | Quarantine the affected workstation (if malware suspected) | Kevin D. Ashworth + IT | CrowdStrike Falcon network containment |
| 5 | Block the phishing domain/IP in AWS WAF and Microsoft 365 Defender | Kevin D. Ashworth | WAF console; M365 Defender portal |
| 6 | Pull message trace and quarantine remaining emails | Kevin D. Ashworth | M365 admin center |
| 7 | Notify HHS CISO | Ina Grace Kamara, ISSO | Phone call within 1 hour of High/Critical declaration |

### Evidence Preservation

Before performing eradication actions, preserve:
- Memory image of affected workstation (if malware suspected)
- Network packet capture from VPC Flow Logs for the period of the incident
- CloudTrail logs for the affected user's session
- HHS IdP authentication logs
- Email headers and full email content from M365 message trace

---

## Phase 4: Eradication and Recovery

### Eradication Steps

| Step | Action | Notes |
|---|---|---|
| 1 | Confirm malware removed from affected workstation | CrowdStrike scan results; if unconfirmed, reimage the workstation |
| 2 | Reset user password in HHS IdP | Require new password that has never been used |
| 3 | Re-enroll MFA device | Do not re-use the same MFA device if it may have been compromised |
| 4 | Verify no persistence mechanisms installed | Check scheduled tasks, startup items, browser extensions |
| 5 | Confirm malicious domain blocked across all systems | WAF, Defender, DNS filtering |

### Recovery Steps

| Step | Action | Notes |
|---|---|---|
| 1 | Re-enable user account | Only after workstation confirmed clean and credentials reset |
| 2 | Monitor user account closely for 30 days | Additional SIEM rules scoped to this account |
| 3 | If data was accessed: complete HIPAA breach assessment | Privacy Officer leads; 60-day notification clock may start |
| 4 | Submit US-CERT report | Required if Moderate or higher; use OMB reporting format |
| 5 | Document full incident timeline | Create ServiceNow incident record with all evidence |

---

## Phase 5: Post-Incident Activity

### Lessons Learned Meeting

Schedule within 2 weeks of incident closure. Attendees: ISSO, Cloud Ops Lead, Privacy Officer, HHS SAISO.

Agenda:
1. Timeline review: detection, containment, eradication times
2. Root cause analysis: Why did the phishing email bypass filters? Why did the user click?
3. Control gaps: What control would have caught this earlier or prevented it entirely?
4. Action items: Specific improvements with owners and deadlines

### Post-Incident Metrics to Track

| Metric | Target | Notes |
|---|---|---|
| Time to detect | Less than 1 hour | From phishing click to detection |
| Time to contain | Less than 15 minutes (High/Critical) | From declaration to account disabled |
| Time to notify AO | Less than 1 hour | Per ClearBridge ConMon plan |
| Phishing click rate (training) | Less than 5% | Measured in quarterly simulation campaigns |

---

## Interview Defense Notes

- **What are the phases of the NIST SP 800-61 incident response lifecycle?** Preparation, Detection and Analysis, Containment, Eradication and Recovery, and Post-Incident Activity. All five phases are present in this playbook. The lifecycle is a cycle, not a line: lessons from Post-Incident Activity feed back into Preparation.
- **What is the difference between containment and eradication?** Containment stops the bleeding: it isolates the affected system and prevents the attacker from doing more damage. Eradication removes the threat: it wipes the malware, resets credentials, and closes the access vector. You must contain before you eradicate, or the attacker can re-establish access while you are cleaning up.
- **What is a HIPAA breach notification obligation and when is it triggered?** If PHI is accessed or acquired by an unauthorized person, HIPAA requires notification to affected individuals (within 60 days), HHS (annually for small breaches, immediately for large), and potentially media if more than 500 individuals are affected in a state. ClearBridge must assess whether the data accessed qualifies as a breach.
- **What is MFA fatigue and how can you defend against it?** MFA fatigue is when an attacker who has stolen credentials sends repeated MFA push notifications hoping the victim approves one out of frustration. Defenses include: number matching (user must enter a displayed code in the MFA app), additional context (app shows location), limiting notification volume, and training users to never approve unexpected MFA pushes.
- **Why does an ISSO need a playbook for phishing rather than just a general IR plan?** A general IR plan defines the process. A playbook provides step-by-step instructions for a specific attack type. Without a playbook, each incident relies on whoever is on call to improvise the steps under pressure. With a playbook, a junior analyst can follow documented steps and know exactly who to call and what to do.

---

*Prepared by: Ina Grace Kamara, ISSO, ClearBridge Technologies*
*System: ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD*
*[GitHub Portfolio](https://github.com/kamaraina277)*
