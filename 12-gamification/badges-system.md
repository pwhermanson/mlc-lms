# Badges System

This document defines the comprehensive badges and achievement system that recognizes student progress, celebrates milestones, and provides motivational feedback throughout their musical learning journey.

## Purpose

The badges system implements the core pedagogical principle that "success is the best motivator" by providing immediate, meaningful recognition of student achievements. It transforms abstract learning progress into concrete, visual accomplishments that students can collect, display, and share. The system encourages continued engagement, celebrates both small wins and major milestones, and creates a sense of progression and mastery that drives long-term learning motivation.

## Scope

**Included**
- Achievement badges for game completion and mastery milestones
- Progress badges for consistent practice and engagement
- Skill-based badges for specific musical concept mastery
- Challenge badges for competitive achievements and leaderboard success
- Collection badges for completing learning sequences and curricula
- Special recognition badges for exceptional performance and dedication
- Badge display and sharing functionality
- Badge notification and celebration system
- Badge analytics and progress tracking

**Excluded**
- Point accumulation system (covered by [Points and Streaks](../12-gamification/points-and-streaks.md))
- Leaderboard rankings (covered by [Leaderboards](../03-student-experience/leaderboards.md))
- Challenge game mechanics (covered by [Game Model](../08-games-registry-and-authoring/game-model.md))
- Assignment progress tracking (covered by [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- Teacher reporting on student achievements (covered by [Teacher Reports](../04-teacher-experience/teacher-reports.md))

## Data model (if applicable)

### Core Entities

**Badge Definition**
- `badge_id`: Unique identifier (e.g., "first_game_complete", "treble_clef_master", "challenge_champion")
- `badge_name`: Display name for the badge
- `badge_description`: Detailed description of what the badge represents
- `badge_category`: "completion", "progress", "skill", "challenge", "collection", "special"
- `badge_rarity`: "common", "uncommon", "rare", "epic", "legendary"
- `badge_icon`: Icon identifier for visual representation
- `badge_color_scheme`: Primary and secondary colors for the badge
- `badge_animation`: Animation type for badge earning and display
- `criteria_type`: "single_game", "game_series", "score_threshold", "streak_count", "time_based", "sequence_completion"
- `criteria_config`: JSON configuration defining specific requirements
- `is_active`: Whether the badge is currently available for earning
- `created_at`: When the badge was created
- `updated_at`: When the badge definition was last modified

**Student Badge**
- `student_badge_id`: Unique identifier for the student's badge instance
- `student_id`: Reference to the student who earned the badge
- `badge_id`: Reference to the badge definition
- `earned_at`: Timestamp when the badge was earned
- `context_data`: JSON data about the specific achievement (score, game, sequence, etc.)
- `is_displayed`: Whether the student has chosen to display this badge
- `is_shared`: Whether the student has shared this badge achievement
- `notification_sent`: Whether the student has been notified about earning this badge

**Badge Collection**
- `collection_id`: Unique identifier for a badge collection
- `collection_name`: Display name for the collection
- `collection_description`: Description of what the collection represents
- `badge_ids`: Array of badge IDs that belong to this collection
- `completion_reward`: Special badge or reward for completing the entire collection
- `is_active`: Whether the collection is currently available

### Badge Categories and Types

**Completion Badges**
- First game completion
- First perfect score
- First Challenge stage completion
- First assignment completion
- First sequence completion

**Progress Badges**
- Daily practice streaks (3, 7, 14, 30, 100 days)
- Weekly practice consistency
- Monthly engagement milestones
- Total games played milestones
- Total time spent learning

**Skill Badges**
- Treble clef mastery
- Bass clef mastery
- Note reading fluency
- Rhythm recognition
- Interval identification
- Chord recognition
- Scale mastery
- Key signature recognition

**Challenge Badges**
- Leaderboard top 10
- Leaderboard top 5
- Leaderboard #1
- Personal best achievements
- Perfect Challenge scores
- Speed mastery (60 Second Club games)

**Collection Badges**
- Complete beginner sequence
- Complete intermediate sequence
- Complete advanced sequence
- Complete specific method alignment (Piano Adventures, RCM, etc.)
- Complete all games in a category
- Complete all Challenge games

**Special Recognition Badges**
- Teacher's choice awards
- Parent recognition
- Peer nominations
- System-generated exceptional performance
- Community contributions

## Behavior and rules

### Badge Earning Criteria

**Single Game Completion**
- Complete any game stage (Learn, Play, Quiz, Challenge, Review)
- Achieve target score on Quiz stage
- Complete Challenge stage with qualifying score
- Achieve perfect score on any stage

**Game Series Completion**
- Complete all stages of a specific game
- Complete all games in a learning sequence
- Complete all games in a skill category
- Complete all Challenge games

**Score-Based Achievements**
- Achieve specific score thresholds
- Beat personal best scores
- Achieve perfect scores
- Achieve scores above certain percentiles

**Streak-Based Achievements**
- Practice for consecutive days
- Complete assignments for consecutive weeks
- Maintain high performance over time periods
- Consistent engagement without breaks

**Time-Based Achievements**
- Complete games within time limits
- Maintain focus for extended periods
- Quick mastery of new concepts
- Efficient learning progression

### Badge Earning Process

**Real-time Evaluation**
1. Student completes an action that could trigger badge criteria
2. System evaluates all active badges against the action
3. If criteria are met, badge is immediately awarded
4. Celebration animation is displayed
5. Notification is sent to student
6. Badge is added to student's collection

**Batch Evaluation**
1. System periodically evaluates progress-based badges
2. Streak calculations are updated daily
3. Collection completion is checked when new badges are earned
4. Special recognition badges are evaluated by teachers/admins

### Badge Display and Sharing

**Student Badge Collection**
- Personal badge showcase on student dashboard
- Filterable by category, rarity, and date earned
- Search functionality for specific badges
- Badge details and earning context display

**Public Badge Display**
- Optional public profile showing earned badges
- Privacy controls for which badges to display
- Sharing functionality for individual badge achievements
- Social features for celebrating peer achievements

**Badge Notifications**
- In-app notifications for newly earned badges
- Email notifications for significant achievements
- Push notifications for mobile users
- Celebration animations and sound effects

## UX requirements (if applicable)

### Badge Display Interface

**Badge Collection View**
- Grid layout showing badge icons with rarity color coding
- Hover/click interactions revealing badge details
- Filter and sort options (category, rarity, date, earned status)
- Search functionality for finding specific badges
- Progress indicators for badges in progress

**Badge Detail Modal**
- Large badge icon with animation
- Badge name and description
- Earning criteria and context
- Date earned and rarity level
- Sharing and display options
- Related badges or collection information

**Badge Earning Celebration**
- Animated badge appearance with sound
- Confetti or particle effects
- Achievement text overlay
- Progress bar updates
- Social sharing prompts

### Visual Design System

**Badge Rarity Color Coding**
- **Common**: Bronze/silver colors
- **Uncommon**: Blue/silver colors  
- **Rare**: Purple/silver colors
- **Epic**: Gold/purple colors
- **Legendary**: Rainbow/multicolor effects

**Badge Icon Design**
- Consistent 64x64 pixel icon size
- Clear, recognizable symbols
- Musical theme integration
- High contrast for accessibility
- Scalable vector graphics

**Animation Guidelines**
- Smooth, celebratory animations for earning
- Subtle hover effects for interaction
- Loading states for badge data
- Transition animations between views

### Accessibility Features

**Visual Accessibility**
- High contrast mode support
- Color-blind friendly rarity indicators
- Large text options for badge descriptions
- Clear focus states for keyboard navigation

**Cognitive Accessibility**
- Clear, simple badge descriptions
- Consistent visual hierarchy
- Predictable interaction patterns
- Progress indicators for complex achievements

**Motor Accessibility**
- Large touch targets for mobile users
- Keyboard navigation support
- Voice-over compatible badge information
- Reduced motion options for animations

## Telemetry

### Events

**Badge Earning Events**
- `badge_earned`: When a student earns any badge
- `badge_celebration_shown`: When celebration animation is displayed
- `badge_notification_sent`: When notification is sent to student
- `badge_shared`: When student shares a badge achievement

**Badge Interaction Events**
- `badge_collection_viewed`: When student views their badge collection
- `badge_details_viewed`: When student views specific badge details
- `badge_filter_applied`: When student uses filter/sort options
- `badge_search_performed`: When student searches for badges

**Badge Progress Events**
- `badge_progress_updated`: When progress toward a badge is updated
- `badge_criteria_met`: When specific criteria for a badge is completed
- `badge_streak_updated`: When streak-based badge progress is updated

### Event Properties

**Badge Earning Events**
- `badge_id`: The specific badge that was earned
- `badge_category`: Category of the earned badge
- `badge_rarity`: Rarity level of the badge
- `earning_context`: Context about how the badge was earned
- `time_to_earn`: Time taken to earn the badge (if applicable)
- `student_level`: Student's current learning level

**Badge Interaction Events**
- `collection_size`: Total number of badges in student's collection
- `filter_criteria`: What filters were applied
- `search_term`: What was searched for
- `view_duration`: How long student spent viewing badges

**Badge Progress Events**
- `progress_percentage`: Current progress toward badge completion
- `remaining_criteria`: What criteria still need to be met
- `streak_count`: Current streak count for streak-based badges
- `time_remaining`: Estimated time to complete badge (if applicable)

## Permissions

### Student Permissions
- **Read**: View their own badge collection and details
- **Share**: Share individual badge achievements publicly
- **Configure**: Set privacy preferences for badge display
- **Earn**: Automatically earn badges based on their actions

### Teacher Permissions
- **Read**: View badge collections for their students
- **Award**: Grant special recognition badges to students
- **Configure**: Enable/disable specific badges for their class
- **Report**: Access badge analytics for their students

### Parent Permissions (if applicable)
- **Read**: View their child's badge collection and achievements
- **Configure**: Set privacy preferences for their child's badge sharing
- **Celebrate**: Receive notifications about their child's badge achievements

### System Administrator Permissions
- **Full Access**: View and modify all badge definitions and student badges
- **Create**: Create new badge definitions and collections
- **Configure**: Modify badge criteria and earning rules
- **Analytics**: Access comprehensive badge system analytics
- **Moderate**: Manage badge sharing and public display

## Supporting Documents Referenced

This badges system specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game completion and achievement tracking requirements
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform messaging and achievement communication
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — User interface text for rewards and achievements

## Dependencies

### Core System Dependencies
- **Game Model**: Game completion and scoring system ([Game Model](../08-games-registry-and-authoring/game-model.md))
- **Student Management**: Student accounts and progress tracking ([Student Dashboard](../03-student-experience/student-dashboard.md))
- **Assignment System**: Assignment completion and progress ([Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- **Sequence System**: Learning sequence completion tracking ([Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))

### Gamification Dependencies
- **Points System**: Point accumulation and display ([Points and Streaks](../12-gamification/points-and-streaks.md))
- **Leaderboards**: Competitive achievements and rankings ([Leaderboards](../03-student-experience/leaderboards.md))
- **Challenge Boards**: Challenge game mechanics ([Challenge Boards](../12-gamification/challenge-boards.md))
- **Achievements Sharing**: Social features and sharing ([Achievements Sharing](../12-gamification/achievements-sharing.md))

### Technical Dependencies
- **Notification System**: Badge earning notifications ([In-app Notifications](../13-messaging-and-notifications/in-app-notifications.md))
- **Analytics Engine**: Event tracking and badge analytics ([Event Model](../15-analytics-and-reporting/event-model.md))
- **Media System**: Badge icons and animations ([CDN and Media](../17-integrations/cdn-and-media.md))
- **Caching System**: High-performance badge data caching

## Open questions

### Badge Design and Content
- Should badges have different visual styles for different age groups?
- How many badges should be available at launch vs. added over time?
- Should badges have expiration dates or be permanent achievements?
- How should we handle badge updates or changes to earning criteria?

### Social and Sharing Features
- Should students be able to comment on or react to peer badge achievements?
- What level of privacy control should students have over badge sharing?
- Should there be group or team badge achievements?
- How should we handle inappropriate badge sharing or display?

### Pedagogical Considerations
- Should badges be tied to specific learning objectives or be more general?
- How can we ensure badges motivate learning rather than just collection?
- Should there be different badge systems for different learning levels?
- How should we balance intrinsic vs. extrinsic motivation in badge design?

### Technical Implementation
- What is the optimal database design for badge data with high read/write volume?
- How should we handle badge data synchronization across devices?
- What caching strategy is best for badge collections and progress tracking?
- How should we handle badge data migration and versioning?

### Performance and Scalability
- What is the maximum number of badges a student can have before performance degrades?
- How should we handle badge analytics with millions of students?
- What are the performance implications of real-time badge evaluation?
- How should we archive or manage historical badge data?

### Future Enhancements
- Should badges be tradeable or transferable between students?
- Would seasonal or limited-time badges add value?
- How could AI be used to create personalized badge recommendations?
- Should badges integrate with external platforms or social media?
