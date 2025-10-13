# Background Jobs

## Purpose

This document defines the background job processing architecture for the MLC LMS platform, specifying how asynchronous, long-running, and scheduled tasks are executed outside the request-response cycle. Background jobs enable the platform to handle resource-intensive operations (CSV imports, report generation, email delivery) without blocking user interactions, implement reliable retry logic for transient failures, and execute scheduled maintenance tasks to ensure system health and data integrity.

The background job system provides a scalable, fault-tolerant foundation for async processing that maintains responsiveness, reliability, and observability while operating within serverless constraints.

## Scope

### Included

**Job Types and Execution Patterns**:
- One-off async jobs triggered by user actions (CSV imports, bulk operations)
- Scheduled recurring jobs (cron-style maintenance tasks, cleanup, aggregation)
- Event-driven jobs triggered by system events (webhook processing, notifications)
- Retry and error handling patterns with exponential backoff
- Job priority and queue management
- Job status tracking and polling
- Job timeout and cancellation handling

**Infrastructure and Implementation**:
- Initial serverless implementation using Vercel Cron and API routes
- Job queue design and message structure
- Future migration path to dedicated job queue (BullMQ + Redis)
- Job persistence and state management
- Concurrency limits and rate limiting per job type
- Dead letter queue for failed jobs

**Observability and Operations**:
- Job logging and error tracking
- Job status dashboard (System Admin view)
- Performance metrics and SLAs
- Failed job inspection and retry
- Job monitoring and alerting

### Excluded

- Real-time WebSocket connections (separate specification)
- Synchronous API operations (see [REST Endpoints](rest-endpoints.md))
- Webhook endpoint definitions (see [Webhooks](webhooks.md))
- Specific business logic for individual job handlers (covered in domain-specific docs)
- Email provider integration details (see [Email Provider](../17-integrations/email-provider.md))
- CSV format specifications (see [CSV Specifications](../20-imports-and-automation/csv-specifications.md))

---

## Job Categories and Types

### 1. Bulk Data Operations

**Purpose**: Process large-scale data imports, exports, and transformations that cannot complete within typical HTTP request timeouts.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `bulk.import.students` | Import students from CSV | 30s - 5min | High | 5 concurrent |
| `bulk.import.teachers` | Import teachers from CSV | 30s - 5min | High | 5 concurrent |
| `bulk.import.games` | Import games from CSV | 1min - 10min | Medium | 3 concurrent |
| `bulk.import.assignments` | Import assignments from CSV | 30s - 5min | High | 5 concurrent |
| `bulk.update.students` | Batch update student records | 30s - 5min | Medium | 5 concurrent |
| `bulk.delete.students` | Batch delete student records | 10s - 2min | Medium | 3 concurrent |
| `bulk.export.students` | Export students to CSV | 30s - 5min | Low | 3 concurrent |
| `bulk.export.scores` | Export score data to CSV | 1min - 10min | Low | 2 concurrent |

**Trigger**: User-initiated via `/api/bulk/*` endpoints

**See**: [Bulk Ops Jobs](../20-imports-and-automation/bulk-ops-jobs.md), [CSV Specifications](../20-imports-and-automation/csv-specifications.md)

---

### 2. Report Generation

**Purpose**: Generate complex, data-intensive reports that require aggregation across large datasets.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `report.teacher.progress` | Teacher progress report for class | 10s - 2min | Medium | 10 concurrent |
| `report.admin.usage` | Admin usage report for organization | 30s - 5min | Medium | 5 concurrent |
| `report.sysadmin.analytics` | System admin analytics dashboard | 1min - 10min | Low | 3 concurrent |
| `report.export.pdf` | Export report to PDF format | 10s - 1min | Low | 5 concurrent |

**Trigger**: User-initiated via `/api/reports/*` endpoints or scheduled (daily/weekly)

**See**: [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md), [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md)

---

### 3. Email and Notification Delivery

**Purpose**: Send emails and notifications reliably with retry logic, avoiding blocking request handlers.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `email.send.transactional` | Send single transactional email | 1s - 5s | High | 50 concurrent |
| `email.send.bulk` | Send bulk email campaign | 10s - 5min | Low | 10 concurrent |
| `email.send.digest` | Send daily/weekly digest emails | 5s - 1min | Medium | 20 concurrent |
| `notification.push` | Send in-app notifications | 1s - 3s | High | 100 concurrent |

**Trigger**: Event-driven (assignment created, score recorded, etc.) or scheduled (digests)

**See**: [Email Templates](../13-messaging-and-notifications/email-templates.md), [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md), [Email Provider](../17-integrations/email-provider.md)

---

### 4. Webhook Processing

**Purpose**: Handle inbound webhook events asynchronously with retry logic for transient failures.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `webhook.process.braintree` | Process Braintree payment webhook | 5s - 30s | High | 20 concurrent |
| `webhook.process.stripe` | Process Stripe payment webhook | 5s - 30s | High | 20 concurrent |
| `webhook.retry.failed` | Retry failed webhook processing | 5s - 30s | Medium | 10 concurrent |
| `webhook.deliver.outbound` | Deliver outbound webhook to external endpoint | 2s - 10s | Medium | 30 concurrent |

**Trigger**: Inbound webhook received at `/api/webhooks/*` or outbound webhook scheduled

**See**: [Webhooks](webhooks.md), [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md)

---

### 5. Data Aggregation and Analytics

**Purpose**: Pre-compute expensive analytics queries and aggregations for dashboard performance.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `analytics.aggregate.daily` | Aggregate daily usage metrics | 2min - 10min | Low | 1 concurrent |
| `analytics.aggregate.weekly` | Aggregate weekly summary | 5min - 20min | Low | 1 concurrent |
| `analytics.student.progress` | Update student progress snapshots | 1min - 5min | Medium | 5 concurrent |
| `analytics.leaderboard.update` | Refresh leaderboard rankings | 30s - 2min | Medium | 3 concurrent |

**Trigger**: Scheduled (daily at 2am UTC, weekly on Sunday)

**See**: [Event Model](../15-analytics-and-reporting/event-model.md), [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md)

---

### 6. Content Processing and Validation

**Purpose**: Validate, process, and transform game assets and content.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `content.game.validate` | Validate game assets and metadata | 10s - 1min | Medium | 5 concurrent |
| `content.asset.transcode` | Transcode audio/video assets | 30s - 5min | Low | 3 concurrent |
| `content.cdn.sync` | Sync content to CDN | 1min - 10min | Low | 2 concurrent |
| `content.sequence.export` | Export sequence to JSON/CSV | 10s - 1min | Low | 5 concurrent |

**Trigger**: User-initiated via content authoring tools or automated validation

**See**: [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md), [Sequence Exports](../09-sequences-and-curriculum-tools/sequence-exports.md)

---

### 7. Scheduled Maintenance

**Purpose**: Perform routine cleanup, optimization, and health checks.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `maintenance.cleanup.sessions` | Clean up expired sessions | 1min - 5min | Low | 1 concurrent |
| `maintenance.cleanup.temp_files` | Remove temporary files from storage | 2min - 10min | Low | 1 concurrent |
| `maintenance.cleanup.logs` | Archive old audit logs | 5min - 20min | Low | 1 concurrent |
| `maintenance.optimize.db` | Run database optimization | 10min - 1hr | Low | 1 concurrent |
| `maintenance.backup.verify` | Verify database backups | 5min - 15min | Low | 1 concurrent |

**Trigger**: Scheduled (daily/weekly/monthly)

---

### 8. External Integration Sync (Future)

**Purpose**: Synchronize data with external LMS platforms and services.

**Job Types**:

| Job Type | Description | Avg Duration | Priority | Concurrency Limit |
|----------|-------------|--------------|----------|-------------------|
| `lti.grade.sync` | Sync grades to LMS platform | 5s - 30s | High | 20 concurrent |
| `lti.roster.sync` | Sync roster from LMS platform | 1min - 5min | Medium | 5 concurrent |
| `sso.user.provision` | Provision users from SSO provider | 10s - 1min | High | 10 concurrent |

**Trigger**: Event-driven (grade recorded) or scheduled (roster sync daily)

**See**: [LMS LTI Spec](../17-integrations/lms-ltispec-future.md), [SSO Integration](../17-integrations/sso-integration.md)

---

## Job Lifecycle and States

### Job State Machine

```
┌─────────────┐
│   QUEUED    │  Initial state when job is enqueued
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   RUNNING   │  Job is being processed
└──┬────┬─────┘
   │    │
   │    └──────────────┐
   │                   │
   ▼                   ▼
┌─────────────┐   ┌─────────────┐
│  COMPLETED  │   │   FAILED    │  Job failed (will retry if retries remain)
└─────────────┘   └──────┬──────┘
                         │
                         ▼
                  ┌─────────────┐
                  │  RETRYING   │  Job queued for retry
                  └──────┬──────┘
                         │
                         ▼
                  (back to QUEUED)
                         │
                         │ (after max retries)
                         ▼
                  ┌─────────────┐
                  │ DEAD_LETTER │  Job exhausted retries
                  └─────────────┘
```

### Job Lifecycle Events

| State | Description | Next States | Telemetry Event |
|-------|-------------|-------------|-----------------|
| `QUEUED` | Job created and waiting for worker | `RUNNING`, `CANCELLED` | `job.queued` |
| `RUNNING` | Job actively being processed | `COMPLETED`, `FAILED`, `TIMEOUT` | `job.started` |
| `COMPLETED` | Job finished successfully | (terminal) | `job.completed` |
| `FAILED` | Job encountered error | `RETRYING`, `DEAD_LETTER` | `job.failed` |
| `RETRYING` | Job queued for retry attempt | `QUEUED` | `job.retrying` |
| `TIMEOUT` | Job exceeded timeout limit | `RETRYING`, `DEAD_LETTER` | `job.timeout` |
| `CANCELLED` | Job cancelled by user/system | (terminal) | `job.cancelled` |
| `DEAD_LETTER` | Job exhausted all retry attempts | (terminal) | `job.dead_letter` |

---

## Job Data Model

### Job Entity

```typescript
interface Job {
  // Identification
  jobId: string;                    // Unique job ID, e.g., "job_abc123xyz"
  jobType: string;                  // Job type identifier, e.g., "bulk.import.students"
  jobCategory: JobCategory;         // Category for grouping
  
  // Ownership
  userId: string;                   // User who initiated job (null for system jobs)
  orgId?: string;                   // Organization context (if applicable)
  
  // State
  status: JobStatus;                // Current status (QUEUED, RUNNING, etc.)
  priority: JobPriority;            // HIGH, MEDIUM, LOW
  
  // Timing
  createdAt: Date;                  // When job was queued
  startedAt?: Date;                 // When job started processing
  completedAt?: Date;               // When job finished (success or failure)
  scheduledFor?: Date;              // For scheduled/delayed jobs
  timeoutAt?: Date;                 // When job will timeout
  
  // Execution
  attempts: number;                 // Current attempt number (starts at 1)
  maxAttempts: number;              // Maximum retry attempts (default 3)
  retryDelay: number;               // Delay before next retry in ms
  
  // Progress
  progress: number;                 // Progress percentage (0-100)
  progressMessage?: string;         // Human-readable progress description
  
  // Payload
  inputData: Record<string, any>;   // Input parameters for job
  outputData?: Record<string, any>; // Result data (if successful)
  errorData?: JobError;             // Error details (if failed)
  
  // Metadata
  metadata: Record<string, any>;    // Additional context
  tags: string[];                   // Searchable tags
  
  // Resources
  resourceLinks?: ResourceLink[];   // Links to created/affected resources
}

interface JobError {
  errorCode: string;                // Machine-readable error code
  errorMessage: string;             // Human-readable error message
  errorStack?: string;              // Stack trace (for debugging)
  retryable: boolean;               // Whether error is retryable
  lastErrorAt: Date;                // When error occurred
}

interface ResourceLink {
  resourceType: string;             // e.g., "student", "assignment", "report"
  resourceId: string;               // ID of created/affected resource
  resourceUrl?: string;             // URL to view resource
}

type JobStatus = 
  | 'QUEUED' 
  | 'RUNNING' 
  | 'COMPLETED' 
  | 'FAILED' 
  | 'RETRYING' 
  | 'TIMEOUT' 
  | 'CANCELLED' 
  | 'DEAD_LETTER';

type JobPriority = 'HIGH' | 'MEDIUM' | 'LOW';

type JobCategory = 
  | 'bulk_operations' 
  | 'reports' 
  | 'email' 
  | 'webhooks' 
  | 'analytics' 
  | 'content' 
  | 'maintenance' 
  | 'integration';
```

---

## Implementation Architecture

### Phase 1: Serverless-First (Initial Implementation)

**Goal**: Leverage Vercel's serverless infrastructure to implement background jobs with minimal operational overhead.

#### Job Queue Implementation

**Option A: Database-Backed Queue** (Recommended for Phase 1)

Use PostgreSQL as job queue with polling workers:

```typescript
// lib/jobs/queue.ts
export class JobQueue {
  /**
   * Enqueue a new job
   */
  static async enqueue(
    jobType: string,
    inputData: Record<string, any>,
    options: JobOptions = {}
  ): Promise<Job> {
    const job: Job = {
      jobId: generateJobId(),
      jobType,
      jobCategory: getJobCategory(jobType),
      userId: options.userId,
      orgId: options.orgId,
      status: 'QUEUED',
      priority: options.priority || 'MEDIUM',
      createdAt: new Date(),
      scheduledFor: options.scheduledFor,
      timeoutAt: options.timeout 
        ? new Date(Date.now() + options.timeout) 
        : undefined,
      attempts: 0,
      maxAttempts: options.maxAttempts || 3,
      retryDelay: options.retryDelay || 5000,
      progress: 0,
      inputData,
      metadata: options.metadata || {},
      tags: options.tags || [],
    };
    
    // Insert into database
    await db.query(
      `INSERT INTO jobs (...) VALUES (...)`,
      [job]
    );
    
    // Emit telemetry event
    await EventBus.emit('job.queued', { jobId: job.jobId, jobType });
    
    return job;
  }
  
  /**
   * Get next job to process (polling worker)
   */
  static async getNextJob(): Promise<Job | null> {
    // Atomic claim with FOR UPDATE SKIP LOCKED
    const result = await db.query(`
      UPDATE jobs
      SET status = 'RUNNING', 
          startedAt = NOW(), 
          attempts = attempts + 1
      WHERE jobId = (
        SELECT jobId FROM jobs
        WHERE status = 'QUEUED'
          AND (scheduledFor IS NULL OR scheduledFor <= NOW())
        ORDER BY 
          priority DESC,
          createdAt ASC
        FOR UPDATE SKIP LOCKED
        LIMIT 1
      )
      RETURNING *
    `);
    
    return result.rows[0] || null;
  }
  
  /**
   * Mark job as completed
   */
  static async complete(
    jobId: string, 
    outputData: Record<string, any>
  ): Promise<void> {
    await db.query(`
      UPDATE jobs
      SET status = 'COMPLETED',
          completedAt = NOW(),
          progress = 100,
          outputData = $2
      WHERE jobId = $1
    `, [jobId, outputData]);
    
    await EventBus.emit('job.completed', { jobId });
  }
  
  /**
   * Mark job as failed (with retry logic)
   */
  static async fail(
    jobId: string,
    error: JobError
  ): Promise<void> {
    const job = await JobQueue.getById(jobId);
    
    if (job.attempts < job.maxAttempts && error.retryable) {
      // Schedule retry with exponential backoff
      const retryDelay = job.retryDelay * Math.pow(2, job.attempts - 1);
      await db.query(`
        UPDATE jobs
        SET status = 'RETRYING',
            scheduledFor = NOW() + INTERVAL '${retryDelay} milliseconds',
            errorData = $2
        WHERE jobId = $1
      `, [jobId, error]);
      
      // Transition back to QUEUED after delay
      setTimeout(async () => {
        await db.query(`
          UPDATE jobs SET status = 'QUEUED' WHERE jobId = $1
        `, [jobId]);
      }, retryDelay);
      
      await EventBus.emit('job.retrying', { jobId, attempt: job.attempts });
    } else {
      // Move to dead letter queue
      await db.query(`
        UPDATE jobs
        SET status = 'DEAD_LETTER',
            completedAt = NOW(),
            errorData = $2
        WHERE jobId = $1
      `, [jobId, error]);
      
      await EventBus.emit('job.dead_letter', { jobId, error });
    }
  }
}
```

#### Job Worker (Vercel Serverless Function)

```typescript
// pages/api/jobs/worker.ts
// This endpoint is called by Vercel Cron every minute

import { JobQueue } from '@/lib/jobs/queue';
import { JobHandlers } from '@/lib/jobs/handlers';

export default async function handler(req, res) {
  // Verify cron secret to prevent unauthorized calls
  if (req.headers['authorization'] !== `Bearer ${process.env.CRON_SECRET}`) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  const maxJobsPerInvocation = 10;
  const processedJobs = [];
  
  // Process up to 10 jobs per invocation
  for (let i = 0; i < maxJobsPerInvocation; i++) {
    const job = await JobQueue.getNextJob();
    if (!job) break; // No more jobs to process
    
    try {
      // Find handler for job type
      const handler = JobHandlers.getHandler(job.jobType);
      if (!handler) {
        throw new Error(`No handler found for job type: ${job.jobType}`);
      }
      
      // Execute job handler
      const outputData = await handler.execute(job);
      
      // Mark as completed
      await JobQueue.complete(job.jobId, outputData);
      
      processedJobs.push({ jobId: job.jobId, status: 'completed' });
    } catch (error) {
      // Mark as failed (with retry logic)
      await JobQueue.fail(job.jobId, {
        errorCode: error.code || 'UNKNOWN_ERROR',
        errorMessage: error.message,
        errorStack: error.stack,
        retryable: isRetryableError(error),
        lastErrorAt: new Date(),
      });
      
      processedJobs.push({ 
        jobId: job.jobId, 
        status: 'failed', 
        error: error.message 
      });
    }
  }
  
  return res.status(200).json({ 
    processedCount: processedJobs.length,
    jobs: processedJobs 
  });
}

function isRetryableError(error: any): boolean {
  // Transient errors that should be retried
  const retryableErrors = [
    'ECONNREFUSED',
    'ETIMEDOUT',
    'ENOTFOUND',
    'DATABASE_BUSY',
    'RATE_LIMITED',
  ];
  
  return retryableErrors.some(code => 
    error.code === code || error.message.includes(code)
  );
}
```

#### Vercel Cron Configuration

```json
// vercel.json
{
  "crons": [
    {
      "path": "/api/jobs/worker",
      "schedule": "* * * * *"
    },
    {
      "path": "/api/jobs/maintenance/cleanup",
      "schedule": "0 2 * * *"
    },
    {
      "path": "/api/jobs/analytics/daily",
      "schedule": "0 3 * * *"
    }
  ]
}
```

**Pros**:
- ✅ No additional infrastructure (PostgreSQL already in use)
- ✅ Simple to implement and debug
- ✅ ACID transactions for job state
- ✅ No external dependencies

**Cons**:
- ❌ Polling overhead (worker checks queue every minute)
- ❌ Limited concurrency (10 jobs per invocation)
- ❌ No push-based notifications

**When to Use**: Initial launch, up to ~10,000 jobs/day

---

### Phase 2: Dedicated Job Queue (Future Enhancement)

**Goal**: Migrate to dedicated job queue infrastructure for better performance, scalability, and observability.

**Recommended Solution**: **BullMQ** + **Upstash Redis**

#### Why BullMQ?

- ✅ Built on Redis for high performance
- ✅ Advanced retry and backoff strategies
- ✅ Job priority and rate limiting
- ✅ Job events and progress tracking
- ✅ Recurring jobs (cron-style)
- ✅ Bull Board UI for monitoring
- ✅ TypeScript support

#### Architecture with BullMQ

```typescript
// lib/jobs/queue-bullmq.ts
import { Queue, Worker, QueueScheduler } from 'bullmq';
import IORedis from 'ioredis';

const connection = new IORedis(process.env.REDIS_URL);

// Create queue
export const jobQueue = new Queue('mlc-jobs', { connection });

// Create scheduler for delayed/recurring jobs
const scheduler = new QueueScheduler('mlc-jobs', { connection });

/**
 * Enqueue job
 */
export async function enqueueJob(
  jobType: string,
  inputData: Record<string, any>,
  options: JobOptions = {}
): Promise<string> {
  const job = await jobQueue.add(
    jobType,
    inputData,
    {
      priority: getPriority(options.priority),
      attempts: options.maxAttempts || 3,
      backoff: {
        type: 'exponential',
        delay: options.retryDelay || 5000,
      },
      timeout: options.timeout || 300000, // 5min default
      removeOnComplete: 1000, // Keep last 1000 completed
      removeOnFail: 5000,     // Keep last 5000 failed
    }
  );
  
  return job.id;
}

/**
 * Create worker to process jobs
 */
export function createWorker(
  concurrency: number = 5
): Worker {
  return new Worker(
    'mlc-jobs',
    async (job) => {
      const handler = JobHandlers.getHandler(job.name);
      if (!handler) {
        throw new Error(`No handler for job type: ${job.name}`);
      }
      
      // Update progress
      await job.updateProgress(0);
      
      // Execute handler
      const result = await handler.execute(job.data, job);
      
      // Update progress
      await job.updateProgress(100);
      
      return result;
    },
    {
      connection,
      concurrency,
      limiter: {
        max: 100,      // Max 100 jobs
        duration: 1000, // Per second
      },
    }
  );
}
```

**Migration Path**:
1. Deploy Redis instance (Upstash for serverless compatibility)
2. Implement BullMQ queue alongside existing database queue
3. Gradually migrate job types to BullMQ
4. Monitor performance and cost
5. Fully migrate once validated
6. Deprecate database queue

**When to Migrate**: When daily job volume exceeds 50,000 or latency becomes problematic

---

## Job Handler Pattern

### Handler Interface

```typescript
// lib/jobs/handlers/interface.ts
export interface JobHandler<TInput = any, TOutput = any> {
  /**
   * Execute job logic
   */
  execute(
    inputData: TInput,
    job?: Job
  ): Promise<TOutput>;
  
  /**
   * Validate input data before execution
   */
  validate?(inputData: TInput): Promise<void>;
  
  /**
   * Estimate job duration (for timeout calculation)
   */
  estimateDuration?(inputData: TInput): number;
  
  /**
   * Determine if error is retryable
   */
  isRetryable?(error: Error): boolean;
}
```

### Example Handler: CSV Student Import

```typescript
// lib/jobs/handlers/bulk-import-students.ts
import { JobHandler } from './interface';
import { StudentService } from '@/lib/domains/users';
import { parseCSV, validateStudentRow } from '@/lib/csv';

interface StudentImportInput {
  fileUrl: string;          // S3/R2 URL to CSV file
  orgId: string;            // Organization context
  teacherId?: string;       // Optional default teacher
  notifyOnComplete: boolean;
}

interface StudentImportOutput {
  created: number;
  updated: number;
  failed: number;
  errors: Array<{ row: number; error: string }>;
}

export const BulkImportStudentsHandler: JobHandler<
  StudentImportInput,
  StudentImportOutput
> = {
  async execute(inputData, job) {
    const { fileUrl, orgId, teacherId, notifyOnComplete } = inputData;
    
    // Download CSV from storage
    const csvContent = await downloadFile(fileUrl);
    
    // Parse CSV
    const rows = await parseCSV(csvContent);
    
    const results: StudentImportOutput = {
      created: 0,
      updated: 0,
      failed: 0,
      errors: [],
    };
    
    // Process each row
    for (let i = 0; i < rows.length; i++) {
      const row = rows[i];
      
      try {
        // Validate row
        const validatedData = await validateStudentRow(row);
        
        // Check if student exists
        const existing = await StudentService.findByEmail(
          validatedData.email
        );
        
        if (existing) {
          // Update existing
          await StudentService.update(existing.id, validatedData);
          results.updated++;
        } else {
          // Create new
          await StudentService.create({
            ...validatedData,
            orgId,
            teacherId,
          });
          results.created++;
        }
        
        // Update progress
        if (job) {
          await job.updateProgress(
            Math.floor(((i + 1) / rows.length) * 100)
          );
        }
      } catch (error) {
        results.failed++;
        results.errors.push({
          row: i + 1,
          error: error.message,
        });
      }
    }
    
    // Send completion notification
    if (notifyOnComplete && job?.userId) {
      await NotificationService.notify(job.userId, {
        type: 'bulk_import_completed',
        data: results,
      });
    }
    
    return results;
  },
  
  async validate(inputData) {
    if (!inputData.fileUrl) {
      throw new Error('fileUrl is required');
    }
    if (!inputData.orgId) {
      throw new Error('orgId is required');
    }
  },
  
  estimateDuration(inputData) {
    // Estimate 100ms per student row
    return 60000; // Default 1 minute
  },
  
  isRetryable(error) {
    // Retry on network/database errors, not validation errors
    return error.code === 'ECONNREFUSED' 
      || error.code === 'DATABASE_BUSY';
  },
};
```

### Job Handler Registry

```typescript
// lib/jobs/handlers/index.ts
export const JobHandlers = {
  // Bulk operations
  'bulk.import.students': BulkImportStudentsHandler,
  'bulk.import.teachers': BulkImportTeachersHandler,
  'bulk.export.scores': BulkExportScoresHandler,
  
  // Reports
  'report.teacher.progress': TeacherProgressReportHandler,
  'report.admin.usage': AdminUsageReportHandler,
  
  // Email
  'email.send.transactional': TransactionalEmailHandler,
  'email.send.bulk': BulkEmailHandler,
  
  // Webhooks
  'webhook.process.braintree': BraintreeWebhookHandler,
  'webhook.deliver.outbound': OutboundWebhookHandler,
  
  // Analytics
  'analytics.aggregate.daily': DailyAnalyticsHandler,
  
  // Maintenance
  'maintenance.cleanup.sessions': CleanupSessionsHandler,
  
  // Get handler by job type
  getHandler(jobType: string): JobHandler | null {
    return this[jobType] || null;
  },
};
```

---

## Job Status API

### Endpoints

#### 1. Enqueue Job (Internal)

Used by other API endpoints to enqueue background jobs.

```typescript
// Internal utility, not exposed as endpoint
import { JobQueue } from '@/lib/jobs/queue';

const jobId = await JobQueue.enqueue('bulk.import.students', {
  fileUrl: 's3://bucket/file.csv',
  orgId: 'org_123',
});

// Return job ID to client
return res.status(202).json({
  message: 'Import queued for processing',
  jobId,
  statusUrl: `/api/jobs/${jobId}`,
});
```

#### 2. Get Job Status

**Endpoint**: `GET /api/jobs/:jobId`

**Purpose**: Poll job status and progress

**Access**: User who created job, or System Admin

**Response**:
```typescript
{
  jobId: "job_abc123",
  jobType: "bulk.import.students",
  status: "RUNNING" | "COMPLETED" | "FAILED" | ...,
  priority: "HIGH" | "MEDIUM" | "LOW",
  createdAt: "2025-01-27T10:00:00Z",
  startedAt: "2025-01-27T10:00:05Z",
  completedAt: null,
  progress: 67,
  progressMessage: "Processing row 670 of 1000",
  attempts: 1,
  maxAttempts: 3,
  outputData: null,      // Populated when COMPLETED
  errorData: null,       // Populated when FAILED
  resourceLinks: [...]   // Links to created resources
}
```

**Implementation**:
```typescript
// pages/api/jobs/[jobId].ts
export default withAuth(async (req, res) => {
  const { jobId } = req.query;
  
  const job = await JobQueue.getById(jobId);
  
  if (!job) {
    return res.status(404).json({ error: 'Job not found' });
  }
  
  // Authorization check
  if (job.userId !== req.user.id && !req.user.isSystemAdmin) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  return res.json(job);
});
```

#### 3. Cancel Job

**Endpoint**: `DELETE /api/jobs/:jobId`

**Purpose**: Cancel a queued or running job

**Access**: User who created job, or System Admin

**Response**:
```typescript
{
  jobId: "job_abc123",
  status: "CANCELLED",
  cancelledAt: "2025-01-27T10:05:00Z"
}
```

#### 4. List User Jobs

**Endpoint**: `GET /api/jobs?userId=:userId&status=:status`

**Purpose**: List all jobs for a user

**Access**: User (own jobs), System Admin (all jobs)

**Query Parameters**:
- `userId` (optional): Filter by user
- `status` (optional): Filter by status
- `jobType` (optional): Filter by job type
- `limit` (optional): Max results (default 50)
- `offset` (optional): Pagination offset

**Response**:
```typescript
{
  jobs: [
    { jobId: "job_1", jobType: "bulk.import.students", status: "COMPLETED", ... },
    { jobId: "job_2", jobType: "report.teacher.progress", status: "RUNNING", ... }
  ],
  total: 42,
  limit: 50,
  offset: 0
}
```

---

## Retry and Error Handling

### Retry Strategy

**Exponential Backoff**:
```
Attempt 1: Immediate
Attempt 2: 5 seconds delay
Attempt 3: 10 seconds delay (5s * 2^1)
Attempt 4: 20 seconds delay (5s * 2^2)
...
```

**Max Attempts by Job Type**:
| Job Category | Max Attempts | Rationale |
|--------------|--------------|-----------|
| Bulk Operations | 3 | Validation errors unlikely to resolve |
| Reports | 5 | Temporary resource contention common |
| Email | 5 | External service may have transient failures |
| Webhooks | 10 | Network issues common, high reliability needed |
| Analytics | 3 | Can be recomputed on next schedule |
| Maintenance | 1 | Should not auto-retry; requires investigation |

### Retryable vs Non-Retryable Errors

**Retryable Errors** (will retry automatically):
- Network timeouts (`ETIMEDOUT`, `ECONNREFUSED`)
- Database busy (`DATABASE_BUSY`, `LOCK_TIMEOUT`)
- Rate limiting (`RATE_LIMITED`, `429`)
- Temporary service unavailability (`503`)

**Non-Retryable Errors** (will not retry):
- Validation errors (`INVALID_INPUT`)
- Authorization errors (`403`, `UNAUTHORIZED`)
- Resource not found (`404`, `NOT_FOUND`)
- Duplicate resource (`CONFLICT`, `DUPLICATE_KEY`)
- Business logic violations

---

## Monitoring and Observability

### Job Metrics

**Key Metrics** (tracked per job type):
- **Throughput**: Jobs processed per minute
- **Latency**: Average job duration (p50, p95, p99)
- **Success Rate**: Percentage of jobs that complete successfully
- **Retry Rate**: Percentage of jobs that require retries
- **Dead Letter Rate**: Percentage of jobs that end in dead letter queue
- **Queue Depth**: Number of queued jobs waiting for processing

### Telemetry Events

All job lifecycle events are tracked via Event Model (see [Event Model](../15-analytics-and-reporting/event-model.md)):

```typescript
{
  eventType: "job.queued",
  eventData: {
    jobId: "job_abc123",
    jobType: "bulk.import.students",
    userId: "usr_456",
    orgId: "org_789",
    priority: "HIGH",
    estimatedDuration: 120000
  },
  timestamp: "2025-01-27T10:00:00Z"
}
```

**Event Types**:
- `job.queued`
- `job.started`
- `job.progress` (emitted at 25%, 50%, 75% progress)
- `job.completed`
- `job.failed`
- `job.retrying`
- `job.timeout`
- `job.cancelled`
- `job.dead_letter`

### System Admin Dashboard

**Job Queue Health Dashboard** (`/sysadmin/jobs`):

**Metrics Displayed**:
- Jobs queued (by priority)
- Jobs running (by type)
- Jobs completed (last 24h)
- Jobs failed (last 24h)
- Average job duration (by type)
- Queue depth over time (chart)
- Failed jobs requiring attention

**Failed Job Inspector**:
- View error details
- Retry failed job manually
- View input/output data
- View related audit logs

**See**: [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md)

---

## Concurrency and Rate Limiting

### Concurrency Limits

To prevent resource exhaustion, jobs have concurrency limits:

| Job Category | Concurrent Jobs | Rationale |
|--------------|-----------------|-----------|
| Bulk Operations | 5 | Database write-heavy |
| Reports | 10 | CPU/memory intensive |
| Email | 50 | I/O bound, can parallelize |
| Webhooks | 30 | Network I/O, external service limits |
| Analytics | 3 | Database read-heavy |
| Content | 5 | Mixed I/O and CPU |
| Maintenance | 1 | Should not run concurrently |

**Implementation** (BullMQ):
```typescript
const worker = new Worker('mlc-jobs', handler, {
  concurrency: getConcurrencyLimit(jobType),
  limiter: {
    max: 100,      // Max 100 jobs
    duration: 1000, // Per second
  },
});
```

### Rate Limiting

**Per-User Limits**:
- Max 10 concurrent jobs per user
- Max 100 jobs per user per hour
- Max 1000 jobs per user per day

**Exceeded Limit Response**:
```typescript
{
  error: "RATE_LIMIT_EXCEEDED",
  message: "You have exceeded your job quota. Please try again later.",
  retryAfter: 3600, // seconds
  quotas: {
    hourly: { used: 100, limit: 100 },
    daily: { used: 856, limit: 1000 }
  }
}
```

---

## Timeouts

### Timeout Configuration

| Job Category | Default Timeout | Max Timeout |
|--------------|-----------------|-------------|
| Bulk Operations | 10 minutes | 30 minutes |
| Reports | 5 minutes | 15 minutes |
| Email | 30 seconds | 2 minutes |
| Webhooks | 30 seconds | 2 minutes |
| Analytics | 15 minutes | 1 hour |
| Content | 10 minutes | 30 minutes |
| Maintenance | 30 minutes | 2 hours |

**Timeout Handling**:
When a job exceeds its timeout:
1. Job is marked as `TIMEOUT`
2. Job process is terminated (best effort)
3. Job enters retry queue (if retries remaining)
4. Telemetry event `job.timeout` is emitted
5. User is notified if notification requested

---

## Dead Letter Queue

### Purpose

Jobs that fail all retry attempts are moved to the Dead Letter Queue (DLQ) for manual inspection and recovery.

### DLQ Management

**System Admin Actions** (`/sysadmin/jobs/dlq`):
1. **View Failed Jobs**: List all jobs in DLQ with error details
2. **Retry Job**: Manually re-enqueue failed job (resets attempts)
3. **Discard Job**: Permanently remove job from DLQ
4. **Export DLQ**: Export failed jobs to CSV for analysis

**Auto-Cleanup**:
- Jobs in DLQ older than 30 days are auto-archived
- Archived jobs retained for 90 days for audit purposes

---

## Performance Targets (SLAs)

| Metric | Target | Measurement |
|--------|--------|-------------|
| Job Queuing Latency | < 500ms | Time from enqueue to QUEUED state |
| Job Start Latency | < 30s | Time from QUEUED to RUNNING (p95) |
| Job Success Rate | > 95% | Percentage completing without DLQ |
| Email Delivery | < 2min | Time from trigger to external provider acceptance |
| CSV Import (1000 rows) | < 5min | End-to-end duration (p95) |
| Report Generation | < 2min | End-to-end duration (p95) |
| Webhook Processing | < 10s | Time from receipt to completion |

---

## Security Considerations

### Job Data Protection

**Sensitive Data Handling**:
- Passwords and secrets **never** stored in job input/output data
- PII encrypted at rest in job data (if applicable)
- Job data access restricted to job owner and System Admin

### Cron Endpoint Protection

**Authorization**:
```typescript
// pages/api/jobs/worker.ts
if (req.headers['authorization'] !== `Bearer ${process.env.CRON_SECRET}`) {
  return res.status(401).json({ error: 'Unauthorized' });
}
```

**Best Practices**:
- Generate strong random secret for `CRON_SECRET`
- Rotate secret periodically
- Use Vercel's secret management (encrypted environment variables)

---

## Supporting Documents Referenced

This background jobs specification draws from the following source documents:

- [Mass Upload.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Mass%20Upload.xlsx%20-%20Sheet1.csv) — Bulk upload job requirements
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Game import job specifications
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence processing jobs

## Dependencies

This specification relies on:
- [System Overview](system-overview.md) - Overall architecture context
- [Service Boundaries](service-boundaries.md) - Domain boundaries for job handlers
- [REST Endpoints](rest-endpoints.md) - API patterns for async operations
- [Webhooks](webhooks.md) - Webhook processing jobs
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry for job events
- [Bulk Ops Jobs](../20-imports-and-automation/bulk-ops-jobs.md) - Bulk operation specifics
- [Email Provider](../17-integrations/email-provider.md) - Email delivery infrastructure
- [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md) - Job monitoring UI

---

## Open Questions

### Technical Decisions Needed

1. **Job Queue Implementation Timeline**  
   **Question**: Should we start with database-backed queue or implement BullMQ + Redis from day one?  
   **Trade-off**: Database queue is simpler but less scalable; BullMQ requires Redis infrastructure but better performance.  
   **Recommendation**: Start with database queue, migrate to BullMQ when job volume exceeds 10,000/day or when latency becomes problematic.

2. **Job Retention Policy**  
   **Question**: How long should we retain completed/failed job records?  
   **Options**: 7 days, 30 days, 90 days  
   **Recommendation**: 30 days for completed, 90 days for failed (for debugging), archive to cold storage after.

3. **Webhook Delivery Isolation**  
   **Question**: Should webhook delivery use dedicated queue to isolate from bulk operations?  
   **Trade-off**: Isolation improves reliability but adds complexity.  
   **Recommendation**: Start with single queue, extract webhook queue if needed.

4. **Progress Tracking Granularity**  
   **Question**: How frequently should jobs update progress (database writes)?  
   **Options**: Every 10%, every 25%, time-based (every 10s)  
   **Recommendation**: Every 25% or every 10 seconds, whichever is less frequent (reduces database load).

5. **Job Result Storage**  
   **Question**: Should large job results (exports, reports) be stored in database or object storage?  
   **Trade-off**: Database is simpler but expensive for large files; object storage requires pre-signed URL generation.  
   **Recommendation**: Store results < 1MB in database, > 1MB in R2 with pre-signed URLs.

---

## Summary

The MLC LMS background job system provides:
- **Async processing** for long-running operations without blocking users
- **Reliable retry logic** with exponential backoff for transient failures
- **Observable job status** with real-time progress tracking
- **Scalable architecture** that grows from serverless to dedicated queue
- **Developer-friendly** handler pattern for implementing new job types
- **Operational visibility** via System Admin dashboard and telemetry

The system enables the platform to handle bulk data operations, report generation, email delivery, and scheduled maintenance reliably and at scale, while maintaining serverless-first simplicity for the initial implementation.
