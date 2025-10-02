# Seat Management

## Purpose

This specification defines the seat management system for the MLC-LMS platform, enabling subscription administrators to manage student seats within their subscription plans. The seat management system provides the core functionality for adding, removing, reassigning, and monitoring seat usage across different subscription tiers.

The system supports the platform's tiered pricing model where seats are the primary unit of billing, with different pricing structures for Solo (up to 19 seats) and Ensemble (unlimited seats) plans. Effective seat management is critical for cost optimization, user access control, and billing accuracy.

## Scope

### Included
- Seat allocation and deallocation operations
- Seat reassignment between users
- Seat usage monitoring and reporting
- Seat limit enforcement per subscription plan
- Prorated billing calculations for seat changes
- Bulk seat operations for large institutions
- Seat history and audit trails
- Integration with user management and billing systems

### Excluded
- User account creation and management (see [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md))
- Billing calculations and payment processing (see [billing-plans-and-seats.md](billing-plans-and-seats.md))
- Subscription plan changes (see [plans-and-pricing.md](plans-and-pricing.md))
- User authentication and authorization (see [session-and-auth-states.md](../02-roles-and-permissions/session-and-auth-states.md))

## Data Model

### Core Entities

#### Seat
- `seat_id` (UUID, Primary Key)
- `subscription_id` (UUID, Foreign Key)
- `user_id` (UUID, Foreign Key, nullable)
- `status` (enum: active, inactive, suspended, pending)
- `created_at` (timestamp)
- `assigned_at` (timestamp, nullable)
- `last_used_at` (timestamp, nullable)
- `seat_type` (enum: regular, generic, trial)
- `metadata` (JSON, for additional seat properties)

#### Seat Assignment History
- `assignment_id` (UUID, Primary Key)
- `seat_id` (UUID, Foreign Key)
- `previous_user_id` (UUID, Foreign Key, nullable)
- `new_user_id` (UUID, Foreign Key, nullable)
- `assigned_by` (UUID, Foreign Key)
- `assignment_type` (enum: assign, unassign, reassign, bulk_assign)
- `created_at` (timestamp)
- `notes` (text, nullable)

#### Seat Usage Log
- `usage_id` (UUID, Primary Key)
- `seat_id` (UUID, Foreign Key)
- `user_id` (UUID, Foreign Key)
- `action` (enum: login, game_play, assignment_access, logout)
- `timestamp` (timestamp)
- `session_duration` (integer, seconds, nullable)
- `metadata` (JSON, for additional usage data)

### Relationships
- One subscription has many seats
- One seat belongs to one subscription
- One seat can be assigned to one user (nullable for unassigned seats)
- One user can have multiple seats (across different subscriptions)
- One seat has many assignment history records
- One seat has many usage log entries

## Behavior and Rules

### Seat Allocation Rules

#### Solo Plan (Studio Plan)
- **Maximum Seats**: 19 seats total
- **Seat Addition**: Can add seats up to the 19-seat limit
- **Seat Removal**: Can remove seats at any time
- **Billing**: Changes prorated to end of current billing period
- **Upgrade Requirement**: Must upgrade to Ensemble plan to exceed 19 seats

#### Ensemble Plan (School Plan)
- **Maximum Seats**: No upper limit
- **Seat Addition**: Can add unlimited seats
- **Seat Removal**: Can remove seats at any time
- **Billing**: Changes prorated to end of current billing period
- **Volume Pricing**: Per-seat cost decreases with higher seat counts

#### Prelude Plan (Free Trial)
- **Maximum Seats**: 19 seats total
- **Duration**: 14 days maximum
- **Seat Management**: Limited to basic assignment operations
- **Conversion**: Must convert to Solo or Ensemble before expiration

### Seat Assignment Rules

#### Regular Seats
- **Assignment**: One seat per user
- **Uniqueness**: Each user can only be assigned one seat per subscription
- **Reassignment**: Seats can be reassigned between users
- **History**: All assignment changes are logged

#### Generic Seats
- **Purpose**: For general music classes where individual tracking isn't needed
- **Assignment**: Multiple students can use the same seat credentials
- **Scoring**: Scores are recorded but not individually tracked
- **Usage**: Primarily for classroom environments

#### Trial Seats
- **Purpose**: For evaluation and testing
- **Duration**: Limited time access
- **Features**: Full platform access with usage restrictions
- **Conversion**: Must convert to regular seat for continued access

### Seat Status Transitions

```
[Unassigned] → [Active] → [Inactive] → [Suspended]
     ↑              ↓           ↓           ↓
     └──────────────┴───────────┴───────────┘
                    [Reassigned]
```

#### Status Definitions
- **Unassigned**: Seat exists but not assigned to any user
- **Active**: Seat assigned to user and available for use
- **Inactive**: Seat temporarily disabled (user account suspended)
- **Suspended**: Seat permanently disabled (subscription issues)
- **Pending**: Seat in process of being assigned

### Validation Rules

#### Seat Limits
- Solo plan cannot exceed 19 seats
- Prelude plan cannot exceed 19 seats
- Ensemble plan has no upper limit
- Seat count cannot be negative

#### Assignment Validation
- User must exist and be active
- User cannot be assigned multiple seats in same subscription
- Seat must be available for assignment
- Assignment must be performed by authorized user

#### Billing Validation
- Seat changes must be within subscription limits
- Proration calculations must be accurate
- Billing cycle changes must be valid
- Payment method must be current and valid

## UX Requirements

### Seat Management Dashboard

#### Overview Section
- **Current Seat Count**: Display total seats and usage
- **Available Seats**: Show unassigned seats
- **Usage Statistics**: Charts showing seat utilization
- **Quick Actions**: Add seats, assign seats, view reports

#### Seat List View
- **Table Layout**: Sortable columns for seat details
- **Filtering**: By status, assignment, creation date
- **Search**: Find specific seats or users
- **Bulk Actions**: Select multiple seats for operations
- **Status Indicators**: Visual indicators for seat status

#### Assignment Interface
- **User Selection**: Dropdown or search for available users
- **Seat Selection**: Choose from available seats
- **Assignment Confirmation**: Review before confirming
- **Bulk Assignment**: Assign multiple seats at once
- **Import/Export**: CSV import for bulk operations

### Key States and Interactions

#### Adding Seats
1. **Seat Count Input**: Number picker with validation
2. **Cost Preview**: Show prorated cost calculation
3. **Confirmation**: Review total cost and billing impact
4. **Processing**: Loading state during seat creation
5. **Success**: Confirmation with new seat details

#### Assigning Seats
1. **Seat Selection**: Choose from available seats
2. **User Search**: Find and select target user
3. **Assignment Details**: Review assignment information
4. **Confirmation**: Final confirmation before assignment
5. **Notification**: Notify user of seat assignment

#### Reassigning Seats
1. **Current Assignment**: Show current user assignment
2. **New User Selection**: Choose new user
3. **Transfer Options**: Handle existing data and progress
4. **Confirmation**: Review reassignment details
5. **Completion**: Update both user accounts

### Accessibility Requirements

#### Keyboard Navigation
- All seat management functions accessible via keyboard
- Tab order follows logical workflow
- Focus indicators clearly visible
- Keyboard shortcuts for common actions

#### Screen Reader Support
- Semantic HTML structure
- ARIA labels for complex interactions
- Status announcements for dynamic content
- Alternative text for visual indicators

#### Visual Design
- High contrast ratios for text and backgrounds
- Color coding supplemented with text labels
- Scalable interface elements
- Clear visual hierarchy

## Telemetry

### Seat Management Events

#### Seat Operations
- `seat_created`
  - `seat_id`, `subscription_id`, `seat_type`, `created_by`
- `seat_assigned`
  - `seat_id`, `user_id`, `assigned_by`, `assignment_type`
- `seat_unassigned`
  - `seat_id`, `previous_user_id`, `unassigned_by`, `reason`
- `seat_reassigned`
  - `seat_id`, `previous_user_id`, `new_user_id`, `reassigned_by`
- `seat_deleted`
  - `seat_id`, `subscription_id`, `deleted_by`, `reason`

#### Usage Tracking
- `seat_accessed`
  - `seat_id`, `user_id`, `access_type`, `timestamp`
- `seat_usage_duration`
  - `seat_id`, `user_id`, `session_duration`, `activities_count`
- `seat_limit_warning`
  - `subscription_id`, `current_seats`, `limit_type`, `threshold`

#### Bulk Operations
- `bulk_seat_operation_started`
  - `operation_type`, `seat_count`, `initiated_by`
- `bulk_seat_operation_completed`
  - `operation_type`, `successful_count`, `failed_count`, `duration`
- `bulk_seat_operation_failed`
  - `operation_type`, `error_type`, `failed_seats[]`

### Performance Metrics
- Seat assignment response time
- Bulk operation completion time
- Seat usage query performance
- Dashboard load time
- Search and filter performance

## Permissions

### Subscriber Administrators
- **Full Access**: All seat management operations
- **Seat Limits**: Can add/remove seats within plan limits
- **User Assignment**: Can assign/unassign any seat
- **Billing Changes**: Can modify seat counts affecting billing
- **Reports**: Access to all seat usage reports
- **Bulk Operations**: Can perform bulk seat operations

### Teacher-Admin (Ensemble Plans)
- **Seat Assignment**: Can assign seats to teachers and students
- **User Management**: Can create and manage user accounts
- **Reports**: Access to seat usage reports for their organization
- **Bulk Operations**: Can perform bulk assignments
- **Restrictions**: Cannot modify subscription or billing settings

### Teachers
- **Student Assignment**: Can assign seats to their students
- **Student Management**: Can create student accounts
- **Reports**: Access to reports for their assigned students
- **Restrictions**: Cannot modify seat counts or billing

### System Administrators
- **Override Access**: Can override all seat limits and rules
- **Emergency Operations**: Can perform emergency seat operations
- **Audit Access**: Full access to all seat management logs
- **Billing Override**: Can modify billing calculations
- **Support Operations**: Can assist with seat management issues

### Read-Only Access
- **Seat Status**: Can view current seat assignments
- **Usage Reports**: Can view seat usage statistics
- **History**: Can view seat assignment history
- **Restrictions**: Cannot modify any seat settings

## Dependencies

### Internal Dependencies
- [plans-and-pricing.md](plans-and-pricing.md) - Pricing structure and plan limits
- [billing-plans-and-seats.md](billing-plans-and-seats.md) - Billing calculations
- [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) - User roles and permissions
- [session-and-auth-states.md](../02-roles-and-permissions/session-and-auth-states.md) - User authentication
- [member-management.md](../05-admin-subscriber-experience/member-management.md) - User account management

### External Dependencies
- **Payment Processor**: For billing seat changes
- **Email Service**: For seat assignment notifications
- **Audit System**: For logging seat management operations
- **Analytics Platform**: For usage tracking and reporting
- **Database**: For seat data persistence and queries

### Integration Points
- **User Management API**: For user account operations
- **Billing API**: For cost calculations and payment processing
- **Notification Service**: For user communications
- **Reporting Engine**: For seat usage analytics
- **Audit Logger**: For compliance and security tracking

## Open Questions

1. **Seat Transfer Between Subscriptions**: Can seats be transferred between different subscriptions, and if so, what are the billing implications?

2. **Concurrent Seat Usage**: How should the system handle cases where a user attempts to use multiple seats simultaneously?

3. **Seat Reservation System**: Should there be a mechanism to reserve seats for future assignment without immediate billing?

4. **Seat Sharing Policies**: Are there scenarios where seats can be shared between users, and what are the technical and billing implications?

5. **Emergency Seat Allocation**: What procedures should be in place for emergency seat allocation during system outages or high-demand periods?

6. **Seat Migration Tools**: What tools are needed to migrate seat assignments during subscription upgrades or organizational changes?

7. **Seat Usage Analytics**: What level of detailed analytics should be provided for seat usage patterns and optimization recommendations?

8. **Seat Expiration Policies**: Should there be automatic expiration policies for unused seats, and how should this integrate with billing?

9. **Multi-Tenant Seat Management**: How should seat management work for organizations with multiple subscription accounts or complex hierarchies?

10. **Seat Performance Monitoring**: What monitoring and alerting should be in place for seat management system performance and availability?
