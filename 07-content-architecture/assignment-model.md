# Assignment Model

This document defines the core data model and behavior for assignments in the MLC system. Assignments are the primary mechanism through which students receive structured learning content, combining games, multimedia, and assessment activities into coherent learning experiences.

## Purpose

The assignment model enables the systematic delivery of educational content to students through structured, trackable learning experiences. This specification provides the foundation for:

- **Pedagogical Structure**: Organizing learning content into logical, progressive sequences that build musical skills systematically
- **Progress Tracking**: Enabling comprehensive monitoring of student advancement through learning objectives
- **Teacher Control**: Providing educators with tools to customize and manage student learning paths
- **Student Engagement**: Creating clear, motivating learning experiences with visible progress indicators
- **Assessment Integration**: Seamlessly combining learning activities with evaluation and mastery demonstration

This model addresses critical pain points identified in the original MLC system, including score recording failures, unclear progression indicators, and the need for seamless integration between assigned and free-play content.

## Scope

### Included
- **Assignment Structure**: Core entities, relationships, and data model for assignments and steps
- **State Management**: How assignments and steps transition through different states
- **Gating Logic**: Rules that control when students can access specific content
- **Progress Tracking**: Mechanisms for recording and evaluating student performance
- **Integration Points**: How assignments connect with sequences, games, and user management
- **Version Control**: Handling of assignment updates and content changes

### Excluded
- **Assignment Generation**: How assignments are created from sequences (covered in [assignment-generation](../10-assignments-engine/assignment-generation.md))
- **Game Mechanics**: Specific game play rules and interactions (covered in [game-model](../08-games-registry-and-authoring/game-model.md))
- **Sequence Management**: How learning sequences are structured (covered in [sequence-model](../09-sequences-and-curriculum-tools/sequence-model.md))
- **User Interface**: Specific UI layouts and interactions (covered in [assignment-play](../03-student-experience/assignment-play.md))

## Data Model

### Core Entities

#### Assignment
The primary container for a student's learning experience, representing a cohesive set of learning activities.

```typescript
interface Assignment {
  assignment_id: string;           // Unique identifier
  student_id: string;             // Target student
  sequence_id: string;            // Source sequence
  sequence_version: string;       // Pinned version for consistency
  group_id: string;               // Assignment group within sequence
  name: string;                   // Human-readable assignment name
  description?: string;           // Optional description
  status: 'open' | 'complete' | 'archived';
  created_at: Date;               // Assignment creation timestamp
  due_at?: Date;                  // Optional due date
  completed_at?: Date;            // Completion timestamp
  policy_snapshot: object;        // Frozen policy at creation time
  metadata: {
    teacher_notes?: string;
    custom_instructions?: string;
    difficulty_level?: string;
  };
}
```

#### Step
Individual learning activities within an assignment, representing specific games, videos, or other content.

```typescript
interface Step {
  step_id: string;                // Unique identifier
  assignment_id: string;          // Parent assignment
  order_index: number;            // Position within assignment
  element_type: 'GAM' | 'VID' | 'AUD' | 'TXT' | 'RWD';
  element_id: string;             // Reference to learning element
  stage?: 'LEARN' | 'PLAY' | 'QUIZ' | 'CHALLENGE' | 'REVIEW';
  gating: {
    require_previous_steps: boolean;
    min_attempts: number;
    prerequisite_steps: string[];
  };
  targets: {
    pass_threshold: number;
    time_limit?: number;
    max_attempts?: number;
  };
  scheduling: {
    available_at?: Date;
    due_at?: Date;
    review_offset_days?: number;
  };
  optional: boolean;              // Whether step is required for progression
  state: 'locked' | 'available' | 'in_progress' | 'complete';
  completion_data?: {
    completed_at: Date;
    final_score: number;
    attempt_count: number;
    time_spent: number;
  };
}
```

#### Assignment Group
Logical grouping of related assignments within a sequence, often corresponding to curriculum units.

```typescript
interface AssignmentGroup {
  group_id: string;               // Unique identifier
  sequence_id: string;            // Parent sequence
  name: string;                   // Group name (e.g., "Primary Level 1A")
  description?: string;           // Group description
  order_index: number;            // Position within sequence
  prerequisites?: string[];       // Required previous groups
  metadata: {
    curriculum_level: string;
    estimated_duration: number;   // In minutes
    concepts_covered: string[];
  };
}
```

### Relationships

#### Assignment → Student
- One-to-many relationship
- Each assignment belongs to exactly one student
- Students can have multiple active assignments

#### Assignment → Sequence
- Many-to-one relationship
- Assignments are generated from sequence templates
- Version pinning ensures consistency over time

#### Assignment → Steps
- One-to-many relationship
- Steps are ordered within assignments
- Steps can have dependencies on other steps

#### Assignment → Assignment Group
- Many-to-one relationship
- Groups provide logical organization within sequences
- Groups can have prerequisites

## Behavior and Rules

### Assignment Lifecycle

#### Creation
1. **Template Resolution**: Assignment is created from a sequence template
2. **Policy Application**: Class and teacher policies are applied
3. **Step Materialization**: Template steps are converted to concrete steps
4. **State Initialization**: Initial states are set based on prerequisites and history
5. **Persistence**: Assignment and steps are saved to the database

#### Progression
1. **Step Completion**: Student completes individual steps
2. **State Updates**: Step states are updated based on performance
3. **Gating Evaluation**: Prerequisites for subsequent steps are evaluated
4. **Assignment Completion**: Assignment is marked complete when all required steps are done
5. **Next Assignment**: System determines next assignment in sequence

#### Archival
1. **Completion Threshold**: Assignment meets completion criteria
2. **Data Preservation**: Completion data is preserved for analytics
3. **Resource Cleanup**: Temporary data is cleaned up
4. **Archive Status**: Assignment is marked as archived

### Step State Transitions

#### Locked → Available
- **Trigger**: All prerequisites are satisfied
- **Conditions**: Previous steps completed, minimum attempts met
- **Actions**: Step becomes accessible to student

#### Available → In Progress
- **Trigger**: Student begins interaction with step
- **Conditions**: Step is available and not already in progress
- **Actions**: Step state updated, start time recorded

#### In Progress → Complete
- **Trigger**: Student meets completion criteria
- **Conditions**: Target score achieved, time requirements met
- **Actions**: Completion data recorded, next steps evaluated

#### In Progress → Available
- **Trigger**: Student abandons step without completion
- **Conditions**: Step was in progress but not completed
- **Actions**: Step returns to available state for retry

### Gating Rules

#### Prerequisite Dependencies
- **Sequential Steps**: Some steps require completion of previous steps
- **Concept Mastery**: Steps may require mastery of specific musical concepts
- **Time-based Gates**: Some steps have minimum time requirements
- **Attempt Requirements**: Steps may require minimum number of attempts

#### Target Achievement
- **Score Thresholds**: Steps have defined pass/fail score criteria
- **Time Limits**: Some steps have maximum time constraints
- **Attempt Limits**: Steps may have maximum attempt restrictions
- **Mastery Demonstration**: Some steps require consistent performance across attempts

### Free Play Reconciliation

The assignment model supports integration with free play activities through a reconciliation system:

- **Score Recognition**: Prior free play scores can count toward assignment completion
- **Fresh Attempt Policy**: Teachers can require fresh attempts regardless of prior scores
- **Progress Preservation**: Student progress in free play is not lost when assignments are created
- **Conflict Resolution**: Clear rules for handling conflicts between free play and assigned content

This addresses the original MLC system's challenge where students would discover games through the "All Games" library but then have to replay them in assignments, leading to frustration and lost progress.

## UX Requirements

### Student Interface
- **Clear Progress Indicators**: Visual representation of assignment and step completion status
- **Next Up Guidance**: Clear indication of what to work on next
- **Score Visibility**: Current scores displayed for completed steps
- **Navigation Controls**: Easy movement between steps and return to dashboard
- **Accessibility**: Full keyboard navigation and screen reader support

### Teacher Interface
- **Assignment Overview**: Clear view of student progress through assignments
- **Step Management**: Ability to modify step requirements and overrides
- **Progress Monitoring**: Real-time visibility into student performance
- **Customization Tools**: Options to adjust pacing and requirements

### Admin Interface
- **Assignment Analytics**: Comprehensive reporting on assignment effectiveness
- **Content Management**: Tools for managing assignment templates and content
- **System Configuration**: Settings for assignment behavior and policies
- **Troubleshooting**: Diagnostic tools for assignment issues

## Telemetry

### Assignment Events
- **assignment_created**: When a new assignment is generated for a student
- **assignment_started**: When a student begins working on an assignment
- **assignment_completed**: When a student finishes an assignment
- **assignment_abandoned**: When a student stops working on an assignment

### Step Events
- **step_unlocked**: When a step becomes available to a student
- **step_started**: When a student begins a step
- **step_completed**: When a student completes a step
- **step_failed**: When a student fails to meet step requirements
- **step_retried**: When a student retries a step

### Analytics Properties
- **assignment_id**: Unique assignment identifier
- **step_id**: Unique step identifier
- **student_id**: Student performing the action
- **sequence_id**: Source sequence for the assignment
- **completion_time**: Time spent on assignment/step
- **score_achieved**: Score obtained (for game steps)
- **attempt_count**: Number of attempts made
- **mastery_status**: Whether target criteria were met

## Permissions

### Student Access
- **Read**: Can view their own assignments and progress
- **Execute**: Can interact with available steps
- **Progress**: Can see their completion status and scores

### Teacher Access
- **Read**: Can view assignments for students in their classes
- **Create**: Can generate new assignments for students
- **Modify**: Can adjust step requirements and overrides
- **Monitor**: Can view detailed progress reports

### Admin Access
- **Full CRUD**: Can create, read, update, and delete assignments
- **Analytics**: Can access comprehensive assignment data
- **Configuration**: Can modify assignment policies and behavior
- **Support**: Can troubleshoot assignment issues

## Dependencies

### Internal Dependencies
- **[Learning Elements Overview](learning-elements-overview.md)**: Defines the content that populates assignment steps
- **[Assignment Generation](../10-assignments-engine/assignment-generation.md)**: Creates assignments from sequence templates
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: Provides the structure for assignment templates
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Defines game-specific mechanics and scoring
- **[Roles and Permissions](../02-roles-and-permissions/roles-matrix.md)**: Controls access to assignment functions

### External Dependencies
- **Content Management System**: For storing and serving learning elements
- **Analytics Platform**: For tracking assignment performance and effectiveness
- **User Management**: For student and teacher account information
- **Notification System**: For assignment due date reminders and updates

## Open Questions

### Pedagogical Design
- How should the system handle students who progress significantly faster or slower than expected?
- What is the optimal balance between structured assignments and student choice?
- How can assignments be adapted for different learning styles and abilities?

### Technical Implementation
- What is the best approach for handling large numbers of concurrent assignments?
- How should the system handle assignment updates when content changes?
- What caching strategy will optimize assignment loading performance?

### User Experience
- How can we make assignment progress more engaging and motivating for students?
- What level of teacher control should be provided over assignment pacing?
- How should the system handle assignment conflicts with school schedules?

### Analytics and Assessment
- What metrics best indicate assignment effectiveness?
- How can we use assignment data to improve content and sequencing?
- What privacy considerations apply to assignment progress tracking?
