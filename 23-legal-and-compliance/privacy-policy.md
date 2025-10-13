# Privacy Policy

## Overview
This document serves as the comprehensive, legally binding privacy policy for the Music Learning Community (MLC-LMS) platform. It defines how user data is collected, used, stored, protected, and deleted, ensuring compliance with all applicable privacy laws and regulations including COPPA, FERPA, GDPR, and CCPA.

## Purpose

This privacy policy:
- **Informs Users**: Provides clear, transparent information about data practices
- **Ensures Compliance**: Meets requirements of COPPA, FERPA, GDPR, CCPA, and other privacy laws
- **Establishes Trust**: Demonstrates commitment to protecting user privacy, especially children's data
- **Defines Rights**: Clarifies user rights regarding their personal data
- **Sets Expectations**: Explains what data is collected and how it will be used

**IMPORTANT**: This privacy policy applies to all users of the MLC-LMS platform, including students, teachers, parents, administrators, and system administrators. For children under 13, special protections apply under COPPA.

**For Technical Implementation**: See [Privacy Policy Outline](./privacy-policy-outline.md) for detailed specifications and [Security and Privacy Architecture](../18-architecture/security-privacy.md) for technical controls.

---

**By Terry Treble**  
Music Learning Community  
**Effective Date: [TBD - MLC 5.0 Launch]**  
**Previous Version: March 27, 2021 (MLC 4.0)**  
**Last Updated: [TBD]**

---

## Table of Contents

1. [What Information We Collect](#what-information-do-we-collect)
2. [How We Use Your Information](#what-do-we-use-your-information-for)
3. [Information Sharing and Disclosure](#do-we-disclose-any-information-to-outside-parties)
4. [Data Security](#how-do-we-protect-your-information)
5. [Data Retention and Deletion](#data-retention-and-deletion)
6. [Your Rights and Choices](#your-rights-and-choices)
7. [Children's Privacy (COPPA)](#childrens-online-privacy-protection-act-compliance)
8. [Educational Records (FERPA)](#educational-records-ferpa)
9. [International Users](#international-users)
10. [Cookies and Tracking](#do-we-use-cookies)
11. [California Privacy Rights (CCPA)](#california-online-privacy-protection-act-compliance)
12. [Changes to This Policy](#changes-to-our-privacy-policy)
13. [Contact Us](#contacting-us)

---

## What information do we collect?

We collect different types of information depending on your role and how you use the platform. We follow the principle of data minimization: we only collect information necessary to provide our educational services.

### Information You Provide to Us

**When You Create an Account:**
- Name (first and last)
- Username
- Email address
- Password (stored as encrypted hash, never in plain text)
- Organization affiliation (school, studio name)
- Phone number (optional for teachers and administrators)
- Billing address (for paid subscriptions)

**When You Subscribe:**
- Subscription plan selection (PRELUDE trial, SOLO, or ENSEMBLE)
- Number of student seats required
- Payment information (processed securely by Braintree, our payment processor)
- Billing preferences (monthly or annual)

**When You Use the Platform:**
- Student enrollment information (names, usernames)
- Class and group assignments
- Messages sent through the platform
- Custom assignment creation
- Profile customization
- Support requests and communications

**Special Note on Student Data:**
- **For students under 13**: We collect only the minimum necessary information (name and username required). Email addresses are optional and require verifiable parental consent per COPPA.
- **Generic Student accounts**: For classroom settings where individual tracking is not needed, we offer Generic Student accounts that collect no personally identifiable information.

### Information We Collect Automatically

**When You Use the Platform:**
- **Game Play Data**: Scores, completion times, number of attempts, stages completed
- **Learning Progress**: Assignment completion, mastery levels, retention tracking
- **Achievement Data**: Badges earned, points accumulated, leaderboard standings (with parental consent for students under 13)
- **Session Information**: Login times, session duration, IP address, device type
- **Technical Data**: Browser type and version, operating system, screen resolution
- **Usage Analytics**: Pages visited, features used, navigation patterns (NOT used on student-facing pages for children under 13)
- **Performance Metrics**: Page load times, error logs (no personal information in logs)

### Information from Third Parties

**Payment Processor (Braintree):**
- Transaction confirmation
- Payment status updates
- Refund processing (we do not store credit card numbers)

**School or Institution Data:**
- For institutional subscriptions, we may receive roster information from schools
- Organization hierarchy and structure

### Information We Do NOT Collect

To protect your privacy, we do NOT collect:
- Social Security Numbers
- Driver's license numbers
- Student home addresses (unless required for billing)
- Student phone numbers
- Precise geolocation beyond general location (city/state from IP)
- Behavioral tracking data across non-MLC websites
- Biometric or health information

**Anonymous Browsing:**
You may visit our public pages, play sample games, and explore our Help Center without creating an account.

---

## What do we use your information for?

We use your information exclusively for educational purposes and to provide our services. Here's how:

### Educational Purposes

**Deliver Learning Content:**
- Provide access to educational games and resources
- Track learning progress and mastery
- Generate personalized assignments based on student level
- Deliver curriculum-aligned sequences
- Award badges and achievements for motivation

**Support Learning:**
- Enable teachers to monitor student progress
- Generate reports showing strengths and areas for improvement
- Provide feedback to students on performance
- Support differentiated instruction
- Track retention and schedule review games

### Account Management

**Create and Maintain Accounts:**
- Authenticate users and maintain secure sessions
- Enable role-based access (students, teachers, admins)
- Manage class rosters and student-teacher relationships
- Maintain subscription and seat allocation

**Provide Customer Support:**
- Respond to support requests and troubleshoot issues
- Assist with account recovery and password resets
- Provide guidance on platform features
- Resolve billing and subscription questions

### Platform Operations

**Improve Our Services:**
- Analyze usage patterns to improve game effectiveness
- Identify and fix technical issues
- Optimize platform performance and loading times
- Develop new features based on user needs
- Conduct educational research with anonymized data

**Ensure Security:**
- Detect and prevent fraud and abuse
- Monitor for unauthorized access
- Protect against security threats
- Maintain audit logs for compliance

### Communication

**Send Important Notifications:**
- Assignment notifications to students
- Progress updates to teachers and parents
- Subscription and billing information
- Platform updates and maintenance notices
- Security alerts if needed

**Optional Marketing Communications:**
- Newsletters with educational tips and platform updates
- New feature announcements
- Special offers and promotions
- You can opt out at any time using the unsubscribe link in every email

### Billing and Compliance

**Process Payments:**
- Manage subscriptions and billing cycles
- Process seat changes and prorations
- Generate invoices and billing statements
- Handle refunds when applicable

**Meet Legal Obligations:**
- Comply with COPPA, FERPA, GDPR, and other privacy laws
- Respond to legal requests (with valid court orders)
- Maintain records for tax and financial audits
- Enforce our Terms of Service

---

## What We Do NOT Do With Your Information

**Your information, whether public or private, will NOT be:**
- **Sold to third parties** for any reason
- **Exchanged or transferred** to outside companies for marketing
- **Used for advertising purposes** (no ads on student pages)
- **Tracked across other websites** for behavioral profiling
- **Used for any non-educational purposes**

This commitment is especially important for student data. We exist to support music education, not to profit from student information.

---

## How do we protect your information?

We implement industry-standard security measures to protect your personal information. Your data security is one of our highest priorities.

### Encryption

**Data in Transit:**
- All data transmitted between your device and our servers is encrypted using TLS 1.3 (Transport Layer Security)
- This includes login credentials, payment information, and all personal data
- We enforce HTTPS on all pages (you'll see the lock icon in your browser)

**Data at Rest:**
- All data stored in our database is encrypted using AES-256 encryption
- Passwords are never stored in plain text - we use industry-standard bcrypt hashing
- Backup data is also fully encrypted

**Payment Security:**
- Credit card information is processed by Braintree (a PayPal service), a PCI DSS Level 1 certified payment processor
- We never store credit card numbers on our servers
- Payment data is encrypted and tokenized by Braintree

### Access Controls

**Role-Based Permissions:**
- Users can only access data appropriate to their role
- Teachers see only their own students' data
- Students see only their own progress and scores
- Administrators have limited access based on their organization

**Authentication:**
- Secure password requirements
- Multi-factor authentication (MFA) available for admin accounts
- Session timeouts to prevent unauthorized access
- Account lockout after repeated failed login attempts

**Audit Logging:**
- All access to sensitive data is logged
- System administrators' actions are tracked
- Support access to user accounts is fully logged and limited

### Infrastructure Security

**Secure Hosting:**
- Platform hosted on Vercel, a SOC 2 Type II certified provider
- Database hosted on Neon, with automatic encryption and backups
- Content delivery via Cloudflare, providing DDoS protection and Web Application Firewall (WAF)

**Regular Security Testing:**
- Automated vulnerability scanning
- Regular security updates and patches
- Dependency scanning for known vulnerabilities
- Annual third-party security audits (planned)

**Network Security:**
- Firewall protection at multiple layers
- DDoS protection via Cloudflare
- Rate limiting to prevent abuse
- IP-based access controls for administrative functions

### Vendor Security

All third-party vendors who handle your data must:
- Sign Data Processing Agreements
- Maintain SOC 2 or equivalent certification
- Use encryption for data at rest and in transit
- Comply with applicable privacy laws
- Undergo regular security audits

**Our key vendors:**
- **Vercel** (Hosting): SOC 2 Type II, ISO 27001
- **Neon** (Database): SOC 2 Type II, encrypted backups
- **Cloudflare** (CDN/Security): SOC 2, ISO 27001, GDPR compliant
- **Braintree** (Payments): PCI DSS Level 1 certified

### Data Breach Response

In the unlikely event of a data breach:
- We will investigate and contain the breach immediately
- Affected users will be notified as required by law
- We will report to regulatory authorities as required (COPPA, FERPA, GDPR)
- We will provide guidance on steps you can take to protect yourself

**No system is 100% secure.** While we implement strong security measures, we cannot guarantee absolute security. We commit to transparency and prompt action if security issues arise.

For detailed technical security specifications, see our [Security and Privacy Architecture](../18-architecture/security-privacy.md) document.

---

## Do we use cookies?

We use cookies and similar technologies only when necessary for the platform to function. We do NOT use advertising or tracking cookies.

### Essential Cookies (Required)

These cookies are necessary for the platform to work:

**Session Cookies:**
- Keep you logged in as you navigate the platform
- Remember your role and permissions
- Maintain security (CSRF protection)
- Expire when you close your browser or log out

**Security Cookies:**
- Protect against cross-site request forgery attacks
- Detect suspicious login attempts
- Prevent unauthorized access

### Functional Cookies (Optional)

These cookies enhance your experience:

**Preference Cookies:**
- Remember your language preference
- Save your dashboard layout choices
- Remember notification settings
- You can disable these in your browser settings

### What We Do NOT Use

**No Advertising Cookies:**
- We do not use cookies for advertising
- We do not participate in ad networks
- We do not sell cookie data to third parties

**No Student Tracking:**
- We do not use behavioral tracking cookies on student-facing pages
- We do not track students across other websites
- Student game play data is collected for educational purposes only, not marketing

### Managing Cookies

**Browser Controls:**
- You can disable cookies in your browser settings
- Note: Disabling essential cookies will prevent you from logging in
- Disabling functional cookies may limit some features

**Mobile Apps (Future):**
- Mobile apps use similar technology (session tokens, not traditional cookies)
- Permissions managed through device settings

For more information about cookies and tracking, see our technical documentation on [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md).

---

## Do we disclose any information to outside parties?

**We do not sell, trade, or otherwise transfer your personally identifiable information to outside parties.** This is our fundamental commitment to you, especially regarding student data.

However, we do share information in specific, limited circumstances:

### With Your Consent

**Teachers and Administrators:**
- Teachers can view progress data for students in their classes (authorized by the subscriber)
- Administrators can view data for users in their organization
- Parents can view their own children's data (when parent accounts are implemented)

**Public Leaderboards:**
- Students can opt to participate in public leaderboards
- Requires parental consent for students under 13
- Can be disabled by parents or administrators at any time

### Service Providers (Data Processors)

We share data with trusted third-party vendors who help us operate the platform. All vendors must:
- Sign Data Processing Agreements
- Use data only for specified purposes
- Implement appropriate security measures
- Comply with applicable privacy laws (COPPA, FERPA, GDPR)

**Our Service Providers:**

**Vercel (Hosting Platform)**
- **Data Shared**: Session data, application logs, performance metrics
- **Purpose**: Host and deliver the platform
- **Security**: SOC 2 Type II, ISO 27001 certified
- **Location**: United States
- **DPA**: Signed Data Processing Agreement

**Neon (Database)**
- **Data Shared**: All user data (encrypted)
- **Purpose**: Secure data storage and retrieval
- **Security**: SOC 2 Type II, AES-256 encryption
- **Location**: United States
- **DPA**: Signed Data Processing Agreement

**Cloudflare (CDN and Security)**
- **Data Shared**: Access logs, cached content
- **Purpose**: Content delivery, DDoS protection, Web Application Firewall
- **Security**: SOC 2, ISO 27001, GDPR compliant
- **Location**: Global CDN with data centers worldwide
- **DPA**: Signed Data Processing Agreement

**Braintree (Payment Processing)**
- **Data Shared**: Billing information, payment card data
- **Purpose**: Secure payment processing
- **Security**: PCI DSS Level 1 certified (highest level)
- **Location**: United States
- **DPA**: Signed Data Processing Agreement
- **Note**: We do NOT store credit card numbers; Braintree handles all payment data

**Email Provider (TBD)**
- **Data Shared**: Email addresses, notification content
- **Purpose**: Send transactional emails and notifications
- **Security**: SOC 2 certified, DKIM/SPF/DMARC compliant
- **DPA**: Will be signed before implementation

For detailed vendor specifications, see [Email Provider](../17-integrations/email-provider.md) and [CDN and Media](../17-integrations/cdn-and-media.md).

### Legal Requirements

We may disclose information when required by law:

**Court Orders and Legal Process:**
- Valid subpoenas, court orders, or search warrants
- Compliance with legal obligations
- Users will be notified unless prohibited by law

**Safety and Protection:**
- Protect the rights, property, or safety of Music Learning Community, our users, or others
- Enforce our Terms of Service
- Investigate and prevent fraud, security issues, or technical problems
- Respond to emergency situations involving danger of death or serious physical injury

**Regulatory Compliance:**
- COPPA compliance reporting to the FTC (if required)
- FERPA disclosure to educational authorities (as permitted)
- GDPR reporting to supervisory authorities (if required)
- State breach notification laws

### Business Transfers

In the event of a merger, acquisition, or sale of assets:
- User data may be transferred to the acquiring entity
- Users will be notified via email and platform announcement
- The new entity must honor this privacy policy
- Users will have the option to delete their accounts before the transfer

### Educational Institutions

For institutional subscriptions:
- Schools may access aggregate usage data for their organization
- Individual student data shared only with authorized teachers and administrators
- Compliant with FERPA requirements for educational records

**FERPA Compliance:**
See our [FERPA Compliance](./ferpa-compliance.md) framework for detailed information about educational records protection.

### What We Do NOT Share

**We will NEVER:**
- Sell student data to data brokers or advertisers
- Share data with marketing companies
- Use student data for targeted advertising
- Share data with unauthorized third parties
- Disclose data without legal basis or your consent

---

## Data Retention and Deletion

We retain your data only as long as necessary to provide our services and comply with legal obligations.

### Retention Periods

**Active Subscriptions:**
- Data is retained while your subscription is active
- You have full access to view, export, and manage your data
- Data is backed up daily with encrypted backups

**Cancelled Subscriptions:**
- Data retained for **13 months** after cancellation
- Allows for subscription reactivation without data loss
- After 13 months, data is automatically deleted
- Teachers and students can be reactivated if subscription is renewed within this period

**Specific Data Types:**
- **Financial Records**: Retained for 7 years (tax/audit requirements)
- **Support Tickets**: Retained for 3 years
- **Security/Audit Logs**: Retained for 1 year minimum
- **Encrypted Backups**: Retained for 90 days, then permanently deleted

### Automated Deletion

**Background Jobs:**
- Daily automated process identifies expired data
- Soft delete (30-day grace period) then permanent deletion
- Cascading deletion of related records (scores, assignments, sessions)
- Backup rotation removes deleted users

For technical details, see [Background Jobs](../18-architecture/background-jobs.md) and [Data Migration](../20-imports-and-automation/data-migration.md).

### User-Requested Deletion

You can request account deletion at any time:

**How to Request:**
1. Log in to your account settings
2. Navigate to Privacy & Data section
3. Click "Delete My Account"
4. Confirm your identity
5. Confirm deletion (this is permanent)

**Processing Timeline:**
- Request acknowledged within 5 business days
- 30-day grace period to cancel deletion
- Permanent deletion completed within 60 days total

**What Gets Deleted:**
- Your account and login credentials
- Personal information (name, email, contact info)
- Profile data and preferences
- Messages and communications
- For students: game scores, progress data, achievements
- For teachers: class rosters, assignments, reports

**What May Be Retained:**
- Financial transaction records (7 years - legal requirement)
- Anonymized, aggregated data for research
- Data required for ongoing legal matters
- Security logs (anonymized)

**Special Cases:**
- **Parents** can request deletion of their children's data (for students under 13)
- **Subscriber-Admins** can delete accounts within their organization
- Deletion requests honored even for cancelled subscriptions

For GDPR "Right to Erasure" details, see [GDPR Compliance](./gdpr-compliance.md).

---

## Your Rights and Choices

You have significant rights regarding your personal data. Here's what you can do:

### All Users

**Access Your Data:**
- View all personal information we have about you
- Access through account settings → Privacy & Data
- See data collection history and usage

**Export Your Data:**
- Download your data in machine-readable format (JSON or CSV)
- Includes all personal data, scores, progress, and account history
- Request through account settings or email privacy@musiclearningcommunity.com
- Delivered within 24-48 hours

**Correct Your Data:**
- Update inaccurate or incomplete information
- Change name, email, contact information through account settings
- Request corrections by emailing privacy@musiclearningcommunity.com

**Delete Your Account:**
- Request permanent deletion of your account and data
- See "Data Retention and Deletion" section above

**Opt Out of Communications:**
- Unsubscribe from marketing emails (link in every email)
- Manage notification preferences in account settings
- Transactional emails (receipts, account security) cannot be disabled

### Students Under 13 (COPPA Rights)

**Parental Rights:**
- **Review**: Parents can review their child's personal information
- **Delete**: Parents can request deletion of their child's data
- **Refuse Collection**: Parents can refuse further data collection (may limit functionality)
- **Consent Management**: Parents control what data is collected

**How Parents Exercise Rights:**
- Contact the teacher or administrator who created the account
- Email coppa@musiclearningcommunity.com
- Provide verification of parental relationship
- Request processed within 30 days

For complete COPPA requirements, see [COPPA/FERPA Notes](./coppa-ferpa-notes.md).

### EU Users (GDPR Rights)

European Union users have additional rights:

**Right to Access:**
- Request copy of all personal data we hold
- Includes data categories, purposes, recipients

**Right to Rectification:**
- Correct inaccurate or incomplete data

**Right to Erasure ("Right to be Forgotten"):**
- Request deletion of your data
- Exceptions for legal obligations

**Right to Data Portability:**
- Receive your data in structured, commonly used format
- Transfer data to another service

**Right to Restrict Processing:**
- Temporarily limit how we use your data

**Right to Object:**
- Object to specific data processing activities
- Opt out of direct marketing

**Right to Withdraw Consent:**
- Withdraw consent at any time (doesn't affect prior processing)

**Right to Lodge a Complaint:**
- File complaint with EU supervisory authority

For complete GDPR implementation, see [GDPR Compliance](./gdpr-compliance.md).

### California Residents (CCPA/CPRA Rights)

California residents have specific rights:

**Know What Data is Collected:**
- Categories of personal information
- Sources of data
- Business purposes for collection

**Access Your Data:**
- Request copy of personal information collected
- Up to twice per year, free of charge

**Delete Your Data:**
- Request deletion of personal information
- Subject to certain exceptions

**Opt Out of "Sale":**
- We do NOT sell personal data, so opt-out not applicable
- But you have the right to opt out if we ever did

**Non-Discrimination:**
- We will not discriminate for exercising your privacy rights
- No denial of service, different prices, or reduced quality

See "California Privacy Rights" section below for more details.

### How to Exercise Your Rights

**Self-Service (Recommended):**
- Log in → Account Settings → Privacy & Data
- Most functions available self-service

**Email Request:**
- Privacy inquiries: privacy@musiclearningcommunity.com
- COPPA inquiries: coppa@musiclearningcommunity.com
- GDPR inquiries: dpo@musiclearningcommunity.com

**Written Request:**
Music Learning Community  
Attn: Privacy Officer  
[Street Address]  
Wildwood, Missouri 63011  
United States

**Response Timeline:**
- Acknowledgment within 5 business days
- Fulfillment within 30 days (45 days for complex requests)
- Identity verification required before processing

For technical specifications of data access features, see [Export Specs](../20-imports-and-automation/export-specs.md).

---

## Children's Online Privacy Protection Act Compliance

Protecting children's privacy is one of our highest priorities. We are fully compliant with COPPA (Children's Online Privacy Protection Act).

### Our Commitment to Children Under 13

**Platform Design:**
- The platform is designed for ages 6-18, with special protections for children under 13
- We collect only the minimum information necessary for educational purposes
- No behavioral tracking or advertising on student-facing pages
- No selling or marketing of children's data

### What Information We Collect from Children Under 13

**Required Information (Minimum Necessary):**
- Student first and last name
- Username (for login)
- Class/teacher assignment

**Optional Information (Requires Parental Consent):**
- Student email address (if parent wants to receive notifications)
- Date of birth (for age-appropriate content)

**Automatically Collected (Educational Purpose Only):**
- Game scores and completion data
- Learning progress and mastery levels
- Assignment completion status
- Login times and session data

**NOT Collected from Children:**
- Home address or phone number
- Photos or videos
- Geolocation data
- Social Security Number
- Any non-educational personal information

### Verifiable Parental Consent

**How Consent Works:**
1. Teacher or administrator creates student account
2. Teacher certifies they have obtained parental consent
3. If student email provided, verification email sent to parent
4. Parent must click verification link to confirm consent
5. Consent record stored with timestamp

**Generic Student Alternative:**
- For classroom settings, we offer Generic Student accounts
- These collect NO personally identifiable information
- Suitable for general music classes without individual tracking

### Parental Rights Under COPPA

**Parents Can:**
- **Review**: Access their child's personal information
- **Delete**: Request deletion of their child's data at any time
- **Refuse**: Refuse further collection (may limit platform functionality)
- **Consent**: Provide or withhold consent for optional data collection

**How to Exercise Parental Rights:**
- Email: coppa@musiclearningcommunity.com
- Phone: 855-687-4240
- Provide: Child's name, username, school/teacher information
- Verification: We'll verify parental relationship before providing access

**Response Time:** Within 30 days of verified request

### Data Security for Children

- All student data encrypted at rest and in transit
- Access limited to authorized teachers and administrators only
- No third-party advertising or tracking on student pages
- Regular security audits and monitoring

### Teacher and School Responsibilities

**Teachers Must:**
- Obtain verifiable parental consent before creating accounts for students under 13
- Maintain documentation of parental consent
- Use student data only for educational purposes
- Notify parents of data collection practices

For complete COPPA compliance specifications, see [COPPA/FERPA Notes](./coppa-ferpa-notes.md).

---

## Educational Records (FERPA)

For educational institutions, we comply with the Family Educational Rights and Privacy Act (FERPA).

### FERPA Protections

**What FERPA Covers:**
- Student educational records (scores, assignments, progress)
- Personally identifiable information in educational context
- Records maintained by educational institutions

**Our FERPA Compliance:**
- Student educational records shared only with authorized school personnel
- Parents/eligible students can inspect and review records
- Written consent required for most disclosures
- Annual notification of FERPA rights to schools

### School Official Exception

Under FERPA, we act as a "school official" with legitimate educational interests:
- Process student data on behalf of the school
- Use data solely for educational purposes specified by the school
- Maintain appropriate security safeguards
- Destroy or return data when no longer needed for the authorized purpose

### Parent and Student Rights Under FERPA

**For K-12 Students:**
- Parents have right to inspect and review educational records
- Parents can request amendment of inaccurate records
- Parents must consent to most disclosures of educational records

**For College Students (18+):**
- Rights transfer to the student (eligible student)
- Student controls access to their educational records

### Permitted Disclosures Under FERPA

We may disclose educational records without consent when:
- Required by teachers with legitimate educational interest
- Requested by school officials
- Required for financial aid determination
- Mandated by law (court order, subpoena)
- Necessary for health or safety emergency

For comprehensive FERPA implementation, see [FERPA Compliance](./ferpa-compliance.md).

---

## International Users

We welcome users from around the world and respect international privacy laws.

### Data Location and Transfers

**Primary Data Storage:**
- Data stored on servers located in the United States
- Neon database (PostgreSQL) hosted in US data centers
- Backups also stored in United States

**Content Delivery:**
- Cloudflare CDN delivers content from edge locations worldwide
- Improves performance for international users
- Does not store personal data, only cached public content

### International Data Transfers

**European Union Users:**
- Data transferred from EU to US under appropriate safeguards
- Standard Contractual Clauses (SCCs) with data processors
- GDPR rights fully respected (see GDPR section)
- Contact: dpo@musiclearningcommunity.com

**Other International Users:**
- We respect privacy laws of your jurisdiction
- Contact us if you have specific privacy requirements
- May implement region-specific controls as needed

### GDPR Compliance for EU Users

If you are in the European Union, you have additional rights under GDPR:

**Legal Basis for Processing:**
- **Contract**: Process data to provide services you've subscribed to
- **Consent**: Process optional data with your explicit consent
- **Legitimate Interests**: Improve platform, ensure security (balanced with your rights)
- **Legal Obligation**: Comply with applicable laws

**Data Protection Officer:**
- Email: dpo@musiclearningcommunity.com
- Handles all GDPR inquiries and data subject requests

**EU Data Protection Rights:**
See "Your Rights and Choices" section for complete list of GDPR rights.

**Supervisory Authority:**
- You have the right to lodge a complaint with your local data protection authority
- Contact information varies by EU member state

For complete GDPR implementation framework, see [GDPR Compliance](./gdpr-compliance.md).

### Canadian Users (PIPEDA)

For Canadian users:
- We comply with Personal Information Protection and Electronic Documents Act (PIPEDA)
- You have rights to access, correct, and withdraw consent for your data
- Contact privacy@musiclearningcommunity.com for PIPEDA inquiries

### Other Jurisdictions

We strive to comply with privacy laws worldwide. If you have concerns about how your jurisdiction's privacy laws apply to our service, please contact us.

---

## California Online Privacy Protection Act Compliance

### CalOPPA Compliance

Because we value your privacy, we have taken necessary precautions to comply with the California Online Privacy Protection Act (CalOPPA).

**We agree to:**
- Allow users to visit our site anonymously (public pages)
- Link to this Privacy Policy on our homepage and in account creation flow
- Notify users of privacy policy changes via email and on-site notice
- Allow users to change their personal information through account settings
- Not distribute personal information to outside parties without consent

### California Consumer Privacy Act (CCPA/CPRA)

California residents have specific rights under CCPA and CPRA:

**Categories of Personal Information Collected:**
- **Identifiers**: Name, email, username, IP address
- **Commercial Information**: Subscription details, payment history
- **Internet Activity**: Usage data, game play patterns
- **Education Information**: Student scores, progress, educational records
- **Inferences**: Learning patterns, skill levels (derived from game play)

**Business Purposes for Collection:**
- Provide educational services and platform functionality
- Process payments and manage subscriptions
- Communicate with users
- Improve platform and develop new features
- Ensure security and prevent fraud
- Comply with legal obligations

**Categories of Third Parties We Share With:**
- Service providers (Vercel, Neon, Cloudflare, Braintree)
- Payment processors
- Email service providers
- Only as described in "Information Sharing" section

**We Do NOT Sell Personal Information:**
- We have NOT sold personal information in the past 12 months
- We do NOT sell personal information of minors under 16
- We will NOT sell personal information in the future

**Your California Privacy Rights:**
- **Right to Know**: Request categories and specific pieces of personal information collected
- **Right to Delete**: Request deletion of personal information (with exceptions)
- **Right to Opt-Out**: Opt out of sale (not applicable - we don't sell data)
- **Right to Non-Discrimination**: No discrimination for exercising your rights

**How to Exercise CCPA Rights:**
- Email: privacy@musiclearningcommunity.com
- Subject line: "California Privacy Request"
- Include: Your name, email, username, specific request
- We'll verify your identity and respond within 45 days

**Authorized Agent:**
- You may designate an authorized agent to make requests on your behalf
- Agent must provide written authorization from you
- We may still require you to verify your identity directly

**CCPA Metrics (Annual Disclosure):**
We will publish annual metrics including:
- Number of access requests received and fulfilled
- Number of deletion requests received and fulfilled
- Number of opt-out requests (if applicable)
- Average response time

---

## Online Privacy Policy Only

This privacy policy applies to information collected through our website and platform (musiclearningcommunity.com and related subdomains). 

**Offline Information:**
- Information collected offline (mail, phone) is handled according to the same privacy standards
- For institutional contracts signed offline, the privacy terms of the contract apply
- We combine online and offline data to provide better service to you

**Third-Party Sites:**
- This policy does not apply to websites linked from our platform
- We are not responsible for privacy practices of external sites
- Please review privacy policies of any external sites you visit

---

## Your Consent

### Account Creation Consent

By creating an account and using our platform, you consent to:
- Collection and use of your information as described in this privacy policy
- Transfer of your information to our service providers
- Storage of your data in the United States
- Email communications related to your account and subscription

### Consent Requirements

**Account Creation:**
- You must explicitly agree to this privacy policy during signup
- Checkbox consent required (cannot be pre-checked)
- Consent timestamp and IP address recorded

**Parental Consent (Students Under 13):**
- Teachers certify they have obtained parental consent
- Optional email verification sent to parents
- Parents can withdraw consent at any time

**Optional Consents:**
- Marketing emails (separate opt-in)
- Public leaderboard participation (students)
- Data usage for research (anonymized)

### Withdrawing Consent

You can withdraw consent at any time:
- Update communication preferences in account settings
- Request account deletion (see Data Retention section)
- Contact privacy@musiclearningcommunity.com

**Note:** Withdrawing consent may limit or prevent use of the platform.

For consent tracking specifications, see [Privacy Policy Outline](./privacy-policy-outline.md#consent-mechanisms).

---

## Changes to our Privacy Policy

We may update this privacy policy from time to time to reflect changes in our practices, legal requirements, or platform features.

### How We Notify You

**Material Changes:**
- Email notification to all account holders
- Prominent banner notification when you log in
- 30-day notice before changes take effect
- For students under 13: May require renewed parental consent

**Minor Changes:**
- Updated "Last Updated" date on this page
- Changes noted in version history
- In-app notification on next login

### Version History

**Current Version:**
- Effective Date: [TBD - MLC 5.0 Launch]
- Major Updates: Complete rewrite for MLC 5.0 platform, added GDPR/CCPA compliance

**Previous Versions:**
- March 27, 2021 (MLC 4.0)
- March 25, 2012 (MLC 3.0)

**Accessing Previous Versions:**
- Visit musiclearningcommunity.com/privacy/history
- Or contact privacy@musiclearningcommunity.com

### Material Changes Requiring Re-Consent

If we make material changes that significantly affect how we collect, use, or share data, we will:
- Provide 30-day advance notice
- Require renewed consent before changes take effect
- Give you the option to decline and close your account
- For students under 13: Obtain renewed parental consent

**Material changes include:**
- Expanding data collection practices
- Sharing data with new types of third parties
- Using data for significantly different purposes
- Reducing data security or privacy protections

### Non-Material Changes

Minor clarifications or updates that don't affect your rights won't require re-consent:
- Adding new contact information
- Clarifying existing practices
- Updating vendor names or certifications
- Fixing typos or improving readability

### Staying Informed

**Subscribe to Updates:**
- Opt-in for privacy policy update notifications
- Managed separately from marketing emails
- Cannot be disabled for material changes

**Review Regularly:**
- Check the "Last Updated" date when you log in
- Review annually as good practice
- Especially important for administrators managing student data

For notification templates, see [Email Templates](../13-messaging-and-notifications/email-templates.md) and [System Announcements](../13-messaging-and-notifications/system-announcements.md).

---

## Contacting Us

We're here to answer your privacy questions and help you exercise your rights.

### General Privacy Inquiries

**Email:** privacy@musiclearningcommunity.com  
**Subject Line:** "Privacy Inquiry" or "Data Request"  
**Response Time:** Within 5 business days (within 30 days for requests)

### COPPA-Specific Questions (Children Under 13)

**Email:** coppa@musiclearningcommunity.com  
**Phone:** 855-687-4240  
**Hours:** Monday-Friday, 9 AM - 5 PM Central Time  
**For:** Parental consent, review child's data, deletion requests

### GDPR Questions (EU Users)

**Data Protection Officer:** dpo@musiclearningcommunity.com  
**For:** GDPR rights, data subject access requests, supervisory authority contacts

### California Privacy Rights (CCPA)

**Email:** privacy@musiclearningcommunity.com  
**Subject Line:** "California Privacy Request"  
**For:** CCPA rights, data access, deletion requests

### FERPA Questions (Educational Records)

**Email:** ferpa@musiclearningcommunity.com  
**For:** Educational records access, institutional compliance

### General Contact Information

**Mailing Address:**  
Music Learning Community  
Attn: Privacy Officer  
[Street Address]  
Wildwood, Missouri 63011  
United States of America

**Website:** https://www.musiclearningcommunity.com  
**Support:** Use Contact form on website  
**Phone:** 855-687-4240

**Business Hours:**  
Monday - Friday: 9:00 AM - 5:00 PM Central Time  
Saturday - Sunday: Closed (email monitored for urgent issues)

### What to Include in Your Privacy Request

**For All Requests:**
- Your full name
- Email address associated with account
- Username (if known)
- Specific request (access, deletion, correction, etc.)
- Preferred response method

**For Parental Requests:**
- Your relationship to the student
- Student's name and username
- School or teacher name
- Verification information (we'll provide instructions)

**For Institutional Requests:**
- Organization name
- Your role (admin, teacher, etc.)
- Scope of request
- Authorization documentation (if applicable)

### Response Timeline

- **Acknowledgment:** Within 5 business days
- **Simple Requests:** Within 10 business days
- **Complex Requests:** Within 30 days (45 days for CCPA)
- **Urgent Security Issues:** Within 24 hours

### Additional Resources

**Documentation:**
- [Privacy Policy Outline](./privacy-policy-outline.md) - Technical specifications
- [COPPA/FERPA Notes](./coppa-ferpa-notes.md) - Compliance details
- [GDPR Compliance](./gdpr-compliance.md) - EU user rights
- [FERPA Compliance](./ferpa-compliance.md) - Educational records
- [Security and Privacy Architecture](../18-architecture/security-privacy.md) - Technical controls

**Related Policies:**
- [Terms and Conditions](./terms-conditions.md) - User agreement
- [Copyright and Licensing](./copyright-and-licensing.md) - Content rights

**For System Administrators:**
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Access controls
- [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md) - Support procedures

---

## Supporting Documents Referenced

This privacy policy draws from the following source documents:

- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Original MLC 4.0 privacy policy language (March 27, 2021 version), terms and conditions, and data collection practices
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role-based data access rights defining what data each user type can view and manage
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription and organizational data management practices, Generic Student account specifications
- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Educational mission and commitment to protecting student data and privacy

---

## Summary of Key Points

**What We Collect:**
- Account information (name, email, username)
- Educational data (scores, progress, assignments)
- Payment information (processed by Braintree)
- Usage data (analytics, performance metrics)

**How We Use It:**
- Provide educational services
- Track learning progress
- Process payments
- Improve the platform
- Communicate with you

**How We Protect It:**
- AES-256 encryption at rest
- TLS 1.3 encryption in transit
- SOC 2 certified hosting
- Regular security audits
- Limited access controls

**Your Rights:**
- Access your data
- Export your data
- Correct inaccurate data
- Delete your account
- Opt out of marketing

**Children's Privacy:**
- COPPA compliant
- Minimum data collection
- Parental consent required for under 13
- No advertising or tracking
- Parents can review and delete

**We Do NOT:**
- Sell your data
- Use student data for advertising
- Track across other websites
- Share data with unauthorized parties

**Questions?** Contact privacy@musiclearningcommunity.com

---

**End of Privacy Policy**

*This privacy policy is effective as of [TBD] and supersedes all previous versions.*
