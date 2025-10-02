# Permissions Table

This document provides detailed permission specifications for each user role in the MLC-LMS platform. It defines granular access controls for all system functions, ensuring proper security and data isolation while enabling effective role-based access control (RBAC).

## Purpose

This specification enables:
- **Security**: Proper access control and data protection across all user roles
- **Compliance**: Meeting educational data privacy requirements (COPPA, FERPA)
- **Scalability**: Supporting different organizational structures and subscription plans
- **Flexibility**: Allowing role delegation and hierarchical management
- **Audit**: Providing clear permission boundaries for compliance and security monitoring

## Scope

**Included:**
- Detailed permission matrices for all user roles
- Functional area permissions (content, users, billing, reporting)
- Data access scopes and boundaries
- Role-specific capabilities and restrictions
- Subscription plan variations
- Security and compliance considerations

**Excluded:**
- Authentication mechanisms (see [Session and Auth States](./session-and-auth-states.md))
- System administration details (see [System Admin](../06-system-admin/) folder)
- Billing implementation specifics (see [Payments and Subscriptions](../14-payments-and-subscriptions/))

## Data Model

### Key Entities
- **User**: Individual user account with assigned role
- **Organization**: Subscription entity containing users and data
- **Role**: Permission set defining user capabilities
- **Permission**: Granular access control for specific functions
- **Scope**: Data access boundary for role-based filtering

### Relationships
- User belongs to Organization
- User has one Role
- Role has many Permissions
- Permission applies to specific functional areas
- Scope defines data access boundaries

## Behavior and Rules

### Permission Inheritance
- **Subscriber-Admin**: Inherits all permissions within their organization
- **Teacher-Admin**: Inherits Teacher permissions plus additional scope
- **Teacher**: Has base teaching permissions for their students
- **Student**: Has limited permissions for personal data only

### Data Access Rules
- **Organization Isolation**: Users can only access data within their organization
- **Role-based Filtering**: Data access filtered by user role and scope
- **Hierarchical Access**: Higher roles can access data from lower roles in their scope
- **Audit Logging**: All permission checks and data access logged

### State Transitions
- **Role Changes**: Permissions updated immediately upon role assignment
- **Scope Changes**: Data access adjusted when user scope changes
- **Subscription Changes**: Role availability changes with plan upgrades/downgrades
- **Deactivation**: All permissions revoked during user deactivation

## UX Requirements

### Permission Indicators
- **Visual Cues**: Clear indicators for available/unavailable actions
- **Contextual Help**: Tooltips explaining permission requirements
- **Progressive Disclosure**: Show relevant options based on user permissions
- **Error Messages**: Clear feedback when permissions are insufficient

### Accessibility
- **Screen Reader Support**: Proper ARIA labels for permission states
- **Keyboard Navigation**: Accessible navigation for all permission levels
- **High Contrast**: Clear visual distinction between permission states
- **Focus Management**: Proper focus handling for permission-dependent UI

## Telemetry

### Permission Events
- **Permission Check**: `permission.checked` - When system validates user permission
- **Access Denied**: `permission.denied` - When user attempts unauthorized action
- **Role Change**: `role.changed` - When user role is modified
- **Scope Change**: `scope.changed` - When user access scope is updated

### Event Properties
- `user_id`: User attempting action
- `role`: Current user role
- `permission`: Specific permission being checked
- `resource`: Resource being accessed
- `organization_id`: Organization context
- `success`: Whether permission was granted
- `reason`: Reason for denial if applicable

## Detailed Permission Matrices

### Content Management Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **View Content** | ✅ (All Org) | ✅ (All Org) | ✅ (All Org) | ✅ (Assigned) | ✅ (Global) |
| **Create Content** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Edit Content** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Delete Content** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **Publish Content** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Assign Content** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **View Drafts** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Content Analytics** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |

### User Management Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **View Users** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ❌ | ✅ (Global) |
| **Create Users** | ✅ (All Roles) | ✅ (Teachers, Students) | ✅ (Students) | ❌ | ✅ (All Roles) |
| **Edit Users** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Self) | ✅ (Global) |
| **Delete Users** | ✅ (All Org) | ❌ | ❌ | ❌ | ✅ (Global) |
| **Assign Roles** | ✅ (All Org) | ❌ | ❌ | ❌ | ✅ (Global) |
| **Manage Permissions** | ✅ (Org) | ❌ | ❌ | ❌ | ✅ (Global) |
| **View User Analytics** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **Bulk Operations** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ❌ | ✅ (Global) |

### Assignment Management Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **View Assignments** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Assigned) | ✅ (Global) |
| **Create Assignments** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Edit Assignments** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Delete Assignments** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Assign to Students** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ❌ | ✅ (Global) |
| **View Progress** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **Grade Assignments** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **Bulk Assignment** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ❌ | ✅ (Global) |

### Game Access Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **Play Games** | ✅ (Individual) | ✅ (Individual) | ✅ (Individual) | ✅ (Full) | ❌ |
| **Score Recording** | ❌ | ❌ | ❌ | ✅ | N/A |
| **Challenge Games** | ❌ | ❌ | ❌ | ✅ | ❌ |
| **View Game Library** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Game Analytics** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **Game Settings** | ✅ | ✅ | ✅ | ✅ (Personal) | ✅ |

### Billing and Subscription Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **View Billing** | ✅ | ✅ (Read-only) | ❌ | ❌ | ✅ |
| **Modify Billing** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **Manage Seats** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **Handle Payments** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **View Invoices** | ✅ | ✅ (Read-only) | ❌ | ❌ | ✅ |
| **Subscription Control** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **Promo Codes** | ✅ | ❌ | ❌ | ❌ | ✅ |
| **Billing Reports** | ✅ | ❌ | ❌ | ❌ | ✅ |

### Reporting and Analytics Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **View Reports** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **Export Data** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **Generate Reports** | ✅ | ✅ | ✅ | ❌ | ✅ |
| **View Analytics** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **Audit Logs** | ✅ (Org) | ✅ (Scope) | ❌ | ❌ | ✅ (Global) |
| **System Metrics** | ❌ | ❌ | ❌ | ❌ | ✅ |

### Communication Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **Send Messages** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Teachers) | ✅ (Global) |
| **View Messages** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **System Announcements** | ✅ | ✅ | ❌ | ❌ | ✅ |
| **Email Notifications** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |
| **Push Notifications** | ✅ (All Org) | ✅ (Scope) | ✅ (Students) | ✅ (Personal) | ✅ (Global) |

### System Administration Permissions

| Permission | Subscriber-Admin | Teacher-Admin | Teacher | Student | System Admin |
|------------|------------------|---------------|---------|---------|--------------|
| **Feature Flags** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **System Config** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Content Updates** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **User Support** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **Audit Access** | ✅ (Org) | ✅ (Scope) | ❌ | ❌ | ✅ (Global) |
| **Impersonation** | ❌ | ❌ | ❌ | ❌ | ✅ |
| **System Maintenance** | ❌ | ❌ | ❌ | ❌ | ✅ |

## Subscription Plan Variations

### Solo Plan Permissions
- **Subscriber-Admin** = **Teacher**: All teacher permissions included
- **Teacher-Admin**: Disabled/grayed out
- **Generic Student**: Available for group learning
- **Seat Management**: Subscriber-Admin can charge students $1-2/month

### Ensemble Plan Permissions
- **Teacher-Admin**: Full educational management capabilities
- **Role Delegation**: Subscriber-Admin can delegate to Teacher-Admin
- **Bulk Operations**: Enhanced bulk management capabilities
- **Institutional Reporting**: Advanced reporting and analytics

### Prelude Plan Permissions
- **Limited Capabilities**: Reduced feature set for trial
- **Teacher-Admin**: Disabled/grayed out
- **Trial Restrictions**: Limited seat count and functionality
- **Upgrade Path**: Clear path to full subscription

## Data Access Scopes

### Organization-Level Access
- **Subscriber-Admin**: Complete organization data access
- **System Admin**: Global access across all organizations
- **Data Isolation**: Strict boundaries between organizations

### Scope-Based Access
- **Teacher-Admin**: Access to assigned teachers and their students
- **Teacher**: Access to assigned students only
- **Student**: Access to personal data only

### Functional Area Access
- **Content**: Based on creation and assignment permissions
- **Users**: Based on role hierarchy and scope
- **Billing**: Restricted to Subscriber-Admin and System Admin
- **Analytics**: Scoped to user's data access level

## Security Considerations

### Permission Validation
- **Real-time Checking**: All actions validated against current permissions
- **Session-based**: Permissions cached in user session for performance
- **Audit Logging**: All permission checks logged for security monitoring
- **Error Handling**: Graceful handling of permission denials

### Data Protection
- **Encryption**: Sensitive data encrypted at rest and in transit
- **Access Logging**: All data access logged for compliance
- **Data Retention**: Proper data retention and deletion policies
- **Privacy Controls**: Compliance with COPPA and FERPA requirements

### Role Security
- **Multi-factor Authentication**: Required for admin roles
- **Session Management**: Secure session handling with timeouts
- **Impersonation Controls**: Safe user impersonation for support
- **Permission Reviews**: Regular review of user permissions

## Dependencies

### Internal Dependencies
- [Roles Matrix](./roles-matrix.md) - Role definitions and hierarchies
- [Session and Auth States](./session-and-auth-states.md) - Authentication mechanisms
- [Impersonation and Support Access](./impersonation-and-support-access.md) - Support access controls
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) - Subscription plan details

### External Dependencies
- **Authentication System**: User authentication and session management
- **Database Layer**: Data access and permission validation
- **Audit System**: Permission and access logging
- **Notification System**: Permission-based communication controls

## Open Questions

### Permission Granularity
- Should we implement more granular permissions within functional areas?
- How detailed should content-level permissions be?
- Should we support custom permission sets for specific use cases?

### Role Flexibility
- Should we allow custom role creation within organizations?
- How should temporary permission elevation work?
- Should we support role delegation for specific time periods?

### Integration Permissions
- What permissions are needed for API access?
- How should third-party integration permissions work?
- What level of data export permissions should be available?

### Future Considerations
- How will AI/ML features affect permission requirements?
- What new permissions will be needed for advanced features?
- How should permission inheritance work with new role types?
