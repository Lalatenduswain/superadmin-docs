# Security Policy

This document describes the security policy for the **SuperAdmin SaaS Platform**, including supported versions, vulnerability reporting procedures, security features, encryption standards, and compliance posture.

---

## Supported Versions

| Version | Supported | Notes |
|---------|-----------|-------|
| 3.x     | Yes       | Current release — receives all security patches |
| 2.x     | Yes       | LTS — critical and high-severity patches only until 2027-06-30 |
| 1.x     | No        | End-of-life — no patches; upgrade immediately |
| 0.x     | No        | Pre-release — never intended for production use |

Security patches are back-ported to all supported versions within the timelines defined below. Customers on unsupported versions must upgrade before reporting issues.

---

## Reporting Vulnerabilities

We follow a **responsible disclosure** process. If you discover a security vulnerability, please report it privately.

### How to Report

1. **Email**: Send a detailed report to **security@{DOMAIN}**.
2. **Subject line**: `[VULNERABILITY] Brief description`.
3. **Encrypt** your report using our PGP key (published at `https://{DOMAIN}/.well-known/security.txt`).
4. **Include**:
   - Description of the vulnerability and its impact.
   - Steps to reproduce (proof of concept if possible).
   - Affected version(s) and component(s).
   - Your suggested severity rating (Critical / High / Medium / Low).
   - Your name and affiliation (for credit, unless you prefer anonymity).

### What NOT to Do

- Do not open a public GitHub issue for security vulnerabilities.
- Do not access, modify, or delete data belonging to other tenants.
- Do not perform denial-of-service attacks against production systems.
- Do not use automated scanners against production without prior written authorization.

---

## Security Response Timeline

| Stage | SLA | Description |
|-------|-----|-------------|
| Acknowledgement | 24 hours | We confirm receipt of your report and assign a tracking ID. |
| Triage | 48 hours | We assess severity, confirm the vulnerability, and assign a priority. |
| Critical fix | 7 calendar days | Patches for CVSS 9.0+ vulnerabilities are developed, tested, and released. |
| High fix | 14 calendar days | Patches for CVSS 7.0–8.9 vulnerabilities. |
| Medium fix | 30 calendar days | Patches for CVSS 4.0–6.9 vulnerabilities. |
| Low fix | 90 calendar days | Patches for CVSS 0.1–3.9 vulnerabilities. |
| Disclosure | Coordinated | Public disclosure occurs after the fix is released and customers have had 14 days to apply it. |

We publish security advisories in our changelog and notify affected customers directly.

---

## Bug Bounty Scope

We operate a private bug bounty program. Eligible researchers who report qualifying vulnerabilities receive recognition and monetary rewards.

### In Scope

- Authentication and session management (MFA bypass, session fixation).
- Authorization flaws (RBAC bypass, privilege escalation, tenant isolation breach).
- Injection vulnerabilities (SQL injection, XSS, command injection).
- Sensitive data exposure (PII leakage, credential disclosure).
- API security (broken object-level authorization, mass assignment).
- Cryptographic weaknesses (weak algorithms, key exposure).

### Out of Scope

- Social engineering and phishing attacks.
- Denial-of-service (DoS/DDoS) attacks.
- Issues in third-party services not under our control.
- Vulnerabilities requiring physical access to infrastructure.
- Reports from automated scanners without manual verification.

### Reward Tiers

| Severity | CVSS Range | Reward Range |
|----------|------------|--------------|
| Critical | 9.0–10.0  | $2,000–$10,000 |
| High     | 7.0–8.9   | $500–$2,000 |
| Medium   | 4.0–6.9   | $100–$500 |
| Low      | 0.1–3.9   | Recognition + swag |

---

## Security Features Summary

The SuperAdmin platform implements defense-in-depth across every layer.

### Authentication

- **Multi-Factor Authentication (MFA)**: TOTP (RFC 6238) and WebAuthn/FIDO2 hardware keys.
- **Passwordless login**: Magic-link email authentication with one-time tokens.
- **Adaptive authentication**: Risk-based step-up authentication triggered by anomalous login patterns (new device, unusual location, impossible travel).
- **Account lockout**: Progressive delays after 5 failed attempts; full lockout after 10 within 15 minutes.
- **Session management**: Secure, HttpOnly, SameSite=Strict cookies; server-side session revocation; configurable idle and absolute timeouts.

### Authorization

- **Role-Based Access Control (RBAC)**: 7-level role hierarchy (Super Admin, Admin, Manager, Team Lead, Member, Viewer, Guest).
- **Row-Level Security (RLS)**: PostgreSQL RLS policies enforce tenant isolation at the database layer — every query is scoped to `tenant_id`.
- **Attribute-Based Access Control (ABAC)**: Fine-grained permissions based on resource attributes, user context, and environmental conditions.
- **70+ granular permissions**: Individually assignable permissions mapped to roles via a permission matrix.

### Data Protection

- **Encryption at rest**: AES-256-GCM for all stored data.
- **Encryption in transit**: TLS 1.3 (minimum TLS 1.2) for all communications.
- **Application-level encryption**: Sensitive fields (PII, API keys, secrets) are encrypted before storage using envelope encryption with AWS KMS / HashiCorp Vault.
- **Database**: Transparent Data Encryption (TDE) enabled on PostgreSQL.

### Audit and Monitoring

- **Comprehensive audit logging**: Every state-changing operation is logged with actor, action, target, timestamp, IP address, user agent, and tenant context.
- **Immutable audit trail**: Audit logs are append-only and cryptographically chained to detect tampering.
- **Real-time threat detection**: Anomaly detection for brute-force attempts, privilege escalation, data exfiltration patterns, and impossible travel.
- **SIEM integration**: Structured JSON logs forwarded to ELK Stack / Splunk / Datadog.

---

## Encryption Standards

### Symmetric Encryption

| Use Case | Algorithm | Key Size | Mode | Notes |
|----------|-----------|----------|------|-------|
| Data at rest | AES | 256-bit | GCM | Authenticated encryption with associated data (AEAD) |
| Database TDE | AES | 256-bit | XTS | Full-disk encryption for database volumes |
| Backup encryption | AES | 256-bit | GCM | All backups encrypted before transfer to cold storage |
| Field-level encryption | AES | 256-bit | GCM | Envelope encryption via KMS for PII fields |

### Asymmetric Encryption

| Use Case | Algorithm | Key Size | Notes |
|----------|-----------|----------|-------|
| TLS certificates | ECDSA | P-384 | RSA-4096 accepted for legacy clients |
| JWT signing | EdDSA | Ed25519 | RS256 supported for backward compatibility |
| API key signing | HMAC-SHA256 | 256-bit | Used for webhook signature verification |

### Password Hashing

| Algorithm | Work Factor | Notes |
|-----------|-------------|-------|
| bcrypt | Cost factor 12 | Primary password hashing algorithm |
| Argon2id | m=65536, t=3, p=4 | Planned migration target for memory-hard hashing |

### Post-Quantum Cryptography (ML-KEM-768)

| Component | Standard | Status |
|-----------|----------|--------|
| Key Encapsulation | ML-KEM-768 (FIPS 203) | Evaluation phase |
| Digital Signatures | ML-DSA-65 (FIPS 204) | Research phase |
| Hash-Based Signatures | SLH-DSA (FIPS 205) | Research phase |

See the **ML-KEM Post-Quantum Encryption Plan** section below for the full roadmap.

---

## Dependency Security

### Automated Scanning Pipeline

All dependencies are scanned at multiple stages.

| Tool | Stage | Purpose | Frequency |
|------|-------|---------|-----------|
| **Trivy** | CI build | Container image and filesystem vulnerability scanning | Every commit |
| **SonarQube** | CI build | Static Application Security Testing (SAST), code smells, security hotspots | Every commit |
| **npm audit / pnpm audit** | CI build | Node.js dependency vulnerability detection | Every commit |
| **OWASP Dependency-Check** | Nightly | Known-vulnerability detection (CVE/NVD database) | Daily |
| **Snyk** | PR review | Real-time dependency vulnerability alerts and fix suggestions | Every PR |
| **Renovate** | Scheduled | Automated dependency update PRs | Weekly |

### Policies

- **Zero critical vulnerabilities**: Builds fail if any dependency has a critical CVE with a known exploit.
- **14-day SLA for high vulnerabilities**: High-severity dependency CVEs must be patched or mitigated within 14 days.
- **License compliance**: Only OSI-approved licenses are permitted; GPL dependencies require legal review.
- **Lock files**: `pnpm-lock.yaml` is committed and integrity-checked in CI. Any unexpected change fails the build.
- **Supply chain**: Packages are sourced from a private registry (Verdaccio / Artifactory) that proxies and caches npm. New packages require security review before allowlisting.

---

## OWASP Top 10 (2021) Coverage

Every item in the OWASP Top 10 is addressed by specific controls in the SuperAdmin platform.

### A01:2021 — Broken Access Control

- **RBAC with 7-level hierarchy** enforces least-privilege access.
- **Row-Level Security (RLS)** at the PostgreSQL layer prevents cross-tenant data access.
- **Server-side authorization checks** on every API endpoint; client-side visibility controls are supplementary only.
- **Automated access control tests** verify that no endpoint is accessible without proper authorization.
- **CORS configuration** restricts allowed origins to known domains.
- **Directory listing disabled**; static files served with explicit allow-lists.

### A02:2021 — Cryptographic Failures

- **AES-256-GCM** encryption for all data at rest.
- **TLS 1.3** enforced for all data in transit; TLS 1.0/1.1 disabled.
- **bcrypt (cost 12)** for password hashing; migration path to Argon2id.
- **No sensitive data in URLs** — tokens, session IDs, and PII are never transmitted via query strings.
- **Secrets management** via HashiCorp Vault; no secrets in code, environment variables rotated on schedule.
- **Certificate pinning** for mobile clients; automated certificate rotation via Let's Encrypt / ACME.

### A03:2021 — Injection

- **Parameterized queries** (prepared statements) for all database operations via Drizzle ORM.
- **Input validation** using Zod schemas on every API endpoint — type, length, format, and range checks.
- **Output encoding** via React's default JSX escaping and server-side HTML entity encoding.
- **Content Security Policy (CSP)** headers prevent inline script execution.
- **SQL injection testing** included in CI via OWASP ZAP active scan.

### A04:2021 — Insecure Design

- **Threat modeling** performed during design of every major feature using STRIDE methodology.
- **Secure design patterns**: Rate limiting, circuit breakers, bulkhead isolation, and fail-secure defaults.
- **Multi-tenancy by design**: Tenant isolation is enforced at database, application, and network layers.
- **Security user stories** in the backlog for every feature (e.g., "As an attacker, I try to access another tenant's data").
- **Architecture review** required for changes affecting authentication, authorization, or data flow.

### A05:2021 — Security Misconfiguration

- **Infrastructure as Code (IaC)**: All configuration managed via Terraform / Helm charts; no manual server configuration.
- **Hardened Docker images**: Distroless base images; non-root container execution; read-only filesystems.
- **Security headers** enforced: CSP, X-Content-Type-Options, X-Frame-Options, Strict-Transport-Security, Permissions-Policy, Referrer-Policy.
- **Unnecessary features disabled**: Default accounts removed, debug endpoints disabled in production, directory listings off.
- **Configuration drift detection**: Automated checks compare running configuration against declared state.

### A06:2021 — Vulnerable and Outdated Components

- **Automated dependency scanning** via Trivy, SonarQube, and npm audit on every build.
- **Renovate** creates automated PRs for dependency updates weekly.
- **Software Bill of Materials (SBOM)** generated for every release using Syft.
- **End-of-life tracking**: Dependencies approaching EOL are flagged 90 days in advance.
- **Private registry** with allowlisting prevents introduction of unvetted packages.

### A07:2021 — Identification and Authentication Failures

- **MFA enforced** for all admin roles (Super Admin, Admin, Manager).
- **Bcrypt password hashing** with cost factor 12.
- **Password policy**: Minimum 12 characters, complexity requirements, breach database check (Have I Been Pwned API).
- **Account lockout** after 10 failed attempts within 15 minutes.
- **Session tokens** are cryptographically random (256-bit), stored server-side, and expire on idle timeout (30 min) and absolute timeout (24 hours).
- **Credential stuffing protection**: Rate limiting + CAPTCHA after 3 failed attempts from the same IP.

### A08:2021 — Software and Data Integrity Failures

- **CI/CD pipeline integrity**: Jenkins pipeline defined as code; no manual build steps.
- **Signed commits**: GPG-signed commits required for protected branches.
- **Container image signing**: Images signed with Cosign (Sigstore) before deployment.
- **Subresource Integrity (SRI)** hashes for all third-party scripts loaded in the browser.
- **Lock file integrity**: `pnpm-lock.yaml` changes trigger additional review.
- **Immutable deployments**: Container images are tagged with SHA digests, not mutable tags.

### A09:2021 — Security Logging and Monitoring Failures

- **Structured audit logging** for all authentication events, authorization decisions, and data mutations.
- **Centralized log aggregation** via ELK Stack (Elasticsearch, Logstash, Kibana) or equivalent.
- **Real-time alerting** for critical security events: brute-force attempts, privilege escalation, cross-tenant access attempts.
- **Log integrity**: Append-only storage with cryptographic chaining to detect tampering.
- **Retention**: Security logs retained for a minimum of 12 months (configurable per compliance requirement).
- **Monitoring dashboards**: Dedicated security operations dashboards with SLA-based alerting.

### A10:2021 — Server-Side Request Forgery (SSRF)

- **URL allow-listing**: Outbound HTTP requests are restricted to an explicit allow-list of domains.
- **Internal network blocking**: Requests to private IP ranges (10.x, 172.16.x, 192.168.x, 169.254.x) are blocked at the application layer.
- **DNS rebinding protection**: Resolved IP addresses are validated before connection.
- **Webhook URL validation**: Tenant-configured webhook URLs are validated against the allow-list and tested before activation.
- **Network segmentation**: Application containers cannot reach internal infrastructure services directly.

---

## Face ID / WebAuthn Biometric Authentication Integration Plan

### Overview

The platform supports the **Web Authentication API (WebAuthn)** to enable biometric authentication via Face ID, Touch ID, Windows Hello, and FIDO2 hardware security keys.

### Architecture

```
User Device                    SuperAdmin Backend
┌─────────────────┐           ┌──────────────────────────┐
│ Browser/App      │           │ WebAuthn Relying Party   │
│                  │  1. Begin │                          │
│ Biometric Prompt │◄─────────│ /api/auth/webauthn/begin │
│ (Face ID, etc.)  │           │                          │
│                  │  2. Sign  │                          │
│ Authenticator    │──────────►│ /api/auth/webauthn/verify│
│ Response         │           │                          │
└─────────────────┘           └──────────────────────────┘
```

### Implementation Phases

| Phase | Timeline | Deliverables |
|-------|----------|-------------|
| Phase 1 | Q2 2026 | WebAuthn registration and authentication endpoints; FIDO2 hardware key support |
| Phase 2 | Q3 2026 | Platform authenticator support (Face ID, Touch ID, Windows Hello) |
| Phase 3 | Q4 2026 | Passwordless-only mode for tenants that opt in |
| Phase 4 | Q1 2027 | Passkey synchronization across devices (multi-device FIDO credentials) |

### Security Considerations

- **Attestation verification**: Authenticator attestation is validated against the FIDO Metadata Service to ensure hardware integrity.
- **Credential storage**: Public keys are stored server-side; private keys never leave the authenticator.
- **User verification**: The `userVerification` parameter is set to `required` to ensure biometric or PIN verification occurs on the device.
- **Resident keys**: Discoverable credentials (resident keys) are supported for passwordless flows.
- **Fallback**: MFA via TOTP remains available as a fallback for devices without biometric capability.

---

## ML-KEM Post-Quantum Encryption Plan

### Background

NIST finalized **FIPS 203 (ML-KEM)** in August 2024 as the primary post-quantum key encapsulation mechanism based on the Module-Lattice-Based Key-Encapsulation Mechanism (formerly CRYSTALS-Kyber). Quantum computers capable of breaking RSA and ECC are projected within the next 10–15 years. Organizations must begin the transition now to protect data subject to "harvest now, decrypt later" attacks.

### ML-KEM Parameter Set

| Parameter | ML-KEM-512 | ML-KEM-768 | ML-KEM-1024 |
|-----------|-----------|-----------|------------|
| Security level | NIST Level 1 | NIST Level 3 | NIST Level 5 |
| Public key size | 800 bytes | 1,184 bytes | 1,568 bytes |
| Ciphertext size | 768 bytes | 1,088 bytes | 1,568 bytes |
| Shared secret | 32 bytes | 32 bytes | 32 bytes |

**Selected parameter set**: ML-KEM-768 (NIST Level 3) — balances security margin against performance overhead. Suitable for general-purpose enterprise use.

### Migration Roadmap

| Phase | Timeline | Scope |
|-------|----------|-------|
| **Assessment** | Q1–Q2 2026 | Inventory all cryptographic assets; identify certificates, key exchanges, and algorithms in use |
| **Hybrid deployment** | Q3–Q4 2026 | Deploy hybrid TLS (X25519 + ML-KEM-768) using OpenSSL 3.5+ or BoringSSL; no breaking changes |
| **Library integration** | Q1 2027 | Integrate liboqs or PQClean into application-level encryption for field-level data protection |
| **Full post-quantum** | Q2–Q3 2027 | Transition to pure ML-KEM-768 for key exchange where ecosystem support permits |
| **Signature migration** | Q4 2027 | Evaluate ML-DSA-65 (FIPS 204) for digital signatures; deploy hybrid signature schemes |

### Hybrid TLS Configuration

During the transition, the platform uses **hybrid key exchange** that combines a classical algorithm with a post-quantum algorithm:

```
# OpenSSL 3.5+ configuration
[ssl_sect]
Groups = x25519_mlkem768:x25519:secp384r1
MinProtocol = TLSv1.3
```

This ensures that if either algorithm is compromised, the other still provides security.

### Testing Strategy

- **Interoperability testing**: Verify TLS handshakes succeed with major clients (Chrome 131+, Firefox 132+, Safari 18+).
- **Performance benchmarks**: Measure handshake latency increase (expected: 0.5–1.5 ms additional).
- **Regression testing**: Full E2E suite runs against hybrid TLS endpoints.
- **Compliance validation**: Verify alignment with CNSA 2.0 (NSA) timeline requirements.

---

## Security Headers Reference

The following HTTP security headers are enforced on all responses.

| Header | Value | Purpose |
|--------|-------|---------|
| `Strict-Transport-Security` | `max-age=63072000; includeSubDomains; preload` | Enforce HTTPS for 2 years; submit to browser preload lists |
| `Content-Security-Policy` | `default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self' https://api.{DOMAIN}; frame-ancestors 'none'; base-uri 'self'; form-action 'self'` | Prevent XSS, clickjacking, and data injection |
| `X-Content-Type-Options` | `nosniff` | Prevent MIME-type sniffing |
| `X-Frame-Options` | `DENY` | Prevent clickjacking via iframes |
| `X-XSS-Protection` | `0` | Disabled — CSP provides superior protection; legacy header can introduce vulnerabilities |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Limit referrer information leakage |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=(), payment=()` | Disable access to sensitive browser APIs |
| `Cross-Origin-Embedder-Policy` | `require-corp` | Enable cross-origin isolation |
| `Cross-Origin-Opener-Policy` | `same-origin` | Prevent cross-origin window references |
| `Cross-Origin-Resource-Policy` | `same-origin` | Restrict resource loading to same origin |
| `Cache-Control` | `no-store, no-cache, must-revalidate` | Prevent caching of authenticated responses |

---

## Contact

For security-related inquiries, vulnerability reports, or questions about this policy:

- **Email**: security@{DOMAIN}
- **PGP Key**: Published at `https://{DOMAIN}/.well-known/security.txt`
- **Security page**: `https://{DOMAIN}/security`
- **Response time**: Acknowledgement within 24 hours

---

*This document is reviewed and updated quarterly. Last review: Q1 2026.*
