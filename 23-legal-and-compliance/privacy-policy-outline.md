# Privacy Policy Outline

This document defines the comprehensive outline, structure, and requirements for the MLC-LMS platform privacy policy. It serves as the specification for legal counsel and compliance teams to create a complete, legally compliant privacy policy that meets all regulatory requirements while being accessible and transparent to users.

## Purpose

This specification enables:

- **Regulatory Compliance**: Ensures the privacy policy addresses all applicable laws (COPPA, FERPA, GDPR, CCPA, state laws)
- **User Transparency**: Provides clear, understandable information about data collection and usage practices
- **Trust Building**: Demonstrates commitment to protecting user privacy, especially children's data
- **Operational Guidance**: Defines what privacy practices must be implemented in the platform
- **Legal Protection**: Establishes clear terms for data handling, liability, and user rights
- **Audit Readiness**: Documents privacy practices for regulatory audits and compliance reviews
- **Cross-Reference Framework**: Connects privacy policy to technical implementation specs

## Scope

### Included

This outline covers:

- **Required Policy Sections**: All mandatory sections for a comprehensive privacy policy
- **User Types**: Data collection practices for all personas (students, teachers, parents, admins, system admins)
- **Data Categories**: Classification of personal data, educational records, and sensitive information
- **Compliance Requirements**: Specific requirements for COPPA, FERPA, GDPR, CCPA, and other applicable laws
- **User Rights**: How users can access, export, correct, and delete their data
- **Consent Mechanisms**: How and when consent is obtained, especially for minors
- **Data Retention**: How long different types of data are retained and deletion schedules
- **Third-Party Sharing**: What data is shared with vendors (Vercel, Neon, Cloudflare, Braintree, email provider)
- **Security Practices**: High-level description of security measures protecting user data
- **Cookies and Tracking**: What tracking technologies are used (or explicitly not used on student pages)
- **International Users**: Data transfers and protections for users outside the United States
- **Policy Updates**: How users will be notified of privacy policy changes

### Excluded

This outline does not cover:

- **Actual Legal Language**: Final legal wording must be drafted by qualified legal counsel
- **Terms of Service**: Covered separately in [Terms and Conditions](./terms-conditions.md)
- **Technical Implementation**: Detailed security architecture in [Security and Privacy](../18-architecture/security-privacy.md)
- **Compliance Procedures**: Operational compliance workflows in [COPPA/FERPA Notes](./coppa-ferpa-notes.md), [GDPR Compliance](./gdpr-compliance.md), [FERPA Compliance](./ferpa-compliance.md)
- **Data Model Details**: Full entity schemas in [Data Model ERD](../18-architecture/data-model-erd.md)
- **Internal Policies**: Staff training, incident response procedures (operational documentation)

## Data Classification and Collection

This section defines what types of data are collected, how they are classified, and from which user types.

### Data Categories

#### Public Data
- Marketing content, help documentation, game descriptions
- Publicly visible leaderboard data (with parental consent for students under 13)
- Public-facing platform information

#### Internal Data
- System configuration, non-PII metadata
- Aggregated usage statistics (anonymized)
- Platform performance metrics

#### Confidential Data
- Subscription and billing details
- Organization information (school names, contacts)
- Teacher contact information
- Usage analytics tied to organizations
- Payment history (stored by Braintree, not in MLC database)

#### Restricted Data (Highest Protection)
- **Student PII (COPPA-protected for children under 13)**:
  - Student names (required)
  - Student usernames (required)
  - Student email addresses (optional, requires parental consent for under 13)
  - Date of birth (optional)
  - Parental consent records
  - Generic Student data (no PII collected by design)

- **Educational Records (FERPA-protected)**:
  - Game scores and performance data
  - Assignment completion records
  - Learning progress and mastery tracking
  - Teacher notes and assessments

- **Payment Information (PCI DSS)**:
  - Credit card data (handled by Braintree, not stored in MLC systems)
  - Purchase order numbers
  - Billing addresses

- **Authentication Data**:
  - Password hashes (bcrypt)
  - MFA secrets (encrypted)
  - Session tokens

### Data Collection by User Type

#### Student Accounts
**Collected Data**:
- Username (required)
- Student name (first and last)
- Email address (optional, requires parental consent for under 13)
- Date of birth (optional)
- Game play data (scores, attempts, duration)
- Assignment progress
- Achievement and badge data
- Session data (IP address, user agent, login times)

**Special Protections**:
- No behavioral tracking or advertising cookies on student pages
- No data sold to third parties
- Educational purpose only
- Parental consent required for students under 13 per COPPA
- Generic Student option available with zero PII collection

#### Generic Student Accounts
**Collected Data**:
- Shared username/password only
- No individual PII collected
- No score tracking tied to individuals
- Session data only (aggregated)

**Use Case**: General music classes where individual tracking is not needed

#### Teacher Accounts
**Collected Data**:
- Username and email (required)
- Full name (required)
- Phone number (optional)
- Organization affiliation
- Students taught (relationships)
- Classes and assignments created
- Communication history (messages sent)
- Session and activity data

#### Parent Accounts (Future Enhancement)
**Collected Data** (if implemented):
- Parent name and email (required)
- Relationship to student(s)
- Consent records for students under 13
- Communication preferences
- Session data

#### Subscriber-Admin and Teacher-Admin Accounts
**Collected Data**:
- Username and email (required)
- Full name (required)
- Phone number (optional)
- Billing address
- Payment method (stored by Braintree)
- Organization details
- Subscription and seat management actions
- Audit logs of administrative actions

#### System Admin Accounts
**Collected Data**:
- Username and email (required)
- Full name (required)
- Phone number (required)
- Audit logs of all platform actions
- Impersonation session logs

### Data Not Collected

To minimize privacy risk, the following data is **not collected**:

- Social Security Numbers
- Driver's license numbers
- Student home addresses (unless required for billing)
- Student phone numbers
- Precise geolocation data
- Behavioral tracking across non-MLC websites
- Advertising or marketing data about students
- Biometric data
- Genetic or health information

## Required Privacy Policy Sections

The privacy policy must include the following sections with clear, accessible language:

### 1. Introduction and Effective Date
- Who we are (Music Learning Community, Terry Treble)
- What this policy covers
- Last updated date
- Link to previous policy versions

### 2. Information We Collect

#### A. Information You Provide
- Account creation data (username, email, name)
- Subscription and billing information
- Student enrollment data
- Communication content (messages, support requests)
- Profile customization

#### B. Information Automatically Collected
- Game play data and scores
- Assignment completion and progress
- Session information (login times, IP addresses)
- Device and browser information
- Usage analytics
- Performance metrics

#### C. Information from Third Parties
- Payment processor (Braintree) transaction confirmations
- School or organization data (for institutional subscriptions)

### 3. How We Use Your Information

**Educational Purposes**:
- Deliver educational content and games
- Track learning progress and mastery
- Generate assignments and sequences
- Provide reports to teachers and parents
- Award badges and achievements

**Account Management**:
- Create and manage user accounts
- Authenticate users and maintain sessions
- Provide customer support
- Send account notifications

**Platform Improvement**:
- Improve game quality and effectiveness
- Optimize platform performance
- Develop new features
- Conduct educational research (with anonymized data)

**Billing and Compliance**:
- Process payments and subscriptions
- Manage seats and organizational hierarchies
- Comply with legal obligations
- Enforce terms of service

**Prohibited Uses**:
- We do NOT sell student data to third parties
- We do NOT use student data for advertising
- We do NOT track students across other websites
- We do NOT use student data for non-educational purposes

### 4. Information Sharing and Disclosure

#### With Your Consent
- Teacher access to student progress (authorized by subscriber)
- Parent access to student data (for their own children)
- Public leaderboards (with parental consent for students under 13)

#### Service Providers
List all third-party vendors and what data they access:

- **Vercel** (Hosting): Session data, application logs
- **Neon** (Database): All user data (encrypted at rest)
- **Cloudflare** (CDN/Security): Access logs, cached content
- **Braintree** (Payments): Billing information, payment card data
- **Email Provider** (TBD): Email addresses, notification content

All vendors must:
- Sign Data Processing Agreements
- Comply with applicable privacy laws
- Use data only for specified purposes
- Implement appropriate security measures

#### Legal Requirements
- Comply with court orders and legal process
- Protect rights, property, and safety
- Investigate violations of terms of service
- Comply with FERPA, COPPA, and other applicable laws

#### Business Transfers
- In event of merger, acquisition, or asset sale
- User data may be transferred with notice to users

### 5. Data Security

High-level description of security practices (details in [Security and Privacy](../18-architecture/security-privacy.md)):

- **Encryption**: Data encrypted at rest (AES-256) and in transit (TLS 1.3)
- **Access Controls**: Role-based permissions, least privilege principle
- **Authentication**: Secure password hashing, MFA for admin accounts
- **Monitoring**: Security logging, anomaly detection, incident response
- **Testing**: Regular security audits and vulnerability assessments
- **Compliance**: SOC 2 certified vendors, PCI DSS compliant payment processing

No system is perfectly secure. We implement industry-standard protections and will notify affected users of any data breach as required by law.

### 6. Data Retention and Deletion

#### Retention Periods
- **Active Subscriptions**: Data retained while subscription is active
- **Cancelled Subscriptions**: Data retained for 13 months post-cancellation
- **Support Data**: Retained for 3 years
- **Security Logs**: Retained for 1 year minimum
- **Backups**: Encrypted backups retained for 90 days

#### Automated Deletion
- Daily background jobs identify and delete expired data
- Soft delete (30-day grace period), then permanent deletion
- Cascading deletion of related records

#### User-Requested Deletion
Users can request account deletion at any time. Process completes within 30 days.

Exceptions:
- Financial records retained for tax/audit purposes (7 years)
- Security logs retained per compliance requirements
- Aggregated, anonymized data may be retained for research

### 7. Your Rights and Choices

#### All Users
- **Access**: View your personal data
- **Export**: Download your data in machine-readable format (JSON)
- **Correction**: Update inaccurate information
- **Deletion**: Request account deletion ("right to be forgotten")
- **Communication**: Opt out of non-essential emails
- **Account Closure**: Cancel subscription and delete account

#### Students Under 13 (COPPA Rights)
- Parents can review student data
- Parents can request deletion of student data
- Parents can refuse further collection of student data
- Verifiable parental consent required for PII collection

#### EU Users (GDPR Rights)
- Right to access, rectification, erasure, portability
- Right to restrict processing
- Right to object to processing
- Right to withdraw consent
- Right to lodge complaint with supervisory authority

#### Educational Records (FERPA Rights)
- Parents/eligible students can inspect educational records
- Request amendment of inaccurate records
- Control disclosure of records (except as permitted by FERPA)

**How to Exercise Rights**:
- Self-service account dashboard
- Email: privacy@musiclearningcommunity.com
- Written request to mailing address
- Response within 30 days of verified request

### 8. Children's Privacy (COPPA Compliance)

**Age Requirements**:
- Platform designed for ages 6-18
- Students under 13 require verifiable parental consent

**Parental Consent Process**:
- Teacher or parent creates student account
- Teacher certifies they have obtained parental consent
- Parent email address collected for verification
- Consent verification email sent to parent

**Data Minimization**:
- Collect only data necessary for educational purposes
- Student name and username required
- Student email optional (requires parental consent)
- No behavioral tracking or advertising on student pages

**Generic Student Option**:
- Zero PII collection alternative
- Suitable for classroom settings without individual tracking

**Parental Rights**:
- Review, export, or delete student data at any time
- Refuse further data collection (may limit functionality)
- Contact: coppa@musiclearningcommunity.com

### 9. International Users

**Data Location**:
- Primary servers located in United States
- CDN nodes distributed globally (Cloudflare)

**International Data Transfers**:
- EU users: Data transferred to US under appropriate safeguards
- Standard Contractual Clauses with EU data processors
- Adequacy decisions where applicable

**Regional Rights**:
- GDPR rights for EU users
- CCPA/CPRA rights for California residents
- Other regional privacy laws respected as applicable

### 10. Cookies and Tracking Technologies

**Essential Cookies**:
- Session authentication
- Security (CSRF protection)
- Load balancing

**Analytics Cookies** (if used):
- Usage statistics (aggregated and anonymized)
- Performance monitoring
- Error tracking
- **NOT used on student-facing pages**

**No Third-Party Advertising**:
- We do not use advertising cookies
- We do not participate in ad networks
- We do not track students across other websites

**Cookie Control**:
- Users can disable non-essential cookies in browser settings
- Essential cookies required for platform functionality

### 11. Changes to This Privacy Policy

**Notification Process**:
- Email notification to account holders
- Prominent notice on website
- In-app notification upon next login
- 30-day notice before changes take effect

**Material Changes**:
- Changes affecting data collection practices
- Changes to data sharing with third parties
- Changes to user rights
- Require renewed consent for students under 13

**Version History**:
- Previous versions archived and accessible
- Change log maintained with dates and summary of changes

### 12. Contact Information

**Privacy Inquiries**:
- Email: privacy@musiclearningcommunity.com
- Phone: 855-687-4240

**COPPA Inquiries**:
- Email: coppa@musiclearningcommunity.com

**GDPR Data Protection Officer** (if required):
- Email: dpo@musiclearningcommunity.com

**Mailing Address**:
Music Learning Community  
[Street Address]  
Wildwood, Missouri 63011  
United States of America

**Response Time**: Within 30 days of receiving verified inquiry

### 13. State-Specific Disclosures

#### California Residents (CCPA/CPRA)
- Categories of personal information collected
- Business purposes for collection
- Categories of third parties receiving data
- Right to opt out of "sale" (we do not sell data)
- Non-discrimination for exercising privacy rights

#### Other States
- Comply with applicable state privacy laws
- Specific disclosures added as laws evolve

## Consent Mechanisms

### Account Creation Consent
- Privacy policy link presented during signup
- "I agree to Privacy Policy" checkbox required
- Cannot proceed without consent
- Consent timestamp recorded

### Parental Consent for Students Under 13
- Teacher creates student account
- Teacher certifies parental consent obtained (checkbox)
- Verification email sent to parent email address
- Parent must confirm consent via email link
- Consent record stored with timestamp and IP address
- Annual re-verification recommended

### Optional Data Collection
- Separate consent for optional fields (student email, date of birth)
- Clear explanation of why data is requested
- Can decline without limiting core functionality

### Marketing Communications
- Separate opt-in for newsletters and promotional emails
- Easy unsubscribe link in every email
- Preferences managed in account settings

### Consent Withdrawal
- Users can withdraw consent at any time
- May limit functionality or require account deletion
- Process completes within 30 days

## Data Subject Access Request (DSAR) Process

### Request Submission
- Online form at musiclearningcommunity.com/privacy/dsar
- Email to privacy@musiclearningcommunity.com
- Written mail to physical address

### Identity Verification
- Verify requestor identity before releasing data
- Multi-factor authentication for account holders
- Additional verification for non-account requests

### Request Fulfillment
- Acknowledge request within 5 business days
- Fulfill request within 30 days (45 days if complex)
- Provide data in machine-readable format (JSON, CSV)
- Include all personal data associated with account

### Request Types
- **Access**: Provide copy of all personal data
- **Correction**: Update inaccurate data
- **Deletion**: Delete account and all associated data
- **Export**: Provide data portability file
- **Restriction**: Temporarily restrict processing (GDPR)
- **Objection**: Object to specific processing activities (GDPR)

### Exceptions
- Cannot delete data required for legal obligations
- Cannot provide data that would reveal other users' information
- Cannot fulfill requests that are manifestly unfounded or excessive

## UX Requirements

### Privacy Policy Presentation

#### Accessibility and Discoverability
- Privacy Policy link in footer of every page
- Privacy Policy link during account creation (required checkbox)
- Prominent link in account settings
- Mobile-responsive design for all screen sizes
- Plain language version for students/parents (WCAG AA compliant)
- Available in multiple languages (if platform is localized)

#### Content Organization
- Table of contents with jump links for long policy
- Collapsible sections for easier navigation
- "Last Updated" date prominently displayed at top
- Summary box at top with key points (optional)
- Search functionality for finding specific topics

#### Readability
- Plain language, avoiding excessive legal jargon
- Sentence structure appropriate for general audience (8th-grade reading level)
- Use bullet points and short paragraphs
- Bold or highlight key commitments ("We do NOT sell student data")
- Visual icons for different sections (optional)

### Consent Interface

#### Account Creation Flow
1. User enters account information
2. Privacy Policy and Terms checkbox displayed
3. Link to full policy opens in new tab/modal
4. Must check box to proceed
5. Consent timestamp and IP recorded in database

**UI Requirements**:
- Checkbox cannot be pre-checked (must be explicit action)
- Link text: "I agree to the Privacy Policy and Terms of Service"
- Links must be clearly clickable and distinguishable
- Error message if user tries to proceed without consent

#### Parental Consent for Students Under 13
1. Teacher creates student account
2. Teacher certifies parental consent obtained (checkbox)
3. Optional: Parent email field for verification
4. If email provided, verification email sent automatically
5. Parent clicks verification link in email
6. Consent confirmed and recorded

**UI Requirements**:
- Clear explanation of COPPA requirements
- Teacher checkbox: "I certify that I have obtained verifiable parental consent for this student under 13"
- Optional parent email field with help text
- Verification email template with clear instructions
- Confirmation page after parent verifies

### Privacy Dashboard

#### User Account Settings - Privacy Section
- **My Data**: View all personal data collected
- **Export Data**: Download data in JSON/CSV format
- **Communication Preferences**: Manage email subscriptions
- **Delete Account**: Request account deletion
- **Privacy Policy**: Link to current policy
- **Data Retention**: Show when data will be deleted (if subscription cancelled)

**For Parents** (if parent accounts implemented):
- View student data
- Manage student privacy settings
- Export or delete student data
- Manage parental consent status

**For Admins**:
- View organization data
- Export organization and user data
- Manage consent records for students under 13
- Access Data Processing Agreements with vendors

### Privacy Notifications

#### In-App Notifications
- Privacy policy updates (with summary of changes)
- Data breach notifications (if required)
- Parental consent expiration reminders
- Data deletion confirmations

#### Email Notifications
- Privacy policy changes (30 days before effective)
- Parental consent verification requests
- Data export completion (with download link)
- Account deletion confirmation
- DSAR acknowledgment and completion

### Privacy-Related Error Messages

See [Error Messages](../22-content-style-guides/error-messages.md) for full catalog. Key privacy-related messages:

- "Parental consent required for students under 13"
- "Privacy policy must be accepted to create account"
- "Email verification required to complete parental consent"
- "Your data export request is being processed. You'll receive an email within 24 hours."
- "Account deletion scheduled. You have 30 days to cancel this request."

### Mobile and Accessibility

- Privacy policy must be fully readable on mobile devices
- No horizontal scrolling required
- Touch-friendly links and buttons (minimum 44x44 pixels)
- Screen reader compatible (proper heading hierarchy)
- Keyboard navigation support
- Color contrast meets WCAG AA standards

### Multilingual Support

If platform supports multiple languages:
- Privacy policy translated into all supported languages
- Language selector at top of privacy policy page
- Consent forms in user's selected language
- Email notifications in user's preferred language
- Legal review of translations for accuracy

## Telemetry

Privacy-related events to be tracked via the platform [Event Model](../15-analytics-and-reporting/event-model.md):

### Consent Events
- `privacy.consent.given` - User accepts privacy policy
  - Properties: `user_id`, `user_role`, `consent_type` (privacy_policy, marketing, parental), `ip_address`, `timestamp`
- `privacy.consent.withdrawn` - User withdraws consent
  - Properties: `user_id`, `consent_type`, `reason`, `timestamp`
- `privacy.parental_consent.requested` - Parental consent email sent
  - Properties: `student_id`, `parent_email`, `timestamp`
- `privacy.parental_consent.verified` - Parent confirms consent via email link
  - Properties: `student_id`, `parent_email`, `verification_token`, `timestamp`
- `privacy.parental_consent.declined` - Parent declines consent
  - Properties: `student_id`, `parent_email`, `reason`, `timestamp`

### Data Access Events
- `privacy.policy.viewed` - User views privacy policy
  - Properties: `user_id`, `page_url`, `source` (signup, footer, settings), `timestamp`
- `privacy.data.accessed` - User views their personal data
  - Properties: `user_id`, `data_categories_viewed`, `timestamp`
- `privacy.data.exported` - User requests data export
  - Properties: `user_id`, `export_format` (JSON, CSV), `data_size_bytes`, `timestamp`
- `privacy.data.export_completed` - Data export file generated
  - Properties: `user_id`, `file_url`, `expiration_time`, `timestamp`
- `privacy.data.export_downloaded` - User downloads export file
  - Properties: `user_id`, `file_url`, `timestamp`

### Deletion Events
- `privacy.deletion.requested` - User requests account deletion
  - Properties: `user_id`, `user_role`, `reason`, `scheduled_deletion_date`, `timestamp`
- `privacy.deletion.cancelled` - User cancels scheduled deletion
  - Properties: `user_id`, `timestamp`
- `privacy.deletion.completed` - Account permanently deleted
  - Properties: `user_id` (anonymized), `data_types_deleted`, `timestamp`
- `privacy.data.anonymized` - User data anonymized for research
  - Properties: `original_user_id` (hashed), `data_categories`, `purpose`, `timestamp`

### Policy Update Events
- `privacy.policy.updated` - Privacy policy revised
  - Properties: `version_number`, `change_summary`, `effective_date`, `notification_sent`, `timestamp`
- `privacy.policy.notification_sent` - Policy update notification sent to users
  - Properties: `recipient_count`, `notification_type` (email, in_app), `timestamp`
- `privacy.policy.re_consent_required` - User must re-consent due to material changes
  - Properties: `user_id`, `reason`, `deadline_date`, `timestamp`
- `privacy.policy.re_consent_given` - User re-accepts updated policy
  - Properties: `user_id`, `new_policy_version`, `timestamp`

### DSAR (Data Subject Access Request) Events
- `privacy.dsar.submitted` - User submits DSAR
  - Properties: `user_id`, `request_type` (access, deletion, correction, export), `contact_email`, `timestamp`
- `privacy.dsar.acknowledged` - DSAR acknowledged by system
  - Properties: `request_id`, `estimated_completion_date`, `timestamp`
- `privacy.dsar.identity_verified` - User identity verified for DSAR
  - Properties: `request_id`, `verification_method`, `timestamp`
- `privacy.dsar.fulfilled` - DSAR completed
  - Properties: `request_id`, `request_type`, `fulfillment_method`, `completion_time_hours`, `timestamp`
- `privacy.dsar.rejected` - DSAR rejected as unfounded or excessive
  - Properties: `request_id`, `rejection_reason`, `timestamp`

### Security and Breach Events
- `privacy.breach.detected` - Potential data breach detected
  - Properties: `severity`, `affected_users_count`, `data_types_affected`, `timestamp`
- `privacy.breach.notification_sent` - Breach notification sent to affected users
  - Properties: `affected_users_count`, `notification_method`, `regulatory_notification_sent`, `timestamp`

### Compliance Audit Events
- `privacy.audit.started` - Privacy audit initiated
  - Properties: `audit_type` (internal, third_party), `auditor`, `scope`, `timestamp`
- `privacy.audit.completed` - Privacy audit completed
  - Properties: `audit_id`, `findings_count`, `issues_identified`, `timestamp`
- `privacy.dpa.signed` - Data Processing Agreement signed with vendor
  - Properties: `vendor_name`, `agreement_version`, `effective_date`, `timestamp`

### Event Properties

All privacy events include:
- `event_id`: Unique event identifier (UUID)
- `timestamp`: ISO 8601 timestamp
- `user_id`: User performing action (nullable for system events)
- `organization_id`: Associated organization (nullable)
- `ip_address`: Source IP address (for consent and security events)
- `user_agent`: Browser/device information
- `session_id`: Current session identifier

### Privacy Event Retention
- Consent events: Retained for life of account + 7 years (legal requirement)
- Access and export events: Retained for 1 year
- Deletion events: Retained for 7 years (legal requirement, user_id anonymized)
- DSAR events: Retained for 3 years (compliance requirement)
- Breach events: Retained for 7 years (legal requirement)

### Event Access Restrictions
- Privacy events contain sensitive data and are restricted:
  - **System Admin**: Full access to all privacy events
  - **Subscriber-Admin**: Access to events for their organization only
  - **Users**: Access to their own privacy events only
  - **Auditors**: Read-only access with explicit authorization

## Permissions

Privacy policy management and data access permissions are defined by role. See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for complete specifications.

### Privacy Policy Management

**View Privacy Policy**:
- All roles: Public access to current privacy policy

**Update Privacy Policy**:
- System Admin only: Can update privacy policy content
- Requires: Legal review and approval workflow
- Triggers: Notification to all active users
- Version control: Previous versions archived and accessible

**Manage Policy Versions**:
- System Admin only: Access to version history
- Can view previous versions and change logs
- Can restore previous version if needed (with approval)

### User Data Access

**View Own Data**:
- All authenticated users: Can view their own personal data
- Students: Can view their own scores and progress
- Teachers: Can view their own teaching data
- Admins: Can view their own administrative data

**View Student Data**:
- Teachers: Can view data for students they teach (within their classes)
- Subscriber-Admin: Can view data for all students in their organization
- Teacher-Admin: Can view data for all students in their scope
- Parents (future): Can view data for their own children only
- System Admin: Can view any student data with audit logging

**Export Data**:
- All users: Can export their own data
- Teachers: Can export class roster and aggregate (not individual) student data
- Admins: Can export organization data and aggregated student data
- System Admin: Can export any data with proper justification and audit logging

**Delete Data**:
- All users: Can request deletion of their own account
- Parents: Can request deletion of student data for children under 13
- Subscriber-Admin: Can delete accounts within their organization
- System Admin: Can delete any account with proper justification
- Cannot delete: Data required for legal/financial obligations

**Correct Data**:
- All users: Can update their own profile information
- Teachers: Can update student names and usernames (with appropriate permission)
- Admins: Can correct inaccurate data for users in their organization
- System Admin: Can correct any data with audit logging

### Consent Management

**Provide Consent**:
- All users: Must consent to privacy policy during account creation
- Parents: Can provide consent for students under 13
- Teachers: Can certify that parental consent has been obtained

**View Consent Records**:
- Users: Can view their own consent history
- Parents: Can view consent records for their children
- Admins: Can view consent records for their organization (aggregate)
- System Admin: Can view all consent records

**Revoke Consent**:
- All users: Can withdraw consent (may require account deletion)
- Parents: Can revoke consent for students under 13

### DSAR (Data Subject Access Request) Processing

**Submit DSAR**:
- All users: Can submit requests for access, export, correction, or deletion
- Parents: Can submit requests on behalf of children under 13

**Process DSAR**:
- System Admin only: Can review and fulfill DSARs
- Requires: Identity verification before processing
- Timeline: Acknowledge within 5 days, fulfill within 30 days

**Audit DSAR**:
- System Admin: Can view DSAR logs and history
- External Auditors: Read-only access with explicit authorization

### Privacy Settings Management

**Manage Own Settings**:
- All users: Can manage their own privacy preferences
  - Communication preferences (email, notifications)
  - Data sharing preferences (leaderboards, public profiles)
  - Marketing opt-in/out

**Manage Organization Settings**:
- Subscriber-Admin: Can set default privacy settings for organization
  - Require parental consent verification
  - Disable public leaderboards
  - Control data retention policies

**System-Wide Settings**:
- System Admin only: Can configure platform-wide privacy settings
  - Enable/disable features with privacy implications
  - Configure data retention schedules
  - Manage Data Processing Agreements with vendors

### Audit and Compliance

**View Privacy Logs**:
- System Admin: Full access to privacy event logs
- Subscriber-Admin: Access to logs for their organization
- External Auditors: Read-only access with authorization

**Generate Compliance Reports**:
- System Admin: Can generate COPPA, FERPA, GDPR compliance reports
- Subscriber-Admin: Can generate reports for their organization
- Reports include: Consent status, data access logs, DSAR fulfillment

**Manage Vendor Agreements**:
- System Admin only: Can view and update Data Processing Agreements
- Can onboard new vendors with privacy review
- Can audit vendor compliance with DPAs

### Special Cases

**Impersonation for Support**:
- System Admin: Can impersonate users for support purposes
- Requires: Explicit permission from user or legitimate support need
- All actions logged: Full audit trail of impersonation session
- Cannot access: Passwords, payment information (handled by Braintree)
- See [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md)

**Law Enforcement Requests**:
- System Admin: Receives and processes legal requests for user data
- Requires: Valid court order, warrant, or subpoena
- Process: Legal review before disclosure
- Notification: Users notified unless prohibited by law

**Emergency Situations**:
- System Admin: Can access data without normal authorization in emergencies
- Examples: Suicide prevention, child safety, imminent harm
- Requires: Documentation and post-incident review

## Supporting Documents Referenced

This privacy policy outline specification draws from the following source documents:

- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Original privacy policy language, terms and conditions, and user data collection practices
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role-based data access rights and permissions for different user types
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription data management and organizational hierarchy information
- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Platform mission emphasizing child safety and educational data protection

## Dependencies

The privacy policy outline relies on and references the following specifications and systems:

### Legal and Compliance Documents

**Primary References**:
- [Privacy Policy](./privacy-policy.md) - Current published privacy policy (this outline guides its creation)
- [Terms and Conditions](./terms-conditions.md) - Complementary legal terms, user responsibilities
- [COPPA/FERPA Notes](./coppa-ferpa-notes.md) - Detailed compliance requirements for children and educational records
- [GDPR Compliance](./gdpr-compliance.md) - EU data protection requirements and implementation
- [FERPA Compliance](./ferpa-compliance.md) - Educational records protection for US institutions
- [Copyright and Licensing](./copyright-and-licensing.md) - Intellectual property and content licensing

### Architecture and Security

**Technical Implementation**:
- [Security and Privacy](../18-architecture/security-privacy.md) - Technical security controls, encryption, data protection measures
- [Data Model ERD](../18-architecture/data-model-erd.md) - Entity relationships, data fields, sensitive data identification
- [Config and Secrets](../18-architecture/config-and-secrets.md) - Secure configuration and credential management
- [Caching Strategy](../18-architecture/caching-strategy.md) - Data caching and privacy implications
- [API Design Principles](../18-architecture/api-design-principles.md) - API security and data access patterns
- [Webhooks](../18-architecture/webhooks.md) - Third-party data sharing via webhooks

### Roles and Permissions

**Access Control**:
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions and capabilities
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permission specifications
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Authentication and session security
- [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md) - Support access controls and audit logging

### Analytics and Telemetry

**Event Tracking**:
- [Event Model](../15-analytics-and-reporting/event-model.md) - Privacy event definitions and tracking
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Reporting with student data
- [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md) - Administrative reporting and analytics
- [SysAdmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md) - System monitoring and privacy event dashboards

### User Experience

**User-Facing Specifications**:
- [Personas](../00-foundations/personas.md) - User types and their data needs
- [Copy Guidelines](../22-content-style-guides/copy-guidelines.md) - Plain language for privacy communications
- [Error Messages](../22-content-style-guides/error-messages.md) - Privacy-related error messages
- [Student Dashboard](../03-student-experience/student-dashboard.md) - Student data visibility
- [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) - Teacher access to student data
- [Subscriber Dashboard](../05-admin-subscriber-experience/subscriber-dashboard.md) - Admin data access

### Integrations

**Third-Party Services**:
- [Email Provider](../17-integrations/email-provider.md) - Email handling and data sharing
- [CDN and Media](../17-integrations/cdn-and-media.md) - Content delivery and user data
- [LMS/LTI Spec](../17-integrations/lms-ltispec-future.md) - Data sharing with external LMS platforms
- [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) - Braintree payment data handling

### Data Management

**Data Operations**:
- [Data Migration](../20-imports-and-automation/data-migration.md) - Legacy data handling and privacy preservation
- [CSV Specifications](../20-imports-and-automation/csv-specifications.md) - Bulk import data requirements
- [Export Specs](../20-imports-and-automation/export-specs.md) - User data export formats
- [Background Jobs](../18-architecture/background-jobs.md) - Automated data retention and deletion jobs

### Notifications and Messaging

**Communication Channels**:
- [Email Templates](../13-messaging-and-notifications/email-templates.md) - Privacy notification templates
- [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md) - Privacy update notifications
- [System Announcements](../13-messaging-and-notifications/system-announcements.md) - Privacy policy change announcements

### Quality and Operations

**Operational Support**:
- [Incident Response](../19-quality-and-operations/incident-response.md) - Data breach response procedures
- [Observability](../19-quality-and-operations/observability.md) - Privacy event monitoring
- [Test Strategy](../19-quality-and-operations/test-strategy.md) - Privacy feature testing requirements

### External Systems

**Third-Party Dependencies**:
- **Vercel** (Hosting Platform): Session data, logs, deployment
- **Neon** (PostgreSQL Database): All user data storage with encryption
- **Cloudflare** (CDN/Security): Content delivery, DDoS protection, WAF
- **Braintree** (Payment Processing): Payment card data (PCI DSS compliant)
- **Email Provider** (TBD): Email delivery and tracking
- **Analytics Service** (if used): Anonymized usage data

### Regulatory Requirements

**External Regulations**:
- **COPPA** (Children's Online Privacy Protection Act): Children under 13 data protection
- **FERPA** (Family Educational Rights and Privacy Act): Educational records protection
- **GDPR** (General Data Protection Regulation): EU user data rights
- **CCPA/CPRA** (California Consumer Privacy Act): California resident rights
- **PCI DSS** (Payment Card Industry Data Security Standard): Payment data security
- **State Privacy Laws**: Various US state privacy regulations

### Version Control

**Policy Management**:
- Privacy policy versioning system (Git-based or CMS)
- Change log and audit trail
- User notification system
- Archive of previous policy versions

## Open Questions

Outstanding decisions to be resolved before finalizing the privacy policy:

### Legal and Regulatory

**Parental Consent Mechanism**:
- Should we implement email verification for all parental consents, or allow teachers to certify they have obtained consent without verification?
- What documentation should teachers maintain to prove parental consent was obtained?
- How frequently should parental consent be re-verified (annually, every 2 years, never)?

**Generic Student Accounts**:
- Should there be any limitations on Generic Student accounts beyond no PII collection?
- How do we ensure Generic Student accounts are only used in appropriate classroom settings?
- Should we require institutional verification before allowing Generic Student accounts?

**Data Retention for Cancelled Accounts**:
- Is 13 months the optimal retention period for cancelled subscriptions?
- Should different user roles have different retention periods?
- How do we handle requests to extend retention for institutional archival purposes?

**International Data Transfers**:
- Do we need to implement specific mechanisms for EU data residency (store EU user data in EU)?
- What Standard Contractual Clauses should be used with vendors?
- Should we offer opt-in for international users who want data stored in specific regions?

### Technical Implementation

**Privacy Dashboard Features**:
- What level of detail should users see in "My Data" view (raw database records vs. formatted summary)?
- Should data export include all historical records or just current data?
- How should we present data retention timelines to users?

**Consent Tracking**:
- Should consent be version-specific (require re-consent for each policy version change)?
- How granular should consent tracking be (single checkbox vs. separate consent for different purposes)?
- Should we track device/browser fingerprint in addition to IP address for consent records?

**Anonymization and Aggregation**:
- What k-anonymity threshold should be used for aggregate reporting?
- Should we implement differential privacy for research data?
- At what level should data be anonymized for different reporting contexts?

**Data Breach Notification**:
- What threshold constitutes a breach requiring user notification (1 user? 100 users? any breach)?
- Should we implement automated breach detection and notification workflows?
- What information should be included in breach notifications to users?

### Operational Considerations

**DSAR Processing**:
- Should DSARs be fully automated or require manual review?
- How should we handle complex DSARs that span multiple systems?
- Should there be a fee for excessive or repeated DSARs?

**Parent Account Implementation**:
- Should we implement dedicated parent accounts in MVP or future phase?
- If implemented, what additional privacy features should parents have?
- How do we handle custody situations where multiple parents may have rights?

**Third-Party Integrations**:
- What privacy requirements should be mandatory for future third-party integrations?
- Should we implement a privacy impact assessment (PIA) process for new integrations?
- How do we ensure LTI/LMS integrations comply with our privacy commitments?

**Marketing and Communications**:
- Should marketing emails be opt-in or opt-out?
- Can we send educational content emails without explicit consent?
- How do we handle communication preferences for institutional subscriptions?

### User Experience

**Privacy Policy Accessibility**:
- Should we create separate student-friendly and parent-friendly versions of the privacy policy?
- Should we implement video explanations of key privacy practices?
- What reading level should we target for the main privacy policy?

**Consent Fatigue**:
- How do we balance explicit consent requirements with user experience?
- Should we implement a "privacy preferences center" vs. point-in-time consent flows?
- How many separate consents is too many during account creation?

**Transparency Features**:
- Should we implement a "data activity log" showing users what data was accessed and by whom?
- Should we show real-time notifications when teacher/admin accesses student data?
- How much visibility into data processing should we provide?

### Compliance Strategy

**Certification and Auditing**:
- Should we pursue SOC 2 Type II certification for the platform?
- What third-party privacy certifications should we obtain (Privacy Shield replacement, etc.)?
- How frequently should we conduct independent privacy audits?

**Privacy by Design**:
- Should we establish a formal Privacy Impact Assessment (PIA) process for new features?
- What privacy review checklist should be used during development?
- Who should be responsible for privacy review and approval?

**Vendor Management**:
- What privacy requirements should be included in all vendor contracts?
- How should we audit vendor compliance with privacy commitments?
- Should we establish an approved vendor list for data processing?

### Special Situations

**Homeschool Users**:
- Do homeschool parents need to provide parental consent for their own children?
- Should homeschool subscriptions have different privacy requirements?
- How do we verify that a subscriber is the parent of students in the account?

**Student Age Verification**:
- Should we collect date of birth for all students to determine if under 13?
- Can we rely on teacher certification that students are over/under 13?
- How do we handle students who turn 13 during their subscription?

**Data Portability Format**:
- What format(s) should be supported for data export (JSON, CSV, PDF, all three)?
- Should we implement a standardized education data format (e.g., xAPI, Caliper)?
- How do we ensure exported data is usable by users without technical expertise?

**Right to Erasure Exceptions**:
- What specific data must be retained despite deletion requests?
- How do we balance erasure requests with legitimate interests (fraud prevention, legal obligations)?
- Should we implement "anonymization" as an alternative to full deletion?

### Future Enhancements

**Advanced Privacy Features**:
- Should we implement end-to-end encryption for student messages (future feature)?
- Should we offer anonymous gameplay modes for sensitive topics?
- Should we implement privacy-preserving analytics (differential privacy)?

**AI and Machine Learning**:
- What privacy protections are needed for AI-powered features (future)?
- Should AI training data be opt-in or opt-out?
- How do we ensure student data is not used for AI training without consent?

**Blockchain and Verifiable Credentials**:
- Should we explore blockchain-based credential verification for achievements?
- What privacy implications does blockchain have (immutability concerns)?
- Is blockchain appropriate for educational records given FERPA requirements?

---

## Next Steps

Before finalizing the privacy policy:

1. **Legal Review**: Engage qualified legal counsel specializing in educational technology and children's privacy
2. **Stakeholder Input**: Gather feedback from teachers, admins, and parents on privacy expectations
3. **Regulatory Consultation**: Consider consultation with FTC (COPPA) and Department of Education (FERPA)
4. **Technical Validation**: Confirm all described privacy practices are technically feasible
5. **Plain Language Review**: Test policy readability with representative users
6. **Vendor Verification**: Confirm all vendor privacy practices match what's described in policy
7. **Translation Planning**: If supporting multiple languages, plan for translation and legal review
8. **Version Control Setup**: Establish system for managing policy versions and change notifications
9. **Training Materials**: Develop training for system admins on privacy policy enforcement
10. **Launch Communications**: Plan announcement and education campaign for privacy policy launch
