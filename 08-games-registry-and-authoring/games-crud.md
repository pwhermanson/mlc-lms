# Games CRUD

This document defines the precise create, read, update, and delete operations for Games and their Stages within the registry, enabling Content Editors and System Administrators to manage the game library efficiently and safely.

> **Related Documents**: For the data model schema, see [Game Model](game-model.md). For catalog views and workflows, see [Games Registry Overview](games-registry-overview.md). For stage lifecycle policies, see [Games Stages Policy](games-stages-policy.md).

## Purpose

This spec enables:
- **Safe creation** of new games with required validation before persistence
- **Efficient retrieval** of game records for editing, display, and reporting
- **Controlled updates** with versioning, validation, and rollback support
- **Soft deletion** with deprecation workflow to preserve historical data
- **Audit trail** for all CRUD operations for compliance and debugging
- **Conflict resolution** for concurrent edits by multiple users

Without clear CRUD specifications, the system risks data inconsistency, loss of historical records, broken assignments referencing deleted games, and inability to track who changed what when.

## Scope

**Included**
- API contracts for create, read, update, delete operations
- Validation rules enforced at write time
- Versioning strategy for breaking changes
- Transaction boundaries and rollback behavior
- Concurrent edit detection and resolution
- Soft delete and deprecation workflow
- Audit logging requirements
- Error handling and user feedback

**Excluded**
- Data model schema definitions (see [Game Model](game-model.md))
- UI layouts and catalog views (see [Games Registry Overview](games-registry-overview.md))
- Bulk import/export mechanics (see [Games Importer](games-importer.md))
- Asset upload pipeline (see [Games Assets Pipeline](games-assets-pipeline.md))
- Stage-specific lifecycle policies (see [Games Stages Policy](games-stages-policy.md))

## Data model (if applicable)

### Key Entities Manipulated

**Game**: The parent entity containing metadata, status, and collection of Stages.  
Fields manipulated by CRUD: `game_id`, `game_version`, `name`, `slug`, `status`, `instrument`, `mode`, `device_profile`, `keyboard_mode`, `level`, `difficulty`, `concept_tags`, `category`, `standards`, `pedagogical_notes`, `replaced_by_game_id`, `created_at`, `created_by`, `updated_at`, `updated_by`.

**Stage**: Child entity of Game defining playable units.  
Fields manipulated by CRUD: `stage_id`, `stage_type`, `stage_version`, `target_score`, `pass_threshold`, `time_limit_seconds`, `enabled`.

**Assets**: Nested object within Game.  
Fields manipulated by CRUD: `html5_bundle_url`, `thumbnail_url`, `audio_sprite_url`, `midi_supported`, `mic_supported`.

**Health**: Nested object tracking quality signals.  
Fields manipulated by CRUD: `last_qc_pass`, `browser_support`, `known_limitations`, `degraded_modes`, `deprecated_on`.

**Audit Record**: Separate entity logging each CRUD operation.  
Fields: `audit_id`, `game_id`, `operation` (create|update|delete|deprecate|restore), `user_id`, `timestamp`, `changed_fields`, `old_values`, `new_values`, `reason`.

### Relationships

- A Game has 1 to 5 Stages (Learn, Play, Quiz, optional Challenge, Review).
- A Game may reference another Game via `replaced_by_game_id` when deprecated.
- A Game is referenced by many Sequences, Assignments, and Challenge Boards (read-only foreign keys from those entities).
- Each CRUD operation generates 1 Audit Record.

### Identifiers

- `game_id`: Stable canonical identifier assigned at creation, never changes. Format: `G-XXXXX` (e.g., `G-01542`).
- `game_version`: Integer incremented on breaking changes, defaults to 1 at creation.
- `stage_id`: Derived from `game_id` + `stage_type`. Format: `G-01542-learn`, `G-01542-play`, etc.
- `stage_version`: Integer incremented independently per stage, defaults to 1 at creation.

## Behavior and rules

### Create Operation

**Endpoint**: `POST /api/admin/games`

**Request Payload**:
```json
{
  "name": "Grand Staff Guide Notes A",
  "slug": "grand-staff-guide-notes-a",
  "instrument": "piano",
  "mode": "visual",
  "device_profile": "any",
  "keyboard_mode": "non_keyboard",
  "level": "beginner",
  "difficulty": 2,
  "concept_tags": ["guide_notes", "treble_staff", "bass_staff"],
  "category": "Staff",
  "standards": ["custom:mlc:L1.staff.guide_notes"],
  "pedagogical_notes": "Teaches spatial relationships between guide notes and staff lines",
  "stages": [
    {
      "stage_type": "learn",
      "target_score": null,
      "pass_threshold": null,
      "time_limit_seconds": null,
      "enabled": true
    },
    {
      "stage_type": "play",
      "target_score": 70,
      "pass_threshold": 60,
      "time_limit_seconds": null,
      "enabled": true
    },
    {
      "stage_type": "quiz",
      "target_score": 85,
      "pass_threshold": 80,
      "time_limit_seconds": 180,
      "enabled": true
    },
    {
      "stage_type": "challenge",
      "target_score": null,
      "pass_threshold": null,
      "time_limit_seconds": 90,
      "enabled": false
    },
    {
      "stage_type": "review",
      "target_score": 85,
      "pass_threshold": 80,
      "time_limit_seconds": 180,
      "enabled": true
    }
  ],
  "assets": {
    "html5_bundle_url": null,
    "thumbnail_url": null,
    "audio_sprite_url": null,
    "midi_supported": true,
    "mic_supported": false
  }
}
```

**Validation Rules (enforced before persistence)**:
- `name` is required and must be unique within the organization.
- `slug` is required, must be unique globally, kebab-case format, max 100 chars.
- `concept_tags` array cannot be empty, max 8 tags, each tag max 50 chars.
- `category` must match one of the predefined taxonomy values (see [Game Taxonomy](../07-content-architecture/game-taxonomy.md)).
- `stages` array must include at minimum: `learn`, `play`, `quiz`. `challenge` is optional. `review` is required.
- Each Stage with type `play`, `quiz`, or `review` must have `target_score` and `pass_threshold` defined.
- `game_version` and `stage_version` default to 1 if not provided.
- `status` defaults to `draft` at creation.

**Processing Steps**:
1. Validate request payload against schema and business rules.
2. Generate new `game_id` in format `G-XXXXX` using sequential numbering.
3. Derive `stage_id` for each stage as `{game_id}-{stage_type}`.
4. Set `game_version` and `stage_version` to 1.
5. Set `status` to `draft`.
6. Set `created_at` to current timestamp and `created_by` to requesting user ID.
7. Persist Game and nested Stages in single transaction.
8. Create Audit Record with operation=`create`.
9. Emit telemetry event `game_created` with properties: `game_id`, `editor_id`, `source`=`manual`.

**Success Response** (HTTP 201):
```json
{
  "game_id": "G-01542",
  "game_version": 1,
  "status": "draft",
  "created_at": "2025-01-27T14:30:45.123Z",
  "created_by": "usr_12345",
  "stages": [
    {
      "stage_id": "G-01542-learn",
      "stage_version": 1
    },
    {
      "stage_id": "G-01542-play",
      "stage_version": 1
    },
    {
      "stage_id": "G-01542-quiz",
      "stage_version": 1
    },
    {
      "stage_id": "G-01542-challenge",
      "stage_version": 1
    },
    {
      "stage_id": "G-01542-review",
      "stage_version": 1
    }
  ]
}
```

**Error Responses**:
- HTTP 400: Validation failure (missing required fields, invalid values, duplicate slug)
- HTTP 401: Unauthorized (user not authenticated)
- HTTP 403: Forbidden (user lacks create permission)
- HTTP 500: Server error (database failure, transaction rollback)

**Concurrent Creation**: Multiple users can create different games simultaneously. Slug uniqueness is enforced at database level with unique constraint.

### Read Operation

**Endpoints**:
- `GET /api/admin/games` - List games with filtering and pagination
- `GET /api/admin/games/{game_id}` - Retrieve single game by ID
- `GET /api/admin/games/{game_id}/versions` - List all versions of a game
- `GET /api/admin/games/{game_id}/versions/{version}` - Retrieve specific version

**List Games** (`GET /api/admin/games`):

**Query Parameters**:
```
?status=draft|live|deprecated
&instrument=piano|general_music|choir
&mode=visual|aural|mixed
&level=beginner|early_intermediate|late_intermediate|advanced
&category=Staff|Rhythm|Intervals|Chords|Scales|...
&concept_tags=guide_notes,treble_staff
&search=grand+staff
&sort=name|difficulty|created_at|updated_at
&order=asc|desc
&page=1
&per_page=50
```

**Response** (HTTP 200):
```json
{
  "games": [
    {
      "game_id": "G-01542",
      "game_version": 3,
      "name": "Grand Staff Guide Notes A",
      "slug": "grand-staff-guide-notes-a",
      "status": "live",
      "category": "Staff",
      "difficulty": 2,
      "concept_tags": ["guide_notes", "treble_staff", "bass_staff"],
      "challenge_enabled": false,
      "created_at": "2025-01-15T10:00:00Z",
      "updated_at": "2025-01-27T14:30:45Z",
      "updated_by": "usr_12345"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 50,
    "total_count": 1250,
    "total_pages": 25
  }
}
```

**Get Single Game** (`GET /api/admin/games/{game_id}`):

Returns full game object with all nested stages, assets, health, and metadata. Response follows schema defined in [Game Model](game-model.md).

**Response** (HTTP 200): Full Game object
**Error Responses**:
- HTTP 404: Game not found
- HTTP 401: Unauthorized
- HTTP 403: Forbidden

**Caching**: Read operations may be cached for 5 minutes. ETags supported for conditional requests.

**Performance**: List queries are indexed on `status`, `category`, `created_at`, `updated_at`. Full-text search on `name`, `concept_tags`, `pedagogical_notes` uses search index.

### Update Operation

**Endpoint**: `PUT /api/admin/games/{game_id}`

**Request Payload**: Partial update supported. Only include fields to be changed.
```json
{
  "name": "Grand Staff Guide Notes A (Revised)",
  "difficulty": 3,
  "stages": [
    {
      "stage_id": "G-01542-quiz",
      "target_score": 90,
      "pass_threshold": 85
    }
  ],
  "pedagogical_notes": "Updated to include additional practice patterns",
  "update_reason": "Increased difficulty based on student feedback"
}
```

**Validation Rules**:
- If `status` is `live`, updating core fields (stages, targets, thresholds) creates a new draft version unless explicitly forcing a breaking change.
- Cannot update `game_id`, `created_at`, `created_by`.
- `slug` can be updated but must remain unique.
- Stage updates validated same as creation.
- If updating `status` from `draft` to `live`, full publish validation runs (see [Games Registry Overview](games-registry-overview.md#publish-validation)).

**Versioning Strategy**:
- **Non-breaking changes** (metadata, pedagogical notes, assets): Increment `updated_at`, do not change `game_version`.
- **Breaking changes** (stage targets, thresholds, enabled flags): Increment `game_version` by 1. Existing assignments continue using old version, new assignments use new version.
- **Stage-level breaking changes**: Increment `stage_version` independently.

**Processing Steps**:
1. Fetch current game record with optimistic lock (e.g., `version` field or `updated_at` timestamp).
2. Validate request payload.
3. Check for concurrent edits: If `updated_at` in request differs from database, return HTTP 409 Conflict.
4. Apply updates to mutable fields.
5. If breaking change, increment `game_version`.
6. Set `updated_at` to current timestamp and `updated_by` to requesting user ID.
7. Persist updates in transaction.
8. Create Audit Record with operation=`update`, capture `changed_fields`, `old_values`, `new_values`, `reason`.
9. Emit telemetry event `game_updated` with properties: `game_id`, `game_version`, `fields_changed`, `breaking_changes` boolean.

**Success Response** (HTTP 200): Full updated Game object

**Error Responses**:
- HTTP 400: Validation failure
- HTTP 404: Game not found
- HTTP 409: Conflict (concurrent edit detected, stale version)
- HTTP 401: Unauthorized
- HTTP 403: Forbidden
- HTTP 500: Server error

**Concurrent Edit Resolution**:
When HTTP 409 returned, client should:
1. Fetch latest version of game.
2. Display diff between user's changes and current version.
3. Allow user to merge or overwrite.
4. Retry update with latest `updated_at` timestamp.

**Partial Updates**: PATCH method supported for updating subset of fields without sending full object.

### Delete Operation

**Important**: Hard delete is **not allowed** for games with play history. Soft delete via deprecation is the standard workflow.

**Deprecation Endpoint**: `POST /api/admin/games/{game_id}/deprecate`

**Request Payload**:
```json
{
  "replaced_by_game_id": "G-02100",
  "deprecation_notes": "Replaced by updated version with improved visuals and audio",
  "deprecate_date": "2025-02-01T00:00:00Z"
}
```

**Processing Steps**:
1. Validate that `replaced_by_game_id` exists and is in `live` status (if provided).
2. Check usage: If game is referenced in active sequences or assignments, show warning with usage count and links.
3. Set `status` to `deprecated`.
4. Set `replaced_by_game_id` if provided.
5. Set `deprecated_on` to current timestamp or specified `deprecate_date`.
6. Set `updated_at` and `updated_by`.
7. Hide game from All Games search and catalog views.
8. Persist updates in transaction.
9. Create Audit Record with operation=`deprecate`.
10. Emit telemetry event `game_deprecated` with properties: `game_id`, `replaced_by_game_id`, `sequences_count`, `assignments_count`.

**Success Response** (HTTP 200):
```json
{
  "game_id": "G-01542",
  "status": "deprecated",
  "deprecated_on": "2025-01-27T14:30:45Z",
  "replaced_by_game_id": "G-02100",
  "usage_impact": {
    "sequences_count": 12,
    "assignments_count": 45,
    "active_students_count": 120
  }
}
```

**Error Responses**:
- HTTP 400: Invalid replacement game or deprecation date in past
- HTTP 404: Game not found
- HTTP 401: Unauthorized
- HTTP 403: Forbidden (only SysAdmin can deprecate)
- HTTP 500: Server error

**Restore from Deprecation**: `POST /api/admin/games/{game_id}/restore`

Moves game from `deprecated` back to `live` status. Only SysAdmin can restore. Clears `deprecated_on` and `replaced_by_game_id`.

**Hard Delete Endpoint** (restricted): `DELETE /api/admin/games/{game_id}`

**Restrictions**:
- Only SysAdmin can execute hard delete.
- Only allowed if game has **zero** play history (no Score records exist).
- Only allowed if game is not referenced in any Sequences or Assignments.
- Requires confirmation with `force=true` query parameter.

**Processing Steps**:
1. Check permissions: Only SysAdmin.
2. Check play history: If any Score records exist, return HTTP 409 with error message.
3. Check references: If any Sequences or Assignments reference game, return HTTP 409 with error message.
4. Require `force=true` query parameter as final confirmation.
5. Delete Game and all child Stages in transaction.
6. Create Audit Record with operation=`delete`, preserve full game object JSON in audit.
7. Emit telemetry event `game_hard_deleted` with properties: `game_id`, `reason`.

**Success Response** (HTTP 204 No Content)

**Error Responses**:
- HTTP 400: Missing `force=true` parameter
- HTTP 409: Game has play history or references, cannot delete
- HTTP 404: Game not found
- HTTP 401: Unauthorized
- HTTP 403: Forbidden (only SysAdmin)

### Special Operations

**Clone Game**: `POST /api/admin/games/{game_id}/clone`

Creates a copy of an existing game with new `game_id` and `status=draft`. Useful for creating variations or A/B test versions.

**Request Payload**:
```json
{
  "new_name": "Grand Staff Guide Notes A (Variant)",
  "new_slug": "grand-staff-guide-notes-a-variant",
  "copy_assets": true
}
```

**Response** (HTTP 201): Returns new Game object with new `game_id`.

**Publish Operation**: `POST /api/admin/games/{game_id}/publish`

Moves game from `draft` to `live` status after full validation. See [Games Registry Overview](games-registry-overview.md#publish-validation) for validation rules.

**Rollback Version**: `POST /api/admin/games/{game_id}/rollback/{version}`

Reverts game to a previous version. Only SysAdmin can rollback. Creates audit record with old and new version numbers.

## UX requirements (if applicable)

### Create Flow

1. **Entry Point**: "Create New Game" button in registry catalog view.
2. **Form Layout**: Multi-step wizard or single scrollable form with sections:
   - Step 1: Basic Info (name, slug, category, instrument, mode, level, difficulty)
   - Step 2: Concept Tagging (tags, standards, pedagogical notes)
   - Step 3: Stage Configuration (enable/disable, targets, thresholds, time limits)
   - Step 4: Assets (optional at creation, can be added later)
3. **Validation Feedback**: Inline validation on blur, summary of errors at top of form.
4. **Auto-slug**: Generate slug from name automatically, allow manual override.
5. **Save as Draft**: Primary action. Does not require full validation.
6. **Cancel**: Discard changes, return to catalog.

### Edit Flow

1. **Entry Point**: Click game name in catalog or "Edit" button on detail view.
2. **Load Current State**: Fetch latest version, display loading indicator.
3. **Form Pre-population**: All fields populated with current values.
4. **Change Tracking**: Visual indicator for changed fields (e.g., colored border).
5. **Conflict Detection**: If another user edited game, show warning banner: "This game was updated by {user} {X minutes ago}. Review changes before saving."
6. **Version Indicator**: Display current `game_version` and `updated_at` prominently.
7. **Save Options**:
   - "Save Changes" (non-breaking update)
   - "Save as New Version" (breaking change, increments version)
   - "Save as Draft" (if currently live, creates draft copy)
8. **Rollback Option**: SysAdmin sees "Rollback to Previous Version" button.

### Delete/Deprecate Flow

1. **Entry Point**: "Deprecate" button on detail view (or "Delete" for SysAdmin if eligible).
2. **Usage Impact Warning**: Modal shows:
   - Number of sequences referencing this game
   - Number of active assignments
   - Number of students affected
   - Links to view each reference
3. **Replacement Selection**: If deprecating, optional field to select replacement game with autocomplete search.
4. **Confirmation**: Require typing game name to confirm deprecation.
5. **Success Feedback**: Toast notification: "Game deprecated successfully. Existing assignments will continue to work."
6. **Hard Delete (SysAdmin only)**: Separate "Hard Delete" button with red warning, requires `force` checkbox and typing game name.

### Read/List Flow

1. **Catalog View**: See [Games Registry Overview](games-registry-overview.md#catalog-view) for full layout.
2. **Detail View**: Tabs for Overview, Stages, Assets, Usage, Health, History.
3. **Version History**: Table showing all versions with timestamp, editor, change summary, and "View" or "Rollback" actions.

### Accessibility

- All forms keyboard navigable with logical tab order.
- Error messages announced to screen readers.
- Confirmation modals use ARIA dialog role and focus management.
- Loading states announced to screen readers.
- Version conflict warnings have high color contrast and icon indicators.

## Telemetry

### CRUD Event List

All events follow schema defined in [Event Model](../15-analytics-and-reporting/event-model.md#event-core-schema).

**Game Created**  
Event: `game_created`  
Properties:
```json
{
  "game_id": "G-01542",
  "editor_id": "usr_12345",
  "source": "manual|import|clone",
  "status": "draft",
  "category": "Staff",
  "instrument": "piano",
  "mode": "visual",
  "stage_count": 5
}
```
Fires: Immediately after successful persistence.

**Game Updated**  
Event: `game_updated`  
Properties:
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "editor_id": "usr_12345",
  "fields_changed": ["difficulty", "stages.quiz.target_score"],
  "breaking_changes": true,
  "update_reason": "Increased difficulty based on student feedback"
}
```
Fires: After successful update persistence.

**Game Published**  
Event: `game_published`  
Properties:
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "editor_id": "usr_12345",
  "validation_passed": true
}
```
Fires: After status changes from `draft` to `live`.

**Game Deprecated**  
Event: `game_deprecated`  
Properties:
```json
{
  "game_id": "G-01542",
  "editor_id": "usr_12345",
  "replaced_by_game_id": "G-02100",
  "sequences_count": 12,
  "assignments_count": 45,
  "deprecation_notes": "Replaced by updated version"
}
```
Fires: After status changes to `deprecated`.

**Game Hard Deleted** (rare)  
Event: `game_hard_deleted`  
Properties:
```json
{
  "game_id": "G-01542",
  "editor_id": "usr_12345",
  "reason": "Test game with no usage"
}
```
Fires: After permanent deletion.

**Version Rollback**  
Event: `game_version_rollback`  
Properties:
```json
{
  "game_id": "G-01542",
  "from_version": 5,
  "to_version": 4,
  "editor_id": "usr_12345",
  "reason": "Reverted problematic update"
}
```
Fires: After successful rollback.

**Concurrent Edit Conflict**  
Event: `game_edit_conflict`  
Properties:
```json
{
  "game_id": "G-01542",
  "editor_id": "usr_12345",
  "conflicting_editor_id": "usr_67890",
  "resolution": "cancelled|merged|overwrite"
}
```
Fires: When HTTP 409 returned and again when conflict resolved.

### Sampling

CRUD events are **never sampled**. All create, update, delete, and deprecate events are captured 100% for audit compliance.

## Permissions

See [Permissions Table](../02-roles-and-permissions/permissions-table.md#content-management-permissions) for detailed matrix.

**Create Game**:
- Content Editors: ✅ Yes
- SysAdmin: ✅ Yes
- All others: ❌ No

**Read Game**:
- Content Editors: ✅ All games (including drafts)
- Teachers: ✅ Live games only (policy-dependent, may exclude drafts)
- SysAdmin: ✅ All games
- Students: ❌ No direct registry access (play games via assignments)

**Update Game**:
- Content Editors: ✅ All games
- SysAdmin: ✅ All games
- All others: ❌ No

**Publish Game** (draft → live):
- Content Editors with publish permission: ✅ Yes
- SysAdmin: ✅ Yes
- Content Editors without publish permission: ❌ No

**Deprecate Game**:
- SysAdmin: ✅ Yes
- Content Editors: ❌ No (can request deprecation, SysAdmin approves)

**Hard Delete Game**:
- SysAdmin: ✅ Yes (with restrictions)
- All others: ❌ No

**Rollback Version**:
- SysAdmin: ✅ Yes
- All others: ❌ No

**Permission Checks**: Enforced at API layer before any operation. Unauthorized attempts logged for security monitoring.

## Supporting Documents Referenced

This games CRUD specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Original game organization and data structure requirements
- [Games Loaded.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Sheet1.csv) — Current game library inventory and metadata examples
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Game import specifications and field definitions
- [EXE specs 2012-0913 - Game Screen.csv](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Game%20Screen.csv) — Game screen specifications and metadata display requirements

## Dependencies

### Internal Dependencies

- **[Game Model](game-model.md)**: Defines schema and validation rules enforced by CRUD operations.
- **[Games Registry Overview](games-registry-overview.md)**: Provides catalog views and workflows that consume CRUD APIs.
- **[Games Stages Policy](games-stages-policy.md)**: Defines stage lifecycle rules enforced during create and update.
- **[Games Importer](games-importer.md)**: Bulk import uses create API under the hood.
- **[Games Assets Pipeline](games-assets-pipeline.md)**: Asset URLs validated and referenced by CRUD operations.
- **[Permissions Table](../02-roles-and-permissions/permissions-table.md)**: Permission checks enforced before operations.
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Telemetry event schemas and properties.
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: Usage impact analysis checks sequence references.
- **[Assignment Model](../07-content-architecture/assignment-model.md)**: Usage impact analysis checks assignment references.

### External Dependencies

- **Database Layer**: PostgreSQL or equivalent for transactional CRUD operations, optimistic locking via `updated_at` timestamp.
- **Search Index**: Elasticsearch or equivalent for full-text search on game catalog, updated asynchronously after CRUD operations.
- **Audit Store**: Separate audit log database or table for immutable audit records.
- **Event Pipeline**: Message queue (e.g., Kafka, RabbitMQ) for emitting telemetry events to analytics warehouse.
- **Asset CDN**: Validation checks asset URLs are reachable, but CRUD does not manage upload/deletion of assets.

### API Design

- RESTful conventions: `POST` for create, `GET` for read, `PUT`/`PATCH` for update, `DELETE` for hard delete, `POST` for special operations (deprecate, publish, rollback).
- Idempotency: `PUT` and `DELETE` operations idempotent. `POST` for create is not idempotent (generates new `game_id` each time).
- Rate Limiting: CRUD operations rate-limited per user to prevent abuse (e.g., 100 requests/minute).
- Versioning: API versioned as `/api/v1/admin/games` to allow future breaking changes.

## Open questions

### Versioning Strategy

- **Question**: Should we support branching versions (e.g., `v3.1`, `v3.2`) or only linear incrementing (`v3`, `v4`)?  
- **Current Assumption**: Linear incrementing only, simpler to reason about.
- **Decision Needed By**: Phase 1 alpha

### Concurrent Edit Locking

- **Question**: Should we implement pessimistic locking (lock game when opened for editing) or optimistic locking (detect conflicts at save time)?  
- **Current Assumption**: Optimistic locking with conflict detection via `updated_at` timestamp.
- **Trade-offs**: Pessimistic locking prevents conflicts but can lock records if user leaves tab open. Optimistic allows better concurrency but requires conflict resolution UX.
- **Decision Needed By**: Phase 1 alpha

### Soft Delete Retention

- **Question**: How long should deprecated games remain in the system before hard deletion?  
- **Current Assumption**: Indefinite retention, no automatic hard deletion.
- **Alternative**: Auto-delete deprecated games after 2 years if no usage.
- **Decision Needed By**: Phase 2

### Version Diff Visualization

- **Question**: Should we provide a visual diff tool showing field-by-field changes between versions?  
- **Current Assumption**: Version history shows change summary only. Full diff is stretch goal.
- **Decision Needed By**: Phase 2

### Bulk Update API

- **Question**: Should we provide an API to update multiple games in one request (e.g., batch-update difficulty)?  
- **Current Assumption**: Single-game updates only. Bulk operations done via import.
- **Alternative**: Add `PATCH /api/admin/games/bulk` endpoint.
- **Decision Needed By**: Phase 2

### Clone Depth

- **Question**: When cloning a game, should we also clone asset files or only reference same URLs?  
- **Current Assumption**: Clone references same asset URLs, assets not duplicated.
- **Alternative**: Provide `deep_clone=true` option to duplicate assets.
- **Decision Needed By**: Phase 1 beta

### Deprecation Auto-Migration

- **Question**: When a game is deprecated with a replacement, should sequences and assignments auto-migrate to the new game?  
- **Current Assumption**: No auto-migration. Sequences show warning and suggest replacement, but require manual update.
- **Alternative**: Provide opt-in auto-migration.
- **Decision Needed By**: Phase 2
