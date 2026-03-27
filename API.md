# API Reference

> Complete REST API documentation for the SuperAdmin SaaS platform.

---

## Table of Contents

1. [Base URL and Versioning](#1-base-url-and-versioning)
2. [Authentication](#2-authentication)
3. [Rate Limiting](#3-rate-limiting)
4. [Common Response Formats](#4-common-response-formats)
5. [Error Codes](#5-error-codes)
6. [Pagination, Filtering, and Sorting](#6-pagination-filtering-and-sorting)
7. [Endpoint Groups](#7-endpoint-groups)
   - [Auth](#71-auth)
   - [Users](#72-users)
   - [Tenants](#73-tenants)
   - [Roles](#74-roles)
   - [Permissions](#75-permissions)
   - [Audit Logs](#76-audit-logs)
   - [System Configuration](#77-system-configuration)
   - [Health and Monitoring](#78-health-and-monitoring)
   - [Feature Flags](#79-feature-flags)
   - [Licenses](#710-licenses)
   - [Webhooks](#711-webhooks)
   - [File Upload](#712-file-upload)
   - [Security](#713-security)
   - [Analytics](#714-analytics)
   - [Database Management](#715-database-management)
8. [WebSocket Events](#8-websocket-events)
9. [OpenAPI and Swagger](#9-openapi-and-swagger)

---

## 1. Base URL and Versioning

All API requests use the following base URL pattern:

```
https://{DOMAIN}/api/v1
```

| Environment | Base URL |
|------------|----------|
| Development | `http://localhost:3000/api/v1` |
| Staging | `https://staging.{DOMAIN}/api/v1` |
| Production | `https://{DOMAIN}/api/v1` |

### Versioning Strategy

The API uses URL-based versioning. When breaking changes are introduced, a new version is created while the previous version remains available for a deprecation period of 12 months.

```
/api/v1/users    # Current stable
/api/v2/users    # Next version (when available)
```

### Content Type

All requests and responses use JSON:

```
Content-Type: application/json
Accept: application/json
```

File upload endpoints accept `multipart/form-data`.

---

## 2. Authentication

### JWT Bearer Tokens

Most endpoints require a valid JWT access token in the `Authorization` header:

```
Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9...
```

Tokens are obtained through the `/auth/login` endpoint and have a configurable expiry (default: 15 minutes). Use the `/auth/refresh` endpoint to obtain a new access token using a refresh token.

### Token Structure

The JWT payload contains:

```json
{
  "sub": "uuid-user-id",
  "email": "user@example.com",
  "role": "ADMIN",
  "tenantId": "uuid-tenant-id",
  "permissions": ["users:read", "users:write"],
  "iat": 1700000000,
  "exp": 1700000900,
  "iss": "superadmin",
  "aud": "superadmin-api"
}
```

### API Keys

For server-to-server integrations, API keys provide stateless authentication:

```
X-API-Key: sa_live_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

API keys are scoped to a tenant and a specific set of permissions. They do not expire but can be revoked at any time. Create and manage API keys through the dashboard or the `/api/v1/api-keys` endpoints.

### Access Levels

| Level | Description |
|-------|-------------|
| Public | No authentication required |
| Protected | Valid JWT or API key required |
| Admin | Requires `ADMIN` role or higher |
| Super Admin | Requires `SUPER_ADMIN` role |

---

## 3. Rate Limiting

All API responses include rate limiting headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 97
X-RateLimit-Reset: 1700001800
Retry-After: 60
```

### Rate Limits by Endpoint Type

| Endpoint Category | Requests | Window |
|-------------------|----------|--------|
| Authentication (login, register) | 5 | 15 minutes |
| Password reset | 3 | 15 minutes |
| MFA verification | 5 | 5 minutes |
| General API (authenticated) | 100 | 15 minutes |
| Admin operations | 200 | 15 minutes |
| File uploads | 10 | 15 minutes |
| Webhooks | 50 | 15 minutes |
| Health checks | 300 | 15 minutes |
| Search/autocomplete | 30 | 1 minute |
| Bulk operations | 5 | 15 minutes |

When rate limited, the API returns `429 Too Many Requests` with a `Retry-After` header indicating seconds until the limit resets.

---

## 4. Common Response Formats

### Success Response

```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "timestamp": "2026-03-27T10:30:00.000Z",
    "requestId": "req_abc123def456"
  }
}
```

### Paginated Response

```json
{
  "success": true,
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  },
  "meta": {
    "timestamp": "2026-03-27T10:30:00.000Z",
    "requestId": "req_abc123def456"
  }
}
```

### Error Response

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "message": "Invalid email format",
        "code": "invalid_string"
      }
    ]
  },
  "meta": {
    "timestamp": "2026-03-27T10:30:00.000Z",
    "requestId": "req_abc123def456"
  }
}
```

---

## 5. Error Codes

### HTTP Status Codes

| Code | Meaning |
|------|---------|
| 200 | Success |
| 201 | Created |
| 204 | No Content (successful deletion) |
| 400 | Bad Request (validation error) |
| 401 | Unauthorized (missing or invalid token) |
| 403 | Forbidden (insufficient permissions) |
| 404 | Not Found |
| 409 | Conflict (duplicate resource) |
| 413 | Payload Too Large (file upload) |
| 422 | Unprocessable Entity |
| 429 | Too Many Requests (rate limited) |
| 500 | Internal Server Error |
| 503 | Service Unavailable |

### Application Error Codes

| Error Code | Description |
|------------|-------------|
| `AUTH_INVALID_CREDENTIALS` | Email or password is incorrect |
| `AUTH_ACCOUNT_LOCKED` | Account locked due to repeated failed attempts |
| `AUTH_MFA_REQUIRED` | MFA verification required to complete login |
| `AUTH_MFA_INVALID` | Invalid MFA code |
| `AUTH_TOKEN_EXPIRED` | JWT access token has expired |
| `AUTH_TOKEN_REVOKED` | Token has been blacklisted |
| `AUTH_REFRESH_EXPIRED` | Refresh token has expired |
| `AUTHZ_INSUFFICIENT_ROLE` | User role does not meet the minimum requirement |
| `AUTHZ_MISSING_PERMISSION` | User lacks the required permission |
| `TENANT_NOT_FOUND` | Specified tenant does not exist |
| `TENANT_SUSPENDED` | Tenant account is suspended |
| `TENANT_LIMIT_REACHED` | Tenant has reached their license tier limit |
| `VALIDATION_ERROR` | Request body failed schema validation |
| `RESOURCE_NOT_FOUND` | Requested resource does not exist |
| `RESOURCE_CONFLICT` | Resource already exists (duplicate) |
| `RATE_LIMIT_EXCEEDED` | Too many requests |
| `FILE_TOO_LARGE` | Upload exceeds maximum file size |
| `FILE_TYPE_NOT_ALLOWED` | File MIME type is not in the allowlist |
| `FILE_VIRUS_DETECTED` | ClamAV scan detected malware |
| `CSRF_TOKEN_INVALID` | CSRF token missing or invalid |
| `IP_NOT_ALLOWED` | Client IP not in the tenant allowlist |
| `GEO_RESTRICTED` | Request from a geo-restricted region |
| `WEBHOOK_DELIVERY_FAILED` | Webhook delivery attempt failed |
| `DATABASE_ERROR` | Database operation failed |
| `STORAGE_ERROR` | Object storage operation failed |
| `SMTP_ERROR` | Email delivery failed |

---

## 6. Pagination, Filtering, and Sorting

### Pagination

All list endpoints support cursor-based and offset pagination:

```
GET /api/v1/users?page=2&limit=20
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | integer | 1 | Page number (1-indexed) |
| `limit` | integer | 20 | Items per page (max: 100) |

### Filtering

Filter results using query parameters:

```
GET /api/v1/users?role=ADMIN&status=active&tenantId=uuid
```

Date range filtering:

```
GET /api/v1/audit-logs?startDate=2026-01-01&endDate=2026-03-27
```

Search across text fields:

```
GET /api/v1/users?search=john
```

### Sorting

Sort by any field with ascending or descending order:

```
GET /api/v1/users?sortBy=createdAt&sortOrder=desc
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `sortBy` | string | `createdAt` | Field name to sort by |
| `sortOrder` | string | `desc` | `asc` or `desc` |

Multiple sort fields:

```
GET /api/v1/users?sortBy=role,createdAt&sortOrder=asc,desc
```

---

## 7. Endpoint Groups

### 7.1 Auth

**Base path:** `/api/v1/auth`

#### POST /auth/login

Authenticate a user and receive JWT tokens.

**Access:** Public

**Request:**
```json
{
  "email": "admin@example.com",
  "password": "SecureP@ss123",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzUxMiI...",
    "refreshToken": "eyJhbGciOiJIUzUxMiI...",
    "expiresIn": 900,
    "tokenType": "Bearer",
    "user": {
      "id": "uuid",
      "email": "admin@example.com",
      "name": "Admin User",
      "role": "ADMIN",
      "tenantId": "uuid",
      "mfaEnabled": true
    }
  }
}
```

**Response (MFA Required, 200):**
```json
{
  "success": true,
  "data": {
    "mfaRequired": true,
    "mfaToken": "temp_mfa_token_xxx",
    "methods": ["totp"]
  }
}
```

#### POST /auth/register

Register a new user account.

**Access:** Public

**Request:**
```json
{
  "email": "newuser@example.com",
  "password": "SecureP@ss123",
  "name": "New User",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "newuser@example.com",
    "name": "New User",
    "role": "CUSTOMER",
    "status": "pending_verification"
  }
}
```

#### POST /auth/refresh

Exchange a refresh token for a new access token.

**Access:** Protected

**Request:**
```json
{
  "refreshToken": "eyJhbGciOiJIUzUxMiI..."
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzUxMiI...",
    "expiresIn": 900
  }
}
```

#### POST /auth/logout

Invalidate the current session and blacklist the token.

**Access:** Protected

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "Logged out successfully"
  }
}
```

#### POST /auth/change-password

Change the authenticated user's password.

**Access:** Protected

**Request:**
```json
{
  "currentPassword": "OldP@ss123",
  "newPassword": "NewSecureP@ss456"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "Password changed successfully"
  }
}
```

#### POST /auth/reset-password

Request a password reset email.

**Access:** Public

**Request:**
```json
{
  "email": "user@example.com"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "If an account exists with this email, a reset link has been sent"
  }
}
```

#### POST /auth/mfa/setup

Generate a TOTP secret and QR code for MFA enrollment.

**Access:** Protected

**Response (200):**
```json
{
  "success": true,
  "data": {
    "secret": "JBSWY3DPEHPK3PXP",
    "qrCodeUrl": "data:image/png;base64,...",
    "backupCodes": [
      "a1b2c3d4",
      "e5f6g7h8",
      "i9j0k1l2",
      "m3n4o5p6",
      "q7r8s9t0"
    ]
  }
}
```

#### POST /auth/mfa/verify

Verify a TOTP code to complete MFA setup or login.

**Access:** Protected

**Request:**
```json
{
  "code": "123456",
  "mfaToken": "temp_mfa_token_xxx"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "accessToken": "eyJhbGciOiJIUzUxMiI...",
    "refreshToken": "eyJhbGciOiJIUzUxMiI...",
    "expiresIn": 900
  }
}
```

#### POST /auth/mfa/disable

Disable MFA on the authenticated user's account.

**Access:** Protected

**Request:**
```json
{
  "password": "SecureP@ss123",
  "code": "123456"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "MFA disabled successfully"
  }
}
```

#### POST /auth/force-logout

Force logout a specific user by invalidating all their sessions.

**Access:** Admin

**Request:**
```json
{
  "userId": "uuid-of-target-user"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "User sessions invalidated",
    "sessionsRevoked": 3
  }
}
```

#### POST /auth/bulk-force-logout

Force logout all users across the platform.

**Access:** Super Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "All user sessions invalidated",
    "sessionsRevoked": 1247
  }
}
```

---

### 7.2 Users

**Base path:** `/api/v1/users`

#### GET /users

List users with pagination, search, and filtering.

**Access:** Admin

**Query Parameters:**
```
GET /users?page=1&limit=20&search=john&role=EMPLOYEE&status=active&sortBy=createdAt&sortOrder=desc
```

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "email": "john@example.com",
      "name": "John Doe",
      "role": "EMPLOYEE",
      "status": "active",
      "mfaEnabled": true,
      "lastLoginAt": "2026-03-26T14:30:00.000Z",
      "createdAt": "2025-06-15T09:00:00.000Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrev": false
  }
}
```

#### GET /users/:id

Retrieve full details for a specific user.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "john@example.com",
    "name": "John Doe",
    "role": "EMPLOYEE",
    "status": "active",
    "mfaEnabled": true,
    "tenantId": "uuid",
    "department": "Engineering",
    "permissions": ["users:read", "reports:read"],
    "lastLoginAt": "2026-03-26T14:30:00.000Z",
    "lastLoginIp": "192.168.1.100",
    "loginCount": 245,
    "createdAt": "2025-06-15T09:00:00.000Z",
    "updatedAt": "2026-03-20T11:15:00.000Z"
  }
}
```

#### POST /users

Create a new user.

**Access:** Admin

**Request:**
```json
{
  "email": "newuser@example.com",
  "name": "New User",
  "password": "TempP@ss123",
  "role": "EMPLOYEE",
  "department": "Marketing",
  "tenantId": "uuid"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "newuser@example.com",
    "name": "New User",
    "role": "EMPLOYEE",
    "status": "active",
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### PUT /users/:id

Update a user profile.

**Access:** Admin

**Request:**
```json
{
  "name": "Updated Name",
  "department": "Engineering",
  "status": "active"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Updated Name",
    "department": "Engineering",
    "status": "active",
    "updatedAt": "2026-03-27T10:35:00.000Z"
  }
}
```

#### PUT /users/:id/role

Change a user's role.

**Access:** Admin

**Request:**
```json
{
  "role": "MANAGER"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "previousRole": "EMPLOYEE",
    "newRole": "MANAGER",
    "updatedAt": "2026-03-27T10:40:00.000Z"
  }
}
```

#### PUT /users/:id/status

Activate or deactivate a user.

**Access:** Admin

**Request:**
```json
{
  "status": "suspended",
  "reason": "Policy violation"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "suspended",
    "reason": "Policy violation",
    "updatedAt": "2026-03-27T10:45:00.000Z"
  }
}
```

#### DELETE /users/:id

Soft delete a user (marks as deleted, retains data for audit).

**Access:** Admin

**Response (204):** No content.

#### DELETE /users/:id/permanent

Permanently delete a user and all associated data. Only available for users inactive for 90+ days.

**Access:** Super Admin

**Response (204):** No content.

#### POST /users/:id/reset-password

Admin-initiated password reset for a specific user.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "Password reset email sent to user"
  }
}
```

#### POST /users/import

Bulk import users from a CSV file.

**Access:** Admin

**Request:** `multipart/form-data` with a CSV file attachment.

**Response (200):**
```json
{
  "success": true,
  "data": {
    "imported": 45,
    "skipped": 3,
    "errors": [
      {
        "row": 12,
        "email": "invalid-email",
        "reason": "Invalid email format"
      }
    ]
  }
}
```

#### GET /users/export

Export users as a CSV file.

**Access:** Admin

**Query Parameters:**
```
GET /users/export?role=EMPLOYEE&maskPii=true&format=csv
```

**Response (200):** CSV file download with `Content-Disposition: attachment`.

#### GET /users/inactive

List users inactive for 90+ days.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "email": "inactive@example.com",
      "name": "Inactive User",
      "lastLoginAt": "2025-11-15T09:00:00.000Z",
      "daysSinceLastLogin": 132
    }
  ]
}
```

---

### 7.3 Tenants

**Base path:** `/api/v1/tenants`

#### GET /tenants

List all tenants.

**Access:** Super Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "Acme Corporation",
      "slug": "acme-corp",
      "status": "active",
      "plan": "Professional",
      "userCount": 87,
      "storageUsed": "12.5 GB",
      "createdAt": "2025-01-10T08:00:00.000Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 34, "totalPages": 2 }
}
```

#### POST /tenants

Create a new tenant.

**Access:** Super Admin

**Request:**
```json
{
  "name": "New Company",
  "slug": "new-company",
  "plan": "Starter",
  "adminEmail": "admin@newcompany.com",
  "adminName": "Tenant Admin",
  "settings": {
    "mfaRequired": false,
    "sessionTimeout": 3600,
    "maxUsers": 25
  }
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "New Company",
    "slug": "new-company",
    "status": "active",
    "plan": "Starter",
    "adminUser": {
      "id": "uuid",
      "email": "admin@newcompany.com",
      "role": "TENANT_ADMIN"
    },
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### PUT /tenants/:id

Update tenant configuration.

**Access:** Super Admin

**Request:**
```json
{
  "name": "Updated Company Name",
  "plan": "Professional",
  "settings": {
    "mfaRequired": true,
    "maxUsers": 100
  }
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "Updated Company Name",
    "plan": "Professional",
    "updatedAt": "2026-03-27T10:35:00.000Z"
  }
}
```

#### PUT /tenants/:id/suspend

Suspend a tenant account.

**Access:** Super Admin

**Request:**
```json
{
  "reason": "Payment overdue",
  "notifyAdmin": true
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "suspended",
    "suspendedAt": "2026-03-27T10:40:00.000Z"
  }
}
```

---

### 7.4 Roles

**Base path:** `/api/v1/roles`

#### GET /roles

List all roles in the RBAC hierarchy.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "SUPER_ADMIN",
      "displayName": "Super Administrator",
      "level": 1,
      "description": "Full platform access across all tenants",
      "isSystem": true,
      "userCount": 2,
      "permissionCount": 70
    },
    {
      "id": "uuid",
      "name": "ADMIN",
      "displayName": "Administrator",
      "level": 2,
      "description": "Full tenant management access",
      "isSystem": true,
      "userCount": 5,
      "permissionCount": 55
    },
    {
      "id": "uuid",
      "name": "TENANT_ADMIN",
      "displayName": "Tenant Administrator",
      "level": 3,
      "description": "Tenant-scoped administrative access",
      "isSystem": true,
      "userCount": 12,
      "permissionCount": 40
    },
    {
      "id": "uuid",
      "name": "MANAGER",
      "displayName": "Manager",
      "level": 4,
      "description": "Department and team management",
      "isSystem": true,
      "userCount": 28,
      "permissionCount": 25
    },
    {
      "id": "uuid",
      "name": "TEAM_LEAD",
      "displayName": "Team Lead",
      "level": 5,
      "description": "Team-level oversight",
      "isSystem": true,
      "userCount": 45,
      "permissionCount": 18
    },
    {
      "id": "uuid",
      "name": "EMPLOYEE",
      "displayName": "Employee",
      "level": 6,
      "description": "Standard user access",
      "isSystem": true,
      "userCount": 320,
      "permissionCount": 10
    },
    {
      "id": "uuid",
      "name": "CUSTOMER",
      "displayName": "Customer",
      "level": 7,
      "description": "External user with limited access to own data",
      "isSystem": true,
      "userCount": 1500,
      "permissionCount": 5
    }
  ]
}
```

#### POST /roles

Create a custom role.

**Access:** Admin

**Request:**
```json
{
  "name": "AUDITOR",
  "displayName": "Auditor",
  "level": 4,
  "description": "Read-only access to audit logs and compliance data",
  "permissions": ["audit:read", "reports:read", "compliance:read"]
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "name": "AUDITOR",
    "displayName": "Auditor",
    "level": 4,
    "isSystem": false,
    "permissionCount": 3,
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

---

### 7.5 Permissions

**Base path:** `/api/v1/permissions`

#### GET /permissions

List all available permissions.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "key": "users:read",
      "group": "Users",
      "description": "View user profiles and lists",
      "roles": ["SUPER_ADMIN", "ADMIN", "TENANT_ADMIN", "MANAGER"]
    },
    {
      "id": "uuid",
      "key": "users:write",
      "group": "Users",
      "description": "Create and update user accounts",
      "roles": ["SUPER_ADMIN", "ADMIN", "TENANT_ADMIN"]
    },
    {
      "id": "uuid",
      "key": "users:delete",
      "group": "Users",
      "description": "Delete user accounts",
      "roles": ["SUPER_ADMIN", "ADMIN"]
    }
  ]
}
```

#### GET /permissions/matrix

Get the full role-permission matrix.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "roles": ["SUPER_ADMIN", "ADMIN", "TENANT_ADMIN", "MANAGER", "TEAM_LEAD", "EMPLOYEE", "CUSTOMER"],
    "permissions": [
      {
        "key": "users:read",
        "group": "Users",
        "matrix": [true, true, true, true, false, false, false]
      },
      {
        "key": "users:write",
        "group": "Users",
        "matrix": [true, true, true, false, false, false, false]
      }
    ]
  }
}
```

#### PUT /roles/:id/permissions

Update permissions for a specific role.

**Access:** Super Admin

**Request:**
```json
{
  "permissions": ["users:read", "users:write", "reports:read", "audit:read"]
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "roleId": "uuid",
    "roleName": "MANAGER",
    "permissionCount": 4,
    "updatedAt": "2026-03-27T10:30:00.000Z"
  }
}
```

---

### 7.6 Audit Logs

**Base path:** `/api/v1/audit-logs`

#### GET /audit-logs

Query audit logs with filtering.

**Access:** Admin

**Query Parameters:**
```
GET /audit-logs?action=USER_LOGIN&userId=uuid&startDate=2026-03-01&endDate=2026-03-27&page=1&limit=50
```

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "action": "USER_LOGIN",
      "category": "authentication",
      "userId": "uuid",
      "userName": "John Doe",
      "userEmail": "john@example.com",
      "tenantId": "uuid",
      "ipAddress": "192.168.1.100",
      "userAgent": "Mozilla/5.0...",
      "details": {
        "method": "password",
        "mfaUsed": true
      },
      "status": "success",
      "createdAt": "2026-03-27T10:30:00.000Z"
    }
  ],
  "pagination": { "page": 1, "limit": 50, "total": 12450, "totalPages": 249 }
}
```

#### GET /audit-logs/export

Export audit logs in various formats.

**Access:** Admin

**Query Parameters:**
```
GET /audit-logs/export?format=csv&startDate=2026-03-01&endDate=2026-03-27
```

Supported formats: `csv`, `json`, `pdf`, `xlsx`.

**Response (200):** File download with `Content-Disposition: attachment`.

#### GET /audit-logs/stats

Aggregate audit log statistics.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "totalEvents": 125430,
    "last24Hours": 1245,
    "last7Days": 8723,
    "topActions": [
      { "action": "USER_LOGIN", "count": 45230 },
      { "action": "PAGE_VIEW", "count": 32100 },
      { "action": "SETTINGS_UPDATED", "count": 890 }
    ],
    "failedLoginAttempts": {
      "last24Hours": 23,
      "last7Days": 156
    }
  }
}
```

---

### 7.7 System Configuration

**Base path:** `/api/v1/config`

#### GET /config/settings

Retrieve all system settings.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "general": {
      "appName": "SuperAdmin",
      "timezone": "UTC",
      "dateFormat": "YYYY-MM-DD",
      "currency": "USD"
    },
    "security": {
      "passwordMinLength": 12,
      "passwordRequireUppercase": true,
      "passwordRequireLowercase": true,
      "passwordRequireNumbers": true,
      "passwordRequireSpecial": true,
      "mfaEnforced": false,
      "sessionTimeout": 3600,
      "maxLoginAttempts": 5,
      "lockoutDuration": 900
    },
    "email": {
      "smtpConfigured": true,
      "fromName": "SuperAdmin",
      "fromEmail": "noreply@example.com"
    },
    "storage": {
      "maxFileSize": 52428800,
      "allowedMimeTypes": ["image/png", "image/jpeg", "application/pdf"],
      "virusScanEnabled": true
    }
  }
}
```

#### PUT /config/settings

Update system settings.

**Access:** Admin

**Request:**
```json
{
  "security": {
    "mfaEnforced": true,
    "sessionTimeout": 1800,
    "maxLoginAttempts": 3
  }
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "Settings updated successfully",
    "changes": [
      { "key": "security.mfaEnforced", "oldValue": false, "newValue": true },
      { "key": "security.sessionTimeout", "oldValue": 3600, "newValue": 1800 },
      { "key": "security.maxLoginAttempts", "oldValue": 5, "newValue": 3 }
    ]
  }
}
```

#### GET /config/changelog

View configuration change history.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "key": "security.mfaEnforced",
      "oldValue": "false",
      "newValue": "true",
      "changedBy": "admin@example.com",
      "changedAt": "2026-03-27T10:30:00.000Z",
      "reason": "Security policy update"
    }
  ]
}
```

#### POST /config/revert

Revert a configuration change.

**Access:** Admin

**Request:**
```json
{
  "changeId": "uuid"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "key": "security.mfaEnforced",
    "revertedTo": "false",
    "revertedAt": "2026-03-27T10:35:00.000Z"
  }
}
```

---

### 7.8 Health and Monitoring

**Base path:** `/api/v1`

#### GET /health

Basic health check.

**Access:** Public

**Response (200):**
```json
{
  "status": "healthy",
  "version": "1.0.0",
  "uptime": 864000,
  "timestamp": "2026-03-27T10:30:00.000Z"
}
```

#### GET /monitoring/system-health

Detailed system health with component status.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "status": "healthy",
    "components": {
      "database": {
        "status": "healthy",
        "latencyMs": 2,
        "connections": { "active": 12, "idle": 8, "max": 20 }
      },
      "redis": {
        "status": "healthy",
        "latencyMs": 1,
        "memoryUsed": "256 MB",
        "connectedClients": 15
      },
      "storage": {
        "status": "healthy",
        "latencyMs": 5,
        "bucketsAccessible": true
      },
      "smtp": {
        "status": "healthy",
        "lastTestAt": "2026-03-27T08:00:00.000Z"
      }
    },
    "system": {
      "cpuUsage": 23.5,
      "memoryUsage": 67.2,
      "diskUsage": 45.8,
      "loadAverage": [1.2, 1.5, 1.3]
    }
  }
}
```

#### GET /monitoring/api-metrics

Per-endpoint performance metrics.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "endpoints": [
      {
        "method": "GET",
        "path": "/api/v1/users",
        "avgResponseMs": 45,
        "p95ResponseMs": 120,
        "p99ResponseMs": 250,
        "requestCount": 15230,
        "errorRate": 0.02
      }
    ],
    "summary": {
      "totalRequests": 1250000,
      "avgResponseMs": 38,
      "errorRate": 0.01,
      "uptime": 99.98
    }
  }
}
```

#### GET /monitoring/active-sessions

List currently active user sessions.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "sessionId": "uuid",
      "userId": "uuid",
      "userName": "John Doe",
      "ipAddress": "192.168.1.100",
      "userAgent": "Chrome/120.0",
      "startedAt": "2026-03-27T09:00:00.000Z",
      "lastActivityAt": "2026-03-27T10:28:00.000Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 89 }
}
```

#### GET /monitoring/database-health

Database connection and performance metrics.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "connections": {
      "active": 12,
      "idle": 8,
      "waiting": 0,
      "max": 200
    },
    "databaseSize": "2.4 GB",
    "slowQueries": {
      "last24Hours": 3,
      "threshold": "1000ms"
    },
    "replication": {
      "enabled": true,
      "lag": "0.2s",
      "replicas": 2
    }
  }
}
```

---

### 7.9 Feature Flags

**Base path:** `/api/v1/feature-flags`

#### GET /feature-flags

List all feature flags.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "key": "new-dashboard",
      "name": "New Dashboard UI",
      "description": "Redesigned analytics dashboard with real-time charts",
      "enabled": true,
      "rolloutPercentage": 50,
      "targetRoles": ["ADMIN", "MANAGER"],
      "targetTenants": [],
      "killSwitch": false,
      "createdAt": "2026-02-01T10:00:00.000Z",
      "updatedAt": "2026-03-15T14:00:00.000Z"
    }
  ]
}
```

#### POST /feature-flags

Create a new feature flag.

**Access:** Admin

**Request:**
```json
{
  "key": "ai-chatbot-v2",
  "name": "AI Chatbot V2",
  "description": "Next-generation AI chatbot with multi-model support",
  "enabled": false,
  "rolloutPercentage": 0,
  "targetRoles": [],
  "targetTenants": []
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "key": "ai-chatbot-v2",
    "name": "AI Chatbot V2",
    "enabled": false,
    "rolloutPercentage": 0,
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### PUT /feature-flags/:id

Update a feature flag, including gradual rollout.

**Access:** Admin

**Request:**
```json
{
  "enabled": true,
  "rolloutPercentage": 25,
  "targetTenants": ["uuid-tenant-1", "uuid-tenant-2"]
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "key": "ai-chatbot-v2",
    "enabled": true,
    "rolloutPercentage": 25,
    "updatedAt": "2026-03-27T10:35:00.000Z"
  }
}
```

#### POST /feature-flags/:id/kill

Activate the kill switch to immediately disable a feature.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "key": "ai-chatbot-v2",
    "enabled": false,
    "killSwitch": true,
    "killedAt": "2026-03-27T10:40:00.000Z",
    "killedBy": "admin@example.com"
  }
}
```

---

### 7.10 Licenses

**Base path:** `/api/v1/licenses`

#### GET /licenses

List all license allocations.

**Access:** Super Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "tenantId": "uuid",
      "tenantName": "Acme Corp",
      "tier": "Professional",
      "maxUsers": 100,
      "currentUsers": 87,
      "storageLimit": "100 GB",
      "storageUsed": "45.2 GB",
      "modules": ["core", "analytics", "reporting", "integrations"],
      "support": "Priority email",
      "expiresAt": "2027-01-10T00:00:00.000Z",
      "status": "active",
      "createdAt": "2026-01-10T08:00:00.000Z"
    }
  ]
}
```

#### POST /licenses

Create a new license allocation.

**Access:** Super Admin

**Request:**
```json
{
  "tenantId": "uuid",
  "tier": "Enterprise",
  "duration": "annual",
  "customLimits": {
    "maxUsers": 500,
    "storageLimit": "1 TB"
  }
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "tenantId": "uuid",
    "tier": "Enterprise",
    "maxUsers": 500,
    "storageLimit": "1 TB",
    "licenseKey": "sa_lic_xxxxxxxxxxxxxxxx",
    "expiresAt": "2027-03-27T00:00:00.000Z",
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### GET /licenses/tiers

List available license tiers.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    { "tier": "Free", "maxUsers": 5, "storage": "1 GB", "modules": "Core only", "support": "Community", "duration": "Perpetual" },
    { "tier": "Starter", "maxUsers": 25, "storage": "10 GB", "modules": "Core + Analytics", "support": "Email", "duration": "Annual" },
    { "tier": "Professional", "maxUsers": 100, "storage": "100 GB", "modules": "All standard", "support": "Priority email", "duration": "Annual" },
    { "tier": "Enterprise", "maxUsers": "Unlimited", "storage": "Unlimited", "modules": "All + Custom", "support": "Dedicated", "duration": "Annual" },
    { "tier": "Trial", "maxUsers": 10, "storage": "5 GB", "modules": "All", "support": "Email", "duration": "14 days" }
  ]
}
```

---

### 7.11 Webhooks

**Base path:** `/api/v1/webhooks`

#### GET /webhooks

List configured webhooks.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "url": "https://hooks.example.com/superadmin",
      "events": ["user.created", "user.deleted", "tenant.suspended"],
      "status": "active",
      "secret": "whsec_xxxx...xxxx",
      "lastDeliveryAt": "2026-03-27T10:15:00.000Z",
      "lastDeliveryStatus": 200,
      "failureCount": 0,
      "createdAt": "2026-01-15T09:00:00.000Z"
    }
  ]
}
```

#### POST /webhooks

Create a new webhook subscription.

**Access:** Admin

**Request:**
```json
{
  "url": "https://hooks.example.com/superadmin",
  "events": ["user.created", "user.updated", "user.deleted"],
  "secret": "my_webhook_secret"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "url": "https://hooks.example.com/superadmin",
    "events": ["user.created", "user.updated", "user.deleted"],
    "status": "active",
    "signingKey": "whsec_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

**Webhook Payload Format:**

All webhook deliveries include HMAC-SHA256 signatures in headers:

```
X-Webhook-Signature: sha256=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
X-Webhook-Id: evt_xxxxxxxx
X-Webhook-Timestamp: 1711526400
```

```json
{
  "id": "evt_xxxxxxxx",
  "event": "user.created",
  "timestamp": "2026-03-27T10:30:00.000Z",
  "data": {
    "id": "uuid",
    "email": "newuser@example.com",
    "name": "New User",
    "role": "EMPLOYEE",
    "tenantId": "uuid"
  }
}
```

#### Available Webhook Events

| Event | Description |
|-------|-------------|
| `user.created` | New user account created |
| `user.updated` | User profile updated |
| `user.deleted` | User account deleted |
| `user.login` | User logged in |
| `user.locked` | User account locked |
| `tenant.created` | New tenant created |
| `tenant.suspended` | Tenant suspended |
| `tenant.reactivated` | Tenant reactivated |
| `role.updated` | Role permissions changed |
| `config.changed` | System configuration changed |
| `license.created` | License allocated |
| `license.expired` | License expired |
| `security.threat_detected` | Security threat detected |
| `security.ip_blocked` | IP address blocked |
| `backup.completed` | Database backup completed |
| `backup.failed` | Database backup failed |

#### POST /webhooks/:id/test

Send a test payload to verify the webhook endpoint.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "delivered": true,
    "responseStatus": 200,
    "responseTimeMs": 145,
    "responseBody": "OK"
  }
}
```

#### GET /webhooks/:id/deliveries

View delivery history for a webhook.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "event": "user.created",
      "url": "https://hooks.example.com/superadmin",
      "requestBody": "{ ... }",
      "responseStatus": 200,
      "responseBody": "OK",
      "responseTimeMs": 145,
      "attempt": 1,
      "deliveredAt": "2026-03-27T10:30:00.000Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 340 }
}
```

---

### 7.12 File Upload

**Base path:** `/api/v1/files`

#### POST /files/upload

Upload a file with virus scanning and MIME validation.

**Access:** Protected

**Request:** `multipart/form-data`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `file` | File | Yes | The file to upload |
| `category` | string | No | Category tag (e.g., `avatar`, `document`, `report`) |
| `description` | string | No | File description |

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "filename": "report-q1-2026.pdf",
    "originalName": "Q1 Report.pdf",
    "mimeType": "application/pdf",
    "size": 2456789,
    "category": "document",
    "url": "/api/v1/files/uuid/download",
    "virusScanStatus": "clean",
    "uploadedBy": "uuid",
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### GET /files/:id/download

Download a file.

**Access:** Protected

**Response (200):** Binary file with appropriate `Content-Type` and `Content-Disposition` headers.

#### DELETE /files/:id

Delete a file.

**Access:** Admin

**Response (204):** No content.

#### File Upload Constraints

| Constraint | Value |
|-----------|-------|
| Maximum file size | 50 MB (configurable) |
| Allowed MIME types | Configured per tenant |
| Virus scanning | ClamAV integration (when enabled) |
| Storage backend | MinIO (S3-compatible) |
| Filename sanitization | Strips path traversal, null bytes, special characters |

---

### 7.13 Security

**Base path:** `/api/v1/security`

#### GET /security/ip-allowlist

Retrieve the tenant IP allowlist.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "ip": "192.168.1.0/24",
      "description": "Office network",
      "createdBy": "admin@example.com",
      "createdAt": "2026-01-15T09:00:00.000Z"
    }
  ]
}
```

#### POST /security/ip-allowlist

Add an IP address or CIDR range to the allowlist.

**Access:** Admin

**Request:**
```json
{
  "ip": "10.0.0.0/8",
  "description": "VPN network"
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "ip": "10.0.0.0/8",
    "description": "VPN network",
    "createdAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### GET /security/threats

List detected security threats.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "type": "brute_force",
      "severity": "high",
      "sourceIp": "203.0.113.50",
      "targetUserId": "uuid",
      "description": "15 failed login attempts in 5 minutes",
      "status": "mitigated",
      "mitigationAction": "ip_blocked",
      "detectedAt": "2026-03-27T09:45:00.000Z"
    }
  ]
}
```

#### GET /security/overview

Security dashboard aggregate data.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "threatLevel": "low",
    "activeThreats": 0,
    "blockedIps": 12,
    "lockedAccounts": 2,
    "mfaAdoptionRate": 78.5,
    "failedLogins24h": 23,
    "activeSessions": 89,
    "securityScore": 92
  }
}
```

#### GET /security/locked-users

List locked user accounts.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "email": "user@example.com",
      "name": "Locked User",
      "lockedAt": "2026-03-27T09:30:00.000Z",
      "lockReason": "Exceeded maximum login attempts (5/5)",
      "failedAttempts": 5,
      "lastAttemptIp": "192.168.1.100"
    }
  ]
}
```

#### POST /security/unlock-user

Unlock a locked user account.

**Access:** Admin

**Request:**
```json
{
  "userId": "uuid"
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid",
    "message": "User account unlocked successfully",
    "unlockedAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### DELETE /security/rate-limits/:key

Clear a specific rate limit key.

**Access:** Admin

**Response (204):** No content.

#### DELETE /security/rate-limits/ip/:ip

Clear all rate limits for a specific IP address.

**Access:** Admin

**Response (204):** No content.

#### DELETE /security/rate-limits

Emergency: clear all rate limits.

**Access:** Super Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "message": "All rate limits cleared",
    "keysRemoved": 1523
  }
}
```

---

### 7.14 Analytics

**Base path:** `/api/v1/analytics`

#### GET /analytics/dashboard

Aggregated dashboard data.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "users": {
      "total": 1856,
      "active": 1623,
      "new7Days": 45,
      "new30Days": 187
    },
    "sessions": {
      "activeSessions": 89,
      "avgSessionDuration": "42m",
      "peakConcurrent": 156
    },
    "security": {
      "threatLevel": "low",
      "failedLogins24h": 23,
      "mfaAdoptionRate": 78.5
    },
    "system": {
      "uptime": "99.98%",
      "avgResponseMs": 38,
      "errorRate": 0.01
    }
  }
}
```

#### GET /analytics/users

User statistics.

**Access:** Admin

**Response (200):**
```json
{
  "success": true,
  "data": {
    "totalUsers": 1856,
    "activeUsers": 1623,
    "inactiveUsers": 233,
    "byRole": {
      "SUPER_ADMIN": 2,
      "ADMIN": 5,
      "TENANT_ADMIN": 12,
      "MANAGER": 28,
      "TEAM_LEAD": 45,
      "EMPLOYEE": 320,
      "CUSTOMER": 1444
    },
    "registrationTrend": [
      { "date": "2026-03-21", "count": 8 },
      { "date": "2026-03-22", "count": 12 },
      { "date": "2026-03-23", "count": 5 }
    ]
  }
}
```

#### GET /analytics/export

Generate and download an analytics report.

**Access:** Admin

**Query Parameters:**
```
GET /analytics/export?type=user_activity&format=pdf&startDate=2026-03-01&endDate=2026-03-27
```

**Response (200):** File download.

---

### 7.15 Database Management

**Base path:** `/api/v1/database`

#### POST /database/backup

Trigger a database backup.

**Access:** Super Admin

**Request:**
```json
{
  "type": "full",
  "compress": true,
  "encrypt": true
}
```

**Response (201):**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "type": "full",
    "status": "in_progress",
    "startedAt": "2026-03-27T10:30:00.000Z"
  }
}
```

#### GET /database/backups

List all backups.

**Access:** Super Admin

**Response (200):**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "type": "full",
      "status": "completed",
      "size": "245 MB",
      "compressed": true,
      "encrypted": true,
      "startedAt": "2026-03-27T02:00:00.000Z",
      "completedAt": "2026-03-27T02:03:45.000Z",
      "expiresAt": "2026-04-27T02:00:00.000Z"
    }
  ]
}
```

#### POST /database/restore

Restore from a backup.

**Access:** Super Admin

**Request:**
```json
{
  "backupId": "uuid",
  "confirmRestore": true
}
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "status": "restoring",
    "backupId": "uuid",
    "startedAt": "2026-03-27T10:30:00.000Z",
    "estimatedDuration": "5 minutes"
  }
}
```

#### GET /database/backups/:id/download

Download a backup file.

**Access:** Super Admin

**Response (200):** Binary file download.

#### DELETE /database/backups/:id

Remove an old backup.

**Access:** Super Admin

**Response (204):** No content.

---

## 8. WebSocket Events

The application provides a WebSocket endpoint for real-time updates:

```
wss://{DOMAIN}/ws?token=<jwt_access_token>
```

### Connection

```javascript
const ws = new WebSocket('wss://example.com/ws?token=eyJhbGci...');

ws.onopen = () => {
  console.log('Connected');
  // Subscribe to channels
  ws.send(JSON.stringify({
    type: 'subscribe',
    channels: ['system', 'security', 'users']
  }));
};

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log(message);
};
```

### Event Types

| Channel | Event | Description |
|---------|-------|-------------|
| `system` | `system.health_changed` | System health status changed |
| `system` | `system.config_updated` | Configuration setting changed |
| `system` | `system.backup_completed` | Database backup finished |
| `security` | `security.threat_detected` | New security threat identified |
| `security` | `security.ip_blocked` | IP address automatically blocked |
| `security` | `security.user_locked` | User account locked |
| `users` | `users.created` | New user registered |
| `users` | `users.login` | User logged in |
| `users` | `users.logout` | User logged out |
| `tenants` | `tenants.created` | New tenant provisioned |
| `tenants` | `tenants.suspended` | Tenant suspended |
| `features` | `features.flag_updated` | Feature flag changed |
| `features` | `features.kill_switch` | Kill switch activated |

### Event Payload Format

```json
{
  "type": "event",
  "channel": "security",
  "event": "security.threat_detected",
  "data": {
    "threatId": "uuid",
    "type": "brute_force",
    "severity": "high",
    "sourceIp": "203.0.113.50"
  },
  "timestamp": "2026-03-27T10:30:00.000Z"
}
```

### Heartbeat

The server sends a ping every 30 seconds. Clients must respond with a pong to keep the connection alive. Connections without a pong response for 90 seconds are terminated.

---

## 9. OpenAPI and Swagger

### Interactive Documentation

Swagger UI is available in non-production environments:

```
http://localhost:3000/api/docs
```

### OpenAPI Specification

Download the OpenAPI 3.1 specification:

```
GET /api/v1/openapi.json
GET /api/v1/openapi.yaml
```

### Generate Client SDKs

Use the OpenAPI spec to generate typed client libraries:

```bash
# TypeScript client
npx @openapitools/openapi-generator-cli generate \
  -i http://localhost:3000/api/v1/openapi.json \
  -g typescript-axios \
  -o ./sdk/typescript

# Python client
npx @openapitools/openapi-generator-cli generate \
  -i http://localhost:3000/api/v1/openapi.json \
  -g python \
  -o ./sdk/python
```

### Postman Collection

Import the API into Postman:

```bash
# Export Postman collection from OpenAPI spec
npx openapi-to-postmanv2 \
  -s http://localhost:3000/api/v1/openapi.json \
  -o superadmin-postman.json
```

---

## Additional References

- See Section 41 of `SUPER_ADMIN_DOCUMENTATION.md` for the full API endpoint inventory
- See `15_api_endpoints.csv` for all endpoints with access levels
- See `04_rate_limiting.csv` for rate limiting configuration details
- See `07_permissions.csv` for the 70+ permission matrix
