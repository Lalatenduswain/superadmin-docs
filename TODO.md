# TODO -- SuperAdmin SaaS Platform

> Consolidated task list for the SuperAdmin multi-tenant security platform.
> Effort estimates: **S** (< 1 day), **M** (1-3 days), **L** (3-7 days), **XL** (1-3 weeks).

---

## Critical (Must Have)

| # | Task | Effort | Dependencies | Notes |
|---|------|--------|-------------|-------|
| 1 | Complete PostgreSQL RLS policies for all tenant tables | L | Database schema finalized (see `17_database_schema.csv`) | Every table with a `tenant_id` column must have SELECT/INSERT/UPDATE/DELETE policies enforcing `current_setting('app.current_tenant')`. |
| 2 | Implement ML-KEM-768 post-quantum encryption for license keys | XL | OpenSSL 3.5+ or liboqs binding, `10_encryption_standards.csv` | NIST FIPS 203 compliant. Hybrid mode with X25519 for backward compatibility. Key encapsulation for license activation payloads. |
| 3 | Deploy Keycloak SSO with multi-realm tenant isolation | L | Docker/Podman runtime, PostgreSQL | One realm per tenant. SAML 2.0 + OIDC support. Map Keycloak roles to `03_rbac_roles.csv`. |
| 4 | Configure HashiCorp Vault / OpenBao for secrets management | L | Keycloak (AppRole auth), Infrastructure network | Transit engine for encryption-as-a-service. KV v2 for API keys, DB credentials, SMTP passwords. Auto-unseal with cloud KMS or Shamir keys. |
| 5 | Set up Jenkins CI/CD pipeline with full security scanning | XL | Gitea, Harbor, SonarQube, OWASP ZAP | Declarative Jenkinsfile. Stages: lint, test, SAST, build, container scan, DAST, deploy. See `INFRASTRUCTURE_STACK.md` Section 6. |
| 6 | Implement Face ID / WebAuthn biometric authentication | M | Keycloak deployed, frontend auth flow | Use `@simplewebauthn/server` + `@simplewebauthn/browser`. Store credentials in `webauthn_credentials` table. Fallback to TOTP. |
| 7 | Deploy Grafana + Prometheus monitoring stack | L | Kubernetes or Docker Compose cluster | Prometheus scrape configs for Node.js, PostgreSQL, Redis, Keycloak exporters. Grafana dashboards for SLA, error rates, tenant usage. Alertmanager for PagerDuty/Slack. |
| 8 | Configure Harbor container registry with Trivy scanning | M | Docker runtime, SSL certificates | Enforce scan-on-push. Block deployments with Critical/High CVEs. Replicate base images from Docker Hub. See `INFRASTRUCTURE_STACK.md` Section 7. |

- [ ] Complete PostgreSQL RLS policies for all tenant tables `[L]`
- [ ] Implement ML-KEM-768 post-quantum encryption for license keys `[XL]`
- [ ] Deploy Keycloak SSO with multi-realm tenant isolation `[L]`
- [ ] Configure HashiCorp Vault / OpenBao for secrets management `[L]`
- [ ] Set up Jenkins CI/CD pipeline with full security scanning `[XL]`
- [ ] Implement Face ID / WebAuthn biometric authentication `[M]`
- [ ] Deploy Grafana + Prometheus monitoring stack `[L]`
- [ ] Configure Harbor container registry with Trivy scanning `[M]`

---

## High Priority

| # | Task | Effort | Dependencies | Notes |
|---|------|--------|-------------|-------|
| 9 | Set up ClickHouse for analytics and audit log querying | L | Audit log schema (`06_audit_log_actions.csv`) | Columnar storage for fast aggregation. Ingest via Kafka/RabbitMQ. Retention policies per tenant tier (`19_license_tiers.csv`). |
| 10 | Implement Temporal workflows for async job processing | L | PostgreSQL, Redis/Valkey | Workflows: tenant provisioning, bulk user import, scheduled report generation, license expiry notifications. |
| 11 | Configure Varnish HTTP cache for read-heavy API endpoints | M | Nginx/Caddy reverse proxy | Cache tenant config, public dashboards, documentation endpoints. VCL rules for tenant-aware cache keys. Purge API for admin. |
| 12 | Deploy Selenium Grid for cross-browser testing | M | Docker runtime, Jenkins | Chrome, Firefox, Edge nodes. Parallel test execution. Integrate with Jenkins pipeline stage. |
| 13 | Set up Gitea self-hosted Git repository | S | PostgreSQL or SQLite, Docker | Mirror critical repos. Webhook integration with Jenkins. See `INFRASTRUCTURE_STACK.md` Section 6. |
| 14 | Configure Nexus Repository for npm/Docker proxy caching | M | Docker runtime, storage volume | npm proxy for `registry.npmjs.org`. Docker proxy for Docker Hub. Reduces external dependency and speeds up CI builds. |
| 15 | Implement Driver.js guided onboarding tours | S | Frontend framework deployed | Tours for: first login, dashboard overview, tenant creation wizard, security settings. Store completion state per user. |
| 16 | Add RabbitMQ message broker for webhook delivery | M | Docker runtime | Dead-letter queues for failed deliveries. Retry with exponential backoff. Per-tenant webhook endpoints stored in DB. |
| 17 | Set up MongoDB for flexible document storage (audit archives) | M | Docker runtime, backup strategy | Archive cold audit logs from ClickHouse. TTL indexes per tenant retention policy. GridFS for large attachments. |
| 18 | Deploy KeyDB as high-performance Redis alternative | M | Docker runtime | Drop-in Redis replacement with multi-threading. Use for session store, rate limiting (`04_rate_limiting.csv`), pub/sub. |

- [ ] Set up ClickHouse for analytics and audit log querying `[L]`
- [ ] Implement Temporal workflows for async job processing `[L]`
- [ ] Configure Varnish HTTP cache for read-heavy API endpoints `[M]`
- [ ] Deploy Selenium Grid for cross-browser testing `[M]`
- [ ] Set up Gitea self-hosted Git repository `[S]`
- [ ] Configure Nexus Repository for npm/Docker proxy caching `[M]`
- [ ] Implement Driver.js guided onboarding tours `[S]`
- [ ] Add RabbitMQ message broker for webhook delivery `[M]`
- [ ] Set up MongoDB for flexible document storage (audit archives) `[M]`
- [ ] Deploy KeyDB as high-performance Redis alternative `[M]`

---

## Medium Priority

| # | Task | Effort | Dependencies | Notes |
|---|------|--------|-------------|-------|
| 19 | SonarQube quality gate enforcement in CI/CD | M | Jenkins, SonarQube deployed | Quality gate: coverage > 80%, no Critical/Blocker issues, < 3% duplication. Fail Jenkins build on gate failure. |
| 20 | OWASP ZAP automated DAST in staging pipeline | M | Jenkins, staging environment, ZAP Docker image | Active scan against staging API. Baseline scan on every PR. Full scan nightly. Map findings to `02_owasp_top10_compliance.csv`. |
| 21 | Playwright E2E test suite for all critical paths | L | Frontend deployed, test database | Critical paths: login, tenant CRUD, user management, license activation, audit log viewing. Run in CI with headed mode for debugging. |
| 22 | Certbot automated SSL certificate renewal | S | DNS access, Nginx/Caddy | Let's Encrypt certificates. Auto-renewal cron or Caddy built-in ACME. Wildcard certs for `*.{DOMAIN}` tenant subdomains. |
| 23 | Kubernetes namespace-per-environment deployment | L | K3s/K8s cluster, Helm charts | Namespaces: `dev`, `staging`, `production`. NetworkPolicies isolating namespaces. Resource quotas per environment. |
| 24 | Cloudflare Workers edge caching | M | Cloudflare account, DNS delegation | Cache static assets and public API responses at the edge. Tenant-aware cache keys via headers. Purge on config change. |
| 25 | Performance benchmarking suite | M | k6 or Artillery installed, staging environment | Scenarios: login storm, concurrent API calls, bulk data import. Establish baselines. Run weekly in CI. Alert on regression > 10%. |

- [ ] SonarQube quality gate enforcement in CI/CD `[M]`
- [ ] OWASP ZAP automated DAST in staging pipeline `[M]`
- [ ] Playwright E2E test suite for all critical paths `[L]`
- [ ] Certbot automated SSL certificate renewal `[S]`
- [ ] Kubernetes namespace-per-environment deployment `[L]`
- [ ] Cloudflare Workers edge caching `[M]`
- [ ] Performance benchmarking suite `[M]`

---

## Low Priority / Future

| # | Task | Effort | Dependencies | Notes |
|---|------|--------|-------------|-------|
| 26 | Rust microservice for threat detection engine | XL | Threat detection rules (`08_threat_detection.csv`), gRPC or REST interface | High-performance pattern matching. Process security events from RabbitMQ. Sub-millisecond rule evaluation. |
| 27 | Deno sandbox for tenant-provided code execution | L | Deno runtime, sandboxing policy | Isolated V8 contexts per tenant. CPU/memory limits. No network access by default. Use for custom webhook transformations, report templates. |
| 28 | Bun migration evaluation for dev tooling | S | Bun runtime installed | Evaluate build speed, test runner, and compatibility with existing dependencies. Keep Node.js for production until ecosystem matures. |
| 29 | ML-based anomaly detection for security events | XL | ClickHouse (training data), Python/TensorFlow or ONNX runtime | Unsupervised learning on login patterns, API usage, data access. Alert on deviation from tenant baseline. |
| 30 | SIEM integration (Splunk / ELK) | L | Elasticsearch or Splunk instance, log shipper (Fluent Bit) | Forward structured audit logs. Correlation rules for multi-step attacks. Dashboard for SOC team. |

- [ ] Rust microservice for threat detection engine `[XL]`
- [ ] Deno sandbox for tenant-provided code execution `[L]`
- [ ] Bun migration evaluation for dev tooling `[S]`
- [ ] ML-based anomaly detection for security events `[XL]`
- [ ] SIEM integration (Splunk / ELK) `[L]`

---

## Dependency Graph (Simplified)

```
PostgreSQL ──> RLS Policies ──> Keycloak ──> WebAuthn
    │                              │
    ├──> ClickHouse                ├──> Vault/OpenBao
    ├──> MongoDB                   │
    └──> Temporal                  └──> Jenkins CI/CD ──> SonarQube
                                           │               │
                                           ├──> Harbor ────> Trivy
                                           ├──> ZAP DAST
                                           ├──> Selenium Grid
                                           └──> Playwright E2E
```

---

## Progress Tracking

| Priority | Total | Done | Remaining |
|----------|-------|------|-----------|
| Critical | 8 | 0 | 8 |
| High | 10 | 0 | 10 |
| Medium | 7 | 0 | 7 |
| Low / Future | 5 | 0 | 5 |
| **Total** | **30** | **0** | **30** |

> Last updated: 2026-03-27
