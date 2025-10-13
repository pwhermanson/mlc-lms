# Export Specifications

## Purpose

This document defines the comprehensive export capabilities of the MLC-LMS platform, enabling users across all roles to extract data in standardized formats for backup, analysis, reporting, migration, compliance, and integration with external systems. Export functionality is a critical operational requirement that supports data portability, institutional record-keeping, system migrations, and business intelligence workflows.

The export system provides a unified framework for extracting diverse data types—including curriculum content, user data, progress records, financial transactions, and analytics—while ensuring data security, format consistency, and compliance with retention policies.

## Scope

### Included
- Overview of all export types and their purposes across the platform
- Common export formats and format-specific specifications (CSV, JSON, PDF, Excel)
- Shared export infrastructure and job management
- Export security model (permissions, encryption, expiration)
- File naming conventions and metadata standards
- Export delivery methods (download, email, API, scheduled)
- Data retention and archival policies for exported data
- Cross-cutting export concerns (encoding, compression, large file handling)
- Export telemetry and monitoring
- Compliance requirements for exported data

### Excluded
- Detailed CSV column specifications for each entity type (see entity-specific CSV specs)
- Import workflows and file validation (see import-related specs)
- API endpoint implementation details (see [REST Endpoints](../18-architecture/rest-endpoints.md))
- Background job queue implementation (see [Background Jobs](../18-architecture/background-jobs.md))
- Report generation logic and KPI calculations (see [Analytics and Reporting](../15-analytics-and-reporting/) documentation)

## Export Types Overview

The MLC-LMS platform supports exports across multiple domains, each serving distinct use cases and user roles:

### 1. Curriculum and Content Exports

**Sequences and Curriculum**
- **Purpose**: Export sequence structures, assignment groups, and learning pathways
- **Users**: Content Editors, Teachers, System Admins
- **Formats**: CSV, JSON, PDF summary
- **Detailed Spec**: See [Sequence Exports](../09-sequences-and-curriculum-tools/sequence-exports.md)
- **Use Cases**: Backup, sharing between organizations, method book distribution, curriculum planning

**Games Registry**
- **Purpose**: Export game metadata, taxonomy, and configuration data
- **Users**: Content Editors, System Admins
- **Formats**: CSV, JSON
- **Related Spec**: See [Games Importer](../08-games-registry-and-authoring/games-importer.md) for inverse operation
- **Use Cases**: Content audits, game catalog generation, migration to external systems

### 2. User and Organization Data Exports

**Student Data**
- **Purpose**: Export student accounts, profiles, and enrollment data
- **Users**: Subscriber-Admins, Teacher-Admins, System Admins
- **Formats**: CSV, Excel
- **Related Spec**: See [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md)
- **CSV Spec**: See [CSV Spec Students](./csv-spec-students.md) for import schema (export follows same column structure)
- **Use Cases**: Year-end archival, SIS synchronization, compliance reporting, data migration

**Teacher Data**
- **Purpose**: Export teacher accounts, class assignments, and organizational hierarchy
- **Users**: Subscriber-Admins, System Admins
- **Formats**: CSV, Excel
- **Related Spec**: See [Member Management](../05-admin-subscriber-experience/member-management.md)
- **CSV Spec**: See [CSV Spec Teachers](./csv-spec-teachers.md) for import schema (export follows same column structure)
- **Use Cases**: HR integration, organizational reporting, directory generation

**Organization Data**
- **Purpose**: Export organization profiles, hierarchies, and settings
- **Users**: System Admins
- **Formats**: JSON, CSV
- **Use Cases**: System backups, organization migrations, compliance documentation

### 3. Progress and Analytics Exports

**Student Progress and Performance**
- **Purpose**: Export detailed student assignment completion, game scores, mastery data
- **Users**: Teachers, Subscriber-Admins, System Admins
- **Formats**: CSV, Excel, PDF
- **Related Spec**: See [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md)
- **Use Cases**: Grade books, parent reports, IEP documentation, learning analytics

**Teacher Reports**
- **Purpose**: Export classroom analytics, student progress summaries, engagement metrics
- **Users**: Teachers, Subscriber-Admins
- **Formats**: CSV, PDF, Excel
- **Related Spec**: See [Teacher Reports](../04-teacher-experience/teacher-reports.md)
- **Use Cases**: Progress reports, parent-teacher conferences, instructional planning

**Admin Analytics**
- **Purpose**: Export organization-wide analytics, utilization reports, learning outcomes
- **Users**: Subscriber-Admins, System Admins
- **Formats**: CSV, Excel, PDF
- **Related Spec**: See [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md)
- **Use Cases**: Board reports, grant reporting, strategic planning, ROI analysis

**Operational Dashboards**
- **Purpose**: Export system health metrics, performance data, usage statistics
- **Users**: System Admins
- **Formats**: CSV, JSON
- **Related Spec**: See [Observability](../19-quality-and-operations/observability.md)
- **Use Cases**: System monitoring, capacity planning, incident analysis

### 4. Financial and Billing Exports

**Billing Statements**
- **Purpose**: Export invoices, payment history, subscription details
- **Users**: Subscriber-Admins, System Admins
- **Formats**: PDF, CSV
- **Related Spec**: See [Billing Statements On Demand](../14-payments-and-subscriptions/billing-statements-on-demand.md)
- **Use Cases**: Accounting integration, audit trails, financial reconciliation

**Transaction History**
- **Purpose**: Export detailed payment transactions, refunds, adjustments
- **Users**: Subscriber-Admins, System Admins
- **Formats**: CSV, Excel
- **Use Cases**: Financial audits, tax reporting, budget tracking

**Seat Utilization Reports**
- **Purpose**: Export seat allocation, usage patterns, capacity forecasting
- **Users**: Subscriber-Admins, System Admins
- **Formats**: CSV, PDF
- **Related Spec**: See [Seat Management](../14-payments-and-subscriptions/seat-management.md)
- **Use Cases**: Budget planning, subscription optimization, usage analysis

### 5. Audit and Compliance Exports

**Audit Logs**
- **Purpose**: Export system access logs, user actions, administrative changes
- **Users**: System Admins, Compliance Officers
- **Formats**: CSV, JSON
- **Related Spec**: See [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)
- **Use Cases**: Security audits, compliance reporting, incident investigation

**User Activity Logs**
- **Purpose**: Export student login history, session data, activity timestamps
- **Users**: Subscriber-Admins (for their org), System Admins
- **Formats**: CSV
- **Use Cases**: FERPA compliance, attendance tracking, usage verification

**Data Retention Reports**
- **Purpose**: Export data lifecycle status, retention schedules, deletion logs
- **Users**: System Admins, Compliance Officers
- **Formats**: CSV, JSON
- **Use Cases**: GDPR compliance, data governance, legal holds

## Export Formats

### CSV (Comma-Separated Values)

**Purpose**: Universal interchange format for tabular data, compatible with spreadsheet applications and data processing tools

**Characteristics**:
- **Encoding**: UTF-8 with BOM for Excel compatibility (configurable)
- **Delimiter**: Comma (`,`) by default; tab-delimited option available for text fields with embedded commas
- **Line Endings**: CRLF (`\r\n`) for Windows compatibility
- **Quoting**: Double quotes (`"`) for text fields containing delimiters, line breaks, or quotes
- **Escape Character**: Double quote (`""`) to represent literal quote characters
- **Header Row**: First row contains column names (required)
- **Null Values**: Empty string for null/missing values (configurable to use `NULL` keyword)
- **Date Format**: ISO 8601 (`YYYY-MM-DD` for dates, `YYYY-MM-DDTHH:mm:ssZ` for timestamps)
- **Boolean Values**: `true`/`false` (lowercase) or `1`/`0` depending on target system
- **Number Format**: Decimal point (`.`) without thousands separators

**Use Cases**:
- Bulk imports to external systems (SIS, LMS, accounting software)
- Data analysis in spreadsheet applications (Excel, Google Sheets)
- Database imports and ETL pipelines
- Human-readable data inspection and editing

**File Extension**: `.csv`

**MIME Type**: `text/csv; charset=utf-8`

**Sample**:
```csv
student_id,first_name,last_name,email,active_status,date_enrolled
STU-001,John,Smith,john.smith@school.edu,true,2025-09-01
STU-002,Jane,Doe,jane.doe@school.edu,true,2025-09-01
```

### JSON (JavaScript Object Notation)

**Purpose**: Structured data format preserving hierarchical relationships, complex types, and metadata

**Characteristics**:
- **Encoding**: UTF-8
- **Schema**: Follows entity-specific schemas with versioning
- **Metadata Block**: Includes export metadata header (timestamp, version, exported_by, etc.)
- **Nested Structures**: Supports complex hierarchies and relationships
- **Null Handling**: Explicit `null` values for missing data
- **Date Format**: ISO 8601 strings
- **Arrays**: Used for one-to-many relationships
- **Pretty Printing**: Indented formatting for human readability (configurable)

**Use Cases**:
- API integrations and programmatic data exchange
- System migrations preserving full data structure
- Backup and archival with complete metadata
- Complex data imports requiring relationship preservation

**File Extension**: `.json`

**MIME Type**: `application/json; charset=utf-8`

**Sample**:
```json
{
  "export_metadata": {
    "mlc_export_version": "1.0",
    "export_timestamp": "2025-10-13T14:30:00Z",
    "exported_by_user_id": "USR-12345",
    "system_version": "MLC-5.0.3"
  },
  "data": [
    {
      "student_id": "STU-001",
      "first_name": "John",
      "last_name": "Smith",
      "email": "john.smith@school.edu",
      "active_status": true,
      "date_enrolled": "2025-09-01T00:00:00Z",
      "assignments": [
        {"assignment_id": "ASG-100", "status": "completed"}
      ]
    }
  ]
}
```

### PDF (Portable Document Format)

**Purpose**: Human-readable, print-ready reports with professional formatting

**Characteristics**:
- **Layout**: Professional formatting with headers, footers, page numbers
- **Fonts**: Embedded fonts for consistent rendering
- **Branding**: Organization logos and branding elements
- **Charts and Graphs**: Embedded visualizations for reports
- **Accessibility**: PDF/UA compliance for screen readers where applicable
- **Security**: Optional password protection and encryption
- **Watermarking**: Draft/final status indicators
- **Digital Signatures**: Support for digital signature fields

**Use Cases**:
- Financial statements and invoices
- Progress reports for parents and administrators
- Compliance documentation and audit reports
- Archival records and official documentation

**File Extension**: `.pdf`

**MIME Type**: `application/pdf`

### Excel (XLSX)

**Purpose**: Enhanced spreadsheet format with formatting, multiple sheets, and formulas

**Characteristics**:
- **Format**: Office Open XML (.xlsx)
- **Multiple Sheets**: Related data organized in tabs
- **Formatting**: Cell formatting, colors, fonts
- **Formulas**: Calculated fields and aggregations
- **Data Types**: Proper typing (dates, numbers, currency)
- **Filters and Sorting**: Pre-configured filters and sorting
- **Charts**: Embedded charts and visualizations

**Use Cases**:
- Executive reports with multiple data views
- Financial reports with calculations
- Complex analytics exports with visualizations
- Reports for non-technical users preferring Excel

**File Extension**: `.xlsx`

**MIME Type**: `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`

## Export Infrastructure

### Export Job Management

All export operations are managed through an asynchronous job system that handles queuing, processing, storage, and delivery:

**Export Job Entity**:
```json
{
  "export_job_id": "UUID",
  "export_type": "students|teachers|sequences|reports|billing|audit_logs",
  "requested_by_user_id": "USR-12345",
  "organization_id": "ORG-67890",
  "requested_at": "2025-10-13T14:30:00Z",
  "status": "queued|processing|completed|failed|cancelled",
  "progress_percentage": 0,
  "format": "csv|json|pdf|xlsx",
  "scope": "full|filtered|date_range",
  "filter_criteria": {},
  "file_url": "https://exports.mlc.com/...",
  "file_size_bytes": 0,
  "expires_at": "2025-10-14T14:30:00Z",
  "error_message": null,
  "completed_at": null
}
```

**Job Lifecycle**:
1. **Request Submission**: User initiates export through UI or API
2. **Validation**: System validates permissions, scope, and parameters
3. **Queuing**: Job is queued in background job system (see [Background Jobs](../18-architecture/background-jobs.md))
4. **Processing**: Worker extracts data, applies filters, formats output
5. **Storage**: Generated file is stored in temporary export bucket with pre-signed URL
6. **Notification**: User is notified via in-app notification and/or email
7. **Download**: User downloads file via secure, time-limited URL
8. **Expiration**: File is automatically deleted after expiration period (default 24 hours)

**Retry Logic**:
- Failed exports are automatically retried up to 3 times with exponential backoff
- Transient failures (database timeouts, network issues) trigger retries
- Permanent failures (invalid permissions, missing data) are not retried
- Users can manually retry failed exports from the export history interface

### Export Storage and Delivery

**Storage Architecture**:
- **Temporary Storage**: S3-compatible object storage with lifecycle policies
- **Pre-signed URLs**: Time-limited, authenticated download links (default 24-hour expiration)
- **Encryption**: All export files encrypted at rest (AES-256) and in transit (TLS 1.3)
- **Access Logging**: All download attempts logged for audit purposes
- **Geographic Distribution**: CDN distribution for large files (>10MB)

**Delivery Methods**:

1. **Direct Download**:
   - User receives in-app notification with download link
   - Click to download file immediately
   - Progress indicator for large files
   - Resume support for interrupted downloads (>10MB files)

2. **Email Delivery**:
   - Export file attached to email (files <10MB) or link provided (files >10MB)
   - Multiple recipients supported (up to 10 addresses)
   - Email subject line includes export type and date range
   - Delivery confirmation and tracking

3. **API Access**:
   - Programmatic export requests via REST API
   - Webhook notifications when export completes
   - Direct API download of export file content
   - Integration with external systems and ETL pipelines

4. **Scheduled Exports** (Future):
   - Recurring exports on defined schedules (daily, weekly, monthly)
   - Automated delivery to SFTP servers or cloud storage
   - Integration with backup systems and data warehouses

### File Naming Conventions

All export files follow standardized naming patterns for traceability and organization:

**General Pattern**:
```
{export_type}_{scope}_{organization_id}_{date_range}_{timestamp}.{extension}
```

**Examples**:
- `students_full_ORG-67890_20251013.csv` - Full student roster
- `teacher_report_USR-12345_20250901-20251001_20251013T143000Z.pdf` - Teacher report for date range
- `sequences_SEQ-LIFE_v7_20251013_full.json` - Sequence export
- `billing_statement_ORG-67890_202509_20251013.pdf` - September billing statement
- `audit_logs_ORG-67890_20251001-20251031_20251013.csv` - October audit logs

**Naming Rules**:
- All lowercase with underscores separating components
- ISO 8601 date formats (YYYYMMDD for dates, YYYYMMDDTHHMMSSZ for timestamps)
- Organization/user IDs included for multi-tenant environments
- Version numbers for versioned entities (sequences, games)
- Scope indicator (full, filtered, summary, etc.)

### Large File Handling

**Chunking and Streaming**:
- Exports exceeding 10,000 records are streamed to prevent memory exhaustion
- CSV files written incrementally as data is retrieved
- JSON files use streaming JSON libraries for large arrays

**Split Files**:
- Very large exports (>100,000 records) are automatically split into multiple files
- Files numbered sequentially: `students_full_ORG-67890_20251013_part1.csv`, `students_full_ORG-67890_20251013_part2.csv`
- Split threshold configurable per export type
- Manifest file included listing all parts and record counts

**Compression**:
- Exports exceeding 1MB are automatically compressed (gzip)
- File extension reflects compression: `.csv.gz`, `.json.gz`
- Compression ratio typically 80-90% for text-based exports
- Decompression instructions provided in download notification

**Progress Tracking**:
- Real-time progress updates for long-running exports
- Estimated time remaining based on record count and processing rate
- In-app progress indicator updates every 5 seconds
- User can navigate away; notification sent when complete

## Security and Permissions

### Export Permissions Model

Export permissions are role-based and scope-limited to ensure data privacy and compliance:

**Student Exports**:
- **Student**: No export permissions (students cannot export their own data)
- **Teacher**: Export own students only (CSV, PDF)
- **Teacher-Admin**: Export all students in assigned classes (CSV, Excel, PDF)
- **Subscriber-Admin**: Export all students in organization (all formats)
- **System Admin**: Export students across organizations (all formats)

**Teacher Exports**:
- **Teacher**: Export own profile only (PDF)
- **Subscriber-Admin**: Export all teachers in organization (CSV, Excel)
- **System Admin**: Export teachers across organizations (all formats)

**Sequence Exports**:
- **Teacher**: Export organization-owned custom sequences only (CSV, PDF)
- **Content Editor**: Export all sequences they have edit access to (CSV, JSON)
- **System Admin**: Export all sequences including method-aligned and proprietary (all formats)

**Report Exports**:
- Follow same permissions as report viewing (see [Roles and Permissions](../02-roles-and-permissions/roles-matrix.md))
- Teachers export reports for their students
- Admins export organization-wide reports
- System Admins export cross-organization reports

**Financial Exports**:
- **Subscriber-Admin**: Export billing statements and transaction history for own organization
- **System Admin**: Export financial data across organizations

**Audit Log Exports**:
- **System Admin Only**: Audit logs contain sensitive security information

### Data Filtering and Masking

**Automatic PII Filtering**:
- Exports for non-admin roles automatically filter sensitive fields
- Email addresses, phone numbers, addresses filtered unless admin role
- Student passwords never included in exports
- Payment card numbers never exported (only last 4 digits)

**Role-Based Field Visibility**:
- Export columns vary based on user role
- Teachers see student names and progress; admins see full profiles
- Financial exports to non-billing admins omit payment details

**Export Scope Restrictions**:
- Organization-scoped exports limited to user's organization unless System Admin
- Date range restrictions prevent excessive historical data extraction
- Maximum record limits configurable per role (e.g., 10,000 students per export for Teacher-Admins)

### Audit and Compliance

**Export Tracking**:
- All export requests logged with user ID, timestamp, export type, scope
- Download events logged separately from generation events
- Failed export attempts logged with failure reasons
- Retention: Export logs retained for 7 years for compliance

**FERPA and COPPA Compliance**:
- Student data exports restricted to authorized educational users only
- Export logs available for audit purposes
- Data retention policies enforced on temporary export files
- Automated deletion of expired exports

**GDPR Right to Data Portability**:
- Users (or parents of minor students) can request complete data export
- Includes all personal data, progress records, and activity logs
- Delivered in machine-readable format (JSON)
- Processed within 30 days of request

**Data Breach Notification**:
- Unauthorized export access triggers security alerts
- Incident response procedures for compromised export files
- Notification to affected users if export contains sensitive data

## Export Telemetry

All export operations emit telemetry events for monitoring, analytics, and troubleshooting:

**Export Requested Event**:
```json
{
  "event": "export.requested",
  "timestamp": "2025-10-13T14:30:00Z",
  "user_id": "USR-12345",
  "organization_id": "ORG-67890",
  "export_type": "students",
  "format": "csv",
  "scope": "full",
  "filter_criteria": {},
  "estimated_records": 1200
}
```

**Export Completed Event**:
```json
{
  "event": "export.completed",
  "timestamp": "2025-10-13T14:32:15Z",
  "export_job_id": "EXP-UUID",
  "user_id": "USR-12345",
  "organization_id": "ORG-67890",
  "export_type": "students",
  "format": "csv",
  "duration_seconds": 135,
  "file_size_bytes": 245680,
  "record_count": 1200
}
```

**Export Downloaded Event**:
```json
{
  "event": "export.downloaded",
  "timestamp": "2025-10-13T14:35:00Z",
  "export_job_id": "EXP-UUID",
  "user_id": "USR-12345",
  "download_method": "direct|email|api",
  "user_agent": "Mozilla/5.0...",
  "ip_address": "192.168.1.1"
}
```

**Export Failed Event**:
```json
{
  "event": "export.failed",
  "timestamp": "2025-10-13T14:32:15Z",
  "export_job_id": "EXP-UUID",
  "user_id": "USR-12345",
  "export_type": "students",
  "error_type": "permission_denied|timeout|data_error|system_error",
  "error_message": "Database query timeout after 300 seconds",
  "retry_count": 2
}
```

**Monitoring and Alerting**:
- Export failure rate threshold alerts (>5% failures)
- Long-running export alerts (>10 minutes for typical exports)
- Large export file alerts (>100MB)
- Spike in export volume alerts (unusual activity patterns)
- Expired export deletion confirmations

**See**: [Event Model](../15-analytics-and-reporting/event-model.md) for canonical event definitions

## Data Retention and Lifecycle

### Temporary Export Files

**Retention Policy**:
- **Default Expiration**: 24 hours from generation
- **Extended Expiration**: System Admins can extend to 7 days for compliance or migration exports
- **Manual Deletion**: Users can manually delete exports before expiration
- **Automatic Cleanup**: Nightly job purges expired exports

**Lifecycle Stages**:
1. **Generating** (0-15 minutes typical): File being created
2. **Available** (0-24 hours): File ready for download
3. **Expiring Soon** (last 2 hours): User receives warning notification
4. **Expired** (after 24 hours): File marked for deletion, download link invalid
5. **Deleted** (within 24 hours of expiration): File permanently removed from storage

**Re-generation**:
- Expired exports can be re-generated with same parameters
- Export history retains request parameters for easy re-generation
- One-click "Re-run Export" button in export history interface

### Archival Exports

**Long-Term Storage**:
- System Admins can mark exports as "archival" for extended retention
- Archival exports retained for 7 years (configurable per organization policy)
- Stored in cold storage with reduced access frequency
- Retrieval time: 1-4 hours for archival exports

**Use Cases**:
- Year-end data backups
- Compliance documentation
- System migration snapshots
- Legal hold requirements

**Access Controls**:
- Archival exports restricted to System Admins and Compliance Officers
- Access requires additional authentication (MFA)
- All access logged for audit purposes

## Common Export Workflows

### User-Initiated Export Workflow

1. **Access Export Interface**: User navigates to relevant section (Students, Reports, Billing, etc.)
2. **Select Export Type**: Choose data to export (e.g., "Export All Students")
3. **Configure Options**: 
   - Select format (CSV, PDF, Excel, JSON)
   - Choose scope (all records, filtered, date range)
   - Set filters and date ranges
4. **Request Export**: Click "Generate Export" button
5. **Confirmation**: System confirms request and provides estimated completion time
6. **Processing**: Export job queued and processed in background
7. **Notification**: User receives in-app and/or email notification when ready
8. **Download**: User clicks download link to retrieve file
9. **Expiration Warning**: User receives warning 2 hours before expiration
10. **Cleanup**: File automatically deleted after 24 hours

### API-Initiated Export Workflow

1. **API Request**: External system calls export API endpoint with authentication
2. **Validation**: System validates permissions and parameters
3. **Job Creation**: Export job created and queued
4. **Polling or Webhook**: Client polls status endpoint or receives webhook when complete
5. **Download**: Client retrieves file via pre-signed URL
6. **Integration**: Client processes export data in external system

### Scheduled Export Workflow (Future)

1. **Schedule Configuration**: Admin sets up recurring export schedule
2. **Automated Trigger**: System automatically initiates export at scheduled time
3. **Processing**: Export generated in background
4. **Automated Delivery**: File delivered to configured destination (email, SFTP, cloud storage)
5. **Confirmation**: Admin receives confirmation email with delivery status
6. **Retention**: Scheduled exports retained per archival policy

## Error Handling and Troubleshooting

### Common Export Errors

**Permission Denied**:
- **Cause**: User lacks permission to export requested data
- **Resolution**: Contact administrator to request export permissions
- **Log Message**: "User USR-12345 attempted to export students without permission"

**Export Timeout**:
- **Cause**: Large dataset exceeds processing time limit (default 10 minutes)
- **Resolution**: Narrow export scope with filters or date ranges; contact admin for large exports
- **Automatic Action**: Job automatically retried with chunking enabled

**Data Integrity Error**:
- **Cause**: Inconsistent or missing data detected during export
- **Resolution**: System Admin reviews data quality; problematic records excluded with warning
- **Log Message**: "Export EXP-UUID skipped 3 records due to data integrity issues"

**Storage Quota Exceeded**:
- **Cause**: Export file size exceeds available storage quota
- **Resolution**: System Admin increases quota or implements split file export
- **Automatic Action**: Export automatically split into multiple files

**Format Conversion Error**:
- **Cause**: Data cannot be represented in requested format (e.g., complex nested data in CSV)
- **Resolution**: Choose alternate format (JSON for complex data) or request structure-only export
- **Log Message**: "Cannot export nested sequence structure to CSV; use JSON format"

### Troubleshooting Guide

**Export Stuck in "Processing" Status**:
1. Check background job queue health
2. Review job logs for errors or warnings
3. Cancel and retry export with narrower scope
4. Contact support if issue persists beyond 30 minutes

**Download Link Not Working**:
1. Verify export is in "completed" status
2. Check link expiration timestamp
3. Ensure no network restrictions blocking download domain
4. Re-generate export if expired

**Empty or Incomplete Export File**:
1. Verify data exists for selected filters and date range
2. Check permissions cover requested scope
3. Review export job logs for filtering or exclusion messages
4. Retry export with adjusted parameters

**File Cannot Be Opened**:
1. Verify file downloaded completely (check file size)
2. Decompress if file is gzipped (.csv.gz, .json.gz)
3. Use correct application for file type
4. Check character encoding (UTF-8 expected)

## Integration with Related Systems

### Import System Integration

Exports are designed to be compatible with corresponding import specifications:

- **Student Exports**: Follow [CSV Spec Students](./csv-spec-students.md) column structure
- **Teacher Exports**: Follow [CSV Spec Teachers](./csv-spec-teachers.md) column structure
- **Sequence Exports**: Follow [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md) structure
- **Round-trip Compatibility**: Exported data can be re-imported with minimal transformation

### External System Integration

**Student Information Systems (SIS)**:
- Student roster exports compatible with common SIS formats (PowerSchool, Skyward, Infinite Campus)
- Configurable field mappings for different SIS vendors
- Scheduled exports for nightly synchronization

**Learning Management Systems (LMS)**:
- Progress data exports in LTI/xAPI format (see [xAPI Implementation](../17-integrations/xapi-implementation.md))
- Gradebook exports in Common Cartridge format
- Assignment completion exports for external grade books

**Accounting and ERP Systems**:
- Billing data exports in QuickBooks, Xero, SAP formats
- Chart of accounts mapping for revenue recognition
- Transaction exports with GL codes

**Business Intelligence and Analytics**:
- Data warehouse exports in star schema format
- Scheduled exports to data lakes (S3, Azure Blob, Google Cloud Storage)
- Real-time data streaming via webhooks (see [Webhooks](../18-architecture/webhooks.md))

## Performance Considerations

### Export Performance Targets

**Small Exports** (<1,000 records):
- **Generation Time**: <30 seconds
- **Download Initiation**: Immediate
- **User Experience**: Synchronous; user waits for file

**Medium Exports** (1,000-10,000 records):
- **Generation Time**: <3 minutes
- **Download Initiation**: Within 5 minutes
- **User Experience**: Asynchronous with notification

**Large Exports** (10,000-100,000 records):
- **Generation Time**: <15 minutes
- **Download Initiation**: Within 20 minutes
- **User Experience**: Background job with email notification

**Very Large Exports** (>100,000 records):
- **Generation Time**: <60 minutes
- **Download Initiation**: Within 90 minutes
- **User Experience**: Split files, background job, admin approval required

### Optimization Strategies

**Database Query Optimization**:
- Use indexed columns for filtering and sorting
- Implement cursor-based pagination for large result sets
- Execute parallel queries for split file exports
- Use read replicas for export queries to avoid production impact

**Caching**:
- Cache frequently exported static data (e.g., game catalog, method alignment)
- Cache export templates and formatting rules
- Reuse recent exports if data unchanged (cache for 5 minutes)

**Resource Management**:
- Limit concurrent export jobs per organization (max 3)
- Priority queue: small exports processed before large exports
- Rate limiting: max 10 export requests per user per hour
- Graceful degradation: disable exports during system maintenance

## Supporting Documents Referenced

This export specifications document draws from the following source documents:

- [Mass Upload.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Mass%20Upload.xlsx%20-%20Sheet1.csv) — Legacy student data format informing student export CSV structure and round-trip import compatibility
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Legacy game data format informing game registry export specifications and metadata column conventions
- [Course import.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Course%20import.xlsx%20-%20Sheet1.csv) — Legacy course/assignment format informing sequence export structure and curriculum data export patterns
- [MLC Tiered Pricing 2020.xlsx - Chart.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Chart.csv) — Pricing structure informing billing statement exports and subscription data export requirements
- [MLC Bid Sheet 6-21-21.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/MLC%20Bid%20Sheet%206-21-21.xlsx%20-%20Sheet1.csv) — Billing and payment specifications informing financial export formats and transaction data export structures
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role specifications informing role-based export permissions and data filtering rules for exports
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator tools informing export job management UI, download interfaces, and admin export capabilities
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional admin specifications for export history, job monitoring, and audit log export features

---

## Open Questions

### Format and Schema
- **Excel Chart Templates**: Should we provide pre-configured Excel chart templates for common analytics exports?
- **Custom Export Formats**: Should we support custom export formats defined by organizations (e.g., proprietary SIS formats)?
- **Export Schema Versioning**: How do we handle backward compatibility when export schemas evolve?

### Delivery and Distribution
- **SFTP Integration**: Should we support direct SFTP delivery for automated exports?
- **Cloud Storage Integration**: Should we integrate with customer-owned cloud storage (S3, Azure Blob, GCS)?
- **Webhook Notifications**: Should we provide webhook callbacks when exports complete for API integrations?

### Performance and Scale
- **Incremental Exports**: Should we support incremental exports (only records changed since last export)?
- **Streaming Exports**: Should we support real-time streaming exports for very large datasets?
- **Export Quotas**: Should we impose export quotas per organization to prevent abuse?

### Compliance and Governance
- **Data Anonymization**: Should we provide anonymized exports for research and analytics (PII removed)?
- **Export Approval Workflows**: Should large or sensitive exports require approval from multiple admins?
- **Audit Report Exports**: Should we automatically generate compliance reports for exports (who exported what when)?

## Related Documentation

### Import and Automation
- [CSV Spec Students](./csv-spec-students.md) - Student import schema (exports follow same structure)
- [CSV Spec Teachers](./csv-spec-teachers.md) - Teacher import schema
- [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md) - Curriculum import schema
- [Bulk Ops Jobs](./bulk-ops-jobs.md) - Background job management for bulk operations
- [Data Migration](./data-migration.md) - System migration and data transfer

### Specific Export Types
- [Sequence Exports](../09-sequences-and-curriculum-tools/sequence-exports.md) - Detailed sequence export specifications
- [Billing Statements On Demand](../14-payments-and-subscriptions/billing-statements-on-demand.md) - Financial export details
- [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md) - Student data export workflows

### Analytics and Reporting
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Teacher report export data
- [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md) - Admin analytics export data
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry event definitions

### Architecture and Infrastructure
- [Background Jobs](../18-architecture/background-jobs.md) - Async job processing
- [REST Endpoints](../18-architecture/rest-endpoints.md) - Export API endpoints
- [Security and Privacy](../18-architecture/security-privacy.md) - Data protection and encryption
- [Caching Strategy](../18-architecture/caching-strategy.md) - Export caching and performance

### Compliance and Operations
- [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) - Audit log exports
- [Observability](../19-quality-and-operations/observability.md) - Export monitoring and alerting
- [FERPA Compliance](../23-legal-and-compliance/ferpa-compliance.md) - Student data export compliance
- [GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md) - Right to data portability
