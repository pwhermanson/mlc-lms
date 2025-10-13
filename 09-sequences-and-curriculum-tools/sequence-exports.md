# Sequence Exports

## Purpose
Define export formats, workflows, and use cases for extracting Sequence data from the MLC-LMS. This spec enables Teachers, Content Editors, Publishers, and System Admins to share, back up, migrate, or distribute curriculum content in standardized formats that preserve the complete sequence structure including Groups, Assignments, Steps, gating rules, targets, and metadata.

For the underlying sequence data structure, see [Sequence Model](sequence-model.md). For import workflows and bulk authoring, see [Sequence Authoring](sequence-authoring.md). For CSV schema details, see [CSV Specifications](../20-imports-and-automation/csv-specifications.md).

## Scope

### Included
- Export format specifications (CSV and JSON)
- Export workflow and user interface
- Export scope options (full sequence, metadata only, structure only)
- Use cases: backup, sharing, method book distribution, migration, offline reference
- Validation and integrity checks before export
- Permissions and access control for export operations
- Export file naming conventions and versioning
- Telemetry for export events

### Excluded
- Import workflows and conflict resolution (see [Sequence Authoring](sequence-authoring.md))
- CSV schema implementation details for other entity types (see [CSV Specifications](../20-imports-and-automation/csv-specifications.md))
- Bulk operations job queue management (see [Bulk Operations Jobs](../20-imports-and-automation/bulk-ops-jobs.md))
- Game asset bundling and media file exports (see [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md))
- Publisher API integration patterns (see [API Design Principles](../18-architecture/api-design-principles.md))

## Use cases

### Backup and archival
**Who**: Content Editors, System Admins  
**Why**: Preserve version history, maintain disaster recovery copies, archive deprecated sequences  
**Format**: JSON (includes all metadata and change history)  
**Frequency**: On-demand and scheduled automated backups

### Sharing between organizations
**Who**: Teachers, Teacher-Admins, Subscriber-Admins  
**Why**: Share custom sequences with peer organizations, distribute curated curriculum  
**Format**: CSV or JSON  
**Constraints**: Export scope limited to organization-owned custom sequences unless System Admin

### Method book distribution
**Who**: Content Editors, Publishers  
**Why**: Distribute method-aligned sequences to licensed organizations, enable offline curriculum planning  
**Format**: CSV with method correlation metadata  
**Constraints**: Export includes publisher metadata, book identifiers, page ranges, and skill mappings

### Migration between systems
**Who**: System Admins  
**Why**: Migrate sequences from legacy MLC-4.0 system or between production environments  
**Format**: JSON (preserves all internal IDs and relationships for exact reconstruction)  
**Constraints**: Full data integrity validation required before export

### Offline reference and curriculum planning
**Who**: Teachers, Curriculum Designers  
**Why**: Review sequence structure offline, plan lessons in spreadsheets, analyze curriculum coverage  
**Format**: CSV (human-readable, spreadsheet-compatible)  
**Constraints**: May include embedded game metadata for context

### Publisher review and approval
**Who**: Content Editors submitting to Publishers  
**Why**: Share sequences with external publishers for pedagogical review and IP verification  
**Format**: CSV or PDF (read-only summary format)  
**Constraints**: Excludes internal system IDs, includes only pedagogical metadata

## Data model (if applicable)

### Export job entity
Each export operation creates a job record for tracking and retrieval:

**Fields**:
- `export_job_id`: UUID for the export operation
- `sequence_id`: Target sequence canonical ID
- `sequence_version`: Specific version being exported
- `requested_by`: User ID who initiated export
- `requested_at`: Timestamp of export request
- `export_format`: Enum: `csv` | `json` | `pdf_summary`
- `export_scope`: Enum: `full` | `structure_only` | `metadata_only`
- `include_game_metadata`: Boolean (embed game names, descriptions, concepts)
- `include_version_history`: Boolean (JSON only, includes change log)
- `export_target`: Enum: `download` | `api` | `backup` | `archive`
- `status`: Enum: `queued` | `processing` | `completed` | `failed`
- `file_url`: Pre-signed URL for download (expires in 24 hours)
- `file_size_bytes`: Size of generated export file
- `expires_at`: Timestamp when download link expires
- `error_message`: Optional error details if status is `failed`

### Export file metadata (embedded in JSON exports)
JSON exports include a header block with metadata for validation and provenance:

```json
{
  "export_metadata": {
    "mlc_export_version": "1.0",
    "export_timestamp": "2025-10-12T14:23:45Z",
    "exported_by_user_id": "USR-12345",
    "exported_by_org_id": "ORG-67890",
    "system_version": "MLC-5.0.2",
    "export_scope": "full",
    "include_game_metadata": true,
    "include_version_history": false
  },
  "sequence": {
    // Sequence data follows...
  }
}
```

### CSV export format
CSV exports follow a flat structure with one row per Step, using columns to represent the hierarchy. This format mirrors the legacy MLC-4.0 sequence definition tables.

**Column headers**:
- `sequence_id`: Canonical sequence identifier (e.g., "SEQ-LIFE", "SEQ-HALL-BOOK-1")
- `sequence_version`: Integer version number
- `sequence_name`: Display name (e.g., "Lifetime Musician")
- `group_id`: Group identifier within sequence
- `group_name`: Group display name (e.g., "Primary Level 1A")
- `group_order`: Integer position in sequence
- `assignment_id`: Assignment identifier within group
- `assignment_name`: Assignment display name (e.g., "Assignment 1")
- `assignment_order`: Integer position within group
- `assignment_optional`: Boolean (0 or 1)
- `step_id`: Step identifier within assignment
- `step_order`: Integer position within assignment
- `element_type`: Enum: `game` | `text` | `video` | `audio` | `reward`
- `element_id`: Reference to content registry (e.g., "G-01542", "VID-INS-2005")
- `stage`: Stage name if element_type is `game`: `learn` | `play` | `quiz` | `challenge` | `review`
- `gating_require_previous_steps`: Boolean (0 or 1)
- `gating_min_attempts`: Integer
- `gating_min_score_from_prior`: Decimal (optional)
- `target_score`: Decimal (optional, for game steps)
- `pass_threshold`: Decimal (optional, for game steps)
- `review_offset_days`: Integer (for review steps)
- `step_notes`: Free text rationale or instructions

**Optional columns when `include_game_metadata` is true**:
- `game_name`: Display name of referenced game
- `game_description`: Short description
- `game_concepts`: Comma-separated concept tags (e.g., "PITCH,STAFF,SCALE")
- `game_difficulty_tier`: Integer 1-6

**Example CSV rows**:
```csv
sequence_id,sequence_version,sequence_name,group_id,group_name,group_order,assignment_id,assignment_name,assignment_order,assignment_optional,step_id,step_order,element_type,element_id,stage,gating_require_previous_steps,gating_min_attempts,target_score,pass_threshold,step_notes
SEQ-LIFE,7,Lifetime Musician,G-01,Foundations,1,A-01,Guide Notes and Beat,1,0,S-01,1,game,G-01542,learn,0,1,,,Introduce staff guide notes
SEQ-LIFE,7,Lifetime Musician,G-01,Foundations,1,A-01,Guide Notes and Beat,1,0,S-02,2,game,G-01542,play,0,1,70,60,
SEQ-LIFE,7,Lifetime Musician,G-01,Foundations,1,A-01,Guide Notes and Beat,1,0,S-03,3,game,G-01542,quiz,1,1,85,80,
SEQ-LIFE,7,Lifetime Musician,G-01,Foundations,1,A-01,Guide Notes and Beat,1,0,S-04,4,game,G-01542,review,1,,85,80,Review after 7 days
SEQ-LIFE,7,Lifetime Musician,G-01,Foundations,1,A-01,Guide Notes and Beat,1,0,S-05,5,text,TXT-0301,,0,,,,"Practice checklist and posture tips"
```

### JSON export format
JSON exports preserve the full nested hierarchy and are suitable for programmatic processing, API integration, and exact system reconstruction.

**Structure** (see [Sequence Model](sequence-model.md) for complete schema):
```json
{
  "export_metadata": { /* ... */ },
  "sequence": {
    "sequence_id": "SEQ-LIFE",
    "sequence_version": 7,
    "name": "Lifetime Musician",
    "visibility": "first_party",
    "instrument": "piano",
    "metadata": {
      "description": "Default long form curriculum",
      "target_age": "8-14",
      "visual_aural_mix": "mixed",
      "estimated_hours": 120,
      "prerequisite_skills": ["basic_rhythm", "note_reading_foundation"],
      "completion_criteria": "all_assignments_mastered",
      "publisher": null,
      "series_name": null,
      "book_level": null,
      "edition_year": null
    },
    "groups": [
      {
        "group_id": "G-01",
        "name": "Foundations",
        "assignments": [
          {
            "assignment_id": "A-01",
            "name": "Guide Notes and Beat",
            "optional": false,
            "steps": [
              {
                "step_id": "S-01",
                "element_type": "game",
                "element_id": "G-01542",
                "stage": "learn",
                "gating": { "min_attempts": 1 },
                "targets": null,
                "notes": "Introduce staff guide notes"
              }
              // ... more steps
            ]
          }
          // ... more assignments
        ]
      }
      // ... more groups
    ]
  }
}
```

When `include_version_history` is true, JSON exports also include:
```json
{
  "version_history": [
    {
      "sequence_version": 6,
      "published_at": "2024-08-15T10:00:00Z",
      "published_by": "USR-98765",
      "change_summary": "Added Review steps for all Quiz stages"
    },
    {
      "sequence_version": 7,
      "published_at": "2025-01-20T14:30:00Z",
      "published_by": "USR-12345",
      "change_summary": "Updated target scores based on Q4 2024 performance data"
    }
  ]
}
```

## Behavior and rules

### Export workflow

#### Initiating an export
1. User navigates to sequence detail view or library management page
2. Clicks "Export Sequence" action button
3. Export configuration modal appears with options:
   - **Format**: CSV, JSON, or PDF Summary (read-only)
   - **Scope**: Full sequence, Structure only (no gating/targets), Metadata only (names and IDs)
   - **Include game metadata**: Checkbox (embeds game names, descriptions, concepts for offline reference)
   - **Include version history**: Checkbox (JSON only, adds change log)
   - **Export target**: Download, API endpoint, Backup archive

#### Validation before export
Before queuing the export, system validates:
- **Content integrity**: All referenced games and learning elements exist and are accessible
- **Permission check**: User has export permission for the target sequence (see Permissions section)
- **Sequence health**: No critical health warnings (see [Games Audit and Health](../08-games-registry-and-authoring/games-audit-and-health.md))
- **Version consistency**: If exporting a pinned version, verify it still exists and hasn't been purged

If validation fails, user sees specific error messages and cannot proceed.

#### Export processing
1. System creates an `export_job` record with status `queued`
2. Background job picks up the export request (see [Background Jobs](../18-architecture/background-jobs.md))
3. Status updates to `processing`
4. System generates the requested file format:
   - CSV: Flattens hierarchy into rows, applies column mapping
   - JSON: Serializes full nested structure with metadata header
   - PDF: Renders read-only formatted summary (not covered in this spec, see separate PDF rendering spec)
5. File is stored in temporary export bucket with pre-signed URL (24-hour expiration)
6. Status updates to `completed`, user is notified
7. If error occurs, status updates to `failed` with error message

#### Download and expiration
- User receives download link via in-app notification and optional email
- Download link expires after 24 hours for security
- After expiration, user can request a new export
- Backup and archive exports are retained in long-term storage with organizational retention policies

### Export scope options

#### Full sequence (default)
Exports all data: Groups, Assignments, Steps, gating rules, targets, scheduling, notes, and sequence metadata. Suitable for complete backups and migrations.

#### Structure only
Exports the hierarchy (Groups → Assignments → Steps) and element references, but excludes:
- Gating rules (min_attempts, require_previous_steps, min_score_from_prior)
- Target scores and pass thresholds
- Review scheduling offsets
- Step notes

Use case: Share the curriculum outline without pedagogical implementation details.

#### Metadata only
Exports only identifying information:
- Sequence ID, version, name, visibility
- Group IDs and names
- Assignment IDs and names
- Step IDs and element IDs

Use case: Generate a table of contents or index for reporting and planning.

### File naming conventions
Generated export files follow a standardized naming pattern for traceability:

**CSV**:  
`{sequence_id}_v{sequence_version}_{YYYYMMDD}_{scope}.csv`  
Example: `SEQ-LIFE_v7_20251012_full.csv`

**JSON**:  
`{sequence_id}_v{sequence_version}_{YYYYMMDD}_{scope}.json`  
Example: `SEQ-HALL-BOOK-1_v3_20251012_full.json`

**PDF**:  
`{sequence_id}_v{sequence_version}_{YYYYMMDD}_summary.pdf`  
Example: `SEQ-SOLF_v2_20251012_summary.pdf`

### Export integrity validation
After generating the export file, system performs integrity checks:
- **Row count match** (CSV): Number of steps matches expected count
- **JSON schema validation**: Output conforms to canonical sequence model schema
- **Reference integrity**: All `element_id` values are valid (not orphaned)
- **Checksum generation**: MD5 hash stored with export job for future verification

If validation fails, export job status is set to `failed` and no file is made available.

### Rate limiting
To prevent abuse and protect system resources:
- **Per user**: Maximum 10 export requests per hour
- **Per organization**: Maximum 100 export requests per hour
- **Large sequences** (>500 steps): Maximum 5 exports per hour, processed with lower priority

When rate limit is exceeded, user sees a clear error message with retry timing.

## UX requirements (if applicable)

### Export configuration modal
**Trigger**: "Export Sequence" button in sequence detail view or library management table  
**Layout**:
- Modal overlay with clear heading: "Export {Sequence Name}"
- Radio button group for format selection: CSV, JSON, PDF Summary
- Radio button group for scope: Full, Structure Only, Metadata Only
- Checkbox options:
  - ☑ Include game metadata (game names, descriptions, concepts)
  - ☑ Include version history (JSON only, disabled for CSV)
- Dropdown for export target: Download, API, Backup Archive
- "Cancel" and "Export" buttons at bottom

**Validation feedback**:
- If sequence has health warnings, show alert banner: "This sequence has {n} content warnings. Export will include flagged elements. Review health status before proceeding."
- If user lacks permission, disable "Export" button with tooltip explaining required role

### Export progress and notification
After user confirms export:
- Modal closes, toast notification appears: "Export queued. You'll be notified when ready."
- In-app notification center shows progress: "Exporting {Sequence Name}... (processing)"
- When complete, notification updates: "Export complete. Download {filename}" with clickable link
- If failed, notification shows error: "Export failed: {error_message}. Try again or contact support."

### Export history view (optional enhancement)
Teachers and Admins can view past exports in a dedicated "Export History" tab:
- Table columns: Sequence Name, Version, Format, Scope, Requested At, Status, Download Link
- Status indicators: Queued (spinner), Processing (progress bar), Completed (checkmark + link), Failed (error icon)
- Download links expire after 24 hours, show "Expired - Request New Export" button
- Sortable by date, filterable by status and sequence

### Accessibility
- Export configuration modal is keyboard navigable
- Screen reader announces format and scope selections
- Download link in notification is accessible with keyboard focus
- Error messages are announced to screen readers
- High-contrast mode supported for all status indicators

## Telemetry

### Events and properties

**`sequence_exported`**
- **Fired when**: User initiates an export and it successfully queues
- **Properties**:
  - `export_job_id`: UUID
  - `sequence_id`: Canonical sequence identifier
  - `sequence_version`: Integer
  - `export_format`: csv | json | pdf_summary
  - `export_scope`: full | structure_only | metadata_only
  - `include_game_metadata`: boolean
  - `include_version_history`: boolean
  - `export_target`: download | api | backup | archive
  - `user_id`: ID of requesting user
  - `org_id`: Organization context
  - `sequence_visibility`: first_party | method_aligned | custom | archived
  - `groups_count`: Number of groups in sequence
  - `assignments_count`: Number of assignments in sequence
  - `steps_count`: Number of steps in sequence

**`sequence_export_completed`**
- **Fired when**: Export job finishes successfully
- **Properties**:
  - `export_job_id`: UUID
  - `sequence_id`: Canonical sequence identifier
  - `export_format`: csv | json | pdf_summary
  - `file_size_bytes`: Size of generated file
  - `processing_duration_ms`: Time from queue to completion
  - `validation_passed`: boolean

**`sequence_export_failed`**
- **Fired when**: Export job fails
- **Properties**:
  - `export_job_id`: UUID
  - `sequence_id`: Canonical sequence identifier
  - `export_format`: csv | json | pdf_summary
  - `error_type`: validation_failed | permission_denied | content_missing | system_error
  - `error_message`: Descriptive error text
  - `processing_duration_ms`: Time from queue to failure

**`sequence_export_downloaded`**
- **Fired when**: User clicks download link
- **Properties**:
  - `export_job_id`: UUID
  - `sequence_id`: Canonical sequence identifier
  - `export_format`: csv | json | pdf_summary
  - `download_method`: in_app_notification | email_link | export_history_view
  - `time_since_export_ms`: Time between export completion and download

**`sequence_export_link_expired`**
- **Fired when**: User attempts to access an expired download link
- **Properties**:
  - `export_job_id`: UUID
  - `sequence_id`: Canonical sequence identifier
  - `time_since_export_hours`: Hours elapsed since export completion

### Sampling rules
- **Never sampled**: `sequence_exported`, `sequence_export_failed` (critical audit events)
- **10% sampling**: `sequence_export_downloaded`, `sequence_export_link_expired` (volume reduction)
- **Never sampled for System Admins**: All events are captured at 100% for audit trail

## Permissions

### Permission matrix by role

| Action | Student | Teacher | Teacher-Admin | Subscriber-Admin | Content Editor | SysAdmin |
|--------|---------|---------|---------------|------------------|----------------|----------|
| Export own custom sequences | ❌ | ✅ | ✅ | ✅ | N/A | ✅ |
| Export org-scoped custom sequences | ❌ | ✅ (own org) | ✅ (own org) | ✅ (own org) | N/A | ✅ (any org) |
| Export first-party sequences (LIFE, SOLF, EVAL) | ❌ | ✅ (structure only) | ✅ (structure only) | ✅ (structure only) | ✅ (full) | ✅ (full) |
| Export method-aligned sequences | ❌ | ✅ (if org has entitlement) | ✅ (if org has entitlement) | ✅ (if org has entitlement) | ✅ | ✅ |
| Export archived sequences | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Include version history in export | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Export to backup/archive target | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |

### Permission rules by sequence visibility

#### First-party sequences (LIFE, SOLF, EVAL)
- Teachers, Teacher-Admins, and Subscriber-Admins can export with `structure_only` or `metadata_only` scope for curriculum planning
- Only Content Editors and SysAdmins can export with `full` scope including gating rules and targets (proprietary pedagogical IP)

#### Method-aligned sequences
- Export requires organization to have active entitlement for the specific method/publisher
- If entitlement expires, past exports remain accessible for 90 days, new exports are blocked

#### Custom sequences
- Creator (original author) can always export with `full` scope
- Other members of the same organization can export if they have Teacher-Admin or Subscriber-Admin role
- Cross-organization export requires explicit sharing (future enhancement, not in initial scope)

#### Archived sequences
- Only Content Editors and SysAdmins can export archived sequences
- Exports of archived sequences include a warning header in the file indicating deprecated status

### Permission enforcement
Permissions are checked at two points:
1. **Export initiation**: UI shows/hides export button and scope options based on user role and sequence ownership
2. **Export job processing**: Background job re-validates permissions before generating file (defense against permission escalation exploits)

If permission check fails during processing, export job status is set to `failed` with `error_type: permission_denied`.

## Supporting Documents Referenced

This sequence exports specification draws from the following source documents:

- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence export format specifications
- [MLC SeqCreate 2020.xlsx - ALPHA LIST.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20ALPHA%20LIST%20.csv) — Sequence metadata for export operations
- [MLC Sequences (1).xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20Sequences%20(1).xlsx%20-%20specs.csv) — Additional sequence export examples

## Dependencies

### Core sequence specifications
- [Sequence Model](sequence-model.md) - Canonical data structure for sequences, groups, assignments, and steps
- [Sequence Authoring](sequence-authoring.md) - Import workflows, validation, and collaboration features
- [Sequence Libraries](sequence-libraries.md) - Sequence catalog, visibility, and discovery

### Data and content
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game stage definitions and metadata schema
- [Games Audit and Health](../08-games-registry-and-authoring/games-audit-and-health.md) - Content validation and health checks
- [Learning Elements Overview](../07-content-architecture/learning-elements-overview.md) - Non-game content types (text, video, audio, rewards)

### Import/export infrastructure
- [CSV Specifications](../20-imports-and-automation/csv-specifications.md) - CSV schema for all entity types
- [Bulk Operations Jobs](../20-imports-and-automation/bulk-ops-jobs.md) - Background job queue and processing
- [Export Specs](../20-imports-and-automation/export-specs.md) - General export infrastructure (if separate from this spec)

### Roles and permissions
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions and capabilities
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permission definitions
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Authentication context for permission checks

### Architecture and infrastructure
- [Background Jobs](../18-architecture/background-jobs.md) - Job queue, retry logic, and error handling
- [API Design Principles](../18-architecture/api-design-principles.md) - Export API endpoint design
- [Security and Privacy](../18-architecture/security-privacy.md) - Pre-signed URLs, file expiration, data protection
- [Observability](../19-quality-and-operations/observability.md) - Export job monitoring and alerting

### Telemetry
- [Event Model](../15-analytics-and-reporting/event-model.md) - Canonical event definitions and property standards

## Open questions

### Export format and schema
- **CSV encoding**: Should CSV exports use UTF-8 with BOM for Excel compatibility, or UTF-8 without BOM for technical users?
- **CSV dialect**: Use comma delimiter (standard) or tab delimiter for better handling of text fields with commas?
- **JSON schema versioning**: How do we handle backward compatibility when the sequence model evolves? Include schema version in export metadata?
- **Null vs empty**: In CSV exports, should missing optional fields be represented as empty string `""` or `NULL` keyword?

### Export scope and content
- **Embedded game metadata depth**: When `include_game_metadata` is true, should we embed full game metadata (concepts, difficulty, prerequisites) or just name and description?
- **Historical score data**: Should exports optionally include aggregated performance data (average scores, completion rates) for pedagogical analysis?
- **Localized content**: If a sequence includes localized game names or instructions, should exports include all languages or only the user's active language?
- **Asset references**: Should exports include URLs or paths to game asset bundles for offline archival, or only reference game IDs?

### Workflow and automation
- **Scheduled exports**: Should organizations be able to schedule automated nightly exports to external backup systems?
- **Bulk export**: Should System Admins be able to export all sequences in a library (e.g., all method-aligned sequences) as a single batch operation?
- **Export to external LMS**: Should exports include LTI or SCORM packaging options for integration with external Learning Management Systems?
- **Differential exports**: For backup use cases, should we support exporting only changes since last export (delta exports)?

### Permissions and sharing
- **Public sequence sharing**: Should there be a "Public" visibility option for sequences that can be exported by anyone without authentication?
- **Export quotas**: Should organizations have monthly export quotas based on subscription tier to prevent abuse?
- **Watermarking**: Should exported CSV/JSON files include embedded creator/organization identifiers for provenance tracking?
- **Cross-org sharing**: Should Teachers be able to generate shareable export links that other organizations can import (with proper attribution)?

### File retention and storage
- **Long-term archival**: How long should backup/archive exports be retained? Should retention policies vary by sequence visibility?
- **Export history limit**: Should users see only their last 10 exports, or all exports within a time window (e.g., 90 days)?
- **Download link extension**: Should users be able to request download link extension beyond 24 hours if they haven't downloaded yet?
- **Export versioning**: If a user exports the same sequence multiple times in a day, should filenames include a counter suffix (e.g., `_v7_20251012_full_001.csv`)?
