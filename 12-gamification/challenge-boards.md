# Challenge Boards

This document defines the challenge board system that enables competitive gameplay and global leaderboards for Challenge stage games, implementing the core pedagogical principle that "success is the best motivator" through healthy competition and achievement recognition.

## Purpose

Challenge boards provide a competitive framework for students to engage with Challenge stage games, where scores have no upper limit and students can compete globally with peers worldwide. This system transforms individual learning into a social, competitive experience that motivates continued practice and mastery. Challenge boards implement the original MLC system's design where "Challenge levels of some games allow students to compete with others around the world" and "high scores are recorded in a Leader Board that can be seen by any member student around the world."

The system encourages students to push beyond basic mastery to achieve higher and higher scores, fostering both individual excellence and healthy competition that drives long-term engagement with musical learning concepts.

## Scope

**Included**
- Challenge stage game mechanics and unlimited scoring system
- Global challenge boards displaying top 20 worldwide scorers
- Class and school-based challenge competitions
- Challenge game discovery and access interface
- Score submission and validation for Challenge stages
- Real-time leaderboard updates and position tracking
- Challenge-specific achievements and recognition
- Integration with existing game progression (Learn → Play → Quiz → Challenge → Review)
- Privacy-safe display of usernames and scores
- Challenge board analytics and performance tracking

**Excluded**
- Individual game scoring and target score achievements (covered by [Game Model](../08-games-registry-and-authoring/game-model.md))
- Global leaderboard display and management (covered by [Leaderboards](../03-student-experience/leaderboards.md))
- Badge and achievement system (covered by [Badges System](../12-gamification/badges-system.md))
- Point accumulation and streak tracking (covered by [Points and Streaks](../12-gamification/points-and-streaks.md))
- Teacher reporting and analytics (covered by [Teacher Reports](../04-teacher-experience/teacher-reports.md))

## Data model (if applicable)

### Core Entities

**Challenge Game**
- `challenge_game_id`: Unique identifier for the Challenge stage
- `game_id`: Reference to the parent game
- `challenge_name`: Display name for the Challenge stage
- `challenge_description`: Description of what the challenge tests
- `min_score_threshold`: Minimum score required to qualify for leaderboards
- `is_unlimited_scoring`: Boolean indicating if scores can exceed target
- `max_duration`: Maximum time allowed for challenge attempt (if applicable)
- `difficulty_level`: Challenge difficulty (beginner, intermediate, advanced)
- `is_active`: Whether the challenge is currently available
- `created_at`: When the challenge was created
- `updated_at`: When challenge settings were last modified

**Challenge Score**
- `challenge_score_id`: Unique identifier for the score record
- `student_id`: Reference to the student who achieved the score
- `challenge_game_id`: Reference to the Challenge stage
- `score`: The achieved score (unlimited for Challenge stages)
- `completion_time`: Time taken to complete the challenge
- `attempt_number`: Which attempt this was for the student
- `achieved_at`: Timestamp when the score was achieved
- `is_personal_best`: Whether this is the student's best score for this challenge
- `is_leaderboard_qualified`: Whether the score qualifies for leaderboards
- `context_data`: JSON data about the specific attempt (difficulty, settings, etc.)

**Challenge Board**
- `challenge_board_id`: Unique identifier for the challenge board
- `challenge_game_id`: Reference to the Challenge stage
- `board_type`: "global", "class", "school", "skill_level"
- `scope_id`: ID of the specific scope (class_id, school_id, skill_level)
- `time_period`: "all_time", "monthly", "weekly", "daily"
- `max_entries`: Maximum number of entries to display (typically 20)
- `created_at`: When the board was created
- `updated_at`: Last score update

**Challenge Board Entry**
- `entry_id`: Unique identifier for the board entry
- `challenge_board_id`: Reference to the challenge board
- `student_id`: Reference to the student (for internal tracking)
- `display_name`: Privacy-safe display name for leaderboard
- `score`: The student's best score for this challenge
- `rank`: Current position in the challenge board
- `achieved_at`: When this score was achieved
- `is_current_student`: Boolean indicating if this is the viewing student's entry

### Challenge Game Categories

Based on the original system's game taxonomy:
- **Pitch Recognition**: Note identification and interval challenges
- **Rhythm Mastery**: Timing and rhythm pattern challenges  
- **Melody Memory**: Melodic pattern recognition and recall
- **Chord Identification**: Chord recognition and progression challenges
- **Scale Mastery**: Scale pattern and key signature challenges
- **Keyboard Skills**: MIDI keyboard performance challenges
- **Aural Skills**: Ear training and listening challenges
- **Visual Skills**: Staff reading and notation challenges

## Behavior and rules

### Challenge Stage Mechanics

**Unlimited Scoring System**
- Challenge stages allow scores to exceed target scores without limit
- Students can attempt challenges multiple times to improve scores
- Only the best score per student per challenge is recorded for leaderboards
- Scores are validated to ensure legitimate completion of the challenge

**Challenge Completion Requirements**
- Student must complete the entire Challenge stage to qualify
- Score must meet minimum threshold (typically 50% of target score)
- Challenge must be completed within maximum time limit (if applicable)
- Student must be logged in with individual account (not generic classroom account)

**Score Validation Rules**
- Scores are validated against game completion requirements
- Suspicious or impossible scores are flagged for review
- Multiple rapid attempts may be rate-limited to prevent gaming
- Scores from incomplete or interrupted challenges are not recorded

### Challenge Board Management

**Global Challenge Boards**
- Display top 20 worldwide scorers for each Challenge stage
- Updated in real-time as new qualifying scores are submitted
- Show student usernames and cities (privacy-safe information only)
- Include time period filters (All Time, Monthly, Weekly, Daily)

**Class and School Challenge Boards**
- Show top performers within the same class or school
- Respect subscription boundaries and privacy settings
- Allow teachers to create custom challenge competitions
- Include progress tracking and improvement indicators

**Ranking Algorithm**
1. Sort by score (highest first)
2. For tied scores, sort by earliest achievement time
3. Display top entries with rank numbers
4. Show current student's position even if outside top entries
5. Highlight personal best achievements

### Challenge Discovery and Access

**Challenge Game Library**
- Dedicated interface showing all available Challenge stages
- Filter by game category, difficulty level, and skill type
- Search functionality for specific challenges
- Progress indicators showing completion status

**Challenge Integration**
- Seamless transition from Quiz stage to Challenge stage
- Clear indication when Challenge stage is available
- Optional challenge participation (students can skip if desired)
- Challenge completion rewards and recognition

## UX requirements (if applicable)

### Challenge Board Display

**Global Challenge Board Layout**
- Header showing challenge name and musical concept
- Time period selector with clear visual indicators
- Top 20 entries in vertical list with rank numbers
- Current student's position highlighted with special styling
- "Your Best Score" section showing personal achievement
- Challenge description and learning objectives

**Class Challenge Board Layout**
- Header showing class name and challenge
- Smaller, more intimate display (top 10-15 entries)
- Teacher's encouraging message or challenge
- Progress indicators showing improvement over time
- Optional team-based competitions within class

**Challenge Game Interface**
- Clear indication that this is a Challenge stage
- Unlimited scoring explanation and motivation
- Real-time score display during gameplay
- Challenge completion celebration and leaderboard update
- Option to view leaderboard immediately after completion

### Visual Design Elements

**Challenge Stage Indicators**
- Distinctive visual styling for Challenge stages
- "Challenge" badge or icon prominently displayed
- Unlimited scoring indicator (infinity symbol or similar)
- Competitive elements (trophy, medal, or crown icons)

**Score Display**
- Large, prominent score numbers with animation
- Score improvement indicators (↑, ↓, =)
- Personal best celebrations with confetti effects
- Time stamps for score achievement

**Leaderboard Styling**
- Gold, silver, bronze styling for top 3 positions
- Numbered ranks for other positions
- Special highlighting for current student's entry
- Animated rank changes with smooth transitions

### Accessibility Features

**Visual Accessibility**
- High contrast mode support for all challenge interfaces
- Large text options for score displays and leaderboards
- Color-blind friendly rank indicators and score displays
- Clear focus states for keyboard navigation

**Cognitive Accessibility**
- Simple, clear challenge instructions and objectives
- Consistent visual hierarchy across all challenge interfaces
- Predictable interaction patterns for challenge gameplay
- Clear feedback for all challenge-related actions

**Motor Accessibility**
- Large touch targets for mobile challenge gameplay
- Keyboard navigation support for all challenge interfaces
- Voice-over compatible challenge information and instructions
- Reduced motion options for challenge animations

## Telemetry

### Events

**Challenge Completion Events**
- `challenge_started`: When student begins a Challenge stage
- `challenge_completed`: When student completes a Challenge stage
- `challenge_score_submitted`: When qualifying score is submitted
- `challenge_personal_best`: When student achieves new personal best
- `challenge_leaderboard_entry`: When student enters a leaderboard

**Challenge Board Events**
- `challenge_board_viewed`: When student views a challenge board
- `challenge_board_time_period_changed`: When time period is switched
- `challenge_board_scope_changed`: When switching between global/class/school views
- `challenge_board_celebration_shown`: When celebration animation is displayed

**Challenge Discovery Events**
- `challenge_library_viewed`: When student views available challenges
- `challenge_filter_applied`: When student uses filter options
- `challenge_selected`: When student selects a specific challenge
- `challenge_instructions_viewed`: When student views challenge instructions

### Event Properties

**Challenge Completion Events**
- `challenge_game_id`: The specific challenge being played
- `score`: The achieved score
- `completion_time`: Time taken to complete the challenge
- `attempt_number`: Which attempt this was for the student
- `is_personal_best`: Whether this is a new personal best
- `is_leaderboard_qualified`: Whether the score qualifies for leaderboards

**Challenge Board Events**
- `board_type`: Global, class, school, or skill level
- `time_period`: All time, monthly, weekly, or daily
- `view_duration`: How long student spent viewing the board
- `interactions_count`: Number of interactions with the board

**Challenge Discovery Events**
- `challenge_category`: Category of challenge selected
- `difficulty_level`: Difficulty level of challenge selected
- `filter_criteria`: What filters were applied
- `search_term`: What was searched for

## Permissions

### Student Permissions
- **Read**: View challenge boards and available challenges
- **Play**: Participate in Challenge stages and submit scores
- **Track**: Have scores automatically recorded and tracked
- **Configure**: Set privacy preferences for challenge participation
- **Opt-out**: Disable participation in specific challenge boards

### Teacher Permissions
- **Read**: View challenge boards and student challenge performance
- **Moderate**: Report inappropriate content or scores
- **Configure**: Enable/disable challenges for their class
- **Create**: Create custom class challenge competitions
- **Override**: Remove specific scores if necessary (with justification)

### System Administrator Permissions
- **Full Access**: View and modify all challenge data and settings
- **Content Moderation**: Remove inappropriate scores and usernames
- **Analytics**: Access comprehensive challenge performance data
- **Configuration**: Modify challenge rules, thresholds, and settings
- **Manage**: Create, update, and deactivate challenge games

### Parent Permissions (if applicable)
- **View**: See their child's challenge participation and achievements
- **Control**: Enable/disable challenge participation for their child
- **Privacy**: Modify display name and privacy settings for challenges

## Supporting Documents Referenced

This challenge boards specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Challenge stage definitions and competitive elements
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform messaging for competitive features
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Teacher controls for student competition features

## Dependencies

### Core System Dependencies
- **Game Model**: Challenge stage games and scoring system ([Game Model](../08-games-registry-and-authoring/game-model.md))
- **Student Management**: Student accounts and authentication ([Roles and Permissions](../02-roles-and-permissions/roles-matrix.md))
- **Score Recording**: Score submission and validation system ([Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- **Privacy System**: Student privacy controls and data protection

### Gamification Dependencies
- **Leaderboards**: Global leaderboard display and management ([Leaderboards](../03-student-experience/leaderboards.md))
- **Badges System**: Challenge-specific achievements and recognition ([Badges System](../12-gamification/badges-system.md))
- **Points System**: Point accumulation for challenge participation ([Points and Streaks](../12-gamification/points-and-streaks.md))
- **Achievements Sharing**: Social features for challenge achievements ([Achievements Sharing](../12-gamification/achievements-sharing.md))

### Technical Dependencies
- **Real-time Updates**: WebSocket or similar technology for live score updates
- **Caching System**: High-performance caching for challenge board data
- **Analytics Engine**: Event tracking and challenge performance monitoring
- **Notification System**: Challenge achievement and leaderboard notifications

## Open questions

### Challenge Design and Content
- Should all games have Challenge stages or only specific skill categories?
- How should we determine which games qualify for unlimited scoring?
- Should Challenge stages have different difficulty levels or adaptive difficulty?
- How should we handle Challenge stages for different age groups?

### Competition and Fairness
- Should there be separate challenge boards for different skill levels?
- How can we prevent cheating or score manipulation in challenges?
- Should Challenge stages have time limits or be completely open-ended?
- How should we handle students who dominate certain challenges?

### Technical Implementation
- What is the optimal database design for high-volume challenge score submissions?
- How should we handle challenge board data consistency during peak usage?
- What caching strategy is best for global challenge boards?
- How should we handle challenge data migration and versioning?

### Pedagogical Considerations
- How can we ensure challenges motivate learning rather than just competition?
- Should Challenge stages be required or optional for learning progression?
- How should we balance competition with collaborative learning?
- What role should teachers play in managing challenge participation?

### Future Enhancements
- Should challenges support team-based competitions or remain individual?
- Would seasonal or tournament-style challenges add value?
- How could AI be used to create personalized challenge recommendations?
- Should challenges integrate with external music learning platforms?
