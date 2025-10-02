# Assignment Builder

## Purpose

The Assignment Builder is the primary tool for teachers to create, customize, and manage learning assignments for their students. It provides an intuitive drag-and-drop interface for selecting and organizing learning elements into structured assignments that align with curriculum goals and student needs.

## Core Functionality

### Assignment Creation
- **Drag-and-Drop Interface**: Intuitive visual editor for building assignments
- **Learning Element Library**: Access to all available games, text, video, and audio content
- **Sequence Integration**: Build assignments from pre-defined learning sequences
- **Custom Organization**: Create custom assignment structures and groupings
- **Template System**: Save and reuse assignment templates

### Content Selection
- **Game Selection**: Choose from categorized games by concept, difficulty, and instrument
- **Stage Selection**: Select specific game stages (Learn, Play, Quiz, Challenge, Review)
- **Content Filtering**: Filter by concept, difficulty, age range, and learning modality
- **Search and Discovery**: Find specific content by name, concept, or keyword
- **Preview System**: Preview content before adding to assignments

### Assignment Structure
- **Sequential Flow**: Define the order of learning elements
- **Grouping**: Organize elements into logical groups or units
- **Prerequisites**: Set up prerequisite requirements between elements
- **Timing**: Configure time limits and pacing for assignments
- **Mastery Requirements**: Set mastery thresholds for progression

## User Interface

### Main Workspace
- **Canvas Area**: Central workspace for building assignments
- **Element Palette**: Sidebar with available learning elements
- **Assignment Outline**: Hierarchical view of assignment structure
- **Properties Panel**: Configuration options for selected elements
- **Preview Mode**: Test assignment flow before publishing

### Navigation and Controls
- **Save and Publish**: Save drafts and publish assignments to students
- **Undo/Redo**: Full undo/redo support for all actions
- **Copy and Duplicate**: Copy elements or entire assignments
- **Import/Export**: Import from templates or export for sharing
- **Version Control**: Track changes and maintain assignment versions

### Assignment Management
- **Assignment Library**: View and manage all created assignments
- **Template Gallery**: Access to shared assignment templates
- **Assignment Status**: Track draft, published, and archived assignments
- **Usage Analytics**: View how assignments are performing with students
- **Bulk Operations**: Manage multiple assignments simultaneously

## Content Organization

### Learning Elements
- **Games**: Interactive learning activities with multiple stages
- **Text Content**: Reading materials, instructions, and explanations
- **Video Content**: Instructional videos and demonstrations
- **Audio Content**: Music examples, instructions, and feedback
- **Rewards**: Badges, points, and achievement elements

### Assignment Types
- **Sequential Assignments**: Linear progression through learning elements
- **Branching Assignments**: Adaptive paths based on student performance
- **Practice Assignments**: Focused practice on specific concepts
- **Assessment Assignments**: Formal evaluation of student understanding
- **Review Assignments**: Spaced repetition and retention activities

### Grouping and Structure
- **Learning Units**: Logical groupings of related content
- **Skill Progressions**: Structured development of specific skills
- **Concept Clusters**: Related concepts grouped together
- **Difficulty Levels**: Graduated difficulty within assignments
- **Time Blocks**: Organized by practice sessions or time periods

## Assignment Configuration

### Basic Settings
- **Assignment Title**: Clear, descriptive name for the assignment
- **Description**: Detailed explanation of assignment goals and content
- **Target Audience**: Age range, skill level, and instrument type
- **Duration**: Estimated time to complete the assignment
- **Difficulty Level**: Overall difficulty rating for the assignment

### Advanced Settings
- **Mastery Requirements**: Minimum scores needed for completion
- **Retry Policies**: How many attempts students can make
- **Time Limits**: Maximum time allowed for completion
- **Prerequisites**: Required prior knowledge or completed assignments
- **Learning Objectives**: Specific goals and outcomes for the assignment

### Assignment Rules
- **Progression Logic**: How students advance through the assignment
- **Scoring System**: How scores are calculated and recorded
- **Feedback Rules**: When and how feedback is provided
- **Completion Criteria**: What constitutes assignment completion
- **Review Scheduling**: When review activities are scheduled

## Student Assignment Experience

### Assignment Delivery
- **Clear Instructions**: Step-by-step guidance for students
- **Progress Tracking**: Visual indicators of assignment progress
- **Navigation Controls**: Easy movement between assignment elements
- **Help and Support**: Access to help resources and teacher contact
- **Mobile Optimization**: Responsive design for all devices

### Learning Support
- **Adaptive Hints**: Contextual help based on student performance
- **Progress Feedback**: Regular updates on learning progress
- **Achievement Recognition**: Celebration of milestones and successes
- **Difficulty Adjustment**: Automatic or manual difficulty adjustment
- **Learning Analytics**: Insights into learning patterns and progress

## Collaboration and Sharing

### Teacher Collaboration
- **Assignment Sharing**: Share assignments with other teachers
- **Template Creation**: Create reusable assignment templates
- **Best Practices**: Share effective assignment strategies
- **Feedback System**: Provide feedback on shared assignments
- **Version Control**: Track changes and maintain assignment history

### Administrative Oversight
- **Approval Workflows**: Review and approve assignments before publication
- **Quality Standards**: Ensure assignments meet curriculum standards
- **Usage Monitoring**: Track assignment usage and effectiveness
- **Content Updates**: Manage updates to assignment content
- **Compliance**: Ensure assignments meet educational standards

## Analytics and Reporting

### Assignment Performance
- **Completion Rates**: Track how many students complete assignments
- **Time Analysis**: Monitor time spent on different elements
- **Score Distribution**: Analyze student performance across elements
- **Difficulty Analysis**: Identify elements that are too easy or difficult
- **Engagement Metrics**: Measure student engagement with content

### Learning Outcomes
- **Mastery Tracking**: Monitor concept mastery across assignments
- **Progress Trends**: Track student improvement over time
- **Skill Development**: Measure development of specific skills
- **Retention Analysis**: Assess long-term retention of concepts
- **Intervention Alerts**: Identify students who need additional support

## Accessibility and Inclusion

### Universal Design
- **Multiple Modalities**: Support for visual, auditory, and kinesthetic learning
- **Accessibility Features**: Screen reader support, keyboard navigation
- **Language Support**: Multi-language content and instructions
- **Cultural Sensitivity**: Inclusive content and examples
- **Learning Differences**: Support for students with different learning needs

### Adaptive Features
- **Difficulty Scaling**: Automatic adjustment based on student performance
- **Pacing Control**: Allow students to control their learning pace
- **Alternative Content**: Provide alternative ways to access content
- **Assistive Technology**: Support for assistive devices and software
- **Customization Options**: Allow students to customize their learning experience

## Future Enhancements

### AI Integration
- **Smart Recommendations**: AI-powered content suggestions
- **Adaptive Assignment Generation**: Automatic assignment creation based on student needs
- **Performance Prediction**: Predict student success with different assignments
- **Content Optimization**: AI-driven content improvement recommendations
- **Personalized Learning Paths**: Custom learning paths for individual students

### Advanced Features
- **Collaborative Assignments**: Group work and peer learning features
- **Real-time Collaboration**: Live collaboration between teachers and students
- **Advanced Analytics**: Deeper insights into learning patterns
- **Integration Tools**: Connect with external educational tools
- **Mobile Apps**: Native mobile applications for assignment management


## Teacher Workflows

### Create From Scratch
- Open Assignment Builder from the Teacher Dashboard (see `../04-teacher-experience/teacher-dashboard.md`).
- Search or browse content via Game/Sequence libraries (see `../08-games-registry-and-authoring/games-registry-overview.md` and `../09-sequences-and-curriculum-tools/sequence-model.md`).
- Drag learning elements onto the canvas, organize in the outline, and set properties.
- Configure progression, mastery, and completion criteria.
- Save draft; preview; publish to selected students or groups.

### Create From Sequence Template
- Choose a predefined sequence and generate an assignment scaffold.
- Adjust order, remove elements not needed, and add custom elements.
- Confirm mastery thresholds align with the sequence’s intent.
- Publish and optionally save as a reusable template.

### Quick Add Individual Games
- Use search to locate a specific game by name or ID.
- Add one or more stages (Learn, Play, Quiz, Challenge, Review) as discrete elements.
- Set per-stage retry and scoring rules to fit the goal (e.g., assessment vs. practice).

### Bulk Assign and Scheduling
- Select multiple classes or groups and schedule start/end dates.
- Use staggered release for weekly practice blocks.
- Apply the same template with differentiated mastery thresholds by group.


## Constraints and Edge Cases
- Assignments should not include unavailable or deprecated games. When a game is archived, show a non-blocking warning with a link to alternatives (see `../08-games-registry-and-authoring/games-stages-policy.md`).
- Stage selection must reflect canonical stage availability per game. Some games may omit certain stages.
- Preview should open game in an isolated context without affecting student analytics.
- When editing a published assignment, use versioning; students continue on the active version until teacher promotes a new version (see Version Control above).
- Respect role permissions: only `Teacher` and `Teacher-Admin` can create/publish; `Subscriber Admin` cannot modify pedagogy (see `../02-roles-and-permissions/roles-matrix.md`).
- Score policy alignment: ensure mastery thresholds map to the platform scoring system (see `../10-assignments-engine/assignment-progress-tracking.md`).


## Dependencies and Related Systems
- Content metadata and taxonomy come from the Games Registry (see `../08-games-registry-and-authoring/game-model.md`).
- Sequences and curriculum alignment rely on sequence tools (see `../09-sequences-and-curriculum-tools/sequence-authoring.md`).
- Student delivery and progress visuals appear in Assignment Play (reference `../03-student-experience/assignment-play.md`).
- Search and filters leverage the search architecture (see `../11-search-and-discovery/search-indexing.md`).
- Analytics roll up to teacher and admin reports (see `../15-analytics-and-reporting/teacher-reports-kpis.md`).


## Telemetry and Events
- Emit assignment lifecycle events aligned to the event model (see `../15-analytics-and-reporting/event-model.md`):
  - `assignment.created`, `assignment.updated`, `assignment.version.promoted`, `assignment.published`, `assignment.archived`.
  - Element-level mutations: `assignment.element.added`, `assignment.element.removed`, `assignment.element.reordered`, `assignment.rule.changed`.
- Include identifiers: assignmentId, versionId, teacherId, organizationId, elementIds, sequenceId (if applicable).
- Capture preview usage separately from student delivery to avoid skewing learning analytics.


## Permissions and Access
- Teachers can create, edit, and publish assignments to their owned students and groups.
- Teacher-Admins can manage assignments across teachers within the subscriber account.
- Subscriber Admins have read access for billing/visibility but cannot change content rules (see `../06-system-admin/user-permissions.md`).


## UX, Copy, and Accessibility References
- Follow copy standards and tone (see `../01-ux-design-system/screen-text-and-copy.md`).
- Ensure accessible keyboard navigation and screen reader labels (see `../16-accessibility-and-inclusion/keyboard-and-midi-access.md` and `../01-ux-design-system/a11y-standards.md`).
- Keep labels consistent with in-app terminology used across the Teacher Dashboard and All Games pages (see `../03-student-experience/all-games-library.md`).