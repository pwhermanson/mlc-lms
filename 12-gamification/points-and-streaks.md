# Points and Streaks

## Purpose

The points and streaks system implements the core pedagogical principle that "success is the best motivator" by providing immediate, quantifiable feedback on student progress and engagement. This system transforms abstract learning achievements into concrete, accumulative rewards that students can track, celebrate, and use to measure their musical learning journey. Points reward individual accomplishments while streaks encourage consistent practice habits, creating a comprehensive motivation framework that complements the existing scoring system and drives long-term learning engagement.

## Scope

**Included**
- Point accumulation system for all game stages (Learn, Play, Quiz, Challenge, Review)
- Streak tracking for daily practice and consistent engagement
- Point-based rewards and milestone celebrations
- Streak-based achievements and recognition
- Point and streak display on student dashboard
- Integration with existing scoring and leaderboard systems
- Point and streak analytics for teachers and administrators
- Gamified progress indicators and visual feedback

**Excluded**
- Individual game scores and target score achievements (covered by [Game Model](../08-games-registry-and-authoring/game-model.md))
- Badge and achievement system (covered by [Badges System](../12-gamification/badges-system.md))
- Leaderboard rankings and competitive scoring (covered by [Leaderboards](../03-student-experience/leaderboards.md))
- Assignment progress tracking (covered by [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- Teacher reporting and analytics (covered by [Teacher Reports](../04-teacher-experience/teacher-reports.md))

## Data model (if applicable)

### Core Entities

**Student Points**
- `student_points_id`: Unique identifier for the points record
- `student_id`: Reference to the student
- `total_points`: Current total point balance
- `lifetime_points`: All-time points earned (never decreases)
- `points_this_week`: Points earned in current week
- `points_this_month`: Points earned in current month
- `last_updated`: Timestamp of last point update
- `created_at`: When points tracking began for this student

**Point Transaction**
- `transaction_id`: Unique identifier for the point transaction
- `student_id`: Reference to the student earning points
- `point_amount`: Number of points earned (positive) or spent (negative)
- `transaction_type`: "game_completion", "streak_bonus", "milestone_reward", "achievement_bonus", "daily_login", "perfect_score", "challenge_bonus"
- `source_id`: ID of the source (game_id, streak_id, achievement_id, etc.)
- `source_context`: JSON data about the specific action that earned points
- `earned_at`: Timestamp when points were earned
- `is_bonus`: Whether these points are bonus points beyond base earning

**Student Streak**
- `streak_id`: Unique identifier for the streak record
- `student_id`: Reference to the student
- `streak_type`: "daily_practice", "weekly_consistency", "assignment_completion", "perfect_scores"
- `current_streak`: Current consecutive count
- `longest_streak`: Highest streak count ever achieved
- `streak_start_date`: When current streak began
- `last_activity_date`: Date of last qualifying activity
- `is_active`: Whether the streak is currently active
- `total_qualifying_activities`: Total count of activities that contributed to streaks

**Streak Milestone**
- `milestone_id`: Unique identifier for the milestone
- `streak_type`: Type of streak this milestone applies to
- `milestone_threshold`: Number of consecutive days/activities required
- `milestone_name`: Display name for the milestone
- `point_reward`: Points awarded for reaching this milestone
- `badge_reward`: Badge ID awarded for reaching this milestone (if any)
- `is_active`: Whether this milestone is currently available

### Point Earning Rules

**Base Point Values**
- **Learn Stage Completion**: 10 points
- **Play Stage Completion**: 15 points  
- **Quiz Stage Completion**: 25 points
- **Review Stage Completion**: 20 points
- **Challenge Stage Completion**: 50 points
- **Perfect Score (any stage)**: +10 bonus points
- **Target Score Achievement**: +5 bonus points
- **Daily Login**: 5 points (once per day)
- **First Game of Day**: +5 bonus points

**Streak Bonus Multipliers**
- **3-day streak**: 1.1x multiplier
- **7-day streak**: 1.2x multiplier
- **14-day streak**: 1.3x multiplier
- **30-day streak**: 1.5x multiplier
- **100-day streak**: 2.0x multiplier

**Special Bonuses**
- **Weekly Consistency**: 25 points for 5+ days of activity in a week
- **Monthly Engagement**: 100 points for 20+ days of activity in a month
- **Sequence Completion**: 50 points for completing a full learning sequence
- **Skill Mastery**: 75 points for achieving mastery in a skill category
- **Challenge Leaderboard Entry**: 25 points for appearing on any leaderboard

## Behavior and rules

### Point Earning Process

**Real-time Point Calculation**
1. Student completes a qualifying activity (game stage, login, etc.)
2. System calculates base points for the activity
3. System checks for applicable bonuses (perfect score, first of day, etc.)
4. System applies current streak multiplier if applicable
5. Points are added to student's total and lifetime totals
6. Point transaction is recorded with full context
7. Celebration animation is displayed if significant points earned

**Streak Calculation and Maintenance**
1. System checks if student's activity qualifies for streak continuation
2. If yes, streak count is incremented and last activity date updated
3. If no, streak is marked as broken and reset to 0
4. System checks for new streak milestones and awards bonuses
5. Streak progress is updated on student dashboard

### Streak Types and Rules

**Daily Practice Streak**
- Qualifying activities: Any game stage completion, assignment completion
- Reset condition: No qualifying activity for 24+ hours
- Milestone rewards: 3, 7, 14, 30, 60, 100, 200, 365 days
- Special rules: Grace period of 6 hours for late-night/early-morning activities

**Weekly Consistency Streak**
- Qualifying activities: 5+ days of activity within a 7-day period
- Reset condition: Less than 5 days of activity in a week
- Milestone rewards: 2, 4, 8, 12, 24, 52 weeks
- Special rules: Week runs Monday-Sunday, partial weeks don't count

**Perfect Score Streak**
- Qualifying activities: Achieving perfect scores on Quiz or Challenge stages
- Reset condition: Any non-perfect score on Quiz or Challenge stages
- Milestone rewards: 3, 5, 10, 15, 25, 50 perfect scores
- Special rules: Only applies to games with clear perfect score criteria

**Assignment Completion Streak**
- Qualifying activities: Completing assigned games or sequences
- Reset condition: Missing or failing to complete an assigned activity
- Milestone rewards: 5, 10, 20, 50, 100 assignments
- Special rules: Only applies to teacher-assigned activities

### Point Display and Management

**Student Dashboard Integration**
- Total points prominently displayed with animated counter
- Current streak status with visual progress indicators
- Recent point earnings with source descriptions
- Streak milestones and upcoming rewards
- Point earning tips and encouragement messages

**Point History and Analytics**
- Detailed transaction history with filtering options
- Point earning trends over time (daily, weekly, monthly)
- Streak history and longest streak achievements
- Comparison with class averages (if enabled by teacher)
- Goal setting and progress tracking

## UX requirements (if applicable)

### Point Display Interface

**Main Points Display**
- Large, animated point counter showing current total
- Streak indicator with fire icon and count
- Recent earnings ticker showing latest point sources
- Progress bars for upcoming milestones
- Celebration animations for significant point gains

**Point History View**
- Chronological list of point transactions
- Filter by transaction type, date range, or point amount
- Search functionality for specific activities
- Export options for personal record keeping
- Visual charts showing earning trends

**Streak Progress Display**
- Current streak count with visual progress bar
- Streak type indicators (daily, weekly, perfect scores)
- Milestone progress showing next reward
- Streak calendar showing activity patterns
- Motivational messages and encouragement

### Visual Design Elements

**Point Counter Design**
- Large, bold numbers with musical note accents
- Color coding: Bronze (0-999), Silver (1000-4999), Gold (5000+)
- Animated counting when points are earned
- Glow effects for milestone achievements
- Responsive sizing for different screen sizes

**Streak Indicators**
- Fire icon with animated flames for active streaks
- Streak count in prominent, easy-to-read font
- Progress rings showing progress to next milestone
- Color coding: Red (active), Gray (broken), Gold (milestone)
- Celebration effects for streak milestones

**Point Earning Animations**
- Floating point numbers that count up from source
- Confetti or particle effects for significant gains
- Sound effects for point earning (optional)
- Smooth transitions between point states
- Celebration overlays for major milestones

### Accessibility Features

**Visual Accessibility**
- High contrast mode for point displays
- Large text options for point counters
- Color-blind friendly streak indicators
- Clear focus states for interactive elements
- Screen reader compatible point announcements

**Cognitive Accessibility**
- Simple, clear point earning explanations
- Consistent visual hierarchy
- Predictable interaction patterns
- Clear feedback for all point-earning actions
- Progress indicators for complex achievements

**Motor Accessibility**
- Large touch targets for mobile users
- Keyboard navigation support
- Voice-over compatible point information
- Reduced motion options for animations
- Easy-to-tap streak progress indicators

## Telemetry

### Events

**Point Earning Events**
- `points_earned`: When student earns any points
- `point_milestone_reached`: When student reaches point milestones
- `point_bonus_awarded`: When bonus points are earned
- `point_multiplier_applied`: When streak multiplier is applied

**Streak Events**
- `streak_started`: When a new streak begins
- `streak_continued`: When streak count increases
- `streak_broken`: When a streak is broken
- `streak_milestone_reached`: When streak milestone is achieved
- `streak_bonus_earned`: When streak-based bonus is awarded

**Interaction Events**
- `points_dashboard_viewed`: When student views points dashboard
- `point_history_viewed`: When student views point history
- `streak_progress_viewed`: When student views streak progress
- `point_celebration_shown`: When point celebration is displayed

### Event Properties

**Point Events**
- `point_amount`: Number of points earned
- `point_source`: What activity earned the points
- `bonus_type`: Type of bonus applied (if any)
- `multiplier_value`: Streak multiplier applied (if any)
- `total_points`: Student's new total point balance

**Streak Events**
- `streak_type`: Type of streak (daily, weekly, perfect scores, etc.)
- `streak_count`: Current streak count
- `previous_count`: Previous streak count
- `milestone_threshold`: Next milestone threshold
- `days_since_last_activity`: Days since last qualifying activity

**Interaction Events**
- `view_duration`: How long student spent viewing points/streaks
- `interactions_count`: Number of interactions with point interface
- `filter_applied`: What filters were applied to point history
- `celebration_duration`: How long celebration animation played

## Permissions

### Student Permissions
- **Read**: View their own points and streak data
- **Track**: Automatically earn points and maintain streaks based on activities
- **Configure**: Set privacy preferences for point/streak sharing
- **Export**: Download their point and streak history

### Teacher Permissions
- **Read**: View points and streak data for their students
- **Award**: Grant bonus points for special achievements
- **Configure**: Enable/disable specific point earning rules for their class
- **Report**: Access point and streak analytics for their students
- **Override**: Adjust point totals if necessary (with justification)

### Parent Permissions (if applicable)
- **Read**: View their child's points and streak progress
- **Configure**: Set privacy preferences for their child's point/streak sharing
- **Celebrate**: Receive notifications about their child's point/streak achievements

### System Administrator Permissions
- **Full Access**: View and modify all point and streak data
- **Configure**: Modify point earning rules and streak criteria
- **Analytics**: Access comprehensive point and streak system analytics
- **Reset**: Reset student points or streaks if necessary
- **Moderate**: Manage point/streak sharing and public display

## Dependencies

### Core System Dependencies
- **Game Model**: Game completion and scoring system ([Game Model](../08-games-registry-and-authoring/game-model.md))
- **Student Management**: Student accounts and progress tracking ([Student Dashboard](../03-student-experience/student-dashboard.md))
- **Assignment System**: Assignment completion and progress ([Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- **Sequence System**: Learning sequence completion tracking ([Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))

### Gamification Dependencies
- **Badges System**: Achievement recognition and celebration ([Badges System](../12-gamification/badges-system.md))
- **Leaderboards**: Competitive achievements and rankings ([Leaderboards](../03-student-experience/leaderboards.md))
- **Challenge Boards**: Challenge game mechanics ([Challenge Boards](../12-gamification/challenge-boards.md))
- **Achievements Sharing**: Social features and sharing ([Achievements Sharing](../12-gamification/achievements-sharing.md))

### Technical Dependencies
- **Notification System**: Point and streak achievement notifications ([In-app Notifications](../13-messaging-and-notifications/in-app-notifications.md))
- **Analytics Engine**: Event tracking and point/streak analytics ([Event Model](../15-analytics-and-reporting/event-model.md))
- **Caching System**: High-performance caching for point and streak data
- **Real-time Updates**: Live updates for point earning and streak changes

## Open questions

### Point System Design
- Should points have different values for different difficulty levels or student ages?
- Should there be a point cap or decay system to prevent inflation?
- How should we handle point earning for students who play games multiple times?
- Should points be tradeable or transferable between students?

### Streak System Design
- Should streaks have different rules for different age groups or learning levels?
- How should we handle time zone differences for daily streak calculations?
- Should there be streak recovery options for students who miss by small margins?
- How should we balance streak pressure with learning motivation?

### Integration and Balance
- How should points and streaks integrate with the existing scoring system?
- Should points influence leaderboard rankings or remain separate?
- How can we ensure points motivate learning rather than just point collection?
- Should there be different point systems for different learning contexts?

### Technical Implementation
- What is the optimal database design for high-volume point transactions?
- How should we handle point calculation during high-concurrency periods?
- What caching strategy is best for point and streak data?
- How should we handle point data migration and versioning?

### Pedagogical Considerations
- How can we ensure points and streaks support learning objectives rather than distract from them?
- Should point earning be tied to learning mastery or just completion?
- How should we handle students who focus on points rather than learning?
- What role should teachers play in managing point and streak systems?

### Future Enhancements
- Should points be redeemable for rewards or remain purely motivational?
- Would seasonal or limited-time point multipliers add value?
- How could AI be used to personalize point earning and streak goals?
- Should points and streaks integrate with external learning platforms?
