# Retention and Review Logic

## Purpose

This specification defines how the MLC system schedules review activities and tracks long-term retention of learned musical concepts. While [Assignment Generation](./assignment-generation.md) covers the creation of assignments including review steps, and [Assignment Progress Tracking](./assignment-progress-tracking.md) covers immediate progress metrics, this document focuses specifically on the **algorithms, rules, and pedagogical strategies that ensure students retain what they've learned over time**.

The retention and review system addresses a fundamental challenge in music education: students may master a concept during initial practice but lose proficiency without strategic reinforcement. This specification provides the foundation for:

- **Spaced Repetition Scheduling**: Algorithmically determining optimal review timing based on learning science principles
- **Retention Measurement**: Tracking how well students retain concepts over time and identifying retention gaps
- **Adaptive Review Strategies**: Adjusting review frequency and difficulty based on individual student performance patterns
- **Early Intervention**: Identifying students at risk of losing proficiency before critical skill erosion occurs
- **Long-term Mastery Tracking**: Distinguishing between temporary passing scores and durable, long-term mastery

**Historical Context**: The original MLC system incorporated spaced repetition principles that were central to its pedagogy. Historical data showed that students who engaged with review content had significantly better long-term retention of musical concepts. This system formalizes those principles with data-driven scheduling and comprehensive retention analytics.

## Scope

### Included

- **Review Scheduling Algorithms**: Rules for determining when review steps should be scheduled based on quiz completion, performance, and retention patterns
- **Retention Metrics and Scoring**: How retention quality is measured and tracked over time
- **Spaced Repetition Patterns**: Configuration of spacing intervals (e.g., 7-day, 21-day, 60-day reviews) and adaptation logic
- **Review Step Generation**: How review steps are created, inserted into assignments, and distinguished from initial learning steps
- **Retention-based Interventions**: Triggers for remediation when retention metrics indicate skill erosion
- **Retention Analytics**: Metrics and reporting interfaces for teachers and admins to monitor long-term retention
- **Review Compliance Tracking**: Monitoring whether students complete reviews on time and how delays affect retention
- **Concept-level Retention**: Tracking retention across related games that teach the same musical concept
- **Review vs. Remediation**: Clear distinction between scheduled reviews (maintenance) and remediation (re-teaching)

### Excluded

- **Initial Assignment Creation**: How assignments are first generated from sequences (covered in [Assignment Generation](./assignment-generation.md))
- **Real-time Progress Tracking**: Immediate step completion and progress percentages (covered in [Assignment Progress Tracking](./assignment-progress-tracking.md))
- **Game Scoring Mechanics**: How individual game scores are calculated (covered in [Game Model](../08-games-registry-and-authoring/game-model.md))
- **Sequence Structure**: Definition of review as a stage type (covered in [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))
- **Student UI Layouts**: Specific interface designs for review activities (covered in [Student Assignments](../03-student-experience/student-assignments.md))
- **Teacher Report Visualizations**: Dashboard layouts and charts (covered in [Teacher Reports](../04-teacher-experience/teacher-reports.md))

## Data model (if applicable)

### Core Retention Entities

#### Review Schedule

Defines when and how review activities are scheduled for a specific concept or game.

```typescript
interface ReviewSchedule {
  schedule_id: string;                    // Unique identifier
  student_id: string;                     // Student this schedule applies to
  concept_id: string;                     // Musical concept being reviewed
  game_id: string;                        // Specific game for review
  stage: 'REVIEW';                        // Always REVIEW stage
  
  // Source tracking
  origin_step_id: string;                 // Original Quiz step that passed
  origin_assignment_id: string;           // Original assignment
  quiz_completed_at: Date;                // When Quiz was passed
  quiz_score: number;                     // Original Quiz score
  
  // Scheduling details
  review_sequence: number;                // Which review (1st, 2nd, 3rd, etc.)
  scheduled_date: Date;                   // When review should be presented
  offset_days: number;                    // Days from quiz completion
  scheduling_algorithm: 'fixed' | 'adaptive' | 'performance_based';
  
  // Execution tracking
  status: 'pending' | 'available' | 'completed' | 'overdue' | 'skipped';
  presented_at?: Date;                    // When review was shown to student
  completed_at?: Date;                    // When student completed review
  review_score?: number;                  // Score achieved on review
  
  // Adaptive parameters
  difficulty_adjustment?: number;         // Modifier based on retention pattern
  urgency_level: 'routine' | 'priority' | 'critical';
}
```

#### Retention Score

Tracks how well a student retains a specific concept over time.

```typescript
interface RetentionScore {
  retention_id: string;                   // Unique identifier
  student_id: string;                     // Student being tracked
  concept_id: string;                     // Musical concept
  
  // Current retention state
  retention_score: number;                // 0-100 score indicating retention quality
  retention_level: 'strong' | 'moderate' | 'weak' | 'at_risk';
  last_assessed_at: Date;                 // Most recent review or assessment
  
  // Historical performance
  initial_mastery_date: Date;             // When concept was first mastered (Quiz passed)
  initial_mastery_score: number;          // Original Quiz score
  review_count: number;                   // Number of reviews completed
  review_scores: number[];                // Array of all review scores
  
  // Retention metrics
  decay_rate: number;                     // Estimated skill decay (0-1, lower is better)
  consistency_score: number;              // Score variance across reviews
  time_since_last_review_days: number;    // Days since last review
  compliance_rate: number;                // % of scheduled reviews completed on time
  
  // Prediction
  next_review_recommended: Date;          // System-recommended next review date
  retention_confidence: number;           // Confidence in retention score (0-1)
  at_risk: boolean;                       // Flag for retention concern
  risk_factors: string[];                 // Specific concerns (e.g., 'declining_scores', 'missed_reviews')
}
```

#### Retention Event

Historical record of a retention-relevant event.

```typescript
interface RetentionEvent {
  event_id: string;                       // Unique identifier
  student_id: string;                     // Student
  concept_id: string;                     // Concept
  game_id: string;                        // Game
  
  // Event details
  event_type: 'quiz_passed' | 'review_completed' | 'review_missed' | 'retention_assessed' | 'intervention_triggered';
  event_date: Date;                       // When event occurred
  
  // Performance data
  score_achieved?: number;                // Score if applicable
  target_score?: number;                  // Expected score
  time_since_last_exposure_days: number;  // Days since last practice
  
  // Context
  assignment_id?: string;                 // Assignment if applicable
  step_id?: string;                       // Step if applicable
  triggered_by: 'scheduled' | 'teacher_initiated' | 'student_initiated' | 'system_intervention';
  
  // Retention impact
  retention_score_before: number;         // Retention score before event
  retention_score_after: number;          // Retention score after event
  retention_delta: number;                // Change in retention score
}
```

#### Concept Retention Aggregate

Organization-level or class-level retention analytics.

```typescript
interface ConceptRetentionAggregate {
  aggregate_id: string;                   // Unique identifier
  concept_id: string;                     // Concept being tracked
  scope_type: 'class' | 'organization' | 'system';
  scope_id: string;                       // Class ID, Org ID, or 'global'
  
  // Aggregate metrics
  student_count: number;                  // Number of students who've learned this concept
  average_retention_score: number;        // Mean retention score across students
  strong_retention_percentage: number;    // % with strong retention
  at_risk_percentage: number;             // % with weak/at-risk retention
  
  // Review completion metrics
  average_review_compliance: number;      // Mean % of reviews completed on time
  average_reviews_per_student: number;    // Mean number of reviews completed
  average_time_to_first_review_days: number;
  
  // Performance trends
  average_quiz_score: number;             // Mean initial Quiz score
  average_first_review_score: number;     // Mean 1st review score
  average_decay_rate: number;             // Mean skill decay rate
  
  // Time window
  measured_from: Date;                    // Start of measurement period
  measured_to: Date;                      // End of measurement period
  last_updated: Date;                     // When aggregate was computed
}
```

### Relationships

- **ReviewSchedule → Step**: Review schedules create review Steps within assignments
- **ReviewSchedule → RetentionScore**: Completed reviews update RetentionScore
- **RetentionScore → Concept**: Each retention score tracks one concept for one student
- **RetentionEvent → RetentionScore**: Events are aggregated to calculate retention scores
- **ConceptRetentionAggregate → RetentionScore**: Aggregates roll up individual scores

### Integration with Assignment Model

Review steps created by the retention system integrate with the standard Step model defined in [Assignment Model](../07-content-architecture/assignment-model.md):

- **`stage`**: Always set to `'REVIEW'`
- **`element_type`**: Always `'GAM'` (review steps are game-based)
- **`scheduling.review_offset_days`**: Set from ReviewSchedule
- **`optional`**: Typically `false` for first review, may be `true` for subsequent reviews depending on policy
- **`gating.require_previous_steps`**: Requires original Quiz step to be complete

The retention system writes to the Step.scheduling field but does not duplicate the core Step structure.

## Behavior and rules

### Review Scheduling Algorithms

#### Initial Review Scheduling (First Review)

When a student passes a Quiz step, the system automatically schedules a first review:

**Default Fixed Scheduling**:
```pseudo
function schedule_first_review(quiz_step, student_id, class_policy):
  offset_days = class_policy.default_review_offset_days ?? 7  // Default 7 days
  scheduled_date = quiz_step.completed_at + offset_days
  
  create ReviewSchedule:
    student_id = student_id
    origin_step_id = quiz_step.step_id
    quiz_completed_at = quiz_step.completed_at
    quiz_score = quiz_step.best_score
    review_sequence = 1
    scheduled_date = scheduled_date
    offset_days = offset_days
    scheduling_algorithm = 'fixed'
    status = 'pending'
    urgency_level = 'routine'
```

**Performance-based Adaptive Scheduling**:
For students with strong initial performance, the system may extend the review interval:

```pseudo
function calculate_adaptive_offset(quiz_score, target_score, student_history):
  performance_ratio = quiz_score / target_score
  
  if (performance_ratio >= 1.2 && student_history.consistency_high):
    // Strong performance with consistent history: extend review
    base_offset = 10 days
  else if (performance_ratio >= 1.0):
    // Standard pass: standard review
    base_offset = 7 days
  else if (performance_ratio >= 0.9):
    // Marginal pass: earlier review
    base_offset = 5 days
  else:
    // Below threshold but teacher allowed: urgent review
    base_offset = 3 days
  
  return base_offset
```

#### Subsequent Review Scheduling (Spaced Repetition)

After the first review is completed, additional reviews can be scheduled using spaced repetition patterns:

**Standard Spaced Pattern** (from class policy `spaced_schedule`):
- Review 1: +7 days from Quiz
- Review 2: +21 days from Quiz (14 days after Review 1)
- Review 3: +60 days from Quiz (39 days after Review 2)

**Adaptive Pattern** (based on review performance):
```pseudo
function schedule_next_review(previous_review, retention_score):
  if (retention_score.retention_level == 'strong'):
    // Strong retention: longer interval
    next_offset = previous_review.offset_days * 2
  else if (retention_score.retention_level == 'moderate'):
    // Moderate retention: standard progression
    next_offset = previous_review.offset_days * 1.5
  else if (retention_score.retention_level == 'weak'):
    // Weak retention: shorter interval
    next_offset = previous_review.offset_days * 1.0  // Same interval
  else if (retention_score.retention_level == 'at_risk'):
    // At risk: immediate re-review
    next_offset = 3 days
    urgency_level = 'critical'
  
  // Cap maximum interval
  next_offset = min(next_offset, 90 days)
  
  return next_offset
```

#### Review Priority and Urgency

Reviews are prioritized to ensure critical retention gaps are addressed first:

**Urgency Levels**:
1. **Critical**: Retention score below 50% or missed reviews causing skill erosion
2. **Priority**: Retention score 50-70% or review overdue by more than 7 days
3. **Routine**: Retention score above 70% and review on schedule

**Priority Queue Logic**:
```pseudo
function prioritize_reviews(student_id):
  pending_reviews = get_pending_reviews(student_id)
  
  sort pending_reviews by:
    1. urgency_level (critical > priority > routine)
    2. days_overdue (descending)
    3. concept_importance (foundational > advanced)
    4. scheduled_date (ascending)
  
  return pending_reviews
```

### Retention Score Calculation

Retention scores measure how well a student retains a concept over time. The score is calculated from multiple signals:

#### Core Retention Formula

```pseudo
function calculate_retention_score(student_id, concept_id):
  history = get_retention_events(student_id, concept_id)
  
  // 1. Score trajectory (40% weight)
  quiz_score = history.initial_mastery_score
  review_scores = history.review_scores
  score_trajectory = analyze_trajectory(quiz_score, review_scores)
  
  // 2. Time decay adjustment (30% weight)
  days_since_last = history.time_since_last_review_days
  decay_penalty = calculate_decay_penalty(days_since_last, history.decay_rate)
  
  // 3. Consistency (20% weight)
  consistency = calculate_consistency(review_scores)
  
  // 4. Compliance (10% weight)
  compliance = history.compliance_rate
  
  retention_score = (
    score_trajectory * 0.4 +
    (100 - decay_penalty) * 0.3 +
    consistency * 0.2 +
    compliance * 0.1
  )
  
  return clamp(retention_score, 0, 100)
```

#### Score Trajectory Analysis

```pseudo
function analyze_trajectory(quiz_score, review_scores):
  if (review_scores.length == 0):
    // No reviews yet: use quiz score as baseline
    return quiz_score
  
  latest_score = review_scores[review_scores.length - 1]
  
  // Calculate trend
  if (review_scores.length >= 2):
    trend = calculate_linear_trend(review_scores)
    if (trend > 0):
      // Improving: bonus
      return min(latest_score + 5, 100)
    else if (trend < -5):
      // Declining: penalty
      return max(latest_score - 10, 0)
  
  // Use latest score with quiz baseline consideration
  return (latest_score * 0.7) + (quiz_score * 0.3)
```

#### Time Decay Penalty

```pseudo
function calculate_decay_penalty(days_since_last, decay_rate):
  // Default decay rate: 0.02 (2% per day)
  // Decay is exponential but capped
  
  if (days_since_last <= 7):
    // Within recent memory: minimal decay
    return days_since_last * 0.5
  else if (days_since_last <= 30):
    // Standard decay window
    return 3.5 + ((days_since_last - 7) * decay_rate * 100)
  else:
    // Extended absence: accelerated decay
    return 3.5 + (23 * decay_rate * 100) + ((days_since_last - 30) * 0.3)
  
  // Cap total penalty at 60 points
  return min(penalty, 60)
```

#### Retention Level Assignment

```pseudo
function determine_retention_level(retention_score):
  if (retention_score >= 85):
    return 'strong'
  else if (retention_score >= 70):
    return 'moderate'
  else if (retention_score >= 50):
    return 'weak'
  else:
    return 'at_risk'
```

### Retention-based Interventions

#### At-Risk Detection

A student is flagged for retention intervention when:

```pseudo
function detect_retention_risk(retention_score):
  risk_factors = []
  
  // Score threshold
  if (retention_score.retention_score < 50):
    risk_factors.push('low_retention_score')
  
  // Declining trend
  if (retention_score.review_scores.length >= 3):
    recent_scores = retention_score.review_scores.slice(-3)
    if (is_declining(recent_scores) && recent_scores[2] < 70):
      risk_factors.push('declining_performance')
  
  // Missed reviews
  if (retention_score.compliance_rate < 0.6):
    risk_factors.push('poor_review_compliance')
  
  // Extended absence
  if (retention_score.time_since_last_review_days > 30):
    risk_factors.push('extended_absence')
  
  // High decay rate
  if (retention_score.decay_rate > 0.04):
    risk_factors.push('rapid_skill_decay')
  
  return {
    at_risk: risk_factors.length >= 2,
    risk_factors: risk_factors
  }
```

#### Intervention Actions

When retention risk is detected, the system triggers interventions:

1. **Immediate Review Scheduling**: Create high-priority review step
2. **Teacher Notification**: Alert teacher with specific retention concerns
3. **Remediation Recommendation**: Suggest re-teaching Learn/Play steps before review
4. **Parent Communication**: Generate parent-friendly retention report
5. **Modified Pacing**: Pause new content until retention stabilizes

**Intervention Logic**:
```pseudo
function trigger_retention_intervention(student_id, concept_id, risk_factors):
  retention_score = get_retention_score(student_id, concept_id)
  
  if ('low_retention_score' in risk_factors || 'declining_performance' in risk_factors):
    // Schedule immediate remediation
    create_remediation_steps(student_id, concept_id, 'retention_gap')
    
    // Schedule urgent review
    schedule_review(student_id, concept_id, offset_days=3, urgency='critical')
  
  if ('poor_review_compliance' in risk_factors):
    // Send engagement reminder
    notify_student_review_overdue(student_id)
    notify_teacher_student_missing_reviews(student_id)
  
  if ('extended_absence' in risk_factors):
    // Assume complete skill loss: full remediation
    create_remediation_steps(student_id, concept_id, 'extended_absence')
  
  // Always notify teacher
  notify_teacher_retention_concern(student_id, concept_id, risk_factors)
  
  // Log intervention
  create_retention_event(student_id, concept_id, 'intervention_triggered')
```

### Review vs. Remediation

It's critical to distinguish between **reviews** (maintenance) and **remediation** (re-teaching):

| Aspect | Review | Remediation |
|--------|--------|-------------|
| **Purpose** | Maintain mastery through practice | Re-teach concept after skill loss |
| **Trigger** | Scheduled based on time | Triggered by poor performance or retention gap |
| **Content** | Review stage game only | Learn + Play + Review sequence |
| **Gating** | Requires original Quiz pass | No prerequisites (fresh start) |
| **Scoring** | Contributes to retention metrics | Resets retention baseline |
| **Frequency** | Spaced intervals (7, 21, 60 days) | One-time until mastery restored |

**Remediation Trigger Logic**:
```pseudo
function determine_if_remediation_needed(retention_score):
  // Remediation if retention has collapsed
  if (retention_score.retention_score < 40):
    return true
  
  // Remediation if review failed badly
  if (retention_score.review_scores.length > 0):
    latest_review = retention_score.review_scores[latest]
    if (latest_review < 50):
      return true
  
  // Otherwise, continue with reviews
  return false
```

### Review Compliance and Overdues

#### Overdue Logic

```pseudo
function update_review_status(review_schedule):
  today = current_date()
  
  if (review_schedule.status == 'pending' && today >= review_schedule.scheduled_date):
    review_schedule.status = 'available'
  
  if (review_schedule.status == 'available' && today > review_schedule.scheduled_date + 7 days):
    review_schedule.status = 'overdue'
    review_schedule.urgency_level = 'priority'
  
  if (review_schedule.status == 'overdue' && today > review_schedule.scheduled_date + 14 days):
    review_schedule.urgency_level = 'critical'
    // Consider intervention
    check_retention_intervention(review_schedule.student_id, review_schedule.concept_id)
```

#### Compliance Tracking

```pseudo
function calculate_compliance_rate(student_id, concept_id):
  all_reviews = get_review_schedules(student_id, concept_id, status=['completed', 'skipped'])
  
  on_time = 0
  late = 0
  missed = 0
  
  for review in all_reviews:
    if (review.status == 'completed'):
      if (review.completed_at <= review.scheduled_date + 3 days):
        on_time += 1
      else:
        late += 1
    else if (review.status == 'skipped'):
      missed += 1
  
  total = on_time + late + missed
  compliance_rate = total > 0 ? on_time / total : 1.0
  
  return compliance_rate
```

### Concept-Level Retention Aggregation

When multiple games teach the same concept, retention is tracked at the concept level:

```pseudo
function aggregate_concept_retention(student_id, concept_id):
  games = get_games_for_concept(concept_id)
  game_retention_scores = []
  
  for game in games:
    if (student_has_mastered(student_id, game.game_id)):
      retention = get_retention_score(student_id, game.game_id)
      game_retention_scores.push(retention.retention_score)
  
  if (game_retention_scores.length == 0):
    return null  // Concept not yet mastered
  
  // Weighted average (more recent reviews weighted higher)
  concept_retention = calculate_weighted_average(game_retention_scores)
  
  return concept_retention
```

## UX requirements (if applicable)

### Student Experience

#### Review Indicators in Assignment View

- **"Due for Review" badge**: Visual indicator on steps that are scheduled reviews
- **Review timing label**: "Review in 3 days" or "Review overdue by 2 days"
- **Urgency color coding**: 
  - Green: Review scheduled in future (routine)
  - Yellow: Review due today or within 2 days (priority)
  - Red: Review overdue (critical)
- **Review vs. New distinction**: Clear labeling that distinguishes review activities from first-time learning

#### Student Dashboard - Review Section

- **"Reviews Due" widget**: Prominent section showing pending reviews prioritized by urgency
- **Review queue**: Ordered list of reviews with:
  - Game name and concept
  - Days since last practice
  - Original mastery date
  - Retention level indicator (strong/moderate/weak)
- **Review streak**: Days of consecutive review completion
- **Retention score visualization**: Simple badge or chart showing overall retention health

#### In-Game Review Experience

- **Context reminder**: Brief text reminding student they previously learned this concept
- **"This is a review" header**: Clear indication that this is retention practice
- **Previous score display**: Show original Quiz score and date for context
- **Retention benefit message**: Positive reinforcement about why reviews matter

### Teacher Experience

#### Class Retention Dashboard

- **Retention heatmap**: Grid showing retention scores for each student × concept
- **At-risk student list**: Highlighted students with retention concerns
- **Review compliance chart**: Visualization of review completion rates
- **Concept retention trends**: Line graphs showing retention over time for key concepts

#### Individual Student Retention Report

- **Retention timeline**: Chronological view of Quiz → Review 1 → Review 2 with scores
- **Retention score breakdown**: Detailed breakdown of score trajectory, decay, consistency, compliance
- **Risk factor summary**: Clear explanation of any retention concerns
- **Intervention recommendations**: Suggested actions (schedule extra review, assign remediation, etc.)
- **Retention vs. Progress**: Distinct sections showing current progress and long-term retention

#### Teacher Controls

- **Manual review scheduling**: Override automatic scheduling for specific students
- **Review policy configuration**: Set class-level review intervals and patterns
- **Skip review**: Mark review as optional if student has demonstrated mastery elsewhere
- **Trigger remediation**: Convert review to full remediation sequence

### Admin Experience

#### Organization Retention Analytics

- **Aggregate retention score**: Overall retention health metric
- **Concept retention rankings**: Which concepts have strongest/weakest retention
- **Review effectiveness**: Correlation between review completion and retention
- **Policy comparison**: Compare retention metrics across different review scheduling policies

### Accessibility Requirements

- **Screen reader announcements**: "Review step: Staff Birds. Retention level: moderate. Due in 3 days."
- **Keyboard navigation**: Tab through review queue, Enter to start review
- **High contrast support**: Retention level indicators visible in high contrast mode
- **Reduced motion option**: Disable animated retention score visualizations
- **Text alternatives**: All retention charts have accessible data tables

## Telemetry

### Retention and Review Events

All events follow the canonical [Event Model](../15-analytics-and-reporting/event-model.md) specification.

#### Review Scheduling Events

```typescript
// Fired when a review is scheduled after Quiz completion
event: 'review_scheduled'
properties: {
  student_id: string
  concept_id: string
  game_id: string
  origin_step_id: string           // Original Quiz step
  review_sequence: number          // 1st, 2nd, 3rd review
  scheduled_date: ISO8601
  offset_days: number
  scheduling_algorithm: 'fixed' | 'adaptive' | 'performance_based'
  quiz_score: number
  target_score: number
}

// Fired when review schedule is adjusted based on performance
event: 'review_rescheduled'
properties: {
  schedule_id: string
  student_id: string
  concept_id: string
  previous_scheduled_date: ISO8601
  new_scheduled_date: ISO8601
  reason: 'performance_adjustment' | 'teacher_override' | 'retention_concern'
  urgency_level: 'routine' | 'priority' | 'critical'
}
```

#### Review Completion Events

```typescript
// Fired when student completes a review step
event: 'review_completed'
properties: {
  schedule_id: string
  student_id: string
  concept_id: string
  game_id: string
  review_sequence: number
  score_achieved: number
  target_score: number
  passed: boolean
  days_since_quiz: number
  days_overdue: number             // 0 if on time, positive if late
  time_on_task_ms: number
  attempt_count: number
}

// Fired when review becomes overdue
event: 'review_overdue'
properties: {
  schedule_id: string
  student_id: string
  concept_id: string
  scheduled_date: ISO8601
  days_overdue: number
  urgency_level: 'priority' | 'critical'
  notification_sent: boolean
}

// Fired when review is skipped by teacher
event: 'review_skipped'
properties: {
  schedule_id: string
  student_id: string
  concept_id: string
  skipped_by: 'teacher' | 'system'
  reason: string
}
```

#### Retention Score Events

```typescript
// Fired when retention score is calculated or updated
event: 'retention_score_updated'
properties: {
  retention_id: string
  student_id: string
  concept_id: string
  retention_score: number          // 0-100
  retention_level: 'strong' | 'moderate' | 'weak' | 'at_risk'
  previous_score: number
  score_delta: number
  decay_rate: number
  consistency_score: number
  compliance_rate: number
  trigger: 'review_completed' | 'periodic_assessment' | 'intervention'
}

// Fired when student is flagged for retention concern
event: 'retention_risk_detected'
properties: {
  student_id: string
  concept_id: string
  retention_score: number
  retention_level: 'weak' | 'at_risk'
  risk_factors: string[]           // ['low_retention_score', 'declining_performance', etc.]
  days_since_last_review: number
  intervention_triggered: boolean
}

// Fired when retention intervention is initiated
event: 'retention_intervention_triggered'
properties: {
  student_id: string
  concept_id: string
  intervention_type: 'immediate_review' | 'remediation' | 'teacher_notification' | 'pacing_adjustment'
  risk_factors: string[]
  retention_score: number
  scheduled_actions: string[]
}
```

#### Aggregate Retention Events

```typescript
// Fired when concept retention aggregate is computed
event: 'concept_retention_aggregated'
properties: {
  concept_id: string
  scope_type: 'class' | 'organization' | 'system'
  scope_id: string
  student_count: number
  average_retention_score: number
  strong_retention_percentage: number
  at_risk_percentage: number
  average_review_compliance: number
  measured_from: ISO8601
  measured_to: ISO8601
}
```

### Sampling Strategy

- **Review scheduling**: Never sampled (100% capture)
- **Review completion**: Never sampled (100% capture)
- **Retention score updates**: Never sampled (100% capture)
- **Retention risk detection**: Never sampled (100% capture)
- **Interventions**: Never sampled (100% capture)
- **Aggregate computations**: Never sampled (100% capture)

Retention and review events are critical for pedagogical analytics and student outcomes, so all events are captured without sampling.

### Event Destinations

- **Real-time events**: Sent to analytics stream for immediate processing
- **Review scheduling/completion**: Sent to both analytics stream and assignment service
- **Retention scores**: Sent to data warehouse for long-term analysis
- **At-risk detection**: Sent to notification service for immediate teacher alerts
- **Aggregates**: Sent to data warehouse for reporting

## Permissions

### Student Access

- **Read own retention scores**: Students can view their own retention levels and review schedules
- **View review queue**: Students can see pending and overdue reviews
- **Complete reviews**: Students can interact with review steps and record scores
- **No modification**: Students cannot modify review schedules or retention scores

### Teacher Access

- **Read class retention data**: Teachers can view retention scores and review schedules for all students in their classes
- **View retention reports**: Teachers can access detailed retention analytics and at-risk student lists
- **Manual review scheduling**: Teachers can override automatic scheduling and schedule additional reviews
- **Skip reviews**: Teachers can mark reviews as optional or skip them for specific students
- **Trigger remediation**: Teachers can manually initiate remediation sequences based on retention concerns
- **Configure class policies**: Teachers can set review intervals and spaced repetition patterns for their classes
- **View aggregate metrics**: Teachers can see class-level retention analytics and trends

### Admin/Subscriber Access

- **Read organization retention data**: Admins can view retention metrics for all students in their organization
- **Organization policies**: Admins can set default review policies and retention thresholds
- **Cross-class analytics**: Admins can compare retention metrics across multiple classes
- **Export retention data**: Admins can export retention reports for compliance or analysis
- **No student-level overrides**: Admins cannot modify individual student retention scores (teacher function)

### System Admin Access

- **Full read access**: Can view all retention data across all organizations
- **Configure algorithms**: Can adjust retention calculation formulas and scheduling algorithms
- **Repair data**: Can manually fix inconsistent retention scores or schedules (with audit log)
- **Performance tuning**: Can adjust batch processing schedules and retention computation frequency
- **Access raw data**: Can query underlying retention tables for troubleshooting

### Audit Requirements

All retention-related modifications (manual scheduling, skipped reviews, retention score adjustments) are logged with:
- User ID and role performing action
- Timestamp
- Original and new values
- Reason/justification (required for overrides)

## Supporting Documents Referenced

This retention review logic specification draws from the following source documents:

- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence specifications including review stage definitions
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game stage definitions including review stage purpose
- [EXE specs 2012-0913 - Assignment Screen.csv](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Assignment%20Screen.csv) — Assignment structure showing review scheduling

## Dependencies

### Internal Dependencies

#### Core Assignment and Sequence Systems

- **[Assignment Generation](./assignment-generation.md)**: The retention system extends assignment generation by scheduling review steps. Assignment generation creates the initial assignment structure; retention logic adds review scheduling on top of it.
- **[Assignment Progress Tracking](./assignment-progress-tracking.md)**: Progress tracking provides real-time completion metrics; retention tracking provides long-term mastery metrics. The two systems share student performance data but serve different analytical purposes.
- **[Assignment Model](../07-content-architecture/assignment-model.md)**: Defines the Step entity structure that review steps integrate with. Retention system writes to Step.scheduling fields.
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: Defines Review as a stage type and provides default review scheduling parameters. Retention logic implements the scheduling algorithms referenced in the sequence model.

#### Content and Game Systems

- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Provides game stage definitions, scoring mechanics, and target thresholds that feed into retention score calculations.
- **[Games Registry](../08-games-registry-and-authoring/games-registry-overview.md)**: Supplies concept taxonomy and game-to-concept mappings required for concept-level retention aggregation.
- **[Game Taxonomy](../07-content-architecture/game-taxonomy.md)**: Provides concept hierarchy and importance weights used in retention prioritization.

#### User Experience

- **[Student Dashboard](../03-student-experience/student-dashboard.md)**: Primary interface for displaying review queue and retention indicators to students.
- **[Student Assignments](../03-student-experience/student-assignments.md)**: Displays review steps within assignment context, showing review timing and urgency.
- **[Teacher Reports](../04-teacher-experience/teacher-reports.md)**: Consumes retention metrics for teacher dashboards and student progress reports.
- **[Admin Reports](../05-admin-subscriber-experience/admin-reports.md)**: Uses aggregate retention data for organization-level analytics.

#### Analytics and Notifications

- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Defines canonical telemetry structure for all retention and review events.
- **[Teacher Notifications](../04-teacher-experience/teacher-notifications.md)**: Receives retention risk alerts and review compliance notifications.
- **[Student Notifications](../03-student-experience/student-notifications.md)**: Displays review due reminders and retention milestone achievements.

#### Access Control

- **[Roles and Permissions](../02-roles-and-permissions/roles-matrix.md)**: Defines who can view retention data, schedule reviews, and override retention policies.
- **[Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md)**: Controls access to retention endpoints and data.

### External Dependencies

- **Database**: PostgreSQL or similar for storing ReviewSchedule, RetentionScore, and RetentionEvent entities
- **Time-series database**: For efficient storage and querying of retention score history over time
- **Data warehouse**: For long-term retention analytics and aggregate computations
- **Scheduler/Job Queue**: For automated review scheduling and periodic retention score updates
- **Notification service**: For sending review reminders and at-risk alerts
- **Analytics platform**: For processing retention telemetry and generating insights

### Integration Points

The retention system provides data to:
- **Remediation Engine**: Retention risk triggers remediation workflows
- **Adaptive Learning**: Retention patterns inform adaptive sequencing decisions
- **Reporting Systems**: Retention metrics feed into teacher and admin dashboards
- **Parent Portals**: Retention summaries included in parent progress reports
- **Learning Analytics**: Retention data contributes to predictive models for student success

## Open questions

### Scheduling Strategy

- **Default review intervals**: Should the default 7-day first review interval be configurable per concept difficulty tier, or remain universal across all content?
- **Maximum review count**: What is the hard cap on scheduled reviews per concept (currently considering 3: 7-day, 21-day, 60-day)? Should some foundational concepts have unlimited reviews?
- **Adaptive vs. fixed scheduling**: Should adaptive scheduling be opt-in or opt-out for teachers? What is the default for new classes?
- **Review density limits**: How many reviews per day should be considered "too many" for a student? Should the system throttle review scheduling to prevent overload?

### Retention Score Calculation

- **Weight tuning**: Are the current weights (40% trajectory, 30% decay, 20% consistency, 10% compliance) optimal, or should they be adjusted based on pilot data?
- **Decay rate calibration**: The default decay rate of 0.02 (2% per day) is an initial estimate. Should this vary by concept type (e.g., rhythm vs. pitch concepts decay at different rates)?
- **Confidence thresholds**: At what retention confidence level (currently 0-1 scale) should the system suppress displaying scores to avoid misleading students/teachers?
- **Concept vs. game retention**: Should retention scores be displayed primarily at the concept level or game level? Or both, with different prominence?

### Intervention Triggers

- **At-risk thresholds**: Are the current thresholds (retention score < 50 for at-risk, < 70 for weak) appropriately calibrated to minimize false positives?
- **Intervention aggressiveness**: Should the system automatically create remediation steps when retention risk is detected, or only recommend them to teachers?
- **Compliance grace period**: How many days overdue should a review be before triggering teacher notification (currently 7 days)?
- **Multiple at-risk concepts**: If a student has 5+ concepts flagged as at-risk, how should interventions be prioritized and staged to avoid overwhelming the student?

### Review vs. Remediation

- **Transition criteria**: At what retention score threshold should the system automatically convert a scheduled review into a full remediation sequence (currently < 40)?
- **Remediation structure**: When remediation is triggered, should it always include Learn → Play → Review, or can it be abbreviated based on the severity of retention loss?
- **Review after remediation**: After completing remediation, how soon should the next review be scheduled? Should it reset to the 7-day interval or continue the original spacing pattern?

### Data Retention and Privacy

- **Historical retention data**: How long should retention scores and review history be retained? Should they be aggregated or deleted after a certain period?
- **Student access to retention data**: Should students see their own retention scores and decay rates, or is this information teacher-only to avoid anxiety?
- **Parent access**: Should parents have automatic access to retention metrics, or require teacher approval to view detailed retention reports?
- **Data export**: What retention data should be included in GDPR/FERPA data export requests? Should raw retention events be included or only summaries?

### Performance and Scale

- **Batch computation frequency**: How often should retention scores be recalculated for inactive students (currently considering daily batch at midnight)?
- **Real-time vs. batch**: Should retention scores update in real-time after every review completion, or batched for performance?
- **Aggregate refresh cadence**: How often should ConceptRetentionAggregate tables be updated (currently considering hourly for classes, daily for organizations)?
- **Scale limits**: At what organization size (student count) should retention computation move from real-time to asynchronous processing?

### User Experience

- **Retention gamification**: Should strong retention be rewarded with badges or points to incentivize review completion? Or does this risk creating anxiety around retention scores?
- **Review flexibility**: Should students be able to postpone reviews (e.g., "remind me in 2 days") or must they complete them when scheduled?
- **Review streak rewards**: Should consecutive days of review completion unlock rewards or bonuses, similar to daily login streaks?
- **Retention visualization**: What is the most effective way to visualize retention trends for teachers without overwhelming them with data?

### Policy and Configuration

- **Organization-level defaults**: Should organizations be able to set custom retention thresholds and review intervals that differ from system defaults?
- **Teacher override scope**: How much can individual teachers deviate from organization policies (e.g., can they completely disable reviews for a class)?
- **Student opt-out**: Can students or parents opt out of review scheduling while still participating in regular assignments? What are the pedagogical implications?
- **A/B testing**: Should the system support A/B testing different review schedules and retention algorithms across cohorts to optimize effectiveness?

### Research and Validation

- **Efficacy measurement**: How will we measure whether the retention system is actually improving long-term concept mastery compared to no reviews?
- **Optimal spacing**: Should the system run controlled experiments to determine optimal review spacing for different concept types and age groups?
- **Retention prediction**: Can we build machine learning models to predict which students will have retention issues before they occur?
- **Cross-concept dependencies**: Do some concepts have stronger retention when other prerequisite concepts are well-retained? Should this inform scheduling?
