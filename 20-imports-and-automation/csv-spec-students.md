# CSV Spec Students

This document defines the CSV file format specification for importing student data into the MLC-LMS platform. It specifies the exact structure, field requirements, validation rules, and formatting guidelines for CSV files used to create student accounts in bulk.

## Purpose

This specification enables:

- **Efficient Student Onboarding**: Schools can import hundreds or thousands of students rapidly using CSV files generated from their Student Information Systems (SIS) or administrative databases
- **Standardized Data Format**: Ensures consistent CSV structure across all organizations, reducing errors and support overhead
- **Automated Account Creation**: Enables automatic generation of usernames and passwords when not provided, following MLC naming conventions
- **Data Validation**: Provides clear validation rules to catch errors before import, ensuring data quality and system integrity
- **Legacy Support**: Maintains compatibility with MLC 4.0 "Mass Upload Utility" format while supporting modern enhancements
- **IT Department Integration**: Allows school IT staff to generate student lists programmatically without manual data entry

This specification is narrowly focused on the CSV file format itself. For the bulk operations workflow, UI, and batch processing details, see [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md).

## Scope

**Included:**
- CSV file structure and formatting requirements
- Required and optional column specifications
- Data type definitions and constraints
- Field-level validation rules
- Username and password generation algorithms
- Character encoding and special character handling
- Example CSV templates and sample data
- Error codes for validation failures

**Excluded:**
- Bulk operations UI and workflow (see [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md))
- Import batch processing and job management (see [Bulk Ops Jobs](./bulk-ops-jobs.md))
- Student data model beyond CSV fields (see [Data Model ERD](../18-architecture/data-model-erd.md))
- Teacher and course assignment CSV specs (see [CSV Spec Teachers](./csv-spec-teachers.md) and [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md))
- API endpoints for import operations (see [REST Endpoints](../18-architecture/rest-endpoints.md))

## Data model (if applicable)

### CSV Column Definitions

The student import CSV supports the following columns. Column order is flexible, but the first row **must** contain column headers that exactly match these names (case-insensitive).

#### Required Columns

| Column Name | Data Type | Max Length | Description | Example |
|-------------|-----------|------------|-------------|---------|
| `student_first_name` | String | 50 | Student's first name (given name) | `John` |
| `student_last_name` | String | 50 | Student's last name (surname/family name) | `Smith` |
| `student_username` | String | 100 | Unique username for login (see Username Generation Rules) | `SmithJohn17502` or leave blank for auto-generation |

**Notes:**
- If `student_username` is blank, the system will auto-generate using the naming convention: `{LastName}{FirstName}{OrgMemberNumber}`
- Auto-generated usernames are normalized: spaces removed, special characters stripped, camelCase applied

#### Optional Columns

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `student_password` | String | 100 | Initial password for the student account | `piano` or leave blank | Auto-generated from standard password list |
| `student_email` | Email | 255 | Student's email address (optional, for future notifications) | `john.smith@school.edu` | `null` |
| `active_status` | String | 1 | Account status: `Y` for active, blank or `N` for inactive | `Y` | `Y` (active) |
| `teacher_real_name` | String | 100 | Full name of teacher to assign student to (must match existing teacher) | `Jane Doe` | `null` (unassigned) |
| `teacher_username` | String | 100 | Username of teacher to assign student to (alternative to teacher_real_name) | `DoeJane17502` | `null` (unassigned) |
| `course_assignment_id` | String | 50 | ID of sequence/course to assign (e.g., `657` for LIFE Primary 1A Assignment 1) | `657` | `null` (no initial assignment) |
| `grade_level` | String | 20 | Student's grade level or year | `Grade 3`, `5th Grade`, `Year 4` | `null` |
| `class_name` | String | 100 | Name of class or section | `Music 101`, `Period 2 - Music` | `null` |
| `student_type` | String | 20 | Student account type: `regular` or `generic` | `regular` | `regular` |
| `external_id` | String | 100 | External system identifier (SIS ID, school ID number) | `STU-2025-001234` | `null` |
| `date_of_birth` | Date | 10 | Student's date of birth in `YYYY-MM-DD` format | `2012-08-15` | `null` |
| `notes` | String | 500 | Administrative notes about the student | `Needs accessible keyboard` | `null` |

**Notes:**
- If `student_password` is blank, the system auto-generates a password from a rotating list of simple musical terms (see Password Generation Rules)
- If `teacher_real_name` or `teacher_username` is provided, the teacher must already exist in the organization
- If `course_assignment_id` is provided, the assignment is created immediately upon import
- `student_type` of `generic` creates a shared account for multiple students (typically used in General Music classes where score tracking is not required)

### Username Generation Rules

When `student_username` is blank or omitted, the system automatically generates usernames using the following algorithm:

**Basic Pattern:**
```
{LastName}{FirstName}{OrgMemberNumber}
```

**Example:**
- Organization Member Number: `17502`
- Student Name: John Smith
- Generated Username: `SmithJohn17502`

**Normalization Steps:**
1. Remove all whitespace from first and last names
2. Strip special characters and accents (e.g., `José` → `Jose`, `O'Brien` → `OBrien`)
3. Capitalize first letter of last name, first letter of first name (PascalCase)
4. Append organization member number
5. Check for duplicate usernames in the organization

**Duplicate Resolution:**
If the generated username already exists (e.g., multiple John Smiths), the system appends a letter suffix to the first name and retries:

```
SmithJohn17502      (first student)
SmithJohnA17502     (second student with same name)
SmithJohnB17502     (third student with same name)
...
SmithJohnZ17502     (26th student)
SmithJohnAA17502    (27th student - double letter suffix)
```

**Collision Detection:**
- Username uniqueness is checked at the **organization level** (not globally)
- If a manually-provided username conflicts with an existing username, the import validation will fail for that row with error code `ERR_USERNAME_DUPLICATE`

### Password Generation Rules

When `student_password` is blank or omitted, the system automatically assigns a password from a rotating list of simple musical terms.

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
2. Assign sequentially to each student in the CSV file
3. When reaching the end of the list, cycle back to the beginning
4. Continue until all students have passwords

**Example for 5 students:**
- Student 1: `music`
- Student 2: `piano`
- Student 3: `note`
- Student 4: `sharp`
- Student 5: `flat`

**Security Notes:**
- These are initial passwords intended for easy distribution to students
- Students (and teachers/admins) can change passwords after first login
- For enhanced security, organizations may provide their own passwords in the CSV file
- Passwords are stored as bcrypt hashes in the database (see [Security & Privacy](../18-architecture/security-privacy.md))

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
| **Maximum File Size** | 10 MB | Approximately 50,000 students |
| **Maximum Rows** | 50,000 rows | Excluding header row |

#### Field Formatting Rules

1. **Text Fields:**
   - Enclose in double quotes if containing commas, quotes, or line breaks
   - Escape internal double quotes by doubling them: `"He said ""hello"""` → `He said "hello"`
   - Leading and trailing whitespace is automatically trimmed
   - Empty fields can be represented as `""` or left blank between commas

2. **Boolean Fields (`active_status`):**
   - `Y`, `y`, `YES`, `yes`, `True`, `true`, `1` → Active
   - `N`, `n`, `NO`, `no`, `False`, `false`, `0`, blank → Inactive

3. **Date Fields (`date_of_birth`):**
   - Format: `YYYY-MM-DD` (ISO 8601)
   - Example: `2012-08-15` for August 15, 2012
   - Invalid dates will generate validation error `ERR_INVALID_DATE`

4. **Email Fields (`student_email`):**
   - Must match standard email format: `local@domain.tld`
   - Maximum length: 255 characters
   - Case-insensitive
   - Validation uses RFC 5322 standard

5. **Special Characters:**
   - Accented characters are supported: `José`, `François`, `Müller`
   - Emoji and non-Latin scripts are supported but discouraged for usernames
   - Commas within fields must be enclosed in quotes: `"Smith, Jr.", John`

#### Example CSV Templates

**Minimal CSV (Required Fields Only with Auto-Generation):**
```csv
student_first_name,student_last_name,student_username
John,Smith,
Emily,Johnson,
Michael,Williams,
```
*Note: Blank usernames will be auto-generated; passwords will be auto-assigned*

**Standard CSV (Common Use Case):**
```csv
student_first_name,student_last_name,student_username,student_password,active_status,teacher_real_name,grade_level,class_name
John,Smith,SmithJohn17502,music,Y,Jane Doe,Grade 3,Music 101
Emily,Johnson,JohnsonEmily17502,piano,Y,Jane Doe,Grade 3,Music 101
Michael,Williams,WilliamsMichael17502,note,Y,Bob Martin,Grade 4,Music 102
Sarah,Davis,DavisSarah17502,sharp,Y,Bob Martin,Grade 4,Music 102
```

**Comprehensive CSV (All Optional Fields):**
```csv
student_first_name,student_last_name,student_username,student_password,student_email,active_status,teacher_real_name,course_assignment_id,grade_level,class_name,student_type,external_id,date_of_birth,notes
John,Smith,SmithJohn17502,music,john.smith@school.edu,Y,Jane Doe,657,Grade 3,Music 101,regular,STU-2025-001234,2012-08-15,Needs accessible keyboard
Emily,Johnson,JohnsonEmily17502,piano,emily.johnson@school.edu,Y,Jane Doe,657,Grade 3,Music 101,regular,STU-2025-001235,2012-11-03,
Michael,Williams,WilliamsMichael17502,note,michael.williams@school.edu,Y,Bob Martin,658,Grade 4,Music 102,regular,STU-2025-001236,2011-05-22,
GenericStudent1,MusicClass,,shared,,,Jane Doe,,Grade 3,Music 101,generic,,,Shared account for general music class
```

**CSV with Special Characters:**
```csv
student_first_name,student_last_name,student_username,student_password,active_status,notes
José,García,GarciaJose17502,music,Y,Original name: José García
François,Müller,MullerFrancois17502,piano,Y,Original name: François Müller
Mary,"O'Brien",OBrienMary17502,note,Y,"Last name contains apostrophe"
"Smith, Jr.",Robert,SmithJrRobert17502,sharp,Y,"Last name contains comma"
```

## Behavior and rules

### Validation Rules

The CSV import system validates each row before creating student accounts. Validation occurs in two phases:

#### Phase 1: File-Level Validation (Pre-Processing)

These checks occur before any data is processed:

| Rule | Description | Error Code |
|------|-------------|------------|
| **File Format** | File must be valid CSV with `.csv` extension | `ERR_INVALID_FILE_FORMAT` |
| **File Size** | File must not exceed 10 MB | `ERR_FILE_TOO_LARGE` |
| **Row Limit** | File must not exceed 50,000 rows (excluding header) | `ERR_TOO_MANY_ROWS` |
| **Character Encoding** | File must be UTF-8 encoded | `ERR_INVALID_ENCODING` |
| **Header Row** | First row must contain valid column headers | `ERR_MISSING_HEADER` |
| **Required Columns** | Must include `student_first_name`, `student_last_name`, `student_username` | `ERR_MISSING_REQUIRED_COLUMN` |

If any file-level validation fails, the entire import is rejected and no rows are processed.

#### Phase 2: Row-Level Validation (Per-Student)

These checks occur for each student row in the CSV:

**Required Field Validation:**

| Field | Rule | Error Code |
|-------|------|------------|
| `student_first_name` | Must not be empty or whitespace-only | `ERR_FIRST_NAME_REQUIRED` |
| `student_first_name` | Must not exceed 50 characters | `ERR_FIRST_NAME_TOO_LONG` |
| `student_first_name` | Must contain at least one letter | `ERR_FIRST_NAME_INVALID` |
| `student_last_name` | Must not be empty or whitespace-only | `ERR_LAST_NAME_REQUIRED` |
| `student_last_name` | Must not exceed 50 characters | `ERR_LAST_NAME_TOO_LONG` |
| `student_last_name` | Must contain at least one letter | `ERR_LAST_NAME_INVALID` |
| `student_username` | If provided, must be 3-100 characters | `ERR_USERNAME_INVALID_LENGTH` |
| `student_username` | If provided, must be unique within organization | `ERR_USERNAME_DUPLICATE` |
| `student_username` | If provided, must contain only alphanumeric characters and underscores | `ERR_USERNAME_INVALID_CHARS` |

**Optional Field Validation:**

| Field | Rule | Error Code |
|-------|------|------------|
| `student_password` | If provided, must be 4-100 characters | `ERR_PASSWORD_INVALID_LENGTH` |
| `student_email` | If provided, must be valid email format | `ERR_EMAIL_INVALID` |
| `student_email` | If provided, must be unique within organization | `ERR_EMAIL_DUPLICATE` |
| `active_status` | If provided, must be Y/N or valid boolean value | `ERR_ACTIVE_STATUS_INVALID` |
| `teacher_real_name` | If provided, teacher must exist in organization | `ERR_TEACHER_NOT_FOUND` |
| `teacher_username` | If provided, teacher must exist in organization | `ERR_TEACHER_NOT_FOUND` |
| `course_assignment_id` | If provided, course/sequence must exist and be accessible | `ERR_COURSE_NOT_FOUND` |
| `grade_level` | If provided, must not exceed 20 characters | `ERR_GRADE_LEVEL_TOO_LONG` |
| `class_name` | If provided, must not exceed 100 characters | `ERR_CLASS_NAME_TOO_LONG` |
| `student_type` | If provided, must be `regular` or `generic` | `ERR_STUDENT_TYPE_INVALID` |
| `external_id` | If provided, must not exceed 100 characters | `ERR_EXTERNAL_ID_TOO_LONG` |
| `date_of_birth` | If provided, must be valid date in YYYY-MM-DD format | `ERR_DATE_OF_BIRTH_INVALID` |
| `date_of_birth` | If provided, student must be between 3 and 120 years old | `ERR_DATE_OF_BIRTH_OUT_OF_RANGE` |
| `notes` | If provided, must not exceed 500 characters | `ERR_NOTES_TOO_LONG` |

### Import Processing Modes

The CSV import system supports two processing modes:

#### 1. Validation-Only Mode (Dry Run)

- Validates the entire CSV file without creating any student accounts
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

- Processes valid rows and creates student accounts
- Skips invalid rows and reports errors
- Generates an import summary report
- Creates import batch record for tracking (see [Bulk Ops Jobs](./bulk-ops-jobs.md))

**Processing Strategy: Partial Success**
- Valid rows are processed and committed
- Invalid rows are skipped and reported
- Import does **not** roll back valid rows if some rows fail
- This allows large imports to succeed even with a few problem rows

**Example Scenario:**
- CSV with 1,000 students
- 975 valid rows → 975 students created successfully
- 25 invalid rows → Skipped with detailed error report
- Import status: **Partially Successful** (975 created, 25 failed)

### Duplicate Detection and Handling

The system detects duplicates using the following criteria:

**Duplicate Username:**
- If `student_username` is manually provided and already exists in the organization → **Validation Error** (`ERR_USERNAME_DUPLICATE`)
- If `student_username` is auto-generated and conflicts → System automatically appends letter suffix (see Username Generation Rules)

**Duplicate Email:**
- If `student_email` is provided and already exists in the organization → **Warning** (import continues, but email is logged as duplicate)
- Email duplicates are allowed but flagged for administrator review

**Duplicate External ID:**
- If `external_id` is provided and already exists in the organization → **Warning** (import continues, but external ID is logged as duplicate)
- This allows for re-imports and updates (future enhancement)

**Duplicate Name (First + Last):**
- Multiple students can have identical first and last names
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
| `processing` | Validating and creating student accounts | `completed`, `partially_completed`, `failed` |
| `completed` | All rows processed successfully, 100% success rate | *(terminal state)* |
| `partially_completed` | Some rows succeeded, some failed | *(terminal state)* |
| `failed` | File-level validation failed or critical error | *(terminal state)* |

See [Bulk Ops Jobs](./bulk-ops-jobs.md) for detailed batch processing workflow and job management.

## UX requirements (if applicable)

### CSV Upload Interface

While the detailed UI workflow is specified in [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md), these UX requirements specifically relate to CSV file handling:

**File Upload Component:**
- Accept only `.csv` files (reject other file types with clear error message)
- Display file name, size, and estimated row count after upload
- Show file encoding detection result (UTF-8 confirmed or error)
- Provide "Download Template" button to get a sample CSV file with correct headers

**Validation Feedback:**
- Display file-level validation errors immediately (before processing)
- Show progress indicator during validation (percentage of rows validated)
- Present validation results in a clear, scannable table:
  - Row number
  - Student name (first + last)
  - Error code and human-readable message
  - Suggested fix (where applicable)

**Error Messages (CSV-Specific):**

| Error Code | User-Friendly Message | Suggested Fix |
|------------|----------------------|---------------|
| `ERR_INVALID_FILE_FORMAT` | "File is not a valid CSV file. Please ensure you're uploading a comma-separated values (.csv) file." | "Save your Excel file as CSV format before uploading." |
| `ERR_MISSING_REQUIRED_COLUMN` | "Required column '{column_name}' is missing from CSV header row." | "Ensure the first row contains: student_first_name, student_last_name, student_username" |
| `ERR_USERNAME_DUPLICATE` | "Row {row_number}: Username '{username}' already exists in your organization." | "Change the username to a unique value or leave blank for auto-generation." |
| `ERR_TEACHER_NOT_FOUND` | "Row {row_number}: Teacher '{teacher_name}' not found in your organization." | "Verify teacher name spelling or leave blank to assign students later." |

**Download Options:**
- "Download Error Report" button (exports failed rows as CSV for correction)
- "Download Template" button (provides empty CSV with correct headers and one example row)

### Accessibility Notes

- File upload component must be keyboard accessible (standard `<input type="file">` or equivalent)
- Validation error table must be screen reader friendly with proper ARIA labels
- Error messages must use sufficient color contrast (WCAG AA compliant)
- Progress indicators must announce status updates to screen readers

See [A11y Standards](../01-ux-design-system/a11y-standards.md) for comprehensive accessibility requirements.

## Telemetry

### Events

All CSV import events follow the standard event model defined in [Event Model](../15-analytics-and-reporting/event-model.md).

#### `csv_import_started`
**Fires when:** Administrator initiates CSV student import (file uploaded and processing begins)

**Properties:**
```json
{
  "event_type": "csv_import_started",
  "user_id": "usr_12345",
  "organization_id": "org_54321",
  "import_batch_id": "batch_abc123def456",
  "file_name": "students_fall_2025.csv",
  "file_size_bytes": 245678,
  "row_count": 350,
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
  "total_rows": 350,
  "valid_rows": 340,
  "invalid_rows": 10,
  "validation_duration_ms": 2500,
  "error_codes": ["ERR_USERNAME_DUPLICATE", "ERR_TEACHER_NOT_FOUND"],
  "error_code_counts": {
    "ERR_USERNAME_DUPLICATE": 7,
    "ERR_TEACHER_NOT_FOUND": 3
  },
  "timestamp": "2025-10-12T14:30:05Z"
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
  "status": "completed | partially_completed | failed",
  "total_rows": 350,
  "students_created": 340,
  "students_failed": 10,
  "usernames_auto_generated": 280,
  "passwords_auto_generated": 320,
  "students_assigned_to_teachers": 310,
  "students_assigned_courses": 250,
  "duplicate_username_collisions": 15,
  "processing_duration_ms": 12500,
  "timestamp": "2025-10-12T14:30:18Z"
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
  "error_row_count": 10,
  "timestamp": "2025-10-12T14:35:00Z"
}
```

### Analytics and Monitoring

**Key Metrics to Track:**
- Import success rate (completed / total imports)
- Average validation errors per import
- Most common error codes
- Average file size and row count
- Processing time percentiles (p50, p95, p99)
- Auto-generation usage rates (usernames, passwords)

**Operational Alerts:**
- Import failure rate > 10% in last 24 hours
- Average processing time > 30 seconds
- File encoding errors > 5% of uploads

See [Observability](../19-quality-and-operations/observability.md) for monitoring infrastructure.

## Permissions

CSV student import operations require specific permissions based on user role.

### Role-Based Access

| Role | Can Upload CSV | Can Validate | Can Import | Can Download Error Reports | Scope |
|------|---------------|--------------|------------|---------------------------|-------|
| **Subscriber-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Entire organization |
| **Teacher-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Entire organization |
| **Teacher** | ❌ No | ❌ No | ❌ No | ❌ No | N/A |
| **Student** | ❌ No | ❌ No | ❌ No | ❌ No | N/A |
| **System-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | Any organization (with audit logging) |

**Permission Codes (from legacy MLC 4.0):**
- `UPU` (Upload Utility Use): Required for CSV import operations
- `STM` (Student Create and Manage): Required to create student accounts

See [Permissions Table](../02-roles-and-permissions/permissions-table.md) for comprehensive role and permission matrix.

### Subscription Plan Restrictions

| Plan | Can Import Students | Max Students per Import | Notes |
|------|---------------------|------------------------|-------|
| **PRELUDE** | ✅ Yes | 5 | Limited to 1 teacher, max 5 students total |
| **SOLO** | ✅ Yes | 5 | Individual instructor, max 5 students |
| **ENSEMBLE** | ✅ Yes | 50,000 | School/institution plan, unlimited teachers |

**Seat Limit Enforcement:**
- Imports that would exceed subscription seat limits are rejected with `ERR_SEAT_LIMIT_EXCEEDED`
- System calculates: `current_students + new_students <= seat_limit`
- Administrators are prompted to upgrade subscription if needed

See [Seat Management](../14-payments-and-subscriptions/seat-management.md) for detailed seat allocation rules.

### Data Privacy and FERPA Compliance

**PII Handling:**
- CSV files contain student Personally Identifiable Information (PII): names, emails, dates of birth
- Files must be transmitted over HTTPS (TLS 1.2+)
- Uploaded CSV files are stored encrypted at rest for 90 days, then permanently deleted
- Import batch metadata is retained for audit purposes (see [Audit Logs](../06-system-admin/sysadmin-audit-logs.md))

**FERPA Compliance:**
- Only authorized educational officials (Admins, Teacher-Admins) can import student data
- All import operations are logged with user ID, timestamp, and affected students
- Students and parents cannot access or export other students' data

See [FERPA Compliance](../23-legal-and-compliance/ferpa-compliance.md) and [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md) for full compliance requirements.

## Supporting Documents Referenced

This CSV specification draws from the following source documents:

- [Mass Upload.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Mass%20Upload.xlsx%20-%20Sheet1.csv) — Legacy MLC 4.0 student bulk upload format informing column structure, username generation patterns, and password assignment algorithms
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role specifications informing student account creation permissions and role-based access control for CSV import operations
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription and seat management specifications informing seat limit validation, subscription plan restrictions, and student allocation rules

---

## Dependencies

This specification relies on the following systems and specifications:

### Internal Dependencies

| Dependency | Purpose | Link |
|------------|---------|------|
| **Student Bulk Operations** | Defines the UI workflow, batch processing, and job management for CSV imports | [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md) |
| **Bulk Ops Jobs** | Background job processing system for handling large CSV imports | [Bulk Ops Jobs](./bulk-ops-jobs.md) |
| **Data Model ERD** | Defines User entity structure and relationships | [Data Model ERD](../18-architecture/data-model-erd.md) |
| **Permissions Table** | Specifies role-based access control for import operations | [Permissions Table](../02-roles-and-permissions/permissions-table.md) |
| **Event Model** | Defines telemetry event structure for tracking imports | [Event Model](../15-analytics-and-reporting/event-model.md) |
| **Seat Management** | Enforces subscription seat limits during import | [Seat Management](../14-payments-and-subscriptions/seat-management.md) |
| **REST Endpoints** | API contracts for programmatic CSV import | [REST Endpoints](../18-architecture/rest-endpoints.md) |
| **Security & Privacy** | Password hashing, encryption, and data protection requirements | [Security & Privacy](../18-architecture/security-privacy.md) |

### External Dependencies

| Dependency | Purpose | Notes |
|------------|---------|-------|
| **CSV Parser Library** | Parse and validate CSV file format | Use battle-tested library (e.g., `csv-parse` for Node.js, `csv` for Python) |
| **Character Encoding Detection** | Detect and validate UTF-8 encoding | Use `chardet` or equivalent library |
| **Email Validator** | Validate email address format per RFC 5322 | Use standard library or `validator.js` |
| **Bcrypt** | Hash student passwords securely | Use bcrypt with cost factor 12 (see Security spec) |
| **Background Job Queue** | Process large imports asynchronously | Use Redis Queue, Sidekiq, or equivalent (see [Background Jobs](../18-architecture/background-jobs.md)) |

### Legacy System Compatibility

**MLC 4.0 Mass Upload Utility:**
- This specification maintains backward compatibility with MLC 4.0 CSV format
- Column names are standardized but aliases are supported:
  - `student_first_name` ← `First Name`, `FirstName`
  - `student_last_name` ← `Last Name`, `LastName`
  - `student_username` ← `User Name`, `Username`
- Username and password generation algorithms are identical to MLC 4.0

See [Legacy Systems](../00-foundations/legacy-systems.md) for migration considerations.

## Open questions

### Technical Decisions Needed

1. **CSV Parser Library:**
   - Which specific CSV parsing library should be used for the backend?
   - Recommendation: `csv-parse` (Node.js) or `csv` (Python) for robustness and standards compliance
   - **Owner:** Backend tech lead
   - **Due:** Phase 1 technical design sprint

2. **Duplicate Username Strategy - Future Enhancement:**
   - Should we support "update existing student" mode when username matches?
   - Current spec: Duplicates are rejected; only "create new" is supported
   - Future: Allow CSV imports to update existing students (idempotent imports)
   - **Owner:** Product manager
   - **Due:** Phase 2 planning (post-MVP)

3. **Asynchronous Processing Threshold:**
   - At what row count should CSV processing move to background jobs?
   - Small files (< 100 rows): Synchronous processing (immediate feedback)
   - Large files (≥ 100 rows): Asynchronous processing (job queue with status polling)
   - **Owner:** Backend tech lead
   - **Due:** Phase 1 technical design sprint

4. **Email Validation Strictness:**
   - Should we allow plus-addressing (e.g., `john+school@example.com`)?
   - Should we validate MX records for email domains?
   - Current spec: Format validation only (RFC 5322), no MX lookup
   - **Owner:** Product manager
   - **Due:** Phase 1 requirements review

5. **Password Complexity Requirements:**
   - Should auto-generated passwords be more complex than simple musical terms?
   - Should we enforce password strength rules for manually-provided passwords?
   - Current spec: Simple musical terms for ease of distribution, no strength requirements
   - **Owner:** Security architect
   - **Due:** Phase 1 security review

### UX and Product Questions

6. **Partial Import Rollback:**
   - Should administrators have the option to rollback a partially-successful import?
   - Current spec: Partial imports commit valid rows; no rollback
   - Alternative: All-or-nothing mode (rollback entire import if any row fails)
   - **Owner:** Product designer, Product manager
   - **Due:** Phase 1 UX design sprint

7. **CSV Template Variants:**
   - How many template variants should we provide? (minimal, standard, comprehensive)
   - Should templates include example data or be empty?
   - **Owner:** Product designer
   - **Due:** Phase 1 UX design sprint

8. **Error Report Format:**
   - Should error reports be CSV, Excel, PDF, or all three?
   - Current spec: CSV format for easy re-import after corrections
   - **Owner:** Product designer
   - **Due:** Phase 1 UX design sprint

### Business and Compliance Questions

9. **Data Retention for Uploaded CSVs:**
   - How long should original CSV files be retained?
   - Current spec: 90 days encrypted, then permanently deleted
   - FERPA considerations: Minimize PII retention
   - **Owner:** Legal counsel, Compliance officer
   - **Due:** Before Phase 1 launch

10. **Student Age Restrictions:**
    - Should we enforce minimum age (e.g., 3 years old) based on date_of_birth?
    - Current spec: 3-120 years allowed
    - COPPA considerations: Students under 13 have additional privacy protections
    - **Owner:** Legal counsel, Product manager
    - **Due:** Phase 1 compliance review

### Integration Questions

11. **SIS Integration Roadmap:**
    - Which Student Information Systems should we integrate with directly?
    - Popular systems: PowerSchool, Infinite Campus, Skyward, Blackbaud
    - Should CSV import be the primary integration method or should we build native connectors?
    - **Owner:** Product manager, Partnerships lead
    - **Due:** Phase 2 planning

12. **API for Programmatic Import:**
    - Should we expose a REST API endpoint for programmatic CSV import?
    - Use case: School IT departments automating nightly student roster syncs
    - **Owner:** API architect, Product manager
    - **Due:** Phase 2 planning

---

**Last Updated:** 2025-10-12  
**Status:** Draft - Ready for Review  
**Owner:** Product Team  
**Stakeholders:** Engineering, UX Design, Legal/Compliance, Customer Success
