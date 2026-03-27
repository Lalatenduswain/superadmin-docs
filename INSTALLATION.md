# Installation Guide

> Complete setup instructions for the SuperAdmin SaaS platform — from prerequisites to production deployment.

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Clone and Install](#2-clone-and-install)
3. [Environment Variables](#3-environment-variables)
4. [Database Setup (PostgreSQL)](#4-database-setup-postgresql)
5. [Cache Setup (Redis/Valkey)](#5-cache-setup-redisvalkey)
6. [Object Storage Setup (MinIO)](#6-object-storage-setup-minio)
7. [Identity and Access Management (Keycloak)](#7-identity-and-access-management-keycloak)
8. [First Run and Setup Wizard](#8-first-run-and-setup-wizard)
9. [Docker Compose Quick Start](#9-docker-compose-quick-start)
10. [Kubernetes Deployment](#10-kubernetes-deployment)
11. [Troubleshooting Common Install Issues](#11-troubleshooting-common-install-issues)

---

## 1. Prerequisites

Ensure the following tools and services are available on your development machine or server before proceeding.

### Required Software

| Tool | Minimum Version | Purpose |
|------|----------------|---------|
| Node.js | 24.x LTS | Application runtime |
| npm | 10.x | Package management (ships with Node.js) |
| PostgreSQL | 18.x | Primary relational database |
| Redis or Valkey | 7.x / 8.x | Caching, sessions, rate limiting |
| Docker | 27.x | Containerized services |
| Docker Compose | 2.30+ | Multi-container orchestration |
| Git | 2.40+ | Source control |
| OpenSSL | 3.x | Certificate generation |

### Optional Software

| Tool | Version | Purpose |
|------|---------|---------|
| Bun | 1.x | Alternative runtime (faster installs) |
| kubectl | 1.31+ | Kubernetes cluster management |
| Helm | 3.16+ | Kubernetes package management |
| MinIO Client (mc) | Latest | Object storage administration |
| Keycloak | 26.x | External IAM provider |

### Hardware Requirements

| Environment | CPU | RAM | Disk |
|------------|-----|-----|------|
| Development | 2 cores | 4 GB | 20 GB |
| Staging | 4 cores | 8 GB | 50 GB |
| Production | 8+ cores | 16+ GB | 100+ GB SSD |

### Verify Prerequisites

```bash
# Check Node.js
node --version
# Expected: v24.x.x

# Check npm
npm --version
# Expected: 10.x.x

# Check PostgreSQL
psql --version
# Expected: psql (PostgreSQL) 18.x

# Check Redis
redis-server --version
# Expected: Redis server v=7.x or Valkey v=8.x

# Check Docker
docker --version && docker compose version
# Expected: Docker version 27.x, Docker Compose v2.30+
```

---

## 2. Clone and Install

### Clone the Repository

```bash
git clone https://github.com/{COMPANY_NAME}/superadmin.git
cd superadmin
```

### Install Dependencies

```bash
# Using npm
npm ci

# Or using Bun (alternative)
bun install
```

The `npm ci` command performs a clean install from `package-lock.json`, ensuring deterministic builds across environments.

### Verify Installation

```bash
# Run type checking
npx tsc --noEmit

# Run linting
npm run lint

# Run unit tests
npm run test
```

### Project Structure

```
superadmin/
├── packages/
│   ├── api/              # Backend API (Node.js/TypeScript)
│   │   ├── src/
│   │   │   ├── controllers/
│   │   │   ├── middleware/
│   │   │   ├── models/
│   │   │   ├── routes/
│   │   │   ├── services/
│   │   │   ├── utils/
│   │   │   └── index.ts
│   │   └── package.json
│   └── shared/           # Shared types and utilities
├── apps/
│   └── web/              # Frontend application
├── migrations/           # Database migrations
├── docker/               # Docker configurations
├── k8s/                  # Kubernetes manifests
├── .env.example          # Environment variable template
├── docker-compose.yml    # Development services
└── package.json          # Root workspace
```

---

## 3. Environment Variables

### Create Your Environment File

```bash
cp .env.example .env
```

### Required Variables

Edit `.env` and configure the following groups:

#### Application

```env
# Application
NODE_ENV=development
APP_PORT=3000
APP_NAME=SuperAdmin
APP_URL=http://localhost:3000
APP_SECRET=<generate-with-openssl-rand-hex-64>
```

#### Database

```env
# PostgreSQL
DB_HOST=localhost
DB_PORT=5432
DB_NAME=superadmin_db
DB_USER=superadmin
DB_PASSWORD=<strong-password-here>
DB_SSL=false
DB_POOL_MIN=2
DB_POOL_MAX=20
DB_STATEMENT_TIMEOUT=30000
```

#### Cache

```env
# Redis / Valkey
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=<strong-password-here>
REDIS_DB=0
REDIS_KEY_PREFIX=sa:
REDIS_TLS=false
```

#### Authentication

```env
# JWT
JWT_SECRET=<generate-with-openssl-rand-hex-64>
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=7d
JWT_ISSUER=superadmin
JWT_AUDIENCE=superadmin-api

# MFA
MFA_ISSUER=SuperAdmin
MFA_WINDOW=1
```

#### Object Storage

```env
# MinIO
MINIO_ENDPOINT=localhost
MINIO_PORT=9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=<strong-password-here>
MINIO_BUCKET=superadmin-uploads
MINIO_USE_SSL=false
MINIO_REGION=us-east-1
```

#### Email

```env
# SMTP
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_SECURE=true
SMTP_USER=noreply@example.com
SMTP_PASSWORD=<smtp-password>
SMTP_FROM_NAME=SuperAdmin
SMTP_FROM_EMAIL=noreply@example.com
```

#### Keycloak (Optional)

```env
# Keycloak IAM
KEYCLOAK_ENABLED=false
KEYCLOAK_URL=http://localhost:8080
KEYCLOAK_REALM=superadmin
KEYCLOAK_CLIENT_ID=superadmin-api
KEYCLOAK_CLIENT_SECRET=<client-secret>
KEYCLOAK_ADMIN_USER=admin
KEYCLOAK_ADMIN_PASSWORD=<admin-password>
```

#### Security

```env
# Encryption
ENCRYPTION_KEY=<generate-with-openssl-rand-hex-32>
ENCRYPTION_ALGORITHM=aes-256-gcm

# CORS
CORS_ORIGINS=http://localhost:5173,http://localhost:3000
CORS_CREDENTIALS=true

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100
```

### Generate Secrets

```bash
# Generate JWT secret
openssl rand -hex 64

# Generate encryption key
openssl rand -hex 32

# Generate app secret
openssl rand -hex 64
```

### Environment Variable Validation

The application validates all environment variables at startup using Zod schemas. Missing or invalid variables will produce a clear error message identifying the exact variable and expected format. See Section 39 of `SUPER_ADMIN_DOCUMENTATION.md` for the full validation schema.

---

## 4. Database Setup (PostgreSQL)

### Option A: Native PostgreSQL

#### Install PostgreSQL 18

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install -y postgresql-18 postgresql-client-18

# macOS (Homebrew)
brew install postgresql@18

# Start the service
sudo systemctl enable --now postgresql
# or on macOS:
brew services start postgresql@18
```

#### Create Database and User

```bash
sudo -u postgres psql
```

```sql
-- Create the application user
CREATE ROLE superadmin WITH LOGIN PASSWORD 'your_strong_password';

-- Create the database
CREATE DATABASE superadmin_db OWNER superadmin;

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE superadmin_db TO superadmin;

-- Connect to the database
\c superadmin_db

-- Enable required extensions
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";

-- Exit
\q
```

#### Enable Row-Level Security

Row-Level Security (RLS) is the foundation of multi-tenant data isolation. The migrations will create the RLS policies, but you need to ensure PostgreSQL is configured to support them.

```sql
-- Verify RLS is available (should return 'on')
SHOW row_security;

-- Example RLS policy applied during migration:
-- ALTER TABLE users ENABLE ROW LEVEL SECURITY;
-- CREATE POLICY tenant_isolation ON users
--   USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

#### Configure PostgreSQL for Production

Edit `postgresql.conf`:

```ini
# Connection limits
max_connections = 200
superuser_reserved_connections = 3

# Memory
shared_buffers = 4GB
effective_cache_size = 12GB
work_mem = 64MB
maintenance_work_mem = 1GB

# WAL
wal_level = replica
max_wal_senders = 10
wal_keep_size = 1GB

# Logging
log_statement = 'mod'
log_min_duration_statement = 1000
log_connections = on
log_disconnections = on

# Row-Level Security
row_security = on
```

Edit `pg_hba.conf` to restrict access:

```
# TYPE  DATABASE        USER            ADDRESS          METHOD
local   all             postgres                         peer
host    superadmin_db   superadmin      127.0.0.1/32     scram-sha-256
host    superadmin_db   superadmin      ::1/128          scram-sha-256
```

Restart PostgreSQL:

```bash
sudo systemctl restart postgresql
```

### Option B: Docker PostgreSQL

```bash
docker run -d \
  --name superadmin-postgres \
  -e POSTGRES_USER=superadmin \
  -e POSTGRES_PASSWORD=your_strong_password \
  -e POSTGRES_DB=superadmin_db \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:18-alpine
```

### Run Migrations

```bash
# Run all pending migrations
npm run db:migrate

# Seed initial data (roles, permissions, default admin)
npm run db:seed

# Verify migration status
npm run db:status
```

The migration system creates all tables with tenant scoping, RLS policies, indexes, and the 7-level RBAC hierarchy (SUPER_ADMIN, ADMIN, TENANT_ADMIN, MANAGER, TEAM_LEAD, EMPLOYEE, CUSTOMER). See Section 43 of `SUPER_ADMIN_DOCUMENTATION.md` for the full schema reference.

---

## 5. Cache Setup (Redis/Valkey)

### Option A: Native Redis/Valkey

```bash
# Ubuntu/Debian — Redis
sudo apt update
sudo apt install -y redis-server

# Or Valkey (Redis fork)
# Follow https://valkey.io/download/ for your platform

# Start the service
sudo systemctl enable --now redis-server
```

#### Configure Redis for Production

Edit `/etc/redis/redis.conf`:

```ini
# Binding
bind 127.0.0.1 -::1

# Authentication
requirepass your_strong_redis_password

# Memory management
maxmemory 2gb
maxmemory-policy allkeys-lru

# Persistence
appendonly yes
appendfsync everysec

# Security
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command DEBUG ""
```

Restart Redis:

```bash
sudo systemctl restart redis-server
```

### Option B: Docker Redis/Valkey

```bash
# Redis
docker run -d \
  --name superadmin-redis \
  -p 6379:6379 \
  -v redis-data:/data \
  redis:7-alpine \
  redis-server --requirepass your_strong_redis_password --appendonly yes

# Or Valkey
docker run -d \
  --name superadmin-valkey \
  -p 6379:6379 \
  -v valkey-data:/data \
  valkey/valkey:8-alpine \
  valkey-server --requirepass your_strong_redis_password --appendonly yes
```

### Verify Connection

```bash
redis-cli -a your_strong_redis_password ping
# Expected: PONG
```

### Cache Key Namespaces

The application uses the following key prefixes (configurable via `REDIS_KEY_PREFIX`):

| Prefix | Purpose |
|--------|---------|
| `sa:session:` | User session data |
| `sa:token:` | JWT blacklist (revoked tokens) |
| `sa:rate:` | Rate limiting counters |
| `sa:cache:` | General response cache |
| `sa:lock:` | Distributed locks |
| `sa:mfa:` | MFA challenge state |
| `sa:feature:` | Feature flag cache |

---

## 6. Object Storage Setup (MinIO)

MinIO provides S3-compatible object storage for file uploads, backups, report exports, and audit log archives.

### Option A: Docker (Recommended)

```bash
docker run -d \
  --name superadmin-minio \
  -p 9000:9000 \
  -p 9001:9001 \
  -e MINIO_ROOT_USER=minioadmin \
  -e MINIO_ROOT_PASSWORD=your_strong_minio_password \
  -v minio-data:/data \
  minio/minio server /data --console-address ":9001"
```

### Option B: Native Installation

```bash
# Download MinIO binary
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
sudo mv minio /usr/local/bin/

# Create data directory
sudo mkdir -p /data/minio
sudo chown -R $USER:$USER /data/minio

# Run MinIO
MINIO_ROOT_USER=minioadmin \
MINIO_ROOT_PASSWORD=your_strong_minio_password \
minio server /data/minio --console-address ":9001"
```

### Create Required Buckets

Using the MinIO Client (`mc`):

```bash
# Configure the client alias
mc alias set superadmin http://localhost:9000 minioadmin your_strong_minio_password

# Create buckets
mc mb superadmin/superadmin-uploads
mc mb superadmin/superadmin-backups
mc mb superadmin/superadmin-exports
mc mb superadmin/superadmin-audit-logs

# Set lifecycle policy (auto-delete temp files after 7 days)
mc ilm rule add superadmin/superadmin-uploads \
  --prefix "temp/" \
  --expiry-days 7

# Set bucket policy for uploads (private by default)
mc anonymous set none superadmin/superadmin-uploads
```

### Verify MinIO

Open `http://localhost:9001` in a browser to access the MinIO Console. Log in with the root credentials and verify the buckets were created.

The application integrates MinIO with ClamAV virus scanning for all uploaded files. See Section 46 of `SUPER_ADMIN_DOCUMENTATION.md` for the complete file upload security pipeline.

---

## 7. Identity and Access Management (Keycloak)

Keycloak integration is optional. The application includes a built-in JWT authentication system. Enable Keycloak when you need centralized SSO, LDAP/Active Directory federation, or social login providers.

### Deploy Keycloak

```bash
docker run -d \
  --name superadmin-keycloak \
  -p 8080:8080 \
  -e KC_DB=postgres \
  -e KC_DB_URL=jdbc:postgresql://host.docker.internal:5432/keycloak_db \
  -e KC_DB_USERNAME=keycloak \
  -e KC_DB_PASSWORD=your_keycloak_db_password \
  -e KEYCLOAK_ADMIN=admin \
  -e KEYCLOAK_ADMIN_PASSWORD=your_admin_password \
  quay.io/keycloak/keycloak:26.0 start-dev
```

### Configure the Realm

1. Open `http://localhost:8080` and log in with the admin credentials.
2. Create a new realm named `superadmin`.
3. Create a client:
   - **Client ID:** `superadmin-api`
   - **Client Protocol:** `openid-connect`
   - **Access Type:** `confidential`
   - **Valid Redirect URIs:** `http://localhost:3000/*`
   - **Web Origins:** `http://localhost:3000`
4. Navigate to the **Credentials** tab and copy the client secret into your `.env` file as `KEYCLOAK_CLIENT_SECRET`.

### Configure Role Mapping

Map the 7-level RBAC hierarchy in Keycloak:

1. Go to **Realm Roles** and create roles:
   - `SUPER_ADMIN`
   - `ADMIN`
   - `TENANT_ADMIN`
   - `MANAGER`
   - `TEAM_LEAD`
   - `EMPLOYEE`
   - `CUSTOMER`
2. Create a client scope named `superadmin-roles` with a mapper that includes realm roles in the `roles` claim of the access token.

### Enable in Application

Set the following in `.env`:

```env
KEYCLOAK_ENABLED=true
KEYCLOAK_URL=http://localhost:8080
KEYCLOAK_REALM=superadmin
KEYCLOAK_CLIENT_ID=superadmin-api
KEYCLOAK_CLIENT_SECRET=<your-client-secret>
```

When Keycloak is enabled, the application delegates authentication to Keycloak while retaining local RBAC policy enforcement and audit logging.

---

## 8. First Run and Setup Wizard

### Start the Application

```bash
# Development mode with hot reload
npm run dev

# Or production mode
npm run build && npm start
```

### Setup Wizard

On first launch, the application detects an empty database and automatically presents the 12-step setup wizard. The wizard guides you through:

| Step | Configuration |
|------|--------------|
| 1 | Organization name and branding |
| 2 | Super Admin account creation |
| 3 | Database connection verification |
| 4 | Cache connection verification |
| 5 | SMTP email configuration |
| 6 | Object storage connection |
| 7 | Security policies (password, MFA, session) |
| 8 | Tenant configuration |
| 9 | RBAC default roles and permissions |
| 10 | License tier selection |
| 11 | Feature flags initial state |
| 12 | Review and finalize |

See `SETUP_WIZARD_PROMPT.md` for the complete wizard specification including Zod validation schemas, UI layouts, and API architecture.

### Verify the Installation

After the wizard completes:

```bash
# Health check
curl http://localhost:3000/api/v1/health
# Expected: {"status":"healthy","version":"1.0.0","uptime":...}

# Login with Super Admin credentials created during wizard
curl -X POST http://localhost:3000/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@example.com","password":"your_password","tenantId":"default"}'
```

---

## 9. Docker Compose Quick Start

The fastest way to get the entire stack running locally.

### Full Stack Startup

```bash
# Start all services
docker compose up -d

# View logs
docker compose logs -f app

# Check service health
docker compose ps
```

### docker-compose.yml Overview

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DB_HOST=postgres
      - REDIS_HOST=redis
      - MINIO_ENDPOINT=minio
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      minio:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/api/v1/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: postgres:18-alpine
    environment:
      POSTGRES_USER: superadmin
      POSTGRES_PASSWORD: ${DB_PASSWORD:-changeme}
      POSTGRES_DB: superadmin_db
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
      - ./migrations/init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U superadmin -d superadmin_db"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD:-changeme} --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "-a", "${REDIS_PASSWORD:-changeme}", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5

  minio:
    image: minio/minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ACCESS_KEY:-minioadmin}
      MINIO_ROOT_PASSWORD: ${MINIO_SECRET_KEY:-changeme}
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio-data:/data
    healthcheck:
      test: ["CMD", "mc", "ready", "local"]
      interval: 10s
      timeout: 5s
      retries: 5

  keycloak:
    image: quay.io/keycloak/keycloak:26.0
    command: start-dev
    environment:
      KC_DB: postgres
      KC_DB_URL: jdbc:postgresql://postgres:5432/keycloak_db
      KC_DB_USERNAME: superadmin
      KC_DB_PASSWORD: ${DB_PASSWORD:-changeme}
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: ${KEYCLOAK_ADMIN_PASSWORD:-changeme}
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
    profiles:
      - keycloak

volumes:
  pgdata:
  redis-data:
  minio-data:
```

### Start with Keycloak

```bash
# Include the Keycloak profile
docker compose --profile keycloak up -d
```

### Reset Everything

```bash
# Stop and remove all containers, volumes, and networks
docker compose down -v
```

---

## 10. Kubernetes Deployment

### Namespace Setup

```bash
kubectl create namespace superadmin
kubectl config set-context --current --namespace=superadmin
```

### Deploy with Helm

```bash
# Add the Helm repo (if published)
helm repo add superadmin https://charts.example.com/superadmin
helm repo update

# Install with default values
helm install superadmin superadmin/superadmin \
  --namespace superadmin \
  --values k8s/values-production.yaml

# Or install from local charts
helm install superadmin ./k8s/helm/superadmin \
  --namespace superadmin \
  --values k8s/values-production.yaml
```

### Secrets Management

```bash
# Create database secret
kubectl create secret generic superadmin-db \
  --from-literal=username=superadmin \
  --from-literal=password='your_strong_password' \
  --namespace superadmin

# Create Redis secret
kubectl create secret generic superadmin-redis \
  --from-literal=password='your_strong_redis_password' \
  --namespace superadmin

# Create JWT secret
kubectl create secret generic superadmin-jwt \
  --from-literal=secret="$(openssl rand -hex 64)" \
  --namespace superadmin

# Create encryption key
kubectl create secret generic superadmin-encryption \
  --from-literal=key="$(openssl rand -hex 32)" \
  --namespace superadmin
```

### Verify Deployment

```bash
# Check pod status
kubectl get pods -n superadmin

# Check services
kubectl get svc -n superadmin

# View application logs
kubectl logs -f deployment/superadmin-api -n superadmin

# Port-forward for local access
kubectl port-forward svc/superadmin-api 3000:3000 -n superadmin
```

See `DEPLOYMENT.md` for the complete Kubernetes production deployment guide with Helm chart details, Ingress configuration, and scaling policies.

---

## 11. Troubleshooting Common Install Issues

### Database Connection Refused

**Symptom:** `ECONNREFUSED 127.0.0.1:5432` on startup.

**Solution:**
```bash
# Verify PostgreSQL is running
sudo systemctl status postgresql

# Check it is listening on the correct port
sudo ss -tlnp | grep 5432

# If using Docker, check the container
docker ps | grep postgres
docker logs superadmin-postgres
```

### Redis Authentication Failed

**Symptom:** `NOAUTH Authentication required` or `ERR invalid password`.

**Solution:**
```bash
# Verify the password matches your .env
redis-cli -a your_password ping

# If Redis was started without a password, set one
redis-cli CONFIG SET requirepass "your_password"
```

### Migration Failures

**Symptom:** `relation "users" already exists` or `migration lock timeout`.

**Solution:**
```bash
# Check migration status
npm run db:status

# Release stuck migration lock
npm run db:unlock

# Roll back the last migration and retry
npm run db:rollback
npm run db:migrate
```

### MinIO Bucket Not Found

**Symptom:** `The specified bucket does not exist` on file upload.

**Solution:**
```bash
# Create the missing bucket
mc alias set local http://localhost:9000 minioadmin your_password
mc mb local/superadmin-uploads
mc mb local/superadmin-backups
```

### Port Already in Use

**Symptom:** `EADDRINUSE: address already in use :::3000`.

**Solution:**
```bash
# Find the process using the port
lsof -i :3000

# Kill it
kill -9 <PID>

# Or change APP_PORT in .env
```

### TypeScript Compilation Errors

**Symptom:** `Cannot find module` or type errors after install.

**Solution:**
```bash
# Clean and reinstall
rm -rf node_modules package-lock.json
npm install

# Rebuild TypeScript project references
npx tsc --build --clean
npx tsc --build
```

### Docker Build Fails on ARM (Apple Silicon)

**Symptom:** `exec format error` when running the container.

**Solution:**
```bash
# Build with the correct platform
docker build --platform linux/amd64 -t superadmin .

# Or use Docker Compose with platform specification
# Add to your service in docker-compose.yml:
#   platform: linux/amd64
```

### Environment Variable Validation Errors

**Symptom:** `ZodError: Invalid environment configuration` with a list of fields.

**Solution:** Compare your `.env` file against `.env.example` line by line. The error message identifies each invalid or missing variable. Common mistakes include missing quotes around values with special characters and incorrect URL formats.

---

## Next Steps

- Review `SUPER_ADMIN_DOCUMENTATION.md` for the full 50-section implementation guide
- Read `DEPLOYMENT.md` for production deployment and scaling
- Read `API.md` for the complete API reference
- Read `TROUBLESHOOTING.md` for runtime issue resolution
