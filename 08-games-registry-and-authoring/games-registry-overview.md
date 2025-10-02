# Games Registry Overview

## Purpose
Define the administrative source of truth for all Games and their Stages so content editors can create, update, validate, publish, audit, and retire content with confidence. The registry powers search and discovery, sequence authoring, assignment generation, AI mapping, and reporting.

> **Legacy Context**: This registry builds upon the original MLC system's game structure where each game was identified by sequential numbers (GAM-0010, GAM-0020, etc.) and included five distinct stages: Learn (1), Play (2), Quiz (3), Challenge (4), and Review (5). The new system maintains this pedagogical foundation while modernizing the technical implementation and administrative capabilities.

## Scope
Included
- Admin-facing catalog, detail, usage, and health views
- CRUD workflows for Game and Stage metadata
- Bulk import and validation from spreadsheets
- Deprecation and replacement policies
- Indexing fields for search and AI
- Telemetry and permissions

Excluded
- Individual game runtime mechanics
- Student and teacher play surfaces
- Billing, pricing, or plan logic
- Deep asset pipeline internals

## Data model (if applicable)
Key entities displayed or edited here
- **Game** with child **Stages**. See Game Model spec for full schema.
- **Replacement mapping** from deprecated Game to replacement Game.
- **Usage links** from Game to Sequences, Assignments, and Challenge Boards.
- **Asset bundle** metadata and health signals.

Core listing fields
- `game_id`, `game_version`, `slug`, `name`, `status` draft|live|deprecated
- `instrument`, `mode` visual|aural|mixed, `device_profile` any|midi_required|mic_optional|mic_required
- `keyboard_mode` keyboard|non_keyboard
- `level`, `difficulty` 1..10
- `concept_tags` array and a single `category` for faceting
- `challenge_enabled` boolean
- `last_published_at`, `last_editor`
- `health_summary` pass|warn|fail

Detail fields and nested Stage table
- Stages: `stage_id`, `stage_type`, `stage_version`, `enabled`, `target_score`, `pass_threshold`, `time_limit_seconds`
- Assets: `html5_bundle_url`, `thumbnail_url`, `audio_sprite_url`, `midi_supported`, `mic_supported`
- Health: `last_qc_pass`, `browser_support[]`, `known_limitations[]`, `degraded_modes[]`
- Usage counts: `sequences_count`, `assignments_count`, `plays_30d`, `unique_students_30d`
- Replacement: `replaced_by_game_id`, `deprecation_notes`

Relationships
- One Game to many Stages
- One Game to many Sequence Steps across many Sequence versions
- Optional one to one replacement link when deprecated

## Behavior and rules
Catalog behaviors
- Filter by status, instrument, mode, device profile, level, category, concept tags, challenge flag, health status, and updated date.
- Sort by name, concept category, difficulty, plays, and last updated.

Create and edit
- Create Game in draft with at least Learn, Play, and Quiz Stage rows present.
- Enforce validation on publish:
  - Learn, Play, Quiz must be enabled with valid targets where applicable.
  - Review is recommended and must have its own score targets if enabled.
  - Challenge is optional and never required for publish.
  - Assets must be reachable and pass smoke tests.
  - Concept tags cannot be empty.
- Edits to a live Game create a draft version until published.

Deprecation and replacement
- Set `status = deprecated` to hide from new assignments and All Games search.
- Provide `replaced_by_game_id` to support auto suggestions in authoring and migration tooling.
- Existing historical scores remain visible. Students can still view results pages, but play is disabled unless teacher explicitly allows legacy replays.

Usage awareness
- The detail view shows where a Game is used across Sequences and Assignments with links.
- Before deprecating or changing targets, show an impact panel listing top referencing sequences and classes.

**Usage Analytics and Tracking**
Based on the original system's approach to game utilization:
- **Play Statistics**: Track 30-day play counts and unique student engagement per game
- **Sequence Integration**: Monitor which sequences reference each game for curriculum impact assessment
- **Assignment Dependencies**: Identify games used in active assignments to prevent breaking changes
- **Student Performance**: Aggregate completion rates, average scores, and common failure points
- **Teacher Preferences**: Track which games are most frequently assigned by teachers
- **Legacy Migration**: Monitor usage of deprecated games to prioritize replacement development

Health checks
- Asset reachability and bundle version match
- Browser matrix support and critical bugs by browser or OS
- MIDI or mic device checks where applicable
- Large asset warnings and load time budgets
- Stage completeness checks: missing targets, disabled required stages
- Deprecated references found in sequences or assignments

**Health Monitoring Details**
Based on the original system's quality assurance approach:
- **Browser Compatibility**: Track support for Chromium 114+, Safari 17+, iOS 17+, Android 13+
- **Asset Validation**: Verify HTML5 bundle integrity, thumbnail availability, audio sprite loading
- **Device Requirements**: Validate MIDI pairing for required games, microphone access for aural games
- **Performance Monitoring**: Bundle size limits (50MB), loading timeouts (30s), memory usage patterns
- **Known Issues Tracking**: Maintain database of reported problems with resolution status and dates
- **Legacy Game Status**: Monitor games marked as "Blank Screen" or other critical issues from original system

Bulk import and validation
- Accept CSV or XLSX based on the Games Loaded and Games import master formats.
- Dry run produces a validation report with errors and warnings.
- Partial accept allows valid rows to import while exporting a fix file for failures.
- Duplicate detection by `game_id` and `slug` with merge preview.

**Import Format Details**
Based on the original Games Loaded spreadsheet structure, the import system supports:
- Game identification using legacy GAM-XXXX format with stage suffixes (-1E, -2E, -3E, -4E, -5E)
- Status tracking: D (Delivered), A (Active), P (Playable), N (Not available)
- Core metadata: game_number, title, description, thumbnail_img, Category, skill_level, skill_area_id
- Device requirements: keyboard, timed, solfege_text, midi_capable, language
- Stage-specific data: stage_id, game_path, target_score, Free flag
- Health tracking: Problems column for known issues and resolution dates

Indexing and AI signals
- Maintain a search index document per Game with fields for quick lookup and faceting.
- Expose `concept_tags`, `difficulty`, `instrument`, `mode`, `keyboard_mode`, `category`, `standards[]` for AI sequence mapping.

## UX requirements (if applicable)
Catalog view
- Left filter rail with sticky apply and clear actions.
- Main table with frozen columns: Name, Category, Status, Health. Additional columns toggled via a column picker.
- Row badges for Challenge, MIDI, Mic, and Draft.

Detail view
- Header with Game name, status, and quick actions Publish, Deprecate, Duplicate.
- Tabs
  - Overview: core fields, concept tags, category
  - Stages: editable table for stage targets, thresholds, enable toggles, time limits
  - Assets: URLs, previews, auto checks and manual QC checklist
  - Usage: sequences, assignments, and recent plays with links
  - Health: check results with severities and suggested fixes
  - History: audit trail of edits and publishes

Import view
- Upload widget supporting CSV and XLSX
- Validation results table with filters by error type
- Download buttons for Fix File and Full Report
- Commit button gated behind zero critical errors

Accessibility
- All table controls and editors keyboard reachable
- Screen reader labels for validation messages and health statuses
- Color choices meet contrast standards and rely on icons plus labels to convey state

## Telemetry
Key events
- `game_created`
  - `game_id`, `editor_id`, `source` manual|import
- `game_published`
  - `game_id`, `game_version`, `breaking_changes` boolean
- `game_deprecated`
  - `game_id`, `replaced_by_game_id?`
- `game_updated`
  - `game_id`, `fields_changed[]`
- `registry_import_started` and `registry_import_finished`
  - counts for rows, passed, failed, warnings
- `registry_validation_issue`
  - `game_id?`, `severity` error|warning, `code`, `message`
- `health_check_run`
  - `game_id`, `result` pass|warn|fail, `checks_failed[]`
- `usage_panel_opened`
  - `game_id`, `sequences_count`, `assignments_count`

Sampling
- Publish, deprecate, and import events are never sampled.
- Health check and panel open events may sample at 10 percent.

## Permissions
- Read
  - Content Editors, Teachers, Admins, and SysAdmin can read. Teachers may be restricted from draft visibility by policy.
- Create and edit
  - Content Editors and SysAdmin.
- Publish and deprecate
  - Content Editors with publish rights and SysAdmin.
- Import
  - Content Editors with import permission and SysAdmin.
- Delete
  - Hard delete is not allowed once a Game has any play history. Soft delete through deprecation only.

## Dependencies
- [Game Model](game-model.md) for schema, stages, targets, and lifecycle rules.
- [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) for usage mapping and replacement migration.
- [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md) and [QA Checklists](../19-quality-and-operations/qa-checklists.md) for health validations.
- [CSV Import Specifications](../20-imports-and-automation/csv-specifications.md) for column mapping and error handling.
- [Search Architecture](../18-architecture/search-architecture.md) for indexing documents and facets.
- [Event Model](../15-analytics-and-reporting/event-model.md) for canonical names and properties.
- [Feature Flags](../06-system-admin/sysadmin-feature-flags.md) for staged rollout of new registry capabilities.
- [Assignment Play](../03-student-experience/assignment-play.md) for student-facing gameplay integration.
- [All Games Library](../03-student-experience/all-games-library.md) for student game selection interface.

## Open questions
- Do we allow per organization overrides of Game metadata for labeling or only per assignment targets.
- Should a deprecated Game be auto swapped in sequences when `replaced_by_game_id` is present or remain a manual review step.
- Minimum health score required to publish to live.
- Limit on concept tags per Game to keep search precise, for example max 8 tags.
- Policy for large bundle sizes and progressive download thresholds.

## Legacy System Integration
**Migration from Original MLC System**
The registry must support seamless migration from the original system's game structure:
- **Game ID Mapping**: Convert GAM-XXXX format to new G-XXXXX format while maintaining backward compatibility
- **Stage Structure**: Preserve the five-stage pedagogical model (Learn, Play, Quiz, Challenge, Review) with enhanced metadata
- **Asset Migration**: Support migration of existing HTML5 bundles, thumbnails, and audio assets
- **Data Preservation**: Maintain historical scores and student progress during the transition
- **Teacher Workflow**: Ensure existing teacher assignments and sequences continue to function

**Quality Assurance from Legacy**
Based on the original system's 88% delivery rate and known issues:
- **Critical Issues**: Address games with "Blank Screen" problems and MIDI compatibility issues
- **Performance Optimization**: Improve loading times and reduce bundle sizes for better student experience
- **Device Compatibility**: Enhance MIDI and microphone support based on original system limitations
- **User Experience**: Implement the two-tier game selection (Game → Stage) as specified in original redesign requirements

## Key views
- Catalog - searchable table with filters for concept, level, stage availability
- Detail - stage configuration, target scores, assets, concept tags
- Usage - which Sequences and Assignments reference this Game
- Health - asset checks and browser matrix flags

## Admin actions
- Create or update Game metadata
- Toggle stage availability
- Set target scores per stage
- Attach assets or mark missing
- Deprecate and replace

## Bulk import
- Import from Games Loaded and Games import master
- Validation report and partial accept
- Dry run mode

## Audit and security
- Change history
- Role-based permissions for edit vs publish
