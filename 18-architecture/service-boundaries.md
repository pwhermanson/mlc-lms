# Service Boundaries

## Purpose

This document defines the internal boundaries between modules and domain contexts within the MLC LMS monolithic application. It establishes clear contracts, responsibilities, and communication patterns that maintain separation of concerns while enabling feature development velocity. These boundaries prepare the system for potential future service extraction while optimizing for a unified codebase today.

## Scope

This specification covers:
- **Domain boundaries**: Logical separation of business capabilities into bounded contexts
- **Module structure**: How the Next.js monolith is organized internally
- **Inter-module communication**: Contracts and patterns for modules to interact
- **Data ownership**: Which modules own which data domains
- **API design patterns**: How internal and external APIs respect boundaries
- **Dependency rules**: What can depend on what to prevent tight coupling
- **Future service extraction**: How boundaries prepare for potential microservices

This document does **not** duplicate:
- Overall system architecture (see [System Overview](system-overview.md))
- Specific API endpoints (see [REST Endpoints](rest-endpoints.md))
- Database schema details (see [Data Model ERD](data-model-erd.md))
- External integration patterns (see [Integrations](../17-integrations/))
- Technology stack decisions (see [Technology Stack](../00-foundations/techstack.md))

---

## Domain Boundaries (Bounded Contexts)

The MLC LMS is organized into the following domain-driven bounded contexts. Each context has clear responsibilities and owns specific data domains.

### 1. Identity & Access Domain

**Responsibility**: Authentication, authorization, user accounts, and session management.

**Owned Entities**:
- Users (all types)
- Roles and permissions
- Sessions and tokens
- Password credentials
- Audit logs for access events

**Module Location**: `/lib/domains/identity`, `/pages/api/auth/*`

**Key Capabilities**:
- User registration and login
- Password management (reset, change)
- Role-based access control (RBAC)
- Session lifecycle management
- Impersonation and support access (see [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md))

**External Dependencies**: NextAuth.js

**Internal Dependencies**: None (foundational domain)

**API Boundaries**:
- `POST /api/auth/register` - Create new user
- `POST /api/auth/login` - Authenticate user
- `POST /api/auth/logout` - End session
- `GET /api/auth/session` - Get current session
- `POST /api/auth/password-reset` - Request password reset

---

### 2. Member & Subscription Domain

**Responsibility**: Organizational hierarchies, subscriptions, billing, and seat management.

**Owned Entities**:
- Members (subscribers/organizations)
- Subscriptions
- Billing history
- Seat allocations
- Payment methods
- Promo codes and sponsor codes

**Module Location**: `/lib/domains/subscriptions`, `/pages/api/subscriptions/*`, `/pages/api/billing/*`

**Key Capabilities**:
- Subscription lifecycle (create, renew, pause, cancel)
- Seat management and pricing calculations (see [Seat Management](../14-payments-and-subscriptions/seat-management.md))
- Payment processing via Braintree
- Invoice generation
- Organizational hierarchy management
- Promo and sponsor code application (see [Promo Codes](../14-payments-and-subscriptions/promo-codes.md), [Sponsor Codes](../14-payments-and-subscriptions/sponsor-codes.md))

**External Dependencies**: Braintree API

**Internal Dependencies**: Identity & Access (for user association)

**API Boundaries**:
- `POST /api/subscriptions` - Create subscription
- `GET /api/subscriptions/:id` - Get subscription details
- `PATCH /api/subscriptions/:id` - Update subscription
- `POST /api/billing/invoice` - Generate invoice
- `GET /api/billing/seats` - Calculate seat pricing

---

### 3. User Management Domain

**Responsibility**: Student, teacher, and admin profiles, relationships, and class/group management.

**Owned Entities**:
- Student profiles
- Teacher profiles
- Admin profiles
- Class/group associations
- Teacher-student relationships
- Organizational hierarchies (shared with Subscription domain)

**Module Location**: `/lib/domains/users`, `/pages/api/students/*`, `/pages/api/teachers/*`, `/pages/api/admins/*`

**Key Capabilities**:
- CRUD operations for all user types
- Bulk user operations (CSV import/export)
- Class and group management
- Teacher-student assignment
- Profile customization
- User search and filtering

**External Dependencies**: None

**Internal Dependencies**: 
- Identity & Access (for authentication)
- Member & Subscription (for seat validation)

**API Boundaries**:
- `POST /api/students` - Create student
- `GET /api/students/:id` - Get student profile
- `POST /api/teachers/:id/students` - Assign students to teacher
- `GET /api/admins/:id/hierarchy` - Get organizational structure

---

### 4. Content Domain

**Responsibility**: Games registry, sequences, curriculum, and content versioning.

**Owned Entities**:
- Games (definitions, metadata, taxonomy)
- Sequences (learning paths)
- Curriculum alignments (Piano Adventures, method books)
- Content versions
- Game assets (references to R2 storage)

**Module Location**: `/lib/domains/content`, `/pages/api/games/*`, `/pages/api/sequences/*`

**Key Capabilities**:
- Games registry and CRUD (see [Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md))
- Game metadata and taxonomy management (see [Game Model](../08-games-registry-and-authoring/game-model.md))
- Sequence authoring and libraries (see [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))
- Content versioning (see [Content Versioning](../07-content-architecture/content-versioning.md))
- Curriculum alignment tools
- Game asset management (R2 storage integration)

**External Dependencies**: Cloudflare R2 (for asset storage)

**Internal Dependencies**: Identity & Access (for access control)

**API Boundaries**:
- `POST /api/games` - Create game
- `GET /api/games/:id` - Get game details
- `POST /api/sequences` - Create sequence
- `GET /api/sequences/:id/games` - Get games in sequence

---

### 5. Learning Engine Domain

**Responsibility**: Assignment generation, game progression, scoring, and progress tracking.

**Owned Entities**:
- Assignments (instances)
- Progress records
- Scores (per game stage)
- Retention schedules
- Game completion states

**Module Location**: `/lib/domains/learning`, `/pages/api/assignments/*`, `/pages/api/progress/*`, `/pages/api/scores/*`

**Key Capabilities**:
- Assignment generation from sequences (see [Assignment Generation](../10-assignments-engine/assignment-generation.md))
- Game stage progression (Learn → Play → Quiz → Challenge → Review)
- Score recording and validation
- Progress tracking and analytics
- Retention review logic (see [Retention Review Logic](../10-assignments-engine/retention-review-logic.md))
- Free play vs. assigned mode differentiation (see [Free Play vs Assigned](../10-assignments-engine/free-play-vs-assigned.md))

**External Dependencies**: None

**Internal Dependencies**: 
- Content (for game and sequence definitions)
- User Management (for student and teacher associations)
- Gamification (for badge/streak updates)

**API Boundaries**:
- `POST /api/assignments` - Create assignment
- `GET /api/assignments/:id/next` - Get next game in assignment
- `POST /api/scores` - Record game score
- `GET /api/progress/:studentId` - Get student progress

---

### 6. Gamification Domain

**Responsibility**: Badges, points, streaks, leaderboards, and achievements.

**Owned Entities**:
- Badge definitions
- Badge awards
- Points and streaks
- Leaderboard entries
- Achievement records

**Module Location**: `/lib/domains/gamification`, `/pages/api/badges/*`, `/pages/api/leaderboards/*`

**Key Capabilities**:
- Badge system (see [Badges System](../12-gamification/badges-system.md))
- Points and streaks tracking (see [Points and Streaks](../12-gamification/points-and-streaks.md))
- Leaderboard generation (see [Leaderboards](../03-student-experience/leaderboards.md))
- Challenge boards (see [Challenge Boards](../12-gamification/challenge-boards.md))
- Achievement sharing (see [Achievements Sharing](../12-gamification/achievements-sharing.md))

**External Dependencies**: None

**Internal Dependencies**: 
- Learning Engine (for score events)
- User Management (for student profiles)

**API Boundaries**:
- `GET /api/badges/:studentId` - Get student badges
- `POST /api/badges/award` - Award badge (internal)
- `GET /api/leaderboards/:type` - Get leaderboard data
- `POST /api/points` - Update points (internal)

---

### 7. Reporting & Analytics Domain

**Responsibility**: Reports, dashboards, event tracking, and analytics aggregation.

**Owned Entities**:
- Event logs
- Report definitions
- Analytics aggregations
- Dashboard configurations
- KPI calculations

**Module Location**: `/lib/domains/analytics`, `/pages/api/reports/*`, `/pages/api/analytics/*`

**Key Capabilities**:
- Event tracking (see [Event Model](../15-analytics-and-reporting/event-model.md))
- Teacher reports and KPIs (see [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md))
- Admin reports and KPIs (see [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md))
- System admin dashboards (see [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md))
- Data aggregation and caching

**External Dependencies**: None (future: analytics platform integration)

**Internal Dependencies**: 
- All domains (consume events from all)
- User Management (for report scoping)

**API Boundaries**:
- `POST /api/events` - Track event (internal)
- `GET /api/reports/teacher/:id` - Get teacher report
- `GET /api/reports/admin/:id` - Get admin report
- `GET /api/analytics/summary` - Get system summary

---

### 8. Messaging & Notifications Domain

**Responsibility**: In-app notifications, emails, system announcements, and teacher-student messaging.

**Owned Entities**:
- Notifications
- Messages
- System announcements
- Email templates
- Delivery logs

**Module Location**: `/lib/domains/messaging`, `/pages/api/messages/*`, `/pages/api/notifications/*`

**Key Capabilities**:
- In-app notifications (see [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md))
- Email template rendering and delivery (see [Email Templates](../13-messaging-and-notifications/email-templates.md))
- System announcements (see [System Announcements](../13-messaging-and-notifications/system-announcements.md))
- Teacher-student messaging (see [Teacher Messaging](../04-teacher-experience/teacher-messaging.md))
- Notification preferences

**External Dependencies**: Email provider API (see [Email Provider](../17-integrations/email-provider.md))

**Internal Dependencies**: 
- Identity & Access (for recipient lookup)
- User Management (for teacher-student relationships)

**API Boundaries**:
- `POST /api/notifications` - Create notification
- `GET /api/notifications/:userId` - Get user notifications
- `POST /api/messages` - Send message
- `GET /api/messages/:userId` - Get user messages
- `POST /api/announcements` - Create system announcement

---

### 9. Search & Discovery Domain

**Responsibility**: Full-text search, filtering, faceting, and content discovery.

**Owned Entities**:
- Search indices
- Filter configurations
- Facet definitions
- Search history (optional)

**Module Location**: `/lib/domains/search`, `/pages/api/search/*`

**Key Capabilities**:
- Full-text search across games, sequences, content (see [Search Indexing](../11-search-and-discovery/search-indexing.md))
- Faceted filtering (see [Filters and Facets](../11-search-and-discovery/filters-and-facets.md))
- Teacher quick-find tools (see [Teacher Quick Find](../11-search-and-discovery/teacher-quick-find.md))
- Search result ranking and relevance

**External Dependencies**: PostgreSQL full-text search (future: dedicated search engine)

**Internal Dependencies**: 
- Content (for game and sequence data)
- User Management (for scoping results by permissions)

**API Boundaries**:
- `GET /api/search?q=:query` - Full-text search
- `GET /api/search/filters` - Get available filters
- `GET /api/search/facets` - Get facet counts

---

## Module Structure and Organization

### File System Layout

The Next.js application is organized to reflect domain boundaries:

```
mlc-lms/
├── pages/
│   ├── api/                    # API routes (external-facing)
│   │   ├── auth/              # Identity & Access endpoints
│   │   ├── subscriptions/     # Member & Subscription endpoints
│   │   ├── students/          # User Management endpoints
│   │   ├── teachers/          # User Management endpoints
│   │   ├── games/             # Content endpoints
│   │   ├── sequences/         # Content endpoints
│   │   ├── assignments/       # Learning Engine endpoints
│   │   ├── progress/          # Learning Engine endpoints
│   │   ├── badges/            # Gamification endpoints
│   │   ├── reports/           # Analytics endpoints
│   │   ├── notifications/     # Messaging endpoints
│   │   └── search/            # Search & Discovery endpoints
│   ├── student/               # Student portal pages
│   ├── teacher/               # Teacher portal pages
│   ├── admin/                 # Admin portal pages
│   └── sysadmin/              # System admin portal pages
├── lib/
│   ├── domains/               # Domain logic (internal modules)
│   │   ├── identity/          # Identity & Access domain
│   │   ├── subscriptions/     # Member & Subscription domain
│   │   ├── users/             # User Management domain
│   │   ├── content/           # Content domain
│   │   ├── learning/          # Learning Engine domain
│   │   ├── gamification/      # Gamification domain
│   │   ├── analytics/         # Reporting & Analytics domain
│   │   ├── messaging/         # Messaging & Notifications domain
│   │   └── search/            # Search & Discovery domain
│   ├── database/              # Database client and utilities
│   ├── storage/               # R2 storage client
│   └── shared/                # Cross-cutting utilities
├── components/                # React components (by domain)
│   ├── student/
│   ├── teacher/
│   ├── admin/
│   └── shared/
└── types/                     # TypeScript type definitions
    ├── entities/              # Domain entity types
    ├── api/                   # API request/response types
    └── shared/                # Shared types
```

### Domain Module Structure

Each domain module follows a consistent internal structure:

```
lib/domains/{domain}/
├── index.ts              # Public API exports (facade)
├── entities.ts           # Entity type definitions
├── repository.ts         # Data access layer
├── service.ts            # Business logic
├── validators.ts         # Input validation
├── errors.ts             # Domain-specific errors
└── events.ts             # Domain events (optional)
```

**Example: Content Domain**

```
lib/domains/content/
├── index.ts              # Exports: GameService, SequenceService
├── entities.ts           # Types: Game, Sequence, GameMetadata
├── game-repository.ts    # Data access for games
├── game-service.ts       # Business logic for games
├── game-validators.ts    # Validation schemas for games
├── sequence-repository.ts
├── sequence-service.ts
├── sequence-validators.ts
└── content-errors.ts     # GameNotFoundError, etc.
```

---

## Inter-Module Communication Patterns

### 1. Direct Function Calls (Preferred for Monolith)

Modules communicate via direct function calls to exported service methods. This is the fastest and simplest approach within a monolith.

**Example**: Learning Engine calls Content Domain

```typescript
// In Learning Engine domain
import { GameService } from '@/lib/domains/content';

export class AssignmentService {
  async generateAssignment(sequenceId: string) {
    const sequence = await GameService.getSequence(sequenceId);
    const games = await GameService.getGamesInSequence(sequenceId);
    // ... generate assignment
  }
}
```

**Rules**:
- Only call public APIs exported from domain's `index.ts`
- Never import internal implementation files (e.g., `repository.ts`) directly
- Use TypeScript interfaces to define contracts

---

### 2. Event-Based Communication (For Decoupling)

For loosely coupled interactions, domains emit events that other domains can subscribe to.

**Example**: Learning Engine emits score event → Gamification domain listens

```typescript
// In Learning Engine domain
import { EventBus } from '@/lib/shared/event-bus';

export class ScoreService {
  async recordScore(studentId: string, gameId: string, score: number) {
    // ... save score to database
    
    // Emit event for other domains
    await EventBus.emit('score.recorded', {
      studentId,
      gameId,
      score,
      timestamp: new Date(),
    });
  }
}

// In Gamification domain
import { EventBus } from '@/lib/shared/event-bus';

EventBus.on('score.recorded', async (event) => {
  await BadgeService.checkForBadgeAwards(event.studentId);
  await PointsService.updatePoints(event.studentId, event.score);
});
```

**Use Cases**:
- Triggering side effects (badges, notifications)
- Analytics and telemetry
- Audit logging
- Async operations that don't need immediate response

---

### 3. API Route Layer (External-Facing)

API routes act as the external interface and orchestrate calls to multiple domains.

**Example**: Assignment creation API orchestrates multiple domains

```typescript
// pages/api/assignments/index.ts
import { withAuth } from '@/lib/middleware/auth';
import { AssignmentService } from '@/lib/domains/learning';
import { UserService } from '@/lib/domains/users';
import { NotificationService } from '@/lib/domains/messaging';

export default withAuth(async (req, res) => {
  const { sequenceId, studentIds, dueDate } = req.body;
  
  // Orchestrate multiple domains
  const assignment = await AssignmentService.create({
    sequenceId,
    createdBy: req.user.id,
    dueDate,
  });
  
  await AssignmentService.assignToStudents(assignment.id, studentIds);
  
  const students = await UserService.getStudentsByIds(studentIds);
  await NotificationService.notifyAssignmentCreated(students, assignment);
  
  res.json({ assignment });
});
```

**Rules**:
- API routes orchestrate, domains execute
- No business logic in API routes
- Validate input at API layer
- Handle errors consistently

---

## Data Ownership and Access Patterns

### Principle: Each Domain Owns Its Data

- **Direct database access**: Only through domain's repository layer
- **Cross-domain queries**: Use domain services, not direct SQL joins
- **Shared data**: Duplicate or use consistent references (IDs)

### Data Access Rules

| Domain | Owns Data | Can Read From | Can Write To |
|--------|-----------|---------------|--------------|
| Identity & Access | users, roles, sessions | (own only) | (own only) |
| Subscriptions | subscriptions, billing | Identity (user IDs) | (own only) |
| User Management | profiles, relationships | Identity (user IDs), Subscriptions (seat limits) | (own only) |
| Content | games, sequences | Identity (permissions) | (own only) |
| Learning Engine | assignments, scores, progress | Content (games, sequences), Users (students, teachers) | (own only), Gamification (via events) |
| Gamification | badges, points, streaks | Learning Engine (scores), Users (students) | (own only) |
| Analytics | events, reports | All domains (read-only) | (own only) |
| Messaging | notifications, messages | Identity (users), Users (relationships) | (own only) |
| Search | indices | Content, Users (for indexing) | (own only) |

### Example: Cross-Domain Data Access

**Scenario**: Get student's assignment progress with badge info

```typescript
// ❌ BAD: Direct SQL join across domains
const result = await db.query(`
  SELECT a.*, p.*, b.*
  FROM assignments a
  JOIN progress p ON a.id = p.assignment_id
  JOIN badges b ON p.student_id = b.student_id
`);

// ✅ GOOD: Call each domain's service
const assignment = await AssignmentService.getById(assignmentId);
const progress = await ProgressService.getForAssignment(assignmentId, studentId);
const badges = await BadgeService.getForStudent(studentId);

return {
  assignment,
  progress,
  badges,
};
```

---

## Dependency Rules and Constraints

### Dependency Direction

Domains follow a layered dependency structure:

```
┌─────────────────────────────────────────────────────┐
│         Presentation Layer (Pages/Components)        │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│         API Layer (Next.js API Routes)               │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│         Domain Layer (Business Logic)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐          │
│  │ Learning │  │  Users   │  │Messaging │          │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘          │
│       │             │              │                 │
│  ┌────▼─────┐  ┌───▼──────┐  ┌───▼──────┐          │
│  │ Content  │  │ Identity │  │Analytics │          │
│  └──────────┘  └──────────┘  └──────────┘          │
└───────────────────────┬─────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────┐
│      Infrastructure Layer (Database, Storage)        │
└─────────────────────────────────────────────────────┘
```

### Rules

1. **Higher layers depend on lower layers** (never reverse)
2. **Domains at same layer** should minimize dependencies
3. **Shared utilities** can be used by all layers
4. **Infrastructure** (database, storage) is accessed only via domain repositories

### Allowed Dependencies

- ✅ API Routes → Domain Services
- ✅ Domain Services → Domain Repositories
- ✅ Domain Repositories → Database Client
- ✅ Any Layer → Shared Utilities
- ✅ Domain A Service → Domain B Service (public API only)

### Forbidden Dependencies

- ❌ Domain Repositories → Domain Services (circular)
- ❌ Domain A Repository → Domain B Repository (cross-domain data access)
- ❌ API Routes → Domain Repositories (skip service layer)
- ❌ Any code → Internal domain files (must use public exports)

---

## Contracts and Interfaces

### TypeScript Interfaces for Boundary Contracts

Each domain defines interfaces for its public API:

```typescript
// lib/domains/content/index.ts
export interface IGameService {
  getById(gameId: string): Promise<Game>;
  search(criteria: GameSearchCriteria): Promise<Game[]>;
  create(data: CreateGameInput): Promise<Game>;
  update(gameId: string, data: UpdateGameInput): Promise<Game>;
  delete(gameId: string): Promise<void>;
}

export const GameService: IGameService = {
  // implementation
};
```

**Benefits**:
- Clear contract definition
- Easy to mock for testing
- Future-proof for service extraction

### API Request/Response Types

All API endpoints define strict input/output types:

```typescript
// types/api/assignments.ts
export interface CreateAssignmentRequest {
  sequenceId: string;
  studentIds: string[];
  dueDate: string;
  notes?: string;
}

export interface CreateAssignmentResponse {
  assignment: Assignment;
  assignedCount: number;
}
```

---

## Future Service Extraction Readiness

### When to Extract a Service

Consider extracting a domain into a separate service when:

1. **Team scaling**: Domain requires dedicated team (6+ developers)
2. **Performance isolation**: Domain needs independent scaling characteristics
3. **Technology requirements**: Domain needs different tech stack
4. **Deployment independence**: Domain changes frequently and needs isolated deploys
5. **Third-party integration**: Domain is primarily a wrapper for external service

### Extraction Process

The current architecture makes extraction straightforward:

**Step 1: Copy domain module** → Separate repository
**Step 2: Add HTTP API layer** → RESTful endpoints
**Step 3: Replace direct calls with HTTP** → API client in main app
**Step 4: Migrate database** → Extract domain's tables to separate database
**Step 5: Deploy independently** → Separate hosting

**Example**: Extracting Gamification Domain

```typescript
// Before: Direct function call
import { BadgeService } from '@/lib/domains/gamification';
const badges = await BadgeService.getForStudent(studentId);

// After: HTTP API call
import { GamificationClient } from '@/lib/clients/gamification';
const badges = await GamificationClient.getStudentBadges(studentId);
```

### Maintaining Boundaries Prevents Extraction Pain

By enforcing strict boundaries today:
- ✅ Clear ownership of data
- ✅ Well-defined APIs
- ✅ Minimal cross-domain coupling
- ✅ Service extraction is feasible without major refactoring

---

## Permissions

- **System Admins**: Can view all module boundaries and internal structure
- **Developers**: Must understand and respect domain boundaries when implementing features
- **API Consumers**: Interact only with public API routes, not internal domain modules

---

## Supporting Documents Referenced

This service boundaries specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Content service architecture requirements
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User service and authentication architecture
- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Billing service requirements

## Dependencies

This specification relies on:
- [System Overview](system-overview.md) - Overall architecture context
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Authorization patterns
- [Event Model](../15-analytics-and-reporting/event-model.md) - Event-driven communication
- [API Design Principles](api-design-principles.md) - RESTful API patterns

---

## Open questions

1. **Event Bus Implementation**: Should we use a lightweight in-process event bus, or introduce a message queue (Redis Pub/Sub, BullMQ) for event-driven communication?
2. **Transaction Boundaries**: How do we handle cross-domain transactions (e.g., assignment creation + notification)? Use database transactions, saga pattern, or eventual consistency?
3. **Shared Entities**: Should some entities (e.g., User) be shared across domains, or should each domain maintain its own copy/projection?
4. **Service Mesh**: At what scale should we consider a service mesh (Istio, Linkerd) if we extract services?
5. **API Gateway**: If we extract services, do we need an API gateway (Kong, Tyk) or is Next.js API layer sufficient?
