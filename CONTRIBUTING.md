# Contributing Guide

Thank you for your interest in contributing to the SuperAdmin SaaS platform. This document provides guidelines and procedures for contributing to the project.

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Development Setup](#development-setup)
3. [Branch Naming Convention](#branch-naming-convention)
4. [Commit Message Format](#commit-message-format)
5. [Pull Request Process](#pull-request-process)
6. [Code Style](#code-style)
7. [Testing Requirements](#testing-requirements)
8. [Documentation Requirements](#documentation-requirements)
9. [Security Vulnerability Reporting](#security-vulnerability-reporting)

---

## Code of Conduct

All contributors are expected to adhere to our Code of Conduct. We are committed to providing a welcoming and inclusive experience for everyone. Please read the full [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md) before participating.

Key principles:

- Be respectful and inclusive in all interactions
- Provide constructive feedback focused on the work, not the person
- Accept responsibility for mistakes and learn from them
- Prioritize the health and security of the project and its users

Reports of abusive, harassing, or otherwise unacceptable behavior can be directed to the project maintainers at `{ADMIN_EMAIL}`.

---

## Development Setup

### Prerequisites

Ensure you have the required tools installed. See [INSTALLATION.md](INSTALLATION.md) for full details.

| Tool | Version |
|------|---------|
| Node.js | 24.x LTS |
| PostgreSQL | 18.x |
| Redis/Valkey | 7.x / 8.x |
| Docker | 27.x |
| Git | 2.40+ |

### Fork and Clone

```bash
# Fork the repository on GitHub, then clone your fork
git clone https://github.com/<your-username>/superadmin.git
cd superadmin

# Add the upstream remote
git remote add upstream https://github.com/{COMPANY_NAME}/superadmin.git

# Verify remotes
git remote -v
```

### Install Dependencies

```bash
npm ci
```

### Start Development Services

```bash
# Start PostgreSQL, Redis, and MinIO via Docker Compose
docker compose up -d postgres redis minio

# Run database migrations
npm run db:migrate

# Seed development data
npm run db:seed

# Start the application in development mode
npm run dev
```

### Verify Your Setup

```bash
# Run the full test suite
npm run test

# Run linting
npm run lint

# Run type checking
npx tsc --noEmit
```

### Keep Your Fork Updated

```bash
# Fetch upstream changes
git fetch upstream

# Merge upstream main into your local main
git checkout main
git merge upstream/main

# Push updated main to your fork
git push origin main
```

---

## Branch Naming Convention

All branches must follow this naming pattern:

```
<type>/<short-description>
```

### Branch Types

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New feature or capability | `feature/webhook-retry-backoff` |
| `fix/` | Bug fix | `fix/jwt-refresh-race-condition` |
| `docs/` | Documentation changes | `docs/api-rate-limiting-guide` |
| `refactor/` | Code refactoring (no behavior change) | `refactor/extract-auth-middleware` |
| `test/` | Adding or updating tests | `test/rbac-permission-matrix` |
| `perf/` | Performance improvement | `perf/optimize-user-list-query` |
| `chore/` | Build, CI, tooling, dependencies | `chore/upgrade-typescript-5.7` |
| `security/` | Security patch or hardening | `security/patch-xss-sanitization` |

### Rules

- Use lowercase with hyphens (kebab-case)
- Keep descriptions concise (3-5 words)
- Reference an issue number when applicable: `fix/123-jwt-refresh-race`
- Never commit directly to `main`

---

## Commit Message Format

This project follows the [Conventional Commits](https://www.conventionalcommits.org/) specification.

### Format

```
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Types

| Type | Description |
|------|-------------|
| `feat` | New feature |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, missing semicolons (no code change) |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `perf` | Performance improvement |
| `test` | Adding or correcting tests |
| `build` | Changes to build system or external dependencies |
| `ci` | Changes to CI configuration |
| `chore` | Other changes that do not modify src or test files |
| `revert` | Reverts a previous commit |
| `security` | Security-related change |

### Scope

The scope indicates the area of the codebase affected:

| Scope | Area |
|-------|------|
| `auth` | Authentication and authorization |
| `users` | User management |
| `tenants` | Multi-tenant system |
| `rbac` | Roles and permissions |
| `audit` | Audit logging |
| `api` | API routes and controllers |
| `db` | Database, migrations, queries |
| `cache` | Redis/Valkey caching |
| `storage` | MinIO object storage |
| `webhook` | Webhook system |
| `config` | System configuration |
| `security` | Security middleware and policies |
| `monitoring` | Health checks and metrics |
| `ui` | Frontend components |
| `ci` | CI/CD pipeline |
| `deps` | Dependencies |

### Examples

```
feat(auth): add TOTP-based MFA enrollment flow

Implements the /auth/mfa/setup and /auth/mfa/verify endpoints with
QR code generation and backup code support.

Closes #42
```

```
fix(rbac): prevent privilege escalation in role assignment

Users can no longer assign roles with a higher privilege level than
their own. Added validation in the role update middleware.

Security: CVE-2026-XXXX
```

```
perf(db): add composite index on audit_logs(tenant_id, created_at)

Reduces audit log query time from ~2s to ~50ms for large tenants
with 100k+ log entries.
```

```
docs(api): document webhook delivery retry behavior
```

### Breaking Changes

Indicate breaking changes with a `!` after the type/scope and a `BREAKING CHANGE` footer:

```
feat(api)!: change pagination response format

BREAKING CHANGE: Pagination metadata is now nested under a
"pagination" key instead of being at the top level of the response.
```

---

## Pull Request Process

### Before Submitting

1. Ensure your branch is up to date with `main`:
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```
2. Run the full validation suite:
   ```bash
   npm run lint
   npx tsc --noEmit
   npm run test
   ```
3. Verify no secrets, credentials, or `.env` files are included in your changes.
4. Write or update tests for your changes (see [Testing Requirements](#testing-requirements)).
5. Update documentation if your change affects the public API or configuration.

### Submitting

1. Push your branch to your fork:
   ```bash
   git push origin feature/your-feature-name
   ```
2. Open a Pull Request against the `main` branch of the upstream repository.
3. Fill in the PR template completely.

### PR Template

```markdown
## Summary

<!-- 1-3 sentences describing the change -->

## Type of Change

- [ ] New feature
- [ ] Bug fix
- [ ] Performance improvement
- [ ] Refactoring
- [ ] Documentation
- [ ] Security patch
- [ ] Dependency update

## Related Issues

<!-- Link to related issues: Closes #123, Relates to #456 -->

## Changes Made

<!-- Bulleted list of changes -->

## Testing

<!-- Describe how you tested these changes -->
- [ ] Unit tests pass
- [ ] Integration tests pass
- [ ] Manual testing completed

## Checklist

- [ ] Code follows the project style guide (ESLint + Prettier)
- [ ] TypeScript strict mode passes without errors
- [ ] All new code has test coverage
- [ ] Documentation updated (if applicable)
- [ ] No secrets or credentials committed
- [ ] Commit messages follow conventional commits
- [ ] Self-reviewed the diff for unintended changes
- [ ] Considered security implications
```

### Review Process

1. At least one maintainer must approve the PR before merging.
2. All CI checks must pass (linting, type checking, tests, security scans).
3. Reviewers will evaluate:
   - Correctness and completeness of the implementation
   - Test coverage and quality
   - Security implications (especially for auth, RBAC, and data access changes)
   - Performance impact
   - Documentation accuracy
   - Adherence to code style and conventions
4. Address all review feedback with new commits (do not force-push during review).
5. Once approved, a maintainer will merge using a squash merge.

### Review Checklist (for Reviewers)

- [ ] The change does what the PR description claims
- [ ] No security regressions (SQL injection, XSS, privilege escalation, data leaks)
- [ ] Multi-tenant isolation is maintained (all queries scoped by `tenant_id`)
- [ ] Audit logging is present for state-changing operations
- [ ] Rate limiting is configured for new endpoints
- [ ] Input validation uses Zod schemas
- [ ] Error responses follow the standard format
- [ ] No hardcoded secrets or configuration values
- [ ] Database queries use parameterized statements
- [ ] New dependencies are justified and audited

---

## Code Style

### Tooling

| Tool | Purpose | Config File |
|------|---------|-------------|
| ESLint | Linting and static analysis | `.eslintrc.cjs` |
| Prettier | Code formatting | `.prettierrc` |
| TypeScript | Type checking (strict mode) | `tsconfig.json` |
| Husky | Git hooks (pre-commit, commit-msg) | `.husky/` |
| lint-staged | Run linters on staged files | `package.json` |

### TypeScript Rules

- **Strict mode** is enabled (`strict: true` in `tsconfig.json`)
- All function parameters and return types must be explicitly typed
- Avoid `any` — use `unknown` with type guards when the type is genuinely unknown
- Prefer `interface` over `type` for object shapes
- Use `enum` sparingly; prefer union types for string literals
- All API request and response bodies must have Zod validation schemas

```typescript
// Preferred
interface CreateUserRequest {
  email: string;
  name: string;
  password: string;
  role: UserRole;
  tenantId: string;
}

// Avoid
type CreateUserRequest = any;
```

### ESLint Rules

Key rules enforced:

- No unused variables or imports
- No `console.log` in production code (use the structured logger)
- Explicit return types on exported functions
- No floating promises (all promises must be awaited or explicitly voided)
- No non-null assertions (`!`) without justification
- Consistent import ordering (external, internal, relative)

### Prettier Configuration

```json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2,
  "useTabs": false,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### Run Code Style Checks

```bash
# Lint and auto-fix
npm run lint:fix

# Format with Prettier
npm run format

# Check formatting without writing
npm run format:check

# Type check
npx tsc --noEmit
```

### Pre-commit Hooks

Husky and lint-staged run automatically on every commit:

1. **ESLint** runs on all staged `.ts` and `.tsx` files
2. **Prettier** formats all staged files
3. **commitlint** validates the commit message against conventional commits

If a hook fails, the commit is rejected. Fix the issues and try again.

---

## Testing Requirements

### Test Framework

| Tool | Purpose |
|------|---------|
| Vitest | Unit and integration test runner |
| Supertest | HTTP endpoint testing |
| Testcontainers | Isolated PostgreSQL/Redis for integration tests |
| Faker | Realistic test data generation |

### Coverage Requirements

| Metric | Minimum Threshold |
|--------|------------------|
| Statements | 80% |
| Branches | 75% |
| Functions | 80% |
| Lines | 80% |

New code should aim for 90%+ coverage. Critical paths (authentication, authorization, data access) require 95%+ coverage.

### Test Types

#### Unit Tests

Test individual functions and classes in isolation with mocked dependencies.

```typescript
// Example: services/user.service.test.ts
import { describe, it, expect, vi } from 'vitest';
import { UserService } from './user.service';

describe('UserService', () => {
  describe('createUser', () => {
    it('should hash the password before storing', async () => {
      const mockRepo = {
        create: vi.fn().mockResolvedValue({ id: 'uuid', email: 'test@example.com' }),
      };
      const service = new UserService(mockRepo);

      await service.createUser({
        email: 'test@example.com',
        password: 'SecureP@ss123',
        name: 'Test User',
        role: 'EMPLOYEE',
        tenantId: 'tenant-uuid',
      });

      const savedUser = mockRepo.create.mock.calls[0][0];
      expect(savedUser.password).not.toBe('SecureP@ss123');
      expect(savedUser.password).toMatch(/^\$2[aby]\$/);
    });
  });
});
```

#### Integration Tests

Test API endpoints with real database and cache connections using Testcontainers.

```typescript
// Example: routes/auth.integration.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import request from 'supertest';
import { createApp } from '../app';

describe('POST /api/v1/auth/login', () => {
  let app: Express;

  beforeAll(async () => {
    app = await createApp({ testing: true });
  });

  it('should return 401 for invalid credentials', async () => {
    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'nonexistent@example.com',
        password: 'WrongPassword',
        tenantId: 'default',
      });

    expect(response.status).toBe(401);
    expect(response.body.success).toBe(false);
    expect(response.body.error.code).toBe('AUTH_INVALID_CREDENTIALS');
  });

  it('should return tokens for valid credentials', async () => {
    const response = await request(app)
      .post('/api/v1/auth/login')
      .send({
        email: 'admin@example.com',
        password: 'TestP@ss123',
        tenantId: 'default',
      });

    expect(response.status).toBe(200);
    expect(response.body.data.accessToken).toBeDefined();
    expect(response.body.data.refreshToken).toBeDefined();
  });
});
```

#### Security Tests

Test for common vulnerabilities:

```typescript
describe('Security', () => {
  it('should reject SQL injection in search parameter', async () => {
    const response = await request(app)
      .get('/api/v1/users?search=\' OR 1=1 --')
      .set('Authorization', `Bearer ${adminToken}`);

    expect(response.status).toBe(400);
  });

  it('should prevent role escalation', async () => {
    const response = await request(app)
      .put('/api/v1/users/some-uuid/role')
      .set('Authorization', `Bearer ${managerToken}`)
      .send({ role: 'SUPER_ADMIN' });

    expect(response.status).toBe(403);
  });

  it('should enforce tenant isolation', async () => {
    const response = await request(app)
      .get('/api/v1/users/user-from-other-tenant')
      .set('Authorization', `Bearer ${tenantAToken}`);

    expect(response.status).toBe(404);
  });
});
```

### Running Tests

```bash
# Run all tests
npm run test

# Run with coverage
npm run test:coverage

# Run a specific test file
npx vitest run src/services/user.service.test.ts

# Run tests in watch mode
npm run test:watch

# Run only integration tests
npm run test:integration
```

---

## Documentation Requirements

### When to Update Documentation

- Adding a new API endpoint: update `API.md` and add the endpoint to `15_api_endpoints.csv`
- Adding a new configuration option: update `INSTALLATION.md` environment variables section
- Changing authentication or authorization behavior: update `SUPER_ADMIN_DOCUMENTATION.md`
- Adding a new deployment target: update `DEPLOYMENT.md`
- Fixing a common issue: add to `TROUBLESHOOTING.md`
- Releasing a new version: update `CHANGELOG.md`

### Documentation Standards

- Use clear, concise language
- Include code examples for all API endpoints (request and response)
- Document error cases, not just happy paths
- Keep placeholder values consistent with the project convention (`{PLACEHOLDER}` format)
- Use tables for structured data
- Maintain the table of contents in each document

### Inline Code Documentation

- All exported functions must have JSDoc comments describing parameters, return values, and thrown exceptions
- Complex business logic must have inline comments explaining the "why"
- Middleware must document what headers it reads and sets

```typescript
/**
 * Validates the JWT access token and attaches the decoded user
 * to the request object. Returns 401 if the token is missing,
 * expired, or blacklisted.
 *
 * @throws {UnauthorizedError} When token validation fails
 */
export async function authenticateToken(
  req: AuthenticatedRequest,
  res: Response,
  next: NextFunction,
): Promise<void> {
  // ...
}
```

---

## Security Vulnerability Reporting

If you discover a security vulnerability, do **not** open a public issue. Instead, follow the responsible disclosure process outlined in [SECURITY.md](SECURITY.md).

**Contact:** Send an encrypted email to `security@{DOMAIN}` with:

1. Description of the vulnerability
2. Steps to reproduce
3. Potential impact assessment
4. Suggested fix (if available)

We aim to acknowledge all reports within 48 hours and provide a resolution timeline within 7 business days. Contributors who report valid vulnerabilities will be credited in the security advisory (unless they prefer to remain anonymous).

---

## Questions?

If you have questions about contributing that are not covered here, open a Discussion on GitHub or reach out to the maintainers at `{ADMIN_EMAIL}`.
