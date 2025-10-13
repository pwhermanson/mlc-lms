# Subscriber Dashboard

## Purpose

The Subscriber Dashboard serves as the central command center for subscription management, providing Subscriber-Admins with comprehensive oversight of their organization's usage, billing, and administrative functions. It offers real-time insights into platform adoption, user activity, and financial metrics.

The dashboard is specifically designed for the MLC-LMS platform's unique role hierarchy, supporting both Solo and Ensemble subscription models. For Solo plans, the Subscriber-Admin and Teacher are the same person, while Ensemble plans allow for delegation of educational management to Teacher-Admins while maintaining billing control at the Subscriber-Admin level.

## Dashboard Layout

### Header Section
- **Organization Name**: Clear identification of the subscription
- **Plan Information**: Current plan type (Prelude/Solo/Ensemble) and seat count
- **Billing Status**: Payment status and next billing date
- **Quick Actions**: Most common administrative tasks
- **Notifications**: Alerts for billing, usage, and system updates
- **Member Number**: Unique subscriber identification number
- **Sponsor Code**: Available for Ensemble plans to support Christine Hermanson Grant Program

### Main Content Areas

#### Overview Metrics
- **Active Users**: Current number of active users
- **Seat Utilization**: Percentage of seats being used
- **Monthly Usage**: Platform usage statistics
- **Billing Summary**: Current billing status and costs
- **Growth Trends**: User and usage growth over time

#### User Management
- **User Directory**: Complete list of all users in the organization
- **Role Distribution**: Breakdown of users by role type (Subscriber-Admin, Teacher-Admin, Teacher, Student, Generic Student)
- **Recent Activity**: Latest user actions and logins
- **Invitation Status**: Pending invitations and their status
- **User Health**: Users who may need attention or support
- **Teacher-Admin Management**: For Ensemble plans, manage Teacher-Admin roles and their scope
- **Student Seat Utilization**: Track which seats are actively used vs. available

#### Billing and Subscription
- **Current Plan**: Plan details and features
- **Seat Management**: Add, remove, or reassign seats
- **Billing History**: Past invoices and payments
- **Usage Reports**: Detailed usage and cost analysis
- **Upgrade Options**: Available plan upgrades and features

## Key Metrics and KPIs

### Usage Metrics
- **Daily Active Users**: Number of users active each day
- **Monthly Active Users**: Number of users active each month
- **Session Duration**: Average time users spend on the platform
- **Feature Adoption**: Which features are being used most
- **Content Engagement**: How users interact with learning content

### Financial Metrics
- **Monthly Recurring Revenue**: Current subscription revenue
- **Seat Utilization Rate**: Percentage of seats actively used
- **Cost Per User**: Average cost per active user
- **Billing Efficiency**: Payment success and collection rates
- **Growth Rate**: Month-over-month growth in revenue and users

### Learning Outcomes
- **Assignment Completion**: Percentage of assignments completed
- **Mastery Rates**: Student mastery of learning concepts (target score achievement)
- **Teacher Satisfaction**: Feedback from teachers on platform effectiveness
- **Student Engagement**: Student participation and progress metrics
- **Learning Progress**: Overall learning outcomes and improvements
- **Game Performance**: Individual game scores and completion rates
- **Sequence Progress**: Progress through assigned learning sequences (Lifetime Musician, Solfege, Evaluation)
- **Challenge Participation**: Student engagement with worldwide challenge games

## User Management Features

### User Directory
- **Comprehensive List**: All users with role, status, and activity information
- **Advanced Filtering**: Filter by role, status, activity, and other criteria
- **Bulk Operations**: Select multiple users for bulk actions
- **Search and Sort**: Find specific users quickly
- **Export Options**: Download user data for external analysis

*For detailed member management capabilities, see [Member Management](./member-management.md)*

### User Administration
- **Add Users**: Invite new users to the platform
- **Edit Users**: Modify user information and roles
- **Deactivate Users**: Temporarily disable user accounts
- **Role Management**: Assign and modify user roles
- **Permission Control**: Manage user access and permissions

### Teacher-Admin Management (Ensemble Plans)
- **Teacher-Admin Assignment**: Assign Teacher-Admin roles with unique usernames and passwords
- **Scope Management**: Define what each Teacher-Admin can access (teachers and students under their management)
- **Delegation Control**: Manage what responsibilities are delegated (educational vs. business management)
- **Oversight Tools**: Monitor Teacher-Admin activities and performance
- **Support Access**: Provide support to Teacher-Admins
- **Role Separation**: Handle cases where Teacher-Admin is same person as Subscriber-Admin vs. separate individuals
- **Educational Management**: Delegate teacher creation, student assignment, and sequence management

*For detailed role definitions and permissions, see [Roles Matrix](../02-roles-and-permissions/roles-matrix.md)*

## Billing and Subscription Management

### Subscription Overview
- **Plan Details**: Current plan features and limitations (Prelude/Solo/Ensemble)
- **Seat Count**: Total seats and utilization
- **Billing Cycle**: Monthly or annual billing information
- **Renewal Date**: When the subscription renews
- **Contract Terms**: Contract length and terms
- **Member Number**: Unique subscriber identification
- **Subscription Status**: Active, suspended, or terminated status
- **Trial Information**: For Prelude plans, trial duration and conversion status

### Seat Management
- **Add Seats**: Increase seat count with prorated billing
- **Remove Seats**: Decrease seat count with credits
- **Seat Assignment**: Assign seats to specific users
- **Seat Transfers**: Move seats between users
- **Overage Management**: Handle seats beyond plan limits
- **Solo Plan Seats**: Add/reduce in increments as low as one student slot ($0.80 per month over 5)
- **Ensemble Plan Seats**: Add/reduce in increments of 5 student slots ($0.20 per month over 20)
- **Annual Seat Changes**: Prorated billing for increases, no decreases until renewal

### Billing Operations
- **Payment Methods**: Manage credit cards and payment options (Discover, American Express, MasterCard, Visa, PayPal)
- **Invoice Management**: View and download invoices
- **Payment History**: Complete payment and transaction history
- **Billing Alerts**: Notifications for payment issues
- **Refund Processing**: Handle refunds and credits
- **Subscription Suspension**: Schedule pause and restart of monthly subscriptions
- **Data Retention**: Student scores data retained for 13 months after subscription termination
- **Christine Hermanson Grant Program**: Manage sponsor codes for free school subscriptions

*For detailed billing specifications, see [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) and [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md)*

## Reporting and Analytics

### Usage Reports
- **User Activity**: Detailed user activity and engagement
- **Feature Usage**: Which features are being used most
- **Content Analytics**: Learning content performance
- **Time Tracking**: How much time users spend on the platform
- **Device Analytics**: Usage across different devices and browsers

### Financial Reports
- **Revenue Tracking**: Monthly and annual revenue trends
- **Cost Analysis**: Breakdown of costs by user, feature, and time period
- **ROI Analysis**: Return on investment for the platform
- **Growth Projections**: Forecast future revenue and costs
- **Comparative Analysis**: Compare performance across time periods

### Learning Analytics
- **Student Progress**: Overall student learning progress
- **Teacher Effectiveness**: Teacher usage and effectiveness metrics
- **Content Performance**: Which content is most effective
- **Learning Outcomes**: Measurable learning achievements
- **Intervention Alerts**: Students or teachers who need support
- **Game Mastery Tracking**: Target score achievement rates across all games
- **Sequence Completion**: Progress through Lifetime Musician, Solfege, and Evaluation sequences
- **Challenge Game Performance**: Student participation and performance in worldwide competitions
- **Retention Analysis**: Review stage completion rates for concept retention

*For detailed KPI specifications and data models, see [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md)*

## Administrative Tools

### System Configuration
- **Organization Settings**: Configure organization-wide settings
- **Feature Flags**: Enable or disable platform features
- **Integration Settings**: Configure external integrations
- **Security Settings**: Manage security and access controls
- **Notification Preferences**: Configure system notifications

### Bulk Operations
- **CSV Import/Export**: Bulk user management via CSV files
- **Mass Assignment**: Assign content to multiple users
- **Bulk Communications**: Send messages to multiple users
- **Data Migration**: Import data from other systems
- **Backup and Recovery**: Data backup and recovery operations
- **Student Information Upload**: For Ensemble plans, bulk student data import utility
- **Generic Student Management**: Create and manage shared usernames for group learning
- **Sequence Assignment**: Bulk assignment of learning sequences to student groups

### Support and Maintenance
- **Support Tickets**: Track and manage support requests
- **System Status**: Monitor platform health and performance
- **Maintenance Windows**: Schedule and manage maintenance
- **Update Management**: Handle platform updates and upgrades
- **Documentation**: Access to administrative documentation

## Communication and Notifications

### User Communication
- **Announcements**: Send announcements to all users
- **Targeted Messages**: Send messages to specific user groups
- **Email Templates**: Pre-built email templates for common communications
- **Notification Center**: Centralized notification management
- **Message History**: Archive of all communications

### System Notifications
- **Billing Alerts**: Notifications about billing and payment issues
- **Usage Alerts**: Warnings about usage limits or unusual activity
- **System Updates**: Information about platform updates and changes
- **Security Alerts**: Notifications about security-related events
- **Maintenance Notices**: Advance notice of scheduled maintenance

## Mobile and Responsive Design

### Mobile Dashboard
- **Responsive Layout**: Optimized for mobile devices
- **Touch-friendly**: Large buttons and touch targets
- **Key Metrics**: Most important metrics visible on mobile
- **Quick Actions**: Common tasks easily accessible
- **Offline Support**: Basic functionality without internet connection

### Tablet Experience
- **Split View**: Content and navigation side-by-side
- **Touch and Mouse**: Support for both input methods
- **Enhanced Analytics**: More detailed analytics on larger screens
- **Multi-tasking**: Support for multiple tasks simultaneously
- **Portrait/Landscape**: Optimized for both orientations

## Security and Compliance

### Data Security
- **Access Controls**: Role-based access to sensitive data
- **Audit Logging**: Complete audit trail of all actions
- **Data Encryption**: Sensitive data encrypted at rest and in transit
- **Session Management**: Secure session handling
- **Multi-factor Authentication**: Enhanced security for admin accounts

### Compliance Features
- **Data Privacy**: Compliance with privacy regulations
- **Audit Reports**: Generate reports for compliance audits
- **Data Retention**: Manage data retention policies
- **User Consent**: Track and manage user consent
- **Regulatory Compliance**: Meet educational and data regulations

## MLC-LMS Specific Features

### Music Learning Platform Integration
- **Game Registry Access**: Direct access to the complete library of music learning games
- **Sequence Management**: Manage and assign learning sequences (Lifetime Musician, Solfege, Evaluation)
- **MIDI Support**: Monitor and configure MIDI keyboard integration for games
- **Challenge Game Administration**: Oversee worldwide student competitions and leaderboards
- **Terry Treble Branding**: Access to platform mascot and branding elements

### Educational Content Management
- **Learning Element Tracking**: Monitor text, graphics, music, video, and awards in sequences
- **Method Alignment**: Track alignment with popular piano methods and curricula
- **Student Pages Management**: Access to archived curriculum mapping materials
- **Progressive Learning**: Monitor student progression through Learn, Play, Quiz, Challenge, and Review stages
- **Mastery Indicators**: Track green check marks and target score achievements

### Platform-Specific Analytics
- **Game Performance Metrics**: Individual game completion and mastery rates
- **Aural Skills Development**: Track ear training and music theory progress
- **Sight-Reading Improvement**: Monitor development of sight-reading skills
- **Retention Analysis**: Track concept retention through Review stages
- **Pedagogical Effectiveness**: Measure alignment with music education standards

## Supporting Documents Referenced

This subscriber dashboard specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Subscriber-Admin role capabilities, permissions, and organizational management
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Administrative interface content, navigation, and dashboard layout
- [Dashboard Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Dashboard%20Chart.xlsx%20-%20Sheet1.csv) — Dashboard component specifications and layout requirements
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Seat management and subscription administration specifications

## Future Enhancements

### Advanced Analytics
- **Predictive Analytics**: Forecast usage and growth trends
- **AI Insights**: AI-powered insights and recommendations
- **Custom Dashboards**: Personalized dashboard configurations
- **Real-time Monitoring**: Live monitoring of platform activity
- **Advanced Reporting**: More sophisticated reporting capabilities

### Integration Features
- **API Access**: Programmatic access to platform data
- **Third-party Integrations**: Connect with external tools and systems
- **Data Export**: Enhanced data export capabilities
- **Webhook Support**: Real-time notifications to external systems
- **Single Sign-On**: Integration with identity providers

