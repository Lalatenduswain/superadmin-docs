# ISO 27001 Compliance Guide

This document maps the SuperAdmin SaaS platform's security controls to **ISO/IEC 27001:2022** requirements and Annex A controls. It serves as a reference for implementing, maintaining, and auditing the Information Security Management System (ISMS).

---

## Table of Contents

1. [ISMS Scope and Context](#isms-scope-and-context)
2. [Annex A Controls Mapping](#annex-a-controls-mapping)
3. [Control Categories](#control-categories)
4. [Risk Assessment Methodology](#risk-assessment-methodology)
5. [Statement of Applicability](#statement-of-applicability)
6. [Internal Audit Checklist](#internal-audit-checklist)
7. [Continuous Improvement (PDCA Cycle)](#continuous-improvement-pdca-cycle)
8. [Key Controls Mapping to SuperAdmin](#key-controls-mapping-to-superadmin)

---

## ISMS Scope and Context

### Scope Statement

The ISMS covers the design, development, deployment, operation, and maintenance of the **SuperAdmin multi-tenant SaaS platform**, including:

- All application components (frontend, backend API, background workers).
- Supporting infrastructure (cloud hosting, databases, caching, message queues, CI/CD pipelines).
- Administrative processes (access management, change management, incident response, vendor management).
- Personnel involved in development, operations, security, and customer support.

### Boundaries

| In Scope | Out of Scope |
|----------|-------------|
| Application code and configuration | Customer-side network infrastructure |
| Cloud infrastructure (AWS eu-central-1) | Customer endpoint devices |
| CI/CD pipeline (Jenkins, Docker, K8s) | Third-party SaaS tools used by customers |
| Employee workstations and access | Physical office security (covered by separate policy) |
| Sub-processor management | Customer's compliance obligations beyond our processor role |

### Context of the Organization

**Internal factors:**

- Multi-tenant architecture requires tenant-level isolation as a core security objective.
- Rapid development cycles require security to be embedded in the SDLC (shift-left approach).
- Distributed engineering team requires secure remote access controls.

**External factors:**

- Regulatory requirements: GDPR, sector-specific regulations of customer industries.
- Customer contractual requirements for security certifications (ISO 27001, SOC 2).
- Evolving threat landscape: Ransomware, supply chain attacks, API abuse, post-quantum threats.

### Interested Parties

| Party | Expectations |
|-------|-------------|
| Customers (tenants) | Data confidentiality, availability, integrity; compliance certifications |
| End users | Privacy, data accuracy, secure access |
| Regulatory authorities | GDPR compliance, breach notification, ROPA |
| Employees | Clear security policies, training, incident reporting channels |
| Shareholders / Management | Risk management, business continuity, reputation protection |
| Sub-processors | Clear DPAs, security requirements, audit rights |

---

## Annex A Controls Mapping

ISO 27001:2022 Annex A contains **93 controls** organized into **4 themes**. The table below provides the high-level mapping.

### Summary by Theme

| Theme | Control Count | Applicable | Implemented | Partially Implemented | Not Applicable |
|-------|--------------|-----------|------------|----------------------|----------------|
| A.5 Organizational (37 controls) | 37 | 35 | 32 | 3 | 2 |
| A.6 People (8 controls) | 8 | 8 | 7 | 1 | 0 |
| A.7 Physical (14 controls) | 14 | 6 | 5 | 1 | 8 |
| A.8 Technological (34 controls) | 34 | 33 | 30 | 3 | 1 |
| **Total** | **93** | **82** | **74** | **8** | **11** |

---

## Control Categories

### A.5 — Organizational Controls

Organizational controls establish governance, policies, and management structures for information security.

| Control ID | Control Name | Applicability | Implementation Status | SuperAdmin Mapping |
|-----------|-------------|--------------|----------------------|-------------------|
| A.5.1 | Policies for information security | Applicable | Implemented | Security policies documented in SUPER_ADMIN_DOCUMENTATION.md; policy management module in admin panel |
| A.5.2 | Information security roles and responsibilities | Applicable | Implemented | RBAC with 7-level hierarchy; documented role descriptions; security responsibilities in job descriptions |
| A.5.3 | Segregation of duties | Applicable | Implemented | RBAC prevents same user from approving own requests; 4-eyes principle for config changes |
| A.5.4 | Management responsibilities | Applicable | Implemented | Quarterly security review by management; security metrics dashboard |
| A.5.5 | Contact with authorities | Applicable | Implemented | DPO registered with supervisory authority; incident response plan includes authority contacts |
| A.5.6 | Contact with special interest groups | Applicable | Implemented | Membership in ISACs; OWASP participation; security mailing list subscriptions |
| A.5.7 | Threat intelligence | Applicable | Implemented | Automated CVE monitoring; threat intelligence feeds integrated into SIEM |
| A.5.8 | Information security in project management | Applicable | Implemented | Security review gate in SDLC; threat modeling for new features |
| A.5.9 | Inventory of information and other associated assets | Applicable | Implemented | Data catalog with classification; infrastructure inventory via Terraform state |
| A.5.10 | Acceptable use of information and other associated assets | Applicable | Implemented | Acceptable use policy; enforced via DLP and access controls |
| A.5.11 | Return of assets | Applicable | Implemented | Offboarding checklist includes asset return; remote wipe for company devices |
| A.5.12 | Classification of information | Applicable | Implemented | 4-level classification (Public, Internal, Confidential, Restricted); enforced in data catalog |
| A.5.13 | Labelling of information | Applicable | Implemented | Automated labeling in data catalog; email classification labels |
| A.5.14 | Information transfer | Applicable | Implemented | TLS 1.3 for all data in transit; encrypted email for sensitive communications |
| A.5.15 | Access control | Applicable | Implemented | RBAC, ABAC, RLS; documented access control policy |
| A.5.16 | Identity management | Applicable | Implemented | Centralized identity provider; unique user IDs; lifecycle management |
| A.5.17 | Authentication information | Applicable | Implemented | bcrypt password hashing; MFA; password policy (min 12 chars, breach check) |
| A.5.18 | Access rights | Applicable | Implemented | Least privilege; quarterly access reviews; automated deprovisioning |
| A.5.19 | Information security in supplier relationships | Applicable | Implemented | Sub-processor DPAs; vendor security assessments; supplier register |
| A.5.20 | Addressing information security within supplier agreements | Applicable | Implemented | Security clauses in all vendor contracts; right to audit |
| A.5.21 | Managing information security in the ICT supply chain | Applicable | Implemented | SBOM generation; dependency scanning; private package registry |
| A.5.22 | Monitoring, review and change management of supplier services | Applicable | Implemented | Annual vendor reviews; automated uptime monitoring |
| A.5.23 | Information security for use of cloud services | Applicable | Implemented | Cloud security configuration; CIS benchmarks; Infrastructure as Code |
| A.5.24 | Information security incident management planning and preparation | Applicable | Implemented | Incident response plan; runbooks; on-call rotation |
| A.5.25 | Assessment and decision on information security events | Applicable | Implemented | SIEM with severity classification; triage procedures |
| A.5.26 | Response to information security incidents | Applicable | Implemented | Incident response playbooks; automated containment for critical events |
| A.5.27 | Learning from information security incidents | Applicable | Implemented | Post-incident reviews; lessons learned database; process improvements |
| A.5.28 | Collection of evidence | Applicable | Implemented | Immutable audit logs; forensic imaging procedures; chain of custody |
| A.5.29 | Information security during disruption | Applicable | Implemented | BCP; multi-AZ deployment; disaster recovery testing |
| A.5.30 | ICT readiness for business continuity | Applicable | Implemented | RPO < 1h, RTO < 4h; automated failover; backup verification |
| A.5.31 | Legal, statutory, regulatory and contractual requirements | Applicable | Implemented | Compliance module tracking GDPR, PCI DSS, SOX; legal register |
| A.5.32 | Intellectual property rights | Applicable | Implemented | License compliance scanning; OSS license audit |
| A.5.33 | Protection of records | Applicable | Implemented | Immutable audit logs; retention policies; backup encryption |
| A.5.34 | Privacy and protection of PII | Applicable | Implemented | GDPR compliance module; consent management; DPIAs; ROPA |
| A.5.35 | Independent review of information security | Applicable | Partially Implemented | Annual external audit planned; internal reviews active |
| A.5.36 | Compliance with policies, rules and standards | Applicable | Implemented | Automated compliance checks; policy acknowledgement tracking |
| A.5.37 | Documented operating procedures | Applicable | Implemented | Runbooks in Confluence; IaC documentation; API documentation |

### A.6 — People Controls

People controls address human resource security from hiring through termination.

| Control ID | Control Name | Applicability | Implementation Status | SuperAdmin Mapping |
|-----------|-------------|--------------|----------------------|-------------------|
| A.6.1 | Screening | Applicable | Implemented | Background checks for all roles with system access; reference verification |
| A.6.2 | Terms and conditions of employment | Applicable | Implemented | Security responsibilities in employment contracts; NDA signed at hire |
| A.6.3 | Information security awareness, education and training | Applicable | Implemented | Annual security awareness training; phishing simulations; role-specific training |
| A.6.4 | Disciplinary process | Applicable | Implemented | Documented disciplinary process for security policy violations |
| A.6.5 | Responsibilities after termination or change of employment | Applicable | Implemented | NDA survives termination; offboarding checklist; access revocation within 4 hours |
| A.6.6 | Confidentiality or non-disclosure agreements | Applicable | Implemented | NDA for all employees and contractors; reviewed annually |
| A.6.7 | Remote working | Applicable | Implemented | Remote work security policy; VPN required; endpoint protection; encrypted storage |
| A.6.8 | Information security event reporting | Applicable | Partially Implemented | Reporting channels established; dedicated Slack channel and email; whistleblower protection in progress |

### A.7 — Physical Controls

Physical controls address physical security of premises and equipment. As a cloud-native SaaS platform, many physical controls are addressed by the cloud service provider (AWS).

| Control ID | Control Name | Applicability | Implementation Status | Notes |
|-----------|-------------|--------------|----------------------|-------|
| A.7.1 | Physical security perimeters | Not Applicable | N/A | AWS manages data center physical security (SOC 2 Type II attested) |
| A.7.2 | Physical entry | Not Applicable | N/A | AWS manages data center access |
| A.7.3 | Securing offices, rooms and facilities | Applicable | Implemented | Office access controls; visitor management |
| A.7.4 | Physical security monitoring | Not Applicable | N/A | AWS manages data center monitoring |
| A.7.5 | Protecting against physical and environmental threats | Not Applicable | N/A | AWS multi-AZ deployment mitigates environmental risks |
| A.7.6 | Working in secure areas | Not Applicable | N/A | No on-premise secure areas for platform operations |
| A.7.7 | Clear desk and clear screen | Applicable | Implemented | Clear desk policy; auto-lock after 5 minutes of inactivity |
| A.7.8 | Equipment siting and protection | Not Applicable | N/A | Cloud-native — no on-premise servers |
| A.7.9 | Security of assets off-premises | Applicable | Implemented | Encrypted laptops; remote wipe capability; VPN requirement |
| A.7.10 | Storage media | Applicable | Implemented | Encrypted storage; secure disposal of media via vendor |
| A.7.11 | Supporting utilities | Not Applicable | N/A | AWS manages power, cooling, connectivity |
| A.7.12 | Cabling security | Not Applicable | N/A | AWS manages data center cabling |
| A.7.13 | Equipment maintenance | Applicable | Implemented | Laptop maintenance and replacement schedule; vendor service agreements |
| A.7.14 | Secure disposal or re-use of equipment | Applicable | Implemented | NIST 800-88 media sanitization; certificate of destruction |

### A.8 — Technological Controls

Technological controls address technical security measures for systems, networks, and applications.

| Control ID | Control Name | Applicability | Implementation Status | SuperAdmin Mapping |
|-----------|-------------|--------------|----------------------|-------------------|
| A.8.1 | User endpoint devices | Applicable | Implemented | MDM enrollment; disk encryption; OS patching; endpoint detection |
| A.8.2 | Privileged access rights | Applicable | Implemented | Super Admin and Admin roles restricted; JIT access for production; MFA enforced |
| A.8.3 | Information access restriction | Applicable | Implemented | RBAC + RLS; 70+ granular permissions; field-level access control |
| A.8.4 | Access to source code | Applicable | Implemented | Branch protection; code review required; signed commits |
| A.8.5 | Secure authentication | Applicable | Implemented | MFA; WebAuthn; bcrypt; session management; account lockout |
| A.8.6 | Capacity management | Applicable | Implemented | Auto-scaling (K8s HPA); resource monitoring; capacity planning |
| A.8.7 | Protection against malware | Applicable | Implemented | Container scanning (Trivy); no user-executable code; WAF |
| A.8.8 | Management of technical vulnerabilities | Applicable | Implemented | Trivy, SonarQube, OWASP ZAP; vulnerability SLAs |
| A.8.9 | Configuration management | Applicable | Implemented | IaC (Terraform/Helm); immutable deployments; drift detection |
| A.8.10 | Information deletion | Applicable | Implemented | GDPR deletion workflows; data retention automation; secure overwrite |
| A.8.11 | Data masking | Applicable | Implemented | PII masking in logs; data pseudonymization; test data generation |
| A.8.12 | Data leakage prevention | Applicable | Implemented | DLP rules; API response filtering; output encoding; CSP |
| A.8.13 | Information backup | Applicable | Implemented | Automated backups every 6h; encrypted; cross-region replication; tested recovery |
| A.8.14 | Redundancy of information processing facilities | Applicable | Implemented | Multi-AZ deployment; database replicas; automatic failover |
| A.8.15 | Logging | Applicable | Implemented | Comprehensive audit logging; structured JSON; centralized aggregation (ELK) |
| A.8.16 | Monitoring activities | Applicable | Implemented | Real-time monitoring (Prometheus/Grafana); alerting; anomaly detection |
| A.8.17 | Clock synchronization | Applicable | Implemented | NTP synchronization across all servers; UTC timestamps in all logs |
| A.8.18 | Use of privileged utility programs | Applicable | Implemented | Restricted access to database admin tools; all usage logged |
| A.8.19 | Installation of software on operational systems | Applicable | Implemented | Immutable container images; no runtime package installation |
| A.8.20 | Networks security | Applicable | Implemented | VPC isolation; security groups; network policies in K8s |
| A.8.21 | Security of network services | Applicable | Implemented | TLS 1.3; WAF; DDoS protection (Cloudflare); API gateway |
| A.8.22 | Segregation of networks | Applicable | Implemented | Separate VPCs for production, staging, development; private subnets for databases |
| A.8.23 | Web filtering | Applicable | Partially Implemented | Egress filtering; URL allow-listing for outbound requests |
| A.8.24 | Use of cryptography | Applicable | Implemented | AES-256-GCM at rest; TLS 1.3 in transit; bcrypt for passwords; key management via KMS |
| A.8.25 | Secure development life cycle | Applicable | Implemented | Security in SDLC; threat modeling; secure code review; SAST/DAST |
| A.8.26 | Application security requirements | Applicable | Implemented | OWASP Top 10 coverage; input validation; output encoding; CSRF protection |
| A.8.27 | Secure system architecture and engineering principles | Applicable | Implemented | Defense in depth; zero trust; least privilege; fail-secure |
| A.8.28 | Secure coding | Applicable | Implemented | Coding standards; SonarQube quality gate; mandatory code review |
| A.8.29 | Security testing in development and acceptance | Applicable | Implemented | Unit, integration, E2E, SAST, DAST, penetration testing |
| A.8.30 | Outsourced development | Applicable | Partially Implemented | Contractor security requirements; code review of all external contributions |
| A.8.31 | Separation of development, test and production environments | Applicable | Implemented | Separate environments with distinct credentials, networks, and data |
| A.8.32 | Change management | Applicable | Implemented | Git-based change management; PR reviews; approval gates; rollback capability |
| A.8.33 | Test information | Applicable | Implemented | No production data in test environments; synthetic data generators |
| A.8.34 | Protection of information systems during audit testing | Applicable | Partially Implemented | Audit testing in isolated environments; read-only access for auditors |

---

## Risk Assessment Methodology

### Approach

The ISMS uses a **qualitative risk assessment** methodology based on the ISO 27005 framework.

### Risk Identification

Risks are identified through:

- Annual threat landscape review.
- Vulnerability scan results (Trivy, SonarQube, OWASP ZAP).
- Incident post-mortems.
- Audit findings (internal and external).
- Regulatory and customer requirement changes.
- Threat intelligence feeds.

### Risk Evaluation

Each risk is evaluated on two dimensions:

**Likelihood Scale:**

| Level | Rating | Description |
|-------|--------|-------------|
| 1 | Rare | May occur only in exceptional circumstances (< once per 5 years) |
| 2 | Unlikely | Could occur at some time (once per 1–5 years) |
| 3 | Possible | Might occur at some time (once per year) |
| 4 | Likely | Will probably occur in most circumstances (once per quarter) |
| 5 | Almost Certain | Expected to occur frequently (monthly or more) |

**Impact Scale:**

| Level | Rating | Description |
|-------|--------|-------------|
| 1 | Negligible | No financial loss; no reputational impact; no regulatory consequence |
| 2 | Minor | < $10K financial impact; minor reputational impact; no regulatory breach |
| 3 | Moderate | $10K–$100K financial impact; moderate reputational impact; regulatory warning |
| 4 | Major | $100K–$1M financial impact; significant reputational damage; regulatory fine |
| 5 | Catastrophic | > $1M financial impact; severe reputational damage; regulatory sanctions; business continuity threat |

### Risk Matrix

| | Negligible (1) | Minor (2) | Moderate (3) | Major (4) | Catastrophic (5) |
|---|---|---|---|---|---|
| **Almost Certain (5)** | Medium (5) | High (10) | High (15) | Critical (20) | Critical (25) |
| **Likely (4)** | Low (4) | Medium (8) | High (12) | High (16) | Critical (20) |
| **Possible (3)** | Low (3) | Medium (6) | Medium (9) | High (12) | High (15) |
| **Unlikely (2)** | Low (2) | Low (4) | Medium (6) | Medium (8) | High (10) |
| **Rare (1)** | Low (1) | Low (2) | Low (3) | Low (4) | Medium (5) |

### Risk Treatment

| Risk Level | Score | Treatment |
|-----------|-------|-----------|
| Critical | 20–25 | Immediate action required; escalated to management; treatment plan within 7 days |
| High | 10–16 | Treatment plan within 30 days; tracked in risk register |
| Medium | 5–9 | Treatment plan within 90 days; monitored quarterly |
| Low | 1–4 | Accept or monitor; reviewed annually |

### Risk Treatment Options

1. **Mitigate**: Implement controls to reduce likelihood or impact.
2. **Transfer**: Transfer the risk via insurance or outsourcing (e.g., DDoS protection to Cloudflare).
3. **Avoid**: Eliminate the activity that creates the risk.
4. **Accept**: Accept the residual risk with documented management approval.

---

## Statement of Applicability

### SoA Template

The Statement of Applicability (SoA) documents which Annex A controls are applicable, their implementation status, and justification for exclusion of non-applicable controls.

| Control | Control Name | Applicable | Justification | Implementation Status | Reference |
|---------|-------------|-----------|--------------|----------------------|-----------|
| A.5.1 | Policies for information security | Yes | Required for ISMS governance | Implemented | SECURITY.md, SUPER_ADMIN_DOCUMENTATION.md |
| A.5.2 | Information security roles and responsibilities | Yes | Required for accountability | Implemented | RBAC matrix (07_permissions.csv) |
| A.5.3 | Segregation of duties | Yes | Required to prevent unauthorized actions | Implemented | RBAC (03_rbac_roles.csv) |
| A.7.1 | Physical security perimeters | No | Cloud-native platform; physical security managed by AWS (SOC 2 attested) | N/A | AWS SOC 2 report |
| A.7.2 | Physical entry | No | Cloud-native platform; managed by AWS | N/A | AWS SOC 2 report |
| A.8.24 | Use of cryptography | Yes | Required for data protection | Implemented | 10_encryption_standards.csv |
| ... | ... | ... | ... | ... | ... |

The complete SoA is maintained in the compliance module and contains all 93 controls with full justification and evidence references.

---

## Internal Audit Checklist

### Pre-Audit Activities

- [ ] Define audit scope and objectives.
- [ ] Select audit team (independent of area being audited).
- [ ] Prepare audit plan and schedule.
- [ ] Notify auditees at least 2 weeks in advance.
- [ ] Review previous audit findings and corrective actions.
- [ ] Gather relevant documentation (policies, procedures, logs, reports).

### ISMS Core Requirements (Clauses 4–10)

- [ ] **Clause 4 — Context**: Scope document current; interested parties identified; internal/external issues reviewed.
- [ ] **Clause 5 — Leadership**: Management commitment demonstrated; security policy current and communicated; roles and responsibilities assigned.
- [ ] **Clause 6 — Planning**: Risk assessment completed within last 12 months; risk treatment plan current; SoA current.
- [ ] **Clause 7 — Support**: Resources adequate; competence records maintained; awareness training completed; documented information controlled.
- [ ] **Clause 8 — Operation**: Risk treatment implemented; operational planning controls effective; outsourced processes controlled.
- [ ] **Clause 9 — Performance evaluation**: Internal audit completed; management review conducted; monitoring and measurement effective.
- [ ] **Clause 10 — Improvement**: Nonconformities addressed with corrective actions; continual improvement demonstrated.

### Technical Controls Audit

- [ ] **Access control**: RBAC configured correctly; no orphaned accounts; admin access reviewed quarterly.
- [ ] **Encryption**: TLS 1.3 enforced; AES-256 at rest verified; key rotation on schedule.
- [ ] **Logging**: Audit logs complete; no gaps in coverage; log integrity verified.
- [ ] **Vulnerability management**: Scan results reviewed; critical vulnerabilities patched within SLA; no overdue findings.
- [ ] **Change management**: All production changes have PRs; approval records; rollback tested.
- [ ] **Backup and recovery**: Backups successful; recovery tested within last quarter; RPO/RTO met.
- [ ] **Network security**: Security groups reviewed; no unnecessary open ports; egress filtering active.
- [ ] **Incident response**: Plan tested within last 12 months; contact lists current; communication templates ready.

### Post-Audit Activities

- [ ] Draft audit report with findings, nonconformities, and observations.
- [ ] Classify findings: Major nonconformity, Minor nonconformity, Observation, Opportunity for improvement.
- [ ] Present findings to auditees and management.
- [ ] Agree on corrective action plans with owners and deadlines.
- [ ] Schedule follow-up audit to verify corrective actions.
- [ ] Update risk register with any new risks identified.

---

## Continuous Improvement (PDCA Cycle)

The ISMS follows the **Plan-Do-Check-Act (PDCA)** cycle for continuous improvement.

### Plan

| Activity | Frequency | Owner | Output |
|----------|-----------|-------|--------|
| Risk assessment | Annually (or after significant change) | CISO / Security Lead | Updated risk register |
| Security objectives | Annually | Management | Measurable objectives with targets |
| Treatment plan | After risk assessment | Risk owners | Action plans with timelines |
| Policy review | Annually | CISO | Updated policies |
| Training plan | Annually | HR + Security | Training calendar |

### Do

| Activity | Frequency | Owner | Output |
|----------|-----------|-------|--------|
| Implement risk treatments | Per treatment plan | Risk owners | Completed treatments |
| Security awareness training | Annually + on hire | Security Team | Training completion records |
| Vulnerability scanning | Every commit + daily | DevSecOps | Scan reports |
| Incident response drills | Semi-annually | Security Team | Drill reports |
| Vendor security reviews | Annually | Security Team | Vendor assessment reports |

### Check

| Activity | Frequency | Owner | Output |
|----------|-----------|-------|--------|
| Internal audit | Annually | Internal Audit | Audit report with findings |
| Management review | Annually | Management | Meeting minutes with decisions |
| KPI monitoring | Monthly | CISO | Security metrics dashboard |
| Compliance checks | Quarterly | Compliance Team | Compliance status report |
| Penetration testing | Annually | External assessor | Penetration test report |

### Act

| Activity | Trigger | Owner | Output |
|----------|---------|-------|--------|
| Corrective actions | Audit finding or incident | Finding owner | Corrective action record |
| Process improvements | Audit observations | Process owner | Updated procedures |
| Risk register update | New threat or incident | CISO | Updated risk register |
| Policy updates | Regulatory change or finding | CISO | Updated policies |
| Architecture improvements | Technology review | Engineering Lead | Architecture decision records |

### KPI Metrics

| Metric | Target | Measurement Frequency |
|--------|--------|----------------------|
| Mean time to detect (MTTD) security incidents | < 1 hour | Monthly |
| Mean time to respond (MTTR) to security incidents | < 4 hours | Monthly |
| Critical vulnerability remediation time | < 7 days | Monthly |
| Security training completion rate | 100% | Quarterly |
| Access review completion rate | 100% within 30 days of schedule | Quarterly |
| Overdue risk treatments | 0 | Monthly |
| Audit nonconformities open past deadline | 0 | Monthly |
| Backup recovery success rate | 100% | Quarterly |
| Patch compliance rate (critical patches) | > 95% within SLA | Monthly |
| Phishing simulation click rate | < 5% | Quarterly |

---

## Key Controls Mapping to SuperAdmin

### A.5 — Information Security Policies (A.5.1)

| Requirement | SuperAdmin Feature | Evidence |
|-------------|-------------------|----------|
| Policies shall be defined, approved by management, published, and communicated | Security documentation suite: SECURITY.md, SUPER_ADMIN_DOCUMENTATION.md, COMPLIANCE_FRAMEWORKS.md | Version-controlled markdown files; policy acknowledgment tracking in the platform |
| Policies shall be reviewed at planned intervals or when significant changes occur | Policy versioning in the compliance module; review schedule tracked in the admin panel | Audit log of policy changes; review dates in the compliance dashboard |

### A.5.2 — Organization of Information Security

| Requirement | SuperAdmin Feature | Evidence |
|-------------|-------------------|----------|
| Roles and responsibilities clearly defined | 7-level RBAC hierarchy (Super Admin, Admin, Manager, Team Lead, Member, Viewer, Guest) | 03_rbac_roles.csv; 07_permissions.csv |
| Segregation of duties | RBAC prevents: same user creating and approving changes; same user managing and auditing access | RBAC configuration; audit logs showing separate actors |

### A.5.15–A.5.18 — Access Control

| Requirement | SuperAdmin Feature | Evidence |
|-------------|-------------------|----------|
| Access control policy | RBAC + ABAC + RLS documented and enforced | SUPER_ADMIN_DOCUMENTATION.md Sections 3, 6, 8 |
| Identity management | Centralized identity; unique user IDs; lifecycle management | User management module; provisioning/deprovisioning logs |
| Authentication information | bcrypt (cost 12); MFA enforced for admins; password policy | 12_password_policy.csv; MFA configuration |
| Access rights provisioning | Least privilege; role-based assignment; quarterly reviews | Permission matrix; access review records |

### A.7 — Human Resource Security (A.6.1–A.6.8)

| Requirement | SuperAdmin Feature | Evidence |
|-------------|-------------------|----------|
| Security awareness training | Training tracking module; completion dashboards | Training completion records; quiz scores |
| Information security event reporting | In-app incident reporting; dedicated security channel | Incident tickets; response times |
| Remote working | VPN enforcement; encrypted endpoints; session management | VPN logs; endpoint compliance reports |

### A.8 — Asset Management (A.5.9–A.5.13)

| Requirement | SuperAdmin Feature | Evidence |
|-------------|-------------------|----------|
| Inventory of information assets | Data catalog with classification levels; infrastructure inventory via Terraform | Data catalog entries; Terraform state |
| Classification of information | 4-level classification scheme; automated tagging | Data classification labels; field-level tags |
| Handling of assets | Classification-based handling rules enforced by RLS and encryption | Access control policies; encryption configuration |

### A.9 — Access Control (A.5.15–A.5.18, A.8.2–A.8.5)

| Requirement | SuperAdmin Feature | Evidence |
|-------------|-------------------|----------|
| Access control policy | Documented in SUPER_ADMIN_DOCUMENTATION.md | Policy document version history |
| User registration and de-registration | Automated provisioning via SCIM; deprovisioning within 4 hours of termination | Provisioning logs; offboarding audit trail |
| User access provisioning | Role assignment workflow with approval | Approval records in audit log |
| Privileged access management | Super Admin role restricted; JIT access; MFA enforced; all actions logged | Privileged access logs; MFA records |
| Secure authentication | bcrypt; MFA (TOTP + WebAuthn); session management; account lockout | Authentication configuration; failed login reports |
| Access review | Quarterly automated access review reports | Access review reports; remediation records |

---

*This document is reviewed annually as part of the ISMS management review cycle. Last review: Q1 2026.*
