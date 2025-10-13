# CSV Spec Teachers

This document defines the CSV file format specification for importing teacher data into the MLC-LMS platform. It specifies the exact structure, field requirements, validation rules, and formatting guidelines for CSV files used to create teacher accounts in bulk.

## Purpose

This specification enables:

- **Efficient Teacher Onboarding**: Schools and organizations can import multiple teacher accounts rapidly using CSV files generated from their HR systems or administrative databases
- **Standardized Data Format**: Ensures consistent CSV structure across all organizations, reducing errors and support overhead
- **Automated Account Creation**: Enables automatic generation of usernames and passwords when not provided, following MLC naming conventions
- **Role-Based Setup**: Supports creation of both Teacher and Teacher-Admin roles within Ensemble plan organizations
- **Data Validation**: Provides clear validation rules to catch errors before import, ensuring data quality and system integrity
- **Legacy Compatibility**: Maintains compatibility with MLC 4.0 teacher management workflows while supporting modern enhancements
- **IT Department Integration**: Allows school IT staff to generate teacher lists programmatically without manual data entry

This specification is narrowly focused on the CSV file format itself. For the bulk operations workflow, UI, and batch processing details, see [Member Management](../05-admin-subscriber-experience/member-management.md).

## Scope

**Included:**
- CSV file structure and formatting requirements for teacher imports
- Required and optional column specifications
- Data type definitions and constraints
- Field-level validation rules for teacher accounts
- Username and password generation algorithms (identical to student generation)
- Role assignment specifications (Teacher vs. Teacher-Admin)
- Character encoding and special character handling
- Example CSV templates and sample data
- Error codes for validation failures specific to teacher imports

**Excluded:**
- Member management UI and workflow (see [Member Management](../05-admin-subscriber-experience/member-management.md))
- Import batch processing and job management (see [Bulk Ops Jobs](./bulk-ops-jobs.md))
- Teacher data model beyond CSV fields (see [Data Model ERD](../18-architecture/data-model-erd.md))
- Student and course assignment CSV specs (see [CSV Spec Students](./csv-spec-students.md) and [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md))
- API endpoints for import operations (see [REST Endpoints](../18-architecture/rest-endpoints.md))
- Teacher role permissions and capabilities (see [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md))

## Data model (if applicable)

### CSV Column Definitions

The teacher import CSV supports the following columns. Column order is flexible, but the first row **must** contain column headers that exactly match these names (case-insensitive).

#### Required Columns

| Column Name | Data Type | Max Length | Description | Example |
|-------------|-----------|------------|-------------|---------|
| `teacher_first_name` | String | 50 | Teacher's first name (given name) | `Jane` |
| `teacher_last_name` | String | 50 | Teacher's last name (surname/family name) | `Doe` |
| `teacher_username` | String | 100 | Unique username for login (see Username Generation Rules) | `DoeJane17502` or leave blank for auto-generation |

**Notes:**
- If `teacher_username` is blank, the system will auto-generate using the naming convention: `{LastName}{FirstName}{OrgMemberNumber}`
- Auto-generated usernames are normalized: spaces removed, special characters stripped, camelCase applied
- Username generation algorithm is identical to student username generation (see [CSV Spec Students](./csv-spec-students.md))

#### Optional Columns

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `teacher_password` | String | 100 | Initial password for the teacher account | `music123` or leave blank | Auto-generated from standard password list |
| `teacher_email` | Email | 255 | Teacher's email address (recommended for notifications and communications) | `jane.doe@school.edu` | `null` |
| `teacher_phone` | String | 20 | Teacher's phone number | `(555) 123-4567` | `null` |
| `active_status` | String | 1 | Account status: `Y` for active, blank or `N` for inactive | `Y` | `Y` (active) |
| `role` | String | 20 | Teacher role: `teacher` or `teacher_admin` | `teacher` | `teacher` |
| `school_name` | String | 200 | Name of school or institution | `Pine Valley Elementary` | `null` |
| `department` | String | 100 | Department or grade level taught | `Music Department`, `K-5 Music` | `null` |
| `external_id` | String | 100 | External system identifier (HR system ID, district ID) | `TCHR-2025-001234` | `null` |
| `notes` | String | 500 | Administrative notes about the teacher | `Department head, piano specialist` | `null` |

**Notes:**
- If `teacher_password` is blank, the system auto-generates a password from a rotating list of simple musical terms (identical to student password generation)
- If `teacher_email` is provided, it will be used for system notifications, assignment alerts, and communications
- The `role` field determines whether the teacher is created as a `teacher` (standard) or `teacher_admin` (Ensemble plans only)
- `teacher_admin` role can only be assigned in Ensemble subscriptions; attempts to create Teacher-Admin in Solo/Prelude plans will result in validation error

### Username Generation Rules

When `teacher_username` is blank or omitted, the system automatically generates usernames using the same algorithm as student usernames:

**Basic Pattern:**
```
{LastName}{FirstName}{OrgMemberNumber}
```

**Example:**
- Organization Member Number: `17502`
- Teacher Name: Jane Doe
- Generated Username: `DoeJane17502`

**Normalization Steps:**
1. Remove all whitespace from first and last names
2. Strip special characters and accents (e.g., `José` → `Jose`, `O'Brien` → `OBrien`)
3. Capitalize first letter of last name, first letter of first name (PascalCase)
4. Append organization member number
5. Check for duplicate usernames in the organization

**Duplicate Resolution:**
If the generated username already exists (e.g., multiple Jane Does), the system appends a letter suffix to the first name and retries:

```
DoeJane17502      (first teacher)
DoeJaneA17502     (second teacher with same name)
DoeJaneB17502     (third teacher with same name)
...
DoeJaneZ17502     (26th teacher)
DoeJaneAA17502    (27th teacher - double letter suffix)
```

**Collision Detection:**
- Username uniqueness is checked at the **organization level** (not globally)
- If a manually-provided username conflicts with an existing username (student or teacher), the import validation will fail for that row with error code `ERR_USERNAME_DUPLICATE`

For complete username generation algorithm details, see [CSV Spec Students - Username Generation Rules](./csv-spec-students.md#username-generation-rules).

### Password Generation Rules

When `teacher_password` is blank or omitted, the system automatically assigns a password from a rotating list of simple musical terms, using the same algorithm as student passwords.

**Standard Password List (50 terms, cycling):**
```
music, piano, note, sharp, flat, key, chord, tempo, scale, melody,
rhythm, harmony, octave, pitch, staff, clef, treble, bass, rest, beat,
measure, bar, forte, piano, allegro, adagio, andante, crescendo, staccato,
legato, glissando, arpeggio, cadence, consonant, dissonant, dynamics,
fermata, interval, motif, phrase, refrain, repeat, slur, syncopation,
timbre, tone, transpose, vibrato, waltz, whole
```

**Assignment Algorithm:**
1. Start with the first password in the list (`music`)
2. Assign sequentially to each teacher in the CSV file
3. When reaching the end of the list, cycle back to the beginning
4. Continue until all teachers have passwords

**Security Notes:**
- These are initial passwords intended for easy distribution to teachers
- Teachers can change passwords after first login
- For enhanced security, organizations may provide their own passwords in the CSV file
- Passwords are stored as bcrypt hashes in the database (see [Security & Privacy](../18-architecture/security-privacy.md))

For complete password generation algorithm details, see [CSV Spec Students - Password Generation Rules](./csv-spec-students.md#password-generation-rules).

### CSV File Format Requirements

#### Technical Specifications

| Property | Requirement | Notes |
|----------|-------------|-------|
| **File Extension** | `.csv` | Text files with comma-separated values |
| **Character Encoding** | `UTF-8` (with or without BOM) | Supports international characters and accents |
| **Line Endings** | `CRLF` (Windows) or `LF` (Unix/Mac) | Either format is accepted |
| **Delimiter** | Comma (`,`) | Field separator |
| **Text Qualifier** | Double quote (`"`) | Used to escape commas and quotes within fields |
| **Header Row** | **Required** (first row) | Column names, case-insensitive |
| **Maximum File Size** | 10 MB | Approximately 50,000 teachers (rare edge case) |
| **Maximum Rows** | 10,000 rows | Excluding header row; typical school districts have 50-500 teachers |

#### Field Formatting Rules

1. **Text Fields:**
   - Enclose in double quotes if containing commas, quotes, or line breaks
   - Escape internal double quotes by doubling them: `"He said ""hello"""` → `He said "hello"`
   - Leading and trailing whitespace is automatically trimmed
   - Empty fields can be represented as `""` or left blank between commas

2. **Boolean Fields (`active_status`):**
   - `Y`, `y`, `YES`, `yes`, `True`, `true`, `1` → Active
   - `N`, `n`, `NO`, `no`, `False`, `false`, `0`, blank → Inactive

3. **Email Fields (`teacher_email`):**
   - Must match standard email format: `local@domain.tld`
   - Maximum length: 255 characters
   - Case-insensitive
   - Validation uses RFC 5322 standard

4. **Phone Fields (`teacher_phone`):**
   - Any format accepted: `(555) 123-4567`, `555-123-4567`, `5551234567`, `+1-555-123-4567`
   - Formatting is preserved as entered
   - International phone numbers supported

5. **Role Fields (`role`):**
   - Case-insensitive: `teacher`, `TEACHER`, `Teacher` all accepted
   - `teacher_admin`, `teacher-admin`, `TeacherAdmin` all map to Teacher-Admin role
   - Invalid values trigger validation error `ERR_ROLE_INVALID`

6. **Special Characters:**
   - Accented characters are supported: `José`, `François`, `Müller`
   - Emoji and non-Latin scripts are supported but discouraged for usernames
   - Commas within fields must be enclosed in quotes: `"Doe, Jr.", Jane`

#### Example CSV Templates

**Minimal CSV (Required Fields Only with Auto-Generation):**
```csv
teacher_first_name,teacher_last_name,teacher_username
Jane,Doe,
Bob,Martin,
Sarah,Williams,
```
*Note: Blank usernames will be auto-generated; passwords will be auto-assigned*

**Standard CSV (Common Use Case):**
```csv
teacher_first_name,teacher_last_name,teacher_username,teacher_password,teacher_email,active_status,role,school_name
Jane,Doe,DoeJane17502,music,jane.doe@school.edu,Y,teacher,Pine Valley Elementary
Bob,Martin,MartinBob17502,piano,bob.martin@school.edu,Y,teacher,Pine Valley Elementary
Sarah,Williams,WilliamsSarah17502,note,sarah.williams@school.edu,Y,teacher_admin,Pine Valley Elementary
```

**Comprehensive CSV (All Optional Fields):**
```csv
teacher_first_name,teacher_last_name,teacher_username,teacher_password,teacher_email,teacher_phone,active_status,role,school_name,department,external_id,notes
Jane,Doe,DoeJane17502,music,jane.doe@school.edu,(555) 123-4567,Y,teacher,Pine Valley Elementary,Music Department,TCHR-2025-001234,Department head
Bob,Martin,MartinBob17502,piano,bob.martin@school.edu,(555) 123-4568,Y,teacher,Pine Valley Elementary,K-5 Music,TCHR-2025-001235,Piano specialist
Sarah,Williams,WilliamsSarah17502,note,sarah.williams@school.edu,(555) 123-4569,Y,teacher_admin,Pine Valley Elementary,District Music Coordinator,TCHR-2025-001236,Oversees 5 schools
```

**CSV with Special Characters:**
```csv
teacher_first_name,teacher_last_name,teacher_username,teacher_password,teacher_email,active_status,notes
José,García,GarciaJose17502,music,jose.garcia@school.edu,Y,Original name: José García
François,Müller,MullerFrancois17502,piano,francois.muller@school.edu,Y,Original name: François Müller
Mary,"O'Brien",OBrienMary17502,note,mary.obrien@school.edu,Y,"Last name contains apostrophe"
"Smith, Jr.",Robert,SmithJrRobert17502,sharp,robert.smith@school.edu,Y,"Last name contains comma"
```

## Behavior and rules

### Validation Rules

The CSV import system validates each row before creating teacher accounts. Validation occurs in two phases:

#### Phase 1: File-Level Validation (Pre-Processing)

These checks occur before any data is processed:

| Rule | Description | Error Code |
|------|-------------|------------|
| **File Format** | File must be valid CSV with `.csv` extension | `ERR_INVALID_FILE_FORMAT` |
| **File Size** | File must not exceed 10 MB | `ERR_FILE_TOO_LARGE` |
| **Row Limit** | File must not exceed 10,000 rows (excluding header) | `ERR_TOO_MANY_ROWS` |
| **Character Encoding** | File must be UTF-8 encoded | `ERR_INVALID_ENCODING` |
| **Header Row** | First row must contain valid column headers | `ERR_MISSING_HEADER` |
| **Required Columns** | Must include `teacher_first_name`, `teacher_last_name`, `teacher_username` | `ERR_MISSING_REQUIRED_COLUMN` |

If any file-level validation fails, the entire import is rejected and no rows are processed.

#### Phase 2: Row-Level Validation (Per-Teacher)

These checks occur for each teacher row in the CSV:

**Required Field Validation:**

| Field | Rule | Error Code |
|-------|------|------------|
| `teacher_first_name` | Must not be empty or whitespace-only | `ERR_FIRST_NAME_REQUIRED` |
| `teacher_first_name` | Must not exceed 50 characters | `ERR_FIRST_NAME_TOO_LONG` |
| `teacher_first_name` | Must contain at least one letter | `ERR_FIRST_NAME_INVALID` |
| `teacher_last_name` | Must not be empty or whitespace-only | `ERR_LAST_NAME_REQUIRED` |
| `teacher_last_name` | Must not exceed 50 characters | `ERR_LAST_NAME_TOO_LONG` |
| `teacher_last_name` | Must contain at least one letter | `ERR_LAST_NAME_INVALID` |
| `teacher_username` | If provided, must be 3-100 characters | `ERR_USERNAME_INVALID_LENGTH` |
| `teacher_username` | If provided, must be unique within organization (across all users) | `ERR_USERNAME_DUPLICATE` |
| `teacher_username` | If provided, must contain only alphanumeric characters and underscores | `ERR_USERNAME_INVALID_CHARS` |

**Optional Field Validation:**

| Field | Rule | Error Code |
|-------|------|------------|
| `teacher_password` | If provided, must be 4-100 characters | `ERR_PASSWORD_INVALID_LENGTH` |
| `teacher_email` | If provided, must be valid email format | `ERR_EMAIL_INVALID` |
| `teacher_email` | If provided, must be unique within organization | `ERR_EMAIL_DUPLICATE` |
| `teacher_phone` | If provided, must not exceed 20 characters | `ERR_PHONE_TOO_LONG` |
| `active_status` | If provided, must be Y/N or valid boolean value | `ERR_ACTIVE_STATUS_INVALID` |
| `role` | If provided, must be `teacher` or `teacher_admin` | `ERR_ROLE_INVALID` |
| `role` | If `teacher_admin`, subscription must be Ensemble plan | `ERR_ROLE_NOT_ALLOWED_IN_PLAN` |
| `school_name` | If provided, must not exceed 200 characters | `ERR_SCHOOL_NAME_TOO_LONG` |
| `department` | If provided, must not exceed 100 characters | `ERR_DEPARTMENT_TOO_LONG` |
| `external_id` | If provided, must not exceed 100 characters | `ERR_EXTERNAL_ID_TOO_LONG` |
| `notes` | If provided, must not exceed 500 characters | `ERR_NOTES_TOO_LONG` |

### Import Processing Modes

The CSV import system supports two processing modes:

#### 1. Validation-Only Mode (Dry Run)

- Validates the entire CSV file without creating any teacher accounts
- Returns a detailed validation report with all errors and warnings
- Recommended as a first step before committing the import
- No data is modified in the system
- Allows administrators to fix errors in the CSV before re-uploading

**Workflow:**
1. Upload CSV file in validation-only mode
2. Review validation report
3. Fix errors in CSV file
4. Re-upload and validate until no errors
5. Switch to import mode to commit changes

#### 2. Import Mode (Commit)

- Processes valid rows and creates teacher accounts
- Skips invalid rows and reports errors
- Generates an import summary report
- Creates import batch record for tracking (see [Bulk Ops Jobs](./bulk-ops-jobs.md))

**Processing Strategy: Partial Success**
- Valid rows are processed and committed
- Invalid rows are skipped and reported
- Import does **not** roll back valid rows if some rows fail
- This allows large imports to succeed even with a few problem rows

**Example Scenario:**
- CSV with 50 teachers
- 47 valid rows → 47 teachers created successfully
- 3 invalid rows → Skipped with detailed error report
- Import status: **Partially Successful** (47 created, 3 failed)

### Subscription Plan Restrictions

Teacher import operations are subject to subscription plan limits:

| Plan | Can Import Teachers | Teacher-Admin Role Allowed | Max Teachers | Notes |
|------|---------------------|---------------------------|--------------|-------|
| **PRELUDE** | ✅ Yes | ❌ No | 1 | Free trial, limited to 1 teacher account |
| **SOLO** | ✅ Yes | ❌ No | 1 | Individual instructor, teacher = subscriber-admin |
| **ENSEMBLE** | ✅ Yes | ✅ Yes | Unlimited | School/institution plan, supports multiple teachers and teacher-admins |

**Validation Rules:**
- Importing teachers to PRELUDE subscription that would exceed 1 teacher triggers `ERR_SEAT_LIMIT_EXCEEDED`
- Importing teachers to SOLO subscription that would exceed 1 teacher triggers `ERR_SEAT_LIMIT_EXCEEDED`
- Attempting to create `teacher_admin` role in PRELUDE or SOLO plans triggers `ERR_ROLE_NOT_ALLOWED_IN_PLAN`
- ENSEMBLE subscriptions have no hard limit on teacher count, but best practices recommend reasonable limits based on organizational size

See [Seat Management](../14-payments-and-subscriptions/seat-management.md) for detailed seat allocation rules.

### Role Assignment and Permissions

**Teacher Role:**
- Created by default when `role` field is blank or set to `teacher`
- Has standard teacher permissions: create students, assign sequences, view student progress
- Cannot create other teachers or modify organizational settings
- Available in all subscription plans

**Teacher-Admin Role:**
- Created when `role` field is set to `teacher_admin` or `teacher-admin`
- Has elevated permissions: create teachers, manage all students, view all organizational data
- Cannot manage subscription or billing (reserved for Subscriber-Admin)
- **Only available in Ensemble plans**
- Requires special permission code `TEM` (Teacher Create and Manage) to create other teachers

For complete role and permission specifications, see:
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md)
- [Permissions Table](../02-roles-and-permissions/permissions-table.md)

### Duplicate Detection and Handling

The system detects duplicates using the following criteria:

**Duplicate Username:**
- If `teacher_username` is manually provided and already exists in the organization (across students and teachers) → **Validation Error** (`ERR_USERNAME_DUPLICATE`)
- If `teacher_username` is auto-generated and conflicts → System automatically appends letter suffix (see Username Generation Rules)

**Duplicate Email:**
- If `teacher_email` is provided and already exists in the organization → **Warning** (import continues, but email is logged as duplicate)
- Email duplicates are allowed but flagged for administrator review
- Recommended practice: Each teacher should have unique email for proper notification delivery

**Duplicate External ID:**
- If `external_id` is provided and already exists in the organization → **Warning** (import continues, but external ID is logged as duplicate)
- This allows for re-imports and updates (future enhancement)

**Duplicate Name (First + Last):**
- Multiple teachers can have identical first and last names
- No validation error is generated
- Auto-generated usernames will differ via suffix mechanism

### Character Encoding and Internationalization

**Supported Characters:**
- All UTF-8 characters are supported in name fields
- Accented characters are preserved in the database: `José`, `François`, `Müller`
- Non-Latin scripts are supported: Arabic, Chinese, Cyrillic, Japanese, Korean, etc.

**Username Normalization:**
- When auto-generating usernames, accented characters are transliterated to ASCII equivalents:
  - `José` → `Jose`
  - `François` → `Francois`
  - `Müller` → `Muller`
  - `O'Brien` → `OBrien`
- Manual usernames in CSV can contain accented characters if preferred by the organization

**CSV File Encoding:**
- Files **must** be UTF-8 encoded
- Byte Order Mark (BOM) is optional but supported
- Files encoded in Windows-1252, ISO-8859-1, or other encodings will trigger `ERR_INVALID_ENCODING`

### State Transitions

**Import Batch States:**

```
pending → processing → completed
                    → partially_completed
                    → failed
```

| State | Description | Next States |
|-------|-------------|-------------|
| `pending` | CSV uploaded, awaiting processing | `processing`, `failed` |
| `processing` | Validating and creating teacher accounts | `completed`, `partially_completed`, `failed` |
| `completed` | All rows processed successfully, 100% success rate | *(terminal state)* |
| `partially_completed` | Some rows succeeded, some failed | *(terminal state)* |
| `failed` | File-level validation failed or critical error | *(terminal state)* |

See [Bulk Ops Jobs](./bulk-ops-jobs.md) for detailed batch processing workflow and job management.

## UX requirements (if applicable)

### CSV Upload Interface

While the detailed UI workflow is specified in [Member Management](../05-admin-subscriber-experience/member-management.md), these UX requirements specifically relate to CSV file handling for teachers:

**File Upload Component:**
- Accept only `.csv` files (reject other file types with clear error message)
- Display file name, size, and estimated row count after upload
- Show file encoding detection result (UTF-8 confirmed or error)
- Provide "Download Template" button to get a sample CSV file with correct headers
- Display clear indication of import type: "Teacher Import" vs. "Student Import"

**Validation Feedback:**
- Display file-level validation errors immediately (before processing)
- Show progress indicator during validation (percentage of rows validated)
- Present validation results in a clear, scannable table:
  - Row number
  - Teacher name (first + last)
  - Error code and human-readable message
  - Suggested fix (where applicable)

**Error Messages (CSV-Specific):**

| Error Code | User-Friendly Message | Suggested Fix |
|------------|----------------------|---------------|
| `ERR_INVALID_FILE_FORMAT` | "File is not a valid CSV file. Please ensure you're uploading a comma-separated values (.csv) file." | "Save your Excel file as CSV format before uploading." |
| `ERR_MISSING_REQUIRED_COLUMN` | "Required column '{column_name}' is missing from CSV header row." | "Ensure the first row contains: teacher_first_name, teacher_last_name, teacher_username" |
| `ERR_USERNAME_DUPLICATE` | "Row {row_number}: Username '{username}' already exists in your organization." | "Change the username to a unique value or leave blank for auto-generation." |
| `ERR_ROLE_NOT_ALLOWED_IN_PLAN` | "Row {row_number}: Teacher-Admin role is not available in your current subscription plan ({plan_name})." | "Upgrade to Ensemble plan to create Teacher-Admin accounts, or change role to 'teacher'." |
| `ERR_EMAIL_INVALID` | "Row {row_number}: Email address '{email}' is not valid." | "Check email format (example: teacher@school.edu)." |

**Download Options:**
- "Download Error Report" button (exports failed rows as CSV for correction)
- "Download Template" button (provides empty CSV with correct headers and one example row)
- "Download Template with Sample Data" button (includes 3 example teachers for reference)

**Plan-Specific Warnings:**
- Display warning banner if importing teachers to PRELUDE or SOLO plan: "Your plan allows only 1 teacher. Bulk teacher import may fail if limit is exceeded."
- Display info banner if importing Teacher-Admin roles: "Teacher-Admin role is only available in Ensemble plans."

### Accessibility Notes

- File upload component must be keyboard accessible (standard `<input type="file">` or equivalent)
- Validation error table must be screen reader friendly with proper ARIA labels
- Error messages must use sufficient color contrast (WCAG AA compliant)
- Progress indicators must announce status updates to screen readers
- Role selection errors must be clearly announced to assistive technologies

See [A11y Standards](../01-ux-design-system/a11y-standards.md) for comprehensive accessibility requirements.

## Telemetry

### Events

All CSV import events follow the standard event model defined in [Event Model](../15-analytics-and-reporting/event-model.md).

#### `csv_import_started`
**Fires when:** Administrator initiates CSV teacher import (file uploaded and processing begins)

**Properties:**
```json
{
  "event_type": "csv_import_started",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "import_batch_id": "batch_abc123def456",
  "import_entity_type": "teacher",
  "file_name": "teachers_fall_2025.csv",
  "file_size_bytes": 45678,
  "row_count": 25,
  "mode": "validation_only | import",
  "uploaded_via": "web_ui | api",
  "timestamp": "2025-10-12T14:30:00Z"
}
```

#### `csv_import_validated`
**Fires when:** CSV file validation completes (before import)

**Properties:**
```json
{
  "event_type": "csv_import_validated",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "import_batch_id": "batch_abc123def456",
  "import_entity_type": "teacher",
  "total_rows": 25,
  "valid_rows": 23,
  "invalid_rows": 2,
  "validation_duration_ms": 1500,
  "error_codes": ["ERR_USERNAME_DUPLICATE", "ERR_ROLE_NOT_ALLOWED_IN_PLAN"],
  "error_code_counts": {
    "ERR_USERNAME_DUPLICATE": 1,
    "ERR_ROLE_NOT_ALLOWED_IN_PLAN": 1
  },
  "timestamp": "2025-10-12T14:30:03Z"
}
```

#### `csv_import_completed`
**Fires when:** CSV import processing finishes (all valid rows processed)

**Properties:**
```json
{
  "event_type": "csv_import_completed",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "import_batch_id": "batch_abc123def456",
  "import_entity_type": "teacher",
  "status": "completed | partially_completed | failed",
  "total_rows": 25,
  "teachers_created": 23,
  "teachers_failed": 2,
  "usernames_auto_generated": 15,
  "passwords_auto_generated": 20,
  "teacher_role_count": 20,
  "teacher_admin_role_count": 3,
  "processing_duration_ms": 8500,
  "timestamp": "2025-10-12T14:30:12Z"
}
```

#### `csv_import_failed`
**Fires when:** CSV import encounters a critical error and cannot proceed

**Properties:**
```json
{
  "event_type": "csv_import_failed",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "import_batch_id": "batch_abc123def456",
  "import_entity_type": "teacher",
  "failure_reason": "file_too_large | invalid_encoding | missing_header | system_error",
  "error_code": "ERR_FILE_TOO_LARGE",
  "error_message": "File size exceeds 10 MB limit",
  "timestamp": "2025-10-12T14:30:02Z"
}
```

#### `csv_template_downloaded`
**Fires when:** User downloads CSV template file

**Properties:**
```json
{
  "event_type": "csv_template_downloaded",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "template_type": "minimal | standard | comprehensive",
  "entity_type": "teacher",
  "timestamp": "2025-10-12T14:25:00Z"
}
```

#### `csv_error_report_downloaded`
**Fires when:** User downloads error report for failed rows

**Properties:**
```json
{
  "event_type": "csv_error_report_downloaded",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "import_batch_id": "batch_abc123def456",
  "entity_type": "teacher",
  "error_row_count": 2,
  "timestamp": "2025-10-12T14:35:00Z"
}
```

### Analytics and Monitoring

**Key Metrics to Track:**
- Teacher import success rate (completed / total imports)
- Average validation errors per import
- Most common error codes (especially role-related errors)
- Average file size and row count for teacher imports
- Processing time percentiles (p50, p95, p99)
- Auto-generation usage rates (usernames, passwords)
- Teacher vs. Teacher-Admin creation ratio by plan type

**Operational Alerts:**
- Teacher import failure rate > 10% in last 24 hours
- Average processing time > 15 seconds (teacher imports typically smaller than student imports)
- Role permission errors > 20% of imports (indicates plan confusion)
- File encoding errors > 5% of uploads

See [Observability](../19-quality-and-operations/observability.md) for monitoring infrastructure.

## Permissions

CSV teacher import operations require specific permissions based on user role.

### Role-Based Access

| Role | Can Upload CSV | Can Validate | Can Import | Can Download Error Reports | Scope |
|------|---------------|--------------|------------|---------------------------|-------|
| **Subscriber-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Entire organization |
| **Teacher-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Entire organization (Ensemble only) |
| **Teacher** | ❌ No | ❌ No | ❌ No | ❌ No | N/A |
| **Student** | ❌ No | ❌ No | ❌ No | ❌ No | N/A |
| **System-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Any organization (with audit logging) |

**Permission Codes (from legacy MLC 4.0):**
- `TEM` (Teacher Create and Manage): Required for CSV teacher import operations
- `UPU` (Upload Utility Use): Required for CSV import operations (applies to both students and teachers)

**Important Notes:**
- Only Subscriber-Admin and Teacher-Admin roles have the `TEM` permission to create and manage teachers
- Teacher-Admin can create other teachers but cannot create additional Teacher-Admin accounts (reserved for Subscriber-Admin)
- Solo plan subscribers have teacher permissions but cannot import additional teachers (plan limit: 1 teacher)

See [Permissions Table](../02-roles-and-permissions/permissions-table.md) for comprehensive role and permission matrix.

### Subscription Plan Restrictions

| Plan | Can Import Teachers | Max Teachers per Import | Notes |
|------|---------------------|------------------------|-------|
| **PRELUDE** | ✅ Yes | 1 | Free trial, CSV import typically not used |
| **SOLO** | ✅ Yes | 1 | Individual instructor, CSV import typically not used |
| **ENSEMBLE** | ✅ Yes | 10,000 | School/institution plan, primary use case for CSV imports |

**Enforcement:**
- Imports that would exceed subscription teacher limits are rejected with `ERR_SEAT_LIMIT_EXCEEDED`
- Teacher-Admin role assignment is blocked in PRELUDE/SOLO plans with `ERR_ROLE_NOT_ALLOWED_IN_PLAN`

See [Seat Management](../14-payments-and-subscriptions/seat-management.md) for detailed seat allocation rules.

### Data Privacy and Security

**PII Handling:**
- CSV files contain teacher Personally Identifiable Information (PII): names, emails, phone numbers
- Files must be transmitted over HTTPS (TLS 1.2+)
- Uploaded CSV files are stored encrypted at rest for 90 days, then permanently deleted
- Import batch metadata is retained for audit purposes (see [Audit Logs](../06-system-admin/sysadmin-audit-logs.md))

**Employment Data:**
- Teacher data may be considered employment records in some jurisdictions
- Organizations are responsible for compliance with local employment data regulations
- External IDs should not contain sensitive employee information (SSN, etc.)

**Access Control:**
- Only authorized administrators (Subscriber-Admin, Teacher-Admin) can import teacher data
- All import operations are logged with user ID, timestamp, and affected teachers
- Teacher accounts cannot access or export other teacher data

See [Security & Privacy](../18-architecture/security-privacy.md) for comprehensive security requirements.

## Supporting Documents Referenced

This CSV specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role specifications informing teacher and teacher-admin role definitions, permission assignments, and role-based access control for CSV import operations
- [Roles Chart.pdf](../00-ORIG-CONTEXT/Roles%20Chart.pdf) — Visual role hierarchy chart informing teacher role creation permissions and organizational structure requirements
- [Roles Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Roles%20Chart.xlsx%20-%20Sheet1.csv) — Detailed role permissions matrix informing teacher account creation permissions and CSV import authorization rules
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription and seat management specifications informing teacher seat limit validation, subscription plan restrictions, and teacher allocation rules for Solo and Ensemble plans
- [Mass Upload.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Mass%20Upload.xlsx%20-%20Sheet1.csv) — Legacy bulk upload format informing username generation patterns (shared with student imports) and password assignment algorithms

---

## Dependencies

This specification relies on the following systems and specifications:

### Internal Dependencies

| Dependency | Purpose | Link |
|------------|---------|------|
| **Member Management** | Defines the UI workflow and organizational structure for managing teachers | [Member Management](../05-admin-subscriber-experience/member-management.md) |
| **Bulk Ops Jobs** | Background job processing system for handling CSV imports | [Bulk Ops Jobs](./bulk-ops-jobs.md) |
| **Data Model ERD** | Defines User entity structure and relationships for teachers | [Data Model ERD](../18-architecture/data-model-erd.md) |
| **Roles Matrix** | Specifies teacher and teacher-admin role definitions and capabilities | [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) |
| **Permissions Table** | Specifies role-based access control for import operations | [Permissions Table](../02-roles-and-permissions/permissions-table.md) |
| **Event Model** | Defines telemetry event structure for tracking imports | [Event Model](../15-analytics-and-reporting/event-model.md) |
| **Seat Management** | Enforces subscription seat limits during import | [Seat Management](../14-payments-and-subscriptions/seat-management.md) |
| **REST Endpoints** | API contracts for programmatic CSV import | [REST Endpoints](../18-architecture/rest-endpoints.md) |
| **Security & Privacy** | Password hashing, encryption, and data protection requirements | [Security & Privacy](../18-architecture/security-privacy.md) |
| **CSV Spec Students** | Parallel specification for student imports, shares username/password generation logic | [CSV Spec Students](./csv-spec-students.md) |

### External Dependencies

| Dependency | Purpose | Notes |
|------------|---------|-------|
| **CSV Parser Library** | Parse and validate CSV file format | Use battle-tested library (e.g., `csv-parse` for Node.js, `csv` for Python) |
| **Character Encoding Detection** | Detect and validate UTF-8 encoding | Use `chardet` or equivalent library |
| **Email Validator** | Validate email address format per RFC 5322 | Use standard library or `validator.js` |
| **Bcrypt** | Hash teacher passwords securely | Use bcrypt with cost factor 12 (see Security spec) |
| **Background Job Queue** | Process large imports asynchronously | Use Redis Queue, Sidekiq, or equivalent (see [Background Jobs](../18-architecture/background-jobs.md)) |

### Legacy System Compatibility

**MLC 4.0 Compatibility:**
- This specification maintains backward compatibility with MLC 4.0 teacher management workflows
- Username and password generation algorithms are identical to MLC 4.0
- Permission codes (`TEM`, `UPU`) are preserved from legacy system
- Teacher role structure (Teacher vs. Teacher-Admin) remains consistent

See [Legacy Systems](../00-foundations/legacy-systems.md) for migration considerations.

## Open questions

### Technical Decisions Needed

1. **CSV Parser Library:**
   - Which specific CSV parsing library should be used for the backend?
   - Recommendation: `csv-parse` (Node.js) or `csv` (Python) for robustness and standards compliance
   - **Owner:** Backend tech lead
   - **Due:** Phase 1 technical design sprint

2. **Teacher Update via CSV (Future Enhancement):**
   - Should we support "update existing teacher" mode when username matches?
   - Current spec: Duplicates are rejected; only "create new" is supported
   - Future: Allow CSV imports to update existing teachers (idempotent imports)
   - Use case: Updating teacher emails, departments, or other profile information
   - **Owner:** Product manager
   - **Due:** Phase 2 planning (post-MVP)

3. **Teacher-Admin Creation Restrictions:**
   - Should only Subscriber-Admin be allowed to create Teacher-Admin via CSV?
   - Should existing Teacher-Admin be allowed to create other Teacher-Admin accounts via CSV?
   - Current spec: Teacher-Admin role creation is allowed via CSV in Ensemble plans
   - **Owner:** Product manager, Security architect
   - **Due:** Phase 1 requirements review

4. **Asynchronous Processing Threshold:**
   - At what row count should CSV processing move to background jobs?
   - Small files (< 50 rows): Synchronous processing (immediate feedback)
   - Large files (≥ 50 rows): Asynchronous processing (job queue with status polling)
   - Note: Teacher imports typically much smaller than student imports
   - **Owner:** Backend tech lead
   - **Due:** Phase 1 technical design sprint

### UX and Product Questions

5. **Teacher Notification on Account Creation:**
   - Should teachers receive an email notification when their account is created via CSV import?
   - Should the email include their username and temporary password?
   - Current spec: No automatic notification specified
   - Alternative: Send welcome email with login instructions
   - **Owner:** Product designer, Product manager
   - **Due:** Phase 1 UX design sprint

6. **CSV Template Variants:**
   - How many template variants should we provide for teachers? (minimal, standard, comprehensive)
   - Should templates include example data or be empty?
   - Should we provide separate templates for Teacher vs. Teacher-Admin imports?
   - **Owner:** Product designer
   - **Due:** Phase 1 UX design sprint

7. **Error Report Format:**
   - Should error reports be CSV, Excel, PDF, or all three?
   - Current spec: CSV format for easy re-import after corrections
   - **Owner:** Product designer
   - **Due:** Phase 1 UX design sprint

### Business and Compliance Questions

8. **Data Retention for Uploaded CSVs:**
   - How long should original teacher CSV files be retained?
   - Current spec: 90 days encrypted, then permanently deleted
   - Employment data considerations: Some jurisdictions require longer retention
   - **Owner:** Legal counsel, Compliance officer
   - **Due:** Before Phase 1 launch

9. **Teacher Email Requirement:**
   - Should teacher email be required (not optional)?
   - Current spec: Email is optional
   - Recommendation: Make email required for proper system notifications and communication
   - **Owner:** Product manager
   - **Due:** Phase 1 requirements review

10. **External HR System Integration:**
    - Which HR systems should we integrate with directly for teacher data sync?
    - Popular systems: BambooHR, ADP Workforce Now, Workday, UKG
    - Should CSV import be the primary integration method or should we build native connectors?
    - **Owner:** Product manager, Partnerships lead
    - **Due:** Phase 2 planning

### Integration Questions

11. **API for Programmatic Import:**
    - Should we expose a REST API endpoint for programmatic teacher CSV import?
    - Use case: School district IT departments automating nightly teacher roster syncs
    - **Owner:** API architect, Product manager
    - **Due:** Phase 2 planning

12. **Student-Teacher Association During Import:**
    - Should teacher CSV imports support student assignment at import time?
    - Alternative approach: Separate CSV for teacher-student associations
    - See [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md) for related functionality
    - **Owner:** Product manager
    - **Due:** Phase 1 requirements review

---

**Last Updated:** 2025-10-12  
**Status:** Draft - Ready for Review  
**Owner:** Product Team  
**Stakeholders:** Engineering, UX Design, Legal/Compliance, Customer Success
