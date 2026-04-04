# {PROJECT_NAME} — Super Admin Documentation

> **Universal SaaS SuperAdmin Template** — Drop into any project and customize placeholders.
>
> **Version:** 1.0.0 | **Last Updated:** {DATE} | **Author:** {AUTHOR}

---

## Table of Contents

1. [Overview & Access](#1-overview--access)
2. [Authentication System](#2-authentication-system)
3. [Multi-Factor Authentication](#3-multi-factor-authentication)
4. [Password Security](#4-password-security)
5. [Session Management](#5-session-management)
6. [CSRF Protection](#6-csrf-protection)
7. [Input Validation & Sanitization](#7-input-validation--sanitization)
8. [Role-Based Access Control](#8-role-based-access-control)
9. [Permission System](#9-permission-system)
10. [Rate Limiting](#10-rate-limiting)
11. [IP Allowlisting & Geo-Restriction](#11-ip-allowlisting--geo-restriction)
12. [Security Headers & CSP](#12-security-headers--csp)
13. [Audit Logging & Compliance](#13-audit-logging--compliance)
14. [Multi-Tenant Isolation / Row-Level Security](#14-multi-tenant-isolation--row-level-security)
15. [Threat Detection & Suspicious Activity](#15-threat-detection--suspicious-activity)
16. [Super Admin Dashboard Tabs](#16-super-admin-dashboard-tabs)
17. [User Management](#17-user-management)
18. [System Configuration](#18-system-configuration)
19. [Database Management](#19-database-management)
20. [API Health Monitoring](#20-api-health-monitoring)
21. [System Health Monitor](#21-system-health-monitor)
22. [Business Intelligence & Analytics](#22-business-intelligence--analytics)
23. [Analytics Integration (Privacy-First)](#23-analytics-integration-privacy-first)
24. [Object Storage Management](#24-object-storage-management)
25. [Email / SMTP Configuration](#25-email--smtp-configuration)
26. [Webhook & Integration System](#26-webhook--integration-system)
27. [Notification Channels](#27-notification-channels)
28. [Feature Flags & Progressive Delivery](#28-feature-flags--progressive-delivery)
29. [License & Subscription Management](#29-license--subscription-management)
30. [Training & Compliance Management](#30-training--compliance-management)
31. [SLA Management](#31-sla-management)
32. [Customer / User Satisfaction Surveys](#32-customer--user-satisfaction-surveys)
33. [Organization Hierarchy](#33-organization-hierarchy)
34. [User Import / Export (CSV)](#34-user-import--export-csv)
35. [Configuration Change Logging](#35-configuration-change-logging)
36. [Performance Optimization](#36-performance-optimization)
37. [Security Testing Suite](#37-security-testing-suite)
38. [Infrastructure Security](#38-infrastructure-security)
39. [Environment Variable Validation](#39-environment-variable-validation)
40. [Safety Guards & Self-Protection](#40-safety-guards--self-protection)
41. [Admin API Endpoints Summary](#41-admin-api-endpoints-summary)
42. [Middleware Execution Order](#42-middleware-execution-order)
43. [Database Schema Reference](#43-database-schema-reference)
44. [File Reference Template](#44-file-reference-template)
45. [Implementation Checklist Summary](#45-implementation-checklist-summary)
46. [File Upload Security](#46-file-upload-security)
47. [Security Monitoring & Incident Response](#47-security-monitoring--incident-response)
48. [Compliance Framework Alignment](#48-compliance-framework-alignment)
49. [Future Security Roadmap](#49-future-security-roadmap)
50. [Live Implementation Reference — EHS SuperAdmin](#50-live-implementation-reference--ehs-superadmin)
- [Appendix: Placeholder Reference](#appendix-placeholder-reference)

---

## Placeholder Convention

All project-specific values use `{PLACEHOLDER}` format. Find-and-replace before use.

| Placeholder | Description | Example |
|-------------|-------------|---------|
| `{PROJECT_NAME}` | Application name | MyApp SaaS |
| `{DOMAIN}` | Production domain | app.example.com |
| `{APP_PORT}` | Application port | 3000 |
| `{DB_PORT}` | Database port | 5432 |
| `{CACHE_PORT}` | Redis/Valkey port | 6379 |
| `{STORAGE_PORT}` | Object storage port | 9000 |
| `{QUEUE_PORT}` | Message queue port | 5672 |
| `{TAILSCALE_IP}` | VPN access IP | 10.x.x.x |
| `{TUNNEL_ID}` | Cloudflare Tunnel ID | uuid |
| `{ROLE_1}` through `{ROLE_7}` | Custom role names | SUPER_ADMIN, TENANT_ADMIN, ... |
| `{MODEL_LIST}` | AI model names | llama3, mistral |
| `{PATH_PREFIX}` | Source code root | packages/api/src |
| `{FRONTEND_PREFIX}` | Frontend root | apps/web/src |
| `{DB_NAME}` | Database name | myapp_db |
| `{ADMIN_EMAIL}` | Default admin email | admin@example.com |
| `{COMPANY_NAME}` | Company/org name | Acme Corp |
| `{CURRENCY}` | Default currency | USD |
| `{TIMEZONE}` | Default timezone | UTC |
| `{RETENTION_YEARS}` | Audit log retention | 7 |

---

## 1. Overview & Access

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Core |

### 1.1 Project Identity

| Field | Value |
|-------|-------|
| Project Name | {PROJECT_NAME} |
| Version | {VERSION} |
| Domain | https://{DOMAIN} |
| Application Port | {APP_PORT} |
| Environment | Production / Staging / Development |
| Framework | {FRAMEWORK} (e.g., Next.js, Nuxt, SvelteKit) |
| Architecture | Multi-tenant SaaS, Monorepo |

### 1.2 Access URLs

| URL | Method | Purpose |
|-----|--------|---------|
| `https://{DOMAIN}` | Cloudflare Tunnel | Public access |
| `http://{TAILSCALE_IP}:{APP_PORT}` | Tailscale VPN | Admin/dev access |
| `http://localhost:{APP_PORT}` | Direct | Local development |

### 1.3 Super Admin Access Requirements

- Role: `SUPER_ADMIN` (highest privilege)
- Dual-layer check: `role = 'admin'` **AND** `is_super_admin = true`
- MFA required for super admin accounts
- IP allowlist enforced (when configured)
- All actions audit-logged

### 1.4 Network Architecture Diagram

```
Internet
  │
  ├─ CDN + DDoS Protection (e.g., Cloudflare)
  │   └─ Encrypted Tunnel
  │       └─ Tunnel daemon on VM
  │
  ├─ VPN (e.g., Tailscale)
  │   └─ {TAILSCALE_IP}:{APP_PORT}
  │
  └─ Firewall (e.g., pfSense)
      └─ Host VM
          ├─ Application Server (port {APP_PORT})
          ├─ PostgreSQL (port {DB_PORT})
          ├─ Redis / Valkey (port {CACHE_PORT})
          ├─ Object Storage (port {STORAGE_PORT})
          ├─ Message Queue (port {QUEUE_PORT})
          └─ AI Models (port {AI_PORT})
```

### 1.5 Default Admin Credentials (Development Only)

| Field | Value |
|-------|-------|
| Tenant | `{DEFAULT_TENANT}` |
| Email | `{ADMIN_EMAIL}` |
| Password | `{ADMIN_PASSWORD}` |

> **WARNING:** Change all default credentials before production deployment.

---

## 2. Authentication System

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Security |

### 2.1 Login Flow

```
User submits credentials (email + password + tenant slug)
  │
  ├─ Rate limit check (login: 5 req / 15 min per IP)
  ├─ Tenant resolution (slug → tenantId)
  ├─ User lookup (email + tenantId)
  ├─ Account lock check (failedLoginAttempts >= 5 → locked 15 min)
  ├─ Password verification (bcrypt compare)
  ├─ MFA check (if enabled → redirect to MFA verification)
  ├─ Session creation (Redis + JWT)
  ├─ Login statistics update (last login, IP, user agent)
  └─ Audit log entry (LOGIN_SUCCESS or LOGIN_FAILURE)
```

### 2.2 JWT Token Configuration

| Parameter | Value |
|-----------|-------|
| Algorithm | HS256 |
| Access Token Expiry | 1 hour |
| Refresh Token Expiry | 7 days |
| Token Storage | HttpOnly cookie |
| Cookie Flags | Secure, SameSite=Lax, HttpOnly |
| Library | `jose` (Edge-compatible) or `jsonwebtoken` |

### 2.3 JWT Claims Structure

```json
{
  "sub": "user-id",
  "email": "user@example.com",
  "role": "SUPER_ADMIN",
  "tenantId": "tenant-id",
  "sessionId": "session-id",
  "iat": 1700000000,
  "exp": 1700003600
}
```

### 2.4 Account Lockout

| Parameter | Value |
|-----------|-------|
| Max Failed Attempts | 5 |
| Lock Duration | 15 minutes |
| Reset On | Successful login or admin unlock |
| Stored Fields | `failedLoginAttempts`, `lockedUntil` |

### 2.5 Login Statistics Tracking

| Metric | Description |
|--------|-------------|
| Last Login | Timestamp of most recent successful login |
| Login IP | Client IP address (with proxy resolution) |
| User Agent | Browser and OS details |
| Login Count | Total successful logins |
| Failed Count | Consecutive failed attempts |
| Country | GeoIP lookup of login IP |
| Browser | Parsed from user agent |

### 2.6 Registration

| Field | Validation |
|-------|------------|
| Name | Required, 2-100 characters |
| Email | Required, valid format, unique per tenant |
| Password | Required, meets complexity rules (see Section 4) |
| Tenant | Required, valid tenant slug |
| Role | Defaults to lowest privilege role |

### 2.6.1 Signup & Verification Security

| Rule | Detail |
|------|--------|
| Rate limit signup | 3 per IP per hour |
| Rate limit verify | 10 per IP per hour |
| Verification token | Cryptographically random UUID, 24hr expiry |
| Password | Min 8 chars (existing PBKDF2 hashing) |
| Email uniqueness | Enforced globally |
| Duplicate signup | Rejected if email already exists |

### 2.7 Logout

- Destroys server-side session (Redis)
- Clears JWT cookies (access + refresh)
- Adds token to blacklist (if force logout)
- Audit log entry: `LOGOUT`

### 2.8 Forced Logout & Token Blacklist

| Feature | Description |
|---------|-------------|
| Force Logout (Single) | Sets `forced_logout_at` timestamp; tokens issued before are rejected |
| Force Logout (All) | Updates `forced_logout_at` for all sessions of a user |
| Bulk Logout | Force logout all users in tenant (emergency) |
| Token Blacklist | SHA-256 hash stored in `token_blacklist` table with expiry |
| Blacklist Cleanup | Cron job removes expired entries |

```
Token Validation Order:
  1. Verify JWT signature
  2. Check token_blacklist table (SHA-256 hash)
  3. Compare token.iat against user.forced_logout_at
  4. Verify user.isActive === true
  5. Fresh DB lookup on every request
```

### 2.9 Single Sign-On (SSO)

| Provider | Protocol | Status |
|----------|----------|--------|
| Google | OAuth 2.0 / OIDC | To Do / In Progress / Done |
| Microsoft / Azure AD | OAuth 2.0 / OIDC | To Do / In Progress / Done |
| Okta | OAuth 2.0 / SAML | To Do / In Progress / Done |
| Custom SAML 2.0 | SAML | To Do / In Progress / Done |
| LDAP / Active Directory | LDAP | To Do / In Progress / Done |

**SSO Security:**
- State parameter validation (CSRF protection)
- PKCE for mobile apps
- Nonce for ID token validation
- Redirect URI whitelist
- Auto-provision users on first SSO login
- Token encryption at rest
- JIT (Just-In-Time) user provisioning

---

## 3. Multi-Factor Authentication

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Security |

### 3.1 TOTP Configuration

| Parameter | Value |
|-----------|-------|
| Algorithm | SHA-1 (RFC 6238) |
| Digits | 6 |
| Period | 30 seconds |
| Time Window | ±1 step (60-second tolerance) |
| Secret Length | 20 bytes (Base32 encoded) |
| QR Code Format | `otpauth://totp/{PROJECT_NAME}:{email}?secret={secret}&issuer={PROJECT_NAME}` |
| Library | `speakeasy` or `otpauth` |
| Secret Encryption | AES-256 at rest |

### 3.2 MFA Lifecycle

```
Setup:
  1. User requests MFA setup → generate TOTP secret
  2. Display QR code + manual entry key
  3. User scans QR in authenticator app (Google Authenticator, Authy, 1Password)
  4. User enters 6-digit code to verify
  5. System generates 10 backup codes
  6. MFA enabled on account

Login with MFA:
  1. Email/password authentication succeeds
  2. MFA check → prompt for TOTP code
  3. Verify TOTP or backup code
  4. Create full session on success

Disable MFA:
  1. Requires current password + valid TOTP code
  2. Admin cannot disable another user's MFA directly
  3. Event is audit-logged
  4. Email notification sent to user
```

### 3.3 Backup Codes

| Parameter | Value |
|-----------|-------|
| Count | 10 per user |
| Format | 8-character uppercase hex (e.g., `A1B2C3D4`) |
| Storage | SHA-256 hashed (one-way) |
| Usage | One-time use; removed after consumption |
| Regeneration | User can regenerate (invalidates all previous codes) |

### 3.4 Admin MFA Controls

| Action | Who Can Do It | Guard |
|--------|---------------|-------|
| View MFA adoption rate | SUPER_ADMIN, TENANT_ADMIN | Read-only |
| Require MFA tenant-wide | SUPER_ADMIN, TENANT_ADMIN | Config toggle |
| View individual MFA status | SUPER_ADMIN | User management |
| Force MFA reset | SUPER_ADMIN | Audit-logged |
| Disable own MFA | Any user | Requires password + TOTP |

### 3.5 MFA Database Schema

```sql
CREATE TABLE user_mfa (
  id            TEXT PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id       TEXT UNIQUE NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  is_enabled    BOOLEAN DEFAULT false,
  secret        TEXT NOT NULL,           -- AES-256 encrypted TOTP secret
  backup_codes  TEXT[] DEFAULT '{}',     -- SHA-256 hashed
  last_used_at  TIMESTAMPTZ,
  created_at    TIMESTAMPTZ DEFAULT now(),
  updated_at    TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_user_mfa_user_id ON user_mfa(user_id);
```

---

## 4. Password Security

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Security |

### 4.1 Hashing Configuration

| Parameter | Value |
|-----------|-------|
| Algorithm | bcrypt |
| Salt Rounds | 12 |
| Library | `bcryptjs` or `bcrypt` |
| Storage | Hashed value only (never plaintext) |

### 4.2 Complexity Rules

| Rule | Requirement |
|------|-------------|
| Minimum Length | 8 characters |
| Maximum Length | 128 characters |
| Uppercase | At least 1 uppercase letter |
| Lowercase | At least 1 lowercase letter |
| Number | At least 1 digit |
| Special Character | At least 1 special character (`!@#$%^&*...`) |
| No Common Passwords | Checked against 10,000+ password blacklist |
| No User Info | Cannot contain email, name, or tenant slug |

### 4.3 Password Blacklist

- Loaded from static file (`common-passwords.txt`) or database table
- Checked on registration, password change, and admin reset
- Contains top 10,000 most common passwords
- Case-insensitive matching

### 4.4 Secure Password Generator

| Parameter | Value |
|-----------|-------|
| Default Length | 16 characters |
| Character Set | Upper + Lower + Digits + Special |
| Entropy Source | `crypto.randomBytes()` |
| Available To | Admin password reset, user self-service |

### 4.5 Admin Password Reset

- Admin generates temporary password for user
- User forced to change password on next login
- Reset event audit-logged with admin ID
- Notification sent to user email

### 4.6 Password Change Flow

```
1. User provides current password (verified via bcrypt)
2. User provides new password
3. New password validated against complexity rules + blacklist
4. New password hashed with bcrypt (12 rounds)
5. Old password hash replaced in database
6. All other sessions invalidated (optional)
7. Audit log entry: PASSWORD_CHANGE
```

---

## 5. Session Management

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Security |

### 5.1 Session Storage (Redis)

| Parameter | Value |
|-----------|-------|
| Backend | Redis / Valkey (port {CACHE_PORT}) |
| Key Format | `session:{tenantId}:{userId}:{sessionId}` |
| TTL | Configurable (default: 60 minutes) |
| Auto-Extend | On activity (sliding window) |
| Serialization | JSON |

### 5.2 Session Data Structure

```json
{
  "userId": "user-id",
  "email": "user@example.com",
  "role": "SUPER_ADMIN",
  "tenantId": "tenant-id",
  "ipAddress": "192.168.1.1",
  "userAgent": "Mozilla/5.0...",
  "createdAt": "2025-01-01T00:00:00Z",
  "lastActivity": "2025-01-01T00:30:00Z",
  "mfaVerified": true
}
```

### 5.3 Session Operations

| Operation | Description |
|-----------|-------------|
| Create | On successful login (with MFA if enabled) |
| Read | On every authenticated request |
| Extend | Reset TTL on user activity |
| Destroy | On logout or force logout |
| List | Admin can view all active sessions per user |
| Bulk Destroy | Force logout all sessions for a user |

### 5.4 Active Session Tracking (Multi-Device)

| Column | Description |
|--------|-------------|
| Session ID | Unique session identifier |
| Device | Browser + OS from user agent parsing |
| IP Address | Client IP |
| Location | GeoIP-derived city/country |
| Created | Session start time |
| Last Active | Most recent activity timestamp |
| Status | Active / Expired |
| Actions | Revoke individual session |

### 5.5 Temporary Session Values

| Key | Purpose | TTL |
|-----|---------|-----|
| `mfa_pending:{userId}` | Awaiting MFA verification | 5 minutes |
| `password_reset:{token}` | Password reset token | 1 hour |
| `email_verify:{token}` | Email verification | 24 hours |
| `invitation:{code}` | User invitation | 7 days |
| `csrf:{sessionId}` | CSRF token binding | Session duration |

---

## 6. CSRF Protection

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 6.1 Double-Submit Cookie Pattern

```
1. Server generates cryptographically random token (64-char hex)
   └─ Source: crypto.getRandomValues() (Web Crypto API)
2. Token set as HttpOnly cookie: `csrf-token`
   └─ Flags: HttpOnly, SameSite=Strict, Secure (prod), Max-Age=86400
3. Client reads token and sends in request header: `x-csrf-token`
4. Server compares cookie value === header value
5. Timing-safe comparison (crypto.timingSafeEqual) to prevent side-channel attacks
```

### 6.2 CSRF Token Configuration

| Parameter | Value |
|-----------|-------|
| Token Length | 64 hexadecimal characters (256 bits) |
| Generation | Web Crypto API (`crypto.getRandomValues`) |
| Cookie Name | `csrf-token` |
| Header Name | `x-csrf-token` |
| Cookie Max Age | 24 hours |
| Cookie Flags | HttpOnly, SameSite=Strict, Secure (prod) |
| Protected Methods | POST, PUT, DELETE, PATCH |
| Exempt Methods | GET, HEAD, OPTIONS |

### 6.3 Validation Rules

| Check | Action on Failure |
|-------|-------------------|
| Missing cookie | 403 Forbidden |
| Missing header | 403 Forbidden |
| Cookie/header mismatch | 403 Forbidden |
| Expired token | Generate new token, prompt retry |

### 6.4 Implementation Files

| File | Purpose |
|------|---------|
| `{PATH_PREFIX}/middleware/csrf.ts` | Server-side validation middleware |
| `{PATH_PREFIX}/lib/csrf.ts` | Token generation and validation utility |
| `{FRONTEND_PREFIX}/hooks/use-csrf.ts` | Client-side React hook |

---

## 7. Input Validation & Sanitization

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 7.1 Sanitization Functions

| Function | Purpose | Example |
|----------|---------|---------|
| `escapeHtml(input)` | Replace `<>&"'` with HTML entities | `<script>` → `&lt;script&gt;` |
| `sanitizeInput(input, maxLen?)` | Remove null bytes, trim, enforce length | General text fields |
| `sanitizeFilename(name)` | Remove `../`, `/`, `\`, control chars | File upload names |
| `sanitizeUrl(url)` | Allow only `http://` and `https://` protocols | URL fields |
| `sanitizeEmail(email)` | Lowercase, trim, format validation | Email inputs |
| `stripHtml(input)` | Remove all HTML tags entirely | Rich text → plain text |
| `sanitizeSearchQuery(query)` | Remove SQL wildcards (`%`, `_`) | Search boxes |
| `sanitizeJson(input)` | Parse and re-serialize; prevent prototype pollution | API body parsing |

### 7.2 Zod Schema Validation

All API inputs validated with Zod schemas at the router/controller level:

```typescript
const createEntitySchema = z.object({
  name: z.string().min(2).max(100).transform(sanitizeInput),
  email: z.string().email().transform(sanitizeEmail),
  description: z.string().max(2000).optional().transform(val => val ? escapeHtml(val) : val),
});
```

### 7.3 SQL Injection Prevention

| Layer | Mechanism |
|-------|-----------|
| ORM | Prisma / Drizzle (parameterized queries by default) |
| Raw Queries | Tagged template literals only (`$queryRaw\`...\``) |
| Search | Sanitized wildcards before LIKE/ILIKE |
| Type Safety | TypeScript enforces types at compile time |

### 7.4 XSS Prevention

| Layer | Mechanism |
|-------|-----------|
| Output | React/Vue auto-escapes JSX/template output |
| Input | `escapeHtml()` on stored user-generated content |
| Headers | CSP `script-src 'self'` (see Section 12) |
| Cookies | HttpOnly flag prevents JS access |
| Rich Text | DOMPurify or similar allowlist sanitizer |

---

## 8. Role-Based Access Control

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Access Control |

### 8.1 Generic Role Hierarchy

| Level | Role | Description |
|-------|------|-------------|
| 7 | `SUPER_ADMIN` | Platform-level control, cross-tenant access |
| 6 | `TENANT_ADMIN` | Full control within their tenant |
| 5 | `MANAGER` | Department/team management, reporting |
| 4 | `OPERATOR` | Day-to-day operations, record management |
| 3 | `STAFF` | Standard user, limited write access |
| 2 | `VIEWER` | Read-only access to assigned resources |
| 1 | `CUSTOMER` | Self-service portal, own data only |

> Customize role names using `{ROLE_1}` through `{ROLE_7}` placeholders. The hierarchy level determines permission inheritance.

### 8.2 Role Capabilities Matrix

| Capability | SUPER_ADMIN | TENANT_ADMIN | MANAGER | OPERATOR | STAFF | VIEWER | CUSTOMER |
|------------|:-----------:|:------------:|:-------:|:--------:|:-----:|:------:|:--------:|
| System Configuration | X | | | | | | |
| Cross-Tenant Access | X | | | | | | |
| Tenant Configuration | X | X | | | | | |
| User Management | X | X | | | | | |
| Role Assignment | X | X | | | | | |
| Audit Log Access | X | X | X | | | | |
| Report Generation | X | X | X | | | | |
| Entity CRUD | X | X | X | X | | | |
| Record Read/Update | X | X | X | X | X | | |
| Read-Only Access | X | X | X | X | X | X | |
| Own Data Access | X | X | X | X | X | X | X |

### 8.3 Role Storage

```sql
-- User role stored as enum on user record
ALTER TYPE user_role ADD VALUE 'SUPER_ADMIN';
ALTER TYPE user_role ADD VALUE 'TENANT_ADMIN';
ALTER TYPE user_role ADD VALUE 'MANAGER';
ALTER TYPE user_role ADD VALUE 'OPERATOR';
ALTER TYPE user_role ADD VALUE 'STAFF';
ALTER TYPE user_role ADD VALUE 'VIEWER';
ALTER TYPE user_role ADD VALUE 'CUSTOMER';
```

### 8.4 Super Admin Dual-Check

```typescript
// Super admin requires BOTH conditions:
const isSuperAdmin = user.role === 'SUPER_ADMIN' && user.isSuperAdmin === true;
// This prevents role-only escalation attacks
```

---

## 9. Permission System

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Access Control |

### 9.1 Permission Categories (70+ Permissions)

| Category | Permissions |
|----------|------------|
| **Users** | `user:create`, `user:read`, `user:update`, `user:delete`, `user:list`, `user:invite`, `user:import`, `user:export` |
| **Roles** | `role:assign`, `role:view`, `role:manage` |
| **Entities** | `entity:create`, `entity:read`, `entity:update`, `entity:delete`, `entity:list`, `entity:export` |
| **Records** | `record:create`, `record:read`, `record:update`, `record:delete`, `record:list`, `record:export`, `record:archive` |
| **Documents** | `document:upload`, `document:read`, `document:delete`, `document:share` |
| **Reports** | `report:view`, `report:create`, `report:export`, `report:schedule` |
| **Billing** | `billing:view`, `billing:create`, `billing:update`, `billing:refund`, `billing:export` |
| **Settings** | `settings:view`, `settings:update`, `settings:security`, `settings:integrations` |
| **Audit** | `audit:view`, `audit:export`, `audit:manage` |
| **Communications** | `comms:email`, `comms:sms`, `comms:push`, `comms:webhook` |
| **Storage** | `storage:upload`, `storage:download`, `storage:delete`, `storage:manage` |
| **System** | `system:backup`, `system:restore`, `system:monitor`, `system:configure` |
| **Analytics** | `analytics:view`, `analytics:export`, `analytics:configure` |
| **Integrations** | `integration:view`, `integration:configure`, `integration:test` |

### 9.2 Permission Check Middleware

```typescript
// Usage in route definitions:
router.get('/entities', hasPermission('entity:list'), listEntities);
router.post('/entities', hasPermission('entity:create'), createEntity);
router.delete('/entities/:id', hasPermission('entity:delete'), deleteEntity);
```

### 9.3 Permission Matrix View (Admin UI)

The admin dashboard renders an interactive permission matrix:

| Permission | SUPER_ADMIN | TENANT_ADMIN | MANAGER | OPERATOR | STAFF | VIEWER | CUSTOMER |
|------------|:-----------:|:------------:|:-------:|:--------:|:-----:|:------:|:--------:|
| `user:create` | ✓ | ✓ | | | | | |
| `user:read` | ✓ | ✓ | ✓ | | | | |
| `entity:create` | ✓ | ✓ | ✓ | ✓ | | | |
| `entity:read` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | |
| `audit:view` | ✓ | ✓ | ✓ | | | | |
| `system:configure` | ✓ | | | | | | |
| ... | ... | ... | ... | ... | ... | ... | ... |

> Full matrix rendered dynamically from permission constants. All 70+ permissions shown as rows, all 7 roles as columns.

---

## 10. Rate Limiting

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 10.1 Rate Limit Configuration (10 Endpoint Types)

| Endpoint Type | Limit | Window | Key | Response |
|---------------|-------|--------|-----|----------|
| Login | 5 requests | 15 minutes | IP | 429 + lockout warning |
| Registration | 3 requests | 1 hour | IP | 429 |
| Password Reset | 3 requests | 1 hour | IP | 429 |
| MFA Verify | 5 requests | 5 minutes | IP + userId | 429 + account lock |
| API (Standard) | 100 requests | 1 minute | IP | 429 |
| API (Admin) | 50 requests | 1 minute | IP + role | 429 |
| File Upload | 20 requests | 1 minute | IP + userId | 429 |
| Export (CSV/PDF) | 10 requests | 1 minute | IP + userId | 429 |
| Webhook Delivery | 100 requests | 1 minute | tenantId | Queue delay |
| Search | 30 requests | 1 minute | IP + userId | 429 |

### 10.2 Sliding Window Algorithm (Redis ZSET)

```
Key: ratelimit:{type}:{identifier}
Member: timestamp (milliseconds)
Score: timestamp (for range queries)

Check:
  1. ZREMRANGEBYSCORE key 0 (now - window)    -- Remove expired entries
  2. ZCARD key                                 -- Count remaining
  3. If count >= limit → REJECT (429)
  4. Else → ZADD key now now                   -- Add current request
  5. EXPIRE key window                         -- Set TTL
```

### 10.3 Rate Limit Response Headers

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum requests allowed in window |
| `X-RateLimit-Remaining` | Requests remaining in current window |
| `X-RateLimit-Reset` | Unix timestamp when window resets |
| `Retry-After` | Seconds to wait (only on 429) |

### 10.4 Admin Rate Limit Monitoring Dashboard

| Column | Description |
|--------|-------------|
| Key | Rate limit Redis key |
| Type | Category (login, api, upload, etc.) |
| Identifier | IP address or user being limited |
| Count | Current request count in window |
| TTL | Time remaining before limit resets |
| Expiration | Exact expiration timestamp |
| Actions | Clear individual limit |

**Bulk Actions:**
- Clear by IP — remove all rate limits for a specific IP
- Clear by Type — remove all limits of a specific category
- Clear All — emergency reset of all rate limits system-wide

---

## 11. IP Allowlisting & Geo-Restriction

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 11.1 IP Allowlist Configuration (Per-Tenant)

| Feature | Description |
|---------|-------------|
| Scope | Per-tenant IP allowlists |
| Format | Individual IPs or CIDR ranges (e.g., `192.168.1.0/24`) |
| Storage | Database table (`ip_allowlist`) |
| Enforcement | Middleware check after authentication |
| Bypass | SUPER_ADMIN can bypass when allowlist is empty |
| Dev Bypass | `localhost` / `127.0.0.1` / `::1` always allowed in development |

### 11.2 IP Allowlist Table

```sql
CREATE TABLE ip_allowlist (
  id          TEXT PRIMARY KEY DEFAULT gen_random_uuid(),
  tenant_id   TEXT NOT NULL REFERENCES tenants(id),
  ip_address  TEXT NOT NULL,             -- Single IP or CIDR
  label       TEXT,                       -- Description (e.g., "Office VPN")
  added_by    TEXT REFERENCES users(id),
  created_at  TIMESTAMPTZ DEFAULT now(),
  UNIQUE(tenant_id, ip_address)
);
```

### 11.3 IP Allowlist Admin UI

| Action | Description |
|--------|-------------|
| View | List all allowlisted IPs with labels |
| Add | Add individual IP or CIDR with validation |
| Remove | Remove IP from allowlist |
| Bulk Import | Import IPs from CSV |
| Test | Check if a specific IP would be allowed |

### 11.4 Geo-Restriction (Optional)

| Feature | Description |
|---------|-------------|
| GeoIP Database | MaxMind GeoLite2 or IP2Location |
| Country Blocking | Block logins from specific countries |
| Geo-Anomaly Detection | Alert on unusual login locations (see Section 15) |
| Tracking Fields | `country`, `region`, `city`, `lat`, `lng` |

---

## 12. Security Headers & CSP

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 12.1 OWASP Security Headers

| Header | Value | Purpose |
|--------|-------|---------|
| `X-Frame-Options` | `DENY` | Clickjacking prevention |
| `X-Content-Type-Options` | `nosniff` | MIME sniffing prevention |
| `X-XSS-Protection` | `1; mode=block` | Legacy XSS filter |
| `Referrer-Policy` | `strict-origin-when-cross-origin` | Data leak prevention |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=(), interest-cohort=()` | Browser feature restriction |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` | HSTS enforcement |
| `Cache-Control` | `no-store, no-cache, must-revalidate` | Sensitive data caching prevention |
| `X-Permitted-Cross-Domain-Policies` | `none` | Flash/PDF cross-domain prevention |

### 12.2 Content Security Policy

```
default-src 'self';
script-src 'self' 'unsafe-eval' 'unsafe-inline';
style-src 'self' 'unsafe-inline';
img-src 'self' data: blob: https:;
font-src 'self' data:;
connect-src 'self' https://{DOMAIN} wss://{DOMAIN};
frame-ancestors 'none';
object-src 'none';
base-uri 'self';
form-action 'self';
upgrade-insecure-requests;
```

> **Note:** Remove `'unsafe-eval'` and `'unsafe-inline'` for stricter CSP when possible. Use nonce-based CSP for inline scripts.

### 12.3 Implementation

Headers applied via reverse proxy middleware on all responses:

```typescript
// {PATH_PREFIX}/middleware/security-headers.ts
function applySecurityHeaders(response: Response): Response {
  response.headers.set('X-Frame-Options', 'DENY');
  response.headers.set('X-Content-Type-Options', 'nosniff');
  // ... all headers from table above
  return response;
}
```

---

## 13. Audit Logging & Compliance

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Compliance |

### 13.1 Audit Log Entry Structure

```json
{
  "id": "audit-entry-id",
  "timestamp": "2025-01-01T12:00:00Z",
  "tenantId": "tenant-id",
  "userId": "user-id",
  "userName": "John Doe",
  "action": "ENTITY_UPDATE",
  "entityType": "Record",
  "entityId": "record-id",
  "status": "SUCCESS",
  "ipAddress": "192.168.1.1",
  "userAgent": "Mozilla/5.0...",
  "requestId": "req-uuid",
  "sessionId": "session-uuid",
  "phiAccessed": true,
  "phiFields": ["ssn", "dateOfBirth"],
  "accessReason": "Routine review",
  "changes": {
    "status": { "old": "PENDING", "new": "APPROVED" },
    "assignedTo": { "old": null, "new": "user-id-2" }
  },
  "metadata": {},
  "retainUntil": "2032-01-01T12:00:00Z"
}
```

### 13.2 Tracked Action Types (35+)

| Category | Actions |
|----------|---------|
| **Authentication** | `LOGIN_SUCCESS`, `LOGIN_FAILURE`, `LOGOUT`, `LOGIN_LOCKED`, `PASSWORD_CHANGE`, `PASSWORD_RESET`, `MFA_SETUP`, `MFA_VERIFY`, `MFA_DISABLE` |
| **User Management** | `USER_CREATE`, `USER_UPDATE`, `USER_DELETE`, `USER_ACTIVATE`, `USER_DEACTIVATE`, `ROLE_CHANGE`, `USER_INVITE`, `USER_IMPORT` |
| **Entity Operations** | `ENTITY_CREATE`, `ENTITY_READ`, `ENTITY_UPDATE`, `ENTITY_DELETE`, `ENTITY_ARCHIVE`, `ENTITY_EXPORT` |
| **Security Events** | `RATE_LIMIT_HIT`, `IP_BLOCKED`, `SUSPICIOUS_ACTIVITY`, `FORCE_LOGOUT`, `SESSION_REVOKE`, `ALLOWLIST_UPDATE` |
| **System Events** | `CONFIG_CHANGE`, `BACKUP_CREATE`, `BACKUP_RESTORE`, `INTEGRATION_UPDATE`, `WEBHOOK_FIRE` |
| **Data Access** | `PHI_ACCESS`, `REPORT_GENERATE`, `DATA_EXPORT`, `BULK_OPERATION` |

### 13.3 Logging Functions

| Function | Purpose |
|----------|---------|
| `log(action, details)` | Generic audit logging |
| `logSensitiveAccess(fields, reason)` | Sensitive data access with justification |
| `logAuth(action, success, details)` | Login/logout/password events |
| `logSecurityEvent(type, details)` | Unauthorized access, rate limits, threats |
| `logDataChange(entity, id, changes)` | Update with before/after value diff |

### 13.4 Audit Log Admin UI

**Path:** `{FRONTEND_PREFIX}/dashboard/admin/audit`

| Filter | Description |
|--------|-------------|
| Search | Full-text search by user, action, entity |
| Action Type | Filter by specific action (35+ types) |
| Entity Type | Filter by entity type |
| Status | SUCCESS / FAILURE / BLOCKED |
| Sensitive Data Only | Toggle to show only sensitive access events |
| Date Range | Start date to end date picker |
| User | Filter by specific user |

| Column | Description |
|--------|-------------|
| Timestamp | When the action occurred |
| User | Who performed the action |
| Action | What action was taken |
| Entity | Which entity type was affected |
| Entity ID | Specific record identifier |
| Status | Success (green) / Failure (red) / Blocked (orange) |
| Sensitive | Sensitive data access indicator |
| Details | Expandable detail view |

### 13.5 Export Options

| Format | Description |
|--------|-------------|
| CSV | Comma-separated values for spreadsheet analysis |
| JSON | Structured data for programmatic processing |
| Excel | Formatted spreadsheet with headers and styling |
| PDF | Printable report format with branding |

### 13.6 Compliance Features

| Feature | Description |
|---------|-------------|
| Retention Policy | `{RETENTION_YEARS}` years enforced via `retainUntil` field |
| Immutable Records | Audit logs cannot be modified or deleted |
| Sensitive Access Tracking | Field-level detection of sensitive data access |
| Access Reason | Business justification captured per sensitive access |
| Compliance Export | Formatted reports for regulatory audits |
| Statistics | Total logs, sensitive access count, failure rate, unique users |

### 13.7 Audit Statistics Dashboard

| Metric | Description |
|--------|-------------|
| Total Logs | All-time audit log count |
| Sensitive Access Count | Total sensitive data access events |
| Failed Actions | Total failed action count |
| Login Events | Total login-related events |
| Unique Active Users | Distinct users with recent activity |
| Action Breakdown | Top 10 actions by frequency (chart) |
| Entity Breakdown | Top 10 entities by access count (chart) |
| Sensitive Access Rate | Percentage of events involving sensitive data |
| Failure Rate | Percentage of failed events |

---

## 14. Multi-Tenant Isolation / Row-Level Security

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Security |

### 14.1 ORM Middleware RLS

Automatic `tenantId` filtering applied to all database operations:

| Operation | Behavior |
|-----------|----------|
| `findFirst` | Auto-adds `WHERE tenantId = ?` |
| `findMany` | Auto-adds `WHERE tenantId = ?` |
| `count` | Auto-adds `WHERE tenantId = ?` |
| `aggregate` | Auto-adds `WHERE tenantId = ?` |
| `groupBy` | Auto-adds `WHERE tenantId = ?` |
| `create` | Auto-injects `tenantId` into data |
| `update` | Filters by `tenantId` |
| `delete` | Filters by `tenantId` |
| `upsert` | Applies tenant context |

### 14.2 Tenant-Scoped Models

All business-domain models include a `tenantId` foreign key:

```
User, Entity, Record, Document, Message, Appointment,
Notification, AuditTrail, Setting, Invoice, Payment,
Report, Integration, Webhook, Tag, Category, Comment,
Attachment, Template, Workflow, ...
```

> Add your domain-specific models to this list.

### 14.3 SUPER_ADMIN Bypass

- SUPER_ADMIN bypasses all RLS filters (cross-tenant access)
- This is the **only** role with cross-tenant visibility
- All cross-tenant access is audit-logged

### 14.4 PostgreSQL Native RLS (Additional Layer)

```sql
-- Enable RLS on tenant-scoped tables
ALTER TABLE entities ENABLE ROW LEVEL SECURITY;

-- Policy for regular users
CREATE POLICY tenant_isolation ON entities
  USING (tenant_id = current_setting('app.tenant_id'));

-- Bypass for super admin
CREATE POLICY super_admin_bypass ON entities
  USING (current_setting('app.is_super_admin') = 'true');
```

### 14.5 Cross-Tenant Data Leakage Prevention

| Check | Description |
|-------|-------------|
| ORM Middleware | Automatic tenant filtering at application layer |
| Database RLS | PostgreSQL policies as defense-in-depth |
| API Validation | Tenant ID from JWT, never from client input |
| URL Validation | Tenant slug in URL must match session tenant |
| Join Protection | Cross-tenant joins blocked at ORM level |

---

## 15. Threat Detection & Suspicious Activity

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 15.1 Threat Types (6 Categories)

| Type | Detection Logic | Risk Level |
|------|----------------|------------|
| **Brute Force** | 3+ failed login attempts from same IP in 15 min | High |
| **Geographic Anomaly** | Login from unusual country (no prior history) | Medium |
| **Time-Based Anomaly** | Login outside normal hours (e.g., 1-5 AM local) | Low |
| **Device Anomaly** | Bot-like user agent or unknown device fingerprint | Medium |
| **Rapid Attempts** | < 5 second interval between consecutive attempts | High |
| **Credential Stuffing** | Multiple email attempts from single IP | Critical |

### 15.2 Suspicious IP Tracking

| Field | Description |
|-------|-------------|
| IP Address | Source IP of suspicious activity |
| Threat Type | Category from above |
| Risk Score | Calculated 0-100 score |
| First Seen | First suspicious activity timestamp |
| Last Seen | Most recent suspicious activity |
| Attempt Count | Total suspicious attempts |
| Status | Active / Blocked / Resolved |
| Action Taken | None / Alert / Block / Rate Limit |

### 15.3 Threat Response Actions

| Risk Level | Automatic Action |
|------------|-----------------|
| Low | Log only, visible in dashboard |
| Medium | Log + email alert to SUPER_ADMIN |
| High | Log + alert + temporary rate limit increase |
| Critical | Log + alert + temporary IP block + force password reset |

### 15.4 Security Overview Dashboard

| Metric | Description |
|--------|-------------|
| Active Threats | Current unresolved threats |
| Blocked IPs | Currently blocked IP addresses |
| Failed Logins (24h) | Failed login attempts in last 24 hours |
| Suspicious Countries | Countries with anomalous login patterns |
| Risk Score Trend | Average risk score over time (chart) |
| Top Threat Types | Breakdown by threat category (chart) |

---

## 16. Super Admin Dashboard Tabs

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Admin UI |

### 16.1 Dashboard Tab Inventory

| # | Tab | Path | Purpose | Update Frequency |
|---|-----|------|---------|-----------------|
| 1 | Statistics | `/dashboard/admin` | Operational KPIs, entity counts, revenue | Real-time (5s) |
| 2 | API Performance | `/dashboard/admin/api-performance` | P50/P95/P99 latency, error rates | Real-time (10s) |
| 3 | Security Center | `/dashboard/admin/security` | MFA, sessions, rate limits, threats | Real-time |
| 4 | System Monitor | `/dashboard/admin/system-monitor` | CPU, memory, disk, DB connections | Real-time (5s) |
| 5 | Business Intelligence | `/dashboard/admin/analytics` | Trends, forecasts, departmental stats | Hourly |
| 6 | User Management | `/dashboard/admin/users` | CRUD, roles, invitations, permissions | On-demand |
| 7 | Audit Logs | `/dashboard/admin/audit` | Compliance, sensitive access, exports | Real-time |
| 8 | Object Storage | `/dashboard/admin/storage` | File storage monitoring, configuration | On-demand |
| 9 | Email / SMTP | `/dashboard/admin/email` | SMTP config, test email, templates | On-demand |
| 10 | Integrations | `/dashboard/admin/integrations` | Webhooks, APIs, external connections | On-demand |
| 11 | System Configuration | `/dashboard/admin/system` | DB, security, branding, feature toggles | On-demand |
| 12 | Inactive Users | `/dashboard/admin/inactive-users` | 90+ day dormant account management | Daily |
| 13 | User Logs | `/dashboard/admin/user-logs` | Login history, geo-location, devices | On-demand |
| 14 | MFA Management | `/dashboard/admin/mfa` | MFA adoption, controls, reset | On-demand |
| 15 | Database Management | `/dashboard/admin/database` | Backup, restore, connection pool | On-demand |
| 16 | License Management | `/dashboard/admin/license` | Subscription tiers, keys, expiry | On-demand |
| 17 | Alert Rules | `/dashboard/admin/alert-rules` | Notification rules, triggers, channels | On-demand |
| 18 | Feature Flags | `/dashboard/admin/feature-flags` | Progressive rollout, kill switches | On-demand |
| 19 | Surveys | `/dashboard/admin/surveys` | Satisfaction templates, analytics | On-demand |

### 16.2 Statistics Dashboard Cards

```
┌────────────────┬────────────────┬────────────────┬──────────────────┐
│  Total Users   │  Active Today  │  Total Records │  Revenue (MTD)   │
│     1,247      │     342        │     18,920     │   {CURRENCY} 45K │
│   ↑12% vs LW   │   ↑8% vs LW    │   ↑5% vs LW    │   ↑15% vs LW     │
└────────────────┴────────────────┴────────────────┴──────────────────┘
```

### 16.3 Security Overview Cards

| Card | Metric | Description |
|------|--------|-------------|
| Total Users | Count | All registered users in tenant |
| Locked Accounts | Count | Currently locked user accounts |
| MFA Enabled | Count + % | Users with MFA active |
| Active Rate Limits | Count | IPs currently under rate limiting |
| Session Timeout | Minutes | Configured session timeout duration |
| IP Allowlist | Count | Whitelisted IP addresses |
| Active Sessions | Count | Currently active user sessions |
| Failed Logins (24h) | Count | Failed login attempts |

---

## 17. User Management

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Admin UI |

### 17.1 User List Table

| Column | Description |
|--------|-------------|
| Avatar | User profile image or initials |
| Name | Full name |
| Email | User email address |
| Role | Role chip (color-coded) |
| Status | Active (green) / Inactive (red) |
| MFA | Enabled / Disabled indicator |
| Last Login | Last login timestamp or "Never" |
| Created | Account creation date |
| Actions | Edit, delete, view details, reset password |

### 17.2 Search & Filters

| Filter | Options |
|--------|---------|
| Search | Name or email (partial match) |
| Role | All roles available as dropdown |
| Status | Active / Inactive / All |
| MFA Status | Enabled / Disabled / All |
| Last Login | Date range picker |
| Created | Date range picker |

### 17.3 User CRUD Operations

| Action | Description | Guards |
|--------|-------------|--------|
| Create User | New user with role assignment | Admin only |
| Edit User | Update name, phone, timezone, locale | Admin only |
| Change Role | Assign different role | Cannot demote SUPER_ADMIN |
| Toggle Status | Activate/deactivate account | Cannot change own status |
| Reset Password | Generate new password for user | Admin only, audit-logged |
| Delete User | Soft delete (preserves audit trail) | Cannot delete own account |
| Permanent Delete | Hard delete after 90+ days inactive | SUPER_ADMIN only |

### 17.4 Invitations Tab

| Feature | Description |
|---------|-------------|
| Send Invitation | Email with registration link |
| View Pending | List pending invitations with status |
| Resend | Resend invitation email |
| Copy Link | Copy invitation link to clipboard |
| Expire | Cancel pending invitation |
| Bulk Invite | Upload CSV with name, email, role |

### 17.5 Roles & Permissions Tab

Interactive permission matrix view:
- All 7 roles displayed as columns
- All 70+ permissions displayed as rows
- Visual checkmarks showing which role has which permission
- Grouped by permission category
- Read-only view (role definitions managed in code)

### 17.6 Inactive User Management

| Feature | Description |
|---------|-------------|
| Detection Threshold | 90+ days since last login |
| Color Coding | Green (active), Yellow (30-90 days), Red (90+ days) |
| Bulk Actions | Deactivate all inactive, send re-engagement email |
| Permanent Deletion | Remove user and all associated data |
| Audit Trail | All inactive user actions logged |
| Export | CSV of inactive users for review |

---

## 18. System Configuration

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Admin UI |

### 18.1 Database Configuration

| Field | Description |
|-------|-------------|
| Host | Database server hostname |
| Port | Database port (default: {DB_PORT}) |
| Database Name | Database name |
| User | Database username |
| Connection Pool Size | Max connections (default: 20) |
| SSL Mode | disable / require / verify-full |

### 18.2 SMTP Email Configuration

| Field | Description |
|-------|-------------|
| SMTP Host | Mail server hostname |
| SMTP Port | Mail server port (25/465/587) |
| Encryption | NONE / TLS / SSL |
| Username | SMTP authentication username |
| Password | SMTP authentication password (masked) |
| From Email | Sender email address |
| From Name | Sender display name |

### 18.3 Storage Providers

| Provider | Configuration Fields |
|----------|---------------------|
| MinIO (self-hosted) | Endpoint, access key, secret key, bucket, SSL toggle |
| AWS S3 | Region, bucket, access key, secret key |
| Azure Blob | Account name, container, connection string |
| Google Cloud Storage | Bucket, service account JSON |

### 18.4 Security Settings

| Setting | Type | Description |
|---------|------|-------------|
| Session Timeout | Number (minutes) | Minutes before session expires |
| Max Login Attempts | Number | Failed attempts before account lock |
| Lock Duration | Number (minutes) | Account lock duration |
| Password Min Length | Number | Minimum password length |
| MFA Requirement | Toggle | Require MFA for all users |
| IP Allowlist Enabled | Toggle | Enforce IP allowlisting |
| Force HTTPS | Toggle | Redirect HTTP to HTTPS |

### 18.5 Branding

| Setting | Type | Description |
|---------|------|-------------|
| Company Name | Text | Organization display name |
| Logo URL | File upload | Company logo |
| Favicon URL | File upload | Browser tab icon |
| Primary Color | Color picker | Brand color for UI |
| Default Timezone | Select | System-wide timezone |
| Default Locale | Select | System-wide language |
| Login Page Message | Rich text | Custom message on login page |

### 18.6 Feature Toggles

| Feature | Type | Description |
|---------|------|-------------|
| Feature A | Toggle | `{FEATURE_A_NAME}` |
| Feature B | Toggle | `{FEATURE_B_NAME}` |
| Feature C | Toggle | `{FEATURE_C_NAME}` |
| Feature D | Toggle | `{FEATURE_D_NAME}` |
| AI Features | Toggle | Enable AI-powered tools |
| API Access | Toggle | Enable external API access |
| Webhooks | Toggle | Enable webhook delivery |

> Customize feature toggle names with your project-specific features.

### 18.7 System Limits (Per-Tenant)

| Limit | Type | Description |
|-------|------|-------------|
| Max Users | Number | Maximum users per tenant |
| Max Records | Number | Maximum primary records |
| Storage Quota | Number (GB) | Maximum storage allocation |
| API Rate Limit | Number (req/min) | Custom API rate limit |
| Max File Size | Number (MB) | Maximum upload file size |

**Restart Requirements:** Database, storage, and security settings changes may require application restart. Mark fields that require restart in the UI.

---

## 19. Database Management

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Infrastructure |

### 19.1 Backup Types

| Type | Description | Size | Speed |
|------|-------------|------|-------|
| FULL | Complete database dump (schema + data) | Large | Slow |
| SCHEMA_ONLY | Schema definitions only (no data) | Small | Fast |
| DATA_ONLY | Data only (no schema) | Large | Medium |

### 19.2 Backup Operations

| Action | Description |
|--------|-------------|
| Create Backup | Trigger manual backup (pg_dump) |
| Schedule Backup | Cron-based automatic backups |
| Download | Download backup file |
| Upload to Storage | Push backup to S3/MinIO |
| List Backups | View all backups with timestamps and sizes |
| Delete Old Backups | Cleanup backups older than retention period |

### 19.3 Restore Operations

| Action | Description | Risk |
|--------|-------------|------|
| Restore Full | Replace entire database | High — requires confirmation |
| Restore Schema | Apply schema changes | Medium |
| Restore Data | Import data into existing schema | Medium |
| Point-in-Time | Restore to specific timestamp (WAL) | High |
| Dry Run | Validate backup without applying | None |

### 19.4 Connection Management

| Metric | Description |
|--------|-------------|
| Active Connections | Current open connections |
| Idle Connections | Connections in pool but unused |
| Max Connections | PostgreSQL `max_connections` setting |
| Pool Size | Application-level pool size |
| Connection Wait | Average wait time for connection from pool |

### 19.5 Database Encryption

| Layer | Algorithm | Key Management |
|-------|-----------|----------------|
| At Rest | AES-256-GCM | Environment variable or Vault |
| In Transit | TLS 1.3 | Certificate-based |
| Column-Level | AES-256-GCM | Application-layer encryption for sensitive fields |
| Backup | AES-256 | Separate backup encryption key |

### 19.6 Database Templates

| Template | Description |
|----------|-------------|
| Fresh Install | Empty schema with seed data |
| Demo Data | Schema + realistic demo data |
| Migration | Schema changes for version upgrade |

---

## 20. API Health Monitoring

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Infrastructure |

### 20.1 Per-Endpoint Metrics

| Metric | Description |
|--------|-------------|
| Endpoint | API route path |
| Method | HTTP method (GET, POST, PUT, DELETE) |
| Total Requests | Total call count |
| Success Rate | Percentage of 2xx responses |
| Error Rate | Percentage of 4xx/5xx responses |
| P50 Latency | 50th percentile response time |
| P95 Latency | 95th percentile response time |
| P99 Latency | 99th percentile response time |
| Avg Response Size | Average response body size (bytes) |

### 20.2 System Resource Metrics

| Metric | Description | Threshold |
|--------|-------------|-----------|
| Memory Usage | Heap used / heap total | < 80% |
| CPU Usage | Process CPU percentage | < 70% |
| Event Loop Lag | Node.js event loop delay | < 100ms |
| Active Handles | Open file descriptors, sockets | < 1000 |
| Uptime | Process uptime | N/A |

### 20.3 Error Dashboard

| Column | Description |
|--------|-------------|
| Endpoint | API route that errored |
| Error Code | HTTP status code |
| Error Message | Error description |
| Count | Occurrence count |
| Last Occurred | Most recent occurrence |
| Stack Trace | Error stack (expandable) |

### 20.4 Slowest Endpoints

| Column | Description |
|--------|-------------|
| Rank | Ordered by P95 latency |
| Endpoint | API route |
| P95 (ms) | 95th percentile response time |
| Avg (ms) | Average response time |
| Calls | Total request count |
| Trend | Improving / Degrading / Stable |

---

## 21. System Health Monitor

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Infrastructure |

### 21.1 Health Check Endpoint

```
GET /api/health

Response:
{
  "status": "healthy" | "degraded" | "unhealthy",
  "timestamp": "2025-01-01T12:00:00Z",
  "version": "{VERSION}",
  "uptime": "5d 12h 30m",
  "checks": {
    "database": { "status": "up", "latency": "3ms" },
    "cache": { "status": "up", "latency": "1ms" },
    "storage": { "status": "up", "latency": "5ms" },
    "queue": { "status": "up", "latency": "2ms" }
  }
}
```

### 21.2 System Metrics Dashboard

| Metric | Description | Alert Threshold |
|--------|-------------|-----------------|
| CPU Usage | System CPU percentage | > 80% |
| Memory Usage | RAM used / total | > 85% |
| Disk Usage | Disk used / total | > 90% |
| DB Connections | Active / max connections | > 80% |
| Cache Hit Rate | Redis cache effectiveness | < 90% |
| Queue Depth | Pending message count | > 1000 |
| Uptime | System uptime duration | N/A |
| Load Average | 1/5/15 minute load | > CPU count |

### 21.3 System Tools

| Tool | Description |
|------|-------------|
| Clear Cache | Flush Redis/Valkey cache |
| Test DB Connection | Verify database connectivity |
| Test Storage Connection | Verify S3/MinIO connectivity |
| Test SMTP Connection | Send test email |
| View Logs | Real-time application log viewer |
| Process Manager | View PM2/systemd process status |

### 21.4 Database Health

| Metric | Description |
|--------|-------------|
| Database Size | Total size on disk |
| Table Count | Number of tables |
| Largest Tables | Top 10 tables by size |
| Slow Queries | Queries exceeding threshold |
| Index Usage | Index hit rate percentage |
| Dead Tuples | Rows needing vacuum |
| Replication Lag | Replica delay (if applicable) |

---

## 22. Business Intelligence & Analytics

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Analytics |

### 22.1 Operational Metrics

| Category | Metrics |
|----------|---------|
| **User Engagement** | DAU, WAU, MAU, session duration, feature adoption |
| **Entity Performance** | Creation rate, completion rate, average processing time |
| **Revenue** | MRR, ARR, ARPU, churn rate, LTV |
| **Departmental** | Per-department/team throughput, efficiency, backlog |
| **Temporal** | Peak hours, day-of-week patterns, seasonal trends |

### 22.2 Report Types

| Report | Description | Frequency |
|--------|-------------|-----------|
| Executive Summary | High-level KPIs and trends | Weekly |
| User Activity | Login patterns, feature usage | Daily |
| Revenue Analysis | Financial metrics by segment | Monthly |
| Performance | System and team efficiency | Weekly |
| Compliance | Audit summary, access reviews | Monthly |
| Custom | Ad-hoc report builder | On-demand |

### 22.3 Chart Types (Recharts / Chart.js)

| Chart | Use Case |
|-------|----------|
| Line | Trends over time (users, revenue) |
| Bar | Comparisons (departments, categories) |
| Pie/Donut | Distribution (roles, statuses) |
| Area | Cumulative metrics |
| Heatmap | Activity patterns (time × day) |
| Funnel | Conversion/workflow stages |

### 22.4 Export Options

| Format | Description |
|--------|-------------|
| PDF | Branded report with charts |
| Excel | Spreadsheet with pivot-ready data |
| CSV | Raw data for external tools |
| API | JSON endpoint for BI tool integration |
| Scheduled Email | Automatic report delivery |

---

## 23. Analytics Integration (Privacy-First)

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Low | Analytics |

### 23.1 Recommended Analytics Platforms

| Platform | Type | Privacy | Cost |
|----------|------|---------|------|
| Matomo | Self-hosted | Full control | Free (self-hosted) |
| Plausible | Self-hosted or cloud | Privacy-first | Free (self-hosted) |
| PostHog | Self-hosted or cloud | Full control | Free tier available |
| Umami | Self-hosted | Privacy-first | Free |

### 23.2 Privacy Controls

| Feature | Description |
|---------|-------------|
| Do Not Track | Respect browser DNT header |
| Cookie-less Tracking | Fingerprint-free analytics option |
| IP Anonymization | Hash or truncate IP addresses |
| Data Residency | Self-hosted = data stays on your server |
| Opt-Out | User-facing opt-out mechanism |
| Consent Banner | GDPR-compliant consent collection |
| Data Retention | Configurable auto-delete after N months |

### 23.3 Compliance Compatibility

| Regulation | Requirement | Implementation |
|------------|-------------|----------------|
| GDPR | Consent, data minimization | Opt-in analytics, anonymization |
| HIPAA | PHI protection | No PHI in analytics, self-hosted |
| CCPA | Right to opt-out | Opt-out mechanism, data deletion |
| SOC 2 | Audit controls | Self-hosted with access logs |

---

## 24. Object Storage Management

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Infrastructure |

### 24.1 Storage Dashboard

| Metric | Description |
|--------|-------------|
| Total Storage Used | Bytes across all buckets |
| File Count | Total number of stored objects |
| Bucket Count | Number of storage buckets |
| Largest Files | Top 10 files by size |
| Storage Trend | Usage over time (chart) |
| Upload Rate | Files uploaded per day |

### 24.2 Storage Configuration

| Field | Description |
|-------|-------------|
| Provider | MinIO / AWS S3 / Azure Blob / GCS |
| Endpoint | Storage server URL |
| Access Key | Authentication key |
| Secret Key | Authentication secret (masked) |
| Bucket | Default bucket name |
| Region | Storage region |
| SSL | Enable HTTPS for storage connections |

### 24.3 Bucket Management

| Action | Description |
|--------|-------------|
| List Buckets | View all buckets with sizes |
| Create Bucket | Create new storage bucket |
| Set Policy | Configure access policies (public/private) |
| Enable Versioning | File version history |
| Lifecycle Rules | Auto-delete or archive old files |

### 24.4 File Restrictions

| Restriction | Value |
|-------------|-------|
| Max File Size | Configurable (default: 50 MB) |
| Allowed Types | Configurable MIME type allowlist |
| Filename Sanitization | Remove `../`, special characters |
| Virus Scanning | Optional ClamAV integration |
| Rate Limit | 20 uploads per minute per user |

### 24.5 Test Connection

Admin can test storage connectivity:
- Verify credentials
- Check bucket exists
- Test upload/download
- Measure latency

---

## 25. Email / SMTP Configuration

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Communications |

### 25.1 SMTP Configuration Fields

| Field | Description | Example |
|-------|-------------|---------|
| SMTP Host | Mail server hostname | smtp.gmail.com |
| SMTP Port | Mail server port | 587 |
| Encryption | NONE / TLS / SSL | TLS |
| Username | SMTP auth username | user@example.com |
| Password | SMTP auth password (masked) | ******** |
| From Email | Sender address | noreply@{DOMAIN} |
| From Name | Sender display name | {PROJECT_NAME} |

### 25.2 Common SMTP Providers

| Provider | Host | Port | Notes |
|----------|------|------|-------|
| Gmail | smtp.gmail.com | 587 | Requires App Password |
| SendGrid | smtp.sendgrid.net | 587 | API key as password |
| Mailgun | smtp.mailgun.org | 587 | Domain verification required |
| AWS SES | email-smtp.{region}.amazonaws.com | 587 | IAM credentials |
| Outlook | smtp-mail.outlook.com | 587 | Microsoft 365 |
| Custom | {host} | {port} | Self-hosted SMTP |

### 25.3 Test Email

Admin can send a test email to verify SMTP configuration:
- Recipient: admin's own email
- Subject: `[{PROJECT_NAME}] SMTP Test`
- Body: Confirmation with timestamp and server details
- Reports success/failure with error details

### 25.4 Inbound Email Integration (Optional)

| Feature | Description |
|---------|-------------|
| IMAP Monitoring | Poll inbox for incoming emails |
| Auto-Processing | Parse email → create record/ticket |
| Smart Priority | AI-based priority detection from subject/body |
| Attachment Handling | Save attachments to object storage |
| Reply Threading | Match replies to existing records |

### 25.5 Email Templates

| Template | Trigger |
|----------|---------|
| Welcome | New user registration |
| Invitation | User invitation sent |
| Password Reset | Password reset requested |
| MFA Setup | MFA enabled confirmation |
| Account Locked | Account lockout notification |
| License Key | License issued/renewed |
| Alert | System alert notification |

---

## 26. Webhook & Integration System

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Integrations |

### 26.1 Webhook Configuration

| Field | Description |
|-------|-------------|
| URL | Destination endpoint (HTTPS required) |
| Secret | HMAC-SHA256 signing secret |
| Events | Subscribed event types |
| Status | Active / Paused / Disabled |
| Retry Policy | Max retries, backoff strategy |
| Timeout | Request timeout (default: 30s) |

### 26.2 Webhook Events

| Event | Trigger |
|-------|---------|
| `user.created` | New user registered |
| `user.updated` | User profile changed |
| `user.deleted` | User deleted |
| `entity.created` | New entity created |
| `entity.updated` | Entity modified |
| `entity.deleted` | Entity deleted |
| `record.status_changed` | Record status transition |
| `payment.completed` | Payment processed |
| `subscription.changed` | License/subscription updated |
| `security.alert` | Security event triggered |

### 26.3 Webhook Payload Format

```json
{
  "id": "webhook-delivery-id",
  "event": "entity.created",
  "timestamp": "2025-01-01T12:00:00Z",
  "tenantId": "tenant-id",
  "data": {
    "id": "entity-id",
    "type": "Entity",
    "attributes": { ... }
  },
  "signature": "sha256=..."
}
```

### 26.4 HMAC-SHA256 Signing

```
Signature = HMAC-SHA256(webhook_secret, JSON.stringify(payload))
Header: X-Webhook-Signature: sha256={signature}
```

Recipient verifies:
1. Extract signature from header
2. Compute HMAC of raw body with shared secret
3. Timing-safe comparison

### 26.5 Retry & Auto-Disable

| Parameter | Value |
|-----------|-------|
| Max Retries | 5 |
| Backoff | Exponential (1s, 2s, 4s, 8s, 16s) |
| Timeout | 30 seconds |
| Auto-Disable | After 50 consecutive failures |
| Re-Enable | Manual re-enable from admin dashboard |

### 26.6 Delivery Log

| Column | Description |
|--------|-------------|
| Delivery ID | Unique delivery identifier |
| Event | Event type |
| URL | Destination endpoint |
| Status | Success / Failed / Pending |
| Response Code | HTTP response code |
| Response Time | Delivery latency (ms) |
| Attempts | Number of delivery attempts |
| Last Attempt | Most recent attempt timestamp |
| Error | Error message (if failed) |

---

## 27. Notification Channels

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Communications |

### 27.1 Supported Channels

| Channel | Provider | Status |
|---------|----------|--------|
| Email | SMTP (see Section 25) | To Do / Done |
| In-App | WebSocket / SSE | To Do / Done |
| Push | Web Push API / FCM | To Do / Done |
| SMS | Twilio / Vonage | To Do / Done |
| WhatsApp | Twilio WhatsApp API | To Do / Done |
| Slack | Slack Webhooks | To Do / Done |
| Microsoft Teams | Teams Webhooks | To Do / Done |

### 27.2 SMS Configuration (Twilio)

| Field | Description |
|-------|-------------|
| Account SID | Twilio account identifier |
| Auth Token | Twilio auth token (masked) |
| From Number | Twilio phone number |
| Status | Enabled / Disabled |

### 27.3 WhatsApp Configuration (Twilio)

| Field | Description |
|-------|-------------|
| Account SID | Twilio account identifier |
| Auth Token | Twilio auth token (masked) |
| WhatsApp Number | Twilio WhatsApp-enabled number |
| Template Messages | Pre-approved message templates |
| Status | Enabled / Disabled |

### 27.4 Alert Rules Configuration

| Field | Options |
|-------|---------|
| Trigger Type | Entity created, status changed, threshold exceeded, scheduled, security event |
| Priority | Low, Normal, High, Critical |
| Channels | Select notification channels (multi-select) |
| Target Roles | Select which roles receive the alert |
| Schedule | Immediate / Batched (hourly/daily digest) |
| Status | Enable / Disable per rule |
| Quiet Hours | Suppress non-critical alerts during off-hours |

### 27.5 Notification Preferences (Per-User)

| Setting | Description |
|---------|-------------|
| Email Notifications | Enable/disable email alerts |
| Push Notifications | Enable/disable browser push |
| SMS Notifications | Enable/disable SMS (if configured) |
| Quiet Hours | User-defined quiet hours |
| Digest Frequency | Immediate / Hourly / Daily |
| Channel Preferences | Per-event-type channel selection |

---

## 28. Feature Flags & Progressive Delivery

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Advanced |

### 28.1 Flag Types

| Type | Description | Example |
|------|-------------|---------|
| Boolean | Simple on/off toggle | `enable_new_dashboard` |
| Percentage | Gradual rollout (0-100%) | `rollout_new_editor: 25%` |
| Tenant-Specific | Enabled for specific tenants only | `beta_feature: [tenant-a, tenant-b]` |
| Role-Specific | Enabled for specific roles | `admin_tools: [SUPER_ADMIN, TENANT_ADMIN]` |
| User-Specific | Enabled for specific users | `alpha_tester: [user-1, user-2]` |
| Time-Based | Enabled between start/end dates | `holiday_theme: Dec 20 - Jan 5` |

### 28.2 Flag Configuration

| Field | Description |
|-------|-------------|
| Key | Unique flag identifier (snake_case) |
| Name | Human-readable name |
| Description | Purpose and impact description |
| Type | Boolean / Percentage / Tenant / Role / User / Time |
| Value | Current flag value |
| Default | Fallback value if evaluation fails |
| Created By | Admin who created the flag |
| Last Modified | Most recent change timestamp |
| Status | Active / Archived |

### 28.3 Kill Switch

A special boolean flag type for emergency feature disablement:

| Feature | Description |
|---------|-------------|
| Instant Disable | One-click disable of any feature |
| No Cache | Kill switch bypasses all caches |
| Audit Log | All kill switch activations logged |
| Notification | Alert sent to all admins on activation |
| Rollback | Automatically re-enables after configurable timeout (optional) |

### 28.4 A/B Testing Support

| Feature | Description |
|---------|-------------|
| Variants | Multiple variants per flag (A, B, C...) |
| Traffic Split | Percentage allocation per variant |
| Sticky Assignment | User sees same variant consistently (hash-based) |
| Metrics Tracking | Conversion/engagement per variant |
| Statistical Significance | Auto-detect winner |

### 28.5 Flag Evaluation Order

```
1. Kill switch override (highest priority)
2. User-specific override
3. Tenant-specific override
4. Role-specific override
5. Percentage rollout (deterministic hash of userId)
6. Time-based rule
7. Default value (lowest priority)
```

---

## 29. License & Subscription Management

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Advanced |

### 29.1 License Tiers

| Tier | Max Users | Storage | Modules | Support |
|------|-----------|---------|---------|---------|
| Free | 5 | 1 GB | Core only | Community |
| Starter | 25 | 10 GB | Core + Analytics | Email |
| Professional | 100 | 100 GB | All standard | Priority email |
| Enterprise | Unlimited | Unlimited | All + Custom | Dedicated |
| Trial | 10 | 5 GB | All | Email (14 days) |

### 29.2 License Key Format

```
{PREFIX}::V{VERSION}::{TIER}::{ENCRYPTED_PAYLOAD}::{SIGNATURE}

Example: APP::V2::PROFESSIONAL::base64payload==::hmacsha256sig==
```

### 29.3 License Lifecycle

```
1. Customer created → License generated
2. License key sent via email
3. License activated on first use
4. Periodic validation (daily cron)
5. Expiry warning (30/14/7/1 days before)
6. Grace period (7 days after expiry)
7. Downgrade to Free tier (after grace period)
8. Renewal → new license key issued
```

### 29.4 License Validation Middleware

```
Every request:
  1. Read license from database
  2. Detect version (V1 / V2)
  3. Validate signature (HMAC-SHA256 or ML-KEM)
  4. Check expiry date
  5. Check module access
  6. Check user count limit
  7. Check storage quota
  8. Allow or reject request
```

### 29.5 Subscription Management

| Feature | Description |
|---------|-------------|
| Payment Integration | Stripe / Razorpay / PayPal |
| Auto-Renewal | Automatic subscription renewal |
| Upgrade/Downgrade | Self-service tier changes |
| Proration | Prorated charges on mid-cycle changes |
| Invoice Generation | Automatic invoice creation |
| Usage Tracking | Track resource usage against limits |

### 29.6 License Encryption (Post-Quantum Ready)

| Version | Encryption | Quantum Safe |
|---------|-----------|--------------|
| V1 (Legacy) | AES-256-GCM + HMAC-SHA256 | No |
| V2 (Current) | ML-KEM-768 + AES-256-GCM + HMAC-SHA-512 | Yes (NIST FIPS 203) |

### 29.7 Auto-Downgrade Cron

```
Schedule: Daily at 00:00 UTC
  1. Find expired licenses past grace period
  2. Downgrade to Free tier
  3. Send notification to customer
  4. Log downgrade in audit trail
  5. Disable premium modules
```

---

## 30. Training & Compliance Management

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Low | Advanced |

### 30.1 Training Programs

| Feature | Description |
|---------|-------------|
| Program Creation | Create training programs with modules |
| Content Types | Video, document, quiz, interactive |
| Enrollment | Assign users to programs by role |
| Progress Tracking | Per-user completion percentage |
| Deadlines | Required completion dates |
| Reminders | Automated reminder emails |

### 30.2 Certification Management

| Feature | Description |
|---------|-------------|
| Certificate Templates | Customizable completion certificates |
| Expiry Tracking | Certificates with expiration dates |
| Renewal Alerts | Automated alerts before expiry (30/14/7 days) |
| Compliance Reports | Training completion by department/role |
| Bulk Assignment | Assign training to all users in a role |

### 30.3 Compliance Dashboard

| Metric | Description |
|--------|-------------|
| Overall Completion Rate | Percentage of assigned training completed |
| Overdue Training | Users with past-due training |
| Expiring Certifications | Certs expiring in next 30 days |
| Department Compliance | Completion rate per department |
| Training History | Per-user training timeline |

---

## 31. SLA Management

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Advanced |

### 31.1 SLA Policy Configuration

| Field | Description |
|-------|-------------|
| Policy Name | Human-readable SLA policy name |
| Priority | Low / Medium / High / Critical |
| First Response Target | Time to first response (e.g., 4 hours) |
| Resolution Target | Time to resolution (e.g., 24 hours) |
| Business Hours | Enable business hours calculation |
| Escalation Rules | Auto-escalate on breach |

### 31.2 Priority Targets (Example)

| Priority | First Response | Resolution | Escalation |
|----------|---------------|------------|------------|
| Critical | 15 minutes | 4 hours | Immediate to manager |
| High | 1 hour | 8 hours | After 4 hours |
| Medium | 4 hours | 24 hours | After 12 hours |
| Low | 8 hours | 72 hours | After 48 hours |

### 31.3 Business Hours Configuration

| Field | Description |
|-------|-------------|
| Work Days | Monday-Friday (configurable) |
| Start Time | 09:00 (configurable) |
| End Time | 17:00 (configurable) |
| Timezone | {TIMEZONE} |
| Holidays | List of excluded dates |
| 24/7 Mode | Disable business hours for critical priority |

### 31.4 SLA Compliance Dashboard

| Metric | Description |
|--------|-------------|
| SLA Compliance Rate | % of records meeting SLA targets |
| Average Response Time | Mean first response time |
| Average Resolution Time | Mean resolution time |
| Breach Count | Records that exceeded SLA targets |
| At-Risk Count | Records approaching SLA breach |
| By Priority | Compliance breakdown per priority |
| By Team | Compliance breakdown per team |

---

## 32. Customer / User Satisfaction Surveys

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Low | Advanced |

### 32.1 Survey Templates

| Template | Questions | Scale |
|----------|-----------|-------|
| CSAT | "How satisfied are you with this service?" | 1-5 stars |
| NPS | "How likely are you to recommend us?" | 0-10 scale |
| CES | "How easy was it to get your issue resolved?" | 1-7 scale |
| Custom | User-defined questions | Configurable |

### 32.2 Survey Deployment

| Trigger | Description |
|---------|-------------|
| Record Closed | Auto-send after record/ticket closed |
| Scheduled | Periodic satisfaction check |
| Manual | Admin-triggered survey |
| Milestone | After specific workflow milestone |
| Time-Based | After N days of service usage |

### 32.3 Survey Analytics

| Metric | Description |
|--------|-------------|
| Response Rate | Percentage of surveys completed |
| Average Score | Mean satisfaction score |
| NPS Score | Net Promoter Score (-100 to 100) |
| Trend | Score trend over time (chart) |
| By Category | Scores broken down by department/type |
| Comments | Free-text feedback with sentiment analysis |
| Actionable Insights | Auto-generated improvement suggestions |

### 32.4 Survey Aggregation

| View | Description |
|------|-------------|
| Overall | Organization-wide satisfaction |
| By Team | Per-team satisfaction scores |
| By Agent/Operator | Per-user satisfaction scores |
| By Category | Per-category satisfaction |
| Time-Based | Daily / Weekly / Monthly trends |
| Benchmark | Compare against industry averages |

---

## 33. Organization Hierarchy

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Advanced |

### 33.1 Hierarchy Levels

| Level | Example | Description |
|-------|---------|-------------|
| 1 | Organization | Top-level entity (tenant) |
| 2 | Division | Major business division |
| 3 | Department | Functional department |
| 4 | Team | Working team |
| 5 | Unit | Smallest organizational unit |

### 33.2 Organization Unit Fields

| Field | Description |
|-------|-------------|
| ID | Unique identifier |
| Name | Unit display name |
| Code | Short code (e.g., `ENG`, `MKT`) |
| Parent ID | Reference to parent unit |
| Level | Hierarchy depth (1-5) |
| Manager | Assigned manager (user reference) |
| User Count | Number of assigned users |
| Status | Active / Inactive |

### 33.3 Views

| View | Description |
|------|-------------|
| Tree View | Hierarchical tree with expand/collapse |
| List View | Flat table with parent column |
| Chart View | Organizational chart visualization |

### 33.4 Data Isolation

| Feature | Description |
|---------|-------------|
| Unit-Level Access | Users see only their unit's data |
| Manager Visibility | Managers see their unit + sub-units |
| Cross-Unit Reports | Only available to TENANT_ADMIN+ |
| Unit Assignment | Users assigned to exactly one unit |

---

## 34. User Import / Export (CSV)

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Operations |

### 34.1 Import Features

| Feature | Description |
|---------|-------------|
| Template Download | Pre-formatted CSV template with headers |
| Validation | Pre-import validation with error report |
| Duplicate Detection | Skip or update existing users (by email) |
| Role Assignment | Map CSV column to role |
| Batch Processing | Process large files in chunks |
| Error Report | Downloadable report of failed rows |
| Dry Run | Validate without creating users |

### 34.2 CSV Import Template

```csv
name,email,role,department,phone,timezone
John Doe,john@example.com,STAFF,Engineering,+1234567890,America/New_York
Jane Smith,jane@example.com,MANAGER,Marketing,+0987654321,Europe/London
```

### 34.3 Import Validation Rules

| Rule | Description |
|------|-------------|
| Email Format | Valid email address |
| Email Unique | No duplicate within tenant |
| Role Valid | Must be a valid role enum |
| Name Required | Non-empty, 2-100 characters |
| Phone Format | E.164 format (optional) |
| Timezone Valid | IANA timezone string (optional) |

### 34.4 Export Features

| Feature | Description |
|---------|-------------|
| Filtered Export | Export only filtered subset of users |
| Column Selection | Choose which columns to export |
| PII Masking | Option to mask sensitive fields (email, phone) |
| Format | CSV with UTF-8 BOM for Excel compatibility |
| Audit Log | Export action logged |

---

## 35. Configuration Change Logging

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Operations |

### 35.1 Tracked Configuration Categories

| Category | Examples |
|----------|---------|
| Security Settings | Session timeout, MFA requirement, lockout rules |
| SMTP Configuration | Host, port, credentials changes |
| Storage Configuration | Bucket, endpoint, credentials changes |
| Branding | Company name, logo, colors |
| Feature Toggles | Feature flag changes |
| Integration Settings | Webhook URLs, API keys |
| System Limits | Max users, storage quota changes |
| License | Tier changes, key rotation |

### 35.2 Change Log Entry Structure

```json
{
  "id": "change-id",
  "timestamp": "2025-01-01T12:00:00Z",
  "adminUser": "admin-user-id",
  "adminName": "John Admin",
  "category": "security",
  "setting": "session_timeout",
  "oldValue": "60",
  "newValue": "30",
  "ipAddress": "192.168.1.1",
  "userAgent": "Mozilla/5.0..."
}
```

### 35.3 Change Log UI

| Column | Description |
|--------|-------------|
| Timestamp | When the change was made |
| Admin | Who made the change |
| Category | Configuration category |
| Setting | Specific setting name |
| Old Value | Previous value |
| New Value | New value |
| IP Address | Admin's IP address |

### 35.4 Rollback Support

| Feature | Description |
|---------|-------------|
| View History | Full history per setting |
| Compare | Side-by-side old vs new values |
| Revert | One-click revert to previous value |
| Revert Audit | Revert action is also logged |

---

## 36. Performance Optimization

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Operations |

### 36.1 Frontend Performance

| Technique | Description |
|-----------|-------------|
| Code Splitting | Dynamic imports for route-based splitting |
| Lazy Loading | Components loaded on demand |
| Virtual Scrolling | Render only visible rows in large lists |
| Image Optimization | Next/Image with WebP, responsive sizes |
| Bundle Analysis | Webpack/Turbo bundle size tracking |
| Service Worker | PWA caching for offline support |

### 36.2 Backend Performance

| Technique | Description |
|-----------|-------------|
| Query Optimization | Strategic database indexes |
| Connection Pooling | Prisma/pg connection pool |
| Response Caching | Redis cache for read-heavy endpoints |
| Request Batching | Combine multiple queries into one |
| Pagination | Cursor-based pagination for large datasets |
| Background Jobs | Queue-based processing for heavy operations |

### 36.3 Caching Strategy

| Layer | Technology | TTL | Invalidation |
|-------|-----------|-----|-------------|
| CDN | Cloudflare | Varies | Purge API |
| Application | Redis/Valkey | 5-60 min | Event-based |
| Query | TanStack Query | 5-30 min | Stale-while-revalidate |
| Static Assets | Service Worker | 30 days | Version hash |

### 36.4 Core Web Vitals Targets

| Metric | Target | Description |
|--------|--------|-------------|
| LCP | < 2.5s | Largest Contentful Paint |
| FID / INP | < 100ms | First Input Delay / Interaction to Next Paint |
| CLS | < 0.1 | Cumulative Layout Shift |
| TTFB | < 200ms | Time to First Byte |

### 36.5 Performance Monitoring

| Tool | Purpose |
|------|---------|
| Lighthouse | Automated performance audits |
| Web Vitals | Real User Monitoring (RUM) |
| Sentry Performance | APM and transaction tracing |
| pg_stat_statements | PostgreSQL query performance |

---

## 37. Security Testing Suite

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Medium | Operations |

### 37.1 Test Categories (8 Types)

| Category | Description | Tool |
|----------|-------------|------|
| SAST | Static Application Security Testing | SonarQube, CodeQL |
| DAST | Dynamic Application Security Testing | OWASP ZAP |
| SCA | Software Composition Analysis | npm audit, Snyk |
| RBAC Testing | Permission boundary verification | Custom test suite |
| Privilege Escalation | Horizontal/vertical privilege checks | Custom test suite |
| Session Security | Hijacking, fixation, replay testing | Custom test suite |
| Input Fuzzing | Malformed input injection testing | Custom test suite |
| API Security | Endpoint authentication/authorization | Custom test suite |

### 37.2 CI/CD Security Pipeline

```
Push to main:
  ├─ Job 1: Dependency Scan (npm audit)        ~2 min
  ├─ Job 2: Static Analysis (SonarQube SAST)   ~5-10 min
  ├─ Job 3: Dynamic Scan (OWASP ZAP DAST)      ~15-30 min
  └─ Job 4: Code Scanning (GitHub CodeQL)       ~5-10 min
      │
      ├─ All pass → Deploy allowed
      └─ Any fail → Block deployment + alert

Weekly Scheduled (e.g., Monday 2 AM):
  └─ Full security scan suite with report generation
```

### 37.3 Quality Gates

| Metric | Threshold |
|--------|-----------|
| Critical Vulnerabilities | 0 |
| High Vulnerabilities | 0 |
| Medium Vulnerabilities | < 10 |
| Security Rating | A |
| Test Coverage | > 80% |
| Code Duplication | < 3% |
| Technical Debt | < 5 days |
| Code Smells | < 100 |

### 37.4 OWASP Top 10 Coverage

| # | Vulnerability | Status | Mitigation |
|---|--------------|--------|------------|
| A01 | Broken Access Control | Protected | RBAC + RLS + tenant isolation |
| A02 | Cryptographic Failures | Protected | TLS 1.3, AES-256, bcrypt |
| A03 | Injection | Protected | Parameterized queries (ORM), input validation |
| A04 | Insecure Design | Protected | Threat modeling, secure defaults |
| A05 | Security Misconfiguration | Protected | Env validation, setup wizard |
| A06 | Vulnerable Components | Mitigated | Automated dependency scanning |
| A07 | Auth Failures | Protected | MFA, bcrypt, session management, lockout |
| A08 | Data Integrity Failures | Protected | HMAC signatures, code signing |
| A09 | Logging Failures | Protected | Comprehensive audit logging |
| A10 | SSRF | Protected | URL validation, network segmentation |

### 37.5 Scan Reports

| Format | Description |
|--------|-------------|
| HTML | Visual report with severity charts |
| JSON | Machine-readable for CI/CD integration |
| XML | Standard interchange format |
| Markdown | For documentation and PRs |
| PDF | Printable report for compliance |

---

## 38. Infrastructure Security

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Reference |

### 38.1 Zero Trust Principles

| Principle | Implementation |
|-----------|----------------|
| **Verify Explicitly** | Authenticate and authorize every request; no implicit trust |
| **Least Privilege** | Minimum permissions needed for each role; time-limited access |
| **Assume Breach** | Defense in depth; micro-segmentation; blast radius minimization |

### 38.2 Network Architecture

```
Internet
  │
  ├─ Layer 3-4 DDoS Protection (Cloudflare / AWS Shield)
  │   └─ WAF Rules (OWASP Core Rule Set)
  │       └─ Encrypted Tunnel (Cloudflare Tunnel / WireGuard)
  │           └─ Tunnel Daemon on VM
  │
  ├─ VPN Layer (Tailscale / WireGuard)
  │   └─ Admin-only access to management interfaces
  │
  └─ Firewall (pfSense / iptables / Security Groups)
      └─ Host VM
          ├─ Application (port {APP_PORT}, process manager)
          ├─ Database (port {DB_PORT}, Docker/native)
          │   └─ Accepts connections only from localhost
          ├─ Cache (port {CACHE_PORT}, Docker/native)
          │   └─ Bound to 127.0.0.1 only
          ├─ Object Storage (port {STORAGE_PORT})
          │   └─ Internal network only
          ├─ Message Queue (port {QUEUE_PORT})
          │   └─ Internal network only
          └─ AI Models (port {AI_PORT})
              └─ Local inference only, no data leaves server
```

### 38.3 Encryption Standards

| Data State | Algorithm | Key Length | Rotation |
|------------|-----------|------------|----------|
| In Transit | TLS 1.3 | 256-bit | Per-session |
| At Rest (DB) | AES-256-GCM | 256-bit | Annual |
| At Rest (Files) | AES-256-CBC | 256-bit | Annual |
| Passwords | bcrypt | 12 rounds | Per-change |
| Sessions | HMAC-SHA256 | 256-bit | Per-session |
| License Keys | ML-KEM-768 + AES-256-GCM | Post-quantum | Per-generation |
| Webhook Signatures | HMAC-SHA256 | 256-bit | Per-webhook |
| Backup Encryption | AES-256 | 256-bit | Per-backup |

### 38.4 IP Extraction Priority

```
1. X-Forwarded-For header (CDN / reverse proxy)
2. X-Real-IP header (Nginx proxy)
3. CF-Connecting-IP header (Cloudflare-specific)
4. socket.remoteAddress (direct connection)
```

### 38.5 Secrets Management

| Approach | Use Case | Tools |
|----------|----------|-------|
| Environment Variables | Development, simple deployments | `.env` files (gitignored) |
| Secret Manager | Production, team environments | HashiCorp Vault, AWS Secrets Manager, GCP Secret Manager |
| Key Management Service | Encryption keys | AWS KMS, Azure Key Vault, GCP KMS |
| Certificate Manager | TLS certificates | Let's Encrypt, AWS ACM |

### 38.6 Access Methods Summary

| URL | Method | Purpose | Who |
|-----|--------|---------|-----|
| `https://{DOMAIN}` | CDN Tunnel | Public access | All users |
| `http://{TAILSCALE_IP}:{APP_PORT}` | VPN | Admin/dev access | Admins |
| `http://localhost:{APP_PORT}` | Direct | Local dev | Developers |
| `ssh://{TAILSCALE_IP}:22` | SSH over VPN | Server management | Ops |

---

## 39. Environment Variable Validation

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Reference |

### 39.1 Fail-Fast Startup Validation

Application validates all required environment variables at startup and refuses to start if critical variables are missing or malformed.

### 39.2 Required Variables

| Variable | Type | Validation | Critical |
|----------|------|------------|----------|
| `DATABASE_URL` | String | Valid PostgreSQL connection string | Yes |
| `JWT_SECRET` | String | Min 32 characters | Yes |
| `JWT_REFRESH_SECRET` | String | Min 32 characters, different from JWT_SECRET | Yes |
| `REDIS_URL` | String | Valid Redis connection string | Yes |
| `APP_URL` | String | Valid URL format | Yes |
| `NODE_ENV` | Enum | `development` / `staging` / `production` | Yes |

### 39.3 Optional Variables with Defaults

| Variable | Type | Default | Validation |
|----------|------|---------|------------|
| `PORT` | Number | {APP_PORT} | 1-65535 |
| `SESSION_TIMEOUT` | Number | 60 | 5-1440 (minutes) |
| `MAX_LOGIN_ATTEMPTS` | Number | 5 | 1-20 |
| `LOCK_DURATION` | Number | 15 | 1-1440 (minutes) |
| `BCRYPT_ROUNDS` | Number | 12 | 10-15 |
| `LOG_LEVEL` | Enum | `info` | `debug`/`info`/`warn`/`error` |

### 39.4 Production Enforcement

| Check | Description |
|-------|-------------|
| HTTPS Required | `APP_URL` must use `https://` in production |
| Strong Secrets | JWT secrets must be 64+ characters in production |
| No Default Passwords | Reject known default values |
| SSL Mode | Database SSL required in production |
| Secure Cookies | Cookie `Secure` flag enforced |
| Debug Disabled | Debug/verbose logging disabled |

### 39.5 Validation Implementation

```typescript
// {PATH_PREFIX}/lib/env-validation.ts
function validateEnvironment() {
  const errors: string[] = [];

  if (!process.env.DATABASE_URL) errors.push('DATABASE_URL is required');
  if (!process.env.JWT_SECRET || process.env.JWT_SECRET.length < 32)
    errors.push('JWT_SECRET must be at least 32 characters');

  if (process.env.NODE_ENV === 'production') {
    if (!process.env.APP_URL?.startsWith('https://'))
      errors.push('APP_URL must use HTTPS in production');
  }

  if (errors.length > 0) {
    console.error('Environment validation failed:', errors);
    process.exit(1); // Fail fast
  }
}
```

---

## 40. Safety Guards & Self-Protection

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Critical | Reference |

### 40.1 Authorization Guards

| Guard | Description | Applies To |
|-------|-------------|------------|
| Cannot demote SUPER_ADMIN | Prevents removing platform-level access | Role change |
| Cannot change own role | Prevents accidental self-lockout | Role change |
| Cannot change own status | Prevents self-deactivation | Status toggle |
| Cannot delete own account | Prevents accidental self-removal | User delete |
| Cannot delete other SUPER_ADMINs | Prevents admin power plays | User delete |
| Soft deletes only | Preserves audit trail | User delete |
| Self-MFA protection | Admin cannot remove own MFA via admin UI | MFA management |

### 40.2 Confirmation Requirements

| Action | Confirmation Type |
|--------|-------------------|
| Delete user | Confirmation dialog with user name |
| Permanent delete | Type "DELETE" to confirm |
| Unlock account | Confirmation dialog |
| Clear all rate limits | Double confirmation ("Are you sure?" + "This cannot be undone") |
| Force logout all | Confirmation with warning message |
| Database restore | Type database name to confirm |
| Kill switch activation | Double confirmation + reason required |
| Configuration rollback | Confirmation with diff preview |

### 40.3 Data Protection

| Protection | Description |
|------------|-------------|
| Password masking | Passwords never returned in API responses |
| API key masking | Only last 4 characters visible |
| SMTP password masking | Displayed as `********` |
| Storage credentials | Not exposed in API responses |
| Session tokens | HttpOnly (not accessible via JavaScript) |
| Audit log immutability | Audit records cannot be modified or deleted |
| Backup encryption | All backups encrypted at rest |
| PII masking in exports | Optional masking for compliance |

### 40.4 Periodic Access Reviews

| Feature | Description |
|---------|-------------|
| Stale Credentials | Detect users who haven't logged in for 90+ days |
| Excessive Permissions | Alert on users with more permissions than needed |
| Admin Account Audit | Periodic review of all admin-level accounts |
| API Key Rotation | Reminder to rotate API keys older than 90 days |
| Certificate Expiry | Alert on expiring SSL/TLS certificates |

---

## 41. Admin API Endpoints Summary

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Reference |

### 41.1 Security Router (`security.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `getIpAllowlist` | GET | Admin | Fetch tenant's allowed IPs |
| `addIpToAllowlist` | POST | Admin | Add IP with validation |
| `removeIpFromAllowlist` | DELETE | Admin | Remove specific IP |
| `updateIpAllowlist` | PUT | Admin | Bulk replace allowlist |
| `getRateLimits` | GET | Admin | List active rate limits with TTL |
| `clearRateLimit` | DELETE | Admin | Clear specific rate limit key |
| `clearRateLimitByIp` | DELETE | Admin | Clear all limits for an IP |
| `clearAllRateLimits` | DELETE | Admin | Emergency clear all limits |
| `getLockedUsers` | GET | Admin | List locked accounts |
| `unlockUser` | POST | Admin | Unlock user by ID |
| `getSecurityStats` | GET | Admin | Aggregate security metrics |
| `getThreats` | GET | Admin | Detected threats list |
| `getSecurityOverview` | GET | Admin | Security dashboard data |

### 41.2 User Router (`user.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `list` | GET | Admin | Paginated users with search/filters |
| `getById` | GET | Admin | Full user details |
| `create` | POST | Admin | New user creation |
| `update` | PUT | Admin | Profile update |
| `updateRole` | PUT | Admin | Change user role |
| `updateStatus` | PUT | Admin | Activate/deactivate |
| `delete` | DELETE | Admin | Soft delete user |
| `permanentDelete` | DELETE | Super Admin | Hard delete (90+ days inactive) |
| `resetPassword` | POST | Admin | Admin-initiated password reset |
| `search` | GET | Protected | User autocomplete |
| `import` | POST | Admin | Bulk CSV import |
| `export` | GET | Admin | CSV export with optional PII masking |
| `getInactiveUsers` | GET | Admin | Users inactive 90+ days |

### 41.3 Auth Router (`auth.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `login` | POST | Public | Email/password + tenant authentication |
| `register` | POST | Public | New user registration |
| `logout` | POST | Protected | Session destruction |
| `refresh` | POST | Protected | Token refresh |
| `changePassword` | POST | Protected | Requires current password |
| `resetPassword` | POST | Public | Request password reset email |
| `setupMfa` | POST | Protected | Generate MFA secret + QR |
| `verifyMfa` | POST | Protected | Confirm TOTP and enable MFA |
| `disableMfa` | POST | Protected | Requires password + TOTP |
| `forceLogout` | POST | Admin | Force logout specific user |
| `bulkForceLogout` | POST | Super Admin | Force logout all users |

### 41.4 SMTP Router (`smtp.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `getConfig` | GET | Admin | Get SMTP settings |
| `updateConfig` | PUT | Admin | Save SMTP configuration |
| `testEmail` | POST | Admin | Send test email |

### 41.5 Monitoring Router (`monitoring.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `getApiMetrics` | GET | Admin | Per-endpoint performance |
| `getSystemHealth` | GET | Admin | CPU, memory, disk, DB |
| `getHealthCheck` | GET | Public | Basic health status |
| `getDatabaseHealth` | GET | Admin | DB connections, size, slow queries |
| `getActiveSessions` | GET | Admin | Current active sessions |

### 41.6 BI / Analytics Router (`analytics.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `getUserStats` | GET | Admin | User counts, login stats |
| `getEntityStats` | GET | Admin | Entity metrics |
| `getRevenueStats` | GET | Admin | Revenue and billing metrics |
| `getPerformanceStats` | GET | Admin | Team/department performance |
| `getDashboardData` | GET | Admin | Aggregated dashboard data |
| `exportReport` | GET | Admin | Generate and download report |

### 41.7 Database Router (`database.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `createBackup` | POST | Super Admin | Trigger database backup |
| `listBackups` | GET | Super Admin | List all backups |
| `restoreBackup` | POST | Super Admin | Restore from backup |
| `downloadBackup` | GET | Super Admin | Download backup file |
| `deleteBackup` | DELETE | Super Admin | Remove old backup |
| `getConnectionStats` | GET | Admin | Connection pool metrics |

### 41.8 Configuration Router (`config.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `getSettings` | GET | Admin | Get all system settings |
| `updateSettings` | PUT | Admin | Update system settings |
| `getChangeLog` | GET | Admin | Configuration change history |
| `revertChange` | POST | Admin | Revert to previous value |

### 41.9 Webhook Router (`webhook.*`)

| Endpoint | Method | Access | Description |
|----------|--------|--------|-------------|
| `list` | GET | Admin | List configured webhooks |
| `create` | POST | Admin | Create new webhook |
| `update` | PUT | Admin | Update webhook config |
| `delete` | DELETE | Admin | Remove webhook |
| `test` | POST | Admin | Send test payload |
| `getDeliveryLog` | GET | Admin | View delivery history |

---

## 42. Middleware Execution Order

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Reference |

### 42.1 Full Request Flow

```
Incoming Request
  │
  ├─ 1. Security Headers Applied
  │     └─ HSTS, CSP, X-Frame-Options, Cache-Control, etc.
  │
  ├─ 2. CSRF Token Validation (POST/PUT/DELETE/PATCH)
  │     └─ Cookie vs Header comparison
  │
  ├─ 3. Rate Limiting Check
  │     ├─ Login: 5 req/15 min
  │     ├─ Registration: 3 req/hour
  │     ├─ API: 100 req/min
  │     └─ Admin: 50 req/min
  │
  ├─ 4. JWT Authentication
  │     ├─ Extract token from HttpOnly cookie
  │     ├─ Verify signature (HS256)
  │     ├─ Check token blacklist
  │     ├─ Check forced_logout_at
  │     └─ Fresh DB lookup (user active + exists)
  │
  ├─ 5. IP Allowlist Check
  │     └─ Verify IP is in tenant's allowlist (if enabled)
  │
  ├─ 6. Tenant Resolution
  │     └─ Set tenant context from JWT claims
  │
  ├─ 7. Permission Check
  │     └─ Verify user has required permission for the action
  │
  ├─ 8. Role Check
  │     └─ Verify user role meets minimum requirement
  │
  ├─ 9. Input Validation (Zod)
  │     └─ Validate and sanitize request body/params/query
  │
  ├─ 10. Sensitive Data Context (if applicable)
  │      └─ Mark request as accessing sensitive data
  │
  ▼ Route Handler
  │
  ├─ 11. Business Logic Execution
  │      └─ ORM queries with automatic tenant filtering (RLS)
  │
  └─ 12. Audit Logging
         └─ Log action result (success/failure/blocked)
```

### 42.2 Middleware Stack (Code)

```typescript
// Public routes
const publicProcedure = t.procedure;

// Authenticated routes
const protectedProcedure = t.procedure
  .use(isAuthenticated)
  .use(checkIpAllowlist);

// Admin routes
const adminProcedure = t.procedure
  .use(isAuthenticated)
  .use(checkIpAllowlist)
  .use(hasRole(['SUPER_ADMIN', 'TENANT_ADMIN']));

// Super admin routes
const superAdminProcedure = t.procedure
  .use(isAuthenticated)
  .use(checkIpAllowlist)
  .use(requireSuperAdmin);

// Permission-based routes
const permissionProcedure = (permission: string) =>
  t.procedure
    .use(isAuthenticated)
    .use(checkIpAllowlist)
    .use(hasPermission(permission));
```

---

## 43. Database Schema Reference

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Reference |

### 43.1 Core Tables

| Table | Purpose | Tenant-Scoped |
|-------|---------|:-------------:|
| `tenants` | Multi-tenant organizations | No |
| `users` | User accounts | Yes |
| `sessions` | Active user sessions (or Redis) | Yes |
| `invitations` | Pending user invitations | Yes |

### 43.2 Security Tables

| Table | Purpose | Tenant-Scoped |
|-------|---------|:-------------:|
| `user_mfa` | TOTP secrets and backup codes | Yes |
| `user_sso_providers` | SSO provider linkage | Yes |
| `token_blacklist` | Revoked JWT tokens | No |
| `ip_allowlist` | Per-tenant IP allowlists | Yes |
| `audit_logs` | Immutable audit trail | Yes |
| `user_logs` | Login history and tracking | Yes |
| `suspicious_activity` | Threat detection records | Yes |

### 43.3 Analytics Tables

| Table | Purpose | Tenant-Scoped |
|-------|---------|:-------------:|
| `api_metrics` | Per-endpoint performance data | No |
| `system_health` | System resource snapshots | No |
| `user_analytics` | User engagement metrics | Yes |

### 43.4 License Tables

| Table | Purpose | Tenant-Scoped |
|-------|---------|:-------------:|
| `licenses` | License keys and subscription data | Yes |
| `license_keys` | Key encryption details | Yes |
| `payments` | Payment history | Yes |
| `invoices` | Generated invoices | Yes |

### 43.5 Storage Tables

| Table | Purpose | Tenant-Scoped |
|-------|---------|:-------------:|
| `documents` | File metadata and references | Yes |
| `storage_config` | Storage provider configuration | Yes |

### 43.6 Communication Tables

| Table | Purpose | Tenant-Scoped |
|-------|---------|:-------------:|
| `email_templates` | Email template definitions | Yes |
| `email_logs` | Sent email history | Yes |
| `notifications` | In-app notifications | Yes |
| `webhooks` | Webhook configurations | Yes |
| `webhook_deliveries` | Webhook delivery log | Yes |

### 43.7 Configuration Tables

| Table | Purpose | Tenant-Scoped |
|-------|---------|:-------------:|
| `system_settings` | System-wide configuration | Yes |
| `feature_flags` | Feature flag definitions | No |
| `config_change_log` | Configuration change history | Yes |

### 43.8 Key Indexes

```sql
-- Performance-critical indexes
CREATE INDEX idx_users_tenant_email ON users(tenant_id, email);
CREATE INDEX idx_users_tenant_role ON users(tenant_id, role);
CREATE INDEX idx_users_last_login ON users(last_login_at);
CREATE INDEX idx_audit_logs_tenant_timestamp ON audit_logs(tenant_id, timestamp DESC);
CREATE INDEX idx_audit_logs_tenant_action ON audit_logs(tenant_id, action);
CREATE INDEX idx_audit_logs_user ON audit_logs(user_id);
CREATE INDEX idx_token_blacklist_hash ON token_blacklist(token_hash);
CREATE INDEX idx_token_blacklist_expires ON token_blacklist(expires_at);
CREATE INDEX idx_user_logs_tenant_user ON user_logs(tenant_id, user_id);
CREATE INDEX idx_user_logs_ip ON user_logs(ip_address);
CREATE INDEX idx_suspicious_activity_ip ON suspicious_activity(ip_address);
```

---

## 44. File Reference Template

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Reference | Reference |

### 44.1 Backend Files

| Feature | Path |
|---------|------|
| Auth Router | `{PATH_PREFIX}/routers/auth.ts` |
| Security Router | `{PATH_PREFIX}/routers/security.ts` |
| User Router | `{PATH_PREFIX}/routers/user.ts` |
| SMTP Router | `{PATH_PREFIX}/routers/smtp.ts` |
| Monitoring Router | `{PATH_PREFIX}/routers/monitoring.ts` |
| Analytics Router | `{PATH_PREFIX}/routers/analytics.ts` |
| Database Router | `{PATH_PREFIX}/routers/database.ts` |
| Webhook Router | `{PATH_PREFIX}/routers/webhook.ts` |
| Config Router | `{PATH_PREFIX}/routers/config.ts` |
| Rate Limiting | `{PATH_PREFIX}/lib/rateLimit.ts` |
| Security Headers | `{PATH_PREFIX}/middleware/security-headers.ts` |
| CSRF Middleware | `{PATH_PREFIX}/middleware/csrf.ts` |
| IP Allowlist | `{PATH_PREFIX}/middleware/ipAllowlist.ts` |
| Auth Middleware | `{PATH_PREFIX}/middleware/auth.ts` |
| RLS Middleware | `{PATH_PREFIX}/middleware/rls.ts` |
| Password Utilities | `{PATH_PREFIX}/lib/password.ts` |
| MFA Utilities | `{PATH_PREFIX}/lib/mfa.ts` |
| Session Management | `{PATH_PREFIX}/lib/session.ts` |
| Audit Logger | `{PATH_PREFIX}/lib/audit.ts` |
| Token Blacklist | `{PATH_PREFIX}/lib/tokenBlacklist.ts` |
| RBAC Constants | `{PATH_PREFIX}/constants/roles.ts` |
| Permission Constants | `{PATH_PREFIX}/constants/permissions.ts` |
| Sanitization | `{PATH_PREFIX}/lib/sanitize.ts` |
| Env Validation | `{PATH_PREFIX}/lib/env-validation.ts` |
| CSRF Utility | `{PATH_PREFIX}/lib/csrf.ts` |
| License Crypto | `{PATH_PREFIX}/lib/license/crypto.ts` |
| Request Context | `{PATH_PREFIX}/context.ts` |
| Middleware Stack | `{PATH_PREFIX}/trpc.ts` or `{PATH_PREFIX}/middleware/index.ts` |

### 44.2 Frontend Files

| Feature | Path |
|---------|------|
| Security Admin | `{FRONTEND_PREFIX}/app/dashboard/admin/security/page.tsx` |
| Audit Admin | `{FRONTEND_PREFIX}/app/dashboard/admin/audit/page.tsx` |
| User Admin | `{FRONTEND_PREFIX}/app/dashboard/admin/users/page.tsx` |
| System Config | `{FRONTEND_PREFIX}/app/dashboard/admin/system/page.tsx` |
| Integrations | `{FRONTEND_PREFIX}/app/dashboard/admin/integrations/page.tsx` |
| Storage Admin | `{FRONTEND_PREFIX}/app/dashboard/admin/storage/page.tsx` |
| Database Admin | `{FRONTEND_PREFIX}/app/dashboard/admin/database/page.tsx` |
| Analytics | `{FRONTEND_PREFIX}/app/dashboard/admin/analytics/page.tsx` |
| System Monitor | `{FRONTEND_PREFIX}/app/dashboard/admin/system-monitor/page.tsx` |
| API Performance | `{FRONTEND_PREFIX}/app/dashboard/admin/api-performance/page.tsx` |
| Inactive Users | `{FRONTEND_PREFIX}/app/dashboard/admin/inactive-users/page.tsx` |
| User Logs | `{FRONTEND_PREFIX}/app/dashboard/admin/user-logs/page.tsx` |
| MFA Management | `{FRONTEND_PREFIX}/app/dashboard/admin/mfa/page.tsx` |
| License Management | `{FRONTEND_PREFIX}/app/dashboard/admin/license/page.tsx` |
| Alert Rules | `{FRONTEND_PREFIX}/app/dashboard/admin/alert-rules/page.tsx` |
| Feature Flags | `{FRONTEND_PREFIX}/app/dashboard/admin/feature-flags/page.tsx` |
| CSRF Hook | `{FRONTEND_PREFIX}/hooks/use-csrf.ts` |
| Auth Components | `{FRONTEND_PREFIX}/components/auth/*.tsx` |
| Admin Components | `{FRONTEND_PREFIX}/components/admin/*.tsx` |

### 44.3 Configuration Files

| Feature | Path |
|---------|------|
| Environment | `.env` / `.env.example` |
| Database Schema | `prisma/schema.prisma` or `drizzle/schema.ts` |
| Seed Data | `prisma/seed.ts` or `scripts/seed.ts` |
| Init SQL | `docker/init.sql` |
| Docker Compose | `docker/docker-compose.yml` |
| CI Workflow | `.github/workflows/ci.yml` |
| E2E Workflow | `.github/workflows/e2e.yml` |
| Security Workflow | `.github/workflows/security.yml` |
| Process Manager | `ecosystem.config.js` |
| Tunnel Config | `/etc/cloudflared/config.yml` |

---

## 45. Implementation Checklist Summary

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | Reference | Reference |

### 45.1 Master Implementation Checklist

| # | Feature | Section | Priority | Status | Notes |
|---|---------|---------|----------|--------|-------|
| 1 | Project setup and configuration | 1 | Critical | To Do / Done | |
| 2 | Login/logout flow | 2 | Critical | To Do / Done | |
| 3 | JWT token management | 2 | Critical | To Do / Done | |
| 4 | Account lockout | 2 | Critical | To Do / Done | |
| 5 | Login statistics tracking | 2 | Medium | To Do / Done | |
| 6 | Registration | 2 | Critical | To Do / Done | |
| 7 | Forced logout & token blacklist | 2 | High | To Do / Done | |
| 8 | SSO integration | 2 | Medium | To Do / Done | |
| 9 | TOTP MFA setup | 3 | Critical | To Do / Done | |
| 10 | MFA backup codes | 3 | Critical | To Do / Done | |
| 11 | Admin MFA controls | 3 | High | To Do / Done | |
| 12 | Bcrypt password hashing | 4 | Critical | To Do / Done | |
| 13 | Password complexity rules | 4 | Critical | To Do / Done | |
| 14 | Password blacklist | 4 | High | To Do / Done | |
| 15 | Admin password reset | 4 | High | To Do / Done | |
| 16 | Redis session management | 5 | Critical | To Do / Done | |
| 17 | Active session tracking | 5 | High | To Do / Done | |
| 18 | Session auto-extend | 5 | Medium | To Do / Done | |
| 19 | CSRF double-submit cookie | 6 | High | To Do / Done | |
| 20 | Input sanitization functions | 7 | High | To Do / Done | |
| 21 | Zod schema validation | 7 | High | To Do / Done | |
| 22 | Role hierarchy implementation | 8 | Critical | To Do / Done | |
| 23 | 70+ granular permissions | 9 | Critical | To Do / Done | |
| 24 | Permission matrix view | 9 | Medium | To Do / Done | |
| 25 | Rate limiting (10 types) | 10 | High | To Do / Done | |
| 26 | Rate limit monitoring dashboard | 10 | Medium | To Do / Done | |
| 27 | IP allowlisting | 11 | High | To Do / Done | |
| 28 | Geo-restriction | 11 | Low | To Do / Done | |
| 29 | Security headers (8 headers) | 12 | High | To Do / Done | |
| 30 | Content Security Policy | 12 | High | To Do / Done | |
| 31 | Audit logging (35+ actions) | 13 | Critical | To Do / Done | |
| 32 | Audit log export (4 formats) | 13 | Medium | To Do / Done | |
| 33 | Compliance reporting | 13 | Medium | To Do / Done | |
| 34 | Row-level security (ORM) | 14 | Critical | To Do / Done | |
| 35 | PostgreSQL native RLS | 14 | High | To Do / Done | |
| 36 | Threat detection (6 types) | 15 | High | To Do / Done | |
| 37 | Suspicious IP tracking | 15 | Medium | To Do / Done | |
| 38 | Admin dashboard tabs (19 pages) | 16 | High | To Do / Done | |
| 39 | User CRUD + search/filters | 17 | Critical | To Do / Done | |
| 40 | User invitations | 17 | Medium | To Do / Done | |
| 41 | Inactive user management | 17 | Medium | To Do / Done | |
| 42 | System configuration UI | 18 | High | To Do / Done | |
| 43 | Feature toggles | 18 | Medium | To Do / Done | |
| 44 | Branding configuration | 18 | Low | To Do / Done | |
| 45 | Database backup/restore | 19 | High | To Do / Done | |
| 46 | Backup scheduling | 19 | Medium | To Do / Done | |
| 47 | API health monitoring | 20 | High | To Do / Done | |
| 48 | Error dashboard | 20 | Medium | To Do / Done | |
| 49 | System health monitor | 21 | High | To Do / Done | |
| 50 | Health check endpoint | 21 | High | To Do / Done | |
| 51 | BI / analytics dashboard | 22 | Medium | To Do / Done | |
| 52 | Report generation & export | 22 | Medium | To Do / Done | |
| 53 | Privacy-first analytics | 23 | Low | To Do / Done | |
| 54 | Object storage dashboard | 24 | Medium | To Do / Done | |
| 55 | Storage test connection | 24 | Medium | To Do / Done | |
| 56 | SMTP configuration | 25 | Medium | To Do / Done | |
| 57 | Test email | 25 | Medium | To Do / Done | |
| 58 | Email templates | 25 | Low | To Do / Done | |
| 59 | Inbound email integration | 25 | Low | To Do / Done | |
| 60 | Webhook system | 26 | Medium | To Do / Done | |
| 61 | HMAC-SHA256 signing | 26 | Medium | To Do / Done | |
| 62 | Webhook retry & auto-disable | 26 | Medium | To Do / Done | |
| 63 | Webhook delivery log | 26 | Medium | To Do / Done | |
| 64 | Email notifications | 27 | Medium | To Do / Done | |
| 65 | In-app notifications | 27 | Medium | To Do / Done | |
| 66 | Push notifications | 27 | Low | To Do / Done | |
| 67 | SMS (Twilio) | 27 | Low | To Do / Done | |
| 68 | WhatsApp (Twilio) | 27 | Low | To Do / Done | |
| 69 | Slack / Teams integration | 27 | Low | To Do / Done | |
| 70 | Alert rules configuration | 27 | Medium | To Do / Done | |
| 71 | Feature flags (boolean) | 28 | Medium | To Do / Done | |
| 72 | Percentage rollouts | 28 | Low | To Do / Done | |
| 73 | Kill switches | 28 | Medium | To Do / Done | |
| 74 | A/B testing | 28 | Low | To Do / Done | |
| 75 | License tier management | 29 | Medium | To Do / Done | |
| 76 | License key generation | 29 | Medium | To Do / Done | |
| 77 | License validation middleware | 29 | Medium | To Do / Done | |
| 78 | Post-quantum encryption (V2) | 29 | Low | To Do / Done | |
| 79 | Payment integration | 29 | Medium | To Do / Done | |
| 80 | Auto-downgrade cron | 29 | Medium | To Do / Done | |
| 81 | Training programs | 30 | Low | To Do / Done | |
| 82 | Certification tracking | 30 | Low | To Do / Done | |
| 83 | SLA policies | 31 | Medium | To Do / Done | |
| 84 | SLA compliance dashboard | 31 | Medium | To Do / Done | |
| 85 | Business hours calculation | 31 | Medium | To Do / Done | |
| 86 | Survey templates | 32 | Low | To Do / Done | |
| 87 | Auto-deploy surveys | 32 | Low | To Do / Done | |
| 88 | Survey analytics | 32 | Low | To Do / Done | |
| 89 | Organization hierarchy | 33 | Medium | To Do / Done | |
| 90 | Unit-level data isolation | 33 | Medium | To Do / Done | |
| 91 | CSV user import | 34 | Medium | To Do / Done | |
| 92 | CSV user export with PII masking | 34 | Medium | To Do / Done | |
| 93 | Configuration change logging | 35 | Medium | To Do / Done | |
| 94 | Configuration rollback | 35 | Low | To Do / Done | |
| 95 | Frontend performance optimization | 36 | Medium | To Do / Done | |
| 96 | Backend query optimization | 36 | Medium | To Do / Done | |
| 97 | Caching strategy | 36 | Medium | To Do / Done | |
| 98 | SAST (SonarQube/CodeQL) | 37 | Medium | To Do / Done | |
| 99 | DAST (OWASP ZAP) | 37 | Medium | To Do / Done | |
| 100 | SCA (npm audit/Snyk) | 37 | Medium | To Do / Done | |
| 101 | CI/CD security pipeline | 37 | Medium | To Do / Done | |
| 102 | Network security (tunnel + VPN) | 38 | Critical | To Do / Done | |
| 103 | Secrets management | 38 | High | To Do / Done | |
| 104 | Environment variable validation | 39 | High | To Do / Done | |
| 105 | Production enforcement checks | 39 | High | To Do / Done | |
| 106 | Self-protection guards | 40 | Critical | To Do / Done | |
| 107 | Confirmation dialogs | 40 | High | To Do / Done | |
| 108 | Periodic access reviews | 40 | Medium | To Do / Done | |

### 45.2 Category Summary Dashboard

| Category | Total Features | Critical | High | Medium | Low |
|----------|:--------------:|:--------:|:----:|:------:|:---:|
| Core Security (1-7) | 21 | 10 | 8 | 3 | 0 |
| Access Control (8-9) | 4 | 3 | 0 | 1 | 0 |
| Security Controls (10-15) | 15 | 3 | 7 | 3 | 2 |
| Admin Dashboard (16-18) | 8 | 2 | 3 | 2 | 1 |
| Infrastructure (19-21) | 9 | 0 | 5 | 4 | 0 |
| Analytics (22-23) | 3 | 0 | 0 | 2 | 1 |
| Storage & Comms (24-27) | 16 | 0 | 0 | 10 | 6 |
| Advanced Features (28-33) | 17 | 0 | 0 | 10 | 7 |
| Operations (34-37) | 10 | 0 | 0 | 7 | 3 |
| Reference (38-45) | 5 | 2 | 3 | 0 | 0 |
| **TOTAL** | **108** | **20** | **26** | **42** | **20** |

### 45.3 Recommended Implementation Order

| Phase | Sections | Description | Est. Effort |
|-------|----------|-------------|-------------|
| 1 | 1-5, 8 | Core auth, sessions, roles | 2-3 weeks |
| 2 | 4, 7, 12, 14 | Password, sanitization, headers, RLS | 1-2 weeks |
| 3 | 6, 9-11, 13, 15 | CSRF, permissions, rate limiting, audit, threats | 2-3 weeks |
| 4 | 16-18, 39-40 | Admin dashboard, config, guards | 2-3 weeks |
| 5 | 3, 19-21 | MFA, DB management, monitoring | 2-3 weeks |
| 6 | 25-27, 34 | Email, webhooks, notifications, import/export | 2-3 weeks |
| 7 | 22-24, 35-37 | Analytics, storage, change log, security tests | 2-3 weeks |
| 8 | 28-33, 38 | Feature flags, licensing, SLA, surveys, org hierarchy | 3-4 weeks |

---

## Appendix: Placeholder Reference

All `{PLACEHOLDER}` variables used in this document with descriptions and examples.

| Placeholder | Description | Example Value |
|-------------|-------------|---------------|
| `{PROJECT_NAME}` | Application display name | MyApp SaaS |
| `{VERSION}` | Current application version | 1.0.0 |
| `{DOMAIN}` | Production domain | app.example.com |
| `{APP_PORT}` | Application server port | 3000 |
| `{DB_PORT}` | PostgreSQL port | 5432 |
| `{CACHE_PORT}` | Redis/Valkey port | 6379 |
| `{STORAGE_PORT}` | Object storage API port | 9000 |
| `{QUEUE_PORT}` | Message queue port | 5672 |
| `{AI_PORT}` | AI model server port | 11434 |
| `{TAILSCALE_IP}` | VPN access IP | 10.x.x.x |
| `{TUNNEL_ID}` | Cloudflare Tunnel UUID | c8a1b58a-... |
| `{DEFAULT_TENANT}` | Default tenant slug | demo-company |
| `{ADMIN_EMAIL}` | Default admin email | admin@example.com |
| `{ADMIN_PASSWORD}` | Default admin password | ChangeMe123! |
| `{COMPANY_NAME}` | Company/org name | Acme Corp |
| `{CURRENCY}` | Default currency | USD |
| `{TIMEZONE}` | Default timezone | UTC |
| `{RETENTION_YEARS}` | Audit log retention period | 7 |
| `{FRAMEWORK}` | Web framework | Next.js 15 |
| `{DATE}` | Document last updated date | 2025-06-15 |
| `{AUTHOR}` | Document author | Engineering Team |
| `{PREFIX}` | License key prefix | APP |
| `{ROLE_1}` | Role level 7 (highest) | SUPER_ADMIN |
| `{ROLE_2}` | Role level 6 | TENANT_ADMIN |
| `{ROLE_3}` | Role level 5 | MANAGER |
| `{ROLE_4}` | Role level 4 | OPERATOR |
| `{ROLE_5}` | Role level 3 | STAFF |
| `{ROLE_6}` | Role level 2 | VIEWER |
| `{ROLE_7}` | Role level 1 (lowest) | CUSTOMER |
| `{MODEL_LIST}` | Available AI models | llama3, mistral |
| `{PATH_PREFIX}` | Backend source root | packages/api/src |
| `{FRONTEND_PREFIX}` | Frontend source root | apps/web/src |
| `{DB_NAME}` | Database name | myapp_db |
| `{FEATURE_A_NAME}` | Feature toggle A name | Telehealth |
| `{FEATURE_B_NAME}` | Feature toggle B name | AI Assistant |
| `{FEATURE_C_NAME}` | Feature toggle C name | Advanced Reporting |
| `{FEATURE_D_NAME}` | Feature toggle D name | API Access |

---

## 46. File Upload Security

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 46.1 Multi-Layer File Protection

| Layer | Purpose | Description |
|-------|---------|-------------|
| MIME Type Validation | Verify file types | Check Content-Type header against allowed list |
| File Extension Filtering | Block dangerous extensions | Reject `.exe`, `.bat`, `.sh`, `.php`, `.js`, etc. |
| File Size Limits | Prevent oversized uploads | Enforce per-file and per-request size limits |
| Virus Scanning | Detect malicious content | Pattern detection for known malware signatures |
| Secure Storage | Prevent direct access | Randomized filenames, stored outside web root |
| Content Validation | Prevent MIME spoofing | Verify file headers (magic bytes) match declared MIME type |

### 46.2 Virus Scanning Framework

| Feature | Description |
|---------|-------------|
| Pattern Detection | Scan for common malicious patterns in file content |
| Header Validation | Verify file magic bytes to prevent MIME spoofing |
| Suspicious Content Detection | Flag JavaScript, VBScript, and embedded scripts |
| Automatic Cleanup | Quarantine and remove detected threats |
| ClamAV Integration | Production-ready integration with ClamAV antivirus |

### 46.3 Allowed File Types (Example)

| Category | Extensions | Max Size |
|----------|-----------|----------|
| Images | `.jpg`, `.jpeg`, `.png`, `.gif`, `.webp`, `.svg` | 10 MB |
| Documents | `.pdf`, `.doc`, `.docx`, `.xls`, `.xlsx`, `.csv` | 25 MB |
| Archives | `.zip`, `.tar.gz` | 50 MB |
| Videos | `.mp4`, `.webm` | 100 MB |

### 46.4 Additional Audit Events for File Operations

| Event | Description |
|-------|-------------|
| `FILE_UPLOADED` | Successful file upload with metadata |
| `FILE_DOWNLOADED` | File access/download event |
| `FILE_DELETED` | File removal operation |
| `MALICIOUS_FILE_BLOCKED` | Dangerous file upload blocked by scanner |

---

## 47. Security Monitoring & Incident Response

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Security |

### 47.1 Real-Time Security Monitoring Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/security-health` | GET | Security status dashboard data |
| `/csp-violation-report` | POST | CSP violation reporting endpoint |
| `/health` | GET | General system health check |

### 47.2 CSP Violation Reporting

| Feature | Description |
|---------|-------------|
| Real-Time Logging | All CSP violations logged with full context |
| Violation Reports | Detailed reports including violated directive, blocked URI, source file |
| IP Tracking | Source IP tracked for each violation |
| SIEM Integration | Ready for integration with Splunk, ELK Stack, Azure Sentinel |

### 47.3 Penetration Testing Readiness Checklist

| Test Category | Description | Priority |
|---------------|-------------|----------|
| Authentication Bypass | Attempt to bypass login, JWT, session controls | Critical |
| Authorization Escalation | Test role/permission escalation vectors | Critical |
| Input Validation Boundary | Fuzz all inputs for injection, overflow, edge cases | High |
| File Upload Vulnerability | Test for malicious uploads, path traversal, MIME spoofing | High |
| Session Management | Test fixation, hijacking, replay, concurrent sessions | High |
| Error Handling | Verify no sensitive info leaked in error responses | Medium |
| Information Disclosure | Check for exposed debug data, stack traces, version info | Medium |

### 47.4 Forensics & Investigation

| Evidence Type | Description |
|---------------|-------------|
| Audit Trails | Complete event history with timestamps and actor info |
| IP Tracking | Source IP logged for every security-relevant event |
| User Agent Fingerprinting | Browser and device identification for session correlation |
| Session Correlation | Link multiple requests to a single user session |
| File Integrity Monitoring | Detect unauthorized changes to system files |

---

## 48. Compliance Framework Alignment

| Status | Priority | Category |
|--------|----------|----------|
| To Do / In Progress / Done | High | Compliance |

### 48.1 Core Security Framework Checklist

| # | Control | Implementation |
|---|---------|---------------|
| 1 | OWASP Top 10 compliance | All 10 vulnerabilities addressed with comprehensive controls |
| 2 | ISO 27001 audit trail | Complete audit logging with JSONB change tracking |
| 3 | GDPR data protection | Encryption, user deactivation (not deletion), audit trails |
| 4 | SOC 2 security controls | Comprehensive security framework with monitoring |
| 5 | Secure HTTP headers | 15+ headers including CSP, HSTS, X-Frame-Options (Helmet.js) |
| 6 | CORS restricted origins | Domain-specific CORS policy |
| 7 | Multi-tier rate limiting | IP + User-Agent tracking, progressive delays, retry-after headers |
| 8 | Password hashing | bcrypt with 12 salt rounds, strong password policies |
| 9 | Input validation | Zod / express-validator across all endpoints |
| 10 | Session management | Secure cookies: HttpOnly, SameSite=Strict, configurable timeout |
| 11 | HTTPS enforcement | HSTS with 1-year max-age and includeSubDomains |
| 12 | CSRF protection | Double-submit cookie with HMAC tokens |
| 13 | XSS prevention | Multi-level sanitization (escapeHtml, DOMPurify, CSP) |
| 14 | SQL injection prevention | Parameterized queries via ORM (Prisma/Drizzle) |
| 15 | Encryption at rest | AES-256 for sensitive data fields |
| 16 | RBAC | 7-tier role hierarchy with 70+ permissions |
| 17 | Security event logging | Structured logging with audit export (JSON, CSV, PDF) |
| 18 | Content Security Policy | Strict CSP with violation reporting |
| 19 | Error handling | Sanitized error messages preventing information disclosure |
| 20 | Health check endpoints | `/health` and `/security-health` endpoints |
| 21 | Environment variables | Validated at startup with fail-fast (see Section 39) |
| 22 | IP allowlist management | Admin UI with real-time updates and audit logging |

### 48.2 ISO 27001 Control Domain Mapping

| Control Domain | Implementation |
|----------------|---------------|
| Access Control | RBAC with 7 roles, account lockout, MFA, IP allowlisting |
| Cryptography | AES-256 encryption, bcrypt hashing, TLS 1.3, secure key management |
| Operations Security | Audit logging, change management, incident response procedures |
| Communications Security | HTTPS, HSTS, secure headers, network protection |
| System Acquisition | Secure development lifecycle, code review, dependency scanning |
| Supplier Relationships | Dependency management, security assessment, SCA tools |
| Information Security Incidents | Logging, monitoring, automated response, forensics |
| Business Continuity | Automated backup, restore procedures, replication |

### 48.3 GDPR Compliance Matrix

| GDPR Requirement | Status | Implementation |
|------------------|--------|---------------|
| Data Protection by Design | Required | Privacy-first architecture, encryption by default |
| Right to be Forgotten | Required | User deactivation with data anonymization (preserves audit compliance) |
| Data Minimization | Required | Only necessary data collected per purpose |
| Consent Management | Required | Email verification, opt-in notifications, cookie consent |
| Data Breach Notification | Required | Audit logging, incident response, 72-hour notification workflow |
| Data Portability | Required | Export functionality for all user data (JSON, CSV) |
| Encryption at Rest | Required | AES-256 encryption for all sensitive data fields |
| Access Logging | Required | Complete audit trail with IP tracking and user attribution |

### 48.4 Compliance Maturity Scoring

| Level | Framework | Status | Notes |
|-------|-----------|--------|-------|
| Full | OWASP Top 10 2021 | 100% compliant | All 10 vulnerability categories addressed |
| Full | ISO 27001 | Audit-ready | Complete control domain coverage |
| Full | GDPR | Compliant | Data protection by design |
| Full | SOC 2 | Controls implemented | Security framework in place |
| Partial | NIST Cybersecurity | Aligned | Needs formal mapping document |
| Partial | MFA Enforcement | Framework ready | Activation per-tenant configurable |
| Partial | Dependency Scanning | Manual | Automate via CI/CD pipeline (npm audit / Snyk) |
| Partial | Static Code Analysis | Basic | Add security-focused ESLint plugins (eslint-plugin-security) |

### 48.5 Advanced / Government Compliance (Optional)

These are **not required for standard enterprise** but may apply to government or defense contracts:

| Framework | Description | When Needed |
|-----------|-------------|-------------|
| NIST 800-53 | Military-grade security controls | Government/DoD deployments |
| FIPS 140-2/3 | Validated cryptographic modules | Hardware security module requirements |
| Zero Trust | Continuous verification architecture | High-security environments |
| SIEM | External security monitoring | Splunk, ELK Stack, Azure Sentinel integration |
| WAF | Web Application Firewall | CloudFlare WAF, AWS WAF deployment |
| IDS/IPS | Intrusion detection/prevention | Network-level monitoring |
| STIG | DoD security hardening standards | Military deployment targets |
| DLP | Data Loss Prevention | Specialized monitoring for data exfiltration |

---

## 49. Future Security Roadmap

| Status | Priority | Category |
|--------|----------|----------|
| Planned | Medium | Roadmap |

### 49.1 Phase 3 Planned Features

| Feature | Description | Priority |
|---------|-------------|----------|
| Advanced Threat Detection | Machine learning-based anomaly detection | High |
| Behavioral Analytics | User behavior monitoring and profiling | Medium |
| Device Fingerprinting | Enhanced device tracking beyond user agent | Medium |
| Web Application Firewall | Enhanced WAF rules at application layer | High |
| Zero Trust Architecture | Micro-segmentation and continuous verification | Low |

### 49.2 Integration Opportunities

| Category | Tools | Purpose |
|----------|-------|---------|
| SIEM | Splunk, ELK Stack, Azure Sentinel | Centralized security event monitoring |
| Threat Intelligence | VirusTotal, ThreatConnect | External threat feed integration |
| Identity Providers | Active Directory, LDAP, OAuth2, SAML | Enterprise SSO federation |
| Monitoring Platforms | Datadog, New Relic, Prometheus | Application performance and security monitoring |

---


## 50. Live Implementation Reference — EHS SuperAdmin

| Status | Priority | Category |
|--------|----------|----------|
| Done | Critical | Reference Implementation |

> This section documents the **actual implemented features** from the EHS ISO ISMS SuperAdmin dashboard at `https://audit.ceruleaninfotech.com/superadmin-lala`. It serves as a real-world reference for how the template sections above translate into a working production system.

---

### 50.1 Implementation Overview

| Field | Value |
|-------|-------|
| URL | `/superadmin-lala` (Next.js App Router, stealth — not in sidebar) |
| Framework | Next.js 16 (App Router) with TypeScript |
| Authentication | NextAuth.js v5 session + `ADMIN` role check |
| Token Auth | Bearer token support for API clients (`api-auth.ts`) |
| API Route | `/api/admin/superadmin` (unified GET sections + POST actions) |
| Component | `src/components/superadmin/superadmin-dashboard.tsx` (~4,172 lines) |
| Queries | `src/lib/superadmin-queries.ts` (~510 lines) |
| API Handler | `src/app/api/admin/superadmin/route.ts` (~793 lines) |
| Database | PostgreSQL 16 via Prisma v7 (PrismaPg adapter) |
| UI Library | Tailwind CSS v4 + shadcn-style components |
| Icons | Lucide React |
| Process Manager | PM2 (`iso-isms` process) |

### 50.2 Architecture — Single Page, 13 Tabs

**Design Philosophy:**
- **One stealth page** — Not listed in sidebar; known only to administrators
- **Single API route** — All 13 tabs share ONE endpoint, dispatched by `section` (GET) or `action` (POST)
- **Server + Client split** — Server component fetches 5 initial datasets in parallel; client handles tab switching and mutations
- **Tab state persisted** — Active tab saved in `localStorage` for return visits

**Data Flow:**

```
page.tsx (Server Component)
  │
  ├── auth() → check ADMIN role → redirect("/dashboard") if not
  │
  ├── Promise.all([
  │     getSystemHealth(),        // Tab 1: System Health
  │     getDatabaseStats(),       // Tab 2: Database
  │     getUserSecurity(),        // Tab 3: Users & Security
  │     getAuditAnalytics(),      // Tab 5: Audit Analytics
  │     getEnvironmentConfig(),   // Tab 6: Environment
  │   ])
  │
  └── <SuperAdminDashboard initialData={...} />
        │
        ├── Tab state in localStorage ("superadmin-tab")
        ├── Per-tab refresh via GET /api/admin/superadmin?section=xxx
        └── Mutations via POST /api/admin/superadmin { action: "xxx", ...params }
```

**Page-Level Auth (Server Component):**

```typescript
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function SuperAdminPage() {
  const session = await auth();
  if (!session?.user?.id || (session.user as { role?: string }).role !== "ADMIN") {
    redirect("/dashboard");
  }

  const [health, database, users, audit, environment] = await Promise.all([
    getSystemHealth(),
    getDatabaseStats(),
    getUserSecurity(),
    getAuditAnalytics(),
    getEnvironmentConfig(),
  ]);

  return <SuperAdminDashboard initialData={{ health, database, users, audit, environment }} />;
}
```

**API-Level Auth (Route Handler):**

```typescript
import { authenticateRequest } from "@/lib/api-auth";

const user = await authenticateRequest(request);
// Supports: 1) NextAuth session cookies (browser)  2) Bearer token (API clients)
if (!user) return NextResponse.json({ error: "Unauthorized" }, { status: 401 });
if (user.role !== "ADMIN") return NextResponse.json({ error: "Forbidden" }, { status: 403 });
```

### 50.3 Dashboard Tab Inventory (13 Tabs)

| # | Tab ID | Label | Icon | Data Source | Status |
|---|--------|-------|------|-------------|--------|
| 1 | `health` | System Health | Activity | `getSystemHealth()` | Done |
| 2 | `database` | Database | Database | `getDatabaseStats()` | Done |
| 3 | `users` | Users & Security | Shield | `getUserSecurity()` | Done |
| 4 | `activity` | User Activity | Eye | `/api/user/activity` | Done |
| 5 | `audit` | Audit Analytics | FileText | `getAuditAnalytics()` | Done |
| 6 | `environment` | Environment | Settings | `getEnvironmentConfig()` | Done |
| 7 | `chatbot` | AI Chatbot | MessageSquare | `systemConfig` table | Done |
| 8 | `storage` | Storage | HardDrive | `getStorageConfig()` + S3 client | Done |
| 9 | `ocs` | OCS Servers | Monitor | `ocsServer` table | Done |
| 10 | `gravityzone` | GravityZone | ShieldCheck | `gravityZoneServer` table | Done |
| 11 | `modules` | Modules | Package | `systemConfig` (DISABLED_MODULES) | Done |
| 12 | `bulkactions` | Bulk Actions | Layers | `/api/bulk-actions/history` | Done |
| 13 | `datamanage` | Data Management | Trash2 | Assets + Acknowledgments | Done |

**Tab Definition:**

```typescript
const TABS = [
  { id: "health",      label: "System Health",     icon: Activity },
  { id: "database",    label: "Database",           icon: Database },
  { id: "users",       label: "Users & Security",   icon: Shield },
  { id: "activity",    label: "User Activity",      icon: Eye },
  { id: "audit",       label: "Audit Analytics",    icon: FileText },
  { id: "environment", label: "Environment",        icon: Settings },
  { id: "chatbot",     label: "AI Chatbot",         icon: MessageSquare },
  { id: "storage",     label: "Storage",            icon: HardDrive },
  { id: "ocs",         label: "OCS Servers",        icon: Monitor },
  { id: "gravityzone", label: "GravityZone",        icon: ShieldCheck },
  { id: "modules",     label: "Modules",            icon: Package },
  { id: "bulkactions", label: "Bulk Actions",       icon: Layers },
  { id: "datamanage",  label: "Data Management",    icon: Trash2 },
] as const;
```

### 50.4 API Route Dispatch Pattern

**GET Handler — Route by `?section=`**

```typescript
export async function GET(request: Request) {
  const url = new URL(request.url);
  const section = url.searchParams.get("section");
  switch (section) {
    case "health":         return NextResponse.json(await getSystemHealth());
    case "database":       return NextResponse.json(await getDatabaseStats());
    case "users":          return NextResponse.json(await getUserSecurity());
    case "audit":          return NextResponse.json(await getAuditAnalytics());
    case "environment":    return NextResponse.json(await getEnvironmentConfig());
    case "chatbot":        return NextResponse.json(await getChatbotConfig());
    case "storage":        return NextResponse.json(await getStorageConfig());
    case "ocs_servers":    return NextResponse.json(await getOcsServers());
    case "gz_servers":     return NextResponse.json(await getGzServers());
    case "backups_list":   return NextResponse.json(await listS3Backups());
    case "module_toggles": return NextResponse.json(await getModuleToggles());
    case "audit_logs":     return NextResponse.json(await getAuditLogs(params));
    default:               return NextResponse.json(await getAllSections());
  }
}
```

**POST Handler — Route by `body.action`**

```typescript
export async function POST(request: Request) {
  const body = await request.json();
  switch (body.action) {
    case "backup":                     // Tab 2: DB backup download
    case "backup_to_s3":               // Tab 2: DB backup to S3
    case "unlock_user":                // Tab 3: Unlock locked account
    case "deactivate_user":            // Tab 3: Deactivate user
    case "purge_audit_logs":           // Tab 5: Purge old logs (by age)
    case "delete_audit_logs":          // Tab 5: Delete selected logs
    case "delete_all_audit_logs":      // Tab 5: Delete ALL logs
    case "save_ai_key":                // Tab 7: Save AI provider config
    case "test_ai_key":                // Tab 7: Test AI connection
    case "delete_ai_key":              // Tab 7: Remove AI config
    case "save_storage_config":        // Tab 8: Save S3 config
    case "test_storage_connection":    // Tab 8: Test S3 connectivity
    case "delete_storage_config":      // Tab 8: Remove S3 config
    case "migrate_to_s3":              // Tab 8: Migrate local files to S3
    case "save_ocs_server":            // Tab 9: Create/update OCS server
    case "test_ocs_server":            // Tab 9: Test OCS connection
    case "delete_ocs_server":          // Tab 9: Delete OCS server
    case "save_gz_server":             // Tab 10: Create/update GZ server
    case "test_gz_server":             // Tab 10: Test GZ connection
    case "delete_gz_server":           // Tab 10: Delete GZ server
    case "save_module_toggles":        // Tab 11: Save disabled modules
    case "bulk_delete_assets":         // Tab 13: Cascade-delete assets
    case "bulk_delete_acknowledgments":// Tab 13: Delete acks + notifications
  }
}
```

---

### 50.5 Tab 1: System Health

Real-time server health monitoring with live metrics.

**Stat Cards (4):**

| Card | Metric | Source |
|------|--------|--------|
| CPU Usage | Percentage + core count + model | `os.cpus()` |
| Memory Usage | Used/Total + percentage | `os.totalmem()` / `os.freemem()` |
| Disk Usage | Used/Total/Available + percentage | `execSync("df -h / \| tail -1")` |
| System Uptime | System + process uptime | `os.uptime()` / `process.uptime()` |

**Collapsible Detail Sections:**

| Section | Content |
|---------|---------|
| CPU | Progress bar + load averages (1m/5m/15m) + core count + model name |
| Memory | Progress bar + Node.js RSS, heap used/total, external memory |
| Disk | Progress bar + available space |
| Database Connectivity | Health status badge (Connected/Disconnected) + latency in ms |
| OS & Runtime | OS type, release, hostname, architecture, Node.js version, app version |
| PM2 Process | Process name, status badge (online/stopped), uptime, restart count, memory usage |

**Color-Coded Progress Bars:** Green (<70%), Yellow (70-90%), Red (>90%)

**UI Layout:**

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│  CPU Usage   │  Memory      │  Disk Usage  │  Uptime      │
│  45.2%       │  6.8/16 GB   │  78/120 GB   │  42d 7h      │
└──────────────┴──────────────┴──────────────┴──────────────┘
┌──────────────────┬──────────────────┬──────────────────────┐
│  CPU Details     │  Memory Details  │  Disk Details        │
│  ████████░░ 80%  │  ██████░░░░ 60%  │  ████████░░ 65%      │
│  Load: 2.1/1.8/1.5│ RSS: 245MB     │  Avail: 42 GB        │
│  Cores: 8        │  Heap: 180MB     │                      │
└──────────────────┴──────────────────┴──────────────────────┘
┌──────────────────────────────┬──────────────────────────────┐
│  Database Connectivity       │  OS & Runtime Info           │
│  ✅ Connected — 2ms          │  Linux 6.17 / x86_64        │
│                              │  Node.js v20.11.0            │
└──────────────────────────────┴──────────────────────────────┘
┌──────────────────────────────────────────────────────────────┐
│  PM2 Processes                                              │
│  iso-isms  │ 🟢 online │ 42d 7h │ 0 restarts │ 245 MB     │
└──────────────────────────────────────────────────────────────┘
```

**Implementation:**

```typescript
// CPU usage calculation
const cpus = os.cpus();
const cpuUsage = cpus.reduce((acc, cpu) => {
  const total = Object.values(cpu.times).reduce((a, b) => a + b, 0);
  const idle = cpu.times.idle;
  return acc + ((total - idle) / total) * 100;
}, 0) / cpus.length;

// DB ping with latency
const start = Date.now();
await prisma.$queryRawUnsafe("SELECT 1");
const latency = Date.now() - start;

// PM2 status
const pm2List = JSON.parse(execSync("pm2 jlist").toString());
```

---

### 50.6 Tab 2: Database Management

Full PostgreSQL database administration panel.

**KPI Cards (5):**

| Card | Metric | SQL Source |
|------|--------|-----------|
| Database Size | Human-readable (e.g., "245 MB") | `SELECT pg_database_size(current_database())` |
| Tables | Count of public schema tables | `SELECT count(*) FROM pg_stat_user_tables` |
| Total Rows | Sum of all table row counts | `SELECT sum(n_live_tup) FROM pg_stat_user_tables` |
| Connections | Active PostgreSQL connections | `SELECT count(*) FROM pg_stat_activity` |
| Dead Tuples | Dead tuple count (vacuum indicator) | `SELECT sum(n_dead_tup) FROM pg_stat_user_tables` |

**Features:**

| Feature | Description |
|---------|-------------|
| Download DB Backup | Full `pg_dump` → stream as `.sql` file download |
| S3 Automated Backups | Daily backups to S3/MinIO with 7-day retention (2:00 AM IST cron) |
| Backup Now to S3 | On-demand S3 backup with old backup cleanup |
| S3 Backup List | Table showing all backups with file name, size, and date |
| All Tables View | Sortable table (name, rows, size) with ascending/descending toggle |
| Prisma Migrations | Last 20 migrations with name, date, and Applied/Pending status |

**Actions:**

| Action | Confirmation | API Action |
|--------|-------------|------------|
| Download Backup | Dialog with text | `backup` + `CONFIRM_BACKUP` |
| Backup to S3 | Button click | `backup_to_s3` |

**UI Layout:**

```
┌───────────┬───────────┬───────────┬────────────┬────────────┐
│ DB Size   │ Tables    │ Total Rows│ Connections│ Dead Tuples│
│ 245 MB    │ 167       │ 412,300   │ 8          │ 1,204      │
└───────────┴───────────┴───────────┴────────────┴────────────┘
┌────────────────────────────────��────────────────────────────┐
│ [📥 Download DB Backup]  [☁️ Backup to S3]                  │
├─────────────────────────────────────────────────────────────┤
│ All Tables (sortable by name ↕ / rows ↕ / size ↕)          │
│ Table Name         │ Row Count   │ Size                     │
│ ────────────────────┼─────────────┼──────                    │
│ users              │ 60          │ 128 KB                   │
│ audit_logs         │ 15,230      │ 4.2 MB                   │
│ timesheet_entries  │ 191,000     │ 89 MB                    │
├─────────────────────────────────────────────────────────────┤
│ Prisma Migrations (last 20)                                 │
│ Name                         │ Date        │ Status          │
│ ─────────────────────────────┼─────────────┼──────           │
│ 20260315_add_bulk_actions    │ 2026-03-15  │ ✅ Applied      │
├─────────────────────────────────────────────────────────────┤
│ ☁️ S3 Automated Backups (7-day retention)                    │
│ File Name                │ Size    │ Date                    │
│ ─────────────────────────┼─────────┼─────                    │
│ backup-2026-04-04.sql.gz │ 12.3 MB │ 2026-04-04 02:00       │
└─────────────────────────────────────────────────────────────┘
```

**Backup Implementation:**

```typescript
// Local backup download
const filePath = `/tmp/db-backup-${Date.now()}.sql`;
execSync(`pg_dump "${process.env.DATABASE_URL}" > "${filePath}"`);
const fileBuffer = fs.readFileSync(filePath);
fs.unlinkSync(filePath); // cleanup temp file
return new NextResponse(fileBuffer, {
  headers: {
    "Content-Type": "application/sql",
    "Content-Disposition": `attachment; filename="backup-${date}.sql"`,
  },
});

// S3 backup (db-backup.ts)
// pg_dump → gzip → S3 PutObjectCommand → 7-day retention via ListObjectsV2 + DeleteObjectsCommand
```

---

### 50.7 Tab 3: Users & Security

Comprehensive user management with security controls.

**KPI Cards (4):** Total Users, Active, Inactive, Locked

**Sections:**

| Section | Content |
|---------|---------|
| Role Distribution | Badge counts per role (ADMIN, RISK_OWNER, AUDITOR, VIEWER) |
| Login Activity (24h) | Color-coded badges: LOGIN (green), LOGIN_FAILED (red), LOGOUT (gray) — counts from audit logs |
| Locked Accounts | Table with name, email, locked-until timestamp + "Unlock" button |
| Recent Failed Logins | Last 20 failed logins: entity, IP address, timestamp |
| Stale Users (90+ days) | Users with no login in 90+ days + "Deactivate" button |
| All Users | Full table: name, email, role, department, last login, status, failed attempts + "Reset Password" button |

**User Actions:**

| Action | Confirmation | Implementation |
|--------|-------------|----------------|
| Unlock User | Simple dialog | `prisma.user.update({ data: { failedLoginAttempts: 0, lockedUntil: null } })` |
| Deactivate User | Dialog + `CONFIRM_DEACTIVATE` | `prisma.user.update({ data: { isActive: false } })` |
| Reset Password | Dialog (min 15 chars, show/hide toggle, confirm match) | Calls `PUT /api/users/[id]` with hashed password |

**UI Layout:**

```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│ Total Users  │ Active Users │ Inactive     │ Locked       │
│ 60           │ 58           │ 2            │ 0            │
└──────────────┴──────────────┴──────────────┴──────────────┘

Role Distribution: [ADMIN: 5] [RISK_OWNER: 12] [AUDITOR: 8] [VIEWER: 35]

Login Activity (24h): [LOGIN: 23 🟢] [LOGIN_FAILED: 2 🔴] [LOGOUT: 15 🔵]

┌─────────────────────────────────────────────────────────────┐
│ 🔒 Locked Accounts                                          │
│ Name         │ Email              │ Locked Since  │ Action   │
│ ─────────────┼────────────────────┼───────────────┼──────    │
│ John Doe     │ john@example.com   │ 2h ago        │ [Unlock] │
├─────────────────────────────────────────────────────────────┤
│ 💤 Stale Users (90+ Days)                                    │
│ Name         │ Email              │ Last Login    │ Action   │
│ ─────────────┼────────────────────┼───────────────┼──────────│
│ Jane Smith   │ jane@example.com   │ 120 days ago  │[Deactive]│
├─────────────────────────────────────────────────────────────┤
│ 🔴 Recent Failed Logins (last 20)                            │
│ User         │ Email              │ Time          │ IP       │
├─────────────────────────────────────────────────────────────┤
│ All Users                                                    │
│ Name │ Email │ Role │ Dept │ Last Login │ Failed │ Status │🔑│
└─────────────────────────────────────────────────────────────┘
```

---

### 50.8 Tab 4: User Activity Tracking

Page view analytics per user with time-period filtering.

**Filters:** 7 days / 14 days / 30 days period selector buttons

**API:** `GET /api/user/activity?days=N` (separate dedicated API route)

**User Activity Summary Table:**

| Column | Description |
|--------|-------------|
| User | Name + email |
| Role | Role badge |
| Page Views | Count of page views in selected period |
| Time Spent | Total time formatted as h/m/s |
| Last Active | Timestamp of last activity |
| Most Visited | Most frequently visited page path (human-friendly) |

**Recent Page Views Table (last 50):**

| Column | Description |
|--------|-------------|
| User | Name |
| Page | Friendly-formatted path (e.g., `/dashboard` → "Dashboard") |
| Duration | Time spent on page |
| Time | Timestamp |
| IP Address | Client IP |

**Database Model:**

```prisma
model PageView {
  id        String   @id @default(cuid())
  userId    String
  path      String     // e.g., "/risks", "/dashboard"
  title     String?    // Human-friendly page title
  duration  Int?       // Time spent in seconds
  ip        String?
  userAgent String?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])

  @@map("page_views")
}
```

**Implementation:** Uses raw SQL aggregate query with subquery for per-user summary (top page, total time, view count).

---

### 50.9 Tab 5: Audit Analytics

Comprehensive audit log analysis with multi-tier deletion capabilities.

**KPI Cards (4):** Total Audit Logs, Logs Today, Total Logins, Failed Login Rate (%)

**Analysis Sections:**

| Section | Content |
|---------|---------|
| Action Breakdown | Horizontal bar chart of each audit action type with relative widths |
| Top 10 Active Users | Table ranked by action count: user name/email + count |
| Entity Type Breakdown | Module-by-module action counts table |
| Recent Security Events | Last 20 color-coded events: LOGIN_FAILED (red), SUPERADMIN_ACCESS (yellow), PERMISSION_DENIED, ACCOUNT_LOCKED, ROLE_CHANGED, PASSWORD_CHANGED |

**Activity Log Manager (Embedded Sub-Component):**

Full paginated, searchable, filterable audit log browser built into the tab:

| Feature | Description |
|---------|-------------|
| Paginated Browser | 25 logs per page with Previous/Next navigation |
| Search | Filter by entity label, action, entity type, user name/email |
| Action Filter | Dropdown populated from current page actions |
| Module Filter | Dropdown for entity type filtering |
| Select & Delete | Checkbox selection (individual + "Select All on Page") |
| Delete Selected | Bulk delete checked items |
| Delete All Logs | Nuclear option — requires typing "DELETE ALL" to confirm |

**Three-Tier Deletion System:**

| Method | Confirmation | API Action |
|--------|-------------|------------|
| Purge by Age | Dialog (min 30, max 3650 days, default 365) | `purge_audit_logs` + `CONFIRM_PURGE` |
| Delete Selected | Dialog showing selected count | `delete_audit_logs` + `CONFIRM_DELETE_LOGS` |
| Delete All | Type "DELETE ALL" to enable button | `delete_all_audit_logs` + `CONFIRM_DELETE_ALL_LOGS` |

**UI Layout:**

```
┌───────────┬────────────┬───────────┬──────────────────┐
│ Total Logs│ Logs Today │ Logins    │ Failed Login Rate│
│ 15,230    │ 142        │ 4,560     │ 2.3%             │
└───────────┴────────────┴───────────┴──────────────────┘
┌───────────────────────────────────────────��─────────────────┐
│ [🗑️ Purge Old Logs]                                         │
├─────────────────────────────────────────────────────────────┤
│ Action Breakdown:                                           │
│ CREATE   ████████████████████ 4,230                         │
│ UPDATE   ████████████████ 3,120                             │
│ LOGIN    ████████████ 2,450                                 │
│ DELETE   ████ 890                                           │
├─────────────────────────────────────────────────────────────┤
│ Activity Log Manager                                        │
│ [🔍 Search...          ] [Action ▼] [Entity Type ▼]        │
│ ☑ Select All │ [Delete Selected (3)] │ [Delete All Logs]   │
│ ☐ │ 2026-04-04 10:30 │ CREATE │ Risk │ John │ Risk-001    │
│ ☑ │ 2026-04-04 10:28 │ LOGIN  │ Auth │ Jane │ Logged in   │
│ Page 1 of 50 │ [← Prev] [Next →]                          │
└─────────────────────────────────────────────────────────────┘
```

---

### 50.10 Tab 6: Environment Configuration

Read-only environment and dependency overview.

**Sections:**

| Section | Content |
|---------|---------|
| Application Config | NODE_ENV, NEXTAUTH_URL, NEXTAUTH_SECRET (masked), DATABASE_URL (masked), app version |
| SMTP Configuration | SMTP_ENABLED, SMTP_HOST, SMTP_PORT, SMTP_FROM_EMAIL, SMTP_USER — each shown as "Configured" (green) or "Missing" (red) badge |
| OCS Integration | OCS_API_URL, OCS_API_USER status badges |
| Notification Config | NOTIFICATION_CRON_SCHEDULE, NOTIFICATION_CRON_ENABLED, Slack/Teams/Custom webhook status |
| File Storage | Local file count + total size from `uploads/` directory |
| Dependencies | Grid of all npm packages from `package.json` with name + version |

**Secret Masking:**

```typescript
function maskSecret(value: string): string {
  if (!value || value.length < 12) return "****";
  return value.slice(0, 4) + "........" + value.slice(-4);
}
// "sk-abc123def456xyz789" → "sk-a........x789"
```

**Implementation:** Uses `process.env` for environment variables, `fs.readdirSync` for upload directory stats, `JSON.parse(fs.readFileSync("package.json"))` for dependencies.

---

### 50.11 Tab 7: AI Chatbot Configuration

Multi-provider AI chatbot management with 13 supported providers.

**Supported Providers (13):**

| Provider | Models | Tier |
|----------|--------|------|
| OpenAI | GPT-4o, GPT-4o-mini, GPT-4-turbo, GPT-3.5-turbo, o1-mini, o1-preview | Paid |
| Anthropic Claude | Claude 3.5 Sonnet, Claude 3 Opus, Claude 3 Haiku | Paid |
| Google Gemini | Gemini 1.5 Pro, Gemini 1.5 Flash, Gemini Pro | Free tier available |
| Groq | Llama 3.1 70B/8B, Mixtral 8x7B, Gemma 2 9B | Free |
| Mistral AI | Mistral Large, Mistral Medium, Mistral Small, Codestral | Paid |
| DeepSeek | DeepSeek Chat, DeepSeek Coder | Free |
| OpenRouter | Auto (best), Claude 3.5 Sonnet, GPT-4o, Mixtral | Paid |
| Together AI | Llama 3.1 70B/8B, Mixtral, CodeLlama 34B | Free tier available |
| Perplexity | Llama 3.1 Sonar 70B/8B (online, with citations) | Paid |
| xAI Grok | Grok Beta, Grok 2 | Paid |
| Cohere | Command R+, Command R, Command Light | Free tier available |
| Fireworks AI | Llama 3.1 70B/8B, Mixtral, FireFunction V2 | Free tier available |
| Cerebras | Llama 3.1 70B/8B (fastest inference) | Free |

**Features:**

| Feature | Description |
|---------|-------------|
| Provider Selection | Visual card grid with radio-style selection, tier badge (Free/Paid) |
| Model Dropdown | Pre-configured model list per provider |
| API Key Input | Masked input with show/hide toggle + link to provider's key page |
| Test Connection | Verify API key works before committing |
| Remove Config | Delete all AI settings (AI_PROVIDER, AI_API_KEY, AI_MODEL) |
| Status Display | Shows active provider, model, and masked key when configured |

**Chatbot Capabilities (when configured):**
- Domain-specific knowledge (ISO 27001, CITPL policies)
- Streaming responses
- Per-user conversation persistence in PostgreSQL
- Max 2048 tokens per response
- Last 16 messages context window

**UI Flow:**

```
┌─────────────────────────────────────────────────────────────┐
│ Status: ✅ Configured — Anthropic Claude / claude-3-haiku   │
│ API Key: sk-an........6R4q                                  │
│                                          [Test] [Remove]    │
├─────────────────────────────────────────────────────────────┤
│ Select Provider:                                            │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
│ │ OpenAI   │ │ Claude ✓ │ │ Gemini   │ │ Groq     │       │
│ │ [Paid]   │ │ [Paid]   │ │ [Free]   │ │ [Free]   │       │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
│ ... (9 more providers in grid)                             │
├─────────────────────────────────────────────────────────────┤
│ Model: [claude-3-haiku-20240307 ▼]                         │
│ API Key: [sk-...                        ] [👁]              │
│ 🔗 Get your API key from console.anthropic.com              │
│                                            [Save & Test]    │
└─────────────────────────────────────────────────────────────┘
```

**Database Storage:**

```
SystemConfig table:
  AI_PROVIDER = "anthropic"
  AI_API_KEY  = "sk-ant-..."
  AI_MODEL    = "claude-3-haiku-20240307"
```

**API Actions:** `save_ai_key`, `test_ai_key`, `delete_ai_key`

**Cache Invalidation:** `clearApiKeyCache()` called after every save/delete to invalidate the in-memory config cache.

---

### 50.12 Tab 8: S3/MinIO Storage Configuration

Object storage management with local-to-S3 migration support.

**Status Display:**

| Field | Description |
|-------|-------------|
| Connection Status | "S3/MinIO Connected" (green) or "Local Disk (default)" (gray) |
| Endpoint | S3-compatible endpoint URL |
| Bucket | Target bucket name |
| Path Prefix | Object key prefix for evidence files |
| Evidence Files | Count of evidence records in database |
| Total Size | Aggregate file size |
| Local Files | Count of files still on local disk |

**Configuration Form (6 fields):**

| Field | Required | Notes |
|-------|----------|-------|
| Endpoint URL | Yes | S3-compatible endpoint (e.g., `http://minio:9000`) |
| Bucket Name | Yes | Auto-created via `ensureBucket()` if doesn't exist |
| Access Key | Yes | AWS-style access key |
| Secret Key | Yes | Masked; re-enter to change |
| Region | No | Default: `us-east-1` |
| Path Prefix | No | Object key prefix (e.g., `evidence/`) |

**Actions:**

| Action | Description |
|--------|-------------|
| Save Configuration | Saves S3 config to SystemConfig + auto-creates bucket |
| Test Connection | Verifies S3 credentials and bucket access |
| Remove Config | Reverts to local disk storage |
| Migrate Files to S3 | Moves all local files to S3 with per-file progress |

**Migration Flow:**

```
For each local file in uploads/evidence/:
  1. Read file from disk
  2. Lookup evidence record for moduleType
  3. Upload to S3: {prefix}/{moduleType}/{filename}
  4. Update evidence.filePath in database with S3 key
  5. Delete local file
  6. Report: migrated N/total, errors[]
```

**Database Storage:**

```
SystemConfig table:
  S3_ENDPOINT    = "https://minio.example.com"
  S3_ACCESS_KEY  = "AKIA..."
  S3_SECRET_KEY  = "wJal..."
  S3_BUCKET      = "evidence-bucket"
  S3_REGION      = "us-east-1"
  S3_PATH_PREFIX = "evidence/"
```

**Cache Invalidation:** `clearStorageConfigCache()` called after save/delete.

---

### 50.13 Tab 9: OCS Inventory Servers

Multi-server OCS Inventory NG management for IT asset discovery.

**Server List Table:**

| Column | Description |
|--------|-------------|
| Name | Server name (with ⭐ icon if default) |
| API URL | OCS Inventory REST API endpoint |
| Status | Enabled/Disabled badge |
| Assets | Count of assets linked to this server |

**CRUD Operations:**

| Action | Fields | Confirmation |
|--------|--------|-------------|
| Add Server | Name, API URL, Username, Password, Enabled, Set Default | — |
| Edit Server | Same fields (password optional — keeps current if blank/masked) | — |
| Test Connection | Tests API connectivity, returns computer count | — |
| Delete Server | — | Dialog: "Assets will be unlinked but not deleted" |

**Special Behaviors:**
- Default server designation (only one at a time, auto-unsets others)
- Password masking in API responses (first 4 + `****`)
- Connection test shows: "Connected — N computers found"
- On delete: FK `ocsServerId` nullified on linked Assets (assets preserved, just unlinked)
- Cache invalidation: `clearOcsConfigCache()` after every mutation

**Database Model:**

```prisma
model OcsServer {
  id        String   @id @default(cuid())
  name      String
  apiUrl    String
  username  String
  password  String
  enabled   Boolean  @default(true)
  isDefault Boolean  @default(false)
  assets    Asset[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  @@map("ocs_servers")
}
```

**Password Handling in Edit:**

```typescript
// Only update password if user typed a new value (not the masked placeholder)
if (body.password && !body.password.includes("****")) {
  updateData.password = body.password;
}
```

---

### 50.14 Tab 10: Bitdefender GravityZone Integration

Endpoint protection management via GravityZone JSON-RPC 2.0 API.

**Server List Table:**

| Column | Description |
|--------|-------------|
| Name | Server name (with ⭐ icon if default) |
| Access URL | GravityZone Control Center URL |
| Status | Enabled/Disabled badge |
| Last Sync | Timestamp of last endpoint sync |

**CRUD Operations:**

| Action | Description |
|--------|-------------|
| Add Server | Name, Access URL, API Key (HTTP Basic Auth), Enabled, Default |
| Edit Server | Same fields (API key optional — keeps current if blank/masked) |
| Test Connection | Returns: "Connected — N endpoints found" |
| Sync | Pull endpoint protection data, match to assets by hostname |
| Delete Server | Remove server configuration |

**Sync Results:**

| Metric | Description |
|--------|-------------|
| Total Endpoints | Endpoints scanned from GravityZone |
| Matched to Assets | Endpoints matched to existing asset records by hostname |
| Protection Updated | Endpoint protection status records created/updated |
| Unmatched | Endpoints with no matching asset (expandable list showing names + IPs) |

**API Key Display:** First 8 + `****` + last 4 characters

**Database Model:**

```prisma
model GravityZoneServer {
  id         String    @id @default(cuid())
  name       String
  accessUrl  String
  apiKey     String
  enabled    Boolean   @default(true)
  isDefault  Boolean   @default(false)
  lastSyncAt DateTime?
  createdAt  DateTime  @default(now())
  updatedAt  DateTime  @updatedAt
  @@map("gravityzone_servers")
}
```

**Cache Invalidation:** `clearGzConfigCache()` after every mutation.

---

### 50.15 Tab 11: Module Management

Dynamic sidebar/navigation module visibility control for all users.

**18 Module Groups (~120+ Sub-Modules):**

| # | Group | Key | Example Sub-Modules |
|---|-------|-----|---------------------|
| 1 | Risk & Compliance | risk | Risk Register, Risk Assessment, SoA, Risk Appetite |
| 2 | Asset Management | assets | Asset Register, Hardware, Software, OCS Import |
| 3 | Documents & Records | documents | Policy Library, Document Templates, Record Control |
| 4 | Incidents & Response | incidents | Incident Tracker, Response Plans, SLA Dashboard |
| 5 | Audit & Assurance | audit | Internal Audit, Findings, CAPA |
| 6 | HR Security | hr | Employee Records, Background Checks, Exit Checklists |
| 7 | Security Training | training | Training Programs, Awareness, Certifications |
| 8 | Access & Security | access | Access Reviews, Privilege Management, Break Glass |
| 9 | SDLC & Configuration | sdlc | Secure Development, Code Review, Change Mgmt |
| 10 | Encryption & Backup | encryption | Key Management, Backup Logs, Restoration Tests |
| 11 | Vulnerability Mgmt | vulnerability | Vulnerability Scanner, Patch Management |
| 12 | Third Parties | thirdparty | Vendor Assessment, Supplier Contracts |
| 13 | Physical & Environmental | physical | Facility Security, Environmental Controls |
| 14 | Business Continuity | bcp | BCP Plans, DR Testing, Impact Analysis |
| 15 | IT Operations | operations | Capacity Mgmt, Service Level Mgmt |
| 16 | Project & Timesheet Mgmt | projects | Project Tracker, Timesheets |
| 17 | Legacy Systems Archive | legacy | Archived/deprecated modules |
| 18 | Governance | governance | Management Reviews, ISMS Metrics, Reports |

**Features:**

| Feature | Description |
|---------|-------------|
| Group Toggle | Green/red switch to enable/disable entire module group |
| Sub-Module Toggle | Individual switches for each item within a group |
| Expand/Collapse | Click group row to reveal sub-modules with their paths |
| Enable All / Disable All | Bulk toggle buttons in header |
| Auto-Save | Changes saved immediately on toggle (no explicit save button) |
| Badge Counter | Shows how many sub-modules are hidden per disabled group |
| Summary | "N of 18 groups enabled, M sub-modules hidden" |

**UI Layout:**

```
┌─────────────────────────────────────────────────────────────┐
│ Module Management        [Enable All] [Disable All]         │
│ 16 of 18 groups enabled, 3 sub-modules hidden              │
├─────────────────────────────────────────────────────────────┤
│ ▶ Risk & Compliance                              [🟢 ON]   │
│ ▼ Asset Management                               [🟢 ON]   │
│   ├─ Assets (/assets)                            [🟢 ON]   │
│   ├─ OCS Import (/assets/ocs-import)             [🔴 OFF]  │
│   └─ Endpoint Protection (/endpoint-protection)  [🟢 ON]   │
│ ▶ Documents & Records                            [🟢 ON]   │
│ ▶ Incidents & Response                           [🔴 OFF]  │
│ ...                                                         │
└─────────────────────────────────────────────────────────────┘
```

**Database Storage:**

```json
// SystemConfig key: "DISABLED_MODULES"
{
  "disabledGroups": ["incidents", "legacy"],
  "disabledItems": ["/assets/ocs-import", "/encryption-keys"]
}
```

**Sidebar Integration:**

```typescript
// Sidebar reads config and filters out disabled modules
const disabledConfig = await getSystemConfig("DISABLED_MODULES");
const { disabledGroups, disabledItems } = JSON.parse(disabledConfig || "{}");
// Group-level disable overrides individual item toggles
// Re-enabling a group clears its individual item disables
```

---

### 50.16 Tab 12: Bulk Actions

Overview of bulk operation execution history — lightweight summary linking to dedicated page.

**Stat Cards (3):**

| Card | Metric |
|------|--------|
| Custom Groups | Count of saved bulk action groups |
| Total Executions | Count of all bulk action runs |
| Last Execution | Timestamp of most recent run |

**Recent Executions Table (Last 5):**

| Column | Description |
|--------|-------------|
| Date | Execution timestamp |
| Action | Action type badge |
| User | Who ran it |
| Records | Total records affected |

**Navigation:** "Open Bulk Actions Center" button → `/bulk-actions` page for full CRUD of action groups and execution workflows.

---

### 50.17 Tab 13: Data Management

Bulk data cleanup for assets and policy acknowledgments with cascade delete.

**Asset Management Section:**

| Feature | Description |
|---------|-------------|
| Search | Filter by name, assetId, type |
| Asset Table | Checkbox-selectable: Asset ID (monospace), name, type badge, status badge, source (OCS Import/Manual), created date |
| Select All | Checkbox for filtered results |
| Bulk Delete | Delete selected assets with confirmation dialog showing asset list |

**Asset Deletion Cascade:**

```typescript
for (const id of assetIds) {
  // 1. Delete dependent records first
  await prisma.endpointComplianceLog.deleteMany({ where: { assetId: id } });
  await prisma.endpointProtectionStatus.deleteMany({ where: { assetId: id } });
  await prisma.evidence.deleteMany({ where: { entityId: id, entityType: "Asset" } });
  await prisma.comment.deleteMany({ where: { entityId: id, entityType: "Asset" } });
  // 2. Nullify FK references (preserve vulnerabilities, just unlink)
  await prisma.vulnerability.updateMany({
    where: { affectedAssetId: id },
    data: { affectedAssetId: null },
  });
  // 3. Delete asset (RiskAsset cascade-deletes automatically via Prisma)
  await prisma.asset.delete({ where: { id } });
}
```

**Policy Acknowledgments Section:**

| Feature | Description |
|---------|-------------|
| Search | Filter by document title/number, user name/email |
| Status Filter | Dropdown: All / Pending / Acknowledged / Overdue |
| Summary Stats | Total, Pending, Acknowledged, Overdue counts |
| Acknowledgment Table | Checkbox-selectable: Document number + title, user + email, status badge, date |
| Delete Selected | Bulk delete selected acknowledgments + cascade notifications |
| Delete All | Pre-go-live cleanup — deletes all acknowledgments + related notifications |

**Acknowledgment Deletion Cascade:**

```typescript
for (const id of ackIds) {
  // Delete notifications that contain acknowledge links
  await prisma.notification.deleteMany({
    where: { link: { contains: id } },
  });
  await prisma.policyAcknowledgment.delete({ where: { id } });
}
```

**Confirmation Safety:**

| Action | Confirmation Code |
|--------|------------------|
| Delete Assets | `CONFIRM_DELETE_ASSETS` |
| Delete Acknowledgments | `CONFIRM_DELETE_ACKNOWLEDGMENTS` |
| Delete All Audit Logs | `CONFIRM_DELETE_ALL_LOGS` |

---

### 50.18 GET API Sections Summary

| Section Parameter | Returns |
|-------------------|---------|
| `health` | CPU, memory, disk, uptime, load avg, OS, Node.js, DB latency, PM2 status |
| `database` | DB size, table list with row counts/sizes, connections, dead tuples, migrations |
| `users` | User summary, role breakdown, locked accounts, stale users, failed logins, all users |
| `audit` | Total logs, action breakdown, top users, entity types, security events, failed login rate |
| `environment` | App config, SMTP, OCS, notifications, storage info, dependencies |
| `chatbot` | AI provider, masked API key, model |
| `storage` | S3 config status, evidence stats, local file count |
| `ocs_servers` | OCS server list with masked passwords and asset counts |
| `gz_servers` | GravityZone server list with masked API keys |
| `audit_logs` | Paginated audit logs with search/filter (page, pageSize, action, entityType, search) |
| `module_toggles` | Disabled groups and items arrays |
| `backups_list` | S3 backup file list with sizes and dates |
| _(no section)_ | Full refresh — all 5 core sections in parallel |

### 50.19 POST Actions Summary

| Action | Confirmation | Description |
|--------|-------------|-------------|
| `backup` | `CONFIRM_BACKUP` | Full pg_dump → download as .sql file |
| `backup_to_s3` | — | Automated S3 backup with 7-day retention |
| `purge_audit_logs` | `CONFIRM_PURGE` | Delete logs older than N days |
| `unlock_user` | — | Reset lockout: failedLoginAttempts=0, lockedUntil=null |
| `deactivate_user` | `CONFIRM_DEACTIVATE` | Set user isActive=false |
| `save_ai_key` | — | Save AI provider + API key + model to SystemConfig |
| `test_ai_key` | — | Test AI API connectivity |
| `delete_ai_key` | — | Remove AI configuration from SystemConfig |
| `save_storage_config` | — | Save S3/MinIO config + auto-create bucket |
| `test_storage_connection` | — | Test S3 connectivity and bucket access |
| `delete_storage_config` | — | Remove S3 config, revert to local disk |
| `migrate_to_s3` | — | Migrate all local evidence files to S3 |
| `save_ocs_server` | — | Create/update OCS server record |
| `test_ocs_server` | — | Test OCS API connectivity (returns computer count) |
| `delete_ocs_server` | — | Delete OCS server (unlinks but preserves assets) |
| `save_gz_server` | — | Create/update GravityZone server record |
| `test_gz_server` | — | Test GravityZone API connectivity |
| `delete_gz_server` | — | Delete GravityZone server record |
| `bulk_delete_assets` | `CONFIRM_DELETE_ASSETS` | Cascade-delete selected assets |
| `bulk_delete_acknowledgments` | `CONFIRM_DELETE_ACKNOWLEDGMENTS` | Delete selected/all acknowledgments + notifications |
| `delete_audit_logs` | `CONFIRM_DELETE_LOGS` | Delete selected audit log entries |
| `delete_all_audit_logs` | `CONFIRM_DELETE_ALL_LOGS` | Wipe all audit logs |
| `save_module_toggles` | — | Save disabled module groups/items |

---

### 50.20 Reusable UI Patterns

#### Pattern 1: StatCard Component

KPI card used across System Health, Database, Users, and Audit tabs.

```typescript
interface StatCardProps {
  title: string;       // "Total Users"
  value: string;       // "60"
  subtitle?: string;   // "2 inactive"
  icon: LucideIcon;    // Users icon
}

function StatCard({ title, value, subtitle, icon: Icon }: StatCardProps) {
  return (
    <Card>
      <CardContent className="p-4">
        <div className="flex items-center gap-3">
          <div className="p-2 rounded-lg bg-blue-50 dark:bg-blue-950">
            <Icon className="h-5 w-5 text-blue-600" />
          </div>
          <div>
            <p className="text-sm text-gray-500">{title}</p>
            <p className="text-2xl font-bold">{value}</p>
            {subtitle && <p className="text-xs text-gray-400">{subtitle}</p>}
          </div>
        </div>
      </CardContent>
    </Card>
  );
}
```

#### Pattern 2: CollapsibleCard with localStorage Persistence

```typescript
interface CollapsibleCardProps {
  title: string;
  icon?: LucideIcon;
  description?: string;
  headerActions?: React.ReactNode;
  storageKey: string;         // prefixed "sa-" in localStorage
  defaultOpen?: boolean;
  children: React.ReactNode;
}

function CollapsibleCard({ title, storageKey, children, ... }: CollapsibleCardProps) {
  const [open, setOpen] = useState(() => {
    if (typeof window !== "undefined") {
      return localStorage.getItem(`sa-${storageKey}`) !== "false";
    }
    return true;
  });

  useEffect(() => {
    localStorage.setItem(`sa-${storageKey}`, String(open));
  }, [open, storageKey]);

  return (
    <Card>
      <CardHeader onClick={() => setOpen(!open)} className="cursor-pointer">
        <div className="flex items-center justify-between">
          <CardTitle>{title}</CardTitle>
          {open ? <ChevronUp /> : <ChevronDown />}
        </div>
      </CardHeader>
      {open && <CardContent>{children}</CardContent>}
    </Card>
  );
}
```

#### Pattern 3: Three-Tier Confirmation System

```
Level 1 — Simple confirm dialog:
  Dialog → "Are you sure?" → [Cancel] [Confirm]
  Used for: Unlock user

Level 2 — String confirmation in API payload:
  Dialog → [Cancel] [Confirm]
  Client sends: { ...data, confirm: "CONFIRM_BACKUP" }
  Server rejects if confirm string doesn't match
  Used for: Backup, deactivate, delete assets/acks

Level 3 — Type-to-confirm:
  Dialog → "Type DELETE ALL to confirm: [________]"
  Button disabled until typed string matches exactly
  API payload: { ...data, confirm: "CONFIRM_DELETE_ALL_LOGS" }
  Used for: Delete ALL audit logs (nuclear option)
```

#### Pattern 4: Secret Masking

```typescript
// Display secrets: show first N + dots + last N
function maskSecret(value: string): string {
  if (!value || value.length < 12) return "****";
  return value.slice(0, 4) + "........" + value.slice(-4);
}

// API keys: first 8 + **** + last 4
function maskApiKey(key: string): string {
  return key.slice(0, 8) + "****" + key.slice(-4);
}

// Edit forms: only update if user typed new value (not the mask)
if (body.password && !body.password.includes("****")) {
  updateData.password = body.password;
}

// Show/hide toggle on input
<Input type={showPassword ? "text" : "password"} />
<Button onClick={() => setShowPassword(!show)}>
  {showPassword ? <EyeOff /> : <Eye />}
</Button>
```

#### Pattern 5: Tab State Persistence

```typescript
const [activeTab, setActiveTab] = useState<TabId>(() => {
  if (typeof window !== "undefined") {
    return (localStorage.getItem("superadmin-tab") as TabId) || "health";
  }
  return "health";
});

useEffect(() => {
  localStorage.setItem("superadmin-tab", activeTab);
}, [activeTab]);
```

#### Pattern 6: Per-Tab Refresh

```typescript
const refreshData = useCallback(async (section?: string) => {
  setRefreshing(true);
  const url = section
    ? `/api/admin/superadmin?section=${section}`
    : "/api/admin/superadmin";
  const res = await fetch(url);
  if (res.ok) {
    const json = await res.json();
    if (section) {
      setData(prev => ({ ...prev, [section]: json }));
    } else {
      setData(json);
    }
  }
  setRefreshing(false);
}, []);
```

#### Pattern 7: Cache Clearing After Config Changes

```typescript
// Each integration has an in-memory cache for performance.
// After any config mutation, clear the relevant cache:

import { clearApiKeyCache } from "@/lib/cloudflare-ai";         // AI chatbot
import { clearStorageConfigCache } from "@/lib/storage";         // S3 storage
import { clearOcsConfigCache } from "@/lib/ocs-client";          // OCS Inventory
import { clearGzConfigCache } from "@/lib/gravityzone-client";   // GravityZone
```

---

### 50.21 Database Models Required

```prisma
// Key-value configuration store — backbone for 4+ tabs
model SystemConfig {
  id        String   @id @default(cuid())
  key       String   @unique    // AI_PROVIDER, S3_ENDPOINT, DISABLED_MODULES, etc.
  value     String              // JSON or plaintext
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  @@map("system_config")
}

// Audit trail for all user actions — feeds 2 tabs
model AuditLog {
  id         String   @id @default(cuid())
  userId     String?
  userName   String?
  userEmail  String?
  action     String   // CREATE, UPDATE, DELETE, LOGIN, LOGIN_FAILED, LOGOUT, SUPERADMIN_ACCESS, etc.
  entityType String?  // Module name: "Risk", "Asset", "Document"
  entityId   String?
  entityLabel String? // Human-readable: "RISK-001: Data Breach"
  details    String?  // JSON with before/after values
  ipAddress  String?
  createdAt  DateTime @default(now())
}

// User page view tracking — feeds User Activity tab
model PageView {
  id        String   @id @default(cuid())
  userId    String
  path      String
  title     String?
  duration  Int?     // seconds
  ip        String?
  userAgent String?
  createdAt DateTime @default(now())
  user      User     @relation(fields: [userId], references: [id])
  @@map("page_views")
}

// API token for external access (Bearer auth)
model ApiToken {
  id          String    @id @default(cuid())
  userId      String
  name        String
  tokenHash   String    @unique  // SHA-256 hash (never store plaintext)
  scopes      String[]
  expiresAt   DateTime?
  lastUsedAt  DateTime?
  createdAt   DateTime  @default(now())
  user        User      @relation(fields: [userId], references: [id])
}

// OCS Inventory server configuration
model OcsServer {
  id        String   @id @default(cuid())
  name      String
  apiUrl    String
  username  String
  password  String
  enabled   Boolean  @default(true)
  isDefault Boolean  @default(false)
  assets    Asset[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  @@map("ocs_servers")
}

// Bitdefender GravityZone server configuration
model GravityZoneServer {
  id         String    @id @default(cuid())
  name       String
  accessUrl  String
  apiKey     String
  enabled    Boolean   @default(true)
  isDefault  Boolean   @default(false)
  lastSyncAt DateTime?
  createdAt  DateTime  @default(now())
  updatedAt  DateTime  @updatedAt
  @@map("gravityzone_servers")
}
```

**User Model Fields (required for Users & Security tab):**

```prisma
model User {
  // ... existing fields ...
  failedLoginAttempts Int       @default(0)
  lockedUntil         DateTime?
  isActive            Boolean   @default(true)
  lastLoginAt         DateTime?
  role                String    @default("VIEWER") // ADMIN, RISK_OWNER, AUDITOR, VIEWER
  department          String?
}
```

---

### 50.22 File Structure for Replication

```
src/
├── app/
│   ├── (dashboard)/
│   │   └── superadmin-lala/              # Stealth admin page
│   │       └── page.tsx                  # Server component (~43 lines)
│   └── api/
│       ├── admin/
│       │   └── superadmin/
│       │       └── route.ts              # Unified GET/POST handler (~793 lines)
│       └── user/
│           └── activity/
│               └── route.ts              # Page view tracking API (~83 lines)
├── components/
│   └── superadmin/
│       └── superadmin-dashboard.tsx       # Client component (~4,172 lines)
├── lib/
│   ├── superadmin-queries.ts             # Server-side data fetchers (~510 lines)
│   ├── api-auth.ts                       # Dual auth: session + Bearer token (~109 lines)
│   ├── db-backup.ts                      # S3 backup: pg_dump → gzip → upload
│   ├── storage.ts                        # S3 config management + cache
│   ├── ocs-client.ts                     # OCS Inventory REST client + cache
│   ├── gravityzone-client.ts             # GravityZone JSON-RPC 2.0 client + cache
│   ├── cloudflare-ai.ts                  # Multi-provider AI client + cache
│   └── audit-log.ts                      # Audit trail utility
└── generated/
    └── prisma/
        └── client/                        # Generated Prisma client
```

**npm Dependencies:**

```json
{
  "@aws-sdk/client-s3": "^3.x",        // S3 storage + backup
  "pdfkit": "^0.x",                      // PDF generation (certificates, reports)
  "exceljs": "^4.x",                    // Excel export
  "node-cron": "^3.x",                  // Scheduled tasks (daily backup, overdue check)
  "nodemailer": "^6.x",                 // Email sending (SMTP)
  "lucide-react": "^0.x",              // Icons
  "next-themes": "^0.x"                // Dark mode support
}
```

---

### 50.23 Quick Start Checklist for Replication

| # | Task | Priority | Tab(s) Affected |
|---|------|----------|-----------------|
| 1 | Create `SystemConfig` model (key-value store) | Critical | 7, 8, 11 |
| 2 | Create `AuditLog` model | Critical | 3, 5 |
| 3 | Create `PageView` model | High | 4 |
| 4 | Add security fields to User model (`failedLoginAttempts`, `lockedUntil`, `isActive`, `lastLoginAt`) | Critical | 3 |
| 5 | Build unified API route (single file, `section` GET + `action` POST dispatch) | Critical | All |
| 6 | Build 5 server-side query functions (`getSystemHealth`, `getDatabaseStats`, `getUserSecurity`, `getAuditAnalytics`, `getEnvironmentConfig`) | Critical | 1, 2, 3, 5, 6 |
| 7 | Build server component page (auth check + parallel fetch + pass to client) | Critical | — |
| 8 | Build client dashboard (13 tab sub-components, ~4K lines) | Critical | All |
| 9 | Implement `StatCard` and `CollapsibleCard` reusable components | High | All |
| 10 | Implement 3-tier confirmation dialogs | High | 2, 3, 5, 13 |
| 11 | Implement secret masking (`maskSecret`, `maskApiKey`) | High | 6, 7, 8, 9, 10 |
| 12 | Implement cache clearing pattern for each integration | High | 7, 8, 9, 10 |
| 13 | Set up stealth route (no sidebar link, ADMIN-only redirect) | Medium | — |
| 14 | Add tab state persistence in localStorage | Medium | — |
| 15 | Create `OcsServer` model (if applicable) | Optional | 9 |
| 16 | Create `GravityZoneServer` model (if applicable) | Optional | 10 |
| 17 | Create `ApiToken` model for Bearer auth support | Optional | — |
| 18 | Build page view tracking (client hook + API route) | Medium | 4 |

---

> **Document End** — {PROJECT_NAME} Super Admin Documentation Template
>
> This document covers 50 sections with 130+ implementation items across 13 categories.
> Use the master checklist (Section 45) to track core implementation progress.
> Sections 46-49 cover additional security hardening, compliance, and future roadmap.
> Section 50 documents the live EHS SuperAdmin implementation as a comprehensive reference for replication.
