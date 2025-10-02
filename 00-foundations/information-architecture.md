# Information Architecture

## Purpose

This document outlines the overall information structure and navigation of the MLC-LMS platform. It defines how users navigate through different sections, access features, and find information based on their role and context.

## Top-level Areas

### Student
**Navigation Flow**: Dashboard → Assignments → Assignment Play → All Games → Profile → Badges/Leaderboards → Messages → Settings

**Key Principles**: Student flows emphasize next-up assignments, visible targets, and quick returns to dashboard

### Teacher
**Navigation Flow**: Dashboard → Roster → Assignment Builder → Reports → Messaging → Class Settings → Challenges

**Key Principles**: Aligns to method-based sequences and fast monitoring

### Admin (Subscriber-Admin / Teacher-Admin)
**Navigation Flow**: Subscriber Dashboard → Member Management → Student Bulk Ops → Mass Assignments → Billing (plans, seats, invoices, POs, statements) → Codes (promo, sponsor) → Reports

**Key Principles**: Designed for school scale

### System Admin
**Navigation Flow**: Global User Index → Member Profile View (hierarchies) → Billing Tools → Content Tools → Audit Logs → Feature Flags

**Key Principles**: Operates without storing or exposing member passwords

## Core Entities

### User Management
- **User**: Auth account with one or more roles
- **Member**: Subscription-owning org with seats and billing
- **Student**: Belongs to a Member; may have teacher relationships and assignments
- **Teacher / Teacher-Admin**: Manages students and classes; builds assignments

### Content Structure
- **Game**: Concept-focused set of stages; persists scores by stage
- **Learning Element**: Game stage or media/reward unit with an ID
- **Assignment / Group / Sequence**: Composition layers for pedagogy delivery

### Billing Objects
- **Plan / Invoice / Statement / Code**: Billing and marketing objects

## Subscription Plan Hierarchies

### Plan Types and Role Flexibility
The platform supports three subscription tiers with different role configurations:

**SOLO Plan**: 
- Subscriber-Administrator and Teacher roles are combined in one person
- Teacher-Admin role is not available (grayed out)
- Direct student management by the single administrator

**ENSEMBLE Plan**:
- Subscriber-Administrator can optionally create separate Teacher-Admin accounts
- Supports multiple Teachers under one Teacher-Admin
- Enables separation of business (billing) and educational (teaching) responsibilities
- Ideal for schools where business office and music department are separate

**PRELUDE Plan**:
- Subscriber-Administrator can create one Teacher account
- Teacher-Admin role is not available (grayed out)
- Limited to single teacher configuration

### Role Capability Matrix
Based on subscription type, roles have different capabilities:

**Subscriber-Administrator**:
- Full system access except student game playing
- Payment and subscription management
- Seat management (add/reduce student seats)
- Pause and restart subscriptions
- Christine Hermanson Grant Program administration
- All Teacher-Admin and Teacher capabilities when those roles are absent

**Teacher-Admin** (ENSEMBLE only):
- All Teacher capabilities plus:
- Create and manage multiple Teachers
- Assign students to specific Teachers
- View all student scores across the organization
- Delete student records and score history
- Receive system communications

**Teacher**:
- Create and manage students
- Assign sequences and individual games
- View assigned student scores
- Play games (scores not recorded)
- Access teaching resources and reports

**Student**:
- Play assigned and individual games
- View personal score history
- Participate in challenge games and leaderboards
- Access student-specific resources

## Navigation Principles

### Role-Based Access
- Each user role sees only relevant sections and features
- Navigation adapts based on permissions and context
- Clear visual indicators of current role and available actions
- Global top nav reflects role: Student sees Learn Music and All Games; Teacher sees Learn Music, All Games, Teacher tools; Admin sees Admin tools; System Admin sees SysAdmin tools

### Progressive Disclosure
- Most important features are immediately accessible
- Advanced features are discoverable but not overwhelming
- Contextual help and guidance throughout the interface

### Consistent Patterns
- Similar features are organized in similar ways across roles
- Standard navigation patterns reduce cognitive load
- Clear visual hierarchy guides user attention

## Student Experience Navigation

### Primary Navigation
- **Dashboard**: Home screen with assignments, progress, and quick actions
- **Assignments**: Current and upcoming assignments with progress tracking
- **All Games**: Browse and search the complete games library
- **Profile**: Personal achievements, badges, and settings
- **Messages**: Communication with teachers and parents

### Dashboard Sections
- **Next Up**: Assigned games and activities ready to play
- **Recent Activity**: Recently completed games and scores
- **Achievements**: Badges, points, and streaks earned
- **Progress**: Visual progress indicators for current assignments
- **Teacher Note**: Messages and feedback from teachers

### Assignment Flow
- **Assignment List**: Overview of all assigned content
- **Assignment Detail**: Specific games and activities in an assignment
- **Game Player**: Full-screen game experience
- **Progress Tracking**: Real-time feedback and scoring

## Teacher Experience Navigation

### Primary Navigation
- **Dashboard**: Class overview, alerts, and quick actions
- **Students**: Roster management and individual student views
- **Assignments**: Create, manage, and track assignments
- **Reports**: Analytics and progress reporting
- **Messages**: Communication with students and parents
- **Settings**: Class configuration and preferences

### Dashboard Sections
- **Class Overview**: Student roster with progress indicators
- **Alerts**: Students needing attention or assistance
- **Quick Actions**: Common tasks like creating assignments
- **Recent Activity**: Latest student progress and submissions
- **Upcoming**: Scheduled assignments and deadlines

### Assignment Management
- **Assignment Builder**: Drag-and-drop interface for creating assignments
- **Assignment Library**: Reusable assignment templates
- **Bulk Operations**: Mass assignment and management tools
- **Progress Tracking**: Real-time student progress monitoring

## Admin Experience Navigation

### Subscriber-Admin Navigation
- **Dashboard**: Subscription overview, usage metrics, and alerts
- **Members**: User management and role assignments
- **Billing**: Subscription management, invoices, and payments
- **Reports**: Usage analytics and business metrics
- **Settings**: Account configuration and preferences

### Teacher-Admin Navigation
- **Dashboard**: Teacher and class management overview
- **Teachers**: Manage teacher accounts and permissions
- **Classes**: Organize students and assignments by class
- **Reports**: Program-wide progress and usage analytics
- **Settings**: School-specific configuration

### Admin Dashboard Sections
- **Overview**: Key metrics and system status
- **User Management**: Add, edit, and manage user accounts
- **Bulk Operations**: CSV import/export and mass management
- **Billing Management**: Subscription, seats, and payment processing
- **Reporting**: Comprehensive analytics and data export

## System Admin Navigation

### Primary Navigation
- **Dashboard**: System health, alerts, and key metrics
- **Users**: Global user index with advanced filtering
- **Members**: Detailed member profiles and account management
- **Billing**: Advanced billing tools and overrides
- **Content**: Content management and maintenance tools
- **Audit**: Comprehensive audit logs and activity tracking
- **Settings**: System configuration and feature flags

### System Admin Dashboard
- **System Health**: Performance metrics and error rates
- **User Activity**: Recent user actions and system usage
- **Alerts**: Critical issues requiring attention
- **Quick Actions**: Common administrative tasks
- **Metrics**: Key performance indicators and trends

### System Administration Features
Based on historical requirements and operational needs, the system admin interface provides:

**Member Management**:
- Global user index with advanced filtering capabilities
- Member profile views showing complete organizational structure
- Member data lookup and group data management
- Subscription maintenance and billing integration
- Sponsor code assignment and management

**Content Management**:
- Learning element maintenance and updates
- Named sequence maintenance and definition
- Content library management
- Promotion code administration
- Report generation and management

**Operational Tools**:
- Trial management and reporting
- Scorekeeping and email system testing
- Management reports and analytics
- Communication tools for member outreach
- Audit logging and activity tracking

**System Monitoring**:
- User permission management
- Feature flag controls
- System usage analytics
- Performance monitoring
- Error tracking and resolution

## Content Organization

### Games Library Structure
- **Categories**: Pitch, Rhythm, Melody, Chords, Scales, etc.
- **Difficulty Levels**: Beginner, Intermediate, Advanced
- **Instrument Types**: Piano, General Music, etc.
- **Learning Modalities**: Visual, Aural, Kinesthetic

### Game Navigation Patterns
Based on historical user feedback and design requirements, games follow a specific navigation structure:

**Two-Tier Game Selection**:
1. **Game Selection Screen**: Users choose between default (thumbnail) or list view to select a game
2. **Stage Selection Screen**: Upon game selection, users choose specific stage (Learn, Play, Quiz, Challenge)
3. **Full-Screen Game Play**: Game/stage presented in full-screen experience without header/footer
4. **Return Navigation**: "Quit" returns to Stage Selection Screen, not main navigation

**Game Display Requirements**:
- Games displayed as single entities, not game/stage combinations
- Thumbnails kept small to show more games "above the fold"
- List view shows Game ID, Name, Skill Level, Skill Area, and availability indicator
- Review stages integrated into sequences but not displayed in main game selection

### Assignment Organization
- **Sequences**: Pre-built learning pathways
- **Groups**: Related assignments within sequences
- **Individual Elements**: Specific games and activities
- **Custom Collections**: Teacher-created assignment groups

### Available Learning Sequences
The platform provides several pre-built sequences for different pedagogical approaches:

**Lifetime Musician**: Default sequence designed for comprehensive music literacy development
**Solfege**: Lifetime Musician sequence with fixed-do solfege notation instead of lettered notes
**Evaluation**: Selected Quiz stages for assessing student proficiency and identifying areas for improvement
**Method-Specific Sequences**: Aligned with popular piano methods (Alfred, Faber, Celebrate Piano, etc.)
**State Achievement Programs**: Sequences matching various state music education standards

### Reporting Structure
- **Student Reports**: Individual progress and performance
- **Class Reports**: Group progress and comparison
- **Program Reports**: School-wide analytics and trends
- **System Reports**: Platform usage and performance metrics

## Search and Discovery

### Search Index
- Search index across Games, categories, and sequences
- Filters by level, concept, visual/aural, and stage availability
- Historic wireframes inform student nav patterns

### Global Search
- **Games**: Search by name, concept, or difficulty
- **Students**: Find students by name or class
- **Assignments**: Locate specific assignments or templates
- **Content**: Search across all platform content

### Filtering and Facets
- **Role-based Filters**: Show only relevant content for user role
- **Content Filters**: Difficulty, concept, instrument, modality
- **Status Filters**: Active, completed, pending, overdue
- **Date Filters**: Recent, upcoming, historical

### Quick Access
- **Recent Items**: Recently accessed content
- **Favorites**: Bookmarked games and assignments
- **Quick Actions**: Common tasks and shortcuts
- **Contextual Suggestions**: AI-powered recommendations

## Cross-area Links

### Deep Linking
- From a student score detail, deep-link to teacher reports
- From a teacher report, jump to assignment builder or a Game's registry page
- Contextual navigation between related areas based on user role and current task

### Dashboard Element Access Matrix
Based on user role and subscription type, different dashboard elements are available:

**Student Dashboard Elements**:
- Student Assignment: Next games in assigned sequence (Student access only)
- Individual Games: Any individual game selection (Student, Teacher access)
- Scores History: Personal score tracking (Student access only)
- Student Awards: Achievement recognition and certificates
- Teacher Messages: Communication from assigned teachers

**Teacher Dashboard Elements**:
- Student Management: Create and manage student accounts
- Score Reporting: View assigned student progress (Single Teacher scope)
- Teaching Aides: Instructional resources and materials
- Game Reports: Alpha master listing, level/category views
- Curriculum Reports: Lifetime Musician, method sequences, state programs

**Member Dashboard Elements**:
- Member Credentials: Username and password management
- Member Information: Account and billing details
- Teacher/Student Management: Multi-teacher user administration
- System Tutorials: Platform usage guidance
- Score Reporting: Multi-teacher progress overview
- Upload Utility: Bulk data import/export tools

**System Admin Dashboard Elements**:
- Member Maintenance: Complete organizational management
- Sponsor Code Assignment: Christine Hermanson Grant Program administration
- Communications: System-wide messaging and announcements
- Subscription Maintenance: Billing and payment processing
- Content Library: Learning element and sequence management
- Report Generator: Comprehensive analytics and data export

## Data Flows (High Level)

### Assignment Generation
1. **Assignment generation** pulls steps from a Sequence and presents next-up when the student logs in

### Play Events
2. **Play events** write per-stage scores and durations to history

### Reports
3. **Reports** aggregate at student, class, and member levels for teachers and admins

### Billing
4. **Billing** events sync via provider webhooks; statements generate on demand

## Mobile and Responsive Design

### Mobile Navigation
- **Collapsible Menu**: Hamburger menu for primary navigation
- **Bottom Navigation**: Quick access to main sections
- **Touch-friendly**: Large buttons and touch targets
- **Swipe Gestures**: Natural mobile interaction patterns

### Tablet Navigation
- **Sidebar Navigation**: Persistent navigation panel
- **Split View**: Content and navigation side-by-side
- **Touch and Mouse**: Support for both input methods
- **Adaptive Layout**: Adjusts based on screen orientation

## Accessibility Considerations

### Navigation Accessibility
- **Keyboard Navigation**: Full keyboard accessibility
- **Screen Reader Support**: Proper ARIA labels and landmarks
- **High Contrast**: Clear visual hierarchy and contrast
- **Focus Management**: Clear focus indicators and logical tab order

### Cognitive Accessibility
- **Clear Labels**: Descriptive and consistent navigation labels
- **Predictable Patterns**: Consistent navigation across sections
- **Error Prevention**: Clear feedback and confirmation dialogs
- **Help Integration**: Contextual help and guidance

## Related Documentation

For detailed implementation guidance, refer to:

- **[Roles and Permissions](../02-roles-and-permissions/)** - Detailed role definitions and permission matrices
- **[Student Experience](../03-student-experience/)** - Complete student navigation and interaction patterns
- **[Teacher Experience](../04-teacher-experience/)** - Teacher dashboard and assignment management workflows
- **[Admin Experience](../05-admin-subscriber-experience/)** - Subscriber and admin management interfaces
- **[System Admin](../06-system-admin/)** - System administration tools and capabilities
- **[Content Architecture](../07-content-architecture/)** - Content organization and learning element structure
- **[Games Registry](../08-games-registry-and-authoring/)** - Game management and authoring workflows
- **[Search and Discovery](../11-search-and-discovery/)** - Search functionality and content discovery patterns

## Future Considerations

### Scalability
- **Dynamic Navigation**: Adapt based on user permissions and content
- **Performance**: Efficient loading and caching of navigation elements
- **Personalization**: Customizable navigation based on user preferences
- **Analytics**: Track navigation patterns to improve user experience

### Integration
- **Third-party Tools**: Integration with external educational tools
- **API Access**: Programmatic access to navigation and content
- **Mobile Apps**: Native app navigation patterns
- **Offline Support**: Navigation and content access without internet connection

