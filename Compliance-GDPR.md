# GDPR Compliance Guide

This document describes how the SuperAdmin SaaS platform achieves and maintains compliance with the **General Data Protection Regulation (EU) 2016/679 (GDPR)**. It covers the seven GDPR principles, data subject rights, consent management, cross-border transfers, breach notification, and the specific platform features that support compliance.

---

## Table of Contents

1. [GDPR Principles](#gdpr-principles)
2. [Data Subject Rights](#data-subject-rights)
3. [Consent Management](#consent-management)
4. [Data Processing Agreements](#data-processing-agreements)
5. [Cross-Border Data Transfer](#cross-border-data-transfer)
6. [Data Breach Notification](#data-breach-notification)
7. [Data Protection Impact Assessment](#data-protection-impact-assessment)
8. [Records of Processing Activities](#records-of-processing-activities)
9. [SuperAdmin Features Supporting GDPR](#superadmin-features-supporting-gdpr)
10. [Technical Measures](#technical-measures)

---

## GDPR Principles

The GDPR is built on seven foundational principles defined in **Article 5**. The SuperAdmin platform implements controls to uphold each one.

### 1. Lawfulness, Fairness, and Transparency (Art. 5(1)(a))

| Requirement | Implementation |
|-------------|----------------|
| Processing must have a legal basis | The platform records a legal basis for every processing activity (consent, contract, legitimate interest, legal obligation, vital interest, or public task). |
| Fair processing | Data is only used for purposes that the data subject would reasonably expect. No dark patterns in the UI. |
| Transparency | Privacy notices are presented in clear, plain language at every data collection point. The platform provides a tenant-configurable privacy center. |

**Legal Bases Tracked by the Platform:**

| Legal Basis | GDPR Article | Example Use Case |
|-------------|-------------|------------------|
| Consent | Art. 6(1)(a) | Marketing emails, analytics cookies |
| Contract performance | Art. 6(1)(b) | Providing the SaaS service, user account management |
| Legal obligation | Art. 6(1)(c) | Tax records, audit logs required by regulation |
| Vital interests | Art. 6(1)(d) | Emergency contact processing (rare in SaaS) |
| Public interest | Art. 6(1)(e) | Government SaaS deployments |
| Legitimate interest | Art. 6(1)(f) | Security monitoring, fraud detection, product improvement |

### 2. Purpose Limitation (Art. 5(1)(b))

- Data collected for a specific purpose is not processed for incompatible purposes.
- The platform tags every data field with its processing purpose(s) in the data catalog.
- Purpose changes require a new legal basis assessment, which is logged in the audit trail.
- The `processing_purposes` table links each data category to its declared purpose.

### 3. Data Minimization (Art. 5(1)(c))

- Only data that is necessary for the declared purpose is collected.
- API endpoints validate input via Zod schemas that reject unexpected fields.
- The platform provides field-level configuration so tenants can disable optional data fields.
- Periodic data minimization reviews are scheduled quarterly via the compliance dashboard.

### 4. Accuracy (Art. 5(1)(d))

- Users can update their personal data at any time through the self-service profile page.
- The platform supports data subject rectification requests via the Data Subject Request (DSR) workflow.
- Email verification is required for all accounts.
- Address and phone number validation is performed at the point of entry.

### 5. Storage Limitation (Art. 5(1)(e))

- Retention policies are defined per data category and enforced automatically.
- The platform includes a configurable data retention engine that anonymizes or deletes data when the retention period expires.

| Data Category | Default Retention | Configurable | Anonymize or Delete |
|---------------|-------------------|-------------|---------------------|
| User account data | Duration of contract + 30 days | Yes | Delete |
| Audit logs | 7 years | Yes (minimum 1 year) | Anonymize |
| Session data | 24 hours after expiry | No | Delete |
| Support tickets | 3 years after closure | Yes | Anonymize |
| Marketing consent records | Duration of consent + 3 years | Yes | Delete |
| Backup data | 90 days | Yes | Delete |

### 6. Integrity and Confidentiality (Art. 5(1)(f))

- AES-256-GCM encryption at rest for all personal data.
- TLS 1.3 encryption in transit for all communications.
- Row-Level Security (RLS) prevents cross-tenant data access.
- Bcrypt (cost 12) password hashing.
- Comprehensive access controls (RBAC with 7-level hierarchy).
- Audit logging of all data access and modifications.

### 7. Accountability (Art. 5(2))

- The platform maintains Records of Processing Activities (ROPA) as required by Article 30.
- Data Protection Impact Assessments (DPIAs) are documented and stored in the compliance module.
- All policy changes are version-controlled and audit-logged.
- Annual compliance reviews are tracked with evidence collection.
- The Data Protection Officer (DPO) has a dedicated dashboard with compliance metrics.

---

## Data Subject Rights

The GDPR grants individuals eight rights regarding their personal data. The SuperAdmin platform provides automated and semi-automated workflows to fulfill each one within the **30-day response deadline** (Article 12(3)).

### Right of Access (Art. 15)

**What it requires**: Data subjects can request a copy of all personal data being processed about them, along with the purposes, categories, recipients, retention periods, and the source of the data.

**Implementation**:

- The platform provides a **self-service data export** feature accessible from the user profile.
- The export is generated as a machine-readable JSON file and a human-readable PDF.
- Super Admins can process access requests via the DSR workflow: `Admin Panel > Privacy > Data Subject Requests > New Request > Access`.
- Automated data collection spans all tables tagged as containing personal data in the data catalog.
- Identity verification is required before releasing data (email verification + security question or ID upload).

**Workflow**:

```
Data Subject        SuperAdmin Platform         Admin Review
    │                      │                        │
    ├──Request Access──────►│                        │
    │                      ├──Verify Identity────────►│
    │                      │                        ├──Approve/Deny
    │                      ├──Collect Data───────────│
    │                      ├──Generate Export─────────│
    │◄─────Deliver Export──┤                        │
    │                      ├──Log Fulfillment────────│
```

### Right to Rectification (Art. 16)

**What it requires**: Data subjects can request correction of inaccurate personal data.

**Implementation**:

- Users can directly edit most profile fields (name, email, phone, address) via self-service.
- For fields that require admin approval (e.g., legal name, organization), a rectification request is submitted through the DSR workflow.
- All changes are audit-logged with before/after values.
- Rectification propagates to all systems via an event (`data.rectified`) published to the message queue.

### Right to Erasure (Art. 17) — "Right to Be Forgotten"

**What it requires**: Data subjects can request deletion of their personal data when certain conditions are met.

**Implementation**:

- The **deletion workflow** identifies all personal data across the platform linked to the data subject.
- Data is categorized as:
  - **Deletable**: Removed permanently.
  - **Anonymizable**: Personal identifiers are removed, but the record is retained for statistical or legal purposes.
  - **Retained (legal hold)**: Data subject to legal retention requirements is flagged but not deleted; the data subject is informed of the legal basis for retention.
- Deletion cascades through: user profiles, activity logs (anonymized), support tickets, uploaded files, and any third-party systems via webhook notifications.
- A **deletion certificate** is generated confirming the scope of erasure and the date completed.

**Deletion Cascade Matrix**:

| Data Store | Action | Timeline |
|-----------|--------|----------|
| PostgreSQL (primary) | Hard delete or anonymize | Immediate |
| Redis (cache) | Invalidate all keys for user | Immediate |
| Elasticsearch (search index) | Delete documents | Within 1 hour |
| File storage (S3) | Delete uploaded files | Within 24 hours |
| Backups | Excluded from future restores; old backups expire per retention policy | Up to 90 days |
| Third-party integrations | Webhook notification sent to delete data | Within 72 hours |
| Audit logs | Anonymize (replace PII with hash) | Immediate |

### Right to Data Portability (Art. 20)

**What it requires**: Data subjects can receive their personal data in a structured, commonly used, machine-readable format and transmit it to another controller.

**Implementation**:

- Export formats: JSON (primary), CSV, XML.
- Exported data includes: profile information, activity history, preferences, uploaded content, and consent records.
- The export endpoint (`POST /api/v1/privacy/export`) generates the package asynchronously and delivers it via secure download link (signed URL, valid for 24 hours).
- The data schema follows the Data Transfer Project (DTP) format where applicable.

### Right to Restriction of Processing (Art. 18)

**What it requires**: Data subjects can request that processing of their data be restricted (data is stored but not actively processed).

**Implementation**:

- When restriction is applied, the user record is flagged with `processing_restricted: true`.
- All processing engines (analytics, marketing, automated decisions) check this flag and skip restricted records.
- The data remains stored but is only accessible for storage, legal claims, or consent-based processing.
- The restriction is logged in the audit trail and the data subject is notified when it is lifted.

### Right to Object (Art. 21)

**What it requires**: Data subjects can object to processing based on legitimate interest or public interest, including profiling.

**Implementation**:

- The platform provides an objection workflow: `Admin Panel > Privacy > Data Subject Requests > New Request > Objection`.
- For direct marketing, objection is absolute — processing stops immediately.
- For legitimate interest processing, the admin performs a balancing test and documents the decision.
- Objection records are maintained in the `data_subject_objections` table.

### Right Not to Be Subject to Automated Decision-Making (Art. 22)

**What it requires**: Data subjects have the right not to be subject to decisions based solely on automated processing that produce legal or similarly significant effects.

**Implementation**:

- The platform's threat detection and automated blocking features include a human review override.
- Any automated account suspension or restriction triggers a notification to the admin for human review within 24 hours.
- Data subjects can contest automated decisions through the support workflow.
- The platform logs the logic and data inputs for every automated decision, enabling meaningful explanation.

### Right to Be Informed (Art. 13, 14)

**What it requires**: Data subjects must be informed about how their data is collected and processed.

**Implementation**:

- Privacy notices are displayed at all data collection points (registration, profile update, cookie consent).
- The platform provides a tenant-configurable privacy policy page.
- The privacy notice includes: controller identity, DPO contact, purposes, legal basis, recipients, transfer information, retention periods, and data subject rights.

---

## Consent Management

### Consent Collection

The platform implements a **granular consent management system** that tracks consent per purpose.

| Field | Description |
|-------|-------------|
| `consent_id` | Unique identifier for the consent record |
| `data_subject_id` | User or contact who gave consent |
| `tenant_id` | Tenant that collected the consent |
| `purpose` | Specific processing purpose (e.g., `marketing_email`, `analytics`, `profiling`) |
| `legal_basis` | Always `consent` for this table |
| `granted` | Boolean — current consent state |
| `granted_at` | Timestamp when consent was given |
| `withdrawn_at` | Timestamp when consent was withdrawn (null if active) |
| `collection_method` | How consent was collected (`web_form`, `api`, `checkbox`, `double_opt_in`) |
| `ip_address` | IP address at time of consent (for evidential purposes) |
| `user_agent` | Browser/device information at time of consent |
| `consent_text_version` | Version of the consent text presented to the user |

### Consent Requirements

- **Freely given**: Consent is not bundled with terms of service. Each purpose has a separate opt-in.
- **Specific**: Each consent record is tied to a specific, declared purpose.
- **Informed**: The consent text clearly explains what data will be processed and why.
- **Unambiguous**: Consent requires an affirmative action (opt-in checkbox, button click). Pre-ticked boxes are not used.
- **Withdrawable**: Users can withdraw consent at any time via the privacy center. Withdrawal is as easy as granting consent.
- **Documented**: Every consent event (grant and withdrawal) is logged with timestamp, method, and text version.

### Double Opt-In

For marketing communications, the platform implements **double opt-in**:

1. User checks the consent box and submits the form.
2. A confirmation email is sent with a unique verification link.
3. Consent is only recorded as active after the user clicks the verification link.
4. If the link is not clicked within 48 hours, the pending consent is discarded.

### Cookie Consent

The platform integrates with a **cookie consent management platform (CMP)** that:

- Categorizes cookies as Strictly Necessary, Functional, Analytics, and Marketing.
- Blocks non-essential cookies until consent is granted.
- Stores consent preferences per user and per browser.
- Provides a persistent consent banner that reappears when the cookie policy changes.
- Integrates with the server-side consent database for unified consent tracking.

---

## Data Processing Agreements

### When a DPA Is Required

Under Article 28, a **Data Processing Agreement** must be in place between the data controller (the tenant/customer) and the data processor (the SuperAdmin platform provider) before any personal data processing begins.

### DPA Template Contents

The platform provides a **self-service DPA** that covers:

| Section | Content |
|---------|---------|
| Subject matter and duration | Description of the processing and its duration |
| Nature and purpose | The purposes for which data is processed |
| Type of personal data | Categories of data processed (identifiers, contact data, usage data, etc.) |
| Categories of data subjects | Users, employees, customers of the tenant |
| Processor obligations | Security measures, confidentiality, assistance with DSR, breach notification |
| Sub-processors | List of sub-processors with notification of changes |
| International transfers | Transfer mechanisms (SCCs, adequacy) |
| Audit rights | Controller's right to audit the processor |
| Data return and deletion | Process for data return or deletion upon contract termination |
| Technical and organizational measures | Annex detailing all security controls (encryption, access control, monitoring) |

### Sub-Processor Management

- A current list of sub-processors is maintained at `https://{DOMAIN}/legal/sub-processors`.
- Tenants are notified 30 days before any new sub-processor is added.
- Tenants can object to a new sub-processor within the notification period.
- Each sub-processor has its own DPA with the platform provider.

| Sub-Processor | Purpose | Location | DPA Status |
|---------------|---------|----------|------------|
| AWS | Cloud infrastructure hosting | EU (Frankfurt, eu-central-1) | Active |
| Stripe | Payment processing | US (SCCs in place) | Active |
| SendGrid | Transactional email delivery | US (SCCs in place) | Active |
| Datadog | Infrastructure monitoring | EU (Dublin) | Active |
| Cloudflare | CDN and DDoS protection | Global (EU processing) | Active |

---

## Cross-Border Data Transfer

### Transfer Mechanisms

Post-Schrems II, the platform supports the following transfer mechanisms for personal data leaving the EEA.

| Mechanism | GDPR Article | When Used |
|-----------|-------------|-----------|
| **Adequacy decision** | Art. 45 | Transfers to countries with an adequacy decision (e.g., Japan, UK, South Korea, Canada for commercial organizations, EU-US Data Privacy Framework) |
| **Standard Contractual Clauses (SCCs)** | Art. 46(2)(c) | Transfers to all other third countries (e.g., US sub-processors not covered by DPF) |
| **Binding Corporate Rules (BCRs)** | Art. 47 | Intra-group transfers within multinational organizations |
| **Derogations** | Art. 49 | Explicit consent, contract performance — used only as a last resort |

### Transfer Impact Assessment (TIA)

For each cross-border transfer, a Transfer Impact Assessment is conducted that evaluates:

1. **Legal framework** of the recipient country (surveillance laws, data protection authority, judicial redress).
2. **Supplementary measures** required (encryption in transit and at rest, pseudonymization, access controls that prevent foreign government access to plaintext data).
3. **Sub-processor risk**: Each sub-processor's jurisdiction is assessed independently.

### Data Residency

- **Default**: All tenant data is stored in the **EU (Frankfurt, eu-central-1)** region.
- **Configurable**: Enterprise tenants can select their data residency region (EU, US, APAC) during onboarding.
- **Enforcement**: The platform's routing layer ensures that API requests for a tenant are served from the tenant's designated region.
- **Verification**: Data residency is verified through automated compliance checks that scan database and storage locations.

---

## Data Breach Notification

### 72-Hour Notification Requirement (Art. 33, 34)

Under GDPR, the platform must notify the relevant supervisory authority within **72 hours** of becoming aware of a personal data breach, unless the breach is unlikely to result in a risk to individuals' rights and freedoms.

### Breach Response Workflow

```
Detection (T+0)
    │
    ├── Automated alert from monitoring system
    │   OR manual report from employee/customer
    │
    ▼
Assessment (T+0 to T+4h)
    │
    ├── Severity classification (Low / Medium / High / Critical)
    ├── Scope determination (tenants affected, data types, volume)
    ├── Root cause preliminary analysis
    │
    ▼
Containment (T+4h to T+12h)
    │
    ├── Isolate affected systems
    ├── Revoke compromised credentials
    ├── Patch vulnerability if identified
    │
    ▼
Authority Notification (T+0 to T+72h)
    │
    ├── Notify supervisory authority (Art. 33)
    │   - Nature of breach
    │   - Categories and approximate number of data subjects
    │   - Categories and approximate number of records
    │   - Likely consequences
    │   - Measures taken or proposed
    │   - DPO contact details
    │
    ▼
Data Subject Notification (if high risk) (Art. 34)
    │
    ├── Direct communication to affected individuals
    │   - Plain language description of the breach
    │   - Likely consequences
    │   - Measures taken
    │   - Recommendations for individuals
    │
    ▼
Post-Incident Review (T+7d to T+30d)
    │
    ├── Root cause analysis
    ├── Remediation verification
    ├── Process improvement
    ├── Updated DPIA if needed
    └── Breach register updated
```

### Breach Register

The platform maintains an internal **breach register** as required by Article 33(5).

| Field | Description |
|-------|-------------|
| `breach_id` | Unique identifier |
| `detected_at` | Timestamp of detection |
| `nature` | Description of the breach |
| `data_categories` | Categories of data affected |
| `data_subjects_count` | Approximate number of individuals affected |
| `records_count` | Approximate number of records affected |
| `likely_consequences` | Assessment of potential impact |
| `measures_taken` | Containment and remediation actions |
| `authority_notified` | Whether the supervisory authority was notified |
| `authority_notified_at` | Timestamp of authority notification |
| `data_subjects_notified` | Whether data subjects were notified |
| `root_cause` | Root cause analysis findings |
| `status` | Open / Contained / Resolved / Closed |

---

## Data Protection Impact Assessment

### When a DPIA Is Required (Art. 35)

A DPIA is required before processing that is likely to result in a **high risk** to the rights and freedoms of individuals, including:

- Systematic and extensive profiling with significant effects.
- Large-scale processing of special categories of data.
- Systematic monitoring of publicly accessible areas.
- Use of new technologies that may present high risk.

### DPIA Process

| Step | Description | Responsible |
|------|-------------|-------------|
| 1. Identify need | Determine whether the processing requires a DPIA using the screening checklist | DPO / Project Lead |
| 2. Describe processing | Document the nature, scope, context, and purposes of the processing | Project Lead |
| 3. Assess necessity | Evaluate whether the processing is necessary and proportionate to the purpose | DPO |
| 4. Identify risks | Assess risks to individuals' rights and freedoms | DPO + Security Team |
| 5. Identify measures | Determine measures to mitigate identified risks | Security Team |
| 6. Document and sign off | Record the DPIA, obtain approval, and implement measures | DPO + Management |
| 7. Review | Monitor and review the DPIA regularly or when processing changes | DPO |

### DPIA Template (Stored in Platform)

The SuperAdmin compliance module includes a DPIA template with the following sections:

1. **Processing description**: What data, from whom, how, and why.
2. **Necessity and proportionality**: Legal basis, purpose limitation, data minimization assessment.
3. **Risk identification**: Threat scenarios, likelihood, severity, and risk rating.
4. **Risk mitigation**: Controls in place and additional measures needed.
5. **DPO consultation**: DPO opinion and recommendations.
6. **Approval**: Management sign-off with date.
7. **Review schedule**: Next review date and triggers for earlier review.

---

## Records of Processing Activities

### Controller ROPA (Art. 30(1))

As a data controller for its own operations and as a processor for tenant data, the platform maintains ROPA.

| Field | Content |
|-------|---------|
| Processing activity name | e.g., "User account management" |
| Purpose of processing | e.g., "Provide SaaS service to authenticated users" |
| Legal basis | e.g., "Contract performance (Art. 6(1)(b))" |
| Categories of data subjects | e.g., "Platform users, tenant administrators" |
| Categories of personal data | e.g., "Name, email, phone, role, login history" |
| Recipients | e.g., "Tenant admins (within same tenant), cloud infrastructure provider" |
| Transfers to third countries | e.g., "None (EU hosting) / US (Stripe, SCCs)" |
| Retention period | e.g., "Contract duration + 30 days" |
| Technical and organizational measures | e.g., "AES-256 encryption, RBAC, RLS, audit logging" |

### Processor ROPA (Art. 30(2))

| Field | Content |
|-------|---------|
| Processor name and contact | Platform provider name, DPO contact |
| Controller name | Tenant name (per DPA) |
| Categories of processing | e.g., "Data hosting, authentication, audit logging" |
| Transfers to third countries | Per sub-processor list |
| Technical and organizational measures | Security controls as documented in the DPA annex |

### Automation

- ROPA entries are automatically generated when new processing activities are configured in the platform.
- Changes to processing activities trigger ROPA updates.
- ROPA is exportable as PDF and CSV for supervisory authority requests.
- The compliance dashboard shows ROPA completeness metrics and flags activities without documented legal basis.

---

## SuperAdmin Features Supporting GDPR

The SuperAdmin platform includes built-in features specifically designed to support GDPR compliance.

### Audit Logging

- Every data access, modification, and deletion is logged with actor, action, target, timestamp, IP address, and tenant context.
- Logs are immutable (append-only) and cryptographically chained.
- Audit logs support the accountability principle and provide evidence for supervisory authority requests.
- Log retention is configurable per tenant (minimum 1 year, default 7 years).

### Data Export

- One-click data export for data subject access requests.
- Formats: JSON (machine-readable), CSV, PDF (human-readable).
- Export includes all personal data across all platform modules.
- Export is generated asynchronously and delivered via time-limited signed URL.

### Deletion Workflows

- Multi-step deletion workflow with verification, approval, and confirmation stages.
- Cascading deletion across all data stores (database, cache, search index, file storage).
- Deletion certificate generated as proof of erasure.
- Legal hold capability to prevent deletion when data is subject to litigation or regulatory retention.

### Consent Tracking

- Granular consent records per purpose and per data subject.
- Consent versioning tracks which version of the consent text the data subject agreed to.
- Consent withdrawal is processed in real time and propagated to all downstream systems.
- Consent dashboard shows consent rates, withdrawal trends, and compliance metrics.

### Privacy Center

- Tenant-configurable privacy center accessible to data subjects.
- Self-service access to: data export, consent management, profile editing, and deletion request submission.
- Branded per tenant with configurable privacy policy and cookie policy content.

### Data Classification

- All data fields in the platform are tagged with a classification level: Public, Internal, Confidential, Restricted.
- Personal data fields are further tagged with GDPR categories: Identifier, Contact, Financial, Special Category.
- Classification drives encryption, access control, and retention policies automatically.

---

## Technical Measures

### Encryption

| Layer | Method | Standard |
|-------|--------|----------|
| Data at rest | AES-256-GCM | NIST SP 800-38D |
| Data in transit | TLS 1.3 | RFC 8446 |
| Application-level | Envelope encryption (KMS) | AWS KMS / Vault |
| Password hashing | bcrypt (cost 12) | OpenBSD bcrypt |
| Backup encryption | AES-256-GCM | NIST SP 800-38D |

### Pseudonymization

- The platform supports pseudonymization for analytics and reporting use cases.
- Pseudonymized datasets replace direct identifiers (name, email) with reversible tokens.
- The mapping table is stored separately with restricted access (only the DPO and Security Lead have access).
- Pseudonymization is used for: aggregate analytics, test environments, shared reporting.

### Access Controls

- **RBAC**: 7-level role hierarchy ensures least-privilege access.
- **RLS**: PostgreSQL Row-Level Security policies scope every query to the authenticated tenant.
- **MFA**: Enforced for all admin roles.
- **Session management**: Configurable idle timeouts (default 30 minutes), absolute timeouts (default 24 hours).
- **IP allowlisting**: Available for enterprise tenants to restrict admin access to known IP ranges.

### Data Integrity

- Database transactions with ACID compliance ensure data consistency.
- Audit log cryptographic chaining detects tampering.
- Checksums on file uploads verify integrity during transfer and storage.
- Regular data integrity checks compare primary and replica databases.

### Availability and Resilience

- Multi-AZ deployment ensures high availability.
- Automated backups every 6 hours with point-in-time recovery.
- Disaster recovery plan with RPO < 1 hour and RTO < 4 hours.
- Regular disaster recovery testing (quarterly).

---

*This document is reviewed quarterly by the Data Protection Officer. Last review: Q1 2026.*
