# Phase 1 Scope

This document defines the exact scope, deliverables, and implementation requirements for **Phase 1: Core Foundation** of the MLC LMS rebuild.

> **Context**: See [initial-project-phases.md](../00-foundations/initial-project-phases.md) for strategic overview and phasing philosophy.

## Purpose

Phase 1 establishes the technical foundation and authentication infrastructure necessary for all subsequent development phases. This phase validates the chosen technology stack, creates the deployment pipeline, and delivers a working authentication system that real users can access.

**What this enables:**
- User registration, login, and session management
- Secure database operations with migrations
- Automated deployment to production environment
- Professional UI framework for consistent design
- Foundation for student and teacher experiences in Phase 2+

**Why it exists:**
- **Risk mitigation**: Validates architecture and technology choices early
- **Foundation first**: Creates infrastructure that all features depend on
- **Quick feedback**: Gets deployable system in front of users within 2 weeks
- **Technical validation**: Proves the stack can meet requirements before building complex features

**Success means**: Real users can register, log in securely, and access the system via automated deployments with no critical bugs.

---

## Scope

### What IS Included in Phase 1

#### 1. Project Infrastructure
- Next.js application with TypeScript and strict type checking
- Vercel deployment pipeline with preview and production environments
- GitHub repository with CI/CD workflows
- Environment variable management (staging, production)
- Project documentation and setup instructions

#### 2. Authentication System
- User registration with email/password
- Login/logout functionality
- Session management via NextAuth.js
- Password reset flow (email-based)
- Account activation workflow
- Basic profile management (update email, password)

#### 3. Database Foundation
- Neon PostgreSQL database (managed hosting)
- Core schema for users and authentication
- Migration system (Prisma or similar)
- Connection pooling and error handling
- Seed data for development and testing

#### 4. Core Data Entities (Minimal Schema)
**Users table:**
- `id` (UUID, primary key)
- `email` (unique, indexed)
- `passwordHash` (bcrypt)
- `firstName`, `lastName`
- `role` (enum: student, teacher, admin, sysadmin)
- `status` (enum: pending, active, suspended, deleted)
- `createdAt`, `updatedAt`
- `lastLoginAt`

**Sessions table:**
- Standard NextAuth.js session schema
- Session expiry and refresh logic

**Audit log foundation:**
- Basic structure for tracking auth events
- Login, logout, registration, password reset events

#### 5. UI Framework
- Tailwind CSS for styling
- Component library foundation (buttons, forms, cards, layout)
- Responsive breakpoints (mobile, tablet, desktop)
- Color system and typography scale
- Navigation shell (header, footer, authenticated vs. public)
- Loading states and error boundaries

#### 6. Public Pages
- Marketing homepage (simple, informative)
- About page
- Contact page
- Pricing overview (static, no checkout)
- Terms and Privacy Policy links (placeholder pages)

#### 7. Authenticated Pages (Shell Only)
- Student Dashboard (placeholder: "Welcome, [Name]")
- Teacher Dashboard (placeholder: "Welcome, Teacher [Name]")
- Admin Dashboard (placeholder: "Welcome, Admin [Name]")
- Basic navigation between role-specific areas

#### 8. API Endpoints (Minimal)
- `POST /api/auth/register` - Create new user
- `POST /api/auth/login` - Authenticate user
- `POST /api/auth/logout` - End session
- `POST /api/auth/reset-password` - Initiate password reset
- `POST /api/auth/confirm-reset` - Complete password reset
- `GET /api/auth/session` - Get current session
- `PUT /api/user/profile` - Update user profile

#### 9. Testing Foundation
- Jest and React Testing Library setup
- Unit tests for authentication functions
- Integration tests for registration/login flows
- Test database setup and teardown scripts

#### 10. Development Tools
- ESLint and Prettier configuration
- TypeScript strict mode
- VS Code recommended extensions and settings
- Local development environment documentation

---

### What is EXCLUDED from Phase 1

**NOT in Phase 1** (deferred to later phases):

- ❌ Game integration or game player
- ❌ Student progress tracking
- ❌ Assignment creation or delivery
- ❌ Teacher reporting or analytics
- ❌ Payment processing or subscription management
- ❌ Billing provider integration
- ❌ CSV import/export tools
- ❌ Mass operations or bulk actions
- ❌ Messaging system (student-teacher communication)
- ❌ Notifications (email, push, in-app)
- ❌ Badges, points, leaderboards
- ❌ Search functionality
- ❌ Content management (games, sequences)
- ❌ MIDI keyboard support
- ❌ Mobile app (PWA features deferred)
- ❌ SSO or third-party authentication
- ❌ Advanced admin tools (impersonation, feature flags)
- ❌ Observability beyond basic logs

**Rationale:** Phase 1 focuses exclusively on foundational infrastructure. User-facing features that depend on authentication, database, and deployment are built in Phases 2-4 after the foundation is validated.

---

## Data Model

### Phase 1 Core Schema

See [data-model-erd.md](../18-architecture/data-model-erd.md) for complete schema. Phase 1 implements only the minimum entities required for authentication:

#### Users Table
```
users
  - id: uuid (PK)
  - email: varchar(255) unique, indexed
  - passwordHash: varchar(255)
  - firstName: varchar(100)
  - lastName: varchar(100)
  - role: enum(student, teacher, teacher_admin, subscriber_admin, system_admin)
  - status: enum(pending_activation, active, suspended, deleted)
  - activationToken: varchar(255), nullable
  - activationTokenExpiry: timestamp, nullable
  - resetToken: varchar(255), nullable
  - resetTokenExpiry: timestamp, nullable
  - agreedToTermsAt: timestamp, nullable
  - createdAt: timestamp
  - updatedAt: timestamp
  - lastLoginAt: timestamp, nullable
```

#### Sessions Table (NextAuth.js Standard)
```
sessions
  - id: uuid (PK)
  - userId: uuid (FK -> users.id)
  - sessionToken: varchar(255) unique
  - expires: timestamp
```

#### Accounts Table (NextAuth.js, Future SSO)
```
accounts
  - id: uuid (PK)
  - userId: uuid (FK -> users.id)
  - type: varchar(50)
  - provider: varchar(50)
  - providerAccountId: varchar(255)
  - refresh_token: text, nullable
  - access_token: text, nullable
  - expires_at: integer, nullable
  - token_type: varchar(50), nullable
  - scope: varchar(255), nullable
  - id_token: text, nullable
```

#### Audit Logs (Minimal)
```
audit_logs
  - id: uuid (PK)
  - userId: uuid (FK -> users.id), nullable
  - action: varchar(100) (e.g., "user.login", "user.register")
  - resourceType: varchar(50), nullable
  - resourceId: uuid, nullable
  - metadata: jsonb, nullable
  - ipAddress: inet, nullable
  - userAgent: text, nullable
  - createdAt: timestamp
```

**Indexes:**
- `users.email` (unique)
- `users.status` (for filtering active users)
- `sessions.sessionToken` (unique)
- `sessions.userId` (for user session lookup)
- `audit_logs.userId, createdAt` (for user history queries)
- `audit_logs.action, createdAt` (for action-based queries)

**Foreign Key Constraints:**
- `sessions.userId` references `users.id` ON DELETE CASCADE
- `accounts.userId` references `users.id` ON DELETE CASCADE
- `audit_logs.userId` references `users.id` ON DELETE SET NULL

---

## Behavior and Rules

### Registration Flow
1. User navigates to `/register`
2. Provides: `email`, `password`, `confirmPassword`, `firstName`, `lastName`
3. System validates:
   - Email format is valid
   - Email is not already registered
   - Password meets requirements (min 8 chars, 1 uppercase, 1 lowercase, 1 number)
   - Password and confirmPassword match
4. System creates user record with `status: pending_activation`
5. System generates `activationToken` (UUID) and sets `activationTokenExpiry` (24 hours)
6. System sends activation email (or displays token in dev mode)
7. User clicks activation link: `/auth/activate?token={activationToken}`
8. System validates token and expiry, updates `status: active`, clears token fields
9. User is redirected to login page with success message

**Error handling:**
- Email already exists → "An account with this email already exists"
- Password requirements not met → Specific guidance on requirements
- Activation token expired → "Activation link expired. Request a new one."
- Activation token invalid → "Invalid activation link"

### Login Flow
1. User navigates to `/login`
2. Provides: `email`, `password`
3. System validates:
   - User exists with provided email
   - User status is `active` (not `pending_activation`, `suspended`, or `deleted`)
   - Password matches stored hash (bcrypt compare)
4. System creates session via NextAuth.js
5. System updates `lastLoginAt` timestamp
6. System logs `user.login` event to audit_logs
7. User is redirected to role-appropriate dashboard:
   - `student` → `/student/dashboard`
   - `teacher` → `/teacher/dashboard`
   - `teacher_admin` → `/admin/dashboard`
   - `subscriber_admin` → `/admin/dashboard`
   - `system_admin` → `/sysadmin/dashboard`

**Error handling:**
- Invalid credentials → "Invalid email or password" (generic message for security)
- Account pending activation → "Please activate your account. Check your email for the activation link."
- Account suspended → "Your account has been suspended. Contact support for assistance."
- Account deleted → "Invalid email or password" (same as invalid credentials)

### Password Reset Flow
1. User navigates to `/auth/forgot-password`
2. Provides: `email`
3. System validates email exists (but doesn't reveal if it doesn't for security)
4. If user exists:
   - Generate `resetToken` (UUID) and set `resetTokenExpiry` (1 hour)
   - Send password reset email (or display token in dev mode)
5. Always display: "If an account exists with that email, you will receive reset instructions."
6. User clicks reset link: `/auth/reset-password?token={resetToken}`
7. User provides: `newPassword`, `confirmPassword`
8. System validates token, expiry, and password requirements
9. System updates `passwordHash`, clears reset token fields
10. System logs `user.password_reset` event
11. User is redirected to login with success message

### Logout Flow
1. User clicks logout button
2. System destroys session via NextAuth.js
3. System logs `user.logout` event
4. User is redirected to public homepage or login page

### Session Management
- **Session duration:** 7 days (sliding window)
- **Session refresh:** Automatic on activity within last 24 hours
- **Session expiry:** User must log in again after 7 days of inactivity
- **Concurrent sessions:** Allowed (user can be logged in on multiple devices)
- **Logout all devices:** Not in Phase 1 (future feature)

### Account Status Transitions
- `pending_activation` → `active` (via activation link)
- `active` → `suspended` (admin action, Phase 2+)
- `suspended` → `active` (admin action, Phase 2+)
- `active` → `deleted` (admin action or self-service, Phase 2+)
- `deleted` → (immutable, cannot be reversed)

**Business Rule:** Deleted accounts retain records for audit purposes but cannot log in. Email addresses from deleted accounts can be reused after 90 days (enforced in Phase 3+).

### Validation Rules

**Email:**
- Valid email format (RFC 5322 compliant)
- Max length: 255 characters
- Case-insensitive (stored lowercase)
- Unique across system

**Password:**
- Min length: 8 characters
- Max length: 72 characters (bcrypt limit)
- Must contain: 1 uppercase, 1 lowercase, 1 number
- No dictionary word check in Phase 1 (future enhancement)

**Name fields:**
- Min length: 1 character
- Max length: 100 characters
- Allow unicode characters (international names)
- No profanity filter in Phase 1 (future enhancement)

---

## UX Requirements

### Page Layouts

**Public pages:** 
- Header with logo, navigation (Home, About, Pricing, Login, Register)
- Footer with links (Privacy, Terms, Contact)
- Responsive: hamburger menu on mobile (<768px)

**Authenticated pages:**
- Header with logo, navigation (role-specific), user menu (Profile, Logout)
- Sidebar navigation for multi-section areas (admin, teacher)
- Main content area with consistent padding and max-width
- Footer (minimal, copyright only)

### Key States

**Registration form:**
- Default: All fields empty, submit disabled until valid
- Loading: Button shows spinner, fields disabled during submission
- Success: Redirect to confirmation page with "Check your email" message
- Error: Inline field errors, form-level error message

**Login form:**
- Default: Email and password fields, "Forgot password?" link
- Loading: Button shows spinner during authentication
- Success: Immediate redirect to role-specific dashboard
- Error: Form-level error message (no field-specific errors for security)

**Activation page:**
- Loading: "Activating your account..." spinner
- Success: "Account activated! You can now log in." + login button
- Error: "Activation link expired/invalid" + "Request new link" button

**Password reset:**
- Request form: Email input + submit
- Confirmation: "Check your email" message
- Reset form: New password + confirm password + submit
- Success: "Password updated! You can now log in." + login button
- Error: "Reset link expired/invalid" + "Request new link" button

### Accessibility Notes

See [a11y-standards.md](../01-ux-design-system/a11y-standards.md) for complete requirements. Phase 1 specific:

- ✅ All forms have proper `<label>` elements with `for` attributes
- ✅ Form errors announced to screen readers via `aria-live="polite"`
- ✅ Focus states visible on all interactive elements
- ✅ Keyboard navigation works for all actions (no mouse required)
- ✅ Color contrast meets WCAG AA (4.5:1 for text)
- ✅ Skip to main content link for keyboard users
- ✅ Page titles describe current page/section
- ⚠️ Full WCAG 2.1 Level AA audit deferred to Phase 3

### Mobile Considerations

See [responsive-breakpoints.md](../01-ux-design-system/responsive-breakpoints.md) for detailed specifications. Phase 1 targets:

- **Mobile-first design:** All layouts designed for 375px width first
- **Breakpoints:** 375px (mobile), 768px (tablet), 1024px (desktop)
- **Touch targets:** Min 44x44px for all interactive elements
- **Form inputs:** Native mobile keyboard types (email, password, text)
- **No horizontal scroll:** Content reflows at all breakpoints

---

## Telemetry

See [event-model.md](../15-analytics-and-reporting/event-model.md) for complete event specifications. Phase 1 implements minimal authentication telemetry:

### Events Tracked

#### `user.registered`
**When:** User completes registration form successfully  
**Where:** Server-side after user record created  
**Properties:**
- `userId`: UUID of new user
- `email`: User email (hashed for privacy)
- `role`: User role assigned
- `source`: Registration source (web, mobile, referral code - future)
- `timestamp`: ISO 8601 timestamp

#### `user.activated`
**When:** User clicks activation link and account becomes active  
**Where:** Server-side after status update  
**Properties:**
- `userId`: UUID
- `activationLatency`: Time between registration and activation (seconds)
- `timestamp`: ISO 8601 timestamp

#### `user.login`
**When:** User successfully authenticates  
**Where:** Server-side after session created  
**Properties:**
- `userId`: UUID
- `role`: User role
- `loginMethod`: "email_password" (SSO types in future)
- `deviceType`: "desktop" | "mobile" | "tablet"
- `userAgent`: Browser/device string
- `ipAddress`: User IP (hashed for privacy)
- `timestamp`: ISO 8601 timestamp

#### `user.login.failed`
**When:** Login attempt fails (invalid credentials, suspended account, etc.)  
**Where:** Server-side during authentication  
**Properties:**
- `email`: Email attempted (hashed for privacy)
- `failureReason`: "invalid_credentials" | "account_suspended" | "pending_activation"
- `ipAddress`: User IP (hashed for privacy)
- `timestamp`: ISO 8601 timestamp

#### `user.logout`
**When:** User explicitly logs out  
**Where:** Server-side when session destroyed  
**Properties:**
- `userId`: UUID
- `sessionDuration`: Time between login and logout (seconds)
- `timestamp`: ISO 8601 timestamp

#### `user.password_reset_requested`
**When:** User submits forgot password form  
**Where:** Server-side  
**Properties:**
- `email`: Email provided (hashed for privacy)
- `userExists`: Boolean (for internal analytics only, not exposed to user)
- `timestamp`: ISO 8601 timestamp

#### `user.password_reset_completed`
**When:** User successfully resets password  
**Where:** Server-side after password updated  
**Properties:**
- `userId`: UUID
- `resetLatency`: Time between request and completion (seconds)
- `timestamp`: ISO 8601 timestamp

### Implementation

**Phase 1 telemetry storage:**
- Events written to `audit_logs` table
- Basic console logging in development
- No external analytics service in Phase 1 (deferred to Phase 3)

**Future enhancements:**
- Integration with analytics platform (PostHog, Mixpanel, etc.)
- Real-time dashboards
- Anomaly detection (unusual login patterns, brute force attempts)

---

## Permissions

See [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) for complete permission specifications. Phase 1 implements minimal role-based access:

### Roles in Phase 1

- **Public** (unauthenticated): Can access marketing pages, register, log in
- **Student**: Can access student dashboard placeholder
- **Teacher**: Can access teacher dashboard placeholder
- **Teacher Admin**: Can access admin dashboard placeholder (same as subscriber_admin in Phase 1)
- **Subscriber Admin**: Can access admin dashboard placeholder
- **System Admin**: Can access sysadmin dashboard placeholder

### Permission Checks

**Route protection:**
- Public routes: `/`, `/about`, `/pricing`, `/contact`, `/login`, `/register`, `/auth/*`
- Authenticated routes: `/student/*`, `/teacher/*`, `/admin/*`, `/sysadmin/*`
- Role-specific routes enforced via middleware (redirect if wrong role)

**API endpoints:**
- `/api/auth/*` - Public (rate limited)
- `/api/user/profile` - Authenticated user can read/update own profile only

**Phase 1 Permission Logic:**
```typescript
// Simplified - detailed implementation in code
canAccessRoute(user, route):
  if route.isPublic: return true
  if !user: return false
  if route.requiredRole == user.role: return true
  if user.role == 'system_admin': return true // sysadmin can access all
  return false
```

**NOT in Phase 1:**
- ❌ Impersonation (deferred to Phase 2+)
- ❌ Fine-grained resource permissions (e.g., "can edit assignment X")
- ❌ Permission groups or custom roles
- ❌ Cross-role access (e.g., teacher viewing student's assignment)

---

## Supporting Documents Referenced

This Phase 1 scope specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role definitions (student, teacher, admin, system_admin) and authentication requirements
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform authentication flows and user account structure from legacy system

## Dependencies

### Internal Dependencies

**Foundational documents:**
- [initial-project-phases.md](../00-foundations/initial-project-phases.md) - Strategic context and phasing philosophy
- [product-vision.md](../00-foundations/product-vision.md) - Overall product goals and principles
- [techstack.md](../00-foundations/techstack.md) - Technology choices and rationale

**Design system:**
- [responsive-breakpoints.md](../01-ux-design-system/responsive-breakpoints.md) - Screen size specifications
- [a11y-standards.md](../01-ux-design-system/a11y-standards.md) - Accessibility requirements
- [components-overview.md](../01-ux-design-system/components-overview.md) - Reusable component library

**Architecture:**
- [system-overview.md](../18-architecture/system-overview.md) - High-level architecture
- [data-model-erd.md](../18-architecture/data-model-erd.md) - Complete database schema
- [api-design-principles.md](../18-architecture/api-design-principles.md) - API conventions
- [security-privacy.md](../18-architecture/security-privacy.md) - Security requirements
- [config-and-secrets.md](../18-architecture/config-and-secrets.md) - Environment configuration

**Operations:**
- [test-strategy.md](../19-quality-and-operations/test-strategy.md) - Testing approach
- [release-management.md](../19-quality-and-operations/release-management.md) - Deployment process

### External Dependencies

**Required services:**
- **Next.js** - Web application framework (v14+)
- **Vercel** - Hosting and deployment platform
- **Neon PostgreSQL** - Managed database (Serverless Postgres)
- **NextAuth.js** - Authentication library (v5+)
- **Tailwind CSS** - Styling framework (v3+)
- **TypeScript** - Type safety (v5+)
- **bcrypt** - Password hashing
- **Zod** - Schema validation

**Development tools:**
- **GitHub** - Version control and CI/CD
- **ESLint** - Code linting
- **Prettier** - Code formatting
- **Jest** - Unit testing
- **React Testing Library** - Component testing

**NOT in Phase 1** (deferred):
- ❌ Email service provider (Resend, SendGrid) - Use console logging in dev
- ❌ CDN for static assets (Cloudflare R2) - Use Vercel's built-in CDN
- ❌ External analytics platform
- ❌ Error monitoring (Sentry)
- ❌ Uptime monitoring

### Phase 2 Blockers

Phase 2 (First Game Integration) cannot begin until Phase 1 delivers:
- ✅ Working authentication system (registration, login, sessions)
- ✅ Database with migrations
- ✅ Deployed application accessible via URL
- ✅ Role-based routing (student/teacher/admin dashboards)
- ✅ UI component library
- ✅ Testing infrastructure

**Critical path:** Database schema must support future expansion (games, progress, assignments). Phase 1 creates foundation tables only, but schema design should account for Phase 2+ entities.

---

## Open Questions

### To Resolve Before Build

**1. Email in Development**
- **Question:** How do we handle email verification in local development without an email service?
- **Options:** 
  - A) Console log activation/reset links
  - B) Use a dev email service (Ethereal, MailHog)
  - C) Skip verification in dev mode
- **Recommendation:** Option A (console logging) for Phase 1 simplicity
- **Owner:** Tech Lead
- **Deadline:** Before Phase 1 kickoff

**2. Session Storage**
- **Question:** Where do we store NextAuth.js sessions - database or JWT?
- **Options:**
  - A) Database sessions (more secure, allows session revocation)
  - B) JWT sessions (faster, stateless, but harder to revoke)
- **Recommendation:** Option A (database sessions) for security and future admin features
- **Owner:** Tech Lead
- **Deadline:** Before Phase 1 kickoff

**3. Rate Limiting**
- **Question:** Do we implement rate limiting on auth endpoints in Phase 1?
- **Options:**
  - A) Yes, basic rate limiting (e.g., 5 attempts per 15 min)
  - B) No, defer to Phase 2+
- **Recommendation:** Option A (basic rate limiting) to prevent brute force attacks
- **Owner:** Tech Lead
- **Dependencies:** [rate-limiting-and-abuse.md](../18-architecture/rate-limiting-and-abuse.md)
- **Deadline:** Before Phase 1 kickoff

**4. User Role Assignment**
- **Question:** How is the initial user role assigned during registration?
- **Options:**
  - A) User selects role during registration ("I am a: Student / Teacher / Admin")
  - B) All users default to "student" role
  - C) First user is sysadmin, subsequent users default to student
- **Recommendation:** Option A (user selects role) for Phase 1 simplicity
- **Owner:** Product Lead
- **Deadline:** Before Phase 1 UI work begins

**5. Multi-Role Accounts**
- **Question:** Can a single user have multiple roles (e.g., teacher who is also a student)?
- **Options:**
  - A) Single role per account (create multiple accounts if needed)
  - B) Multiple roles per account (user switches context)
- **Recommendation:** Option A (single role) for Phase 1 simplicity. Revisit in Phase 3+ if user feedback demands it.
- **Owner:** Product Lead
- **Deadline:** Before Phase 1 kickoff

**6. Password Strength Requirements**
- **Question:** Are the proposed password requirements (8 chars, 1 upper, 1 lower, 1 number) sufficient?
- **Options:**
  - A) As proposed (standard requirements)
  - B) Add special character requirement
  - C) Use passphrase approach (longer but simpler)
  - D) Integrate with haveibeenpwned.com API to check against breached passwords
- **Recommendation:** Option A for Phase 1. Add option D in Phase 2.
- **Owner:** Security Lead
- **Deadline:** Before Phase 1 kickoff

**7. Account Deletion**
- **Question:** Do we implement self-service account deletion in Phase 1?
- **Options:**
  - A) Yes, users can delete their own account
  - B) No, contact support to delete (manual process)
- **Recommendation:** Option B (manual via support) for Phase 1. Add self-service in Phase 3+ with proper safeguards.
- **Owner:** Product Lead
- **Deadline:** Before Phase 1 kickoff

### Future Considerations (Not Blocking Phase 1)

**8. SSO Integration**
- Single Sign-On via Google, Microsoft, Clever
- **Timing:** Phase 3+
- **Reference:** [sso-integration.md](../17-integrations/sso-integration.md)

**9. 2FA/MFA**
- Two-factor authentication for enhanced security
- **Timing:** Phase 3+
- **Priority:** Required for school/district customers (FERPA compliance)

**10. Magic Link Authentication**
- Passwordless login via email link
- **Timing:** Phase 4+
- **Rationale:** Simplifies student login, reduces password reset requests

**11. Account Merge**
- Merge duplicate accounts (e.g., user created two accounts with different emails)
- **Timing:** Phase 4+
- **Complexity:** High (data migration, history merging)

**12. Bulk User Import**
- CSV import for schools to add many users at once
- **Timing:** Phase 2 (for admins)
- **Reference:** [csv-spec-students.md](../20-imports-and-automation/csv-spec-students.md)

---

## Phase 1 Timeline

**Total duration:** 2 weeks (10 business days)

### Week 1: Infrastructure and Database
- **Days 1-2:** Project setup, Next.js initialization, Vercel deployment
- **Days 3-5:** Database schema, migrations, authentication backend

### Week 2: UI and Testing
- **Days 6-7:** UI components, registration/login pages
- **Days 8-9:** Testing, bug fixes, polish
- **Day 10:** Final validation, deployment, documentation

**Milestone:** By end of Week 2, real users can register, log in, and access role-appropriate dashboard placeholders.

---

## Success Criteria

Phase 1 is complete when:

✅ **Technical:**
- Application deploys automatically to Vercel on `main` branch push
- Database migrations run successfully in production
- All tests pass (unit and integration)
- No critical security vulnerabilities (npm audit, Snyk scan)

✅ **Functional:**
- Users can register with email/password
- Users receive activation email (or see link in console for dev)
- Users can activate account via link
- Users can log in with valid credentials
- Users are redirected to correct dashboard based on role
- Users can log out
- Users can reset password via email flow
- Sessions persist for 7 days
- Sessions expire after 7 days of inactivity

✅ **Quality:**
- All forms have proper validation and error messages
- All pages are responsive (mobile, tablet, desktop)
- Basic accessibility requirements met (keyboard nav, focus states, labels)
- No console errors or warnings
- Page load time < 2 seconds on 3G connection

✅ **Documentation:**
- Setup instructions in README.md
- Environment variables documented
- Database schema documented
- API endpoints documented
- Deployment process documented

**Phase 1 does NOT require:**
- ❌ Any game functionality
- ❌ Student progress tracking
- ❌ Teacher features beyond dashboard placeholder
- ❌ Payment or billing
- ❌ Email service integration (console logging acceptable)
- ❌ Full WCAG 2.1 AA compliance (basic a11y only)

---

## Next Phase

Upon successful completion of Phase 1, proceed to:
- **[phase-2-scope.md](./phase-2-scope.md)** - First Game Integration

Phase 2 will build on this foundation to deliver:
- Game container system (Phaser 3 or HTML5 iframe)
- Integration with one existing game
- Student progress tracking and scoring
- File storage for game assets (Cloudflare R2)
- Enhanced student dashboard with progress visualization

---

## Related Documentation

**Foundational context:**
- [initial-project-phases.md](../00-foundations/initial-project-phases.md) - Strategic overview and phasing philosophy
- [product-vision.md](../00-foundations/product-vision.md) - Overall product vision and goals
- [techstack.md](../00-foundations/techstack.md) - Technology stack and rationale

**Architecture and design:**
- [system-overview.md](../18-architecture/system-overview.md) - High-level system architecture
- [data-model-erd.md](../18-architecture/data-model-erd.md) - Complete database schema (all phases)
- [api-design-principles.md](../18-architecture/api-design-principles.md) - API design conventions
- [security-privacy.md](../18-architecture/security-privacy.md) - Security requirements

**User experience:**
- [responsive-breakpoints.md](../01-ux-design-system/responsive-breakpoints.md) - Responsive design specifications
- [a11y-standards.md](../01-ux-design-system/a11y-standards.md) - Accessibility standards
- [components-overview.md](../01-ux-design-system/components-overview.md) - UI component library

**Roles and permissions:**
- [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) - Complete role-permission matrix
- [session-and-auth-states.md](../02-roles-and-permissions/session-and-auth-states.md) - Authentication states

**Operations:**
- [test-strategy.md](../19-quality-and-operations/test-strategy.md) - Testing approach
- [release-management.md](../19-quality-and-operations/release-management.md) - Deployment process
- [observability.md](../19-quality-and-operations/observability.md) - Monitoring and logging

---

**Document Status:** ✅ Complete and ready for Phase 1 implementation  
**Last Updated:** 2025-10-13  
**Owner:** Product & Engineering  
**Reviewers:** Tech Lead, UX Lead, Product Manager
