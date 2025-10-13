# Sequence Authoring

This document will define the tools and interface for creating and editing sequences.

## Purpose
Define the authoring experience, workflows, and tooling for creating and editing Sequences in the MLC-LMS. This spec ensures Teachers, Content Editors, and System-Admins can efficiently build curriculum pathways that meet pedagogical goals, align to method books, and maintain content health.

For the underlying sequence data structure and versioning rules, see [Sequence Model](sequence-model.md). For sequence discovery and catalog management, see [Sequence Libraries](sequence-libraries.md).

## Scope

### Included
- Authoring UI components and workflows for creating and editing sequences
- Drag-and-drop organization of Groups, Assignments, and Steps
- In-context game and learning element selector
- Step-level configuration for gating, targets, and scheduling
- Validation, preview, and testing tools during authoring
- Publishing and versioning workflows
- Import/export mechanisms for bulk authoring (CSV and JSON)
- Collaboration and draft management
- Health checks and content warnings
- Permission controls for authoring by role

### Excluded
- Sequence data model schema (see [Sequence Model](sequence-model.md))
- Assignment delivery and student progression logic (see [Assignment Generation](../10-assignments-engine/assignment-generation.md))
- Sequence library browsing and discovery (see [Sequence Libraries](sequence-libraries.md))
- Method book correlation metadata standards (see [Sequence Alignment to Methods](sequence-alignment-to-methods.md))
- AI-assisted sequence generation (see [AI Sequence Generator](ai-sequence-generator.md) - currently on backburner)
- Reporting and analytics (see [Teacher Reports](../04-teacher-experience/teacher-reports.md) and [Admin Reports](../05-admin-subscriber-experience/admin-reports.md))

## Data model (if applicable)

The authoring tool operates on the core sequence entities defined in [Sequence Model](sequence-model.md): Sequence, Group, Assignment, Step. This section covers authoring-specific extensions and state management.

### Authoring states
Each sequence being authored has a lifecycle state:

- **draft**: Work in progress, not visible to end users, can be freely edited
- **review**: Submitted for pedagogical or publisher review, edit-locked except for reviewers
- **published**: Live and visible to the appropriate audience based on `visibility` attribute
- **archived**: No longer actively maintained, hidden from new assignments but retained for historical reporting

### Draft management
- `draft_id`: Unique identifier for draft versions (UUIDv4)
- `created_by`: User ID of the author
- `last_edited_by`: User ID of last editor
- `last_edited_at`: Timestamp of last modification
- `change_summary`: Free text describing recent changes
- `parent_sequence_version`: If editing an existing published sequence, references the base version

### Collaboration metadata
- `collaborators[]`: Array of user IDs with edit permissions for this draft
- `comment_threads[]`: Inline comments on specific steps or assignments for review feedback
  - `thread_id`, `step_id`, `author_id`, `created_at`, `resolved` boolean, `messages[]`

### Change tracking
For published sequences being edited, the authoring tool maintains a diff to show:
- Steps added, removed, or reordered
- Changes to gating rules, targets, or scheduling
- Game or element replacements

This enables teachers to understand what will change when migrating pinned assignments to a new sequence version.

## Behavior and rules

### Authoring workflow

#### Creating a new sequence
1. **Initiate**: User clicks "Create New Sequence" from the sequence library management view
2. **Metadata entry**: Form collects basic metadata:
   - Sequence name
   - Visibility type (first-party, method-aligned, custom)
   - Target instrument (default: piano)
   - Estimated hours, target age, prerequisite skills
   - For method-aligned: publisher, series name, book level, edition year
   - For custom sequences: automatically scoped to user's `org_id`
3. **Template selection** (optional): Choose to start from a blank slate or clone an existing sequence as a starting point
4. **Draft created**: System generates a `draft_id` and opens the authoring canvas

#### Editing an existing sequence
When editing a published sequence:
1. System creates a new draft linked to the current published version via `parent_sequence_version`
2. Diff tracking is enabled to highlight all changes
3. Existing assignments remain pinned to the prior version until authors explicitly publish the new version
4. Teachers are notified of available updates and can opt to migrate

#### Building the sequence structure
The authoring canvas uses a hierarchical drag-and-drop interface:

**Group level**:
- Add, remove, rename, reorder groups
- Collapse/expand groups for easier navigation
- Bulk operations: duplicate group, move group

**Assignment level**:
- Add, remove, rename, reorder assignments within a group
- Set assignment metadata: name, description, optional flag
- Bulk operations: duplicate assignment, move to different group

**Step level**:
- Add learning elements via in-context selector (see UX Requirements)
- Configure step properties:
  - Element type and ID (game, text, video, audio, reward)
  - For games: select stage (Learn, Play, Quiz, Challenge, Review)
  - Gating rules: `require_previous_steps`, `min_attempts`, `min_score_from_prior`
  - Targets: `target_score`, `pass_threshold`
  - Scheduling: `review_offset_days` for Review steps
  - Notes: free text rationale for pedagogical decisions
- Reorder steps via drag-and-drop
- Bulk operations: duplicate step, copy step to clipboard, paste step

### Validation rules

The authoring tool runs validation checks continuously and blocks publishing if critical errors exist:

#### Critical errors (block publishing)
- **Missing element references**: Step references a `game_id` or `element_id` that does not exist in the content registry
- **Invalid stage selection**: Step specifies a stage (e.g., Challenge) that the referenced game does not support
- **Circular dependencies**: Gating rules create impossible progression logic
- **Empty sequence**: Sequence has no groups, or all groups are empty
- **Duplicate IDs**: Within a sequence version, `group_id`, `assignment_id`, or `step_id` must be unique

#### Warnings (allow publishing with acknowledgment)
- **Deprecated content**: Step references a game marked as `deprecated` or `sunset` in the [Games Registry](../08-games-registry-and-authoring/games-registry-overview.md)
- **Missing assets**: Game bundle or media assets not yet uploaded to CDN (see [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md))
- **Unbalanced progression**: Sequence heavily favors one skill area (e.g., all visual, no aural games)
- **Targets out of range**: Target scores set unrealistically high or low based on historical game performance data
- **Review steps without scheduling**: Review step lacks `review_offset_days` (defaults to 7 if not set)

#### Advisory notices (informational only)
- **Long assignments**: Assignment contains more than 15 steps (may overwhelm students)
- **No Challenge stages**: Sequence includes no Challenge stages (fine, but consider for engagement)
- **Inconsistent naming**: Assignment or group names don't follow organizational conventions

### State transitions

#### Draft → Review
- Author clicks "Submit for Review"
- System checks for critical errors; if present, transition is blocked
- If warnings exist, author must acknowledge each before proceeding
- Draft becomes edit-locked except for users with `reviewer` role
- Reviewers can add comment threads and request changes
- Author can revoke submission to return to draft state

#### Review → Published
- Reviewer or System-Admin clicks "Publish"
- System increments `sequence_version` if editing an existing sequence
- Sequence becomes visible based on `visibility` attribute (first-party, method-aligned, custom)
- Notification sent to relevant users (e.g., teachers who have used prior versions)
- Draft is retained for audit trail but marked as superseded

#### Published → Archived
- System-Admin (or Teacher for custom sequences) clicks "Archive"
- Sequence `visibility` set to `archived`
- No longer appears in new assignment workflows
- Existing assignments remain playable and continue to reference the archived version
- Can be unarchived if needed

### Versioning behavior

When a sequence is published as a new version:
- `sequence_version` integer increments
- All `group_id`, `assignment_id`, `step_id` values remain stable if the structure hasn't changed
- If IDs are reused with different content, the system flags this as a breaking change and warns teachers during migration
- Change summary and diff are saved for teacher review

For detailed versioning rules and pinning behavior, see [Sequence Model](sequence-model.md).

## UX requirements (if applicable)

### Authoring canvas layout

The sequence authoring interface uses a three-column layout optimized for desktop workflows:

#### Left sidebar: Sequence structure tree
- Collapsible tree view showing Groups > Assignments > Steps hierarchy
- Drag handles for reordering at any level
- Icons indicate:
  - Element type (game, text, video, audio, reward)
  - Validation status (error, warning, valid)
  - Draft/published state
- Click any node to jump to that section in the main canvas
- Search/filter to quickly find assignments or games by name

#### Center canvas: Detail editor
- Contextual editor based on selected node:
  - **Sequence level**: Metadata form (name, visibility, instrument, estimated hours, etc.)
  - **Group level**: Group name, description, reorder assignments
  - **Assignment level**: Assignment name, description, optional flag, step list with inline editing
  - **Step level**: Full step configuration form (element selector, stage, gating, targets, scheduling, notes)
- Inline validation messages appear next to fields with errors or warnings
- Drag-and-drop zones for reordering steps within an assignment
- "Add Step" button opens the learning element selector modal

#### Right sidebar: Preview and tools
- **Preview tab**: Live preview of how the assignment will appear to students
  - Shows step sequence with gating rules applied
  - Displays target scores and pass thresholds
  - Indicates which steps are locked vs. available
- **History tab**: Change log showing recent edits with timestamps and author names
- **Comments tab**: Inline comment threads for collaboration during review
- **Health tab**: Summary of validation errors, warnings, and advisory notices

### Learning element selector

When adding a new step, the author opens a modal to choose the learning element:

**Game selector**:
- Searchable list of all games with filters:
  - Concept tags (intervals, rhythm, key signatures, etc.)
  - Difficulty level
  - Stage availability (only show games that support the desired stage)
  - Status (live, deprecated, draft)
- Preview pane shows game thumbnail, description, and available stages
- "Select Stage" dropdown: Learn, Play, Quiz, Challenge, Review
- "Add to Assignment" button inserts the step at the current position

**Non-game elements**:
- Tabs for Text, Video, Audio, Reward
- Each tab lists available elements with preview
- Text and Video elements can be created inline for custom instructional content

For game metadata and registry details, see [Games Metadata](../08-games-registry-and-authoring/games-metadata.md) and [Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md).

### Step configuration form

Each step has an inline or expanded form with the following fields:

**Element reference** (read-only after creation):
- Element type and ID
- For games: thumbnail, name, and selected stage

**Gating rules** (expandable section):
- Checkbox: "Require previous steps to be completed"
- Number input: "Minimum attempts required"
- Number input: "Minimum score from prior step" (optional, references previous step)

**Targets** (for game steps only):
- Number input: "Target score" (suggested score for mastery)
- Number input: "Pass threshold" (minimum score to mark step complete)
- Hint text shows historical average score for this game stage

**Scheduling** (for Review steps only):
- Number input: "Review offset (days)" (defaults to 7)
- Calendar picker: "Due date" (optional, for scheduled assignments)

**Notes** (free text):
- Textarea for pedagogical rationale, implementation notes, or teacher guidance

### Bulk operations

Authors can select multiple steps, assignments, or groups and perform bulk actions:

- **Duplicate**: Create a copy with incremented IDs
- **Move**: Relocate to a different assignment or group
- **Delete**: Remove selected items (with confirmation prompt)
- **Copy/Paste**: Copy steps to clipboard for reuse in other sequences (within the same org or for System-Admins across orgs)

### Preview and testing

Before publishing, authors can:

- **Student view preview**: Simulate the assignment flow as a student would see it
  - Step-by-step walkthrough with gating rules applied
  - Shows locked/unlocked states based on progression logic
- **Play test**: Launch the first game in a new browser tab to verify it loads correctly
- **Validation report**: Summary view of all errors, warnings, and notices with direct links to fix each issue

### Accessibility

The authoring interface must meet [WCAG Compliance](../16-accessibility-and-inclusion/wcag-compliance.md) standards:

- **Keyboard navigation**: All drag-and-drop operations have keyboard alternatives (e.g., arrow keys + modifier to reorder)
- **Screen reader support**: Tree structure announces hierarchy level, node type, and validation status
- **Focus indicators**: Clear visual focus on all interactive elements
- **Color contrast**: Error/warning/success states use color plus text labels (not color alone)
- **Keyboard shortcuts**: Common actions (save, undo, add step) have keyboard equivalents displayed in tooltips

### Responsive behavior

The authoring interface is optimized for desktop (1280px+ width). For narrower viewports:

- Left and right sidebars collapse into tabs or drawers
- Center canvas remains at full width
- Drag-and-drop gestures adapt to touch for tablet use

For detailed responsive patterns, see [Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md).

## Telemetry

The authoring tool emits events to track usage patterns, identify workflow bottlenecks, and monitor content quality. All telemetry follows the canonical event structure defined in [Event Model](../15-analytics-and-reporting/event-model.md).

### Authoring workflow events

**`sequence_authoring_started`**
- Fired when: User creates a new draft or begins editing an existing sequence
- Properties:
  - `draft_id`: UUID of the draft
  - `sequence_id`: Canonical ID if editing existing sequence (null for new)
  - `visibility`: first_party | method_aligned | custom
  - `template_used`: boolean (true if cloned from existing)
  - `user_role`: Teacher | Content-Editor | System-Admin
  - `org_id`: Organization scope for custom sequences

**`sequence_draft_saved`**
- Fired when: User clicks "Save Draft" or autosave triggers
- Properties:
  - `draft_id`
  - `change_type`: metadata | structure | step_config
  - `groups_count`, `assignments_count`, `steps_count`
  - `validation_errors_count`, `validation_warnings_count`
  - `duration_since_last_save_ms`

**`sequence_element_added`**
- Fired when: User adds a step to an assignment
- Properties:
  - `draft_id`
  - `element_type`: game | text | video | audio | reward
  - `element_id`: ID of the selected element
  - `stage`: Learn | Play | Quiz | Challenge | Review (for games)
  - `assignment_id`: Target assignment
  - `step_position`: Ordinal position within assignment

**`sequence_step_configured`**
- Fired when: User edits step properties (gating, targets, scheduling)
- Properties:
  - `draft_id`, `step_id`
  - `field_changed`: gating | targets | scheduling | notes
  - `has_custom_gating`: boolean
  - `has_custom_targets`: boolean
  - `has_review_scheduling`: boolean

**`sequence_structure_reordered`**
- Fired when: User drags to reorder groups, assignments, or steps
- Properties:
  - `draft_id`
  - `reorder_level`: group | assignment | step
  - `moved_count`: Number of items moved
  - `method`: drag_drop | keyboard | bulk_operation

**`sequence_validation_run`**
- Fired when: Validation check completes (triggered on save or before publish)
- Properties:
  - `draft_id`
  - `critical_errors_count`, `warnings_count`, `advisories_count`
  - `error_types[]`: Array of error categories (e.g., missing_element, invalid_stage, circular_dependency)

**`sequence_preview_opened`**
- Fired when: User opens student view preview or play test
- Properties:
  - `draft_id`
  - `preview_type`: student_view | play_test
  - `assignment_id`: Which assignment is being previewed

### Publishing and review events

**`sequence_submitted_for_review`**
- Fired when: Author submits draft for review
- Properties:
  - `draft_id`
  - `sequence_id`, `sequence_version` (if updating existing)
  - `visibility`
  - `warnings_acknowledged_count`
  - `reviewer_assigned`: User ID of designated reviewer (if any)

**`sequence_review_comment_added`**
- Fired when: Reviewer adds a comment thread
- Properties:
  - `draft_id`, `thread_id`
  - `target_type`: sequence | group | assignment | step
  - `target_id`: ID of the commented item
  - `reviewer_id`

**`sequence_published`**
- Fired when: Sequence moves from review to published state
- Properties:
  - `draft_id`, `sequence_id`, `sequence_version`
  - `visibility`
  - `is_new_sequence`: boolean (true if first publish, false if update)
  - `publisher_id`: User ID who approved publication
  - `groups_count`, `assignments_count`, `steps_count`
  - `breaking_changes`: boolean (true if IDs changed or structure significantly altered)

**`sequence_archived`**
- Fired when: Sequence is archived
- Properties:
  - `sequence_id`, `sequence_version`
  - `archived_by`: User ID
  - `reason`: free text or enum (outdated | replaced | publisher_request | other)

### Import and export events

**`sequence_imported`**
- Fired when: User imports a sequence from CSV or JSON
- Properties:
  - `draft_id`, `import_format`: csv | json
  - `source`: file_upload | api | bulk_migration
  - `groups_imported`, `assignments_imported`, `steps_imported`
  - `validation_errors_count`

**`sequence_exported`**
- Fired when: User exports a sequence
- Properties:
  - `sequence_id`, `sequence_version`
  - `export_format`: csv | json
  - `include_metadata`: boolean
  - `export_target`: download | api | backup

### Sampling rules

- **No sampling**: Publishing and review events are never sampled (critical audit trail)
- **10% sampling**: Autosave, reorder, and validation events may be sampled in production to reduce volume
- **Full capture**: All errors and warnings are logged at 100% regardless of sampling

For telemetry infrastructure and monitoring, see [Observability](../19-quality-and-operations/observability.md).

## Permissions

Sequence authoring permissions follow the role-based access control (RBAC) system defined in [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md).

### Role capabilities matrix

| Action | Student | Teacher | Teacher-Admin | Subscriber-Admin | Content-Editor | System-Admin |
|--------|---------|---------|---------------|------------------|----------------|--------------|
| Create custom sequence | ❌ | ✅ (org-scoped) | ✅ (org-scoped) | ✅ (org-scoped) | ✅ (first-party) | ✅ (any) |
| Edit own custom sequence | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Edit org custom sequences | ❌ | ❌ | ✅ | ✅ | ❌ | ✅ |
| Edit first-party sequences | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Edit method-aligned sequences | ❌ | ❌ | ❌ | ❌ | ✅ (with approval) | ✅ |
| Submit for review | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Review sequences | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Publish custom sequences | ❌ | ✅ (own only) | ✅ (org-scoped) | ✅ (org-scoped) | N/A | ✅ |
| Publish first-party sequences | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Archive custom sequences | ❌ | ✅ (own only) | ✅ (org-scoped) | ✅ (org-scoped) | N/A | ✅ |
| Archive first-party sequences | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |
| Import/export sequences | ❌ | ✅ (own) | ✅ (org-scoped) | ✅ (org-scoped) | ✅ | ✅ |
| Clone sequences | ❌ | ✅ (as template) | ✅ (as template) | ✅ (as template) | ✅ | ✅ |

### Permission rules by sequence visibility

#### First-party sequences (LIFE, SOLF, EVAL)
- **Create**: Content-Editor, System-Admin only
- **Edit**: Content-Editor, System-Admin only
- **Publish**: Content-Editor, System-Admin only (requires pedagogical review)
- **Archive**: System-Admin only
- **Clone**: Any authenticated user can clone as a starting point for a custom sequence

#### Method-aligned sequences
- **Create**: Content-Editor (via import or authoring UI), System-Admin
- **Edit**: Content-Editor, System-Admin (requires publisher approval metadata)
- **Publish**: Content-Editor, System-Admin (requires publisher approval)
- **Archive**: System-Admin only (typically on publisher request)
- **Clone**: Teachers can clone within their org; Content-Editors and System-Admins can clone globally

#### Custom sequences (org-scoped)
- **Create**: Teacher, Teacher-Admin, Subscriber-Admin within their org
- **Edit**: Original author OR Teacher-Admin/Subscriber-Admin within the same org OR System-Admin (global)
- **Publish**: Original author OR Teacher-Admin/Subscriber-Admin within the same org (no external review required)
- **Archive**: Original author OR Teacher-Admin/Subscriber-Admin within the same org OR System-Admin
- **Clone**: Within org only, unless System-Admin clones to another org

### Collaboration and delegation

Teachers and admins can invite collaborators to co-author custom sequences:

- **Add collaborator**: Author or org admin can add other users from the same org to the `collaborators[]` list
- **Collaborator permissions**: Collaborators can edit draft, add comments, but cannot publish or delete unless explicitly granted
- **Remove collaborator**: Author or org admin can revoke access at any time

For first-party and method-aligned sequences, collaboration is managed via Content-Editor role assignment by System-Admins.

### Content-Editor role

The Content-Editor role is a specialized role for managing first-party and method-aligned content:

- Assigned by System-Admin only
- Can create, edit, and publish first-party sequences
- Can review and approve method-aligned sequences from publishers
- Participates in pedagogical review workflows
- Cannot access subscriber billing or student PII (data separation)

See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) for full role definitions.

### Audit and compliance

All authoring actions are logged for audit purposes:

- Draft creation and edits: logged with user ID, timestamp, and change summary
- Publishing and archiving: logged with user ID, approval metadata, and version increment
- Permission grants and revocations: logged with granter ID, target user ID, and scope

For audit log structure and retention, see [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md).

## Supporting Documents Referenced

This sequence authoring specification draws from the following source documents:

- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence authoring specifications and workflow requirements
- [MLC SeqCreate 2020.xlsx - ALPHA LIST.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20ALPHA%20LIST%20.csv) — Sequence organization and classification schema
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game selection and organization for sequence building
- [EXE specs 2012-0913 - Category Screen.csv](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Category%20Screen.csv) — Category browsing for game selection

## Dependencies

The sequence authoring tool integrates with multiple systems and relies on several foundational specs:

### Core data models
- [Sequence Model](sequence-model.md) - Canonical sequence data structure, versioning, and progression rules
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game and stage definitions for step configuration
- [Learning Elements Overview](../07-content-architecture/learning-elements-overview.md) - Content type taxonomy and relationships
- [Assignment Model](../07-content-architecture/assignment-model.md) - How assignments are delivered to students

### Content and registry systems
- [Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md) - Game availability and health checks
- [Games Metadata](../08-games-registry-and-authoring/games-metadata.md) - Game attributes for search and selection in authoring UI
- [Games Stages Policy](../08-games-registry-and-authoring/games-stages-policy.md) - Stage lifecycle rules and deprecation handling
- [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md) - Asset availability validation
- [Games Audit and Health](../08-games-registry-and-authoring/games-audit-and-health.md) - Content health monitoring for validation
- [Game Catalog](../07-content-architecture/game-catalog.md) - Searchable game inventory

### Sequence management
- [Sequence Libraries](sequence-libraries.md) - Sequence catalog, discovery, and lifecycle management
- [Sequence Alignment to Methods](sequence-alignment-to-methods.md) - Method book correlation patterns and metadata
- [Curriculum Mapping Tools](curriculum-mapping-tools.md) - Tools for mapping sequences to external curricula
- [Sequence Exports](sequence-exports.md) - Export formats and use cases

### Roles and permissions
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions and capabilities
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permission definitions
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Authentication context

### Assignment and delivery
- [Assignment Generation](../10-assignments-engine/assignment-generation.md) - How sequences are transformed into student assignments
- [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md) - Student progression through sequences
- [Retention Review Logic](../10-assignments-engine/retention-review-logic.md) - Review scheduling implementation

### UX and design standards
- [Components Overview](../01-ux-design-system/components-overview.md) - UI component library for authoring interface
- [Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md) - Layout adaptation patterns
- [WCAG Compliance](../16-accessibility-and-inclusion/wcag-compliance.md) - Accessibility standards
- [Screen Text and Copy](../01-ux-design-system/screen-text-and-copy.md) - Instructional text guidelines

### Telemetry and monitoring
- [Event Model](../15-analytics-and-reporting/event-model.md) - Canonical telemetry event definitions
- [Observability](../19-quality-and-operations/observability.md) - System monitoring and alerting
- [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) - Audit trail structure

### Import and export
- [CSV Specifications](../20-imports-and-automation/csv-specifications.md) - CSV import/export schema
- [Bulk Operations Jobs](../20-imports-and-automation/bulk-ops-jobs.md) - Background job processing for imports

### Search and indexing
- [Search Indexing](../11-search-and-discovery/search-indexing.md) - How sequences are indexed for game/element search in authoring UI
- [Filters and Facets](../11-search-and-discovery/filters-and-facets.md) - Filtering patterns for game selector

### Pedagogy and compliance
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) - Educational philosophy guiding sequence design
- [Piano Adventures Alignment](../07-content-architecture/piano-adventures-alignment.md) - Example method correlation
- [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md) - Legal compliance considerations

### Optional integrations
- [AI Sequence Generator](ai-sequence-generator.md) - AI-assisted authoring (currently on backburner)

## Open questions

Outstanding decisions to make before build:

### Authoring workflow
- **Autosave frequency**: How often should drafts autosave? Balance between data safety and server load (current thinking: every 30 seconds of inactivity)
- **Undo/redo**: Should the authoring tool support undo/redo for all actions, or only for structural changes (add/remove/reorder)?
- **Concurrent editing**: If multiple collaborators edit the same draft simultaneously, how do we handle conflicts? (Options: lock sections being edited, real-time collaborative editing like Google Docs, or last-write-wins with conflict warnings)
- **Draft expiration**: How long should unpublished drafts be retained before cleanup? (Current thinking: 90 days of inactivity)

### Validation and testing
- **Validation timing**: Should validation run on every keystroke (live), on field blur, or only on save/publish?
- **Play test scope**: Should authors be able to play test entire assignments in sequence, or only individual games?
- **Preview student context**: Should preview mode simulate specific student proficiency levels or gating states?
- **Bulk validation**: For method-aligned sequences with 100+ assignments, should validation be async with progress bar?

### Import and export
- **CSV format version**: Support multiple CSV schema versions for backward compatibility, or require migration to latest format?
- **Import conflict resolution**: When importing a sequence that conflicts with an existing `sequence_id`, should we auto-increment version, prompt user, or fail?
- **Export scope**: Should export include only the sequence structure, or also embed game metadata for offline reference?
- **Method book import**: Should we support direct import from publisher XML/JSON formats, or require conversion to MLC CSV schema first?

### Collaboration
- **Comment notifications**: How should authors be notified of new review comments? Email, in-app notification, both?
- **Role-based diff visibility**: Should Teachers see full diffs when migrating, or simplified "what changed" summaries?
- **Cross-org cloning**: Should System-Admins be able to clone custom sequences from one org to another (e.g., for replication or troubleshooting)?

### Performance and scale
- **Sequence size limits**: Maximum number of groups, assignments, or steps per sequence to maintain UI responsiveness?
- **Lazy loading**: Should the authoring tree lazy-load assignments and steps for very large sequences (100+ assignments)?
- **Search indexing delay**: How quickly should newly added games appear in the authoring game selector (real-time, 1-minute delay, 5-minute delay)?

### Content health and maintenance
- **Deprecated game warnings**: Should authors be blocked from adding deprecated games to new sequences, or only warned?
- **Automatic game replacement**: If a game is replaced (`replaced_by_game_id` is set), should the authoring tool offer one-click replacement across all draft sequences?
- **Historical sequence migration**: Should there be tools to bulk-migrate old sequence formats (from MLC-4.0 CSV) to the new schema?

### User experience
- **Keyboard shortcuts**: Which authoring actions warrant keyboard shortcuts? (Current candidates: save, undo/redo, add step, reorder, preview)
- **Mobile authoring**: Should sequence authoring be supported on tablets (iPad, Android tablets), or remain desktop-only?
- **Template library**: Should there be a curated library of sequence templates (e.g., "Beginner solfege", "Method book unit"), or rely on cloning existing sequences?
- **Step templates**: Should authors be able to create reusable step templates (e.g., "Standard Learn-Play-Quiz pattern") that insert multiple steps at once?

### Pedagogical guidance
- **Smart suggestions**: Should the authoring tool suggest games based on concept progression (e.g., "This assignment lacks rhythm games, consider adding...")?
- **Target score hints**: Should target scores auto-populate based on historical student performance data, or remain manual entry?
- **Review scheduling recommendations**: Should the tool suggest optimal `review_offset_days` based on concept difficulty and spaced repetition research?

### Integration and extensibility
- **API for programmatic authoring**: Should there be a REST API for creating/editing sequences, or is UI-only sufficient?
- **Webhook notifications**: Should sequence publication trigger webhooks for integration with external systems (e.g., notify LMS when new sequences are available)?
- **Third-party plugins**: Should the authoring tool support plugins or extensions (e.g., custom validators, import formats)?
