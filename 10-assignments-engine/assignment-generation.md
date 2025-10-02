# Assignment Generation

## Purpose
Describe how the system converts Sequences into per student Assignments, how Next Up is determined, how review is scheduled, and how policies and free play scores affect generation. This creates a single source of truth for automation, teacher controls, and analytics.

The assignment generation engine addresses critical pain points identified in the original MLC system, including score recording failures, unclear progression indicators, and the need for seamless integration between free play and assigned content. This system ensures reliable, pedagogically sound assignment delivery that maintains student engagement while providing teachers with clear visibility into student progress.

## Scope
Included
- Inputs to assignment generation and policy resolution
- Creation of Assignments and Steps from a Sequence
- Next Up logic, retries, remediation inserts, and review scheduling
- Reconciliation with free play history
- Version pinning and migration behavior
- Error handling and idempotency
- Telemetry and permissions

Excluded
- Game runtime logic
- Student play surface details
- Registry authoring and sequence editing
- Billing or plan checks beyond simple seat validation

## Data model (if applicable)
### Inputs
- `sequence_id`, `sequence_version`  
- `student_id`, `org_id`, `class_id?`, `teacher_id`  
- Policy objects
  - `class_policy`  
    - gates: require_previous_steps boolean, min_attempts defaults, require_fresh_attempt boolean
    - review defaults: default_review_offset_days, review_count, spaced_schedule pattern
    - targets overrides: optional pass thresholds by category or stage
  - `assignment_overrides` at creation time  
    - optional steps, due dates, target overrides
- Historical context  
  - free play scores by `game_id` and `stage_id`
  - prior assignment outcomes for remediation

### Core entities written
- **Assignment**
  - `assignment_id`, `student_id`, `sequence_id`, `sequence_version`, `group_id`, `name`, `status` open|complete|archived, `created_at`, `due_at?`, `policy_snapshot`
- **Step**
  - `step_id`, `assignment_id`, `order_index`, `element_type`, `element_id`, `stage?`, `gating`, `targets`, `scheduling`, `optional` boolean, `state` locked|available|in_progress|complete
- **Remediation Insert** (synthetic step)
  - Same as Step, with `origin` set to `remediation` and `source_step_id` pointer

### Idempotency and version pinning
- The engine pins `sequence_id` and `sequence_version` on the Assignment at creation
- Re running generation with the same input tuple is idempotent based on a deterministic `assignment_key = sha(sequence_id, sequence_version, student_id, group_id)`

## Behavior and rules
### High level flow
1. **Resolve context**  
   Fetch sequence version, class policy, teacher overrides, student history, and device profile flags.
2. **Select scope**  
   Choose the next Group and the next Assignment template from the pinned sequence.
3. **Materialize steps**  
   Expand template Steps into concrete Steps, apply policy and overrides, and set initial state.
4. **Apply free play reconciliation**  
   Optionally mark some Steps complete if prior scores meet gates.
5. **Insert remediation**  
   Insert targeted Steps when prior outcomes indicate a gap.
6. **Schedule review**  
   Create Review steps or future review assignments based on policy.
7. **Persist and publish**  
   Write assignments and steps, emit telemetry, and notify the student queue.

### Historical context and lessons learned
The original MLC system faced several critical issues that this generation engine specifically addresses:

- **Score recording failures**: Historical data showed instances where students completed games but scores weren't recorded, leading to confusion about progression status
- **Unclear "Next Up" indicators**: Students often couldn't easily identify what to work on next, especially when mixing assigned and free play content
- **Inconsistent progression logic**: The original system lacked robust gating mechanisms, sometimes allowing students to skip foundational concepts
- **Teacher visibility gaps**: Educators needed better tools to understand why students were stuck or progressing slowly

This generation engine incorporates these lessons by ensuring reliable score reconciliation, clear progression indicators, and comprehensive teacher visibility into the assignment generation process.

### Next Up selection
- Within a Group, Next Up is the earliest required Step with `state != complete`
- If all required Steps in the current Assignment are complete, mark the Assignment complete and either
  - advance to the next Assignment in the same Group, or
  - if no more assignments in the Group, advance to the first assignment in the next Group

### State derivation for Steps
- `locked`  
  Gates unmet based on `require_previous_steps`, `min_attempts`, or dependencies
- `available`  
  Gates satisfied and ready to start
- `in_progress`  
  Student has started the step but not completed
- `complete`  
  Step met pass criteria or was accepted by policy via reconciliation

### Default gates
- Quiz requires at least one attempt on Learn and Play for the same Game
- Review requires a passing Quiz for the same Game
- Challenge is optional and never gates progression

### Targets resolution order
1. Assignment override
2. Class policy by category or stage
3. Registry defaults on the Stage

### Free play reconciliation
- If `class_policy.require_fresh_attempt = false` and historical best score for the same `stage_id` meets `pass_threshold`
  - The Step is marked complete on generation
  - The UI still allows replay without changing completion state unless a better score is achieved
- If `require_fresh_attempt = true`  
  - The Step is set to available regardless of prior scores

**Pedagogical rationale**: This reconciliation system addresses the common scenario where students discover and play games through the "All Games" library before encountering them in their assigned sequence. The original MLC system struggled with this integration, sometimes requiring students to re-play content they had already mastered or losing credit for work done outside assignments. This approach maintains pedagogical integrity while respecting student agency and prior learning.

### Remediation inserts
- Trigger conditions  
  - Recent Quiz fails in related concepts  
  - Time on task spikes without mastery  
  - Teacher flag for remediation required
- Inserted content  
  - One or more Learn or Play steps mapped by concept tags to address the gap  
  - Steps are marked required and placed before the blocked Quiz or at the start of the next Assignment
- Limits  
  - Maximum of N remediation steps per assignment, default 2

### Review scheduling
- Default offset from Quiz pass is 7 days
- `review_count` default is 1. A spaced schedule of [7, 21] days can be configured to create multiple Review steps
- Review steps are distinct and record separate scores

**Retention strategy**: The review system implements spaced repetition principles that were central to the original MLC pedagogy. Historical data showed that students who engaged with review content had significantly better long-term retention of musical concepts. The 7-day initial review and 21-day follow-up schedule aligns with established memory consolidation research while fitting within typical lesson planning cycles.

### Deprecated or missing content
- If a Step references a disabled or deprecated Stage
  - Attempt soft swap to `replaced_by_game_id` if present
  - If no replacement exists, mark Step with a content warning and set `optional = true` unless teacher opts to block progression
  - Emit a `content_health_warning` telemetry event

### Idempotent creation
- The engine derives a deterministic `assignment_key`. If an identical request is received, return the existing Assignment without duplication.

### Migration behavior
- When a sequence version updates
  - New assignments use the latest version by default
  - Existing assignments remain pinned until teacher migrates
- Migration assistant
  - Present a diff: added, removed, replaced Steps
  - Safe auto swap when a Stage has a defined replacement
  - Preserve completed Step outcomes where element_id and stage are unchanged

### Pseudocode for generation
```pseudo
function generate_assignment(student_id, class_id, sequence_id, sequence_version):
  ctx = resolve_context(student_id, class_id, sequence_id, sequence_version)
  tpl = next_assignment_template(ctx.sequence, ctx.progress)
  steps = []
  for tstep in tpl.steps:
    step = materialize_step(tstep, ctx.policy, ctx.overrides)
    step.state = derive_initial_state(step, ctx.history, ctx.policy)
    steps.append(step)

  steps = apply_free_play_reconciliation(steps, ctx.history, ctx.policy)
  steps = insert_remediation(steps, ctx.history, ctx.policy)
  steps = schedule_reviews(steps, ctx.policy)

  assignment = persist_assignment(student_id, tpl, steps, ctx)
  emit('assignment_generated', assignment.summary)
  return assignment
```

## UX requirements (if applicable)

* Teacher create flow

  * Choose sequence and target student or class
  * Optionally set due date, optional steps, and overrides
  * Preview materialized steps with gates and any pre completed steps via reconciliation
* Student view

  * Clear Next Up button that jumps to the next required Step
  * Indicators for optional steps and review due labels
* Error states

  * If content is missing, show a teacher note and safe path forward
* Accessibility

  * All configuration controls keyboard reachable
  * Status messages for locked, available, complete are screen reader friendly

### Student experience enhancements
Based on feedback from the original system redesign, the assignment generation engine must support:

- **Clear progression indicators**: Students need immediate visual feedback about their current position in the sequence and what comes next
- **Score visibility**: The system should display current scores for completed steps, addressing the original issue where "scores are not recording" caused student confusion
- **Flexible navigation**: Students should be able to easily return to the dashboard or check scores without losing their place in the assignment
- **Game ID display**: Students and teachers need to see game identifiers for easy reference and troubleshooting

These requirements directly address the pain points identified in the "Assignment Play Re-design" and "All Games Redesign" documentation from the original system.

## Telemetry

Key events and properties

* `assignment_generated`

  * `assignment_id`, `student_id`, `sequence_id`, `sequence_version`, `step_count`, `precompleted_count`
* `free_play_reconciled`

  * `assignment_id`, `step_id`, `stage_id`, `source` all_games, `score`, `threshold`
* `remediation_inserted`

  * `assignment_id`, `step_id`, `concept_tags[]`, `origin_step_id?`
* `review_scheduled`

  * `assignment_id`, `step_id`, `offset_days`
* `content_health_warning`

  * `assignment_id`, `step_id`, `issue` deprecated|disabled|asset_missing, `replacement?`
* `assignment_migrated`

  * `assignment_id`, `from_version`, `to_version`, `autoswaps`, `manual_edits`
* `assignment_publish_failed`

  * `assignment_key`, `error_code`, `message`

Sampling

* Generation, reconciliation, remediation, and migration events are never sampled

## Permissions

* Teacher or Teacher Admin

  * Generate assignments for students in their classes
  * Set overrides and due dates within policy bounds
  * Migrate pinned assignments to new sequence versions
* Subscriber Admin

  * Can generate at organization level and set class default policies
* Student

  * Read only, cannot change targets, gates, or due dates
* SysAdmin

  * Full access for support and diagnostics with audit logs

## Dependencies

* [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) for templates, gates, and stage semantics
* [Game Model](../08-games-registry-and-authoring/game-model.md) for stage validity, targets, and replacements
* [Games Registry](../08-games-registry-and-authoring/games-registry-overview.md) for health checks and deprecation info
* [Assignment Play](../03-student-experience/assignment-play.md) for score semantics and Next Up consistency
* [CSV Import Specs](../20-imports-and-automation/csv-specifications.md) for bulk assignment creation
* [Event Model](../15-analytics-and-reporting/event-model.md) for canonical telemetry names and property types
* [Roles and Permissions](../02-roles-and-permissions/roles-matrix.md) for teacher and admin access controls
* [Student Dashboard](../03-student-experience/student-dashboard.md) for Next Up display and navigation
* [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) for assignment creation and monitoring

## Integration with MLC ecosystem

The assignment generation engine serves as the central coordination point for several key MLC systems:

- **Sequence Management**: Integrates with the [Lifetime Musician Curriculum](../07-content-architecture/piano-adventures-alignment.md) and method-specific sequences to ensure proper pedagogical progression
- **Content Discovery**: Works with the [All Games Library](../03-student-experience/all-games-library.md) to reconcile free play with assigned content
- **Progress Tracking**: Feeds data to [Teacher Reports](../04-teacher-experience/teacher-reports.md) and [Analytics](../15-analytics-and-reporting/learning-analytics.md) for comprehensive progress monitoring
- **Accessibility**: Supports [Neurodiverse Learning](../16-accessibility-and-inclusion/neurodiverse-support.md) through flexible pacing and remediation options

## Open questions

* Should free play reconciliation be on by default or off by default for schools
* Do we allow per student target overrides at generation time or only per assignment and class
* What is the hard cap on remediation inserts per assignment
* Do we allow review steps to create separate micro assignments rather than being embedded steps
* Migration behavior when an assignment has started but not completed
* How should the system handle the transition from the original MLC-4.0 sequence structure to the new flexible assignment model
* What level of teacher control should be provided over automatic remediation insertion
