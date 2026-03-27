# Infrastructure Stack — Self-Hosted DevOps & Platform Architecture

> **Complete infrastructure blueprint** for deploying a production-grade, self-hosted SaaS platform with CI/CD, security scanning, monitoring, and container orchestration.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Database Layer](#2-database-layer)
3. [Caching & In-Memory Layer](#3-caching--in-memory-layer)
4. [Identity, Auth & Secrets Management](#4-identity-auth--secrets-management)
5. [Reverse Proxy & Load Balancing](#5-reverse-proxy--load-balancing)
6. [CI/CD Pipeline](#6-cicd-pipeline)
7. [Artifact & Container Management](#7-artifact--container-management)
8. [Security Scanning & Compliance](#8-security-scanning--compliance)
9. [Container Orchestration](#9-container-orchestration)
10. [Monitoring & Observability](#10-monitoring--observability)
11. [Object Storage](#11-object-storage)
12. [End-to-End & Browser Testing](#12-end-to-end--browser-testing)
13. [Application Runtimes](#13-application-runtimes)
14. [Docker Compose Reference](#14-docker-compose-reference)
15. [Deployment Order & Dependencies](#15-deployment-order--dependencies)
16. [Port Reference](#16-port-reference)
17. [Placeholder Reference](#17-placeholder-reference)

---

## 1. Architecture Overview

```
                         ┌─────────────────────────────────────────────┐
                         │              INTERNET / USERS               │
                         └────────────────────┬────────────────────────┘
                                              │
                                   ┌──────────▼──────────┐
                                   │   Traefik (Proxy)   │
                                   │   :80 / :443 / :8080│
                                   └──────────┬──────────┘
                                              │
                    ┌─────────────┬───────────┼───────────┬──────────────┐
                    │             │           │           │              │
              ┌─────▼─────┐ ┌────▼────┐ ┌────▼────┐ ┌───▼────┐  ┌─────▼─────┐
              │  App API  │ │ Grafana │ │  Gitea  │ │ Jenkins│  │  Harbor   │
              │ (Node/Bun)│ │  :3000  │ │  :3001  │ │  :8080 │  │   :443    │
              └─────┬─────┘ └────┬────┘ └─────────┘ └───┬────┘  └───────────┘
                    │            │                       │
         ┌──────┬──┴──┬─────┐   │            ┌──────────┼──────────┐
         │      │     │     │   │            │          │          │
    ┌────▼──┐┌──▼──┐┌─▼──┐┌▼───▼───┐  ┌─────▼───┐┌────▼────┐┌───▼────┐
    │Postgre││Redis││Mong││Promethe│  │SonarQube││  Nexus  ││  Trivy │
    │  SQL  ││Valke││oDB ││us      │  │         ││         ││        │
    │ :5432 ││y/Key││:270││ :9090  │  │  :9000  ││  :8081  ││  CLI   │
    │       ││DB   ││17  ││        │  │         ││         ││        │
    └───────┘└─────┘└────┘└────────┘  └─────────┘└─────────┘└────────┘
                                              │
                              ┌────────────────┤
                              │                │
                        ┌─────▼─────┐   ┌──────▼──────┐
                        │ ClickHouse│   │  OWASP ZAP  │
                        │  :8123    │   │  (on-demand) │
                        └───────────┘   └─────────────┘
```

### Design Principles

| Principle | Implementation |
|-----------|---------------|
| **Self-Hosted First** | All tools run on your infrastructure — no vendor lock-in |
| **Security by Default** | TLS everywhere, secrets in Vault/OpenBao, image scanning before deploy |
| **Observable** | Grafana + Prometheus for metrics; ClickHouse for analytics/log queries |
| **GitOps Ready** | Gitea as source of truth, Jenkins pipelines as code, K8s manifests in repo |
| **Multi-Runtime** | Node.js (primary), Bun/Deno (fast alternative), Rust (performance-critical services) |

---

## 2. Database Layer

### 2.1 PostgreSQL — Primary Relational Database

| Property | Value |
|----------|-------|
| **Role** | Primary OLTP database, multi-tenant data store |
| **Port** | `{DB_PORT}` (default: 5432) |
| **Key Features** | Row-Level Security (RLS), JSONB, full-text search, partitioning |
| **Multi-Tenancy** | Tenant isolation via RLS policies (see Section 14 of main doc) |
| **Backup** | pg_dump daily + WAL archiving for point-in-time recovery |
| **Monitoring** | `pg_stat_statements` for query performance |

```yaml
# docker-compose snippet
postgresql:
  image: postgres:17-alpine
  environment:
    POSTGRES_DB: ${DB_NAME}
    POSTGRES_USER: ${DB_USER}
    POSTGRES_PASSWORD: ${DB_PASSWORD}
  volumes:
    - pg_data:/var/lib/postgresql/data
    - ./init-scripts:/docker-entrypoint-initdb.d
  ports:
    - "${DB_PORT:-5432}:5432"
  deploy:
    resources:
      limits:
        memory: 2G
  healthcheck:
    test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
    interval: 10s
    timeout: 5s
    retries: 5
```

### 2.2 MongoDB — Document Store

| Property | Value |
|----------|-------|
| **Role** | Flexible document storage for logs, configurations, unstructured data |
| **Port** | 27017 |
| **Key Features** | Schema-less documents, aggregation pipeline, change streams |
| **Use Cases** | Audit log archival, feature flag configs, webhook event payloads, dynamic forms |
| **Why alongside PostgreSQL** | PostgreSQL handles relational/transactional data with RLS; MongoDB handles high-volume append-only logs and flexible schema documents where rigid tables are overkill |

```yaml
mongodb:
  image: mongo:7
  environment:
    MONGO_INITDB_ROOT_USERNAME: ${MONGO_USER}
    MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
  volumes:
    - mongo_data:/data/db
  ports:
    - "27017:27017"
  healthcheck:
    test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
    interval: 10s
    timeout: 5s
    retries: 5
```

### 2.3 ClickHouse — Analytics & Log Engine

| Property | Value |
|----------|-------|
| **Role** | Columnar OLAP database for analytics, event logs, and time-series queries |
| **Ports** | 8123 (HTTP), 9000 (native) |
| **Key Features** | Column-oriented storage, 100x faster aggregation than PostgreSQL for analytics |
| **Use Cases** | Dashboard analytics, audit log querying, tenant usage metrics, BI reporting |
| **Data Flow** | App writes events to ClickHouse async via Kafka/direct insert; Grafana reads from ClickHouse |

```yaml
clickhouse:
  image: clickhouse/clickhouse-server:24
  volumes:
    - clickhouse_data:/var/lib/clickhouse
    - ./clickhouse-config.xml:/etc/clickhouse-server/config.d/custom.xml
  ports:
    - "8123:8123"
    - "9000:9000"
  ulimits:
    nofile:
      soft: 262144
      hard: 262144
  healthcheck:
    test: ["CMD", "clickhouse-client", "--query", "SELECT 1"]
    interval: 10s
    timeout: 5s
    retries: 5
```

---

## 3. Caching & In-Memory Layer

### 3.1 Redis / Valkey — Primary Cache & Session Store

| Property | Value |
|----------|-------|
| **Role** | Session storage, rate limiting (ZSET sliding window), response caching, pub/sub |
| **Port** | `{CACHE_PORT}` (default: 6379) |
| **Valkey** | Drop-in Redis replacement, fully open-source (Linux Foundation), API-compatible |
| **TTL Strategy** | Sessions: 24h, API cache: 5-60min, Rate limit windows: 1-15min |

```yaml
valkey:
  image: valkey/valkey:8-alpine
  command: valkey-server --maxmemory 512mb --maxmemory-policy allkeys-lru --requirepass ${CACHE_PASSWORD}
  volumes:
    - valkey_data:/data
  ports:
    - "${CACHE_PORT:-6379}:6379"
  healthcheck:
    test: ["CMD", "valkey-cli", "-a", "${CACHE_PASSWORD}", "ping"]
    interval: 10s
    timeout: 3s
    retries: 5
```

### 3.2 KeyDB — High-Performance Redis Alternative

| Property | Value |
|----------|-------|
| **Role** | Drop-in Redis replacement with multi-threaded architecture |
| **Port** | 6380 (when running alongside Valkey/Redis) |
| **Key Features** | Multi-threaded (uses all CPU cores), active replication, FLASH storage tier |
| **Use Cases** | High-throughput rate limiting, real-time leaderboards, pub/sub at scale |
| **When to use over Valkey** | When single-threaded Redis/Valkey becomes a bottleneck under heavy load |

```yaml
keydb:
  image: eqalpha/keydb:latest
  command: keydb-server --server-threads 4 --requirepass ${CACHE_PASSWORD}
  volumes:
    - keydb_data:/data
  ports:
    - "6380:6379"
  healthcheck:
    test: ["CMD", "keydb-cli", "-a", "${CACHE_PASSWORD}", "ping"]
    interval: 10s
    timeout: 3s
    retries: 5
```

### 3.3 Varnish — HTTP Cache Accelerator

| Property | Value |
|----------|-------|
| **Role** | HTTP reverse-proxy cache sitting in front of the API for read-heavy endpoints |
| **Port** | 6081 |
| **Key Features** | VCL-based rules, sub-millisecond cache hits, grace mode for stale content |
| **Use Cases** | Public API responses, tenant dashboard data, asset metadata endpoints |
| **Cache Strategy** | Cache GET requests with `Cache-Control` headers; bypass for authenticated mutations |

```yaml
varnish:
  image: varnish:7-alpine
  volumes:
    - ./varnish/default.vcl:/etc/varnish/default.vcl:ro
  ports:
    - "6081:80"
  environment:
    VARNISH_SIZE: 256M
  command: varnishd -F -a :80 -f /etc/varnish/default.vcl -s malloc,256M
  depends_on:
    - app
```

---

## 4. Identity, Auth & Secrets Management

### 4.1 Keycloak — Identity & Access Management (IAM)

| Property | Value |
|----------|-------|
| **Role** | Centralized SSO, OAuth2/OIDC provider, user federation, social login |
| **Port** | 8180 |
| **Key Features** | Multi-realm (one per tenant), LDAP/AD federation, TOTP/WebAuthn MFA, RBAC |
| **Integration** | App validates JWT tokens issued by Keycloak; supports SAML for enterprise SSO |
| **Why Keycloak** | Eliminates custom auth code, provides admin console for user management, supports 20+ identity providers |

```yaml
keycloak:
  image: quay.io/keycloak/keycloak:25
  command: start-dev  # Use 'start' with proper TLS in production
  environment:
    KEYCLOAK_ADMIN: ${KC_ADMIN_USER}
    KEYCLOAK_ADMIN_PASSWORD: ${KC_ADMIN_PASSWORD}
    KC_DB: postgres
    KC_DB_URL: jdbc:postgresql://postgresql:5432/${KC_DB_NAME}
    KC_DB_USERNAME: ${DB_USER}
    KC_DB_PASSWORD: ${DB_PASSWORD}
  ports:
    - "8180:8080"
  depends_on:
    postgresql:
      condition: service_healthy
```

### 4.2 HashiCorp Vault — Secrets Management

| Property | Value |
|----------|-------|
| **Role** | Centralized secrets storage, dynamic credentials, encryption-as-a-service |
| **Port** | 8200 |
| **Key Features** | Secret engines (KV, database, PKI), auto-unseal, audit logging, policies |
| **Use Cases** | Database credentials rotation, API keys, TLS certificates, encryption keys |
| **Integration** | Apps fetch secrets via Vault API or Agent sidecar; CI/CD reads build secrets |

```yaml
vault:
  image: hashicorp/vault:1.17
  cap_add:
    - IPC_LOCK
  environment:
    VAULT_ADDR: "http://0.0.0.0:8200"
    VAULT_API_ADDR: "http://vault:8200"
  volumes:
    - vault_data:/vault/data
    - ./vault/config.hcl:/vault/config/config.hcl
  ports:
    - "8200:8200"
  command: vault server -config=/vault/config/config.hcl
  healthcheck:
    test: ["CMD", "vault", "status"]
    interval: 10s
    timeout: 5s
    retries: 5
```

### 4.3 OpenBao — Open-Source Vault Alternative

| Property | Value |
|----------|-------|
| **Role** | Community-maintained Vault fork (Linux Foundation), API-compatible |
| **Port** | 8200 (use as drop-in replacement for Vault) |
| **Key Features** | Same secret engines as Vault, no BSL license concerns, community-driven |
| **When to use** | When you want Vault functionality without HashiCorp's BSL license restrictions |
| **Migration** | Direct migration path from Vault — same API, same CLI, same config format |

```yaml
openbao:
  image: quay.io/openbao/openbao:latest
  cap_add:
    - IPC_LOCK
  environment:
    BAO_ADDR: "http://0.0.0.0:8200"
  volumes:
    - bao_data:/vault/data
    - ./vault/config.hcl:/openbao/config/config.hcl
  ports:
    - "8200:8200"
  command: bao server -config=/openbao/config/config.hcl
```

> **Decision Guide:** Use **HashiCorp Vault** if your team already has Vault expertise or needs enterprise support. Use **OpenBao** for new deployments wanting a fully open-source solution with no license risk.

---

## 5. Reverse Proxy & Load Balancing

### 5.1 Traefik — Primary Reverse Proxy & Ingress

| Property | Value |
|----------|-------|
| **Role** | Automatic SSL (Let's Encrypt), service discovery, load balancing, K8s ingress |
| **Ports** | 80 (HTTP), 443 (HTTPS), 8080 (dashboard) |
| **Key Features** | Auto-discovery via Docker labels, middleware chains, rate limiting, circuit breaker |
| **Why Traefik** | Zero-config service discovery — add Docker labels and Traefik auto-routes traffic |

```yaml
traefik:
  image: traefik:v3
  command:
    - --api.dashboard=true
    - --providers.docker=true
    - --providers.docker.exposedbydefault=false
    - --entrypoints.web.address=:80
    - --entrypoints.websecure.address=:443
    - --certificatesresolvers.letsencrypt.acme.tlschallenge=true
    - --certificatesresolvers.letsencrypt.acme.email=${ADMIN_EMAIL}
    - --certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json
  ports:
    - "80:80"
    - "443:443"
    - "8080:8080"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock:ro
    - traefik_certs:/letsencrypt
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.dashboard.rule=Host(`traefik.${DOMAIN}`)"
    - "traefik.http.routers.dashboard.tls.certresolver=letsencrypt"
```

### 5.2 Nginx — Alternative / Static Asset Server

| Property | Value |
|----------|-------|
| **Role** | Static file serving, reverse proxy, SSL termination |
| **Port** | 80/443 |
| **Use Cases** | Serving frontend SPA builds, WebSocket proxying, IP detection (`X-Real-IP`) |
| **When to use** | When Traefik's auto-discovery is not needed, or for simple static hosting |

### 5.3 Apache — Legacy / Compatibility Proxy

| Property | Value |
|----------|-------|
| **Role** | Reverse proxy for legacy apps, .htaccess compatibility |
| **Port** | 80/443 |
| **When to use** | Only when integrating with legacy systems that require Apache-specific modules (mod_rewrite, mod_security) |

> **Recommendation:** Use **Traefik** as the primary ingress for all services. Use **Nginx** only for static asset serving if needed. Use **Apache** only for legacy system integration.

---

## 6. CI/CD Pipeline

### 6.1 Gitea — Self-Hosted Git Repository

| Property | Value |
|----------|-------|
| **Role** | Source code repository, code review, issue tracking |
| **Port** | 3001 (HTTP), 2222 (SSH) |
| **Key Features** | Lightweight (~40MB RAM), Git LFS, webhooks, built-in CI (Gitea Actions) |
| **Why Gitea** | Fully self-hosted GitHub alternative; lighter than GitLab; supports GitHub Actions syntax |

```yaml
gitea:
  image: gitea/gitea:1.22
  environment:
    GITEA__database__DB_TYPE: postgres
    GITEA__database__HOST: postgresql:5432
    GITEA__database__NAME: ${GITEA_DB_NAME}
    GITEA__database__USER: ${DB_USER}
    GITEA__database__PASSWD: ${DB_PASSWORD}
  volumes:
    - gitea_data:/data
  ports:
    - "3001:3000"
    - "2222:22"
  depends_on:
    postgresql:
      condition: service_healthy
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.gitea.rule=Host(`git.${DOMAIN}`)"
    - "traefik.http.routers.gitea.tls.certresolver=letsencrypt"
```

### 6.2 Jenkins — CI/CD Engine

| Property | Value |
|----------|-------|
| **Role** | Build automation, pipeline orchestration, deployment engine |
| **Port** | 8082 (HTTP), 50000 (agent) |
| **Key Features** | Declarative pipelines (Jenkinsfile), 1800+ plugins, distributed builds |
| **Pipeline Stages** | Checkout → Build → Test → SonarQube → Trivy Scan → Push to Harbor → Deploy |

```yaml
jenkins:
  image: jenkins/jenkins:lts-jdk21
  environment:
    JAVA_OPTS: "-Djenkins.install.runSetupWizard=false"
  volumes:
    - jenkins_data:/var/jenkins_home
    - /var/run/docker.sock:/var/run/docker.sock
  ports:
    - "8082:8080"
    - "50000:50000"
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.jenkins.rule=Host(`ci.${DOMAIN}`)"
    - "traefik.http.routers.jenkins.tls.certresolver=letsencrypt"
```

#### Jenkinsfile Reference

```groovy
pipeline {
    agent any

    environment {
        HARBOR_REGISTRY = "harbor.${DOMAIN}"
        IMAGE_NAME = "${PROJECT_NAME}"
        IMAGE_TAG = "${BUILD_NUMBER}"
        SONAR_HOST = "http://sonarqube:9000"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install & Lint') {
            steps {
                sh 'npm ci'
                sh 'npm run lint'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm run test:unit -- --coverage'
            }
            post {
                always {
                    junit 'reports/junit.xml'
                    publishHTML(target: [reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Coverage'])
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Image') {
            steps {
                sh "docker build -t ${HARBOR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image --exit-code 1 --severity HIGH,CRITICAL ${HARBOR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo ${PASS} | docker login ${HARBOR_REGISTRY} -u ${USER} --password-stdin"
                    sh "docker push ${HARBOR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage('OWASP ZAP Scan') {
            when { branch 'main' }
            steps {
                sh "docker run --rm -v \$(pwd)/zap-reports:/zap/wrk owasp/zap2docker-stable zap-baseline.py -t https://staging.${DOMAIN} -r report.html"
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh "kubectl set image deployment/${IMAGE_NAME} app=${HARBOR_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
    }

    post {
        failure {
            // Notify via webhook (see Section 26 of main doc)
            sh "curl -X POST ${WEBHOOK_URL} -H 'Content-Type: application/json' -d '{\"text\":\"Build #${BUILD_NUMBER} failed\"}'"
        }
    }
}
```

---

## 7. Artifact & Container Management

### 7.1 Nexus Repository — Artifact Cache & Registry

| Property | Value |
|----------|-------|
| **Role** | Proxy/cache npm, Maven, PyPI, Docker registries; host private packages |
| **Port** | 8081 |
| **Key Features** | Repository groups (proxy + hosted), storage cleanup policies, LDAP auth |
| **Use Cases** | Cache npm packages (faster builds), host private npm packages, proxy Docker Hub |

```yaml
nexus:
  image: sonatype/nexus3:latest
  volumes:
    - nexus_data:/nexus-data
  ports:
    - "8081:8081"
  deploy:
    resources:
      limits:
        memory: 2G
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.nexus.rule=Host(`nexus.${DOMAIN}`)"
    - "traefik.http.routers.nexus.tls.certresolver=letsencrypt"
```

#### npm Configuration for Nexus Proxy

```bash
# .npmrc — point to Nexus proxy for faster, cached installs
registry=https://nexus.${DOMAIN}/repository/npm-group/
//nexus.${DOMAIN}/repository/npm-group/:_authToken=${NEXUS_NPM_TOKEN}
```

### 7.2 Harbor — Container Registry

| Property | Value |
|----------|-------|
| **Role** | Private Docker/OCI image registry with built-in vulnerability scanning |
| **Port** | 443 (HTTPS) |
| **Key Features** | Image signing (Notary/Cosign), replication, RBAC, garbage collection, Trivy integration |
| **Use Cases** | Store production images, enforce scan-before-deploy policies, replicate to DR site |

```yaml
# Harbor uses its own docker-compose — install via official installer
# https://goharbor.io/docs/latest/install-config/
#
# Key configuration in harbor.yml:
#   hostname: harbor.${DOMAIN}
#   https.certificate: /certs/fullchain.pem
#   https.private_key: /certs/privkey.pem
#   database.password: ${HARBOR_DB_PASSWORD}
#   trivy.enabled: true       # Built-in Trivy scanning
```

---

## 8. Security Scanning & Compliance

### 8.1 SonarQube — Static Analysis (SAST)

| Property | Value |
|----------|-------|
| **Role** | Code quality, security vulnerability detection, technical debt tracking |
| **Port** | 9000 |
| **Key Features** | 5000+ rules, quality gates, branch analysis, PR decoration |
| **Languages** | TypeScript, JavaScript, Java, Python, Go, Rust, C/C++, and 30+ more |
| **Integration** | Jenkins pipeline stage (see Section 6.2); blocks merge on quality gate failure |

```yaml
sonarqube:
  image: sonarqube:lts-community
  environment:
    SONAR_JDBC_URL: jdbc:postgresql://postgresql:5432/${SONAR_DB_NAME}
    SONAR_JDBC_USERNAME: ${DB_USER}
    SONAR_JDBC_PASSWORD: ${DB_PASSWORD}
  volumes:
    - sonarqube_data:/opt/sonarqube/data
    - sonarqube_extensions:/opt/sonarqube/extensions
  ports:
    - "9000:9000"
  depends_on:
    postgresql:
      condition: service_healthy
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.sonar.rule=Host(`sonar.${DOMAIN}`)"
    - "traefik.http.routers.sonar.tls.certresolver=letsencrypt"
```

### 8.2 Trivy — Container & IaC Scanning

| Property | Value |
|----------|-------|
| **Role** | Vulnerability scanner for container images, filesystems, IaC, and SBOM |
| **Type** | CLI tool (no persistent service needed) |
| **Key Features** | CVE database, misconfiguration detection, license scanning, SBOM generation |
| **Integration** | Jenkins pipeline stage + Harbor built-in scanner |

```bash
# Scan a container image before pushing to Harbor
trivy image --severity HIGH,CRITICAL --exit-code 1 harbor.${DOMAIN}/${IMAGE_NAME}:${TAG}

# Scan filesystem for vulnerabilities in dependencies
trivy fs --security-checks vuln,config .

# Generate SBOM
trivy image --format spdx-json -o sbom.json harbor.${DOMAIN}/${IMAGE_NAME}:${TAG}
```

### 8.3 OWASP ZAP — Dynamic Security Testing (DAST)

| Property | Value |
|----------|-------|
| **Role** | Automated security testing against running applications |
| **Type** | On-demand (run in CI/CD or manually) |
| **Key Features** | Spider, active/passive scanning, API scanning (OpenAPI), CI integration |
| **Integration** | Jenkins pipeline stage (see Section 6.2); runs against staging environment |

```bash
# Baseline scan (passive, ~2 min)
docker run --rm owasp/zap2docker-stable zap-baseline.py \
  -t https://staging.${DOMAIN} \
  -r zap-baseline-report.html

# Full scan (active, ~15-30 min)
docker run --rm owasp/zap2docker-stable zap-full-scan.py \
  -t https://staging.${DOMAIN} \
  -r zap-full-report.html

# API scan against OpenAPI spec
docker run --rm owasp/zap2docker-stable zap-api-scan.py \
  -t https://staging.${DOMAIN}/api/docs/openapi.json \
  -f openapi \
  -r zap-api-report.html
```

---

## 9. Container Orchestration

### 9.1 Kubernetes (K8s) — Production Orchestration

| Property | Value |
|----------|-------|
| **Role** | Container orchestration, auto-scaling, self-healing, rolling deployments |
| **Distributions** | K3s (lightweight), K8s (full), MicroK8s |
| **Ingress** | Traefik (default in K3s) or Nginx Ingress Controller |
| **Key Features** | Namespaces per tenant, HPA auto-scaling, network policies, RBAC |

#### Namespace Strategy

```
k8s-cluster/
├── namespace: superadmin-prod     # Production workloads
├── namespace: superadmin-staging  # Staging environment
├── namespace: monitoring          # Grafana, Prometheus
├── namespace: cicd                # Jenkins agents
├── namespace: security            # Vault, Keycloak
└── namespace: data                # PostgreSQL, MongoDB, ClickHouse, Redis
```

#### Deployment Manifest Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: superadmin-api
  namespace: superadmin-prod
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: superadmin-api
  template:
    metadata:
      labels:
        app: superadmin-api
    spec:
      containers:
        - name: api
          image: harbor.${DOMAIN}/superadmin/api:latest
          ports:
            - containerPort: 3000
          env:
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: db-credentials
                  key: url
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "512Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /api/health
              port: 3000
            initialDelaySeconds: 15
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /api/health/ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: superadmin-api-hpa
  namespace: superadmin-prod
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: superadmin-api
  minReplicas: 3
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

## 10. Monitoring & Observability

### 10.1 Prometheus — Metrics Collection

| Property | Value |
|----------|-------|
| **Role** | Time-series metrics collection, alerting rules, service discovery |
| **Port** | 9090 |
| **Key Features** | PromQL, multi-dimensional labels, pull-based scraping, AlertManager |
| **Scrape Targets** | App API (/metrics), PostgreSQL exporter, Redis exporter, Node exporter, K8s |

```yaml
prometheus:
  image: prom/prometheus:v2.53
  volumes:
    - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro
    - ./prometheus/alert-rules.yml:/etc/prometheus/alert-rules.yml:ro
    - prometheus_data:/prometheus
  ports:
    - "9090:9090"
  command:
    - '--config.file=/etc/prometheus/prometheus.yml'
    - '--storage.tsdb.retention.time=30d'
    - '--web.enable-lifecycle'
```

#### prometheus.yml

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

rule_files:
  - "alert-rules.yml"

scrape_configs:
  - job_name: 'superadmin-api'
    static_configs:
      - targets: ['app:3000']
    metrics_path: '/api/metrics'

  - job_name: 'postgresql'
    static_configs:
      - targets: ['postgres-exporter:9187']

  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']

  - job_name: 'node'
    static_configs:
      - targets: ['node-exporter:9100']

  - job_name: 'traefik'
    static_configs:
      - targets: ['traefik:8080']
    metrics_path: '/metrics'

  - job_name: 'jenkins'
    static_configs:
      - targets: ['jenkins:8082']
    metrics_path: '/prometheus'
```

### 10.2 Grafana — Dashboards & Visualization

| Property | Value |
|----------|-------|
| **Role** | Metrics visualization, log exploration, alerting, dashboards |
| **Port** | 3000 |
| **Data Sources** | Prometheus (metrics), ClickHouse (analytics), PostgreSQL (app data) |
| **Key Dashboards** | API Performance, Security Events, Tenant Usage, Infrastructure Health |

```yaml
grafana:
  image: grafana/grafana:11
  environment:
    GF_SECURITY_ADMIN_USER: ${GRAFANA_ADMIN_USER}
    GF_SECURITY_ADMIN_PASSWORD: ${GRAFANA_ADMIN_PASSWORD}
    GF_SERVER_ROOT_URL: https://grafana.${DOMAIN}
  volumes:
    - grafana_data:/var/lib/grafana
    - ./grafana/provisioning:/etc/grafana/provisioning
    - ./grafana/dashboards:/var/lib/grafana/dashboards
  ports:
    - "3000:3000"
  depends_on:
    - prometheus
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.grafana.rule=Host(`grafana.${DOMAIN}`)"
    - "traefik.http.routers.grafana.tls.certresolver=letsencrypt"
```

#### Recommended Dashboards

| Dashboard | Data Source | Purpose |
|-----------|------------|---------|
| API Response Times (p50/p95/p99) | Prometheus | Track API latency SLOs |
| Security Event Heatmap | ClickHouse | Visualize failed logins, threat detections |
| Tenant Resource Usage | Prometheus + PostgreSQL | Per-tenant CPU/memory/storage usage |
| Node.js Event Loop Delay | Prometheus | Detect event loop blocking |
| PostgreSQL Performance | Prometheus (pg_exporter) | Slow queries, connections, cache hit ratio |
| Redis/Valkey Metrics | Prometheus (redis_exporter) | Memory, hit rate, connected clients |
| Kubernetes Cluster | Prometheus (kube-state-metrics) | Pod health, resource utilization |
| CI/CD Pipeline | Jenkins + Prometheus | Build duration, failure rate |

---

## 11. Object Storage

### 11.1 MinIO — S3-Compatible Object Storage

| Property | Value |
|----------|-------|
| **Role** | Self-hosted S3-compatible storage for uploads, backups, artifacts |
| **Ports** | `{STORAGE_PORT}` (API: 9000), 9001 (Console) |
| **Key Features** | S3 API compatible, erasure coding, bucket policies, lifecycle rules |
| **Use Cases** | File uploads, database backups (7-day retention), build artifacts, log archival |

```yaml
minio:
  image: minio/minio:latest
  command: server /data --console-address ":9001"
  environment:
    MINIO_ROOT_USER: ${MINIO_ACCESS_KEY}
    MINIO_ROOT_PASSWORD: ${MINIO_SECRET_KEY}
  volumes:
    - minio_data:/data
  ports:
    - "${STORAGE_PORT:-9000}:9000"
    - "9001:9001"
  healthcheck:
    test: ["CMD", "mc", "ready", "local"]
    interval: 10s
    timeout: 5s
    retries: 5
  labels:
    - "traefik.enable=true"
    - "traefik.http.routers.minio.rule=Host(`s3.${DOMAIN}`)"
    - "traefik.http.routers.minio.tls.certresolver=letsencrypt"
    - "traefik.http.routers.minio.service=minio"
    - "traefik.http.services.minio.loadbalancer.server.port=9000"
```

---

## 12. End-to-End & Browser Testing

### 12.1 Playwright — E2E Testing Framework

| Property | Value |
|----------|-------|
| **Role** | End-to-end testing across Chromium, Firefox, WebKit |
| **Key Features** | Auto-wait, network interception, visual comparisons, trace viewer, codegen |
| **Use Cases** | Login flows, RBAC permission testing, multi-tenant isolation verification, dashboard rendering |
| **CI Integration** | Jenkins pipeline stage with HTML report artifact |

```json
// playwright.config.ts (key settings)
{
  "testDir": "./e2e",
  "timeout": 30000,
  "retries": 2,
  "use": {
    "baseURL": "https://staging.${DOMAIN}",
    "screenshot": "only-on-failure",
    "trace": "on-first-retry"
  },
  "projects": [
    { "name": "chromium", "use": { "browserName": "chromium" } },
    { "name": "firefox", "use": { "browserName": "firefox" } },
    { "name": "webkit", "use": { "browserName": "webkit" } }
  ]
}
```

```bash
# Run all E2E tests
npx playwright test

# Run with visual UI
npx playwright test --ui

# Generate test from browser recording
npx playwright codegen https://staging.${DOMAIN}
```

### 12.2 Selenium Grid — Cross-Browser Testing at Scale

| Property | Value |
|----------|-------|
| **Role** | Distributed browser testing grid for parallel cross-browser execution |
| **Ports** | 4444 (Hub), 7900 (VNC) |
| **Key Features** | Parallel execution, Docker-based scaling, video recording, VNC access |
| **Use Cases** | Legacy browser testing, large test suites needing parallelism, visual regression |
| **When to use over Playwright** | When you need true browser instances at scale or legacy browser support (IE/old Edge) |

```yaml
selenium-hub:
  image: selenium/hub:4
  ports:
    - "4444:4444"

selenium-chrome:
  image: selenium/node-chrome:4
  environment:
    SE_EVENT_BUS_HOST: selenium-hub
    SE_EVENT_BUS_PUBLISH_PORT: 4442
    SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
    SE_NODE_MAX_SESSIONS: 4
  depends_on:
    - selenium-hub
  deploy:
    replicas: 2

selenium-firefox:
  image: selenium/node-firefox:4
  environment:
    SE_EVENT_BUS_HOST: selenium-hub
    SE_EVENT_BUS_PUBLISH_PORT: 4442
    SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
    SE_NODE_MAX_SESSIONS: 4
  depends_on:
    - selenium-hub
  deploy:
    replicas: 2
```

> **Recommendation:** Use **Playwright** as the primary E2E framework (faster, modern API, built-in multi-browser). Use **Selenium Grid** when you need to scale to 10+ parallel browser sessions or test against specific browser versions.

---

## 13. Application Runtimes

### 13.1 Node.js — Primary Runtime

| Property | Value |
|----------|-------|
| **Role** | Primary application runtime for the SuperAdmin API and backend services |
| **Version** | 22 LTS (recommended) |
| **Key Features** | Mature ecosystem, TypeScript support, Prisma/Drizzle ORM, extensive npm packages |
| **Use Cases** | API server, background workers, WebSocket handlers, CLI tools |

### 13.2 Bun — High-Performance Alternative Runtime

| Property | Value |
|----------|-------|
| **Role** | Drop-in Node.js replacement with faster startup, built-in bundler, native TypeScript |
| **Version** | 1.x |
| **Key Features** | 4x faster startup, built-in test runner, native SQLite, npm compatible |
| **Use Cases** | Development tooling, scripts, lightweight microservices, serverless functions |
| **Compatibility** | Runs most Node.js code; check Prisma/ORM compatibility before switching |

### 13.3 Deno — Secure-by-Default Runtime

| Property | Value |
|----------|-------|
| **Role** | Sandboxed runtime with built-in security permissions model |
| **Version** | 2.x |
| **Key Features** | Permission-based security, built-in formatter/linter, npm compatibility, TypeScript native |
| **Use Cases** | Security-sensitive microservices, edge functions, isolated plugin execution |
| **When to use** | When the security sandbox model adds value (e.g., running tenant-provided code) |

### 13.4 Rust — Performance-Critical Services

| Property | Value |
|----------|-------|
| **Role** | High-performance microservices where Node.js throughput is insufficient |
| **Key Features** | Zero-cost abstractions, memory safety, no garbage collector, 10-100x faster than Node.js for CPU-bound work |
| **Use Cases** | Real-time threat detection engine, log ingestion pipeline, encryption services, file processing |
| **Integration** | Deploy as separate microservice; communicate via gRPC or REST API |
| **Frameworks** | Actix-web, Axum, Rocket |

```dockerfile
# Multi-stage Rust build
FROM rust:1.79-alpine AS builder
WORKDIR /app
COPY Cargo.toml Cargo.lock ./
COPY src ./src
RUN cargo build --release

FROM alpine:3.20
COPY --from=builder /app/target/release/threat-engine /usr/local/bin/
EXPOSE 8090
CMD ["threat-engine"]
```

> **Runtime Decision Matrix:**
>
> | Criteria | Node.js | Bun | Deno | Rust |
> |----------|---------|-----|------|------|
> | Ecosystem maturity | Best | Good | Good | Different |
> | Startup speed | Moderate | Fastest | Fast | Fast (compiled) |
> | Raw performance | Good | Better | Good | Best |
> | TypeScript support | Via tsc/tsx | Native | Native | N/A |
> | Security sandbox | No | No | Yes | Memory-safe |
> | Use for SuperAdmin | Primary | Dev tooling | Sandbox tasks | Hot paths |

---

## 14. Docker Compose Reference

Complete `docker-compose.yml` combining all services:

```yaml
version: "3.9"

services:
  # --- Reverse Proxy ---
  traefik:
    image: traefik:v3
    restart: unless-stopped
    ports: ["80:80", "443:443", "8080:8080"]
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - traefik_certs:/letsencrypt

  # --- Databases ---
  postgresql:
    image: postgres:17-alpine
    restart: unless-stopped
    ports: ["5432:5432"]
    volumes: [pg_data:/var/lib/postgresql/data]

  mongodb:
    image: mongo:7
    restart: unless-stopped
    ports: ["27017:27017"]
    volumes: [mongo_data:/data/db]

  clickhouse:
    image: clickhouse/clickhouse-server:24
    restart: unless-stopped
    ports: ["8123:8123"]
    volumes: [clickhouse_data:/var/lib/clickhouse]

  # --- Cache ---
  valkey:
    image: valkey/valkey:8-alpine
    restart: unless-stopped
    ports: ["6379:6379"]
    volumes: [valkey_data:/data]

  # --- Auth & Secrets ---
  keycloak:
    image: quay.io/keycloak/keycloak:25
    restart: unless-stopped
    ports: ["8180:8080"]
    depends_on: [postgresql]

  vault:
    image: hashicorp/vault:1.17
    restart: unless-stopped
    ports: ["8200:8200"]
    cap_add: [IPC_LOCK]
    volumes: [vault_data:/vault/data]

  # --- CI/CD ---
  gitea:
    image: gitea/gitea:1.22
    restart: unless-stopped
    ports: ["3001:3000", "2222:22"]
    volumes: [gitea_data:/data]
    depends_on: [postgresql]

  jenkins:
    image: jenkins/jenkins:lts-jdk21
    restart: unless-stopped
    ports: ["8082:8080", "50000:50000"]
    volumes: [jenkins_data:/var/jenkins_home]

  # --- Code Quality & Security ---
  sonarqube:
    image: sonarqube:lts-community
    restart: unless-stopped
    ports: ["9000:9000"]
    depends_on: [postgresql]
    volumes: [sonarqube_data:/opt/sonarqube/data]

  nexus:
    image: sonatype/nexus3:latest
    restart: unless-stopped
    ports: ["8081:8081"]
    volumes: [nexus_data:/nexus-data]

  # --- Monitoring ---
  prometheus:
    image: prom/prometheus:v2.53
    restart: unless-stopped
    ports: ["9090:9090"]
    volumes: [prometheus_data:/prometheus]

  grafana:
    image: grafana/grafana:11
    restart: unless-stopped
    ports: ["3000:3000"]
    volumes: [grafana_data:/var/lib/grafana]
    depends_on: [prometheus]

  # --- Storage ---
  minio:
    image: minio/minio:latest
    restart: unless-stopped
    ports: ["9000:9000", "9001:9001"]
    volumes: [minio_data:/data]
    command: server /data --console-address ":9001"

  # --- Testing (on-demand) ---
  selenium-hub:
    image: selenium/hub:4
    ports: ["4444:4444"]
    profiles: ["testing"]

  selenium-chrome:
    image: selenium/node-chrome:4
    depends_on: [selenium-hub]
    profiles: ["testing"]
    deploy:
      replicas: 2

volumes:
  traefik_certs:
  pg_data:
  mongo_data:
  clickhouse_data:
  valkey_data:
  vault_data:
  gitea_data:
  jenkins_data:
  sonarqube_data:
  nexus_data:
  prometheus_data:
  grafana_data:
  minio_data:
```

---

## 15. Deployment Order & Dependencies

Services must be started in dependency order. Here is the recommended sequence:

```
Phase 1: Foundation (no dependencies)
  ├── PostgreSQL
  ├── MongoDB
  ├── Valkey/Redis/KeyDB
  ├── ClickHouse
  └── MinIO

Phase 2: Infrastructure (depends on Phase 1)
  ├── Traefik
  ├── Vault / OpenBao
  └── Keycloak (→ PostgreSQL)

Phase 3: Development Tools (depends on Phase 1-2)
  ├── Gitea (→ PostgreSQL)
  ├── SonarQube (→ PostgreSQL)
  ├── Nexus
  └── Harbor

Phase 4: CI/CD (depends on Phase 1-3)
  └── Jenkins (→ Gitea, Harbor, SonarQube)

Phase 5: Application (depends on Phase 1-2)
  └── SuperAdmin API (→ PostgreSQL, Valkey, Vault, Keycloak)

Phase 6: Monitoring (depends on Phase 5)
  ├── Prometheus (→ all services)
  └── Grafana (→ Prometheus, ClickHouse)

Phase 7: Testing (on-demand)
  ├── Selenium Grid
  ├── Playwright (npm package, not a service)
  ├── OWASP ZAP (Docker, on-demand)
  └── Trivy (CLI, on-demand)
```

---

## 16. Port Reference

| Port | Service | Protocol |
|------|---------|----------|
| 80 | Traefik (HTTP) | HTTP |
| 443 | Traefik (HTTPS) / Harbor | HTTPS |
| 2222 | Gitea SSH | SSH |
| 3000 | Grafana | HTTP |
| 3001 | Gitea | HTTP |
| 4444 | Selenium Hub | HTTP |
| 5432 | PostgreSQL | TCP |
| 6379 | Valkey / Redis | TCP |
| 6380 | KeyDB | TCP |
| 6081 | Varnish | HTTP |
| 8080 | Traefik Dashboard | HTTP |
| 8081 | Nexus Repository | HTTP |
| 8082 | Jenkins | HTTP |
| 8123 | ClickHouse (HTTP) | HTTP |
| 8180 | Keycloak | HTTP |
| 8200 | Vault / OpenBao | HTTP |
| 9000 | MinIO API / SonarQube / ClickHouse (native) | HTTP/TCP |
| 9001 | MinIO Console | HTTP |
| 9090 | Prometheus | HTTP |
| 27017 | MongoDB | TCP |
| 50000 | Jenkins Agent | TCP |

---

## 17. Placeholder Reference

All infrastructure-specific placeholders used in this document:

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `${DOMAIN}` | Base domain for all services | `example.com` |
| `${DB_NAME}` | PostgreSQL database name | `superadmin_db` |
| `${DB_USER}` | PostgreSQL username | `superadmin` |
| `${DB_PASSWORD}` | PostgreSQL password | (from Vault) |
| `${DB_PORT}` | PostgreSQL port | `5432` |
| `${CACHE_PORT}` | Valkey/Redis port | `6379` |
| `${CACHE_PASSWORD}` | Valkey/Redis password | (from Vault) |
| `${STORAGE_PORT}` | MinIO API port | `9000` |
| `${MINIO_ACCESS_KEY}` | MinIO root access key | (from Vault) |
| `${MINIO_SECRET_KEY}` | MinIO root secret key | (from Vault) |
| `${MONGO_USER}` | MongoDB admin username | `admin` |
| `${MONGO_PASSWORD}` | MongoDB admin password | (from Vault) |
| `${KC_ADMIN_USER}` | Keycloak admin username | `admin` |
| `${KC_ADMIN_PASSWORD}` | Keycloak admin password | (from Vault) |
| `${KC_DB_NAME}` | Keycloak database name | `keycloak_db` |
| `${GITEA_DB_NAME}` | Gitea database name | `gitea_db` |
| `${SONAR_DB_NAME}` | SonarQube database name | `sonarqube_db` |
| `${HARBOR_DB_PASSWORD}` | Harbor database password | (from Vault) |
| `${GRAFANA_ADMIN_USER}` | Grafana admin username | `admin` |
| `${GRAFANA_ADMIN_PASSWORD}` | Grafana admin password | (from Vault) |
| `${ADMIN_EMAIL}` | Admin email (for Let's Encrypt) | `admin@example.com` |
| `${PROJECT_NAME}` | Application/image name | `superadmin` |
| `${WEBHOOK_URL}` | Notification webhook URL | (Slack/Teams URL) |
| `${NEXUS_NPM_TOKEN}` | Nexus npm auth token | (from Vault) |

> **Security Note:** All passwords and secrets should be stored in **Vault/OpenBao** and injected at runtime. Never hardcode secrets in docker-compose files or commit them to Git.

---

## Technology Coverage Summary

| # | Technology | Category | Section | Status |
|---|-----------|----------|---------|--------|
| 1 | **PostgreSQL** | Database | 2.1 | Previously documented + expanded |
| 2 | **MongoDB** | Database | 2.2 | **NEW** |
| 3 | **ClickHouse** | Analytics DB | 2.3 | **NEW** |
| 4 | **Redis / Valkey** | Cache | 3.1 | Previously documented + expanded |
| 5 | **KeyDB** | Cache | 3.2 | **NEW** |
| 6 | **Varnish** | HTTP Cache | 3.3 | **NEW** |
| 7 | **Keycloak** | IAM / SSO | 4.1 | **NEW** |
| 8 | **HashiCorp Vault** | Secrets | 4.2 | Previously documented + expanded |
| 9 | **OpenBao** | Secrets | 4.3 | **NEW** |
| 10 | **Traefik** | Reverse Proxy | 5.1 | **NEW** |
| 11 | **Nginx** | Reverse Proxy | 5.2 | Previously documented + expanded |
| 12 | **Apache** | Reverse Proxy | 5.3 | **NEW** |
| 13 | **Gitea** | Git Repository | 6.1 | **NEW** |
| 14 | **Jenkins** | CI/CD | 6.2 | **NEW** |
| 15 | **Nexus Repository** | Artifacts | 7.1 | **NEW** |
| 16 | **Harbor** | Container Registry | 7.2 | **NEW** |
| 17 | **SonarQube** | SAST | 8.1 | Previously documented + expanded |
| 18 | **Trivy** | Container Scanning | 8.2 | **NEW** |
| 19 | **OWASP ZAP** | DAST | 8.3 | Previously documented + expanded |
| 20 | **Kubernetes** | Orchestration | 9.1 | **NEW** |
| 21 | **Prometheus** | Metrics | 10.1 | Previously documented + expanded |
| 22 | **Grafana** | Dashboards | 10.2 | **NEW** |
| 23 | **MinIO** | Object Storage | 11.1 | Previously documented + expanded |
| 24 | **Playwright** | E2E Testing | 12.1 | **NEW** |
| 25 | **Selenium Grid** | Browser Testing | 12.2 | **NEW** |
| 26 | **Node.js** | Runtime | 13.1 | Previously documented + expanded |
| 27 | **Bun** | Runtime | 13.2 | **NEW** |
| 28 | **Deno** | Runtime | 13.3 | **NEW** |
| 29 | **Rust** | Runtime | 13.4 | **NEW** |

**Previously documented: 11** | **Newly added: 18** | **Total: 29 technologies**

---

**Built for self-hosted, security-first SaaS infrastructure.**
