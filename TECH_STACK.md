# Technology Stack & Dependencies Reference

> **SuperAdmin SaaS Platform** — Complete technology stack, version matrix, and dependency reference.

This document catalogs every technology used across the SuperAdmin platform, organized by category with pinned versions, descriptions, and compatibility notes. Use this as the single source of truth when onboarding, upgrading, or auditing dependencies.

---

## Table of Contents

1. [Backend (Node.js)](#backend-nodejs)
2. [Backend (Python)](#backend-python)
3. [Backend Deployment](#backend-deployment)
4. [Backend Validation](#backend-validation)
5. [Backend Workflow](#backend-workflow)
6. [Caching](#caching)
7. [CI/CD](#cicd)
8. [Code Quality](#code-quality)
9. [Containerization](#containerization)
10. [Database](#database)
11. [Database ORM](#database-orm)
12. [Frontend](#frontend)
13. [Frontend & UI](#frontend--ui)
14. [Frontend Animation](#frontend-animation)
15. [Frontend Data/Routing](#frontend-datarouting)
16. [Frontend/Fullstack](#frontendfullstack)
17. [Messaging](#messaging)
18. [Monitoring & Secrets](#monitoring--secrets)
19. [Object Storage](#object-storage)
20. [Orchestration](#orchestration)
21. [Registry](#registry)
22. [Testing](#testing)
23. [Web Server / SSL](#web-server--ssl)
24. [UX / Onboarding](#ux--onboarding)
25. [Version Update Policy](#version-update-policy)
26. [Compatibility Matrix](#compatibility-matrix)
27. [Package Manager Recommendations](#package-manager-recommendations)
28. [Summary](#summary)

---

## Backend (Node.js)

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Backend (Node.js) | BCrypt | 5.1.x | Secure, battle-tested password hashing library. Uses the Blowfish cipher to produce salted hashes resistant to brute-force and rainbow-table attacks. Industry standard for storing user credentials. |
| Backend (Node.js) | Express | 5.x | Fast, minimalist web framework for Node.js APIs and servers. Provides robust routing, middleware composition, and a vast ecosystem of plugins for building RESTful services. |
| Backend (Node.js) | ioredis | 5.x | High-performance, feature-rich Redis/Valkey client for Node.js. Supports Cluster, Sentinel, Streams, Lua scripting, pipelining, and transparent reconnection out of the box. |
| Backend (Node.js) | Node.js | 25.x (Current) / 24.x (LTS) | JavaScript runtime built on V8 for building scalable server-side applications. Provides the execution environment for all Node.js backend services, CLI tooling, and build pipelines. |

---

## Backend (Python)

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Backend (Python) | FastAPI | 0.115.x+ | High-performance, type-safe Python API framework. Top choice over Flask and Django for async-first microservices, with automatic OpenAPI/Swagger documentation generation and dependency injection. |
| Backend (Python) | Pydantic | 2.12.x | Data validation and settings management for Python. Mandatory companion to FastAPI — enforces runtime type safety, serialization, and configuration parsing using Python type hints. |

---

## Backend Deployment

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Backend Deployment | Cloudflare Workers | N/A (platform) | Quick, cheap, serverless backend and edge deployments. Runs V8 isolates at 300+ global edge locations with sub-millisecond cold starts — ideal for lightweight API endpoints, auth middleware, and edge logic. |

---

## Backend Validation

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Backend Validation | Zod | 4.3.x | TypeScript-first schema validation and parsing library. Mandatory for enforcing type safety at runtime in JavaScript/TypeScript backends and shared API contracts between frontend and backend. |

---

## Backend Workflow

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Backend Workflow | Temporal Server | ~1.28.x+ | Durable, reliable execution engine for distributed systems and complex workflows. Replaces fragile manual queue setups (Redis + RabbitMQ + Celery) with first-class support for retries, saga patterns, long-running processes, and workflow versioning. |

---

## Caching

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Caching | Redis | 8.x | In-memory data structure store used as cache, database, and message broker. Supports strings, hashes, lists, sets, sorted sets, streams, and pub/sub with sub-millisecond latency. |
| Caching | Valkey | 9.x | High-performance open-source Redis fork for caching, queues, and realtime workloads. Community-driven under the Linux Foundation, fully compatible with Redis protocol and clients. |

---

## CI/CD

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| CI/CD | Jenkins | 2.546+ (weekly patches) | Open-source automation server for building, testing, and deploying software. Supports declarative and scripted pipelines, 1,800+ plugins, distributed builds, and integrations with Docker, Harbor, and every major SCM. |

---

## Code Quality

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Code Quality | SonarQube | 2025.6 / 2026.x series | Automated code quality analysis platform. Detects bugs, vulnerabilities, code smells, and technical debt across 30+ languages with quality gates that block merges when thresholds are breached. |

---

## Containerization

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Containerization | Docker | 29.x (Engine) | Platform for developing, shipping, and running applications in isolated containers. Provides reproducible builds, layer caching, multi-stage Dockerfiles, and Compose for local multi-service orchestration. |

---

## Database

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Database | PostgreSQL | 18.1 | Advanced, standards-compliant relational database with strong extensibility. Supports JSONB, full-text search, row-level security, partitioning, logical replication, and a rich extension ecosystem (PostGIS, pgvector, TimescaleDB). |

---

## Database ORM

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Database ORM | @prisma/client | 6.x | Next-generation type-safe ORM for Node.js and TypeScript. Provides auto-generated queries, migrations, an intuitive schema DSL, and a visual Studio interface for database management. |

---

## Frontend

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Frontend | @tanstack/react-query | 5.90.x | Data fetching, caching, mutations, and synchronization for React. Eliminates boilerplate around loading states, error handling, background refetching, and optimistic updates. |
| Frontend | @types/react | 19.x | TypeScript type definitions for React. Provides compile-time type checking for all React APIs, hooks, components, and DOM elements. |
| Frontend | framer-motion | 12.x | Production-ready animation and gesture library for React. Declarative API for transitions, layout animations, scroll-linked effects, drag, and spring physics. |
| Frontend | Prism | 1.30.x | Lightweight syntax highlighter for code blocks and snippets. Supports 300+ languages and themes with a plugin architecture for line numbers, copy buttons, and diff highlighting. |
| Frontend | react-hook-form | 7.71.x | Performant form handling and validation library for React. Minimizes re-renders, integrates with Zod/Yup for schema validation, and supports complex nested and dynamic field arrays. |
| Frontend | Recharts | 3.6.x | Composable charting library built with React and D3 for data visualization. Provides bar, line, area, pie, radar, scatter, and treemap chart types with responsive containers and custom tooltips. |
| Frontend | Tailwind CSS | 4.1.x | Utility-first CSS framework for rapid, customizable styling. Generates only the CSS you use, supports dark mode, responsive design, and arbitrary values with zero-runtime overhead. |
| Frontend | Zustand | 5.x | Lightweight, fast state management for React with no boilerplate. Provides a simple hook-based API, middleware support (persist, devtools, immer), and works seamlessly with React concurrent features. |

---

## Frontend & UI

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Frontend & UI | ShadCN (shadcn/ui) CLI | ~3.7.x | Beautiful, accessible, copy-paste customizable UI components built on Radix primitives and Tailwind CSS. The new standard for UI — components live in your codebase, not in node_modules, giving full ownership and customization. |
| Frontend & UI | Tailwind CSS | 4.1.x | Utility-first CSS framework for rapid, customizable styling. Pairs perfectly with ShadCN for consistent, maintainable design systems without writing custom CSS. |

---

## Frontend Animation

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Frontend Animation | Framer Motion | 12.27.x | Production-ready animations, gestures, and transitions for React. Supports layout animations, shared element transitions, scroll-triggered effects, exit animations via AnimatePresence, and hardware-accelerated spring physics. |

---

## Frontend Data/Routing

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Frontend Data/Routing | TanStack Query | 5.92.x / @tanstack/react-query 5.90.19+ | Powerful async data fetching, caching, mutations, and optimistic updates. Provides automatic background refetching, infinite queries, prefetching, and devtools for debugging cache state. |
| Frontend Data/Routing | TanStack Router | 1.152.x | Fully type-safe router with built-in data loading, caching, and URL state management. Provides search param validation, route-level code splitting, and pending/error UI boundaries out of the box. |

---

## Frontend/Fullstack

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Frontend/Fullstack | Next.js | 16.x (stable) | React framework for SSR, SSG, App Router, Server Actions, and full-stack applications. Provides file-based routing, API routes, middleware, image optimization, and seamless Vercel/self-hosted deployment. |

---

## Messaging

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Messaging | RabbitMQ | 4.2.x | Reliable message broker for queues, pub/sub, and async task processing. Supports AMQP 0-9-1, MQTT, STOMP, streams, quorum queues, dead-letter exchanges, and management UI for monitoring. |

---

## Monitoring & Secrets

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Monitoring & Secrets | HashiCorp Vault | 1.21.x | Secrets management, encryption-as-a-service, and dynamic credentials. Provides centralized secret storage, auto-rotation, PKI, transit encryption, and fine-grained access policies for zero-trust infrastructure. |

---

## Object Storage

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Object Storage | MinIO | RELEASE.2025-xx-xx (monthly) | S3-compatible high-performance object storage server. Supports erasure coding, bitrot protection, bucket notifications, lifecycle policies, and federation — ideal for self-hosted file/media storage. |

---

## Orchestration

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Orchestration | Docker / Jenkins / Harbor | — | Container orchestration (Compose/Swarm), CI/CD pipelines, and image management. Together these tools provide the full build-ship-run lifecycle: Docker builds and runs containers, Jenkins automates pipelines, and Harbor stores and scans images. |

---

## Registry

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Registry | Harbor | 2.14.x | Open-source cloud-native container image registry with vulnerability scanning. Provides RBAC, image signing (Cosign/Notary), replication, garbage collection, and integration with Trivy for CVE scanning. |

---

## Testing

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Testing | Microsoft Playwright | 1.57.x | Fast, reliable end-to-end browser testing and automation across Chromium, Firefox, and WebKit. Supports auto-waiting, network interception, visual comparisons, trace viewer, and parallel test execution. |

---

## Web Server / SSL

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| Web Server / SSL | Certbot (Let's Encrypt) | 5.2.x | Automated tool to obtain, install, and renew free SSL/TLS certificates from Let's Encrypt. Handles domain validation, certificate issuance, Nginx/Apache integration, and cron-based auto-renewal. |
| Web Server / SSL | Nginx | 1.29.x (mainline) | High-performance HTTP server, reverse proxy, load balancer, and SSL terminator. Handles static file serving, upstream proxying, rate limiting, gzip compression, and WebSocket upgrades with minimal resource usage. |

---

## UX / Onboarding

| Category | Technology | Version | Description |
|----------|-----------|---------|-------------|
| UX / Onboarding | Driver.js (driverjs.com) | latest | Lightweight, no-dependency product tour and user onboarding library. Provides step-by-step guided walkthroughs with element highlighting, popover positioning, keyboard navigation, and smooth animated transitions for first-time user experiences. |

---

## Version Update Policy

Keeping dependencies current is critical for security, performance, and compatibility. Follow these guidelines:

### Update Cadence

| Priority | Frequency | Scope | Examples |
|----------|-----------|-------|---------|
| **Critical / Security** | Within 24-48 hours | Patch any CVE rated HIGH or CRITICAL immediately | OpenSSL patches, Express security fixes, Nginx CVEs |
| **Patch Releases** | Weekly | Apply bug fixes and minor patches during sprint maintenance windows | 5.90.x to 5.90.y, 18.0 to 18.1 |
| **Minor Releases** | Bi-weekly to Monthly | Evaluate changelogs, run full test suite, then upgrade | Tailwind 4.0.x to 4.1.x, Prisma 6.0 to 6.1 |
| **Major Releases** | Quarterly evaluation | Create a branch, migrate breaking changes, run full E2E suite before merging | Next.js 15 to 16, Express 4 to 5, Node.js 24 to 25 |

### Update Process

1. **Monitor** — Subscribe to GitHub release feeds, security advisories (e.g., `npm audit`, Snyk, Dependabot) for all listed dependencies.
2. **Evaluate** — Read the changelog and migration guide. Check for breaking changes against your codebase.
3. **Branch** — Create a dedicated `deps/upgrade-<package>` branch for any minor or major upgrade.
4. **Test** — Run unit tests, integration tests, and Playwright E2E tests. Verify SonarQube quality gates pass.
5. **Stage** — Deploy to a staging environment and validate with smoke tests before production.
6. **Merge** — Merge via pull request with at least one reviewer. Tag the deployment.
7. **Document** — Update this file with the new version number and the date of the upgrade.

### Version Pinning Strategy

- **Lock files** (`package-lock.json`, `pnpm-lock.yaml`, `bun.lockb`) must always be committed.
- **Use exact versions** in `package.json` for production dependencies (no `^` or `~`).
- **Use caret ranges** (`^`) only for `devDependencies` where minor updates are unlikely to break builds.
- **Docker images** should pin to specific tags (e.g., `node:24.x-alpine`, `postgres:18.1-alpine`), never `latest`.

---

## Compatibility Matrix

### Frontend Framework to Backend Compatibility

| Frontend | Express 5.x (Node.js) | FastAPI 0.115.x+ (Python) | Cloudflare Workers | Next.js API Routes |
|----------|:-----:|:-----:|:-----:|:-----:|
| **Next.js 16.x** | Yes | Yes | Yes (edge) | Built-in |
| **React + TanStack Router** | Yes | Yes | Yes | N/A |
| **React + TanStack Query** | Yes | Yes | Yes | Yes |

### ORM / Database Compatibility

| ORM / Client | PostgreSQL 18.x | Redis 8.x | Valkey 9.x |
|--------------|:-----:|:-----:|:-----:|
| **@prisma/client 6.x** | Yes | No (use ioredis) | No (use ioredis) |
| **ioredis 5.x** | No | Yes | Yes |

### State Management to Framework Compatibility

| State Manager | Next.js 16.x (App Router) | Next.js 16.x (Pages Router) | React SPA (Vite) |
|---------------|:-----:|:-----:|:-----:|
| **Zustand 5.x** | Yes (client components) | Yes | Yes |
| **TanStack Query 5.x** | Yes (with hydration) | Yes | Yes |

### UI Component to Styling Compatibility

| UI Library | Tailwind CSS 4.1.x | Framer Motion 12.x | react-hook-form 7.x |
|------------|:-----:|:-----:|:-----:|
| **ShadCN (shadcn/ui) ~3.7.x** | Required | Yes | Yes (built-in support) |

### CI/CD to Container Registry Compatibility

| CI/CD Tool | Docker 29.x | Harbor 2.14.x | MinIO (S3) |
|------------|:-----:|:-----:|:-----:|
| **Jenkins 2.546+** | Yes | Yes (push/pull) | Yes (artifact storage) |

### Testing to Browser Engine Support

| Testing Framework | Chromium | Firefox | WebKit (Safari) |
|-------------------|:-----:|:-----:|:-----:|
| **Playwright 1.57.x** | Yes | Yes | Yes |

---

## Package Manager Recommendations

### Comparison

| Feature | npm 11.x | pnpm 10.x | Bun 1.2.x |
|---------|:--------:|:---------:|:---------:|
| **Speed** | Baseline | ~2-3x faster | ~5-10x faster |
| **Disk Efficiency** | Flat node_modules | Content-addressable store (saves 50-70% disk) | Flat node_modules |
| **Monorepo Support** | Workspaces | Workspaces + filtering | Workspaces |
| **Lock File** | `package-lock.json` | `pnpm-lock.yaml` | `bun.lockb` (binary) |
| **Compatibility** | Universal | Strict (better correctness) | Growing (some edge cases) |
| **Native TypeScript** | No | No | Yes (built-in runner) |
| **Bundler Built-in** | No | No | Yes |
| **Maturity** | Production-proven | Production-proven | Rapidly maturing |

### Recommendations by Use Case

| Use Case | Recommended | Rationale |
|----------|-------------|-----------|
| **New greenfield projects** | **pnpm** | Best balance of speed, disk efficiency, and strict correctness. Ideal for monorepos. |
| **Existing projects with npm** | **npm** | No migration cost. Upgrade to npm 11.x for improved performance and security auditing. |
| **Performance-critical CI/CD** | **pnpm** or **Bun** | Dramatically faster install times reduce pipeline duration. pnpm is safer; Bun is faster. |
| **Rapid prototyping / scripts** | **Bun** | Native TypeScript execution, built-in test runner, and fastest install times accelerate iteration. |
| **Enterprise / regulated environments** | **pnpm** | Strictest dependency resolution prevents phantom dependencies. Best audit trail with readable YAML lock file. |

### Installation

```bash
# npm (ships with Node.js)
node -v  # Verify Node.js 24.x / 25.x is installed

# pnpm
corepack enable
corepack prepare pnpm@latest --activate

# Bun
curl -fsSL https://bun.sh/install | bash
```

---

## Summary

### Technology Count by Category

| Category | Count |
|----------|:-----:|
| Backend (Node.js) | 4 |
| Backend (Python) | 2 |
| Backend Deployment | 1 |
| Backend Validation | 1 |
| Backend Workflow | 1 |
| Caching | 2 |
| CI/CD | 1 |
| Code Quality | 1 |
| Containerization | 1 |
| Database | 1 |
| Database ORM | 1 |
| Frontend | 8 |
| Frontend & UI | 2 |
| Frontend Animation | 1 |
| Frontend Data/Routing | 2 |
| Frontend/Fullstack | 1 |
| Messaging | 1 |
| Monitoring & Secrets | 1 |
| Object Storage | 1 |
| Orchestration | 1 |
| Registry | 1 |
| Testing | 1 |
| Web Server / SSL | 2 |
| UX / Onboarding | 1 |
| **Total** | **39** |

### Stack at a Glance

- **Languages**: JavaScript/TypeScript (Node.js 25.x/24.x), Python (FastAPI)
- **Frontend**: Next.js 16.x, React, ShadCN, Tailwind CSS 4.1.x, Zustand, TanStack Query/Router
- **Backend**: Express 5.x, FastAPI 0.115.x+, Temporal Server, Zod 4.3.x
- **Data**: PostgreSQL 18.1, Redis 8.x / Valkey 9.x, Prisma 6.x
- **Infrastructure**: Docker 29.x, Jenkins, Harbor, Nginx, MinIO, Cloudflare Workers
- **Security**: BCrypt, HashiCorp Vault, Certbot, SonarQube
- **Testing**: Playwright 1.57.x (Chromium, Firefox, WebKit)

---

*Last updated: 2026-03-27*
