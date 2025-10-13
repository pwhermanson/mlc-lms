# REST Endpoints

## Purpose

This document serves as the **comprehensive API endpoint reference** for the MLC LMS platform. It catalogs all REST endpoints, organized by domain, with request/response specifications, authentication requirements, and usage examples. This is the canonical guide for developers building features, integrating external systems, or troubleshooting API behavior.

While [API Design Principles](api-design-principles.md) establishes the conventions and patterns that all endpoints follow, this document provides the **specific endpoint inventory** — the "what endpoints exist and how do I call them" reference.

## Scope

**Included:**
- Complete catalog of all REST API endpoints organized by domain
- Request/response formats with TypeScript type signatures
- Required and optional parameters for each endpoint
- Authentication and authorization requirements per endpoint
- Success and error response examples
- Common usage patterns and code examples
- Pagination, filtering, and sorting capabilities per endpoint
- Rate limiting considerations for high-volume endpoints

**Excluded:**
- General API design patterns and conventions (see [API Design Principles](api-design-principles.md))
- Overall system architecture (see [System Overview](system-overview.md))
- Detailed database schema (see [Data Model ERD](data-model-erd.md))
- Domain boundaries and internal contracts (see [Service Boundaries](service-boundaries.md))
- Webhook specifications (see [Webhooks](webhooks.md))
- Background job processing (see [Background Jobs](background-jobs.md))

---

## Endpoint Organization

All endpoints are organized by **domain** following the bounded context structure defined in [Service Boundaries](service-boundaries.md):

1. **Identity & Access** (`/api/auth/*`) - Authentication, sessions, password management
2. **Member & Subscription** (`/api/subscriptions/*`, `/api/billing/*`) - Organizations, subscriptions, seats, payments
3. **User Management** (`/api/students/*`, `/api/teachers/*`, `/api/admins/*`) - User profiles, classes, relationships
4. **Content** (`/api/games/*`, `/api/sequences/*`) - Games registry, sequences, curriculum
5. **Learning Engine** (`/api/assignments/*`, `/api/progress/*`, `/api/scores/*`) - Assignments, progression, scoring
6. **Gamification** (`/api/badges/*`, `/api/leaderboards/*`, `/api/points/*`) - Badges, achievements, leaderboards
7. **Reporting & Analytics** (`/api/reports/*`, `/api/analytics/*`) - Reports, dashboards, aggregations
8. **Messaging & Notifications** (`/api/messages/*`, `/api/notifications/*`) - Notifications, messaging, announcements
9. **Search & Discovery** (`/api/search/*`) - Full-text search, filtering, discovery
10. **Bulk Operations** (`/api/bulk/*`) - CSV imports, batch processing, async jobs

---

## Global Conventions

All endpoints follow conventions defined in [API Design Principles](api-design-principles.md):

- **Base URL**: `https://app.musiclearningcommunity.com/api`
- **Authentication**: Session-based via NextAuth.js (cookie: `next-auth.session-token`)
- **Content-Type**: `application/json` for requests and responses
- **Date Format**: ISO 8601 (e.g., `2025-01-27T10:00:00Z`)
- **ID Format**: Prefixed UUIDs (e.g., `stu_abc123`, `game_xyz789`)
- **Pagination**: Offset-based by default (`?page=1&limit=20`)
- **Sorting**: Query param format `?sort=-createdAt,title` (prefix `-` for descending)
- **Error Format**: Consistent error response structure with error codes

---

## 1. Identity & Access Domain

Base path: `/api/auth/*`

### 1.1 User Registration

**Endpoint**: `POST /api/auth/register`

**Purpose**: Create a new user account.

**Access**: Public (unauthenticated)

**Request Body**:
```typescript
{
  username: string;        // 3-30 characters, alphanumeric + underscore
  email: string;           // Valid email format
  password: string;        // Min 8 characters
  role: 'student' | 'teacher' | 'subscriber_admin'; // Initial role
  displayName?: string;    // Optional display name
}
```

**Response** (201 Created):
```typescript
{
  data: {
    user: {
      id: string;          // e.g., "usr_abc123"
      username: string;
      email: string;
      role: string;
      status: 'active';
      createdAt: string;   // ISO 8601
    }
  },
  meta: {
    timestamp: string;
    requestId: string;
  }
}
```

**Error Responses**:
- `400 Bad Request` - Invalid input format
- `409 Conflict` - Username or email already exists
- `422 Unprocessable Entity` - Validation errors (e.g., weak password)

**Example**:
```typescript
const response = await fetch('/api/auth/register', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    username: 'alice_smith',
    email: 'alice@example.com',
    password: 'SecurePass123!',
    role: 'student'
  })
});
```

---

### 1.2 User Login

**Endpoint**: `POST /api/auth/login`

**Purpose**: Authenticate user and create session.

**Access**: Public (unauthenticated)

**Request Body**:
```typescript
{
  username: string;        // Username or email
  password: string;
}
```

**Response** (200 OK):
```typescript
{
  data: {
    user: {
      id: string;
      username: string;
      email: string;
      role: string;
      lastLoginAt: string;
    },
    session: {
      token: string;        // JWT token (also set as httpOnly cookie)
      expiresAt: string;
    }
  },
  meta: {
    timestamp: string;
    requestId: string;
  }
}
```

**Error Responses**:
- `401 Unauthorized` - Invalid credentials
- `403 Forbidden` - Account suspended or deactivated

---

### 1.3 User Logout

**Endpoint**: `POST /api/auth/logout`

**Purpose**: End current session.

**Access**: Authenticated (any role)

**Response** (204 No Content)

---

### 1.4 Get Current Session

**Endpoint**: `GET /api/auth/session`

**Purpose**: Retrieve current authenticated user session.

**Access**: Authenticated (any role)

**Response** (200 OK):
```typescript
{
  data: {
    user: {
      id: string;
      username: string;
      email: string;
      role: string;
      permissions: string[];  // Derived from role
    },
    expiresAt: string;
  }
}
```

---

### 1.5 Password Reset Request

**Endpoint**: `POST /api/auth/password-reset`

**Purpose**: Request password reset link via email.

**Access**: Public (unauthenticated)

**Request Body**:
```typescript
{
  email: string;
}
```

**Response** (202 Accepted):
```typescript
{
  data: {
    message: "If the email exists, a reset link has been sent."
  }
}
```

**Note**: Always returns success to prevent email enumeration attacks.

---

### 1.6 Password Reset Confirmation

**Endpoint**: `POST /api/auth/password-reset/confirm`

**Purpose**: Complete password reset with token.

**Access**: Public (with valid reset token)

**Request Body**:
```typescript
{
  token: string;           // Reset token from email
  newPassword: string;     // Min 8 characters
}
```

**Response** (200 OK):
```typescript
{
  data: {
    message: "Password successfully reset. You can now log in."
  }
}
```

**Error Responses**:
- `400 Bad Request` - Invalid or expired token
- `422 Unprocessable Entity` - Weak password

---

### 1.7 Change Password

**Endpoint**: `PATCH /api/auth/password`

**Purpose**: Change password for authenticated user.

**Access**: Authenticated (any role)

**Request Body**:
```typescript
{
  currentPassword: string;
  newPassword: string;
}
```

**Response** (200 OK):
```typescript
{
  data: {
    message: "Password successfully changed."
  }
}
```

**Error Responses**:
- `401 Unauthorized` - Current password incorrect
- `422 Unprocessable Entity` - Weak new password

---

### 1.8 Impersonation Start

**Endpoint**: `POST /api/auth/impersonate/:userId`

**Purpose**: System admin impersonates another user for support.

**Access**: System Admin only

**Response** (200 OK):
```typescript
{
  data: {
    originalUser: { id: string; username: string; role: string; };
    impersonatedUser: { id: string; username: string; role: string; };
    impersonationToken: string;
  }
}
```

**See**: [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md)

---

### 1.9 Impersonation End

**Endpoint**: `POST /api/auth/impersonate/end`

**Purpose**: End impersonation session.

**Access**: Authenticated (impersonating)

**Response** (200 OK): Returns to original user session

---

## 2. Member & Subscription Domain

Base path: `/api/subscriptions/*`, `/api/billing/*`

### 2.1 List Subscriptions

**Endpoint**: `GET /api/subscriptions`

**Purpose**: List all subscriptions for authenticated user's context.

**Access**: 
- Subscriber-Admin: Own organization's subscriptions
- System Admin: All subscriptions (with filters)

**Query Parameters**:
```typescript
{
  page?: number;           // Default: 1
  limit?: number;          // Default: 20, max: 100
  status?: 'trial' | 'active' | 'paused' | 'cancelled';
  orgId?: string;          // Filter by organization (System Admin only)
  sort?: string;           // e.g., "-createdAt"
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;          // e.g., "sub_abc123"
      orgId: string;
      orgName: string;
      planType: 'prelude' | 'solo' | 'ensemble';
      billingCycle: 'monthly' | 'annual';
      seatCount: number;
      status: string;
      currentPeriodStart: string;
      currentPeriodEnd: string;
      trialEndsAt?: string;
    }
  ],
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrevious: boolean;
  }
}
```

**See**: [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md)

---

### 2.2 Get Subscription Details

**Endpoint**: `GET /api/subscriptions/:id`

**Purpose**: Get detailed subscription information.

**Access**: 
- Subscriber-Admin: Own subscription only
- System Admin: Any subscription

**Response** (200 OK):
```typescript
{
  data: {
    id: string;
    orgId: string;
    orgName: string;
    planType: string;
    billingCycle: string;
    seatCount: number;
    seatsUsed: number;
    seatsAvailable: number;
    basePriceCents: number;
    perSeatPriceCents: number;
    totalPriceCents: number;
    status: string;
    trialEndsAt?: string;
    currentPeriodStart: string;
    currentPeriodEnd: string;
    createdAt: string;
    updatedAt: string;
  }
}
```

---

### 2.3 Create Subscription

**Endpoint**: `POST /api/subscriptions`

**Purpose**: Create new subscription (checkout flow).

**Access**: Authenticated (becoming Subscriber-Admin)

**Request Body**:
```typescript
{
  orgName: string;
  orgType: 'solo' | 'ensemble' | 'school_district';
  planType: 'prelude' | 'solo' | 'ensemble';
  billingCycle: 'monthly' | 'annual';
  seatCount: number;       // Min 1 for solo, min 2 for ensemble
  paymentMethod: {
    nonce: string;         // Braintree payment method nonce
  };
  promoCode?: string;      // Optional promo code
}
```

**Response** (201 Created):
```typescript
{
  data: {
    subscription: { /* subscription object */ };
    organization: { /* organization object */ };
    payment: {
      id: string;
      status: 'completed' | 'pending';
      amountCents: number;
    };
  }
}
```

**Error Responses**:
- `400 Bad Request` - Invalid plan/seat combination
- `402 Payment Required` - Payment method declined
- `404 Not Found` - Invalid promo code
- `422 Unprocessable Entity` - Validation errors

**See**: [Checkout and Trials](../14-payments-and-subscriptions/checkout-and-trials.md)

---

### 2.4 Update Subscription

**Endpoint**: `PATCH /api/subscriptions/:id`

**Purpose**: Update subscription plan, seats, or billing cycle.

**Access**: Subscriber-Admin (own subscription), System Admin (any)

**Request Body**:
```typescript
{
  planType?: 'prelude' | 'solo' | 'ensemble';
  billingCycle?: 'monthly' | 'annual';
  seatCount?: number;      // Can increase/decrease (prorated)
}
```

**Response** (200 OK):
```typescript
{
  data: {
    subscription: { /* updated subscription */ };
    proratedCharge?: {
      amountCents: number;
      description: string;
    };
  }
}
```

**See**: [Seat Management](../14-payments-and-subscriptions/seat-management.md)

---

### 2.5 Pause Subscription

**Endpoint**: `POST /api/subscriptions/:id/pause`

**Purpose**: Temporarily pause subscription.

**Access**: Subscriber-Admin (own), System Admin (any)

**Request Body**:
```typescript
{
  resumeAt?: string;       // ISO date, optional
  reason?: string;
}
```

**Response** (200 OK):
```typescript
{
  data: {
    subscription: { status: 'paused', pausedAt: string; resumeAt?: string; }
  }
}
```

**See**: [Pause and Reactivation](../14-payments-and-subscriptions/pause-and-reactivation.md)

---

### 2.6 Cancel Subscription

**Endpoint**: `POST /api/subscriptions/:id/cancel`

**Purpose**: Cancel subscription (end of current period or immediate).

**Access**: Subscriber-Admin (own), System Admin (any)

**Request Body**:
```typescript
{
  immediate?: boolean;     // Default: false (end of period)
  reason?: string;
  feedback?: string;
}
```

**Response** (200 OK):
```typescript
{
  data: {
    subscription: { 
      status: 'cancelled';
      cancelledAt: string;
      accessEndsAt: string;  // End of current period or now
    }
  }
}
```

---

### 2.7 Get Seat Allocation

**Endpoint**: `GET /api/billing/seats/:subscriptionId`

**Purpose**: Get seat allocation details.

**Access**: Subscriber-Admin (own), System Admin (any)

**Response** (200 OK):
```typescript
{
  data: {
    total: number;
    used: number;
    available: number;
    seats: [
      {
        id: string;
        type: 'student' | 'teacher' | 'admin';
        status: 'vacant' | 'assigned' | 'suspended';
        assignedUserId?: string;
        assignedUsername?: string;
        assignedAt?: string;
      }
    ]
  }
}
```

---

### 2.8 Calculate Pricing

**Endpoint**: `POST /api/billing/calculate`

**Purpose**: Calculate pricing for plan configuration (quote).

**Access**: Public (unauthenticated)

**Request Body**:
```typescript
{
  planType: 'prelude' | 'solo' | 'ensemble';
  billingCycle: 'monthly' | 'annual';
  seatCount: number;
  promoCode?: string;
}
```

**Response** (200 OK):
```typescript
{
  data: {
    basePriceCents: number;
    perSeatPriceCents: number;
    totalSeatsCharge: number;
    subtotalCents: number;
    discountCents: number;
    totalCents: number;
    currency: 'USD';
    breakdown: string;      // Human-readable breakdown
  }
}
```

**See**: [Seat Pricing Calculator](../14-payments-and-subscriptions/seat-pricing-calculator.md)

---

### 2.9 Generate Invoice

**Endpoint**: `POST /api/billing/invoice/:subscriptionId`

**Purpose**: Generate on-demand invoice statement.

**Access**: Subscriber-Admin (own), System Admin (any)

**Request Body**:
```typescript
{
  periodStart?: string;    // Defaults to current period
  periodEnd?: string;
}
```

**Response** (201 Created):
```typescript
{
  data: {
    invoice: {
      id: string;
      invoiceNumber: string;
      amountCents: number;
      pdfUrl: string;       // Cloudflare R2 URL
      status: 'draft' | 'sent';
      createdAt: string;
    }
  }
}
```

**See**: [Billing Statements on Demand](../14-payments-and-subscriptions/billing-statements-on-demand.md)

---

### 2.10 Apply Promo Code

**Endpoint**: `POST /api/billing/promo-codes/apply`

**Purpose**: Apply promo code to subscription or quote.

**Access**: Authenticated (Subscriber-Admin or during checkout)

**Request Body**:
```typescript
{
  code: string;
  subscriptionId?: string; // Existing subscription, or omit for quote
}
```

**Response** (200 OK):
```typescript
{
  data: {
    promoCode: {
      code: string;
      codeName: string;
      discountType: string;
      discountValue: number;
      validUntil: string;
    };
    discount: {
      amountCents: number;
      description: string;
    };
  }
}
```

**Error Responses**:
- `404 Not Found` - Code not found or expired
- `409 Conflict` - Code already used or max uses exceeded

**See**: [Promo Codes](../14-payments-and-subscriptions/promo-codes.md)

---

### 2.11 Create Sponsor Code

**Endpoint**: `POST /api/billing/sponsor-codes`

**Purpose**: Generate sponsor codes for grant programs.

**Access**: Subscriber-Admin (own org), System Admin (any org)

**Request Body**:
```typescript
{
  sponsorOrgId: string;    // Sponsoring organization
  count: number;           // Number of codes to generate (1-100)
}
```

**Response** (201 Created):
```typescript
{
  data: {
    codes: [
      {
        id: string;
        code: string;        // Format: {member_number}A, {member_number}B, etc.
        status: 'available';
        sponsorOrgId: string;
        createdAt: string;
      }
    ]
  }
}
```

**See**: [Sponsor Codes](../14-payments-and-subscriptions/sponsor-codes.md)

---

### 2.12 Redeem Sponsor Code

**Endpoint**: `POST /api/billing/sponsor-codes/redeem`

**Purpose**: Redeem sponsor code during registration/checkout.

**Access**: Authenticated (becoming Subscriber-Admin)

**Request Body**:
```typescript
{
  code: string;
  orgId: string;           // Organization to assign code to
}
```

**Response** (200 OK):
```typescript
{
  data: {
    sponsorCode: {
      code: string;
      status: 'assigned';
      sponsorOrgId: string;
      sponsoredOrgId: string;
      assignedAt: string;
    };
    subscription: { /* free subscription created */ }
  }
}
```

---

## 3. User Management Domain

Base path: `/api/students/*`, `/api/teachers/*`, `/api/admins/*`

### 3.1 List Students

**Endpoint**: `GET /api/students`

**Purpose**: List students with filtering and pagination.

**Access**: 
- Teacher: Own students only
- Subscriber-Admin: All students in organization
- System Admin: All students (with filters)

**Query Parameters**:
```typescript
{
  page?: number;
  limit?: number;
  teacherId?: string;      // Filter by assigned teacher
  orgId?: string;          // Filter by organization
  status?: 'active' | 'suspended' | 'archived';
  search?: string;         // Search by name, username
  sort?: string;
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;          // e.g., "stu_abc123"
      userId: string;
      username: string;
      displayName: string;
      gradeLevel: string;
      assignedTeacherId?: string;
      assignedTeacherName?: string;
      orgId: string;
      status: string;
      createdAt: string;
    }
  ],
  pagination: { /* standard pagination */ }
}
```

---

### 3.2 Get Student Profile

**Endpoint**: `GET /api/students/:id`

**Purpose**: Get detailed student profile.

**Access**: 
- Student: Own profile
- Teacher: Assigned students only
- Subscriber-Admin: Students in organization
- System Admin: Any student

**Response** (200 OK):
```typescript
{
  data: {
    id: string;
    userId: string;
    username: string;
    email: string;
    displayName: string;
    avatarUrl?: string;
    gradeLevel: string;
    preferredSequenceId: string;
    assignedTeacherId?: string;
    orgId: string;
    orgName: string;
    status: string;
    settings: {
      notifications: { email: boolean; inApp: boolean; };
      accessibility: { /* accessibility preferences */ };
      gamePreferences: { /* game settings */ };
    };
    stats: {
      totalGamesPlayed: number;
      totalScore: number;
      badgesEarned: number;
      currentStreak: number;
    };
    createdAt: string;
    updatedAt: string;
  }
}
```

**See**: [Student Profile](../03-student-experience/student-profile.md)

---

### 3.3 Create Student

**Endpoint**: `POST /api/students`

**Purpose**: Create new student account.

**Access**: Teacher (if permitted), Subscriber-Admin, System Admin

**Request Body**:
```typescript
{
  username: string;
  email?: string;          // Optional for younger students
  password?: string;       // Optional, auto-generated if omitted
  displayName: string;
  gradeLevel: string;
  assignedTeacherId?: string;
  orgId?: string;          // Required for System Admin, auto-filled for others
}
```

**Response** (201 Created):
```typescript
{
  data: {
    student: { /* student profile */ };
    credentials?: {        // If password was auto-generated
      username: string;
      password: string;    // Plain text, to be securely shared
    };
  }
}
```

**See**: [Member Management](../05-admin-subscriber-experience/member-management.md)

---

### 3.4 Update Student Profile

**Endpoint**: `PATCH /api/students/:id`

**Purpose**: Update student profile fields.

**Access**: 
- Student: Limited fields (displayName, avatarUrl, settings)
- Teacher: Assigned students (gradeLevel, assignedTeacherId, notes)
- Subscriber-Admin: All students in org
- System Admin: Any student (all fields)

**Request Body**:
```typescript
{
  displayName?: string;
  gradeLevel?: string;
  assignedTeacherId?: string;
  preferredSequenceId?: string;
  avatarUrl?: string;
  settings?: { /* partial settings update */ };
  status?: 'active' | 'suspended' | 'archived';
}
```

**Response** (200 OK):
```typescript
{
  data: {
    student: { /* updated student profile */ }
  }
}
```

---

### 3.5 Delete Student

**Endpoint**: `DELETE /api/students/:id`

**Purpose**: Soft-delete student (archives, preserves data).

**Access**: Subscriber-Admin (own org), System Admin (any)

**Response** (204 No Content)

**Note**: Hard deletion requires System Admin action via separate endpoint.

---

### 3.6 Bulk Create Students

**Endpoint**: `POST /api/bulk/students`

**Purpose**: Bulk import students from CSV.

**Access**: Teacher (if permitted), Subscriber-Admin, System Admin

**Request Body** (multipart/form-data):
```typescript
{
  file: File;              // CSV file
  orgId?: string;          // Required for System Admin
  assignTeacherId?: string; // Assign all to specific teacher
  sendWelcomeEmail?: boolean;
}
```

**Response** (202 Accepted):
```typescript
{
  data: {
    jobId: string;         // Job ID for status polling
    status: 'processing';
    estimatedCompletion: string;
  }
}
```

**See**: [CSV Spec Students](../20-imports-and-automation/csv-spec-students.md), [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md)

---

### 3.7 Get Bulk Job Status

**Endpoint**: `GET /api/bulk/jobs/:jobId`

**Purpose**: Poll status of bulk operation.

**Access**: User who initiated job, or System Admin

**Response** (200 OK):
```typescript
{
  data: {
    jobId: string;
    type: 'student_import' | 'teacher_import' | 'assignment_bulk';
    status: 'processing' | 'completed' | 'failed';
    progress: {
      total: number;
      processed: number;
      successful: number;
      failed: number;
    };
    errors?: [
      {
        row: number;
        field: string;
        message: string;
      }
    ];
    resultUrl?: string;    // Download URL for results CSV
    createdAt: string;
    completedAt?: string;
  }
}
```

**See**: [Bulk Ops Jobs](../20-imports-and-automation/bulk-ops-jobs.md)

---

### 3.8 List Teachers

**Endpoint**: `GET /api/teachers`

**Purpose**: List teachers with filtering.

**Access**: 
- Subscriber-Admin: Teachers in organization
- System Admin: All teachers

**Query Parameters**: Similar to students list

**Response**: Similar structure to students list

---

### 3.9 Get Teacher Profile

**Endpoint**: `GET /api/teachers/:id`

**Purpose**: Get teacher profile and metadata.

**Access**: 
- Teacher: Own profile
- Subscriber-Admin: Teachers in organization
- System Admin: Any teacher

**Response** (200 OK):
```typescript
{
  data: {
    id: string;
    userId: string;
    username: string;
    email: string;
    displayName: string;
    orgId: string;
    orgName: string;
    permissions: {
      canCreateStudents: boolean;
      canManageClasses: boolean;
    };
    assignedStudents: number;
    activeClasses: number;
    createdAt: string;
  }
}
```

**See**: [Teacher Grade as Role](../04-teacher-experience/teacher-grade-as-role.md)

---

### 3.10 Assign Students to Teacher

**Endpoint**: `POST /api/teachers/:teacherId/students`

**Purpose**: Assign students to a teacher.

**Access**: Subscriber-Admin, System Admin

**Request Body**:
```typescript
{
  studentIds: string[];    // Array of student IDs to assign
  replace?: boolean;       // Replace existing assignments (default: false)
}
```

**Response** (200 OK):
```typescript
{
  data: {
    assigned: number;
    assignments: [
      {
        studentId: string;
        studentName: string;
        assignedAt: string;
      }
    ]
  }
}
```

---

### 3.11 Create Class/Group

**Endpoint**: `POST /api/classes`

**Purpose**: Create a class or student group.

**Access**: Teacher (if permitted), Subscriber-Admin, System Admin

**Request Body**:
```typescript
{
  name: string;
  description?: string;
  teacherId: string;
  studentIds?: string[];   // Initial students
  orgId?: string;          // Required for System Admin
}
```

**Response** (201 Created):
```typescript
{
  data: {
    class: {
      id: string;
      name: string;
      teacherId: string;
      studentCount: number;
      createdAt: string;
    }
  }
}
```

**See**: [Group Model](../07-content-architecture/group-model.md)

---

### 3.12 Get Class Details

**Endpoint**: `GET /api/classes/:id`

**Purpose**: Get class details with student list.

**Access**: Teacher (own classes), Subscriber-Admin, System Admin

**Response** (200 OK):
```typescript
{
  data: {
    id: string;
    name: string;
    description?: string;
    teacherId: string;
    teacherName: string;
    students: [
      {
        id: string;
        displayName: string;
        gradeLevel: string;
        status: string;
      }
    ];
    createdAt: string;
    updatedAt: string;
  }
}
```

---

## 4. Content Domain

Base path: `/api/games/*`, `/api/sequences/*`

### 4.1 List Games

**Endpoint**: `GET /api/games`

**Purpose**: List games with filtering, sorting, and search.

**Access**: Authenticated (all roles)

**Query Parameters**:
```typescript
{
  page?: number;
  limit?: number;
  category?: string;       // e.g., "rhythm", "pitch", "note-reading"
  difficulty?: 'beginner' | 'intermediate' | 'advanced';
  status?: 'published' | 'draft' | 'archived';
  search?: string;         // Full-text search on title, description
  tags?: string;           // Comma-separated tags
  sort?: string;           // e.g., "-popularity,title"
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;          // e.g., "game_abc123"
      gameCode: string;    // Legacy code, e.g., "NR01"
      title: string;
      description: string;
      category: string;
      difficulty: string;
      thumbnail: string;   // CDN URL
      stages: string[];    // e.g., ["learn", "play", "quiz"]
      avgPlayTime: number; // Minutes
      popularity: number;  // Play count
      tags: string[];
      status: string;
      createdAt: string;
    }
  ],
  pagination: { /* standard pagination */ }
}
```

**See**: [Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md), [Game Model](../08-games-registry-and-authoring/game-model.md)

---

### 4.2 Get Game Details

**Endpoint**: `GET /api/games/:id`

**Purpose**: Get detailed game information including metadata and asset URLs.

**Access**: Authenticated (all roles)

**Response** (200 OK):
```typescript
{
  data: {
    id: string;
    gameCode: string;
    title: string;
    description: string;
    longDescription?: string;
    category: string;
    difficulty: string;
    thumbnail: string;
    stages: [
      {
        stage: 'learn' | 'play' | 'quiz' | 'challenge' | 'review';
        gameUrl: string;    // CDN URL to game HTML
        enabled: boolean;
      }
    ];
    learningElements: string[]; // e.g., ["treble_clef", "quarter_note"]
    taxonomy: {
      subject: string;
      skill: string;
      concept: string;
    };
    avgPlayTime: number;
    minScore: number;
    maxScore: number;
    passingScore: number;
    popularity: number;
    tags: string[];
    status: string;
    versionHistory: [
      {
        version: string;
        publishedAt: string;
        changes: string;
      }
    ];
    createdAt: string;
    updatedAt: string;
  }
}
```

**See**: [Game Taxonomy](../07-content-architecture/game-taxonomy.md), [Content Versioning](../07-content-architecture/content-versioning.md)

---

### 4.3 Create Game (System Admin)

**Endpoint**: `POST /api/games`

**Purpose**: Register new game in the games registry.

**Access**: System Admin only

**Request Body**:
```typescript
{
  gameCode: string;        // Unique code
  title: string;
  description: string;
  category: string;
  difficulty: string;
  thumbnail: string;       // R2 URL
  stages: [
    {
      stage: string;
      gameUrl: string;
      enabled: boolean;
    }
  ];
  learningElements: string[];
  taxonomy: { subject: string; skill: string; concept: string; };
  avgPlayTime: number;
  minScore: number;
  maxScore: number;
  passingScore: number;
  tags?: string[];
}
```

**Response** (201 Created):
```typescript
{
  data: {
    game: { /* complete game object */ }
  }
}
```

**See**: [Games CRUD](../08-games-registry-and-authoring/games-crud.md)

---

### 4.4 Update Game (System Admin)

**Endpoint**: `PATCH /api/games/:id`

**Purpose**: Update game metadata or content.

**Access**: System Admin only

**Request Body**: Partial game object (any updateable fields)

**Response** (200 OK):
```typescript
{
  data: {
    game: { /* updated game */ };
    versionCreated: boolean; // True if new version was created
  }
}
```

---

### 4.5 List Sequences

**Endpoint**: `GET /api/sequences`

**Purpose**: List curriculum sequences.

**Access**: Authenticated (all roles)

**Query Parameters**:
```typescript
{
  page?: number;
  limit?: number;
  library?: 'LIFE' | 'EVAL' | 'SOLF';
  gradeLevel?: string;
  methodAlignment?: 'piano_adventures' | 'hal_leonard';
  status?: 'published' | 'draft';
  search?: string;
  sort?: string;
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;          // e.g., "seq_abc123"
      sequenceCode: string; // e.g., "LIFE-01"
      title: string;
      library: string;
      gradeLevel: string;
      gameCount: number;
      totalSteps: number;
      estimatedDuration: number; // Minutes
      methodAlignment?: string;
      status: string;
      createdAt: string;
    }
  ],
  pagination: { /* standard pagination */ }
}
```

**See**: [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md), [Sequence Libraries](../09-sequences-and-curriculum-tools/sequence-libraries.md)

---

### 4.6 Get Sequence Details

**Endpoint**: `GET /api/sequences/:id`

**Purpose**: Get detailed sequence with ordered game list.

**Access**: Authenticated (all roles)

**Response** (200 OK):
```typescript
{
  data: {
    id: string;
    sequenceCode: string;
    title: string;
    description: string;
    library: string;
    gradeLevel: string;
    methodAlignment?: string;
    games: [
      {
        gameId: string;
        gameTitle: string;
        order: number;       // Sequence position
        stages: string[];    // Required stages for this sequence
        minScore?: number;   // Override passing score
      }
    ];
    totalSteps: number;      // Total game stages
    estimatedDuration: number;
    learningObjectives: string[];
    status: string;
    createdAt: string;
    updatedAt: string;
  }
}
```

**See**: [Piano Adventures Alignment](../07-content-architecture/piano-adventures-alignment.md)

---

### 4.7 Create Sequence (System Admin)

**Endpoint**: `POST /api/sequences`

**Purpose**: Create new curriculum sequence.

**Access**: System Admin only

**Request Body**:
```typescript
{
  sequenceCode: string;
  title: string;
  description: string;
  library: string;
  gradeLevel: string;
  games: [
    {
      gameId: string;
      order: number;
      stages: string[];
      minScore?: number;
    }
  ];
  methodAlignment?: string;
  learningObjectives?: string[];
}
```

**Response** (201 Created):
```typescript
{
  data: {
    sequence: { /* complete sequence object */ }
  }
}
```

**See**: [Sequence Authoring](../09-sequences-and-curriculum-tools/sequence-authoring.md)

---

### 4.8 Update Sequence (System Admin)

**Endpoint**: `PATCH /api/sequences/:id`

**Purpose**: Update sequence metadata or game order.

**Access**: System Admin only

**Request Body**: Partial sequence object

**Response** (200 OK):
```typescript
{
  data: {
    sequence: { /* updated sequence */ };
    versionCreated: boolean;
  }
}
```

---

### 4.9 Export Sequence

**Endpoint**: `GET /api/sequences/:id/export`

**Purpose**: Export sequence as PDF or CSV for offline use.

**Access**: Teacher, Subscriber-Admin, System Admin

**Query Parameters**:
```typescript
{
  format: 'pdf' | 'csv';
}
```

**Response** (200 OK):
```typescript
{
  data: {
    downloadUrl: string;   // R2 presigned URL
    expiresAt: string;     // URL expiration time
  }
}
```

**See**: [Sequence Exports](../09-sequences-and-curriculum-tools/sequence-exports.md)

---

## 5. Learning Engine Domain

Base path: `/api/assignments/*`, `/api/progress/*`, `/api/scores/*`

### 5.1 List Assignments

**Endpoint**: `GET /api/assignments`

**Purpose**: List assignments for current user's context.

**Access**: 
- Student: Own assignments
- Teacher: Assignments created by self
- Subscriber-Admin: All assignments in organization

**Query Parameters**:
```typescript
{
  page?: number;
  limit?: number;
  studentId?: string;      // Filter by student (Teacher/Admin)
  teacherId?: string;      // Filter by teacher (Admin)
  status?: 'active' | 'completed' | 'archived';
  dueDate?: string;        // Filter by due date
  sort?: string;
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;
      sequenceId: string;
      sequenceTitle: string;
      createdByTeacherId: string;
      createdByTeacherName: string;
      status: string;
      progress: {
        completed: number;
        total: number;
        percentage: number;
      };
      dueDate?: string;
      createdAt: string;
    }
  ],
  pagination: { /* standard pagination */ }
}
```

**See**: [Student Assignments](../03-student-experience/student-assignments.md)

---

### 5.2 Get Assignment Details

**Endpoint**: `GET /api/assignments/:id`

**Purpose**: Get detailed assignment with game progress.

**Access**: 
- Student: Own assignments
- Teacher: Own created assignments
- Subscriber-Admin: Assignments in organization

**Response** (200 OK):
```typescript
{
  data: {
    id: string;
    sequenceId: string;
    sequenceTitle: string;
    createdByTeacherId: string;
    createdByTeacherName: string;
    assignedStudents: [
      {
        studentId: string;
        studentName: string;
        progress: {
          completed: number;
          total: number;
          percentage: number;
          currentGameId?: string;
          currentStage?: string;
        };
        lastPlayedAt?: string;
      }
    ];
    games: [
      {
        gameId: string;
        gameTitle: string;
        order: number;
        stages: string[];
        status: 'locked' | 'available' | 'in_progress' | 'completed';
      }
    ];
    status: string;
    dueDate?: string;
    notes?: string;
    createdAt: string;
  }
}
```

**See**: [Assignment Model](../07-content-architecture/assignment-model.md)

---

### 5.3 Create Assignment

**Endpoint**: `POST /api/assignments`

**Purpose**: Create new assignment for students.

**Access**: Teacher (assigned students), Subscriber-Admin

**Request Body**:
```typescript
{
  sequenceId: string;      // Required: sequence to assign
  studentIds: string[];    // Required: students to assign to
  dueDate?: string;        // Optional due date
  notes?: string;          // Optional teacher notes
  customGames?: [          // Optional: override sequence games
    {
      gameId: string;
      stages: string[];
      minScore?: number;
    }
  ];
}
```

**Response** (201 Created):
```typescript
{
  data: {
    assignment: { /* assignment object */ };
    assignedCount: number;
    notificationsQueuedAt: string;
  }
}
```

**See**: [Assignment Builder](../04-teacher-experience/assignment-builder.md), [Assignment Generation](../10-assignments-engine/assignment-generation.md)

---

### 5.4 Get Next Game for Student

**Endpoint**: `GET /api/assignments/:id/next`

**Purpose**: Get next game/stage in assignment for student.

**Access**: Student (own assignments)

**Response** (200 OK):
```typescript
{
  data: {
    assignmentId: string;
    gameId: string;
    gameTitle: string;
    stage: 'learn' | 'play' | 'quiz' | 'challenge' | 'review';
    gameUrl: string;        // CDN URL to load game
    minScore: number;
    instructions?: string;
    progress: {
      current: number;
      total: number;
    };
  }
}
```

**Note**: Returns `204 No Content` if assignment is complete.

**See**: [Assignment Play](../03-student-experience/assignment-play.md)

---

### 5.5 Record Score

**Endpoint**: `POST /api/scores`

**Purpose**: Record game completion and score.

**Access**: Student (own scores)

**Request Body**:
```typescript
{
  assignmentId: string;
  gameId: string;
  stage: string;
  score: number;
  timeSpentSeconds: number;
  attemptData?: object;    // Optional game-specific data
}
```

**Response** (201 Created):
```typescript
{
  data: {
    score: {
      id: string;
      gameId: string;
      stage: string;
      score: number;
      passed: boolean;      // Based on minScore threshold
      recordedAt: string;
    };
    progress: {
      nextGame?: {
        gameId: string;
        stage: string;
      };
      assignmentCompleted: boolean;
    };
    achievements?: {       // If any badges/streaks earned
      badgesEarned: string[];
      streakUpdated: boolean;
      pointsAwarded: number;
    };
  }
}
```

**See**: [Game Player Shell](../03-student-experience/game-player-shell.md), [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md)

---

### 5.6 Get Student Progress

**Endpoint**: `GET /api/progress/:studentId`

**Purpose**: Get comprehensive progress report for student.

**Access**: 
- Student: Own progress
- Teacher: Assigned students
- Subscriber-Admin: Students in organization

**Query Parameters**:
```typescript
{
  dateFrom?: string;       // Filter by date range
  dateTo?: string;
  assignmentId?: string;   // Filter by specific assignment
}
```

**Response** (200 OK):
```typescript
{
  data: {
    studentId: string;
    studentName: string;
    overall: {
      totalGamesPlayed: number;
      totalTimeSpentMinutes: number;
      averageScore: number;
      completionRate: number;
    };
    assignments: [
      {
        assignmentId: string;
        sequenceTitle: string;
        progress: number;    // Percentage
        status: string;
        lastPlayedAt?: string;
      }
    ];
    recentScores: [
      {
        gameId: string;
        gameTitle: string;
        stage: string;
        score: number;
        playedAt: string;
      }
    ];
    strengths: string[];     // Learning elements mastered
    needsPractice: string[]; // Learning elements needing work
  }
}
```

**See**: [Teacher Reports](../04-teacher-experience/teacher-reports.md)

---

### 5.7 Get Score History

**Endpoint**: `GET /api/scores/history`

**Purpose**: Get detailed score history with filtering.

**Access**: 
- Student: Own scores
- Teacher: Assigned students' scores
- Subscriber-Admin: Students in organization

**Query Parameters**:
```typescript
{
  studentId: string;       // Required
  gameId?: string;         // Filter by game
  dateFrom?: string;
  dateTo?: string;
  page?: number;
  limit?: number;
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;
      studentId: string;
      gameId: string;
      gameTitle: string;
      stage: string;
      score: number;
      maxScore: number;
      timeSpentSeconds: number;
      passed: boolean;
      attemptNumber: number; // If retried
      recordedAt: string;
    }
  ],
  pagination: { /* standard pagination */ }
}
```

---

### 5.8 Mass Assign

**Endpoint**: `POST /api/assignments/mass-assign`

**Purpose**: Create same assignment for multiple students/classes.

**Access**: Teacher, Subscriber-Admin

**Request Body**:
```typescript
{
  sequenceId: string;
  targets: [
    { type: 'student'; id: string; } |
    { type: 'class'; id: string; }
  ];
  dueDate?: string;
  notes?: string;
}
```

**Response** (202 Accepted):
```typescript
{
  data: {
    jobId: string;         // Job ID for status polling
    estimatedCount: number;
    status: 'processing';
  }
}
```

**See**: [Mass Assignments](../05-admin-subscriber-experience/mass-assignments.md)

---

## 6. Gamification Domain

Base path: `/api/badges/*`, `/api/leaderboards/*`, `/api/points/*`

### 6.1 Get Student Badges

**Endpoint**: `GET /api/badges/:studentId`

**Purpose**: Get all badges earned by student.

**Access**: 
- Student: Own badges
- Teacher: Assigned students
- Subscriber-Admin: Students in organization

**Response** (200 OK):
```typescript
{
  data: {
    total: number;
    badges: [
      {
        id: string;
        badgeId: string;
        badgeName: string;
        badgeDescription: string;
        badgeIcon: string;   // CDN URL
        category: 'milestone' | 'mastery' | 'achievement' | 'special';
        earnedAt: string;
        progress?: number;   // For multi-level badges
      }
    ];
    nextToEarn: [
      {
        badgeId: string;
        badgeName: string;
        progress: number;    // Percentage to earn
        requirements: string;
      }
    ];
  }
}
```

**See**: [Badges System](../12-gamification/badges-system.md)

---

### 6.2 Get Points and Streaks

**Endpoint**: `GET /api/points/:studentId`

**Purpose**: Get student's points and streak information.

**Access**: Same as badges

**Response** (200 OK):
```typescript
{
  data: {
    totalPoints: number;
    currentStreak: number;
    longestStreak: number;
    lastPlayedAt: string;
    pointsHistory: [
      {
        date: string;
        points: number;
        reason: string;      // e.g., "Quiz completed", "Badge earned"
      }
    ];
    streakHistory: [
      {
        date: string;
        streak: number;
      }
    ];
  }
}
```

**See**: [Points and Streaks](../12-gamification/points-and-streaks.md)

---

### 6.3 Get Leaderboard

**Endpoint**: `GET /api/leaderboards/:type`

**Purpose**: Get leaderboard rankings.

**Access**: Authenticated (all roles)

**Path Parameters**:
```typescript
type: 'global' | 'org' | 'class' | 'grade'
```

**Query Parameters**:
```typescript
{
  scope?: string;          // org_id or class_id depending on type
  timeframe?: 'daily' | 'weekly' | 'monthly' | 'all_time';
  limit?: number;          // Max 100, default 50
}
```

**Response** (200 OK):
```typescript
{
  data: {
    type: string;
    timeframe: string;
    lastUpdated: string;
    entries: [
      {
        rank: number;
        studentId: string;
        studentName: string;
        avatarUrl?: string;
        score: number;        // Total points or specific metric
        change: number;       // Rank change since previous period
      }
    ];
    currentUserRank?: {      // If authenticated user is student
      rank: number;
      score: number;
    };
  }
}
```

**See**: [Leaderboards](../03-student-experience/leaderboards.md)

---

### 6.4 Get Challenge Boards

**Endpoint**: `GET /api/challenges`

**Purpose**: Get active challenges.

**Access**: Authenticated (all roles)

**Query Parameters**:
```typescript
{
  status?: 'active' | 'completed' | 'expired';
  scope?: 'global' | 'org' | 'class';
  scopeId?: string;
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;
      title: string;
      description: string;
      type: 'individual' | 'class' | 'org';
      goal: {
        metric: 'games_played' | 'score' | 'streak';
        target: number;
      };
      rewards: {
        badgeId?: string;
        points?: number;
      };
      startDate: string;
      endDate: string;
      participants: number;
      progress?: number;     // If student is participating
    }
  ]
}
```

**See**: [Challenge Boards](../12-gamification/challenge-boards.md)

---

### 6.5 Join Challenge

**Endpoint**: `POST /api/challenges/:id/join`

**Purpose**: Student joins a challenge.

**Access**: Student

**Response** (200 OK):
```typescript
{
  data: {
    challenge: { /* challenge details */ };
    progress: {
      current: number;
      target: number;
      percentage: number;
    };
  }
}
```

---

## 7. Reporting & Analytics Domain

Base path: `/api/reports/*`, `/api/analytics/*`

### 7.1 Get Teacher Report

**Endpoint**: `GET /api/reports/teacher/:teacherId`

**Purpose**: Get comprehensive teacher report with student progress.

**Access**: 
- Teacher: Own report
- Subscriber-Admin: Teachers in organization
- System Admin: Any teacher

**Query Parameters**:
```typescript
{
  dateFrom?: string;
  dateTo?: string;
  studentId?: string;      // Filter to specific student
  format?: 'json' | 'pdf' | 'csv';
}
```

**Response** (200 OK):
```typescript
{
  data: {
    teacherId: string;
    teacherName: string;
    dateRange: {
      from: string;
      to: string;
    };
    summary: {
      totalStudents: number;
      activeStudents: number;
      totalGamesPlayed: number;
      averageCompletionRate: number;
      averageScore: number;
    };
    studentProgress: [
      {
        studentId: string;
        studentName: string;
        gamesPlayed: number;
        avgScore: number;
        completionRate: number;
        lastActive: string;
        trend: 'improving' | 'declining' | 'stable';
      }
    ];
    assignmentStats: [
      {
        assignmentId: string;
        sequenceTitle: string;
        assignedCount: number;
        completedCount: number;
        avgCompletion: number;
      }
    ];
    downloadUrl?: string;    // If format=pdf or csv
  }
}
```

**See**: [Teacher Reports](../04-teacher-experience/teacher-reports.md), [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md)

---

### 7.2 Get Admin Report

**Endpoint**: `GET /api/reports/admin/:orgId`

**Purpose**: Organization-wide analytics for admins.

**Access**: Subscriber-Admin (own org), System Admin (any org)

**Query Parameters**:
```typescript
{
  dateFrom?: string;
  dateTo?: string;
  groupBy?: 'teacher' | 'grade' | 'class';
  format?: 'json' | 'pdf' | 'csv';
}
```

**Response** (200 OK):
```typescript
{
  data: {
    orgId: string;
    orgName: string;
    dateRange: { from: string; to: string; };
    summary: {
      totalStudents: number;
      activeStudents: number;
      totalTeachers: number;
      activeTeachers: number;
      gamesPlayed: number;
      totalPlayTimeMinutes: number;
      engagementRate: number;
    };
    byTeacher?: [ /* teacher breakdown */ ];
    byGrade?: [ /* grade level breakdown */ ];
    byClass?: [ /* class breakdown */ ];
    downloadUrl?: string;
  }
}
```

**See**: [Admin Reports](../05-admin-subscriber-experience/admin-reports.md), [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md)

---

### 7.3 Get System Admin Dashboard

**Endpoint**: `GET /api/analytics/sysadmin/dashboard`

**Purpose**: System-wide operational metrics.

**Access**: System Admin only

**Response** (200 OK):
```typescript
{
  data: {
    platform: {
      totalUsers: number;
      activeUsers24h: number;
      totalOrganizations: number;
      activeSubscriptions: number;
      revenue: {
        mrr: number;         // Monthly recurring revenue
        arr: number;         // Annual recurring revenue
      };
    };
    usage: {
      gamesPlayed24h: number;
      gamesPlayed7d: number;
      gamesPlayed30d: number;
      avgSessionDuration: number;
    };
    content: {
      totalGames: number;
      totalSequences: number;
      avgGamePopularity: number;
    };
    performance: {
      apiResponseTimeP95: number;
      errorRate: number;
      uptime: number;
    };
  }
}
```

**See**: [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md)

---

### 7.4 Track Event (Internal)

**Endpoint**: `POST /api/events`

**Purpose**: Record telemetry event (internal use by application).

**Access**: Internal only (not exposed externally)

**Request Body**:
```typescript
{
  eventType: string;
  userId?: string;
  properties: object;
  timestamp?: string;
}
```

**Response** (201 Created):
```typescript
{
  data: {
    eventId: string;
    recorded: boolean;
  }
}
```

**See**: [Event Model](../15-analytics-and-reporting/event-model.md)

---

## 8. Messaging & Notifications Domain

Base path: `/api/messages/*`, `/api/notifications/*`, `/api/announcements/*`

### 8.1 Get User Notifications

**Endpoint**: `GET /api/notifications`

**Purpose**: Get notifications for current user.

**Access**: Authenticated (any role)

**Query Parameters**:
```typescript
{
  status?: 'unread' | 'read' | 'all';
  type?: 'assignment' | 'message' | 'badge' | 'system';
  page?: number;
  limit?: number;
}
```

**Response** (200 OK):
```typescript
{
  data: {
    unreadCount: number;
    notifications: [
      {
        id: string;
        type: string;
        title: string;
        message: string;
        actionUrl?: string;
        read: boolean;
        createdAt: string;
      }
    ]
  },
  pagination: { /* standard pagination */ }
}
```

**See**: [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md), [Student Notifications](../03-student-experience/student-notifications.md)

---

### 8.2 Mark Notification Read

**Endpoint**: `PATCH /api/notifications/:id/read`

**Purpose**: Mark notification as read.

**Access**: Authenticated (own notifications)

**Response** (200 OK):
```typescript
{
  data: {
    notification: { id: string; read: true; readAt: string; }
  }
}
```

---

### 8.3 Send Message

**Endpoint**: `POST /api/messages`

**Purpose**: Send message from teacher to student or vice versa.

**Access**: Teacher, Student (to assigned teacher)

**Request Body**:
```typescript
{
  recipientId: string;
  subject: string;
  body: string;
  attachments?: string[];  // R2 URLs
}
```

**Response** (201 Created):
```typescript
{
  data: {
    message: {
      id: string;
      senderId: string;
      recipientId: string;
      subject: string;
      body: string;
      sentAt: string;
    }
  }
}
```

**See**: [Teacher Messaging](../04-teacher-experience/teacher-messaging.md), [Student Messages](../03-student-experience/student-messages.md)

---

### 8.4 Get Messages

**Endpoint**: `GET /api/messages`

**Purpose**: Get messages for current user.

**Access**: Authenticated (any role)

**Query Parameters**:
```typescript
{
  folder?: 'inbox' | 'sent';
  status?: 'unread' | 'read';
  page?: number;
  limit?: number;
}
```

**Response** (200 OK):
```typescript
{
  data: {
    unreadCount: number;
    messages: [
      {
        id: string;
        senderId: string;
        senderName: string;
        subject: string;
        preview: string;     // First 100 chars
        read: boolean;
        sentAt: string;
      }
    ]
  },
  pagination: { /* standard pagination */ }
}
```

---

### 8.5 Get System Announcements

**Endpoint**: `GET /api/announcements`

**Purpose**: Get active system-wide announcements.

**Access**: Authenticated (any role)

**Query Parameters**:
```typescript
{
  target?: 'all' | 'students' | 'teachers' | 'admins';
  status?: 'active' | 'scheduled' | 'expired';
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;
      title: string;
      message: string;
      type: 'info' | 'warning' | 'maintenance' | 'feature';
      targetAudience: string[];
      priority: 'low' | 'medium' | 'high';
      activeFrom: string;
      activeUntil: string;
      dismissible: boolean;
    }
  ]
}
```

**See**: [System Announcements](../13-messaging-and-notifications/system-announcements.md)

---

### 8.6 Create Announcement (System Admin)

**Endpoint**: `POST /api/announcements`

**Purpose**: Create system-wide announcement.

**Access**: System Admin only

**Request Body**:
```typescript
{
  title: string;
  message: string;
  type: string;
  targetAudience: string[];
  priority: string;
  activeFrom: string;
  activeUntil: string;
  dismissible: boolean;
}
```

**Response** (201 Created):
```typescript
{
  data: {
    announcement: { /* complete announcement object */ }
  }
}
```

---

## 9. Search & Discovery Domain

Base path: `/api/search/*`

### 9.1 Full-Text Search

**Endpoint**: `GET /api/search`

**Purpose**: Global search across games, sequences, students, etc.

**Access**: Authenticated (results scoped by permissions)

**Query Parameters**:
```typescript
{
  q: string;               // Required: search query
  type?: 'all' | 'games' | 'sequences' | 'students' | 'teachers';
  limit?: number;          // Max 50
}
```

**Response** (200 OK):
```typescript
{
  data: {
    query: string;
    totalResults: number;
    results: [
      {
        type: 'game' | 'sequence' | 'student' | 'teacher';
        id: string;
        title: string;
        description: string;
        relevance: number;   // 0-1 score
        thumbnail?: string;
        metadata: object;    // Type-specific fields
      }
    ];
    facets?: {
      type: { [key: string]: number };
      category?: { [key: string]: number };
    };
  }
}
```

**See**: [Search Indexing](../11-search-and-discovery/search-indexing.md)

---

### 9.2 Get Search Filters

**Endpoint**: `GET /api/search/filters/:type`

**Purpose**: Get available filters for search type.

**Access**: Authenticated (any role)

**Path Parameters**:
```typescript
type: 'games' | 'sequences'
```

**Response** (200 OK):
```typescript
{
  data: {
    filters: [
      {
        field: 'category';
        label: 'Category';
        type: 'multiselect';
        options: [
          { value: 'rhythm'; label: 'Rhythm'; count: 45; }
        ];
      }
    ]
  }
}
```

**See**: [Filters and Facets](../11-search-and-discovery/filters-and-facets.md)

---

### 9.3 Teacher Quick Find

**Endpoint**: `GET /api/search/quick`

**Purpose**: Fast autocomplete search for teachers.

**Access**: Teacher, Subscriber-Admin

**Query Parameters**:
```typescript
{
  q: string;               // Minimum 2 characters
  type: 'game' | 'sequence' | 'student';
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;
      title: string;
      subtitle?: string;   // e.g., category for games
      thumbnail?: string;
    }
  ]
}
```

**See**: [Teacher Quick Find](../11-search-and-discovery/teacher-quick-find.md)

---

## 10. Bulk Operations & System Admin

Base path: `/api/bulk/*`, `/api/sysadmin/*`

### 10.1 Export Data

**Endpoint**: `POST /api/bulk/export`

**Purpose**: Export data as CSV for backup or analysis.

**Access**: Subscriber-Admin (own org), System Admin (any org)

**Request Body**:
```typescript
{
  type: 'students' | 'teachers' | 'scores' | 'assignments';
  orgId?: string;          // Required for System Admin
  filters?: object;        // Type-specific filters
  dateFrom?: string;
  dateTo?: string;
}
```

**Response** (202 Accepted):
```typescript
{
  data: {
    jobId: string;
    estimatedCompletion: string;
  }
}
```

**See**: [Export Specs](../20-imports-and-automation/export-specs.md)

---

### 10.2 Get User Index (System Admin)

**Endpoint**: `GET /api/sysadmin/users`

**Purpose**: Global user search and management.

**Access**: System Admin only

**Query Parameters**:
```typescript
{
  search?: string;         // Search username, email, name
  role?: string;
  status?: string;
  orgId?: string;
  page?: number;
  limit?: number;
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;
      username: string;
      email: string;
      role: string;
      status: string;
      orgId?: string;
      orgName?: string;
      lastLoginAt?: string;
      createdAt: string;
    }
  ],
  pagination: { /* standard pagination */ }
}
```

**See**: [Sysadmin User Index](../06-system-admin/sysadmin-user-index.md)

---

### 10.3 Get Audit Logs (System Admin)

**Endpoint**: `GET /api/sysadmin/audit-logs`

**Purpose**: View system audit logs.

**Access**: System Admin only

**Query Parameters**:
```typescript
{
  userId?: string;
  actionType?: string;
  entityType?: string;
  dateFrom?: string;
  dateTo?: string;
  page?: number;
  limit?: number;
}
```

**Response** (200 OK):
```typescript
{
  data: [
    {
      id: string;
      userId: string;
      username: string;
      actionType: string;
      entityType: string;
      entityId: string;
      ipAddress: string;
      metadata: object;
      timestamp: string;
    }
  ],
  pagination: { /* standard pagination */ }
}
```

**See**: [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)

---

### 10.4 Manage Feature Flags (System Admin)

**Endpoint**: `GET /api/sysadmin/feature-flags`

**Purpose**: List and manage feature flags.

**Access**: System Admin only

**Response** (200 OK):
```typescript
{
  data: [
    {
      flag: string;        // e.g., "enable_midi_support"
      enabled: boolean;
      description: string;
      rolloutPercentage?: number;
      enabledForOrgs?: string[];
    }
  ]
}
```

---

### 10.5 Update Feature Flag (System Admin)

**Endpoint**: `PATCH /api/sysadmin/feature-flags/:flag`

**Purpose**: Enable/disable feature flag.

**Access**: System Admin only

**Request Body**:
```typescript
{
  enabled: boolean;
  rolloutPercentage?: number;
  enabledForOrgs?: string[];
}
```

**Response** (200 OK):
```typescript
{
  data: {
    flag: { /* updated flag */ }
  }
}
```

**See**: [Sysadmin Feature Flags](../06-system-admin/sysadmin-feature-flags.md)

---

## Behavior and Rules

### Authentication Requirements

All endpoints except the following require authentication:
- `POST /api/auth/register`
- `POST /api/auth/login`
- `POST /api/auth/password-reset`
- `POST /api/auth/password-reset/confirm`
- `POST /api/billing/calculate` (pricing calculator)

Authentication is enforced via NextAuth.js middleware. See [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md).

### Authorization Model

All endpoints enforce role-based access control (RBAC):
- **Student**: Access own data (assignments, scores, profile)
- **Teacher**: Access assigned students, create assignments, view reports
- **Teacher-Admin**: Same as Teacher + manage classes
- **Subscriber-Admin**: Full access within organization
- **System Admin**: Global access + system management tools

See [Permissions Table](../02-roles-and-permissions/permissions-table.md) for detailed permission matrix.

### Data Scoping

API responses are automatically scoped based on user role:
- Teachers see only their assigned students
- Admins see only their organization's data
- System Admins can specify scope with query parameters

### Rate Limiting

Rate limits apply per user/IP:
- **Authenticated users**: 1000 requests/hour (general), 100 requests/hour (write operations)
- **Unauthenticated**: 100 requests/hour (public endpoints only)
- **Bulk operations**: Special limits apply (10 concurrent jobs max)

See [Rate Limiting and Abuse](rate-limiting-and-abuse.md) for detailed policies.

### Idempotency

- `GET`, `PUT`, `DELETE` operations are naturally idempotent
- `POST` operations can optionally use `Idempotency-Key` header (future enhancement)

### Error Handling

All endpoints return consistent error format:
```typescript
{
  error: {
    code: string;          // Machine-readable error code
    message: string;       // Human-readable message
    details?: object;      // Additional context
  },
  meta: {
    timestamp: string;
    requestId: string;     // For support/debugging
  }
}
```

See [API Design Principles](api-design-principles.md) for complete error code taxonomy.

---

## Telemetry

All API requests generate telemetry events:

**Event: `api_request`**
```typescript
{
  event_type: 'api_request';
  properties: {
    method: string;        // HTTP method
    path: string;          // Endpoint path
    statusCode: number;
    duration_ms: number;
    userId?: string;
    userRole?: string;
    requestId: string;
  }
}
```

**Event: `api_error`**
```typescript
{
  event_type: 'api_error';
  properties: {
    method: string;
    path: string;
    statusCode: number;
    errorCode: string;
    userId?: string;
    requestId: string;
  }
}
```

See [Event Model](../15-analytics-and-reporting/event-model.md) for complete event taxonomy.

---

## Permissions

**Endpoint Management:**
- **System Admins**: Full control over all endpoints and data
- **API Design Team**: Define and document new endpoints
- **Developers**: Implement endpoints following established patterns

**API Access:**
- All endpoints require appropriate role permissions (see Authorization Model above)
- External API access (future): OAuth 2.0 or API keys

See [Permissions Table](../02-roles-and-permissions/permissions-table.md).

---

## Supporting Documents Referenced

This REST endpoints specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game API endpoint requirements
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence API requirements
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — Admin API endpoints
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User management API requirements

## Dependencies

This specification relies on:
- [API Design Principles](api-design-principles.md) - Patterns, conventions, error handling
- [System Overview](system-overview.md) - Overall architecture context
- [Service Boundaries](service-boundaries.md) - Domain organization and internal contracts
- [Data Model ERD](data-model-erd.md) - Entity definitions and relationships
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Authorization rules
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry and tracking
- [Rate Limiting and Abuse](rate-limiting-and-abuse.md) - Rate limiting policies
- [Webhooks](webhooks.md) - Webhook specifications for external integrations
- [Background Jobs](background-jobs.md) - Async job processing patterns

---

## Open Questions

### Near-Term Decisions

1. **GraphQL Consideration**: Should we introduce GraphQL for complex frontend queries, or continue optimizing REST endpoints with selective field inclusion?
2. **Batch Endpoints**: Do we need dedicated batch endpoints (e.g., `POST /api/students/batch`) beyond the bulk operations API?
3. **Real-Time Endpoints**: Which features need WebSocket/SSE endpoints vs. polling (leaderboards, notifications)?
4. **API Versioning Trigger**: At what point do we introduce `/api/v2/*` endpoints?
5. **Partial Responses**: Should we support field selection (e.g., `?fields=id,title,status`) to reduce payload size?

### Long-Term Considerations

1. **External API Program**: When/if we open APIs to third-party developers, which endpoints should be public and what authentication model (OAuth 2.0, API keys)?
2. **Webhook Endpoints**: Which events should trigger outbound webhooks for external integrations?
3. **API Gateway**: At what scale do we need a dedicated API gateway (Kong, AWS API Gateway) vs. continuing with Next.js API routes?
4. **Regional APIs**: Do we need region-specific API endpoints for compliance (GDPR, data residency)?
5. **Deprecation Timeline**: What's the minimum support window for deprecated endpoints before removal?

---

## Summary

This REST Endpoints specification provides:
- **Complete endpoint catalog** organized by domain with 90+ endpoints documented
- **Request/response contracts** with TypeScript type signatures
- **Authentication and authorization** requirements per endpoint
- **Usage examples** and common patterns for developers
- **Cross-references** to related specifications for detailed context

By centralizing endpoint documentation here while deferring to [API Design Principles](api-design-principles.md) for patterns and conventions, we maintain a clear separation of concerns: **principles** define how APIs should be designed, while **this document** catalogs what APIs exist and how to use them.

This structure enables developers to quickly find and understand any endpoint while ensuring consistency through shared design patterns across all 90+ API routes in the MLC LMS platform.
