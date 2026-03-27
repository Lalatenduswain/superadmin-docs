# Compliance Documents, Standards, Frameworks & Certifications

Major **compliance documents, standards, frameworks, and certifications** reference:

1. **ISO 27001** International standard for an Information Security Management System (ISMS).  
2. **SOC 2** Attestation report for service organizations based on Trust Services Criteria (Security, Availability, Confidentiality, Processing Integrity, Privacy).  
3. **PCI DSS** Payment Card Industry Data Security Standard for organizations handling credit card data.  
4. **HIPAA** Health Insurance Portability and Accountability Act (U.S.) for protecting Protected Health Information (PHI).  
5. **GDPR** General Data Protection Regulation (EU) for personal data protection and privacy.  
6. **FedRAMP** Federal Risk and Authorization Management Program for cloud services used by U.S. federal agencies.  
7. **NIST Cybersecurity Framework (CSF)** Voluntary framework for managing cybersecurity risk (widely used as a baseline).  
8. **CMMC** Cybersecurity Maturity Model Certification (U.S. DoD requirement for contractors handling controlled unclassified information).  
9. **HITRUST CSF** Comprehensive certifiable framework (often used in healthcare, mapping to multiple standards like HIPAA, NIST, ISO).  
10. **NIST SP 800-53** Detailed security and privacy controls for federal information systems (basis for many U.S. government requirements).  
11. **ISO 27701** Privacy Information Management System extension to ISO 27001\.  
12. **SOC 1** Focused on controls relevant to financial reporting.  
13. **CIS Controls** Prioritized set of actionable security controls (practical implementation guide).  
14. **CCPA / CPRA** California Consumer Privacy Act (and amendments) for California resident data privacy.  
15. **DPDP Act** Digital Personal Data Protection Act (India) for personal data processing and privacy.

---

## ISO/IEC 27000 Family of Standards

## The ISO/IEC 27000 series covers various aspects of information security management. Standards can be broadly grouped into four categories: Overview & Vocabulary, Requirements (Normative), General Guidelines, and Sector-Specific Guidelines.

## 

### Foundation

| Standard | Purpose |
| ----- | ----- |
| ISO/IEC 27000 | Vocabulary, overview, and key terms for the entire family |
| ISO/IEC 27001 | Core ISMS requirements — the only certifiable standard |

### Core Supporting Standards (27002–27008)

## ISO 27002 defines a set of good practices for ISMS implementation through 93 controls, structured in 4 major domains. ISO 27003 provides guidance for the correct implementation of an ISMS, focusing on important aspects for successfully carrying out this process. ISO 27004 provides guidelines for defining metrics to evaluate ISMS performance. ISO 27005 defines how to perform risk management for information security, focusing on methodology. ISO 27006 establishes requirements for organizations that want to be accredited to certify others in compliance with ISO 27001\.

## ISO/IEC 27007 focuses on auditing the management system elements of an ISMS, while ISO/IEC TS 27008 focuses on technical checks on information security controls being managed using an ISMS.

### Normative / Requirements Standards

## ISO 27009 specifies requirements for creating sector-specific standards that extend ISO 27001 and complement ISO 27002\. ISO 27701 is for privacy information management — it extends ISO 27001 and ISO 27002 for privacy management and specifies requirements for establishing a Privacy Information Management System (PIMS), applicable to all PII controllers and processors.

### Sector-Specific Standards

| Standard | Sector |
| ----- | ----- |
| ISO 27011 | Telecommunications |
| ISO 27017 | Cloud services (security controls) |
| ISO 27018 | Cloud-based PII / privacy |
| ISO 27019 | Energy/utilities sector |
| ISO 27799 | Healthcare |

## ISO/IEC 27010 provides guidance on information security management for inter-sector and inter-organizational communications, particularly for critical infrastructure. ISO/IEC 27011 is an ISMS implementation guide specifically for the telecom industry.

### Governance, Integration & Other

## ISO 27013 establishes a guide for the integration of ISO 27001 (ISMS) and ISO/IEC 20000 (IT Service Management) in organizations implementing both. ISO 27014 establishes principles for the governance of information security.

## ISO/IEC 27031 provides guidelines for ICT readiness for business continuity. ISO/IEC 27032 covers Internet security and network security controls.

### Privacy Extension

## ISO/IEC 27701 is especially relevant for you given CITPL's compliance scope — it extends ISO 27001 into a full Privacy Information Management System (PIMS), useful when dealing with GDPR-equivalent mandates or client data processing agreements.

In short, ISO 27001 is the certifiable core, while the rest of the family provides guidance, sector specifics, and extensions to build a complete information security governance program.

---

## **Global / International Frameworks**

### **1\. NIST CSF (Cybersecurity Framework)**

The NIST 800-53 framework covers five areas of security: **Identify, Protect, Detect, Respond, and Recover** — to help organizations proactively manage and mitigate risks. Unlike ISO 27001, NIST CSF has no certification or audit process — it's a guide organizations use to establish their cybersecurity posture, with no external proof-points required.

📌 Updated to **CSF 2.0 in 2024**, adding a "Govern" function and supply chain security guidance.

### **2\. SOC 2 (Service Organization Control 2\)**

SOC 2 assesses whether an organization's controls are designed and operated effectively according to five Trust Services Criteria: **security, availability, processing integrity, confidentiality, and privacy**. A Type I report evaluates design of controls at a point in time, while a Type II report assesses how well those controls perform over a sustained period — typically six to twelve months.

📌 Preferred by **US-based** clients; ISO 27001 is preferred **internationally**.

### **3\. COBIT (Control Objectives for Information and Related Technology)**

Developed by ISACA, COBIT is a comprehensive framework designed to help organizations manage their IT resources effectively. It is divided into five categories: Plan & Organize, Acquire & Implement, Deliver & Support, Monitor & Evaluate, and Manage & Assess. It is also the most used framework to achieve **SOX compliance**.

### **4\. CIS Controls (Center for Internet Security)**

The CIS Critical Security Controls list technical security and operational controls applicable to any environment. It does not address risk analysis or risk management like NIST CSF — rather, it solely focuses on **reducing risk and increasing resilience** for technical infrastructures. It was updated in 2024 to align with NIST CSF 2.0.

## **Sector-Specific Frameworks**

### **5\. HIPAA (Healthcare)**

HIPAA is a cybersecurity framework requiring healthcare organizations to implement controls for securing and protecting the privacy of electronic health information. Organizations must also conduct risk assessments to manage and identify emerging risks.

### **6\. PCI-DSS (Payment / Finance)**

PCI-DSS version 4.0 became mandatory on March 31, 2024, now requiring the use of **multi-factor authentication**. It applies to any organization that handles credit/debit card data.

### **7\. NERC-CIP (Energy / Utilities)**

NERC-CIP stipulates controls including categorizing systems and critical assets, training personnel, incident response and planning, recovery plans for critical cyber assets, vulnerability assessments, and more.

### **8\. HITRUST CSF (Healthcare \+ Multi-framework)**

HITRUST consolidates multiple compliance requirements into one framework, making it popular for organizations that need to satisfy HIPAA, PCI-DSS, and ISO simultaneously.

## **Privacy-Focused Frameworks**

### **9\. GDPR (EU Data Protection)**

GDPR was adopted to strengthen data protection procedures and practices for citizens of the European Union. It's a regulation, not just a framework — non-compliance carries heavy financial penalties.

### **10\. CMMC (Defense Contractors)**

CMMC 2.0 was finalized in 2024 and applies to US Department of Defense contractors handling controlled unclassified information (CUI).

### **11\. FISMA (US Federal Government)**

FISMA requires US federal agencies and third-party contractors handling federal systems to develop, document, and implement security programs, including continuous monitoring, annual security reviews, and baseline security controls.

## **Quick Comparison Table**

| Framework | Certifiable | Origin | Best For |
| ----- | ----- | ----- | ----- |
| ISO 27001 | ✅ Yes | International | Global ISMS |
| NIST CSF | ❌ No | USA | US orgs, risk maturity |
| SOC 2 | ✅ (Audit report) | USA | Cloud/SaaS vendors |
| COBIT | ❌ No | International | IT governance, SOX |
| CIS Controls | ❌ No | USA | Technical hardening |
| HIPAA | ✅ (Regulated) | USA | Healthcare |
| PCI-DSS | ✅ Yes | International | Payment card data |
| GDPR | ✅ (Regulated) | EU | Data privacy |
| HITRUST | ✅ Yes | USA | Multi-framework consolidation |
| CMMC | ✅ Yes | USA | Defense contractors |

