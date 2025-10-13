# Games Stages Policy

## Purpose

Define the governance rules and lifecycle policies for Game Stages so content editors, system administrators, and the platform itself can safely manage, enable, disable, modify, and retire stages while protecting student progress, maintaining pedagogical integrity, and preventing breaking changes to live curriculum.

This spec ensures that:
- Required stages (Learn, Play, Quiz) are always present and properly configured before a game goes live
- Optional stages (Challenge, Review) follow clear enablement and usage policies
- Stage parameter changes (targets, thresholds, time limits) are controlled and versioned appropriately
- Students can complete assignments without encountering disabled or misconfigured stages
- Teachers understand which stages are active and how they contribute to mastery assessment
- The platform can enforce validation rules at creation, publication, and update time

> **Related Documents**: For the data model and stage field definitions, see [Game Model](game-model.md). For CRUD operations and versioning mechanics, see [Games CRUD](games-crud.md). For catalog and administrative workflows, see [Games Registry Overview](games-registry-overview.md). For pedagogical principles underlying the five-stage design, see [Pedagogy Principles](../00-foundations/pedagogy-principles.md).

## Scope

**Included**
- Required vs optional stage policies
- Stage enablement and disablement rules
- Validation policies for draft vs live games
- Target score and pass threshold change policies
- Review stage scheduling and retention policies
- Challenge stage participation and competitive scoring policies
- Stage versioning and breaking change rules
- Impact analysis requirements before stage modifications
- Grandfathering policies for students mid-assignment

**Excluded**
- Stage data model schema definitions (see [Game Model](game-model.md))
- CRUD API endpoints and transaction mechanics (see [Games CRUD](games-crud.md))
- Asset upload and bundle management (see [Games Assets Pipeline](games-assets-pipeline.md))
- Player runtime mechanics and UI (see [Assignment Play](../03-student-experience/assignment-play.md) and [Game Player Shell](../03-student-experience/game-player-shell.md))
- Sequence composition and assignment generation logic (see [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) and [Assignment Generation](../10-assignments-engine/assignment-generation.md))
- Scoring algorithms and attempt tracking within individual games

## Data model (if applicable)

### Stage Types and Required Status

Each Game owns a collection of child **Stage** entities. The five stage types follow the original MLC system design:

| Stage Type | Required | Purpose | Legacy Number |
|------------|----------|---------|---------------|
| `learn` | Yes | Interactive tutorial with guided exploration, no scoring pressure | 1 |
| `play` | Yes | Practice mode with immediate feedback and encouragement | 2 |
| `quiz` | Yes | Formal assessment with mastery criteria | 3 |
| `challenge` | No | Advanced competitive practice with unlimited scoring potential | 4 |
| `review` | Recommended | Spaced repetition for long-term retention | 5 |

### Policy-Relevant Fields

**Stage-Level Fields**
- `stage_type` enum: `learn`, `play`, `quiz`, `challenge`, `review` (immutable after creation)
- `enabled` boolean: Whether the stage is active and playable (mutable, policy-controlled)
- `target_score` integer or null: Suggested score for completion (policy: required for play, quiz, review if enabled)
- `pass_threshold` integer or null: Minimum score for mastery (policy: required for quiz, review if enabled)
- `time_limit_seconds` integer or null: Maximum duration for timed stages (policy: optional, typical for quiz and challenge)
- `stage_version` integer: Increments on breaking changes to stage parameters

**Game-Level Fields**
- `status` enum: `draft`, `live`, `deprecated` (policies enforce different validation rules per status)
- `game_version` integer: Increments when stage configuration changes affect existing assignments

### Relationships

- A Game must have at least 3 child Stages (`learn`, `play`, `quiz`) to transition from `draft` to `live`.
- A Game may have up to 5 child Stages if `challenge` and `review` are also present.
- Each Stage writes many **Score** records per student over time, which are preserved even if the Stage is later disabled.
- Review Stages schedule **Review Assignments** via the assignments engine, referencing the original Quiz mastery timestamp.

## Behavior and rules

### Required vs Optional Stages

**Required Stages Policy**
- Every Game must define `learn`, `play`, and `quiz` stages at creation.
- These three stages must have `enabled = true` before the Game can be published to `live` status.
- Disabling a required stage on a live Game triggers a validation error unless the Game status is first changed to `draft` or `deprecated`.
- Required stages must have valid configuration:
  - `play` must have `target_score` and `pass_threshold` defined
  - `quiz` must have `target_score` and `pass_threshold` defined
  - `learn` may have null targets (completion-based, not score-based)

**Optional Stages Policy**
- `challenge` stage is fully optional. A Game may omit the Challenge stage entirely or create it with `enabled = false`.
- Challenge stage does not contribute to mastery assessment and is never required for assignment progression.
- Challenge may have null `target_score` and `pass_threshold` since its purpose is competitive engagement with unlimited scoring potential.
- `review` stage is technically optional but strongly recommended for pedagogical reasons (spaced retention).
- Review stage must have `target_score` and `pass_threshold` if `enabled = true`.
- Review is always scheduled separately from the original Quiz and records a separate score.

### Stage Enablement and Disablement Rules

**Enabling a Stage**
- To enable a previously disabled stage:
  1. If the Game is `live`, first create a draft version via the update workflow (see [Games CRUD](games-crud.md#versioning-strategy)).
  2. Ensure the stage has all required fields populated: `target_score`, `pass_threshold` for play/quiz/review types.
  3. Ensure the stage's asset slice is available in the game bundle (asset validation check).
  4. Publish the updated Game version.
  5. Assignments referencing the old version continue using the old configuration; new assignments use the new configuration.

**Disabling a Stage**
- To disable a currently enabled stage:
  1. If disabling a required stage (`learn`, `play`, `quiz`), the Game status must first be changed to `draft` or `deprecated`. A live Game cannot have required stages disabled.
  2. If disabling an optional stage (`challenge`, `review`), this is allowed on live Games without status change.
  3. Set `enabled = false`.
  4. Increment `stage_version` since this is a breaking change.
  5. Existing student scores for the disabled stage remain in the system and visible in history.
  6. Students currently mid-assignment when stage is disabled continue to see the stage (grandfathering policy, see below).
  7. New assignments created after the change do not include the disabled stage.

**Impact Analysis Required**
- Before disabling any stage, the registry must display:
  - Number of active assignments currently including this stage
  - Number of students who have attempted but not completed this stage in the last 30 days
  - Number of sequences referencing this game (to assess curriculum impact)
- A warning message: "Disabling this stage will affect N active assignments. Students currently working on this stage will be grandfathered, but new assignments will exclude it. Proceed?"

### Stage Parameter Modification Policies

**Target Score and Pass Threshold Changes**
- Changing `target_score` or `pass_threshold` on a live Game is a **breaking change** and requires:
  1. Creating a new draft version via the update workflow.
  2. Incrementing `stage_version` for the modified stage.
  3. Optionally incrementing `game_version` if the change is significant enough to warrant game-level versioning.
  4. Publishing the new version.
- Rationale: Changing thresholds mid-assignment would alter mastery criteria and invalidate existing student progress metrics.

**Grandfathering Policy**
- Students who began an assignment referencing a specific game version continue using that version's stage configuration until they complete or abandon the assignment.
- New assignments created after a stage configuration change automatically use the latest published game version.
- Teachers may explicitly choose an older game version for new assignments if needed (version pinning).

**Non-Breaking Changes**
- Changes that do not affect student scoring or mastery assessment are non-breaking:
  - Updating `pedagogical_notes` (metadata only)
  - Changing asset URLs (bundle updates with same behavior)
  - Modifying health signals or browser support tags
- Non-breaking changes do not increment `stage_version` or require draft/publish workflow.

### Review Stage Scheduling Policy

**Review Stage Purpose**
- The Review stage implements spaced repetition for long-term retention.
- Review is not a pointer to Quiz history; it is a distinct Stage with its own target, threshold, and separate score record.
- Review is typically scheduled N days after Quiz mastery (see [Assignment Generation](../10-assignments-engine/assignment-generation.md) and [Retention Review Logic](../10-assignments-engine/retention-review-logic.md)).

**Review Enablement Policy**
- If a Game has `review` stage with `enabled = true`, the assignments engine schedules Review assignments based on:
  - Sequence-level `review_offset_days` configuration (if present)
  - Adaptive logic based on student performance patterns (future AI enhancement)
  - Default global setting (e.g., 7 days) if no sequence override exists
- If `review` stage is `enabled = false` or omitted, no Review assignments are generated.

**Review Parameter Recommendations**
- Review `pass_threshold` should typically be 5-10 percentage points lower than Quiz `pass_threshold` to account for time delay and forgetting curve.
- Review `time_limit_seconds` should match Quiz time limit unless pedagogical reasons suggest otherwise.
- Example: If Quiz has `pass_threshold = 80`, Review should have `pass_threshold = 75`.

### Challenge Stage Participation Policy

**Challenge Stage Purpose**
- Challenge provides advanced practice with competitive elements and leaderboard integration (see [Leaderboards](../03-student-experience/leaderboards.md) and [Challenge Boards](../12-gamification/challenge-boards.md)).
- Challenge is always optional; students can skip it without affecting mastery status or assignment progression.
- Challenge scores often have no upper limit, encouraging repeated attempts for high scores.

**Challenge Enablement Policy**
- If a Game has `challenge` stage with `enabled = true`, it appears as an optional step in assignment sequences and in the All Games library.
- If `challenge` stage is `enabled = false` or omitted, it does not appear anywhere in student-facing interfaces.
- Challenge does not block access to Review stage or assignment completion.

**Challenge Parameter Recommendations**
- Challenge may have null `target_score` and `pass_threshold` since it is not part of mastery assessment.
- If thresholds are defined, they serve only as motivational benchmarks (e.g., "Try to beat 500 points!").
- Challenge should have a `time_limit_seconds` to create urgency and competitive fairness.

### Validation Policies by Game Status

**Draft Status**
- Minimal validation enforced to allow iterative authoring.
- Required stages (`learn`, `play`, `quiz`) must be present in the stage array, but `enabled` may be false.
- Target scores and thresholds may be null or incomplete.
- Asset URLs may be null or unreachable.
- Concept tags may be empty.
- Draft games are visible only to Content Editors and SysAdmin.

**Live Status (Publish Validation)**
- To transition from `draft` to `live`, full validation enforced:
  - `learn`, `play`, `quiz` stages must exist and have `enabled = true`.
  - `play` stage must have `target_score ≥ 1` and `pass_threshold ≥ 1`.
  - `quiz` stage must have `target_score ≥ 1` and `pass_threshold ≥ 1`.
  - `review` stage, if enabled, must have `target_score ≥ 1` and `pass_threshold ≥ 1`.
  - All enabled stages must have valid asset slices in the bundle.
  - `concept_tags` array cannot be empty (minimum 1 tag).
  - Asset bundle URL must be reachable and pass smoke checks.
  - `thumbnail_url` must be reachable.
- Validation errors block publication and display specific messages to the editor.

**Deprecated Status**
- When a Game is deprecated:
  - All stages remain playable for students with existing assignments (historical access).
  - The Game is hidden from All Games search and sequence authoring tools.
  - New assignments cannot reference the deprecated Game unless explicitly overridden by SysAdmin.
  - If `replaced_by_game_id` is set, authoring tools suggest the replacement game automatically.
- Stages in deprecated games can still be edited (e.g., to fix critical bugs), but standard versioning rules apply.

### Stage Lifecycle State Transitions

```
Draft Game Creation
  ↓
Define Stages (learn, play, quiz required; challenge, review optional)
  ↓
Configure Stage Parameters (targets, thresholds, time limits)
  ↓
Enable Required Stages
  ↓
Publish Validation (enforce full policy rules)
  ↓
Live Game ← → Update (creates draft version) → New Version Published
  ↓
Deprecate (stages remain playable for historical access)
  ↓
Optional: Hard Delete (only if zero play history and zero references)
```

### Edge Cases and Special Situations

**Scenario: Teacher wants to disable Quiz stage for a specific class**
- Policy: Stage enablement is game-level, not class-level. Teachers cannot disable stages per class.
- Workaround: Teacher can create a custom assignment that includes only Learn and Play stages by selecting stages individually if the assignment builder supports stage-level selection (stretch goal).
- Alternative: Clone the Game, disable Quiz stage on the clone, use cloned game for that class.

**Scenario: Content editor wants to lower pass_threshold on a live Quiz stage**
- Policy: This is a breaking change. Editor must create a draft version, modify threshold, increment `stage_version`, and publish.
- Grandfathering: Students with existing assignments continue using old threshold until they complete the assignment.

**Scenario: Student completed Quiz but Review stage is later disabled**
- Policy: Review assignments already generated remain valid and playable. New Review assignments are not generated after disablement.
- Student sees their scheduled Review assignment and can complete it normally.

**Scenario: Challenge stage enabled mid-curriculum**
- Policy: Enabling Challenge on a live Game requires creating a draft version and publishing.
- Impact: Existing assignments do not automatically add Challenge stage (version lock). New assignments include Challenge.
- Teachers can reassign the updated game version if they want their current students to access Challenge.

## UX requirements (if applicable)

### Stage Configuration UI (Content Editor)

**Create/Edit Game Form - Stages Section**
1. **Stage Table**: Display all five stage types in a table with columns:
   - Stage Type (learn, play, quiz, challenge, review) - column header icons for each type
   - Enabled checkbox (required stages show checkbox disabled/checked with tooltip "Required stage, cannot disable on live games")
   - Target Score input field (number, optional for learn and challenge)
   - Pass Threshold input field (number, required for play/quiz/review if enabled)
   - Time Limit input field (number, seconds, optional)
   - Actions: "Edit" button for stage-specific notes or advanced settings

2. **Visual Indicators**:
   - Required stages (learn, play, quiz): Show "Required" badge next to stage type
   - Recommended stages (review): Show "Recommended" badge
   - Optional stages (challenge): Show "Optional" badge
   - Disabled stages: Gray out row and show "Disabled" status

3. **Validation Feedback**:
   - Inline validation on field blur: "Pass threshold cannot exceed target score"
   - Publish-blocking errors highlighted in red: "Quiz stage must have target_score defined"
   - Warnings highlighted in amber: "Review stage is disabled. Consider enabling for spaced retention."

4. **Impact Analysis Panel** (shown when attempting to disable a stage or change thresholds on live game):
   - "Impact Analysis" expandable section
   - Display: "This change will affect:"
     - "12 sequences currently include this game"
     - "45 active assignments in progress"
     - "120 students have scores for this stage in the last 30 days"
   - Links to view affected sequences and assignments
   - Warning message with recommended actions

### Stage Status Display (Registry Catalog)

**Catalog List View - Stages Column**
- Show compact indicator for each game:
  - Example: "L P Q R" (learn, play, quiz, review enabled)
  - Example: "L P Q C R" (all stages enabled)
  - Grayed-out letter for disabled stages: "L P Q R" (challenge disabled)
- Tooltip on hover: "5 stages: Learn, Play, Quiz, Review enabled | Challenge disabled"

**Detail View - Stages Tab**
- Full stage table with all fields visible
- Timeline visualization showing stage sequence: Learn → Play → Quiz → (Challenge) → Review
- Color-coded status: Green (enabled), Gray (disabled), Red (validation error)

### Stage Selection UI (Teacher Assignment Builder)

**Default Behavior**
- When teacher selects a Game for assignment, all enabled stages are included by default.
- Clear visual showing: "This game includes 4 stages: Learn, Play, Quiz, Review"

**Advanced Selection (Stretch Goal)**
- "Customize stages" checkbox allows teacher to select specific stages for this assignment
- Warning: "Selecting only some stages may affect mastery tracking and retention"

### Accessibility

- All stage configuration controls keyboard navigable with logical tab order.
- Required vs optional stage status announced to screen readers: "Quiz stage, required, enabled"
- Validation errors announced immediately when triggered.
- Impact analysis panels use ARIA live regions to announce dynamically updated counts.
- Visual status indicators (colors, badges) always paired with text or icon labels.

## Telemetry

### Stage Lifecycle Events

All events follow schema defined in [Event Model](../15-analytics-and-reporting/event-model.md#event-core-schema).

**Stage Enabled**  
Event: `stage_enabled`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_id": "G-01542-challenge",
  "stage_type": "challenge",
  "editor_id": "usr_12345",
  "was_disabled_duration_days": 45,
  "breaking_change": true
}
```
Fires: When a previously disabled stage is enabled and the change is saved.

**Stage Disabled**  
Event: `stage_disabled`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_id": "G-01542-review",
  "stage_type": "review",
  "editor_id": "usr_12345",
  "impact_sequences_count": 8,
  "impact_assignments_count": 23,
  "impact_students_count": 67,
  "reason": "Low completion rate and student feedback"
}
```
Fires: When an enabled stage is disabled and the change is saved.

**Stage Parameters Modified**  
Event: `stage_parameters_modified`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_id": "G-01542-quiz",
  "stage_type": "quiz",
  "stage_version_before": 2,
  "stage_version_after": 3,
  "editor_id": "usr_12345",
  "fields_changed": ["target_score", "pass_threshold"],
  "old_target_score": 85,
  "new_target_score": 90,
  "old_pass_threshold": 80,
  "new_pass_threshold": 85,
  "breaking_change": true,
  "reason": "Adjusted difficulty based on mastery data"
}
```
Fires: When target_score, pass_threshold, or time_limit_seconds is modified.

**Publish Validation Failed**  
Event: `game_publish_validation_failed`  
Properties:
```json
{
  "game_id": "G-01542",
  "editor_id": "usr_12345",
  "validation_errors": [
    {
      "stage_id": "G-01542-quiz",
      "field": "target_score",
      "error": "Required field missing"
    },
    {
      "stage_id": "G-01542-review",
      "field": "enabled",
      "error": "Review stage enabled but pass_threshold not defined"
    }
  ]
}
```
Fires: When publish attempt fails due to stage policy violations.

**Review Scheduled**  
Event: `review_scheduled`  
Properties:
```json
{
  "assignment_id": "asn_78901",
  "student_id": "stu_45678",
  "game_id": "G-01542",
  "stage_id": "G-01542-review",
  "quiz_completed_at": "2025-01-20T14:30:00Z",
  "review_scheduled_at": "2025-01-27T14:30:00Z",
  "offset_days": 7,
  "sequence_id": "seq_123",
  "sequence_config_offset_days": 7
}
```
Fires: When assignments engine schedules a Review assignment. See [Assignment Generation](../10-assignments-engine/assignment-generation.md#telemetry).

### Sampling

- Stage lifecycle events (enabled, disabled, parameters modified) are **never sampled**. Capture 100% for audit compliance.
- Publish validation failures are never sampled for debugging purposes.
- Review scheduling events may be sampled to 10% in production for performance (sample per student, not per event).

## Permissions

See [Permissions Table](../02-roles-and-permissions/permissions-table.md#content-management-permissions) for detailed matrix.

**View Stage Configuration**:
- Content Editors: ✅ All games
- SysAdmin: ✅ All games
- Teachers: ✅ Live games only (read-only view of stage configuration)
- Students: ❌ No registry access

**Enable/Disable Stages**:
- Content Editors: ✅ All games (subject to policy rules)
- SysAdmin: ✅ All games (subject to policy rules)
- All others: ❌ No

**Modify Stage Parameters** (target_score, pass_threshold, time_limit):
- Content Editors: ✅ All games (creates draft version if live)
- SysAdmin: ✅ All games
- All others: ❌ No

**Override Stage Policy Rules** (e.g., disable required stage on live game):
- SysAdmin: ✅ Yes (with explicit confirmation and audit logging)
- All others: ❌ No (even Content Editors cannot override policy rules)

**Permission Enforcement**: All stage modification operations validate permissions at API layer before applying changes.

## Supporting Documents Referenced

This games stages policy specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Original stage definitions and progression requirements
- [EXE specs 2012-0913 - Assignment Screen.csv](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Assignment%20Screen.csv) — Assignment screen specifications showing stage presentation
- [Assignment Play Re-design.docx.txt](../00-ORIG-CONTEXT/Assignment%20Play%20Re-design.docx.txt) — Stage selection and navigation redesign requirements

## Dependencies

### Internal Dependencies

- **[Game Model](game-model.md)**: Defines stage data schema, field types, and relationships.
- **[Games CRUD](games-crud.md)**: Implements stage enablement, parameter modification, and versioning workflows.
- **[Games Registry Overview](games-registry-overview.md)**: Provides catalog views and publish validation workflows that enforce stage policies.
- **[Assignment Generation](../10-assignments-engine/assignment-generation.md)**: Consumes stage configuration to generate appropriate assignment steps; enforces gating rules based on enabled stages.
- **[Retention Review Logic](../10-assignments-engine/retention-review-logic.md)**: Uses Review stage configuration and offset policies to schedule spaced repetition assignments.
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: Sequences reference games and stages; stage disablement may trigger sequence health warnings.
- **[Assignment Play](../03-student-experience/assignment-play.md)**: Student-facing play surface renders only enabled stages; respects grandfathering policy for version-locked assignments.
- **[Permissions Table](../02-roles-and-permissions/permissions-table.md)**: Defines who can modify stage configuration.
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Stage lifecycle events follow canonical telemetry schema.

### External Dependencies

- **Database Layer**: Stage policy validation rules enforced via database constraints (e.g., foreign key integrity for stage_type enum).
- **Assignments Engine**: Queries stage configuration when generating assignment steps; respects enabled flags and gating rules.
- **Registry API**: Exposes stage policy validation errors via structured error responses.

## Open questions

### Per-Class Stage Configuration Override

- **Question**: Should teachers be able to disable specific stages for their class without creating a custom game clone?  
- **Current Assumption**: No per-class stage overrides. Stage configuration is game-level only.
- **Alternative**: Add class-level or assignment-level stage overrides with clear UI warnings about deviation from standard curriculum.
- **Trade-offs**: Per-class overrides add complexity to assignment generation logic and make it harder to track "what students actually played." Game cloning is more explicit but creates content proliferation.
- **Decision Needed By**: Phase 1 beta (based on teacher feedback during alpha)

### Review Stage Default Scheduling

- **Question**: What is the default number of days between Quiz mastery and Review scheduling if no sequence-level override exists?  
- **Current Assumption**: 7 days (based on spaced repetition research)
- **Alternatives**: 3 days (shorter reinforcement) or 14 days (longer retention test)
- **Decision Needed By**: Phase 1 alpha (requires consultation with pedagogy team)

### Challenge Stage Mastery Contribution

- **Question**: Should exceptional Challenge stage performance ever contribute to mastery status or remain purely cosmetic/motivational?  
- **Current Assumption**: Challenge never contributes to mastery; Quiz is the sole mastery gate.
- **Alternative**: Allow Challenge to grant "advanced mastery" badge or accelerate Review scheduling.
- **Decision Needed By**: Phase 2 (after initial gamification system launch)

### Stage-Level Asset Overrides

- **Question**: Should we allow different asset bundles per stage (e.g., Learn stage uses simplified graphics, Quiz stage uses full complexity)?  
- **Current Assumption**: Single game bundle serves all stages; stages differentiate via internal game logic.
- **Alternative**: Allow per-stage `bundle_url` or `asset_variant` field.
- **Trade-offs**: Per-stage assets increase flexibility but complicate asset pipeline and cache management.
- **Decision Needed By**: Phase 2 (based on content editor feedback)

### Grandfathering Duration Limit

- **Question**: How long should students be grandfathered into an old game version before forced migration to new version?  
- **Current Assumption**: Indefinite grandfathering; students locked to version until assignment completion or abandonment.
- **Alternative**: Force migration after 90 days, or after teacher explicitly approves version upgrade.
- **Trade-offs**: Indefinite grandfathering simplifies logic but may leave students on deprecated versions long-term. Forced migration risks disrupting student progress.
- **Decision Needed By**: Phase 2 (after observing version update patterns in production)

### Stage Validation Strictness Levels

- **Question**: Should we offer "strict" vs "lenient" validation modes for stage configuration?  
- **Current Assumption**: Single validation ruleset enforced at publish time.
- **Alternative**: Allow Content Editors to mark games as "experimental" with relaxed validation for pilot testing.
- **Decision Needed By**: Phase 1 beta (based on content authoring workflow feedback)
