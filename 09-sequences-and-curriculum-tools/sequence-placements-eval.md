# Sequence Placements and EVAL

## Purpose
Define how students are placed into appropriate learning sequences using the EVAL diagnostic sequence and adaptive assessment logic. This spec ensures that new students begin at the right skill level, existing students can be re-evaluated when needed, and the system provides clear recommendations for which sequence and level best matches a student's demonstrated competencies.

For the underlying sequence structure and data model, see [Sequence Model](sequence-model.md). For the full sequence catalog, see [Sequence Libraries](sequence-libraries.md).

## Scope

### Included
- EVAL sequence structure and diagnostic coverage
- Placement testing workflows and triggers
- Scoring and skill proficiency calculations
- Sequence recommendation logic based on placement results
- Re-evaluation scenarios and timing
- Placement override capabilities for teachers
- Student experience during placement testing
- Reporting placement results to teachers and admins
- Integration with assignment generation for post-placement curriculum

### Excluded
- General sequence data model (see [Sequence Model](sequence-model.md))
- Sequence authoring tools (see [Sequence Authoring](sequence-authoring.md))
- Full method-aligned sequence details (see [Sequence Alignment to Methods](sequence-alignment-to-methods.md))
- Detailed assignment delivery mechanics (see [Assignment Generation](../10-assignments-engine/assignment-generation.md))
- Progress tracking for non-placement assignments (see [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))

## Key concepts

### EVAL sequence
The Evaluation (EVAL) sequence is a first-party diagnostic curriculum designed to assess student competency across primary music skill areas. Unlike LIFE or method-aligned sequences that are designed for sequential learning, EVAL focuses on rapid assessment to determine appropriate starting points.

### Placement zones
Based on EVAL performance, students are categorized into placement zones that map to entry points in the LIFE sequence or method-aligned curricula:
- **Primary Level 1A** - Beginning students with minimal or no prior music training
- **Level 1** - Basic note reading and rhythm recognition established
- **Level 2** - Pentascale fluency, interval recognition, basic chord awareness
- **Level 3** - Key signature understanding, scale work, melodic dictation
- **Level 4** - Advanced intervals, triads, inversions, ledger line fluency
- **Level 5** - Complex rhythms, circle of fifths, harmonic progressions, multiple key signatures

### Skill domains assessed
The EVAL sequence evaluates competency across these primary skill areas:
- **A. Chords & Harmony** - Triad quality, inversions, harmonic progressions
- **B. Intervals** - Visual and aural interval identification (2nds through octaves)
- **C. Scales & Key Signatures** - Pentascales, Major/minor scales, key signature recognition
- **D. Pitch & Melody** - Staff reading, melodic patterns, dictation
- **E. Rhythm** - Note values, rhythm patterns, meter, dictation
- **F. Keyboard Elements** - Key identification, finger numbers, keyboard geography
- **G. Music Terms & Symbols** - Notation symbols, terminology, dynamics
- **H. Tonal Memory & Playback** - Aural pattern recognition and reproduction

## Data model (if applicable)

### PlacementTest entity
```json
{
  "placement_test_id": "PT-2024-0542",
  "student_id": "STU-12345",
  "test_type": "initial" | "re_evaluation" | "teacher_requested",
  "started_at": "2024-01-15T14:30:00Z",
  "completed_at": "2024-01-15T15:12:00Z",
  "status": "not_started" | "in_progress" | "completed" | "abandoned",
  "eval_sequence_version": 3,
  "skill_domain_results": [
    {
      "domain": "pitch_melody",
      "games_attempted": 6,
      "games_passed": 4,
      "avg_score": 78,
      "proficiency_level": "level_2"
    }
  ],
  "overall_placement": {
    "recommended_sequence": "SEQ-LIFE",
    "recommended_starting_level": "level_2",
    "recommended_group_id": "G-02",
    "confidence_score": 0.85,
    "reasoning": "Strong pitch and rhythm skills; needs reinforcement on intervals and key signatures"
  },
  "teacher_override": {
    "applied": false,
    "override_level": null,
    "reason": null,
    "overridden_by": null,
    "overridden_at": null
  }
}
```

### Placement result summary
```json
{
  "placement_summary_id": "PS-2024-0542",
  "placement_test_id": "PT-2024-0542",
  "student_id": "STU-12345",
  "skill_domain_breakdown": {
    "chords_harmony": { "level": 1, "strength": "weak" },
    "intervals": { "level": 2, "strength": "moderate" },
    "scales_keys": { "level": 1, "strength": "weak" },
    "pitch_melody": { "level": 2, "strength": "strong" },
    "rhythm": { "level": 2, "strength": "strong" },
    "keyboard_elements": { "level": 1, "strength": "moderate" },
    "music_symbols": { "level": 1, "strength": "moderate" },
    "tonal_memory": { "level": 2, "strength": "strong" }
  },
  "next_steps": [
    "Assign LIFE Level 2 starting with Group G-02 Assignment A-01",
    "Consider supplemental interval recognition practice",
    "Monitor progress on scales and key signatures"
  ]
}
```

## Behavior and rules

### Placement test structure

#### EVAL sequence organization
Based on legacy system data, the EVAL sequence contains:
- Multiple levels (Primary 1A through Level 5)
- Within each level, skill domains are tested via targeted games
- Each game typically appears in both LEARN and QUIZ stages
- LEARN stage introduces the concept and mechanics
- QUIZ stage assesses mastery without hints or guidance

**Example from EVAL sequence**:
```
Level: Primary Level 1A
Domain: D. Pitch & Melody
- GAM-1970-1E (Melody Pix 2) LEARN stage
- GAM-1970-3E (Melody Pix 2) QUIZ stage
- GAM-3480-1E (Songbirds High and Low) LEARN stage
- GAM-3480-3E (Songbirds High and Low) QUIZ stage
```

#### Adaptive assessment flow
1. **Entry point determination**: Student begins at Primary Level 1A or at a teacher-specified starting level if prior experience is documented
2. **Domain sampling**: System presents representative games from each skill domain at the current level
3. **Advancement logic**: 
   - If student passes 80% or more of QUIZ stages at current level, advance to next level
   - If student struggles (below 60% pass rate), remain at current level and complete remaining assessments
   - System can skip LEARN stages if student demonstrates immediate competency
4. **Termination criteria**: Testing ends when student fails to advance two consecutive levels, or when all levels are completed

### Scoring and proficiency calculation

#### Per-game scoring
- QUIZ stage scores are the primary data point (LEARN stages are informational only)
- Target scores from game metadata define "pass" threshold (typically 80-85%)
- Games track: `attempts`, `best_score`, `time_to_complete`, `completion_status`

#### Domain proficiency aggregation
For each skill domain at each level:
- **Pass rate**: Percentage of QUIZ stages passed at target threshold
- **Average score**: Mean score across all QUIZ attempts in that domain
- **Proficiency level**: Assigned based on pass rate
  - **Weak** (0-59%): Significant gaps in understanding
  - **Moderate** (60-79%): Partial mastery, needs reinforcement
  - **Strong** (80-100%): Demonstrated competency at this level

#### Overall placement recommendation
The system calculates a recommended starting point by:
1. Identifying the highest level where the student achieved "moderate" or "strong" proficiency across at least 6 of 8 skill domains
2. Calculating a confidence score based on consistency of performance
3. Providing reasoning text summarizing strengths and gaps
4. Suggesting a specific Group and Assignment within the recommended sequence

**Example recommendation**:
> "Student demonstrates strong rhythm and pitch skills at Level 2, with moderate keyboard element recognition. Intervals and key signature understanding are still developing. Recommended start: LIFE Level 2, Group G-02. Consider supplemental interval practice games in free play mode."

### Placement test triggers

#### Initial placement
- **When**: New student account created, no prior assignment history
- **Trigger**: Automatic prompt on first login or teacher-initiated during onboarding
- **Flow**: Student or teacher selects "Take Placement Test" → EVAL sequence assigned → Results inform first curriculum assignment

#### Re-evaluation placement
- **When**: Student has been working in a sequence but teacher suspects misalignment (too easy or too hard)
- **Trigger**: Teacher manually requests re-evaluation from student dashboard
- **Flow**: Current assignments paused → EVAL sequence assigned → Results compared to current placement → Teacher can adjust based on new data

#### Periodic re-assessment
- **When**: Configurable interval (e.g., every 6 months or after completing a major group)
- **Trigger**: System-generated notification to teacher recommending re-evaluation
- **Flow**: Optional, teacher decides whether to assign EVAL or continue with current sequence

#### Transfer student placement
- **When**: Student transfers from another music program or school
- **Trigger**: Teacher marks student as "transfer" and initiates placement test
- **Flow**: Full EVAL sequence or partial (starting at estimated level) based on teacher input

### Teacher override capabilities

Teachers can override placement recommendations in the following scenarios:
1. **Adjust starting level**: Move student up or down one level from recommended placement
2. **Skip placement test**: Manually assign sequence and level based on known student history (e.g., continuing student from MLC-4.0)
3. **Partial placement**: Assign EVAL for specific skill domains only if most competencies are known
4. **Post-placement adjustment**: Review results and modify assignment within first week if initial placement proves incorrect

Override actions are logged with reasoning for audit and reporting purposes.

## UX requirements (if applicable)

### Student placement test experience

#### Pre-test screen
- Clear explanation: "This helps us find the best starting point for you"
- Estimated time: "20-30 minutes"
- Instructions: "Try your best on each game. It's okay if some are challenging!"
- Option to pause and resume later
- Visual progress indicator showing domains and levels

#### During test
- Minimal navigation chrome to reduce distraction
- Clear "Next" button after each game completes
- No scores displayed during test to reduce anxiety
- Ability to pause and return later (saves progress)
- Encouraging messages between domains: "Great work! Moving to rhythm games next."

#### Results screen for students
- Friendly summary: "You're ready to start at Level 2!"
- Visual skill radar chart showing strengths
- Encouraging tone: "You're strong in rhythm and pitch reading. We'll help you build interval skills as you go."
- "Start Learning" button to begin first assignment

### Teacher placement test management

#### Initiate placement
- Accessible from student profile or class roster
- Form: Test type (initial, re-evaluation, transfer), optional starting level hint
- Assign to single student or bulk assign to multiple students
- Option to set deadline or leave open-ended

#### Monitor in-progress tests
- Dashboard view: List of students with active placement tests
- Status indicators: Not started, In progress (with % complete), Completed, Abandoned
- Ability to send reminder or nudge students who haven't started

#### Review results
- Detailed results page with:
  - Overall recommendation (sequence, level, group)
  - Per-domain proficiency breakdown with bar charts
  - Game-by-game scores table (expandable detail)
  - System reasoning for recommendation
- "Apply Recommendation" button to auto-assign suggested sequence
- "Override Placement" option with dropdown to select different level and required reasoning text field
- Comparison view if this is a re-evaluation (show change from prior placement)

### Admin placement analytics

Admins can view aggregated placement data:
- Distribution of student placements across levels
- Average time to complete placement test
- Most common skill gaps identified
- Placement accuracy metrics (% of students who needed adjustment within first month)
- Trends over time (are new students arriving better/worse prepared)

See [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md) for detailed reporting specifications.

## Telemetry

### Event names and key properties

**Placement test lifecycle events**:
- `placement_test_started`
  - `placement_test_id`, `student_id`, `test_type`, `started_from` (teacher_dashboard | student_onboarding | system_prompt)
- `placement_test_game_complete`
  - `placement_test_id`, `game_id`, `stage`, `score`, `passed`, `duration_ms`
- `placement_test_paused`
  - `placement_test_id`, `percent_complete`, `current_level`, `current_domain`
- `placement_test_resumed`
  - `placement_test_id`, `time_since_pause_ms`
- `placement_test_completed`
  - `placement_test_id`, `total_duration_ms`, `games_attempted`, `overall_proficiency`, `recommended_level`
- `placement_test_abandoned`
  - `placement_test_id`, `percent_complete`, `time_since_last_activity_ms`

**Recommendation and override events**:
- `placement_recommendation_generated`
  - `placement_test_id`, `recommended_sequence`, `recommended_level`, `confidence_score`, `reasoning_summary`
- `placement_recommendation_applied`
  - `placement_test_id`, `applied_by` (teacher_id), `sequence_id`, `starting_group_id`
- `placement_override_applied`
  - `placement_test_id`, `teacher_id`, `original_recommendation`, `override_level`, `reason`
- `placement_adjustment_made`
  - `placement_test_id`, `student_id`, `original_level`, `adjusted_level`, `days_since_placement`, `reason`

### Sampling
- All placement test lifecycle events: **100%** (never sampled, critical for analytics)
- Game-level completion events within placement: **100%** (needed for recommendation accuracy)
- Recommendation and override events: **100%** (audit trail)

## Permissions

### Read placement data
- **Students**: Can view their own placement test results and skill summary
- **Teachers**: Can view placement results for any student in their assigned classes
- **Teacher-Admins**: Can view placement results for any student in their organization
- **Subscriber-Admins**: Can view placement results for any student in their subscription/school
- **System-Admins**: Can view all placement data across the platform

### Initiate placement test
- **Teachers**: Can assign placement tests to students in their classes
- **Teacher-Admins**: Can assign placement tests to any student in their organization
- **Subscriber-Admins**: Can assign placement tests to any student in their subscription
- **Students**: Can self-initiate placement test if explicitly enabled by their teacher or admin

### Override placement recommendations
- **Teachers**: Can override recommendations for students in their classes (logged)
- **Teacher-Admins**: Can override recommendations for any student in their organization
- **Subscriber-Admins**: Can override recommendations for any student in their subscription
- **System-Admins**: Can override any placement recommendation (typically for support/troubleshooting)

### Modify EVAL sequence content
- **Content-Editors**: Can edit the EVAL sequence structure and game selections
- **System-Admins**: Can publish new versions of the EVAL sequence
- **Teachers and Admins**: Read-only access; cannot modify core EVAL sequence (can request changes via support)

See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for complete permission definitions.

## Dependencies

### Core sequence and content systems
- [Sequence Model](sequence-model.md) - Data structure for EVAL sequence, versioning, and step definitions
- [Sequence Libraries](sequence-libraries.md) - EVAL sequence as a first-party library, visibility rules
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game stages, scoring, and target thresholds
- [Games Registry](../08-games-registry-and-authoring/games-registry-overview.md) - Game metadata and health status

### Assignment and progression
- [Assignment Generation](../10-assignments-engine/assignment-generation.md) - Creating first assignment post-placement
- [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md) - Monitoring if initial placement was accurate

### Reporting and analytics
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry schema and event definitions
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Placement accuracy and student progress dashboards
- [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md) - Organization-wide placement analytics

### User experience
- [Student Dashboard](../03-student-experience/student-dashboard.md) - Entry point for students to start placement test
- [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) - Placement test assignment and monitoring
- [Student Assignments](../03-student-experience/student-assignments.md) - How EVAL appears in assignment list

### Permissions and roles
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role capabilities for placement management
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permissions for placement actions

## Open questions

### Placement accuracy and validation
- **Question**: How do we measure placement accuracy over time? What metrics indicate a placement was correct vs. needs adjustment?
- **Proposed approach**: Track percentage of students who required level adjustment within first 30 days of initial placement. Target: <15% adjustment rate.

### Adaptive vs. fixed-length testing
- **Question**: Should EVAL always test all levels and domains, or adaptively end early when proficiency ceiling is reached?
- **Tradeoff**: Adaptive saves time but may miss specific skill gaps. Fixed-length provides comprehensive baseline.
- **Proposed hybrid**: Adaptive termination after two consecutive levels fail to advance, but always complete at least one full level of assessment.

### Re-evaluation frequency
- **Question**: How often should re-evaluation be recommended? What triggers indicate a student has outgrown their current placement?
- **Factors to consider**: Completion velocity, consistent high scores (>90% across multiple assignments), teacher observation
- **Proposed**: System flags for teacher review when student completes 80% of current level assignments with >90% average score

### Placement test fatigue
- **Question**: Is 20-30 minutes too long for initial placement? Will students disengage?
- **Mitigation options**: 
  - Allow pause/resume with progress saved
  - Provide shorter "quick placement" option (10-15 minutes, fewer domains, lower confidence)
  - Break into multiple sessions (one domain per session over several days)

### Partial placement for transfer students
- **Question**: If a teacher knows a transfer student is "around Level 3" but wants to confirm, can we start EVAL at Level 3 instead of Primary 1A?
- **Proposed**: Yes, allow teacher to specify starting level hint when assigning placement test. System begins assessment at that level and adjusts up or down based on performance.

### Skill domain weighting
- **Question**: Should all skill domains be weighted equally in placement recommendation, or should certain domains (e.g., rhythm, pitch reading) have higher priority?
- **Proposed**: Equal weighting by default, but allow teacher preferences to prioritize specific domains (e.g., a teacher focused on sight-reading might weight pitch/melody higher).

### Placement for specialized paths
- **Question**: How does placement work for students following method-aligned sequences (e.g., Piano Adventures) vs. LIFE?
- **Proposed**: EVAL results map to equivalent starting points in both LIFE and major method-aligned sequences. Teacher selects curriculum post-placement.

### Confidence thresholds for recommendations
- **Question**: What confidence score is acceptable for automatic placement vs. requiring teacher review?
- **Proposed**: Confidence >0.80 allows auto-apply. Confidence 0.60-0.79 shows warning and suggests teacher review. Confidence <0.60 requires manual teacher placement decision.
