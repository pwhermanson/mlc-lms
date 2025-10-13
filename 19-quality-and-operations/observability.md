# Observability

## Purpose

This document defines the observability infrastructure for the MLC LMS platform, specifying monitoring, alerting, logging, metrics collection, and error tracking systems that enable proactive system management, rapid incident response, and continuous performance optimization.

Observability provides the instrumentation, tooling, and processes necessary to understand system behavior in production, detect anomalies before they impact users, diagnose issues quickly, and make data-driven decisions about platform health, performance, and reliability.

Given the platform's educational mission and diverse user base (students ages 6-18, teachers, administrators), observability must ensure:

- **User Experience Monitoring**: Track page load times, game performance, and interaction responsiveness
- **Reliability Tracking**: Monitor uptime, error rates, and service availability across all user roles
- **Performance Visibility**: Identify bottlenecks in database queries, API endpoints, and background jobs
- **Security Monitoring**: Detect suspicious patterns, unauthorized access attempts, and security incidents (see [Security Privacy](../18-architecture/security-privacy.md))
- **Proactive Alerting**: Notify operations team before issues escalate to user-facing problems
- **Data-Driven Operations**: Enable informed decisions about capacity planning, optimization priorities, and infrastructure investments

## Scope

### Included

**Monitoring Infrastructure**:
- Application Performance Monitoring (APM) tooling and instrumentation
- Real-time metrics collection and aggregation (system, application, business metrics)
- Distributed tracing for request flows across services
- Error tracking and exception monitoring
- Log aggregation, search, and analysis infrastructure
- Uptime monitoring and synthetic checks
- Resource utilization monitoring (CPU, memory, database connections)

**Alerting System**:
- Alert rules and thresholds for critical, high, medium, and low severity issues
- Alert routing and escalation policies
- Notification channels (email, SMS, Slack, PagerDuty)
- Alert grouping and deduplication
- On-call schedules and incident assignment
- Alert fatigue prevention (intelligent alerting, noise reduction)

**Observability Practices**:
- Instrumentation standards and conventions
- Logging standards and structured logging format
- Metrics naming conventions and labeling
- Trace context propagation
- Correlation IDs for request tracking
- Error context enrichment (user context, request data, stack traces)

**Operational Dashboards**:
- System health overview and status pages
- Real-time metrics visualization
- Historical trend analysis
- SLI/SLO tracking dashboards
- Cost monitoring and budget tracking

### Excluded

- **Security Event Logging Details**: Covered in [Security Privacy](../18-architecture/security-privacy.md) - security-specific events, audit logs, anomaly detection
- **Job-Specific Monitoring**: Covered in [Background Jobs](../18-architecture/background-jobs.md) - job queue health, job execution metrics
- **Dashboard UI Specifications**: Covered in [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md) - layout, components, user interface
- **Incident Response Procedures**: Covered in [Incident Response](incident-response.md) - how to respond when alerts fire
- **Event Schema Definitions**: Covered in [Event Model](../15-analytics-and-reporting/event-model.md) - business analytics event structure
- **Test Monitoring**: Covered in [Test Strategy](test-strategy.md) - test execution monitoring, test coverage tracking

---

## Observability Architecture

### Observability Pillars

The MLC LMS observability strategy is built on three fundamental pillars:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    OBSERVABILITY ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌────────────────┐   ┌────────────────┐   ┌────────────────┐     │
│  │   LOGS         │   │   METRICS      │   │   TRACES       │     │
│  │                │   │                │   │                │     │
│  │ • Application  │   │ • System       │   │ • Request      │     │
│  │ • Access       │   │ • Business     │   │   flows        │     │
│  │ • Error        │   │ • Custom       │   │ • Dependencies │     │
│  │ • Audit        │   │ • SLIs         │   │ • Latency      │     │
│  └────────────────┘   └────────────────┘   └────────────────┘     │
│          │                    │                     │               │
│          └────────────────────┴─────────────────────┘              │
│                              │                                      │
│                              ▼                                      │
│               ┌──────────────────────────────┐                     │
│               │  Correlation & Analysis      │                     │
│               │  • Correlation IDs           │                     │
│               │  • Context enrichment        │                     │
│               │  • Root cause analysis       │                     │
│               └──────────────────────────────┘                     │
│                              │                                      │
│                              ▼                                      │
│               ┌──────────────────────────────┐                     │
│               │  Alerting & Dashboards       │                     │
│               │  • Real-time alerts          │                     │
│               │  • Operational dashboards    │                     │
│               │  • SLO tracking              │                     │
│               └──────────────────────────────┘                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Implementation Phases

**Phase 1: MVP (Vercel Native + Basic Tools)**
- Vercel Analytics for basic performance monitoring
- Vercel Logs for application logging (7-day retention)
- Console-based error tracking with structured error objects
- Manual uptime checks and basic health endpoints
- Simple alert rules via Vercel monitoring dashboards

**Phase 2: Enhanced Observability (Recommended)**
- **Error Tracking**: Sentry for comprehensive error monitoring and release tracking
- **Log Aggregation**: Better Stack (Logtail) or Datadog for centralized logging with 1-year retention
- **Uptime Monitoring**: UptimeRobot or Pingdom for synthetic checks
- **Status Page**: Statuspage.io for public status communication
- **Alerting**: Integrated alerting across error tracking, uptime, and logs

**Phase 3: Full Observability Platform (Future)**
- **APM**: Datadog APM or New Relic for distributed tracing and performance profiling
- **Metrics Platform**: Prometheus + Grafana or Datadog metrics for custom metrics
- **Real User Monitoring (RUM)**: Track real user experience, page load times, interaction timing
- **Synthetic Monitoring**: Multi-location synthetic tests for critical user flows
- **Incident Management**: PagerDuty or Opsgenie for on-call rotation and escalation

---

## Logging

### Logging Standards

**Structured Logging Format:**

All application logs use structured JSON format for consistent parsing and filtering:

```typescript
interface LogEntry {
  timestamp: string;        // ISO 8601: "2025-01-27T14:30:00.123Z"
  level: "debug" | "info" | "warn" | "error" | "fatal";
  message: string;          // Human-readable message
  correlationId: string;    // Trace entire request flow
  userId?: string;          // User involved (if authenticated)
  sessionId?: string;       // Session identifier
  orgId?: string;           // Organization context
  path?: string;            // Request path (for HTTP requests)
  method?: string;          // HTTP method
  statusCode?: number;      // HTTP response status
  duration?: number;        // Request/operation duration (ms)
  errorCode?: string;       // Application error code
  errorStack?: string;      // Stack trace (errors only)
  metadata?: Record<string, any>; // Additional context
  environment: "development" | "staging" | "production";
  service: string;          // "mlc-lms" or future service names
  version: string;          // Deployment version/commit SHA
}
```

**Example Log Entries:**

```json
// Success log
{
  "timestamp": "2025-01-27T14:30:00.123Z",
  "level": "info",
  "message": "Assignment created successfully",
  "correlationId": "req_abc123xyz",
  "userId": "usr_456",
  "orgId": "org_789",
  "path": "/api/assignments",
  "method": "POST",
  "statusCode": 201,
  "duration": 145,
  "metadata": {
    "assignmentId": "asgn_321",
    "gameCount": 5
  },
  "environment": "production",
  "service": "mlc-lms",
  "version": "v1.2.3"
}

// Error log
{
  "timestamp": "2025-01-27T14:31:00.456Z",
  "level": "error",
  "message": "Database query timeout",
  "correlationId": "req_def456uvw",
  "userId": "usr_789",
  "path": "/api/reports/generate",
  "method": "POST",
  "statusCode": 500,
  "duration": 30000,
  "errorCode": "DB_TIMEOUT",
  "errorStack": "Error: Query timeout...\n  at ...",
  "metadata": {
    "query": "SELECT * FROM user_progress WHERE...",
    "timeout": 30000
  },
  "environment": "production",
  "service": "mlc-lms",
  "version": "v1.2.3"
}
```

### Log Categories and Levels

**Log Levels:**

| Level | Usage | Examples | Retention |
|-------|-------|----------|-----------|
| **DEBUG** | Detailed debugging information | Variable values, function entry/exit | 7 days |
| **INFO** | Normal operations, state changes | User login, assignment created, job completed | 90 days |
| **WARN** | Unexpected but recoverable issues | Deprecated API usage, fallback behavior, retry attempts | 180 days |
| **ERROR** | Error conditions requiring attention | API failures, unhandled exceptions, data validation failures | 1 year |
| **FATAL** | Critical failures causing service unavailability | Database connection loss, unrecoverable errors | 1 year |

**Log Categories:**

| Category | Description | Key Metadata |
|----------|-------------|--------------|
| **Application** | Business logic, user actions | userId, orgId, action, resource |
| **Access** | HTTP requests and responses | path, method, statusCode, duration |
| **Error** | Exceptions and failures | errorCode, errorStack, errorContext |
| **Security** | Authentication, authorization, security events (see [Security Privacy](../18-architecture/security-privacy.md)) | eventType, severity, ipAddress |
| **Performance** | Slow queries, high latency | duration, query, threshold |
| **Job** | Background job execution (see [Background Jobs](../18-architecture/background-jobs.md)) | jobId, jobType, status |
| **Audit** | Administrative actions, data changes | actorId, action, resource, before, after |

### Logging Best Practices

**DO:**
- ✅ Use structured logging with consistent field names
- ✅ Include correlation IDs in all logs for request tracing
- ✅ Log at appropriate levels (avoid debug logs in production)
- ✅ Include relevant context (user, org, resource IDs)
- ✅ Log both successes and failures for critical operations
- ✅ Redact sensitive data (passwords, tokens, PII per COPPA/FERPA)
- ✅ Use error context to enrich error logs with request data

**DON'T:**
- ❌ Log sensitive data (passwords, API keys, SSNs, credit cards)
- ❌ Log full user objects (log IDs only)
- ❌ Log in tight loops (causes performance issues and log noise)
- ❌ Use string concatenation for log messages (use structured fields)
- ❌ Log duplicate information (one comprehensive log > multiple redundant logs)
- ❌ Log raw request bodies containing PII without sanitization

### Log Retention and Storage

**Retention Policies:**

| Environment | Retention | Storage | Estimated Cost |
|-------------|-----------|---------|----------------|
| **Development** | 7 days | Vercel Logs | Free (included) |
| **Staging** | 30 days | Better Stack / Datadog | ~$20/month |
| **Production** | 1 year (errors/audit), 90 days (info), 7 days (debug) | Better Stack / Datadog | ~$100-200/month |

**Log Storage Options:**

| Option | Pros | Cons | Cost |
|--------|------|------|------|
| **Vercel Logs** | Free, integrated, easy setup | 7-day retention, basic search, no alerting | Free |
| **Better Stack (Logtail)** | Affordable, good search, SQL queries, long retention | Smaller ecosystem | ~$50-150/month |
| **Datadog Logs** | Powerful search, APM integration, alerting, dashboards | Expensive at scale | ~$200-500/month |
| **AWS CloudWatch** | Integration with AWS services (if using) | Complex pricing, vendor lock-in | Variable |

**Recommended: Better Stack (Logtail)** for Phase 2 due to cost-effectiveness and sufficient features.

### Log Aggregation and Search

**Search Capabilities:**

- **Full-text search**: Search across all log fields and message content
- **Field filtering**: Filter by level, service, userId, orgId, path, errorCode
- **Time range queries**: Search within specific time windows
- **Correlation ID tracking**: View all logs for a single request across services
- **Saved searches**: Pre-configured searches for common investigations
- **Alerting on log patterns**: Trigger alerts when specific log patterns appear

**Common Search Queries:**

```sql
-- All errors for a specific user
level:error AND userId:"usr_456"

-- Slow API requests (> 5 seconds)
category:access AND duration:>5000

-- Database timeout errors
errorCode:"DB_TIMEOUT"

-- All logs for a specific request
correlationId:"req_abc123xyz"

-- Failed login attempts
eventType:"auth.login.failure"
```

---

## Metrics

### Metric Types and Collection

**Metric Categories:**

| Category | Description | Examples | Collection Method |
|----------|-------------|----------|-------------------|
| **System Metrics** | Infrastructure and resource utilization | CPU, memory, disk, network | Vercel / APM agent |
| **Application Metrics** | Application behavior and performance | Request rate, error rate, latency | Custom instrumentation |
| **Business Metrics** | Product and user engagement | Active users, assignments completed, games played | Event tracking |
| **SLI Metrics** | Service level indicators for SLO tracking | Availability, latency p99, error rate | Derived from logs/traces |

### Core Application Metrics

**API Performance Metrics:**

| Metric | Description | Unit | Target / SLI |
|--------|-------------|------|--------------|
| `api.request.count` | Total API requests | count | N/A (informational) |
| `api.request.duration` | Request latency (p50, p95, p99) | milliseconds | p95 < 500ms, p99 < 2000ms |
| `api.request.error_rate` | Percentage of failed requests | percentage | < 1% |
| `api.request.rate` | Requests per second | req/sec | N/A (capacity planning) |
| `api.response.size` | Response payload size | bytes | N/A (bandwidth monitoring) |

**Database Metrics:**

| Metric | Description | Unit | Alert Threshold |
|--------|-------------|------|-----------------|
| `db.connection.pool.size` | Active database connections | count | > 80% of pool |
| `db.connection.pool.wait` | Connections waiting for pool | count | > 5 |
| `db.query.duration` | Query execution time (p95, p99) | milliseconds | p95 > 1000ms |
| `db.query.error_rate` | Failed queries | percentage | > 0.5% |
| `db.transaction.duration` | Transaction time | milliseconds | p95 > 2000ms |

**Background Job Metrics** (see [Background Jobs](../18-architecture/background-jobs.md)):

| Metric | Description | Unit | Alert Threshold |
|--------|-------------|------|-----------------|
| `job.queue.depth` | Jobs waiting in queue | count | > 100 |
| `job.execution.duration` | Job execution time | milliseconds | Varies by job type |
| `job.success_rate` | Percentage of successful jobs | percentage | < 95% |
| `job.retry_rate` | Percentage requiring retries | percentage | > 10% |
| `job.dead_letter.count` | Jobs in dead letter queue | count | > 0 (alert immediately) |

**User Experience Metrics:**

| Metric | Description | Unit | Target |
|--------|-------------|------|--------|
| `page.load.duration` | Full page load time (p75) | milliseconds | < 3000ms |
| `game.load.duration` | Game initialization time | milliseconds | < 5000ms |
| `game.fps` | Game rendering frame rate | fps | > 30 fps |
| `midi.input.latency` | MIDI input to sound latency | milliseconds | < 100ms |

### Metrics Naming Conventions

**Format**: `<namespace>.<entity>.<metric_name>`

**Examples:**
- `mlc.api.request.count`
- `mlc.db.query.duration`
- `mlc.game.load.duration`
- `mlc.user.session.active`

**Labels/Tags** (for metric dimensions):
```typescript
{
  environment: "production",
  service: "mlc-lms",
  endpoint: "/api/assignments",
  method: "POST",
  statusCode: "200",
  orgId: "org_789"
}
```

### Metrics Collection Implementation

**Phase 1: Custom Metrics via Logs**
```typescript
// Log metrics as structured events
logger.info('metric', {
  metricName: 'api.request.duration',
  value: duration,
  labels: {
    endpoint: '/api/assignments',
    method: 'POST',
    statusCode: 200
  }
});
```

**Phase 2: Dedicated Metrics Service**
```typescript
// Using statsd/dogstatsd protocol
import { StatsD } from 'hot-shots';

const statsd = new StatsD({
  host: process.env.STATSD_HOST,
  prefix: 'mlc.',
  globalTags: {
    environment: process.env.ENVIRONMENT,
    service: 'mlc-lms'
  }
});

// Track metrics
statsd.increment('api.request.count', 1, {
  endpoint: '/api/assignments',
  method: 'POST'
});

statsd.histogram('api.request.duration', duration, {
  endpoint: '/api/assignments',
  method: 'POST'
});
```

---

## Distributed Tracing

### Trace Context and Correlation

**Correlation ID Propagation:**

Every request receives a unique correlation ID that flows through the entire request lifecycle:

```typescript
// Generate correlation ID on request entry
import { v4 as uuidv4 } from 'uuid';

export function withCorrelation(handler) {
  return async (req, res) => {
    // Use existing correlation ID from header or generate new one
    const correlationId = req.headers['x-correlation-id'] || `req_${uuidv4()}`;
    
    // Attach to request context
    req.correlationId = correlationId;
    
    // Propagate in response headers
    res.setHeader('x-correlation-id', correlationId);
    
    // Include in all logs
    req.logger = logger.child({ correlationId });
    
    return handler(req, res);
  };
}
```

**Trace Spans:**

For complex operations, create trace spans to measure sub-operations:

```typescript
async function processAssignment(assignmentData, context) {
  const trace = createTrace(context.correlationId, 'process_assignment');
  
  const span1 = trace.startSpan('validate_input');
  const validatedData = await validateAssignment(assignmentData);
  span1.end();
  
  const span2 = trace.startSpan('db_insert');
  const assignment = await db.assignment.create({ data: validatedData });
  span2.end();
  
  const span3 = trace.startSpan('send_notifications');
  await notifyTeacher(assignment);
  span3.end();
  
  trace.end();
  return assignment;
}
```

### Distributed Tracing Implementation

**Phase 1: Manual Tracing via Logs**
- Use correlation IDs in all log entries
- Log operation start/end with duration
- Manually correlate logs for request flow analysis

**Phase 2: OpenTelemetry**
```typescript
import { trace } from '@opentelemetry/api';
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter(),
  instrumentations: [getNodeAutoInstrumentations()],
  serviceName: 'mlc-lms',
});

sdk.start();

// Automatic instrumentation for HTTP, database, etc.
// Custom spans for business logic
const tracer = trace.getTracer('mlc-lms');
const span = tracer.startSpan('process_assignment');
// ... operation
span.end();
```

**Phase 3: Full APM (Datadog / New Relic)**
- Automatic distributed tracing across all services
- Flame graphs for performance profiling
- Transaction tracing with database queries
- Integration with logs and metrics for full context

---

## Error Tracking

### Error Monitoring Strategy

**Error Categories:**

| Category | Severity | Response Time | Examples |
|----------|----------|---------------|----------|
| **Critical** | Fatal | Immediate (< 5 min) | Database unavailable, payment processing failure |
| **High** | Error | 1 hour | API endpoint failure, job dead letter, data corruption |
| **Medium** | Warning | 24 hours | Deprecated API usage, fallback triggered, slow query |
| **Low** | Info | Weekly review | Expected errors (validation failures), rate limit warnings |

### Error Context Enrichment

**Captured Error Context:**

```typescript
interface ErrorContext {
  // Error details
  errorMessage: string;
  errorType: string;
  errorStack: string;
  errorCode?: string;
  
  // Request context
  correlationId: string;
  path: string;
  method: string;
  headers: Record<string, string>;  // Sanitized
  query: Record<string, any>;
  body: Record<string, any>;        // Sanitized (no PII)
  
  // User context
  userId?: string;
  sessionId?: string;
  orgId?: string;
  userRole?: string;
  
  // Environment context
  environment: string;
  version: string;
  commitSha: string;
  deployedAt: string;
  
  // Performance context
  duration?: number;
  memoryUsage?: number;
  
  // Breadcrumbs (last N actions leading to error)
  breadcrumbs: Array<{
    timestamp: string;
    category: string;
    message: string;
    level: string;
    data?: Record<string, any>;
  }>;
}
```

### Error Tracking Implementation

**Phase 1: Structured Error Logging**
```typescript
function logError(error: Error, context: ErrorContext) {
  logger.error('Unhandled error', {
    errorMessage: error.message,
    errorStack: error.stack,
    errorCode: error.code,
    ...context,
  });
}

// Global error handler
process.on('unhandledRejection', (error, promise) => {
  logError(error, {
    correlationId: getCurrentCorrelationId(),
    // ... context
  });
});
```

**Phase 2: Sentry Integration (Recommended)**
```typescript
import * as Sentry from '@sentry/nextjs';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.ENVIRONMENT,
  release: process.env.VERCEL_GIT_COMMIT_SHA,
  tracesSampleRate: 0.1,  // 10% of transactions
  
  beforeSend(event, hint) {
    // Sanitize PII from error context
    if (event.request) {
      delete event.request.cookies;
      // Redact sensitive headers
      if (event.request.headers) {
        delete event.request.headers.Authorization;
        delete event.request.headers['X-API-Key'];
      }
    }
    return event;
  },
  
  ignoreErrors: [
    // Expected errors that shouldn't alert
    'Network request failed',
    'ResizeObserver loop limit exceeded',
  ],
});

// Capture errors with context
Sentry.captureException(error, {
  tags: {
    endpoint: '/api/assignments',
    userId: 'usr_456',
  },
  contexts: {
    assignment: {
      assignmentId: 'asgn_123',
      gameCount: 5,
    },
  },
});
```

### Error Grouping and Deduplication

**Smart Grouping:**
- Group errors by error type, stack trace fingerprint, and error message
- Separate errors by environment (dev, staging, production)
- Track error frequency and first/last occurrence
- Identify new errors vs. recurring errors

**Alert Deduplication:**
- Don't alert on every occurrence of the same error
- Alert on first occurrence of a new error
- Alert if error rate exceeds threshold (e.g., > 10 occurrences in 5 minutes)
- Alert if error affects multiple users (e.g., > 5 unique users in 10 minutes)

---

## Alerting

### Alert Rules and Thresholds

**Critical Alerts** (Page immediately, 24/7):

| Alert | Condition | Threshold | Action |
|-------|-----------|-----------|--------|
| **Service Down** | Uptime check fails | 2 consecutive failures (2 min) | Page on-call engineer |
| **High Error Rate** | Error rate spike | > 5% of requests failing | Page on-call engineer |
| **Database Unavailable** | Database connection fails | Any occurrence | Page on-call engineer |
| **Payment Processing Failed** | Payment API errors | > 3 failures in 5 min | Page on-call + notify finance |
| **Data Corruption Detected** | Integrity check failure | Any occurrence | Page on-call + notify data team |

**High Priority Alerts** (Notify within 1 hour, business hours):

| Alert | Condition | Threshold | Action |
|-------|-----------|-----------|--------|
| **Elevated Error Rate** | Error rate increase | > 2% of requests failing | Notify on-call engineer |
| **Slow API Response** | Latency spike | p95 > 2000ms for 5 min | Notify on-call engineer |
| **Job Queue Backlog** | Jobs accumulating | > 100 jobs queued | Notify operations team |
| **Disk Space Warning** | Storage usage high | > 85% capacity | Notify operations team |
| **Failed Background Jobs** | Jobs in dead letter queue | > 5 jobs | Notify operations team |

**Medium Priority Alerts** (Daily digest):

| Alert | Condition | Threshold | Action |
|-------|-----------|-----------|--------|
| **Slow Database Query** | Query performance degradation | p95 > 1000ms | Daily digest to engineering |
| **High Memory Usage** | Memory utilization | > 80% for 15 min | Daily digest to operations |
| **Deprecated API Usage** | Legacy API calls | > 100 calls/day | Weekly report to product team |
| **Low Cache Hit Rate** | Cache effectiveness | < 70% hit rate | Daily digest to engineering |

**Low Priority Alerts** (Weekly report):

| Alert | Condition | Threshold | Action |
|-------|-----------|-----------|--------|
| **Validation Errors** | Expected user errors | > 1000/day | Weekly report |
| **Rate Limit Warnings** | Users approaching limits | > 50 users/day | Weekly report |
| **Browser Compatibility Issues** | Unsupported browser usage | > 100 users/week | Weekly report |

### Alert Notification Channels

**Notification Routing:**

| Severity | Channels | Response SLA |
|----------|----------|--------------|
| **Critical** | PagerDuty → SMS + Phone Call + Slack #incidents | 5 minutes |
| **High** | Slack #alerts + Email to on-call | 1 hour |
| **Medium** | Slack #monitoring + Daily digest email | 24 hours |
| **Low** | Weekly report email | 1 week |

**Escalation Policy:**

```
Critical Alert Fired
      ↓
Page Primary On-Call
      ↓
No acknowledgment in 5 min
      ↓
Page Secondary On-Call
      ↓
No acknowledgment in 5 min
      ↓
Page Engineering Manager
```

### Alert Configuration

**Alert Rule Structure:**

```typescript
interface AlertRule {
  id: string;
  name: string;
  severity: 'critical' | 'high' | 'medium' | 'low';
  condition: {
    metric: string;
    operator: '>' | '<' | '==' | '!=' | 'contains';
    threshold: number;
    window: number;        // Time window in seconds
    aggregation: 'avg' | 'sum' | 'min' | 'max' | 'count';
  };
  notifications: {
    channels: string[];    // ['pagerduty', 'slack', 'email']
    recipients: string[];  // User IDs or team names
  };
  enabled: boolean;
  quietHours?: {
    start: string;         // "22:00"
    end: string;           // "06:00"
    timezone: string;      // "America/New_York"
  };
  deduplication: {
    window: number;        // Don't alert again within N seconds
    groupBy: string[];     // Group by these fields
  };
}
```

**Example Alert Rules:**

```typescript
// Critical: API error rate spike
{
  id: 'alert_api_error_rate',
  name: 'High API Error Rate',
  severity: 'critical',
  condition: {
    metric: 'api.request.error_rate',
    operator: '>',
    threshold: 5,          // 5%
    window: 300,           // 5 minutes
    aggregation: 'avg'
  },
  notifications: {
    channels: ['pagerduty', 'slack'],
    recipients: ['on-call-engineer', '#incidents']
  },
  enabled: true,
  deduplication: {
    window: 900,           // Don't alert again for 15 min
    groupBy: ['environment']
  }
}

// High: Slow database queries
{
  id: 'alert_slow_db_query',
  name: 'Slow Database Query Performance',
  severity: 'high',
  condition: {
    metric: 'db.query.duration',
    operator: '>',
    threshold: 1000,       // 1000ms
    window: 600,           // 10 minutes
    aggregation: 'p95'
  },
  notifications: {
    channels: ['slack'],
    recipients: ['#alerts']
  },
  enabled: true,
  quietHours: {
    start: '22:00',
    end: '06:00',
    timezone: 'America/New_York'
  },
  deduplication: {
    window: 3600,          // Don't alert again for 1 hour
    groupBy: ['query_type']
  }
}
```

### Alert Fatigue Prevention

**Strategies to Reduce Alert Noise:**

1. **Smart Thresholds**: Use dynamic thresholds based on historical patterns, not static values
2. **Alert Grouping**: Group related alerts into single notification
3. **Deduplication**: Don't alert on same issue multiple times within time window
4. **Quiet Hours**: Suppress low/medium alerts during off-hours
5. **Auto-Resolution**: Automatically resolve alerts when condition clears
6. **Alert Tuning**: Regularly review and adjust alert thresholds based on false positive rate
7. **Severity Calibration**: Ensure severity matches actual impact (avoid "critical" for non-critical issues)

---

## Uptime and Synthetic Monitoring

### Uptime Monitoring

**Monitored Endpoints:**

| Endpoint | Check Interval | Timeout | Expected Status | Locations |
|----------|----------------|---------|-----------------|-----------|
| `https://app.mlc.com` | 1 minute | 10 seconds | 200 | US-East, US-West, EU |
| `https://app.mlc.com/api/health` | 1 minute | 5 seconds | 200 | US-East, US-West, EU |
| `https://app.mlc.com/api/db-health` | 2 minutes | 10 seconds | 200 | US-East |

**Health Check Endpoint:**

```typescript
// pages/api/health.ts
export default async function handler(req, res) {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    version: process.env.VERCEL_GIT_COMMIT_SHA,
    uptime: process.uptime(),
    checks: {
      database: 'healthy',
      cache: 'healthy',
      storage: 'healthy',
    }
  };
  
  try {
    // Check database connectivity
    await prisma.$queryRaw`SELECT 1`;
  } catch (error) {
    health.status = 'unhealthy';
    health.checks.database = 'unhealthy';
  }
  
  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
}
```

### Synthetic Monitoring

**Critical User Flows** (Phase 3):

| Flow | Steps | Frequency | Locations | Alert if Failed |
|------|-------|-----------|-----------|-----------------|
| **Student Login → Play Game** | Login, navigate to assignments, click game, verify load | Every 5 min | US, EU | 2 consecutive failures |
| **Teacher Create Assignment** | Login, navigate to assignments, create new, verify saved | Every 10 min | US, EU | 2 consecutive failures |
| **Admin View Reports** | Login, navigate to reports, generate report, verify data | Every 15 min | US | 2 consecutive failures |

---

## SLIs and SLOs

### Service Level Indicators (SLIs)

**Availability SLIs:**

| Service Component | SLI Definition | Measurement |
|-------------------|----------------|-------------|
| **API Availability** | Percentage of successful API requests (non-5xx) | `(successful_requests / total_requests) * 100` |
| **Page Availability** | Percentage of successful page loads | `(successful_page_loads / total_page_loads) * 100` |
| **Game Availability** | Percentage of games that load successfully | `(successful_game_loads / total_game_loads) * 100` |

**Latency SLIs:**

| Service Component | SLI Definition | Measurement |
|-------------------|----------------|-------------|
| **API Latency** | 95th percentile API response time | `p95(api_request_duration)` |
| **Page Load Time** | 75th percentile full page load | `p75(page_load_duration)` |
| **Database Query Time** | 95th percentile query execution time | `p95(db_query_duration)` |

**Quality SLIs:**

| Service Component | SLI Definition | Measurement |
|-------------------|----------------|-------------|
| **Score Recording Accuracy** | Percentage of scores successfully recorded | `(scores_recorded / scores_submitted) * 100` |
| **Job Success Rate** | Percentage of background jobs completed successfully | `(successful_jobs / total_jobs) * 100` |

### Service Level Objectives (SLOs)

**Platform SLOs:**

| SLO | Target | Measurement Window | Consequence if Breached |
|-----|--------|--------------------|-----------------------|
| **API Availability** | 99.5% | Rolling 30 days | Post-mortem, prioritize reliability work |
| **API Latency (p95)** | < 500ms | Rolling 7 days | Performance optimization sprint |
| **Page Load Time (p75)** | < 3 seconds | Rolling 7 days | Frontend optimization work |
| **Score Recording Accuracy** | 99.9% | Rolling 30 days | Critical: immediate investigation |
| **Job Success Rate** | 95% | Rolling 7 days | Review job error handling |

**SLO Tracking:**

- SLO dashboards visible to engineering and product teams
- Weekly SLO review in engineering standup
- Monthly SLO report to leadership
- Error budget tracking (remaining % before SLO breach)

---

## Implementation Roadmap

### Phase 1: MVP Observability (Month 1-2)

**Goals:**
- Basic visibility into production system
- Error detection and notification
- Foundation for future observability enhancements

**Deliverables:**
- ✅ Structured logging with correlation IDs
- ✅ Health check endpoints
- ✅ Basic Vercel Analytics and Logs
- ✅ Manual uptime checks
- ✅ Error logging with context
- ✅ Slack webhook for critical errors

**Estimated Effort:** 1-2 weeks
**Cost:** $0 (Vercel included)

### Phase 2: Enhanced Observability (Month 3-4)

**Goals:**
- Comprehensive error tracking with automatic alerting
- Centralized log aggregation with long retention
- Proactive uptime monitoring
- Basic metrics and dashboards

**Deliverables:**
- ✅ Sentry integration for error tracking
- ✅ Better Stack (Logtail) for log aggregation (1-year retention)
- ✅ UptimeRobot for synthetic uptime checks
- ✅ Alert rules and notification routing (Slack, email, PagerDuty)
- ✅ SLO tracking dashboards
- ✅ Public status page (Statuspage.io)

**Estimated Effort:** 2-3 weeks
**Cost:** ~$150-200/month (Sentry $79 + Better Stack $50 + UptimeRobot $18 + Statuspage $29)

### Phase 3: Full Observability Platform (Month 6+)

**Goals:**
- APM with distributed tracing and performance profiling
- Real user monitoring (RUM) for frontend performance
- Advanced synthetic monitoring for critical user flows
- Custom metrics platform for business intelligence

**Deliverables:**
- ✅ Datadog APM or New Relic for distributed tracing
- ✅ Real User Monitoring (RUM) for frontend performance
- ✅ Synthetic monitoring for critical user flows (multi-location)
- ✅ Custom metrics platform (Prometheus + Grafana or Datadog metrics)
- ✅ Incident management with PagerDuty
- ✅ Advanced anomaly detection with ML-powered insights

**Estimated Effort:** 4-6 weeks
**Cost:** ~$500-800/month (Datadog APM ~$400 + PagerDuty ~$100 + Synthetics ~$50)

---

## Telemetry

All observability events are tracked via the Event Model (see [Event Model](../15-analytics-and-reporting/event-model.md)). Key event types:

**System Events:**
- `system.health.checked` - Health check performed
- `system.alert.fired` - Alert triggered
- `system.alert.resolved` - Alert resolved
- `system.deployment.started` - New deployment initiated
- `system.deployment.completed` - Deployment finished

**Error Events:**
- `error.unhandled` - Unhandled exception caught
- `error.handled` - Expected error handled gracefully
- `error.rate_limit` - Rate limit exceeded

**Performance Events:**
- `performance.slow_query` - Database query exceeded threshold
- `performance.slow_request` - API request exceeded latency threshold
- `performance.high_memory` - Memory usage exceeded threshold

See [Event Model](../15-analytics-and-reporting/event-model.md) for complete event schema and tracking standards.

---

## Permissions

**Observability Tool Access:**

| Role | Logs | Metrics | Traces | Alerts | SLO Dashboards |
|------|------|---------|--------|--------|----------------|
| **System Admin** | Full access | Full access | Full access | Configure & acknowledge | Full access |
| **Engineering Team** | Read access | Read access | Read access | Acknowledge | Read access |
| **Product Team** | Limited (no PII) | Read access (business metrics) | No access | Read-only | Read access |
| **Support Team** | Limited (user-specific, PII redacted) | No access | No access | Read-only | No access |

See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) for complete permission definitions.

---

---

## Supporting Documents Referenced

This observability specification draws from the following source documents:

- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator monitoring dashboards and operational tools informing observability requirements and metrics visualization
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional admin monitoring features including error tracking, performance monitoring, and incident management capabilities
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role specifications informing role-based access control for observability tools and audit log requirements

---

## Dependencies

**Referenced Specifications:**
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry event structure and tracking
- [Security Privacy](../18-architecture/security-privacy.md) - Security logging and monitoring
- [Background Jobs](../18-architecture/background-jobs.md) - Job monitoring and metrics
- [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md) - Dashboard UI specifications
- [Incident Response](incident-response.md) - Procedures when alerts fire
- [Test Strategy](test-strategy.md) - Testing and QA monitoring approach
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Permission definitions

**External Services:**
- **Vercel**: Platform logging and analytics
- **Sentry** (Phase 2): Error tracking and release monitoring
- **Better Stack / Logtail** (Phase 2): Log aggregation and storage
- **UptimeRobot** (Phase 2): Uptime monitoring
- **Statuspage.io** (Phase 2): Public status communication
- **Datadog / New Relic** (Phase 3): APM and distributed tracing
- **PagerDuty** (Phase 3): Incident management and on-call rotation

---

## Open Questions

### Monitoring Tool Selection
- **Should we start with Sentry + Better Stack (Phase 2), or wait until we need full APM (Phase 3)?**
  - Recommendation: Implement Phase 2 shortly after MVP launch to gain early production insights and establish observability practices.

- **Which log aggregation service best fits our needs: Better Stack, Datadog Logs, or AWS CloudWatch?**
  - Recommendation: Better Stack (Logtail) for Phase 2 due to cost-effectiveness (~$50/month vs. $200+/month for Datadog).

- **Do we need PagerDuty for on-call rotation in Phase 2, or can we manage with Slack + email alerts initially?**
  - Recommendation: Start with Slack + email for Phase 2. Introduce PagerDuty in Phase 3 when on-call rotation becomes formalized.

### Alert Configuration
- **What is the appropriate error rate threshold before alerting? 1%? 5%?**
  - Recommendation: Start with 5% for critical alerts, 2% for high-priority alerts. Tune based on baseline error rates after 2-4 weeks in production.

- **Should we have separate alert thresholds for different endpoints (e.g., stricter for payment API)?**
  - Recommendation: Yes. Payment, authentication, and score recording should have stricter thresholds (1% error rate) due to critical impact.

- **How do we balance alert sensitivity to avoid alert fatigue while catching real issues early?**
  - Recommendation: Implement alert deduplication, quiet hours, and severity calibration. Review false positive rate monthly and adjust thresholds.

### Compliance and Privacy
- **What PII redaction rules apply to logs and error context per COPPA/FERPA?**
  - Recommendation: Redact all student names, emails, addresses, birth dates from logs. Log user IDs only. Implement automatic PII detection and redaction.

- **How long should we retain logs for different severity levels to balance cost and compliance needs?**
  - Recommendation: 1 year for errors/audit logs, 90 days for info logs, 7 days for debug logs. See [Retention Policies](#log-retention-and-storage).

### Cost Optimization
- **At what scale should we move from Better Stack to a more robust (but expensive) solution like Datadog?**
  - Recommendation: Evaluate when log volume exceeds 100GB/month or when we need APM/tracing (typically 6-12 months post-launch).

- **Can we use sampling for traces/metrics to reduce costs while maintaining visibility?**
  - Recommendation: Yes. Start with 10% trace sampling, 100% error capture. Increase sampling if needed for debugging.
