# Free Play vs Assigned

This document defines the critical distinction between **free play** (student-initiated game exploration via the All Games library) and **assigned content** (teacher-directed learning via Sequences and Assignments). Understanding this distinction is essential for:
- Content creators building games that work in both contexts
- Designers crafting interfaces that clearly communicate mode differences
- Engineers implementing reconciliation logic and scoring rules
- Teachers understanding how student agency and structured curriculum interact
- Students navigating between exploratory learning and formal assessment

## Purpose
This spec establishes the behavioral, data model, and policy differences between free play and assigned modes. It defines:
- **When** each mode is active and how students enter/exit each context
- **How** scoring, progression, and gating differ between modes
- **Why** certain behaviors differ (pedagogical rationale)
- **How** free play scores integrate with assignments (reconciliation)
- **What** telemetry distinguishes the two contexts for analytics and reporting

This specification addresses a critical pain point from the original MLC system, where unclear boundaries between free play and assigned content led to student confusion about progression, score recording failures, and teacher visibility gaps.

## Scope

### Included
- **Mode Definitions**: Clear definitions of free play vs assigned contexts and when each applies
- **Entry Points**: How students access each mode and what interface surfaces each
- **Behavioral Differences**: Gating, scoring, progression, and completion rules unique to each mode
- **Data Model Distinctions**: How records differ structurally between free play and assigned sessions
- **Free Play Reconciliation**: The integration layer where free play scores count toward assignments
- **Policy Controls**: Teacher and admin configuration of reconciliation behavior
- **Telemetry Differences**: Event naming, properties, and tagging that distinguish contexts
- **UX Indicators**: Visual cues that communicate current mode to students
- **Permissions**: Who can access each mode and under what conditions

### Excluded
- **Game Runtime Mechanics**: Internal game logic (covered in [Game Model](../08-games-registry-and-authoring/game-model.md))
- **Assignment Creation**: How assignments are generated (covered in [Assignment Generation](./assignment-generation.md))
- **All Games Library UI**: Detailed interface specs (covered in [All Games Library](../03-student-experience/all-games-library.md))
- **Assignment Play UI**: Detailed interface specs (covered in [Assignment Play](../03-student-experience/assignment-play.md))
- **Teacher Assignment Builder**: Creation interface (covered in [Assignment Builder](../04-teacher-experience/assignment-builder.md))
- **Detailed Progress Tracking**: Progress calculation algorithms (covered in [Assignment Progress Tracking](./assignment-progress-tracking.md))

## Data model (if applicable)

### Core Distinction: Context Tagging

Every game session is tagged with a **play context** that distinguishes free play from assigned play:

```typescript
interface GameSession {
  session_id: string;
  game_id: string;
  stage_id: string;
  student_id: string;
  
  // Context discriminator
  play_context: 'free_play' | 'assigned';
  
  // Context-specific references
  assignment_id?: string;      // Present only for assigned context
  step_id?: string;             // Present only for assigned context
  sequence_id?: string;         // Present only for assigned context
  
  // Scoring and completion
  score: number;
  max_score: number;
  passed: boolean;
  target_score: number;         // Source differs by context
  
  // Timestamps
  started_at: Date;
  completed_at?: Date;
  
  // Session metadata
  device_profile: string;
  input_mode: 'keyboard' | 'midi' | 'mic' | 'touch';
}
```

### Free Play Sessions

Free play sessions are initiated through the **All Games Library** and have these characteristics:

```typescript
interface FreePlaySession extends GameSession {
  play_context: 'free_play';
  
  // Free play never has assignment references
  assignment_id: null;
  step_id: null;
  sequence_id: null;
  
  // Target score comes from game registry defaults
  target_score: number;         // From game stage definition
  
  // Free play specific tracking
  discovery_path: 'browse' | 'search' | 'favorite' | 'recommended';
  parent_session_id?: string;   // If replaying same stage
}
```

**Storage**: Free play sessions are stored in a `free_play_sessions` table optimized for:
- Fast lookups by `student_id` + `game_id` + `stage_id` for reconciliation queries
- Historical score retrieval for "best score" calculations
- Analytics on discovery patterns and game popularity

### Assigned Sessions

Assigned sessions are initiated through the **Assignment Play** interface and link to formal assessment structures:

```typescript
interface AssignedSession extends GameSession {
  play_context: 'assigned';
  
  // Assignment context is always present
  assignment_id: string;        // Required
  step_id: string;              // Required
  sequence_id: string;          // Required
  sequence_version: string;     // For pinning and migration
  
  // Target score comes from policy hierarchy
  target_score: number;         // From assignment override > class policy > game default
  
  // Assignment specific tracking
  gating_context: {
    gates_met: string[];        // Which gates were satisfied to unlock this step
    optional: boolean;          // Whether this step was optional
    retry_count: number;        // Attempts on this specific step
  };
  
  // Progress tracking
  contributes_to_progress: boolean;  // Whether this completion counts toward assignment progress
}
```

**Storage**: Assigned sessions are stored in an `assigned_sessions` table optimized for:
- Fast progress calculation by `assignment_id` and `step_id`
- Teacher reporting by `assignment_id` and `student_id`
- Retention analysis by `sequence_id` and `sequence_version`

### Score Records

Both modes write to a unified `score_records` table, but with context-specific metadata:

```typescript
interface ScoreRecord {
  score_id: string;
  student_id: string;
  game_id: string;
  stage_id: string;
  
  // Context tagging
  play_context: 'free_play' | 'assigned';
  session_id: string;           // Links to context-specific session
  
  // Score data
  score: number;
  max_score: number;
  passed: boolean;
  target_score: number;
  
  // Timestamps
  recorded_at: Date;
  
  // Context references (nullable for free play)
  assignment_id?: string;
  step_id?: string;
}
```

This unified structure enables:
- **Best Score Queries**: `SELECT MAX(score) FROM score_records WHERE student_id = ? AND game_id = ? AND stage_id = ?` returns best score regardless of context
- **Reconciliation Queries**: Check if free play score qualifies for assignment completion
- **Analytics**: Compare performance between free play and assigned contexts

### Reconciliation Records

When free play scores count toward assignments, the system creates a **reconciliation record**:

```typescript
interface ReconciliationRecord {
  reconciliation_id: string;
  student_id: string;
  assignment_id: string;
  step_id: string;
  
  // Source free play session
  free_play_session_id: string;
  free_play_score: number;
  free_play_date: Date;
  
  // Reconciliation logic
  threshold_met: boolean;       // Did score meet assignment target?
  reconciliation_policy: string; // Policy version used
  reconciled_at: Date;
  
  // Outcome
  step_marked_complete: boolean; // Whether reconciliation completed the step
  teacher_override_required: boolean; // If policy requires teacher approval
}
```

### Data Model Summary

| Aspect | Free Play | Assigned |
|--------|-----------|----------|
| **Primary Table** | `free_play_sessions` | `assigned_sessions` |
| **Context Tag** | `play_context: 'free_play'` | `play_context: 'assigned'` |
| **Assignment References** | `null` (never present) | Required (`assignment_id`, `step_id`) |
| **Target Score Source** | Game registry default | Policy hierarchy (override > class > default) |
| **Progress Tracking** | No assignment progress impact | Contributes to assignment completion |
| **Score Storage** | Unified `score_records` table with context tag | Unified `score_records` table with context tag |
| **Best Score Query** | Includes scores from both contexts | Includes scores from both contexts |
| **Reconciliation** | Source of scores for assignment completion | Target of reconciliation from free play |

## Behavior and rules

### Mode Entry and Exit

#### Entering Free Play Mode
Students enter free play mode through the **All Games Library** interface accessible from:
1. **Student Dashboard** → "All Games" navigation item
2. **Assignment Play** → "Back to All Games" after game completion (if configured)
3. **Direct URL** → Deep link to specific game in library

**Entry Behavior**:
- No assignment context is set (`assignment_id` remains `null`)
- All non-deprecated games are available (subject to subscription tier)
- No gating rules apply—students can play any stage in any order
- Target scores are pulled from game registry defaults
- Discovery path is tracked for analytics (`browse`, `search`, `favorite`, `recommended`)

#### Entering Assigned Mode
Students enter assigned mode through the **Assignment Play** interface:
1. **Student Dashboard** → "My Assignments" or "Next Up" button
2. **Teacher Link** → Direct assignment URL shared by teacher
3. **Notification Link** → Deep link from assignment notification

**Entry Behavior**:
- Assignment context is set (`assignment_id`, `step_id`, `sequence_id` populated)
- Only games in the current assignment are accessible
- Gating rules apply—some stages may be locked
- Target scores come from policy hierarchy (override > class policy > game default)
- Progress tracking is active—completion contributes to assignment progress

#### Exiting Each Mode
**Free Play Exit**:
- Student navigates away via breadcrumb or menu
- Session persists but doesn't block navigation
- Score is recorded immediately upon game completion
- No assignment progress is updated

**Assigned Play Exit**:
- Student uses "Return to Assignment" or "Return to Dashboard" button
- If mid-game, completion state is preserved for resume
- Score is recorded and assignment progress is updated
- Next Up pointer may advance if step completes

### Gating and Progression

#### Free Play Gating Rules
**No gates apply** in free play mode:
- Students can play Learn, Play, Quiz, Challenge, Review in any order
- Students can repeat any stage unlimited times
- No prerequisites block access to stages
- No "Next Up" logic constrains navigation

**Pedagogical Rationale**: Free play supports exploratory learning, skill building at the student's own pace, and discovery-based engagement. Removing gates encourages experimentation and allows students to find entry points that match their current skill level.

#### Assigned Gating Rules
**Default gates** (see [Assignment Generation](./assignment-generation.md) for full details):
- **Quiz** requires at least one attempt on Learn and Play for the same Game
- **Review** requires a passing Quiz for the same Game (score >= target)
- **Challenge** is optional and never gates progression
- **Sequential steps** may be gated by `require_previous_steps` policy

**Policy Overrides**:
- Teachers can set `require_previous_steps: false` to allow parallel work
- Teachers can mark specific steps as optional, removing gates
- Admins can configure class-level gating policies

**Pedagogical Rationale**: Gating in assigned mode ensures students follow the intended instructional sequence (Learn → Play → Quiz → Review), preventing skill gaps and ensuring formative practice before assessment. Gates protect assessment integrity while allowing teacher flexibility for differentiation.

### Scoring and Target Behavior

#### Free Play Scoring
**Target Score Source**: Game registry default
- Each stage has a default target score defined in the game registry
- Targets are NOT influenced by teacher policies or overrides
- Example: `Quiz` stage for "Staff Birds" has default target of 80%

**Score Recording**:
- Every completion writes to `free_play_sessions` table
- Scores also write to unified `score_records` with `play_context: 'free_play'`
- Best score is tracked per `student_id` + `game_id` + `stage_id`
- Historical scores are retained for analytics and reconciliation

**Pass/Fail Semantics**:
- `passed: true` if `score >= target_score` (game default)
- Pass/fail has no impact on navigation or access—students can proceed regardless
- No assignment progress is updated

#### Assigned Scoring
**Target Score Source**: Policy hierarchy
1. **Assignment Override** (highest priority): Teacher sets specific target for this assignment
2. **Class Policy**: Admin sets class-wide target adjustments (e.g., 85% for advanced class)
3. **Game Default** (fallback): Registry default if no overrides exist

**Score Recording**:
- Every completion writes to `assigned_sessions` table with full assignment context
- Scores also write to unified `score_records` with `play_context: 'assigned'`
- Assignment progress is recalculated immediately (see [Assignment Progress Tracking](./assignment-progress-tracking.md))
- Step state transitions from `in_progress` to `complete` if passed

**Pass/Fail Semantics**:
- `passed: true` if `score >= target_score` (from policy hierarchy)
- **Passing Quiz** unblocks Review and may advance Next Up pointer
- **Failing Quiz** keeps step in `in_progress` state, student can retry
- Retry count is tracked for at-risk detection and intervention triggers

### Free Play Reconciliation

**Reconciliation** is the process where free play scores automatically complete assignment steps, avoiding redundant work while maintaining pedagogical integrity.

#### When Reconciliation Happens
Reconciliation attempts occur at these trigger points:
1. **Assignment Generation**: When assignment is first created, check for qualifying free play scores
2. **Step Unlock**: When a step transitions from `locked` to `available`, check for prior free play score
3. **Manual Trigger**: Teacher can force reconciliation check for specific students
4. **Batch Reconciliation**: Nightly job checks for recent free play scores that qualify for open assignments

#### Reconciliation Rules
```pseudo
function should_reconcile(free_play_score, assignment_step, class_policy):
  # Policy gate
  if (class_policy.require_fresh_attempt == true):
    return false  # Policy requires fresh attempt regardless of free play score
  
  # Threshold check
  if (free_play_score.score < assignment_step.target_score):
    return false  # Free play score didn't meet assignment target
  
  # Timing check (optional)
  if (class_policy.reconciliation_window_days != null):
    days_since_score = today - free_play_score.recorded_at
    if (days_since_score > class_policy.reconciliation_window_days):
      return false  # Score is too old per policy
  
  # Match check
  if (free_play_score.game_id != assignment_step.game_id):
    return false  # Different game
  if (free_play_score.stage_id != assignment_step.stage_id):
    return false  # Different stage
  
  return true  # All conditions met, reconcile
```

#### Reconciliation Actions
When reconciliation qualifies:
1. **Mark Step Complete**: Set `step.state = 'complete'`
2. **Set Completion Source**: `step.completion_source = 'free_play_reconciled'`
3. **Create Reconciliation Record**: Log the free play session that provided the score
4. **Update Progress**: Recalculate assignment completion percentage
5. **Advance Next Up**: Move Next Up pointer if this was the blocking step
6. **Emit Telemetry**: Fire `free_play_reconciled` event (see Telemetry section)

**Student Experience**:
- Student opens assignment and sees some steps already marked complete with indicator "✓ Completed in Free Play"
- Student can still replay the stage if desired, but it doesn't affect completion status unless they achieve a new best score
- Student can proceed directly to subsequent steps without redundant work

**Teacher Experience**:
- Teacher sees completion source in student progress view: "Completed via Free Play on [date]"
- Teacher can view the original free play score and compare to assignment target
- Teacher can override reconciliation if they want to require a fresh attempt (with audit log)

#### Reconciliation Policies
Class-level policies control reconciliation behavior:

```typescript
interface ReconciliationPolicy {
  // Global toggle
  require_fresh_attempt: boolean;          // If true, never reconcile (default: false)
  
  // Threshold adjustments
  reconciliation_score_multiplier: number; // Default 1.0, can set 1.1 to require 10% higher free play score
  
  // Time window
  reconciliation_window_days?: number;     // If set, only scores within N days qualify (default: null = unlimited)
  
  // Stage filters
  reconcile_learn: boolean;                // Allow Learn stage reconciliation (default: true)
  reconcile_play: boolean;                 // Allow Play stage reconciliation (default: true)
  reconcile_quiz: boolean;                 // Allow Quiz stage reconciliation (default: false for assessment integrity)
  reconcile_review: boolean;               // Allow Review stage reconciliation (default: false)
  
  // Teacher approval
  require_teacher_approval: boolean;       // If true, reconciliation is pending until teacher confirms (default: false)
}
```

**Pedagogical Rationale**: Reconciliation respects student agency and prior learning while maintaining assessment integrity. By default, Learn and Play stages reconcile freely (practice doesn't need duplication), but Quiz reconciliation can be disabled to ensure formal assessment occurs in the assigned context where teachers have full visibility and control.

### Replay Behavior

#### Replaying in Free Play
- Students can replay any stage unlimited times
- Each replay creates a new session record
- Best score is updated if new score exceeds prior best
- No impact on assignment progress (even if the game is in an active assignment)

#### Replaying in Assigned Mode
- Students can retry any step in `available`, `in_progress`, or `complete` states
- Retry count is tracked per step (`step.gating_context.retry_count++`)
- New best score updates assignment progress if higher than previous
- Excessive retries trigger at-risk detection (typically > 5 attempts without passing)

### Cross-Mode Behavior

#### Playing Assigned Game in Free Play
If a student plays a game in free play mode that also exists in an active assignment:
- Free play session is recorded normally with `play_context: 'free_play'`
- Score writes to `score_records` with free play context
- Reconciliation check is triggered asynchronously (if policy allows)
- If reconciliation qualifies, assignment step is marked complete
- Student dashboard shows progress update on next page load

**No Real-Time Sync**: Free play and assigned contexts remain independent during play. Reconciliation happens post-session, not mid-game.

#### Playing Free Play Game That Later Gets Assigned
Common scenario:
1. Student plays "Staff Birds - Quiz" in free play on Monday, scores 85%
2. Teacher assigns sequence containing "Staff Birds - Quiz" on Wednesday
3. Assignment generation runs reconciliation check
4. Step is marked complete on assignment creation with source `free_play_reconciled`

**Historical Score Preservation**: All prior free play scores are retained and remain eligible for reconciliation when assignments are created, regardless of how old the score is (unless policy sets `reconciliation_window_days`).

### State Transitions Summary

#### Free Play State Machine
```
[Not Started] → [Playing] → [Completed]
     ↓             ↓             ↓
  (replay)     (replay)      (replay)
     ↓             ↓             ↓
  [Playing] → [Completed] → [Completed with higher score]
```
- No locked states
- No gates
- Unlimited replays
- No external impact

#### Assigned State Machine
```
[Locked] → [Available] → [In Progress] → [Complete]
              ↓              ↓               ↓
           (retry)        (retry)         (replay)
              ↓              ↓               ↓
         [In Progress] → [Complete]    [Complete with higher score]
```
- Gates control transitions from Locked → Available
- Completion (passing score) transitions In Progress → Complete
- Replays allowed at any stage but retry count tracked
- Progress updates trigger reconciliation checks, remediation, and next up logic

## UX requirements (if applicable)

### Mode Indicators

The interface must clearly communicate which mode the student is in at all times to prevent confusion about scoring, progression, and expectations.

#### Free Play Mode Indicators
Visual signals that student is in free play mode:
- **Page Title**: "All Games Library" or "Browse Games"
- **Breadcrumb**: `Dashboard > All Games > [Game Name]`
- **No Assignment Context**: No assignment name or sequence progress displayed
- **Badge/Chip**: Optional "Free Play" badge in header (subtle, not obtrusive)
- **Navigation**: "Back to Library" or "Back to All Games" button prominent
- **Score Display**: Shows personal best score with "Your Best" label, no target comparison
- **No Progress Bar**: No assignment completion percentage shown

#### Assigned Mode Indicators
Visual signals that student is in assigned mode:
- **Page Title**: Assignment name (e.g., "Level 1A - Pages 5-11")
- **Breadcrumb**: `Dashboard > My Assignments > [Assignment Name] > [Game Name]`
- **Assignment Context**: Sequence name and assignment group number visible
- **Progress Bar**: Assignment completion percentage prominently displayed
- **Next Up Button**: "Next Step" or "Continue Assignment" CTA
- **Target Score Display**: Shows required score with comparison to current score
- **Step Counter**: "Step 3 of 12" or similar position indicator
- **Due Date**: If applicable, "Due: [date]" indicator

### Mode Transition Flows

#### Transitioning from Dashboard to Free Play
```
1. Student Dashboard
2. Click "All Games" in navigation
3. → All Games Library page loads with mode indicators
4. Student selects game from grid or list
5. → Stage selection screen (Learn, Play, Quiz, Challenge, Review)
6. Student selects stage
7. → Game launches in chrome-less mode with free play context
```

**Visual Continuity**: Library uses distinct color scheme or header design to signal free play context.

#### Transitioning from Dashboard to Assigned Play
```
1. Student Dashboard
2. Click "Next Up" button or assignment card
3. → Assignment Play page loads with assignment context indicators
4. Student sees locked/available/complete states for steps
5. Student clicks available step
6. → Stage selection if game has multiple stages
7. → Game launches in chrome-less mode with assigned context
```

**Visual Continuity**: Assignment interface uses distinct color scheme, prominent progress indicators, and assignment branding to signal formal learning context.

#### Cross-Mode Navigation
Students should be able to navigate between modes, but with clear context switches:

**Free Play → Assigned**:
- Dashboard shows "Back to Assignments" option if student has active assignments
- Global navigation always shows "My Assignments" menu item
- Notification badges on "My Assignments" indicate pending work

**Assigned → Free Play**:
- Assignment Play interface includes "Browse All Games" link in secondary navigation
- Game completion modal offers "Continue Assignment" (primary) vs. "Explore Library" (secondary)
- Clear exit from assignment context confirmed visually

### Score and Progress Display

#### Free Play Score Display
```
┌─────────────────────────────────────┐
│ Staff Birds - Quiz                  │
│                                     │
│ Your Best Score: 85%                │
│ Last Played: 2 days ago             │
│                                     │
│ [Play Again]                        │
└─────────────────────────────────────┘
```
- Emphasizes personal achievement and history
- No pressure indicators (no "target score" or "you need X%")
- Encourages replay without penalty

#### Assigned Score Display
```
┌─────────────────────────────────────┐
│ Staff Birds - Quiz                  │
│ Step 5 of 12 in "Level 1A"         │
│                                     │
│ Target Score: 80%                   │
│ Your Score: 85% ✓                   │
│                                     │
│ Status: Complete!                   │
│ [Next Step →]                       │
└─────────────────────────────────────┘
```
- Shows target and achievement relative to requirement
- Clear completion status with checkmark
- Guides to next action (progress through assignment)

### Reconciliation Indicators

When a step is completed via free play reconciliation, the UI must clearly communicate:

#### Student View
```
┌─────────────────────────────────────┐
│ ✓ Staff Birds - Quiz                │
│                                     │
│ Completed via Free Play             │
│ Score: 85% (Target: 80%)            │
│ Date: March 10, 2025                │
│                                     │
│ [Play Again] [Next Step →]         │
└─────────────────────────────────────┘
```
- Green checkmark indicates completion
- "Completed via Free Play" badge or label explains source
- Date provides timeline context
- Option to replay if student wants (but not required)

#### Teacher View (Student Progress Report)
```
Step 5: Staff Birds - Quiz
Status: ✓ Complete
Score: 85% (Target: 80%)
Source: Free Play Reconciliation (March 10, 2025)
Attempts: 1 (Free Play)
Time: 3:45 (Free Play session)
[View Details] [Require Fresh Attempt]
```
- Completion source clearly labeled
- Teacher can see original free play details
- Action available to override and require fresh attempt if needed
- Audit trail maintained

### Accessibility Requirements

#### Screen Reader Announcements
- **Mode Context**: Screen reader announces "Free Play mode" or "Assignment mode" on page load
- **Status Changes**: "Step marked complete via free play reconciliation" announced when reconciliation occurs
- **Score Comparisons**: "Your score 85%, target score 80%, passed" announced clearly in assigned mode
- **Navigation**: "Entering assignment [name]" or "Entering All Games Library" when mode switches

#### Keyboard Navigation
- **Tab Order**: Mode indicators focusable for screen reader users
- **Shortcuts**: Distinct shortcuts for "Return to Assignment" (Ctrl+A) vs. "Return to Library" (Ctrl+L)
- **Skip Links**: "Skip to assignment steps" or "Skip to game grid" based on mode

#### High Contrast and Dark Mode
- Mode indicators must maintain sufficient contrast in both light and dark themes
- Free play vs. assigned color coding must not rely solely on color (use icons, text labels)

### Responsive Behavior

#### Mobile View Adaptations
- **Mode Indicator**: Condenses to icon + label at top of screen
- **Score Display**: Stacks vertically with clear hierarchy
- **Navigation**: Hamburger menu clearly shows current mode and transition options
- **Reconciliation Badge**: Scales to smaller size but remains visible

#### Tablet View
- Hybrid layout maintaining key indicators from desktop
- Side-by-side score displays where space permits
- Touch-optimized mode switching

### Error States

#### Reconciliation Failures
If reconciliation fails or is rejected by policy:
```
┌─────────────────────────────────────┐
│ ⚠ Staff Birds - Quiz                │
│                                     │
│ Free Play Score: 85%                │
│ Fresh attempt required by teacher   │
│                                     │
│ [Start Assignment Version]          │
└─────────────────────────────────────┘
```
- Clear explanation of why reconciliation didn't apply
- Guidance on required action
- Link to start fresh attempt in assigned context

#### Mode Confusion Recovery
If student appears confused about mode (analytics detect back-and-forth navigation):
- Subtle educational tooltip: "Tip: You're in Free Play mode. Scores here won't affect your assignment progress unless reconciliation is enabled."
- Dismissible, non-blocking
- Appears once per session maximum

## Telemetry

All game session events must include a `play_context` property to enable segmented analysis of free play vs. assigned behavior. See [Event Model](../15-analytics-and-reporting/event-model.md) for complete event schemas and canonical property names.

### Context Tagging

**Every game-related event** must include:
```typescript
{
  play_context: 'free_play' | 'assigned',
  assignment_id?: string,      // Present only for 'assigned' context
  step_id?: string,             // Present only for 'assigned' context
  sequence_id?: string          // Present only for 'assigned' context
}
```

### Free Play Events

#### Free Play Session Start
```typescript
event: 'free_play_session_start'
properties: {
  play_context: 'free_play',
  session_id: string,
  student_id: string,
  game_id: string,
  stage_id: string,
  discovery_path: 'browse' | 'search' | 'favorite' | 'recommended',
  device_profile: string,
  input_mode: 'keyboard' | 'midi' | 'mic' | 'touch',
  target_score: number,         // From game registry default
  previous_best_score?: number  // If student has played before
}
```

#### Free Play Session Complete
```typescript
event: 'free_play_session_complete'
properties: {
  play_context: 'free_play',
  session_id: string,
  student_id: string,
  game_id: string,
  stage_id: string,
  score: number,
  max_score: number,
  passed: boolean,
  target_score: number,
  duration_ms: number,
  is_new_best_score: boolean,
  previous_best_score?: number,
  score_improvement?: number    // Difference from previous best
}
```

#### Free Play Discovery
```typescript
event: 'free_play_game_discovered'
properties: {
  play_context: 'free_play',
  student_id: string,
  game_id: string,
  discovery_method: 'search' | 'browse' | 'recommendation' | 'related_game',
  search_query?: string,        // If discovered via search
  filter_tags?: string[],       // Active filters when discovered
  position_in_results?: number  // Position in grid/list
}
```

### Assigned Play Events

#### Assigned Session Start
```typescript
event: 'assigned_session_start'
properties: {
  play_context: 'assigned',
  session_id: string,
  assignment_id: string,
  step_id: string,
  sequence_id: string,
  sequence_version: string,
  student_id: string,
  game_id: string,
  stage_id: string,
  target_score: number,         // From policy hierarchy
  gates_met: string[],          // Which gates were satisfied
  optional_step: boolean,
  retry_count: number,          // Attempts on this step
  device_profile: string,
  input_mode: 'keyboard' | 'midi' | 'mic' | 'touch'
}
```

#### Assigned Session Complete
```typescript
event: 'assigned_session_complete'
properties: {
  play_context: 'assigned',
  session_id: string,
  assignment_id: string,
  step_id: string,
  sequence_id: string,
  student_id: string,
  game_id: string,
  stage_id: string,
  score: number,
  max_score: number,
  passed: boolean,
  target_score: number,
  duration_ms: number,
  attempt_number: number,       // Attempt on THIS step
  step_completed: boolean,      // Did this attempt complete the step?
  assignment_progress_before: number,
  assignment_progress_after: number
}
```

### Reconciliation Events

#### Free Play Reconciliation Attempted
```typescript
event: 'free_play_reconciliation_attempted'
properties: {
  student_id: string,
  assignment_id: string,
  step_id: string,
  free_play_session_id: string,
  free_play_score: number,
  free_play_date: Date,
  target_score: number,
  threshold_met: boolean,
  reconciliation_policy: string,
  policy_checks: {
    require_fresh_attempt: boolean,
    score_threshold_met: boolean,
    within_time_window: boolean,
    stage_reconciliation_allowed: boolean
  },
  reconciliation_allowed: boolean  // Overall result
}
```

#### Free Play Reconciliation Completed
```typescript
event: 'free_play_reconciliation_completed'
properties: {
  reconciliation_id: string,
  student_id: string,
  assignment_id: string,
  step_id: string,
  free_play_session_id: string,
  free_play_score: number,
  free_play_date: Date,
  reconciled_at: Date,
  days_since_free_play: number,
  step_marked_complete: boolean,
  assignment_progress_before: number,
  assignment_progress_after: number,
  next_step_unlocked: boolean
}
```

#### Reconciliation Override
```typescript
event: 'reconciliation_override'
properties: {
  reconciliation_id: string,
  student_id: string,
  assignment_id: string,
  step_id: string,
  teacher_id: string,           // Who performed override
  override_action: 'require_fresh_attempt' | 'force_reconcile',
  reason?: string,              // Teacher-provided reason
  original_completion_source: string
}
```

### Mode Transition Events

#### Context Switch
```typescript
event: 'play_context_switch'
properties: {
  student_id: string,
  from_context: 'free_play' | 'assigned' | 'dashboard',
  to_context: 'free_play' | 'assigned',
  navigation_path: string,      // How student navigated
  from_assignment_id?: string,  // If leaving assigned context
  to_assignment_id?: string     // If entering assigned context
}
```

### Telemetry Use Cases

#### Analytics Queries Enabled by Context Tagging

**Free Play Engagement Analysis**:
```sql
SELECT game_id, COUNT(*) as plays, AVG(score) as avg_score
FROM free_play_sessions
WHERE play_context = 'free_play'
AND recorded_at > NOW() - INTERVAL '30 days'
GROUP BY game_id
ORDER BY plays DESC
```

**Assigned vs Free Play Performance Comparison**:
```sql
SELECT 
  game_id,
  AVG(CASE WHEN play_context = 'free_play' THEN score END) as free_play_avg,
  AVG(CASE WHEN play_context = 'assigned' THEN score END) as assigned_avg
FROM score_records
GROUP BY game_id
```

**Reconciliation Success Rate**:
```sql
SELECT 
  COUNT(*) as attempts,
  SUM(CASE WHEN reconciliation_allowed THEN 1 ELSE 0 END) as successful,
  AVG(free_play_score - target_score) as score_margin
FROM free_play_reconciliation_attempted
WHERE recorded_at > NOW() - INTERVAL '7 days'
```

**Discovery to Assignment Funnel**:
```sql
-- Students who discovered game in free play, then encountered in assignment
SELECT 
  f.student_id,
  f.game_id,
  f.recorded_at as free_play_date,
  a.created_at as assignment_date,
  DATEDIFF(a.created_at, f.recorded_at) as days_between
FROM free_play_sessions f
JOIN assigned_sessions a ON f.student_id = a.student_id AND f.game_id = a.game_id
WHERE a.created_at > f.recorded_at
```

### Sampling Strategy

All context-distinguishing events are **never sampled** (100% capture) because:
- Reconciliation logic requires complete historical data
- Teacher reporting depends on accurate completion sources
- Free play vs. assigned performance analysis is a core product metric
- Audit trails for overrides must be complete

**Exceptions** (may be sampled at 10% in production):
- `free_play_game_discovered` for popular games (not critical for features)
- Performance timing metrics for game load (monitoring only)

### Event Destinations

- **Real-time stream**: Assignment completion events for immediate progress updates
- **Analytics warehouse**: All free play and assigned events for reporting and analysis
- **Audit log**: Reconciliation overrides and policy changes
- **Notification service**: Assignment completion events that trigger student/teacher notifications

## Permissions

### Student Permissions

#### Free Play Access
- **Read**: All non-deprecated games in the catalog (subject to subscription tier)
- **Play**: Any stage in any order without gating
- **Write**: Create free play sessions and scores (auto-recorded)
- **View**: Own historical free play scores and best scores

**Restrictions**:
- Cannot view other students' free play activity (privacy protection)
- Cannot modify recorded scores (integrity)
- Cannot access deprecated games (quality control)

#### Assigned Play Access
- **Read**: Only games within current assignments
- **Play**: Only stages that meet gating requirements
- **Write**: Create assigned sessions and scores (auto-recorded)
- **View**: Own assignment progress and step completion

**Restrictions**:
- Cannot bypass gates without teacher override
- Cannot modify target scores or policy settings
- Cannot mark steps complete manually (earned through play)
- Cannot view other students' assignment progress

### Teacher Permissions

#### Free Play Management
- **View**: Can see students' free play activity, scores, and discovery patterns
- **Analyze**: Can generate reports on free play engagement and performance
- **No Write**: Cannot control or restrict free play access (by design for student agency)

**Rationale**: Free play remains student-controlled to support intrinsic motivation and exploration.

#### Assigned Play Management
- **Create**: Generate assignments for students in their classes
- **Configure**: Set target scores, gates, and reconciliation policies
- **Override**: Manually mark steps complete or require fresh attempts
- **View**: See detailed progress, attempts, and completion sources
- **Modify**: Adjust due dates, optional status, and step visibility

**Restrictions**:
- Cannot modify assignments for students outside their classes
- Cannot override scores (only completion status)
- All overrides create audit log entries
- Cannot disable reconciliation globally (admin-level policy)

#### Reconciliation Controls
- **View**: See which steps were completed via reconciliation with source details
- **Override**: Require fresh attempt even if free play score qualified
- **Approve**: If `require_teacher_approval` policy is set, approve reconciliation
- **Reject**: Reject reconciliation and require assigned attempt

**Audit Trail**: Every teacher override is logged with:
- Teacher ID and timestamp
- Student and assignment affected
- Override action and reason (if provided)
- Previous and new state

### Admin (Subscriber-Admin) Permissions

#### Policy Configuration
- **Set Class Policies**: Configure reconciliation rules for classes in their organization
- **View Aggregate Data**: See organization-wide free play and assigned metrics
- **Manage Subscriptions**: Control which games/features are available (tier management)
- **Generate Reports**: Export bulk progress data for compliance or analysis

**Restrictions**:
- Cannot override individual student steps (teacher responsibility)
- Cannot disable free play entirely (core product feature)
- Cannot modify historical scores or sessions

#### Reconciliation Policy Control
- **Configure**: Set default reconciliation policies for organization or classes
- **Review**: See reconciliation success rates and policy impact analytics
- **Adjust**: Modify `require_fresh_attempt`, time windows, and stage filters

### System Admin Permissions

#### Full Access (for Support and Diagnostics)
- **View**: All data across organizations (with audit logging)
- **Repair**: Fix data inconsistencies or reconciliation errors
- **Override**: Can perform any teacher or admin action for support purposes
- **Configure**: System-wide defaults and feature flags

**Audit Requirements**:
- Every SysAdmin action creates detailed audit log
- Access requires justification and is time-limited
- PII access follows strict compliance protocols

#### Permissions Summary Table

| Action | Student | Teacher | Admin | SysAdmin |
|--------|---------|---------|-------|----------|
| **Free Play** | | | | |
| Play any game | ✅ | ❌ | ❌ | ❌ (impersonate only) |
| View own free play scores | ✅ | ❌ | ❌ | ✅ (support) |
| View others' free play activity | ❌ | ✅ (own classes) | ✅ (own org) | ✅ (all) |
| **Assigned Play** | | | | |
| Play assigned games | ✅ | ❌ | ❌ | ❌ (impersonate only) |
| Create assignments | ❌ | ✅ (own classes) | ✅ (own org) | ✅ (all) |
| Override step completion | ❌ | ✅ (own classes) | ❌ | ✅ (support) |
| View assignment progress | ✅ (own) | ✅ (own classes) | ✅ (own org) | ✅ (all) |
| **Reconciliation** | | | | |
| Benefit from reconciliation | ✅ | ❌ | ❌ | ❌ |
| View reconciliation source | ✅ (own) | ✅ (students) | ✅ (org) | ✅ (all) |
| Require fresh attempt | ❌ | ✅ (own classes) | ❌ | ✅ (support) |
| Configure reconciliation policy | ❌ | ❌ | ✅ (own org) | ✅ (all) |
| **Reporting** | | | | |
| View own analytics | ✅ | ✅ | ✅ | ✅ |
| Export student data | ❌ | ✅ (own classes) | ✅ (own org) | ✅ (all) |
| Compare free play vs assigned | ❌ | ✅ (own classes) | ✅ (own org) | ✅ (all) |

## Dependencies

### Internal Specifications
This specification relies on and integrates with these core MLC documents:

#### Assignment System
- **[Assignment Generation](./assignment-generation.md)**: How assignments are created from sequences and when reconciliation first occurs
- **[Assignment Progress Tracking](./assignment-progress-tracking.md)**: How completion from both contexts contributes to progress calculations
- **[Retention and Review Logic](./retention-review-logic.md)**: How review scheduling works for steps completed via reconciliation

#### Content Architecture
- **[Assignment Model](../07-content-architecture/assignment-model.md)**: Core data structures for assignments and steps
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Stage definitions, target scores, and game lifecycle
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: How sequences define learning progressions

#### Student Experience
- **[All Games Library](../03-student-experience/all-games-library.md)**: Free play entry point and game discovery
- **[Assignment Play](../03-student-experience/assignment-play.md)**: Assigned mode interface and progression
- **[Student Dashboard](../03-student-experience/student-dashboard.md)**: How both modes are surfaced to students

#### Teacher Experience
- **[Assignment Builder](../04-teacher-experience/assignment-builder.md)**: How teachers create assignments and set policies
- **[Teacher Reports](../04-teacher-experience/teacher-reports.md)**: How completion sources are displayed in reports

#### System Foundation
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Canonical telemetry naming and schemas
- **[Roles and Permissions](../02-roles-and-permissions/roles-matrix.md)**: Role definitions and access control
- **[Games Registry](../08-games-registry-and-authoring/games-registry-overview.md)**: Game availability and default targets

### External Systems
- **Database**: PostgreSQL or similar for transactional session and score storage
- **Cache**: Redis for fast best-score lookups during reconciliation checks
- **Analytics Pipeline**: Real-time stream processing for immediate reconciliation triggers
- **Notification Service**: Alerts for assignment completion from reconciliation

### Critical Integration Points

#### With Assignment Generation
- **Trigger**: Assignment generation must query free play history for reconciliation
- **Data Flow**: `free_play_sessions` → reconciliation check → `step.state = complete`
- **Policy**: Class-level `ReconciliationPolicy` controls behavior

#### With Progress Tracking
- **Trigger**: Any score recording (free play or assigned) may update progress
- **Data Flow**: Session complete → score record → progress recalculation
- **Context**: Progress tracking must respect `play_context` and `completion_source`

#### With Student Dashboard
- **Display**: Dashboard must show both free play activity and assignment progress
- **Navigation**: Clear transitions between All Games and My Assignments
- **Indicators**: Completion source badges for reconciled steps

## Open questions

### Reconciliation Policy Defaults

**Question**: Should free play reconciliation be **enabled by default** for all classes, or should it be opt-in?

**Considerations**:
- **Enabled by default**: Reduces student friction, respects prior learning, encourages free play engagement
- **Opt-in**: Gives teachers explicit control, ensures assessment integrity for formal evaluation contexts
- **Current recommendation**: Enabled by default for Learn and Play stages, disabled by default for Quiz and Review stages

**Decision needed by**: Phase 1 implementation kickoff

---

**Question**: What should the default `reconciliation_window_days` be? Should there be a time limit on how old a free play score can be?

**Considerations**:
- **No time limit (null)**: Maximum flexibility, respects all prior learning regardless of age
- **30-90 day window**: Balances currency of knowledge with practicality
- **No default, admin must set**: Forces intentional policy decision per organization
- **Current recommendation**: No default time limit (null), but allow admins to set if desired

**Decision needed by**: Phase 1 implementation kickoff

---

### Quiz Reconciliation

**Question**: Should Quiz stage reconciliation be available, and if so, under what conditions?

**Considerations**:
- **Never allow**: Maintains strongest assessment integrity, ensures teacher visibility into formal evaluation
- **Allow with teacher approval**: Compromise that requires explicit teacher sign-off
- **Allow but disabled by default**: Available for flexible classrooms but requires opt-in
- **Context matters**: Home/solo learners may have different needs than classroom settings
- **Current recommendation**: Available but disabled by default, can be enabled per class with admin permission

**Decision needed by**: Phase 1 Beta testing with teacher feedback

---

### Score Threshold Multipliers

**Question**: Should we support `reconciliation_score_multiplier` (e.g., requiring 90% free play score for 80% assignment target)?

**Considerations**:
- **Pro**: Accounts for context differences (free play may be lower stakes, assigned may have distractions)
- **Con**: Adds complexity to policy system and may be hard for teachers to understand
- **Alternative**: Let teachers adjust target scores directly in assignment overrides
- **Current recommendation**: Phase 2 feature if user research shows need

**Decision needed by**: Post-Phase 1 based on actual reconciliation data and teacher feedback

---

### Real-Time vs. Batch Reconciliation

**Question**: When should reconciliation checks run?

**Options**:
1. **Real-time**: Check immediately when free play session completes (if student has active assignments)
2. **On assignment load**: Check when student opens assignment for first time
3. **Nightly batch**: Run reconciliation job every night for all active assignments
4. **Hybrid**: Real-time for critical events + nightly sweep for edge cases

**Considerations**:
- Real-time provides immediate feedback but adds latency to score recording
- Batch is simpler but creates delay in progress visibility
- Hybrid offers best of both but increases system complexity
- **Current recommendation**: Hybrid approach with real-time for assignment generation and nightly batch for recent free play

**Decision needed by**: Phase 1 architecture review

---

### Cross-Mode Best Score Display

**Question**: When displaying "best score" in free play mode, should we include scores achieved in assigned mode?

**Scenario**: Student scores 85% on "Staff Birds - Quiz" in assignment, then plays same game in free play mode and sees "Your Best: 85%" vs. "No free play score yet"

**Options**:
1. **Unified best score**: Show highest score from any context (more encouraging)
2. **Context-specific best**: Only show free play scores in free play mode (clearer separation)
3. **Show both**: "Assignment Best: 85%, Free Play Best: 78%" (most information, more complex)

**Current recommendation**: Option 3 (show both) with clear labeling

**Decision needed by**: UX design review in Phase 1

---

### Teacher Notification on Reconciliation

**Question**: Should teachers receive notifications when reconciliation completes steps?

**Considerations**:
- **Pro**: Teachers stay informed, can intervene if needed
- **Con**: High volume if many students benefit from reconciliation, potential notification fatigue
- **Options**: 
  - No notification (passive, teacher discovers in reports)
  - Digest notification (daily or weekly summary)
  - Real-time notification (immediate but potentially overwhelming)
  - Configurable per teacher
- **Current recommendation**: Weekly digest showing reconciliation activity across all classes

**Decision needed by**: Phase 1 notification system design

---

### Retroactive Reconciliation

**Question**: If an admin changes reconciliation policy (e.g., enables Quiz reconciliation), should we retroactively reconcile existing assignments?

**Considerations**:
- **Pro**: Applies new policy fairly to all students, reduces manual work
- **Con**: May contradict teacher intent if policy change happens mid-term
- **Options**:
  - Never retroactive (new policy only applies to new assignments)
  - Retroactive with teacher opt-in (show preview, let teacher approve)
  - Retroactive for new assignments only (policy change doesn't affect in-progress work)
- **Current recommendation**: Option 3 (new assignments only), with manual reconciliation tool available for teachers

**Decision needed by**: Phase 1 policy management implementation

---

### Free Play in Subscription Tiers

**Question**: Should free play access differ by subscription tier (Free Trial, Solo, Ensemble)?

**Current State** (from original MLC):
- Free Trial: Limited games available
- Solo/Ensemble: Full catalog access

**Considerations**:
- Should free trial users see locked games with upgrade prompts, or hide unavailable games entirely?
- Should free play scores from trial period carry over after subscription?
- **Current recommendation**: Show all games with clear trial/premium badges, scores always persist for reconciliation if user upgrades

**Decision needed by**: Phase 1 subscription integration

---

### Historical Data Migration

**Question**: How should we handle existing MLC-4.0 scores when migrating to MLC-5.0?

**Considerations**:
- Millions of historical free play sessions and scores exist
- Assignment context may not exist for older scores (pre-Sequences)
- Data quality varies (some known score recording failures)
- **Options**:
  - Migrate all historical scores with `play_context: 'free_play'` (assume no assignment context unless provable)
  - Migrate only recent scores (last 12 months)
  - Let users opt-in to historical data import
- **Current recommendation**: Migrate all scores as free play, flag for reconciliation eligibility on first MLC-5.0 assignment

**Decision needed by**: Migration planning (pre-Phase 1)

---

### Performance at Scale

**Question**: What are acceptable latency targets for reconciliation checks?

**Scenarios to consider**:
- Assignment generation with 50-step sequence for student with 500+ historical free play scores
- Nightly batch reconciliation across 10,000 active students
- Real-time reconciliation check when student completes free play session with 5 active assignments

**Current targets** (to be validated):
- Individual reconciliation check: < 100ms
- Assignment generation with reconciliation: < 2 seconds total
- Nightly batch: Complete within 4-hour maintenance window

**Decision needed by**: Performance testing in Phase 1 beta

---

### Audit Log Retention

**Question**: How long should reconciliation audit logs be retained?

**Considerations**:
- FERPA/COPPA compliance may require extended retention
- Storage costs for high-volume audit data
- Support and dispute resolution needs
- **Current recommendation**: 7 years (matches typical educational record retention), with option to extend per organization policy

**Decision needed by**: Compliance review before Phase 1 launch
