# Group Model

This document defines the data model for groups and group management within the MLC-LMS platform. Groups are organizational structures that enable hierarchical management of users, content, and learning activities across different subscription plans and educational contexts.

## Purpose

This specification enables the systematic organization and management of users and content within organizational hierarchies. It establishes the foundation for:

- **Organizational Structure**: Defining how users are organized within subscription plans and educational institutions
- **Content Access Control**: Managing which users can access specific content and features
- **Learning Management**: Enabling teachers to organize students into classes and manage group learning activities
- **Progress Tracking**: Supporting group-level analytics and reporting
- **Permission Management**: Controlling access to features and data based on group membership
- **Subscription Management**: Aligning group structures with billing and seat allocation

## Scope

### Included
- **User Groups**: Organizational hierarchies (Subscriber-Admin, Teacher-Admin, Teacher, Student)
- **Class Groups**: Student groupings for learning management and assignment distribution
- **Sequence Groups**: Content organization for curriculum delivery (LIFE, SOL sequences)
- **Subscription Groups**: Plan-based groupings (Solo, Ensemble, Prelude)
- **Generic Groups**: Shared access groups for classroom learning environments
- **Group Hierarchies**: Multi-level organizational structures with delegation capabilities

### Excluded
- **Individual User Management**: User account creation and authentication (covered in [roles-and-permissions](../02-roles-and-permissions/))
- **Content Creation**: Learning element development (covered in [learning-elements-overview](learning-elements-overview.md))
- **Assignment Logic**: How content is presented to groups (covered in [assignment-model](assignment-model.md))
- **Billing Management**: Subscription and payment processing (covered in [plans-and-pricing](../14-payments-and-subscriptions/plans-and-pricing.md))

## Data Model

### Core Group Types

#### Organizational Groups
**Purpose**: Define the hierarchical structure of users within a subscription

**Structure**:
- **Subscriber Group**: Top-level organization (school, studio, district)
- **Teacher-Admin Group**: Educational management layer (optional in Ensemble plans)
- **Teacher Groups**: Individual teacher accounts with assigned students
- **Student Groups**: Individual student accounts or shared classroom accounts

**Key Fields**:
- `group_id`: Unique identifier for the group
- `group_type`: Type of group (subscriber, teacher_admin, teacher, student, generic)
- `parent_group_id`: Reference to parent group in hierarchy
- `subscription_plan`: Associated subscription plan (solo, ensemble, prelude)
- `created_at`: Group creation timestamp
- `created_by`: User who created the group
- `status`: Group status (active, suspended, archived)

#### Class Groups
**Purpose**: Organize students for learning management and assignment distribution

**Structure**:
- **Regular Classes**: Individual student accounts with unique tracking
- **Generic Classes**: Shared accounts for group learning environments
- **Grade Levels**: Age-based groupings (Primary, Level 1-6)
- **Subject Groups**: Content-based groupings (LIFE, SOL sequences)

**Key Fields**:
- `class_id`: Unique identifier for the class
- `class_name`: Human-readable class name
- `teacher_id`: Assigned teacher for the class
- `grade_level`: Educational level of the class
- `sequence_type`: Associated sequence type (LIFE, SOL, etc.)
- `max_students`: Maximum number of students allowed
- `current_enrollment`: Current number of enrolled students
- `is_generic`: Whether this is a shared access class

#### Sequence Groups
**Purpose**: Organize learning content into structured curriculum groups

**Structure**:
- **LIFE Sequences**: Comprehensive music learning sequences
- **SOL Sequences**: Solfege-focused learning sequences
- **Level Groups**: Difficulty-based groupings (Primary through Level 6)
- **Assignment Groups**: Specific assignment collections

**Key Fields**:
- `sequence_group_id`: Unique identifier for the sequence group
- `sequence_code`: Sequence identifier (LIFE, SOL, etc.)
- `group_number`: Numeric group identifier within sequence
- `level_title`: Educational level name
- `unit_title`: Specific unit or assignment title
- `prerequisite_groups`: Required completion of other groups
- `target_audience`: Intended user group for this sequence

### Group Relationships

#### Hierarchical Relationships
```
Subscriber Group (Organization)
├── Teacher-Admin Group (Optional)
│   ├── Teacher Group A
│   │   ├── Class Group 1 (Regular Students)
│   │   └── Class Group 2 (Generic Students)
│   └── Teacher Group B
│       ├── Class Group 3 (Regular Students)
│       └── Class Group 4 (Generic Students)
└── Direct Teacher Groups (Solo Plan)
    ├── Class Group 5 (Regular Students)
    └── Class Group 6 (Generic Students)
```

#### Content Relationships
```
Sequence Groups
├── LIFE Sequence Groups
│   ├── Primary Level 1A (Groups 004A-100A)
│   ├── Level 1 (Groups 105A-200A)
│   ├── Level 2 (Groups 205A-300A)
│   └── Level 3+ (Groups 305A+)
└── SOL Sequence Groups
    ├── Primary Level 1A (Groups 004A-100A)
    ├── Level 1 (Groups 105A-200A)
    └── Level 2+ (Groups 205A+)
```

## Behavior and Rules

### Group Creation and Management

#### Organizational Group Rules
- **Subscriber Groups**: Created during subscription signup, cannot be deleted
- **Teacher-Admin Groups**: Created by Subscriber-Admin in Ensemble plans only
- **Teacher Groups**: Created by Teacher-Admin or Subscriber-Admin
- **Student Groups**: Created by Teachers, Teacher-Admin, or Subscriber-Admin
- **Generic Groups**: Special student groups for shared access learning

#### Class Group Rules
- **Regular Classes**: Each student has unique account and tracking
- **Generic Classes**: Multiple students share same credentials
- **Enrollment Limits**: Classes respect subscription seat limits
- **Teacher Assignment**: Each class must have assigned teacher
- **Grade Level Alignment**: Classes align with appropriate sequence groups

#### Sequence Group Rules
- **Prerequisite Completion**: Students must complete prerequisite groups before advancing
- **Level Progression**: Students progress through levels in sequence
- **Assignment Distribution**: Teachers can assign specific sequence groups to classes
- **Progress Tracking**: System tracks completion of each group within sequences

### Group State Management

#### Group Lifecycle States
1. **Draft**: Group created but not yet active
2. **Active**: Group is operational and accepting members
3. **Suspended**: Group temporarily disabled (billing issues, etc.)
4. **Archived**: Group no longer active but data preserved
5. **Deleted**: Group and associated data removed (with retention policies)

#### State Transitions
- **Draft → Active**: When group is ready for use
- **Active → Suspended**: When billing issues or administrative action
- **Suspended → Active**: When issues resolved
- **Active → Archived**: When group no longer needed
- **Archived → Deleted**: After retention period expires

### Group Access Control

#### Permission Inheritance
- **Parent Groups**: Inherit permissions from parent organizational group
- **Role-based Access**: Access controlled by user role within group
- **Scope Limitations**: Users can only access groups within their scope
- **Delegation**: Teacher-Admin can delegate management to Teachers

#### Data Access Rules
- **Subscriber-Admin**: Access to all groups within their organization
- **Teacher-Admin**: Access to groups within their assigned scope
- **Teacher**: Access to their assigned classes and students
- **Student**: Access to their own group membership and progress

## UX Requirements

### Group Management Interface

#### Subscriber-Admin Dashboard
- **Organization Overview**: Visual hierarchy of all groups
- **Group Creation**: Tools for creating new groups and assigning roles
- **User Management**: Interface for adding/removing users from groups
- **Billing Integration**: Seat allocation and subscription management
- **Analytics**: Group-level performance and usage metrics

#### Teacher-Admin Dashboard
- **Scope Management**: View and manage groups within their scope
- **Teacher Assignment**: Assign teachers to specific groups
- **Class Organization**: Create and manage class groups
- **Progress Monitoring**: Track group performance and completion
- **Communication**: Tools for group-wide messaging

#### Teacher Dashboard
- **Class Management**: View and manage assigned classes
- **Student Organization**: Group students within classes
- **Assignment Distribution**: Assign content to specific groups
- **Progress Tracking**: Monitor group and individual progress
- **Communication**: Message students and parents

#### Student Interface
- **Group Membership**: View their group memberships and roles
- **Class Access**: Access assigned class content and activities
- **Progress View**: See their progress within group contexts
- **Collaboration**: Participate in group learning activities

### Group Visualization

#### Hierarchy Display
- **Tree Structure**: Visual representation of group hierarchy
- **Expandable Nodes**: Click to expand/collapse group levels
- **Role Indicators**: Visual indicators for different role types
- **Status Indicators**: Color coding for group status (active, suspended, etc.)

#### Member Management
- **Drag-and-Drop**: Move users between groups
- **Bulk Operations**: Select multiple users for group operations
- **Search and Filter**: Find users and groups quickly
- **Permission Preview**: Show what permissions changes will affect

## Telemetry

### Group Management Events
- **group_created**: When a new group is created
- **group_updated**: When group properties are modified
- **group_deleted**: When a group is deleted or archived
- **user_added_to_group**: When a user is added to a group
- **user_removed_from_group**: When a user is removed from a group
- **group_permissions_changed**: When group permissions are modified

### Group Activity Events
- **group_content_assigned**: When content is assigned to a group
- **group_progress_updated**: When group progress is updated
- **group_communication_sent**: When messages are sent to group members
- **group_analytics_generated**: When group reports are generated

### Analytics Properties
- **group_id**: Unique identifier for the group
- **group_type**: Type of group (organizational, class, sequence)
- **parent_group_id**: Parent group in hierarchy
- **subscription_plan**: Associated subscription plan
- **user_count**: Number of users in the group
- **activity_level**: Level of activity within the group
- **completion_rate**: Percentage of group members who completed assigned content

## Permissions

### Group Management Permissions

#### Subscriber-Admin Permissions
- **Full Group Management**: Create, read, update, delete all groups in organization
- **User Assignment**: Add/remove users from any group
- **Permission Management**: Modify group permissions and access controls
- **Billing Integration**: Manage group-related billing and seat allocation
- **Analytics Access**: View comprehensive group analytics and reports

#### Teacher-Admin Permissions
- **Scope Group Management**: Manage groups within their assigned scope
- **Teacher Assignment**: Assign teachers to specific groups
- **Class Management**: Create and manage class groups
- **User Management**: Add/remove users within their scope
- **Progress Monitoring**: View group progress and analytics

#### Teacher Permissions
- **Class Management**: Manage assigned class groups
- **Student Organization**: Group students within classes
- **Content Assignment**: Assign content to specific groups
- **Progress Tracking**: Monitor group and individual progress
- **Communication**: Send messages to group members

#### Student Permissions
- **Group Membership**: View their group memberships
- **Content Access**: Access content assigned to their groups
- **Progress View**: See their progress within group contexts
- **Communication**: Receive messages from teachers and admins

### Group Data Access

#### Organizational Data Access
- **Subscriber-Admin**: Access to all organizational data
- **Teacher-Admin**: Access to data within their scope
- **Teacher**: Access to data for their assigned groups
- **Student**: Access to their own data and group context

#### Content Access Control
- **Sequence Groups**: Access based on subscription plan and user level
- **Class Content**: Access based on group membership and assignments
- **Administrative Content**: Access based on role and permissions
- **Shared Content**: Access based on group type and settings

## Dependencies

### Internal Dependencies
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: Defines user roles and permissions within groups
- **[Learning Elements Overview](learning-elements-overview.md)**: Defines content that can be assigned to groups
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: Defines how content is organized into sequences
- **[Assignment Model](assignment-model.md)**: Defines how content is assigned to groups
- **[Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md)**: Defines subscription plans that determine group capabilities

### External Dependencies
- **User Management System**: For user account creation and authentication
- **Content Management System**: For storing and serving group-assigned content
- **Analytics Platform**: For tracking group activity and performance
- **Communication System**: For group messaging and notifications
- **Billing System**: For seat allocation and subscription management

## Open Questions

### Group Management
- How should we handle group merging when organizations consolidate?
- What is the optimal group size for different learning activities?
- How can we prevent group fragmentation while maintaining flexibility?

### Content Organization
- How should sequence groups be updated when curriculum changes?
- What is the best approach for handling group-specific content variations?
- How can we ensure content accessibility across different group types?

### Technical Implementation
- What is the optimal database structure for hierarchical group relationships?
- How should we handle group data migration during system updates?
- What caching strategy will optimize group-based content delivery?

### User Experience
- How can we improve group discovery and navigation for teachers?
- What accessibility features are needed for group management interfaces?
- How should we handle group transitions when users change roles?

### Analytics and Reporting
- What group-level metrics are most valuable for educational outcomes?
- How can we use group analytics to improve learning experiences?
- What privacy considerations apply to group-level data collection?
