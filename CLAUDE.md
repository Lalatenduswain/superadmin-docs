# CLAUDE.md -- Project Instructions for Claude Code

> This file provides context and conventions for Claude Code when working in this repository.

---

## Project Overview

**SuperAdmin** is a multi-tenant SaaS platform for centralized security administration. This repository contains the complete documentation, schemas, and architecture specifications covering 50+ sections of security, multi-tenancy, RBAC, OWASP compliance, encryption, infrastructure, and operational procedures.

The platform manages tenant lifecycle, user identity, license enforcement, audit logging, threat detection, and compliance reporting for enterprise customers.

---

## Repository Structure

### Documentation Files

| File | Purpose |
|------|---------|
| `SUPER_ADMIN_DOCUMENTATION.md` | Primary documentation: 3200+ lines, 50 sections covering every aspect of the platform |
| `SETUP_WIZARD_PROMPT.md` | 12-step universal SaaS setup wizard with Zod schemas, UI specs, and API architecture |
| `INFRASTRUCTURE_STACK.md` | 29 self-hosted infrastructure technologies with Docker Compose configs and deployment order |
| `COMPLIANCE_FRAMEWORKS.md` | SOC 2, ISO 27001, GDPR, HIPAA, PCI-DSS compliance mapping and evidence requirements |
| `README.md` | Repository overview and quick-start guide |
| `TODO.md` | Prioritized task list with effort estimates and dependencies |
| `PENDING-LIST.md` | Detailed pending items tracker organized by module with status and ownership |
| `CLAUDE_ARCHIVE.md` | Session history and architectural decision records |

### CSV Reference Data (Generated -- Do Not Edit Manually)

| File | Content |
|------|---------|
| `01_implementation_checklist.csv` | Master implementation checklist with phases and status |
| `02_owasp_top10_compliance.csv` | OWASP Top 10 mapping with mitigations per vulnerability |
| `03_rbac_roles.csv` | Role definitions: SuperAdmin, TenantAdmin, User, ReadOnly, etc. |
| `04_rate_limiting.csv` | Rate limiting rules per endpoint and tenant tier |
| `05_security_headers.csv` | HTTP security headers: CSP, HSTS, X-Frame-Options, etc. |
| `06_audit_log_actions.csv` | Audit log event types and severity classifications |
| `07_permissions.csv` | Granular permission definitions mapped to roles |
| `08_threat_detection.csv` | Threat detection rules: brute force, injection, anomaly patterns |
| `09_admin_dashboard_tabs.csv` | Admin dashboard tab definitions (13 tabs) |
| `10_encryption_standards.csv` | Encryption algorithms: AES-256-GCM, ML-KEM-768, Argon2id, etc. |
| `11_jwt_configuration.csv` | JWT settings: algorithm, expiry, issuer, audience, rotation |
| `12_password_policy.csv` | Password requirements: length, complexity, history, expiry |
| `13_input_sanitization.csv` | Input validation and sanitization rules per field type |
| `14_compliance_summary.csv` | Compliance requirement summary across frameworks |
| `15_api_endpoints.csv` | API endpoint inventory with methods, auth, rate limits |
| `16_implementation_phases.csv` | Phased rollout plan with milestones |
| `17_database_schema.csv` | PostgreSQL table definitions with columns and constraints |
| `18_file_reference.csv` | Cross-reference of all documentation files |
| `19_license_tiers.csv` | License tier definitions: Free, Pro, Enterprise, Enterprise+ |
| `20_safety_guards.csv` | Safety guard rules for destructive operations |

---

## Conventions

### Placeholder Format

All placeholders use the `{PLACEHOLDER}` format with uppercase and underscores:

```
{DOMAIN}           -- Primary domain (e.g., admin.example.com)
{DB_HOST}          -- PostgreSQL host
{DB_PASSWORD}      -- Database password (never hardcode)
{KEYCLOAK_URL}     -- Keycloak base URL
{VAULT_TOKEN}      -- Vault root/AppRole token
{TENANT_ID}        -- UUID tenant identifier
{SMTP_HOST}        -- Email relay host
{OWNER}            -- Task owner name
{DUE_DATE}         -- Target completion date
```

Never replace placeholders with real credentials. They are meant to be substituted at deployment time by environment variables or secrets management.

### Code Style

- **Language:** TypeScript (strict mode enabled)
- **Linting:** ESLint with `@typescript-eslint` and security-focused rules
- **Formatting:** Prettier (2-space indent, single quotes, trailing commas)
- **Commits:** Conventional Commits format
  - `feat:` new feature
  - `fix:` bug fix
  - `docs:` documentation only
  - `refactor:` code restructuring
  - `test:` adding or fixing tests
  - `chore:` maintenance, dependencies
  - `security:` security-related changes
- **Branch naming:** `feature/`, `fix/`, `docs/`, `security/` prefixes

### Commit Message Format

```
<type>(<scope>): <short description>

<optional body with details>

<optional footer: BREAKING CHANGE, references>
```

Example:
```
feat(auth): add WebAuthn biometric authentication

Implement FIDO2/WebAuthn for Face ID and fingerprint login.
Uses @simplewebauthn/server v10. Stores credentials in
webauthn_credentials table with per-tenant isolation.

Refs: AUTH-02
```

---

## Architecture

### Backend
- **Runtime:** Node.js 20 LTS with Express.js
- **Language:** TypeScript 5.x (strict mode)
- **Validation:** Zod schemas for all API inputs
- **ORM:** Drizzle ORM with raw SQL for complex queries
- **API format:** RESTful JSON, versioned (`/api/v1/...`)

### Database
- **Primary:** PostgreSQL 16 with Row-Level Security (RLS)
- **Analytics:** ClickHouse (columnar, audit log aggregation)
- **Documents:** MongoDB (audit archives, flexible schemas)
- **Cache:** Redis/Valkey/KeyDB (sessions, rate limiting, pub/sub)

### Identity & Access Management
- **SSO:** Keycloak (one realm per tenant, SAML 2.0 + OIDC)
- **Secrets:** HashiCorp Vault or OpenBao (Transit, KV v2)
- **Auth:** JWT (short-lived) + refresh token rotation
- **MFA:** TOTP + WebAuthn (Face ID, fingerprint)
- **Passwords:** Argon2id hashing (see `12_password_policy.csv`)

### Infrastructure (29 technologies)
See `INFRASTRUCTURE_STACK.md` for the complete list, including: PostgreSQL, ClickHouse, MongoDB, KeyDB, Keycloak, Vault, Caddy, Varnish, Jenkins, Gitea, Nexus, Harbor, SonarQube, OWASP ZAP, Trivy, K3s, Grafana, Prometheus, Loki, MinIO, Selenium Grid, Temporal, Node.js, Deno, Bun, RabbitMQ, Certbot, Cloudflare Workers, and Driver.js.

---

## Testing

### Unit Tests
- **Framework:** Vitest
- **Coverage target:** > 80% line coverage
- **Convention:** Co-locate test files as `*.test.ts` next to source
- **Run:** `npm run test:unit`

### Integration Tests
- **Database:** Use real PostgreSQL via Testcontainers (no mocks for DB layer)
- **Pattern:** Test actual RLS policies, migrations, and query behavior
- **Run:** `npm run test:integration`

### End-to-End Tests
- **Framework:** Playwright
- **Browsers:** Chromium, Firefox, WebKit
- **Critical paths:** Login, tenant CRUD, user management, license activation, audit log viewing
- **Run:** `npm run test:e2e`

### Security Tests
- **SAST:** SonarQube (integrated in Jenkins pipeline)
- **DAST:** OWASP ZAP (automated scans against staging)
- **Container scan:** Trivy (scan-on-push in Harbor)
- **Dependency audit:** `npm audit` in CI, Snyk for continuous monitoring

---

## Security-First Principles

These rules are non-negotiable. Enforce them in every code change:

1. **Never commit secrets.** No API keys, passwords, tokens, or certificates in source control. Use `{PLACEHOLDER}` format and load from Vault/environment.

2. **Validate all inputs.** Every API endpoint must validate request body, query params, and path params with Zod schemas. Reject unknown fields.

3. **Parameterized queries only.** Never concatenate user input into SQL strings. Use parameterized queries or ORM-generated SQL exclusively.

4. **Tenant isolation is mandatory.** Every database query must be scoped by `tenant_id` via RLS. Never trust client-provided tenant IDs -- derive from the authenticated session.

5. **Least privilege.** Service accounts, database roles, and API tokens must have the minimum permissions required. No wildcard grants.

6. **Audit everything.** All state-changing operations must emit audit log events per `06_audit_log_actions.csv`.

7. **OWASP Top 10 awareness.** When reviewing code, check for all OWASP Top 10 vulnerabilities. Reference `02_owasp_top10_compliance.csv` for mitigations.

8. **Encryption standards.** ML-KEM-768 (FIPS 203) for license key encapsulation. AES-256-GCM for data at rest. Argon2id for password hashing. TLS 1.3 for data in transit. See `10_encryption_standards.csv`.

9. **Security headers.** All responses must include headers from `05_security_headers.csv`: strict CSP, HSTS, X-Content-Type-Options, Referrer-Policy, Permissions-Policy.

10. **Rate limiting.** All endpoints must enforce rate limits per `04_rate_limiting.csv`. Use sliding window algorithm backed by Redis/KeyDB.

---

## Common Tasks

### Adding a New API Endpoint
1. Define the route in the appropriate router module
2. Create Zod input/output schemas
3. Add RLS-aware database queries (always scope by `tenant_id`)
4. Add to `15_api_endpoints.csv` with method, auth requirement, and rate limit
5. Add audit log event to `06_audit_log_actions.csv`
6. Write unit and integration tests
7. Update OpenAPI specification

### Adding a New Tenant-Scoped Table
1. Create migration with `tenant_id UUID NOT NULL REFERENCES tenants(id)`
2. Add RLS policy: `CREATE POLICY tenant_isolation ON {table} USING (tenant_id = current_setting('app.current_tenant')::uuid)`
3. Add to `17_database_schema.csv`
4. Write integration test verifying cross-tenant isolation

### Updating Documentation
1. Edit the relevant `.md` file directly
2. Do NOT manually edit CSV files -- they are generated from the documentation processing pipeline
3. If adding a new section to `SUPER_ADMIN_DOCUMENTATION.md`, update the table of contents
4. Use conventional commit: `docs(<scope>): <description>`

### Security Review Checklist
When reviewing any code change, verify:
- [ ] No hardcoded secrets or credentials
- [ ] All inputs validated with Zod schemas
- [ ] SQL uses parameterized queries (no string concatenation)
- [ ] Tenant isolation enforced (RLS or explicit `WHERE tenant_id =`)
- [ ] Audit log events emitted for state changes
- [ ] Rate limiting applied to new endpoints
- [ ] Security headers present on new response paths
- [ ] Error messages do not leak internal details
- [ ] Authentication and authorization checks in place
- [ ] OWASP Top 10 mitigations verified

---

## Files NOT to Modify

- **`01_*.csv` through `20_*.csv`** -- These CSV files are generated from the documentation pipeline. To change their content, update the source documentation and regenerate.
- **`.git/`** -- Obviously, never manually modify git internals.

---

## Post-Quantum Encryption Reference

The platform implements quantum-resistant cryptography in preparation for the post-quantum era:

| Algorithm | Standard | Use Case | Key Size |
|-----------|----------|----------|----------|
| ML-KEM-768 | NIST FIPS 203 | License key encapsulation | 1184 bytes (public) |
| ML-DSA-65 | NIST FIPS 204 | Digital signatures (future) | 1952 bytes (public) |
| AES-256-GCM | NIST SP 800-38D | Data at rest encryption | 256 bits |
| Argon2id | RFC 9106 | Password hashing | N/A (hash function) |
| X25519 | RFC 7748 | Key exchange (classical, hybrid) | 32 bytes |

Hybrid mode (ML-KEM-768 + X25519) is used during the transition period for backward compatibility. See `10_encryption_standards.csv` and `SUPER_ADMIN_DOCUMENTATION.md` encryption sections for full details.

---

## Quick Reference

```bash
# Development
npm install              # Install dependencies
npm run dev              # Start development server
npm run build            # Production build
npm run lint             # ESLint check
npm run format           # Prettier format

# Testing
npm run test:unit        # Vitest unit tests
npm run test:integration # Integration tests (needs Docker)
npm run test:e2e         # Playwright E2E tests
npm run test:security    # OWASP ZAP + SonarQube

# Infrastructure
docker compose up -d     # Start all services (see INFRASTRUCTURE_STACK.md)
docker compose ps        # Check service health
```
