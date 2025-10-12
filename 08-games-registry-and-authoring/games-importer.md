# Games Importer

## Purpose

This specification defines the bulk import system for games, enabling System Administrators and Content Editors to efficiently create or update multiple game records simultaneously via CSV upload. The importer provides a controlled, validated pathway for migrating legacy game data, onboarding new game collections, and performing mass updates to metadata across the game library.

> **Related Documents**: For individual game CRUD operations, see [Games CRUD](games-crud.md). For metadata field definitions and validation rules, see [Games Metadata](games-metadata.md). For asset upload workflows, see [Games Assets Pipeline](games-assets-pipeline.md). For CSV field specifications, see [CSV Spec Games](../20-imports-and-automation/csv-spec-games.md).

The games importer enables:
- **Efficient migration**: Transfer hundreds of legacy games from MLC 4.0 to MLC 5.0 with minimal manual data entry
- **Batch operations**: Create, update, or deprecate multiple games in a single validated transaction
- **Quality assurance**: Preview imports before commit, catch validation errors early, and provide actionable feedback
- **Data integrity**: Maintain referential integrity, enforce business rules, and prevent duplicate or conflicting records
- **Audit trail**: Track who imported what data when, with ability to rollback failed imports
- **Conflict resolution**: Detect and resolve duplicate game IDs, names, slugs, and legacy references during import
- **Metadata completeness**: Validate required fields, concept tags, standards alignment, and asset references in bulk

Without a structured import system, migrating 1,000+ games from legacy systems would require error-prone manual entry, risking inconsistent metadata, broken references, and inability to validate data quality at scale.

## Scope

**Included**
- CSV file format specification with required and optional columns
- Upload workflow for System Administrators and Content Editors
- Validation pipeline with field-level and cross-record checks
- Preview mode showing proposed changes before commit
- Error reporting with actionable feedback for each invalid row
- Conflict detection and resolution strategies (create vs. update, duplicate handling)
- Batch transaction boundaries and rollback on partial failure
- Import job status tracking and progress reporting
- Metadata mapping from legacy game IDs to new system structure
- Stage auto-generation based on imported game type and level
- Asset reference validation against CDN and storage buckets
- Dry-run mode for testing imports without database writes
- Import history and audit logging
- Permissions and access control for import operations

**Excluded**
- Individual game CRUD operations (see [Games CRUD](games-crud.md))
- Metadata field definitions and validation rules (see [Games Metadata](games-metadata.md))
- Asset file upload and processing pipeline (see [Games Assets Pipeline](games-assets-pipeline.md))
- Sequence and assignment import workflows (covered separately in [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))
- Student or teacher bulk import (see [CSV Spec Students](../20-imports-and-automation/csv-spec-students.md) and [CSV Spec Teachers](../20-imports-and-automation/csv-spec-teachers.md))
- Export functionality (see [Export Specs](../20-imports-and-automation/export-specs.md))

## Data model (if applicable)

### Import Job Entity

Each import operation creates an Import Job record to track progress and results:

```json
{
  "import_job_id": "IMP-20250112-00042",
  "job_type": "games_create",
  "status": "completed",
  "uploaded_by": "admin@mlc.com",
  "uploaded_at": "2025-01-12T14:32:15Z",
  "started_at": "2025-01-12T14:32:30Z",
  "completed_at": "2025-01-12T14:45:12Z",
  "total_rows": 150,
  "processed_rows": 150,
  "successful_rows": 147,
  "failed_rows": 3,
  "validation_errors": [
    {
      "row": 23,
      "field": "slug",
      "error": "Duplicate slug 'grand-staff-guide-notes-1' found in row 18",
      "severity": "error"
    },
    {
      "row": 67,
      "field": "concept_tags",
      "error": "Invalid concept tag 'rythm' - did you mean 'rhythm'?",
      "severity": "error"
    },
    {
      "row": 102,
      "field": "thumbnail_url",
      "error": "Asset not found at specified URL",
      "severity": "warning"
    }
  ],
  "file_url": "s3://mlc-imports/games/import-20250112-00042.csv",
  "preview_url": "s3://mlc-imports/games/preview-20250112-00042.json",
  "rollback_available": true,
  "dry_run": false
}
```

**Field Definitions**:
- `import_job_id`: Unique identifier for this import operation, format `IMP-YYYYMMDD-NNNNN`
- `job_type`: Type of import - `games_create`, `games_update`, `games_upsert`, `games_deprecate`
- `status`: Current state - `queued`, `validating`, `previewing`, `processing`, `completed`, `failed`, `rolled_back`
- `uploaded_by`: Email or user ID of administrator who initiated the import
- `total_rows`: Total number of data rows in CSV (excluding header)
- `processed_rows`: Number of rows processed so far (for progress tracking)
- `successful_rows`: Number of rows successfully imported
- `failed_rows`: Number of rows with validation or processing errors
- `validation_errors`: Array of error objects with row, field, error message, and severity
- `file_url`: Location of original uploaded CSV file (retained for audit)
- `preview_url`: Location of preview JSON showing proposed changes
- `rollback_available`: Boolean indicating if import can be undone
- `dry_run`: If true, validation occurred but no database writes were performed

### CSV Row Mapping

Each CSV row maps to one Game record. The import system transforms CSV columns into the Game entity structure defined in [Game Model](game-model.md).

**Minimum Required Columns**:
- `name`: Game title (e.g., "Grand Staff Guide Notes 1")
- `slug`: URL-safe identifier (e.g., "grand-staff-guide-notes-1")
- `instrument`: Piano, general_music, choir, guitar, band, orchestra
- `category`: Staff, Pitch, Rhythm, Intervals, Chords, Scales, Symbols, etc.
- `level`: beginner, elementary, intermediate, advanced, mastery
- `difficulty`: Integer 1-6 representing granular difficulty within level

**Optional Columns** (see [CSV Spec Games](../20-imports-and-automation/csv-spec-games.md) for complete list):
- `legacy_id`: Original MLC 4.0 game ID (e.g., "GAM-0950") for migration tracking
- `description`: Full text description of game mechanics and learning objectives
- `short_description`: Brief summary for catalog cards (max 200 chars)
- `mode`: visual, aural, kinesthetic, or multi_sensory
- `device_profile`: any, desktop_preferred, or touch_friendly
- `keyboard_mode`: non_keyboard, virtual_keyboard, midi_required, or midi_optional
- `concept_tags`: Comma-separated list of concept slugs (e.g., "guide_notes,treble_staff,bass_staff")
- `standards`: Comma-separated list of standard codes (e.g., "custom:mlc:L1.staff.guide_notes")
- `prerequisites`: Comma-separated list of game IDs that should be completed before this game
- `pedagogical_notes`: Free-text notes for teachers about how to use the game
- `html5_bundle_url`: CDN URL to game HTML5 bundle
- `thumbnail_url`: CDN URL to game thumbnail image
- `audio_sprite_url`: CDN URL to audio sprite file
- `target_score`: Default target score for completion (can be overridden per stage)
- `pass_threshold`: Minimum score to pass (can be overridden per stage)
- `time_limit_seconds`: Default time limit (can be overridden per stage)
- `midi_supported`: Boolean - does game support MIDI input
- `mic_supported`: Boolean - does game support microphone input
- `language`: Default language code (e.g., "en", "es", "fr")
- `solfege_mode`: Boolean - does game use solfege syllables instead of letter names
- `free_trial_eligible`: Boolean - is game available in free trial mode

**Stage-Specific Columns** (optional, auto-generated if omitted):
- `stages`: JSON string defining which stage types to create (e.g., `["learn","play","quiz","review"]`)
- `learn_enabled`: Boolean - create Learn stage
- `play_enabled`: Boolean - create Play stage
- `quiz_enabled`: Boolean - create Quiz stage
- `challenge_enabled`: Boolean - create Challenge stage
- `review_enabled`: Boolean - create Review stage

**Example CSV Row**:
```csv
name,slug,instrument,category,level,difficulty,legacy_id,concept_tags,html5_bundle_url,thumbnail_url,stages
"Grand Staff Guide Notes 1","grand-staff-guide-notes-1","piano","Staff","beginner",2,"GAM-0950","guide_notes,treble_staff,bass_staff","https://cdn.mlc/games/G-01542/v3/index.html","https://cdn.mlc/thumbs/G-01542.png","[\"learn\",\"play\",\"quiz\",\"review\"]"
```

### Validation Rules

The import system enforces the same validation rules as individual game creation (see [Games CRUD](games-crud.md#validation-rules)) plus additional bulk-specific checks:

**Field-Level Validation**:
- All required fields must be present and non-empty
- Slugs must match pattern `[a-z0-9-]+` (lowercase, alphanumeric, hyphens only)
- Email addresses, URLs, and numeric fields must be well-formed
- Enumerated fields (instrument, category, level, mode, etc.) must match allowed values
- Concept tags and standards must reference existing taxonomy entries
- Asset URLs must be accessible and return HTTP 200 status

**Cross-Record Validation**:
- Slugs must be unique across all rows in the import file
- Game names should be unique (warning if duplicates found)
- Legacy IDs must be unique if provided (prevents duplicate migration)
- Prerequisite game IDs must refer to existing games or games being created in the same import

**Referential Integrity Validation**:
- Concept tags must exist in taxonomy (see [Game Taxonomy](../07-content-architecture/game-taxonomy.md))
- Standards codes must exist in standards registry
- Asset URLs must resolve to valid files on CDN or storage bucket
- Prerequisite game IDs must refer to existing games or games being created in same batch

**Business Rule Validation**:
- Games with `keyboard_mode: midi_required` must have `midi_supported: true`
- Games with `solfege_mode: true` should be tagged with appropriate solfege concepts
- Free trial eligible games should have complete metadata and assets

## Behavior and rules

### Import Workflow

The import process follows a multi-stage pipeline with validation gates and rollback points:

```
┌─────────────────┐
│   1. UPLOAD     │  User uploads CSV file via admin interface
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│  2. VALIDATE    │  Parse CSV, check format, validate each row
└────────┬────────┘  (Errors block progression to preview)
         │
         ↓
┌─────────────────┐
│  3. PREVIEW     │  Show user proposed changes, conflicts, warnings
└────────┬────────┘  (User reviews and confirms or cancels)
         │
         ↓
┌─────────────────┐
│  4. PROCESS     │  Begin database writes in transaction
└────────┬────────┘  (Progress tracked, can rollback on error)
         │
         ↓
┌─────────────────┐
│  5. COMPLETE    │  Finalize import, generate audit records
└─────────────────┘
```

**Step 1: Upload**
- User navigates to **System Admin → Content Tools → Import Games**
- Clicks "Upload CSV" button and selects local CSV file
- System validates file format (must be valid CSV with UTF-8 encoding)
- File uploaded to secure staging bucket, Import Job record created with status `queued`
- User sees confirmation: "File uploaded successfully. Validating..."

**Step 2: Validate**
- Background job parses CSV and validates each row
- Field-level validation checks data types, formats, required fields
- Cross-record validation checks for duplicates, conflicts
- Referential integrity validation checks external references
- Business rule validation enforces domain logic
- If any **errors** detected, job status becomes `failed` and user sees error report
- If only **warnings** detected, job status becomes `previewing` and user can proceed
- Validation errors are grouped by severity: `error` (blocks import) vs. `warning` (allows import with caution)

**Step 3: Preview**
- System generates preview JSON showing proposed changes
- For `games_create`: Shows all games to be created with full metadata
- For `games_update`: Shows diff of changed fields per game
- For `games_upsert`: Shows which games will be created vs. updated
- User reviews preview in admin interface with expandable row details
- User can download preview JSON for external review
- User clicks "Confirm Import" or "Cancel" based on preview

**Step 4: Process**
- If user confirms, job status changes to `processing`
- System begins transaction and processes rows in batches of 50
- For each row, system calls appropriate CRUD operation (see [Games CRUD](games-crud.md))
- Progress tracked and displayed to user: "Processing row 50 of 150..."
- If any row fails, transaction rolls back and job status becomes `failed`
- Partial imports not allowed - either all rows succeed or none persist

**Step 5: Complete**
- If all rows succeed, transaction commits and job status becomes `completed`
- Audit records generated for each game created/updated (see [Games CRUD](games-crud.md#audit-record))
- User sees success notification: "Import completed successfully. 147 games created."
- User can view import job history and download import report
- Rollback option available for 30 days after import

### Import Modes

The importer supports three operational modes:

**1. Create Mode** (`games_create`)
- All games in CSV must be new (no existing `game_id` or matching `slug`)
- Fails if any game already exists
- Use case: Initial migration from legacy system, onboarding new game collection

**2. Update Mode** (`games_update`)
- All games in CSV must exist (matched by `game_id` or `legacy_id`)
- Fails if any game not found
- Only fields present in CSV are updated, others remain unchanged
- Use case: Bulk metadata updates, re-tagging games, fixing typos across collection

**3. Upsert Mode** (`games_upsert`)
- Creates new games if not found, updates existing games if found
- Match by `game_id`, `legacy_id`, or `slug` (in that priority order)
- Most flexible but requires careful validation to avoid unintended updates
- Use case: Incremental imports, synchronizing external game database

**Mode Selection**:
User selects mode in upload interface before confirming import. Default is `games_create` for safety.

### Conflict Resolution

When importing games, conflicts can arise from duplicates or ambiguous references:

**Duplicate Slugs**:
- **Within CSV file**: Validation error, import blocked until user fixes CSV
- **With existing games**: 
  - Create mode: Error, import blocked
  - Update/Upsert mode: Warning, user must confirm overwrite

**Duplicate Names**:
- Warning issued but import allowed (names should be unique but not enforced at database level)
- User can choose to auto-append suffix "(2)" to duplicate names

**Duplicate Legacy IDs**:
- Error, import blocked (legacy IDs must map 1:1 to new games for migration integrity)

**Matching Strategy for Updates/Upserts**:
1. If `game_id` provided in CSV, match by `game_id` (highest priority)
2. If `legacy_id` provided and no `game_id`, match by `legacy_id`
3. If neither provided, match by `slug`
4. If no match found in Update mode, row fails
5. If no match found in Upsert mode, create new game

### Asset Handling During Import

The importer validates asset references but does not upload asset files. Asset upload must happen separately via [Games Assets Pipeline](games-assets-pipeline.md).

**Asset URL Validation**:
- If `html5_bundle_url`, `thumbnail_url`, or `audio_sprite_url` provided, system performs HTTP HEAD request to verify accessibility
- If asset URL returns 404 or 5xx error, validation warning issued: "Asset not found at specified URL"
- Import proceeds with warning (game created but may not be playable until assets uploaded)
- Game status set to `draft` if critical assets missing, must be manually reviewed before publishing

**Asset Migration Workflow**:
1. Upload CSV with game metadata (including placeholder asset URLs)
2. Import creates game records with `status: draft`
3. Separately, upload asset files via Assets Pipeline (see [Games Assets Pipeline](games-assets-pipeline.md#upload-workflow))
4. Asset URLs auto-populate in game records when upload completes
5. System Admin reviews games and publishes to `status: live` when assets verified

**Legacy Asset Migration**:
For migrating MLC 4.0 games, legacy asset paths follow pattern:
```
Legacy: /game_uploads/GAM-0950-2E/index.html
New:    https://cdn.mlc/games/G-01542/v1/play/index.html
```

Import CSV should include both `legacy_id` and `legacy_asset_path` columns to facilitate asset migration mapping.

### Stage Auto-Generation

Games typically have multiple stages (Learn, Play, Quiz, Challenge, Review) - see [Games Stages Policy](games-stages-policy.md). The importer can auto-generate stages based on game level and type.

**Auto-Generation Rules**:
- If `stages` column present in CSV (JSON array), use explicit stage list
- If `stages` column omitted, apply default stage generation:
  - **Beginner/Elementary games**: Create `learn`, `play`, `quiz`, `review` stages
  - **Intermediate/Advanced games**: Create `play`, `quiz`, `challenge`, `review` stages
  - **Mastery games**: Create `play`, `challenge` stages only

**Stage-Specific Overrides**:
CSV can include per-stage columns to override defaults:
- `learn_target_score`, `play_target_score`, `quiz_target_score`, etc.
- `learn_pass_threshold`, `play_pass_threshold`, etc.
- `learn_time_limit`, `play_time_limit`, etc.

If stage-specific columns omitted, stages inherit game-level defaults.

### Dry-Run Mode

The importer supports a `dry_run` flag for testing imports without database writes.

**Dry-Run Behavior**:
- All validation steps execute normally
- Preview generated showing proposed changes
- No database writes occur (no games created/updated)
- Job status ends at `previewing` and never progresses to `processing`
- User can review validation errors and preview, then re-upload corrected CSV

**Use Cases**:
- Testing CSV format before production import
- Validating metadata completeness and asset references
- Training new administrators on import workflow
- Auditing legacy data quality before migration

**Enabling Dry-Run**:
Checkbox in upload interface: "Dry-run mode (validate only, do not import)"

### Rollback and Recovery

Failed imports can be rolled back to restore system state before import.

**Rollback Eligibility**:
- Available for `completed` imports within 30 days
- Not available for `dry_run` imports (nothing to rollback)
- Not available if games from import have been subsequently edited manually

**Rollback Process**:
1. System Admin navigates to **Import History** and selects completed import job
2. Clicks "Rollback Import" button
3. System checks eligibility and warns if any games have been edited since import
4. If eligible, system:
   - Deletes games created by import (if no external references exist)
   - Reverts updated games to pre-import state (restores old field values)
   - Creates audit records for rollback operation
5. Job status changes to `rolled_back`
6. User sees confirmation: "Import rolled back successfully. 147 games deleted."

**Rollback Limitations**:
- Cannot rollback if games have been assigned to sequences or students
- Cannot rollback if games have been edited manually (prevents data loss)
- Cannot rollback after 30-day retention window (audit records archived)

### Batch Transaction Boundaries

The importer processes CSV rows in batches to balance performance and atomicity.

**Batch Size**: 50 rows per batch (configurable by system administrators)

**Transaction Scope**:
- Each batch processed within a single database transaction
- If any row in batch fails, entire batch rolls back
- Successfully processed batches commit immediately
- If later batch fails, earlier batches remain committed (partial import possible)

**All-or-Nothing Mode** (default):
- Single transaction wraps entire import
- If any row fails, entire import rolls back
- Ensures no partial imports

**Progressive Mode** (optional):
- Each batch commits independently
- Failed batches reported but don't affect successful batches
- Allows partial imports with detailed error reporting
- User must manually handle failed rows (re-upload corrected subset)

**Mode Selection**:
Checkbox in upload interface: "Allow partial imports (commit successful batches even if some rows fail)"

## UX requirements (if applicable)

### Import Interface Layout

**Location**: System Admin → Content Tools → Import Games

**Main Elements**:
1. **Upload Panel** (top)
   - File selector input: "Choose CSV file"
   - Import mode dropdown: Create / Update / Upsert
   - Dry-run mode checkbox
   - Allow partial imports checkbox
   - "Upload and Validate" primary button

2. **Validation Results Panel** (shows after upload)
   - Summary cards:
     - Total rows: 150
     - Validation errors: 3
     - Validation warnings: 12
     - Estimated import time: ~5 minutes
   - Expandable error list with row numbers and actionable messages
   - "Download Error Report" link (CSV with error annotations)
   - "Fix and Re-upload" button or "Proceed to Preview" button (if no errors)

3. **Preview Panel** (shows after validation)
   - Tabbed view: "Create (120) | Update (27) | Skip (3)"
   - Data table showing game name, slug, category, level, and primary changes
   - Expandable row detail showing full metadata diff
   - "Download Preview JSON" link
   - "Confirm Import" primary button or "Cancel" secondary button

4. **Processing Panel** (shows during import)
   - Progress bar: "Processing row 75 of 150 (50%)"
   - Animated spinner
   - Estimated time remaining
   - "Cancel Import" button (available until 50% progress)

5. **Completion Panel** (shows after import)
   - Success message: "Import completed successfully!"
   - Summary cards:
     - Games created: 120
     - Games updated: 27
     - Games skipped: 3
     - Import duration: 4m 32s
   - "View Import History" link
   - "Rollback Import" button (if eligible)
   - "Import Another File" button

### Import History View

**Location**: System Admin → Content Tools → Import History

**Elements**:
- Table of recent import jobs with columns:
  - Import ID (e.g., IMP-20250112-00042)
  - Uploaded by
  - Upload date/time
  - Status (badge with color coding)
  - Total rows
  - Success/Fail counts
  - Actions: View Details / Download Report / Rollback (if eligible)
- Filter controls: By status, by date range, by uploader
- Search bar: Search by import ID or file name
- Pagination: 25 imports per page

### Error Message Examples

**Actionable error messages** guide users to fix specific issues:

- `Row 23, field 'slug': Duplicate slug 'grand-staff-guide-notes-1' found in row 18. Each slug must be unique.`
- `Row 67, field 'concept_tags': Invalid concept tag 'rythm'. Did you mean 'rhythm'? See taxonomy for valid tags.`
- `Row 102, field 'html5_bundle_url': Asset not found at 'https://cdn.mlc/games/G-01542/v3/index.html'. Upload assets first or use placeholder URL.`
- `Row 145, field 'prerequisites': Game ID 'G-99999' not found. Prerequisite must reference existing game.`

### Accessibility Notes

- Upload interface supports keyboard navigation (Tab, Enter, Esc)
- File input has accessible label and ARIA attributes
- Progress bar has `role="progressbar"` with `aria-valuenow`, `aria-valuemin`, `aria-valuemax`
- Error messages announced via ARIA live region for screen readers
- Color-coded status badges include text labels (not color-only)
- Data tables have proper header associations for screen reader navigation

## Telemetry

Track all import operations for analytics, debugging, and audit compliance:

### Events

**`import.games.upload_started`**  
Fired when user uploads CSV file  
Properties:
- `import_job_id`: Unique job identifier
- `uploaded_by`: User ID or email
- `file_name`: Original CSV filename
- `file_size_bytes`: Size of uploaded file
- `import_mode`: create | update | upsert
- `dry_run`: boolean

**`import.games.validation_completed`**  
Fired when validation phase completes  
Properties:
- `import_job_id`
- `total_rows`: Number of data rows
- `validation_errors`: Count of error-level issues
- `validation_warnings`: Count of warning-level issues
- `validation_duration_ms`: Time spent validating

**`import.games.preview_viewed`**  
Fired when user views preview panel  
Properties:
- `import_job_id`
- `games_to_create`: Count
- `games_to_update`: Count
- `games_to_skip`: Count

**`import.games.import_confirmed`**  
Fired when user confirms import after preview  
Properties:
- `import_job_id`
- `import_mode`: create | update | upsert
- `allow_partial`: boolean

**`import.games.import_completed`**  
Fired when import successfully completes  
Properties:
- `import_job_id`
- `successful_rows`: Count
- `failed_rows`: Count
- `import_duration_ms`: Total time from confirm to complete

**`import.games.import_failed`**  
Fired when import fails due to errors  
Properties:
- `import_job_id`
- `failure_reason`: Error message or code
- `failed_at_row`: Row number where failure occurred
- `processed_rows`: Rows successfully processed before failure

**`import.games.rollback_completed`**  
Fired when import successfully rolled back  
Properties:
- `import_job_id`
- `rolled_back_by`: User ID or email
- `games_deleted`: Count of games deleted
- `games_reverted`: Count of games reverted to previous state

## Permissions

Who can perform import operations and access import history:

### Permission Roles

**System Administrator**:
- Can upload and execute game imports
- Can view all import history
- Can rollback imports
- Can configure import settings (batch size, validation rules)

**Content Editor** (limited):
- Can upload and execute game imports in `dry_run` mode only
- Can view preview and validation results
- Cannot execute live imports (must request System Administrator approval)
- Can view own import history only

**Teacher / Student**:
- No access to import interface

### Permission Checks

**Before Upload**:
- Verify user has `games.import` permission
- If Content Editor, enforce `dry_run` mode only

**Before Import Execution**:
- Verify user has `games.write` permission
- Verify user is System Administrator for non-dry-run imports

**Before Rollback**:
- Verify user has `games.delete` permission
- Verify user is System Administrator

### Audit Logging

All import operations logged to audit trail (see [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)):
- User ID and email of uploader
- Import job ID and status transitions
- Timestamp of each operation
- Changes made (games created/updated/deleted)
- Rollback operations with reason

## Dependencies

### Internal Dependencies

- **[Games CRUD](games-crud.md)**: Importer calls create/update operations for each validated row
- **[Games Metadata](games-metadata.md)**: Importer enforces metadata field definitions and validation rules
- **[Games Assets Pipeline](games-assets-pipeline.md)**: Asset URLs validated against CDN, but asset upload is separate workflow
- **[Game Taxonomy](../07-content-architecture/game-taxonomy.md)**: Concept tags and categories validated against taxonomy
- **[CSV Spec Games](../20-imports-and-automation/csv-spec-games.md)**: Defines CSV column specifications and format
- **[Bulk Ops Jobs](../20-imports-and-automation/bulk-ops-jobs.md)**: Generic background job infrastructure for batch processing
- **[Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)**: Import operations generate audit trail records

### External Dependencies

- **File Storage**: CSV files and preview JSON stored in S3 or equivalent object storage
- **CDN**: Asset URL validation requires HTTP requests to CDN endpoints
- **Database**: Postgres with transaction support for atomic batch imports and rollback
- **Background Jobs**: Async processing framework (e.g., Sidekiq, Bull, Celery) for validation and import pipelines
- **Notification System**: Email or in-app notifications for long-running import completions

### Migration Dependencies

For MLC 4.0 to 5.0 migration specifically:

- **Legacy Database Access**: Read-only connection to MLC 4.0 database for cross-referencing `legacy_id` values
- **Asset Migration Tools**: Scripts to transfer game bundles from old hosting to new CDN structure
- **Taxonomy Mapping**: Mapping table from legacy concept codes to new taxonomy slugs

## Open questions

### Outstanding Decisions

1. **Partial Import Policy**  
   **Question**: Should progressive mode (partial imports) be enabled by default or require explicit opt-in?  
   **Options**:
   - A) Default to all-or-nothing, checkbox to enable partial (safer, clearer failure modes)
   - B) Default to partial, checkbox to enforce all-or-nothing (more flexible, faster feedback)  
   **Recommendation**: Option A - all-or-nothing by default for data integrity, partial mode opt-in for advanced users

2. **Validation Strictness**  
   **Question**: Should certain metadata warnings be promoted to errors that block import?  
   **Examples**: Missing pedagogical notes, incomplete asset URLs, unvalidated concept tags  
   **Options**:
   - A) Allow import with warnings, rely on post-import QA workflow
   - B) Block import on critical warnings, require metadata completeness  
   **Recommendation**: Option A with post-import review dashboard flagging incomplete games

3. **Duplicate Name Handling**  
   **Question**: How should duplicate game names be resolved during import?  
   **Options**:
   - A) Block import on duplicate names (strictest)
   - B) Allow duplicates with warning (most flexible)
   - C) Auto-append suffix like "(2)" to duplicates (auto-resolution)  
   **Recommendation**: Option B with prominent warning in preview, let admin decide manually

4. **Stage Auto-Generation Defaults**  
   **Question**: Should all games default to 4-stage structure (Learn/Play/Quiz/Review) or only create requested stages?  
   **Options**:
   - A) Always create all 4 stages (consistent structure, easier sequencing)
   - B) Only create stages explicitly listed in CSV (more flexible, less clutter)  
   **Recommendation**: Option A but allow `stages: []` column to override and specify exactly which stages to create

5. **Rollback Time Window**  
   **Question**: How long should rollback be available after import?  
   **Options**:
   - A) 24 hours (short window, encourages immediate validation)
   - B) 30 days (longer window, safer for large migrations)
   - C) Indefinitely (maximum flexibility, higher storage cost)  
   **Recommendation**: Option B (30 days) with audit log retention for forensics beyond rollback window

6. **Asset Validation Depth**  
   **Question**: Should asset validation perform HTTP HEAD requests synchronously during validation or async after import?  
   **Options**:
   - A) Synchronous during validation (slower import, immediate feedback)
   - B) Async after import (faster import, delayed asset error detection)  
   **Recommendation**: Option A with timeout of 5 seconds per asset URL, fail gracefully with warning on timeout

7. **Concept Tag Auto-Suggestion**  
   **Question**: Should system auto-suggest concept tags based on game name and category?  
   **Example**: Game named "Treble Staff Lines" auto-tagged with `treble_staff`, `staff_reading`  
   **Options**:
   - A) No auto-suggestion, require explicit tags in CSV
   - B) Auto-suggest during preview, admin can accept/reject
   - C) Auto-apply tags, admin can edit post-import  
   **Recommendation**: Option B - show suggestions in preview panel with one-click accept/reject per game

8. **Import File Size Limit**  
   **Question**: What is reasonable maximum CSV file size for import?  
   **Options**:
   - A) 1,000 rows (~500 KB) - handles most migrations comfortably
   - B) 5,000 rows (~2.5 MB) - handles very large collections
   - C) No limit, process in streaming fashion  
   **Recommendation**: Option A with guidance to split larger imports into multiple files for manageable error handling
