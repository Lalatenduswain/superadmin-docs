# Testing Guide

Comprehensive testing strategy for the SuperAdmin SaaS platform. This document covers the testing philosophy, tooling, test types, coverage requirements, and CI/CD integration.

---

## Table of Contents

1. [Testing Philosophy](#testing-philosophy)
2. [Unit Testing](#unit-testing)
3. [Integration Testing](#integration-testing)
4. [E2E Testing with Playwright](#e2e-testing-with-playwright)
5. [E2E Testing with Selenium Grid](#e2e-testing-with-selenium-grid)
6. [API Testing](#api-testing)
7. [Security Testing](#security-testing)
8. [Performance and Load Testing](#performance-and-load-testing)
9. [Coverage Requirements](#coverage-requirements)
10. [CI Pipeline Integration](#ci-pipeline-integration)
11. [Test Data Management](#test-data-management)
12. [Mocking Strategy](#mocking-strategy)

---

## Testing Philosophy

### The Test Pyramid

The SuperAdmin platform follows the **test pyramid** model. The majority of tests are fast, isolated unit tests. Integration tests verify component interactions with real dependencies. End-to-end tests cover critical user journeys.

```
        ╱  E2E  ╲           ~10% of tests
       ╱──────────╲         Slow, expensive, high confidence
      ╱ Integration ╲       ~20% of tests
     ╱────────────────╲     Moderate speed, real dependencies
    ╱    Unit Tests     ╲   ~70% of tests
   ╱──────────────────────╲ Fast, isolated, cheap to run
```

### Guiding Principles

1. **Test behavior, not implementation**: Tests verify what the system does, not how it does it. Refactoring should not break tests.
2. **Real dependencies over mocks for integration tests**: Integration tests use a real PostgreSQL database, real Redis instance, and real message queues. Mocks are reserved for external third-party services.
3. **Every bug gets a regression test**: When a bug is fixed, a test is written first to reproduce it. That test must fail before the fix and pass after.
4. **Tests are first-class code**: Tests follow the same code quality standards as production code — linted, reviewed, and refactored.
5. **Fast feedback**: Unit tests complete in under 30 seconds. The full CI pipeline completes in under 15 minutes.
6. **Deterministic**: Tests produce the same result every time. No flaky tests in the main branch — flaky tests are quarantined and fixed within 48 hours.
7. **Tenant isolation is always tested**: Multi-tenancy is a security boundary. Every data-access test verifies that Tenant A cannot see Tenant B's data.

---

## Unit Testing

### Tooling

- **Framework**: Vitest (primary) — compatible with Jest API, native ESM support, fast execution via esbuild.
- **Assertion library**: Vitest built-in `expect` (Jest-compatible).
- **Mocking**: Vitest `vi.mock()`, `vi.spyOn()`, `vi.fn()`.

### Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import path from 'path';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    include: ['src/**/*.test.ts', 'src/**/*.spec.ts'],
    exclude: ['src/**/*.e2e.ts', 'src/**/*.integration.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html', 'lcov'],
      thresholds: {
        branches: 80,
        functions: 80,
        lines: 80,
        statements: 80,
      },
      exclude: [
        'node_modules/',
        'src/**/*.d.ts',
        'src/**/*.test.ts',
        'src/**/index.ts',
        'src/types/',
      ],
    },
    setupFiles: ['./src/test/setup.ts'],
  },
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
});
```

### Running Unit Tests

```bash
# Run all unit tests
pnpm test

# Run in watch mode during development
pnpm test:watch

# Run with coverage
pnpm test:coverage

# Run a specific file
pnpm test src/services/auth.test.ts

# Run tests matching a pattern
pnpm test --grep "RBAC"
```

### Unit Test Examples

```typescript
// src/services/auth.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { AuthService } from './auth.service';
import { hashPassword, verifyPassword } from '../utils/crypto';

vi.mock('../utils/crypto');

describe('AuthService', () => {
  let authService: AuthService;

  beforeEach(() => {
    authService = new AuthService();
    vi.clearAllMocks();
  });

  describe('validatePasswordStrength', () => {
    it('rejects passwords shorter than 12 characters', () => {
      const result = authService.validatePasswordStrength('Short1!abc');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('Password must be at least 12 characters');
    });

    it('rejects passwords without uppercase letters', () => {
      const result = authService.validatePasswordStrength('alllowercase123!');
      expect(result.valid).toBe(false);
      expect(result.errors).toContain('Password must contain at least one uppercase letter');
    });

    it('accepts valid passwords', () => {
      const result = authService.validatePasswordStrength('SecureP@ssw0rd2026!');
      expect(result.valid).toBe(true);
      expect(result.errors).toHaveLength(0);
    });
  });

  describe('generateSessionToken', () => {
    it('returns a 64-character hex string', () => {
      const token = authService.generateSessionToken();
      expect(token).toMatch(/^[a-f0-9]{64}$/);
    });

    it('generates unique tokens on each call', () => {
      const token1 = authService.generateSessionToken();
      const token2 = authService.generateSessionToken();
      expect(token1).not.toBe(token2);
    });
  });
});
```

```typescript
// src/middleware/tenant-isolation.test.ts
import { describe, it, expect } from 'vitest';
import { enforceTenantScope } from './tenant-isolation';

describe('Tenant Isolation Middleware', () => {
  it('injects tenant_id into all database queries', () => {
    const query = enforceTenantScope(
      { table: 'users', where: { active: true } },
      'tenant_abc123'
    );
    expect(query.where).toHaveProperty('tenant_id', 'tenant_abc123');
  });

  it('rejects queries without a tenant context', () => {
    expect(() =>
      enforceTenantScope({ table: 'users', where: {} }, undefined)
    ).toThrow('Tenant context is required');
  });

  it('prevents tenant_id from being overridden by user input', () => {
    const query = enforceTenantScope(
      { table: 'users', where: { tenant_id: 'malicious_tenant', active: true } },
      'tenant_abc123'
    );
    expect(query.where.tenant_id).toBe('tenant_abc123');
  });
});
```

### What to Unit Test

- **Business logic**: Validation rules, calculations, state machines, permission checks.
- **Utility functions**: Crypto helpers, date formatting, data transformations.
- **Middleware**: Request parsing, tenant scoping, error handling.
- **Domain models**: Entity creation, validation, serialization.

### What NOT to Unit Test

- Database queries (use integration tests).
- HTTP request/response cycles (use API tests).
- UI rendering and user flows (use E2E tests).
- Third-party library internals.

---

## Integration Testing

### Philosophy

Integration tests verify that components work together correctly with **real dependencies**. We use a real PostgreSQL database, a real Redis instance, and real message queues — not mocks.

### Test Infrastructure

```yaml
# docker-compose.test.yml
services:
  postgres-test:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: superadmin_test
      POSTGRES_USER: test_user
      POSTGRES_PASSWORD: test_password
    ports:
      - "5433:5432"
    tmpfs:
      - /var/lib/postgresql/data  # RAM disk for speed

  redis-test:
    image: redis:7-alpine
    ports:
      - "6380:6379"
    command: redis-server --save ""  # Disable persistence for speed
```

### Database Setup and Teardown

```typescript
// src/test/integration-setup.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { migrate } from 'drizzle-orm/node-postgres/migrator';
import { Pool } from 'pg';

let pool: Pool;
let db: ReturnType<typeof drizzle>;

export async function setupTestDatabase() {
  pool = new Pool({
    connectionString: 'postgresql://test_user:test_password@localhost:5433/superadmin_test',
  });
  db = drizzle(pool);
  await migrate(db, { migrationsFolder: './drizzle' });
  return db;
}

export async function teardownTestDatabase() {
  // Truncate all tables between tests for isolation
  await pool.query(`
    DO $$ DECLARE
      r RECORD;
    BEGIN
      FOR r IN (SELECT tablename FROM pg_tables WHERE schemaname = 'public') LOOP
        EXECUTE 'TRUNCATE TABLE ' || quote_ident(r.tablename) || ' CASCADE';
      END LOOP;
    END $$;
  `);
}

export async function closeTestDatabase() {
  await pool.end();
}
```

### Running Integration Tests

```bash
# Start test infrastructure
docker compose -f docker-compose.test.yml up -d

# Run integration tests
pnpm test:integration

# Tear down
docker compose -f docker-compose.test.yml down -v
```

### Integration Test Example

```typescript
// src/services/tenant.integration.ts
import { describe, it, expect, beforeAll, beforeEach, afterAll } from 'vitest';
import { setupTestDatabase, teardownTestDatabase, closeTestDatabase } from '../test/integration-setup';
import { TenantService } from './tenant.service';
import { UserService } from './user.service';

describe('Tenant Isolation (Integration)', () => {
  let db: any;
  let tenantService: TenantService;
  let userService: UserService;

  beforeAll(async () => {
    db = await setupTestDatabase();
    tenantService = new TenantService(db);
    userService = new UserService(db);
  });

  beforeEach(async () => {
    await teardownTestDatabase();
  });

  afterAll(async () => {
    await closeTestDatabase();
  });

  it('ensures Tenant A cannot access Tenant B users', async () => {
    // Create two tenants
    const tenantA = await tenantService.create({ name: 'Tenant A', slug: 'tenant-a' });
    const tenantB = await tenantService.create({ name: 'Tenant B', slug: 'tenant-b' });

    // Create users in each tenant
    await userService.create({ email: 'alice@a.com', tenantId: tenantA.id, role: 'admin' });
    await userService.create({ email: 'bob@b.com', tenantId: tenantB.id, role: 'admin' });

    // Query users scoped to Tenant A
    const tenantAUsers = await userService.listByTenant(tenantA.id);
    expect(tenantAUsers).toHaveLength(1);
    expect(tenantAUsers[0].email).toBe('alice@a.com');

    // Verify Tenant A cannot see Tenant B users
    const crossTenantAttempt = tenantAUsers.filter(u => u.tenantId === tenantB.id);
    expect(crossTenantAttempt).toHaveLength(0);
  });

  it('enforces RLS when accessing data through raw queries', async () => {
    const tenantA = await tenantService.create({ name: 'Tenant A', slug: 'tenant-a' });

    // Set RLS context
    await db.execute(`SET app.current_tenant_id = '${tenantA.id}'`);

    // Insert via RLS-enabled table
    await userService.create({ email: 'alice@a.com', tenantId: tenantA.id, role: 'member' });

    // Attempt to query without tenant context should return no results
    await db.execute(`RESET app.current_tenant_id`);
    const results = await db.execute(`SELECT * FROM users`);
    expect(results.rows).toHaveLength(0); // RLS blocks access
  });
});
```

### What to Integration Test

- **Database operations**: CRUD operations, complex queries, migrations, RLS enforcement.
- **Service interactions**: Service A calling Service B with real implementations.
- **Cache behavior**: Redis caching, cache invalidation, cache-aside patterns.
- **Queue processing**: Message publishing, consuming, dead-letter handling.
- **Transaction boundaries**: Multi-step operations that must be atomic.

---

## E2E Testing with Playwright

### Why Playwright

Playwright is the primary E2E testing framework. It provides cross-browser testing (Chromium, Firefox, WebKit), auto-waiting, network interception, and parallel execution.

### Setup

```bash
# Install Playwright
pnpm add -D @playwright/test
npx playwright install --with-deps
```

### Configuration

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 4 : undefined,
  reporter: [
    ['html', { outputFolder: 'playwright-report' }],
    ['junit', { outputFile: 'test-results/e2e-results.xml' }],
  ],
  use: {
    baseURL: process.env.BASE_URL || 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
    {
      name: 'mobile-chrome',
      use: { ...devices['Pixel 7'] },
    },
    {
      name: 'mobile-safari',
      use: { ...devices['iPhone 14'] },
    },
  ],
  webServer: {
    command: 'pnpm dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120000,
  },
});
```

### Running Playwright Tests

```bash
# Run all E2E tests
pnpm test:e2e

# Run in headed mode (see the browser)
pnpm test:e2e -- --headed

# Run a specific test file
pnpm test:e2e e2e/auth/login.spec.ts

# Run in a specific browser
pnpm test:e2e -- --project=firefox

# Open the test report
npx playwright show-report

# Run in debug mode
pnpm test:e2e -- --debug

# Update snapshots
pnpm test:e2e -- --update-snapshots
```

### Playwright Test Examples

```typescript
// e2e/auth/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('SuperAdmin Login', () => {
  test('successful login with valid credentials', async ({ page }) => {
    await page.goto('/login');
    await page.fill('[data-testid="email-input"]', 'admin@example.com');
    await page.fill('[data-testid="password-input"]', 'SecureP@ssw0rd!');
    await page.click('[data-testid="login-button"]');

    await expect(page).toHaveURL('/dashboard');
    await expect(page.locator('[data-testid="user-menu"]')).toBeVisible();
  });

  test('shows error for invalid credentials', async ({ page }) => {
    await page.goto('/login');
    await page.fill('[data-testid="email-input"]', 'admin@example.com');
    await page.fill('[data-testid="password-input"]', 'wrong-password');
    await page.click('[data-testid="login-button"]');

    await expect(page.locator('[data-testid="error-message"]')).toContainText(
      'Invalid email or password'
    );
    await expect(page).toHaveURL('/login');
  });

  test('enforces MFA for admin roles', async ({ page }) => {
    await page.goto('/login');
    await page.fill('[data-testid="email-input"]', 'superadmin@example.com');
    await page.fill('[data-testid="password-input"]', 'SecureP@ssw0rd!');
    await page.click('[data-testid="login-button"]');

    // Should redirect to MFA verification
    await expect(page).toHaveURL('/mfa/verify');
    await expect(page.locator('[data-testid="totp-input"]')).toBeVisible();
  });

  test('locks account after 10 failed attempts', async ({ page }) => {
    await page.goto('/login');

    for (let i = 0; i < 10; i++) {
      await page.fill('[data-testid="email-input"]', 'admin@example.com');
      await page.fill('[data-testid="password-input"]', `wrong-password-${i}`);
      await page.click('[data-testid="login-button"]');
      await page.waitForTimeout(500);
    }

    await expect(page.locator('[data-testid="error-message"]')).toContainText(
      'Account locked'
    );
  });
});
```

```typescript
// e2e/tenants/tenant-isolation.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Tenant Isolation', () => {
  test('Tenant A admin cannot see Tenant B data', async ({ browser }) => {
    // Login as Tenant A admin
    const contextA = await browser.newContext();
    const pageA = await contextA.newPage();
    await pageA.goto('/login');
    await pageA.fill('[data-testid="email-input"]', 'admin@tenant-a.com');
    await pageA.fill('[data-testid="password-input"]', 'SecureP@ssw0rd!');
    await pageA.click('[data-testid="login-button"]');
    await pageA.waitForURL('/dashboard');

    // Navigate to users list
    await pageA.click('[data-testid="nav-users"]');
    const usersList = await pageA.locator('[data-testid="user-row"]').allTextContents();

    // Verify no Tenant B users are visible
    expect(usersList.every(u => !u.includes('tenant-b.com'))).toBe(true);

    await contextA.close();
  });
});
```

### Page Object Model

```typescript
// e2e/pages/login.page.ts
import { Page, Locator, expect } from '@playwright/test';

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly loginButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.locator('[data-testid="email-input"]');
    this.passwordInput = page.locator('[data-testid="password-input"]');
    this.loginButton = page.locator('[data-testid="login-button"]');
    this.errorMessage = page.locator('[data-testid="error-message"]');
  }

  async goto() {
    await this.page.goto('/login');
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.loginButton.click();
  }

  async expectError(message: string) {
    await expect(this.errorMessage).toContainText(message);
  }
}
```

---

## E2E Testing with Selenium Grid

### When to Use Selenium Grid

Use Selenium Grid instead of (or alongside) Playwright when:

- **Legacy browser testing**: Testing on older browser versions not supported by Playwright (e.g., IE11 for specific enterprise clients).
- **Distributed execution at scale**: Running hundreds of parallel tests across a large Selenium Grid cluster.
- **Mobile device testing**: Testing on real devices via Appium integrated with Selenium Grid.
- **Existing infrastructure**: Teams with an established Selenium Grid infrastructure and existing test suites.

For all other cases, Playwright is preferred due to faster execution, better auto-waiting, and simpler setup.

### Selenium Grid Setup

```yaml
# docker-compose.selenium.yml
services:
  selenium-hub:
    image: selenium/hub:4.18
    ports:
      - "4442:4442"
      - "4443:4443"
      - "4444:4444"

  chrome-node:
    image: selenium/node-chrome:4.18
    depends_on:
      - selenium-hub
    environment:
      SE_EVENT_BUS_HOST: selenium-hub
      SE_EVENT_BUS_PUBLISH_PORT: 4442
      SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
      SE_NODE_MAX_SESSIONS: 4
    shm_size: '2g'

  firefox-node:
    image: selenium/node-firefox:4.18
    depends_on:
      - selenium-hub
    environment:
      SE_EVENT_BUS_HOST: selenium-hub
      SE_EVENT_BUS_PUBLISH_PORT: 4442
      SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
      SE_NODE_MAX_SESSIONS: 4
    shm_size: '2g'

  edge-node:
    image: selenium/node-edge:4.18
    depends_on:
      - selenium-hub
    environment:
      SE_EVENT_BUS_HOST: selenium-hub
      SE_EVENT_BUS_PUBLISH_PORT: 4442
      SE_EVENT_BUS_SUBSCRIBE_PORT: 4443
      SE_NODE_MAX_SESSIONS: 4
    shm_size: '2g'
```

### Running Selenium Tests

```bash
# Start Selenium Grid
docker compose -f docker-compose.selenium.yml up -d

# Verify Grid is running
curl http://localhost:4444/status

# Run Selenium tests
pnpm test:selenium

# View Grid console
open http://localhost:4444/ui
```

### Selenium Test Example (WebDriverIO)

```typescript
// selenium/specs/login.spec.ts
describe('SuperAdmin Login (Selenium)', () => {
  it('should login successfully with valid credentials', async () => {
    await browser.url('/login');
    await $('[data-testid="email-input"]').setValue('admin@example.com');
    await $('[data-testid="password-input"]').setValue('SecureP@ssw0rd!');
    await $('[data-testid="login-button"]').click();

    await expect(browser).toHaveUrl(expect.stringContaining('/dashboard'));
  });
});
```

---

## API Testing

### Tooling

- **Framework**: Supertest — HTTP assertion library that works with any Node.js HTTP server.
- **Validation**: Zod schemas to validate response shapes.

### API Test Example

```typescript
// src/api/__tests__/tenants.api.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import supertest from 'supertest';
import { createApp } from '../app';
import { setupTestDatabase, closeTestDatabase } from '../../test/integration-setup';

describe('Tenants API', () => {
  let app: any;
  let request: supertest.SuperTest<supertest.Test>;
  let adminToken: string;

  beforeAll(async () => {
    const db = await setupTestDatabase();
    app = createApp(db);
    request = supertest(app);

    // Obtain an admin token
    const loginRes = await request
      .post('/api/v1/auth/login')
      .send({ email: 'superadmin@test.com', password: 'TestP@ssw0rd!' });
    adminToken = loginRes.body.token;
  });

  afterAll(async () => {
    await closeTestDatabase();
  });

  describe('POST /api/v1/tenants', () => {
    it('creates a tenant with valid data', async () => {
      const res = await request
        .post('/api/v1/tenants')
        .set('Authorization', `Bearer ${adminToken}`)
        .send({
          name: 'Acme Corp',
          slug: 'acme-corp',
          plan: 'professional',
        });

      expect(res.status).toBe(201);
      expect(res.body).toMatchObject({
        id: expect.any(String),
        name: 'Acme Corp',
        slug: 'acme-corp',
        plan: 'professional',
        status: 'active',
      });
    });

    it('returns 400 for invalid tenant data', async () => {
      const res = await request
        .post('/api/v1/tenants')
        .set('Authorization', `Bearer ${adminToken}`)
        .send({ name: '' }); // Missing required fields

      expect(res.status).toBe(400);
      expect(res.body.errors).toBeDefined();
    });

    it('returns 401 without authentication', async () => {
      const res = await request
        .post('/api/v1/tenants')
        .send({ name: 'Unauthorized Tenant', slug: 'unauth' });

      expect(res.status).toBe(401);
    });

    it('returns 403 for non-admin roles', async () => {
      // Login as a regular user
      const loginRes = await request
        .post('/api/v1/auth/login')
        .send({ email: 'viewer@test.com', password: 'TestP@ssw0rd!' });

      const res = await request
        .post('/api/v1/tenants')
        .set('Authorization', `Bearer ${loginRes.body.token}`)
        .send({ name: 'Forbidden Tenant', slug: 'forbidden' });

      expect(res.status).toBe(403);
    });
  });

  describe('GET /api/v1/tenants', () => {
    it('returns paginated tenant list', async () => {
      const res = await request
        .get('/api/v1/tenants?page=1&limit=10')
        .set('Authorization', `Bearer ${adminToken}`);

      expect(res.status).toBe(200);
      expect(res.body).toHaveProperty('data');
      expect(res.body).toHaveProperty('pagination');
      expect(res.body.pagination).toMatchObject({
        page: 1,
        limit: 10,
        total: expect.any(Number),
      });
    });
  });
});
```

### API Test Patterns

- **Authentication tests**: Verify 401 for missing tokens, 403 for insufficient permissions.
- **Input validation tests**: Verify 400 for invalid payloads; test boundary values.
- **RBAC tests**: Verify each endpoint respects the role hierarchy.
- **Tenant isolation tests**: Verify cross-tenant API calls are rejected.
- **Rate limiting tests**: Verify 429 responses when rate limits are exceeded.
- **Pagination tests**: Verify correct pagination metadata and data slicing.
- **Idempotency tests**: Verify that repeated POST requests with the same idempotency key produce the same result.

---

## Security Testing

### OWASP ZAP (Dynamic Application Security Testing)

OWASP ZAP performs automated DAST scans against running application instances.

```bash
# Run ZAP baseline scan (passive scanning)
docker run --rm -t \
  --network host \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t http://localhost:3000 \
  -r zap-report.html \
  -c zap-config.conf

# Run ZAP full scan (active scanning — use only against test environments)
docker run --rm -t \
  --network host \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-full-scan.py \
  -t http://localhost:3000 \
  -r zap-full-report.html \
  -c zap-config.conf

# Run ZAP API scan against OpenAPI spec
docker run --rm -t \
  --network host \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-api-scan.py \
  -t http://localhost:3000/api/v1/openapi.json \
  -f openapi \
  -r zap-api-report.html
```

### ZAP Configuration

```
# zap-config.conf
# Fail the build on high-severity alerts
10016	FAIL	(Web Browser XSS Protection Not Enabled - informational only)
10038	FAIL	(Content Security Policy Header Not Set)
40012	FAIL	(Cross Site Scripting - Reflected)
40014	FAIL	(Cross Site Scripting - Persistent)
40018	FAIL	(SQL Injection)
40019	FAIL	(SQL Injection - MySQL)
90019	FAIL	(Server Side Code Injection)
90020	FAIL	(Remote OS Command Injection)
```

### Trivy (Container and Dependency Scanning)

```bash
# Scan container image
trivy image --severity HIGH,CRITICAL --exit-code 1 superadmin:latest

# Scan filesystem for vulnerabilities
trivy fs --severity HIGH,CRITICAL --exit-code 1 .

# Scan for secrets
trivy fs --scanners secret --exit-code 1 .

# Generate SBOM
trivy image --format spdx-json --output sbom.json superadmin:latest
```

### SonarQube (Static Analysis)

```bash
# Run SonarQube scanner
sonar-scanner \
  -Dsonar.projectKey=superadmin \
  -Dsonar.sources=src \
  -Dsonar.tests=src \
  -Dsonar.test.inclusions=**/*.test.ts,**/*.spec.ts \
  -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=$SONAR_TOKEN
```

### Quality Gate Criteria

| Metric | Threshold | Action on Failure |
|--------|-----------|-------------------|
| New bugs | 0 | Block merge |
| New vulnerabilities | 0 | Block merge |
| New security hotspots reviewed | 100% | Block merge |
| New code coverage | >= 80% | Block merge |
| New duplicated lines | <= 3% | Warning |
| Trivy critical CVEs | 0 | Block deployment |
| ZAP high-severity alerts | 0 | Block deployment |

---

## Performance and Load Testing

### Tooling

- **k6** (primary): Modern load testing tool with JavaScript scripting.
- **Artillery**: Used for quick API load tests and smoke tests.

### k6 Load Test Example

```javascript
// load-tests/tenant-api.js
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const errorRate = new Rate('errors');
const tenantListDuration = new Trend('tenant_list_duration');

export const options = {
  stages: [
    { duration: '1m', target: 50 },    // Ramp up to 50 users
    { duration: '5m', target: 50 },    // Stay at 50 users
    { duration: '2m', target: 200 },   // Ramp up to 200 users
    { duration: '5m', target: 200 },   // Stay at 200 users
    { duration: '2m', target: 0 },     // Ramp down
  ],
  thresholds: {
    http_req_duration: ['p(95)<500', 'p(99)<1000'], // 95th < 500ms, 99th < 1s
    errors: ['rate<0.01'],                            // Error rate < 1%
    tenant_list_duration: ['p(95)<300'],              // Tenant list < 300ms
  },
};

const BASE_URL = __ENV.BASE_URL || 'http://localhost:3000';

export function setup() {
  const loginRes = http.post(`${BASE_URL}/api/v1/auth/login`, JSON.stringify({
    email: 'loadtest@example.com',
    password: 'LoadTestP@ss!',
  }), { headers: { 'Content-Type': 'application/json' } });

  return { token: loginRes.json('token') };
}

export default function (data) {
  const headers = {
    Authorization: `Bearer ${data.token}`,
    'Content-Type': 'application/json',
  };

  // GET /api/v1/tenants
  const listRes = http.get(`${BASE_URL}/api/v1/tenants?page=1&limit=20`, { headers });
  tenantListDuration.add(listRes.timings.duration);
  check(listRes, {
    'tenant list status 200': (r) => r.status === 200,
    'tenant list has data': (r) => r.json('data').length > 0,
  });
  errorRate.add(listRes.status !== 200);

  sleep(1);
}
```

### Running Load Tests

```bash
# Run load test
k6 run load-tests/tenant-api.js

# Run with environment variables
k6 run --env BASE_URL=https://staging.{DOMAIN} load-tests/tenant-api.js

# Run with HTML report
k6 run --out json=results.json load-tests/tenant-api.js
```

### Performance Benchmarks (Targets)

| Endpoint Category | p50 | p95 | p99 | Max Throughput |
|-------------------|-----|-----|-----|----------------|
| Authentication | < 200ms | < 500ms | < 1s | 500 req/s |
| Dashboard data | < 150ms | < 400ms | < 800ms | 1,000 req/s |
| Tenant CRUD | < 100ms | < 300ms | < 600ms | 2,000 req/s |
| User management | < 100ms | < 300ms | < 600ms | 2,000 req/s |
| Audit log queries | < 300ms | < 800ms | < 1.5s | 500 req/s |
| Report generation | < 2s | < 5s | < 10s | 50 req/s |

---

## Coverage Requirements

### Coverage Thresholds

| Category | Line Coverage | Branch Coverage | Function Coverage |
|----------|--------------|-----------------|-------------------|
| **Critical paths** (auth, RBAC, tenant isolation, encryption) | >= 90% | >= 85% | >= 90% |
| **Business logic** (services, middleware) | >= 80% | >= 80% | >= 80% |
| **API routes** | >= 80% | >= 75% | >= 80% |
| **Utilities** | >= 80% | >= 80% | >= 80% |
| **UI components** | >= 70% | >= 65% | >= 70% |
| **Overall project** | >= 80% | >= 75% | >= 80% |

### Coverage Enforcement

- Coverage thresholds are enforced in CI. PRs that reduce coverage below the threshold are blocked.
- New code must have >= 80% coverage. The `--changedSince` flag is used to measure coverage on new code only.
- Coverage reports are posted as PR comments via the CI bot.

### Viewing Coverage

```bash
# Generate coverage report
pnpm test:coverage

# Open HTML coverage report
open coverage/index.html
```

---

## CI Pipeline Integration

### Jenkins Pipeline Stages

The testing strategy integrates into a multi-stage Jenkins pipeline.

```groovy
// Jenkinsfile (abridged)
pipeline {
  agent { kubernetes { yamlFile 'ci/pod-template.yaml' } }

  environment {
    NODE_ENV = 'test'
    DATABASE_URL = 'postgresql://test:test@postgres-test:5432/superadmin_test'
  }

  stages {
    stage('Install') {
      steps {
        sh 'pnpm install --frozen-lockfile'
      }
    }

    stage('Lint & Type Check') {
      parallel {
        stage('ESLint') {
          steps { sh 'pnpm lint' }
        }
        stage('TypeScript') {
          steps { sh 'pnpm type-check' }
        }
        stage('Prettier') {
          steps { sh 'pnpm format:check' }
        }
      }
    }

    stage('Unit Tests') {
      steps {
        sh 'pnpm test:coverage'
      }
      post {
        always {
          junit 'test-results/unit-results.xml'
          publishHTML(target: [
            reportDir: 'coverage',
            reportFiles: 'index.html',
            reportName: 'Coverage Report',
          ])
        }
      }
    }

    stage('Integration Tests') {
      steps {
        sh 'pnpm test:integration'
      }
      post {
        always {
          junit 'test-results/integration-results.xml'
        }
      }
    }

    stage('Security Scans') {
      parallel {
        stage('SonarQube') {
          steps {
            withSonarQubeEnv('sonar') {
              sh 'sonar-scanner'
            }
            waitForQualityGate abortPipeline: true
          }
        }
        stage('Trivy') {
          steps {
            sh 'trivy fs --severity HIGH,CRITICAL --exit-code 1 .'
          }
        }
        stage('Secret Scan') {
          steps {
            sh 'trivy fs --scanners secret --exit-code 1 .'
          }
        }
      }
    }

    stage('Build') {
      steps {
        sh 'pnpm build'
        sh 'docker build -t superadmin:${GIT_COMMIT} .'
      }
    }

    stage('Container Scan') {
      steps {
        sh 'trivy image --severity HIGH,CRITICAL --exit-code 1 superadmin:${GIT_COMMIT}'
      }
    }

    stage('E2E Tests') {
      steps {
        sh 'pnpm test:e2e'
      }
      post {
        always {
          junit 'test-results/e2e-results.xml'
          publishHTML(target: [
            reportDir: 'playwright-report',
            reportFiles: 'index.html',
            reportName: 'Playwright Report',
          ])
        }
      }
    }

    stage('OWASP ZAP') {
      when { branch 'main' }
      steps {
        sh '''
          docker run --rm --network host \
            ghcr.io/zaproxy/zaproxy:stable \
            zap-baseline.py -t http://localhost:3000 \
            -r zap-report.html -c zap-config.conf
        '''
      }
      post {
        always {
          publishHTML(target: [
            reportDir: '.',
            reportFiles: 'zap-report.html',
            reportName: 'ZAP Report',
          ])
        }
      }
    }

    stage('Performance Tests') {
      when { branch 'main' }
      steps {
        sh 'k6 run --out json=k6-results.json load-tests/tenant-api.js'
      }
    }
  }

  post {
    failure {
      slackSend channel: '#ci-alerts',
        message: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
    success {
      slackSend channel: '#ci-alerts',
        message: "Build PASSED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
    }
  }
}
```

### Pipeline Summary

| Stage | Duration Target | Blocking? |
|-------|----------------|-----------|
| Install | < 1 min | Yes |
| Lint & Type Check | < 2 min | Yes |
| Unit Tests | < 3 min | Yes |
| Integration Tests | < 5 min | Yes |
| Security Scans | < 5 min | Yes (on quality gate failure) |
| Build | < 2 min | Yes |
| Container Scan | < 2 min | Yes (critical CVEs) |
| E2E Tests | < 10 min | Yes |
| OWASP ZAP | < 10 min | Yes (high alerts, main branch only) |
| Performance Tests | < 15 min | No (reports only, main branch only) |
| **Total** | **< 15 min** | |

---

## Test Data Management

### Principles

1. **Isolation**: Each test creates its own data and cleans up after itself. Tests never depend on shared mutable state.
2. **Factories over fixtures**: Use factory functions to create test data programmatically. Factories produce valid default data that can be overridden per test.
3. **Seeding**: A seed script populates the test database with baseline data (tenants, roles, permissions) before the test suite runs.
4. **No production data**: Test environments never contain production data. Synthetic data generators create realistic but fictitious data.

### Test Data Factory Example

```typescript
// src/test/factories/tenant.factory.ts
import { faker } from '@faker-js/faker';
import { v4 as uuid } from 'uuid';

interface TenantOverrides {
  id?: string;
  name?: string;
  slug?: string;
  plan?: string;
  status?: string;
}

export function buildTenant(overrides: TenantOverrides = {}) {
  const name = overrides.name ?? faker.company.name();
  return {
    id: overrides.id ?? uuid(),
    name,
    slug: overrides.slug ?? faker.helpers.slugify(name).toLowerCase(),
    plan: overrides.plan ?? 'professional',
    status: overrides.status ?? 'active',
    createdAt: new Date(),
    updatedAt: new Date(),
  };
}

export function buildUser(overrides: Partial<any> = {}) {
  return {
    id: overrides.id ?? uuid(),
    email: overrides.email ?? faker.internet.email(),
    name: overrides.name ?? faker.person.fullName(),
    role: overrides.role ?? 'member',
    tenantId: overrides.tenantId ?? uuid(),
    active: overrides.active ?? true,
    createdAt: new Date(),
    updatedAt: new Date(),
  };
}
```

### Database Reset Strategy

| Strategy | When Used | How |
|----------|-----------|-----|
| Truncate all tables | Between integration test files | `TRUNCATE ... CASCADE` on all tables |
| Transaction rollback | Between individual integration tests | Begin transaction before test, rollback after |
| Drop and recreate | Before full test suite | Drop database, re-run migrations |
| Seed data | Before full test suite | Insert baseline tenants, roles, permissions |

---

## Mocking Strategy

### When to Mock

| Mock | Why |
|------|-----|
| **External APIs** (Stripe, SendGrid, Twilio) | Avoid network calls, cost, and rate limits in tests |
| **Time/Date** | Make tests deterministic (`vi.useFakeTimers()`) |
| **Random values** | Make tests reproducible (`vi.spyOn(Math, 'random')`) |
| **File system** (for unit tests) | Avoid side effects |
| **Email sending** | Prevent actual emails during tests |
| **SMS/push notifications** | Prevent actual notifications |

### When NOT to Mock

| Do Not Mock | Why |
|-------------|-----|
| **Database** (in integration tests) | Mocking the database hides real query issues, RLS enforcement, and migration problems |
| **Redis** (in integration tests) | Real caching behavior differs from mocked behavior |
| **Your own code** | Mocking internal modules makes tests brittle and tests the mock, not the code |
| **Validation logic** | Validation must be tested end-to-end with real schemas |
| **Authorization logic** | RBAC and RLS must be tested against real enforcement layers |

### Mocking External Services

```typescript
// src/test/mocks/email.mock.ts
import { vi } from 'vitest';

export const mockEmailService = {
  send: vi.fn().mockResolvedValue({ messageId: 'mock-msg-001', status: 'sent' }),
  sendBatch: vi.fn().mockResolvedValue({ count: 5, status: 'sent' }),
  verifyAddress: vi.fn().mockResolvedValue({ valid: true }),
};

// Usage in tests:
// vi.mock('../services/email.service', () => ({ EmailService: vi.fn(() => mockEmailService) }));
```

```typescript
// src/test/mocks/stripe.mock.ts
import { vi } from 'vitest';

export const mockStripeClient = {
  customers: {
    create: vi.fn().mockResolvedValue({ id: 'cus_mock123' }),
    retrieve: vi.fn().mockResolvedValue({ id: 'cus_mock123', email: 'test@example.com' }),
  },
  subscriptions: {
    create: vi.fn().mockResolvedValue({ id: 'sub_mock123', status: 'active' }),
    update: vi.fn().mockResolvedValue({ id: 'sub_mock123', status: 'active' }),
    cancel: vi.fn().mockResolvedValue({ id: 'sub_mock123', status: 'canceled' }),
  },
};
```

### Mock Hygiene Rules

1. **Clear mocks between tests**: Always call `vi.clearAllMocks()` in `beforeEach` to prevent test cross-contamination.
2. **Verify mock calls**: Assert that mocks were called with expected arguments, not just that the test passed.
3. **Limit mock scope**: Mock at the narrowest possible scope. Prefer `vi.spyOn()` over `vi.mock()` when possible.
4. **No mocking in E2E tests**: E2E tests run against the real application with real dependencies. The only exception is external payment and notification services, which use sandbox/test modes instead of mocks.

---

*This document is maintained alongside the codebase. Update it when testing tools, thresholds, or strategies change.*
