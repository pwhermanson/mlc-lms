# Session and Auth States

This document defines authentication and session management states for the MLC-LMS platform, ensuring secure user access and proper role-based authorization across all subscription plans and user types.

## Purpose

This specification enables:
- **Secure Authentication**: Multi-layered authentication system supporting different user types and subscription plans
- **Session Management**: Robust session handling with appropriate timeouts and security controls
- **Role-based Access**: Seamless integration with the role hierarchy system for proper authorization
- **Compliance**: Meeting educational data privacy requirements (COPPA, FERPA) and security standards
- **User Experience**: Smooth authentication flows that work across different organizational structures
- **Support Operations**: Safe user impersonation and support access for technical assistance

## Scope

**Included:**
- Authentication mechanisms for all user roles (Subscriber-Admin, Teacher-Admin, Teacher, Student, Generic Student)
- Session state management and lifecycle
- Multi-factor authentication for admin roles
- Password policies and security requirements
- Session timeout and refresh mechanisms
- Impersonation and support access controls
- Subscription plan-specific authentication behaviors
- Security and compliance considerations

**Excluded:**
- Detailed permission specifications (see [Permissions Table](./permissions-table.md))
- Role definitions and hierarchies (see [Roles Matrix](./roles-matrix.md))
- Billing and subscription management (see [Payments and Subscriptions](../14-payments-and-subscriptions/))
- System administration details (see [System Admin](../06-system-admin/) folder)

## Data Model

### Key Entities

#### User Authentication
- **User ID**: Unique identifier for each user account
- **Username**: Unique login identifier (email or custom username)
- **Password Hash**: Securely hashed password using bcrypt or similar
- **Role**: User role (Subscriber-Admin, Teacher-Admin, Teacher, Student, Generic Student)
- **Organization ID**: Subscription/organization the user belongs to
- **MFA Enabled**: Boolean flag for multi-factor authentication status
- **MFA Secret**: Encrypted MFA secret for TOTP apps
- **Last Login**: Timestamp of last successful authentication
- **Failed Login Attempts**: Counter for failed login attempts
- **Account Locked**: Boolean flag for account lockout status
- **Password Reset Token**: Temporary token for password reset
- **Password Reset Expiry**: Expiration timestamp for reset token

#### Session Management
- **Session ID**: Unique session identifier
- **User ID**: Associated user account
- **Organization ID**: User's organization context
- **Role**: User's role at session creation
- **Created At**: Session creation timestamp
- **Last Activity**: Last activity timestamp
- **Expires At**: Session expiration timestamp
- **IP Address**: Client IP address
- **User Agent**: Client browser/device information
- **Is Active**: Boolean flag for session status
- **Impersonation Data**: Support impersonation context (if applicable)

#### Authentication Events
- **Event ID**: Unique event identifier
- **User ID**: User involved in the event
- **Event Type**: Type of authentication event
- **Timestamp**: When the event occurred
- **IP Address**: Client IP address
- **Success**: Whether the event was successful
- **Details**: Additional event context

### Relationships
- User belongs to Organization
- User has many Sessions (one active at a time)
- User has many Authentication Events
- Session belongs to User
- Session can have Impersonation Data (for support access)

## Behavior and Rules

### Authentication Flow

#### Standard Login Process
1. **Username/Password Entry**: User enters credentials
2. **Credential Validation**: System validates username and password hash
3. **Account Status Check**: Verify account is not locked or disabled
4. **MFA Challenge**: If MFA enabled, prompt for second factor
5. **Session Creation**: Create new session with appropriate timeout
6. **Role Assignment**: Assign user role and permissions
7. **Audit Logging**: Log successful authentication event

#### Multi-Factor Authentication (MFA)
- **Required For**: Subscriber-Admin, Teacher-Admin, System Admin
- **Optional For**: Teachers (can be enabled by Subscriber-Admin)
- **Not Available For**: Students and Generic Students
- **Methods**: TOTP apps (Google Authenticator, Authy), SMS backup
- **Setup Process**: QR code generation for TOTP app setup
- **Recovery**: Backup codes provided during MFA setup

#### Password Requirements
- **Minimum Length**: 8 characters
- **Complexity**: Must include uppercase, lowercase, number, and special character
- **History**: Cannot reuse last 5 passwords
- **Expiration**: 90 days for admin roles, 180 days for teachers, no expiration for students
- **Reset Process**: Email-based password reset with 24-hour token expiry

### Session Management

#### Session Lifecycle
- **Creation**: New session created on successful authentication
- **Activity Tracking**: Session extended on user activity
- **Timeout Handling**: Automatic logout after inactivity period
- **Refresh**: Session tokens refreshed on activity
- **Termination**: Explicit logout or session expiry

#### Session Timeouts
- **Subscriber-Admin**: 8 hours of inactivity
- **Teacher-Admin**: 8 hours of inactivity
- **Teacher**: 4 hours of inactivity
- **Student**: 2 hours of inactivity
- **Generic Student**: 1 hour of inactivity
- **System Admin**: 1 hour of inactivity (enhanced security)

#### Session Security
- **Token Rotation**: Session tokens rotated on refresh
- **IP Validation**: Optional IP address validation for admin roles
- **Device Tracking**: Track and manage multiple device sessions
- **Concurrent Sessions**: Limit concurrent sessions per user role

### State Transitions

#### Authentication States
- **Unauthenticated**: No valid session, must log in
- **Authenticating**: In process of authentication (MFA challenge)
- **Authenticated**: Valid session, full access based on role
- **Session Expired**: Session timeout, must re-authenticate
- **Account Locked**: Too many failed attempts, temporary lockout
- **Account Disabled**: Account deactivated by admin

#### Session States
- **Active**: Valid session with recent activity
- **Idle**: Session exists but no recent activity
- **Expired**: Session past timeout threshold
- **Terminated**: Session explicitly ended by user or system
- **Impersonated**: Support user impersonating another user

### Subscription Plan Variations

#### Solo Plan Authentication
- **Subscriber-Admin = Teacher**: Single user with dual role capabilities
- **Teacher-Admin**: Disabled/grayed out in UI
- **Student Management**: Direct student creation by Subscriber-Admin
- **Simplified Flow**: Streamlined authentication for single-user scenarios

#### Ensemble Plan Authentication
- **Role Delegation**: Subscriber-Admin can create Teacher-Admin accounts
- **Teacher Management**: Teacher-Admin can create Teacher accounts
- **Student Management**: Teachers can create Student accounts
- **Complex Hierarchy**: Multi-level authentication and authorization

#### Prelude Plan Authentication
- **Trial Limitations**: Limited functionality and seat count
- **Teacher-Admin**: Disabled/grayed out
- **Upgrade Path**: Clear authentication flow to full subscription

## UX Requirements

### Login Interface
- **Clean Design**: Simple, accessible login form
- **Role Indicators**: Clear indication of available user types
- **Error Handling**: Clear, helpful error messages for failed attempts
- **MFA Integration**: Seamless MFA challenge flow
- **Password Reset**: Easy password reset process with clear instructions

### Session Management UI
- **Session Status**: Clear indication of session state and remaining time
- **Activity Indicators**: Visual feedback for session activity
- **Logout Options**: Clear logout button and session termination
- **Security Warnings**: Alerts for suspicious activity or session issues

### Accessibility
- **Screen Reader Support**: Proper ARIA labels and descriptions
- **Keyboard Navigation**: Full keyboard accessibility for all auth flows
- **High Contrast**: Clear visual distinction for all authentication states
- **Focus Management**: Proper focus handling during authentication flows
- **Error Announcements**: Screen reader announcements for authentication errors

### Mobile Considerations
- **Responsive Design**: Authentication flows work on all device sizes
- **Touch-Friendly**: Large touch targets for mobile devices
- **Biometric Integration**: Support for fingerprint/face authentication where available
- **Offline Handling**: Graceful handling of network connectivity issues

## Telemetry

### Authentication Events
- **Login Attempt**: `auth.login.attempt` - User attempts to log in
- **Login Success**: `auth.login.success` - Successful authentication
- **Login Failure**: `auth.login.failure` - Failed authentication attempt
- **MFA Setup**: `auth.mfa.setup` - User sets up multi-factor authentication
- **MFA Challenge**: `auth.mfa.challenge` - MFA challenge presented
- **MFA Success**: `auth.mfa.success` - Successful MFA verification
- **MFA Failure**: `auth.mfa.failure` - Failed MFA verification
- **Password Reset**: `auth.password.reset` - Password reset initiated
- **Password Change**: `auth.password.change` - Password successfully changed
- **Account Locked**: `auth.account.locked` - Account locked due to failed attempts
- **Account Unlocked**: `auth.account.unlocked` - Account unlocked by admin

### Session Events
- **Session Created**: `session.created` - New session established
- **Session Refreshed**: `session.refreshed` - Session token refreshed
- **Session Expired**: `session.expired` - Session timed out
- **Session Terminated**: `session.terminated` - Session ended by user
- **Session Invalidated**: `session.invalidated` - Session ended by system
- **Impersonation Started**: `session.impersonation.started` - Support impersonation begins
- **Impersonation Ended**: `session.impersonation.ended` - Support impersonation ends

### Event Properties
- `user_id`: User involved in the event
- `role`: User's role at time of event
- `organization_id`: User's organization
- `session_id`: Session identifier (for session events)
- `ip_address`: Client IP address
- `user_agent`: Client browser/device information
- `success`: Whether the event was successful
- `failure_reason`: Reason for failure if applicable
- `mfa_method`: MFA method used (if applicable)
- `impersonated_user_id`: User being impersonated (if applicable)

## Permissions

### Authentication Management
- **Subscriber-Admin**: Can manage authentication settings for their organization
- **Teacher-Admin**: Can manage authentication for teachers and students in their scope
- **Teacher**: Can manage authentication for their students
- **Student**: Can manage their own password and MFA settings
- **System Admin**: Can manage authentication for all users globally

### Session Management
- **Subscriber-Admin**: Can view and terminate sessions for their organization
- **Teacher-Admin**: Can view sessions for teachers and students in their scope
- **Teacher**: Can view sessions for their students
- **Student**: Can view and terminate their own sessions
- **System Admin**: Can view and manage all sessions globally

### Impersonation Access
- **System Admin Only**: Can impersonate any user for support purposes
- **Audit Required**: All impersonation activities must be logged
- **Time Limits**: Impersonation sessions have shorter timeouts
- **Notification**: Impersonated users notified of impersonation activity

## Supporting Documents Referenced

This session and authentication states specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Authentication requirements and session management rules for different user roles
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System admin authentication and security requirements
- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Overview of security and compliance requirements

## Dependencies

### Internal Dependencies
- [Roles Matrix](./roles-matrix.md) - User role definitions and hierarchies
- [Permissions Table](./permissions-table.md) - Detailed permission specifications
- [Impersonation and Support Access](./impersonation-and-support-access.md) - Support access controls
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) - Subscription plan details

### External Dependencies
- **Email Service**: Password reset and MFA setup notifications
- **SMS Service**: MFA backup codes and security alerts
- **Encryption Service**: Password hashing and token encryption
- **Audit System**: Authentication and session event logging
- **Notification System**: Security alerts and session notifications

### Technical Dependencies
- **Database**: User authentication and session storage
- **Cache Layer**: Session token caching and validation
- **Security Headers**: CSRF protection and secure cookie handling
- **Rate Limiting**: Brute force protection and abuse prevention

## Security Considerations

### Authentication Security
- **Password Hashing**: Use bcrypt with appropriate cost factor (12+)
- **Rate Limiting**: Implement progressive delays for failed login attempts
- **Account Lockout**: Temporary lockout after 5 failed attempts
- **MFA Enforcement**: Mandatory MFA for all admin roles
- **Session Security**: Secure session tokens with proper entropy

### Session Security
- **Token Rotation**: Regular rotation of session tokens
- **Secure Cookies**: HttpOnly, Secure, SameSite cookie attributes
- **CSRF Protection**: Anti-CSRF tokens for state-changing operations
- **IP Validation**: Optional IP address validation for admin sessions
- **Concurrent Session Limits**: Prevent session hijacking through limits

### Data Protection
- **Encryption**: Sensitive data encrypted at rest and in transit
- **Audit Logging**: Comprehensive logging of all authentication events
- **Data Retention**: Proper retention and deletion of authentication data
- **Privacy Compliance**: COPPA and FERPA compliance for student data

### Compliance Requirements
- **COPPA Compliance**: Special handling for students under 13
- **FERPA Compliance**: Educational data privacy protection
- **GDPR Compliance**: Data protection and user rights
- **SOC 2**: Security controls and audit requirements

## Open Questions

### Authentication Enhancements
- Should we support social login (Google, Microsoft) for teachers and admins?
- How should we handle passwordless authentication for mobile devices?
- Should we implement adaptive authentication based on risk factors?
- What biometric authentication options should we support?

### Session Management
- Should we implement device-specific session management?
- How should we handle session sharing for classroom environments?
- Should we support "remember me" functionality for trusted devices?
- What level of session analytics should we provide to admins?

### Security and Compliance
- Should we implement additional security measures for high-risk accounts?
- How should we handle authentication for international users?
- What additional compliance requirements should we consider?
- Should we implement automated threat detection for authentication?

### Integration Considerations
- How should authentication integrate with existing school systems (SSO)?
- What level of API authentication should we provide?
- Should we support SAML or OAuth for enterprise integrations?
- How should we handle authentication for third-party integrations?
