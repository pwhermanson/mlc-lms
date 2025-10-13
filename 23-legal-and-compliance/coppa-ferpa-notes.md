# COPPA and FERPA Notes

This document defines the technical and operational implementation requirements for COPPA (Children's Online Privacy Protection Act) and FERPA (Family Educational Rights and Privacy Act) compliance in the MLC-LMS platform. It provides specific implementation guidance, technical requirements, and operational procedures that complement the user-facing [Privacy Policy](./privacy-policy.md) and [Privacy Policy Outline](./privacy-policy-outline.md).

## Purpose

This specification enables:

- **COPPA Compliance**: Technical implementation of child privacy protections for users under 13
- **FERPA Compliance**: Proper handling and protection of educational records as required for educational institutions
- **Developer Guidance**: Clear technical requirements for implementing privacy controls
- **Operational Procedures**: Step-by-step processes for handling consent, parental requests, and educational records
- **Audit Readiness**: Documentation of compliance measures for regulatory audits
- **Risk Mitigation**: Systematic approach to child data protection and educational records privacy

This document bridges the gap between legal requirements (stated in privacy policy) and technical implementation (detailed in architecture specs).

## Scope

### Included

This specification covers:

- **COPPA Requirements**:
  - Age determination and verification processes
  - Parental consent mechanisms and workflows
  - Verifiable parental consent methods
  - Child data minimization requirements
  - Parental rights implementation (review, delete, refuse collection)
  - Generic Student account specifications
  - Age gate UI and logic

- **FERPA Requirements**:
  - Educational records definition and classification
  - School official exception implementation
  - Directory information handling
  - Parental/eligible student access rights
  - Consent requirements for disclosures
  - Annual notification requirements
  - Record amendment procedures

- **Technical Implementation**:
  - Database schema additions for consent tracking
  - Age-related flags and data fields
  - Consent verification workflows
  - Data access restrictions based on age
  - Audit logging for child data access
  - Deletion cascades for child accounts

- **Operational Procedures**:
  - Parental request handling workflows
  - Identity verification procedures
  - Consent documentation requirements
  - Educational records access procedures
  - Compliance reporting processes

### Excluded

This specification does not cover:

- **Legal Policy Language**: Covered in [Privacy Policy](./privacy-policy.md)
- **Privacy Policy Structure**: Covered in [Privacy Policy Outline](./privacy-policy-outline.md)
- **GDPR-Specific Requirements**: Covered in [GDPR Compliance](./gdpr-compliance.md)
- **General Security Architecture**: Covered in [Security and Privacy](../18-architecture/security-privacy.md)
- **User Interface Design**: Covered in [UX Design System](../01-ux-design-system/) and role-specific experience docs
- **Terms of Service**: Covered in [Terms and Conditions](./terms-conditions.md) and [TOS Outline](./tos-outline.md)

## COPPA (Children's Online Privacy Protection Act)

### Regulatory Overview

**COPPA Applies When**:
- Platform is directed to children under 13, OR
- Platform has actual knowledge that users are under 13

**MLC-LMS Status**: Our platform is directed to children ages 6-18, therefore COPPA applies to users under 13.

**Key COPPA Requirements**:
1. Direct notice to parents about data collection practices
2. Verifiable parental consent before collecting PII from children under 13
3. Allow parents to review, delete, or refuse further collection of child's information
4. Maintain confidentiality and security of children's information
5. Retain children's PII only as long as necessary

### Age Determination and Verification

#### Age Gate Implementation

**Where Age Gates Appear**:
- Student account creation flow (teacher/admin creating student)
- Generic Student vs. Regular Student selection
- Optional features requiring age verification (email, public leaderboards)

**Age Gate Options**:

**Option 1: Teacher Certification (Recommended for MVP)**
- Teacher certifies student is under/over 13 via checkbox
- Teacher bears responsibility for accuracy
- Simple implementation, minimal friction
- Relies on school's existing age verification

**Option 2: Date of Birth Collection (Future Enhancement)**
- Optional date of birth field during student creation
- System automatically determines if student is under 13
- More accurate but collects additional PII
- Requires parental consent to collect DOB itself

**Option 3: No Age Collection (Generic Student Only)**
- Generic Student accounts bypass age determination entirely
- No PII collected, no age verification needed
- Suitable for classroom settings only

**Recommended Approach**:
- MVP: Option 1 (Teacher Certification)
- Future: Option 2 (DOB) as optional enhancement

#### Database Schema for Age Tracking

**`students` Table Additions**:
```sql
students:
  id: uuid (primary key)
  username: string (required)
  first_name: string (required)
  last_name: string (required)
  email: string (nullable) -- requires parental consent if under 13
  date_of_birth: date (nullable)
  is_under_13: boolean (nullable) -- TRUE, FALSE, or NULL (unknown)
  is_generic: boolean (default: false) -- Generic Student account flag
  parental_consent_status: enum (nullable) -- see below
  parental_consent_date: timestamp (nullable)
  parental_consent_method: string (nullable) -- 'teacher_certification', 'email_verification'
  parent_email: string (nullable) -- for consent verification
  parent_consent_verified_at: timestamp (nullable)
  parent_consent_verified_by: uuid (nullable) -- user_id who verified
  created_by: uuid (foreign key to users.id)
  created_at: timestamp
  updated_at: timestamp
```

**`parental_consent_status` Enum Values**:
- `not_required` - Student is over 13 or Generic Student
- `pending_certification` - Teacher needs to certify consent
- `certified_by_teacher` - Teacher certified parental consent obtained
- `pending_verification` - Email sent to parent, awaiting confirmation
- `verified` - Parent confirmed consent via email
- `declined` - Parent declined to provide consent
- `expired` - Consent needs renewal (if implementing annual renewal)

#### Age-Related Data Restrictions

**Data Collection Restrictions for Users Under 13**:

**Allowed Without Parental Consent**:
- First name, last name (required for educational purpose)
- Username (required for authentication)
- Game scores and educational progress (FERPA educational records)
- Assignment completion and mastery tracking
- Session data (IP address, login times) for security
- Teacher assignment and class membership

**Requires Parental Consent**:
- Email address (optional)
- Date of birth (optional)
- Public leaderboard participation
- Profile photo (if feature added)
- Any non-essential personal information

**Prohibited for Users Under 13**:
- Behavioral tracking across other websites
- Advertising cookies or marketing data
- Precise geolocation beyond city/state
- Social media connections
- Contact lists or friend requests (unless COPPA-compliant social features added)

#### Validation Rules

**Backend Validation**:
```typescript
// Example validation logic
function validateStudentCreation(studentData, createdBy) {
  if (studentData.is_under_13 === true || studentData.is_under_13 === null) {
    // Under 13 or age unknown - apply COPPA restrictions
    
    if (studentData.email && !studentData.parent_email) {
      throw new ValidationError('Parent email required for students under 13 with email addresses');
    }
    
    if (studentData.parental_consent_status === null) {
      throw new ValidationError('Parental consent status must be specified for students under 13');
    }
    
    // Ensure no prohibited data is being collected
    prohibitedFields = ['phone', 'address', 'social_media', 'profile_photo'];
    if (hasAnyField(studentData, prohibitedFields)) {
      throw new ValidationError('Cannot collect prohibited data for students under 13');
    }
  }
  
  if (studentData.is_generic === true) {
    // Generic Student - ensure no PII collected
    if (studentData.email || studentData.date_of_birth || studentData.parent_email) {
      throw new ValidationError('Generic Student accounts cannot collect PII');
    }
    studentData.parental_consent_status = 'not_required';
  }
  
  return true;
}
```

### Verifiable Parental Consent

#### Consent Methods

**Method 1: Teacher Certification (Primary Method)**

**Process**:
1. Teacher creates student account
2. System detects student is/may be under 13
3. Teacher presented with certification checkbox:
   - "I certify that I have obtained verifiable parental consent for this student under the age of 13, as required by COPPA."
4. Teacher checks box to proceed
5. System records consent with timestamp and certifying user ID
6. Student account created with `parental_consent_status = 'certified_by_teacher'`

**Acceptable for COPPA Because**:
- Teachers act as agents of the school
- Schools obtain parental consent during enrollment
- Teacher certification relies on school's existing consent documentation
- Common practice in educational technology

**Documentation Requirements**:
- Teachers must maintain documentation of parental consent
- Schools should have consent forms signed by parents
- Platform provides consent form templates (see UX Requirements section)

**Method 2: Email Verification (Secondary Method)**

**Process**:
1. Teacher creates student account and provides parent email
2. Teacher certifies consent (Method 1)
3. System sends verification email to parent (optional enhancement)
4. Parent receives email with:
   - Explanation of data collection
   - Link to privacy policy
   - "Verify Consent" button with unique token
   - "Decline Consent" button
5. Parent clicks "Verify Consent"
6. System updates: `parental_consent_status = 'verified'`, `parent_consent_verified_at = NOW()`
7. Optional: Send confirmation email to teacher and parent

**When Email Verification is Triggered**:
- Automatically if parent email is provided
- Manually by system admin or compliance team
- Annually for consent renewal (if policy requires)

**Email Verification Token**:
```sql
parental_consent_verifications:
  id: uuid (primary key)
  student_id: uuid (foreign key to students.id)
  parent_email: string (required)
  verification_token: string (unique, random, 32 chars)
  sent_at: timestamp
  verified_at: timestamp (nullable)
  declined_at: timestamp (nullable)
  expires_at: timestamp (default: sent_at + 30 days)
  ip_address: string (recorded when verified/declined)
  user_agent: string (recorded when verified/declined)
```

**Token Generation**:
- Cryptographically secure random token
- Unique per student per request
- Expires after 30 days
- Single use (cannot be reused after verification)

**Email Template** (see [Email Templates](../13-messaging-and-notifications/email-templates.md) for full template):
```
Subject: Verify Parental Consent for [Student Name] - Music Learning Community

Dear Parent/Guardian,

[Teacher Name] has created an account for [Student Name] on Music Learning Community, an educational music platform. 

Because [Student Name] is under 13 years of age, we are required by federal law (COPPA) to obtain your consent before collecting certain information.

What information we collect:
- Student name and username (required)
- Game scores and learning progress (educational purpose)
- [Student email] (optional - only if you consent)

Please click below to verify your consent, or to decline:

[Verify Consent Button] [Decline Consent Button]

This verification link expires in 30 days.

For questions, contact: coppa@musiclearningcommunity.com

Privacy Policy: [link]
```

#### Consent Withdrawal

**Parent-Initiated Withdrawal**:

**Process**:
1. Parent contacts platform via email/phone
2. System admin verifies parent identity (see Identity Verification section)
3. System admin marks consent as withdrawn
4. Student account options:
   - **Option A (Recommended)**: Convert to Generic Student (loses individual tracking)
   - **Option B**: Delete account and all data (if parent requests)
   - **Option C**: Suspend account pending resolution

**Database Update**:
```sql
UPDATE students 
SET 
  parental_consent_status = 'withdrawn',
  email = NULL, -- remove any PII requiring consent
  parent_email = NULL,
  consent_withdrawn_at = NOW(),
  consent_withdrawn_by = <system_admin_user_id>
WHERE id = <student_id>;

-- Log event
INSERT INTO audit_logs (event_type, user_id, target_id, details) 
VALUES ('parental_consent_withdrawn', <admin_id>, <student_id>, <details>);
```

**Teacher/Admin Notification**:
- Email sent to teacher and subscriber-admin
- In-app notification when they view student
- Message: "Parental consent withdrawn. Student converted to limited access pending resolution."

#### Generic Student Accounts (COPPA-Safe Alternative)

**Purpose**: Provide COPPA-compliant option for classroom settings where individual student tracking is not needed.

**Characteristics**:
- Shared username/password (e.g., "musicclass1", "thirdgrade2")
- No PII collected (no names, emails, or individual identifiers)
- Scores recorded per username but not tied to individuals
- Multiple students use same login
- No parental consent required (no PII = no COPPA requirements)

**Use Cases**:
- General music classes in K-12
- Group lessons or camps
- Trial periods before individual accounts
- Classes where teacher only needs aggregate progress

**Database Implementation**:
```sql
-- Generic Student record
students:
  id: uuid
  username: "musicclass1"
  first_name: "Music" -- generic placeholder
  last_name: "Class 1" -- generic placeholder
  email: NULL
  is_generic: TRUE
  is_under_13: NULL -- not applicable
  parental_consent_status: 'not_required'
  created_by: <teacher_id>
```

**Backend Restrictions for Generic Students**:
- Cannot collect any PII fields
- Cannot send emails (no email address)
- Cannot participate in public leaderboards (no individual identity)
- Can play all games and record scores (aggregate)
- Can view aggregate scores for that username

**UI Indicators**:
- Badge: "Generic Student" on student list
- Help text: "Shared account used by multiple students"
- Teacher reports show username but indicate "Generic" type

### Parental Rights Implementation

#### Right to Review

**Parent Request Process**:
1. Parent contacts platform (email/phone)
2. Identity verification performed (see Identity Verification section)
3. System admin generates data export for student
4. Data export includes:
   - Student profile information
   - All game scores and completion data
   - Assignment history
   - Login history (dates/times, no detailed session data)
   - Badge and achievement data
   - Any messages sent by student
5. Data sent to verified parent email as PDF or CSV
6. Fulfillment logged in audit trail

**Self-Service (Future Enhancement)**:
- Parent account feature allowing direct data access
- Parent logs in, views student data dashboard
- Automated, no admin intervention required

**Timeline**: Within 30 days of verified request (COPPA requirement)

#### Right to Delete

**Parent Request Process**:
1. Parent requests deletion via email/phone
2. Identity verification performed
3. System admin initiates deletion process
4. Options presented to parent:
   - **Full Deletion**: Permanent deletion of all student data
   - **Conversion to Generic**: Remove PII, keep aggregated data
5. Parent confirms choice
6. Deletion executed (see Deletion Process section)
7. Confirmation sent to parent
8. Teacher/admin notified

**Deletion Process**:
```sql
-- Option 1: Full Deletion
DELETE FROM assignment_completions WHERE student_id = <student_id>;
DELETE FROM game_scores WHERE student_id = <student_id>;
DELETE FROM student_badges WHERE student_id = <student_id>;
DELETE FROM student_messages WHERE student_id = <student_id>;
DELETE FROM students WHERE id = <student_id>;

-- Option 2: Convert to Generic (PII removal)
UPDATE students 
SET 
  first_name = 'Former',
  last_name = 'Student',
  username = 'deleted_' || id::text,
  email = NULL,
  date_of_birth = NULL,
  parent_email = NULL,
  is_generic = TRUE,
  parental_consent_status = 'not_required',
  deletion_requested_at = NOW(),
  deleted_at = NOW()
WHERE id = <student_id>;
```

**Cascading Deletions**:
- Game scores: DELETED
- Assignments: Unassigned or deleted
- Messages: Deleted (both sent and received)
- Group memberships: Removed
- Session tokens: Invalidated
- Cached data: Purged

**Exceptions (Not Deleted)**:
- Financial records (required for tax/audit - 7 years)
- Anonymized aggregated data used for research
- Security logs (anonymized)

**Timeline**: Within 30 days of verified request

#### Right to Refuse Further Collection

**Parent Request Process**:
1. Parent requests to stop data collection
2. Identity verification performed
3. System admin updates account to "limited mode":
   - Set `data_collection_refused = TRUE`
   - Remove optional PII (email)
   - Maintain required data only (name, scores for current assignments)
4. Account functionality limited:
   - Can continue assigned work
   - Cannot collect new optional data
   - No public features (leaderboards)
5. Teacher notified of limitations

**Implementation**:
```sql
ALTER TABLE students ADD COLUMN data_collection_refused BOOLEAN DEFAULT FALSE;

-- When parent refuses further collection
UPDATE students 
SET 
  data_collection_refused = TRUE,
  email = NULL, -- remove optional PII
  parental_consent_status = 'refused',
  data_collection_refused_at = NOW()
WHERE id = <student_id>;
```

**Behavioral Changes When `data_collection_refused = TRUE`**:
- No new optional fields can be added
- Public leaderboards disabled for this student
- Messages sent to teacher only (no peer messages)
- Assignment scores still tracked (required educational purpose)
- Analytics events limited to essential only

### Identity Verification for Parental Requests

**Challenge**: Verify that requester is actually the parent/guardian before providing child's data or making changes.

**Acceptable Verification Methods**:

**Method 1: Teacher/Admin Confirmation**
- Parent contacts teacher or subscriber-admin directly
- Teacher verifies parent identity (school has existing relationship)
- Teacher/admin submits request on parent's behalf
- Lowest friction, relies on school's existing verification

**Method 2: Email Verification**
- Request sent from email address on file (parent_email)
- Verification link sent to that email
- Parent clicks link to confirm identity
- Suitable for: Review data, minor changes

**Method 3: Knowledge-Based Verification**
- Parent provides information only they would know:
  - Student username (not publicly visible)
  - Date of birth
  - Recent game scores or assignments
  - School or teacher name
- System admin verifies information matches
- Suitable for: Deletion requests, consent withdrawal

**Method 4: Copy of ID (High-Security)**
- Parent emails copy of government-issued ID
- System admin verifies name matches student's parent
- Used for: Major changes, legal disputes
- ID securely deleted after verification

**Method 5: Parental Account (Future Enhancement)**
- Parent creates verified parent account during enrollment
- Multi-factor authentication required
- Parent can self-service all rights
- Best user experience, requires development

**Recommended Approach**:
- MVP: Methods 1, 2, and 3
- Future: Method 5 (parent accounts)
- Method 4 reserved for edge cases

### COPPA-Specific UI Requirements

#### Student Account Creation Flow

**Step 1: Student Type Selection**
```
Create New Student

○ Regular Student (Individual tracking with scores and progress)
○ Generic Student (Shared account for multiple students, no individual data)

[ What's the difference? ] -- help link
```

**Step 2: Student Information (Regular Student)**
```
Student Information

First Name: [________] *
Last Name:  [________] *
Username:   [________] * (will be used for login)
Password:   [________] * (minimum 8 characters)

Optional Information:
Email:      [________] (requires parental consent if student is under 13)

Is this student under 13 years old?
○ Yes, under 13
○ No, 13 or older
○ Unknown

[Next Step]
```

**Step 3: Parental Consent (if Under 13)**
```
Parental Consent Required

This student is under 13 years old. Federal law (COPPA) requires that we
obtain verifiable parental consent before collecting certain information.

☐ I certify that I have obtained verifiable parental consent from this 
  student's parent or guardian as required by COPPA.

Optional: Verify consent via email
Parent Email: [________] (we'll send verification link to parent)

[ Learn more about COPPA requirements ]

[Cancel]  [Create Student Account]
```

**Step 4: Confirmation**
```
Student Account Created

[Student Name] has been added to your class.

[if email provided]
A verification email has been sent to [parent email]. The parent must
verify consent within 30 days.

[View Student]  [Add Another Student]
```

#### Student List Indicators

**Visual Indicators for Consent Status**:
- 🟢 Green check: Consent verified
- 🟡 Yellow warning: Consent pending verification
- 🔴 Red alert: Consent declined or expired
- ⚪ Gray: Over 13, consent not required
- 👥 Generic Student icon

**Example Student List**:
```
Students                                            Consent Status
─────────────────────────────────────────────────────────────────
🟢 Smith, Jane (jsmith)                            Verified
🟡 Doe, John (jdoe)                                Pending Email
⚪ Williams, Sarah (swilliams)                     Over 13
👥 Music Class 1 (musicclass1)                     Generic (No PII)
🔴 Brown, Michael (mbrown)                         ⚠️ Action Needed
```

**"Action Needed" Scenarios**:
- Consent pending > 30 days
- Parent declined consent
- Consent expired (if annual renewal required)

#### Privacy Settings Panel (Student)

**For Students Under 13**:
```
Privacy Settings - [Student Name]

Parental Consent Status: ✓ Verified on [date]
Parent Email: parent@example.com

Data Collection:
✓ Name and username (required for education)
✓ Game scores and progress (required for education)
✗ Email address (not collected - no parental consent)
✗ Public leaderboard participation (requires parental consent)

Parent Rights:
Parents can review, download, or delete this student's data at any time.
Contact: coppa@musiclearningcommunity.com

[Request Data Export for Parent]  [Contact Parent]
```

**For Generic Students**:
```
Privacy Settings - Music Class 1

Account Type: Generic Student (Shared Account)

This is a shared account used by multiple students. No personally
identifiable information is collected.

Data Collected:
✓ Shared username only
✓ Aggregate game scores (not tied to individuals)
✗ No names, emails, or personal information

COPPA Compliance: Not applicable (no PII collected)
```

### COPPA Audit and Reporting

#### Required Documentation

**Consent Records**:
- All parental consent records retained for life of account + 7 years
- Includes: timestamp, method, certifying user, IP address
- Immutable audit trail

**Deletion Logs**:
- All child account deletions logged
- Retained for 7 years after deletion
- Includes: reason, requestor, what was deleted

**Access Logs**:
- Who accessed child data and when
- Especially for system admin access
- Retained for 1 year minimum

#### Compliance Reports

**Monthly COPPA Compliance Report**:
- Total students under 13
- Consent status breakdown (verified, pending, declined)
- Pending verifications > 30 days (needs follow-up)
- Parental requests received and fulfilled
- Average fulfillment time

**Annual COPPA Audit Checklist**:
- [ ] All students under 13 have documented parental consent
- [ ] No prohibited data collected for children under 13
- [ ] Parental rights procedures documented and tested
- [ ] Consent verification emails reviewed for compliance
- [ ] Data retention policies followed
- [ ] Third-party vendors COPPA-compliant
- [ ] Privacy policy updated and accurate
- [ ] Staff trained on COPPA requirements

#### System Admin Tools

**COPPA Compliance Dashboard**:
```
COPPA Compliance Overview

Students Under 13: 1,247
─────────────────────────────────────────────────
Consent Verified:           1,089 (87%)
Pending Email Verification:   142 (11%)
Action Required:               16 (1%)

Recent Parental Requests:
─────────────────────────────────────────────────
5 requests in last 30 days
4 completed within SLA (30 days)
1 pending (15 days old)

[View Details]  [Generate Report]
```

## FERPA (Family Educational Rights and Privacy Act)

### Regulatory Overview

**FERPA Applies When**:
- Institution receives federal education funding, AND
- Institution maintains educational records

**MLC-LMS Status**: We process educational records on behalf of schools that are subject to FERPA.

**Key FERPA Requirements**:
1. Schools must provide annual notification of FERPA rights
2. Parents/eligible students can inspect and review educational records
3. Schools must obtain consent before disclosing educational records (with exceptions)
4. Parents/eligible students can request amendment of inaccurate records
5. Schools must maintain records of disclosures

### What Constitutes an "Educational Record" Under FERPA

**Educational Records Include**:
- Student game scores and performance data
- Assignment completion records
- Progress and mastery tracking
- Teacher notes and assessments about student learning
- Learning analytics tied to individual students
- Any record directly related to a student maintained by the educational institution

**NOT Educational Records**:
- Generic Student data (no individual student identified)
- Sole possession records (teacher's private notes not shared)
- Law enforcement records
- Employment records
- Records created after student is no longer in attendance
- Grades on peer-graded papers before collected by teacher

**MLC-LMS Educational Records**:
```
students table:
  - All fields (name, username, etc.)
  
game_scores table:
  - All game play results tied to individual students
  
assignment_completions table:
  - All assignment progress and completion data
  
teacher_notes table (if implemented):
  - Notes and assessments by teachers about students
  
student_badges table:
  - Achievement tracking
  
Any data that is:
  1. Directly related to a student, AND
  2. Maintained by or for the school
```

### School Official Exception

**MLC-LMS as School Official**:
- We process educational records on behalf of the school
- We have legitimate educational interest
- We use data only for purposes specified by the school
- We maintain appropriate security and confidentiality

**Requirements for School Official Exception**:
1. Written agreement with school (Data Processing Agreement)
2. Use data only for school's authorized purposes
3. Do not re-disclose records without consent
4. Maintain security and confidentiality
5. Destroy or return data when no longer needed

**Data Processing Agreement (DPA) Must Include**:
- Purpose of data processing
- Types of educational records accessed
- Security measures implemented
- Prohibition on re-disclosure
- Data retention and destruction schedule
- Audit rights for the school
- Breach notification procedures

See [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) for DPA template requirements.

### FERPA Data Access Controls

#### Who Can Access Educational Records

**Teachers**:
- Can access records for students in their classes only
- Legitimate educational interest
- Cannot access students not assigned to them
- See [Permissions Table](../02-roles-and-permissions/permissions-table.md)

**Subscriber-Admin**:
- Can access records for all students in their organization
- Must have legitimate educational interest
- Access logged for audit purposes

**System Admin**:
- Can access any educational records for support and compliance purposes
- All access logged with justification
- Subject to audit
- See [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md)

**Third-Party Vendors**:
- Only vendors with signed DPAs
- Access limited to necessary data only
- School Official Exception applies
- Examples: Vercel (hosting), Neon (database)

**Implementation**:
```typescript
// Access control check before returning student educational records
async function getStudentData(requestingUserId, studentId) {
  const student = await db.students.findById(studentId);
  const requestingUser = await db.users.findById(requestingUserId);
  
  // Check access rights
  const hasAccess = await checkEducationalRecordAccess(
    requestingUser,
    student
  );
  
  if (!hasAccess) {
    await logAccessDenied(requestingUserId, studentId, 'FERPA');
    throw new ForbiddenError('No legitimate educational interest');
  }
  
  // Log access for audit
  await logEducationalRecordAccess(
    requestingUserId,
    studentId,
    'read',
    'student_data_view'
  );
  
  return student;
}

async function checkEducationalRecordAccess(user, student) {
  if (user.role === 'system_admin') return true; // always allowed
  
  if (user.role === 'subscriber_admin' || user.role === 'teacher_admin') {
    // Check if student belongs to their organization
    return student.organization_id === user.organization_id;
  }
  
  if (user.role === 'teacher') {
    // Check if student is in any of teacher's classes
    const teacherStudents = await db.class_enrollments
      .where({ teacher_id: user.id, student_id: student.id })
      .first();
    return !!teacherStudents;
  }
  
  return false; // deny by default
}
```

### Parent and Eligible Student Rights

#### Right to Inspect and Review

**Who Has Rights**:
- **K-12**: Parents have rights until student turns 18
- **Higher Ed**: Students (18+) have rights (eligible students)
- **Homeschool**: Parent acting as both parent and school

**Request Process**:
1. Parent/eligible student requests access
2. School or MLC verifies identity
3. Educational records provided within 45 days (FERPA requirement)
4. Access can be via:
   - Online dashboard (self-service)
   - PDF export
   - In-person review (if applicable)

**Self-Service Access** (Recommended):
- Teachers provide dashboard access to parents (view-only mode)
- Parents can log in with special access code
- View all scores, assignments, progress
- Export data as PDF or CSV
- No admin intervention needed

**Admin-Facilitated Access**:
- Parent contacts school or MLC
- Identity verified
- Admin generates complete data export
- Sent via secure email or made available for download

**What Must Be Provided**:
- All student educational data in MLC-LMS
- Explanation of any codes or abbreviations used
- Opportunity to discuss records with school official
- Copies provided at reasonable cost or free

#### Right to Request Amendment

**When Parents/Students Can Request Amendment**:
- Record is inaccurate
- Record is misleading
- Record violates student's privacy rights

**Amendment Request Process**:
1. Parent/eligible student submits amendment request
2. Request includes:
   - Specific record to be amended
   - Reason why record is inaccurate/misleading
   - Proposed correction
3. School/MLC reviews request within 30 days
4. If amendment approved: Record corrected and parties notified
5. If amendment denied: Parent notified with explanation and right to hearing

**Implementation**:
```typescript
// Amendment request tracking
amendment_requests:
  id: uuid (primary key)
  student_id: uuid (foreign key)
  requested_by: uuid (parent/student user_id)
  record_type: string ('game_score', 'assignment', 'student_profile')
  record_id: uuid (id of specific record)
  current_value: jsonb (snapshot of current data)
  requested_change: jsonb (proposed new data)
  reason: text
  status: enum ('pending', 'approved', 'denied', 'hearing_requested')
  reviewed_by: uuid (system admin who reviewed)
  review_notes: text
  created_at: timestamp
  reviewed_at: timestamp
```

**Note**: Most amendment requests in educational platforms involve correcting data entry errors (wrong name spelling, incorrect scores due to technical issues). Substantive disputes about grades or teacher assessments are rare and handled by the school, not the platform.

#### Consent for Disclosure

**When Consent is Required**:
- FERPA requires written consent before disclosing educational records to third parties
- Exceptions exist for school officials, authorized audits, safety emergencies

**Exceptions (No Consent Required)**:
1. **School Officials** with legitimate educational interest (MLC-LMS under this exception)
2. **Other Schools** to which student is transferring
3. **Authorized Audits** by federal/state education agencies
4. **Financial Aid** determinations
5. **Health and Safety Emergencies**
6. **Court Orders** or lawfully issued subpoenas
7. **Directory Information** (if designated and parent didn't opt out)

**Directory Information**:
- Name, address, phone, email, dates of attendance, grade level, awards
- Can be disclosed without consent IF:
  - School designated as directory information in annual notice
  - Parents given opportunity to opt out
  - MLC-LMS recommendation: Do NOT treat student data as directory information

**Implementation for Third-Party Disclosure**:
```sql
-- Track all disclosures (FERPA requires disclosure log)
ferpa_disclosures:
  id: uuid (primary key)
  student_id: uuid (foreign key)
  disclosed_to: string (recipient name/organization)
  disclosure_type: enum ('school_official', 'audit', 'emergency', 'court_order', 'other')
  purpose: text (reason for disclosure)
  records_disclosed: jsonb (what data was shared)
  consent_obtained: boolean (true if parent consented)
  consent_record_id: uuid (link to consent record if applicable)
  disclosed_by: uuid (system admin who authorized)
  disclosed_at: timestamp
  legal_basis: text (FERPA exception or consent)
```

**Parental Consent Form** (if disclosure requires consent):
```
FERPA Consent for Disclosure of Educational Records

I, [Parent Name], parent/guardian of [Student Name], hereby consent to the 
disclosure of my child's educational records to:

Recipient: [Organization/Person Name]
Purpose: [Why records are being disclosed]
Records to be disclosed: [Specific data]

I understand that:
- This consent is voluntary
- I may revoke this consent at any time
- The school will not disclose records without my consent except as permitted by FERPA
- The recipient may not re-disclose these records without my consent

Signature: _____________ Date: _______
```

### Annual FERPA Notification

**Requirement**: Schools must notify parents/eligible students annually of their FERPA rights.

**MLC-LMS Role**:
- We do not send annual notifications (school's responsibility)
- We provide notification template schools can customize
- We ensure platform implements all rights described in notification

**Template for Schools** (provided by MLC-LMS):
```
Annual Notification of FERPA Rights

The Family Educational Rights and Privacy Act (FERPA) affords parents and 
students who are 18 years of age or older ("eligible students") certain rights 
with respect to the student's education records. These rights include:

1. The right to inspect and review the student's education records within 45 days 
   of the request.

2. The right to request the amendment of records that are believed to be inaccurate, 
   misleading, or otherwise in violation of the student's privacy rights.

3. The right to provide written consent before the school discloses personally 
   identifiable information (PII) from the student's education records, except to 
   the extent that FERPA authorizes disclosure without consent.

4. The right to file a complaint with the U.S. Department of Education concerning 
   alleged failures by the school to comply with FERPA requirements.

For our use of Music Learning Community educational platform, student educational 
records include: [list what's collected]

To exercise these rights or for questions, contact: [School Contact Information]

For more information about FERPA, see: https://studentprivacy.ed.gov/
```

### Data Retention and Destruction

**FERPA Requirements**:
- Schools must destroy educational records when no longer needed
- Must not retain records indefinitely
- Must have documented retention policy

**MLC-LMS Retention Policy**:
- **Active Subscriptions**: Retained while subscription active
- **Cancelled Subscriptions**: Retained 13 months post-cancellation
- **After 13 Months**: Automatic permanent deletion
- **School Requests Early Destruction**: Honored within 30 days
- **Legal Hold**: Records preserved if subject to legal process

**Data Destruction Process**:
```typescript
// Background job runs daily
async function deleteExpiredEducationalRecords() {
  const expirationDate = new Date();
  expirationDate.setMonth(expirationDate.getMonth() - 13);
  
  // Find students from cancelled subscriptions > 13 months old
  const expiredStudents = await db.students
    .join('organizations', 'students.organization_id', 'organizations.id')
    .where('organizations.subscription_status', 'cancelled')
    .where('organizations.cancelled_at', '<', expirationDate)
    .select('students.id');
  
  for (const student of expiredStudents) {
    // Check for legal hold
    const legalHold = await db.legal_holds
      .where({ student_id: student.id, active: true })
      .first();
    
    if (legalHold) {
      console.log(`Skipping deletion for student ${student.id} - legal hold`);
      continue;
    }
    
    // Permanent deletion
    await permanentlyDeleteStudent(student.id);
    
    // Log deletion
    await db.audit_logs.insert({
      event_type: 'ferpa_record_destroyed',
      target_id: student.id,
      details: { reason: 'retention_period_expired', date: new Date() }
    });
  }
}

async function permanentlyDeleteStudent(studentId) {
  // Delete all related educational records
  await db.game_scores.where({ student_id: studentId }).delete();
  await db.assignment_completions.where({ student_id: studentId }).delete();
  await db.student_badges.where({ student_id: studentId }).delete();
  await db.student_messages.where({ student_id: studentId }).delete();
  await db.class_enrollments.where({ student_id: studentId }).delete();
  await db.students.where({ id: studentId }).delete();
  
  // Purge from backups (marked for exclusion in next backup cycle)
  await markForPurge(studentId);
}
```

**School-Requested Data Return or Destruction**:
- Schools can request data export before destruction
- MLC exports all student data in standard format
- After school confirms receipt, MLC destroys records
- Process documented and confirmed via email

## Telemetry

All COPPA and FERPA related events are tracked via the platform [Event Model](../15-analytics-and-reporting/event-model.md).

### COPPA Events

**Consent Tracking**:
- `coppa.parental_consent.certified` - Teacher certifies parental consent obtained
  - Properties: `student_id`, `teacher_id`, `consent_method` ('teacher_certification'), `timestamp`
- `coppa.parental_consent.email_sent` - Verification email sent to parent
  - Properties: `student_id`, `parent_email`, `verification_token`, `sent_at`
- `coppa.parental_consent.verified` - Parent verified consent via email
  - Properties: `student_id`, `parent_email`, `verification_token`, `verified_at`, `ip_address`
- `coppa.parental_consent.declined` - Parent declined consent
  - Properties: `student_id`, `parent_email`, `reason`, `declined_at`
- `coppa.parental_consent.withdrawn` - Parent withdrew consent
  - Properties: `student_id`, `withdrawn_by`, `action_taken` ('converted_generic', 'deleted'), `timestamp`

**Parental Rights Exercised**:
- `coppa.parent_request.review` - Parent requested to review child's data
  - Properties: `student_id`, `parent_email`, `verification_method`, `requested_at`
- `coppa.parent_request.export` - Data export generated for parent
  - Properties: `student_id`, `export_format`, `file_size`, `completed_at`
- `coppa.parent_request.delete` - Parent requested deletion of child's data
  - Properties: `student_id`, `deletion_type` ('full', 'convert_generic'), `requested_at`
- `coppa.parent_request.refuse_collection` - Parent refused further data collection
  - Properties: `student_id`, `limitations_applied`, `timestamp`

**Generic Student Events**:
- `coppa.generic_student.created` - Generic Student account created
  - Properties: `student_id`, `username`, `created_by`, `organization_id`

### FERPA Events

**Educational Record Access**:
- `ferpa.record.accessed` - Educational record accessed
  - Properties: `student_id`, `accessed_by`, `user_role`, `record_type`, `access_method`, `timestamp`
- `ferpa.record.exported` - Educational records exported
  - Properties: `student_id`, `exported_by`, `export_format`, `record_count`, `timestamp`
- `ferpa.access.denied` - Access to educational record denied (no legitimate interest)
  - Properties: `student_id`, `denied_to`, `reason`, `timestamp`

**Record Amendment**:
- `ferpa.amendment.requested` - Amendment to record requested
  - Properties: `student_id`, `requested_by`, `record_type`, `record_id`, `reason`, `timestamp`
- `ferpa.amendment.approved` - Amendment request approved and applied
  - Properties: `amendment_request_id`, `approved_by`, `changes_made`, `timestamp`
- `ferpa.amendment.denied` - Amendment request denied
  - Properties: `amendment_request_id`, `denied_by`, `reason`, `timestamp`

**Disclosure Tracking**:
- `ferpa.disclosure.made` - Educational records disclosed to third party
  - Properties: `student_id`, `disclosed_to`, `disclosure_type`, `legal_basis`, `consent_obtained`, `timestamp`
- `ferpa.consent.obtained` - Parent consent obtained for disclosure
  - Properties: `student_id`, `consent_for`, `consent_method`, `timestamp`

**Data Retention and Destruction**:
- `ferpa.retention.deletion_scheduled` - Educational records scheduled for deletion
  - Properties: `student_id`, `organization_id`, `deletion_date`, `reason`
- `ferpa.retention.deleted` - Educational records permanently destroyed
  - Properties: `student_id`, `record_types_deleted`, `deletion_reason`, `timestamp`
- `ferpa.retention.hold_applied` - Legal hold prevents deletion
  - Properties: `student_id`, `hold_reason`, `applied_by`, `timestamp`

### Event Retention

**COPPA/FERPA Event Logs**:
- Consent events: Retained for life of account + 7 years (legal requirement)
- Access events: Retained for 3 years
- Disclosure events: Retained for 5 years (FERPA recommendation)
- Deletion events: Retained for 7 years
- Amendment events: Retained for 3 years

**Event Access Restrictions**:
- System Admin: Full access to all COPPA/FERPA events
- Subscriber-Admin: Access to events for their organization only
- Teachers: Cannot access COPPA/FERPA audit logs
- External Auditors: Read-only access with explicit authorization

## Permissions

COPPA and FERPA compliance permissions are defined by role. See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for complete specifications.

### Parental Consent Management

**View Consent Status**:
- Teachers: Can view consent status for students in their classes
- Subscriber-Admin/Teacher-Admin: Can view for all students in organization
- System Admin: Can view all consent statuses

**Certify Parental Consent**:
- Teachers: Can certify consent when creating student accounts
- Subscriber-Admin/Teacher-Admin: Can certify consent
- System Admin: Can manually update consent status

**Send Verification Emails**:
- System Admin only: Can trigger parental consent verification emails
- Automated: Triggered automatically if parent email provided during creation

**Verify/Update Consent**:
- Parents: Can verify consent via email link (no login required)
- System Admin: Can manually mark consent as verified after identity verification

### Educational Records Access

**Access Student Educational Records**:
- Students: Can access their own records (if eligible student, 18+)
- Teachers: Can access records for students they teach (legitimate educational interest)
- Subscriber-Admin: Can access records for students in their organization
- System Admin: Can access any educational records (with audit logging)
- Parents: Can access their child's records (K-12, under 18)

**Export Educational Records**:
- All roles with access rights can export records within their scope
- Exports logged for FERPA compliance

**Modify Educational Records**:
- Teachers: Can update scores, assignments, notes for their students
- Subscriber-Admin: Can modify student profiles and assignments
- System Admin: Can modify any records (with audit logging)
- Students/Parents: Cannot directly modify (must request amendment)

**Delete Educational Records**:
- Teachers: Cannot permanently delete (can remove assignments)
- Subscriber-Admin: Can delete students and related records in organization
- System Admin: Can delete any records
- Parents: Can request deletion (COPPA right)

### Amendment Requests

**Submit Amendment Request**:
- Parents: Can submit requests for their children
- Eligible Students (18+): Can submit requests for their own records

**Review Amendment Request**:
- Subscriber-Admin: Can review requests for their organization
- System Admin: Can review any requests
- Final decision rests with the school (Subscriber-Admin), not MLC

**Approve/Deny Amendment**:
- Subscriber-Admin: Can approve/deny for their organization
- System Admin: Can approve/deny with school authorization

### Compliance Reporting

**Generate COPPA Compliance Reports**:
- System Admin only: Can generate platform-wide reports
- Subscriber-Admin: Can generate reports for their organization

**Generate FERPA Disclosure Logs**:
- System Admin only: Platform-wide disclosure logs
- Subscriber-Admin: Disclosure logs for their organization

**Access Audit Logs**:
- System Admin: Full access to all audit logs
- Subscriber-Admin: Access to logs for their organization
- External Auditors: Read-only with explicit authorization

## Supporting Documents Referenced

This COPPA and FERPA compliance specification draws from the following source documents:

- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Educational mission and user types including student age ranges (ages 6-18), subscription levels, and data collection practices
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User hierarchy defining student account types, Generic Student specifications, and role-based data access rights
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription and organizational data management, seat allocation tied to individual student records

## Dependencies

This specification relies on and references the following systems and specifications:

### Legal and Compliance

**Primary References**:
- [Privacy Policy](./privacy-policy.md) - User-facing privacy policy with COPPA/FERPA sections
- [Privacy Policy Outline](./privacy-policy-outline.md) - Detailed privacy requirements and consent mechanisms
- [Terms and Conditions](./terms-conditions.md) - User agreement including age requirements and acceptable use
- [TOS Outline](./tos-outline.md) - Terms of Service specifications including educational use provisions
- [GDPR Compliance](./gdpr-compliance.md) - EU data protection (complementary to COPPA for EU children)
- [FERPA Compliance](./ferpa-compliance.md) - Full FERPA compliance framework (if separate document exists)

### Architecture and Security

**Technical Implementation**:
- [Security and Privacy](../18-architecture/security-privacy.md) - Data encryption, access controls, audit logging
- [Data Model ERD](../18-architecture/data-model-erd.md) - Database schema for students, consent tracking, educational records
- [API Design Principles](../18-architecture/api-design-principles.md) - API security and data access patterns
- [Background Jobs](../18-architecture/background-jobs.md) - Automated data retention and deletion jobs

### Roles and Permissions

**Access Control**:
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions and capabilities
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permission specifications
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Authentication and session security
- [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md) - System admin access to student data

### User Experience

**User-Facing Interfaces**:
- [Student Dashboard](../03-student-experience/student-dashboard.md) - Student data visibility and controls
- [Student Profile](../03-student-experience/student-profile.md) - Student profile data and privacy settings
- [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) - Teacher access to student educational records
- [Assignment Builder](../04-teacher-experience/assignment-builder.md) - Creating assignments and tracking student progress
- [Member Management](../05-admin-subscriber-experience/member-management.md) - Admin management of student accounts
- [Subscriber Dashboard](../05-admin-subscriber-experience/subscriber-dashboard.md) - Organization-level data access

### Analytics and Telemetry

**Event Tracking**:
- [Event Model](../15-analytics-and-reporting/event-model.md) - COPPA and FERPA event definitions
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Reporting with educational records
- [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md) - Organization-level educational data reporting

### Messaging and Notifications

**Communication**:
- [Email Templates](../13-messaging-and-notifications/email-templates.md) - Parental consent verification emails, FERPA notification templates
- [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md) - Consent status updates, parental request notifications
- [Communication Framework](../13-messaging-and-notifications/communication-framework.md) - Messaging policies including student-to-student restrictions

### Payments and Subscriptions

**Organizational Management**:
- [Seat Management](../14-payments-and-subscriptions/seat-management.md) - Seat allocation tied to individual student records
- [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) - Data Processing Agreement templates for vendors

### Data Management

**Import/Export**:
- [Export Specs](../20-imports-and-automation/export-specs.md) - Educational record export formats for FERPA/COPPA requests
- [CSV Specifications](../20-imports-and-automation/csv-specifications.md) - Student data import requirements
- [Data Migration](../20-imports-and-automation/data-migration.md) - Legacy data handling and retention

### Content and Copy

**UI Text**:
- [Copy Guidelines](../22-content-style-guides/copy-guidelines.md) - Plain language for privacy communications
- [Error Messages](../22-content-style-guides/error-messages.md) - COPPA/FERPA related error messages
- [Copy Guide - Teachers](../22-content-style-guides/copy-guide-teachers.md) - Teacher messaging about consent and privacy

### Quality and Operations

**Operational Procedures**:
- [Incident Response](../19-quality-and-operations/incident-response.md) - Data breach response for child data
- [Observability](../19-quality-and-operations/observability.md) - Monitoring for COPPA/FERPA compliance issues
- [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) - Audit trail for educational record access

### External Regulations

**Regulatory Authorities**:
- **Federal Trade Commission (FTC)**: COPPA enforcement
- **U.S. Department of Education**: FERPA guidance and enforcement
- **Student Privacy Pledge**: Voluntary industry commitment (consideration for future)
- **State Privacy Laws**: Various state student privacy laws (e.g., California SOPIPA, New York Education Law 2-d)

## Open Questions

Outstanding decisions to be made before build:

### COPPA Implementation

**Parental Consent Verification**:
- Should email verification be required for ALL students under 13, or only when email is collected?
- How frequently should parental consent be renewed (annually, biannually, never)?
- Should we implement a formal parental account system in MVP or Phase 2?

**Age Verification**:
- Is teacher certification sufficient for age determination, or should we collect date of birth?
- How do we handle students who turn 13 during their subscription (does consent requirement change)?
- Should we implement age gates for specific features beyond email collection?

**Generic Student Limitations**:
- Should Generic Students have any feature limitations beyond no PII collection?
- Should there be a maximum number of Generic Students per organization?
- How do we prevent misuse of Generic Students for avoiding COPPA compliance?

### FERPA Implementation

**Educational Records Definition**:
- Are teacher-created custom assignments considered "sole possession records" (exempt) or educational records?
- Should platform usage analytics (time spent, pages visited) be classified as educational records?
- How do we handle student-to-student messages (are they educational records)?

**Parental Access for Homeschool**:
- Do homeschool parents need separate parent accounts, or is their admin access sufficient?
- How do we verify homeschool parent identity (they are both school and parent)?

**Amendment Requests**:
- Who has final authority to approve/deny amendments - MLC or the school?
- What is the process if parent disagrees with denial and requests hearing?
- Should technical errors (wrong score recorded due to bug) be auto-corrected without formal amendment process?

### Operational Procedures

**Identity Verification**:
- What is the acceptable threshold for parent identity verification?
- Should we require government ID for high-risk requests (deletion), or is email sufficient?
- How do we handle requests from divorced/separated parents with custody disputes?

**Data Retention**:
- Is 13 months the optimal retention period for cancelled subscriptions?
- Should schools have option to request immediate deletion upon subscription cancellation?
- How do we handle partial retention (keep aggregated data, delete PII)?

**Compliance Reporting**:
- What metrics should be tracked in monthly COPPA/FERPA compliance reports?
- Who should receive compliance reports automatically?
- Should schools have access to compliance dashboard for their organization?

### Third-Party Integrations

**Vendor Compliance**:
- What COPPA/FERPA requirements must ALL vendors meet?
- Should we maintain a list of pre-approved COPPA/FERPA compliant vendors?
- What audit rights should schools have over our vendors?

**LTI/LMS Integrations** (Future):
- How do we handle FERPA when student data flows between MLC and external LMS?
- Are LMS integrations considered "school official" access or third-party disclosure?
- What consent is needed for LMS data syncing?

### Legal and Regulatory

**Multi-State Compliance**:
- Beyond federal COPPA/FERPA, what state laws apply (California SOPIPA, New York Ed Law 2-d)?
- How do we handle conflicts between state and federal requirements?
- Should we implement strictest requirement across all states, or geo-specific controls?

**International Users**:
- How do COPPA/FERPA interact with GDPR for EU students attending US schools?
- Do COPPA protections apply to children outside the US using our platform?
- Should we implement different consent flows for international users?

**Student Privacy Pledge**:
- Should MLC sign the Student Privacy Pledge (voluntary industry commitment)?
- What additional requirements does the Pledge impose beyond COPPA/FERPA?

---

**Document Version**: 1.0  
**Last Updated**: [Current Date]  
**Next Review Date**: [Current Date + 6 months]  
**Owner**: Legal & Compliance Team, Product Team
