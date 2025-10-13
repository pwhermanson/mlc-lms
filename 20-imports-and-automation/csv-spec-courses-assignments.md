# CSV Spec Courses and Assignments

This document defines the CSV file format specifications for importing course sequences, assignment groups, and assignment steps into the MLC-LMS platform. It enables system administrators and content managers to bulk-load curriculum structures, learning pathways, and assignment configurations.

## Purpose

This specification enables:

- **Curriculum Import**: Import complete sequences (LIFE, SOLF, EVAL) with all assignments and steps from legacy MLC 4.0 system or external curriculum sources
- **Assignment Structure Definition**: Define assignment groups, steps, gating rules, and target scores in bulk
- **Content Migration**: Support migration from MLC 4.0 to MLC 5.0 with preserved sequence structures and IDs
- **Method Alignment**: Import method-aligned sequences correlated to publisher curricula (e.g., Hal Leonard Student Piano Library, Alfred's Premier Piano Course)
- **Batch Content Updates**: Enable content teams to update assignment metadata, targets, and ordering without database access
- **Standardized Format**: Ensure consistent sequence structure across all content sources

This specification is narrowly focused on the CSV file format for defining curriculum structure. For student assignment of courses, see [Student Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md) and [CSV Spec Students](./csv-spec-students.md).

## Scope

**Included:**
- CSV structure for sequence definitions, assignment groups, assignments, and steps
- Course/assignment ID mapping compatible with legacy MLC 4.0 system (IDs 657+)
- Element type codes (GAM, VID, AUD, TXT, RWD) and stage suffixes (LEARN, PLAY, QUIZ, CHALLENGE, REVIEW)
- Gating rules, target scores, and prerequisite definitions
- Sequence metadata (categories, tags, descriptions, ordering)
- Multi-step assignment structures with mixed content types
- Validation rules for course import data
- Example CSV templates for common import scenarios

**Excluded:**
- Student progress data and assignment completion records (covered in [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- Game content and asset files (covered in [Games Importer](../08-games-registry-and-authoring/games-importer.md))
- Real-time assignment generation logic (covered in [Assignment Generation](../10-assignments-engine/assignment-generation.md))
- Teacher-specific assignment customizations (covered in [Assignment Builder](../04-teacher-experience/assignment-builder.md))
- Student enrollment in courses (covered in [CSV Spec Students](./csv-spec-students.md))

## Data model (if applicable)

### CSV Structure Overview

The courses and assignments CSV import supports **two CSV formats**:

1. **Assignment Groups CSV**: Defines the high-level course structure and assignment groups within sequences
2. **Assignment Steps CSV**: Defines the detailed step-by-step content for each assignment

These two CSVs work together: Groups CSV establishes the hierarchy, Steps CSV populates the content.

### Format 1: Assignment Groups CSV

Defines assignment groups within a sequence (e.g., "Primary Level 1A", "Level 1").

#### Column Definitions

| Column Name | Data Type | Max Length | Required | Description | Example |
|-------------|-----------|------------|----------|-------------|---------|
| `sequence_code` | String | 10 | ✅ Yes | Sequence identifier (LIFE, SOLF, EVAL, or custom) | `LIFE` |
| `group_id` | String | 10 | ✅ Yes | Unique group identifier within sequence | `005A` |
| `level_title` | String | 100 | ✅ Yes | Human-readable level name | `Primary Level 1A` |
| `unit_title` | String | 100 | ✅ Yes | Assignment name within level | `Assignment 1` |
| `assignment_number` | Integer | 5 | No | Legacy assignment ID (for MLC 4.0 compatibility) | `657` |
| `description` | String | 500 | No | Optional description of assignment content | `Introduction to staff guide notes` |
| `estimated_minutes` | Integer | 5 | No | Expected completion time | `20` |
| `concepts_covered` | String | 200 | No | Comma-separated concept tags | `Pitch & Melody,High vs Low` |
| `active_status` | String | 1 | No | `A` for active, `X` for development, blank for inactive | `A` |

**Notes:**
- `group_id` format follows legacy MLC 4.0 pattern: 3-digit number + letter suffix (e.g., `005A`, `010A`)
- `sequence_code` must match an existing sequence or be a new custom sequence identifier
- `assignment_number` maps to legacy MLC 4.0 course assignment IDs (657-935 range documented in source data)

### Format 2: Assignment Steps CSV

Defines the detailed learning elements (games, videos, etc.) within each assignment.

#### Column Definitions

| Column Name | Data Type | Max Length | Required | Description | Example |
|-------------|-----------|------------|----------|-------------|---------|
| `sequence_code` | String | 10 | ✅ Yes | Must match sequence from Groups CSV | `LIFE` |
| `group_id` | String | 10 | ✅ Yes | Must match group_id from Groups CSV | `005A` |
| `seq_order` | Integer | 8 | ✅ Yes | Step ordering within assignment (e.g., 100, 150, 200) | `150` |
| `element_type` | String | 4 | ✅ Yes | Content type: `GAM`, `VID`, `AUD`, `TXT`, `RWD` | `GAM` |
| `element_id` | String | 20 | ✅ Yes | Element reference with stage suffix (e.g., `3480-1`) | `3480-1` |
| `stage` | String | 10 | Conditional | Required for GAM type: `LEARN`, `PLAY`, `QUIZ`, `CHALLENGE`, `REVIEW` | `LEARN` |
| `element_name` | String | 200 | ✅ Yes | Human-readable name of learning element | `Songbirds High and Low` |
| `element_description` | String | 500 | No | Description of learning objective | `Identify high sounds and low sounds` |
| `target_score` | Integer | 3 | No | Target score for mastery (typically 70-90) | `80` |
| `pass_threshold` | Integer | 3 | No | Minimum passing score (typically 60-80) | `70` |
| `require_previous` | Boolean | 1 | No | `Y` if previous steps must be completed; blank or `N` otherwise | `Y` |
| `min_attempts` | Integer | 2 | No | Minimum attempts required (typically 1) | `1` |
| `optional` | Boolean | 1 | No | `Y` if step is optional; blank or `N` if required | `N` |
| `keyboard_required` | Boolean | 1 | No | `K` or `Y` if MIDI keyboard required; blank otherwise | `K` |
| `active_status` | String | 1 | No | `A` for active, `X` for development, blank for inactive | `A` |
| `video_url` | String | 500 | No | Vimeo or YouTube URL for video elements | `https://vimeo.com/161290191` |
| `pdf_filename` | String | 200 | No | Filename for PDF document elements | `LifetimeAssignmentSheet.pdf` |
| `category` | String | 100 | No | Concept category (e.g., `Pitch & Melody`) | `Music Terms & Symbols` |
| `tags` | String | 200 | No | Comma-separated tags for filtering | `Pre-reading,High vs Low` |

**Notes:**
- `element_type` + `element_id` + `stage` uniquely identify the learning element
- Element ID format: `{BaseGameNumber}-{StageSuffix}` where suffix is `1`=LEARN, `2`=PLAY, `3`=QUIZ, `4`=CHALLENGE, `5`=REVIEW
- `seq_order` uses increments of 50 or 100 to allow insertion of steps without renumbering (legacy MLC 4.0 pattern)
- For non-game elements (VID, AUD, TXT, RWD), `stage` field should be empty or set to `INS` (instructional)

### Element Type Codes

| Code | Type | Description | Stage Support | Example |
|------|------|-------------|---------------|---------|
| `GAM` | Game | Interactive learning game with scoring | LEARN, PLAY, QUIZ, CHALLENGE, REVIEW | Songbirds High and Low |
| `VID` | Video | Instructional or demonstration video | INS (instructional) | How to Use Sequence |
| `AUD` | Audio | Audio clip or music example | INS | Rhythm Pattern Example |
| `TXT` | Text | Text-based instruction or explanation | INS | Practice Checklist |
| `RWD` | Reward | Badge, certificate, or other reward | N/A | Completion Badge |

### Stage Codes and Suffixes

| Stage | Suffix | Purpose | Typical Gating | Target Score Range |
|-------|--------|---------|----------------|-------------------|
| `LEARN` | `-1` | Introduce concept without pressure | None (always available) | N/A or 50-60 |
| `PLAY` | `-2` | Practice and reinforce concept | Requires LEARN attempt | 60-75 |
| `QUIZ` | `-3` | Demonstrate mastery | Requires LEARN and PLAY | 75-90 |
| `CHALLENGE` | `-4` | Advanced optional challenge | Requires QUIZ passed | 85-95 |
| `REVIEW` | `-5` | Spaced repetition for retention | Requires QUIZ passed, scheduled 7+ days later | 75-90 |

### Sequence Codes

| Code | Name | Type | Description |
|------|------|------|-------------|
| `LIFE` | Lifetime Musician | First-party | Default comprehensive music curriculum |
| `SOLF` | Solfege | First-party | Solfege-focused curriculum variant |
| `EVAL` | Evaluation | First-party | Placement and assessment sequence |
| `ALFR` | Alfred's Premier Piano Course | Method-aligned | Correlated to Alfred's method books |
| `HLSL` | Hal Leonard Student Piano Library | Method-aligned | Correlated to Hal Leonard books |
| Custom | (Organization-defined) | Custom | Teacher or organization-created sequences |

### CSV File Format Requirements

#### Technical Specifications

| Property | Requirement | Notes |
|----------|-------------|-------|
| **File Extension** | `.csv` | Text files with comma-separated values |
| **Character Encoding** | `UTF-8` (with or without BOM) | Supports international characters |
| **Line Endings** | `CRLF` (Windows) or `LF` (Unix/Mac) | Either format accepted |
| **Delimiter** | Comma (`,`) | Field separator |
| **Text Qualifier** | Double quote (`"`) | Used to escape commas within fields |
| **Header Row** | **Required** (first row) | Column names, case-insensitive |
| **Maximum File Size** | 25 MB | Approximately 100,000 steps |
| **Maximum Rows** | 100,000 rows | Excluding header row |

### Example CSV Templates

#### Example 1: Assignment Groups CSV (Minimal)

```csv
sequence_code,group_id,level_title,unit_title,active_status
LIFE,004A,Introduction,How to Use Assignments,X
LIFE,005A,Primary Level 1A,Assignment 1,A
LIFE,010A,Primary Level 1A,Assignment 2,A
LIFE,015A,Primary Level 1A,Assignment 3,A
```

#### Example 2: Assignment Steps CSV (Complete Assignment)

```csv
sequence_code,group_id,seq_order,element_type,element_id,stage,element_name,element_description,target_score,pass_threshold,require_previous,active_status,category,tags
LIFE,005A,100,VID,2005-2,INS,How to Use Sequence,How to navigate sequence assignment,,,N,X,,
LIFE,005A,150,GAM,3480-1,LEARN,Songbirds High and Low,Identify high sounds and low sounds,,,N,A,Pitch & Melody,"Pre-reading,High vs Low"
LIFE,005A,200,GAM,3480-2,PLAY,Songbirds High and Low,Identify high sounds and low sounds,70,60,N,A,Pitch & Melody,"Pre-reading,High vs Low"
LIFE,005A,250,GAM,3480-3,QUIZ,Songbirds High and Low,Identify high sounds and low sounds,85,80,Y,A,Pitch & Melody,"Pre-reading,High vs Low"
LIFE,005A,300,GAM,3720-1,LEARN,Storm Chasers 1,Aurally identify Up and Down,,,N,A,Pitch & Melody,Melodic Direction
LIFE,005A,350,GAM,3720-2,PLAY,Storm Chasers 1,Aurally identify Up and Down,70,60,N,A,Pitch & Melody,Melodic Direction
LIFE,005A,400,GAM,3720-3,QUIZ,Storm Chasers 1,Aurally identify Up and Down,85,80,Y,A,Pitch & Melody,Melodic Direction
```

#### Example 3: Assignment Steps CSV (With MIDI Games)

```csv
sequence_code,group_id,seq_order,element_type,element_id,stage,element_name,element_description,target_score,pass_threshold,keyboard_required,active_status
LIFE,005A,750,GAM,3850-1,LEARN,Tommy Tiger's 2's & 3's,Identify the groups of 2 and 3 black keys on the keyboard,,,K,A
LIFE,005A,800,GAM,3850-2,PLAY,Tommy Tiger's 2's & 3's,Identify the groups of 2 and 3 black keys on the keyboard,70,60,K,A
LIFE,005A,850,GAM,3850-3,QUIZ,Tommy Tiger's 2's & 3's,Identify the groups of 2 and 3 black keys on the keyboard,85,80,K,A
```

#### Example 4: Method-Aligned Assignment Steps CSV

```csv
sequence_code,group_id,seq_order,element_type,element_id,stage,element_name,element_description,target_score,pass_threshold,active_status,category,tags,book_pages,book_unit
HLSL,10A,100,GAM,3480-1,LEARN,Songbirds High and Low,Identify high sounds and low sounds,,,A,Pitch & Melody,"Pre-reading,High vs Low","5-7",Unit 1
HLSL,10A,150,GAM,3480-2,PLAY,Songbirds High and Low,Identify high sounds and low sounds,70,60,A,Pitch & Melody,"Pre-reading,High vs Low","5-7",Unit 1
HLSL,10A,200,GAM,3480-3,QUIZ,Songbirds High and Low,Identify high sounds and low sounds,85,80,A,Pitch & Melody,"Pre-reading,High vs Low","5-7",Unit 1
```

## Behavior and rules

### Validation Rules

#### File-Level Validation (Pre-Processing)

| Rule | Description | Error Code |
|------|-------------|------------|
| **File Format** | Must be valid CSV with `.csv` extension | `ERR_INVALID_FILE_FORMAT` |
| **File Size** | Must not exceed 25 MB | `ERR_FILE_TOO_LARGE` |
| **Row Limit** | Must not exceed 100,000 rows (excluding header) | `ERR_TOO_MANY_ROWS` |
| **Header Row** | First row must contain valid column headers | `ERR_MISSING_HEADER` |
| **Required Columns** | Must include all required columns per format | `ERR_MISSING_REQUIRED_COLUMN` |

#### Row-Level Validation (Assignment Groups CSV)

| Field | Rule | Error Code |
|-------|------|------------|
| `sequence_code` | Must not be empty, 2-10 alphanumeric characters | `ERR_SEQUENCE_CODE_INVALID` |
| `group_id` | Must not be empty, unique within sequence | `ERR_GROUP_ID_REQUIRED` |
| `group_id` | Must be 4-10 characters | `ERR_GROUP_ID_INVALID_LENGTH` |
| `level_title` | Must not be empty, max 100 characters | `ERR_LEVEL_TITLE_REQUIRED` |
| `unit_title` | Must not be empty, max 100 characters | `ERR_UNIT_TITLE_REQUIRED` |
| `assignment_number` | If provided, must be positive integer | `ERR_ASSIGNMENT_NUMBER_INVALID` |
| `active_status` | If provided, must be `A`, `X`, or blank | `ERR_ACTIVE_STATUS_INVALID` |

#### Row-Level Validation (Assignment Steps CSV)

| Field | Rule | Error Code |
|-------|------|------------|
| `sequence_code` | Must match existing sequence from Groups CSV | `ERR_SEQUENCE_NOT_FOUND` |
| `group_id` | Must match existing group from Groups CSV | `ERR_GROUP_NOT_FOUND` |
| `seq_order` | Must be positive integer | `ERR_SEQ_ORDER_INVALID` |
| `seq_order` | Must be unique within group | `ERR_SEQ_ORDER_DUPLICATE` |
| `element_type` | Must be `GAM`, `VID`, `AUD`, `TXT`, or `RWD` | `ERR_ELEMENT_TYPE_INVALID` |
| `element_id` | Must not be empty, max 20 characters | `ERR_ELEMENT_ID_REQUIRED` |
| `element_id` | If element_type is `GAM`, must exist in games registry | `ERR_GAME_NOT_FOUND` |
| `stage` | If element_type is `GAM`, must be `LEARN`, `PLAY`, `QUIZ`, `CHALLENGE`, or `REVIEW` | `ERR_STAGE_REQUIRED` |
| `element_name` | Must not be empty, max 200 characters | `ERR_ELEMENT_NAME_REQUIRED` |
| `target_score` | If provided, must be 0-100 | `ERR_TARGET_SCORE_OUT_OF_RANGE` |
| `pass_threshold` | If provided, must be 0-100 and ≤ target_score | `ERR_PASS_THRESHOLD_INVALID` |
| `min_attempts` | If provided, must be 1-99 | `ERR_MIN_ATTEMPTS_INVALID` |

### Import Processing Modes

#### 1. Validation-Only Mode (Dry Run)

- Validates both Groups and Steps CSVs without creating any content
- Returns detailed validation report with errors and warnings
- Checks for missing game references and broken dependencies
- Recommended first step before committing import

#### 2. Import Mode (Create New)

- Creates new sequences, groups, and assignments from CSV data
- If sequence already exists, import fails with `ERR_SEQUENCE_EXISTS`
- All valid rows are processed; invalid rows are skipped with errors
- Generates import batch record for tracking

#### 3. Update Mode (Merge)

- Updates existing sequences with new or modified assignments
- Matching group_id rows are updated; new group_ids are inserted
- Existing student assignments are **not** affected (version pinning)
- Increments `sequence_version` if breaking changes detected

### Sequence Versioning Rules

**Breaking Changes** (increment `sequence_version`):
- Removing required steps
- Changing step order significantly (>50% reordered)
- Modifying game IDs for existing steps
- Changing pass thresholds that affect existing student progress

**Non-Breaking Changes** (no version increment):
- Adding new optional steps
- Updating descriptions or metadata
- Adding new assignments to end of sequence
- Adjusting target scores (as long as pass_threshold unchanged)

**Version Pinning Behavior:**
- Student assignments pin to `sequence_version` at creation time
- Importing updates creates new version; existing assignments unaffected
- Teachers can migrate student assignments to new version via UI

### Gating and Prerequisite Rules

**Default Gating:**
- `LEARN` stages: Always available, no prerequisites
- `PLAY` stages: Requires ≥1 attempt on corresponding `LEARN` stage
- `QUIZ` stages: Requires ≥1 attempt on `LEARN` and `PLAY` stages
- `CHALLENGE` stages: Requires `QUIZ` passed with ≥ pass_threshold
- `REVIEW` stages: Requires `QUIZ` passed, scheduled 7+ days later

**Custom Gating via CSV:**
- Set `require_previous = Y` to enforce sequential completion
- Set `min_attempts = N` to require N attempts before proceeding
- Leave blank to use default gating rules per stage type

### Element ID and Stage Mapping

**Legacy MLC 4.0 Format:**
- Element IDs follow pattern: `{BaseGameID}-{StageSuffix}`
- Example: Game 3480 with stages → `3480-1` (LEARN), `3480-2` (PLAY), `3480-3` (QUIZ), `3480-4` (CHALLENGE)
- When importing legacy data, system maps stage suffix to stage name automatically

**Modern MLC 5.0 Format:**
- Element IDs use canonical game registry IDs: `G-{5-digit-number}`
- Stage specified separately in `stage` column
- System supports both formats for backward compatibility

### Missing or Deprecated Content Handling

**During Import:**
- If referenced game (`element_id`) does not exist in games registry → **Warning** (`WARN_GAME_NOT_FOUND`)
- Import continues but flags step with content warning
- Step is marked as `needs_content_review` in database

**During Runtime:**
- If game is deprecated after import, assignment generation flags issue
- Teachers see "Content Unavailable" indicator in assignment builder
- Students see "Coming Soon" placeholder if assigned deprecated content

### CSV Column Order Flexibility

- Column order in CSV is **flexible** (columns can appear in any order)
- System matches columns by header name, not position
- Extra columns not defined in spec are ignored (allows for custom metadata)
- Missing required columns trigger `ERR_MISSING_REQUIRED_COLUMN` error

## UX requirements (if applicable)

### CSV Upload Interface

While detailed UI workflow is specified in [Bulk Ops Jobs](./bulk-ops-jobs.md), these UX requirements relate specifically to course CSV handling:

**File Upload Component:**
- Accept `.csv` files for both Groups and Steps formats
- Provide separate upload zones or sequential upload workflow:
  1. Upload Groups CSV first
  2. Validate and preview group structure
  3. Upload Steps CSV second
  4. Validate steps against groups
- Display file name, size, and row count after upload
- Show validation summary: groups loaded, steps loaded, errors/warnings count

**Validation Feedback:**
- Show file-level errors immediately (before processing)
- Display row-level errors in sortable/filterable table:
  - Row number
  - Field name
  - Error code and message
  - Suggested fix
- Highlight warnings (e.g., game not found) separately from errors
- Provide "Download Error Report" CSV with failed rows for correction

**Preview and Confirmation:**
- Show tree view of sequence structure before committing:
  - Sequence → Groups → Assignments → Steps
- Display summary counts: X sequences, Y groups, Z assignments, W steps
- Show version increment indicator if updating existing sequence
- Confirm action button: "Import N new assignments" or "Update sequence (version X → Y)"

**Success and Results:**
- Display import summary: rows processed, created, skipped, failed
- Provide link to view imported sequences in content management UI
- Show list of warnings that require review (missing games, etc.)

### Accessibility Notes

- File upload must be keyboard accessible
- Validation error table must use proper ARIA labels and roles
- Tree view preview must be navigable via keyboard (arrow keys)
- Screen readers must announce import progress and results

See [A11y Standards](../01-ux-design-system/a11y-standards.md) for comprehensive accessibility requirements.

## Telemetry

All course/assignment import events follow the standard event model defined in [Event Model](../15-analytics-and-reporting/event-model.md).

### Events

#### `course_csv_import_started`
**Fires when:** System admin or content manager initiates course CSV import

**Properties:**
```json
{
  "event_type": "course_csv_import_started",
  "user_id": "usr_sysadmin_001",
  "import_batch_id": "batch_course_xyz789",
  "import_mode": "create_new | update | validation_only",
  "file_count": 2,
  "files": [
    {"type": "groups", "file_name": "life_groups.csv", "row_count": 50},
    {"type": "steps", "file_name": "life_steps.csv", "row_count": 1200}
  ],
  "uploaded_via": "web_ui | api",
  "timestamp": "2025-10-12T10:00:00Z"
}
```

#### `course_csv_import_validated`
**Fires when:** CSV validation completes (before import)

**Properties:**
```json
{
  "event_type": "course_csv_import_validated",
  "user_id": "usr_sysadmin_001",
  "import_batch_id": "batch_course_xyz789",
  "validation_result": "passed | failed | passed_with_warnings",
  "groups_validated": 50,
  "groups_valid": 48,
  "groups_invalid": 2,
  "steps_validated": 1200,
  "steps_valid": 1180,
  "steps_invalid": 20,
  "error_codes": ["ERR_GAME_NOT_FOUND", "ERR_GROUP_NOT_FOUND"],
  "error_code_counts": {
    "ERR_GAME_NOT_FOUND": 15,
    "ERR_GROUP_NOT_FOUND": 5
  },
  "validation_duration_ms": 5000,
  "timestamp": "2025-10-12T10:00:10Z"
}
```

#### `course_csv_import_completed`
**Fires when:** CSV import processing finishes

**Properties:**
```json
{
  "event_type": "course_csv_import_completed",
  "user_id": "usr_sysadmin_001",
  "import_batch_id": "batch_course_xyz789",
  "status": "completed | partially_completed | failed",
  "sequences_created": 1,
  "sequences_updated": 0,
  "groups_created": 48,
  "groups_updated": 0,
  "steps_created": 1180,
  "steps_failed": 20,
  "version_incremented": true,
  "new_sequence_version": 8,
  "processing_duration_ms": 25000,
  "timestamp": "2025-10-12T10:00:35Z"
}
```

#### `course_csv_import_failed`
**Fires when:** CSV import encounters critical error

**Properties:**
```json
{
  "event_type": "course_csv_import_failed",
  "user_id": "usr_sysadmin_001",
  "import_batch_id": "batch_course_xyz789",
  "failure_reason": "file_too_large | invalid_format | system_error",
  "error_code": "ERR_FILE_TOO_LARGE",
  "error_message": "Steps CSV exceeds 25 MB limit",
  "timestamp": "2025-10-12T10:00:05Z"
}
```

### Analytics and Monitoring

**Key Metrics to Track:**
- Import success rate by mode (create vs update)
- Average validation error count per import
- Most common error codes
- Processing time by row count (performance benchmarks)
- Game reference failures (indicates content gaps)
- Sequence version increment frequency

**Operational Alerts:**
- Import failure rate > 10% in last 7 days
- Average processing time > 60 seconds for <5000 rows
- Game reference errors > 5% of steps imported
- Critical games referenced in imports but marked deprecated

See [Observability](../19-quality-and-operations/observability.md) for monitoring infrastructure.

## Permissions

Course and assignment CSV import operations require elevated permissions limited to content management roles.

### Role-Based Access

| Role | Can Upload CSV | Can Validate | Can Import (Create New) | Can Update Existing | Scope |
|------|---------------|--------------|------------------------|---------------------|-------|
| **System-Admin** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | All sequences (first-party, method-aligned, custom) |
| **Content-Editor** | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes | First-party sequences only (LIFE, SOLF, EVAL) |
| **Subscriber-Admin** | ⚠️ Custom Only | ⚠️ Custom Only | ⚠️ Custom Only | ⚠️ Custom Only | Organization-scoped custom sequences only |
| **Teacher-Admin** | ❌ No | ❌ No | ❌ No | ❌ No | Cannot import sequences (can only assign existing) |
| **Teacher** | ❌ No | ❌ No | ❌ No | ❌ No | Cannot import sequences |
| **Student** | ❌ No | ❌ No | ❌ No | ❌ No | Cannot import sequences |

**Permission Codes:**
- `SQI` (Sequence Import): Required for CSV import operations
- `CTM` (Content Management): Required to create/update first-party sequences
- `CSQ` (Custom Sequence): Required to create organization-scoped sequences

See [Permissions Table](../02-roles-and-permissions/permissions-table.md) for comprehensive permission matrix.

### Audit Logging

All course import operations are logged for compliance and troubleshooting:

- **Who**: User ID and role of importer
- **What**: Files imported, sequences affected, changes made
- **When**: Timestamp of operation
- **Result**: Success/failure, error details
- **Impact**: Number of assignments affected, version changes

See [Audit Logs](../06-system-admin/sysadmin-audit-logs.md) for detailed logging requirements.

### Content Approval Workflow

**First-Party Sequences (LIFE, SOLF, EVAL):**
- Require content review and approval before publishing
- Import creates draft version, not immediately visible to teachers
- Content-Editor or System-Admin must explicitly publish to make live
- Publishing increments `sequence_version` and notifies affected teachers

**Method-Aligned Sequences:**
- Require publisher approval and licensing verification
- Import validates against publisher metadata standards
- System-Admin must confirm licensing before enabling

**Custom Sequences:**
- No approval required for organization-scoped sequences
- Immediately available to teachers within organization
- Not visible to other organizations

## Supporting Documents Referenced

This CSV specification draws from the following source documents:

- [MLC SeqCreate 2020.xlsx - ALPHA LIST .csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20ALPHA%20LIST%20.csv) — Legacy sequence alpha list informing assignment group IDs, level titles, and sequence organization structure
- [MLC SeqCreate 2020.xlsx - LIFE SEQ.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20LIFE%20SEQ.csv) — Legacy LIFE sequence specifications informing assignment step ordering, element types, and gating rules
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Legacy sequence specifications providing detailed game-stage mappings and target score configurations
- [MLC Sequences (1).xlsx - ALPHA LIST .csv](../00-ORIG-CONTEXT/MLC%20Sequences%20(1).xlsx%20-%20ALPHA%20LIST%20.csv) — Additional sequence list informing assignment numbering and curriculum structure hierarchy
- [MLC Sequences (1).xlsx - LIFE.csv](../00-ORIG-CONTEXT/MLC%20Sequences%20(1).xlsx%20-%20LIFE.csv) — Alternative LIFE sequence format providing cross-reference for assignment step validation
- [MLC Sequences (1).xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20Sequences%20(1).xlsx%20-%20specs.csv) — Additional specifications for element configuration and learning objectives
- [AlphaMasterList2020.xlsx - Master.csv](../00-ORIG-CONTEXT/AlphaMasterList2020.xlsx%20-%20Master.csv) — Master sequence list informing assignment ID ranges (657-935) and assignment-to-level mappings
- [Course import.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Course%20import.xlsx%20-%20Sheet1.csv) — Legacy course import format informing CSV structure and column naming conventions
- [Couurse ID numbers.xlsx - Sheet 1 - tbl_contents.csv](../00-ORIG-CONTEXT/Couurse%20ID%20numbers.xlsx%20-%20Sheet%201%20-%20tbl_contents.csv) — Legacy course ID numbering scheme informing assignment number validation and backward compatibility
- [Level1CurriculumMap.xls - Sheet1.csv](../00-ORIG-CONTEXT/Level1CurriculumMap.xls%20-%20Sheet1.csv) — Level 1 curriculum mapping informing concept tag hierarchies and pedagogical progression

---

## Dependencies

This specification relies on the following systems and specifications:

### Internal Dependencies

| Dependency | Purpose | Link |
|------------|---------|------|
| **Sequence Model** | Defines sequence, group, assignment, and step data structure | [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) |
| **Assignment Model** | Defines assignment lifecycle, gating rules, and state transitions | [Assignment Model](../07-content-architecture/assignment-model.md) |
| **Game Model** | Defines game stages, scoring, and valid game IDs for reference validation | [Game Model](../08-games-registry-and-authoring/game-model.md) |
| **Games Registry** | Source of truth for valid game IDs and availability status | [Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md) |
| **Assignment Generation** | Consumes imported sequences to generate student assignments | [Assignment Generation](../10-assignments-engine/assignment-generation.md) |
| **Bulk Ops Jobs** | Background job processing system for handling large CSV imports | [Bulk Ops Jobs](./bulk-ops-jobs.md) |
| **Permissions Table** | Defines role-based access control for import operations | [Permissions Table](../02-roles-and-permissions/permissions-table.md) |
| **Event Model** | Defines telemetry event structure for tracking imports | [Event Model](../15-analytics-and-reporting/event-model.md) |
| **Audit Logs** | Records all content import operations for compliance | [Audit Logs](../06-system-admin/sysadmin-audit-logs.md) |

### External Dependencies

| Dependency | Purpose | Notes |
|------------|---------|-------|
| **CSV Parser Library** | Parse and validate CSV file format | Use battle-tested library (e.g., `csv-parse` for Node.js, `csv` for Python) |
| **Games Database** | Validate game IDs exist and are active | Query games registry before import commit |
| **Background Job Queue** | Process large imports asynchronously | Use Redis Queue, Sidekiq, or equivalent |
| **Version Control System** | Track sequence version history | Store sequence diffs and version metadata |

### Legacy System Compatibility

**MLC 4.0 Sequence Import:**
- This specification maintains backward compatibility with MLC 4.0 CSV exports
- Column name aliases supported:
  - `sequence_code` ← `Sequence`, `Code`
  - `group_id` ← `Group`, `Group Code`
  - `element_type` ← `Type`, `Element Type`
  - `element_id` ← `#`, `Element #`, `Game (Element) #`
  - `stage` ← `Stage` (with suffix mapping)
- Legacy assignment IDs (657-935) preserved during migration
- Element ID format `{BaseGameID}-{StageSuffix}` automatically converted to modern format

See [Legacy Systems](../00-foundations/legacy-systems.md) for migration strategies.

## Open questions

### Technical Decisions Needed

1. **CSV Parser Library Selection:**
   - Which specific CSV parsing library for backend?
   - Recommendation: `csv-parse` (Node.js) or `pandas` (Python) for robustness
   - **Owner:** Backend tech lead
   - **Due:** Phase 1 technical design sprint

2. **Sequence Version Diff Algorithm:**
   - How to automatically detect breaking vs non-breaking changes?
   - Current spec: Manual rules based on change type
   - Future: Automated diff analysis with impact prediction
   - **Owner:** Backend tech lead, Content team
   - **Due:** Phase 1 technical design sprint

3. **Missing Game Reference Handling:**
   - Should import fail or continue with warnings when games not found?
   - Current spec: Continue with warnings, flag for review
   - Alternative: Fail entire import if >5% game references invalid
   - **Owner:** Product manager, Content team
   - **Due:** Phase 1 requirements review

4. **Asynchronous Processing Threshold:**
   - At what row count should processing move to background jobs?
   - Recommendation: Synchronous for <500 steps, asynchronous for ≥500 steps
   - **Owner:** Backend tech lead
   - **Due:** Phase 1 technical design sprint

5. **Sequence Complexity Limits:**
   - Maximum groups, assignments, steps per sequence?
   - Performance impact of very large sequences (>10,000 steps)?
   - **Owner:** Backend tech lead, Performance engineer
   - **Due:** Phase 1 performance testing

### UX and Product Questions

6. **Two-File Upload UX:**
   - Should Groups and Steps be uploaded separately or combined in single CSV?
   - Current spec: Separate files for clarity and validation
   - Alternative: Single combined CSV with inline group definitions
   - **Owner:** Product designer, Content team
   - **Due:** Phase 1 UX design sprint

7. **Preview and Confirmation UI:**
   - How detailed should sequence preview be before import commit?
   - Tree view vs tabular list vs summary counts only?
   - **Owner:** Product designer
   - **Due:** Phase 1 UX design sprint

8. **Error Report Format:**
   - Should error reports be CSV, Excel, PDF, or all three?
   - Current spec: CSV format for easy correction and re-import
   - **Owner:** Product designer
   - **Due:** Phase 1 UX design sprint

### Business and Content Questions

9. **Method-Aligned Sequence Licensing:**
   - How to verify publisher licensing before enabling method-aligned imports?
   - Legal agreements needed with Alfred, Hal Leonard, Faber, etc.?
   - **Owner:** Legal counsel, Partnerships lead
   - **Due:** Before Phase 1 launch

10. **Custom Sequence Governance:**
    - Should organizations be limited in number/size of custom sequences?
    - Potential for abuse (creating thousands of sequences)?
    - **Owner:** Product manager, Customer success
    - **Due:** Phase 1 planning

11. **Content Approval Workflow:**
    - Formal review process for first-party sequence updates?
    - Who approves: Content team, pedagogical review board, both?
    - **Owner:** Content team lead, Product manager
    - **Due:** Phase 1 planning

### Integration Questions

12. **Automated Content Pipelines:**
    - Should CSV import support scheduled/automated imports from external sources?
    - Use case: Nightly sync from publisher content management system
    - **Owner:** Integrations architect, Product manager
    - **Due:** Phase 2 planning

13. **Version Migration API:**
    - Should system provide API for programmatic version migration?
    - Use case: Bulk migrate all students to new sequence version
    - **Owner:** API architect, Product manager
    - **Due:** Phase 2 planning

---

**Last Updated:** 2025-10-12  
**Status:** Draft - Ready for Review  
**Owner:** Product Team, Content Team  
**Stakeholders:** Engineering, UX Design, Content Management, System Administrators
