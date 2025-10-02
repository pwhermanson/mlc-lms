# Sequence Model

## Purpose
Define how Learning Elements are organized and delivered as Sequences that power Assignments, adaptive progress, reporting, and AI generation. This spec creates one canonical model so Students, Teachers, Admins, the Assignments Engine, and the AI Sequence Generator all operate against the same structure.

Note that AI generator feature is on hold until further notice. This is on the backburner.

## Scope
Included
- Sequence, Group, Assignment, and Step data models
- First party sequences and method aligned sequences
- Gating, progression, mastery, and Review scheduling rules
- Versioning, pinning, and compatibility when content changes
- Interaction with free play scores
- Minimal UX requirements for assignment consumption
- Telemetry and permissions

Excluded
- Game runtime mechanics
- Teacher reports visualization
- Billing and plan logic
- CSV schema details for import and export

## Key concepts
- Learning Element - atomic unit: Game stage, Text, Video, Audio, Reward.
- Game - a concept-focused set of stages: Learn, Play, Quiz, Challenge, Review. Review is tracked separately. Not all Games include Challenge.
- Assignment - an ordered set of Learning Elements delivered to the student.
- Group - a set of Assignments within a Sequence.
- Sequence - an ordered set of Groups aligned to a curriculum or method book.

## Primary first-party sequences
- LIFE - Lifetime Musician curriculum.
- SOLF - Solfege sequence.
- EVAL - Evaluation sequence for placement.

## Method-aligned sequences
- Method books are represented as Sequences that mirror the book structure and correlate each step to specific Games and stages. For example, the Hal Leonard Student Piano Library correlations enumerate Units, pages, category, game, and the exact skills taught, plus target scores per stage. See Book A to 5 extracts for concrete patterns.

### Method alignment metadata requirements
Method-aligned sequences require standardized metadata to ensure compatibility across publishers and enable automated import/export workflows:

- **Book identification**: `publisher`, `series_name`, `book_level`, `edition_year`
- **Page correlation**: `unit_number`, `page_range`, `lesson_title`
- **Skill mapping**: `concept_tags[]`, `difficulty_tier`, `prerequisite_skills[]`
- **Progression markers**: `milestone_assignments[]`, `assessment_points[]`

This metadata enables the [AI Sequence Generator](ai-sequence-generator.md) to automatically create method-aligned sequences from publisher content and ensures consistent structure across different curriculum series.

## Data model (if applicable)

### Entities and relationships
- **Sequence**
  - Ordered curriculum pathway made of **Groups**
  - Variants
    - First party sequences: LIFE, SOLF, EVAL
    - Method aligned sequences: mirror book structures by level, unit, page
    - Custom teacher sequences: organization scoped versions
- **Group**
  - Logical chapter or unit inside a Sequence
  - Ordered list of **Assignments**
- **Assignment**
  - Student facing bundle of Steps for a practice session
  - Steps reference Learning Elements
- **Step**
  - One Learning Element reference with rules for gating and targets
  - If the element is a Game, Step specifies the Stage that must be played

### Identifiers and versioning
- `sequence_id` string. Stable canonical id, scoped globally
- `sequence_version` integer. Increments on breaking curriculum changes
- `group_id`, `assignment_id`, `step_id` strings. Stable within a sequence_version
- `visibility` enum. `first_party` | `method_aligned` | `custom`
- `org_id` optional. Present when a custom sequence is organization scoped
- Pinning
  - Assignments store `sequence_id` and `sequence_version` used at creation time
  - Reports and replays resolve against the pinned version for grading stability

### Core payload example
```json
{
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
    "completion_criteria": "all_assignments_mastered"
  },
  "groups": [
    {
      "group_id": "G-01",
      "name": "Foundations",
      "assignments": [
        {
          "assignment_id": "A-01",
          "name": "Guide Notes and Beat",
          "steps": [
            {
              "step_id": "S-01",
              "element_type": "game",
              "element_id": "G-01542",
              "stage": "learn",
              "gating": { "min_attempts": 1 },
              "targets": null,
              "notes": "Introduce staff guide notes"
            },
            {
              "step_id": "S-02",
              "element_type": "game",
              "element_id": "G-01542",
              "stage": "play",
              "gating": { "min_attempts": 1 },
              "targets": { "target_score": 70, "pass_threshold": 60 }
            },
            {
              "step_id": "S-03",
              "element_type": "game",
              "element_id": "G-01542",
              "stage": "quiz",
              "gating": { "require_previous_steps": true },
              "targets": { "target_score": 85, "pass_threshold": 80 }
            },
            {
              "step_id": "S-04",
              "element_type": "game",
              "element_id": "G-01542",
              "stage": "review",
              "scheduling": { "review_offset_days": 7 },
              "targets": { "target_score": 85, "pass_threshold": 80 }
            },
            {
              "step_id": "S-05",
              "element_type": "text",
              "element_id": "TXT-0301",
              "notes": "Practice checklist and posture tips"
            }
          ]
        }
      ]
    }
  ]
}
```

### Step fields

* `element_type` enum. `game` | `text` | `video` | `audio` | `reward`
* `element_id` string. Canonical id from the content registry
* `stage` enum when `element_type = game`. `learn` | `play` | `quiz` | `challenge` | `review`
* `gating` object

  * `require_previous_steps` boolean. Default true for Quiz and Review
  * `min_attempts` integer per step
  * `min_score_from_prior` number. Optional gate using a prior step result
* `targets` object for game steps

  * `target_score` number
  * `pass_threshold` number
* `scheduling` object

  * `review_offset_days` integer for Review steps
  * `due_at` optional timestamp used for calendar views
* `notes` free text rationale

## Behavior and rules

### Stage semantics inside sequences

* Learn and Play introduce and reinforce the concept
* Quiz establishes mastery for that Game. Passing Quiz marks the concept mastered
* Review is a distinct step with its own score record and is scheduled later for retention
* Challenge is optional and never gates progression

### Progression and completion

* Assignment completion occurs when all required steps are completed and gated targets are met
* Sequence progression moves to the next Assignment. Next up logic defaults to the earliest incomplete required step in the current Assignment
* Teachers can mark an Assignment optional. Optional steps are excluded from completion gates but still record scores

### Default gates and overrides

* Default

  * Quiz requires at least one attempt on Learn and Play
  * Review requires Quiz passed
* Overrides

  * Teachers can relax gates per Assignment. Example allow direct Quiz for quick checks
  * Admin policies can enforce stricter global gates for a class or school

### Review scheduling

* Default `review_offset_days` is 7. Configurable at step or policy level
* Multiple Reviews may be scheduled for spaced repetition. Later reviews can increase the offset incrementally
* Reviews never overwrite the original Quiz mastery timestamp. They update retention metrics

#### Advanced scheduling patterns
Based on the original system's pedagogical approach, different sequence types support specialized scheduling:

- **First-party sequences (LIFE, SOLF)**: Standard 7-day review offset with adaptive spacing based on performance
- **Method-aligned sequences**: Variable scheduling based on book pacing (typically 3-14 days)
- **Evaluation sequences**: Immediate review scheduling for rapid assessment cycles
- **Custom sequences**: Teacher-configurable patterns including daily, weekly, or milestone-based reviews

See [Assignment Generation](../10-assignments-engine/assignment-generation.md) for detailed scheduling implementation and policy resolution.

### Free play reconciliation

* If a student achieves a passing score in free play for the same Game and Stage before encountering it in a Sequence

  * The first time the Step is opened inside the Assignment it reads the best historical score and may mark as complete if it meets the gate
  * Teachers can require a fresh attempt even if free play passed. Policy flag `require_fresh_attempt` at class or assignment level

### Missing or deprecated content

* If a referenced Game Stage is disabled or deprecated

  * The Assignments Engine flags the Step with a content warning
  * If `replaced_by_game_id` is defined, the engine can auto suggest a swap while preserving targets and gating
  * Sequences remain playable but highlight issues in authoring and admin health views

### Sequence health monitoring

Sequences include health indicators to ensure content quality and system reliability:

- **Content validation**: All referenced games and stages must exist and be in `live` status
- **Asset availability**: Required game bundles and media assets must be accessible
- **Performance metrics**: Track completion rates, average scores, and time-to-mastery per sequence
- **Deprecation warnings**: Alert when referenced content is marked for replacement
- **Usage analytics**: Monitor sequence adoption, completion rates, and teacher feedback

Health status affects sequence visibility and assignment generation. See [Games Registry Health](../08-games-registry-and-authoring/games-audit-and-health.md) for detailed health monitoring implementation.

### Versioning and pinning

* Assignments pin `sequence_id` and `sequence_version` at creation time
* When a sequence is updated

  * New assignments use the latest version
  * Existing assignments continue using the pinned version until teacher opts to migrate
* Migration path

  * The system shows diffs and allows a one click migrate that updates steps where safe and preserves scores

## UX requirements (if applicable)

* Student view always shows

  * Assignment name, step list with states: locked, in progress, complete, optional
  * Clear next action button and expected outcome text
  * For Review steps show the due in label when scheduled
* Teacher authoring view must provide

  * Reorder groups, assignments, steps with drag and drop
  * Per step gating and targets editor
  * Health indicators for missing or deprecated elements
* Accessibility

  * Keyboard reachable step navigation and reorder handles
  * Screen reader friendly status text for gates and completion

## Telemetry

Event names and key properties

* `sequence_assigned`

  * `sequence_id`, `sequence_version`, `assignment_count`, `student_count`, `org_id`
* `assignment_open`

  * `assignment_id`, `sequence_id`, `step_count`
* `step_start`

  * `step_id`, `element_type`, `element_id`, `stage?`, `gated` boolean
* `step_complete`

  * `step_id`, `passed` boolean, `score?`, `max_score?`, `duration_ms`
* `review_scheduled`

  * `step_id`, `review_offset_days`, `due_at?`
* `assignment_complete`

  * `assignment_id`, `duration_ms`, `passes`, `retries`
* `sequence_progress`

  * `sequence_id`, `percent_complete`, `last_assignment_id`
* `sequence_version_migrated`

  * `from_version`, `to_version`, `auto_swaps`, `manual_edits`

Sampling

* Completion events are never sampled. Authoring edits and migration events may be sampled to 10 percent in production

## Permissions

* Read

  * Students can read only their assigned sequences and public All Games library
  * Teachers and Admins can read first party and enabled method aligned libraries
* Create and edit

  * Teachers can create organization scoped custom sequences
  * Content editors can modify first party sequences
* Publish

  * Content editors and SysAdmin publish sequence versions
* Migrate

  * Teachers can migrate pinned assignments to a later sequence_version within their org
* Delete or retire

  * SysAdmin only for first party sequences. Teachers can retire their custom sequences

## Dependencies

* [Game Model](../08-games-registry-and-authoring/game-model.md) for stage definitions and target semantics
* [Assignment Generation](../10-assignments-engine/assignment-generation.md) for next up logic, gating enforcement, and Review scheduling
* [Games Registry](../08-games-registry-and-authoring/games-registry-overview.md) for content health and replacement mapping
* [Search and Discovery](../11-search-and-discovery/search-indexing.md) for indexing sequence names, concepts, and method metadata
* [AI Sequence Generator](ai-sequence-generator.md) for method ingestion and proposal workflows
* [CSV Import and Export](../20-imports-and-automation/csv-specifications.md) for bulk sequence operations
* [Event Model](../15-analytics-and-reporting/event-model.md) for canonical telemetry names and property definitions
* [Learning Elements Overview](../07-content-architecture/learning-elements-overview.md) for content type definitions and relationships

## Open questions

* Global defaults

  * Should we standardize default `review_offset_days` by concept difficulty tiers rather than one global value
* Per student targets

  * Do we allow per student target overrides or limit overrides to class and assignment levels
* Optional steps

  * Should optional steps count toward badges or only toward practice minutes
* Free play policy

  * Default behavior when free play score meets gate. Require a fresh attempt or auto complete
* Method alignment metadata

  * Standardize a minimal schema for level, unit, page, and skill statements to guarantee importer compatibility across publishers
* Sequence complexity limits

  * Maximum number of groups, assignments, and steps per sequence to ensure performance and usability
* Cross-sequence dependencies

  * Support for sequences that build upon other sequences (e.g., SOLF building on LIFE foundations)
* Adaptive sequencing

  * Dynamic adjustment of sequence content based on student performance patterns and learning analytics
