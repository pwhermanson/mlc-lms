# Student Profile

## Purpose

The student profile serves as the central repository for student identity, preferences, learning data, and personalization settings within the MLC-LMS platform. This specification defines how student information is collected, stored, displayed, and managed to support personalized learning experiences while maintaining privacy and security standards required for educational environments.

The student profile enables:
- **Personalized Learning Experience**: Customized content delivery based on student preferences, learning style, and performance history
- **Progress Tracking**: Comprehensive view of learning achievements, mastery levels, and skill development over time
- **Social Learning**: Safe sharing of achievements and participation in class leaderboards while maintaining privacy
- **Accessibility Support**: Accommodation of diverse learning needs through customizable interface and content options
- **Parent Communication**: Transparent progress sharing with parents and guardians through appropriate privacy controls
- **Teacher Insights**: Rich data for teachers to understand student needs and customize instruction

## Scope

**Included:**
- Student identity and basic information management
- Learning preferences and personalization settings
- Achievement and progress display with privacy controls
- Avatar and visual customization options
- Accessibility and learning accommodation settings
- Parent/guardian communication preferences
- Social learning features (leaderboards, sharing) with privacy controls
- Integration with [Student Dashboard](../03-student-experience/student-dashboard.md) and [Assignment Play](../03-student-experience/assignment-play.md)
- Compliance with COPPA and FERPA requirements for educational data privacy

**Excluded:**
- Student account creation and authentication (handled by [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md))
- Detailed learning analytics and reporting (covered in [Teacher Reports](../04-teacher-experience/teacher-reports.md))
- Assignment management and creation (handled by [Assignment Builder](../04-teacher-experience/assignment-builder.md))
- Billing and subscription management (covered in [Payments and Subscriptions](../14-payments-and-subscriptions/))
- System administration and user management (covered in [System Admin](../06-system-admin/) folder)

## Data Model

### Core Student Profile Entity
- **student_id**: Unique identifier for the student account
- **username**: Unique login identifier (created by teacher/admin)
- **display_name**: Student's preferred display name for profile and leaderboards
- **first_name**: Student's first name (for teacher/admin reference)
- **last_name**: Student's last name (for teacher/admin reference)
- **avatar_id**: Selected avatar identifier from available options
- **current_level**: Current learning level based on mastery progression
- **total_points**: Lifetime points earned across all activities
- **current_streak**: Consecutive days of practice
- **longest_streak**: Record for longest practice streak
- **account_created_at**: When the student account was created
- **last_activity_at**: Timestamp of last platform activity
- **is_active**: Boolean flag for account status

### Learning Preferences
- **learning_style**: Visual, aural, or kinesthetic preference
- **difficulty_preference**: Preferred challenge level (auto, easy, medium, hard)
- **theme_preference**: Light, dark, or high contrast mode
- **language_preference**: Interface language setting
- **font_size**: Text size preference (small, medium, large, extra-large)
- **motion_preference**: Reduced motion setting for accessibility
- **sound_preference**: Audio feedback and music volume settings
- **input_method**: Preferred input method (mouse, touch, MIDI keyboard)

### Accessibility Settings
- **high_contrast_mode**: Enhanced contrast for visual accessibility
- **screen_reader_support**: Optimized content for assistive technology
- **keyboard_navigation**: Full keyboard accessibility preferences
- **voice_control**: Voice command support settings
- **switch_control**: Support for assistive input devices
- **color_blind_support**: Color accessibility accommodations
- **dyslexia_support**: Font and spacing accommodations for reading difficulties

### Social Learning Settings
- **leaderboard_participation**: Opt-in/out of class leaderboards
- **achievement_sharing**: Control over sharing achievements with class
- **privacy_level**: Public, class-only, or private profile visibility
- **parent_notifications**: Whether parents receive progress updates
- **teacher_notes_visible**: Allow teachers to see personal notes and preferences

### Progress and Achievement Data
- **mastery_levels**: Bronze, Silver, Gold achievement levels by skill area
- **badges_earned**: Collection of earned achievement badges
- **games_completed**: Total count of completed games and stages
- **total_practice_time**: Cumulative time spent in learning activities
- **best_scores**: Highest scores achieved in each game category
- **improvement_trends**: Performance improvement metrics over time
- **current_sequence_id**: Active learning sequence assignment
- **sequence_progress**: Progress through current learning sequence

### Parent/Guardian Information
- **parent_email**: Contact email for progress updates (optional)
- **parent_name**: Parent/guardian name for communication
- **communication_preferences**: How often to receive progress updates
- **emergency_contact**: Additional contact information if needed

### Relationships
- Student belongs to Organization (via subscription)
- Student has assigned Teacher(s) for instruction and oversight
- Student has many Game Scores linked to specific games and stages
- Student has many Achievements and Badges earned over time
- Student has one active Learning Sequence at a time
- Student profile data is accessible to assigned Teachers and Teacher-Admin
- Student profile respects privacy settings for parent and peer visibility

## Behavior and Rules

### Profile Creation and Management
- **Account Creation**: Student accounts are created by Teachers, Teacher-Admin, or Subscriber-Admin with unique usernames and passwords
- **Username Uniqueness**: System enforces unique usernames across all organizations to prevent conflicts
- **Password Security**: Students cannot change their own passwords; password changes must be initiated by teachers or administrators
- **Profile Updates**: Students can update display preferences, avatar, and learning settings
- **Privacy Controls**: Students can adjust privacy settings within teacher-defined boundaries

### Learning Progression and Leveling
- **Level Advancement**: Student levels advance based on mastery achievement across skill areas
- **Point Accumulation**: Points are earned through game completion, mastery achievement, and practice streaks
- **Streak Tracking**: Practice streaks are maintained through consecutive days of activity
- **Mastery Recognition**: Bronze, Silver, Gold levels are achieved through consistent high performance

### Social Learning and Privacy
- **Leaderboard Participation**: Students can opt-in to class leaderboards while maintaining privacy
- **Achievement Sharing**: Students control which achievements are visible to classmates
- **Parent Communication**: Progress updates to parents respect student privacy preferences
- **Teacher Visibility**: Teachers can see detailed progress data for instructional purposes

### Accessibility and Accommodation
- **Universal Design**: Profile supports multiple learning styles and accessibility needs
- **Customization Options**: Extensive personalization options for diverse learning needs
- **Assistive Technology**: Full compatibility with screen readers and other assistive devices
- **Input Flexibility**: Support for mouse, touch, keyboard, and MIDI input methods

### Data Privacy and Compliance
- **COPPA Compliance**: No personal information collected directly from students under 13
- **FERPA Compliance**: Educational data privacy protection for all students
- **Data Retention**: Student data retained for 13 months after account deactivation
- **Parental Consent**: Required for students under 13, managed through teacher/admin interface

## UX Requirements

### Profile Display Layout
- **Header Section**
  - Student avatar and display name
  - Current level indicator with progress to next level
  - Total points and current streak display
  - Quick access to settings and preferences
- **Main Content Areas**
  - **Achievements Panel**: Recently earned badges and milestone celebrations
  - **Progress Overview**: Visual representation of learning progress across skill areas
  - **Social Elements**: Class leaderboard position and shared achievements (if opted-in)
  - **Learning Stats**: Games completed, practice time, improvement trends
  - **Current Focus**: Active assignments and next learning goals

### Personalization Interface
- **Avatar Selection**: Grid of available avatars including Terry Treble mascot
- **Theme Customization**: Light/dark mode toggle with high contrast option
- **Accessibility Settings**: Comprehensive accessibility options panel
- **Learning Preferences**: Learning style and difficulty preference selectors
- **Privacy Controls**: Clear, easy-to-understand privacy setting toggles

### Responsive Design
- **Mobile Layout**: Stacked cards with touch-friendly controls and simplified navigation
- **Tablet Layout**: Grid layout with sidebar for settings and detailed information
- **Desktop Layout**: Multi-column layout with comprehensive information display
- **Accessibility**: Full keyboard navigation and screen reader compatibility

### Visual Design Elements
- **Achievement Badges**: Colorful, engaging badges with clear earning criteria
- **Progress Indicators**: Circular and linear progress bars with clear completion states
- **Level Progression**: Visual level indicators showing advancement through learning stages
- **Streak Counters**: Animated counters showing practice streaks and milestones
- **Social Elements**: Class leaderboard integration with privacy-safe display

### Error States and Feedback
- **Loading States**: Skeleton screens and progress indicators during data loading
- **Error Messages**: Clear, actionable error messages with recovery options
- **Success Feedback**: Positive reinforcement for profile updates and achievements
- **Validation**: Real-time validation for profile updates and setting changes

## Telemetry

### Profile Interaction Events
- `profile_viewed`
  - `student_id`, `viewer_role`, `viewer_id`, `profile_section`
- `profile_updated`
  - `student_id`, `field_updated`, `old_value`, `new_value`, `update_source`
- `avatar_changed`
  - `student_id`, `new_avatar_id`, `previous_avatar_id`
- `privacy_setting_changed`
  - `student_id`, `setting_name`, `new_value`, `previous_value`
- `achievement_earned`
  - `student_id`, `achievement_id`, `achievement_type`, `points_earned`
- `level_advanced`
  - `student_id`, `new_level`, `previous_level`, `advancement_reason`

### Learning Progress Events
- `streak_updated`
  - `student_id`, `current_streak`, `longest_streak`, `streak_type`
- `points_earned`
  - `student_id`, `points_added`, `total_points`, `source_activity`
- `mastery_achieved`
  - `student_id`, `skill_area`, `mastery_level`, `games_completed`
- `preference_updated`
  - `student_id`, `preference_type`, `new_value`, `learning_impact`

### Social Learning Events
- `leaderboard_participation_changed`
  - `student_id`, `participation_status`, `class_id`
- `achievement_shared`
  - `student_id`, `achievement_id`, `sharing_scope`, `recipient_count`
- `parent_notification_sent`
  - `student_id`, `parent_email`, `notification_type`, `content_summary`

### Performance Metrics
- `profile_load_time`: Time to display complete profile information
- `settings_update_time`: Time to process profile setting changes
- `achievement_display_time`: Time to show new achievements and celebrations
- `privacy_control_usage`: Frequency of privacy setting adjustments

## Permissions

### Student Permissions
- **Read Access**: View own profile information and progress data
- **Update Access**: Modify personal preferences, avatar, and privacy settings
- **Achievement Access**: View earned badges and progress milestones
- **Social Access**: Participate in leaderboards and sharing (with privacy controls)

### Teacher Permissions
- **Profile Visibility**: View student profile information for instructional purposes
- **Progress Access**: See detailed learning progress and achievement data
- **Settings Override**: Modify certain student settings for accessibility or learning needs
- **Communication**: Send messages and notes visible in student profile

### Teacher-Admin Permissions
- **Class Profile Access**: View profiles of all students in their scope
- **Bulk Operations**: Update multiple student profiles for class-wide changes
- **Privacy Management**: Oversee privacy settings and social learning participation
- **Progress Monitoring**: Access comprehensive progress data for program oversight

### Parent Permissions (if enabled)
- **Progress Visibility**: View student progress and achievement summaries
- **Communication**: Receive progress updates and milestone notifications
- **Limited Profile Access**: See basic profile information with privacy controls
- **No Direct Modification**: Cannot change student profile settings directly

### System Permissions
- **Data Management**: System can update progress data and achievement records
- **Privacy Enforcement**: System enforces privacy settings and data access controls
- **Compliance Monitoring**: System tracks data access for privacy compliance
- **Backup and Recovery**: System can backup and restore profile data

## Supporting Documents Referenced

This student profile specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Student role capabilities and profile data access specifications
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Student profile content, messaging, and interface elements
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Profile screen text and user-facing messages

## Dependencies

### Core System Dependencies
- [Student Dashboard](../03-student-experience/student-dashboard.md) for profile display and navigation
- [Assignment Play](../03-student-experience/assignment-play.md) for learning progress integration
- [Student Assignments](../03-student-experience/student-assignments.md) for assignment progress tracking
- [Roles and Permissions](../02-roles-and-permissions/roles-matrix.md) for access control and privacy management

### Learning and Content Dependencies
- [Game Model](../08-games-registry-and-authoring/game-model.md) for achievement and scoring integration
- [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) for learning progression tracking
- [Gamification System](../12-gamification/badges-system.md) for achievement and badge management
- [Leaderboards](../03-student-experience/leaderboards.md) for social learning integration

### Technical Dependencies
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) for authentication and security
- [Accessibility Standards](../01-ux-design-system/a11y-standards.md) for inclusive design implementation
- [Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md) for multi-device support
- [Event Model](../15-analytics-and-reporting/event-model.md) for telemetry and analytics

### Privacy and Compliance Dependencies
- [COPPA Compliance](../23-legal-and-compliance/coppa-ferpa-notes.md) for children's privacy protection
- [FERPA Compliance](../23-legal-and-compliance/ferpa-compliance.md) for educational data privacy
- [Privacy Policy](../23-legal-and-compliance/privacy-policy.md) for data handling and user rights
- [Terms and Conditions](../23-legal-and-compliance/terms-conditions.md) for user agreement and data usage

## Integration with MLC Ecosystem

The student profile serves as a central coordination point for personalized learning experiences across the MLC platform:

### Learning Personalization
- **Adaptive Content**: Profile preferences drive content presentation and difficulty adjustment
- **Learning Style Support**: Visual, aural, and kinesthetic preferences influence game presentation
- **Accessibility Integration**: Profile accommodations ensure inclusive learning experiences
- **Progress Tracking**: Comprehensive learning data feeds into personalized recommendations

### Social Learning Integration
- **Class Community**: Profile settings control participation in class leaderboards and sharing
- **Achievement Recognition**: Badges and milestones are displayed and shared based on privacy preferences
- **Peer Learning**: Safe social features encourage collaborative learning while maintaining privacy
- **Teacher Communication**: Profile data enables personalized teacher-student interactions

### Parent Engagement
- **Progress Transparency**: Parent-visible progress summaries respect student privacy preferences
- **Communication Preferences**: Profile settings control frequency and content of parent updates
- **Learning Support**: Parents can understand student needs and preferences for home support
- **Achievement Celebration**: Family-friendly achievement sharing encourages continued engagement

### Teacher Support
- **Instructional Insights**: Profile data helps teachers understand individual student needs
- **Accommodation Support**: Accessibility settings guide teachers in providing appropriate support
- **Progress Monitoring**: Comprehensive progress data enables effective intervention and support
- **Communication Facilitation**: Profile preferences guide teacher-student communication approaches

## Addressing Historical Pain Points

Based on feedback from the original MLC system and user research, the student profile specifically addresses these key issues:

### Personalization and Engagement
- **Individual Identity**: Students can create a personalized profile with avatar and preferences
- **Achievement Recognition**: Clear display of badges, points, and progress milestones
- **Social Connection**: Safe participation in class activities while maintaining privacy
- **Motivation Support**: Visual progress tracking and achievement celebration

### Accessibility and Inclusion
- **Diverse Learning Needs**: Comprehensive accessibility options for students with special needs
- **Multiple Learning Styles**: Support for visual, aural, and kinesthetic learners
- **Input Flexibility**: Support for mouse, touch, keyboard, and MIDI input methods
- **Customizable Interface**: Extensive personalization options for individual preferences

### Privacy and Safety
- **Age-Appropriate Privacy**: COPPA-compliant data handling for students under 13
- **Parental Control**: Appropriate parent visibility and communication preferences
- **Safe Social Features**: Privacy-controlled participation in class activities
- **Data Protection**: FERPA-compliant educational data privacy protection

### Progress Visibility
- **Clear Achievement Display**: Easy-to-understand progress indicators and milestones
- **Learning Insights**: Visual representation of skill development and improvement
- **Goal Setting**: Clear next steps and learning objectives
- **Celebration of Success**: Positive reinforcement for learning achievements

## Open Questions

- Should students be able to create custom avatars or only select from predefined options?
- How should the system handle profile data migration when students move between teachers or schools?
- What level of parent access should be available for different age groups?
- Should there be different profile templates for different learning contexts (private studio vs. school)?
- How should the system handle profile data for Generic Students in classroom settings?
- What privacy controls should be available for students in different age groups?
- Should there be profile sharing capabilities between students in the same class?
- How should the system handle profile data backup and recovery for long-term student records?
- What level of profile customization should be available to teachers for their students?
- How should the system handle profile data when students transition from trial to paid subscriptions?

