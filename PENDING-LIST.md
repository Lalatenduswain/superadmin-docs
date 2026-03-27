# Pending Items Tracker -- SuperAdmin SaaS Platform

> Detailed tracking of all pending work items organized by module.
> Statuses: **Not Started** | **In Progress** | **Blocked** | **Review** | **Done**

---

## Kanban Summary

| Not Started | In Progress | Blocked | Review | Done |
|:-----------:|:-----------:|:-------:|:------:|:----:|
| 28 | 0 | 3 | 0 | 0 |

### Blocked Items at a Glance

| Item | Blocker | Module |
|------|---------|--------|
| Vault AppRole integration with Keycloak | Keycloak multi-realm deployment not yet complete | Authentication & IAM |
| OWASP ZAP DAST in staging | Staging environment not provisioned; Jenkins pipeline incomplete | Security & Compliance |
| Playwright E2E suite | Frontend application not yet deployed to test environment | Testing |

---

## 1. Authentication & IAM

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| AUTH-01 | Deploy Keycloak SSO with multi-realm tenant isolation | Not Started | {OWNER} | {DUE_DATE} | L | PostgreSQL database ready | One realm per tenant. Configure SAML 2.0 + OIDC. Map roles from `03_rbac_roles.csv`. Admin realm for SuperAdmin operators. |
| AUTH-02 | Implement Face ID / WebAuthn biometric authentication | Not Started | {OWNER} | {DUE_DATE} | M | AUTH-01 (Keycloak) | `@simplewebauthn/server` v10+. Store attestation in `webauthn_credentials` table. TOTP fallback required. |
| AUTH-03 | Configure Vault / OpenBao AppRole auth with Keycloak | Blocked | {OWNER} | {DUE_DATE} | M | AUTH-01 (Keycloak deployed) | **Blocker:** Keycloak multi-realm deployment not yet complete. Vault policies map to Keycloak roles. Transit engine for encryption-as-a-service. |
| AUTH-04 | JWT token rotation and revocation strategy | Not Started | {OWNER} | {DUE_DATE} | M | AUTH-01 | Short-lived access tokens (15 min). Refresh token rotation. Redis-backed revocation list. See `11_jwt_configuration.csv`. |
| AUTH-05 | RBAC permission sync between Keycloak and application | Not Started | {OWNER} | {DUE_DATE} | M | AUTH-01, AUTH-03 | Sync `07_permissions.csv` definitions into Keycloak. Event listener for role changes. |

---

## 2. Database & Storage

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| DB-01 | Complete PostgreSQL RLS policies for all tenant tables | Not Started | {OWNER} | {DUE_DATE} | L | Schema from `17_database_schema.csv` | Every table with `tenant_id` must enforce `current_setting('app.current_tenant')`. Unit test each policy. |
| DB-02 | Set up ClickHouse for analytics and audit log querying | Not Started | {OWNER} | {DUE_DATE} | L | Audit schema (`06_audit_log_actions.csv`) | Materialized views for common queries. Data retention per license tier (`19_license_tiers.csv`). |
| DB-03 | Set up MongoDB for audit archive storage | Not Started | {OWNER} | {DUE_DATE} | M | DB-02 (ClickHouse as primary) | Cold storage for audit logs past ClickHouse retention window. TTL indexes. GridFS for large attachments. |
| DB-04 | Deploy KeyDB as Redis alternative | Not Started | {OWNER} | {DUE_DATE} | M | Docker runtime | Multi-threaded performance. Session store, rate limiting counters (`04_rate_limiting.csv`), pub/sub channels. |
| DB-05 | Database backup and point-in-time recovery strategy | Not Started | {OWNER} | {DUE_DATE} | M | DB-01 | WAL archiving with pgBackRest. Automated daily backups. Tested restore procedure. Per-tenant backup isolation. |
| DB-06 | MinIO object storage deployment | Not Started | {OWNER} | {DUE_DATE} | M | Docker runtime | S3-compatible API. Bucket-per-tenant isolation. Lifecycle policies for temp files. Versioning enabled for compliance. |

---

## 3. CI/CD Pipeline

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| CI-01 | Set up Jenkins with declarative pipeline | Not Started | {OWNER} | {DUE_DATE} | XL | Gitea, Docker runtime | Stages: checkout, lint, unit test, SAST, build, container scan, integration test, DAST, deploy. Agent pods on Kubernetes. |
| CI-02 | Set up Gitea self-hosted Git repository | Not Started | {OWNER} | {DUE_DATE} | S | PostgreSQL or SQLite | Webhook triggers for Jenkins. Branch protection rules. Mirror to backup location. |
| CI-03 | Configure Nexus Repository for proxy caching | Not Started | {OWNER} | {DUE_DATE} | M | Docker runtime, storage | npm proxy for `registry.npmjs.org`. Docker proxy for Docker Hub and Harbor. Reduce external dependency in CI. |
| CI-04 | Configure Harbor container registry | Not Started | {OWNER} | {DUE_DATE} | M | Docker, SSL certificates | Trivy scan-on-push. Block images with Critical CVEs from deployment. Robot accounts for CI. |
| CI-05 | SonarQube quality gate integration | Not Started | {OWNER} | {DUE_DATE} | M | CI-01 (Jenkins) | Coverage > 80%, 0 Critical/Blocker, duplication < 3%. Fail build on gate failure. |

---

## 4. Security & Compliance

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| SEC-01 | OWASP ZAP automated DAST in staging | Blocked | {OWNER} | {DUE_DATE} | M | CI-01, staging environment | **Blocker:** Staging environment not provisioned and Jenkins pipeline is incomplete. Active scan against all API endpoints. Map to `02_owasp_top10_compliance.csv`. |
| SEC-02 | Implement rate limiting middleware | Not Started | {OWNER} | {DUE_DATE} | M | DB-04 (KeyDB/Redis) | Sliding window algorithm. Tenant-aware limits from `04_rate_limiting.csv`. Return `Retry-After` header. |
| SEC-03 | Security headers middleware | Not Started | {OWNER} | {DUE_DATE} | S | None | Implement all headers from `05_security_headers.csv`. Use `helmet` with custom configuration. CSP with nonce-based inline scripts. |
| SEC-04 | Input sanitization and validation layer | Not Started | {OWNER} | {DUE_DATE} | M | None | Zod schemas for all API inputs. Sanitize HTML with DOMPurify. Rules from `13_input_sanitization.csv`. |
| SEC-05 | Certbot automated SSL renewal | Not Started | {OWNER} | {DUE_DATE} | S | DNS access, reverse proxy | Let's Encrypt wildcard certificates for `*.{DOMAIN}`. Cron-based renewal or Caddy built-in ACME. |
| SEC-06 | Compliance framework audit preparation | Not Started | {OWNER} | {DUE_DATE} | L | Multiple security items | SOC 2 Type II, ISO 27001, GDPR readiness. Evidence collection. See `COMPLIANCE_FRAMEWORKS.md`. |
| SEC-07 | File upload security hardening | Not Started | {OWNER} | {DUE_DATE} | M | MinIO (DB-06) | Magic byte validation, ClamAV scanning, size limits per tenant tier. Isolated storage paths. |

---

## 5. Monitoring & Observability

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| MON-01 | Deploy Grafana + Prometheus stack | Not Started | {OWNER} | {DUE_DATE} | L | Kubernetes/Docker | Exporters: node, postgres, redis, keycloak, application custom metrics. 15s scrape interval. |
| MON-02 | Alertmanager routing and notification setup | Not Started | {OWNER} | {DUE_DATE} | M | MON-01 | Routes: critical to PagerDuty, high to Slack, medium to email. Silencing and inhibition rules. |
| MON-03 | Structured logging with correlation IDs | Not Started | {OWNER} | {DUE_DATE} | M | None | Pino logger with `request_id`, `tenant_id`, `user_id` in every log line. JSON format for log aggregation. |
| MON-04 | Distributed tracing with OpenTelemetry | Not Started | {OWNER} | {DUE_DATE} | L | MON-01 (Grafana for Tempo) | Instrument HTTP, database, cache, and external API calls. Trace sampling at 10% in production. |
| MON-05 | Performance benchmarking suite | Not Started | {OWNER} | {DUE_DATE} | M | Staging environment | k6 scenarios: login storm, concurrent CRUD, bulk import. Weekly CI run. Regression alert at > 10% degradation. |

---

## 6. Testing

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| TEST-01 | Playwright E2E test suite | Blocked | {OWNER} | {DUE_DATE} | L | Frontend deployed to test env | **Blocker:** Frontend application not yet deployed to test environment. Critical paths: login, tenant CRUD, user management, license activation, audit log viewing. |
| TEST-02 | Deploy Selenium Grid for cross-browser testing | Not Started | {OWNER} | {DUE_DATE} | M | Docker, CI-01 (Jenkins) | Chrome, Firefox, Edge nodes. Parallel execution. VNC for debugging. |
| TEST-03 | Integration test suite with real database | Not Started | {OWNER} | {DUE_DATE} | L | PostgreSQL, test fixtures | Testcontainers for ephemeral Postgres. No mocks for DB layer. Test RLS policies, migrations, seeds. |
| TEST-04 | API contract testing | Not Started | {OWNER} | {DUE_DATE} | M | API schema (OpenAPI spec) | Validate all `15_api_endpoints.csv` against OpenAPI. Consumer-driven contracts for multi-service communication. |
| TEST-05 | Load testing baseline establishment | Not Started | {OWNER} | {DUE_DATE} | M | Staging environment | Establish p50/p95/p99 latency baselines. Determine max concurrent users per instance. Document capacity plan. |

---

## 7. Frontend

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| FE-01 | Implement Driver.js guided onboarding tours | Not Started | {OWNER} | {DUE_DATE} | S | Frontend framework | Tours: first login, dashboard overview, tenant creation, security settings. Persist completion state. |
| FE-02 | Admin dashboard tab implementation | Not Started | {OWNER} | {DUE_DATE} | L | AUTH-01, API layer | All 13 tabs from `09_admin_dashboard_tabs.csv`. Responsive layout. Role-based tab visibility per `07_permissions.csv`. |
| FE-03 | Setup Wizard frontend implementation | Not Started | {OWNER} | {DUE_DATE} | L | `SETUP_WIZARD_PROMPT.md` spec | 12-step wizard with Zod validation. Progress persistence. Stepper UI. Backend API integration. |
| FE-04 | Accessibility audit (WCAG 2.1 AA) | Not Started | {OWNER} | {DUE_DATE} | M | FE-02, FE-03 | axe-core automated checks. Manual keyboard navigation testing. Screen reader compatibility. |

---

## 8. Infrastructure

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| INFRA-01 | Kubernetes namespace-per-environment deployment | Not Started | {OWNER} | {DUE_DATE} | L | K3s/K8s cluster | Namespaces: `dev`, `staging`, `production`. NetworkPolicies. Resource quotas. See `INFRASTRUCTURE_STACK.md`. |
| INFRA-02 | Implement Temporal workflows for async jobs | Not Started | {OWNER} | {DUE_DATE} | L | PostgreSQL, Redis | Workflows: tenant provisioning, bulk import, scheduled reports, license expiry notifications. |
| INFRA-03 | Configure Varnish HTTP cache | Not Started | {OWNER} | {DUE_DATE} | M | Reverse proxy | Tenant-aware cache keys via `X-Tenant-ID` header. VCL rules. Purge API for cache invalidation on writes. |
| INFRA-04 | Add RabbitMQ message broker | Not Started | {OWNER} | {DUE_DATE} | M | Docker runtime | Queues: webhook delivery, email notifications, audit log ingestion. Dead-letter exchange. Per-tenant routing keys. |
| INFRA-05 | Cloudflare Workers edge caching | Not Started | {OWNER} | {DUE_DATE} | M | Cloudflare account | Cache static assets. Tenant-aware cache keys. Purge on config change. Analytics for cache hit ratio. |
| INFRA-06 | Docker Compose development environment | Not Started | {OWNER} | {DUE_DATE} | M | All service images | Single `docker-compose.yml` for local development. All 29 infrastructure services. Health checks. Volume mounts for code reload. |

---

## 9. Encryption & Post-Quantum

| ID | Item | Status | Owner | Due | Effort | Dependencies | Notes |
|----|------|--------|-------|-----|--------|-------------|-------|
| ENC-01 | ML-KEM-768 post-quantum encryption for license keys | Not Started | {OWNER} | {DUE_DATE} | XL | OpenSSL 3.5+ or liboqs | NIST FIPS 203 compliant. Hybrid mode: ML-KEM-768 + X25519. Key encapsulation for license activation payloads. See `10_encryption_standards.csv`. |
| ENC-02 | AES-256-GCM encryption for data at rest | Not Started | {OWNER} | {DUE_DATE} | M | Vault/OpenBao (AUTH-03) | Vault Transit engine for key management. Envelope encryption pattern. Rotate keys quarterly. |
| ENC-03 | TLS 1.3 enforcement across all services | Not Started | {OWNER} | {DUE_DATE} | S | SSL certificates (SEC-05) | Disable TLS 1.0/1.1/1.2 for internal traffic. Mutual TLS between microservices. Certificate pinning for critical paths. |
| ENC-04 | Database column-level encryption for PII | Not Started | {OWNER} | {DUE_DATE} | L | ENC-02, DB-01 | Encrypt: email, phone, SSN, address. Searchable encryption using blind index. Key-per-tenant isolation. |
| ENC-05 | Post-quantum TLS evaluation (ML-KEM for TLS handshake) | Not Started | {OWNER} | {DUE_DATE} | L | ENC-01 | Evaluate browser support for hybrid PQ/classical TLS. Chrome/Firefox PQ experiment status. Prepare migration path. |

---

## Status Change Log

| Date | Item ID | Old Status | New Status | Changed By | Notes |
|------|---------|-----------|-----------|-----------|-------|
| 2026-03-27 | ALL | -- | Not Started | System | Initial tracking document created |

---

> Last updated: 2026-03-27
