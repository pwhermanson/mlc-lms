# Bulk Ops Jobs

## Purpose

This document defines the operational aspects of bulk operation jobs in the MLC-LMS platform - the user-facing job management, monitoring, and interaction patterns for long-running batch operations. While the general background job infrastructure is covered in [Background Jobs](../18-architecture/background-jobs.md), this specification focuses specifically on how administrators interact with, monitor, and manage bulk data operations such as CSV imports, exports, mass updates, and batch deletions.

Bulk operations are critical to the platform's ability to serve schools and institutions with hundreds or thousands of students. These operations typically involve:
- Processing large CSV files that cannot complete within HTTP request timeouts
- Providing real-time progress feedback to administrators during long-running operations
- Handling partial failures gracefully (some rows succeed, some fail)
- Enabling administrators to review, retry, and fix failed operations
- Maintaining audit trails for compliance and troubleshooting

This specification bridges three concerns:
1. **Background Job Infrastructure**: General async processing patterns ([Background Jobs](../18-architecture/background-jobs.md))
2. **Data Specifications**: CSV formats and validation rules ([CSV Spec Students](./csv-spec-students.md), [CSV Spec Teachers](./csv-spec-teachers.md), etc.)
3. **User Experience**: UI workflows for bulk operations ([Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md), [Member Management](../05-admin-subscriber-experience/member-management.md))

## Scope

### Included

**Bulk Operation Types and Workflows**:
- CSV import operations (students, teachers, courses/assignments, games)
- Data export operations (students, teachers, scores, activity logs)
- Mass update operations (bulk edit student/teacher records)
- Batch delete operations (remove multiple records)
- Mass assignment operations (assign students to teachers, courses to students)

**Job Management Interface**:
- Job status tracking and real-time progress updates
- Partial success handling (some rows succeed, some fail)
- Detailed error reporting per row/record
- Job cancellation and timeout handling
- Manual retry for failed rows
- Job history and audit logging

**User-Facing Features**:
- Import preview and validation (dry-run mode)
- Progress indicators and estimated completion times
- Success/error summary reports
- Downloadable error reports for fixing data
- Email notifications on job completion
- Job queue visibility (pending jobs)

**Data Quality and Validation**:
- Pre-import validation (file-level and row-level)
- Duplicate detection and resolution
- Referential integrity checks (e.g., teacher exists before assigning students)
- Data sanitization and normalization
- Validation error codes and messages

### Excluded

- General background job architecture and queue implementation (see [Background Jobs](../18-architecture/background-jobs.md))
- CSV file format specifications (see [CSV Spec Students](./csv-spec-students.md), [CSV Spec Teachers](./csv-spec-teachers.md), [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md))
- UI wireframes and detailed UX flows (see [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md))
- Data model definitions (see [Data Model ERD](../18-architecture/data-model-erd.md))
- API endpoint specifications (see [REST Endpoints](../18-architecture/rest-endpoints.md))

---

## Bulk Operation Categories

The platform supports the following categories of bulk operations, each with unique characteristics and requirements:

### 1. CSV Import Operations

**Purpose**: Create or update multiple records by uploading CSV files generated from external systems (SIS, HR databases, spreadsheets).

**Supported Import Types**:

| Import Type | Entity | Use Case | Typical Volume | Processing Time (p95) |
|-------------|--------|----------|----------------|-----------------------|
| Student Import | Students | Onboard class rosters at term start | 50-5,000 students | 30s - 5min |
| Teacher Import | Teachers | Setup staff accounts for new year | 5-200 teachers | 10s - 2min |
| Course Assignment Import | Assignments | Bulk assign sequences to students | 100-10,000 assignments | 1min - 10min |
| Game Import | Games | Load content library (System Admin) | 10-500 games | 2min - 15min |

**Processing Workflow**:
```
1. File Upload → 2. File Validation → 3. Preview/Dry Run → 4. User Confirms → 
5. Background Job Queued → 6. Row-by-Row Processing → 7. Completion Report
```

**Key Features**:
- **Validation-only mode (dry run)**: Validates entire file without committing changes
- **Partial success**: Valid rows are committed even if some rows fail
- **Duplicate detection**: Configurable behavior for existing records (skip, update, or error)
- **Row-level error reporting**: Each failed row includes error code and message
- **Downloadable error report**: Failed rows exported as CSV for correction and re-import

**See**:
- [CSV Spec Students](./csv-spec-students.md)
- [CSV Spec Teachers](./csv-spec-teachers.md)
- [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md)

---

### 2. Data Export Operations

**Purpose**: Extract data from the platform for analysis, archival, or migration to external systems.

**Supported Export Types**:

| Export Type | Entity | Use Case | Typical Volume | Processing Time (p95) |
|-------------|--------|----------|----------------|-----------------------|
| Student Roster Export | Students | Generate enrollment reports | 50-5,000 students | 30s - 2min |
| Teacher List Export | Teachers | HR reporting and payroll | 5-200 teachers | 10s - 1min |
| Score Export | Student Scores | Academic reporting, grade books | 1,000-50,000 scores | 1min - 10min |
| Activity Log Export | User Activity | Usage analytics, compliance audits | 10,000-100,000 logs | 2min - 15min |
| Full Data Export | All Entities | Account closure, data portability (GDPR) | Organization-wide | 5min - 30min |

**Processing Workflow**:
```
1. User Selects Export Type & Filters → 2. Background Job Queued → 
3. Data Extraction & CSV Generation → 4. File Storage (S3/R2) → 
5. Download Link Sent via Email/UI
```

**Key Features**:
- **Filtered exports**: Apply date ranges, roles, status filters before exporting
- **Format options**: CSV, Excel (XLSX), PDF (for reports)
- **Secure download links**: Time-limited pre-signed URLs (expires in 7 days)
- **Email notification**: Download link sent to user's email on completion
- **Large file handling**: Exports > 10MB compressed as ZIP files

**See**:
- [Export Specs](./export-specs.md)
- [Admin Reports](../05-admin-subscriber-experience/admin-reports.md)

---

### 3. Mass Update Operations

**Purpose**: Modify multiple records simultaneously without uploading CSV files (UI-driven bulk edits).

**Supported Update Types**:

| Update Type | Entity | Use Case | Typical Volume | Processing Time (p95) |
|-------------|--------|----------|----------------|-----------------------|
| Bulk Status Change | Students/Teachers | Activate/suspend accounts at term boundaries | 50-1,000 accounts | 10s - 1min |
| Mass Reassignment | Students | Move students between teachers/classes | 20-500 students | 10s - 2min |
| Bulk Course Assignment | Students | Assign new sequence to entire class | 20-100 students | 10s - 1min |
| Bulk Role Change | Teachers | Promote teachers to Teacher-Admin role | 5-50 teachers | 5s - 30s |

**Processing Workflow**:
```
1. User Selects Multiple Records → 2. User Chooses Bulk Action → 
3. User Confirms Changes → 4. Background Job Queued (if > 50 records) → 
5. Changes Applied → 6. Confirmation Report
```

**Key Features**:
- **Selection-based**: User selects records via checkboxes in UI
- **Action preview**: Shows summary of changes before confirmation
- **Atomic updates**: All-or-nothing mode optional (default: partial success allowed)
- **Undo capability**: Recent bulk updates can be rolled back (within 24 hours)
- **Change audit log**: All modifications tracked in audit log

**See**:
- [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md)
- [Member Management](../05-admin-subscriber-experience/member-management.md)

---

### 4. Batch Delete Operations

**Purpose**: Remove multiple records in a single operation.

**Supported Delete Types**:

| Delete Type | Entity | Use Case | Typical Volume | Processing Time (p95) |
|-------------|--------|----------|----------------|-----------------------|
| Bulk Student Deletion | Students | Remove graduated students at year-end | 50-1,000 students | 30s - 2min |
| Bulk Account Cleanup | Inactive Accounts | Archive unused trial accounts | 100-500 accounts | 30s - 1min |

**Processing Workflow**:
```
1. User Selects Records for Deletion → 2. System Shows Warnings (dependencies) → 
3. User Confirms (with typed confirmation) → 4. Background Job Queued → 
5. Records Soft-Deleted → 6. Confirmation Report
```

**Key Features**:
- **Soft delete**: Records marked as deleted but retained for 90 days (compliance)
- **Dependency warnings**: Shows related records that will be affected (assignments, scores)
- **Confirmation safeguard**: User must type "DELETE" to confirm bulk deletion
- **Cascade handling**: Related records (assignments, scores) handled per data model rules
- **Audit trail**: All deletions logged with user ID, timestamp, and reason

**See**:
- [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md)
- [Data Model ERD](../18-architecture/data-model-erd.md)

---

### 5. Mass Assignment Operations

**Purpose**: Assign learning content or students to teachers in bulk.

**Supported Assignment Types**:

| Assignment Type | Use Case | Typical Volume | Processing Time (p95) |
|-----------------|----------|----------------|-----------------------|
| Students → Teacher | Assign roster to teacher at term start | 20-200 students | 5s - 30s |
| Courses → Students | Assign sequence to entire class | 20-100 assignments | 10s - 1min |
| Courses → Teacher's Students | Teacher assigns sequence to all students | 20-100 assignments | 10s - 1min |

**Processing Workflow**:
```
1. User Selects Source (students or courses) → 2. User Selects Target (teacher or students) → 
3. User Confirms Assignment → 4. Background Job Queued (if > 50 assignments) → 
5. Assignments Created → 6. Notification Sent to Affected Users
```

**Key Features**:
- **Duplicate prevention**: Skip assignments that already exist
- **Batch notifications**: Single email summarizing all new assignments (not per-assignment)
- **Assignment preview**: Shows list of assignments to be created before confirmation
- **Conditional logic**: Skip students who already completed the sequence

**See**:
- [Mass Assignments](../05-admin-subscriber-experience/mass-assignments.md)
- [Assignment Generation](../10-assignments-engine/assignment-generation.md)

---

## Job Lifecycle and State Management

Bulk operation jobs follow the standard job lifecycle defined in [Background Jobs](../18-architecture/background-jobs.md), with additional states and behaviors specific to bulk operations.

### Bulk Operation States

```
┌─────────────┐
│  UPLOADED   │  CSV file uploaded, not yet validated (CSV imports only)
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ VALIDATING  │  File-level and row-level validation in progress
└──────┬──────┘
       │
       ├──────────────┐
       │              │
       ▼              ▼
┌─────────────┐   ┌──────────────┐
│  VALIDATED  │   │ VALIDATION   │  Validation failed (file-level errors)
│             │   │   FAILED     │  Terminal state for this upload
└──────┬──────┘   └──────────────┘
       │
       │ (User confirms import)
       ▼
┌─────────────┐
│   QUEUED    │  Job queued for background processing
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  PROCESSING │  Importing/exporting/updating records
└──┬───┬──┬───┘
   │   │  │
   │   │  └───────────────────┐
   │   │                      │
   │   └──────────┐           │
   │              │           │
   ▼              ▼           ▼
┌─────────────┐ ┌──────────┐ ┌──────────────┐
│  COMPLETED  │ │ PARTIAL  │ │   FAILED     │
│  (100%)     │ │ SUCCESS  │ │  (Critical   │
└─────────────┘ │  (Mixed) │ │   Error)     │
                └──────────┘ └──────────────┘
```

### State Definitions

| State | Description | User Actions Available | Next States |
|-------|-------------|------------------------|-------------|
| `UPLOADED` | CSV file uploaded to server, awaiting validation | Cancel | `VALIDATING`, `CANCELLED` |
| `VALIDATING` | Validation in progress (file format, row validation) | Cancel | `VALIDATED`, `VALIDATION_FAILED`, `CANCELLED` |
| `VALIDATED` | Validation passed, ready for user confirmation | Confirm Import, Cancel | `QUEUED`, `CANCELLED` |
| `VALIDATION_FAILED` | Validation errors found, user must fix data | Download Error Report | *(terminal)* |
| `QUEUED` | Job queued for processing | Cancel (if not started) | `PROCESSING`, `CANCELLED` |
| `PROCESSING` | Background job actively processing records | Cancel (best effort) | `COMPLETED`, `PARTIAL_SUCCESS`, `FAILED` |
| `COMPLETED` | All records processed successfully (100% success) | Download Report, View Details | *(terminal)* |
| `PARTIAL_SUCCESS` | Some records succeeded, some failed | Download Error Report, Retry Failed Rows | *(terminal)* |
| `FAILED` | Critical error prevented completion | View Error, Retry Job | *(terminal)* |
| `CANCELLED` | User cancelled job | - | *(terminal)* |

### Processing Modes

Bulk operations support two validation modes before committing changes:

#### 1. Validation-Only Mode (Dry Run)

**Purpose**: Test CSV file validity without making any changes to the database.

**Workflow**:
1. User uploads CSV file and selects "Validate Only"
2. System validates file format and each row
3. System returns detailed validation report (no database changes)
4. User fixes errors in CSV
5. User re-uploads until validation passes
6. User switches to "Import Mode" to commit

**Use Cases**:
- First-time bulk import by unfamiliar administrators
- Large imports where errors are expected
- Testing CSV format before production import
- Training and documentation

**Output**:
- Validation report showing all errors (grouped by error code)
- Statistics: total rows, valid rows, invalid rows, error breakdown
- Sample of first 10 errors (with row numbers and messages)
- No records created or modified

#### 2. Import Mode (Commit)

**Purpose**: Process the CSV file and commit changes to the database.

**Workflow**:
1. User uploads CSV file and selects "Import"
2. System validates file (brief validation, < 5 seconds)
3. If validation passes, job queued for background processing
4. Background worker processes rows (commits valid rows, skips invalid)
5. User receives completion notification with summary

**Partial Success Behavior**:
- Valid rows are committed to database
- Invalid rows are skipped and logged
- Job completes as `PARTIAL_SUCCESS` if any failures
- User receives error report for failed rows only

**Example Scenario**:
- CSV with 500 students
- 485 valid rows → 485 students created
- 15 invalid rows → Skipped with error report
- Job status: `PARTIAL_SUCCESS` (97% success rate)
- User downloads error report, fixes 15 rows, re-imports

---

## Job Progress Tracking

Bulk operations provide real-time progress feedback to administrators during processing.

### Progress Calculation

Progress is calculated as:
```
Progress % = (Processed Rows / Total Rows) × 100
```

**Update Frequency**:
- Progress updates every **25%** or **every 10 seconds**, whichever is less frequent
- This reduces database write load while providing responsive feedback

**Example**:
- 1,000 row CSV
- Progress updates at: 25%, 50%, 75%, 100%
- Or: 10s, 20s, 30s, ... (if processing slower than 4 rows/second)

### Progress Indicators (UI)

**Progress Bar**:
- Shows percentage complete (0-100%)
- Displays estimated time remaining (calculated from processing rate)
- Color-coded:
  - Blue: Processing normally
  - Yellow: Slowed (> 2x expected duration)
  - Red: Critical errors detected (job may fail)

**Detailed Progress Message**:
```
Processing row 487 of 1,000
- Created: 470 students
- Updated: 10 students  
- Failed: 7 rows (view errors)
- Estimated time remaining: 2 minutes
```

**Polling Interval**:
- UI polls job status every **2 seconds** while job is `PROCESSING`
- Polling stops when job reaches terminal state (`COMPLETED`, `PARTIAL_SUCCESS`, `FAILED`)

---

## Error Handling and Reporting

### Error Types

Bulk operations distinguish between three levels of errors:

#### 1. File-Level Errors (Critical)

**Definition**: Errors that prevent the entire file from being processed.

**Examples**:
- Invalid file format (not a valid CSV)
- File too large (> 10MB)
- Missing required columns in header row
- Invalid character encoding (not UTF-8)
- File is empty or contains only header row

**Handling**:
- Job immediately transitions to `VALIDATION_FAILED`
- No rows are processed
- User must fix file and re-upload

**Error Codes**:
- `ERR_INVALID_FILE_FORMAT`
- `ERR_FILE_TOO_LARGE`
- `ERR_MISSING_REQUIRED_COLUMN`
- `ERR_INVALID_ENCODING`
- `ERR_EMPTY_FILE`

#### 2. Row-Level Errors (Non-Critical)

**Definition**: Errors in individual rows that do not prevent other rows from processing.

**Examples**:
- Duplicate username
- Invalid email format
- Teacher not found (for student import)
- Required field missing or invalid
- Data type mismatch

**Handling**:
- Row is skipped and error logged
- Processing continues with remaining rows
- Job completes as `PARTIAL_SUCCESS`
- User downloads error report for failed rows

**Error Codes**:
- `ERR_USERNAME_DUPLICATE`
- `ERR_EMAIL_INVALID`
- `ERR_TEACHER_NOT_FOUND`
- `ERR_REQUIRED_FIELD_MISSING`
- `ERR_DATA_TYPE_INVALID`

*See CSV spec documents for complete error code lists:*
- [CSV Spec Students](./csv-spec-students.md#validation-rules)
- [CSV Spec Teachers](./csv-spec-teachers.md#validation-rules)

#### 3. System Errors (Critical)

**Definition**: Infrastructure or system failures that prevent job completion.

**Examples**:
- Database connection failure
- Out of memory
- Job timeout exceeded
- External service unavailable (email provider, storage)

**Handling**:
- Job transitions to `FAILED`
- All processed rows up to failure point are committed (not rolled back)
- Job can be manually retried by System Admin
- Alert sent to engineering team (high severity)

**Error Codes**:
- `ERR_DATABASE_CONNECTION_FAILED`
- `ERR_OUT_OF_MEMORY`
- `ERR_JOB_TIMEOUT`
- `ERR_EXTERNAL_SERVICE_UNAVAILABLE`

### Error Reports

When a job completes with errors (`PARTIAL_SUCCESS` or `VALIDATION_FAILED`), the system generates a downloadable error report.

**Error Report Format**: CSV file

**Columns**:
| Column | Description | Example |
|--------|-------------|---------|
| `row_number` | Row number in original CSV (1-indexed, excludes header) | `47` |
| `error_code` | Machine-readable error code | `ERR_USERNAME_DUPLICATE` |
| `error_message` | Human-readable error description | `Username 'SmithJohn17502' already exists` |
| `suggested_fix` | Actionable guidance for fixing error | `Change username or leave blank for auto-generation` |
| *(original columns)* | All columns from original CSV row | *(varies)* |

**Example Error Report**:
```csv
row_number,error_code,error_message,suggested_fix,student_first_name,student_last_name,student_username,student_password,teacher_real_name
47,ERR_USERNAME_DUPLICATE,Username 'SmithJohn17502' already exists,Change username or leave blank for auto-generation,John,Smith,SmithJohn17502,music,Jane Doe
103,ERR_TEACHER_NOT_FOUND,Teacher 'Bob Martin' not found in organization,Verify teacher name spelling or leave blank,Emily,Johnson,JohnsonEmily17502,piano,Bob Martin
256,ERR_EMAIL_INVALID,Email 'student@invalid' is not valid,Provide valid email or leave blank,Michael,Williams,WilliamsMichael17502,note,student@invalid
```

**User Workflow**:
1. User downloads error report
2. User opens in Excel/Google Sheets
3. User corrects errors in the rows
4. User saves as new CSV file
5. User re-uploads corrected CSV (only failed rows, or full file)
6. System processes corrected rows

---

## Job Cancellation and Timeout

### User-Initiated Cancellation

**When Available**: User can cancel job in states: `UPLOADED`, `VALIDATING`, `VALIDATED`, `QUEUED`, `PROCESSING`

**Behavior by State**:

| State | Cancellation Behavior | Database Changes |
|-------|----------------------|------------------|
| `UPLOADED` | Immediate cancellation, file deleted | None |
| `VALIDATING` | Validation stops, partial results discarded | None |
| `VALIDATED` | Immediate cancellation, file deleted | None |
| `QUEUED` | Job removed from queue | None |
| `PROCESSING` | Best-effort stop signal sent to worker | Rows processed before cancellation are committed |

**Processing State Cancellation**:
- User clicks "Cancel Job" button
- System sets job status to `CANCELLING`
- Background worker checks cancellation flag after each row
- Worker stops processing and transitions job to `CANCELLED`
- Already-processed rows are **not rolled back** (committed)

**UI Messaging**:
```
Cancelling job...
- Processed 347 of 1,000 rows before cancellation
- 340 students created successfully
- 7 rows failed (view error report)
```

### Timeout Handling

**Timeout Limits** (per job type):

| Operation Type | Default Timeout | Max Timeout |
|----------------|-----------------|-------------|
| Student CSV Import | 10 minutes | 30 minutes |
| Teacher CSV Import | 5 minutes | 15 minutes |
| Data Export | 10 minutes | 30 minutes |
| Mass Update | 5 minutes | 15 minutes |
| Batch Delete | 5 minutes | 15 minutes |

**Timeout Behavior**:
1. Job exceeds timeout limit
2. Worker process terminates (best effort)
3. Job transitions to `FAILED` with error code `ERR_JOB_TIMEOUT`
4. Already-processed rows are committed (not rolled back)
5. User can retry job or contact support

**Timeout Prevention**:
- Large CSV files (> 5,000 rows) automatically split into multiple batches
- Each batch processed as separate job (max 5,000 rows per job)
- User sees aggregate progress across all batches

---

## Retry and Recovery

### Failed Row Retry

When a job completes as `PARTIAL_SUCCESS`, administrators can retry failed rows without re-processing successful rows.

**Retry Workflow**:
1. User views job completion report
2. User clicks "Retry Failed Rows"
3. System creates new job with only failed rows
4. New job processes failed rows (may succeed if transient issues resolved)
5. User receives updated completion report

**Retry Strategy**:
- Failed rows are re-validated (data may have changed since first attempt)
- Errors may resolve automatically (e.g., teacher was created in between attempts)
- Persistent errors require user to download error report and fix data

**Retry Limits**:
- Maximum **3 automatic retries** per row
- After 3 failures, row requires manual correction

### Full Job Retry

When a job fails with system error (`FAILED` state), System Admins can manually retry the entire job.

**System Admin Actions**:
1. Navigate to System Admin → Jobs → Failed Jobs
2. Select failed job
3. Click "Retry Job"
4. System creates new job with same input data
5. New job queued for processing

**Retry Conditions**:
- Only System Admins can retry failed jobs
- Original job must be in `FAILED` state (not `VALIDATION_FAILED` or `PARTIAL_SUCCESS`)
- Retry resets all progress (starts from row 1)
- Original job retained for audit trail

---

## Job History and Audit Trail

All bulk operations are logged for audit, compliance, and troubleshooting purposes.

### Job History Records

**Stored Fields**:
- Job ID (unique identifier)
- Operation type (import, export, update, delete, assign)
- Entity type (students, teachers, assignments, etc.)
- User ID (who initiated the job)
- Organization ID (context)
- Job status (completed, partial success, failed, cancelled)
- File name (for CSV operations)
- Total rows / Processed rows / Successful rows / Failed rows
- Start timestamp / End timestamp / Duration
- Error summary (error codes and counts)

**Retention Policy**:
- Job records retained for **2 years**
- CSV files (uploaded and error reports) retained for **90 days** (encrypted)
- Audit logs retained per [Audit Logs](../06-system-admin/sysadmin-audit-logs.md) policy

### Job History UI

**Administrator Views**:

**Subscriber-Admin / Teacher-Admin**:
- View job history for their organization only
- Filter by: operation type, status, date range, initiated by user
- See summary statistics (success rate, avg duration)
- Download job reports and error files (within retention period)

**System Admin**:
- View all jobs across all organizations
- Advanced filtering: organization, job type, status, error codes
- Drill down to individual job details
- Retry failed jobs
- Export job history for analysis

**Job Detail View**:
```
Job ID: job_abc123def456
Type: Student CSV Import
Status: Partial Success (97% success rate)
Initiated by: Jane Doe (jane@school.edu)
Organization: Pine Valley School District
Started: 2025-10-12 14:30:00 UTC
Completed: 2025-10-12 14:32:35 UTC
Duration: 2 minutes 35 seconds

Results:
- Total Rows: 500
- Successfully Created: 485 students
- Failed: 15 rows
- Error Breakdown:
  - ERR_USERNAME_DUPLICATE: 10 rows
  - ERR_TEACHER_NOT_FOUND: 5 rows

Actions:
- Download Completion Report
- Download Error Report
- Retry Failed Rows
```

---

## Notifications

Users receive notifications when bulk operations complete (or fail).

### Notification Triggers

| Event | Notification Type | Recipients | Timing |
|-------|-------------------|-----------|---------|
| Import Completed | In-app + Email | User who initiated job | Immediately on completion |
| Import Failed | In-app + Email | User who initiated job | Immediately on failure |
| Partial Success | In-app + Email | User who initiated job | Immediately on completion |
| Export Ready | In-app + Email (with download link) | User who initiated job | When file ready |
| Job Cancelled | In-app | User who initiated job | Immediately on cancellation |

### Notification Content

**Email Template (Import Completed)**:
```
Subject: Student Import Completed - 485 of 500 students created

Hi Jane,

Your student import has completed successfully.

Import Summary:
- Total Rows: 500
- Successfully Created: 485 students
- Failed: 15 rows (3% failure rate)

Next Steps:
- View detailed results: [View Report]
- Download error report to fix failed rows: [Download]
- Need help? Contact support: support@mlc.com

Thanks,
MLC LMS Team
```

**In-App Notification**:
```
🎉 Student Import Completed
485 of 500 students created successfully
15 rows failed - Download error report to fix
[View Details] [Dismiss]
```

**See**: [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md), [Email Templates](../13-messaging-and-notifications/email-templates.md)

---

## Performance and Scalability

### Processing Performance Targets

| Operation Type | Volume | Target Duration (p95) | Throughput Target |
|----------------|--------|----------------------|-------------------|
| Student CSV Import | 1,000 students | 2 minutes | 8-10 rows/second |
| Teacher CSV Import | 100 teachers | 30 seconds | 3-5 rows/second |
| Data Export | 5,000 records | 2 minutes | 40-50 records/second |
| Mass Update | 500 records | 1 minute | 8-10 records/second |
| Batch Delete | 1,000 records | 2 minutes | 8-10 records/second |

### Large File Handling

**Automatic Batching**:
- CSV files with > 5,000 rows are automatically split into batches
- Each batch: max 5,000 rows
- Batches processed sequentially (not parallel) to avoid database contention
- User sees aggregate progress across all batches

**Example**:
- CSV with 12,000 students
- Split into 3 batches: [1-5,000], [5,001-10,000], [10,001-12,000]
- Batch 1 processed first → Batch 2 → Batch 3
- User sees: "Processing row 7,500 of 12,000 (62% complete)"

### Concurrency Limits

**Per-Organization Limits**:
- Max **5 concurrent bulk jobs** per organization
- Additional jobs queued until slot available
- Prevents resource exhaustion and database lock contention

**System-Wide Limits**:
- Max **50 concurrent bulk jobs** across all organizations (Phase 1)
- Scalable to 200+ concurrent jobs with dedicated job queue (Phase 2)

**See**: [Background Jobs - Concurrency and Rate Limiting](../18-architecture/background-jobs.md#concurrency-and-rate-limiting)

---

## Telemetry

Bulk operation events follow the standard event model defined in [Event Model](../15-analytics-and-reporting/event-model.md).

### Key Events

#### `bulk_job_started`
**Fires when**: Bulk operation job begins processing (transitions from `QUEUED` to `PROCESSING`)

**Properties**:
```json
{
  "event_type": "bulk_job_started",
  "job_id": "job_abc123def456",
  "job_type": "student_import | teacher_import | data_export | mass_update | batch_delete",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "total_rows": 500,
  "file_name": "students_fall_2025.csv",
  "operation_mode": "import | validation_only",
  "timestamp": "2025-10-12T14:30:00Z"
}
```

#### `bulk_job_completed`
**Fires when**: Bulk operation job completes (success, partial success, or failure)

**Properties**:
```json
{
  "event_type": "bulk_job_completed",
  "job_id": "job_abc123def456",
  "job_type": "student_import",
  "status": "completed | partial_success | failed",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "total_rows": 500,
  "successful_rows": 485,
  "failed_rows": 15,
  "processing_duration_ms": 155000,
  "error_codes": ["ERR_USERNAME_DUPLICATE", "ERR_TEACHER_NOT_FOUND"],
  "error_code_counts": {
    "ERR_USERNAME_DUPLICATE": 10,
    "ERR_TEACHER_NOT_FOUND": 5
  },
  "timestamp": "2025-10-12T14:32:35Z"
}
```

#### `bulk_job_cancelled`
**Fires when**: User cancels bulk operation job

**Properties**:
```json
{
  "event_type": "bulk_job_cancelled",
  "job_id": "job_abc123def456",
  "job_type": "student_import",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "rows_processed_before_cancel": 347,
  "successful_rows": 340,
  "failed_rows": 7,
  "timestamp": "2025-10-12T14:31:15Z"
}
```

### Monitoring Metrics

**Operational Dashboards** (System Admin):
- Bulk job success rate (by operation type)
- Average job duration (by operation type and file size)
- Jobs in progress (by organization)
- Failed job queue depth
- Most common error codes (last 7 days)

**See**: [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md)

---

## Permissions

Bulk operation access is controlled by role-based permissions.

### Role-Based Access

| Role | Can Import CSV | Can Export Data | Can Mass Update | Can Batch Delete | Scope |
|------|---------------|-----------------|-----------------|------------------|-------|
| **Subscriber-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Entire organization |
| **Teacher-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes (with confirmation) | Entire organization |
| **Teacher** | ❌ No | ✅ Limited (own students only) | ❌ No | ❌ No | Own students only |
| **Student** | ❌ No | ❌ No | ❌ No | ❌ No | N/A |
| **System-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Any organization (with audit) |

**Permission Codes**:
- `UPU` (Upload Utility Use): Required for CSV import operations
- `BED` (Bulk Edit): Required for mass update operations
- `BDE` (Bulk Delete): Required for batch delete operations
- `DEX` (Data Export): Required for data export operations

**See**: [Permissions Table](../02-roles-and-permissions/permissions-table.md)

### Subscription Plan Restrictions

| Plan | Max Concurrent Jobs | Max File Size | Max Rows per Import |
|------|---------------------|---------------|---------------------|
| **PRELUDE** | 1 | 1 MB | 5 rows (matches seat limit) |
| **SOLO** | 2 | 1 MB | 5 rows (matches seat limit) |
| **ENSEMBLE** | 5 | 10 MB | 50,000 rows |

**See**: [Seat Management](../14-payments-and-subscriptions/seat-management.md)

---

## Supporting Documents Referenced

This bulk operations specification draws from the following source documents:

- [Mass Upload.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Mass%20Upload.xlsx%20-%20Sheet1.csv) — Legacy student bulk upload format informing CSV import validation rules and error handling patterns
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Legacy game import format informing bulk content import workflows and metadata validation
- [Course import.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Course%20import.xlsx%20-%20Sheet1.csv) — Legacy course/assignment import format informing curriculum bulk operations and assignment creation workflows
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role specifications informing bulk operation permissions and role-based access control for import/export features
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator tools and job management UI informing bulk operations monitoring dashboards and admin workflows
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional admin interface specifications for job queue management, progress tracking, and error report generation
- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser compatibility requirements informing file upload component specifications and CSV processing limitations

---

## Dependencies

This specification relies on the following systems and specifications:

### Internal Dependencies

| Dependency | Purpose | Link |
|------------|---------|------|
| **Background Jobs** | General async job infrastructure and queue management | [Background Jobs](../18-architecture/background-jobs.md) |
| **CSV Spec Students** | Student CSV file format and validation rules | [CSV Spec Students](./csv-spec-students.md) |
| **CSV Spec Teachers** | Teacher CSV file format and validation rules | [CSV Spec Teachers](./csv-spec-teachers.md) |
| **CSV Spec Courses Assignments** | Course/assignment CSV format and validation rules | [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md) |
| **Student Bulk Operations** | UI workflows for bulk student management | [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md) |
| **Member Management** | UI workflows for member/teacher management | [Member Management](../05-admin-subscriber-experience/member-management.md) |
| **Event Model** | Telemetry event structure for tracking bulk operations | [Event Model](../15-analytics-and-reporting/event-model.md) |
| **Permissions Table** | Role-based access control for bulk operations | [Permissions Table](../02-roles-and-permissions/permissions-table.md) |
| **Data Model ERD** | Entity relationships and cascade rules | [Data Model ERD](../18-architecture/data-model-erd.md) |
| **Sysadmin Ops Dashboards** | Job monitoring and failed job management UI | [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md) |

### External Dependencies

| Dependency | Purpose | Notes |
|------------|---------|-------|
| **CSV Parser Library** | Parse and validate CSV file format | Use `csv-parse` (Node.js) or equivalent |
| **Background Job Queue** | Async job processing (Redis Queue, BullMQ) | See [Background Jobs](../18-architecture/background-jobs.md) |
| **Object Storage** | Store uploaded CSV files and error reports | S3-compatible (Cloudflare R2) |
| **Email Provider** | Send job completion notifications | See [Email Provider](../17-integrations/email-provider.md) |

---

## Open Questions

### Technical Decisions Needed

1. **Batch Size Optimization**  
   **Question**: What is the optimal batch size for splitting large CSV files?  
   **Current Spec**: 5,000 rows per batch  
   **Options**: 1,000 / 5,000 / 10,000 rows  
   **Trade-off**: Smaller batches = more jobs, better progress feedback; Larger batches = fewer jobs, better throughput  
   **Owner**: Backend tech lead  
   **Due**: Phase 1 technical design

2. **Row Processing Parallelism**  
   **Question**: Should rows within a batch be processed in parallel or sequentially?  
   **Current Spec**: Sequential processing (one row at a time)  
   **Options**: Sequential / Parallel (5-10 rows at a time)  
   **Trade-off**: Parallel = faster but more complex error handling and database contention  
   **Owner**: Backend tech lead  
   **Due**: Phase 1 technical design

3. **Failed Row Storage**  
   **Question**: Where should failed row details be stored (database vs object storage)?  
   **Current Spec**: Database for error metadata, object storage for full error reports  
   **Trade-off**: Database = faster queries but more expensive; Object storage = cheaper but requires pre-signed URLs  
   **Owner**: Backend tech lead  
   **Due**: Phase 1 technical design

### UX and Product Questions

4. **Undo Capability**  
   **Question**: Should bulk operations support "undo" functionality?  
   **Current Spec**: Soft delete allows recovery within 90 days, but no built-in undo  
   **Options**: No undo / Time-limited undo (24 hours) / Full undo  
   **Trade-off**: Undo = better UX but complex implementation and database overhead  
   **Owner**: Product designer, Product manager  
   **Due**: Phase 1 UX design sprint

5. **Real-time Progress Streaming**  
   **Question**: Should progress updates use polling or WebSocket streaming?  
   **Current Spec**: HTTP polling every 2 seconds  
   **Options**: Polling / WebSocket / Server-Sent Events (SSE)  
   **Trade-off**: WebSocket = real-time but more complex infrastructure; Polling = simple but higher latency  
   **Owner**: Backend tech lead, Product designer  
   **Due**: Phase 1 technical design

6. **Error Report Format Options**  
   **Question**: Should error reports be available in multiple formats (CSV, Excel, PDF)?  
   **Current Spec**: CSV only  
   **Options**: CSV only / CSV + Excel / CSV + Excel + PDF  
   **Trade-off**: Multiple formats = better UX but more implementation work  
   **Owner**: Product designer  
   **Due**: Phase 1 UX design sprint

---

## Summary

The MLC-LMS bulk operations system provides administrators with powerful, reliable tools for managing large-scale data operations:

- **Multiple Operation Types**: CSV imports, data exports, mass updates, batch deletes, and mass assignments
- **Validation-First Workflow**: Dry-run mode prevents errors before committing changes
- **Partial Success Handling**: Valid rows commit even if some rows fail, maximizing import success
- **Real-Time Progress**: Live progress tracking with estimated completion times
- **Detailed Error Reporting**: Row-level error reports with actionable guidance for fixing data
- **Retry and Recovery**: Failed rows can be retried; system errors can be manually retried by admins
- **Audit and Compliance**: Full job history and audit trails for regulatory compliance
- **Performance at Scale**: Handles imports up to 50,000 rows with automatic batching

The system balances ease of use, reliability, and performance to serve schools with thousands of students while maintaining data quality and providing administrators with visibility and control throughout the bulk operation lifecycle.
