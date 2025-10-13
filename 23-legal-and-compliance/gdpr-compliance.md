# GDPR Compliance Framework

## Overview
General Data Protection Regulation (GDPR) compliance framework and implementation specifications for the MLC-LMS platform.

## Description
Comprehensive GDPR compliance requirements including data processing principles, user rights implementation, consent management, data breach procedures, and privacy by design specifications for European users.

---

## Purpose

This document defines the technical, operational, and legal requirements for GDPR (General Data Protection Regulation) compliance when processing personal data of users in the European Union and EEA. It provides implementation guidance for engineers, operational procedures for support staff, and policy frameworks for legal compliance.

**This specification enables:**

- **GDPR Compliance**: Technical implementation of EU data protection requirements
- **User Rights Implementation**: Systems and procedures to honor data subject rights
- **Legal Basis Documentation**: Clear legal justification for all data processing activities
- **Breach Response**: GDPR-compliant data breach notification procedures
- **Cross-Border Transfers**: Lawful mechanisms for transferring data from EU to US
- **Vendor Management**: Data Processing Agreements with all third-party processors
- **Privacy Engineering**: Privacy by design and by default throughout the platform
- **Audit Readiness**: Documentation and evidence for regulatory audits

---

## Scope

### Included

This specification covers:

- **GDPR Fundamentals**:
  - Territorial scope and applicability to MLC-LMS
  - GDPR principles (lawfulness, fairness, transparency, purpose limitation, etc.)
  - Legal bases for processing (consent, contract, legitimate interests)
  - Data controller vs. processor roles

- **Data Subject Rights Implementation**:
  - Right to access (data portability)
  - Right to rectification (correction)
  - Right to erasure ("right to be forgotten")
  - Right to restrict processing
  - Right to object
  - Rights related to automated decision-making

- **Consent Management**:
  - GDPR-compliant consent mechanisms
  - Granular consent options
  - Consent withdrawal procedures
  - Consent records and audit trails

- **Data Protection Officer (DPO)**:
  - DPO role and responsibilities
  - Contact information and accessibility
  - Independence and reporting structure

- **Data Processing Agreements (DPAs)**:
  - Requirements for DPAs with vendors
  - Standard Contractual Clauses (SCCs) for international transfers
  - Processor obligations and liabilities

- **Data Breach Procedures**:
  - 72-hour notification requirement to supervisory authority
  - Data subject notification requirements
  - Breach assessment and documentation
  - Incident response coordination

- **Privacy by Design and Default**:
  - Technical privacy controls
  - Data minimization implementation
  - Pseudonymization and anonymization
  - Privacy-preserving design patterns

- **Data Protection Impact Assessments (DPIAs)**:
  - When DPIAs are required
  - DPIA process and templates
  - Risk mitigation measures
  - Documentation requirements

- **International Data Transfers**:
  - EU-US data transfer mechanisms
  - Standard Contractual Clauses implementation
  - Adequacy decisions and alternatives
  - Transfer impact assessments

- **Accountability and Governance**:
  - Records of processing activities
  - Compliance monitoring and auditing
  - Staff training requirements
  - Documentation and evidence collection

### Excluded

This specification does not cover:

- **US-Specific Privacy Laws**: COPPA and FERPA covered in [COPPA/FERPA Notes](./coppa-ferpa-notes.md)
- **General Privacy Policy**: User-facing policy covered in [Privacy Policy](./privacy-policy.md)
- **California Privacy Rights**: CCPA/CPRA covered in Privacy Policy
- **General Security Architecture**: Technical controls covered in [Security and Privacy](../18-architecture/security-privacy.md)
- **Terms of Service**: User agreements covered in [TOS Outline](./tos-outline.md)
- **Payment Processing**: Billing specifics covered in [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md)

---

## GDPR Applicability to MLC-LMS

### Territorial Scope

**GDPR Applies When:**
- Processing personal data of individuals located in the EU/EEA, regardless of where MLC is located
- Offering services to EU residents (even if free)
- Monitoring behavior of individuals in the EU

**MLC-LMS Status:**
- Primary operations: United States (Missouri)
- User base: Global, including EU/EEA users
- Data storage: United States (Neon database, Vercel hosting)
- **GDPR applies**: We process personal data of EU residents and offer services to EU users

### MLC as Data Controller

For most processing activities, Music Learning Community acts as a **Data Controller**:

**What This Means:**
- We determine the purposes and means of processing personal data
- We are primarily responsible for GDPR compliance
- We make decisions about what data to collect and how to use it
- We bear primary liability for violations

**Data We Control:**
- Student accounts and profiles (names, usernames, email addresses)
- Game scores and learning progress
- Teacher and administrator accounts
- Subscription and billing information
- Platform usage analytics
- Messages sent through platform

### MLC as Data Processor (School Official)

For educational institutions subject to FERPA, MLC may act as a **Data Processor** on behalf of the school:

**What This Means:**
- School is the controller; MLC processes data per school's instructions
- School determines purposes; MLC determines technical means
- Data Processing Agreement required with each school
- MLC obligations: security, confidentiality, assist with data subject rights, delete/return data when requested

**Applies To:**
- K-12 schools with FERPA obligations
- EU schools subject to GDPR
- Institutional subscriptions where school is primary controller

**Dual Role Considerations:**
- For some processing (billing, platform operations), MLC is controller
- For educational records processing, MLC is processor
- Clear separation of roles documented in DPA

---

## GDPR Principles

MLC-LMS must comply with six core GDPR principles:

### 1. Lawfulness, Fairness, and Transparency

**Requirements:**
- Process data lawfully based on valid legal basis
- Process data fairly, not in misleading or hidden ways
- Be transparent about data processing with clear, plain language

**Implementation:**
- [Privacy Policy](./privacy-policy.md) written in clear, accessible language
- Explicit consent obtained where required
- Processing purposes clearly explained to users
- No hidden data collection or unexpected uses

### 2. Purpose Limitation

**Requirements:**
- Collect data for specified, explicit, and legitimate purposes
- Not process data in ways incompatible with original purposes

**Implementation:**
- Data collected only for educational purposes
- No repurposing of student data for marketing or advertising
- Each data category has documented purpose (see Data Inventory section)
- Any new purposes require user notification and consent (if required)

### 3. Data Minimization

**Requirements:**
- Collect only data that is adequate, relevant, and limited to what's necessary

**Implementation:**
- Student accounts require only name and username (email optional)
- Generic Student option for classroom use (no PII)
- No collection of unnecessary personal data (SSN, precise geolocation, biometrics)
- Regular review to ensure continued necessity

### 4. Accuracy

**Requirements:**
- Keep personal data accurate and up to date
- Provide mechanisms to correct inaccurate data

**Implementation:**
- Users can update profile information through account settings
- Data subject right to rectification procedures (see Rights Implementation section)
- Teachers/admins can correct student profile data
- Amendment request process for disputed educational records

### 5. Storage Limitation

**Requirements:**
- Retain data only as long as necessary for processing purposes
- Delete or anonymize data when no longer needed

**Implementation:**
- **Active subscriptions**: Retained while subscription is active
- **Cancelled subscriptions**: 13 months retention, then automatic deletion
- **Financial records**: 7 years (legal requirement)
- **Audit logs**: 1-3 years depending on type
- Automated deletion jobs for expired data
- See [Data Retention Policy](#data-retention-policy) section

### 6. Integrity and Confidentiality (Security)

**Requirements:**
- Process data securely with appropriate technical and organizational measures
- Protect against unauthorized access, loss, or damage

**Implementation:**
- AES-256 encryption at rest
- TLS 1.3 encryption in transit
- Role-based access controls
- Multi-factor authentication for admin accounts
- Security monitoring and logging
- See [Security and Privacy Architecture](../18-architecture/security-privacy.md)

### 7. Accountability

**Requirements:**
- Demonstrate compliance with GDPR principles
- Maintain documentation and evidence
- Implement policies and procedures

**Implementation:**
- Records of processing activities maintained
- Data Processing Agreements with all vendors
- Privacy by design practices documented
- Regular compliance audits and reviews
- Staff training on GDPR requirements
- Incident response procedures documented

---

## Legal Bases for Processing

Under GDPR Article 6, personal data processing must have a valid legal basis. MLC relies on the following legal bases:

### 1. Contract (Article 6(1)(b))

**Legal Basis:** Processing necessary to perform a contract with the data subject or to take steps at their request before entering into a contract.

**Applies To:**
- Creating and maintaining user accounts
- Providing access to educational games and platform features
- Processing subscription and billing
- Delivering customer support
- Fulfilling services requested by subscriber

**Data Processed:**
- Subscriber name, email, username, password
- Teacher and student account information
- Subscription level and seat allocation
- Payment information (processed by Braintree as our processor)

**Rationale:** Without processing this data, we cannot provide the service you subscribed to.

### 2. Consent (Article 6(1)(a))

**Legal Basis:** Data subject has given explicit, informed, freely given consent.

**Applies To:**
- Email marketing and newsletters (opt-in)
- Student email addresses for notifications (optional, requires consent)
- Public leaderboard participation for students
- Cookies and tracking (beyond strictly necessary cookies)
- Research using non-anonymized data

**Consent Requirements:**
- Freely given (not bundled with required consents)
- Specific (granular, per purpose)
- Informed (clear explanation of processing)
- Unambiguous indication (affirmative action, no pre-ticked boxes)
- Easy to withdraw (as easy as giving consent)
- Records maintained (timestamp, method, scope of consent)

**Implementation:**
- Separate checkboxes for optional consents during account creation
- Consent management in account settings
- Withdrawal button next to each consent
- Audit trail of all consent actions

**Example Consent Text:**
```
☐ I consent to receiving email newsletters with educational tips, 
  platform updates, and special offers. I understand I can unsubscribe 
  at any time.
  
☐ [For students under 13] I consent to MLC collecting my child's email 
  address to send assignment notifications and progress updates.
```

### 3. Legal Obligation (Article 6(1)(c))

**Legal Basis:** Processing necessary to comply with legal obligations.

**Applies To:**
- Tax and financial record retention
- Responding to lawful requests from authorities
- COPPA compliance reporting (if required by FTC)
- Breach notifications to regulators
- Anti-money laundering checks (if applicable)

**Data Processed:**
- Financial transaction records
- Audit logs for compliance
- Legal hold data
- Records required by subpoena or court order

**Rationale:** We are legally required to retain certain data and comply with valid legal requests.

### 4. Legitimate Interests (Article 6(1)(f))

**Legal Basis:** Processing necessary for legitimate interests pursued by controller or third party, except where overridden by data subject rights.

**Requires Legitimate Interests Assessment (LIA) balancing:**
- Legitimate interest of MLC or users
- Necessity of processing for that interest
- Balance against rights and interests of data subjects
- Appropriate safeguards implemented

**Applies To:**
- Fraud prevention and platform security
- Network and information security monitoring
- Platform usage analytics for product improvement
- Direct marketing to existing customers (soft opt-in)
- Internal administrative purposes
- Research using anonymized or pseudonymized data

**Example LIA: Platform Usage Analytics**

| Factor | Assessment |
|--------|-----------|
| **Legitimate Interest** | Improve platform usability, identify bugs, optimize performance |
| **Necessity** | Analytics needed to understand how features are used and identify issues |
| **Data Subject Impact** | Low - analytics on teacher/admin pages only (not student-facing), no behavioral tracking, data not sold |
| **Safeguards** | No tracking on student pages for children under 13, data minimization, aggregation where possible, opt-out available |
| **Balancing Result** | Legitimate interest justified - minimal impact on users, significant benefit for platform improvement |

**Legitimate Interests We Do NOT Rely On:**
- Student data for advertising or marketing (not legitimate given child users)
- Selling user data to third parties (fundamentally incompatible with educational mission)
- Extensive behavioral profiling of students
- Cross-site tracking or data broker activities

---

## Special Category Data

### GDPR Article 9: Special Categories of Personal Data

**Special category data (sensitive data) includes:**
- Racial or ethnic origin
- Political opinions
- Religious or philosophical beliefs
- Trade union membership
- Genetic data
- Biometric data for identification
- Health data
- Sex life or sexual orientation

**MLC-LMS Position:** We do NOT collect or process special category data.

**Verification:**
- Student and teacher accounts: No health, biometric, or other sensitive data collected
- Optional profile fields: No sensitive data fields offered
- Game content: No collection of sensitive personal information through games
- Messages: No deliberate collection, but users instructed not to share sensitive information

**If Special Category Data Is Encountered:**
- If user voluntarily includes in message or profile (against policy), data deleted immediately
- Support staff instructed to redact/delete if encountered in support tickets
- No processing of such data; treated as policy violation

**Future Considerations:**
- If accessibility features require health data (e.g., vision impairment settings), explicit consent required
- If profile photo feature added, biometric data considerations
- Any new features assessed for special category data implications

---

## Children's Data (Article 8)

### GDPR Requirements for Children

**Article 8: Age of Consent for Information Society Services**
- Member states set age of consent between 13-16 years
- Below that age, parental consent required
- Most EU countries: 13-16 years (varies by country)

**MLC-LMS Approach:**
- **Conservative approach**: Treat all students under 16 as requiring parental consent (strictest EU requirement)
- **US COPPA**: Already requires parental consent for children under 13
- **Combined approach**: COPPA compliance for <13, extended to <16 for EU users

### Parental Consent for EU Children

**For EU Students Under 16:**
- Parental or guardian consent required before processing personal data
- Consent obtained and verified by teacher/school (acts as agent)
- Teacher certification that parental consent obtained
- Optional email verification to parent

**Consent Mechanisms:**
- Same as COPPA consent mechanisms (see [COPPA/FERPA Notes](./coppa-ferpa-notes.md))
- Extended age threshold to 16 for EU users
- Database flag: `is_eu_user` and `is_under_16` (in addition to `is_under_13`)

**Implementation:**
```sql
students:
  id: uuid
  is_under_13: boolean
  is_under_16: boolean (for EU users)
  is_eu_user: boolean (detected by country or explicit selection)
  eu_parental_consent_status: enum
  eu_parental_consent_date: timestamp
```

**EU-Specific Consent Status Values:**
- `not_required` - Student is 16+ or not in EU
- `certified_by_teacher` - Teacher certified parental consent obtained
- `verified` - Parent confirmed via email
- `pending` - Awaiting parental verification
- `withdrawn` - Parent withdrew consent

### Determining EU User Status

**Methods:**
1. **IP Geolocation** (automatic detection during signup)
   - Detect user location based on IP address
   - Flag as `is_eu_user` if IP in EU/EEA country
   - Not 100% accurate (VPNs, proxies)

2. **User Declaration** (explicit selection)
   - Account creation: "Are you located in the European Union?"
   - Honor user's declaration even if IP suggests otherwise
   - Required for accurate GDPR compliance

3. **School Location** (for institutional subscriptions)
   - School provides country/location during subscription
   - All students under that subscription flagged as EU users if school is in EU

**Recommendation:** Combination of IP detection + user declaration + school location

### Enhanced Rights for Children

**Stronger Protections:**
- No profiling or automated decision-making affecting children
- No direct marketing to children without parental consent
- Data minimization especially important for children
- Simplified privacy information accessible to children
- Parental access to child's data at any time

**MLC-LMS Implementation:**
- No advertising or marketing to students (regardless of age)
- No behavioral tracking on student-facing pages
- Generic Student option (no PII collection) remains GDPR-compliant
- Privacy policy includes child-friendly section

---

## Data Subject Rights Implementation

Under GDPR Articles 15-22, data subjects have specific rights regarding their personal data. MLC must provide technical and procedural mechanisms to honor these rights within required timeframes.

### Right to Access (Article 15) & Data Portability (Article 20)

**Data Subject Right:** Obtain confirmation of whether we process their personal data, and if so, receive a copy of that data.

**Fulfillment Timeline:** Within 1 month of request (extendable to 3 months for complex requests)

**What Must Be Provided:**
- Copy of all personal data we hold about the data subject
- Categories of data processed
- Purposes of processing
- Recipients of data (who we've shared it with)
- Retention periods
- Information about data source (if not collected directly)
- Existence of automated decision-making (if applicable)
- Right to lodge complaint with supervisory authority

**Data Portability Additions (Article 20):**
- Data provided in structured, commonly used, machine-readable format (JSON, CSV)
- Ability to transmit data to another controller (where technically feasible)
- Only applies to data provided by data subject and processed based on consent or contract

**Implementation:**

**Self-Service Access (Preferred):**
```
Account Settings → Privacy & Data → Download My Data

Options:
☐ Account Information (profile, settings)
☐ Educational Records (scores, assignments, progress)
☐ Messages and Communications
☐ Login History and Activity Logs
☐ All of the Above

Format: ○ JSON ○ CSV ○ PDF Report

[Generate Data Export]
```

**Export Contents:**
- `account_data.json` - Profile information, account settings
- `educational_records.csv` - Game scores, assignments, mastery data
- `messages.json` - All messages sent and received
- `activity_log.csv` - Login times, IP addresses, platform actions
- `consent_history.json` - All consent actions and timestamps
- `README.txt` - Explanation of files and data structure

**Admin-Facilitated Access (for complex requests or user assistance):**
1. User contacts dpo@musiclearningcommunity.com
2. Identity verification performed
3. System admin runs data export job
4. Export delivered via secure download link (expires in 7 days)
5. Request logged in audit trail

**Technical Implementation:**
```typescript
// Data export service
async function generateGDPRDataExport(userId: string): Promise<DataExportBundle> {
  const user = await db.users.findById(userId);
  
  // Gather all personal data
  const accountData = await getAccountData(userId);
  const educationalRecords = await getEducationalRecords(userId);
  const messages = await getMessages(userId);
  const activityLog = await getActivityLog(userId);
  const consentHistory = await getConsentHistory(userId);
  
  // Bundle into machine-readable formats
  const exportBundle = {
    account: accountData, // JSON
    educational: educationalRecords, // CSV
    messages: messages, // JSON
    activity: activityLog, // CSV
    consents: consentHistory, // JSON
    metadata: {
      exportDate: new Date().toISOString(),
      dataSubject: user.email,
      format: 'GDPR Article 15/20 Compliant Export'
    }
  };
  
  // Create ZIP archive
  const zipFile = await createZipArchive(exportBundle);
  
  // Log export for audit
  await logGDPREvent('data_access_request_fulfilled', userId);
  
  return zipFile;
}
```

**Database Schema for Request Tracking:**
```sql
gdpr_access_requests:
  id: uuid (primary key)
  user_id: uuid (foreign key)
  request_type: enum ('access', 'portability', 'rectification', 'erasure', 'restrict', 'object')
  requested_at: timestamp
  fulfilled_at: timestamp (nullable)
  status: enum ('pending', 'in_progress', 'fulfilled', 'rejected')
  rejection_reason: text (nullable)
  export_file_url: text (nullable) -- secure download link
  file_expiry: timestamp (nullable) -- link expires after 7 days
  requested_by: uuid -- user_id who made request
  fulfilled_by: uuid -- system admin who processed (if manual)
```

### Right to Rectification (Article 16)

**Data Subject Right:** Correct inaccurate or incomplete personal data.

**Fulfillment Timeline:** Within 1 month of request

**Implementation:**

**Self-Service Correction:**
- Users can edit profile information directly in account settings
- Name, email, contact information, preferences
- Changes logged in audit trail

**Admin-Assisted Correction:**
- For data users cannot edit themselves (e.g., historical scores due to technical error)
- Correction request submitted via form or email
- Admin reviews and applies correction
- Data subject notified when complete

**Example Correction Request Form:**
```
Data Correction Request

Field to Correct: [dropdown: Profile Information / Educational Record / Other]
Current Value: [__________]
Correct Value: [__________]
Reason for Correction: [___________________________]

[Submit Request]
```

**Process Flow:**
1. User submits correction request
2. Request logged in `gdpr_access_requests` table with type `rectification`
3. System admin reviews request
4. If approved: Data corrected, user notified
5. If rejected: User notified with explanation and right to appeal
6. Request marked as fulfilled

**Amendment vs. Correction:**
- **Correction**: Fixing objectively inaccurate data (typo in name, wrong email)
- **Amendment**: Disputing accuracy of educational assessment (score was wrong due to bug)
- Amendments may involve FERPA procedures if educational record (see [COPPA/FERPA Notes](./coppa-ferpa-notes.md))

### Right to Erasure / "Right to be Forgotten" (Article 17)

**Data Subject Right:** Request deletion of personal data under certain conditions.

**Grounds for Erasure:**
1. Data no longer necessary for purposes collected
2. Data subject withdraws consent (and no other legal basis)
3. Data subject objects and no overriding legitimate grounds
4. Data processed unlawfully
5. Legal obligation requires deletion
6. Data collected from child without proper consent

**Exceptions (When Erasure Can Be Refused):**
- Compliance with legal obligation (e.g., financial records for tax law)
- Exercise or defense of legal claims
- Public interest in the field of public health
- Archiving purposes in the public interest, scientific or historical research (with safeguards)

**Fulfillment Timeline:** Within 1 month of request

**Implementation:**

**Student Account Deletion Request:**

**Process Flow:**
1. Parent/guardian/student requests deletion via email to dpo@musiclearningcommunity.com
2. Identity verification performed
3. System admin assesses request:
   - Can deletion be honored? (check for exceptions)
   - What data must be retained? (financial, legal holds)
   - Notify data subject of assessment
4. If approved, 30-day grace period begins
5. Data subject can cancel deletion during grace period
6. After grace period, permanent deletion executed
7. Confirmation sent to data subject

**What Gets Deleted:**
- Student account and login credentials
- Personal information (name, email if collected, date of birth)
- Profile data and preferences
- Game scores and educational records
- Assignments and progress data
- Messages and communications
- Session data and activity logs (beyond legal retention requirement)

**What May Be Retained (With Justification):**
- Financial transaction records (7 years - legal obligation)
- Aggregated, anonymized data (no longer personal data)
- Data under legal hold (litigation, investigation)
- Security logs required for breach investigation (limited retention)

**Technical Implementation:**
```typescript
async function processErasureRequest(requestId: string) {
  const request = await db.gdpr_access_requests.findById(requestId);
  const userId = request.user_id;
  
  // Check for legal holds
  const legalHold = await db.legal_holds.where({ user_id: userId, active: true }).first();
  if (legalHold) {
    await updateRequestStatus(requestId, 'rejected', 'Data subject to legal hold');
    return;
  }
  
  // 30-day grace period check
  const gracePeriodEnd = addDays(request.requested_at, 30);
  if (new Date() < gracePeriodEnd) {
    // Grace period not expired, wait
    return;
  }
  
  // Execute deletion
  await anonymizeFinancialRecords(userId); // Keep records, remove PII
  await deleteEducationalRecords(userId);
  await deleteMessages(userId);
  await deleteSessionData(userId);
  await deleteUserAccount(userId);
  
  // Purge from backups (marked for exclusion in next backup cycle)
  await markForPurgeFromBackups(userId);
  
  // Log erasure for audit
  await logGDPREvent('right_to_erasure_fulfilled', userId, {
    requestId: requestId,
    deletionDate: new Date(),
    retainedData: ['financial_records_anonymized']
  });
  
  // Update request status
  await updateRequestStatus(requestId, 'fulfilled');
  
  // Send confirmation email
  await sendEmail(request.email, 'erasure_confirmation');
}
```

**Database Considerations:**
- Cascading deletes for related records
- Foreign key constraints handled properly
- Soft delete (30-day grace) then hard delete
- Backup rotation to purge deleted user data

### Right to Restrict Processing (Article 18)

**Data Subject Right:** Request temporary restriction of processing under certain conditions.

**When Right Applies:**
1. Data subject contests accuracy of data (during verification period)
2. Processing is unlawful, but data subject prefers restriction over erasure
3. We no longer need data, but data subject needs it for legal claims
4. Data subject objected to processing (pending verification of legitimate grounds)

**Fulfillment Timeline:** Immediately upon request; verification within 1 month

**What Restriction Means:**
- Data can be stored but not otherwise processed
- Processing only with data subject consent, for legal claims, for protection of another person's rights, or for public interest
- Data subject must be informed before restriction is lifted

**Implementation:**

**Database Flag:**
```sql
ALTER TABLE users ADD COLUMN processing_restricted BOOLEAN DEFAULT FALSE;
ALTER TABLE users ADD COLUMN restriction_reason TEXT;
ALTER TABLE users ADD COLUMN restriction_start_date TIMESTAMP;
```

**Application Logic:**
```typescript
async function checkProcessingRestriction(userId: string, operation: string): Promise<boolean> {
  const user = await db.users.findById(userId);
  
  if (!user.processing_restricted) {
    return true; // No restriction, proceed
  }
  
  // Allow storage but restrict other processing
  const allowedOperations = ['store', 'legal_claim', 'consent_based'];
  
  if (!allowedOperations.includes(operation)) {
    await logGDPREvent('processing_restricted_blocked', userId, { operation });
    throw new ProcessingRestrictedError('Processing restricted for this user');
  }
  
  return true;
}

// Before any data processing operation
await checkProcessingRestriction(userId, 'send_email'); // Would throw error if restricted
```

**User Experience:**
- Restricted user can still log in
- Account marked as "Processing Restricted" in admin panel
- No marketing emails sent
- No usage analytics collected
- Educational services continue (with consent or contract basis)
- Admin notified of restriction when viewing user

### Right to Object (Article 21)

**Data Subject Right:** Object to processing based on legitimate interests or direct marketing.

**Fulfillment Timeline:** Immediately stop processing unless compelling legitimate grounds override

**Implementation:**

**Object to Direct Marketing:**
- Must stop immediately, no exceptions
- Unsubscribe link in every marketing email
- Self-service opt-out in account settings
- No further marketing emails sent

**Object to Processing Based on Legitimate Interests:**
- Must assess whether MLC has compelling legitimate grounds to continue
- If no compelling grounds, stop processing
- If compelling grounds exist, explain to data subject and continue

**Example: Object to Analytics:**
```
User objects to usage analytics collection based on legitimate interests.

Assessment:
- Legitimate Interest: Improve platform usability and identify bugs
- Compelling Grounds Check:
  * User already opted out of analytics in settings
  * Analytics can be disabled for individual user without affecting service
  * No compelling grounds to override user's objection
  
Decision: Honor objection. Disable analytics for this user.
```

**Technical Implementation:**
```sql
users:
  marketing_opt_out: boolean (default: false)
  analytics_opt_out: boolean (default: false)
  legitimate_interest_objection: jsonb -- detailed objections
```

**Application Logic:**
- Check opt-out flags before any marketing or analytics processing
- Respect objections immediately
- Log objection in audit trail

### Rights Related to Automated Decision-Making (Article 22)

**Data Subject Right:** Not be subject to decisions based solely on automated processing with legal or significant effects.

**MLC-LMS Status:** We do NOT engage in automated decision-making with legal or similarly significant effects.

**Verification:**
- No automated credit decisions
- No automated account suspensions (admin reviews all suspensions)
- No automated profiling with significant impact
- Assignment generation is educational tool, not decision with legal effect
- Leaderboards are opt-in and gamification, not decisions affecting rights

**If Implemented in Future:**
- Inform data subject of logic involved
- Right to human review of decision
- Right to express point of view
- Right to contest decision

### GDPR Rights Summary Table

| Right | Article | Timeline | Self-Service | Admin-Assisted | Cost to User |
|-------|---------|----------|--------------|----------------|--------------|
| Access | 15 | 1 month | Yes (Download My Data) | Yes | Free |
| Data Portability | 20 | 1 month | Yes (Export JSON/CSV) | Yes | Free |
| Rectification | 16 | 1 month | Yes (Edit Profile) | Yes (for restricted fields) | Free |
| Erasure | 17 | 1 month | Request via email | Yes (with verification) | Free |
| Restrict Processing | 18 | Immediate | Request via email | Yes | Free |
| Object | 21 | Immediate | Yes (Unsubscribe/Opt-out) | Yes | Free |
| Automated Decision | 22 | N/A | N/A (we don't do this) | N/A | N/A |

**Note:** First request is always free. For "manifestly unfounded or excessive" repeat requests, MLC may charge reasonable fee or refuse request (Article 12(5)).

---

## Data Breach Procedures (Article 33-34)

### GDPR Breach Notification Requirements

**Article 33: Notification to Supervisory Authority**
- Within 72 hours of becoming aware of breach
- Describe nature of breach, categories and number of affected individuals
- Describe likely consequences and measures taken to address
- If not notified within 72 hours, must explain delay

**Article 34: Notification to Data Subjects**
- Required if breach likely to result in high risk to rights and freedoms
- Describe nature of breach in clear, plain language
- Provide contact point for more information
- Describe likely consequences and mitigation measures
- Not required if: data encrypted, subsequent measures reduce risk, or notification would be disproportionate effort (can use public communication instead)

### Breach Assessment Framework

**Step 1: Breach Discovery and Containment (Immediate)**
- Incident detected via monitoring, user report, or vendor notification
- Immediate containment actions to limit scope
- Preserve evidence for investigation
- Document timeline of discovery

**Step 2: Breach Classification (Within 24 Hours)**

Assess severity based on:
- **Nature of Data**: PII, children's data, sensitive data
- **Scope**: Number of affected individuals
- **Confidentiality**: Was data accessed by unauthorized party?
- **Integrity**: Was data altered or deleted?
- **Availability**: Was data made unavailable?
- **Impact**: Likely consequences for data subjects (identity theft, discrimination, financial loss, reputational damage)

**Severity Levels:**

| Level | Criteria | Example | Supervisory Authority Notification | Data Subject Notification |
|-------|----------|---------|-----------------------------------|-----------------------|
| **High Risk** | Likely to result in high risk to rights and freedoms | Unencrypted student PII exposed to public internet | Required (72 hours) | Required (without undue delay) |
| **Moderate Risk** | Some risk but not high risk | Encrypted data accessed, decryption keys not compromised | Required (72 hours) | Not required (but consider) |
| **Low Risk** | Unlikely to result in risk | Internal employee accidentally accessed one student record beyond authorization | Required (72 hours) | Not required |
| **No Breach** | No personal data compromised | Attempted intrusion blocked, no data accessed | Not required | Not required |

**Step 3: Notification Decision (Within 48 Hours)**

**Supervisory Authority Notification:**
- Required for all breaches except when "unlikely to result in risk to rights and freedoms"
- Err on side of notification if uncertain

**Data Subject Notification:**
- Required when breach "likely to result in high risk"
- Not required if:
  - Data was encrypted or otherwise unintelligible
  - Subsequent measures taken eliminate high risk
  - Notification would require disproportionate effort (public communication instead)

**Step 4: Notification Execution (Within 72 Hours of Awareness)**

### Notification Templates

**Template: Supervisory Authority Notification**

Submit to relevant Data Protection Authority (DPA):
- Ireland: Data Protection Commission (if using Vercel EU hosting)
- User's member state DPA (if known)
- All affected member states if multi-country breach

**Required Content:**
```
GDPR Article 33 Data Breach Notification

Breach Reference ID: [MLC-BREACH-YYYY-MM-DD-001]
Date of Notification: [Date]
Controller: Music Learning Community, Wildwood, Missouri, USA
DPO Contact: dpo@musiclearningcommunity.com

1. Nature of Personal Data Breach:
   - Date/time breach discovered: [DateTime]
   - Date/time breach occurred (if known): [DateTime]
   - Description: [What happened]
   - Type of breach: [Confidentiality / Integrity / Availability]

2. Categories and Numbers of Data Subjects and Records:
   - Data subject categories: [Students, Teachers, Administrators]
   - Approximate number of affected data subjects: [Number]
   - Categories of personal data: [Names, email addresses, game scores, etc.]
   - Approximate number of records: [Number]

3. Likely Consequences of the Breach:
   - [Describe potential impact on data subjects]
   - Risk level: [High / Moderate / Low]
   - Justification: [Why this risk level]

4. Measures Taken or Proposed:
   - Immediate containment: [Actions taken]
   - Mitigation measures: [How risk reduced]
   - Preventive measures: [Future prevention]
   - Data subject notification plan: [If applicable]

5. DPO Contact Information:
   - Name: [DPO Name]
   - Email: dpo@musiclearningcommunity.com
   - Phone: [Phone]

Submitted by: [Name, Title]
Signature: [Digital Signature]
```

**Template: Data Subject Notification Email**

```
Subject: Important Security Notice - Music Learning Community

Dear [User Name],

We are writing to inform you of a security incident involving your personal information on the Music Learning Community platform.

What Happened:
[Clear, plain language description of the breach]

What Information Was Involved:
The following information may have been affected:
- [List specific data categories: name, email, scores, etc.]

What We Are Doing:
- [Immediate actions taken to secure data]
- [Measures to prevent recurrence]
- [Resources offered: credit monitoring, password reset, etc.]

What You Can Do:
1. [Recommended actions for user: change password, monitor accounts, etc.]
2. [How to contact us for questions]

Your Rights:
You have the right to lodge a complaint with your data protection authority. 
Contact information: [Relevant DPA]

More Information:
For more details or questions, contact our Data Protection Officer:
Email: dpo@musiclearningcommunity.com
Phone: [Phone]

We sincerely apologize for this incident and are committed to protecting your information.

Sincerely,
Music Learning Community
Data Protection Officer
```

### Breach Response Procedures

**Incident Response Team:**
- **Incident Commander**: CTO or designated security lead
- **Data Protection Officer**: GDPR compliance and notification lead
- **Legal Counsel**: Legal implications and liability assessment
- **Communications**: Internal and external communication coordination
- **Technical Lead**: Investigation and remediation

**Response Workflow:**
1. **Detection** → Incident identified and logged
2. **Containment** → Immediate actions to stop breach (isolate systems, revoke access)
3. **Assessment** → Determine scope, severity, affected data subjects
4. **Notification Decision** → DPO decides on supervisory authority and data subject notification
5. **Notification Execution** → Submit notifications within 72 hours
6. **Investigation** → Root cause analysis and evidence collection
7. **Remediation** → Fix vulnerabilities and implement preventive measures
8. **Documentation** → Complete incident report for audit records
9. **Post-Incident Review** → Lessons learned and process improvements

**Breach Log:**
```sql
gdpr_breach_log:
  id: uuid (primary key)
  breach_ref_id: string (e.g., 'MLC-BREACH-2025-01-15-001')
  discovered_at: timestamp
  occurred_at: timestamp (estimated)
  breach_type: enum ('confidentiality', 'integrity', 'availability', 'combined')
  affected_user_count: integer
  affected_data_categories: jsonb
  risk_level: enum ('high', 'moderate', 'low', 'none')
  supervisory_authority_notified: boolean
  supervisory_authority_notification_date: timestamp
  data_subjects_notified: boolean
  data_subjects_notification_date: timestamp
  root_cause: text
  containment_actions: text
  remediation_actions: text
  status: enum ('discovered', 'contained', 'notified', 'remediated', 'closed')
  incident_commander: uuid (user_id)
  dpo_assigned: uuid (user_id)
  created_at: timestamp
  updated_at: timestamp
```

**Documentation Requirements:**
- Maintain internal record of all breaches (even if not notified to authority)
- Document risk assessment and notification decisions
- Evidence to demonstrate GDPR compliance (72-hour timeline adherence)
- Post-incident report with lessons learned

For detailed incident response procedures, see [Incident Response](../19-quality-and-operations/incident-response.md).

---

## International Data Transfers (Chapter V)

### Legal Challenge of EU-US Data Transfers

**The Problem:**
- MLC is based in the United States (Missouri)
- Data stored on US servers (Neon database, Vercel hosting)
- We process personal data of EU residents
- GDPR restricts transfers of personal data outside the EU/EEA

**Post-Schrems II Landscape:**
- EU-US Privacy Shield invalidated by CJEU (2020)
- Standard Contractual Clauses (SCCs) remain valid but require additional safeguards
- Transfer Impact Assessments (TIAs) required
- US surveillance laws (FISA 702, E.O. 12333) create compliance challenges

### Legal Mechanisms for EU-US Transfers

**Option 1: Standard Contractual Clauses (SCCs) - Primary Mechanism**

**What Are SCCs:**
- European Commission-approved contract templates
- Impose data protection obligations on data importers (MLC)
- Provide data subjects with enforceable rights
- Updated SCCs adopted June 2021 (Commission Implementing Decision 2021/914)

**Implementation:**
1. **Execute SCCs with EU Data Subjects** (Module One: Controller to Controller)
   - Include SCCs in Terms of Service or as separate addendum
   - EU users explicitly accept SCCs during account creation

2. **Execute SCCs with EU Schools** (Module Two: Controller to Processor)
   - For institutional subscriptions with EU schools
   - School is controller, MLC is processor
   - SCCs included in Data Processing Agreement

3. **Execute SCCs with Vendors** (Module Three: Processor to Sub-Processor)
   - Vercel, Neon, Cloudflare, Braintree as sub-processors
   - Ensure each vendor has signed SCCs or equivalent

**SCC Documentation:**
- File SCCs with EU data protection authorities (if required by member state)
- Maintain copies of all executed SCCs
- Update SCCs when required (Commission may update templates)

**Option 2: Adequacy Decision**

**Current Status:**
- No adequacy decision for United States (Privacy Shield invalidated)
- Monitor for new EU-US data transfer framework
- If new adequacy decision adopted, can rely on that instead of SCCs

**Option 3: Explicit Consent (Limited Use)**

**Can Use Consent For:**
- Occasional, non-repetitive transfers
- User fully informed of risks
- Not suitable for ongoing, systematic transfers (like MLC-LMS operations)

**Why We Don't Rely on Consent:**
- MLC transfers are ongoing and systematic (every time EU user logs in)
- Consent must be freely given (hard to argue when necessary for service)
- SCCs are more robust and legally certain

**Option 4: Binding Corporate Rules (BCRs)**

**Not Applicable:**
- MLC is single legal entity, not multinational corporation
- BCRs designed for intra-group transfers
- Not pursuing BCRs at this time

### Transfer Impact Assessment (TIA)

**Requirement:** Under Schrems II, must assess whether EU-US transfer provides essentially equivalent protection to GDPR.

**TIA Components:**

**1. Assessment of US Surveillance Laws**

**FISA Section 702:**
- Allows US intelligence agencies to collect communications of non-US persons
- MLC is small educational platform, unlikely to be subject to FISA 702 orders
- MLC does not have "communications service provider" status
- **Risk: Low** - MLC is not telecommunications or large tech company typically targeted

**Executive Order 12333:**
- Broad surveillance authority for foreign intelligence
- Does not require warrant or individualized suspicion
- Applies to data outside US (in transit or storage)
- **Risk: Medium** - US government could potentially access data at rest

**Stored Communications Act (SCA):**
- Allows US law enforcement to request data with warrant or subpoena
- Applies to any provider storing communications
- MLC could receive lawful requests
- **Risk: Medium** - Standard law enforcement risk, similar to EU

**2. Assessment of Technical and Organizational Safeguards**

**MLC Safeguards:**
- **Encryption at Rest (AES-256)**: Data encrypted in database, decryption keys controlled by MLC
- **Encryption in Transit (TLS 1.3)**: All data transfers encrypted
- **Access Controls**: Role-based access, limited employees have database access
- **Audit Logging**: All data access logged for accountability
- **Data Minimization**: Collect only necessary data
- **Pseudonymization**: Where feasible (e.g., student usernames instead of real names)
- **Vendor Selection**: Use vendors with strong security postures (Vercel, Neon)

**Supplementary Measures:**
- **Encryption of particularly sensitive data**: Student email addresses encrypted with additional layer
- **Segmented storage**: EU user data logically separated (future enhancement)
- **Transparency**: Privacy policy clearly explains US storage
- **Legal Challenge Fund**: Commit to challenging any overbroad government data requests

**3. Balancing Assessment**

| Factor | Assessment |
|--------|------------|
| **Nature of Data** | Educational records, student names, game scores. Not highly sensitive (no health, financial, biometric data). |
| **Purposes of Processing** | Educational purposes only. No advertising, profiling, or surveillance. |
| **Data Subjects** | Children and educators. Vulnerable population requiring stronger protections. |
| **US Legal Framework** | Surveillance laws exist but MLC is unlikely target. Can challenge overbroad requests. |
| **Technical Safeguards** | Strong encryption, access controls, audit logging. Data is not easily accessible even to US government. |
| **Vendor Protections** | Vercel, Neon have no known history of US government data access. Committed to legal challenges. |
| **User Expectations** | Users informed of US storage in Privacy Policy. EU users can choose not to use service. |
| **Alternatives** | No viable EU-only educational platform with equivalent features. |

**TIA Conclusion:**
- **Risks exist** from US surveillance laws
- **Risks are mitigated** by technical safeguards (encryption, access controls)
- **Risks are proportionate** to benefits of service (high-quality music education platform)
- **Users are informed** and can make informed choice
- **MLC will challenge** any unlawful or overbroad government requests
- **Transfer can proceed** under SCCs with supplementary measures

**TIA Documentation:**
- Conduct formal TIA and document conclusions
- Review and update TIA annually or when circumstances change
- Make TIA available to data subjects and supervisory authorities upon request

### Specific Vendor Transfer Mechanisms

**Vercel (Hosting Platform)**
- Location: United States (with global edge network)
- Role: Processor
- Mechanism: Standard Contractual Clauses
- Additional Safeguards: SOC 2 Type II, encryption, access controls
- DPA: Vercel provides GDPR-compliant DPA with SCCs

**Neon (Database)**
- Location: United States
- Role: Processor
- Mechanism: Standard Contractual Clauses
- Additional Safeguards: SOC 2, AES-256 encryption at rest, isolated tenants
- DPA: Neon provides GDPR-compliant DPA with SCCs

**Cloudflare (CDN and Security)**
- Location: Global CDN, US-based company
- Role: Processor
- Mechanism: Standard Contractual Clauses
- Additional Safeguards: GDPR-compliant, no persistent storage of personal data in CDN cache
- DPA: Cloudflare provides GDPR-compliant DPA with SCCs

**Braintree (Payment Processing)**
- Location: United States (PayPal subsidiary)
- Role: Processor
- Mechanism: Standard Contractual Clauses
- Additional Safeguards: PCI DSS Level 1, tokenization, no raw card data to MLC
- DPA: Braintree provides GDPR-compliant DPA with SCCs

**Email Provider (TBD)**
- Will require SCC-based DPA before implementation

### Data Localization Considerations

**Current Approach: US-Based Storage**
- All data stored in United States
- Compliant via SCCs + TIA + supplementary measures
- Simplifies architecture and reduces costs

**Future Enhancement: EU Data Residency (Phase 2+)**

**If MLC grows EU user base significantly:**
- Deploy EU region database (Neon supports EU regions)
- Store EU user data exclusively in EU
- EU edge hosting via Vercel EU regions
- Still requires DPAs with vendors, but no international transfer

**Benefits of EU Data Residency:**
- Eliminates international transfer concerns
- Stronger compliance posture
- Marketing advantage in EU market

**Costs of EU Data Residency:**
- Increased infrastructure complexity
- Higher hosting costs
- Data synchronization challenges
- Support for multiple regions

**Decision Point:** Implement EU data residency when >10% of users are EU-based or if regulatory pressure increases.

---

## Data Processing Agreements (DPAs)

### When DPAs Are Required

**GDPR Article 28** requires written contracts (Data Processing Agreements) between:
- **Controller and Processor**: MLC (controller) and Vercel/Neon/Cloudflare (processors)
- **Processor and Sub-Processor**: Vercel (processor) and its sub-processors
- **Controller (School) and Processor (MLC)**: For institutional subscriptions where school is controller

**DPA vs. Terms of Service:**
- **TOS**: General user agreement, public-facing
- **DPA**: Specific data processing contract, detailed obligations
- Both required; DPA supplements TOS for data processing specifics

### Required DPA Contents (Article 28(3))

A GDPR-compliant DPA must include:

1. **Subject Matter and Duration of Processing**
   - What data is being processed
   - For how long
   - Purpose of processing

2. **Nature and Purpose of Processing**
   - Educational records management
   - Platform operations and support
   - Billing and subscription management

3. **Type of Personal Data and Categories of Data Subjects**
   - Students (including children under 16)
   - Teachers, administrators
   - Data types: names, email, game scores, messages

4. **Obligations and Rights of Controller**
   - School (if controller) has right to audit MLC's processing
   - School can request data deletion or return
   - School provides instructions to MLC on processing

5. **Processor Obligations (Article 28(3)):**
   - Process data only on documented instructions from controller
   - Ensure confidentiality of personnel processing data
   - Implement appropriate technical and organizational measures (security)
   - Engage sub-processors only with controller's authorization
   - Assist controller with data subject rights requests
   - Assist controller with security, breach notification, DPIAs
   - Delete or return personal data at end of services (at controller's choice)
   - Make available information necessary to demonstrate compliance
   - Allow for and contribute to audits

6. **Sub-Processor Provisions**
   - List of approved sub-processors (Vercel, Neon, Cloudflare, Braintree)
   - Mechanism for controller to object to new sub-processors
   - Processor remains liable for sub-processor failures

7. **International Transfers**
   - If data transferred outside EU/EEA, specify mechanism (SCCs)
   - Include Standard Contractual Clauses as appendix

8. **Security Measures**
   - Encryption at rest and in transit
   - Access controls and authentication
   - Logging and monitoring
   - Incident response procedures

9. **Data Breach Notification**
   - Processor must notify controller without undue delay upon breach
   - Provide sufficient information for controller to meet its own notification obligations

10. **Audit Rights**
    - Controller can audit processor's compliance
    - Processor must provide evidence of compliance
    - On-site audits (with reasonable notice) or third-party audits

11. **Liability and Indemnification**
    - Liability allocation between controller and processor
    - Indemnification for breaches of DPA obligations

12. **Term and Termination**
    - Duration of DPA (typically coterminous with service agreement)
    - What happens to data upon termination (deletion or return)

### MLC Vendor DPAs

**DPA Tracking Table:**

| Vendor | Role | Services | DPA Status | SCC Included | Last Reviewed |
|--------|------|----------|------------|--------------|---------------|
| Vercel | Processor | Hosting platform | ✅ Executed | Yes | 2024-Q4 |
| Neon | Processor | Database | ✅ Executed | Yes | 2024-Q4 |
| Cloudflare | Processor | CDN, security | ✅ Executed | Yes | 2024-Q3 |
| Braintree | Processor | Payment processing | ✅ Executed | Yes | 2024-Q2 |
| Email Provider | Processor | Transactional email | ⏳ Pending | Pending | N/A |

**Verification Steps:**
1. Obtain DPA from each vendor (most provide standard GDPR DPAs)
2. Review DPA to ensure Article 28 compliance
3. Verify SCCs are included (for US-based vendors)
4. Execute DPA (both parties sign)
5. Store executed DPA securely
6. Review annually and when vendor terms change

### School-MLC DPA (For Institutional Subscriptions)

**When Required:**
- EU schools subscribing to MLC-LMS
- School is the data controller (owns student data)
- MLC is the data processor (processes data on school's behalf)
- FERPA + GDPR combined compliance

**MLC Standard DPA Template:**

Provide schools with MLC's standard DPA covering:
- Article 28 requirements (listed above)
- FERPA school official exception provisions
- Appendix 1: Details of processing (data types, purposes, duration)
- Appendix 2: Technical and organizational security measures
- Appendix 3: List of sub-processors with right to object
- Appendix 4: Standard Contractual Clauses (for EU schools)

**School DPA Process:**
1. School requests ENSEMBLE subscription
2. MLC provides standard DPA for review
3. School reviews DPA (may have legal counsel review)
4. Both parties execute DPA
5. Subscription activated only after DPA executed
6. DPA stored securely and linked to school account

**Self-Service DPA Portal (Future Enhancement):**
- Schools can download, review, and e-sign DPA online
- Automated DPA generation with school-specific details
- Version control and amendment tracking
- Accessible from admin dashboard

---

## Privacy by Design and by Default (Article 25)

### Privacy by Design Principles

**Definition:** Integrate data protection into the development process from the start, not as an afterthought.

**7 Foundational Principles (Ann Cavoukian):**

1. **Proactive not Reactive; Preventative not Remedial**
   - Anticipate and prevent privacy issues before they occur
   - Security controls built into architecture (encryption, access control)
   - Threat modeling during design phase

2. **Privacy as the Default Setting**
   - Strongest privacy settings enabled by default
   - Students protected by default (no public leaderboards without opt-in)
   - Marketing opt-out by default (must opt-in for newsletters)
   - Minimal data collection by default (email optional)

3. **Privacy Embedded into Design**
   - Not bolted on as add-on feature
   - Privacy considerations in every user story and feature spec
   - Security reviews required for all new features

4. **Full Functionality — Positive-Sum, not Zero-Sum**
   - Privacy AND functionality, not privacy OR functionality
   - Example: Encrypted database doesn't prevent full platform features
   - Generic Students provide educational value without PII collection

5. **End-to-End Security — Full Lifecycle Protection**
   - Data protected from collection through deletion
   - Encryption at rest and in transit
   - Secure data retention and automated deletion
   - Backup encryption and secure disposal

6. **Visibility and Transparency**
   - Clear privacy policy in plain language
   - Users can see what data is collected
   - Audit logs for data access
   - Transparent about sub-processors and data locations

7. **Respect for User Privacy — User-Centric**
   - Self-service data access and export
   - Easy consent withdrawal
   - Control over optional data collection
   - Respect for data subject rights

### Privacy by Default Implementation

**Default Privacy Settings:**

| Feature | Privacy-Protective Default | User Can Change To |
|---------|---------------------------|-------------------|
| Student Email | Not collected | Opt-in with parental consent |
| Public Leaderboard | Disabled | Opt-in (requires parental consent <13) |
| Marketing Emails | Opt-out | Opt-in |
| Usage Analytics (Students) | Disabled for <13 | Not available for students |
| Profile Visibility | Private (teacher only) | Not available for students |
| Generic Student | Recommended for classroom | Regular student with tracking |
| Data Retention | 13 months post-cancellation | Can request earlier deletion |

**Privacy-Protective Architecture Examples:**

**Example 1: Password Storage**
- ❌ **Bad**: Store passwords in plain text or reversible encryption
- ✅ **Good**: Hash passwords with bcrypt (one-way, salted)
- **Privacy Benefit**: Even if database breached, passwords not recoverable

**Example 2: Session Management**
- ❌ **Bad**: Never expire sessions, store session ID in URL
- ✅ **Good**: 30-minute idle timeout, secure httpOnly cookies, CSRF tokens
- **Privacy Benefit**: Reduces risk of unauthorized access if device left unlocked

**Example 3: Data Collection**
- ❌ **Bad**: Collect student home address, phone number "just in case"
- ✅ **Good**: Collect only name and username (email optional)
- **Privacy Benefit**: Minimizes data at risk in event of breach

**Example 4: Logging and Monitoring**
- ❌ **Bad**: Log full student records in application logs
- ✅ **Good**: Log user IDs only, redact PII from logs
- **Privacy Benefit**: Debugging possible without exposing personal data

### Privacy-Enhancing Technologies (PETs)

**Pseudonymization:**
- Replace directly identifying information with pseudonyms
- Example: Student usernames instead of real names in leaderboards
- Reversible with additional information (unlike anonymization)

**Anonymization:**
- Remove all identifying information permanently
- Used for research and aggregate analytics
- Example: "X% of students improved scores" without identifying individuals

**Encryption:**
- AES-256 encryption for data at rest
- TLS 1.3 for data in transit
- Additional encryption layer for sensitive fields (emails)

**Access Controls:**
- Role-based access control (RBAC)
- Principle of least privilege
- Multi-factor authentication for admins

**Audit Logging:**
- All access to personal data logged
- Immutable audit trail
- Regular review for anomalous access patterns

### Privacy Impact in Feature Development

**New Feature Privacy Checklist:**

Before implementing any new feature, assess:

- [ ] **Data Collection**: What personal data does this feature collect?
- [ ] **Legal Basis**: What is the legal basis for processing this data?
- [ ] **Necessity**: Is this data necessary for the feature to function?
- [ ] **Retention**: How long will this data be retained?
- [ ] **Access**: Who will have access to this data?
- [ ] **Security**: What security controls protect this data?
- [ ] **Consent**: Does this require user consent? (If yes, implement consent mechanism)
- [ ] **Data Subject Rights**: How will data subject rights be honored for this data?
- [ ] **Third Parties**: Will this data be shared with third parties? (If yes, DPA required)
- [ ] **Children**: Special considerations for users under 13/16?
- [ ] **DPIA**: Does this feature require a Data Protection Impact Assessment?

**Privacy Review Gate:**
- All new features undergo privacy review before deployment
- DPO or designated privacy lead signs off
- High-risk features require full DPIA

---

## Data Protection Impact Assessments (DPIAs)

### When DPIAs Are Required (Article 35)

**DPIA Required When Processing is Likely to Result in High Risk:**

1. **Systematic and extensive evaluation** of personal aspects based on automated processing (profiling)
2. **Processing on a large scale** of special category data or criminal conviction data
3. **Systematic monitoring** of publicly accessible area on a large scale (e.g., CCTV)

**DPIA Required for MLC-LMS if:**
- Implement AI-powered student profiling or automated assessment
- Collect special category data (health data for accessibility features)
- Systematic monitoring of student behavior for non-educational purposes
- Large-scale processing of children's data with new technology

**Current MLC-LMS Status:**
- **No automated profiling** with legal/significant effects
- **No special category data** collection
- **Educational purpose only** - not systematic surveillance
- **Children's data processed** but with established safeguards
- **DPIA recommended** but not strictly required (precautionary DPIA advised)

### DPIA Process

**Step 1: Describe the Processing**
- What data is collected?
- Why is it collected?
- How is it processed?
- Who has access?
- How long is it retained?
- Where is it stored?
- Who are the data subjects?

**Step 2: Assess Necessity and Proportionality**
- Is the processing necessary for the specified purpose?
- Could the purpose be achieved with less data or different methods?
- Are the privacy intrusions proportionate to the benefits?

**Step 3: Identify and Assess Risks**
- What could go wrong?
- What is the likelihood of each risk?
- What is the severity of impact if risk occurs?
- Risk to data subjects (not business risk)

**Step 4: Identify Measures to Mitigate Risks**
- What technical measures reduce risk? (encryption, access controls)
- What organizational measures reduce risk? (policies, training)
- What legal measures reduce risk? (DPAs, SCCs)

**Step 5: Document and Review**
- Complete DPIA report
- DPO reviews and approves
- Obtain senior management sign-off
- Review periodically (annually or when circumstances change)

**Step 6: Consult Supervisory Authority (if necessary)**
- If residual high risk remains after mitigation, consult DPA before processing
- DPA has 8 weeks to respond (extendable to 14 weeks)
- Must implement DPA's recommendations

### DPIA Template

**Sample DPIA Structure:**

```
DATA PROTECTION IMPACT ASSESSMENT

Project/Feature: [Name]
Date: [Date]
DPO: [Name]
Project Owner: [Name]

1. DESCRIPTION OF PROCESSING
   1.1 Purpose and Nature
   1.2 Data Subjects
   1.3 Personal Data Processed
   1.4 Recipients and Transfers
   1.5 Retention Period
   1.6 Technical Description

2. NECESSITY AND PROPORTIONALITY
   2.1 Legal Basis
   2.2 Necessity Assessment
   2.3 Data Minimization
   2.4 Alternatives Considered
   2.5 Proportionality Justification

3. RISK ASSESSMENT
   3.1 Risk Identification
   3.2 Likelihood and Severity
   3.3 Impact on Data Subjects
   
   Risk Matrix:
   | Risk | Likelihood | Severity | Overall Risk | Mitigation |
   |------|-----------|----------|--------------|------------|
   | Unauthorized access | Medium | High | High | Encryption, MFA |
   | Data breach | Low | High | Medium | Security monitoring |
   | Excessive data collection | Low | Medium | Low | Data minimization review |

4. RISK MITIGATION MEASURES
   4.1 Technical Measures
   4.2 Organizational Measures
   4.3 Legal Measures
   4.4 Residual Risk Assessment

5. CONSULTATION
   5.1 Data Subjects Consulted?
   5.2 DPO Opinion
   5.3 Other Stakeholders
   5.4 Supervisory Authority (if required)

6. SIGN-OFF
   DPO Approval: [Signature] [Date]
   Project Owner: [Signature] [Date]
   Senior Management: [Signature] [Date]

7. REVIEW SCHEDULE
   Next Review Date: [Date]
   Trigger for Ad-Hoc Review: [Major changes to processing]
```

### Example DPIA: Student Progress Tracking

**Processing Description:**
- Collect student game scores, completion times, assignment progress
- Purpose: Track learning progress, generate reports for teachers
- Data subjects: Students (ages 6-18, including children)
- Retention: Life of subscription + 13 months

**Risks Identified:**
1. **Unauthorized access by teacher to student not in their class**
   - Likelihood: Low (access controls)
   - Severity: Medium (privacy violation)
   - Mitigation: Role-based permissions, audit logging

2. **Data breach exposing student scores**
   - Likelihood: Low (security measures)
   - Severity: Medium (embarrassment, but not financial harm)
   - Mitigation: Encryption, security monitoring, incident response plan

3. **Excessive data retention**
   - Likelihood: Medium (without automated deletion)
   - Severity: Low (data not highly sensitive)
   - Mitigation: Automated deletion after 13 months

**Conclusion:**
- Residual risk: Low
- No consultation with supervisory authority required
- Proceed with processing under documented safeguards
- Review annually

### DPIA Register

Maintain register of all DPIAs:

```sql
dpia_register:
  id: uuid (primary key)
  feature_name: string
  processing_description: text
  data_subjects: text
  personal_data_types: jsonb
  legal_basis: string
  risk_level: enum ('low', 'medium', 'high')
  residual_risk_level: enum ('low', 'medium', 'high')
  dpo_approved: boolean
  dpo_approved_date: timestamp
  senior_mgmt_approved: boolean
  supervisory_authority_consulted: boolean
  status: enum ('draft', 'approved', 'implemented', 'archived')
  next_review_date: date
  created_at: timestamp
  updated_at: timestamp
```

---

## Data Protection Officer (DPO)

### DPO Requirement (Article 37)

**DPO Required When:**
1. Processing carried out by public authority (not applicable to MLC)
2. Core activities consist of regular and systematic monitoring of data subjects on a large scale
3. Core activities consist of large-scale processing of special category data or criminal data

**MLC-LMS Assessment:**
- Not public authority
- Educational games are core activity, but not "systematic monitoring" in GDPR sense
- No large-scale processing of special category data
- **DPO not strictly required** under Article 37

**Best Practice: Designate DPO Anyway**
- Demonstrates commitment to GDPR compliance
- Single point of contact for supervisory authorities and data subjects
- Provides expertise and oversight for privacy program
- Marketing advantage ("GDPR-compliant with designated DPO")

### DPO Responsibilities (Article 39)

**Inform and Advise:**
- Inform MLC and employees of GDPR obligations
- Advise on DPIAs and monitor their performance
- Provide training and awareness to staff

**Monitor Compliance:**
- Monitor GDPR compliance and internal policies
- Assign responsibilities and conduct audits
- Maintain records of processing activities

**Cooperate with Supervisory Authority:**
- Serve as contact point for data protection authority
- Facilitate audits and investigations
- Cooperate on prior consultations (Article 36)

**Act as Contact Point for Data Subjects:**
- Receive data subject rights requests
- Provide information on how to exercise rights
- Handle complaints and escalations

### DPO Independence and Resources

**Requirements:**
- DPO reports directly to highest management level
- Not receive instructions regarding performance of DPO tasks
- Not dismissed or penalized for performing DPO duties
- Provided with adequate resources (budget, time, support staff)
- Involved timely in all data protection matters

**Internal vs. External DPO:**

**Internal DPO (Employee):**
- Pros: Institutional knowledge, always available, cultural fit
- Cons: Potential conflict of interest, resource constraints
- Suitable for: Organizations with ongoing, complex data processing

**External DPO (Consultant/DPO-as-a-Service):**
- Pros: Expertise, independence, cost-effective for small organizations
- Cons: Less institutional knowledge, availability limitations
- Suitable for: Small to medium organizations, limited data processing

**MLC Recommendation:** External DPO or DPO-as-a-Service for Phase 1 (cost-effective, expertise). Transition to internal DPO if organization grows significantly.

### DPO Contact Information

**Required Publication:**
- Publish DPO contact details in Privacy Policy
- Make DPO contact info available to supervisory authorities
- Easy for data subjects to contact DPO

**MLC DPO Contact:**
- Email: dpo@musiclearningcommunity.com
- Dedicated inbox monitored daily
- Responses within 5 business days for inquiries
- Within 1 month for data subject rights requests

### DPO Task Register

Track DPO activities:

```sql
dpo_task_register:
  id: uuid (primary key)
  task_type: enum ('advice', 'audit', 'dpia_review', 'training', 'data_subject_request', 'authority_liaison', 'other')
  description: text
  assigned_to: uuid (if delegated)
  priority: enum ('low', 'medium', 'high', 'urgent')
  status: enum ('open', 'in_progress', 'completed', 'deferred')
  due_date: date
  completed_date: date (nullable)
  created_at: timestamp
```

---

## Records of Processing Activities (Article 30)

### GDPR Requirement

**Article 30** requires controllers and processors to maintain written records of all processing activities. This is a foundational accountability measure.

**What Must Be Documented:**
- Name and contact details of controller, DPO, and representative (if applicable)
- Purposes of processing
- Categories of data subjects
- Categories of personal data
- Categories of recipients (including third countries/international organizations)
- International data transfers and safeguards
- Retention periods
- General description of technical and organizational security measures

**Exemption:** Organizations with fewer than 250 employees are exempt UNLESS:
- Processing is likely to result in risk to data subjects
- Processing is not occasional
- Processing includes special category data or criminal conviction data

**MLC-LMS Status:** No exemption applies (processing children's data, not occasional). Must maintain records of processing activities.

### Record of Processing Activities (ROPA)

**MLC-LMS ROPA:**

| ID | Processing Activity | Purpose | Legal Basis | Data Categories | Data Subjects | Recipients | Retention | Security Measures |
|----|-------------------|---------|-------------|-----------------|---------------|-----------|-----------|-------------------|
| ROPA-001 | User Account Management | Provide platform access, authentication | Contract | Name, email, username, password (hashed) | Students, Teachers, Admins | Vercel (hosting), Neon (database) | Active + 13 months | Encryption, MFA, access controls |
| ROPA-002 | Educational Records Processing | Track learning progress, generate reports | Contract, Legal Obligation (FERPA) | Game scores, assignments, progress data | Students | Teachers, Admins, Neon (database) | Active + 13 months | Encryption, RBAC, audit logs |
| ROPA-003 | Subscription & Billing | Process payments, manage subscriptions | Contract | Billing info, payment card (tokenized) | Subscriber-Admins | Braintree (payment processor) | Active + 7 years (financial records) | PCI DSS compliance, tokenization |
| ROPA-004 | Platform Analytics | Improve platform, identify bugs | Legitimate Interests | Usage data, IP address, browser info | Teachers, Admins | Vercel (hosting) | 1 year | Aggregation, no student tracking <13 |
| ROPA-005 | Email Communications | Send notifications, support, marketing (opt-in) | Contract, Consent | Email addresses, message content | All users (opt-in for marketing) | Email provider (TBD) | 3 years | Encryption in transit, access controls |
| ROPA-006 | Parental Consent Management | COPPA/GDPR compliance for children | Legal Obligation | Parental email, consent status, timestamps | Parents/Guardians of students <13/<16 | None | Life of student account + 7 years | Audit logging, immutable records |
| ROPA-007 | Customer Support | Respond to inquiries, troubleshoot | Contract, Legitimate Interests | Support ticket content, user details | All users | Support staff | 3 years | Access controls, confidentiality agreements |
| ROPA-008 | Security Monitoring | Detect and prevent security incidents | Legitimate Interests | IP addresses, login attempts, access logs | All users | Security team | 1 year | Access controls, log anonymization |

**ROPA Maintenance:**
- Updated whenever processing activities change
- Reviewed quarterly by DPO
- Made available to supervisory authority upon request
- Stored securely with access limited to DPO and senior management

**ROPA Storage:**
```sql
records_of_processing:
  id: uuid (primary key)
  ropa_id: string (e.g., 'ROPA-001')
  activity_name: string
  purpose: text
  legal_basis: text
  data_categories: jsonb
  data_subjects: jsonb
  recipients: jsonb
  international_transfers: jsonb
  retention_period: string
  security_measures: text
  last_updated: timestamp
  updated_by: uuid (DPO or admin)
  version: integer
```

---

## Data Retention Policy

### Retention Periods Summary

| Data Category | Retention Period | Justification | Deletion Method |
|--------------|------------------|---------------|-----------------|
| **Active Subscription Data** | Duration of subscription | Contract - necessary for service delivery | N/A (active) |
| **Cancelled Subscription Data** | 13 months post-cancellation | Allow reactivation, legitimate interests | Automated deletion job |
| **Financial Records** | 7 years | Legal obligation - tax/audit requirements | Anonymize PII, retain transaction data |
| **Consent Records** | Life of account + 7 years | Legal obligation - demonstrate compliance | Permanent retention in audit logs |
| **Security/Audit Logs** | 1-3 years depending on type | Legal obligation, legitimate interests | Automated deletion, anonymization |
| **Support Tickets** | 3 years | Legitimate interests - quality improvement | Automated deletion |
| **Encrypted Backups** | 90 days rolling | Disaster recovery | Secure deletion, rotation |
| **Email Communications** | 3 years | Legitimate interests - records of communications | Automated deletion |
| **Parental Consent Verifications** | Life of student account + 7 years | Legal obligation - COPPA/GDPR compliance | Permanent retention |
| **GDPR Breach Records** | Indefinite | Legal obligation - demonstrate compliance | Permanent retention |

### Automated Deletion Procedures

**Daily Background Job:**
```typescript
// Runs daily at 2:00 AM UTC
async function automaticDataDeletion() {
  const today = new Date();
  
  // 1. Delete expired subscriptions (13 months post-cancellation)
  const deletionDate = subMonths(today, 13);
  const expiredSubscriptions = await db.organizations
    .where('subscription_status', 'cancelled')
    .where('cancelled_at', '<', deletionDate)
    .select('id');
  
  for (const org of expiredSubscriptions) {
    await deleteOrganizationData(org.id);
    await logGDPREvent('data_retention_deletion', org.id, {
      reason: 'retention_period_expired',
      deletionDate: today
    });
  }
  
  // 2. Delete old audit logs (1-3 years depending on type)
  await deleteExpiredAuditLogs();
  
  // 3. Delete old support tickets (3 years)
  const ticketDeletionDate = subYears(today, 3);
  await db.support_tickets
    .where('created_at', '<', ticketDeletionDate)
    .delete();
  
  // 4. Rotate backups (90 days)
  await rotateBackups(90);
  
  // 5. Process pending GDPR deletion requests (30-day grace period expired)
  await processPendingErasureRequests();
}
```

**User-Initiated Deletion:**
- Data subject can request deletion via dpo@musiclearningcommunity.com
- 30-day grace period before permanent deletion
- See [Right to Erasure](#right-to-erasure--right-to-be-forgotten-article-17) section

---

## Telemetry

All GDPR-related events are tracked via the platform [Event Model](../15-analytics-and-reporting/event-model.md).

### GDPR Events

**Data Subject Rights Events:**
- `gdpr.access_request.submitted` - Data access request submitted
- `gdpr.access_request.fulfilled` - Data export generated and delivered
- `gdpr.rectification_request.submitted` - Data correction requested
- `gdpr.rectification_request.fulfilled` - Data corrected
- `gdpr.erasure_request.submitted` - Deletion requested
- `gdpr.erasure_request.fulfilled` - Data permanently deleted
- `gdpr.restrict_processing.enabled` - Processing restriction activated
- `gdpr.restrict_processing.disabled` - Processing restriction lifted
- `gdpr.objection.submitted` - Data subject objected to processing
- `gdpr.objection.honored` - Objection honored, processing stopped

**Consent Events:**
- `gdpr.consent.given` - User gave consent for specific processing
  - Properties: `consent_type`, `consent_scope`, `timestamp`, `ip_address`, `method`
- `gdpr.consent.withdrawn` - User withdrew consent
  - Properties: `consent_type`, `withdrawal_timestamp`, `reason`
- `gdpr.consent.renewed` - Consent renewed (if periodic renewal required)

**Breach Events:**
- `gdpr.breach.discovered` - Data breach discovered
  - Properties: `breach_id`, `discovered_at`, `severity`, `affected_users_count`
- `gdpr.breach.authority_notified` - Supervisory authority notified (72-hour requirement)
  - Properties: `breach_id`, `notification_timestamp`, `authority`
- `gdpr.breach.data_subjects_notified` - Data subjects notified
  - Properties: `breach_id`, `notification_timestamp`, `notification_method`

**International Transfer Events:**
- `gdpr.transfer.scc_executed` - Standard Contractual Clauses executed
  - Properties: `party`, `scc_version`, `execution_date`
- `gdpr.transfer.tia_completed` - Transfer Impact Assessment completed
  - Properties: `assessment_date`, `conclusion`, `risk_level`

**DPA Events:**
- `gdpr.dpa.executed` - Data Processing Agreement signed
  - Properties: `processor`, `dpa_version`, `execution_date`, `sccs_included`
- `gdpr.dpa.amended` - DPA amended
  - Properties: `processor`, `amendment_reason`, `amendment_date`

**DPIA Events:**
- `gdpr.dpia.started` - Data Protection Impact Assessment initiated
- `gdpr.dpia.completed` - DPIA completed and approved
  - Properties: `feature_name`, `risk_level`, `dpo_approved`, `completion_date`

**Audit and Compliance Events:**
- `gdpr.audit.conducted` - Internal GDPR audit conducted
  - Properties: `audit_date`, `auditor`, `findings_count`, `compliance_score`
- `gdpr.training.completed` - Staff completed GDPR training
  - Properties: `user_id`, `training_module`, `completion_date`, `score`

### Event Retention

**GDPR Event Logs:**
- Consent events: Retained for life of account + 7 years (legal requirement)
- Access/erasure events: Retained for 5 years
- Breach events: Retained indefinitely (legal requirement)
- Audit events: Retained for 3 years
- Training events: Retained for 3 years

### Event Access Restrictions

**Who Can Access GDPR Events:**
- **System Admin**: Full access to all GDPR events
- **DPO**: Full access to all GDPR events
- **Subscriber-Admin**: Access to events for their organization only
- **External Auditors**: Read-only access with explicit authorization
- **Supervisory Authorities**: Access upon lawful request

---

## Permissions

GDPR compliance permissions are defined by role. See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for complete specifications.

### Data Subject Rights Requests

**Submit Access Request:**
- Any user: Can request access to their own data
- Parents: Can request access to their child's data (<13 or <16 EU)
- Teachers: Cannot request on behalf of students (must be parent/student)

**Submit Erasure Request:**
- Any user: Can request deletion of their own account
- Parents: Can request deletion of child's data
- Subscriber-Admin: Can delete accounts within their organization
- System Admin: Can delete any account (with justification)

**Submit Rectification Request:**
- Any user: Can request correction of inaccurate data
- Self-service correction: Can edit own profile information

**Restrict Processing:**
- Any user: Can request processing restriction
- System Admin: Can implement restriction

**Object to Processing:**
- Any user: Can object to marketing or analytics
- Self-service: Can opt-out of marketing emails

### Data Processing Management

**View Records of Processing Activities (ROPA):**
- DPO: Full access to ROPA
- System Admin: Read access to ROPA
- Senior Management: Read access to ROPA
- Supervisory Authority: Access upon request

**Manage Data Processing Agreements:**
- DPO: Can review and approve DPAs
- Legal Counsel: Can negotiate and sign DPAs
- System Admin: Read access to DPA register

**Conduct DPIAs:**
- DPO: Responsible for conducting and approving DPIAs
- Feature Owner: Provides input for DPIA
- System Admin: Can view completed DPIAs

### Breach Response

**Report Data Breach:**
- Any employee: Can report suspected breach
- System Admin: Triages and assesses breach
- DPO: Decides on notifications, coordinates response

**Notify Supervisory Authority:**
- DPO only: Submits Article 33 notifications

**Notify Data Subjects:**
- DPO: Decides when notification required
- System Admin: Executes notification (sends emails)

### Audit and Compliance

**Generate GDPR Compliance Reports:**
- DPO: Can generate all GDPR reports
- System Admin: Can generate technical compliance reports
- Senior Management: Can view reports

**Access GDPR Audit Logs:**
- DPO: Full access
- System Admin: Full access
- External Auditors: Read-only with authorization

---

## Supporting Documents Referenced

This GDPR Compliance Framework draws from the following source documents:

- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Educational mission, user types, subscription models, and data collection practices for music education platform
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User hierarchy defining roles, capabilities, and data access rights for students, teachers, admins, and system administrators
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Original privacy policy language from MLC 4.0 (March 25, 2012 version) and user-facing content about data practices
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription seat management, Generic Student account specifications, and organizational data handling practices

---

## Dependencies

This specification relies on and references the following systems and specifications:

### Legal and Compliance

**Primary References:**
- [Privacy Policy](./privacy-policy.md) - User-facing privacy policy with GDPR section for EU users
- [Privacy Policy Outline](./privacy-policy-outline.md) - Detailed privacy requirements and specifications
- [COPPA/FERPA Notes](./coppa-ferpa-notes.md) - US child privacy and educational records compliance
- [Terms and Conditions](./terms-conditions.md) - User agreement including data processing terms
- [TOS Outline](./tos-outline.md) - Terms of Service specifications
- [Copyright and Licensing](./copyright-and-licensing.md) - Intellectual property and content rights

### Architecture and Security

**Technical Implementation:**
- [Security and Privacy](../18-architecture/security-privacy.md) - Data encryption, access controls, security architecture
- [Data Model ERD](../18-architecture/data-model-erd.md) - Database schema including GDPR-related tables
- [API Design Principles](../18-architecture/api-design-principles.md) - API security and data access patterns
- [Background Jobs](../18-architecture/background-jobs.md) - Automated data retention and deletion processes
- [Caching Strategy](../18-architecture/caching-strategy.md) - Personal data caching policies

### Roles and Permissions

**Access Control:**
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions and capabilities
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permission specifications
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Authentication and session security
- [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md) - System admin access controls

### User Experience

**User-Facing Interfaces:**
- [Student Dashboard](../03-student-experience/student-dashboard.md) - Student data visibility and controls
- [Student Profile](../03-student-experience/student-profile.md) - Profile data and privacy settings
- [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) - Teacher access to student educational records
- [Member Management](../05-admin-subscriber-experience/member-management.md) - Admin management of user accounts
- [Subscriber Dashboard](../05-admin-subscriber-experience/subscriber-dashboard.md) - Organization-level data access

### Analytics and Telemetry

**Event Tracking:**
- [Event Model](../15-analytics-and-reporting/event-model.md) - GDPR event definitions and tracking
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Reporting with personal data
- [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md) - Organization-level reporting

### Messaging and Notifications

**Communication:**
- [Email Templates](../13-messaging-and-notifications/email-templates.md) - GDPR notification templates, consent emails
- [Communication Framework](../13-messaging-and-notifications/communication-framework.md) - Messaging policies and opt-outs

### Payments and Subscriptions

**Organizational Management:**
- [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) - Braintree integration and DPA requirements
- [Seat Management](../14-payments-and-subscriptions/seat-management.md) - Subscription and user data management

### Data Management

**Import/Export:**
- [Export Specs](../20-imports-and-automation/export-specs.md) - GDPR data export formats (JSON, CSV)
- [Data Migration](../20-imports-and-automation/data-migration.md) - Data handling and retention during migrations

### Quality and Operations

**Operational Procedures:**
- [Incident Response](../19-quality-and-operations/incident-response.md) - Data breach response procedures
- [Observability](../19-quality-and-operations/observability.md) - Monitoring for compliance issues
- [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) - Audit trail for data access

### External Regulations

**Regulatory Authorities:**
- **European Data Protection Board (EDPB)**: GDPR guidance and guidelines
- **National Supervisory Authorities**: Individual EU member state data protection authorities
- **European Commission**: Standard Contractual Clauses, adequacy decisions
- **Article 29 Working Party**: Pre-GDPR guidance (still relevant)

---

## Open Questions

Outstanding decisions to be made before build:

### Data Localization

**EU Data Residency:**
- Should MLC implement EU data residency from day one, or start with US storage + SCCs?
- What is the threshold for implementing EU data residency (% of EU users, revenue, regulatory pressure)?
- If implemented, should EU data residency be mandatory for all EU users, or opt-in?

**Cost-Benefit Analysis:**
- What are the incremental costs of EU data residency (hosting, complexity)?
- What is the competitive advantage of EU data residency vs. SCC approach?
- Can we offer EU data residency as premium feature for EU schools?

### Data Protection Officer

**DPO Selection:**
- Internal employee DPO or external DPO-as-a-Service?
- What are the qualifications required for DPO?
- What is the budget for DPO (salary or consultant fee)?
- Should DPO be full-time or part-time?

**DPO Responsibilities:**
- Should DPO also handle COPPA/FERPA compliance, or separate privacy officer?
- How much time will DPO spend on GDPR vs. other privacy laws?
- What tools and resources does DPO need?

### Consent Management

**Consent Renewal:**
- Should consent be renewed periodically (annually, every 2 years), or last indefinitely until withdrawn?
- For EU children under 16, should consent be re-verified when they turn 16?
- How to handle consent for users who signed up before GDPR implementation?

**Granular Consent:**
- How granular should consent options be? (e.g., separate consents for different processing purposes)
- Should there be a "consent dashboard" where users can see all consents and withdraw selectively?
- How to handle consent when user has multiple roles (teacher who is also parent)?

### Data Subject Rights

**Fee for Excessive Requests:**
- Under what circumstances will MLC charge fee for "manifestly unfounded or excessive" requests?
- What is reasonable fee amount?
- How to determine if request is excessive (e.g., third request in 6 months)?

**Identity Verification:**
- What level of identity verification is required for high-risk requests (erasure, access to child's data)?
- Is email verification sufficient, or require government ID?
- How to handle requests where identity cannot be verified?

### Breach Notification

**Notification Thresholds:**
- For low-severity breaches, should MLC always notify supervisory authority (within 72 hours), or only if "likely to result in risk"?
- What constitutes "high risk" requiring data subject notification (how many users, what data types)?
- Should MLC adopt more conservative thresholds than GDPR requires?

**Breach Communication:**
- Should data subjects be notified via email, platform notification, both?
- If breach involves children, should parents be notified separately?
- Should there be a public breach notification page on website?

### International Transfers

**Standard Contractual Clauses:**
- Should MLC use Controller-to-Controller SCCs, or rely solely on DPAs with schools?
- How to handle situations where SCCs might be challenged (Schrems III)?
- Should MLC prepare alternative transfer mechanisms (e.g., adequacy decision if EU-US framework adopted)?

**Transfer Impact Assessments:**
- How often should TIA be updated (annually, when circumstances change)?
- Should TIA be published publicly for transparency, or kept internal?
- What level of detail should TIA include about US surveillance laws?

### Vendor Management

**Sub-Processor Approval:**
- Should schools have right to object to specific sub-processors (Vercel, Neon, etc.)?
- How much advance notice to provide before changing sub-processors (30 days, 60 days)?
- What happens if school objects to new sub-processor (terminate contract, find alternative)?

**Vendor Audits:**
- How often should MLC audit vendors' GDPR compliance?
- Is vendor's SOC 2 report sufficient, or require independent GDPR audit?
- Who bears cost of audits (MLC, vendor, or shared)?

### Data Protection Impact Assessments

**DPIA Triggers:**
- Beyond GDPR requirements, what internal thresholds trigger DPIA (e.g., processing data of >1,000 children)?
- Should DPIA be required for all new features, or only high-risk features?
- Who has authority to waive DPIA requirement if low risk?

**DPIA Publication:**
- Should completed DPIAs be published for transparency?
- If published, how much detail to include (full DPIA or summary)?
- Should DPIAs be available to data subjects upon request?

### Compliance Monitoring

**GDPR Compliance Metrics:**
- What KPIs should be tracked for GDPR compliance (e.g., % of data subject requests fulfilled within 1 month)?
- Should GDPR compliance metrics be published in annual transparency report?
- What is acceptable target for each metric?

**Regulatory Engagement:**
- Should MLC proactively engage with EU supervisory authorities before launch?
- Should MLC register with lead supervisory authority (if applicable)?
- Should MLC join industry associations for GDPR guidance (IAPP, trade associations)?

---

**Document Version**: 1.0  
**Last Updated**: [Current Date]  
**Next Review Date**: [Current Date + 6 months]  
**Owner**: Data Protection Officer, Legal Team, Product Team