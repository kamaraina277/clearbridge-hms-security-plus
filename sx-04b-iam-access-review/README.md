# SX-04b: IAM Access Review and Provisioning Workflow

---

**Project Title:** IAM Access Review and Provisioning Workflow for ClearBridge Health Management System
**System Reference:** ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD
**Framework / Standard:** NIST SP 800-53 Rev 5 AC-2, AC-3, IA-2, IA-5 | CompTIA Security+ SY0-701 Domain 4
**Author:** Ina Grace Kamara, ISSO, ClearBridge Technologies
**Date:** May 2024
**Status:** Complete

---

## Purpose

Identity and Access Management (IAM) is one of the highest-impact security disciplines in federal cloud systems. Access creep, orphaned accounts, and over-privileged roles are among the most common findings in federal system security assessments. This document covers two related topics: the formal account provisioning workflow for ClearBridge HMS, and the quarterly access review process used to detect and remediate access issues.

---

## Section 1: Identity Architecture Overview

ClearBridge HMS uses a federated identity model. User identity is managed by HHS, not by ClearBridge. AWS IAM manages infrastructure access.

| Identity Layer | Technology | Who Manages It | Purpose |
|---|---|---|---|
| End-user authentication | HHS Central IdP (Okta + Active Directory) | HHS IAM Team | Authenticates federal beneficiaries and agency staff to HMS application |
| Federation | SAML 2.0 assertion from HHS IdP to ClearBridge HMS | HHS IdP, ClearBridge app config | Single sign-on; no passwords stored in ClearBridge HMS |
| MFA | Okta Verify (software token) or PIV card | HHS IdP | Required for all users; enforced at IdP level |
| Infrastructure access (AWS) | AWS IAM roles and policies | ClearBridge ISSO and Cloud Ops | Controls access to EC2, RDS, S3, CloudTrail |
| Privileged access | AWS IAM + Systems Manager Session Manager | ClearBridge Cloud Ops | Admin access to all infrastructure; hardware MFA required |
| Application-level RBAC | ClearBridge HMS application roles | ClearBridge ISSO (provisioning) | Controls what data and functions each user can access within HMS |

---

## Section 2: Account Provisioning Workflow

### New User Provisioning

The following workflow applies to all new users requiring access to ClearBridge HMS.

**Step 1: Access Request**
The user's HHS supervisor submits an Access Request Form (ARF) to the ISSO via ServiceNow. The ARF must include: user's name, HHS employee ID, job function, required HMS role (see role definitions below), and supervisor signature. Requests without all fields are rejected.

**Step 2: ISSO Review**
The ISSO reviews the ARF and verifies: the requested role is appropriate for the user's job function (least privilege principle), the user has completed the required security awareness training, and the user has signed the ClearBridge HMS Rules of Behavior (RoB).

**Step 3: Account Creation**
For application access: ISSO creates the user's HMS application role in the ClearBridge HMS admin panel, linked to the user's HHS IdP account. No passwords are created in ClearBridge HMS; authentication is federated.
For infrastructure access (ClearBridge staff only): Cloud Ops creates an IAM user with a specific role assignment; hardware MFA token is issued.

**Step 4: Confirmation**
ISSO sends a confirmation to the user and supervisor. A record of the provisioning is logged in ServiceNow with the ARF, role assigned, and date.

**Step 5: On-Boarding Documentation**
ISSO files: signed ARF, signed Rules of Behavior, training completion certificate.

### Provisioning Timelines

| Request Type | Target Completion |
|---|---|
| New federal beneficiary (standard) | 3 business days |
| New agency staff member | 2 business days |
| New ClearBridge employee (infrastructure access) | 1 business day |
| Emergency access (AO approval required) | 4 hours |

---

## Section 3: HMS Application Role Definitions

| Role | Description | Data Access | Sensitive Operations |
|---|---|---|---|
| Beneficiary | Federal beneficiary self-service | Own records only | View eligibility status, benefit payments |
| Agency Staff - Read Only | Regional office staff; inquiry only | Records for assigned region | No write access |
| Agency Staff - Case Manager | Regional office case managers | Records for assigned region | Update eligibility records; submit payment requests |
| Agency Staff - Supervisor | Regional office supervisors | All records in assigned region | Override case manager decisions; access audit logs |
| HHS Administrator | HHS central office | All records, all regions | System configuration; user management |
| ClearBridge ISSO | Security monitoring | Audit logs, security events | Access review; incident investigation |
| ClearBridge Cloud Ops | Infrastructure administration | AWS console (no application data) | EC2, RDS, S3, networking administration |
| 3PAO Assessor | Assessment only (time-limited) | Audit logs, scan results | Read-only; expires after assessment period |

---

## Section 4: Quarterly Access Review Process

NIST SP 800-53 control AC-2(j) requires that organizations review user accounts at a defined frequency. ClearBridge HMS conducts quarterly access reviews.

### Review Schedule

| Quarter | Review Period | Completion Deadline | Reviewer |
|---|---|---|---|
| Q1 | January 1 - March 31 | April 15 | Ina Grace Kamara, ISSO |
| Q2 | April 1 - June 30 | July 15 | Ina Grace Kamara, ISSO |
| Q3 | July 1 - September 30 | October 15 | Ina Grace Kamara, ISSO |
| Q4 | October 1 - December 31 | January 15 | Ina Grace Kamara, ISSO |

### Q2 2024 Access Review Results (Sample)

**Review Date:** July 10, 2024
**Review Scope:** All 342 active ClearBridge HMS user accounts

| Finding Type | Count | Action Required |
|---|---|---|
| Accounts with no login in 90+ days | 8 | Disable accounts; notify supervisors |
| Accounts with role mismatch (user transferred roles) | 3 | Adjust role to match current function |
| Accounts for users who separated (not flagged by HR) | 2 | Immediate disable; incident investigation |
| Accounts with shared access (not permitted) | 0 | No action |
| Over-privileged accounts (role higher than needed) | 4 | Downgrade to appropriate role |
| Accounts with expired Rules of Behavior | 5 | Send renewal request; disable in 7 days if not signed |
| **Total findings** | **22** | All actioned within 5 business days |

### Access Review Certification

After the review, the ISSO produces an Access Review Certification memo stating: the review was completed, the total number of accounts reviewed, the number of findings, the actions taken, and the date. This memo is submitted to the system owner (Dr. Carolyn M. Fletcher) and filed as evidence for NIST AC-2 and FedRAMP ConMon.

---

## Section 5: Account Deprovisioning Workflow

Timely account deprovisioning is critical for preventing unauthorized access by former employees or contractors.

### Deprovisioning Triggers

| Trigger | Notification Source | Required Action Timeframe |
|---|---|---|
| Employee termination | HHS HR or ClearBridge HR | Disable account within 4 hours of notification |
| Employee transfer (role change) | Supervisor via ServiceNow | Update role within 2 business days |
| Contract expiration | Contract management system | Disable access on contract end date |
| Security incident (credential compromise) | ISSO determination | Immediate (within 15 minutes) |
| Long-term leave (90+ days) | HR notification | Disable account; re-enable on return with new access review |

### Deprovisioning Steps

1. ISSO receives notification (HR email, ServiceNow ticket, or direct ISSO determination for security incidents).
2. ISSO disables the user's HMS application account in the admin panel.
3. If the user has AWS IAM access: Cloud Ops disables the IAM user and rotates any access keys.
4. ISSO notifies HHS IAM team to disable the user's HHS IdP account if the user is leaving HHS entirely.
5. ISSO documents the deprovisioning in ServiceNow with timestamp, reason, and requestor.
6. After 30 days, the disabled account is reviewed for archival or permanent deletion.

---

## Section 6: Privileged Access Management

Privileged accounts (ClearBridge Cloud Ops and ISSO) have elevated access and require additional controls.

| Control | Implementation |
|---|---|
| Hardware MFA | YubiKey required for all privileged AWS console access; enforced by IAM policy condition |
| Just-in-Time Access (in progress) | AWS Systems Manager Change Manager for requesting temporary elevated privileges; target October 2024 |
| Session recording | All SSM Session Manager sessions logged to CloudWatch and S3; reviewed monthly |
| Separate admin accounts | Cloud Ops staff have separate administrative IAM users distinct from their regular user accounts |
| Monthly privileged access review | ISSO reviews all privileged accounts monthly, not just quarterly |
| No shared credentials | Each privileged user has their own named IAM user; shared accounts prohibited |

---

## Interview Defense Notes

- **What is the principle of least privilege and how does it apply to IAM?** Least privilege means users should have only the minimum access necessary to do their job. In ClearBridge HMS, a regional case manager can only access records for their region, not all regions. An agency staff member with a read-only role cannot submit payment requests. When you review accounts, you look for violations of this principle.
- **What is an orphaned account and why is it a risk?** An orphaned account is one that still exists in the system after the person it belongs to has left the organization. An attacker who discovers orphaned credentials can use them without the original user's knowledge. ClearBridge found 2 orphaned accounts in the Q2 2024 review, which triggered an immediate disable and investigation.
- **What is role-based access control (RBAC)?** RBAC assigns permissions to roles (job functions) rather than individual users. You assign users to roles. This makes it easier to manage access: when a user changes jobs, you change their role rather than reviewing individual permissions. ClearBridge HMS has 8 defined roles each with specific data access and operation permissions.
- **What is the difference between authentication and authorization?** Authentication verifies who you are (identity). Authorization determines what you are allowed to do (access rights). In ClearBridge HMS, the HHS IdP handles authentication (SAML, MFA). The HMS application handles authorization (RBAC roles). A user can be authenticated but still denied access to specific data based on their role.
- **How would you handle a situation where a manager asks you to give their employee access to a higher-privileged role for a temporary project?** Document the request formally in ServiceNow, get the manager's written approval, set an expiration date for the elevated access, and schedule a review after the project ends. Never grant elevated access based on a verbal request without documentation, and never grant it without a defined end date.

---

*Prepared by: Ina Grace Kamara, ISSO, ClearBridge Technologies*
*System: ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD*
*[GitHub Portfolio](https://github.com/kamaraina277)*
