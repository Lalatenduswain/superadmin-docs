# Universal SaaS Setup Wizard — Complete Prompt

> Copy everything below and use it as a prompt for any SaaS application.

---

## Prompt: Build a Production-Grade Initial Setup Wizard for a SaaS Application

Build a multi-step initial setup wizard at `/setup` that configures a SaaS application on first launch. This wizard runs once, is blocked after completion, and follows 2024-2026 industry best practices from Slack, Notion, Linear, Vercel, Supabase, Stripe, HubSpot, and GitLab.

---

## Design Principles (Non-Negotiable)

1. **Time-to-value under 3 minutes** — users must see the product working within the wizard
2. **Smart defaults everywhere** — pre-fill using geo-detection, industry templates, and sensible defaults so users can click "Next" without typing
3. **Auto-save progress after every step** — if the user closes the tab, resume where they left off
4. **Single-purpose steps** — one decision per screen, never more than 4-5 fields per step
5. **Skip-friendly** — every non-essential step has a prominent "Skip for now" link with text: "You can configure this later in Settings"
6. **Mobile-first** — single column on mobile, full-width buttons (min 48px touch targets), bottom-fixed CTAs, appropriate `inputmode` attributes
7. **Accessible (WCAG 2.1 AA)** — visible labels (not placeholder-only), `aria-current="step"`, `aria-describedby` for errors, focus management on step change, 4.5:1 contrast, full keyboard navigation, respect `prefers-reduced-motion`
8. **Dark mode from pixel one** — respect `prefers-color-scheme` on first load, no flash of wrong theme
9. **Show estimated time** — display "Takes about 3 minutes" on the welcome screen
10. **Celebrate completion** — confetti or checkmark animation on the final step, then redirect to a guided dashboard

---

## Overall UI Layout

```
┌─────────────────────────────────────────────────────┐
│  [App Icon]  App Name Setup                         │
│  Configure your application                         │
│                                                     │
│  ● ── ○ ── ○ ── ○ ── ○ ── ○ ── ○ ── ○ ── ○ ── ○   │
│  1    2    3    4    5    6    7    8    9    10      │
├─────────────────────────────────────────────────────┤
│                                                     │
│              [ Step Content Area ]                   │
│              min-height: 400px                       │
│              max-width: 3xl (768px)                  │
│                                                     │
├─────────────────────────────────────────────────────┤
│  [← Back]                    [Skip]  [Continue →]   │
└─────────────────────────────────────────────────────┘
```

- **Background:** Full-screen gradient (light: gray-50 → gray-100, dark: gray-900 → gray-950)
- **Card:** Centered, rounded-xl, border, shadow-lg, glassmorphism optional
- **Step indicator:** Horizontal numbered badges with labels:
  - Completed: green bg + checkmark icon
  - Current: primary color + ring/pulse border
  - Future: muted gray
  - On mobile: collapse to "Step 2 of 10" text
- **Transitions:** Slide-left (forward) / slide-right (back), 200-300ms, ease-out enter / ease-in exit. Respect `prefers-reduced-motion`.
- **Validation:** Inline on blur (not while typing), 300ms debounce, green checkmarks for valid fields, specific actionable error messages below each field. Never disable the Next button for empty fields — show errors on click instead.

---

## Step 0: Welcome & Environment Check

**Purpose:** First impression + verify system requirements.

**UI:**
- Large app icon/logo centered
- Welcome heading: "Welcome to [App Name]"
- Subtitle: "Let's get your workspace ready. This takes about 3 minutes."
- Auto-detected info displayed as read-only cards:
  - Browser: detected browser + version (green check or warning)
  - Timezone: auto-detected from `Intl.DateTimeFormat().resolvedOptions().timeZone`
  - Locale: auto-detected from `navigator.language`
  - System theme: Light/Dark (auto-detected from `prefers-color-scheme`)
- Allow user to override timezone and locale here
- Social proof: "Trusted by X,000+ organizations" (if applicable)

**Fields:**

| Field | Type | Required | Default |
|-------|------|----------|---------|
| Timezone | searchable select | Yes | Auto-detected (IANA identifier) |
| Language/Locale | select | Yes | Auto-detected |
| Date Format | select | No | Based on locale (DD/MM/YYYY or MM/DD/YYYY) |
| Theme Preference | toggle (System/Light/Dark) | No | System |

**Single CTA:** "Get Started →"

---

## Step 1: Database Configuration

**Purpose:** Configure and test database connection.

> For cloud-hosted SaaS (where you manage the DB), skip this step entirely. Include only for self-hosted / on-premises deployments.

**UI:**
- Preset selector (visual cards with icons):
  - On-Premises PostgreSQL
  - AWS RDS
  - Azure Database
  - Google Cloud SQL
  - Supabase
  - Neon
  - PlanetScale (MySQL)
  - Custom

**Connection fields (shown based on preset):**

| Field | Type | Default | Shown When |
|-------|------|---------|------------|
| Connection String | textarea | — | Preset = cloud providers |
| Host | text | localhost | Preset = on-premises/custom |
| Port | number | 5432 | Preset = on-premises/custom |
| Database Name | text | app_db | Preset = on-premises/custom |
| Username | text | postgres | Preset = on-premises/custom |
| Password | password | — | Preset = on-premises/custom |
| SSL | toggle | Off (on-prem) / On (cloud) | Always |

**"Test Connection" button with states:**
- Idle: outline button
- Testing: spinner + "Testing..."
- Success: green badge — "Connected - PostgreSQL 16.2 - 0 tables"
- Failure: red badge — friendly message (see error mapping below)

**Error mapping:**

| Error | Friendly Message |
|-------|-----------------|
| ECONNREFUSED | "Cannot reach the database server. Is PostgreSQL running on {host}:{port}?" |
| Host not found | "Could not resolve hostname '{host}'. Check the address." |
| Auth failure | "Authentication failed. Check username and password." |
| SSL required | "This server requires SSL. Enable the SSL toggle above." |
| Timeout | "Connection timed out after 10 seconds. Check firewall rules." |

**API:** `POST /api/setup/test-db` — connects with 10s timeout, returns `{ success, version, tableCount, hasExistingData }`

**Validation:** Cannot proceed until `dbTested = true`.

---

## Step 2: Organization Information

**Purpose:** Capture organization identity for branding, templates, and compliance context.

**Fields (2-column grid on desktop, 1-column on mobile):**

| Field | Type | Required | Validation | Options/Notes |
|-------|------|----------|------------|---------------|
| Organization Name | text | Yes | 2-200 chars | Micro-copy: "This appears in reports, emails, and exports" |
| Industry | searchable select | Yes | — | Technology, Healthcare, Finance & Banking, Manufacturing, Government, Education, Energy & Utilities, Retail & E-commerce, Telecommunications, Legal & Professional Services, Non-Profit, Media & Entertainment, Real Estate, Transportation & Logistics, Agriculture, Other |
| Organization Size | radio cards | Yes | — | 1-10, 11-50, 51-200, 201-500, 501-1,000, 1,000-5,000, 5,000+ |
| Country | searchable select | Yes | — | Full ISO 3166 country list, default from geo-IP |
| Website | url | No | Valid URL format | placeholder: "https://example.com" |
| Fiscal Year Start | select | No | — | January (default), April, July, October |

**Smart behavior:**
- Auto-detect country from IP geolocation, pre-select it
- When industry is selected, store it for use in later steps (template selection, default configurations)
- Organization size affects default role templates and module recommendations

---

## Step 3: Admin Account

**Purpose:** Create the super-admin user.

**Fields:**

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| Full Name | text | Yes | 2-200 chars |
| Email | email | Yes | Valid email, trimmed & lowercased |
| Password | password + show/hide toggle | Yes | Min 12 chars, show strength meter |
| Confirm Password | password (same toggle) | Yes | Must match |
| Job Title | text | No | placeholder: "CISO", "IT Manager", "CTO" |
| Department | text | No | placeholder: "IT / Information Security" |
| Phone | tel | No | placeholder: "+91 98765 43210" |

**Password UX (critical — top cause of setup drop-off):**
- Show requirements upfront (not after failure): "At least 12 characters"
- Real-time strength meter (Weak / Fair / Good / Strong) with color bar
- Character counter: "3 more characters needed" when below minimum
- Show/hide toggle applies to both password and confirm fields
- On mismatch: inline red text "Passwords don't match" below confirm field (shown on blur, not while typing)

**Alternative auth options (if supported):**
- "Sign up with Google" / "Sign up with SSO" buttons above the form
- Divider: "or create an account with email"
- If SSO: collect SSO provider URL → validate → redirect for auth → return and continue wizard

**API:** Hash password with bcrypt (12 rounds minimum). Create user with role `SUPER_ADMIN`, `isActive: true`.

---

## Step 4: Branding & Appearance

**Purpose:** Customize the look and feel. Entirely optional — smart defaults provided.

**Fields:**

| Field | Type | Default | Notes |
|-------|------|---------|-------|
| Primary Color | color picker + hex input | #2563eb (blue-600) | Used for buttons, links, active states |
| Accent Color | color picker + hex input | #6366f1 (indigo-500) | Used for highlights, badges, secondary actions |
| Logo | file upload (PNG/SVG, max 2MB) | App default | Show preview at actual render size |
| Favicon | file upload (ICO/PNG, max 512KB) | App default | 32x32 preview |
| Font Family | select | System default (Inter) | Inter, Plus Jakarta Sans, DM Sans, Poppins, Roboto, System Default |

**Live preview panel (right side on desktop, below on mobile):**
- Mini mockup showing how the dashboard header, sidebar, and buttons look with chosen colors
- Updates in real-time as colors change

Micro-copy: "You can refine these later in Settings → Branding"

---

## Step 5: Email / SMTP Configuration (Optional — Can Skip)

**Purpose:** Configure outgoing email for notifications, invitations, and alerts.

Header note: "You can skip this and configure later. Some features (invitations, alerts) won't work until email is set up."

**Preset selector (cards):**
- Gmail / Google Workspace
- Microsoft 365 / Outlook
- Amazon SES
- SendGrid
- Mailgun
- Custom SMTP
- Skip for now

**Fields (shown when preset selected):**

| Field | Type | Placeholder/Default | Required |
|-------|------|---------------------|----------|
| SMTP Host | text | Auto-filled per preset | Yes (if not skipped) |
| SMTP Port | number | 587 (TLS) or 465 (SSL) | Yes |
| Encryption | select | STARTTLS | TLS, SSL, None |
| Username | text | — | Yes |
| Password | password | — | Yes |
| From Name | text | defaults to org name | No |
| From Email | email | defaults to admin email | No |

**"Send Test Email" button:**
- Sends a test email to the admin's email address
- Shows SMTP conversation log in a collapsible debug panel on failure
- Success: green badge "Test email sent to admin@example.com"
- Failure: specific error (auth failed, connection refused, TLS error, relay denied)

**API:** `POST /api/setup/test-email` — connects, sends test, returns detailed diagnostics

---

## Step 6: File Storage Configuration (Optional — Can Skip)

**Purpose:** Configure where uploaded files, evidence, and attachments are stored.

**Preset selector (visual cards):**

| Preset | Icon | Auto-config |
|--------|------|-------------|
| Local Server (default) | Local | Path: uploads/, test write access |
| Amazon S3 | Cloud | Standard S3 fields |
| Google Cloud Storage | Cloud | GCS fields |
| Azure Blob Storage | Cloud | Azure fields |
| MinIO (Self-Hosted) | Self-hosted | S3-compatible + forcePathStyle=true |
| DigitalOcean Spaces | Cloud | S3-compatible |
| Wasabi | Cloud | S3-compatible |
| Backblaze B2 | Cloud | S3-compatible |
| Cloudflare R2 | Cloud | S3-compatible |

**S3-compatible fields (shown when cloud selected):**

| Field | Type | Required | Default |
|-------|------|----------|---------|
| Endpoint URL | url | Yes | Auto-filled per preset |
| Region | text | No | us-east-1 |
| Bucket Name | text | Yes | app-uploads |
| Path Prefix | text | No | uploads |
| Access Key ID | text | Yes | — |
| Secret Access Key | password | Yes | — |
| Force Path Style | checkbox | No | Auto-on for MinIO |

**"Test Connection" button:**
- Local: verifies write permission to directory
- Cloud: HeadBucket → create bucket if 404 → upload test file → delete test file
- Shows clear error for: access denied, bucket not found, endpoint unreachable, SSL error

**API:** `POST /api/setup/test-storage`

---

## Step 7: Authentication & Security Settings

**Purpose:** Configure how users log in and security policies.

### Section A — Login Methods (checkboxes):
- Email + Password (always on)
- Google OAuth
- Microsoft / Azure AD
- GitHub
- SAML SSO (Enterprise)
- LDAP / Active Directory

For each enabled OAuth provider, show fields: Client ID, Client Secret, Tenant/Domain (if Azure)

### Section B — Password Policy:

| Setting | Type | Default |
|---------|------|---------|
| Minimum Length | number (range 8-128) | 12 |
| Require Uppercase | toggle | On |
| Require Numbers | toggle | On |
| Require Special Characters | toggle | On |
| Password Expiry | select | 90 days (Never, 30, 60, 90, 180, 365 days) |
| Max Login Attempts | number | 5 (then lockout) |
| Lockout Duration | select | 15 minutes |

### Section C — Multi-Factor Authentication:

| Setting | Type | Default |
|---------|------|---------|
| MFA Enforcement | select | Optional for all users |
| Options | — | Required for admins, Required for all, Optional |
| Allowed Methods | checkboxes | [x] Authenticator App (TOTP), [ ] SMS, [ ] Hardware Key (WebAuthn) |

### Section D — Session Settings:

| Setting | Type | Default |
|---------|------|---------|
| Session Timeout | select | 8 hours |
| Idle Timeout | select | 30 minutes |
| Concurrent Sessions | select | Unlimited |

Micro-copy: "Recommended defaults are pre-selected based on industry best practices. Adjust only if your organization has specific requirements."

---

## Step 8: Module / Feature Selection

**Purpose:** Enable/disable application modules to reduce overwhelm.

**UI pattern:** Scrollable grid of module cards with toggle switches. Each card shows:
- Icon + Module Name
- One-line description
- ISO clause reference (if applicable)
- Toggle switch (on/off)

**Quick-select buttons at top:**
- "Enable All" — turns everything on
- "Essentials Only" — enables only core modules (see below)
- "Recommended for [Industry]" — AI-selected based on Step 2's industry choice

**Module list (adapt to your application domain):**

| # | Module | Description | Essential? |
|---|--------|-------------|------------|
| 1 | Dashboard & Analytics | Central overview, KPIs, trends | Always on |
| 2 | User & Team Management | Users, roles, departments, permissions | Always on |
| 3 | [Core Module 1] | e.g., Risk Management | Yes |
| 4 | [Core Module 2] | e.g., Asset Management | Yes |
| 5 | [Core Module 3] | e.g., Document Control | Yes |
| 6 | [Core Module 4] | e.g., Incident Management | Yes |
| 7 | [Core Module 5] | e.g., Audit Management | Yes |
| 8 | [Optional Module 1] | e.g., Training & Awareness | No |
| 9 | [Optional Module 2] | e.g., Vendor/Third-Party Mgmt | No |
| 10 | [Optional Module 3] | e.g., Business Continuity | No |
| 11 | [Optional Module 4] | e.g., Vulnerability Management | No |
| 12 | [Optional Module 5] | e.g., Change Management | No |
| 13 | [Optional Module 6] | e.g., Physical Security | No |
| 14 | [Optional Module 7] | e.g., Compliance Tracking | No |
| 15 | Reports & Exports | PDF reports, CSV exports, scheduled reports | Yes |

**Data model:** `disabledModules: string[]` stored in SystemConfig.

Note at bottom: "Disabled modules can be activated anytime from Settings → Modules"

---

## Step 9: Team Invitations (Optional — Can Skip)

**Purpose:** Invite initial team members before launch.

**UI:**
- Dynamic list of invite rows (start with 3 empty rows, "Add another" button)
- Each row:

| Field | Type | Required |
|-------|------|----------|
| Email | email | Yes (per row) |
| Role | select | Yes — Admin, Manager, User, Viewer, Auditor (adapt to your RBAC) |
| Department | text | No |

**Bulk invite options:**
- **CSV Upload:** Upload a CSV with columns: Name, Email, Role, Department
  - Show preview table with validation (green rows = valid, red = errors)
  - Template download link: "Download CSV template"
- **Copy invite link:** Generate a shareable link with pre-assigned role

**Behavior:**
- Invitations are queued, not sent immediately (sent on setup completion if email is configured)
- If email is not configured, show invite links that can be shared manually
- Show note: "Invitations will be sent when you complete setup" or "Share these links directly since email is not configured"

---

## Step 10: Sample Data & Initial Content

**Purpose:** Optionally populate the system with reference data and demo content to accelerate onboarding.

**UI:** Toggle cards with descriptions.

**Toggle 1: Reference Data (Recommended: ON)**
- Pre-populates the system with industry-standard reference data
- Examples: ISO 27001 controls, risk categories, asset types, compliance frameworks, document templates
- Micro-copy: "Gives you a head start. You can modify or delete this data anytime."

**Toggle 2: Demo / Sample Data (Default: ON for trial, OFF for production)**
- Creates realistic sample records so you can explore the product immediately
- Shows breakdown: "Will create: ~10 sample [records], ~5 sample [items], ~8 sample [documents]"
- Micro-copy: "Perfect for exploring. Can be removed with one click from Settings → Data → Clear Demo Data"

**Toggle 3: Demo Users (Default: OFF)**
- Creates test users with different roles for testing RBAC
- Lists the users that will be created with their roles
- Micro-copy: "For testing only. Uses the same password as your admin account."

**Toggle 4: Industry Templates (Default: ON)**
- Pre-loads templates relevant to the selected industry
- Examples: Policy templates, risk assessment templates, audit checklists
- Micro-copy: "Customized for [Industry] based on your selection in Step 2"

---

## Step 11: Review & Complete

**Purpose:** Read-only summary of all configuration before finalizing.

**Layout:** Card sections with edit buttons (pencil icon → jumps back to that step).

| Section | Display |
|---------|---------|
| Environment | Timezone, locale, date format, theme |
| Database | Preset + connection info (masked password) |
| Organization | Name, industry, size, country, fiscal year |
| Admin | Name, email (no password shown) |
| Branding | Color swatches + logo thumbnail |
| Email | Host:port or "Not configured — skip" |
| Storage | Provider + bucket/path or "Local disk" |
| Security | Login methods, MFA policy, password policy summary |
| Modules | "X of Y enabled" with list |
| Team | "X invitations queued" or "Skipped" |
| Sample Data | What will be seeded |

**Legal section (before the button):**
- "I agree to the [Terms of Service] and [Privacy Policy]" (required checkbox, not pre-checked)
- Link to privacy policy and terms opens in new tab

**CTA Button:** "Complete Setup & Launch" (primary, full-width)

**Progress overlay on click:**

```
Setting up your workspace...

  ✓ Creating admin account          [done]
  ✓ Applying organization settings  [done]
  ⟳ Configuring security policies   [in progress]
  ○ Seeding reference data
  ○ Sending team invitations
  ○ Finalizing
```

On error: Red alert with specific error + "Retry" button. Never lose the user's data.

---

## Completion API (`POST /api/setup/complete`)

**Sequence:**

1. **Guard:** Check `setup_complete` flag → reject with 400 if already done
2. **Validate:** All input through Zod schema
3. **Transaction** (wrap steps 4-10 in a DB transaction):
4. **Create admin user** — hash password (bcrypt, 12 rounds), role: `SUPER_ADMIN`
5. **Store SystemConfig** — upsert all key-value pairs:
   - `setup_complete: "true"`, `setup_completed_at: ISO timestamp`
   - Organization fields, branding, SMTP, storage, security policies, disabled modules
6. **Save infrastructure config to disk** — `data/config.json` (outside web root)
7. **Seed reference data** (if enabled)
8. **Seed demo data** (if enabled) — link to admin user
9. **Create demo users** (if enabled) — hash with same password
10. **Queue team invitations** (if any) — send via configured email or generate invite links
11. **Set up default notification preferences**
12. **Create audit log entry:** "System setup completed by {admin}"

**Response:**

```json
{
  "success": true,
  "adminId": "uuid",
  "adminEmail": "admin@company.com",
  "referenceDataSeeded": 93,
  "demoDataSeeded": { "records": 10, "items": 5, "documents": 8 },
  "demoUsersCreated": 5,
  "invitationsQueued": 3,
  "redirectUrl": "/dashboard?onboarding=true"
}
```

---

## Post-Setup: Guided Dashboard

After setup completes, redirect to `/dashboard?onboarding=true` which shows:

**Onboarding checklist widget** (persistent sidebar or banner until dismissed):
- Explore the dashboard
- Create your first [record]
- Upload a document
- Import existing data (CSV)
- Invite a team member
- Review security settings
- Schedule your first [audit/review]

**Progress ring:** "3 of 7 complete" with percentage. Completing all items triggers a celebration animation.

**Empty states:** Every module page should show a helpful empty state with:
- Illustration
- "No [items] yet"
- Primary CTA: "Create your first [item]"
- Secondary: "Import from CSV"

---

## Security & Protection

| Concern | Implementation |
|---------|---------------|
| One-time setup | After `setup_complete = "true"`, `/setup` redirects to `/login` |
| No auth required | All `/api/setup/*` routes are public (only before setup) |
| Password hashing | bcrypt with 12+ rounds |
| Input validation | Zod schemas server-side on every API route |
| CSRF protection | Include CSRF token in setup form |
| Rate limiting | Max 5 setup attempts per IP per hour |
| Config file security | DB credentials in `data/config.json` outside web root, chmod 600 |
| Secrets in transit | All setup API calls over HTTPS only |
| Audit trail | Log setup completion with IP, timestamp, user agent |

---

## Zod Validation Schema

```typescript
const setupSchema = z.object({
  // Step 0: Environment
  timezone: z.string().default("UTC"),
  locale: z.string().default("en-US"),
  dateFormat: z.enum(["DD/MM/YYYY", "MM/DD/YYYY", "YYYY-MM-DD"]).default("DD/MM/YYYY"),
  theme: z.enum(["system", "light", "dark"]).default("system"),

  // Step 1: Database (optional for cloud-hosted)
  dbPreset: z.string().optional(),
  dbConnectionString: z.string().optional(),
  dbHost: z.string().optional(),
  dbPort: z.string().default("5432"),
  dbName: z.string().optional(),
  dbUser: z.string().optional(),
  dbPassword: z.string().optional(),
  dbSsl: z.boolean().default(false),

  // Step 2: Organization
  orgName: z.string().min(2).max(200),
  industry: z.string().min(1),
  orgSize: z.string().min(1),
  country: z.string().min(1),
  website: z.string().url().optional().or(z.literal("")),
  fiscalYearStart: z.enum(["january", "april", "july", "october"]).default("january"),

  // Step 3: Admin
  adminName: z.string().min(2).max(200),
  adminEmail: z.string().email().transform(v => v.trim().toLowerCase()),
  adminPassword: z.string().min(12),
  confirmPassword: z.string(),
  adminTitle: z.string().optional(),
  adminDepartment: z.string().optional(),
  adminPhone: z.string().optional(),

  // Step 4: Branding
  primaryColor: z.string().regex(/^#[0-9a-fA-F]{6}$/).default("#2563eb"),
  accentColor: z.string().regex(/^#[0-9a-fA-F]{6}$/).default("#6366f1"),
  fontFamily: z.string().default("inter"),

  // Step 5: Email
  smtpHost: z.string().optional(),
  smtpPort: z.coerce.number().default(587),
  smtpEncryption: z.enum(["tls", "ssl", "none"]).default("tls"),
  smtpUser: z.string().optional(),
  smtpPassword: z.string().optional(),
  smtpFromName: z.string().optional(),
  smtpFromEmail: z.string().email().optional(),

  // Step 6: Storage
  storageProvider: z.enum(["local", "s3", "gcs", "azure"]).default("local"),
  s3Endpoint: z.string().optional(),
  s3Region: z.string().default("us-east-1"),
  s3Bucket: z.string().optional(),
  s3PathPrefix: z.string().default("uploads"),
  s3AccessKey: z.string().optional(),
  s3SecretKey: z.string().optional(),
  s3ForcePathStyle: z.boolean().default(false),

  // Step 7: Security
  authMethods: z.array(z.string()).default(["email"]),
  passwordMinLength: z.number().min(8).default(12),
  passwordRequireUppercase: z.boolean().default(true),
  passwordRequireNumbers: z.boolean().default(true),
  passwordRequireSpecial: z.boolean().default(true),
  passwordExpiry: z.number().default(90),
  maxLoginAttempts: z.number().default(5),
  lockoutDuration: z.number().default(15),
  mfaPolicy: z.enum(["off", "optional", "admins", "all"]).default("optional"),
  mfaMethods: z.array(z.string()).default(["totp"]),
  sessionTimeout: z.number().default(480),
  idleTimeout: z.number().default(30),

  // Step 8: Modules
  disabledModules: z.array(z.string()).default([]),

  // Step 9: Team
  invitations: z.array(z.object({
    email: z.string().email(),
    role: z.string(),
    department: z.string().optional(),
  })).default([]),

  // Step 10: Seed Data
  seedReferenceData: z.boolean().default(true),
  seedDemoData: z.boolean().default(true),
  seedDemoUsers: z.boolean().default(false),
  seedIndustryTemplates: z.boolean().default(true),

  // Step 11: Legal
  acceptedTerms: z.boolean().refine(v => v === true, "You must accept the terms"),
}).refine(data => data.adminPassword === data.confirmPassword, {
  message: "Passwords don't match",
  path: ["confirmPassword"],
});
```

---

## Analytics Events to Track

| Event | When | Why |
|-------|------|-----|
| `setup_started` | Step 0 loaded | Measure setup funnel entry |
| `setup_step_completed` | Each step "Next" click | Identify drop-off points |
| `setup_step_skipped` | Each "Skip" click | Know which steps users skip most |
| `setup_step_back` | Each "Back" click | Identify confusing steps |
| `setup_test_db` | DB test clicked | Track infra issues |
| `setup_test_email` | Email test clicked | Track email config issues |
| `setup_test_storage` | Storage test clicked | Track storage config issues |
| `setup_completed` | Final submit success | Measure completion rate (target: 60%+) |
| `setup_error` | Final submit failure | Track error types |
| `setup_abandoned` | Tab closed mid-wizard | Measure abandonment (trigger re-engagement email) |
| `setup_time_total` | Completion | Measure time-to-value (target: <3 min) |

---

## Key Implementation Notes

1. **State management:** Use React state (or Zustand/Jotai) for wizard data. Persist to `localStorage` after each step for resume capability.
2. **Step navigation:** Allow clicking completed steps in the stepper to go back. Preserve all data when navigating.
3. **Conditional steps:** Skip database step for cloud-hosted deployments. Skip storage step if using built-in storage.
4. **Error recovery:** If the completion API fails mid-way, return which steps succeeded so the user can retry without re-doing everything.
5. **First-run detection:** Middleware checks `setup_complete` flag. If false, redirect ALL routes to `/setup`. If true, redirect `/setup` to `/login`.
6. **Responsive images:** Use SVG illustrations that adapt to light/dark mode.
7. **Internationalization:** All wizard text should go through i18n from day one if you plan to support multiple languages.

---

> Adapt the module names, reference data, industry options, and domain-specific fields to match your application's domain. The wizard structure, UX patterns, validation approach, API architecture, and security model are universal and production-ready for any SaaS application.
