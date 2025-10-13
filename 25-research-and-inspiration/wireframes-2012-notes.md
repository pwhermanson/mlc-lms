# Wireframes 2012 Notes

This document captures insights and design decisions from the MLC Learning UI wireframes dated December 2012, along with related redesign documents and executable specifications from that period.

## Purpose

The 2012 wireframes represent the foundational UX thinking for MLC 4.0, including detailed screen layouts, information architecture, and interaction patterns. This historical documentation informs MLC 5.0 by showing:

- **What worked well** and should be preserved or enhanced
- **Pain points** that required redesigns and iterations
- **Core pedagogical flows** that remain central to the platform
- **System admin and teacher workflows** that shaped role-based permissions

These materials provide critical context for understanding why certain features exist and how they evolved.

**Primary Source:** [MLC_LearningUI_wireframes_201210.pdf](../00-ORIG-CONTEXT/MLC_LearningUI_wireframes_201210.pdf) — visual wireframes from December 2012 (contains embedded images)

## Scope

### Included
- **All Games Page** redesign requirements and two-tier navigation structure
- **Assignment Play Screen** specifications including pre/main/post assignment panes
- **System Admin Panel** requirements for member management and filtering
- **Game Categorization System** (EXE specs from September 2012)
- **Student Assignment Screen** with scoring and progress tracking
- **Content categorization** using concept parameters (BASE, STAGE, RHYTHM, MELODY, etc.)
- **Screen text and copy** for landing pages, pricing, role descriptions

**Supporting Documents Referenced:**
- [All Games Redesign.docx.txt](../00-ORIG-CONTEXT/All%20Games%20Redesign.docx.txt)
- [Assignment Play Re-design.docx.txt](../00-ORIG-CONTEXT/Assignment%20Play%20Re-design.docx.txt)
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt)
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt)
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt)
- [EXE specs 2012-0913 (Assignment Screen)](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Assignment%20Screen.csv)
- [EXE specs 2012-0913 (Game Screen)](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Game%20Screen.csv)
- [EXE specs 2012-0913 (Category Screen)](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Category%20Screen.csv)

### Excluded
- Actual wireframe images (PDF is binary, not directly readable)
- Implementation-specific technical details from the 2012 codebase
- Deprecated features that are not part of MLC 5.0 scope

See also:
- [Product Vision](../00-foundations/product-vision.md) — how 2012 principles inform current direction
- [Legacy Systems](../00-foundations/legacy-systems.md) — MLC 4.0 architecture context
- [Information Architecture](../00-foundations/information-architecture.md) — evolution from 2012 to 5.0

## Key Design Insights from 2012

### All Games Page Redesign (Priority: Highest)

**Problem Identified:**
- Too few games showing per page (only 12 thumbnails visible)
- Showing game/stage combinations instead of games as the primary unit
- Excessive scrolling required to browse content
- Missing game name search capability

**Solution Specified:**
1. **Two-tier structure:**
   - **Screen 1:** User selects a game from default (thumbnail) or list view
   - **Screen 2:** Upon game selection, user chooses stage (Learn, Play, Quiz, Challenge)
   - **Screen 3:** Full-screen game play with header/footer removed
   - **Screen 4:** Upon quit, return to Screen 2 (stage selection)

2. **Thumbnail view improvements:**
   - Significantly smaller thumbnails (24-36 games per page instead of 12)
   - Show games, not game/stages
   - More content "above the fold" to reduce scrolling

3. **List view improvements:**
   - Single line per game (not game/stage)
   - Minimum info: Game ID, Game Name, Skill Level, Skill Area
   - Simple availability indicator
   - Tiny thumbnails or eliminate if necessary for single-line layout

4. **Game information display:**
   - Game ID
   - Game Name
   - Skill Level
   - Skill Area
   - Visual or Aural indicator

5. **Search and filtering:**
   - Game name search (implemented per notes)
   - Existing filter controls deemed sufficient once search was added

**Rationale:** Efficient game selection is the highest priority—the platform has no value without it.

See also: [All Games Library](../03-student-experience/all-games-library.md) — current implementation

### Assignment Play Screen Structure

**Source:** EXE specs 2012-09-13 Assignment Screen

**Structural Components:**

1. **Assignment Group Header**
   - Sequence Name (e.g., "Alfred Premier")
   - Level Field (e.g., "Level 1A")
   - Unit Field (if applicable; note Alfred has no Unit)
   - Notation Field (e.g., "Pages 5-11")
   - Group Number
   - Recess Level

2. **Pre-Assignment Pane** (gating)
   - Text introduction
   - Video clips (if applicable)
   - Must be completed before other panes can be entered
   - Completion indicator: P (pending) → Completed

3. **Main Assignment Pane** (core work)
   - Organized by concept groupings
   - Each concept shows:
     - Concept description (pulled from Learn stage game)
     - Game stages: Learn, Play, Quiz (Review omitted from UI but used in sequences)
     - Target Score
     - Times Played
     - Best Score (with date)
     - Last Score (with date)
   - Color coding for score status:
     - **Achieved Perfect Score**
     - **Achieved Target Score** (100% of target)
     - **Almost Achieved Target Score** (90% of target)
     - **Played but Low Score** (<90% of target)

4. **MIDI Sub-Pane** (conditional)
   - Appears only if assignment includes MIDI-enabled games
   - MIDI games replace or hide their non-MIDI equivalents
   - Labeled "These games require a MIDI keyboard attached"

5. **Custom Sub-Pane** (optional)
   - Additional games specified by teacher
   - Usually repeating Learn games for extra drill on a concept

6. **Post-Assignment Pane** (unlocked upon completion)
   - Challenge games
   - Video clips
   - Appears only when all above panes completed

**Key Insights:**
- Students need to see quantitative scores, not just completion status
- Teachers need to view results with scores to assess progress
- Gating ensures proper sequencing of instruction
- MIDI support requires conditional UI presentation

See also:
- [Assignment Play](../03-student-experience/assignment-play.md) — current student experience
- [Assignment Model](../07-content-architecture/assignment-model.md) — data structure
- [Assignment Generation](../10-assignments-engine/assignment-generation.md) — logic

### Assignment Play Redesign Issues (Later Iteration)

**Problems Identified:**

**Issue A:** No easy way for student to navigate back to dashboard to check scores  
**Issue B:** Logo/placeholder on top is too large and obtrusive  
**Issue C:** Game ID not displayed  
**Issue D:** "Similar Courses" should show next assignments in the sequence, not unrelated content  
**Issue E:** Scores should display in the list so students can see progress at a glance  
**Issue F:** Scores not recording (specific bug from 4-4-21 testing with Staff Birds, Song Birds, Low Middle High)

**Solution Requirements:**
- Add "Return to Dashboard" or "Check Scores" button
- Reduce header size
- Display Game ID prominently
- Show upcoming assignments in sequence context
- Inline score display in assignment list

See also: [Assignment Play Redesign](../03-student-experience/assignment-play-redesign.md) — detailed solutions

### Game Categorization System (Concept Parameters)

**Source:** EXE specs 2012-09-13 Game Screen and Category Screen

The 2012 system used a hierarchical concept parameter structure for categorizing and searching games:

**Primary Categories (with codes):**
1. **BASE** — Basic Elements
   - `BASE-ACT` — Active
   - `BASE-AUR` — Aural
   - `BASE-METH-ID` — Identification
   - `BASE-MIDI` — MIDI

2. **STAGE** — Stage
   - `STAGE-LERN` — Learn
   - `STAGE-PLAY` — Play
   - `STAGE-QUIZ` — Quiz

3. **RHYTM** — Rhythm
4. **METER** — Meter
5. **STAFF** — Staff & Pitch
6. **MELOD** — Melody
   - `MELOD-PRER-UPDN` — Up, down

7. **SCALE** — Scale
8. **KYSIG** — Key Signatures
9. **INTVL** — Intervals
10. **KEYBD** — Keyboard Elements
    - `KEYB-PRER-LOW` — Low vs High
    - `KEYB-PRER-2BLK` — 2 Black keys vs 3 Black keys

11. **CHORD** — Chords
12. **HARM** — Harmonic Progression
13. **TERM** — Music Terms
14. **MEMO** — Playback-Tonal Memory
    - `MEMO-KEYB-3NOT` — Keyboard 3 Note

**Game Identification Format:**
- Game Name: e.g., "Mad Cap Melodies"
- Game Number: e.g., 21341
- LMC Group Number: e.g., 13.0
- Full identifier: `LMC-21341-13.0`

**Usage:**
- **Categorization Screen:** Used to assign concept parameters to each game or supporting media
- **Category/Search Screen:** Used to find games matching specific concept parameters for sequence authoring

**Purpose:**
- Enables curriculum mapping to method books (Alfred, Faber, etc.)
- Supports filtered game search for assignment building
- Provides structured metadata for pedagogical alignment

See also:
- [Game Model](../08-games-registry-and-authoring/game-model.md) — evolved data structure
- [Game Taxonomy](../07-content-architecture/game-taxonomy.md) — current categorization
- [Sequence Authoring](../09-sequences-and-curriculum-tools/sequence-authoring.md) — how categories enable sequence building

### System Admin Panel Redesign

**Problems Identified:**

**Member Profile/Structure:**
- In legacy system, admin could see member's entire hierarchy (subscriber → teachers → students)
- In MLC live, admin must log in with member credentials to see structure
- Administrator panels don't provide member credentials (password)

**Users List:**
- Shows all users (admin, teacher, student) in reverse chronological order by registration
- Becomes unwieldy when new school with many students is added
- Need better filtering and sorting

**Solutions Specified:**

**Priority A:** Using member number filter, show all users sorted by type:
1. Subscriber
2. Teacher-Admin
3. Teacher
4. Student

**Priority B:** Add filter to show only Subscribers (because thousands of students make full list unusable)

**Suggestions:**
1. Include username and password in user profile or edit panel
2. Add "Role" field showing user type (Admin, Teacher-Admin, Teacher, Student)
3. Move phone to profile view instead of main list

**Rationale:** System admins need efficient tools to support members without requiring credential juggling or manual searching through unfiltered lists.

See also:
- [System Admin User Index](../06-system-admin/sysadmin-user-index.md) — current implementation
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) — role definitions

### Screen Pages Text and Copy (2012)

The 2012 Screens Text document provides original marketing copy, feature descriptions, and workflow explanations. Key elements:

**Role Types Defined:**
- Prelude Subscriber (free trial)
- Solo Subscriber
- Ensemble Subscriber
- Teacher-Admin
- Teacher
- Student

**Workflow Descriptions:**
- **SOLO:** One teacher → Set up students → Evaluate → Assign sequences → Students play → Review scores
- **ENSEMBLE:** One or more teachers → Set up teachers → [rest same as Solo]

**Pricing Tiers (2012):**
- **Prelude:** 30-day free trial, 2 teachers, up to 19 student seats
- **Solo Monthly:** $7.95/month base, 5 seats included, up to 19 total, $0.80 per additional seat
- **Solo Annually:** $95.40/year, 5 seats included, up to 19 total, $2.40 per additional seat
- **Ensemble Monthly:** Starts at $19.95/month, 20 seats minimum, unlimited additional seats
- **Ensemble Annually:** Starts at $239.40/year, 20 seats minimum, unlimited additional seats

**Lifetime Musician Philosophy:**
> "The Lifetime Musician is an active participant in music-making, not a spectator. Lifetime Musicians have a music 'skill-set' broad enough to be able to learn new music without having to listen to a recording."

See also:
- [Personas](../00-foundations/personas.md) — role descriptions evolved
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) — Lifetime Musician philosophy
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) — current pricing model
- [Copy Guidelines](../22-content-style-guides/copy-guidelines.md) — evolved writing standards

## Data Model Implications

The 2012 specs implied these key entities and relationships:

**Assignment Group:**
- `assignment_group_id`
- `sequence_name` (e.g., "Alfred Premier")
- `level` (e.g., "Level 1A")
- `unit` (optional; method-dependent)
- `notation` (e.g., "Pages 5-11")
- `group_number`
- `recess_level`

**Assignment Pane Types:**
- Pre-Assignment (gating: must complete first)
- Main Assignment (core work)
- MIDI Sub-Pane (conditional)
- Custom Sub-Pane (teacher additions)
- Post-Assignment (unlocks upon completion)

**Game Metadata:**
- Game Number (e.g., 21341)
- LMC Group Number (e.g., 13.0)
- Concept Parameters (multi-select from 14 categories)
- Stage (Learn/Play/Quiz)
- MIDI variant flag

**Score Tracking (per game/stage):**
- `target_score`
- `times_played`
- `best_score` + date
- `last_score` + date
- Status: Perfect | Target | Almost | Low

See also:
- [Data Model ERD](../18-architecture/data-model-erd.md) — evolved schema
- [Assignment Model](../07-content-architecture/assignment-model.md) — current structure

## Behavior and Rules (from 2012 Specs)

### Gating Logic
1. Pre-Assignment pane must be completed before Main Assignment can be entered
2. Post-Assignment pane unlocks only when all preceding panes are completed
3. Review stage is used in sequences but not shown in UI

### Score Color Coding
- **Green (Perfect Score):** 100% accuracy or maximum points
- **Green (Target Score):** 100% of target_score value
- **Yellow (Almost):** 90-99% of target_score
- **Red (Low):** <90% of target_score

### MIDI Game Presentation
- If MIDI variant exists, it replaces or hides non-MIDI version in UI
- MIDI pane appears only when MIDI games are present in assignment
- Label: "These games require a MIDI keyboard attached"

### Navigation Flow (All Games)
1. Student enters All Games (thumbnail or list view)
2. Student selects a game → Stage selection screen appears
3. Student selects stage (Learn/Play/Quiz/Challenge) → Full-screen game launches
4. Student completes or quits game → Returns to Stage selection screen (not All Games)

See also:
- [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md) — current gating logic
- [Game Player Shell](../03-student-experience/game-player-shell.md) — navigation implementation

## UX Requirements (2012 Patterns)

### Visual Hierarchy
- **Assignment Concept Grouping:** Each concept shows title + list of game stages beneath
- **Score Visibility:** Date and score displayed inline (not hidden behind clicks)
- **Completion Indicators:** "P" (pending) → "Completed" for panes
- **Game ID Prominence:** Always visible for troubleshooting and teacher reference

### Layout Efficiency
- **Reduce chrome:** Remove header/footer during game play to maximize play area
- **Smaller thumbnails:** More games visible "above the fold" reduces scrolling
- **Single-line list items:** Maximum information density for browsing

### Information Display
- **Quantitative over qualitative:** Show actual scores, not just "done" status
- **Dates for context:** Last played and best score dates help teachers assess recency
- **Target score transparency:** Students and teachers both see target to understand expectations

### Conditional UI
- **MIDI sub-pane:** Appears only when MIDI games present
- **Custom sub-pane:** Appears only when teacher added extra games
- **Post-assignment pane:** Hidden until all prior work completed

See also:
- [Components Overview](../01-ux-design-system/components-overview.md) — evolved component library
- [Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md) — mobile considerations not in 2012 scope

## Telemetry (Implied by 2012 Specs)

The 2012 design implied these tracking needs:

**Game Play Events:**
- `game_started` (game_id, stage, timestamp)
- `game_completed` (game_id, stage, score, timestamp)
- `game_quit` (game_id, stage, elapsed_time)

**Assignment Progress Events:**
- `pane_completed` (assignment_id, pane_type, timestamp)
- `assignment_completed` (assignment_id, timestamp)

**Search/Discovery Events:**
- `game_searched` (query_text)
- `game_filtered` (concept_parameters)

**Score Recording:**
- Update `times_played` counter
- Update `best_score` if current score exceeds previous
- Always update `last_score` and date

See also:
- [Event Model](../15-analytics-and-reporting/event-model.md) — comprehensive telemetry specification
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) — how scores surface to teachers

## Permissions (2012 Context)

The 2012 materials implied these access patterns:

**System Admin:**
- View all members and their hierarchies
- Access member credentials for support purposes
- Filter/sort user lists by role type
- View student scores across all members

**Subscriber Admin:**
- Manage own account hierarchy
- Add/edit teachers and students
- View billing and subscription details

**Teacher-Admin:**
- Same as Teacher, plus manage other teachers

**Teacher:**
- Create assignments
- View student scores
- Customize assignments with additional games

**Student:**
- Play assigned and free-choice games
- View own scores and progress

See also:
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) — comprehensive role definitions
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) — granular permissions
- [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md) — modern support tooling

## Dependencies

The 2012 wireframes and specs depended on:

**Content Systems:**
- Game registry with concept parameter tagging
- Sequence authoring tool with drag-and-drop game selection
- Method book alignment data (Alfred, Faber, Hal Leonard, etc.)

**Scoring Systems:**
- Target score definition per game/stage
- Best score tracking with timestamp
- Completion status calculation

**Role Management:**
- Multi-tiered account structures (subscriber → teacher → student)
- Role-based UI presentation (MIDI pane, custom pane visibility)

**Media Delivery:**
- Video clip hosting and playback
- Game asset loading (thumbnails, SWF files in 2012)

See also:
- [Service Boundaries](../18-architecture/service-boundaries.md) — how dependencies are now organized
- [Content Versioning](../07-content-architecture/content-versioning.md) — managing evolving game library

## Evolution to MLC 5.0

### Preserved Principles
- Two-tier game selection (game → stage) remains core to All Games
- Score visibility and quantitative feedback central to student/teacher experience
- Gating logic for assignment panes ensures pedagogical sequencing
- Concept parameter system (evolved but conceptually similar) powers search and alignment

### Evolved Solutions
- **Responsive design:** 2012 was desktop-only; 5.0 supports mobile/tablet
- **Modern frameworks:** 2012 used Flash (SWF); 5.0 uses HTML5/JS
- **API-first architecture:** 2012 was monolithic; 5.0 separates concerns
- **Accessibility:** WCAG compliance now central (not addressed in 2012)
- **Webhooks and integrations:** LTI, LMS, SSO not in 2012 scope

### Lessons Applied
- **Efficiency over aesthetics:** Dense information display when browsing games
- **Visibility of system status:** Always show scores, dates, completion state
- **Reduce friction:** Minimize navigation steps between game selection and play
- **Support scalability:** System admin tools must handle schools with 1000+ students

See also:
- [Phase 1 Scope](../24-roadmap-and-phasing/phase-1-scope.md) — what's built first in 5.0
- [Initial Project Phases](../00-foundations/initial-project-phases.md) — overall strategy

## Open Questions

### Historical Context
1. **Were the 2012 wireframes fully implemented in MLC 4.0?** Or were some deferred/changed?
2. **What were the biggest user complaints** post-launch of the redesigned All Games?
3. **Did the two-tier game selection** (game → stage) test well with students?
4. **How well did the concept parameter system scale** as the game library grew?

### Applicability to MLC 5.0
1. **Should we preserve the exact same concept parameter taxonomy** or modernize it?
2. **Is the gating logic** (pre/main/post panes) still pedagogically optimal, or can we simplify?
3. **Do we still need separate MIDI and non-MIDI variants** of games, or can we detect/adapt dynamically?
4. **Should System Admin credential access** be preserved or replaced with secure impersonation?

### Design Validation Needed
1. **Mobile/tablet UX:** How does the two-tier structure adapt to small screens?
2. **Accessibility:** Can screen readers effectively navigate the score tables and assignment panes?
3. **Performance:** With 450+ games, does the All Games page load/render acceptably?

See also:
- [Non-Goals](../00-foundations/non-goals.md) — what we're explicitly not doing
- [User Feedback](../25-research-and-inspiration/user-feedback.md) — input from current MLC 4.0 users
