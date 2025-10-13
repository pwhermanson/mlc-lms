# API Design Principles

## Purpose

This document establishes the core principles, patterns, and conventions for designing RESTful APIs within the MLC LMS platform. These guidelines ensure consistency, maintainability, and developer experience across all API endpoints, enabling both internal frontend developers and potential future external integrations to work efficiently with the platform's API surface.

## Scope

**Included:**
- RESTful API design principles and conventions
- Request/response format standards
- Error handling patterns and status codes
- Authentication and authorization integration
- Versioning strategy
- Pagination, filtering, and sorting patterns
- Data validation and type safety
- API documentation requirements
- Rate limiting principles (reference to [Rate Limiting and Abuse](rate-limiting-and-abuse.md))
- Caching headers and strategies (reference to [Caching Strategy](caching-strategy.md))

**Excluded:**
- Specific API endpoint specifications (see [REST Endpoints](rest-endpoints.md))
- Database schema details (see [Data Model ERD](data-model-erd.md))
- Service boundaries and domain contracts (see [Service Boundaries](service-boundaries.md))
- External integration protocols (see [Integrations](../17-integrations/))
- Infrastructure and deployment details (see [System Overview](system-overview.md))

---

## Core Design Principles

### 1. RESTful Resource-Oriented Design

All APIs follow REST principles with resources as first-class entities:

**Resource Naming:**
- Use **plural nouns** for collections: `/api/students`, `/api/games`, `/api/assignments`
- Use **singular identifiers** for specific resources: `/api/students/:id`, `/api/games/:gameId`
- Use **nested resources** for relationships: `/api/teachers/:teacherId/students`, `/api/sequences/:seqId/games`
- Use **kebab-case** for multi-word resources: `/api/subscription-plans`, `/api/promo-codes`

**HTTP Methods:**
- `GET` - Retrieve resource(s) (read-only, idempotent)
- `POST` - Create new resource
- `PUT` - Replace entire resource (idempotent)
- `PATCH` - Partially update resource (preferred for updates)
- `DELETE` - Remove resource (idempotent)

**Example:**
```typescript
GET    /api/assignments          // List all assignments (user-scoped)
GET    /api/assignments/:id      // Get specific assignment
POST   /api/assignments          // Create new assignment
PATCH  /api/assignments/:id      // Update assignment
DELETE /api/assignments/:id      // Delete assignment
GET    /api/assignments/:id/progress  // Get assignment progress (sub-resource)
```

---

### 2. Consistent Request/Response Formats

#### Request Format

**JSON Body** (for POST, PUT, PATCH):
```json
{
  "sequenceId": "seq_12345",
  "studentIds": ["stu_001", "stu_002"],
  "dueDate": "2025-02-15T23:59:59Z",
  "notes": "Complete by end of week"
}
```

**Query Parameters** (for GET):
```
GET /api/games?category=rhythm&difficulty=beginner&page=2&limit=20&sort=-createdAt
```

**Type Safety:**
- All requests/responses have TypeScript type definitions in `/types/api/`
- Use Zod or similar for runtime validation
- Validate input at API route boundary before calling domain services

#### Response Format

**Success Response:**
```json
{
  "data": {
    "id": "asg_67890",
    "sequenceId": "seq_12345",
    "status": "active",
    "createdAt": "2025-01-27T10:00:00Z"
  },
  "meta": {
    "timestamp": "2025-01-27T10:00:00Z",
    "requestId": "req_abc123"
  }
}
```

**Collection Response (with pagination):**
```json
{
  "data": [
    { "id": "game_001", "title": "Note Reading A" },
    { "id": "game_002", "title": "Rhythm Basics" }
  ],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrevious": true
  },
  "meta": {
    "timestamp": "2025-01-27T10:00:00Z",
    "requestId": "req_def456"
  }
}
```

**Error Response:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input data",
    "details": [
      {
        "field": "dueDate",
        "message": "Due date must be in the future",
        "code": "DATE_IN_PAST"
      }
    ]
  },
  "meta": {
    "timestamp": "2025-01-27T10:00:00Z",
    "requestId": "req_ghi789"
  }
}
```

---

### 3. HTTP Status Codes

Use appropriate HTTP status codes consistently:

**Success Codes:**
- `200 OK` - Successful GET, PATCH, PUT (with response body)
- `201 Created` - Successful POST (new resource created)
- `204 No Content` - Successful DELETE or operation with no response body
- `206 Partial Content` - Partial GET response (future use for chunked data)

**Client Error Codes:**
- `400 Bad Request` - Invalid request syntax, validation errors
- `401 Unauthorized` - Authentication required or failed
- `403 Forbidden` - Authenticated but lacks permissions (see [Permissions Table](../02-roles-and-permissions/permissions-table.md))
- `404 Not Found` - Resource does not exist
- `409 Conflict` - Resource state conflict (e.g., duplicate, version mismatch)
- `422 Unprocessable Entity` - Validation errors with detailed field-level feedback
- `429 Too Many Requests` - Rate limit exceeded (see [Rate Limiting and Abuse](rate-limiting-and-abuse.md))

**Server Error Codes:**
- `500 Internal Server Error` - Unexpected server error
- `502 Bad Gateway` - External service failure (Braintree, email provider)
- `503 Service Unavailable` - Temporary service unavailability (maintenance, overload)
- `504 Gateway Timeout` - External service timeout

**Error Code Mapping:**
```typescript
// types/api/errors.ts
export enum ApiErrorCode {
  VALIDATION_ERROR = 'VALIDATION_ERROR',
  AUTHENTICATION_REQUIRED = 'AUTHENTICATION_REQUIRED',
  PERMISSION_DENIED = 'PERMISSION_DENIED',
  RESOURCE_NOT_FOUND = 'RESOURCE_NOT_FOUND',
  RESOURCE_CONFLICT = 'RESOURCE_CONFLICT',
  RATE_LIMIT_EXCEEDED = 'RATE_LIMIT_EXCEEDED',
  EXTERNAL_SERVICE_ERROR = 'EXTERNAL_SERVICE_ERROR',
  INTERNAL_SERVER_ERROR = 'INTERNAL_SERVER_ERROR',
}
```

---

### 4. Authentication & Authorization

All API routes are secured by default using NextAuth.js middleware:

**Authentication Flow:**
1. Client sends request with session cookie (JWT)
2. Middleware validates session and extracts user context
3. API route receives `req.user` with authenticated user data
4. Domain service performs permission checks via RBAC

**Request Headers:**
```
Cookie: next-auth.session-token=<JWT>
Content-Type: application/json
Accept: application/json
```

**Authorization Pattern:**
```typescript
// pages/api/assignments/[id].ts
import { withAuth } from '@/lib/middleware/auth';
import { AssignmentService } from '@/lib/domains/learning';

export default withAuth(async (req, res) => {
  const { id } = req.query;
  const user = req.user; // Injected by withAuth middleware
  
  // Permission check
  if (!user.canAccessAssignment(id)) {
    return res.status(403).json({
      error: {
        code: 'PERMISSION_DENIED',
        message: 'You do not have permission to access this assignment'
      }
    });
  }
  
  const assignment = await AssignmentService.getById(id);
  res.json({ data: assignment });
});
```

**Role-Based Access Control:**
- Permissions enforced at API middleware layer
- Fine-grained permissions defined in [Permissions Table](../02-roles-and-permissions/permissions-table.md)
- Data scoped by user role and organization (see [Service Boundaries](service-boundaries.md))

---

### 5. Versioning Strategy

**Current Approach: No Explicit Versioning (V1 implicit)**

For the initial launch, APIs are unversioned (implicitly V1) to maximize velocity:
- All endpoints live under `/api/*`
- Breaking changes avoided through additive evolution
- Deprecated fields marked in documentation before removal

**Future Versioning (When Needed):**

When breaking changes become necessary, adopt URL-based versioning:
```
/api/v2/assignments
/api/v2/students
```

**Deprecation Process:**
1. Announce deprecation with 6-month notice
2. Add deprecation headers: `Deprecation: true`, `Sunset: 2026-01-01`
3. Log warnings for deprecated endpoint usage (see [Event Model](../15-analytics-and-reporting/event-model.md))
4. Maintain deprecated version for minimum 6 months
5. Remove after transition period

**Versioning Decision Criteria:**
- External integrations depend on API stability
- Breaking changes impact multiple client applications
- Migration cost justifies maintaining parallel versions

---

### 6. Pagination, Filtering, and Sorting

#### Pagination

**Offset-Based Pagination (Default):**
```
GET /api/games?page=2&limit=20
```

**Response:**
```json
{
  "data": [...],
  "pagination": {
    "page": 2,
    "limit": 20,
    "total": 156,
    "totalPages": 8,
    "hasNext": true,
    "hasPrevious": true
  }
}
```

**Cursor-Based Pagination (Future for high-volume endpoints):**
```
GET /api/events?cursor=evt_abc123&limit=50
```

**Response:**
```json
{
  "data": [...],
  "pagination": {
    "cursor": "evt_xyz789",
    "hasNext": true,
    "limit": 50
  }
}
```

#### Filtering

**Query Parameters:**
```
GET /api/games?category=rhythm&difficulty=beginner&status=published
```

**Complex Filters (JSON-encoded for advanced use cases):**
```
GET /api/students?filter={"grade":{"gte":3},"status":"active"}
```

**Filter Operators:**
- `eq` (equals, default)
- `ne` (not equals)
- `gt`, `gte` (greater than, greater than or equal)
- `lt`, `lte` (less than, less than or equal)
- `in` (value in list)
- `contains` (substring match for text fields)

#### Sorting

**Sort Syntax:**
```
GET /api/games?sort=-createdAt,title
```
- Prefix with `-` for descending order
- Comma-separated for multiple sort fields
- Default sort defined per endpoint

**Example:**
```typescript
// Types for filter/sort
interface PaginationParams {
  page?: number;
  limit?: number;
  cursor?: string;
}

interface SortParams {
  sort?: string; // e.g., "-createdAt,title"
}

interface FilterParams {
  [key: string]: string | number | boolean;
}
```

---

### 7. Data Validation and Type Safety

**Input Validation:**
- Define TypeScript types for all request/response schemas
- Use Zod or Yup for runtime validation
- Validate at API route boundary before calling domain services
- Return `422 Unprocessable Entity` with field-level errors

**Validation Example:**
```typescript
import { z } from 'zod';

const CreateAssignmentSchema = z.object({
  sequenceId: z.string().regex(/^seq_[a-zA-Z0-9]+$/),
  studentIds: z.array(z.string()).min(1).max(500),
  dueDate: z.string().datetime(),
  notes: z.string().max(2000).optional(),
});

export default withAuth(async (req, res) => {
  try {
    const input = CreateAssignmentSchema.parse(req.body);
    const assignment = await AssignmentService.create(input);
    res.status(201).json({ data: assignment });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(422).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Invalid input data',
          details: error.errors.map(e => ({
            field: e.path.join('.'),
            message: e.message,
            code: e.code,
          })),
        }
      });
    }
    throw error; // Let error handler catch
  }
});
```

**Type Safety Benefits:**
- Compile-time type checking prevents errors
- IDE autocomplete for better developer experience
- Consistent validation across all endpoints
- Generated documentation from types

---

### 8. Error Handling and Resilience

**Consistent Error Response Format:**
```json
{
  "error": {
    "code": "RESOURCE_NOT_FOUND",
    "message": "Assignment not found",
    "details": {
      "resourceType": "assignment",
      "resourceId": "asg_12345"
    }
  },
  "meta": {
    "timestamp": "2025-01-27T10:00:00Z",
    "requestId": "req_abc123"
  }
}
```

**Error Handling Pattern:**
```typescript
// lib/middleware/error-handler.ts
export function withErrorHandler(handler: ApiHandler) {
  return async (req, res) => {
    try {
      await handler(req, res);
    } catch (error) {
      if (error instanceof ValidationError) {
        return res.status(422).json({ error: formatValidationError(error) });
      }
      if (error instanceof NotFoundError) {
        return res.status(404).json({ error: formatNotFoundError(error) });
      }
      if (error instanceof PermissionError) {
        return res.status(403).json({ error: formatPermissionError(error) });
      }
      
      // Log unexpected errors
      console.error('Unexpected API error:', error);
      await EventBus.emit('api.error', { error, req });
      
      return res.status(500).json({
        error: {
          code: 'INTERNAL_SERVER_ERROR',
          message: 'An unexpected error occurred',
          requestId: req.id,
        }
      });
    }
  };
}
```

**Retry Strategy (for clients):**
- `429 Too Many Requests`: Respect `Retry-After` header
- `500/502/503/504`: Exponential backoff (1s, 2s, 4s, 8s)
- `400/401/403/404/409/422`: Do not retry (client error)

---

### 9. Caching and Performance

**Cache-Control Headers:**
```typescript
// Static/public data (games, sequences)
res.setHeader('Cache-Control', 'public, max-age=3600, s-maxage=3600');

// User-specific data (assignments, progress)
res.setHeader('Cache-Control', 'private, max-age=300, s-maxage=0');

// Real-time data (leaderboards, notifications)
res.setHeader('Cache-Control', 'no-cache, no-store, must-revalidate');
```

**ETag Support:**
```typescript
const etag = generateEtag(data);
res.setHeader('ETag', etag);

if (req.headers['if-none-match'] === etag) {
  return res.status(304).end(); // Not Modified
}
```

**Conditional Requests:**
- Support `If-None-Match` (ETag) for GET requests
- Support `If-Modified-Since` for time-based caching
- Return `304 Not Modified` when appropriate

See [Caching Strategy](caching-strategy.md) for detailed caching policies.

---

### 10. API Documentation

**Documentation Requirements:**
- Every endpoint documented in [REST Endpoints](rest-endpoints.md)
- TypeScript types serve as source of truth
- OpenAPI/Swagger spec generated from types (future)
- Code examples for common use cases

**Inline Documentation:**
```typescript
/**
 * Create a new assignment for students.
 * 
 * @route POST /api/assignments
 * @access Teacher, Teacher-Admin, Subscriber-Admin
 * @body CreateAssignmentRequest
 * @returns CreateAssignmentResponse
 * @throws 422 - Validation error
 * @throws 403 - Permission denied
 * @throws 404 - Sequence not found
 */
export default withAuth(async (req, res) => {
  // implementation
});
```

---

### 11. Rate Limiting

**Rate Limit Strategy:**
- Implemented at Cloudflare edge (primary protection)
- Application-level rate limiting for sensitive endpoints
- Rate limits vary by user role and endpoint type

**Rate Limit Headers:**
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1643284800
Retry-After: 60
```

**Rate Limit Response:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 60
  }
}
```

See [Rate Limiting and Abuse](rate-limiting-and-abuse.md) for detailed rate limiting policies.

---

### 12. Idempotency

**Idempotent Operations:**
- `GET`, `PUT`, `DELETE` are naturally idempotent
- `POST` can be made idempotent with `Idempotency-Key` header (future)

**Idempotency Key Pattern (Future):**
```typescript
// Client sends
POST /api/assignments
Idempotency-Key: idem_abc123xyz
{ ... }

// Server checks if this key was already processed
const existing = await cache.get(`idempotency:${idempotencyKey}`);
if (existing) {
  return res.json(existing); // Return cached result
}

// Process request and cache result
const result = await AssignmentService.create(data);
await cache.set(`idempotency:${idempotencyKey}`, result, 86400); // 24h TTL
```

---

### 13. API Security Best Practices

**Input Sanitization:**
- Validate and sanitize all user input
- Use parameterized queries to prevent SQL injection
- Escape HTML to prevent XSS attacks
- Validate file uploads (type, size, content)

**Output Encoding:**
- JSON responses automatically escaped by Next.js
- HTML content sanitized before rendering
- User-generated content filtered for malicious code

**Security Headers:**
```typescript
res.setHeader('X-Content-Type-Options', 'nosniff');
res.setHeader('X-Frame-Options', 'DENY');
res.setHeader('X-XSS-Protection', '1; mode=block');
res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
```

**Secrets and Sensitive Data:**
- Never log sensitive data (passwords, tokens, PII)
- Redact sensitive fields in error messages
- Use environment variables for secrets (see [Config and Secrets](config-and-secrets.md))

---

### 14. Bulk Operations and Async Processing

**Bulk Endpoint Pattern:**
```typescript
POST /api/bulk/students
{
  "students": [
    { "firstName": "Alice", "lastName": "Smith", ... },
    { "firstName": "Bob", "lastName": "Jones", ... }
  ]
}

// Response (202 Accepted - async processing)
{
  "data": {
    "jobId": "job_abc123",
    "status": "processing",
    "estimatedCompletion": "2025-01-27T10:05:00Z"
  }
}

// Poll for status
GET /api/bulk/jobs/job_abc123
{
  "data": {
    "jobId": "job_abc123",
    "status": "completed",
    "results": {
      "total": 150,
      "successful": 148,
      "failed": 2,
      "errors": [...]
    }
  }
}
```

**Async Processing:**
- Return `202 Accepted` for long-running operations
- Provide job ID for status polling
- Emit events for completion (see [Event Model](../15-analytics-and-reporting/event-model.md))
- Consider webhooks for external integrations (see [Webhooks](webhooks.md))

See [Background Jobs](background-jobs.md) for async processing patterns.

---

### 15. API Naming Conventions

**Resource Names:**
- Use **plural nouns**: `/api/students`, `/api/games`, `/api/badges`
- Use **kebab-case** for multi-word resources: `/api/promo-codes`, `/api/seat-allocations`
- Avoid verbs in resource names (use HTTP methods instead)

**Action Endpoints (exceptions):**
```
POST /api/assignments/:id/submit
POST /api/passwords/reset
POST /api/students/:id/activate
```

**Field Naming (camelCase in JSON):**
```json
{
  "firstName": "Alice",
  "lastName": "Smith",
  "dateOfBirth": "2012-03-15",
  "enrollmentDate": "2024-09-01"
}
```

**ID Fields:**
- Use descriptive ID field names: `studentId`, `gameId`, `assignmentId`
- Prefix IDs with type: `stu_12345`, `game_67890`, `asg_abc123`
- Avoid generic `id` when ambiguous

---

### 16. Webhook Design Patterns

**Webhook Payload Format:**
```json
{
  "event": "assignment.completed",
  "data": {
    "assignmentId": "asg_12345",
    "studentId": "stu_67890",
    "completedAt": "2025-01-27T15:30:00Z",
    "score": 92
  },
  "meta": {
    "webhookId": "wh_abc123",
    "timestamp": "2025-01-27T15:30:01Z",
    "signature": "sha256=..."
  }
}
```

**Webhook Security:**
- HMAC signature verification
- Retry with exponential backoff
- Timeout handling (5s default)
- Idempotency key to prevent duplicate processing

See [Webhooks](webhooks.md) for detailed webhook specifications.

---

## Behavior and Rules

### API Lifecycle

**Development:**
1. Define TypeScript request/response types
2. Implement validation schema (Zod)
3. Add authentication/authorization middleware
4. Implement domain service integration
5. Add error handling
6. Write unit and integration tests
7. Document in [REST Endpoints](rest-endpoints.md)

**Deprecation:**
1. Announce deprecation with 6-month notice minimum
2. Add `Deprecation: true` header
3. Log usage metrics for migration planning
4. Maintain deprecated endpoint during transition
5. Remove after transition period

### Breaking Changes

**Considered Breaking:**
- Removing or renaming endpoints
- Removing or renaming fields
- Changing field types
- Adding required fields
- Changing error response format
- Changing authentication/authorization requirements

**Not Breaking (Additive):**
- Adding new endpoints
- Adding optional fields
- Adding new error codes
- Expanding enum values (if clients handle unknown values)
- Performance improvements
- Bug fixes

---

## UX Requirements

### API Developer Experience

**Type Safety:**
- All endpoints have TypeScript types
- IDE autocomplete for request/response objects
- Compile-time validation prevents runtime errors

**Clear Error Messages:**
- Actionable error messages with field-level details
- Error codes enable programmatic error handling
- Request IDs for support and debugging

**Consistent Patterns:**
- Same pagination strategy across all collection endpoints
- Same filtering/sorting syntax across all list endpoints
- Same authentication/authorization approach for all secured endpoints

**Performance:**
- API response time p95 < 500ms (see [System Overview](system-overview.md))
- Support conditional requests (ETags) to reduce bandwidth
- Implement caching where appropriate

---

## Telemetry

All API requests generate telemetry events for monitoring and analytics:

**API Request Event:**
```json
{
  "event_type": "api_request",
  "properties": {
    "method": "POST",
    "path": "/api/assignments",
    "statusCode": 201,
    "duration_ms": 145,
    "userId": "usr_12345",
    "userRole": "teacher",
    "requestId": "req_abc123"
  }
}
```

**API Error Event:**
```json
{
  "event_type": "api_error",
  "properties": {
    "method": "GET",
    "path": "/api/students/stu_999",
    "statusCode": 404,
    "errorCode": "RESOURCE_NOT_FOUND",
    "duration_ms": 23,
    "userId": "usr_12345",
    "requestId": "req_def456"
  }
}
```

See [Event Model](../15-analytics-and-reporting/event-model.md) for complete event taxonomy.

---

## Permissions

**API Design Authority:**
- **System Admins**: Full authority over API design decisions
- **Lead Developers**: Propose and implement API changes
- **All Developers**: Must follow established conventions

**API Access:**
- All endpoints require authentication unless explicitly public
- Authorization enforced per [Permissions Table](../02-roles-and-permissions/permissions-table.md)
- Data scoped by user role and organization (see [Service Boundaries](service-boundaries.md))

---

## Supporting Documents Referenced

This API design principles specification draws from the following source documents:

- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — API requirements for admin interfaces
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role-based API access requirements

## Dependencies

This specification relies on:
- [System Overview](system-overview.md) - Overall architecture and technology choices
- [Service Boundaries](service-boundaries.md) - Domain boundaries and internal APIs
- [REST Endpoints](rest-endpoints.md) - Specific endpoint specifications
- [Data Model ERD](data-model-erd.md) - Database schema and entity relationships
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Authorization rules
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry and tracking
- [Caching Strategy](caching-strategy.md) - Cache policies and TTLs
- [Rate Limiting and Abuse](rate-limiting-and-abuse.md) - Rate limiting policies
- [Background Jobs](background-jobs.md) - Async processing patterns
- [Webhooks](webhooks.md) - Webhook specifications
- [Config and Secrets](config-and-secrets.md) - Environment configuration

---

## Open Questions

### Near-Term Decisions

1. **OpenAPI Specification**: Should we generate OpenAPI/Swagger specs from TypeScript types, or maintain them manually?
2. **GraphQL Consideration**: For complex frontend data requirements, should we introduce GraphQL alongside REST, or continue with REST + optimized endpoints?
3. **Idempotency Keys**: Should we implement idempotency keys for POST requests in Phase 1, or defer to Phase 2?
4. **Request ID Generation**: Use UUID v4, ULID, or custom format for request IDs?
5. **Cursor Pagination**: Which high-volume endpoints need cursor pagination from day one vs. can start with offset pagination?

### Long-Term Considerations

1. **API Gateway**: At what scale do we need a dedicated API gateway (Kong, Tyk) vs. continuing with Next.js API routes?
2. **Real-Time APIs**: When do we introduce WebSocket or Server-Sent Events (SSE) for real-time features (leaderboards, notifications)?
3. **External API Program**: If/when we open APIs to third-party developers, what authentication mechanism (OAuth 2.0, API keys)?
4. **API Versioning Trigger**: What constitutes enough breaking changes to justify moving to v2?
5. **Multi-Tenancy**: As the platform scales, do we need tenant-specific API rate limits or quotas?

---

## Summary

The MLC LMS API design principles prioritize:
- **Consistency**: Predictable patterns across all endpoints
- **Type Safety**: End-to-end TypeScript for reliability
- **Developer Experience**: Clear documentation, good error messages, IDE support
- **Security**: Authentication, authorization, input validation, rate limiting
- **Performance**: Caching, pagination, conditional requests
- **Maintainability**: Clear conventions, versioning strategy, deprecation process
- **Observability**: Comprehensive telemetry and monitoring

By following these principles, we ensure that the API surface is intuitive, reliable, and scalable as the platform grows from individual studios to large school districts.
