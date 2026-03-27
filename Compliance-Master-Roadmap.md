# Compliance Master Roadmap

This document provides a unified roadmap for achieving compliance across all target frameworks: **ISO 27001, SOC 2, GDPR, PCI DSS, SOX, and HIPAA**. It covers prioritization, timelines, resource requirements, cross-framework synergies, maturity levels, and budget planning.

---

## Table of Contents

1. [Priority Order and Rationale](#priority-order-and-rationale)
2. [Timeline View (Q1–Q4)](#timeline-view-q1q4)
3. [Resource Requirements](#resource-requirements)
4. [Cross-Framework Synergies](#cross-framework-synergies)
5. [Compliance Maturity Model](#compliance-maturity-model)
6. [Audit Preparation Checklist](#audit-preparation-checklist)
7. [Third-Party Assessor Selection](#third-party-assessor-selection)
8. [Budget Estimation Template](#budget-estimation-template)

---

## Priority Order and Rationale

| Priority | Framework | Rationale | Target Completion |
|----------|-----------|-----------|-------------------|
| 1 | **ISO 27001** | Foundation framework; establishes the ISMS that all other frameworks build upon; most requested by enterprise customers; internationally recognized | Q2 2026 |
| 2 | **SOC 2 Type I** | High customer demand for SaaS providers; builds directly on ISO 27001 controls; Type I (point-in-time) achievable quickly after ISMS is established | Q3 2026 |
| 3 | **GDPR** | Legal requirement for EU operations and EU customer data; significant regulatory fines for non-compliance; many controls overlap with ISO 27001 | Q3 2026 |
| 4 | **PCI DSS** | Required for payment processing; scope is reduced by using Stripe (SAQ A); builds on access control and logging controls from ISO 27001 | Q4 2026 |
| 5 | **SOX** | Required when the organization or its customers are publicly traded; IT controls leverage existing ITGC framework from ISO 27001 and SOC 2 | Q1 2027 |
| 6 | **HIPAA** | Required only if processing Protected Health Information (PHI); significant overlap with existing controls; addressed last unless customer contracts require earlier | Q2 2027 |
| — | **SOC 2 Type II** | Requires 6–12 months of operating evidence after Type I; runs in parallel with other frameworks | Q2 2027 |

---

## Timeline View (Q1–Q4)

### Year 1 (2026)

#### Q1 2026: Foundation and ISO 27001 Preparation

| Week | Activity | Deliverable |
|------|----------|-------------|
| 1–2 | Gap analysis against ISO 27001:2022 | Gap analysis report |
| 3–4 | Define ISMS scope, context, and interested parties | ISMS scope document |
| 5–6 | Risk assessment (asset inventory, threat identification, vulnerability assessment) | Risk register |
| 7–8 | Risk treatment plan and Statement of Applicability (SoA) | SoA; risk treatment plan |
| 9–10 | Policy development (information security policy, access control, acceptable use, incident response, etc.) | Policy suite (12+ policies) |
| 11–12 | Implement priority controls (access management, logging, encryption, change management) | Control implementation evidence |
| 13 | Internal audit | Internal audit report |

#### Q2 2026: ISO 27001 Certification and SOC 2 Preparation

| Week | Activity | Deliverable |
|------|----------|-------------|
| 14–15 | Remediate internal audit findings | Corrective action records |
| 16 | Management review | Management review minutes |
| 17–19 | ISO 27001 Stage 1 audit (documentation review) | Stage 1 audit report |
| 20–22 | Address Stage 1 findings; operate controls | Remediation evidence |
| 23–25 | ISO 27001 Stage 2 audit (implementation assessment) | **ISO 27001 Certificate** |
| 26 | Begin SOC 2 readiness: map Trust Services Criteria to existing controls | TSC mapping document |

#### Q3 2026: SOC 2 Type I and GDPR Compliance

| Week | Activity | Deliverable |
|------|----------|-------------|
| 27–28 | SOC 2 gap analysis; identify additional controls needed beyond ISO 27001 | SOC 2 gap report |
| 29–30 | Implement SOC 2-specific controls (availability, processing integrity, privacy criteria) | Control evidence |
| 31–32 | SOC 2 Type I examination (point-in-time assessment) | **SOC 2 Type I Report** |
| 33 | Begin SOC 2 Type II observation period (6–12 months) | Monitoring plan |
| 34–35 | GDPR readiness: complete ROPA, DPIAs, consent management, DSR workflows | ROPA; DPIA register; consent management system |
| 36–37 | Implement data subject rights automation (access, erasure, portability) | DSR workflow system |
| 38–39 | DPA template finalization; sub-processor documentation; cross-border transfer assessments | DPA template; TIA reports; **GDPR compliance declaration** |

#### Q4 2026: PCI DSS Compliance

| Week | Activity | Deliverable |
|------|----------|-------------|
| 40–41 | PCI DSS scope definition; validate SAQ A eligibility | Scope document |
| 42–43 | Implement PCI-specific controls (payment page security, network segmentation, ASV scanning) | Control evidence |
| 44–45 | Complete SAQ A questionnaire | SAQ A questionnaire |
| 46 | Quarterly ASV scan (first scan) | ASV scan report |
| 47–48 | Submit Attestation of Compliance (AOC) | **PCI DSS AOC** |
| 49–52 | Continuous improvement: address findings from all frameworks; refine monitoring | Updated risk register; improvement log |

### Year 2 (2027)

#### Q1 2027: SOX Compliance

| Week | Activity | Deliverable |
|------|----------|-------------|
| 1–3 | SOX scoping: identify financially significant systems and processes | SOX scope document |
| 4–6 | ITGC assessment: evaluate access, change, operations, SDLC controls | ITGC assessment report |
| 7–9 | Implement SOX-specific controls (segregation of duties matrix, financial reconciliation, enhanced audit trail retention) | SOX control evidence |
| 10–12 | Control testing: design and operating effectiveness | Control testing results |
| 13 | SOX readiness assessment | **SOX readiness report** |

#### Q2 2027: HIPAA and SOC 2 Type II

| Week | Activity | Deliverable |
|------|----------|-------------|
| 14–16 | HIPAA gap analysis (if PHI processing is in scope) | HIPAA gap report |
| 17–19 | Implement HIPAA-specific controls (BAAs, PHI access controls, minimum necessary standard) | BAA template; PHI handling procedures |
| 20–22 | HIPAA risk analysis and management | Risk analysis; **HIPAA compliance documentation** |
| 23–26 | SOC 2 Type II examination (6+ months of operating evidence) | **SOC 2 Type II Report** |

---

## Resource Requirements

### Team Roles

| Role | FTE Allocation | Duration | Responsibilities |
|------|---------------|----------|-----------------|
| **CISO / Security Lead** | 1.0 FTE | Permanent | ISMS ownership; risk management; policy approval; management liaison |
| **Compliance Manager** | 1.0 FTE | Permanent | Framework mapping; audit coordination; evidence collection; vendor management |
| **Security Engineer** | 0.5 FTE | Permanent | Control implementation; security tooling; vulnerability management |
| **DevOps Engineer** | 0.5 FTE | Permanent | IaC; monitoring; logging infrastructure; backup/recovery |
| **DPO (Data Protection Officer)** | 0.5 FTE | Permanent | GDPR compliance; DPIA; DSR management; privacy advisory |
| **Internal Auditor** | 0.25 FTE | Q1–Q2 (then annual) | Internal audit execution; findings tracking |
| **Legal Counsel** | As needed | Ongoing | Contract review; DPA drafting; regulatory interpretation |
| **Project Manager** | 0.5 FTE | 12 months | Roadmap tracking; milestone management; reporting |

### External Resources

| Resource | Engagement | Estimated Duration |
|----------|-----------|-------------------|
| ISO 27001 certification body | Stage 1 + Stage 2 audit | 4–6 weeks |
| SOC 2 CPA firm | Type I + Type II examination | Type I: 2–4 weeks; Type II: 3–6 months observation |
| PCI QSA / ASV | SAQ validation + quarterly scans | Ongoing quarterly |
| Penetration testing firm | Annual penetration test | 2–3 weeks |
| GDPR legal advisor | DPA review; cross-border transfer assessment | 4–6 weeks |
| SOX auditor (external) | ITGC assessment (if required by external audit firm) | 4–6 weeks |

---

## Cross-Framework Synergies

Many controls satisfy requirements across multiple frameworks. Implementing a control once provides evidence for multiple certifications.

### Control Synergy Matrix

| Control Area | ISO 27001 | SOC 2 | GDPR | PCI DSS | SOX | HIPAA |
|-------------|-----------|-------|------|---------|-----|-------|
| **Access control (RBAC)** | A.5.15–A.5.18; A.8.2–A.8.5 | CC6.1–CC6.3 | Art. 5(1)(f); Art. 32 | Req 7, 8 | ITGC Access | 164.312(a) |
| **Encryption at rest** | A.8.24 | CC6.1, CC6.7 | Art. 32(1)(a) | Req 3 | — | 164.312(a)(2)(iv) |
| **Encryption in transit** | A.8.24; A.5.14 | CC6.1, CC6.7 | Art. 32(1)(a) | Req 4 | — | 164.312(e)(1) |
| **Audit logging** | A.8.15 | CC7.2 | Art. 5(2) | Req 10 | ITGC Operations; Sec 802 | 164.312(b) |
| **Vulnerability management** | A.8.8 | CC7.1 | Art. 32(1)(d) | Req 6, 11 | ITGC SDLC | 164.308(a)(1) |
| **Incident response** | A.5.24–A.5.28 | CC7.3–CC7.5 | Art. 33, 34 | Req 12.10 | ITGC Operations | 164.308(a)(6) |
| **Change management** | A.8.32 | CC8.1 | — | Req 6.5 | ITGC Change Mgmt | 164.308(a)(1)(ii)(A) |
| **Backup and recovery** | A.8.13; A.5.30 | A1.2 | Art. 32(1)(c) | Req 12.10 | ITGC Operations | 164.308(a)(7) |
| **Security awareness training** | A.6.3 | CC1.4 | Art. 39(1)(b) | Req 12.6 | COSO Control Environment | 164.308(a)(5) |
| **Risk assessment** | Clause 6.1 | CC3.1–CC3.4 | Art. 35 (DPIA) | Req 12.3 | COSO Risk Assessment | 164.308(a)(1)(ii)(A) |
| **MFA** | A.8.5 | CC6.1 | Art. 32(1)(b) | Req 8.4 | ITGC Access | 164.312(d) |
| **Data retention/deletion** | A.8.10 | P6.1 | Art. 5(1)(e); Art. 17 | Req 3.1 | Sec 802 | 164.530(j) |
| **Segregation of duties** | A.5.3 | CC6.1 | — | Req 7 | ITGC Access; COSO | 164.312(a)(1) |
| **Vendor management** | A.5.19–A.5.22 | CC9.2 | Art. 28 (DPA) | Req 12.8 | COSO Control Activities | 164.308(b)(1) |

### Efficiency Gains

By implementing controls in the recommended priority order (ISO 27001 first), approximately **65–75%** of controls for subsequent frameworks are already satisfied:

| Framework | % Controls Already Met After ISO 27001 | Additional Effort |
|-----------|----------------------------------------|-------------------|
| SOC 2 | ~70% | Availability criteria; privacy criteria; formal reporting |
| GDPR | ~60% | Consent management; DSR workflows; DPA; DPIA; ROPA |
| PCI DSS (SAQ A) | ~75% | Payment-specific controls; ASV scanning |
| SOX | ~65% | Segregation of duties; financial reconciliation; 7-year retention |
| HIPAA | ~60% | BAAs; PHI-specific access controls; minimum necessary standard |

---

## Compliance Maturity Model

### Maturity Levels

| Level | Name | Description | Characteristics |
|-------|------|-------------|----------------|
| **1** | Initial | Ad hoc, reactive | No formal policies; security depends on individual effort; no documentation; no risk assessment |
| **2** | Developing | Policies defined, partially implemented | Security policies written but not fully enforced; some controls in place; risk awareness exists but not formalized |
| **3** | Defined | Standardized processes, consistently applied | ISMS established; controls documented and operating; risk assessment completed; staff trained; audit trail in place |
| **4** | Managed | Measured and monitored | KPIs tracked; control effectiveness measured; continuous monitoring active; regular audits; quantitative risk management |
| **5** | Optimized | Continuously improving | Automated compliance monitoring; predictive risk analytics; industry-leading practices; proactive threat hunting; compliance as code |

### Current State Assessment

| Control Area | Current Level | Target Level (12 months) | Target Level (24 months) |
|-------------|--------------|--------------------------|--------------------------|
| Access control | 3 — Defined | 4 — Managed | 5 — Optimized |
| Encryption | 3 — Defined | 4 — Managed | 4 — Managed |
| Audit logging | 3 — Defined | 4 — Managed | 5 — Optimized |
| Vulnerability management | 3 — Defined | 4 — Managed | 4 — Managed |
| Incident response | 2 — Developing | 3 — Defined | 4 — Managed |
| Change management | 3 — Defined | 4 — Managed | 4 — Managed |
| Risk management | 2 — Developing | 3 — Defined | 4 — Managed |
| Data protection (GDPR) | 2 — Developing | 3 — Defined | 4 — Managed |
| Security awareness | 2 — Developing | 3 — Defined | 4 — Managed |
| Vendor management | 2 — Developing | 3 — Defined | 3 — Defined |
| Business continuity | 3 — Defined | 3 — Defined | 4 — Managed |
| Monitoring and detection | 3 — Defined | 4 — Managed | 5 — Optimized |

### Maturity Advancement Activities

**Level 2 to Level 3 (Developing to Defined):**

- Formalize all security policies and get management approval.
- Complete risk assessment and risk treatment plan.
- Implement all high-priority controls from the SoA.
- Deploy comprehensive audit logging.
- Conduct first internal audit.
- Complete security awareness training for all staff.

**Level 3 to Level 4 (Defined to Managed):**

- Establish security KPIs and measure monthly.
- Implement continuous vulnerability scanning (every commit).
- Deploy automated compliance monitoring dashboards.
- Conduct regular penetration testing (annual external, semi-annual internal).
- Implement automated access reviews.
- Achieve ISO 27001 certification and SOC 2 Type II.

**Level 4 to Level 5 (Managed to Optimized):**

- Implement compliance-as-code (automated evidence collection and control verification).
- Deploy AI/ML-based anomaly detection and threat hunting.
- Achieve proactive risk management with predictive analytics.
- Integrate security metrics into business decision-making.
- Contribute to industry standards and best practices.
- Automate incident response for common threat patterns.

---

## Audit Preparation Checklist

### 8 Weeks Before Audit

- [ ] Confirm audit dates with the certification body / assessor.
- [ ] Assign an audit liaison to coordinate logistics and evidence collection.
- [ ] Review scope document and ensure it is current.
- [ ] Update all policies and procedures (ensure review dates are within the last 12 months).
- [ ] Verify all corrective actions from previous audits are closed.
- [ ] Conduct a pre-audit self-assessment using the internal audit checklist.

### 6 Weeks Before Audit

- [ ] Collect and organize evidence for all in-scope controls.
- [ ] Verify audit log integrity and availability for the review period.
- [ ] Run vulnerability scans and remediate any outstanding critical/high findings.
- [ ] Confirm all staff have completed security awareness training.
- [ ] Test backup restoration procedures.
- [ ] Verify incident response plan is current and on-call roster is accurate.

### 4 Weeks Before Audit

- [ ] Complete internal audit (if not done within the last 6 months).
- [ ] Conduct management review (if not done within the last 6 months).
- [ ] Brief all staff who may be interviewed by the auditor on the audit process and expectations.
- [ ] Prepare a dedicated workspace or virtual meeting room for the auditor.
- [ ] Stage evidence in a shared repository (organized by control / requirement number).
- [ ] Verify all access reviews are complete and documented.

### 2 Weeks Before Audit

- [ ] Final review of evidence completeness — check each control has at least one evidence artifact.
- [ ] Confirm auditor access to systems (read-only auditor account, demo environment).
- [ ] Prepare opening meeting presentation (scope, team, key contacts, schedule).
- [ ] Verify no outstanding critical security incidents.
- [ ] Test all compliance dashboards and reporting tools.

### During Audit

- [ ] Attend opening meeting; confirm scope, schedule, and communication channels.
- [ ] Designate a single point of contact for the audit liaison role.
- [ ] Respond to evidence requests within 4 hours.
- [ ] Document any observations or preliminary findings communicated by the auditor.
- [ ] Conduct daily debrief with internal team.
- [ ] Attend closing meeting; note any nonconformities.

### After Audit

- [ ] Receive and review the audit report.
- [ ] Create corrective action plans for any nonconformities (within 30 days).
- [ ] Submit corrective action evidence to the certification body (if required).
- [ ] Update risk register with any new risks identified during the audit.
- [ ] Conduct internal lessons-learned session.
- [ ] Communicate results to management and relevant stakeholders.

---

## Third-Party Assessor Selection

### Selection Criteria

| Criterion | Weight | Evaluation Questions |
|-----------|--------|---------------------|
| **Accreditation** | 25% | Is the assessor accredited by an appropriate body? (e.g., ANAB for ISO 27001; AICPA for SOC 2; PCI SSC for QSA) |
| **Industry experience** | 20% | Has the assessor audited SaaS / cloud-native / multi-tenant platforms? How many similar engagements in the last 3 years? |
| **Technical expertise** | 20% | Does the audit team understand modern tech stacks (containerization, K8s, IaC, CI/CD, PostgreSQL, Node.js/TypeScript)? |
| **Reputation and references** | 15% | Can the assessor provide references from similar companies? What is their reputation in the market? |
| **Cost and timeline** | 10% | Is the pricing competitive? Can they meet our timeline? What is the total cost (Stage 1 + Stage 2 for ISO; Type I + Type II for SOC 2)? |
| **Communication and support** | 10% | Do they provide pre-audit guidance? How responsive are they to questions? Do they offer remediation support? |

### Evaluation Scorecard

| Assessor | Accreditation (25) | Experience (20) | Technical (20) | Reputation (15) | Cost (10) | Communication (10) | Total (100) |
|----------|-------------------|-----------------|----------------|-----------------|-----------|--------------------|-----------  |
| Assessor A | | | | | | | |
| Assessor B | | | | | | | |
| Assessor C | | | | | | | |

### Red Flags

- Assessor guarantees certification before the audit.
- Assessor offers consulting and audit services for the same engagement (independence conflict).
- Audit team lacks experience with cloud-native architectures.
- No references from SaaS or technology companies.
- Significantly below-market pricing (may indicate inexperience or superficial assessment).

---

## Budget Estimation Template

### Year 1 Budget (2026)

| Category | Item | Estimated Cost | Notes |
|----------|------|---------------|-------|
| **Personnel** | | | |
| | CISO / Security Lead (1.0 FTE) | $150,000–$200,000 | Full-time hire or fractional CISO |
| | Compliance Manager (1.0 FTE) | $100,000–$140,000 | Full-time hire |
| | Security Engineer (0.5 FTE allocation) | $60,000–$80,000 | Existing team; partial allocation |
| | DevOps Engineer (0.5 FTE allocation) | $55,000–$75,000 | Existing team; partial allocation |
| | DPO (0.5 FTE) | $50,000–$70,000 | Part-time or shared role |
| | **Personnel subtotal** | **$415,000–$565,000** | |
| **Certification / Audit** | | | |
| | ISO 27001 Stage 1 + Stage 2 | $15,000–$30,000 | Varies by scope and assessor |
| | SOC 2 Type I examination | $20,000–$40,000 | CPA firm engagement |
| | PCI DSS ASV scanning (4 quarters) | $4,000–$8,000 | Quarterly scans |
| | Penetration testing (external) | $15,000–$30,000 | Annual comprehensive test |
| | **Certification subtotal** | **$54,000–$108,000** | |
| **Tooling** | | | |
| | GRC platform (Vanta, Drata, or Sprinto) | $15,000–$30,000/year | Automated evidence collection; compliance monitoring |
| | SonarQube (Enterprise) | $10,000–$20,000/year | SAST; quality gates |
| | Trivy (Enterprise / Aqua Security) | $5,000–$15,000/year | Container scanning; SBOM |
| | SIEM (ELK Cloud / Datadog Security) | $12,000–$36,000/year | Log aggregation; threat detection |
| | Secrets management (HashiCorp Vault) | $5,000–$15,000/year | Key management; secrets rotation |
| | **Tooling subtotal** | **$47,000–$116,000** | |
| **Training** | | | |
| | Security awareness training platform | $3,000–$8,000/year | All-staff training; phishing simulations |
| | Specialized training (ISO LA, CISSP, etc.) | $5,000–$15,000 | Certifications for security team members |
| | **Training subtotal** | **$8,000–$23,000** | |
| **Legal** | | | |
| | GDPR legal advisory | $10,000–$25,000 | DPA review; TIA; regulatory guidance |
| | Contract review (DPAs, BAAs) | $5,000–$10,000 | Legal counsel for agreements |
| | **Legal subtotal** | **$15,000–$35,000** | |
| **Contingency** | 10% of total | $54,000–$85,000 | Unexpected findings; remediation costs |
| | | | |
| **Year 1 Total** | | **$593,000–$932,000** | |

### Year 2 Budget (2027)

| Category | Estimated Cost | Notes |
|----------|---------------|-------|
| Personnel (ongoing) | $415,000–$565,000 | Same team; potential for reduced DPO/PM allocation |
| ISO 27001 surveillance audit | $8,000–$15,000 | Annual surveillance |
| SOC 2 Type II examination | $30,000–$50,000 | 6–12 month observation period |
| PCI DSS ASV scanning (4 quarters) | $4,000–$8,000 | Ongoing quarterly |
| Penetration testing | $15,000–$30,000 | Annual |
| HIPAA readiness (if applicable) | $20,000–$40,000 | Gap analysis + implementation |
| Tooling renewals | $47,000–$116,000 | Annual renewals |
| Training | $8,000–$23,000 | Annual refresher + new certifications |
| Legal | $5,000–$15,000 | Contract maintenance |
| Contingency (10%) | $55,000–$86,000 | |
| **Year 2 Total** | **$607,000–$948,000** | |

### Cost Optimization Strategies

1. **Leverage cross-framework synergies**: Implementing ISO 27001 first reduces the marginal cost of each subsequent framework by 60–75%.
2. **Use a GRC platform**: Automated evidence collection reduces compliance manager workload by an estimated 40%.
3. **Combine audits**: Schedule ISO 27001 surveillance and SOC 2 Type II audits in the same period to reduce auditor overlap.
4. **Fractional CISO**: If a full-time CISO is not justified by company size, engage a fractional CISO (0.25–0.5 FTE) at 30–50% of full-time cost.
5. **Open-source tooling**: Use open-source alternatives where appropriate (Trivy OSS, SonarQube Community, ELK self-hosted) to reduce tooling costs.

---

*This roadmap is a living document reviewed quarterly by the CISO and Compliance Manager. Timelines are adjusted based on organizational priorities, customer requirements, and audit findings. Last review: Q1 2026.*
