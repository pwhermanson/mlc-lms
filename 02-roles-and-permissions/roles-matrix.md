# Roles Matrix

## Purpose

This document defines the role-based access control (RBAC) system for the MLC-LMS platform. It outlines the different user roles, their permissions, and how they interact within the organizational hierarchy to ensure proper access control and data security.

## Role Definitions

### Subscriber-Admin
**Primary Responsibility**: Owns the subscription and has full administrative control over the organization.

**Key Capabilities**:
- Manage subscription and billing
- Add/remove seats and users
- Access all organizational data
- Configure system settings
- Manage Teacher-Admin roles (in Ensemble plans)
- View comprehensive reports and analytics
- Handle billing disputes and refunds
- **Payment Management**: Add and edit payment information, schedule pause/restart of monthly subscriptions
- **Seat Management**: Add student seats (annual) or add/reduce seats (monthly subscriptions)
- **Subscription Control**: Terminate subscriptions with 13-month data retention
- **Communication**: Receive copies of all system communications via email
- **Grant Program Administration**: Manage Christine Hermanson Grant Program (future feature)

**Typical Users**: Studio owners, school administrators, program directors

**Subscription Context**: 
- **Solo Plan**: Subscriber-Admin has all Teacher capabilities and is the same person
- **Ensemble Plan**: Can delegate educational activities to Teacher-Admin while retaining billing control
- **Prelude Plan**: Free trial with limited capabilities, Teacher-Admin role is disabled

### Teacher-Admin (Ensemble Only)
**Primary Responsibility**: Manages teachers and classes within a larger organization.

**Key Capabilities**:
- Manage teacher accounts and permissions
- Organize students into classes and grades
- Assign teachers to specific classes
- View program-wide progress and analytics
- Access bulk operations for their scope
- Communicate with teachers and parents
- Request additional seats from Subscriber-Admin
- **Teacher Management**: Create Teachers by providing usernames and passwords
- **Student Management**: Create students, assign/move students to specific Teachers
- **Score Access**: View game scores history of all students in their scope
- **Data Management**: Delete individual student and scores history records
- **Game Access**: Play online games individually (scores not recorded for Teacher-Admin)
- **Communication**: Receive copies of system communications via email

**Typical Users**: Department heads, program coordinators, lead teachers

**Availability**: Only available in Ensemble plans; grayed out/disabled in Solo and Prelude plans

### Teacher
**Primary Responsibility**: Manages students, creates assignments, and tracks learning progress.

**Key Capabilities**:
- Create and manage assignments
- Track student progress and performance
- Communicate with students and parents
- Access teaching resources and sequences
- Generate progress reports
- Manage class settings and preferences
- View student learning analytics
- **Student Management**: Create students by providing usernames and passwords
- **Assignment Control**: Create specific game sequence assignments to students
- **Score Tracking**: View game scores history of their students
- **Game Access**: Play online games individually (scores not recorded for Teachers)
- **Dashboard Access**: Full Teacher/Admin dashboard access

**Typical Users**: Music teachers, private instructors, classroom teachers

**Creation Context**: 
- **Ensemble Plan**: Created by Teacher-Admin with unique username and password
- **Solo Plan**: Defaults from Subscriber-Admin (same person)

### Student
**Primary Responsibility**: Engages with learning content and completes assignments.

**Key Capabilities**:
- Play games and complete assignments
- Track personal progress and achievements
- Access learning resources
- Communicate with teachers
- View personal learning analytics
- Participate in challenges and leaderboards
- Manage personal settings and preferences
- **Game Access**: Play games either sequenced or individual via student dashboard
- **Score Tracking**: View individual game scores history
- **Challenge Participation**: Compete with other students worldwide in challenge games
- **Dashboard Access**: Access to student dashboard for all activities

**Typical Users**: Music students of all ages

**Creation Context**: Created by Teacher, Teacher-Admin, or Subscriber-Admin with unique username and password

### Generic Student
**Primary Responsibility**: Provides shared access for classroom or group learning environments.

**Key Capabilities**:
- **Shared Access**: Use any of several general usernames and passwords created by Teacher or Teacher-Admin
- **Game Access**: Play games either sequenced or individual via student dashboard
- **Score Recording**: Scores are recorded for each username (though irrelevant due to shared access)
- **No Individual Tracking**: No automated sequence assignment used

**Typical Users**: General Music classes, group learning environments

**Use Cases**: 
- **General Music Classes**: Elementary school music classes where individual tracking isn't necessary
- **Cost-Effective Solutions**: Schools can provide access to multiple students with fewer seat requirements
- **Group Learning**: Classroom activities using interactive whiteboards or projectors

**Note**: Scores are recorded per username but are not meaningful since multiple students use the same credentials

### System Admin
**Primary Responsibility**: Platform administration and technical support.

**Key Capabilities**:
- Global user management and support
- System configuration and maintenance
- Billing and subscription management
- Content management and updates
- Audit logging and compliance
- Feature flag management
- Technical support and troubleshooting

**Typical Users**: Platform administrators, technical support staff

## Subscription Plans and Role Availability

### Solo Plan
**Target**: Individual music teachers, home school parents, small studios (< 20 students)

**Role Structure**:
- **Subscriber-Admin**: Has all Teacher capabilities (same person)
- **Teacher**: Defaults from Subscriber-Admin (same person)
- **Student**: Regular students with individual tracking
- **Teacher-Admin**: Grayed out/disabled
- **Generic Student**: Available for group learning

**Characteristics**:
- Simple, flat hierarchy
- Direct relationship between admin and teachers
- Typically 1-50 seats
- Personal relationship between all users
- Subscriber-Admin can charge students $1-2/month to cover subscription costs

### Ensemble Plan
**Target**: Music schools, public schools, music teacher associations (20+ students)

**Role Structure**:
- **Subscriber-Admin**: Full capabilities except Student capabilities
- **Teacher-Admin**: Optional role for educational management
- **Teacher**: Created by Teacher-Admin or Subscriber-Admin
- **Student**: Regular students with individual tracking
- **Generic Student**: Available for group learning

**Characteristics**:
- Multi-level hierarchy with optional delegation
- Teacher-Admin can be same person as Subscriber-Admin or separate
- Typically 25+ seats
- Institutional oversight and reporting
- Flexible role delegation for business vs. educational management

### Prelude Plan
**Target**: Free trial users

**Role Structure**:
- **Subscriber-Admin**: Limited capabilities
- **Teacher**: Defaults from Subscriber-Admin
- **Student**: Regular students with individual tracking
- **Teacher-Admin**: Grayed out/disabled
- **Generic Student**: Available

**Characteristics**:
- Free trial with limited capabilities
- Teacher-Admin role is disabled
- Limited seat count for trial purposes

## Organizational Hierarchies

### Solo Studio Structure
```
Subscriber-Admin (Studio Owner) = Teacher
    └── Student (Music Student)
```

**Characteristics**:
- Simple, flat hierarchy
- Subscriber-Admin and Teacher are the same person
- Typically 1-50 seats
- Personal relationship between all users

### Ensemble School Structure (Without Teacher-Admin)
```
Subscriber-Admin (School Administrator) = Teacher-Admin
    └── Teacher (Music Teacher)
        └── Student (Music Student)
```

**Characteristics**:
- Subscriber-Admin handles both billing and educational management
- Direct management of teachers
- Typically 25+ seats

### Ensemble School Structure (With Teacher-Admin)
```
Subscriber-Admin (School Administrator)
    └── Teacher-Admin (Department Head)
        └── Teacher (Music Teacher)
            └── Student (Music Student)
```

**Characteristics**:
- Multi-level hierarchy with role delegation
- Subscriber-Admin handles billing, Teacher-Admin handles education
- Typically 25+ seats
- Institutional oversight and reporting

## Permission Matrix

### Content Management
| Role | View Content | Create Content | Edit Content | Delete Content | Assign Content |
|------|-------------|----------------|--------------|----------------|----------------|
| Subscriber-Admin | ✅ | ✅ | ✅ | ✅ | ✅ |
| Teacher-Admin | ✅ | ✅ | ✅ | ❌ | ✅ |
| Teacher | ✅ | ✅ | ✅ | ❌ | ✅ |
| Student | ✅ | ❌ | ❌ | ❌ | ❌ |
| System Admin | ✅ | ✅ | ✅ | ✅ | ✅ |

### User Management
| Role | View Users | Create Users | Edit Users | Delete Users | Assign Roles |
|------|------------|--------------|------------|--------------|--------------|
| Subscriber-Admin | ✅ (Org) | ✅ (Org) | ✅ (Org) | ✅ (Org) | ✅ (Org) |
| Teacher-Admin | ✅ (Scope) | ✅ (Scope) | ✅ (Scope) | ❌ | ❌ |
| Teacher | ✅ (Students) | ✅ (Students) | ✅ (Students) | ❌ | ❌ |
| Student | ❌ | ❌ | ❌ | ❌ | ❌ |
| System Admin | ✅ (Global) | ✅ (Global) | ✅ (Global) | ✅ (Global) | ✅ (Global) |

### Billing and Subscription
| Role | View Billing | Modify Billing | Access Reports | Manage Seats | Handle Payments |
|------|--------------|----------------|----------------|--------------|-----------------|
| Subscriber-Admin | ✅ | ✅ | ✅ | ✅ | ✅ |
| Teacher-Admin | ✅ (Read-only) | ❌ | ✅ (Scope) | ❌ | ❌ |
| Teacher | ❌ | ❌ | ❌ | ❌ | ❌ |
| Student | ❌ | ❌ | ❌ | ❌ | ❌ |
| System Admin | ✅ | ✅ | ✅ | ✅ | ✅ |

### Reporting and Analytics
| Role | View Reports | Export Data | Access Analytics | View Audit Logs | Generate Statements |
|------|-------------|-------------|------------------|-----------------|-------------------|
| Subscriber-Admin | ✅ (Org) | ✅ (Org) | ✅ (Org) | ✅ (Org) | ✅ |
| Teacher-Admin | ✅ (Scope) | ✅ (Scope) | ✅ (Scope) | ✅ (Scope) | ❌ |
| Teacher | ✅ (Students) | ✅ (Students) | ✅ (Students) | ❌ | ❌ |
| Student | ✅ (Personal) | ✅ (Personal) | ✅ (Personal) | ❌ | ❌ |
| System Admin | ✅ (Global) | ✅ (Global) | ✅ (Global) | ✅ (Global) | ✅ |

### System Administration
| Role | Feature Flags | System Config | Content Updates | User Support | Audit Access |
|------|---------------|---------------|-----------------|--------------|--------------|
| Subscriber-Admin | ❌ | ❌ | ❌ | ❌ | ✅ (Org) |
| Teacher-Admin | ❌ | ❌ | ❌ | ❌ | ✅ (Scope) |
| Teacher | ❌ | ❌ | ❌ | ❌ | ❌ |
| Student | ❌ | ❌ | ❌ | ❌ | ❌ |
| System Admin | ✅ | ✅ | ✅ | ✅ | ✅ (Global) |

## Data Access Scopes

### Subscriber-Admin Data Access
- **Full Organization**: All users, data, and settings within their subscription
- **Billing Data**: Complete billing history and payment information
- **Usage Analytics**: Comprehensive usage and performance metrics
- **Audit Logs**: All actions within their organization

### Teacher-Admin Data Access
- **Assigned Scope**: Teachers and students under their management
- **Class Data**: All data related to their assigned classes
- **Progress Reports**: Analytics for their scope of responsibility
- **Limited Billing**: Read-only access to relevant billing information

### Teacher Data Access
- **Student Data**: Only students assigned to them
- **Assignment Data**: Assignments they create and manage
- **Progress Data**: Learning progress for their students
- **Communication**: Messages and interactions with their students

### Student Data Access
- **Personal Data**: Only their own learning data and progress
- **Assignment Data**: Assignments assigned to them
- **Achievement Data**: Their own badges, points, and achievements
- **Communication**: Messages from teachers and system

### System Admin Data Access
- **Global Data**: Access to all data across all organizations
- **System Data**: Platform configuration and performance data
- **Audit Data**: Complete audit trail of all system actions
- **Support Data**: User support requests and resolution data

## Security Considerations

### Data Isolation
- **Organization Boundaries**: Strict data isolation between organizations
- **Role-based Filtering**: Data access filtered by user role and scope
- **Audit Logging**: All data access logged for security monitoring
- **Encryption**: Sensitive data encrypted at rest and in transit

### Access Control
- **Authentication**: Multi-factor authentication for admin roles
- **Session Management**: Secure session handling with timeouts
- **Impersonation**: Safe user impersonation for support (System Admin only)
- **Permission Validation**: Real-time permission checking for all actions

### Compliance
- **Data Privacy**: Compliance with privacy regulations (COPPA, FERPA)
- **Audit Requirements**: Comprehensive audit trails for compliance
- **Data Retention**: Proper data retention and deletion policies
- **Access Reviews**: Regular review of user permissions and access

## Role Transitions

### Adding New Roles
- **Subscriber-Admin**: Can create Teacher-Admin and Teacher roles
- **Teacher-Admin**: Can create Teacher roles within their scope
- **Teacher**: Can create Student accounts
- **System Admin**: Can create any role type

### Role Changes
- **Upgrade**: Move users to higher privilege roles
- **Downgrade**: Reduce user privileges (with data migration)
- **Scope Changes**: Modify user access scope
- **Deactivation**: Temporarily disable user access

### Data Migration
- **Role Changes**: Migrate data access when roles change
- **Scope Changes**: Adjust data access when scope changes
- **Deactivation**: Preserve data during user deactivation
- **Reactivation**: Restore access when users are reactivated

## Game Access and Scoring Rules

### Game Playing Capabilities
All roles except System Admin can play games, but with different scoring behaviors:

| Role | Game Access | Score Recording | Purpose |
|------|-------------|-----------------|---------|
| **Student** | ✅ Full access | ✅ Scores recorded | Learning and assessment |
| **Generic Student** | ✅ Full access | ✅ Scores recorded (shared) | Group learning |
| **Teacher** | ✅ Individual games | ❌ No scores recorded | Testing and demonstration |
| **Teacher-Admin** | ✅ Individual games | ❌ No scores recorded | Testing and demonstration |
| **Subscriber-Admin** | ✅ Individual games | ❌ No scores recorded | Testing and demonstration |
| **System Admin** | ❌ No game access | N/A | Platform administration only |

### Game Assignment Capabilities
- **Teachers**: Can assign specific game sequences to their students
- **Teacher-Admin**: Can assign sequences to all students in their scope
- **Subscriber-Admin**: Can assign sequences to any student in the organization
- **Students**: Can play assigned sequences or individual games
- **Generic Students**: Can play assigned sequences or individual games (no individual tracking)

### Challenge Games
- **Students**: Can participate in worldwide challenge games and compete with other students
- **Generic Students**: Can participate in challenge games (scores recorded per username)
- **Teachers/Admins**: Cannot participate in challenge games (no score recording)

For detailed game mechanics and scoring systems, see [Game Model](../08-games-registry-and-authoring/game-model.md) and [Assignment Play](../03-student-experience/assignment-play.md).

## Future Considerations

### Advanced Permissions
- **Granular Permissions**: More specific permission controls
- **Custom Roles**: Allow creation of custom role combinations
- **Temporary Permissions**: Time-limited elevated permissions
- **Delegation**: Allow temporary delegation of permissions

### Integration Permissions
- **API Access**: Role-based API access controls
- **Third-party Integration**: Permissions for external tool access
- **Data Export**: Controlled data export permissions
- **System Integration**: Permissions for system-to-system access

## Related Documentation

For comprehensive understanding of the role system, see:
- [Personas](../00-foundations/personas.md) - Detailed user personas and their needs
- [Permissions Table](./permissions-table.md) - Detailed permission specifications
- [Session and Auth States](./session-and-auth-states.md) - Authentication and session management
- [Impersonation and Support Access](./impersonation-and-support-access.md) - Support access controls
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) - Subscription plan details
