# Assignment Play

## Purpose
Define the student-facing play experience for completing an Assignment that contains a sequence of Learning Elements. Establish the canonical layout, states, scoring behavior, accessibility requirements, and analytics so engineering, design, and content align on one implementation.

## Scope
Included
- Page layout and navigation for Assignment Play
- Stage selection and chrome-less game play
- Score recording rules for Learn, Play, Quiz, Challenge, Review
- Default gates and progression behaviors
- Resilience and edge cases such as refresh, reconnect, or exit
- Accessibility, performance, and telemetry requirements

Excluded
- Game runtime internals and mechanics
- Teacher authoring UI and registry editing
- Billing or role creation flows
- Detailed copy library for every feedback message

## Data model (if applicable)
Key entities referenced by this surface
- Assignment
  - assignment_id, assignment_name, sequence_id, sequence_version, step_count
- Step
  - step_id, element_type game|text|video|audio|reward, element_id, stage when game, gating, targets, scheduling for review
- Game Stage runtime state
  - run_id, started_at, completed_at, score, max_score, passed boolean, duration_ms, attempts
- Player context
  - device_profile, input_mode keyboard|midi|mic|touch, connectivity_state online|offline_lite

Relationships
- Assignment has ordered Steps
- Steps referencing a Game specify one Stage
- Each completed Stage writes a Score record and updates Assignment progress

## Behavior and rules
State machine overview
1. Load Assignment
   - Fetch steps and determine next required step
   - Resolve any free play reconciliation if enabled
2. Stage selection
   - Two tier selector
     - Tier 1: select Game within this Assignment
     - Tier 2: select Stage within the selected Game
   - Locked states reflect gating rules
3. Play state
   - Enter chrome-less mode
   - Handle pause, resume, and quit
4. Result state
   - Persist score then present feedback and CTAs
   - Update progress and determine next step

Gating defaults
- Quiz requires at least one attempt on Learn and Play
- Review requires a prior passing Quiz
- Challenge is optional and never gates progression
- Teachers or policies may override gates per Assignment

Score semantics
- Each Stage completion writes a Score record
  - Learn and Play may write scores if provided by the runtime, otherwise record attempts and duration
  - Quiz writes the mastery score that determines pass or try again
  - Review writes a separate score that does not overwrite Quiz but contributes to retention metrics
  - Challenge writes a competitive score and does not affect mastery gates
- Best score per Stage is used for summary tiles; attempt history is retained for analytics

Free play reconciliation
- If a matching Game Stage has an equal or higher prior score from All Games
  - On first open of that Step, mark complete if policy allows and thresholds are met
  - Otherwise require a fresh attempt

Resilience and edge cases
- Page refresh during play
  - Auto restore to the current Stage with a warning that in-progress runs are not counted until completion
- Offline lite
  - Allow cached assets to run when possible
  - Queue score submission and retry on reconnect
- Quit behavior
  - From Learn or Play: exit immediately and record duration only
  - From Quiz or Review with a completed run that is not yet submitted: confirm before discarding
- Timeouts and errors
  - Surface friendly error panels with Retry and Return to Assignment options
  - Log error codes for support

Progression
- Next Up logic
  - Prefer the next required Step in the current Assignment
  - If all required Steps are complete, show Completion panel and link to next Assignment or Dashboard

## UX requirements (if applicable)
Layout
- Header strip
  - Assignment name, breadcrumb back to Assignments list or Dashboard
  - Next Up button that focuses the next required Step
  - Game ID display for easy reference and support
- Stage selector panel
  - Left column: list of Games in this Assignment with progress pills
  - Right column: Stages for the selected Game with states locked, available, in progress, complete
- Game canvas
  - Full viewport chrome-less player (header and footer removed during play)
  - Show minimal overlay controls for pause, exit, and help
  - Return to Dashboard button for easy navigation
- Result panel
  - Score, pass or try again, suggested next action
  - CTAs: Retry, Next Stage, Return to Assignment
  - Score recording confirmation and progress indicators

Accessibility
- Full keyboard operability for selector and overlays
- Screen reader labels on Step states and result messages
- High contrast compliant color tokens
- Respect reduced motion and sound off preferences
- Focus management when entering and exiting chrome-less play

Performance
- Preload next Stage assets when idle and on stable network
- Target time to interactive less than 2 seconds on modern devices
- Avoid layout shifts when transitioning into or out of chrome-less mode

Copy system
- Result messages keyed by pass, near miss, or miss bands
- Hints unlocked after N failed attempts if teacher allows
- Clear feedback on score recording status
- Progress indicators showing completion status at a glance

## Telemetry
Event names and properties
- assignment_open
  - assignment_id, sequence_id, sequence_version, step_count
- step_start
  - step_id, element_type, element_id, stage when game, gated boolean
- game_stage_start
  - game_id, stage_id, assignment_id, device_profile, input_mode
- game_stage_complete
  - game_id, stage_id, score, max_score, passed, duration_ms, attempts
- score_submit
  - stage_id, accepted boolean, latency_ms, offline_queued boolean
- game_stage_quit
  - stage_id, elapsed_ms, reason user_exit|timeout|error
- step_complete
  - step_id, passed boolean, score if present, duration_ms
- assignment_complete
  - assignment_id, duration_ms, passes, retries
- review_scheduled
  - step_id, review_offset_days, due_at if set
- error_panel_shown
  - code, recoverable boolean

Sampling
- Stage completion and score submissions are never sampled
- Asset load and error events may sample at 10 percent in production

## Permissions
- Student
  - Read and execute assigned Steps
  - Cannot edit targets or gates
- Teacher or Teacher-Admin
  - Can override gates or targets at the Assignment or class policy level
  - Can toggle visibility of optional Steps
- Subscriber-Admin
  - No special powers inside play surface beyond class policies
- SysAdmin
  - View-only diagnostic overlays when impersonating with audit logs

## Pedagogical Approach
The Assignment Play interface implements the core pedagogical principles outlined in [Pedagogy Principles](../00-foundations/pedagogy-principles.md) through structured, progressive learning:

### Stage-Based Learning Progression
Each game follows the established Learn → Play → Quiz → Review sequence:
- **Learn Stage**: Interactive tutorial with guided exploration, no scoring pressure
- **Play Stage**: Practice mode with immediate feedback and encouragement
- **Quiz Stage**: Formal assessment determining mastery and progression eligibility
- **Challenge Stage**: Optional advanced practice with competitive elements
- **Review Stage**: Spaced repetition for long-term retention

### Mastery-Based Progression
- Students must demonstrate mastery (typically 80-85% on Quiz stage) before advancing
- Failed attempts provide learning opportunities rather than punitive consequences
- Teachers can adjust mastery thresholds based on individual student needs
- Review stages ensure long-term retention of mastered concepts

### Adaptive Learning Support
- Free play reconciliation allows students to leverage prior learning
- Remediation steps are automatically inserted when students struggle with concepts
- Progress tracking provides insights for both students and teachers
- Flexible gating allows teachers to customize learning paths

## Supporting Documents Referenced

This assignment play specification draws from the following source documents:

- [Assignment Play Re-design.docx.txt](../00-ORIG-CONTEXT/Assignment%20Play%20Re-design.docx.txt) — Original assignment play interface redesign specifications and workflow
- [EXE specs 2012-0913 (Assignment Screen)](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Assignment%20Screen.csv) — Detailed assignment screen specifications and behaviors
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game architecture and stage progression specifications
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence and assignment structure specifications

## Dependencies
- Game Model for Stage definitions, targets, and lifecycle
- Sequence Model for step composition, gates, and review scheduling
- Games Registry for asset availability and health flags
- Assignments Engine for Next Up logic and policy overrides
- Telemetry Event Model for canonical naming and schemas
- Accessibility standards and design tokens from the design system
- Offline and queueing utilities for deferred score submission

## Integration with All Games Library
The Assignment Play interface integrates with the broader [All Games Library](../03-student-experience/all-games-library.md) through a two-tier selection structure:

### Two-Tier Game Selection
Based on the original system requirements, students follow a structured selection process:
1. **Game Selection**: Students first choose a Game from the Assignment's game list
2. **Stage Selection**: After selecting a Game, students choose the specific Stage (Learn, Play, Quiz, Challenge, Review)
3. **Chrome-less Play**: Games run in full-screen mode with header and footer removed
4. **Return Navigation**: Upon completion, students return to the Stage selection screen

### Free Play Reconciliation
The system reconciles scores from the All Games library with Assignment requirements:
- If a student has already played a game in free play mode with a qualifying score, the Assignment Play interface can mark that stage as complete
- Teachers can configure whether fresh attempts are required regardless of prior free play scores
- This integration ensures students don't repeat work unnecessarily while maintaining assessment integrity

### Score Recording and Display
- All scores are recorded consistently whether played through Assignments or All Games
- Students can see their progress at a glance with clear completion indicators
- Score recording status is clearly communicated to prevent confusion about progress tracking

## Addressing Historical Pain Points
Based on feedback from the original system, the Assignment Play interface specifically addresses these key issues:

### Navigation and User Experience
- **Easy Dashboard Return**: Clear "Return to Dashboard" button provides quick navigation back to check scores and progress
- **Game ID Display**: Game identifiers are prominently displayed for easy reference and support purposes
- **Reduced Visual Clutter**: Header elements are minimized during gameplay to maximize screen real estate for the learning experience

### Progress Visibility
- **At-a-Glance Progress**: Students can see their completion status and scores without navigating away from the assignment
- **Score Recording Confirmation**: Clear feedback ensures students know their progress is being tracked
- **Next Assignment Preview**: "Similar Courses" section shows upcoming assignments in the student's sequence

### Technical Reliability
- **Score Recording Integrity**: Robust score submission ensures no progress is lost due to technical issues
- **Offline Resilience**: Cached content and queued submissions maintain functionality during connectivity issues
- **Error Recovery**: Clear error messages and recovery options prevent students from getting stuck

## Open questions
- Should there be a global policy to always require a fresh attempt even if free play met thresholds
- Do we allow per student target overrides inside the play surface or only at assignment and class levels
- Default handling when a Stage asset bundle is temporarily unreachable
- Minimum network checks before entering chrome-less play for mic or MIDI required games
- Should Challenge results feed any badges or remain purely cosmetic