# SOX Compliance Guide

This document describes how the SuperAdmin SaaS platform supports compliance with the **Sarbanes-Oxley Act (SOX)**, specifically the IT-related provisions of Sections 302 and 404. It covers IT General Controls (ITGCs), access controls, segregation of duties, change management, audit trails, and financial data integrity.

---

## Table of Contents

1. [SOX Overview and IT Relevance](#sox-overview-and-it-relevance)
2. [Section 302: Corporate Responsibility for Financial Reports](#section-302-corporate-responsibility-for-financial-reports)
3. [Section 404: Management Assessment of Internal Controls](#section-404-management-assessment-of-internal-controls)
4. [IT General Controls (ITGCs)](#it-general-controls-itgcs)
5. [Access Controls and Segregation of Duties](#access-controls-and-segregation-of-duties)
6. [Change Management Controls](#change-management-controls)
7. [Audit Trail Requirements](#audit-trail-requirements)
8. [Financial Data Integrity](#financial-data-integrity)
9. [Mapping to SuperAdmin Features](#mapping-to-superadmin-features)

---

## SOX Overview and IT Relevance

The Sarbanes-Oxley Act of 2002 was enacted to protect investors from fraudulent financial reporting. While SOX is primarily a financial reporting regulation, its requirements have significant implications for IT systems that process, store, or report financial data.

### Key SOX Sections Relevant to IT

| Section | Title | IT Relevance |
|---------|-------|-------------|
| **Section 302** | Corporate Responsibility for Financial Reports | CEOs and CFOs must certify the accuracy of financial reports. IT systems that generate or process financial data must have adequate controls. |
| **Section 404** | Management Assessment of Internal Controls | Management must assess the effectiveness of internal controls over financial reporting (ICFR). IT controls are a critical component of ICFR. |
| Section 802 | Criminal Penalties for Altering Documents | IT systems must prevent unauthorized alteration of financial records. Audit logs must be immutable. |
| Section 806 | Protection for Whistleblowers | IT systems must provide secure, anonymous reporting channels. |
| Section 409 | Real-Time Issuer Disclosures | IT systems must support rapid disclosure of material changes to financial condition. |

### COSO Framework Mapping

SOX compliance is typically assessed against the **COSO Internal Control — Integrated Framework (2013)**. The five COSO components map to IT controls as follows:

| COSO Component | IT Control Area |
|----------------|----------------|
| Control Environment | IT governance, security policies, organizational structure |
| Risk Assessment | IT risk assessment, vulnerability management |
| Control Activities | ITGCs (access, change, operations, SDLC) |
| Information and Communication | System-generated reports, data integrity, audit trails |
| Monitoring Activities | Continuous monitoring, audit log review, compliance dashboards |

---

## Section 302: Corporate Responsibility for Financial Reports

### Requirements

Section 302 requires the CEO and CFO to certify that:

1. Financial statements are accurate and complete.
2. They are responsible for establishing and maintaining internal controls.
3. They have designed internal controls to ensure material information is made known to them.
4. They have evaluated the effectiveness of internal controls within 90 days prior to the report.
5. They have disclosed any significant changes in internal controls.

### IT Implications

| Certification Requirement | IT Control Needed | SuperAdmin Implementation |
|--------------------------|------------------|--------------------------|
| Financial statements are accurate | Data integrity controls; input validation; reconciliation | Zod schema validation on all financial data inputs; database constraints; automated reconciliation reports |
| Internal controls are established | Documented IT policies; access controls; change management | SECURITY.md; RBAC with 7-level hierarchy; Git-based change management |
| Material information flows to management | Financial dashboards; automated alerts; audit trails | Real-time financial dashboards; alert thresholds for material transactions; comprehensive audit logging |
| Controls are evaluated regularly | Periodic access reviews; control testing; monitoring | Quarterly access reviews; automated compliance checks; continuous monitoring dashboards |
| Changes in controls are disclosed | Change logging; version control; approval tracking | All config changes audit-logged; Git history for all code changes; approval records |

---

## Section 404: Management Assessment of Internal Controls

### Requirements

Section 404 requires:

- **404(a)**: Management must include an assessment of ICFR effectiveness in the annual report.
- **404(b)**: The external auditor must attest to management's assessment (for accelerated filers).

### IT Control Objectives for Section 404

| Control Objective | Description | Key Controls |
|------------------|-------------|-------------|
| **Completeness** | All transactions are recorded | Automated transaction logging; reconciliation checks; missing-transaction alerts |
| **Accuracy** | Transactions are recorded correctly | Input validation; calculation verification; data type enforcement |
| **Validity** | Only authorized transactions are recorded | Authentication + authorization checks; approval workflows |
| **Restricted access** | Only authorized users can access financial data | RBAC; RLS; MFA; segregation of duties |
| **Timeliness** | Transactions are recorded promptly | Real-time processing; batch job monitoring; SLA alerts |

### ICFR Assessment Workflow

```
1. Identify Financially Significant Systems
   └── SuperAdmin processes: billing, subscriptions, tenant financials, licensing

2. Identify Risks to Financial Reporting
   └── Unauthorized transactions, data manipulation, incomplete records, access violations

3. Identify Controls that Mitigate Risks
   └── RBAC, audit logging, approval workflows, input validation, reconciliation

4. Test Controls for Design Effectiveness
   └── Review control documentation; walk through processes; verify design

5. Test Controls for Operating Effectiveness
   └── Sample testing; re-performance; observation; inquiry

6. Evaluate Deficiencies
   └── Classify as: Control deficiency, Significant deficiency, Material weakness

7. Report Results
   └── Management assessment report; remediation plans for deficiencies
```

---

## IT General Controls (ITGCs)

ITGCs are the foundation of SOX IT compliance. They ensure that application-level controls operate effectively. The four ITGC domains are:

### 1. Logical Access Controls

Controls that ensure only authorized users can access systems and data.

| Control | Description | SuperAdmin Implementation |
|---------|-------------|--------------------------|
| **User provisioning** | New users are granted access based on approved requests | Role-based provisioning workflow; manager approval required; documented in audit log |
| **User deprovisioning** | Access is revoked promptly when no longer needed | Automated deactivation within 4 hours of termination; 90-day inactive account suspension |
| **Access reviews** | User access is reviewed periodically | Quarterly automated access reviews; managers certify their team's access; exceptions tracked |
| **Privileged access** | Elevated access is tightly controlled | Super Admin and Admin roles require MFA; JIT access for production; all privileged actions logged |
| **Password management** | Strong passwords are enforced | 12-character minimum; complexity requirements; breach database check; 90-day rotation (waived with MFA) |
| **MFA** | Multi-factor authentication for sensitive access | TOTP and WebAuthn for all admin roles; enforced at platform level |
| **Segregation of duties** | No single user can perform conflicting actions | RBAC prevents self-approval; 4-eyes principle for financial transactions |

### 2. Change Management Controls

Controls that ensure changes to systems are authorized, tested, and documented.

| Control | Description | SuperAdmin Implementation |
|---------|-------------|--------------------------|
| **Change request** | All changes are formally requested and documented | Git-based workflow; all changes require a pull request with description |
| **Change approval** | Changes are approved before implementation | Minimum 2 PR approvals required; protected branches; no direct push to main |
| **Testing** | Changes are tested before deployment | CI pipeline: unit, integration, E2E, security, performance tests |
| **Deployment** | Changes are deployed through a controlled process | Jenkins pipeline; automated build and deploy; no manual production deployments |
| **Rollback** | Failed changes can be reversed | Blue-green deployment; K8s rolling updates with automatic rollback; database migrations are reversible |
| **Emergency changes** | Emergency changes follow a documented exception process | Emergency change procedure with post-hoc approval within 24 hours; documented in audit log |
| **Separation of duties** | Developers cannot deploy their own code without review | PR author cannot be the sole approver; deployment requires CI pipeline approval |

### 3. Computer Operations

Controls that ensure systems operate reliably and data is protected.

| Control | Description | SuperAdmin Implementation |
|---------|-------------|--------------------------|
| **Job scheduling** | Batch jobs run as scheduled | K8s CronJobs; monitoring for failed jobs; alerting on missed schedules |
| **Backup and recovery** | Data is backed up and can be restored | Automated backups every 6 hours; encrypted; cross-region; tested quarterly |
| **Incident management** | Incidents are detected, reported, and resolved | PagerDuty on-call; incident response playbooks; post-incident reviews |
| **Monitoring** | Systems are monitored for availability and performance | Prometheus/Grafana; uptime monitoring; SLA dashboards |
| **Disaster recovery** | Business continuity in case of disaster | Multi-AZ deployment; RPO < 1h, RTO < 4h; DR testing quarterly |

### 4. Program Development (SDLC)

Controls that ensure new systems and changes are developed securely.

| Control | Description | SuperAdmin Implementation |
|---------|-------------|--------------------------|
| **Requirements** | Business requirements are documented and approved | User stories with acceptance criteria; product owner sign-off |
| **Design review** | System design includes security considerations | Threat modeling for new features; architecture review for changes affecting auth/data |
| **Coding standards** | Code follows secure coding standards | ESLint rules; SonarQube quality gates; TypeScript strict mode |
| **Code review** | All code is peer-reviewed | Mandatory code review (minimum 2 approvals); automated checks |
| **Security testing** | Code is tested for vulnerabilities | SAST (SonarQube), DAST (OWASP ZAP), dependency scanning (Trivy) |
| **User acceptance** | Changes are validated by stakeholders | UAT environment; product owner approval before production deployment |
| **Documentation** | System and user documentation is maintained | API documentation (OpenAPI); runbooks; architecture decision records |

---

## Access Controls and Segregation of Duties

### RBAC for SOX Compliance

The SuperAdmin RBAC model enforces segregation of duties required by SOX.

| Role | Financial Capabilities | Restricted From |
|------|----------------------|----------------|
| **Super Admin** | Full system configuration; user management; audit log access | Cannot process financial transactions (conflicting duty) |
| **Admin** | Tenant management; reporting; user management within tenant | Cannot approve own financial actions |
| **Finance Manager** | Create and approve invoices, process refunds | Cannot modify RBAC roles; cannot access system configuration |
| **Finance Analyst** | View financial reports; create billing adjustments | Cannot approve transactions above threshold |
| **Auditor** | Read-only access to all financial data, audit logs, and reports | Cannot modify any data; cannot create transactions |
| **Member** | View own billing history; make payments | Cannot access other users' financial data |

### Segregation of Duties Matrix

The following conflicting duties are enforced by the RBAC system.

| Duty A | Duty B | Conflict | Enforcement |
|--------|--------|---------|------------|
| Create purchase order | Approve purchase order | Self-approval of purchases | Different roles required; system blocks same-user approval |
| Create user account | Approve user access | Self-provisioning of access | Admin creates; Security reviews and approves |
| Develop code | Deploy to production | Unreviewed code deployment | CI/CD pipeline requires separate approver; code author cannot deploy |
| Process refund | Record refund in ledger | Concealing unauthorized refunds | Finance Manager processes; system auto-records; Auditor reviews |
| Manage RBAC | Perform financial transactions | Self-granting financial permissions | Super Admin manages RBAC; Finance Manager handles transactions |
| Generate financial report | Modify underlying data | Report manipulation | Auditor role is read-only; data modifications logged separately |

### Access Review Process

```
Quarterly Access Review Workflow:

1. System generates access review report
   ├── All users with their current roles and permissions
   ├── Users added/removed since last review
   ├── Users with elevated (admin+) access
   └── Inactive accounts (no login in 60+ days)

2. Managers certify their team's access
   ├── Confirm each user's role is appropriate
   ├── Flag any excessive permissions
   └── Sign off on the review

3. Security Team reviews privileged access
   ├── Super Admin and Admin accounts verified
   ├── Service accounts reviewed
   └── Cross-tenant access verified as zero

4. Exceptions remediated
   ├── Excessive permissions removed
   ├── Orphaned accounts deactivated
   └── Remediation logged in audit trail

5. Review results documented
   ├── Completion report generated
   ├── Signed off by CISO
   └── Evidence retained for audit
```

---

## Change Management Controls

### Change Types

| Type | Approval Required | Testing Required | Documentation |
|------|------------------|-----------------|---------------|
| **Standard change** | 2 PR approvals + CI pipeline pass | Full test suite (unit, integration, E2E, security) | PR description; commit messages; CI reports |
| **Emergency change** | 1 approval + post-hoc review within 24h | Minimum: unit tests + manual smoke test | Emergency change form; post-hoc PR with full testing |
| **Infrastructure change** | Terraform plan review + Security Team approval | Terraform plan + staging validation | IaC diff; approval records; deployment log |
| **Configuration change** | Admin approval + Security Team review (if security-relevant) | Staging validation | Config change audit log; before/after values |
| **Database migration** | 2 PR approvals + DBA review | Migration tested on staging with production-like data | Migration script; rollback script; test results |

### Change Control Evidence

For SOX audit purposes, the following evidence is maintained for every change.

| Evidence | Source | Retention |
|----------|--------|-----------|
| Change request (PR description) | GitHub/GitLab | Indefinite (git history) |
| Approval records | PR approvals; deployment gate records | Indefinite (git history) |
| Test results | CI pipeline artifacts (JUnit XML, coverage reports) | 12 months |
| Deployment log | Jenkins build log; K8s deployment events | 12 months |
| Rollback verification | Rollback test results (for database migrations) | 12 months |
| Post-implementation review | PR comments; deployment verification | Indefinite (git history) |

---

## Audit Trail Requirements

### SOX Audit Trail Scope

SOX requires an audit trail for all activities that affect financial reporting. The SuperAdmin platform logs the following categories.

| Category | Events Logged | Example |
|----------|--------------|---------|
| **Authentication** | Login, logout, failed login, MFA events, session creation/expiry | `user.login.success { userId: 'usr_123', ip: '10.0.0.1', mfaMethod: 'totp' }` |
| **Authorization** | Permission checks, access denied events, role changes | `access.denied { userId: 'usr_123', resource: '/api/v1/billing', reason: 'insufficient_role' }` |
| **Financial transactions** | Invoice creation, payment processing, refund, subscription changes | `billing.invoice.created { invoiceId: 'inv_456', tenantId: 'tnt_789', amount: 9900, currency: 'USD' }` |
| **Data modifications** | Create, update, delete operations on financial data | `tenant.plan.updated { tenantId: 'tnt_789', from: 'starter', to: 'professional', approvedBy: 'usr_admin' }` |
| **Configuration changes** | RBAC changes, system settings, feature flags | `rbac.role.assigned { targetUserId: 'usr_456', role: 'finance_manager', assignedBy: 'usr_admin' }` |
| **System events** | Deployment, backup, restore, scheduled job execution | `system.deploy { version: '3.2.1', commitSha: 'abc123', deployedBy: 'jenkins', environment: 'production' }` |

### Audit Log Record Structure

Every audit log entry contains the following fields.

| Field | Type | Description | Required |
|-------|------|-------------|----------|
| `id` | UUID | Unique log entry identifier | Yes |
| `timestamp` | ISO 8601 | Event timestamp in UTC | Yes |
| `event_type` | String | Hierarchical event type (e.g., `billing.payment.processed`) | Yes |
| `actor_id` | UUID | User or service account that performed the action | Yes |
| `actor_role` | String | Role of the actor at the time of the event | Yes |
| `actor_ip` | String | IP address of the actor | Yes |
| `tenant_id` | UUID | Tenant context of the event | Yes |
| `resource_type` | String | Type of resource affected (e.g., `invoice`, `user`, `config`) | Yes |
| `resource_id` | UUID | Identifier of the affected resource | Yes |
| `action` | String | Action performed (create, read, update, delete, approve, deny) | Yes |
| `details` | JSON | Action-specific details (before/after values for updates) | Yes |
| `result` | String | Outcome (success, failure, denied) | Yes |
| `correlation_id` | UUID | Request correlation ID for tracing | Yes |
| `checksum` | String | SHA-256 hash chained to previous entry for integrity | Yes |

### Audit Log Integrity

To comply with SOX Section 802 (criminal penalties for altering documents):

- **Append-only storage**: Audit logs are written to append-only storage. No delete or update operations are permitted.
- **Cryptographic chaining**: Each log entry includes a SHA-256 hash that chains to the previous entry, creating a tamper-evident chain.
- **Immutable backup**: Audit logs are replicated to S3 with Object Lock (WORM — Write Once Read Many) enabled.
- **Integrity verification**: A daily automated job verifies the cryptographic chain and reports any breaks.
- **Retention**: Audit logs are retained for a minimum of 7 years (SOX requirement for financial records).

### Audit Log Access Control

| Role | Permissions |
|------|------------|
| **Auditor** | Read all audit logs; export logs; run reports |
| **Super Admin** | Read all audit logs; cannot delete or modify |
| **Admin** | Read audit logs within own tenant; cannot access cross-tenant logs |
| **All other roles** | No direct access to audit logs |

---

## Financial Data Integrity

### Controls for Financial Data Accuracy

| Control | Implementation | Verification |
|---------|---------------|-------------|
| **Input validation** | Zod schemas validate all financial data inputs: amounts (positive integers in cents), currencies (ISO 4217), dates (ISO 8601), tax IDs (format-specific regex) | Automated unit tests for all validation schemas |
| **Calculation verification** | All financial calculations are performed server-side; client-side calculations are for display only and re-verified on submission | Unit tests for all calculation functions; integration tests verify end-to-end accuracy |
| **Data type enforcement** | Financial amounts stored as integers (cents) to avoid floating-point errors; currencies stored as ISO 4217 codes | Database column constraints; ORM type enforcement |
| **Referential integrity** | Foreign key constraints ensure all financial records reference valid entities | Database constraints; migration tests |
| **Reconciliation** | Automated daily reconciliation between internal records and Stripe | Reconciliation reports; discrepancy alerts |
| **Idempotency** | Payment operations use idempotency keys to prevent duplicate transactions | Unique constraint on idempotency keys; retry logic verification |
| **Immutable records** | Financial transactions are append-only; corrections are recorded as new entries (credit/debit), not overwrites | Soft-delete only; no UPDATE on transaction tables |

### Financial Report Generation Controls

| Control | Description |
|---------|-------------|
| **Access restriction** | Only Auditor and Finance Manager roles can generate financial reports |
| **Parameter validation** | Report date ranges, filters, and groupings are validated against allowed values |
| **Consistency checks** | Reports include checksums; totals are independently calculated and cross-verified |
| **Versioning** | Generated reports are stored with a version identifier and timestamp |
| **Non-repudiation** | Report generation events are audit-logged with the generating user, parameters, and output hash |

---

## Mapping to SuperAdmin Features

### Comprehensive Feature Mapping

| SOX Requirement | SuperAdmin Feature | Documentation Reference |
|----------------|-------------------|------------------------|
| **Section 302 — CEO/CFO Certification** | | |
| Accurate financial statements | Input validation (Zod); calculation verification; reconciliation | SUPER_ADMIN_DOCUMENTATION.md Section 12 |
| Internal controls maintained | RBAC; audit logging; change management; monitoring | SECURITY.md; 03_rbac_roles.csv |
| Material information reporting | Financial dashboards; alert thresholds; real-time notifications | 09_admin_dashboard_tabs.csv |
| **Section 404 — ICFR Assessment** | | |
| Completeness | Transaction logging; reconciliation; missing-transaction alerts | 06_audit_log_actions.csv |
| Accuracy | Zod validation; integer arithmetic for financial amounts; type enforcement | 13_input_sanitization.csv |
| Validity | Authentication (MFA); authorization (RBAC + RLS); approval workflows | 07_permissions.csv; 12_password_policy.csv |
| Restricted access | RBAC (7 levels); RLS; MFA; segregation of duties | 03_rbac_roles.csv; 07_permissions.csv |
| Timeliness | Real-time processing; cron job monitoring; SLA alerting | INFRASTRUCTURE_STACK.md |
| **ITGC — Access Controls** | | |
| User provisioning | Role-based provisioning workflow; manager approval | User management module |
| User deprovisioning | 4-hour deactivation; 90-day inactivity suspension | Offboarding checklist |
| Privileged access | MFA enforced; JIT access; all actions logged | 08_threat_detection.csv |
| Access reviews | Quarterly automated reports; manager certification | Compliance dashboard |
| **ITGC — Change Management** | | |
| Change authorization | PR approvals (2 required); protected branches | Git configuration |
| Change testing | CI pipeline (unit, integration, E2E, security) | TESTING.md; Jenkinsfile |
| Change deployment | Automated pipeline; no manual production access | INFRASTRUCTURE_STACK.md |
| Change rollback | Blue-green deployment; K8s rolling updates | K8s deployment configuration |
| **ITGC — Computer Operations** | | |
| Backup and recovery | 6-hour automated backups; encrypted; tested quarterly | INFRASTRUCTURE_STACK.md |
| Monitoring | Prometheus/Grafana; PagerDuty; SLA dashboards | Monitoring configuration |
| Incident management | Incident response plan; on-call rotation; post-incident reviews | Security-Audit-Plan.md |
| **ITGC — Program Development** | | |
| Secure coding | ESLint; SonarQube; TypeScript strict; code review | TESTING.md |
| Security testing | Trivy; SonarQube; OWASP ZAP; penetration testing | TESTING.md; Security-Audit-Plan.md |
| **Audit Trail** | | |
| Transaction logging | Comprehensive audit logging for all financial events | 06_audit_log_actions.csv |
| Log integrity | Cryptographic chaining; append-only; WORM storage | SECURITY.md |
| Log retention | 7-year retention for financial audit logs | Retention configuration |
| Log access control | Auditor role (read-only); no delete/modify permissions | 07_permissions.csv |
| **Financial Data Integrity** | | |
| Data validation | Zod schemas; database constraints; referential integrity | 13_input_sanitization.csv |
| Reconciliation | Automated daily reconciliation with Stripe | Reconciliation reports |
| Immutable records | Append-only financial transactions; soft-delete only | Database schema design |

### Evidence Collection for Auditors

The platform provides a dedicated **SOX Audit Evidence Package** that auditors can request, containing:

1. **Access control evidence**: Current RBAC configuration; quarterly access review reports; provisioning/deprovisioning logs.
2. **Change management evidence**: Git history for all production changes; PR approval records; CI pipeline results; deployment logs.
3. **Audit trail evidence**: Audit log exports for the review period; integrity verification reports.
4. **Financial controls evidence**: Reconciliation reports; transaction logs; report generation logs.
5. **IT operations evidence**: Backup success reports; recovery test results; incident records; monitoring dashboards.
6. **Policy evidence**: Current versions of all security and IT policies; policy review records; training completion records.

---

*This document is reviewed annually in preparation for the SOX audit cycle. Last review: Q1 2026.*
