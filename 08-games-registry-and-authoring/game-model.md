# Game Model

## Purpose
Define the canonical data structures and rules for a Game and its Stages so the registry, authoring tools, player, sequences, reporting, and AI features work against one source of truth. This spec ensures every Game teaches a single concept in measured steps and can be safely versioned and maintained at scale.

> **Related Documents**: For pedagogical principles underlying game design, see [Pedagogy Principles](../00-foundations/pedagogy-principles.md). For legacy system context and historical game structure, see [Legacy Systems](../00-foundations/legacy-systems.md). For sequence composition and assignment generation, see [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md).

## Scope
**Included**
- Game object model and its Stage children
- Lifecycle states and validation
- Scoring, targets, and mastery semantics
- Concept tagging and metadata used by search and AI sequencing
- Asset requirements and health signals
- Minimal UX requirements that the player shell must honor
- Telemetry events and permissions

**Excluded**
- Player runtime mechanics for individual games (covered by runtime-specific docs)
- Sequence composition and assignment generation logic
- Full asset pipeline implementation details
- Billing, pricing, or plan logic

## Data model (if applicable)
### Entities
- **Game**
  - Teaches a single concept or tightly related concept set
  - Owns child **Stages** for Learn, Play, Quiz, optional Challenge, and Review
- **Stage**
  - A playable unit with its own target score, attempts, and result record
  - Review is distinct and records a separate score from Quiz
  - Challenge is optional and competitive

### Identifiers and versioning
- `game_id` string  
  Stable canonical identifier. Example `G-01542`. Follows legacy pattern from original system where games were numbered sequentially (GAM-0010, GAM-0020, etc.) and now use simplified format for better readability.
- `game_version` integer  
  Increments on breaking changes to game logic or content. Maintains backward compatibility with existing assignments and sequences.
- `stage_id` string  
  Derived from game_id and stage type. Example `G-01542-quiz`. Legacy system used numeric suffixes (1=Learn, 2=Play, 3=Quiz, 4=Challenge, 5=Review).
- `stage_version` integer  
  Increments on breaking changes to the stage. Independent versioning allows stages to evolve at different rates.
- `slug` kebab case string for URLs and search. Human-readable identifier for web URLs and search indexing.

### Core fields
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "slug": "grand-staff-guide-notes-a",
  "name": "Grand Staff Guide Notes A",
  "status": "live",           // draft | live | deprecated
  "replaced_by_game_id": null,
  "instrument": "piano",      // piano | general_music | choir | ...
  "mode": "visual",           // visual | aural | mixed
  "device_profile": "any",    // any | midi_required | mic_optional | mic_required
  "keyboard_mode": "non_keyboard", // keyboard | non_keyboard
  "level": "beginner",        // beginner | early_intermediate | ...
  "difficulty": 2,            // 1..10 scale for ranking
  "concept_tags": ["guide_notes", "treble_staff", "bass_staff"],
  "category": "Staff",
  "standards": ["custom:mlc:L1.staff.guide_notes"],
  "pedagogical_notes": "Teaches spatial relationships between guide notes and staff lines, building foundation for note reading",

  "stages": [
    {
      "stage_id": "G-01542-learn",
      "stage_type": "learn",
      "stage_version": 2,
      "target_score": null,   // learn and play may not require a target
      "pass_threshold": null,
      "time_limit_seconds": null,
      "enabled": true
    },
    {
      "stage_id": "G-01542-play",
      "stage_type": "play",
      "stage_version": 2,
      "target_score": 70,
      "pass_threshold": 60,
      "time_limit_seconds": null,
      "enabled": true
    },
    {
      "stage_id": "G-01542-quiz",
      "stage_type": "quiz",
      "stage_version": 3,
      "target_score": 85,
      "pass_threshold": 80,
      "time_limit_seconds": 180,
      "enabled": true
    },
    {
      "stage_id": "G-01542-challenge",
      "stage_type": "challenge",
      "stage_version": 1,
      "target_score": null,
      "pass_threshold": null,
      "time_limit_seconds": 90,
      "enabled": false          // challenge is optional
    },
    {
      "stage_id": "G-01542-review",
      "stage_type": "review",
      "stage_version": 1,
      "target_score": 85,
      "pass_threshold": 80,
      "time_limit_seconds": 180,
      "enabled": true
    }
  ],

  "assets": {
    "html5_bundle_url": "https://cdn.mlc/games/G-01542/v3/index.html",
    "thumbnail_url": "https://cdn.mlc/thumbs/G-01542.png",
    "audio_sprite_url": "https://cdn.mlc/audio/G-01542-sprite.mp3",
    "midi_supported": true,
    "mic_supported": false,
    "asset_requirements": {
      "minimum_resolution": "1024x768",
      "audio_formats": ["mp3", "ogg"],
      "bundle_size_limit_mb": 50,
      "loading_timeout_seconds": 30
    }
  },

  "health": {
    "last_qc_pass": "2025-09-20T12:01:00Z",
    "browser_support": ["chromium_114plus", "safari_17plus", "ios_17plus", "android_13plus"],
    "known_limitations": ["safari_low_latency_midi_off"],
    "degraded_modes": ["fallback_keyboard_input"],
    "deprecated_on": null
  },

  "telemetry_flags": {
    "emit_per_note": false,
    "emit_per_prompt": true
  }
}
```

### Relationships

* A **Game** can be referenced by many **Sequences**, **Assignments**, and **Challenge Boards**.
* A **Stage** writes many **Score** records per student over time.
* Deprecation uses `replaced_by_game_id` for automatic migration helpers.

## Behavior and rules

### Stage set

* Valid types: `learn`, `play`, `quiz`, `challenge`, `review`.
* A Game must include `learn`, `play`, and `quiz`. `challenge` is optional. `review` is required for spaced retention and records a separate score.
* Review is not a pointer to Quiz history. It is a distinct Stage with its own target and pass threshold.

#### Stage Behavior Patterns
Based on the original system design, each stage follows specific pedagogical patterns:

- **Learn Stage**: Interactive tutorial mode with guided exploration. No scoring pressure, focuses on concept introduction and understanding. Students can repeat sections without penalty.
- **Play Stage**: Practice mode with immediate feedback and encouragement. Scoring is motivational rather than evaluative. Students can retry as needed to build confidence.
- **Quiz Stage**: Formal assessment with clear mastery criteria. Limited attempts encourage focused effort. Score determines progression eligibility.
- **Challenge Stage**: Advanced practice with competitive elements. Optional participation, higher difficulty, unlimited scoring potential for engagement.
- **Review Stage**: Spaced repetition for long-term retention. Scheduled after time delay, separate from original Quiz performance.

### Mastery

* A student achieves mastery for the Game when the `quiz` pass_threshold is met or exceeded.
* Review is scheduled later by the assignments engine. Passing Review strengthens retention metrics but does not change the original Quiz mastery timestamp.

### Gating and sequencing

* Default gate: Quiz is locked until Learn and Play have been attempted at least once. This default can be overridden at assignment level.
* Challenge is never required for progression.
* Review is scheduled by sequence rules with `review_offset_days` or by adaptive logic.

### Score semantics

* Each Stage completion writes a Score record with fields: `student_id`, `stage_id`, `score`, `max_score`, `passed`, `duration_ms`, `attempts`, `device_profile`, `input_mode`.
* Store best score per stage for summary views. Keep full attempt history for analytics.

#### Score Interpretation Guidelines
Based on the original system's scoring philosophy:

- **Learn Stage**: No formal scoring, completion tracking only. Focus on engagement and understanding.
- **Play Stage**: Practice scores for motivation. Typically 0-100 scale, no mastery requirement.
- **Quiz Stage**: Mastery scores with clear thresholds. Original system used 60-80% as typical mastery targets.
- **Challenge Stage**: Advanced scores, often unlimited ceiling for competitive engagement.
- **Review Stage**: Retention scores, typically lower thresholds (70-75%) acknowledging time delay factor.

### Lifecycle

* `draft`
  Visible to SysAdmin and Content Editors only. Not assignable.
* `live`
  Searchable, assignable, and visible in All Games if not locked by teacher settings.
* `deprecated`
  Hidden from new assignments. Still playable for history. Sequences referencing a deprecated Game surface a health warning and a suggested replacement if `replaced_by_game_id` is set.

### Validation

* A live Game must have

  * At least Learn, Play, Quiz defined and enabled
  * Assets bundle reachable and passing smoke checks
  * Target and pass_threshold on Play, Quiz, Review if enabled
  * Concept tags not empty
* A Stage cannot be enabled without its asset slice available in the bundle.

### Device and input rules

* If `device_profile = midi_required`, the player must gate entry until MIDI is paired or offer a teacher-configurable fallback if defined.
* Mic-required games must expose mic permission prompts and fallbacks.

## UX requirements (if applicable)

* Player must hide global chrome during Stage play and restore it on exit.
* On exit confirm from Quiz or Review if there is an unsubmitted run.
* Provide consistent result panels

  * Score, pass or try again, next suggested action
  * Clear CTA hierarchy: Retry, Next Stage, Return to Assignment
* Accessibility

  * All interactive controls must be keyboard reachable
  * Announce stage transitions and result summaries to screen readers
  * Reduced motion respect at the Stage level

### Legacy UX Considerations
Based on the original system's user experience patterns:

- **Two-tier selection**: Students first select Game, then Stage within that Game. This matches the original "All Games" redesign requirements.
- **Full-screen gameplay**: Games should occupy entire viewport during play, removing header/footer chrome for immersive experience.
- **Clear progression indicators**: Students need to understand their progress through Learn → Play → Quiz → Review sequence.
- **Immediate feedback**: Each stage provides appropriate feedback level - gentle guidance in Learn, encouragement in Play, clear results in Quiz.

## Telemetry

### Event list

* `game_stage_start`
  Properties: `game_id`, `stage_id`, `assignment_id?`, `sequence_id?`, `device_profile`, `input_mode`
* `game_stage_complete`
  Properties: plus `score`, `max_score`, `passed`, `duration_ms`, `mistake_count?`
* `game_stage_quit`
  Properties: `elapsed_ms`, `reason` values `user_exit|timeout|error`
* `game_stage_error`
  Properties: `error_code`, `context`, `asset_url?`
* `game_asset_loaded`
  Properties: `bundle_version`, `latency_ms`, `cache_hit`
* `midi_pair_attempt` and `midi_pair_result`
  Properties: `device_name`, `success`, `latency_ms`
* `score_submit`
  Mirrors completion with server acceptance status
* `target_met`
  Fires when pass_threshold is first achieved for the Stage
* `review_scheduled`
  Properties: `sequence_id`, `offset_days`

### Sampling

* Gameplay completions are never sampled. Asset and load events may be sampled to 10 percent in production.

## Permissions

* **Read**

  * Everyone after login. Teachers may hide Games from All Games at class level.
* **Create and edit**

  * SysAdmin and Content Editors only.
* **Publish to live**

  * SysAdmin and Content Editors with publish permission.
* **Deprecate or replace**

  * SysAdmin only.
* **Bulk import**

  * SysAdmin and Content Editors with import permission.
* **Override target scores for a class**

  * Teacher or Teacher-Admin at assignment or class policy level. Core registry targets remain unchanged.

## Dependencies

* Player shell spec for layout, navigation, and exit behavior
* Games registry overview for catalog, usage, and health checks
* Sequence model for review scheduling and progression
* Assignment generation for gating overrides and next-up logic
* Search and discovery for indexing fields and facets
* Telemetry event model for canonical names and properties
* CSV import specs for bulk population from the Games Loaded sheet

### Integration Points
- **Assignment Play**: See [Assignment Play](../03-student-experience/assignment-play.md) for student-facing gameplay experience
- **Games Registry**: See [Games Registry Overview](games-registry-overview.md) for administrative management
- **Sequence Model**: See [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) for curriculum integration
- **Content Architecture**: See [Game Taxonomy](../07-content-architecture/game-taxonomy.md) for content categorization

## Open questions

* Do we allow per-student target overrides or only per-class and per-assignment
* Default spacing between Quiz and Review in days for first-party sequences
* Minimum hardware requirements and hard blocks for mic or MIDI required games
* Should Challenge results ever contribute to mastery badges or remain cosmetic
* Policy for deprecating a live Game referenced by many sequences
  Options: soft warnings only vs auto-swap when a replacement exists
