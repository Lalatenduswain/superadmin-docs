# PCI DSS v4.0 Compliance Guide

This document maps the SuperAdmin SaaS platform's security controls to **PCI DSS v4.0** (Payment Card Industry Data Security Standard). It covers all 12 requirements with specific implementation details, scope reduction strategies, and Self-Assessment Questionnaire (SAQ) guidance.

---

## Table of Contents

1. [Overview](#overview)
2. [Scope Definition and Reduction](#scope-definition-and-reduction)
3. [Requirement 1: Network Security Controls](#requirement-1-network-security-controls)
4. [Requirement 2: Secure Configurations](#requirement-2-secure-configurations)
5. [Requirement 3: Protect Stored Account Data](#requirement-3-protect-stored-account-data)
6. [Requirement 4: Protect Data in Transit](#requirement-4-protect-data-in-transit)
7. [Requirement 5: Protect Against Malicious Software](#requirement-5-protect-against-malicious-software)
8. [Requirement 6: Develop Secure Systems](#requirement-6-develop-secure-systems)
9. [Requirement 7: Restrict Access by Business Need](#requirement-7-restrict-access-by-business-need)
10. [Requirement 8: Identify Users and Authenticate Access](#requirement-8-identify-users-and-authenticate-access)
11. [Requirement 9: Restrict Physical Access](#requirement-9-restrict-physical-access)
12. [Requirement 10: Log and Monitor Access](#requirement-10-log-and-monitor-access)
13. [Requirement 11: Test Security Regularly](#requirement-11-test-security-regularly)
14. [Requirement 12: Support Security with Policies](#requirement-12-support-security-with-policies)
15. [SAQ Guidance](#saq-guidance)

---

## Overview

PCI DSS v4.0 was released in March 2022 and becomes mandatory on **March 31, 2025**, replacing v3.2.1. The standard contains **12 requirements** organized under **6 goals**. PCI DSS v4.0 introduces a **customized approach** as an alternative to the defined approach, allowing organizations to meet security objectives through alternative controls.

### Applicability

The SuperAdmin platform processes, stores, or transmits cardholder data through its payment integration module (Stripe). The platform's PCI DSS scope is minimized by delegating payment processing to a PCI-certified third-party processor (Stripe), which handles the Cardholder Data Environment (CDE) directly.

### Goals and Requirements Summary

| Goal | Requirements | Summary |
|------|-------------|---------|
| Build and Maintain a Secure Network and Systems | 1, 2 | Install and maintain network security controls; apply secure configurations |
| Protect Account Data | 3, 4 | Protect stored account data; protect data with strong cryptography during transmission |
| Maintain a Vulnerability Management Program | 5, 6 | Protect systems against malicious software; develop and maintain secure systems |
| Implement Strong Access Control Measures | 7, 8, 9 | Restrict access by business need to know; identify users and authenticate access; restrict physical access |
| Regularly Monitor and Test Networks | 10, 11 | Log and monitor all access; test security of systems regularly |
| Maintain an Information Security Policy | 12 | Support information security with organizational policies and programs |

---

## Scope Definition and Reduction

### Cardholder Data Environment (CDE)

The CDE includes all systems that store, process, or transmit cardholder data, plus any systems connected to those systems.

### Scope Reduction Strategies

The SuperAdmin platform employs multiple strategies to minimize PCI DSS scope.

| Strategy | Implementation | Scope Impact |
|----------|---------------|-------------|
| **Third-party payment processing** | Stripe handles all cardholder data; the platform never receives or stores PANs | Eliminates CDE from the application |
| **Stripe Elements / Payment Links** | Client-side Stripe.js collects card data directly; card numbers never touch our servers | Reduces SAQ type from D to A |
| **Tokenization** | Stripe provides payment method tokens (pm_xxx); only tokens are stored in our database | No cardholder data in our database |
| **Network segmentation** | Payment-related API endpoints are isolated in a dedicated microservice with restricted network access | Limits connected systems |
| **iFrame isolation** | Stripe Elements renders in an isolated iFrame; the parent page cannot access card data | Prevents accidental card data capture |

### Data Flow

```
Customer Browser                SuperAdmin API              Stripe API
┌─────────────────┐           ┌──────────────────┐        ┌──────────────┐
│                  │           │                  │        │              │
│ Stripe Elements  │──Card────►│ (card data never │        │              │
│ (iFrame)         │  data     │  reaches our     │        │              │
│                  │  direct   │  server)          │        │              │
│                  │  to       │                  │        │              │
│                  │  Stripe──►│──────────────────►│──────►│ Process      │
│                  │           │  Token (pm_xxx)   │        │ Payment      │
│                  │           │                  │◄──────│ Result       │
│                  │◄──────────┤  Confirmation    │        │              │
└─────────────────┘           └──────────────────┘        └──────────────┘
```

Because cardholder data never reaches the SuperAdmin servers, the platform operates under **SAQ A** eligibility criteria.

---

## Requirement 1: Network Security Controls

*Install and maintain network security controls.*

### 1.1 — Processes and mechanisms for managing network security controls

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 1.1.1 | Security policies and operational procedures documented and maintained | SECURITY.md; network security procedures in runbooks |
| 1.1.2 | Roles and responsibilities assigned | DevOps team manages network configuration; changes require Security Team approval |

### 1.2 — Network security controls configured and maintained

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 1.2.1 | Inbound and outbound traffic restricted | AWS Security Groups: ingress limited to HTTPS (443), SSH (22 from bastion only); egress allow-listed |
| 1.2.2 | Configuration changes reviewed semi-annually | Terraform plan review in PR; quarterly security group audit |
| 1.2.3 | Accurate network diagram maintained | Architecture diagram in INFRASTRUCTURE_STACK.md; updated with every infrastructure change |
| 1.2.4 | Accurate data flow diagram maintained | Payment data flow documented above; updated with every integration change |
| 1.2.5 | All permitted services, protocols, and ports documented | Security group rules defined in Terraform; documented in infrastructure runbook |
| 1.2.6 | Security features for insecure services/protocols | No insecure protocols permitted; FTP, Telnet, HTTP disabled |
| 1.2.7 | Review of security rules at least every six months | Scheduled quarterly review; findings tracked in compliance dashboard |

### 1.3 — Network access to and from the CDE is restricted

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 1.3.1 | Inbound traffic to CDE restricted to necessary | Payment microservice accepts traffic only from API gateway; no direct internet access |
| 1.3.2 | Outbound traffic from CDE restricted to necessary | Egress limited to Stripe API endpoints (api.stripe.com) and internal services |
| 1.3.3 | NSCs between wireless and CDE | N/A — cloud-native; no wireless networks in scope |

### 1.4 — Network connections between trusted and untrusted networks

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 1.4.1 | NSCs between trusted/untrusted networks | AWS VPC with public/private subnet separation; NAT gateway for outbound |
| 1.4.2 | Inbound traffic from untrusted networks restricted | ALB with WAF inspects all inbound traffic; only HTTPS permitted |
| 1.4.3 | Anti-spoofing measures | AWS VPC source/destination check enabled; no IP spoofing possible |
| 1.4.4 | Personal devices do not connect to CDE | Production access only via bastion host; no direct connections from personal devices |

### 1.5 — Risks to the CDE from computing devices

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 1.5.1 | Controls for computing devices connecting to CDE | Bastion host with MFA; JIT access; all sessions logged |

---

## Requirement 2: Secure Configurations

*Apply secure configurations to all system components.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 2.1.1 | Security policies documented | Server hardening guide; container security policy |
| 2.2.1 | Configuration standards developed | CIS benchmarks for Docker, K8s, PostgreSQL; documented in IaC |
| 2.2.2 | Vendor default accounts removed/disabled | Default PostgreSQL user disabled; K8s default service accounts restricted |
| 2.2.3 | Primary functions separated | Database, application, cache on separate containers/services |
| 2.2.4 | Only necessary services enabled | Distroless Docker images; no SSH in containers; minimal packages |
| 2.2.5 | Insecure services secured | All services use TLS; no plaintext protocols |
| 2.2.6 | System security parameters configured | Security headers enforced; cookie flags set; CSP active |
| 2.2.7 | Non-console admin access encrypted | All admin access via HTTPS; SSH via bastion only |
| 2.3.1 | Wireless environments not applicable | Cloud-native; no wireless in CDE scope |

---

## Requirement 3: Protect Stored Account Data

*Protect stored account data.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 3.1.1 | Security policies documented | Data classification policy; encryption standards (10_encryption_standards.csv) |
| 3.2.1 | Data storage minimized | No cardholder data stored; Stripe tokens only; retention policies enforced |
| 3.2.2 | Sensitive authentication data not stored after authorization | Platform never receives sensitive auth data (Stripe handles directly) |
| 3.3.1 | PAN not stored unless business-justified | PAN never stored; only Stripe token references |
| 3.3.2 | PAN rendered unreadable (if stored) | N/A — PAN never stored |
| 3.3.3 | PAN masked when displayed | N/A — only last 4 digits received from Stripe for display |
| 3.4.1 | PAN secured with strong cryptography (if stored) | N/A — PAN never stored on our systems |
| 3.5.1 | Account data on removable media protected | No removable media in use; cloud-native architecture |
| 3.6.1 | Cryptographic key management processes | AWS KMS for key management; automatic rotation; key access logged |
| 3.7.1 | Cryptographic key management procedures | Key rotation every 365 days; old keys retained for decryption only; separated key custodians |

---

## Requirement 4: Protect Data in Transit

*Protect cardholder data with strong cryptography during transmission over open, public networks.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 4.1.1 | Security policies documented | Encryption in transit policy; TLS configuration |
| 4.2.1 | Strong cryptography for PAN transmission | TLS 1.3 (minimum 1.2); HSTS enabled with preload; only strong cipher suites |
| 4.2.1.1 | Trusted certificates used | Certificates from public CA (Let's Encrypt/AWS ACM); automated renewal |
| 4.2.1.2 | Certificate validity verified | Certificate chain validation; OCSP stapling enabled |
| 4.2.2 | PAN secured when sent via end-user messaging | N/A — PAN never sent via messaging; only token references |

### TLS Configuration

```
# Allowed cipher suites (TLS 1.3)
TLS_AES_256_GCM_SHA384
TLS_AES_128_GCM_SHA256
TLS_CHACHA20_POLY1305_SHA256

# Allowed cipher suites (TLS 1.2 — legacy clients)
ECDHE-ECDSA-AES256-GCM-SHA384
ECDHE-RSA-AES256-GCM-SHA384
ECDHE-ECDSA-AES128-GCM-SHA256
ECDHE-RSA-AES128-GCM-SHA256

# Disabled protocols
SSLv2, SSLv3, TLS 1.0, TLS 1.1
```

---

## Requirement 5: Protect Against Malicious Software

*Protect all systems and networks from malicious software.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 5.1.1 | Security policies documented | Anti-malware policy; container security standards |
| 5.2.1 | Anti-malware solution deployed | Trivy for container scanning; no traditional AV needed (containers are immutable) |
| 5.2.2 | Anti-malware performs periodic scans | Trivy scans on every build and nightly; filesystem scanning for secrets |
| 5.2.3 | Anti-malware for non-risk systems | All components scanned; risk assessment documented |
| 5.3.1 | Anti-malware mechanisms active and maintained | Trivy vulnerability database updated daily; alerts on new CVEs |
| 5.3.2 | Anti-malware logs retained | Trivy scan results stored in CI artifacts; retained for 12 months |
| 5.3.3 | Anti-malware cannot be disabled by users | Trivy scan is a required CI stage; cannot be skipped |
| 5.4.1 | Anti-phishing mechanisms | Email authentication (SPF, DKIM, DMARC); phishing simulation training |

---

## Requirement 6: Develop Secure Systems

*Develop and maintain secure systems and software.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 6.1.1 | Security policies documented | SDLC security policy; TESTING.md |
| 6.2.1 | Custom software developed securely | Secure coding standards; SonarQube quality gates; mandatory code review |
| 6.2.2 | Software development personnel trained | Annual secure coding training; OWASP training for new hires |
| 6.2.3 | Software reviewed prior to release | Code review (2 approvals); SAST scan; DAST scan on staging |
| 6.2.4 | Software engineering techniques prevent attacks | Parameterized queries; input validation (Zod); output encoding; CSRF tokens |
| 6.3.1 | Security vulnerabilities identified and managed | Trivy, SonarQube, npm audit; vulnerability SLA (7d critical, 14d high) |
| 6.3.2 | Software inventory maintained | SBOM generated per release; pnpm-lock.yaml tracked |
| 6.3.3 | Software updated for known vulnerabilities | Renovate automated updates; critical patches within 7 days |
| 6.4.1 | Public-facing web applications protected | WAF (Cloudflare/AWS WAF); rate limiting; bot detection |
| 6.4.2 | Public-facing web apps protected against attacks | OWASP Top 10 coverage; automated DAST scans; annual penetration test |
| 6.4.3 | Payment page scripts managed | Stripe Elements loaded from Stripe CDN; SRI hashes; CSP restricts script sources |
| 6.5.1 | Change management for production | Git-based workflow; PR reviews; Jenkins pipeline; rollback capability |
| 6.5.2 | Approval required for production changes | Minimum 2 approvals for PRs to main; deployment gated by CI pipeline |
| 6.5.3 | Pre-production testing | Full test suite (unit, integration, E2E, security) runs before production deploy |
| 6.5.4 | Rollback procedures | Blue-green deployments; K8s rolling updates with automatic rollback on health check failure |
| 6.5.5 | Test/development data separated from production | No production data in non-production environments; synthetic data generators |
| 6.5.6 | Test accounts/data removed before production | CI pipeline removes test data; environment-specific configurations |

---

## Requirement 7: Restrict Access by Business Need

*Restrict access to system components and cardholder data by business need to know.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 7.1.1 | Security policies documented | Access control policy; RBAC documentation |
| 7.2.1 | Access control model defined | RBAC with 7-level hierarchy; 70+ granular permissions |
| 7.2.2 | Access assigned based on job function and need to know | Role assignment tied to job title; principle of least privilege |
| 7.2.3 | Required privileges approved by authorized personnel | Access requests require manager + Security Team approval |
| 7.2.4 | Access reviews performed periodically | Quarterly access reviews; automated reports flagging unused permissions |
| 7.2.5 | Access for application and system accounts assigned based on least privileges | Service accounts have minimal permissions; no shared accounts |
| 7.2.5.1 | Access for application/system accounts reviewed periodically | Service account permissions reviewed quarterly |
| 7.2.6 | Access to database objects restricted | PostgreSQL RLS; application-level query scoping; no direct DB access from users |
| 7.3.1 | Access control system enforces restrictions | RBAC enforced at API gateway, application middleware, and database levels |
| 7.3.2 | Access control system set to "deny all" by default | Default deny; permissions explicitly granted per role |
| 7.3.3 | Access control system based on least privileges | Each role has minimum permissions needed; custom roles available |

---

## Requirement 8: Identify Users and Authenticate Access

*Identify users and authenticate access to system components.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 8.1.1 | Security policies documented | Authentication policy; password policy (12_password_policy.csv) |
| 8.2.1 | All users assigned unique IDs | UUID-based user IDs; no shared accounts permitted |
| 8.2.2 | Group/shared accounts not used (or controlled) | Shared accounts prohibited; service accounts have named owners |
| 8.2.3 | Additional requirements for service accounts | Service accounts have rotation schedules; scoped to minimum permissions |
| 8.2.4 | User lifecycle managed | Provisioning/deprovisioning workflows; account disabled within 4 hours of termination |
| 8.2.5 | Terminated user accounts deactivated | Automated deactivation via HR system integration; manual verification |
| 8.2.6 | Inactive accounts removed/disabled within 90 days | Automated 90-day inactivity detection; account suspended with admin notification |
| 8.2.7 | Accounts for third parties managed | Vendor accounts have expiration dates; reviewed quarterly |
| 8.2.8 | Account not locked out for more than 15 minutes unless identity confirmed | 15-minute lockout period; admin can unlock earlier with identity verification |
| 8.3.1 | All user access authenticated | All API endpoints require authentication; no anonymous access to data |
| 8.3.2 | Strong cryptography for authentication | bcrypt (cost 12) for passwords; TLS for credential transmission |
| 8.3.4 | Invalid authentication attempts limited | Account lockout after 10 failed attempts; progressive delay after 5 |
| 8.3.5 | Inactivity timeout for sessions | 30-minute idle timeout; 24-hour absolute timeout |
| 8.3.6 | Password/passphrase complexity requirements | Minimum 12 characters; uppercase, lowercase, number, special character; breach database check |
| 8.3.7 | New password different from previous four | Password history check (last 4 passwords) |
| 8.3.9 | Password/passphrase changed at least every 90 days (if only authentication factor) | 90-day rotation enforced; waived if MFA is active |
| 8.3.10 | If used as an authentication factor, passwords/passphrases meet minimum complexity | All password requirements enforced via validation middleware |
| 8.3.10.1 | Service account passwords/passphrases changed periodically | Service account credential rotation every 90 days via Vault |
| 8.4.1 | MFA for non-console admin access to CDE | MFA enforced for all admin roles (TOTP + WebAuthn) |
| 8.4.2 | MFA for all access to CDE | MFA enforced for any user accessing payment-related endpoints |
| 8.4.3 | MFA for all remote network access | VPN access requires MFA; bastion host requires MFA |
| 8.5.1 | MFA systems properly configured | MFA cannot be bypassed; replay attacks prevented (TOTP time window = 30s) |
| 8.6.1 | System/application accounts managed | API keys managed via Vault; rotation enforced; usage logged |
| 8.6.2 | Passwords/passphrases for system/application accounts not hard-coded | No credentials in source code; secrets injected at runtime via environment/Vault |
| 8.6.3 | Passwords/passphrases for system/application accounts protected | Stored in HashiCorp Vault; encrypted at rest; access-logged |

---

## Requirement 9: Restrict Physical Access

*Restrict physical access to cardholder data.*

As a cloud-native SaaS platform, physical security of the CDE is managed by **AWS**, which maintains PCI DSS Level 1 compliance for its data centers.

| Sub-Requirement | Implementation | Notes |
|----------------|---------------|-------|
| 9.1.1 | Security policies documented | Physical security delegated to AWS; documented in shared responsibility model |
| 9.2–9.4 | Physical access controls | Managed by AWS (SOC 2 Type II, PCI DSS Level 1 attested) |
| 9.5.1 | POI devices protected from tampering | N/A — no point-of-interaction devices; Stripe handles card capture |

---

## Requirement 10: Log and Monitor Access

*Log and monitor all access to system components and cardholder data.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 10.1.1 | Security policies documented | Logging policy; audit log specifications (06_audit_log_actions.csv) |
| 10.2.1 | Audit logs enabled and active | Comprehensive audit logging for all state-changing operations |
| 10.2.1.1 | All individual user access to cardholder data logged | All access to payment endpoints logged with user, timestamp, IP |
| 10.2.1.2 | All actions taken by any individual with admin access logged | Super Admin and Admin actions fully logged in audit trail |
| 10.2.1.3 | All access to audit logs logged | Meta-logging: access to audit log viewer is itself logged |
| 10.2.1.4 | All invalid logical access attempts logged | Failed authentication logged; failed authorization logged with attempted resource |
| 10.2.1.5 | All changes to identification and authentication mechanisms logged | Password changes, MFA enrollment/removal, role changes logged |
| 10.2.1.6 | All initialization of new audit logs logged | Log rotation events logged; new log stream creation logged |
| 10.2.1.7 | All creation and deletion of system-level objects logged | Database schema changes, K8s resource changes, container lifecycle logged |
| 10.2.2 | Audit logs capture required details | Each log entry: user ID, event type, date/time, success/failure, origination, identity/name of affected data/resource |
| 10.3.1 | Audit log access read-only | Append-only storage; no delete/modify permissions on audit logs |
| 10.3.2 | Audit logs protected from modification | Cryptographic chaining; integrity verification; immutable storage (S3 Object Lock) |
| 10.3.3 | Audit log files backed up promptly | Real-time streaming to centralized log platform (ELK/Datadog) |
| 10.3.4 | File integrity monitoring for audit logs | SHA-256 checksums; tamper detection alerts |
| 10.4.1 | Audit logs reviewed daily | Automated anomaly detection; daily security operations review |
| 10.4.1.1 | Automated mechanisms for audit log review | SIEM correlation rules; alerting on suspicious patterns |
| 10.4.2 | All other audit logs reviewed periodically | Weekly review of non-critical logs |
| 10.4.2.1 | Frequency of review defined in security policy | Documented in logging policy: critical=real-time, high=daily, medium=weekly |
| 10.4.3 | Exceptions and anomalies identified during review are addressed | Findings tracked in incident management system; resolution SLA enforced |
| 10.5.1 | Audit log history retained for 12 months | Minimum 12 months online; 3 months immediately available; configurable per tenant |
| 10.6.1 | Time synchronization with authoritative time sources | NTP synchronization with AWS time service; UTC for all timestamps |
| 10.6.2 | Time data protected from unauthorized access | NTP configuration managed via IaC; changes require approval |
| 10.6.3 | Time settings received from industry-accepted sources | AWS Time Sync Service (based on GPS and atomic clocks) |
| 10.7.1 | Critical security control failures detected and reported | Monitoring alerts for: log pipeline failures, scan failures, WAF failures |
| 10.7.2 | Critical security control failures responded to promptly | PagerDuty alerting for critical failures; 15-minute response SLA |
| 10.7.3 | Security control failures documented and addressed | Incident tickets created; root cause analysis; preventive measures |

---

## Requirement 11: Test Security Regularly

*Test security of systems and networks regularly.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 11.1.1 | Security policies documented | TESTING.md; vulnerability management policy |
| 11.2.1 | Wireless access points managed | N/A — cloud-native; no wireless APs in CDE |
| 11.3.1 | Internal vulnerability scans performed quarterly | Trivy scans on every build + quarterly comprehensive scan |
| 11.3.1.1 | All high-risk and critical vulnerabilities resolved | Tracked in vulnerability register; SLA enforcement |
| 11.3.1.2 | Internal scans repeated after resolution | Re-scan required after remediation; CI pipeline enforces clean scan |
| 11.3.1.3 | Internal scans performed by qualified personnel | DevSecOps team trained in vulnerability assessment |
| 11.3.2 | External vulnerability scans performed quarterly by ASV | Quarterly ASV scan by approved scanning vendor |
| 11.3.2.1 | External scans repeated after resolution | Re-scan after remediation; passing report obtained |
| 11.4.1 | External penetration testing annually | Annual penetration test by qualified third-party assessor |
| 11.4.2 | Internal penetration testing annually | Annual internal penetration test; covers application and infrastructure |
| 11.4.3 | Exploitable vulnerabilities corrected and re-tested | Findings from penetration tests tracked; re-test after fix |
| 11.4.4 | Segmentation controls tested annually | Network segmentation verified during penetration test |
| 11.5.1 | Intrusion-detection/prevention techniques detect intrusion | WAF with intrusion detection; network-level IDS via AWS GuardDuty |
| 11.5.1.1 | Intrusion-detection alerts monitored 24/7 | PagerDuty integration; on-call rotation; 15-minute acknowledgement SLA |
| 11.5.2 | Change-detection mechanism deployed | File integrity monitoring; Terraform drift detection; container image verification |
| 11.6.1 | Payment page tampering detection | CSP prevents unauthorized scripts; SRI for Stripe Elements; manual checks quarterly |

---

## Requirement 12: Support Security with Policies

*Support information security with organizational policies and programs.*

| Sub-Requirement | Implementation | Evidence |
|----------------|---------------|----------|
| 12.1.1 | Overall information security policy established | SECURITY.md; SUPER_ADMIN_DOCUMENTATION.md |
| 12.1.2 | Security policy reviewed annually | Annual review; versioned document with review dates |
| 12.1.3 | Security policy defines roles and responsibilities | RBAC documentation; role descriptions; RACI matrix |
| 12.1.4 | CISO responsibility assigned | CISO role assigned; reports to CTO/CEO |
| 12.2.1 | Acceptable use policies documented | Acceptable use policy for employees and contractors |
| 12.3.1 | Risk assessment performed annually | Annual risk assessment using ISO 27005 methodology |
| 12.3.2 | Targeted risk analysis for customized approach | Risk analysis documented for each customized approach control |
| 12.3.3 | Cryptographic cipher suites and protocols documented | TLS configuration documented; cipher suite list maintained |
| 12.3.4 | Hardware and software technologies reviewed annually | Technology stack reviewed annually; EOL components identified |
| 12.4.1 | PCI DSS compliance managed | Quarterly compliance review; compliance dashboard |
| 12.5.1 | Scope documented and validated | Scope document reviewed annually; validated during assessment |
| 12.5.2 | Scope documented annually | Updated annually and upon significant change |
| 12.5.3 | Scope validated upon significant change | Infrastructure changes trigger scope review |
| 12.6.1 | Security awareness program | Annual security awareness training for all personnel |
| 12.6.2 | Security awareness reviewed annually | Training content updated annually |
| 12.6.3 | Training upon hire and annually | New hire training within 30 days; annual refresher |
| 12.6.3.1 | Training includes awareness of threats | Training covers phishing, social engineering, data handling |
| 12.6.3.2 | Training includes acceptable use | Acceptable use covered in annual training |
| 12.7.1 | Personnel screened prior to hire | Background checks for employees with CDE access |
| 12.8.1 | List of service providers maintained | Sub-processor register with PCI compliance status |
| 12.8.2 | Written agreements with service providers | DPAs and security addenda for all processors |
| 12.8.3 | Established process for engaging service providers | Vendor security assessment before onboarding |
| 12.8.4 | Service providers' PCI DSS compliance monitored | Annual review of provider compliance certificates |
| 12.8.5 | Service provider responsibility documented | Shared responsibility matrix for each provider |
| 12.9.1 | Service providers acknowledge PCI DSS responsibilities | Written acknowledgement in contracts |
| 12.9.2 | Service providers support customer compliance requests | Compliance documentation provided upon request |
| 12.10.1 | Incident response plan documented | Incident response plan with PCI-specific procedures |
| 12.10.2 | Incident response plan reviewed annually | Annual review and tabletop exercises |
| 12.10.3 | Designated personnel available 24/7 | On-call rotation; PagerDuty alerting |
| 12.10.4 | Personnel trained on incident response | Incident response training; tabletop exercises |
| 12.10.4.1 | Training frequency defined | Semi-annual incident response training |
| 12.10.5 | Incident response plan includes monitoring alerts | SIEM alerts trigger incident response procedures |
| 12.10.6 | Incident response plan modified based on lessons learned | Post-incident reviews update procedures |
| 12.10.7 | Incident response procedures for unexpected PAN detection | Procedure to securely delete unexpected cardholder data; forensic imaging |

---

## SAQ Guidance

### Determining the Correct SAQ

Based on the SuperAdmin platform's payment integration architecture (Stripe Elements with no server-side cardholder data), the applicable SAQ is:

| SAQ Type | Description | Applicable? |
|----------|-------------|------------|
| **SAQ A** | Card-not-present merchants, all cardholder data functions fully outsourced | **Yes — Primary SAQ** |
| SAQ A-EP | E-commerce merchants with partial outsourcing (website impacts card data security) | Potential, if redirecting to Stripe Checkout |
| SAQ B | Merchants with only imprint machines or standalone dial-out terminals | No |
| SAQ C | Merchants with payment application systems connected to the Internet | No |
| SAQ D | All other merchants / service providers | No (scope reduced) |

### SAQ A Eligibility Criteria

To maintain SAQ A eligibility, the SuperAdmin platform ensures:

1. All payment processing is fully outsourced to Stripe (PCI Level 1 certified).
2. No electronic storage, processing, or transmission of cardholder data on our systems.
3. Stripe Elements (iFrame-based) is used for card data collection.
4. The platform confirms that the third-party payment processor is PCI DSS compliant.
5. All access to the payment page uses HTTPS.

### SAQ A Completion Checklist

- [ ] Confirm all cardholder data functions outsourced to PCI-certified processor (Stripe).
- [ ] Verify no electronic storage of cardholder data anywhere on our systems.
- [ ] Confirm payment page delivered over HTTPS.
- [ ] Verify Stripe's PCI DSS compliance certificate is current.
- [ ] Complete SAQ A questionnaire.
- [ ] Conduct quarterly ASV scan (if applicable per acquirer requirement).
- [ ] Submit Attestation of Compliance (AOC) to acquirer.

---

*This document is reviewed annually and whenever the payment integration architecture changes. Last review: Q1 2026.*
