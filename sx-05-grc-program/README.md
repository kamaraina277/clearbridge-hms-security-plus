# SX-05: GRC Program Overview

---

**Project Title:** GRC Program Overview for ClearBridge Health Management System
**System Reference:** ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD
**Framework / Standard:** NIST SP 800-53 Rev 5 | HIPAA | GDPR (reference only) | CompTIA Security+ SY0-701 Domain 5
**Author:** Ina Grace Kamara, ISSO, ClearBridge Technologies
**Date:** May 2024
**Status:** Complete

---

## Purpose

Governance, Risk, and Compliance (GRC) is the overarching program that ties together an organization's security governance structure, risk management processes, and regulatory compliance activities. This document provides a GRC program overview for ClearBridge HMS, covering: the Acceptable Use Policy (AUP), a HIPAA to NIST SP 800-53 control mapping, the risk assessment summary, and a sample compliance dashboard.

---

## Section 1: Acceptable Use Policy (AUP)

### ClearBridge Health Management System Acceptable Use Policy

**Version:** 2.0
**Effective Date:** January 15, 2024
**Owner:** Ina Grace Kamara, ISSO, ClearBridge Technologies
**Approved By:** Brian A. Holloway, VP Engineering, ClearBridge Technologies

---

**1. Purpose**

This Acceptable Use Policy (AUP) defines the rules and restrictions governing use of the ClearBridge Health Management System (HMS) by all authorized users including federal beneficiaries, HHS agency staff, and ClearBridge employees. Violations of this policy may result in loss of access privileges, disciplinary action, and civil or criminal liability.

**2. Scope**

This policy applies to all individuals with authorized access to ClearBridge HMS, including: federal beneficiaries, HHS regional office staff, HHS central administrators, ClearBridge Technologies employees, and any contractors or third parties with system access.

**3. Authorized Use**

Authorized users may only use ClearBridge HMS to perform functions that are within the scope of their assigned role. Authorized uses include: viewing or updating personal benefit eligibility information (beneficiaries), processing benefit requests and managing case records (agency staff), and administering system configuration and security (ClearBridge staff).

**4. Prohibited Activities**

The following activities are strictly prohibited:

- Accessing, viewing, copying, or sharing any data you are not authorized to access based on your role.
- Sharing your login credentials with any other person.
- Using ClearBridge HMS for personal gain, commercial purposes, or activities unrelated to your assigned duties.
- Attempting to bypass, circumvent, or test security controls.
- Transmitting malware, phishing links, or malicious content through ClearBridge HMS.
- Accessing ClearBridge HMS from a device that is not government-furnished or personal equipment approved by ClearBridge Technologies.
- Downloading, printing, or exporting beneficiary data except as specifically required for your job function and authorized by your supervisor.
- Using unauthorized software, browser extensions, or tools to interact with the system.

**5. Remote Access**

All remote access to ClearBridge HMS requires MFA authentication through the HHS IdP. Users must not access the system from public Wi-Fi without a VPN connection. Users must lock their devices when stepping away.

**6. Reporting Obligations**

All users must report suspected security incidents, unauthorized access attempts, phishing emails, or unusual system behavior to the ISSO (ikamara@clearbridge.com) and the HHS help desk immediately. Do not wait to determine whether an incident is "real" before reporting.

**7. Consequences of Violations**

Violations of this AUP may result in: immediate suspension or termination of access, disciplinary action by HHS or ClearBridge HR, civil liability under HIPAA and the Privacy Act, and criminal prosecution under 18 U.S.C. 1030 (Computer Fraud and Abuse Act).

**8. Acknowledgment**

All users must sign an acknowledgment of this policy before receiving access. Re-acknowledgment is required annually.

---

## Section 2: HIPAA to NIST SP 800-53 Control Mapping

HIPAA Security Rule requires covered entities and business associates to implement administrative, physical, and technical safeguards to protect electronic PHI (ePHI). The table below maps key HIPAA Security Rule requirements to corresponding NIST SP 800-53 Rev 5 controls implemented in ClearBridge HMS.

| HIPAA Security Rule Requirement | HIPAA Reference | NIST SP 800-53 Controls | ClearBridge Implementation |
|---|---|---|---|
| Assign unique user identification | 164.312(a)(2)(i) | IA-2, IA-4 | Federated identity via HHS IdP; unique user IDs; no shared accounts |
| Emergency access procedure | 164.312(a)(2)(ii) | CP-2, IR-4 | Emergency access documented in Contingency Plan; ISSO-approved break-glass procedure |
| Automatic logoff | 164.312(a)(2)(iii) | SC-10 | 30-minute session timeout enforced at application layer |
| Encryption and decryption (at rest) | 164.312(a)(2)(iv) | SC-28 | AES-256 via AWS KMS for RDS, S3, EBS |
| Audit controls (activity logs) | 164.312(b) | AU-2, AU-3, AU-12 | CloudTrail, CloudWatch, application audit logs; SIEM integration |
| Integrity (ePHI not altered improperly) | 164.312(c)(1) | SI-7, SC-8 | TLS for data in transit; database constraints; transaction logging |
| Mechanism to authenticate ePHI | 164.312(c)(2) | SI-7, AU-10 | Digital signatures on critical transactions; immutable logs |
| Transmission security (encryption in transit) | 164.312(e)(2)(ii) | SC-8 | TLS 1.2+ enforced; all data in transit encrypted |
| Access control (who can access ePHI) | 164.312(a)(1) | AC-2, AC-3, AC-17 | RBAC; quarterly access reviews; federated authentication |
| Person or entity authentication | 164.312(d) | IA-2, IA-5 | MFA required; PIV or Okta Verify; hardware MFA for privileged users |
| Workforce training | 164.308(a)(5) | AT-2, AT-3 | Annual security awareness training; role-specific training for ISSO |
| Risk analysis | 164.308(a)(1)(ii)(A) | RA-3, RA-5 | Annual risk assessment; monthly vulnerability scans |
| Sanction policy | 164.308(a)(1)(ii)(C) | PS-8 | Documented disciplinary policy; AUP acknowledgment required |
| Business associate agreements | 164.308(b)(1) | SA-9 | BAA executed with AWS; MSA includes security requirements |
| Breach notification | 164.400 - 164.414 | IR-6, IR-8 | Breach assessment procedure in IRP; Privacy Officer oversight |

---

## Section 3: HIPAA vs. GDPR Reference Comparison

While ClearBridge HMS is a U.S. federal system and does not process data subject to GDPR, the table below provides a conceptual comparison that is relevant to the Security+ curriculum and useful for organizations that operate internationally.

| Principle | HIPAA | GDPR |
|---|---|---|
| Jurisdiction | U.S. federal law; applies to covered entities and BAs | EU law; applies to anyone processing EU data subjects |
| Covered Data | ePHI (electronic Protected Health Information) | Personal data (broad definition including any identifier) |
| Consent | Not the primary basis; minimum necessary standard | Explicit consent is one of six lawful bases |
| Breach Notification | Covered entities: 60 days; large breaches: immediate | 72 hours to supervisory authority; high-risk breaches: notify individuals |
| Right to Access | Patients can request their records | Data subjects have a right to access, portability, and erasure |
| Enforcement | HHS Office for Civil Rights; fines up to $1.9M per category | National data protection authorities; fines up to 4% of global annual turnover |
| Privacy Officer | Not mandated but best practice | DPO required for large-scale processing |

---

## Section 4: Risk Assessment Summary

This section summarizes the current risk posture of ClearBridge HMS based on the most recent annual risk assessment (April 2024) and Q2 2024 ConMon activities.

| Risk Category | Number of Identified Risks | Residual Risk Level | Trend |
|---|---|---|---|
| Access Control | 2 (RISK-001, RISK-010) | Moderate, Low | Improving (RISK-001 treatment underway) |
| Data Integrity | 1 (RISK-004) | Low | Stable |
| Availability | 1 (RISK-007) | Low | Stable |
| Malware / Ransomware | 1 (RISK-002) | Low | Stable |
| Vulnerability Management | 1 (RISK-003) | Low | Improving (Inspector integration in progress) |
| Insider Threat | 2 (RISK-005, RISK-009) | Low | Improving (DLP enhancements in progress) |
| Third-Party / Supply Chain | 2 (RISK-008, RISK-011) | Low | Stable |
| Regulatory Compliance | 2 (RISK-012) | Low | Stable |

**Overall Risk Rating: LOW to MODERATE**

No risks are currently above the Moderate threshold. Two risks (RISK-001, RISK-005) are at Moderate residual and are being actively treated.

---

## Section 5: Compliance Dashboard (Sample)

**Reporting Period:** Q2 2024 (April - June 2024)
**Dashboard Date:** July 15, 2024

| Compliance Area | Status | Metric | Target | Current |
|---|---|---|---|---|
| FedRAMP ATO | Active | ATO status | Active | Active (granted July 22, 2024) |
| POA&M High-Risk Items | On Track | High-risk items open past 30-day SLA | 0 | 0 (both closed July 31) |
| Monthly Vulnerability Scans | On Track | Scan completion rate | 100% | 100% |
| Security Awareness Training | Minor Gap | Training completion rate | 100% | 97% (3 users in progress) |
| Critical Patch SLA | On Track | Critical vulnerabilities past 15-day SLA | 0 | 0 |
| Incident Response SLA | On Track | Moderate+ incidents past 1-hour notification SLA | 0 | N/A (no Moderate+ incidents in period) |
| Quarterly Access Review | On Track | Review completion | Q2 complete by July 15 | Complete July 10, 2024 |
| Risk Assessment | On Track | Annual risk assessment | Due April 2025 | Complete April 2024 |
| HIPAA Compliance | On Track | Open HIPAA-related findings | 0 | 0 |
| POA&M Total Items | On Track | Items open past target date | 0 | 0 (all items within SLA) |

**Overall Compliance Status: GREEN (with one minor gap in training completion)**

---

## Interview Defense Notes

- **What is GRC and why is it a domain in Security+?** GRC stands for Governance, Risk, and Compliance. It is the framework that ties together how an organization makes security decisions (governance), how it identifies and manages threats (risk), and how it meets its legal and regulatory obligations (compliance). Security+ covers GRC because security professionals at any level need to understand the business and legal context for technical controls.
- **What is the purpose of an Acceptable Use Policy?** An AUP sets the expectations for how users can and cannot use an information system. It serves as both a deterrent (users know the rules) and as a legal foundation for disciplinary action if rules are violated. Without an AUP, it is harder to hold users accountable. FedRAMP requires all users to acknowledge the AUP before receiving access.
- **How does HIPAA relate to NIST SP 800-53?** HIPAA sets the requirement: protect ePHI. NIST SP 800-53 provides the controls: specific security practices that collectively satisfy that requirement. For ClearBridge HMS, the NIST controls are the implementation mechanism for HIPAA compliance. The mapping table shows which controls address which HIPAA rules.
- **What is a compliance dashboard and who uses it?** A compliance dashboard is a summary view of the organization's compliance status across key areas. The AO and system owner use it to understand the overall health of the security program without reading every report. A red item on the dashboard signals a problem that needs attention. A green dashboard does not mean zero risk; it means known risks are being managed.
- **What is the difference between governance, risk, and compliance?** Governance is the policies, roles, and structures that guide security decision-making. Risk is the process of identifying threats, assessing their likelihood and impact, and deciding how to respond. Compliance is verifying that the organization meets its legal, regulatory, and contractual obligations. All three are necessary: governance without risk management is just paper; compliance without governance lacks direction.

---

*Prepared by: Ina Grace Kamara, ISSO, ClearBridge Technologies*
*System: ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD*
*[GitHub Portfolio](https://github.com/kamaraina277)*
