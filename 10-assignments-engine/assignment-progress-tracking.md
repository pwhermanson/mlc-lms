# Assignment Progress Tracking

This document defines how assignment progress is calculated, stored, aggregated, and reported across the MLC system. While the [Assignment Model](../07-content-architecture/assignment-model.md) defines the core data structures and state transitions, and the [Assignment Generation Engine](./assignment-generation.md) defines how assignments are created, this specification focuses specifically on **progress calculation, aggregation, history tracking, and reporting interfaces** that enable students, teachers, and admins to understand learning advancement.

## Purpose

Assignment progress tracking enables comprehensive visibility into student learning advancement at multiple granularities (step, assignment, sequence, student, class, organization). This specification provides the foundation for:

- **Student Motivation**: Clear, real-time visibility into personal progress and achievement
- **Teacher Insights**: Actionable data for identifying struggling students, pacing adjustments, and intervention needs
- **Admin Analytics**: Aggregate progress metrics for curriculum effectiveness and resource allocation
- **Parent Communication**: Transparent, accessible reports on student learning advancement
- **System Intelligence**: Progress-based triggers for remediation, review scheduling, and adaptive learning paths

This system addresses critical pain points identified in the original MLC system, where score recording failures and unclear progression indicators led to student confusion, teacher visibility gaps, and lost engagement opportunities.

## Scope

### Included
- **Progress Calculation Rules**: How completion percentages, mastery levels, and progress metrics are computed at each granularity
- **Progress State Storage**: Data structures for current progress, historical snapshots, and audit trails
- **Progress Aggregation**: Rules for rolling up step → assignment → sequence → student → class progress
- **Progress Reporting Interfaces**: APIs and views for different roles accessing progress data
- **Progress History and Snapshots**: Point-in-time progress states for trend analysis and rollback
- **Progress-based Triggers**: Conditions that trigger interventions, notifications, or adaptive behavior
- **Batch Progress Operations**: Efficient computation for analytics, reporting, and bulk queries
- **Progress Reconciliation**: Handling progress when assignments change, content is deprecated, or errors occur

### Excluded
- **Assignment Creation**: How assignments are generated (covered in [Assignment Generation](./assignment-generation.md))
- **Assignment Structure**: Core data models and relationships (covered in [Assignment Model](../07-content-architecture/assignment-model.md))
- **Game Scoring Mechanics**: How individual game scores are calculated (covered in [Game Model](../08-games-registry-and-authoring/game-model.md))
- **Student UI Layouts**: Specific interface designs (covered in [Student Assignments](../03-student-experience/student-assignments.md))
- **Detailed Analytics Dashboards**: KPI visualizations and report layouts (covered in [Teacher Reports](../04-teacher-experience/teacher-reports.md) and [Admin Reports](../05-admin-subscriber-experience/admin-reports.md))

## Data model (if applicable)

### Progress State Entities

#### Step Progress
Tracks progress for an individual step within an assignment.

```typescript
interface StepProgress {
  step_id: string;                    // Reference to Step entity
  assignment_id: string;              // Parent assignment
  student_id: string;                 // Student performing the step
  
  // Current state (see Assignment Model for state definitions)
  state: 'locked' | 'available' | 'in_progress' | 'complete';
  
  // Completion tracking
  started_at?: Date;                  // First interaction timestamp
  completed_at?: Date;                // Completion timestamp
  last_activity_at?: Date;            // Most recent interaction
  
  // Performance metrics
  attempt_count: number;              // Total attempts made
  best_score: number;                 // Highest score achieved
  latest_score: number;               // Most recent score
  target_score: number;               // Required score from policy
  passed: boolean;                    // Whether target was met
  
  // Time tracking
  time_on_task_ms: number;            // Cumulative active time
  session_count: number;              // Number of separate sessions
  
  // Mastery indicators
  mastery_level: 'not_started' | 'learning' | 'proficient' | 'mastered';
  consistency_score?: number;         // Score variance across attempts (lower is better)
  
  // Source tracking
  completion_source: 'fresh_attempt' | 'free_play_reconciled' | 'teacher_override' | 'system_migration';
  reconciliation_metadata?: {
    original_score: number;
    original_context: 'all_games' | 'previous_assignment';
    reconciled_at: Date;
  };
}
```

#### Assignment Progress
Aggregated progress for an entire assignment.

```typescript
interface AssignmentProgress {
  assignment_id: string;              // Reference to Assignment entity
  student_id: string;                 // Student performing the assignment
  
  // Overall completion
  status: 'not_started' | 'in_progress' | 'complete' | 'archived';
  completion_percentage: number;      // 0-100, based on required steps only
  steps_total: number;                // Total steps in assignment
  steps_required: number;             // Required steps (non-optional)
  steps_completed: number;            // Steps in 'complete' state
  steps_in_progress: number;          // Steps in 'in_progress' state
  steps_locked: number;               // Steps in 'locked' state
  
  // Time tracking
  started_at?: Date;                  // First step interaction
  completed_at?: Date;                // All required steps complete
  estimated_completion_date?: Date;   // Projected completion based on pacing
  time_on_task_ms: number;            // Cumulative time across all steps
  
  // Performance metrics
  average_score: number;              // Mean score across completed steps
  mastery_score: number;              // Weighted mastery level (0-100)
  pass_rate: number;                  // Percentage of steps passed on first attempt
  
  // Next up pointer
  next_step_id?: string;              // Next step to work on (null if complete)
  next_step_due_at?: Date;            // Due date for next step
  
  // Progress velocity
  steps_per_week: number;             // Rolling average completion rate
  pacing_status: 'ahead' | 'on_track' | 'behind' | 'at_risk';
}
```

#### Sequence Progress
Tracks progress through an entire sequence (multiple assignments).

```typescript
interface SequenceProgress {
  sequence_id: string;                // Reference to Sequence
  student_id: string;                 // Student progressing through sequence
  
  // Overall sequence completion
  completion_percentage: number;      // 0-100
  assignments_total: number;          // Total assignments in sequence
  assignments_completed: number;      // Completed assignments
  current_assignment_id?: string;     // Assignment currently in progress
  
  // Time tracking
  started_at: Date;                   // First assignment interaction
  estimated_completion_date?: Date;   // Projected sequence completion
  
  // Mastery indicators
  overall_mastery_score: number;      // Weighted mastery across all assignments
  concepts_mastered: string[];        // List of concept IDs mastered
  concepts_in_progress: string[];     // Concepts being learned
  
  // Remediation tracking
  remediation_count: number;          // Number of remediation inserts triggered
  review_compliance_rate: number;     // Percentage of review steps completed on time
}
```

#### Progress Snapshot
Point-in-time snapshot of progress for trend analysis and audit trails.

```typescript
interface ProgressSnapshot {
  snapshot_id: string;                // Unique snapshot identifier
  snapshot_date: Date;                // Date of snapshot
  student_id: string;                 // Student whose progress is captured
  
  // Snapshot scope
  scope_type: 'step' | 'assignment' | 'sequence' | 'student';
  scope_id: string;                   // ID of the scoped entity
  
  // Captured state
  state_snapshot: object;             // Full progress state at snapshot time
  metrics_snapshot: {
    completion_percentage: number;
    mastery_score: number;
    time_on_task_ms: number;
    [key: string]: any;               // Additional metrics
  };
  
  // Context
  trigger: 'scheduled' | 'manual' | 'milestone' | 'intervention';
  notes?: string;                     // Optional annotation
}
```

### Progress Aggregation Tables

#### Student Progress Summary
Aggregated progress across all active learning.

```typescript
interface StudentProgressSummary {
  student_id: string;                 // Student identifier
  organization_id: string;            // For multi-tenant filtering
  
  // Overall activity
  active_assignments: number;         // Assignments in progress
  completed_assignments: number;      // Completed assignments (all-time)
  total_time_on_task_ms: number;      // Cumulative learning time
  
  // Current week metrics
  steps_completed_this_week: number;
  time_on_task_this_week_ms: number;
  games_played_this_week: number;
  
  // Overall mastery
  overall_mastery_score: number;      // Weighted average across all content
  concepts_mastered_count: number;
  badges_earned: number;
  
  // Engagement indicators
  last_activity_at: Date;             // Most recent learning activity
  streak_days: number;                // Consecutive days with activity
  engagement_score: number;           // Composite engagement metric (0-100)
  
  // Risk indicators
  at_risk: boolean;                   // Flag for struggling students
  risk_factors: string[];             // Reasons for at-risk status
}
```

## Behavior and rules

### Progress Calculation Rules

#### Step Completion
A step is considered "complete" when:
1. Student has achieved `best_score >= target_score`, OR
2. Teacher has overridden the step to complete, OR
3. Free play score was reconciled and met threshold, OR
4. Step was marked optional and student has moved past it

**Mastery Level Calculation** (per step):
```pseudo
if (attempt_count == 0) {
  mastery_level = 'not_started'
} else if (best_score < target_score * 0.7) {
  mastery_level = 'learning'
} else if (best_score < target_score) {
  mastery_level = 'proficient'
} else if (best_score >= target_score && consistency_score < 0.15) {
  mastery_level = 'mastered'
} else {
  mastery_level = 'proficient'
}
```

**Consistency Score** measures score variance across attempts:
```pseudo
consistency_score = standard_deviation(scores) / mean(scores)
```
Lower values indicate more consistent performance (better mastery).

#### Assignment Completion Percentage
```pseudo
completion_percentage = (steps_completed / steps_required) * 100
```
- Optional steps are excluded from the denominator
- Pre-completed steps (via reconciliation) count as completed
- In-progress steps do NOT contribute partial credit

#### Assignment Mastery Score
Weighted average that accounts for both completion and performance:
```pseudo
mastery_score = sum(step_mastery_weight * step_score) / sum(step_mastery_weight)

where step_mastery_weight varies by stage:
- LEARN: 0.5
- PLAY: 1.0
- QUIZ: 2.0
- CHALLENGE: 1.5
- REVIEW: 1.0
```
This weighting prioritizes assessment stages (QUIZ) while still valuing practice.

#### Pacing Status Determination
Based on comparison between actual and expected progress:
```pseudo
expected_completion_date = assignment.created_at + estimated_duration
days_until_due = assignment.due_at - today
expected_percentage_by_now = (days_elapsed / total_days) * 100
actual_percentage = completion_percentage

if (actual_percentage >= expected_percentage_by_now + 20%) {
  pacing_status = 'ahead'
} else if (actual_percentage >= expected_percentage_by_now - 10%) {
  pacing_status = 'on_track'
} else if (actual_percentage >= expected_percentage_by_now - 30%) {
  pacing_status = 'behind'
} else {
  pacing_status = 'at_risk'
}
```

### Progress Update Triggers

#### Real-time Updates
Progress is recalculated and persisted when:
- **Step completed**: Student finishes a game stage
- **Step started**: Student begins a new step
- **Score improved**: Student achieves a new best score
- **Teacher override**: Teacher manually marks step complete or adjusts target
- **Free play reconciliation**: System detects a qualifying free play score

#### Batch Updates
Progress aggregations are recomputed on a schedule:
- **Student summary**: Updated every 5 minutes for active students, hourly for inactive
- **Class summaries**: Updated every 15 minutes
- **Organization dashboards**: Updated hourly
- **Historical snapshots**: Created daily at midnight UTC

### Progress History and Snapshots

#### Automatic Snapshot Triggers
Snapshots are automatically created when:
- **Assignment started**: Captures baseline state
- **Assignment completed**: Captures final state
- **Milestone reached**: Every 25% completion (25%, 50%, 75%, 100%)
- **Weekly rollup**: Every Sunday at midnight UTC
- **Intervention triggered**: When at-risk status changes
- **Teacher review**: When teacher views detailed progress report

#### Snapshot Retention Policy
- **Step-level snapshots**: Retained for 90 days, then aggregated
- **Assignment-level snapshots**: Retained for 1 year
- **Sequence-level snapshots**: Retained for 3 years
- **Student summary snapshots**: Retained indefinitely (aggregated monthly after 1 year)

### Progress Reconciliation Rules

#### When Assignments Change
If an assignment is updated (e.g., sequence version migrates):
1. **Preserve completed progress**: Steps already complete remain complete
2. **Re-map in-progress steps**: Match by `element_id` and `stage` to new template
3. **Handle removed steps**: Mark as complete with `completion_source = 'system_migration'`
4. **Handle added steps**: Insert as locked/available based on current position
5. **Recalculate percentages**: Update completion percentage based on new structure
6. **Create audit snapshot**: Capture pre-migration state for rollback

#### When Content is Deprecated
If a game or stage referenced in an assignment is deprecated:
1. **Check for replacement**: If `replaced_by_game_id` exists, offer migration
2. **Mark as optional**: If no replacement, mark step optional (doesn't block progression)
3. **Adjust completion calculation**: Exclude deprecated steps from required count
4. **Notify teacher**: Send notification about content change and impact
5. **Preserve existing scores**: Historical scores remain in progress history

#### When Errors Occur
If progress data becomes inconsistent:
1. **Detect inconsistency**: Monitor for impossible states (e.g., 110% complete, negative time)
2. **Trigger audit**: Log inconsistency with full context for investigation
3. **Attempt repair**: Use repair rules to fix common issues (e.g., recalculate from step data)
4. **Notify support**: If repair fails, alert support team with diagnostic data
5. **Preserve raw data**: Never delete original data, only add correction records

### Progress-based Interventions

#### At-Risk Student Detection
A student is flagged as "at-risk" if any of these conditions are met:
- **Pacing**: More than 30% behind expected progress
- **Mastery**: Average mastery score < 60% across last 5 completed steps
- **Engagement**: No activity in last 7 days with active assignments
- **Struggle**: More than 5 attempts on same step without passing
- **Consistency**: High score variance indicating guessing or lack of understanding

**Risk Factor Examples**:
- `"behind_pace_40_percent"`: 40% behind expected progress
- `"low_mastery_avg_52"`: Average mastery score of 52%
- `"inactive_9_days"`: No activity in 9 days
- `"stuck_on_step_7_attempts"`: 7 attempts on same step without passing

#### Intervention Triggers
When a student is flagged as at-risk:
1. **Teacher notification**: Email/in-app alert with specific risk factors
2. **Remediation recommendation**: Suggest specific games or concepts to review
3. **Pacing adjustment**: Offer to extend due dates or reduce assignment scope
4. **Support resources**: Recommend help videos or tutoring resources
5. **Parent communication**: Generate parent-friendly progress report

## UX requirements (if applicable)

### Student Progress Views

#### Progress Dashboard Widget
- **Circular progress indicator**: Shows overall completion percentage
- **Next step highlight**: Clear "Continue" button pointing to next step
- **Recent activity timeline**: Last 5-10 learning activities with scores
- **Mastery level badges**: Visual indicators for mastered concepts
- **Streak indicator**: Days of consecutive learning activity

#### Assignment Progress Detail
- **Linear progress bar**: Step-by-step visual showing completed/in-progress/locked steps
- **Step cards**: Individual cards showing score, attempts, time spent, and mastery level
- **Performance graph**: Line chart showing score improvement over time
- **Time summary**: Total time on assignment, average time per step
- **Next up section**: Highlighted next step with description and target score

### Teacher Progress Views

#### Class Progress Overview
- **Student roster grid**: Rows for students, columns for assignments, cells showing completion percentage
- **Color coding**: Green (ahead/complete), Yellow (on-track), Orange (behind), Red (at-risk)
- **Sort and filter**: By completion, mastery score, last activity, pacing status
- **Quick actions**: One-click to view details, send message, adjust due date

#### Student Progress Detail (Teacher View)
- **Progress timeline**: Chronological view of all student activity
- **Mastery heatmap**: Grid showing mastery level for each concept
- **Struggle indicators**: Highlighted steps with many attempts or low scores
- **Intervention history**: Log of all interventions triggered and actions taken
- **Override controls**: Buttons to mark steps complete, adjust targets, extend deadlines

### Admin Progress Views

#### Organization Dashboard
- **Aggregate metrics**: Total students, active assignments, completion rate, average mastery
- **Trend graphs**: Progress metrics over time (week, month, quarter)
- **Class comparison**: Side-by-side comparison of class performance
- **Content effectiveness**: Which assignments/sequences have highest/lowest completion rates
- **Engagement metrics**: Active users, time on task, streak persistence

### Accessibility Requirements
- **Screen reader support**: All progress metrics announced clearly
- **Keyboard navigation**: Tab through progress cards, Enter to drill down
- **High contrast mode**: Progress indicators visible in high contrast settings
- **Reduced motion**: Option to disable animated progress bars
- **Text scaling**: Progress percentages remain readable at 200% zoom

## Telemetry

### Progress Events

#### Core Progress Events
```typescript
// Fired when step progress is updated
event: 'step_progress_updated'
properties: {
  step_id: string
  assignment_id: string
  student_id: string
  previous_state: string
  new_state: string
  score_achieved: number
  target_score: number
  passed: boolean
  attempt_count: number
  time_on_task_ms: number
  mastery_level: string
  completion_source: string
}

// Fired when assignment completion percentage changes significantly
event: 'assignment_progress_milestone'
properties: {
  assignment_id: string
  student_id: string
  previous_percentage: number
  new_percentage: number
  milestone: '25' | '50' | '75' | '100'
  time_to_milestone_ms: number
  steps_completed: number
  steps_remaining: number
}

// Fired when sequence progress reaches a major milestone
event: 'sequence_progress_milestone'
properties: {
  sequence_id: string
  student_id: string
  milestone: 'started' | 'quarter' | 'half' | 'three_quarters' | 'complete'
  assignments_completed: number
  overall_mastery_score: number
  concepts_mastered_count: number
}

// Fired when student is flagged as at-risk
event: 'student_at_risk_detected'
properties: {
  student_id: string
  assignment_id: string
  risk_factors: string[]
  pacing_status: string
  mastery_score: number
  days_since_activity: number
  intervention_recommended: string
}

// Fired when progress snapshot is created
event: 'progress_snapshot_created'
properties: {
  snapshot_id: string
  student_id: string
  scope_type: string
  scope_id: string
  trigger: string
  completion_percentage: number
  mastery_score: number
}

// Fired when progress reconciliation occurs
event: 'progress_reconciled'
properties: {
  assignment_id: string
  student_id: string
  reconciliation_type: 'free_play' | 'content_change' | 'error_repair' | 'migration'
  steps_affected: number
  previous_completion_percentage: number
  new_completion_percentage: number
}
```

#### Performance Metrics
```typescript
// Measures progress calculation performance
metric: 'progress_calculation_time_ms'
dimensions: {
  calculation_type: 'step' | 'assignment' | 'sequence' | 'student'
  step_count: number
}

// Measures batch aggregation performance
metric: 'progress_aggregation_time_ms'
dimensions: {
  aggregation_scope: 'class' | 'organization' | 'system'
  record_count: number
}

// Measures snapshot creation performance
metric: 'snapshot_creation_time_ms'
dimensions: {
  snapshot_scope: 'step' | 'assignment' | 'sequence'
}
```

### Sampling Strategy
- **Step progress updates**: Never sampled (100% capture)
- **Assignment milestones**: Never sampled (100% capture)
- **Sequence milestones**: Never sampled (100% capture)
- **At-risk detection**: Never sampled (100% capture)
- **Progress snapshots**: Never sampled (100% capture)
- **Performance metrics**: Sampled at 10% for high-frequency calculations
- **Batch aggregation**: Never sampled (100% capture)

### Event Destinations
- **Real-time events**: Sent to analytics stream for immediate processing
- **Snapshot events**: Sent to data warehouse for long-term storage
- **At-risk events**: Sent to notification service for intervention triggers
- **Performance metrics**: Sent to monitoring service for alerting

## Permissions

### Student Access
- **Read own progress**: Students can view their own progress data
- **View own history**: Students can see historical progress snapshots
- **No edit access**: Students cannot modify progress data (scores come from game completion)

### Teacher Access
- **Read class progress**: Teachers can view progress for all students in their classes
- **View detailed progress**: Teachers can drill down into step-level details
- **Override progress**: Teachers can manually mark steps complete or adjust targets (with audit log)
- **Generate reports**: Teachers can export progress reports for students and classes
- **View snapshots**: Teachers can view historical progress snapshots for their students

### Admin Access
- **Read organization progress**: Admins can view progress for all students in their organization
- **View aggregate metrics**: Admins can access organization-wide progress analytics
- **Export bulk data**: Admins can export progress data for compliance or analysis
- **Configure policies**: Admins can set progress calculation rules and thresholds

### System Admin Access
- **Full read access**: Can view progress data across all organizations
- **Repair progress**: Can manually fix inconsistent progress data (with audit log)
- **Configure system**: Can adjust progress calculation algorithms and thresholds
- **Access raw data**: Can query underlying progress tables for troubleshooting

### Parent Access (Future)
- **Read child progress**: Parents can view progress for their children (with student consent)
- **View summaries**: Parents see simplified progress summaries, not detailed step data
- **No edit access**: Parents cannot modify progress or targets

## Dependencies

### Internal Dependencies
- **[Assignment Model](../07-content-architecture/assignment-model.md)**: Defines core assignment and step structures that progress tracking builds upon
- **[Assignment Generation](./assignment-generation.md)**: Provides the assignments whose progress is tracked
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Defines stage semantics and scoring that feed into progress calculations
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Defines telemetry infrastructure for progress events
- **[Roles and Permissions](../02-roles-and-permissions/roles-matrix.md)**: Controls who can access and modify progress data
- **[Student Dashboard](../03-student-experience/student-dashboard.md)**: Primary consumer of student progress summaries
- **[Teacher Reports](../04-teacher-experience/teacher-reports.md)**: Primary consumer of class and student progress analytics
- **[Admin Reports](../05-admin-subscriber-experience/admin-reports.md)**: Consumer of organization-wide progress metrics

### External Dependencies
- **Database**: PostgreSQL or similar for transactional progress storage
- **Time-series database**: InfluxDB or similar for performance metrics
- **Data warehouse**: BigQuery, Snowflake, or similar for historical snapshots and analytics
- **Caching layer**: Redis for frequently accessed progress summaries
- **Message queue**: For asynchronous progress calculations and event publishing

## Open questions

### Progress Calculation
- Should mastery level calculation be configurable per organization or standardized system-wide?
- What is the optimal consistency score threshold for distinguishing "proficient" from "mastered"?
- Should completion percentage include optional steps in any scenario, or always exclude them?
- How should the system handle steps with variable target scores (e.g., adaptive difficulty)?

### Performance and Scale
- What is the acceptable latency for real-time progress calculations (p50, p95, p99)?
- Should progress summaries be computed on-demand or pre-computed and cached?
- At what scale should progress calculations move to asynchronous batch processing?
- How can we efficiently handle progress queries across large organizations (10,000+ students)?

### Reconciliation and Migration
- When content changes, should students be required to re-complete affected steps or grandfathered in?
- How should progress be reconciled when a student transfers between classes with different policies?
- What is the appropriate balance between preserving historical data and cleaning up orphaned progress records?

### Interventions and Notifications
- What are the optimal thresholds for at-risk detection to balance sensitivity and noise?
- Should at-risk notifications be sent immediately or batched daily to avoid alert fatigue?
- How can we avoid false positives for students who are legitimately taking breaks or have irregular schedules?

### Privacy and Data Retention
- What progress data should be included in student data export requests (GDPR, FERPA)?
- How long should step-level progress be retained before aggregating to assignment summaries?
- Should parents have automatic access to progress data, or require explicit student consent?
- What progress data should be deleted when a student account is deactivated?

### User Experience
- Should students be able to see their own at-risk status, or only teachers?
- What level of detail should be visible in progress reports to avoid overwhelming users?
- Should progress dashboards show predictive completion dates, or only retrospective data?
