# SX-03: Cryptography and PKI Controls Reference Guide

---

**Project Title:** Cryptography and PKI Controls Reference Guide for ClearBridge Health Management System
**System Reference:** ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD
**Framework / Standard:** NIST SP 800-57 Part 1 Rev 5 | NIST SP 800-52 Rev 2 | CompTIA Security+ SY0-701 Domain 3
**Author:** Ina Grace Kamara, ISSO, ClearBridge Technologies
**Date:** May 2024
**Status:** Complete

---

## Purpose

Cryptography is one of the most tested areas on the CompTIA Security+ SY0-701 exam and one of the most critical components of federal information system security. This document serves as a practical reference guide for all cryptographic controls deployed in ClearBridge HMS, mapping each control to the specific use case, algorithm, key length, and relevant NIST guidance.

The guide is organized by cryptographic function: symmetric encryption, asymmetric encryption and key exchange, hashing, digital signatures, and PKI certificate management.

---

## Section 1: Cryptography Inventory for ClearBridge HMS

| Control | Use Case | Algorithm | Key Length | NIST Reference | Implementation |
|---|---|---|---|---|---|
| Data at Rest (RDS) | PostgreSQL database encryption | AES-256-GCM | 256-bit | NIST SP 800-111, FIPS 140-3 | AWS KMS CMK + RDS storage encryption |
| Data at Rest (S3) | Backup and log storage encryption | AES-256 (SSE-KMS) | 256-bit | NIST SP 800-111 | AWS KMS CMK applied to all S3 buckets |
| Data at Rest (EBS) | EC2 instance root and data volumes | AES-256 (SSE-KMS) | 256-bit | NIST SP 800-111 | AWS KMS CMK; all volumes encrypted at creation |
| Data in Transit (TLS) | All HTTPS connections (ALB to clients, app to RDS) | TLS 1.3 preferred, TLS 1.2 minimum | N/A (cipher suite dependent) | NIST SP 800-52 Rev 2 | ALB TLS security policy: ELBSecurityPolicy-TLS13-1-2-2021-06 |
| TLS Cipher Suites | HTTPS encryption | ECDHE-ECDSA-AES256-GCM-SHA384, ECDHE-RSA-AES256-GCM-SHA384 | 256-bit | NIST SP 800-52 Rev 2 | Configured in ALB and application server |
| API Transaction Integrity | Treasury Disbursement API payload signing | HMAC-SHA256 | 256-bit (HMAC key) | NIST SP 800-107 Rev 1 | Application-layer signing; verified by Treasury endpoint |
| Key Management | All ClearBridge KMS keys | RSA-2048 (key wrapping), AES-256 (data keys) | 256-bit (data keys) | NIST SP 800-57 Part 1 | AWS KMS Customer Managed Keys (CMKs) |
| Key Rotation | Automatic rotation of KMS CMKs | Annual rotation | N/A | NIST SP 800-57 Part 1 | AWS KMS automatic rotation enabled |
| SSH Key Pair | EC2 administrative access (backup method) | RSA-2048 or ECDSA P-256 | 2048 or 256-bit | NIST SP 800-186 | Stored in AWS Secrets Manager; primary access via SSM |
| Code Signing | Application deployment artifacts | SHA-256 | 256-bit | NIST SP 800-131A Rev 2 | GitHub Actions pipeline signs container images |

---

## Section 2: Symmetric Encryption

**Definition:** Symmetric encryption uses the same key for both encryption and decryption.

**ClearBridge Primary Algorithm:** AES (Advanced Encryption Standard), specifically AES-256 in GCM (Galois/Counter Mode).

**Why AES-256-GCM:**
- AES is approved by NIST and FIPS for federal government use (FIPS 197).
- 256-bit key length provides 256-bit security strength, which is resistant to brute force with current and foreseeable quantum computing capabilities.
- GCM mode provides both confidentiality (encryption) and integrity (authentication tag). This is an Authenticated Encryption with Associated Data (AEAD) mode.
- GCM is more efficient than CBC mode and does not require padding.

**Use Cases in ClearBridge HMS:**
- RDS database encryption at rest
- S3 bucket object encryption
- EBS volume encryption

**NIST Reference:** NIST SP 800-38D (GCM mode), FIPS 197 (AES), NIST SP 800-111 (storage encryption)

---

## Section 3: Asymmetric Encryption and Key Exchange

**Definition:** Asymmetric encryption uses a public/private key pair. What is encrypted with the public key can only be decrypted with the private key.

**ClearBridge Use Cases:**

| Use Case | Algorithm | Key Length | Notes |
|---|---|---|---|
| TLS session key establishment | ECDHE (Elliptic Curve Diffie-Hellman Ephemeral) | P-256 or P-384 | Provides perfect forward secrecy; each session uses a new ephemeral key |
| TLS certificate (ALB) | RSA-2048 with SHA-256 signature | 2048-bit | AWS Certificate Manager (ACM) issues and manages the TLS certificate |
| Mutual TLS (Treasury API) | RSA-2048 client certificate | 2048-bit | ClearBridge presents client certificate to Treasury API; mutual authentication |
| SSH key pairs (backup admin) | ECDSA P-256 | 256-bit (equivalent to 3072-bit RSA) | Stored in Secrets Manager; used only when SSM is unavailable |

**Why Perfect Forward Secrecy (PFS) Matters:**
ECDHE provides PFS because the server's private key is not used to encrypt session keys. Even if an attacker records encrypted traffic and later compromises the server's private key, they cannot decrypt historical sessions. All ClearBridge TLS cipher suites require ECDHE for PFS.

**NIST Reference:** NIST SP 800-56A Rev 3 (key establishment), NIST SP 800-186 (elliptic curve parameters)

---

## Section 4: Hashing

**Definition:** A cryptographic hash function takes an input of any length and produces a fixed-length output (digest). A good hash function is deterministic, fast, preimage-resistant, and collision-resistant.

| Algorithm | Output Length | ClearBridge Use Case | Status |
|---|---|---|---|
| SHA-256 | 256 bits | HMAC transaction signing (Treasury API); code signing; TLS certificate signatures | Approved (NIST SP 800-131A Rev 2) |
| SHA-384 | 384 bits | TLS cipher suite MAC (ECDHE-ECDSA-AES256-GCM-SHA384) | Approved |
| SHA-1 | 160 bits | Legacy certificate on monitoring-01 (Finding 6 from vulnerability scan) | Deprecated. Remediation in progress. |
| MD5 | 128 bits | Not in use | Not approved for cryptographic purposes |

**Why SHA-1 is Deprecated:**
SHA-1 was formally deprecated by NIST in NIST SP 800-131A Rev 2 (2019). Theoretical collision attacks against SHA-1 became practical in 2017 (SHAttered attack). ClearBridge is migrating the monitoring server certificate to SHA-256 by December 31, 2024.

**HMAC Explained:**
HMAC (Hash-based Message Authentication Code) combines a cryptographic key with a hash function. ClearBridge uses HMAC-SHA256 for Treasury API transaction signing: the shared secret is combined with the transaction payload to produce a 256-bit authentication tag. This provides both integrity (the data was not modified) and authenticity (only the party with the shared secret could have generated the tag).

---

## Section 5: PKI and Certificate Management

**Definition:** Public Key Infrastructure (PKI) is the system of policies, processes, and technologies used to create, manage, distribute, use, and revoke digital certificates.

### ClearBridge HMS PKI Overview

| Certificate | Issued By | Type | Use | Expiration | Management |
|---|---|---|---|---|---|
| ALB TLS Certificate (*.clearbridge-hms.hhs.gov) | Amazon Certificate Manager (ACM) | Domain Validation (DV) | HTTPS termination at ALB | Auto-renewed by ACM | AWS ACM |
| Internal Code Signing Certificate | GitHub Actions (internal CA) | Code signing | Container image signing | 1 year | GitHub Secrets |
| Mutual TLS Client Certificate (Treasury API) | Treasury PKI | Client authentication | Authenticate ClearBridge to Treasury API | 2 years | AWS Secrets Manager |
| Monitoring Server Certificate (monitoring-01) | Self-signed | Self-signed | Internal only | 2 years | Manual (action item) |

### Certificate Lifecycle Management

**Issuance:** AWS ACM automatically provisions TLS certificates for public-facing endpoints. Certificates are validated via DNS validation with Route 53.

**Renewal:** ACM automatically renews certificates 60 days before expiration. ISSO receives email notification if renewal fails. The monitoring server self-signed certificate requires manual renewal (tracked as a remediation action).

**Revocation:** If a private key is believed compromised, the certificate is revoked via ACM (for ACM-managed certificates) or by contacting the issuing CA. Revocation status is checked via OCSP (Online Certificate Status Protocol) or CRL (Certificate Revocation List).

**Key Storage:** All private keys for ACM-managed certificates are stored and managed by AWS and are never exported. The Treasury API mutual TLS private key is stored in AWS Secrets Manager with access restricted to the application service account only.

---

## Section 6: Quantum Computing and Cryptographic Agility

NIST finalized the first post-quantum cryptography (PQC) standards in 2024 (FIPS 203, 204, 205). While current quantum computers cannot break AES-256 or ECDSA at current key lengths, NIST recommends that organizations begin planning for cryptographic agility: the ability to replace cryptographic algorithms without rebuilding entire systems.

ClearBridge HMS is monitoring NIST PQC guidance. The current architecture uses algorithm-agnostic libraries (OpenSSL 3.x) that support PQC extensions. No immediate migration is required, but the ISSO will include PQC readiness in the 2025 annual risk assessment.

---

## Interview Defense Notes

- **What is the difference between symmetric and asymmetric encryption?** Symmetric uses the same key for encryption and decryption (faster, used for bulk data: AES). Asymmetric uses a public/private key pair (slower, used for key exchange and signatures: RSA, ECDSA). TLS uses asymmetric for key exchange and symmetric for the bulk data stream.
- **What is perfect forward secrecy and why does ClearBridge require it?** PFS means that compromising the server's private key in the future cannot decrypt previously recorded sessions. ECDHE provides PFS by generating a new ephemeral key for each session. Without PFS, an attacker who records encrypted sessions today can decrypt them later if they ever compromise the server's private key.
- **What is HMAC and how is it different from a regular hash?** A regular hash provides integrity (has the data changed) but not authenticity (who created it). HMAC adds a secret key to the hash, so only parties with the key can produce a valid HMAC. ClearBridge uses HMAC-SHA256 for the Treasury API to ensure payment instructions came from ClearBridge, not an impersonator.
- **Why is AES-256 preferred over AES-128 for federal systems?** NIST SP 800-57 recommends AES-256 for data that needs to remain confidential beyond 2030. For federal health data with long retention requirements, AES-256 provides a larger security margin against future advances in computing power, including potential quantum threats.
- **What is a certificate transparency log?** It is a public, append-only log of all SSL/TLS certificates issued by certificate authorities. It was created to detect mis-issuance and rogue certificates. Web browsers increasingly require that certificates appear in transparency logs. AWS ACM automatically submits certificates to transparency logs.

---

*Prepared by: Ina Grace Kamara, ISSO, ClearBridge Technologies*
*System: ClearBridge Health Management System (HMS) | HHS-CB-HMS-2024-MOD*
*[GitHub Portfolio](https://github.com/kamaraina277)*
