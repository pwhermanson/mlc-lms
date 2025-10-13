# System Overview

## Purpose

This document provides a high-level architectural view of the MusicLearningCommunity (MLC) LMS platform. It describes the major system components, their interactions, deployment architecture, and key technical decisions that enable the platform to deliver game-based music education at scale.

## Scope

This overview covers:
- Overall system architecture and component structure
- Major subsystems and their responsibilities
- Integration points and data flows
- Deployment model and infrastructure approach
- Key architectural qualities (scalability, security, performance)
- Cross-cutting concerns (authentication, caching, logging)

This document does **not** duplicate:
- Detailed technology choices (see [Technology Stack](../00-foundations/techstack.md))
- Product capabilities and user flows (see [Product Vision](../00-foundations/product-vision.md) and [Information Architecture](../00-foundations/information-architecture.md))
- Specific API endpoints (see [REST Endpoints](rest-endpoints.md))
- Database schema details (see [Data Model ERD](data-model-erd.md))

---

## System Architecture

### High-Level Architecture

The MLC LMS follows a modern, serverless-first architecture built on a monolithic Next.js application with clear internal boundaries. This approach balances developer velocity with architectural clarity while maintaining a path to service extraction when needed.

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                                │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │   Browser    │  │    Tablet    │  │    Mobile    │              │
│  │  (Desktop)   │  │              │  │   Browser    │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│         │                  │                  │                      │
│         └──────────────────┴──────────────────┘                     │
│                            │                                         │
└────────────────────────────┼─────────────────────────────────────────┘
                             │
                             │ HTTPS
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     CLOUDFLARE CDN & SECURITY                        │
│  - Global CDN & edge caching                                         │
│  - DDoS protection & WAF                                             │
│  - SSL/TLS termination                                               │
└────────────────────────────┬─────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    VERCEL HOSTING PLATFORM                           │
│  ┌───────────────────────────────────────────────────────┐          │
│  │             NEXT.JS APPLICATION                        │          │
│  │  ┌─────────────────────────────────────────────────┐  │          │
│  │  │         FRONTEND (React/TypeScript)             │  │          │
│  │  │  - Server-side rendering (SSR)                  │  │          │
│  │  │  - Static generation (SSG)                      │  │          │
│  │  │  - Client-side hydration                        │  │          │
│  │  │  - shadcn/ui components                         │  │          │
│  │  │  - Game player shell (Phaser wrapper - future) │  │          │
│  │  └─────────────────────────────────────────────────┘  │          │
│  │  ┌─────────────────────────────────────────────────┐  │          │
│  │  │         BACKEND (API Routes)                    │  │          │
│  │  │  - RESTful endpoints                            │  │          │
│  │  │  - Business logic layer                         │  │          │
│  │  │  - NextAuth.js authentication                   │  │          │
│  │  │  - Middleware & request validation              │  │          │
│  │  └─────────────────────────────────────────────────┘  │          │
│  └───────────────────────────────────────────────────────┘          │
│             │                          │                             │
└─────────────┼──────────────────────────┼─────────────────────────────┘
              │                          │
              ▼                          ▼
┌──────────────────────────┐   ┌──────────────────────────┐
│   NEON POSTGRESQL        │   │   CLOUDFLARE R2          │
│   (Serverless Database)  │   │   (Object Storage)       │
│                          │   │                          │
│  - User data             │   │  - Game assets           │
│  - Progress tracking     │   │  - Audio files           │
│  - Assignment history    │   │  - Graphics & media      │
│  - Scores & analytics    │   │  - User uploads          │
│  - Subscription data     │   │  - Content backups       │
│  - Connection pooling    │   │                          │
│  - Automatic backups     │   │  - S3-compatible API     │
└──────────────────────────┘   └──────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│                    EXTERNAL INTEGRATIONS                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │  Braintree   │  │    Email     │  │  Analytics   │              │
│  │  Payments    │  │   Provider   │  │  (Future)    │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
```

### Architectural Style

**Modular Monolith with Serverless Deployment**

The system is architected as a well-structured monolithic application with clear internal boundaries that could evolve into microservices if needed. This approach provides:

- **Fast iteration**: Single codebase with shared types and utilities
- **Deployment simplicity**: One application to deploy and monitor
- **Cost efficiency**: Serverless functions scale to zero, no idle infrastructure costs
- **Clear boundaries**: Internal module structure prepares for future service extraction
- **Type safety**: End-to-end TypeScript ensures consistency

---

## Major System Components

### 1. Frontend Layer (React/Next.js)

**Responsibility**: Deliver role-specific user interfaces with server-side rendering for performance and SEO.

**Key Subsystems**:
- **Student Portal**: Dashboard, assignment play, game library, badges, leaderboards
- **Teacher Portal**: Assignment builder, progress reports, class management, messaging
- **Admin Portal**: Member management, bulk operations, billing, CSV imports, reports
- **System Admin Portal**: Global user index, audit logs, feature flags, content tools
- **Game Player Shell**: Unified interface for HTML5 games with consistent UI/UX (Phaser wrapper - future enhancement)

**Technology**: React 18+, Next.js 14+, TypeScript, Tailwind CSS, shadcn/ui components

**Rendering Strategy**:
- **SSR (Server-Side Rendering)**: Dynamic pages requiring authentication and personalization
- **SSG (Static Site Generation)**: Marketing pages, documentation, public content
- **CSR (Client-Side Rendering)**: Interactive game play, real-time updates

### 2. Backend Layer (Next.js API Routes)

**Responsibility**: Handle business logic, data access, authentication, and external integrations.

**Key Subsystems**:

**Authentication & Authorization** (`/api/auth/*`)
- NextAuth.js session management
- Role-based access control (RBAC)
- JWT token validation
- Password hashing and security

**User Management** (`/api/users/*`, `/api/members/*`, `/api/students/*`, `/api/teachers/*`)
- CRUD operations for all user types
- Role assignment and hierarchy management
- Bulk user operations (CSV import/export)
- Impersonation and support access (see [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md))

**Content Management** (`/api/games/*`, `/api/sequences/*`, `/api/assignments/*`)
- Games registry and metadata
- Sequence libraries and curriculum alignment
- Assignment generation and tracking
- Content versioning (see [Content Versioning](../07-content-architecture/content-versioning.md))

**Learning Engine** (`/api/play/*`, `/api/progress/*`, `/api/scores/*`)
- Assignment generation from sequences
- Game stage progression (Learn → Play → Quiz → Challenge → Review)
- Score recording and validation
- Progress tracking and retention logic (see [Retention Review Logic](../10-assignments-engine/retention-review-logic.md))

**Reporting & Analytics** (`/api/reports/*`, `/api/analytics/*`)
- Student progress aggregation
- Teacher and admin reports
- System admin dashboards
- Event tracking (see [Event Model](../15-analytics-and-reporting/event-model.md))

**Billing & Subscriptions** (`/api/billing/*`, `/api/subscriptions/*`)
- Braintree payment integration
- Seat management and pricing calculations (see [Seat Management](../14-payments-and-subscriptions/seat-management.md))
- Subscription lifecycle (trials, renewals, pauses)
- Invoice and statement generation
- Promo and sponsor codes (see [Promo Codes](../14-payments-and-subscriptions/promo-codes.md) and [Sponsor Codes](../14-payments-and-subscriptions/sponsor-codes.md))

**Messaging & Notifications** (`/api/messages/*`, `/api/notifications/*`)
- In-app notifications
- Email template rendering and delivery (see [Email Provider](../17-integrations/email-provider.md))
- System announcements (see [System Announcements](../13-messaging-and-notifications/system-announcements.md))
- Teacher-student messaging

**Search & Discovery** (`/api/search/*`)
- Full-text search across games, sequences, and content
- Faceted filtering (see [Filters and Facets](../11-search-and-discovery/filters-and-facets.md))
- Teacher quick-find tools

### 3. Data Layer (Neon PostgreSQL)

**Responsibility**: Persistent storage for all application data with ACID guarantees.

**Key Data Domains**:
- **Users & Permissions**: Authentication, roles, session data
- **Members & Subscriptions**: Organizations, billing, seat allocations
- **Students & Teachers**: User profiles, relationships, class assignments
- **Games & Content**: Game definitions, metadata, taxonomies (see [Game Model](../08-games-registry-and-authoring/game-model.md))
- **Sequences & Curriculum**: Learning paths, method alignments (see [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))
- **Assignments & Progress**: Assignment instances, completion tracking
- **Scores & History**: Per-stage scores, timestamps, review schedules
- **Events & Analytics**: Audit logs, telemetry, usage metrics

**Database Features**:
- Serverless connection pooling for scalability
- Automatic backups and point-in-time recovery
- Read replicas for heavy read workloads (future)
- Full PostgreSQL feature set (JSON, arrays, full-text search)

### 4. Storage Layer (Cloudflare R2)

**Responsibility**: Cost-effective object storage for static assets and user uploads.

**Stored Content**:
- **Game Assets**: HTML5 game files, images, sprites, sounds
- **Audio Files**: Music clips, aural training samples, MIDI playback
- **Graphics & Media**: UI elements, badges, certificates, animations
- **User Uploads**: CSV imports, profile images, teacher resources
- **Content Backups**: Versioned game exports, sequence snapshots

**Key Benefits**:
- S3-compatible API (easy migration path)
- No egress fees (unlike AWS S3)
- Global CDN distribution via Cloudflare
- Integrated with Cloudflare security and caching

### 5. CDN & Security Layer (Cloudflare)

**Responsibility**: Global content delivery, security, and edge caching.

**Capabilities**:
- **CDN**: Cache static assets at edge locations worldwide
- **DDoS Protection**: Automatic mitigation of volumetric attacks
- **Web Application Firewall (WAF)**: Block malicious requests and bots
- **SSL/TLS**: Encrypted connections with automatic certificate management
- **Rate Limiting**: Protect APIs from abuse (see [Rate Limiting and Abuse](rate-limiting-and-abuse.md))
- **Bot Management**: Distinguish legitimate users from bots

### 6. Hosting & Deployment (Vercel)

**Responsibility**: Serverless application hosting with automatic deployments and global edge network.

**Features**:
- **Automatic Deployments**: Push to GitHub triggers builds and deploys
- **Preview Environments**: Every pull request gets a unique URL
- **Edge Functions**: Run serverless functions close to users
- **Analytics**: Real-time performance and usage metrics
- **Monitoring**: Error tracking and performance insights

---

## Data Flow Patterns

### 1. Student Assignment Play Flow

```
Student → Next.js Frontend → SSR fetch → API Route (/api/assignments/next)
    ↓
  Query current student assignments from PostgreSQL
    ↓
  Return next game/stage in sequence
    ↓
  Frontend loads game from R2/CDN
    ↓
  Student plays game
    ↓
  On completion → POST /api/scores → Record score in PostgreSQL
    ↓
  Trigger event telemetry → Event Model table
    ↓
  Check for badge/streak updates → Update student profile
    ↓
  Return to dashboard with updated progress
```

### 2. Teacher Assignment Creation Flow

```
Teacher → Assignment Builder UI → Select sequence/games
    ↓
  POST /api/assignments → API Route validates input
    ↓
  Create assignment record in PostgreSQL
    ↓
  Link assignment to selected students
    ↓
  Trigger notification → Email + In-app notification
    ↓
  Return assignment ID and confirmation
```

### 3. Admin Bulk CSV Import Flow

```
Admin → Upload CSV file → POST /api/bulk/students
    ↓
  Upload file to R2 storage
    ↓
  Enqueue background job (see [Background Jobs](background-jobs.md))
    ↓
  Parse CSV and validate data
    ↓
  Batch insert students to PostgreSQL (transaction)
    ↓
  Send completion notification
    ↓
  Return import summary with errors (if any)
```

### 4. Billing Webhook Flow

```
Braintree → Webhook /api/webhooks/braintree → Verify signature
    ↓
  Parse event (subscription_charged, subscription_canceled, etc.)
    ↓
  Update subscription status in PostgreSQL
    ↓
  Adjust seat counts if needed
    ↓
  Trigger email notification to subscriber
    ↓
  Log audit event
    ↓
  Return 200 OK
```

---

## Deployment Architecture

### Infrastructure Model

**Serverless-First with Managed Services**

All infrastructure is hosted on managed platforms to minimize operational overhead and maximize developer productivity. This approach allows the small team to focus on product development rather than server management.

### Deployment Topology

**Production Environment**:
- **Vercel**: Next.js application (automatic scaling)
- **Neon**: PostgreSQL database (serverless, auto-scales connections)
- **Cloudflare R2**: Object storage (global distribution)
- **Cloudflare CDN**: Content delivery and security
- **Braintree**: Payment processing (PCI-compliant)

**Development Environment**:
- **Local**: Docker Compose for PostgreSQL, Next.js dev server
- **Staging**: Vercel preview deployments per pull request

### Deployment Pipeline

```
Developer → Git commit → Push to GitHub
    ↓
  GitHub Actions → Run tests + linting
    ↓
  Vercel → Detect Next.js app
    ↓
  Build optimized production bundle
    ↓
  Deploy to global edge network
    ↓
  Run database migrations (if needed)
    ↓
  Smoke tests (health checks)
    ↓
  Production live 🎉
```

**Rollback Strategy**: Vercel preserves previous deployments; instant rollback via UI or CLI.

### Scaling Model

**Automatic Horizontal Scaling**:
- **Serverless Functions**: Vercel automatically scales based on request load
- **Database Connections**: Neon connection pooling handles burst traffic
- **Static Assets**: Cloudflare CDN scales to millions of requests
- **Object Storage**: R2 scales seamlessly with no capacity planning

**No Manual Scaling Required**: Infrastructure automatically adapts to load without configuration changes.

---

## Key Architectural Qualities

### 1. Scalability

**Design for Growth**:
- Serverless functions scale horizontally without configuration
- Database connection pooling prevents connection exhaustion
- CDN caching reduces origin load
- Stateless API design enables easy horizontal scaling
- Background job queues handle heavy batch operations asynchronously (see [Background Jobs](background-jobs.md))

**Current Capacity**: Supports thousands of concurrent users with room to scale to hundreds of thousands.

### 2. Security

**Defense in Depth**:
- **Authentication**: NextAuth.js with JWT tokens, bcrypt password hashing
- **Authorization**: Role-based access control (RBAC) at API and UI layers (see [Roles Matrix](../02-roles-and-permissions/roles-matrix.md))
- **Data Protection**: HTTPS everywhere, secure cookie handling, CSRF protection
- **Input Validation**: Type-safe validation at API boundaries
- **SQL Injection Prevention**: Parameterized queries via PostgreSQL client
- **XSS Protection**: React's built-in escaping + Content Security Policy (CSP)
- **DDoS Mitigation**: Cloudflare protection + rate limiting
- **Audit Logging**: All sensitive actions logged (see [Audit Logs](../06-system-admin/sysadmin-audit-logs.md))
- **Secrets Management**: Environment variables, never committed to Git (see [Config and Secrets](config-and-secrets.md))

**Compliance**: COPPA, FERPA, GDPR-ready architecture (see [Legal and Compliance](../23-legal-and-compliance/))

### 3. Performance

**Fast by Default**:
- **Server-Side Rendering (SSR)**: Fast initial page loads with pre-rendered HTML
- **Static Site Generation (SSG)**: Marketing pages served instantly from CDN
- **Code Splitting**: Next.js automatically splits bundles for faster loading
- **Image Optimization**: Next.js Image component with lazy loading and responsive sizes
- **Caching Strategy**: Multi-layer caching (CDN, API, database) (see [Caching Strategy](caching-strategy.md))
- **Database Indexing**: Optimized queries with proper indexes
- **Asset Compression**: Gzip/Brotli compression for all assets

**Performance Targets**:
- First Contentful Paint (FCP): < 1.5s
- Time to Interactive (TTI): < 3s
- API Response Time (p95): < 500ms

### 4. Reliability

**Built for Uptime**:
- **Automatic Failover**: Vercel and Neon provide multi-region redundancy
- **Database Backups**: Automatic daily backups with point-in-time recovery
- **Zero-Downtime Deploys**: Vercel atomic deployments with instant rollback
- **Health Checks**: Automated monitoring and alerting
- **Error Tracking**: Comprehensive logging and error reporting
- **Graceful Degradation**: Fallbacks for non-critical features

**Target SLA**: 99.9% uptime (< 9 hours downtime per year)

### 5. Maintainability

**Developer-Friendly Architecture**:
- **Type Safety**: End-to-end TypeScript eliminates entire classes of bugs
- **Consistent Patterns**: Standardized API design and component structure (see [API Design Principles](api-design-principles.md))
- **Code Organization**: Clear separation of concerns with modular structure
- **Testing Strategy**: Unit, integration, and E2E tests (see [Test Strategy](../19-quality-and-operations/test-strategy.md))
- **Documentation**: Inline code comments, API docs, architecture decision records (ADRs)
- **Tooling**: ESLint, Prettier, TypeScript strict mode enforce code quality

### 6. Cost Efficiency

**Start Small, Scale Smart**:
- **$0/month to start**: Free tiers for Vercel, Neon, Cloudflare R2, GitHub
- **Pay-as-you-grow**: Only pay for actual usage, no idle infrastructure costs
- **No egress fees**: R2 eliminates AWS S3's expensive egress charges
- **Serverless economics**: No cost for unused capacity

**Estimated Costs at Scale**:
- 0-1,000 users: ~$0-50/month
- 1,000-10,000 users: ~$50-300/month
- 10,000-100,000 users: ~$300-2,000/month

---

## Cross-Cutting Concerns

### Authentication & Session Management

**NextAuth.js** handles all authentication flows:
- Username/password login with bcrypt hashing
- JWT tokens for stateless sessions
- Secure cookie-based session storage
- Role claims in JWT for authorization
- Session refresh and expiration handling

See [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) for detailed flows.

### Authorization & Permissions

**Role-Based Access Control (RBAC)**:
- Roles: Student, Teacher, Teacher-Admin, Subscriber-Admin, System Admin
- Permissions checked at API middleware layer
- UI elements conditionally rendered based on role
- Fine-grained permissions per resource type (read, write, delete, override)

See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md).

### Caching Strategy

**Multi-Layer Caching**:
1. **CDN Cache (Cloudflare)**: Static assets, public pages (TTL: 1 hour - 1 year)
2. **API Cache (Next.js)**: Frequently accessed data (TTL: 5-60 minutes)
3. **Database Cache (Neon)**: Query result caching (automatic)
4. **Client Cache (React Query/SWR)**: In-memory data cache with stale-while-revalidate

See [Caching Strategy](caching-strategy.md) for detailed policies.

### Logging & Observability

**Comprehensive Monitoring**:
- **Application Logs**: Structured JSON logs via Next.js logger
- **Error Tracking**: Vercel Analytics + optional Sentry integration (future)
- **Performance Monitoring**: Vercel Analytics for Web Vitals
- **Audit Logs**: All critical actions logged to database (see [Audit Logs](../06-system-admin/sysadmin-audit-logs.md))
- **Telemetry Events**: User actions tracked via Event Model (see [Event Model](../15-analytics-and-reporting/event-model.md))

See [Observability](../19-quality-and-operations/observability.md) for monitoring and alerting strategy.

### Background Jobs

**Asynchronous Processing**:
- CSV imports and bulk operations
- Report generation
- Email delivery
- Data exports
- Scheduled maintenance tasks

**Implementation**: Initially handled via Vercel serverless functions with queue (e.g., Vercel Cron). Future: dedicated job queue (BullMQ + Redis) if needed.

See [Background Jobs](background-jobs.md) for detailed job types and processing.

### Error Handling & Resilience

**Graceful Failure**:
- **API Errors**: Consistent error response format with error codes
- **Retry Logic**: Automatic retries for transient failures (database, external APIs)
- **Circuit Breakers**: Prevent cascading failures (future enhancement)
- **Fallbacks**: Graceful degradation when non-critical services fail
- **User Feedback**: Clear error messages with actionable guidance

### Internationalization (Future)

**Prepared for Global Reach**:
- Next.js i18n routing support
- Locale-aware formatting (dates, numbers, currency)
- Translation files for UI strings
- Right-to-left (RTL) layout support

See [Localization and Internationalization](../01-ux-design-system/localization-internationalization.md).

---

## Integration Points

### External Systems

**Braintree (Payments)**:
- Subscription creation, updates, cancellations
- Payment processing (credit cards, PayPal)
- Webhook events for billing lifecycle
- PCI-compliant card storage

See [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md).

**Email Provider**:
- Transactional emails (welcome, password reset, notifications)
- Template rendering with dynamic content
- Delivery tracking and bounce handling

See [Email Provider](../17-integrations/email-provider.md).

**MIDI Support (Browser API)**:
- Web MIDI API for USB and Bluetooth keyboards
- Real-time note detection and feedback
- Browser compatibility matrix

See [MIDI Support](../17-integrations/midi-support.md) and [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md).

**CDN & Media Delivery (Cloudflare R2)**:
- Game assets (HTML5, images, audio)
- User uploads (CSV, profile images)
- Static content delivery

See [CDN and Media](../17-integrations/cdn-and-media.md).

### Future Integrations

**LTI (Learning Tools Interoperability)**:
- Integration with Canvas, Blackboard, Moodle, Google Classroom
- Single sign-on (SSO) from LMS platforms
- Grade passback to institutional systems

See [LMS LTI Spec (Future)](../17-integrations/lms-ltispec-future.md).

**xAPI (Experience API)**:
- Detailed learning analytics and activity tracking
- Integration with Learning Record Stores (LRS)
- Competency-based progression tracking

See [xAPI Implementation](../17-integrations/xapi-implementation.md) (future).

**SCORM Compliance**:
- Package games as SCORM-compliant content
- Integration with corporate LMS platforms
- Standardized content packaging

See [SCORM Compliance](../17-integrations/scorm-compliance.md) (future).

---

## Migration & Upgrade Paths

### From Legacy System (MLC 4.0)

**Data Migration Strategy**:
1. Extract user, game, and score data from legacy database
2. Transform to new schema format
3. Validate data integrity
4. Import via bulk API endpoints
5. Verify and reconcile

See [Data Migration](../20-imports-and-automation/data-migration.md) for detailed migration plan.

### Future Self-Hosting Option

If the platform outgrows managed services, the architecture supports migration to self-hosted infrastructure:
- **Next.js**: Deploy to any Node.js hosting (AWS, GCP, DigitalOcean)
- **PostgreSQL**: Migrate Neon data to self-hosted PostgreSQL
- **R2**: Migrate to AWS S3, GCS, or self-hosted MinIO
- **Vercel**: Replace with Kubernetes or Docker Swarm

All technology choices use open-source software with no vendor lock-in.

---

## Development Workflow

### Local Development

**Prerequisites**:
- Node.js 18+ and npm/yarn
- Docker Desktop (for local PostgreSQL)
- Git

**Setup**:
```bash
git clone [repo]
cd mlc-lms
npm install
docker-compose up -d  # Start local PostgreSQL
cp .env.example .env  # Configure environment variables
npm run dev           # Start Next.js dev server
```

**Development Server**: http://localhost:3000

### Testing Strategy

**Test Pyramid**:
1. **Unit Tests**: Business logic, utility functions (Jest)
2. **Integration Tests**: API routes, database queries (Jest + Supertest)
3. **E2E Tests**: Critical user flows (Playwright)
4. **Visual Regression**: UI component snapshots (Storybook + Chromatic - future)

See [Test Strategy](../19-quality-and-operations/test-strategy.md) for comprehensive testing approach.

### CI/CD Pipeline

**GitHub Actions Workflow**:
1. Lint code (ESLint)
2. Type check (TypeScript)
3. Run unit tests
4. Run integration tests
5. Build Next.js application
6. Deploy to Vercel (automatic)
7. Run E2E tests on preview deployment
8. Merge to main → Deploy to production

See [Release Management](../19-quality-and-operations/release-management.md).

---

## Technology Decision Rationale

### Why Monolithic Architecture (Initially)?

**Advantages for MLC**:
- **Fast iteration**: Single codebase, shared types, no network overhead
- **Simple deployment**: One application to build and deploy
- **Cost efficiency**: No inter-service communication costs
- **Team size**: Small team benefits from unified codebase
- **Clear boundaries**: Internal modules prepare for future service extraction

**When to Consider Microservices**:
- Team grows beyond 10-15 developers
- Specific components need independent scaling
- Different components have conflicting technology needs
- Deployment coordination becomes a bottleneck

### Why Next.js?

**Perfect Fit for MLC**:
- **Full-stack framework**: Frontend + backend in one codebase
- **SSR + SSG**: Fast initial loads + SEO benefits
- **TypeScript support**: End-to-end type safety
- **API routes**: No need for separate backend framework
- **Excellent DX**: Hot reloading, fast refresh, great tooling
- **Vercel hosting**: Optimized deployment and scaling

### Why Serverless-First?

**Aligned with MLC's Needs**:
- **Cost efficiency**: Pay only for actual usage, scale to zero
- **No ops burden**: No servers to manage, patch, or monitor
- **Automatic scaling**: Handles traffic spikes without configuration
- **Fast deployment**: Push code, infrastructure handles the rest
- **Global reach**: Edge network for fast response worldwide

### Why PostgreSQL (Neon)?

**Database Choice**:
- **Relational model**: Perfect fit for MLC's structured data (users, scores, assignments)
- **ACID guarantees**: Critical for billing and score accuracy
- **Rich query capabilities**: Joins, aggregations, full-text search
- **Serverless**: No capacity planning, automatic connection pooling
- **Open source**: No vendor lock-in, easy to migrate

---

## Architectural Principles

### 1. Convention over Configuration

Standardized patterns reduce cognitive load:
- File-based routing (Next.js pages)
- Consistent API response format
- Shared component library (shadcn/ui)
- Standard naming conventions

### 2. Separation of Concerns

Clear boundaries between layers:
- UI components don't access database directly
- API routes handle all data access
- Business logic separated from HTTP concerns
- Type definitions shared across layers

### 3. Fail Fast, Fail Safe

Catch errors early and fail gracefully:
- Type checking at compile time
- Input validation at API boundaries
- Database constraints prevent invalid data
- Graceful error messages for users

### 4. Measure Everything

Instrumentation from day one:
- Event tracking for all user actions (see [Event Model](../15-analytics-and-reporting/event-model.md))
- Performance monitoring (Web Vitals)
- Error tracking and alerting
- Audit logs for critical actions

### 5. Security by Default

Security is not an afterthought:
- HTTPS everywhere, no HTTP fallback
- Secure headers (CSP, HSTS, X-Frame-Options)
- Input sanitization and validation
- Role-based access control enforced at API layer

### 6. Progressive Enhancement

Works for everyone, enhanced for modern browsers:
- Core functionality without JavaScript (SSR)
- Enhanced UX with JavaScript enabled
- Responsive design for all screen sizes
- Accessible to keyboard and screen reader users

---

## Related Documentation

**Architecture Deep-Dives**:
- [Technology Stack](../00-foundations/techstack.md) - Detailed technology choices and rationale
- [Service Boundaries](service-boundaries.md) - Internal module boundaries and contracts
- [Data Model ERD](data-model-erd.md) - Database schema and relationships
- [API Design Principles](api-design-principles.md) - RESTful API conventions
- [REST Endpoints](rest-endpoints.md) - Complete API reference
- [Webhooks](webhooks.md) - Webhook specifications for external integrations
- [Background Jobs](background-jobs.md) - Asynchronous job processing
- [Search Architecture](search-architecture.md) - Full-text search implementation
- [Caching Strategy](caching-strategy.md) - Multi-layer caching policies
- [Security & Privacy](security-privacy.md) - Security controls and privacy safeguards
- [Rate Limiting and Abuse](rate-limiting-and-abuse.md) - API protection strategies
- [Config and Secrets](config-and-secrets.md) - Environment configuration management

**Product & User Experience**:
- [Product Vision](../00-foundations/product-vision.md) - Product goals and success metrics
- [Information Architecture](../00-foundations/information-architecture.md) - User flows and navigation
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) - Educational methodology

**Operations & Quality**:
- [Test Strategy](../19-quality-and-operations/test-strategy.md) - Testing approach and tools
- [Release Management](../19-quality-and-operations/release-management.md) - Deployment process
- [Observability](../19-quality-and-operations/observability.md) - Monitoring and alerting
- [Incident Response](../19-quality-and-operations/incident-response.md) - On-call procedures

---

## Supporting Documents Referenced

This system overview specification draws from the following source documents:

- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Platform overview and business context
- [MLC History Draft 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20History%20Draft%202020.docx.txt) — Historical system architecture evolution
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System architecture requirements
- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser support requirements

## Open Questions

### Near-Term Decisions

1. **Redis for Caching**: Do we need Redis for session storage and caching, or is Vercel's built-in caching sufficient?
2. **Job Queue**: Should we implement a dedicated job queue (BullMQ) from day one, or start with Vercel Cron?
3. **Analytics Platform**: Which analytics platform should we integrate (Plausible, PostHog, custom)?
4. **Error Tracking**: Do we need Sentry or similar, or is Vercel's built-in error tracking sufficient?

### Long-Term Considerations

1. **Service Extraction**: At what scale should we consider extracting specific services (billing, game engine)?
2. **Real-Time Features**: Do we need WebSocket support for real-time leaderboards and messaging?
3. **Mobile Apps**: Should we build native mobile apps (React Native) or continue with PWA approach?
4. **Self-Hosting**: At what revenue threshold does self-hosting become more cost-effective?

---

## Summary

The MLC LMS system architecture is designed to be:
- **Simple to start**: Managed services, minimal configuration
- **Easy to scale**: Serverless auto-scaling, no capacity planning
- **Cost-efficient**: Pay only for usage, scale to zero
- **Fast to iterate**: Single codebase, type-safe, modern tooling
- **Secure by default**: Defense-in-depth, RBAC, audit logging
- **Future-proof**: Clear boundaries, open source, no vendor lock-in

This architecture enables a small team to deliver a world-class music education platform that serves students from individual studios to large school districts, with the proven pedagogical approach that has made MLC successful for over 15 years.
