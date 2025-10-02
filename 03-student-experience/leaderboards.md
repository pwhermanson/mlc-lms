# Leaderboards

## Purpose

Leaderboards provide competitive motivation and social engagement for students through Challenge stage games. They display the top performers globally and within specific contexts (class, school, or skill level) to encourage continued practice and mastery. Leaderboards implement the core pedagogical principle that "success is the best motivator" by celebrating achievements and providing healthy competition that drives learning engagement.

## Scope

**Included**
- Global leaderboards for Challenge stage games with top 20 worldwide scorers
- Class-based leaderboards showing peer performance within the same teacher's students
- School-based leaderboards for ensemble subscriptions
- Skill-level leaderboards grouping students by similar ability levels
- Historical leaderboards showing past performance periods
- Privacy-safe display of usernames and scores without personal information
- Real-time score updates and position changes
- Achievement celebrations for leaderboard milestones

**Excluded**
- Individual student score history (covered by [Student Dashboard](../03-student-experience/student-dashboard.md))
- Teacher reporting and analytics (covered by [Teacher Reports](../04-teacher-experience/teacher-reports.md))
- Badge and achievement system (covered by [Badges System](../12-gamification/badges-system.md))
- Challenge game mechanics (covered by [Game Model](../08-games-registry-and-authoring/game-model.md))

## Data model (if applicable)

### Core Entities

**Leaderboard**
- `leaderboard_id`: Unique identifier (e.g., "global-challenge-G-01542", "class-12345-G-01542")
- `game_id`: Reference to the Challenge stage game
- `scope_type`: "global", "class", "school", "skill_level"
- `scope_id`: ID of the specific scope (class_id, school_id, skill_level)
- `time_period`: "all_time", "monthly", "weekly", "daily"
- `created_at`: When the leaderboard was created
- `updated_at`: Last score update

**Leaderboard Entry**
- `entry_id`: Unique identifier for the leaderboard entry
- `leaderboard_id`: Reference to the leaderboard
- `student_id`: Reference to the student (for internal tracking)
- `display_name`: Privacy-safe display name (username or anonymized identifier)
- `score`: The student's best score for this Challenge stage
- `rank`: Current position in the leaderboard
- `achieved_at`: When this score was achieved
- `is_current_student`: Boolean indicating if this is the viewing student's entry

### Score Aggregation Rules

Based on the original system's Challenge stage design:
- Only Challenge stage scores are eligible for leaderboards
- Best score per student per game is used (not cumulative)
- Scores are recorded with timestamp for historical tracking
- Minimum score threshold may be applied to prevent low-quality entries
- Scores are validated against game completion requirements

### Privacy and Display Rules

- Student usernames are displayed as provided during account creation
- No personal information (real names, emails) are shown on leaderboards
- Students can opt-out of leaderboard participation through privacy settings
- Generic student accounts (used in classroom settings) are excluded from global leaderboards
- Class and school leaderboards respect subscription boundaries

## Behavior and rules

### Score Eligibility

**Challenge Stage Requirements**
- Only games with Challenge stages are eligible for leaderboards
- Student must complete the Challenge stage to the end to qualify
- Score must meet minimum threshold (typically 50% of target score)
- Only one score per student per game per time period is recorded

**Time Period Rules**
- **All Time**: Best score ever achieved by the student
- **Monthly**: Best score achieved in the current calendar month
- **Weekly**: Best score achieved in the current week (Monday-Sunday)
- **Daily**: Best score achieved in the current day

### Ranking and Display

**Ranking Algorithm**
1. Sort by score (highest first)
2. For tied scores, sort by earliest achievement time
3. Display top 20 entries with rank numbers
4. Show current student's position even if outside top 20

**Real-time Updates**
- Leaderboards update immediately when new qualifying scores are submitted
- Position changes are highlighted for a brief period
- New entries trigger celebration animations
- Rank changes are logged for analytics

### Privacy and Safety

**Student Privacy Controls**
- Students can opt-out of global leaderboards while remaining in class leaderboards
- Display names can be customized for leaderboard visibility
- Parents can disable leaderboard participation for their children
- Teachers can disable leaderboards for their entire class

**Content Moderation**
- Inappropriate usernames are automatically filtered
- Teachers can report inappropriate content
- System administrators can remove entries that violate terms

## UX requirements (if applicable)

### Leaderboard Display

**Global Leaderboard Layout**
- Header showing game name and concept being tested
- Time period selector (All Time, Monthly, Weekly, Daily)
- Top 20 entries in vertical list format
- Current student's position highlighted with special styling
- "Your Best Score" section showing personal achievement

**Class Leaderboard Layout**
- Header showing class name and game
- Smaller, more intimate display (top 10-15 entries)
- Teacher's encouraging message or challenge
- Progress indicators showing improvement over time
- Optional: Team-based competitions within class

**Mobile Responsive Design**
- Stacked layout for small screens
- Swipeable time period selection
- Touch-friendly rank indicators
- Optimized for portrait orientation

### Visual Design Elements

**Rank Indicators**
- Gold, silver, bronze styling for top 3 positions
- Numbered ranks for positions 4-20
- Special highlighting for current student's entry
- Animated rank changes with smooth transitions

**Score Display**
- Large, prominent score numbers
- Score improvement indicators (↑, ↓, =)
- Achievement badges for milestones
- Time stamps for score achievement

**Celebration Animations**
- Confetti for new personal bests
- Rank improvement celebrations
- Leaderboard entry celebrations
- Milestone achievement notifications

### Accessibility Features

**Visual Accessibility**
- High contrast mode support
- Large text options for score displays
- Color-blind friendly rank indicators
- Clear focus states for keyboard navigation

**Cognitive Accessibility**
- Simple, clear ranking system
- Consistent visual hierarchy
- Predictable interaction patterns
- Clear feedback for all actions

## Telemetry

### Events

**Score Submission Events**
- `leaderboard_score_submitted`: When a qualifying Challenge score is submitted
- `leaderboard_rank_achieved`: When student achieves a new rank position
- `leaderboard_personal_best`: When student achieves new personal best score
- `leaderboard_milestone_reached`: When student reaches rank milestones (top 10, top 5, etc.)

**Interaction Events**
- `leaderboard_viewed`: When student views a leaderboard
- `leaderboard_time_period_changed`: When student switches time periods
- `leaderboard_scope_changed`: When student switches between global/class/school views
- `leaderboard_celebration_shown`: When celebration animation is displayed

**Privacy Events**
- `leaderboard_opt_out`: When student opts out of leaderboard participation
- `leaderboard_display_name_changed`: When student changes display name
- `leaderboard_privacy_settings_updated`: When privacy settings are modified

### Event Properties

**Score Events**
- `game_id`: The Challenge game being scored
- `score`: The achieved score
- `previous_best`: Previous best score for comparison
- `rank`: New rank position achieved
- `previous_rank`: Previous rank position
- `time_period`: Which time period the score applies to
- `scope_type`: Global, class, school, or skill level

**Interaction Events**
- `leaderboard_type`: Global, class, school, or skill level
- `time_period`: All time, monthly, weekly, or daily
- `view_duration`: How long student spent viewing leaderboard
- `interactions_count`: Number of interactions (time period changes, etc.)

## Permissions

### Student Permissions
- **Read**: View all leaderboards they have access to
- **Update**: Submit scores for Challenge stages they complete
- **Modify**: Change their display name and privacy settings
- **Opt-out**: Disable their participation in specific leaderboard scopes

### Teacher Permissions
- **Read**: View class and school leaderboards for their students
- **Moderate**: Report inappropriate content or usernames
- **Configure**: Enable/disable leaderboards for their class
- **Override**: Remove specific entries if necessary (with justification)

### System Administrator Permissions
- **Full Access**: View and modify all leaderboards
- **Content Moderation**: Remove inappropriate entries and usernames
- **Analytics**: Access detailed leaderboard performance data
- **Configuration**: Modify leaderboard rules and thresholds

### Parent Permissions (if applicable)
- **View**: See their child's leaderboard participation and achievements
- **Control**: Enable/disable leaderboard participation for their child
- **Privacy**: Modify display name and privacy settings

## Dependencies

### Core System Dependencies
- **Game Model**: Challenge stage games and scoring system ([Game Model](../08-games-registry-and-authoring/game-model.md))
- **Student Management**: Student accounts and authentication ([Roles and Permissions](../02-roles-and-permissions/roles-matrix.md))
- **Score Recording**: Score submission and validation system ([Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- **Privacy System**: Student privacy controls and data protection

### Gamification Dependencies
- **Badges System**: Achievement recognition and celebration ([Badges System](../12-gamification/badges-system.md))
- **Points System**: Point accumulation and display ([Points and Streaks](../12-gamification/points-and-streaks.md))
- **Challenge Boards**: Competitive game mechanics ([Challenge Boards](../12-gamification/challenge-boards.md))

### Technical Dependencies
- **Real-time Updates**: WebSocket or similar technology for live score updates
- **Caching System**: High-performance caching for leaderboard data
- **Analytics Engine**: Event tracking and performance monitoring
- **Notification System**: Achievement and rank change notifications

## Open questions

### Technical Implementation
- Should leaderboards use real-time updates or periodic refresh cycles?
- What is the optimal caching strategy for global leaderboards with high traffic?
- How should we handle leaderboard data consistency during high-concurrency score submissions?

### Privacy and Safety
- Should students be able to see their exact rank even when outside the top 20?
- What level of anonymity should be provided for students who want to participate but remain private?
- How should we handle students who change their display names frequently?

### Pedagogical Considerations
- Should leaderboards be available for all Challenge games or only specific ones?
- How can we ensure leaderboards motivate learning rather than create unhealthy competition?
- Should there be different leaderboard rules for different age groups or skill levels?

### Performance and Scalability
- What is the maximum number of students that can be supported on global leaderboards?
- How should we handle leaderboard data archival for historical periods?
- What are the performance implications of real-time leaderboard updates during peak usage?

### Future Enhancements
- Should leaderboards support team-based competitions?
- Would seasonal or tournament-style leaderboards add value?
- How could AI be used to create more personalized and motivating leaderboard experiences?

