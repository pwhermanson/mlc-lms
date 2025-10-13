# CSV Import/Export Specifications

## Overview

This document serves as the central index and overview for all CSV-based data operations in the MLC-LMS platform. It establishes common formatting conventions, design principles, and validation patterns that apply across all CSV import and export types, while linking to detailed specifications for each data entity.

CSV (Comma-Separated Values) files are the primary mechanism for bulk data operations in MLC 5.0, enabling schools and organizations to efficiently onboard students, migrate content, export records for compliance, and integrate with external systems (Student Information Systems, HR databases, accounting software, etc.).

## Purpose

This overview specification enables:

- **Unified CSV Standards**: Establishes consistent formatting rules, character encoding, and validation patterns across all import/export types
- **Efficient Navigation**: Provides organized index to detailed entity-specific CSV specifications (students, teachers, courses, games, etc.)
- **Design Philosophy**: Documents the principles behind CSV format design, including backward compatibility with MLC 4.0 and modern enhancements
- **Common Patterns**: Defines reusable patterns for error handling, validation, username generation, and field normalization
- **Integration Guidance**: Explains how CSV specifications integrate with bulk operations workflows, background jobs, and user interfaces
- **Migration Support**: Facilitates transition from MLC 4.0 to MLC 5.0 by documenting legacy format compatibility and migration paths

**This page is an overview and index only.** For detailed column specifications, validation rules, and entity-specific examples, see the linked detailed specifications below.

## Scope

### Included

- Common CSV formatting conventions (encoding, delimiters, quoting, line endings)
- Universal validation patterns and error handling approaches
- CSV file structure requirements (headers, column order flexibility, required vs. optional fields)
- Character normalization and special character handling
- Index to all entity-specific CSV specifications
- Design principles and backward compatibility with MLC 4.0
- Integration touchpoints with bulk operations UI, background jobs, and APIs
- General security and compliance considerations for CSV data

### Excluded

- Detailed column definitions for specific entities (see entity-specific CSV specs below)
- Bulk operations UI and user workflows (see [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md), [Member Management](../05-admin-subscriber-experience/member-management.md))
- Background job processing and queue management (see [Bulk Ops Jobs](./bulk-ops-jobs.md))
- API endpoint specifications (see [REST Endpoints](../18-architecture/rest-endpoints.md))
- Export job management and delivery methods (see [Export Specs](./export-specs.md))
- Data model definitions beyond CSV representation (see [Data Model ERD](../18-architecture/data-model-erd.md))

---

## CSV Specification Index

The MLC-LMS platform supports CSV operations for the following entity types. Each link leads to a detailed specification with complete column definitions, validation rules, and examples:

### User Data Imports

**[CSV Spec Students](./csv-spec-students.md)**
- **Purpose**: Import student accounts in bulk for class rosters, school onboarding
- **Typical Volume**: 50-5,000 students per import
- **Key Features**: Auto-generated usernames and passwords, teacher assignment, course enrollment
- **Legacy Compatibility**: Compatible with MLC 4.0 "Mass Upload Utility" format

**[CSV Spec Teachers](./csv-spec-teachers.md)**
- **Purpose**: Import teacher accounts for school staff setup
- **Typical Volume**: 5-200 teachers per import
- **Key Features**: Role assignment (Teacher vs. Teacher-Admin), school hierarchy, department assignment
- **Legacy Compatibility**: Username generation identical to MLC 4.0 pattern

### Curriculum and Content Imports

**[CSV Spec Courses and Assignments](./csv-spec-courses-assignments.md)**
- **Purpose**: Import sequence structures, assignment groups, learning elements
- **Typical Volume**: 10-1,000 assignments per import
- **Key Features**: Multi-CSV format (Groups + Steps), element type codes (GAM, VID, AUD, TXT, RWD), gating rules
- **Legacy Compatibility**: Preserves MLC 4.0 assignment IDs (657-935 range) and sequence codes (LIFE, SOLF, EVAL)

**[CSV Spec Games](./csv-spec-games.md)**
- **Purpose**: Import game metadata and registry entries for content library
- **Typical Volume**: 10-500 games per import
- **Key Features**: Game taxonomy, stage configuration, asset URL validation, concept tagging
- **Legacy Compatibility**: Maps MLC 4.0 "GAM-XXXX" IDs to MLC 5.0 format, supports legacy game path references

### Data Exports

**[Export Specs](./export-specs.md)**
- **Purpose**: Export user data, content, analytics, and financial records for backup, compliance, reporting
- **Supported Formats**: CSV, JSON, PDF, Excel (XLSX)
- **Key Features**: Asynchronous job processing, time-limited download URLs, automatic compression, role-based filtering
- **Export Types**: Students, teachers, sequences, reports, billing statements, audit logs
- **CSV Compatibility**: Exports follow same column structure as corresponding import specs for round-trip compatibility

---

## Common CSV Formatting Conventions

All CSV files used in MLC-LMS imports and exports follow standardized formatting conventions to ensure cross-platform compatibility, data integrity, and ease of integration with external systems.

### File Encoding

**UTF-8 with BOM (Byte Order Mark)**:
- All CSV files must be encoded as UTF-8
- BOM (`EF BB BF`) is **recommended** for Excel compatibility on Windows
- BOM is optional for programmatic imports but improves human usability
- If BOM is absent, UTF-8 encoding is still assumed and validated

**Character Support**:
- Full Unicode support for international characters, accents, and special symbols
- Student/teacher names with accents handled correctly: `José`, `O'Brien`, `François`, `李明`
- Emoji and non-Latin scripts supported in text fields (names, descriptions, notes)
- See [Localization and Internationalization](../01-ux-design-system/localization-internationalization.md) for character handling details

### Delimiters and Structure

**Delimiter**: Comma (`,`)
- Standard comma delimiter for field separation
- Tab-delimited format (TSV) is **not** supported in MLC 5.0 imports
- Systems exporting tab-delimited files must convert to comma-delimited

**Line Endings**: CRLF (`\r\n`) or LF (`\n`)
- Windows-style CRLF (`\r\n`) is recommended for cross-platform compatibility
- Unix-style LF (`\n`) is accepted and automatically normalized during import
- Mixed line endings within a single file are supported but discouraged

**Quoting Rules**:
- Text fields containing commas, quotes, or line breaks **must** be enclosed in double quotes (`"`)
- Example: `"Smith, John"` for a last name containing a comma
- Embedded double quotes are escaped by doubling: `""` represents a literal `"`
- Example: `"He said ""Hello"""` → `He said "Hello"`

**Empty Values**:
- Empty/null values represented as blank fields: `field1,,field3` (field2 is empty)
- Leading/trailing whitespace in unquoted fields is trimmed during import
- Quoted empty string `""` treated identically to blank field (both represent null/empty)

### Header Row Requirements

**Mandatory Header Row**:
- First row of every CSV file **must** contain column headers
- Header names must match specification exactly (case-insensitive matching supported)
- Example: `student_first_name`, `Student_First_Name`, and `STUDENT_FIRST_NAME` are equivalent

**Column Order Flexibility**:
- Columns can appear in any order; system matches by header name, not position
- This allows organizations to rearrange columns to match their export systems
- Extra columns not in specification are ignored (forward compatibility)

**Required vs. Optional Columns**:
- Required columns must be present in header row (even if some values are blank)
- Optional columns can be omitted entirely from CSV file
- See entity-specific specs for lists of required vs. optional columns

### Data Type Formatting

**Strings**:
- No special formatting required for plain text
- Maximum lengths enforced per field (see entity-specific specs)
- Strings exceeding max length are truncated with validation warning

**Integers**:
- Plain numeric values without thousands separators: `1000` not `1,000`
- Leading zeros optional: `007` and `7` both accepted
- Negative numbers use minus sign: `-5`

**Booleans**:
- Accepted values: `true`/`false`, `1`/`0`, `Y`/`N`, `yes`/`no` (case-insensitive)
- Blank values treated as `false` for optional boolean fields
- Inconsistent boolean formats within same file generate warnings but are accepted

**Dates**:
- ISO 8601 format: `YYYY-MM-DD` (e.g., `2025-10-13`)
- US format `MM/DD/YYYY` is **not supported** (ambiguity with international formats)
- Timestamps: `YYYY-MM-DDTHH:MM:SSZ` (e.g., `2025-10-13T14:30:00Z`)
- Timezone: UTC assumed if not specified; Z suffix or +HH:MM offset supported

**Email Addresses**:
- Standard RFC 5322 format: `user@domain.com`
- Validated using regex pattern during import
- Invalid emails generate validation errors and block import of that row

**URLs**:
- Full URLs including protocol: `https://example.com/path`
- Validated via HTTP HEAD request during import (for asset URLs)
- Missing or inaccessible URLs generate warnings but do not block import

### Special Character Handling

**Name Normalization**:
- Accents and diacritics: `José` → `Jose`, `François` → `Francois` (when generating usernames)
- Apostrophes and hyphens: `O'Brien` → `OBrien`, `Mary-Jane` → `MaryJane` (username generation)
- Full Unicode preserved in display names; normalization only for usernames/slugs
- See username generation algorithms in [CSV Spec Students](./csv-spec-students.md) and [CSV Spec Teachers](./csv-spec-teachers.md)

**Whitespace Normalization**:
- Leading and trailing whitespace trimmed from all text fields
- Internal multiple spaces collapsed to single space: `John    Smith` → `John Smith`
- Tab characters replaced with single space
- Line breaks within quoted fields preserved (allowed in `notes` and `description` fields)

### File Size and Performance

**Recommended Limits**:
- **Small imports** (<100 rows): Process synchronously, user waits for completion (<10 seconds)
- **Medium imports** (100-1,000 rows): Asynchronous background job, completion notification (<2 minutes)
- **Large imports** (1,000-10,000 rows): Background job with progress tracking (<10 minutes)
- **Very large imports** (>10,000 rows): Chunked processing, may require split files or admin approval

**Maximum File Size**:
- Soft limit: 10 MB per CSV file (approximately 50,000 rows typical)
- Hard limit: 50 MB per CSV file (system enforced)
- Files exceeding limits should be split into multiple smaller files and imported sequentially
- See [Bulk Ops Jobs](./bulk-ops-jobs.md) for large file handling strategies

---

## Design Principles

### 1. Backward Compatibility with MLC 4.0

MLC 5.0 CSV specifications maintain compatibility with MLC 4.0 formats to ease migration and reduce retraining:

**Preserved Patterns**:
- Username generation algorithm: `{LastName}{FirstName}{OrgMemberNumber}` (e.g., `SmithJohn17502`)
- Password auto-generation from standard musical term list
- Legacy game IDs: `GAM-XXXX` format mapped to MLC 5.0 IDs
- Sequence codes: `LIFE`, `SOLF`, `EVAL` preserved
- Course assignment IDs: 657-935 range preserved for backward compatibility

**Enhancements in MLC 5.0**:
- Column order flexibility (MLC 4.0 required fixed column positions)
- Optional columns can be omitted entirely (MLC 4.0 required all columns present)
- Improved validation with per-row error reporting (MLC 4.0 failed entire import on first error)
- Preview/dry-run mode before committing import
- Support for external IDs (`external_id` column for SIS integration)
- Extended metadata fields for modern pedagogical tagging

### 2. Human-Readable and Machine-Parsable

**Designed for Dual Use**:
- **Human editors**: School administrators can open CSV files in Excel, edit directly, and re-import
- **Programmatic generation**: IT departments can generate CSV files from SIS/HR systems without manual editing
- Column headers use descriptive names: `student_first_name` not `sfn` or `col_a`
- Example data and templates provided in each entity-specific spec

### 3. Fail-Fast Validation with Helpful Errors

**Pre-Import Validation**:
- File-level validation before processing any rows: encoding, structure, required columns
- Row-level validation with specific error messages: "Row 47: Invalid email format for 'jane.doe@school'"
- Validation errors include row number, column name, invalid value, and suggested fix
- Import preview mode (dry-run) allows validation without committing changes

**Partial Success Handling**:
- Valid rows are imported successfully; invalid rows are skipped with detailed error report
- Summary report shows: X rows succeeded, Y rows failed, downloadable error CSV for fixing
- Failed rows can be corrected and re-imported without duplicating successful rows
- See [Bulk Ops Jobs](./bulk-ops-jobs.md) for error handling workflows

### 4. Secure by Default

**Data Sanitization**:
- All text fields sanitized to prevent injection attacks (SQL, XSS, CSV injection)
- Leading `=`, `+`, `-`, `@` characters in fields escaped to prevent formula injection in Excel
- HTML tags stripped from text fields not explicitly allowing HTML
- File upload validation: MIME type checking, virus scanning, size limits

**Permission Enforcement**:
- Import operations enforce role-based permissions (see [Roles Matrix](../02-roles-and-permissions/roles-matrix.md))
- Teachers cannot import other teachers; Subscriber-Admins cannot import System Admin accounts
- Cross-organization imports blocked (users can only import to their own organization)
- Audit logging for all import operations (see [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md))

### 5. Round-Trip Compatible (Import ↔ Export)

**Export Follows Import Schema**:
- Student exports use same column structure as student imports (see [CSV Spec Students](./csv-spec-students.md))
- Exported CSV files can be re-imported with minimal or no modifications
- This enables workflows like: export students → edit in Excel → re-import with updates
- See [Export Specs](./export-specs.md) for export-specific formatting details

**Data Integrity Preservation**:
- Exported data includes all fields required for round-trip import
- IDs, external references, and metadata preserved in exports
- Computed fields (e.g., auto-generated usernames) included in exports for transparency

---

## Validation Patterns and Error Handling

All CSV imports follow consistent validation and error handling patterns to ensure data quality and provide actionable feedback to users.

### Validation Stages

**Stage 1: File-Level Validation** (before opening file):
- File size within limits (<50 MB)
- Valid CSV structure (parseable by CSV parser)
- UTF-8 encoding (BOM optional but recommended)
- MIME type: `text/csv` or `application/vnd.ms-excel`

**Stage 2: Structure Validation** (after parsing headers):
- Header row present (first row contains column names, not data)
- Required columns present (all required columns exist in header row)
- Column names match specification (case-insensitive, whitespace-insensitive)
- No duplicate column names

**Stage 3: Row-Level Validation** (each data row):
- Required fields not blank (e.g., `student_first_name`, `student_last_name`)
- Data types match specification (e.g., integers are numeric, emails are valid format)
- Field lengths within limits (e.g., `student_first_name` ≤ 50 characters)
- Referential integrity (e.g., assigned teacher exists, course assignment ID is valid)
- Business rule validation (e.g., `teacher_admin` role only valid in Ensemble plans)

**Stage 4: Batch Validation** (across all rows):
- No duplicate usernames within file
- No duplicate external IDs within file
- Total row count within limits (e.g., <10,000 rows for standard imports)

### Error Types and Severity

**Critical Errors** (block entire import):
- File cannot be parsed as CSV
- Character encoding is not UTF-8
- Required columns missing from header row
- File size exceeds hard limit (50 MB)

**Row Errors** (skip row, continue processing):
- Required field is blank: `Row 23: student_first_name is required but blank`
- Data type mismatch: `Row 47: student_email 'not-an-email' is not a valid email address`
- Field too long: `Row 102: student_first_name exceeds 50 character limit`
- Referential integrity failure: `Row 58: teacher_username 'DoeJane99999' not found in organization`
- Duplicate username: `Row 75: student_username 'SmithJohn17502' already exists`

**Warnings** (row imported with warning logged):
- Optional field has minor formatting issue: `Row 12: student_phone '555-1234' missing area code`
- External ID is blank (recommended but optional): `Row 34: external_id is blank`
- Asset URL is inaccessible (for game imports): `Row 8: thumbnail_url returns 404 Not Found`
- Deprecated field used: `Row 45: 'old_field_name' is deprecated; use 'new_field_name' instead`

### Error Reporting

**Import Summary**:
```
Import completed with partial success
- Total rows processed: 1,247
- Rows imported successfully: 1,198 (96%)
- Rows failed with errors: 49 (4%)
- Warnings logged: 12

Download error report: import_errors_2025-10-13T14-30-00Z.csv
```

**Error Report CSV**:
- Downloadable CSV file with failed rows and error details
- Columns: `row_number`, `error_type`, `field_name`, `invalid_value`, `error_message`, `suggested_fix`, `original_row_data`
- Users can fix errors in spreadsheet and re-import only failed rows

**Error Codes** (see entity-specific specs for complete lists):
- `E001`: Required field missing
- `E002`: Invalid data type
- `E003`: Field length exceeded
- `E004`: Referential integrity violation
- `E005`: Duplicate value detected
- `E006`: Business rule violation
- `W001`: Minor formatting issue (warning)
- `W002`: Optional recommended field missing (warning)

---

## Integration Touchpoints

CSV specifications integrate with multiple platform components and user workflows:

### Bulk Operations UI

**User-Facing Interfaces**:
- [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md): UI for uploading student CSV files, reviewing preview, managing import jobs
- [Member Management](../05-admin-subscriber-experience/member-management.md): UI for teacher/admin CSV imports
- [Games Importer](../08-games-registry-and-authoring/games-importer.md): System Admin interface for game registry CSV imports

**Workflow**:
1. User selects CSV file via file picker
2. System validates file structure and displays preview (first 50 rows)
3. User confirms import or cancels to fix errors
4. Background job processes import asynchronously
5. User receives notification with success/error summary
6. User downloads error report to fix failed rows if needed

### Background Jobs

**Asynchronous Processing**:
- CSV imports >100 rows are processed as background jobs (see [Bulk Ops Jobs](./bulk-ops-jobs.md))
- Job status tracking: queued → processing → completed/failed
- Progress updates: percentage complete, rows processed, estimated time remaining
- Cancellation support: users can cancel long-running imports
- Retry logic: transient failures (database timeout) auto-retry; permanent failures (validation error) do not retry

**Queue Management**:
- Import jobs queued in priority order: small imports before large imports
- Per-organization concurrency limits: max 3 concurrent import jobs per org
- System-wide rate limiting: max 10 imports per user per hour
- See [Background Jobs](../18-architecture/background-jobs.md) for queue infrastructure details

### API Endpoints

**Programmatic Import**:
- `POST /api/v1/imports/students` - Upload student CSV via API
- `POST /api/v1/imports/teachers` - Upload teacher CSV
- `POST /api/v1/imports/games` - Upload game CSV (System Admin only)
- `GET /api/v1/imports/{job_id}` - Get import job status
- `GET /api/v1/imports/{job_id}/errors` - Download error report CSV
- See [REST Endpoints](../18-architecture/rest-endpoints.md) for complete API specifications

**Export APIs**:
- `POST /api/v1/exports/students` - Request student data export
- `GET /api/v1/exports/{job_id}` - Get export job status and download URL
- See [Export Specs](./export-specs.md) for export API details

---

## Security and Compliance Considerations

### Data Privacy

**PII Protection**:
- CSV files containing student data classified as Personally Identifiable Information (PII)
- Temporary CSV files encrypted at rest during processing
- CSV files deleted from temporary storage after import completion (within 24 hours)
- Export CSV files expire after 24 hours and are automatically purged
- See [Security and Privacy](../18-architecture/security-privacy.md) for encryption details

**FERPA and COPPA Compliance**:
- Student CSV imports restricted to authorized educational users (Teachers, Subscriber-Admins, System Admins)
- Student exports require parental consent for students under 13 (COPPA)
- Audit logging for all CSV operations involving student data
- See [FERPA Compliance](../23-legal-and-compliance/ferpa-compliance.md) and [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md)

### Audit Logging

**Import Audit Events**:
- `import.requested`: User initiated CSV import (user_id, org_id, import_type, row_count)
- `import.validated`: File passed validation (row_count, validation_duration_ms)
- `import.completed`: Import finished successfully (rows_imported, rows_failed, duration_ms)
- `import.failed`: Import failed with critical error (error_type, error_message)
- See [Event Model](../15-analytics-and-reporting/event-model.md) for canonical event definitions

**Retention**:
- Import audit logs retained for 7 years (compliance requirement)
- CSV file content not retained; only metadata (filename, row count, success/failure)
- Error reports retained for 30 days after import completion
- See [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)

---

## Migration from MLC 4.0

Schools migrating from MLC 4.0 to MLC 5.0 can reuse existing CSV files with minimal modifications:

### Student Mass Upload Utility

**MLC 4.0 Format**:
- Fixed column positions: First Name (col 1), Last Name (col 2), Username (col 3), Password (col 4), Active Status (col 5), Teacher Name (col 6), Course Assignment ID (col 7)
- Tab-delimited or comma-delimited (both supported)
- No header row (data starts on row 1)

**MLC 5.0 Migration**:
1. Add header row with column names: `student_first_name,student_last_name,student_username,student_password,active_status,teacher_real_name,course_assignment_id`
2. Ensure comma-delimited (convert tab-delimited files if necessary)
3. Verify UTF-8 encoding (MLC 4.0 files may be ANSI/Windows-1252)
4. Optional: Add new MLC 5.0 columns for enhanced features (`external_id`, `grade_level`, `class_name`)

**Username and Password Compatibility**:
- MLC 5.0 uses identical username generation algorithm as MLC 4.0 (no conflicts)
- Existing student usernames preserved during migration
- Standard password list identical to MLC 4.0 list

### Games Import Master

**MLC 4.0 Format**:
- Columns: `id`, `game_number` (GAM-XXXX), `title`, `description`, `thumbnail_img`, `Category`, `skill_level`, `active`, `keyboard`, `timed`, `stage_id`, `game_path`, `target_score`, `Free`
- Legacy game IDs: `GAM-1680-2E` format

**MLC 5.0 Migration**:
- Rename columns to match MLC 5.0 spec: `game_number` → `legacy_id`, `title` → `name`, `Category` → `category`, `skill_level` → `level`
- Add new required columns: `instrument`, `difficulty`, `slug` (can be auto-generated)
- Map legacy `stage_id` to MLC 5.0 stage configuration (see [CSV Spec Games](./csv-spec-games.md))
- Asset URLs updated to point to CDN (legacy `game_path` preserved for reference)

### Course Import

**MLC 4.0 Format**:
- Columns: `code`, `group`, `level_title`, `unit_title`, `seq_order`, `part_type`, `part_number`, `stage`, `name`, `description`, `vimeo_url`, `pdf`
- Element types: `VID`, `DOC`, `GAM` with stage suffixes (`-1E`, `-2E`, etc.)

**MLC 5.0 Migration**:
- Format split into two CSVs: Assignment Groups CSV + Assignment Steps CSV
- Rename columns: `code` → `sequence_code`, `group` → `group_id`, `part_type` → `element_type`, `part_number` → `element_id`
- Map element types: `VID` → `VID`, `DOC` → `TXT`, `GAM` → `GAM`
- Preserve legacy assignment numbers (657-935 range) in `assignment_number` column
- See [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md) for detailed mapping

---

## Supporting Documents Referenced

This CSV specifications overview draws from the following source documents:

- [Mass Upload.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Mass%20Upload.xlsx%20-%20Sheet1.csv) — Legacy MLC 4.0 student bulk upload format informing common CSV formatting conventions, field naming patterns, and validation approaches
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Legacy game import master format informing CSV structure standards and metadata column conventions
- [Course import.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Course%20import.xlsx%20-%20Sheet1.csv) — Legacy course/assignment import format informing curriculum CSV specifications and hierarchical data representation
- [MLC SeqCreate 2020.xlsx - ALPHA LIST .csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20ALPHA%20LIST%20.csv) — Legacy sequence creation format informing sequence CSV specifications and assignment group structures
- [MLC History Draft 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20History%20Draft%202020.docx.txt) — Historical context and MLC 4.0 evolution informing backward compatibility requirements and migration considerations
- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — System overview informing CSV design principles and integration requirements with existing workflows

---

## Related Documentation

### Detailed CSV Specifications
- [CSV Spec Students](./csv-spec-students.md) - Complete student import column definitions, validation rules, examples
- [CSV Spec Teachers](./csv-spec-teachers.md) - Complete teacher import column definitions, role assignment, examples
- [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md) - Sequence and assignment import specifications
- [CSV Spec Games](./csv-spec-games.md) - Game registry import specifications, metadata, asset URLs
- [Export Specs](./export-specs.md) - Data export formats, job management, delivery methods

### Bulk Operations and Workflows
- [Bulk Ops Jobs](./bulk-ops-jobs.md) - Background job processing, error handling, progress tracking
- [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md) - UI workflows for student CSV imports
- [Member Management](../05-admin-subscriber-experience/member-management.md) - UI workflows for teacher/admin management
- [Games Importer](../08-games-registry-and-authoring/games-importer.md) - System Admin game import interface

### Data Model and Architecture
- [Data Model ERD](../18-architecture/data-model-erd.md) - Database schema and entity relationships
- [Background Jobs](../18-architecture/background-jobs.md) - Async job queue infrastructure
- [REST Endpoints](../18-architecture/rest-endpoints.md) - Import/export API specifications
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry events for imports/exports

### Compliance and Security
- [Security and Privacy](../18-architecture/security-privacy.md) - Encryption, PII protection, data handling
- [FERPA Compliance](../23-legal-and-compliance/ferpa-compliance.md) - Student data protection requirements
- [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md) - Student privacy considerations
- [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) - Audit logging for CSV operations

### Design System and Standards
- [Localization and Internationalization](../01-ux-design-system/localization-internationalization.md) - Unicode and character handling
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Permission requirements for import/export operations
