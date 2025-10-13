# FERPA Compliance Framework

## Overview
Family Educational Rights and Privacy Act (FERPA) compliance specifications for educational institutions using the MLC-LMS platform.

## Description
Detailed FERPA compliance requirements for protecting student educational records, including data access controls, disclosure limitations, and institutional responsibilities for educational data privacy.

---

## Purpose

This document defines the regulatory, legal, and operational requirements for FERPA (Family Educational Rights and Privacy Act) compliance when processing educational records through the MLC-LMS platform. It provides clear guidance for educational institutions, outlines MLC's role and responsibilities, and documents how the platform supports schools in meeting their FERPA obligations.

**This specification enables:**

- **Institutional Compliance**: Clear framework for schools to meet FERPA requirements
- **Educational Records Protection**: Proper classification and safeguarding of student data
- **Rights Implementation**: Procedures for parents and eligible students to exercise FERPA rights
- **School Official Framework**: Documentation of MLC's role as a school official with legitimate educational interest
- **Data Processing Agreements**: Requirements for contracts between schools and MLC
- **Disclosure Management**: Lawful processes for sharing educational records
- **Audit Readiness**: Documentation and procedures for FERPA compliance audits
- **Role Clarity**: Distinction between school responsibilities and MLC platform capabilities

---

## Scope

### Included

This specification covers:

- **FERPA Regulatory Framework**:
  - When FERPA applies to educational institutions
  - Definition and scope of "educational records"
  - Distinction between educational records and non-records
  - Applicability to K-12 vs. higher education vs. homeschool

- **Educational Records Classification**:
  - What data constitutes an educational record in MLC-LMS
  - Student performance and progress data
  - Teacher assessments and notes
  - Records exempt from FERPA (sole possession records, directory information)

- **Parent and Student Rights**:
  - Right to inspect and review educational records
  - Right to request amendment of inaccurate records
  - Right to control disclosure of educational records
  - Distinction between parental rights (K-12) and eligible student rights (18+)

- **School Official Exception**:
  - MLC's designation as school official with legitimate educational interest
  - Requirements for school official status
  - Data Processing Agreement specifications
  - Limitations on re-disclosure

- **Consent and Disclosure Requirements**:
  - When written consent is required for disclosures
  - FERPA exceptions to consent requirement
  - Directory information designations
  - Disclosure logging and record-keeping

- **Annual Notification Requirements**:
  - School obligations to notify parents/students of FERPA rights
  - Template notification language
  - MLC's role in supporting annual notifications

- **Data Retention and Destruction**:
  - FERPA-compliant retention periods
  - Destruction procedures when no longer needed
  - School-requested data return or destruction

- **Institutional Responsibilities**:
  - School's role as data controller
  - Subscription administrator's FERPA obligations
  - Teacher responsibilities for educational records
  - Documentation and audit requirements

### Excluded

This specification does not cover:

- **Technical Implementation Details**: Covered in [COPPA/FERPA Notes](./coppa-ferpa-notes.md)
- **COPPA Compliance**: Children under 13 covered in [COPPA/FERPA Notes](./coppa-ferpa-notes.md)
- **GDPR Requirements**: EU data protection covered in [GDPR Compliance](./gdpr-compliance.md)
- **General Privacy Policy**: User-facing policy in [Privacy Policy](./privacy-policy.md)
- **Security Architecture**: Technical controls in [Security and Privacy](../18-architecture/security-privacy.md)
- **Payment and Billing**: Subscription management in [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md)
- **Access Control Implementation**: Technical permissions in [Permissions Table](../02-roles-and-permissions/permissions-table.md)

---

## FERPA Regulatory Overview

### What is FERPA?

The **Family Educational Rights and Privacy Act (FERPA)** is a federal law that protects the privacy of student education records. Enacted in 1974 and codified at 20 U.S.C. § 1232g, FERPA applies to all schools and educational agencies that receive federal funding from the U.S. Department of Education.

**Key FERPA Provisions:**
- Grants parents/eligible students the right to inspect and review educational records
- Requires schools to obtain written consent before disclosing educational records (with specific exceptions)
- Allows parents/eligible students to request amendment of inaccurate or misleading records
- Requires schools to provide annual notification of FERPA rights

**Enforcement:**
- U.S. Department of Education, Family Policy Compliance Office (FPCO)
- Can withhold federal funding for non-compliance
- Does not provide private right of action (no individual lawsuits)

### When FERPA Applies

**FERPA Applies When:**
1. Institution receives funds under any program administered by the U.S. Department of Education, **AND**
2. Institution maintains education records on students

**Institutions Subject to FERPA:**
- Public elementary and secondary schools
- Private schools receiving federal education funding
- Public and private colleges and universities
- School districts and local educational agencies (LEAs)
- State educational agencies (SEAs)

**Institutions NOT Subject to FERPA:**
- Purely private schools receiving no federal education funds
- Homeschools in states where they're not considered schools
- Childcare centers not part of elementary school
- Religious institutions receiving no federal education funds

### MLC-LMS and FERPA Applicability

**MLC's Status:**
- MLC is a **service provider** to educational institutions
- MLC is **not directly subject to FERPA** (we do not receive federal education funding)
- However, when processing educational records on behalf of schools subject to FERPA, MLC must:
  - Act as a "school official" with legitimate educational interest
  - Comply with FERPA requirements as agent of the school
  - Maintain confidentiality and security of educational records
  - Not re-disclose records except as directed by the school
  - Return or destroy data when no longer needed for authorized purpose

**MLC's Commitment:**
- Even when serving non-FERPA institutions (private studios, international schools), we apply FERPA-level protections as best practice
- Our platform is designed to support FERPA compliance for all institutional subscribers

### FERPA vs. Other Privacy Laws

**FERPA Interaction with COPPA:**
- FERPA governs educational records at schools
- COPPA governs collection of personal information from children under 13
- Both can apply simultaneously (school with students under 13)
- See [COPPA/FERPA Notes](./coppa-ferpa-notes.md) for detailed interaction

**FERPA Interaction with GDPR:**
- FERPA applies to US schools with students' educational records
- GDPR applies to EU residents' personal data
- EU students at US schools: both FERPA and GDPR may apply
- See [GDPR Compliance](./gdpr-compliance.md) for EU-specific requirements

**FERPA Interaction with State Privacy Laws:**
- State laws may provide additional protections (California SOPIPA, New York Education Law 2-d)
- Strictest requirement applies when laws overlap
- MLC platform designed to support most stringent requirements

---

## Educational Records Under FERPA

### Definition of Educational Records

**FERPA Definition (20 U.S.C. § 1232g(a)(4)(A)):**

"Educational records" means records that are:
1. Directly related to a student, **AND**
2. Maintained by an educational agency or institution or by a party acting for the agency or institution

**Key Characteristics:**
- **Student-specific**: Must relate to an identified or identifiable student
- **Maintained by school**: School or its agent (like MLC) maintains the record
- **Educational purpose**: Related to education, not just incidental presence at school

### Educational Records in MLC-LMS

The following data in MLC-LMS constitutes educational records under FERPA:

**Student Profile Information:**
- Student name (first and last)
- Username
- Email address (if provided)
- Date of birth (if collected)
- Class and group assignments
- Teacher assignments
- School/organization affiliation
- Account creation and modification dates

**Learning Performance Data:**
- Game scores and completion results
- Assignment completion status and grades
- Mastery level tracking
- Progress through curriculum sequences
- Time spent on educational activities
- Number of attempts and practice sessions
- Retention review performance
- Badge and achievement data

**Teacher Assessments:**
- Teacher-created custom assignments
- Teacher notes about student performance (if feature implemented)
- Teacher-assigned difficulty levels or placement
- Feedback provided to students
- Progress reports generated by teachers

**Communications:**
- Messages between teacher and student regarding educational matters
- Assignment instructions and clarifications
- Progress updates and notifications
- Parent communications about student performance

**Aggregate Data Tied to Individual Students:**
- Learning analytics showing individual student patterns
- Reports comparing individual student to class or cohort
- Predictive data about student performance or needs

### What is NOT an Educational Record

FERPA excludes certain types of records from the definition of "educational records":

**Sole Possession Records:**
- Personal notes made by teacher for their own use
- Not shared with others (except temporary substitute)
- Kept in sole possession of maker
- **MLC Consideration**: Teacher notes feature (if implemented) would NOT be sole possession if accessible to admins or stored centrally

**Law Enforcement Records:**
- Records created and maintained by school law enforcement unit
- Created for law enforcement purpose
- **MLC Status**: Not applicable (we don't process law enforcement records)

**Employment Records:**
- Records related to individual employed by school in capacity other than student
- **MLC Status**: Not applicable (no employment records)

**Medical and Treatment Records:**
- Records on students 18+ made by physician, psychiatrist, psychologist, or other professional
- Used only for treatment and not disclosed to others
- **MLC Status**: Not applicable (we don't process medical records)

**Alumni Records:**
- Records created after student no longer in attendance
- Do not relate to person as a student
- **MLC Consideration**: Data from active subscription retained 13 months after cancellation is still educational record during that period

**Directory Information (If Designated):**
- School may designate certain information as "directory information"
- Can include: name, address, phone, email, dates of attendance, awards
- Can be disclosed without consent IF parents given notice and opt-out opportunity
- **MLC Recommendation**: Do NOT treat student data as directory information

**Generic Student Accounts:**
- Shared accounts with no individual student identification
- No PII collected, aggregate scores only
- Not tied to individual students, therefore not "educational records"
- **MLC Implementation**: See [COPPA/FERPA Notes](./coppa-ferpa-notes.md) for Generic Student specifications

---

## Parent and Eligible Student Rights

FERPA grants specific rights to parents (for K-12 students under 18) and eligible students (18+ or attending postsecondary institution).

### Right to Inspect and Review Educational Records

#### Who Has This Right

**Parents (K-12, Under 18):**
- Parents have right to inspect and review child's educational records
- Applies to custodial and non-custodial parents unless court order restricts access
- Rights transfer to student at age 18 or when attending postsecondary institution

**Eligible Students:**
- Students 18 years or older
- Students attending postsecondary institution (regardless of age)
- Emancipated minors (in some states)

**Homeschool Context:**
- Parent is both "school" and "parent"
- FERPA rights still apply if homeschool receives federal funding
- MLC treats homeschool subscribers same as other institutions for privacy

#### School Obligations

**Timeline:**
- School must provide access within **45 days** of request
- Most schools provide access much sooner (within 5-10 business days)

**What Must Be Provided:**
- Right to inspect and review all educational records maintained on the student
- Explanation of any codes, abbreviations, or terms used in records
- Opportunity to discuss records with school official
- Copies of records if failure to provide copies would prevent access (distance, disability)

**Cost:**
- School may charge reasonable fee for copies (cannot charge for search/retrieval)
- Cannot charge fee that prevents low-income parents from exercising rights

#### MLC-LMS Implementation

**Self-Service Access (Recommended):**
- Teachers can provide parent/student view-only access to dashboard
- Parents log in with special access code (no separate account needed)
- View all scores, assignments, progress, achievements
- Export data as PDF or CSV
- No admin intervention required for routine access

**Technical Implementation:**
See [COPPA/FERPA Notes](./coppa-ferpa-notes.md#right-to-inspect-and-review) for detailed self-service and admin-facilitated access procedures.

**School Responsibilities:**
1. Respond to parent/eligible student requests within 45 days
2. Verify identity of requester before granting access
3. Provide either direct platform access or data export
4. Explain any unclear data or terminology
5. Document access provided (maintain log)

**MLC Platform Support:**
- Dashboard views for parents (view-only mode)
- Data export functionality (PDF, CSV formats)
- Admin tools to generate complete student data package
- Audit logging of data access requests
- See [Export Specs](../20-imports-and-automation/export-specs.md) for export formats

### Right to Request Amendment of Records

#### When Amendment Can Be Requested

Parents or eligible students can request amendment when they believe a record is:
- **Inaccurate**: Factually incorrect (wrong score, wrong name spelling)
- **Misleading**: Misrepresents the student or their performance
- **Violates privacy**: Contains information that should not be in educational record

**Cannot Challenge:**
- Grades or academic evaluations (FERPA does not govern academic judgments)
- Disciplinary decisions
- Subjective teacher opinions or assessments

#### Amendment Process

**1. Parent/Student Submits Written Request:**
- Identifies specific record to be amended
- Explains why it's inaccurate, misleading, or privacy-violating
- Proposes correction (optional but helpful)

**2. School Reviews Request (30 Days Reasonable):**
- Investigates claim
- Determines if amendment is warranted
- Decides to grant or deny

**3. If Amendment Granted:**
- Record is corrected
- Parent/eligible student notified of correction
- If record was disclosed to others, notify those parties of correction

**4. If Amendment Denied:**
- School notifies parent/eligible student with explanation
- Parent/eligible student has right to formal hearing
- Can place statement of disagreement in record if hearing denies amendment

#### MLC-LMS Implementation

**Common Amendment Scenarios:**
- **Data entry errors**: Wrong student name, typo in username
- **Technical glitches**: Score recorded incorrectly due to bug
- **Misattributed work**: Student completed work under wrong account
- **Outdated information**: Email address or contact info needs update

**Amendment Request Tracking:**
Technical implementation in [COPPA/FERPA Notes](./coppa-ferpa-notes.md#right-to-request-amendment) includes:
- Amendment request database table
- Status workflow (pending, approved, denied, hearing requested)
- Audit trail of changes made
- Notification system for all parties

**School Responsibilities:**
1. Receive and review amendment requests
2. Investigate validity of claim
3. Make determination within reasonable time (30 days recommended)
4. Document decision and rationale
5. Notify parent/eligible student of outcome
6. Provide hearing if request denied and parent/student requests one

**MLC Platform Support:**
- Amendment request submission interface
- Admin dashboard for reviewing requests
- Approval/denial workflow with reason codes
- Audit logging of all amendments
- Ability to append explanatory notes to records

**Who Decides:**
- **School decides**: Final authority to approve or deny amendments
- **MLC facilitates**: Platform provides tools but school makes determination
- **Exception**: Clear technical errors (wrong score due to bug) can be corrected by MLC with school notification

### Right to Consent to Disclosures

#### General Rule

Schools must obtain **written consent** from parent or eligible student before disclosing personally identifiable information (PII) from educational records to third parties.

**Written Consent Must Specify:**
- Records to be disclosed
- Purpose of disclosure
- Party to whom disclosure will be made
- Parent/eligible student signature and date

#### FERPA Exceptions to Consent Requirement

Schools may disclose educational records **without consent** in specific circumstances:

**1. School Officials with Legitimate Educational Interest:**
- Teachers, administrators, staff with educational need to know
- Third-party contractors acting as school officials (like MLC)
- Must have legitimate reason related to student's education
- See [School Official Exception](#school-official-exception) section below

**2. Officials of Other Schools:**
- School to which student is transferring or intends to enroll
- Reasonable attempt to notify parent/student (unless annual notice covers transfers)

**3. Authorized Representatives for Audit or Evaluation:**
- US Department of Education
- State educational authorities
- Auditors conducting program evaluations
- Must destroy records when no longer needed

**4. Financial Aid Determinations:**
- Disclosure to financial aid officers for aid determinations

**5. Organizations Conducting Studies:**
- Studies for or on behalf of school for developing, validating tests, administering aid, or improving instruction
- Must have agreement protecting confidentiality
- Must destroy data when study complete

**6. Accrediting Organizations:**
- Disclosure to accrediting bodies conducting accreditation activities

**7. Compliance with Judicial Order or Subpoena:**
- Court orders and lawfully issued subpoenas
- School must make reasonable effort to notify parent/student before complying (unless order prohibits notification)

**8. Health and Safety Emergencies:**
- Disclosure to appropriate parties if knowledge of information is necessary to protect health or safety
- Must be timely and limited to emergency period

**9. Directory Information:**
- If school has designated directory information and provided notice/opt-out opportunity
- **MLC Recommendation**: Do not treat student data as directory information

**10. Parents of Dependent Student (Higher Ed):**
- Higher ed institution may disclose to parents if student is dependent for tax purposes

**11. Victims of Violent Crimes:**
- Disciplinary proceedings results to alleged victim (higher ed)

#### MLC-LMS Disclosure Considerations

**Routine Disclosures Under School Official Exception:**
- Teachers viewing student data in their classes
- Subscriber-admin viewing student data in their organization
- MLC system admin accessing data for support/compliance (with logging)
- These do NOT require individual parental consent if covered in annual notice

**Third-Party Disclosures Requiring Consent:**
- Sharing student data with external tutoring service (unless acting as school official)
- Providing student data to research organization for non-school study
- Disclosing to vendors not acting as school officials
- Marketing or non-educational purposes (never permitted)

**Disclosure Logging:**
Schools must maintain record of disclosures, including:
- Who received educational records
- Legitimate interest for disclosure
- Date of disclosure
- What records were disclosed

**Technical Implementation:**
See [COPPA/FERPA Notes](./coppa-ferpa-notes.md#consent-for-disclosure) for disclosure tracking database schema and implementation.

**School Responsibilities:**
1. Determine if disclosure requires consent or falls under exception
2. Obtain written consent when required
3. Maintain disclosure logs
4. Provide disclosure logs to parent/eligible student upon request
5. Inform recipients that they cannot re-disclose without consent

---

## School Official Exception

The "school official exception" is the primary legal basis for MLC to process educational records without individual parental consent. This exception allows schools to share educational records with third-party contractors who perform institutional services or functions.

### Requirements for School Official Status

Under FERPA regulations (34 CFR § 99.31(a)(1)), a school official with legitimate educational interest includes:

**Criteria MLC Must Meet:**
1. **Performs Institutional Service**: MLC provides educational technology services on behalf of the school
2. **Replaces School Employee**: MLC performs function the school would otherwise do itself (instruction, progress tracking)
3. **Under Direct Control**: School exercises control over use and maintenance of records
4. **Legitimate Educational Interest**: MLC needs access to records to fulfill educational purpose
5. **Subject to FERPA Requirements**: MLC bound by same FERPA protections as school employees
6. **Written Agreement**: Data Processing Agreement between school and MLC specifies terms

### MLC as School Official

**MLC's Role:**
- Technology service provider delivering music education platform
- Processes student educational records on behalf of subscribing schools
- Uses data solely for purposes authorized by school
- Does not use or disclose records for any purpose other than those specified by school
- Subject to FERPA prohibitions on re-disclosure

**Legitimate Educational Interest:**
- Delivering educational games and learning content to students
- Tracking student progress and mastery
- Generating reports for teachers and administrators
- Supporting differentiated instruction
- Technical platform maintenance and support
- Compliance and security monitoring

### Data Processing Agreement (DPA) Requirements

Every institutional subscription must include a Data Processing Agreement that specifies:

**Required DPA Terms:**
1. **Purpose of Processing**:
   - MLC processes educational records to provide music learning platform services
   - Specific services: game delivery, progress tracking, reporting, assignment management

2. **Types of Educational Records**:
   - Student names, usernames, email addresses
   - Game scores and performance data
   - Assignment completion and progress
   - Teacher-created assignments and assessments
   - Communications between students and teachers

3. **Use Restrictions**:
   - MLC uses data ONLY for purposes specified by school
   - No use for MLC's own purposes (marketing, product development with identified student data)
   - No sale or marketing of student data to third parties
   - Anonymized, aggregate data may be used for platform improvement (with school consent)

4. **Security Measures**:
   - Encryption at rest and in transit
   - Role-based access controls
   - Security monitoring and logging
   - Regular security audits
   - See [Security and Privacy](../18-architecture/security-privacy.md) for details

5. **Sub-Processors**:
   - List of sub-processors (Vercel, Neon, Cloudflare, Braintree)
   - Each sub-processor has signed Data Processing Agreement with MLC
   - School notified of changes to sub-processor list
   - See [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) for vendor agreements

6. **Prohibition on Re-Disclosure**:
   - MLC will not re-disclose educational records to other parties
   - Exception: MLC's sub-processors acting under equivalent restrictions
   - Exception: As required by law (court order, safety emergency)

7. **Assistance with Data Subject Rights**:
   - MLC will assist school in responding to parent/eligible student requests
   - Access requests: MLC provides data export tools
   - Amendment requests: MLC provides interface for corrections
   - Deletion requests: MLC deletes data upon school's instruction

8. **Data Retention and Destruction**:
   - Active subscriptions: Data retained while subscription active
   - Cancelled subscriptions: 13-month retention period
   - School can request earlier return or destruction
   - Upon termination, MLC will delete or return all educational records as directed

9. **Audit Rights**:
   - School may audit MLC's compliance with DPA
   - MLC will provide compliance reports (SOC 2, security audits)
   - School may request specific compliance attestations

10. **Breach Notification**:
    - MLC will notify school of any security breach involving educational records
    - Notification within 72 hours of discovery
    - School determines whether parent/student notification is required
    - See [Incident Response](../19-quality-and-operations/incident-response.md)

**DPA Template:**
MLC provides a standard Data Processing Agreement template that incorporates all required FERPA terms. Schools may request modifications consistent with their specific requirements.

### School's Obligations as Data Controller

Even with MLC as school official, the school retains primary responsibility:

**School Must:**
1. Include MLC in annual FERPA notice as school official
2. Ensure MLC has legitimate educational interest for access
3. Monitor MLC's compliance with FERPA and DPA
4. Respond to parent/eligible student requests for access, amendment, disclosure logs
5. Determine whether consent is required for any specific use or disclosure
6. Instruct MLC on data retention, return, or destruction
7. Notify parents if MLC is included in directory information disclosures (not recommended)

**School Cannot:**
- Transfer FERPA compliance responsibility entirely to MLC
- Delegate decision-making about consent requirements
- Avoid responding to parent/student requests by directing them to MLC
- Blame MLC for school's own FERPA violations (though MLC may share liability)

---

## Annual Notification of FERPA Rights

### School's Obligation

FERPA requires schools to notify parents and eligible students annually of their rights under FERPA:

**Required Content of Annual Notice:**
1. Right to inspect and review educational records
2. Right to seek amendment of records believed to be inaccurate or misleading
3. Right to consent to disclosures of PII (with exceptions)
4. Right to file complaint with U.S. Department of Education

**Additional Recommended Content:**
- How to request access to records
- Types of records maintained by school
- School officials with access and their legitimate interests
- Criteria for determining legitimate interest
- Where to file FERPA complaint (US Dept of Education, Family Policy Compliance Office)

### MLC's Role in Annual Notice

**MLC Must Be Included:**
Schools should include MLC-LMS in their annual FERPA notice as a "school official" with access to educational records.

**Recommended Language for Schools:**
```
Third-Party Service Providers as School Officials:

[School Name] uses Music Learning Community (MLC-LMS), an online music education platform, 
to deliver instruction and track student progress. MLC-LMS is designated as a school official 
with legitimate educational interest in student educational records. MLC-LMS has access to 
student names, usernames, game scores, assignment completion data, and related educational 
information necessary to provide the educational service. MLC-LMS is subject to the same 
FERPA requirements as school employees and may not re-disclose educational records or use 
them for any purpose other than providing the educational service.
```

**MLC Provides:**
- Template language schools can incorporate into their annual notice
- Sample annual notice covering FERPA rights related to MLC-LMS use
- Technical support for schools drafting notices
- Clarification of what data MLC accesses and for what purposes

**MLC Does Not:**
- Send annual notices directly to parents (school's responsibility)
- Determine content of school's annual notice
- Replace school's FERPA compliance officer or legal counsel

### Notification Methods

Schools must provide annual notification through one or more of the following:

**Acceptable Methods:**
- Student or parent handbook (distribute annually)
- School newsletter or website
- Email to all parents and eligible students
- School registration materials
- Student ID card or agenda book

**Timing:**
- At beginning of each school year
- When student enrolls mid-year (provide notice at enrollment)
- Changes to FERPA notice require additional notification

---

## Data Retention and Destruction

### FERPA Requirements

FERPA does not specify retention periods but requires schools to:
- Retain records only as long as necessary for educational purposes
- Not retain records indefinitely without justification
- Destroy records when no longer needed (unless exception applies)

### MLC-LMS Retention Policy

**Active Subscriptions:**
- All educational records retained while subscription is active
- Full access for authorized school officials and students
- Daily encrypted backups maintained

**Cancelled Subscriptions:**
- **13-month retention period** after subscription cancellation
- Allows schools to reactivate without data loss
- After 13 months: automatic permanent deletion of all educational records
- Exception: Financial records retained 7 years (legal requirement for tax/audit)

**School-Requested Early Destruction:**
- Schools can request immediate deletion upon cancellation
- MLC will delete within 30 days of verified request
- Optional: Data export provided before deletion
- Deletion logged and confirmed to school

**Legal Hold Exception:**
- Records preserved if subject to legal hold (litigation, investigation)
- Destruction paused until legal hold lifted
- School notified of legal hold status

### Data Destruction Process

**Technical Implementation:**
See [COPPA/FERPA Notes](./coppa-ferpa-notes.md#data-retention-and-destruction) for automated deletion job specifications.

**What Gets Deleted:**
- Student accounts and profile information
- All game scores and performance data
- Assignment completions and progress tracking
- Teacher notes and assessments (if feature exists)
- Messages and communications
- Session logs and access records (after 1 year)

**What May Be Retained:**
- Financial transaction records (7 years - IRS requirement)
- Anonymized, aggregated research data (no individual student identification)
- Security logs (1 year minimum, anonymized after retention period)
- Audit trail of deletion itself (proof of compliance)

**Permanent Deletion Guarantees:**
- Data deleted from production database
- Data deleted from encrypted backups (within next backup cycle)
- Data deleted from all caches and replicas
- No recovery possible after permanent deletion
- Deletion logged in audit trail

### School's Retention Authority

**School Determines Retention:**
- Schools may request earlier deletion (overrides MLC's 13-month policy)
- Schools may request data export for their own retention
- Schools responsible for complying with their own state/local retention requirements
- MLC supports school's retention decisions within legal requirements

**State Retention Requirements:**
- Some states mandate minimum retention periods for educational records (e.g., 5 years after graduation)
- Schools must export data from MLC if state law requires longer retention than MLC's policy
- MLC will provide data export in standard formats upon request
- See [Export Specs](../20-imports-and-automation/export-specs.md)

---

## Disclosure Logging and Records

### FERPA Requirement

Schools must maintain a record of each disclosure of PII from educational records (with limited exceptions).

**Required Information in Disclosure Log:**
- Name of party to whom disclosure was made
- Legitimate interest party had in the information
- Date of disclosure

**Exceptions (No Logging Required):**
- Disclosures to school officials with legitimate educational interest
- Disclosures pursuant to parent/eligible student consent (if consent specifies record will not be maintained)
- Directory information disclosures (if designated and notice provided)
- Disclosures to parent or eligible student themselves

### MLC-LMS Disclosure Tracking

**Automatic Logging:**
MLC automatically logs certain access to educational records:

**Logged Events:**
- System admin access to student educational records (with justification)
- Data exports generated for parents or eligible students
- Data disclosed to third parties (if feature added)
- Court orders or legal requests for student data
- Emergency disclosures for health and safety

**Technical Implementation:**
See [COPPA/FERPA Notes](./coppa-ferpa-notes.md#disclosure-tracking) for ferpa_disclosures database table schema.

**Not Logged (School Official Exception):**
- Teacher viewing students in their own classes (legitimate educational interest)
- Subscriber-admin viewing students in their organization (legitimate educational interest)
- Student viewing their own data
- Routine platform operations by MLC staff

### Parent/Eligible Student Access to Disclosure Logs

**FERPA Right:**
Parents and eligible students have right to review disclosure log showing who has accessed their educational records.

**MLC Implementation:**
- Disclosure log accessible to school administrators
- School administrator can export disclosure log for specific student
- Self-service option (future): Parent/student dashboard showing access history
- Delivered within 45 days of request (same as inspection of records)

**School Responsibilities:**
1. Respond to parent/eligible student requests for disclosure log
2. Work with MLC to retrieve disclosure log from platform
3. Explain entries in disclosure log to parent/student
4. Maintain disclosure logs for as long as records are maintained

---

## Homeschool Considerations

### FERPA Status of Homeschools

**FERPA Application:**
- FERPA applies to "educational agencies or institutions" receiving federal education funding
- Most homeschools do NOT receive federal funding
- Therefore, most homeschools are NOT subject to FERPA

**Exceptions:**
- Homeschools that are chartered or part of public school district (may receive federal funds)
- Homeschools in states where they're legally considered private schools receiving federal aid
- Homeschool cooperatives that receive federal education funding

**MLC Approach:**
- Even if homeschool is not subject to FERPA, MLC applies FERPA-level protections as best practice
- Homeschool subscribers benefit from same privacy safeguards as traditional schools
- Data Processing Agreement available for homeschool subscribers upon request

### Parent as Both "School" and "Parent"

**Unique Homeschool Context:**
- Parent acts as both the educational institution AND the parent
- Parent has both school's authority and parental rights
- Simplified data access (parent has full control)

**MLC Implementation for Homeschools:**
- Parent (subscriber-admin) has full access to all student data
- No separate parental consent process needed (parent is school official)
- Parental rights and school official rights merge
- Parent determines all uses of student data

**Privacy Protection Still Important:**
- Other household members should not have access without authorization
- Siblings using shared devices should have separate logins
- Parent responsible for securing their own administrator credentials

---

## Institutional Responsibilities

### School/Institution as Data Controller

**Primary Responsibilities:**
1. **Determine Purposes**: School decides how student data will be used
2. **Authorize MLC**: School designates MLC as school official
3. **Annual Notice**: School notifies parents of FERPA rights and MLC's access
4. **Respond to Requests**: School responds to parent/student access, amendment, consent requests
5. **Disclosure Decisions**: School determines when consent is required for disclosures
6. **Data Governance**: School maintains policies for educational records
7. **Training**: School trains staff on FERPA requirements
8. **Monitoring**: School monitors MLC's compliance with DPA and FERPA

### Subscriber-Admin Responsibilities

The Subscriber-Admin (school administrator using MLC platform) has specific operational duties:

**Data Management:**
- Create and manage student accounts in compliance with FERPA and COPPA
- Ensure parental consent obtained where required (students under 13)
- Configure appropriate access permissions for teachers
- Review and approve teacher requests for student data access
- Manage data retention and deletion requests

**Rights Fulfillment:**
- Respond to parent/eligible student requests for data access
- Generate data exports for parents using MLC tools
- Process amendment requests for inaccurate records
- Maintain disclosure logs accessible to parents
- Coordinate with MLC support for complex requests

**Compliance:**
- Include MLC in school's annual FERPA notice
- Ensure Data Processing Agreement is executed with MLC
- Monitor teachers' use of student data for educational purposes
- Report any suspected misuse of student data
- Coordinate with school's FERPA compliance officer

**Training:**
- Ensure teachers understand FERPA obligations
- Provide guidance on appropriate use of student data
- Clarify boundaries between educational use and impermissible uses
- Document training provided to staff

### Teacher Responsibilities

Teachers using MLC-LMS have FERPA obligations as school officials:

**Appropriate Use:**
- Access only data for students in their assigned classes
- Use student data only for legitimate educational purposes
- Do not share student data with unauthorized persons
- Do not use student data for personal purposes
- Maintain confidentiality of student educational records

**Security:**
- Secure login credentials (do not share accounts)
- Log out when leaving device unattended
- Do not access student records from unsecured public networks
- Report any suspected unauthorized access immediately

**Parental Communication:**
- Respond to parent questions about student progress
- Refer formal FERPA requests to subscriber-admin
- Understand limitations on disclosure without consent
- Document any disclosures made (for disclosure log)

**Technical Implementation:**
See [Permissions Table](../02-roles-and-permissions/permissions-table.md) for technical enforcement of teacher access restrictions.

---

## Higher Education vs. K-12 Differences

### Rights Transfer at Age 18

**K-12 (Under Age 18):**
- Parents have FERPA rights
- School communicates with parents about educational records
- Parents consent to disclosures
- Parents request access and amendment

**Age 18 or Postsecondary:**
- Rights transfer to student (now "eligible student")
- School communicates with student, not parent (unless exception applies)
- Student consents to disclosures
- Student requests access and amendment

**MLC Implementation:**
- Platform can flag students as "eligible students" based on age or education level
- Subscriber-admin configures whether rights belong to parent or student
- Access controls adjusted based on who has FERPA rights
- Communications directed appropriately

### Parental Access in Higher Education

**General Rule:**
Once student turns 18 or attends postsecondary institution, parents do NOT have automatic access to educational records.

**Exceptions:**
1. **Dependent Student Exception**: Higher ed institution MAY (but not required to) disclose to parents if student is dependent for tax purposes
2. **Health and Safety Emergency**: May disclose to parents if necessary to protect student's health or safety
3. **Student Consent**: Eligible student can provide written consent for parents to access records
4. **Violation of Law**: May disclose to parents if student under 21 violates law regarding alcohol or controlled substances

**MLC Higher Ed Implementation:**
- Student controls their own account and data access
- Parent access requires student's explicit consent
- Platform supports student consent workflow for parental access
- Higher ed institutions configure parent access policies in platform

---

## Multi-Institutional Data Sharing

### Student Transfers

**FERPA Allows Disclosure:**
Schools may disclose educational records without consent to officials of another school in which student seeks or intends to enroll.

**Requirements:**
- Reasonable attempt to notify parent/eligible student of disclosure
- Provide copy of records if requested
- Opportunity for hearing to challenge content (if requested)
- Exception: If annual notice states school forwards records on request

**MLC Implementation:**
- School can export student data to provide to new school
- Receiving school can import student data (if they also use MLC)
- Transfer logged in disclosure log
- See [Data Migration](../20-imports-and-automation/data-migration.md) for transfer procedures

### Multi-District or Consortia

**Shared Services:**
- Multiple schools may share single MLC institutional subscription
- Each school is a separate organization within MLC platform
- Data segregation by organization enforced by platform

**FERPA Considerations:**
- Each school is separate legal entity under FERPA
- School cannot access another school's students without authorization
- Consortium agreement should specify data sharing arrangements
- MLC enforces data isolation through access controls

**Technical Implementation:**
See [School Hierarchy View](../05-admin-subscriber-experience/school-hierarchy-view.md) for multi-school organizational structures.

---

## Telemetry and Audit Logging

All FERPA-related events are tracked via the platform [Event Model](../15-analytics-and-reporting/event-model.md).

### FERPA Events

**Educational Record Access:**
- `ferpa.record.accessed` - Educational record viewed
  - Properties: `student_id`, `accessed_by`, `user_role`, `record_type`, `access_method`, `timestamp`
- `ferpa.record.exported` - Educational records exported
  - Properties: `student_id`, `exported_by`, `export_format`, `record_count`, `purpose`, `timestamp`
- `ferpa.access.denied` - Access to educational record denied
  - Properties: `student_id`, `denied_to`, `reason`, `timestamp`

**Record Amendment:**
- `ferpa.amendment.requested` - Amendment request submitted
  - Properties: `student_id`, `requested_by`, `record_type`, `reason`, `timestamp`
- `ferpa.amendment.approved` - Amendment request approved
  - Properties: `amendment_request_id`, `approved_by`, `changes_made`, `timestamp`
- `ferpa.amendment.denied` - Amendment request denied
  - Properties: `amendment_request_id`, `denied_by`, `reason`, `timestamp`

**Disclosure Tracking:**
- `ferpa.disclosure.made` - Educational records disclosed to third party
  - Properties: `student_id`, `disclosed_to`, `disclosure_type`, `legal_basis`, `consent_obtained`, `timestamp`
- `ferpa.consent.obtained` - Written consent obtained for disclosure
  - Properties: `student_id`, `consent_for`, `consent_method`, `timestamp`

**Data Retention:**
- `ferpa.retention.deletion_scheduled` - Educational records scheduled for deletion
  - Properties: `student_id`, `organization_id`, `deletion_date`, `reason`
- `ferpa.retention.deleted` - Educational records permanently destroyed
  - Properties: `student_id`, `record_types_deleted`, `deletion_reason`, `timestamp`
- `ferpa.retention.hold_applied` - Legal hold prevents deletion
  - Properties: `student_id`, `hold_reason`, `applied_by`, `timestamp`

### Event Retention

**FERPA Event Logs:**
- Access events: Retained for 3 years
- Amendment events: Retained for 3 years
- Disclosure events: Retained for 5 years (FERPA recommendation)
- Deletion events: Retained for 7 years
- Consent records: Retained for life of record + 3 years

**Event Access Restrictions:**
- System Admin: Full access to all FERPA events
- Subscriber-Admin: Access to events for their organization only
- Teachers: Cannot access FERPA audit logs (only their own teaching actions)
- External Auditors: Read-only access with explicit authorization

**Technical Implementation:**
See [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) and [Event Model](../15-analytics-and-reporting/event-model.md).

---

## Permissions

FERPA compliance permissions are defined by role. See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for complete specifications.

### Educational Records Access

**Access Student Educational Records:**
- **Students**: Can access their own records (if eligible student, 18+)
- **Parents**: Can access child's records (K-12, under 18) - requires separate parent access feature
- **Teachers**: Can access records for students they teach (legitimate educational interest)
- **Subscriber-Admin/Teacher-Admin**: Can access records for all students in organization
- **System Admin**: Can access any educational records with audit logging and justification

**Export Educational Records:**
- All roles with access rights can export records within their scope
- Exports logged for FERPA disclosure tracking
- Export formats: PDF, CSV, JSON
- See [Export Specs](../20-imports-and-automation/export-specs.md)

**Modify Educational Records:**
- **Teachers**: Can update scores, assignments, notes for their students
- **Subscriber-Admin**: Can modify student profiles, class assignments
- **System Admin**: Can modify any records (with audit logging)
- **Students/Parents**: Cannot directly modify (must request amendment)

**Delete Educational Records:**
- **Teachers**: Cannot permanently delete educational records
- **Subscriber-Admin**: Can delete students and related records in organization
- **System Admin**: Can delete any records (with justification and logging)
- **Parents**: Can request deletion for students under 13 (COPPA right)

### Amendment Request Management

**Submit Amendment Request:**
- **Parents**: Can submit requests for their children (K-12, under 18)
- **Eligible Students** (18+): Can submit requests for their own records
- **Teachers**: Can submit on behalf of parent/student with authorization

**Review Amendment Request:**
- **Subscriber-Admin**: Can review and approve/deny requests for their organization
- **System Admin**: Can review any requests and assist schools
- **Final Decision**: School (Subscriber-Admin) makes final determination, not MLC

**Approve/Deny Amendment:**
- **Subscriber-Admin**: Authority to approve/deny for their organization
- **System Admin**: Can implement approved changes with school authorization
- **Process**: Amendment decision documented with rationale

### Compliance Reporting

**Generate FERPA Disclosure Logs:**
- **System Admin**: Platform-wide disclosure logs
- **Subscriber-Admin**: Disclosure logs for their organization
- **Teachers**: Cannot access disclosure logs

**Generate FERPA Compliance Reports:**
- **System Admin**: Platform-wide compliance reporting
- **Subscriber-Admin**: Organization-level compliance reports (future feature)
- Reports include: access patterns, amendment requests, disclosure summary

**Access Audit Logs:**
- **System Admin**: Full access to all FERPA audit logs
- **Subscriber-Admin**: Access to logs for their organization
- **External Auditors**: Read-only with explicit authorization from school

---

## Supporting Documents Referenced

This FERPA compliance framework draws from the following source documents:

- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Educational mission statement, user types, and commitment to protecting student educational records in accordance with privacy laws
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role-based access controls defining which users can access educational records, teacher vs. admin permissions, and legitimate educational interest criteria
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Institutional subscription structures, organizational hierarchies, and data segregation between schools and districts

**Note**: No FERPA-specific source documents were found in the original context. This framework is built on standard FERPA requirements from U.S. Department of Education guidance, the existing FERPA implementation details in [COPPA/FERPA Notes](./coppa-ferpa-notes.md), and alignment with the platform's educational mission and privacy commitments.

---

## Dependencies

This specification relies on and references the following systems and specifications:

### Legal and Compliance

**Primary References:**
- [COPPA/FERPA Notes](./coppa-ferpa-notes.md) - Technical implementation of both COPPA and FERPA (lines 730-1476 for FERPA)
- [Privacy Policy](./privacy-policy.md) - User-facing privacy policy with FERPA section
- [Privacy Policy Outline](./privacy-policy-outline.md) - Detailed privacy requirements
- [GDPR Compliance](./gdpr-compliance.md) - EU data protection (complementary for EU students at US schools)
- [Terms and Conditions](./terms-conditions.md) - User agreement including educational use provisions
- [TOS Outline](./tos-outline.md) - Terms of Service specifications
- [Copyright and Licensing](./copyright-and-licensing.md) - Content ownership and educational use rights

### Architecture and Security

**Technical Implementation:**
- [Security and Privacy](../18-architecture/security-privacy.md) - Data encryption, access controls, audit logging
- [Data Model ERD](../18-architecture/data-model-erd.md) - Database schema for students and educational records
- [API Design Principles](../18-architecture/api-design-principles.md) - API security and data access patterns
- [Background Jobs](../18-architecture/background-jobs.md) - Automated data retention and deletion jobs
- [Webhooks](../18-architecture/webhooks.md) - Event-driven notifications for compliance events

### Roles and Permissions

**Access Control:**
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions and capabilities
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permission specifications
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Authentication and session security
- [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md) - System admin access procedures

### User Experience

**User-Facing Interfaces:**
- [Student Dashboard](../03-student-experience/student-dashboard.md) - Student access to their own records
- [Student Profile](../03-student-experience/student-profile.md) - Student profile data and settings
- [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) - Teacher access to student educational records
- [Teacher Reports](../04-teacher-experience/teacher-reports.md) - Educational record reporting to teachers
- [Assignment Builder](../04-teacher-experience/assignment-builder.md) - Creating and tracking assignments
- [Member Management](../05-admin-subscriber-experience/member-management.md) - Admin management of student accounts
- [Subscriber Dashboard](../05-admin-subscriber-experience/subscriber-dashboard.md) - Organization-level data access
- [School Hierarchy View](../05-admin-subscriber-experience/school-hierarchy-view.md) - Multi-school organizational structures
- [Admin Reports](../05-admin-subscriber-experience/admin-reports.md) - Organization-level reporting

### Content and Educational Records

**Learning Data:**
- [Assignment Model](../07-content-architecture/assignment-model.md) - Assignment data structure (educational record)
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game structure and scoring (educational record)
- [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) - Curriculum sequences
- [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md) - Progress data (educational record)

### Analytics and Telemetry

**Event Tracking:**
- [Event Model](../15-analytics-and-reporting/event-model.md) - FERPA event definitions and tracking
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Reporting with educational records
- [Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md) - Organization-level educational data reporting
- [Sysadmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md) - Compliance monitoring dashboards

### System Administration

**Admin Tools:**
- [Sysadmin User Index](../06-system-admin/sysadmin-user-index.md) - User management interface
- [Sysadmin Member Profile](../06-system-admin/sysadmin-member-profile.md) - Individual user data access
- [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) - Compliance audit trail
- [User Permissions](../06-system-admin/user-permissions.md) - Permission management

### Messaging and Notifications

**Communication:**
- [Email Templates](../13-messaging-and-notifications/email-templates.md) - FERPA notification email templates
- [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md) - Access request notifications
- [Communication Framework](../13-messaging-and-notifications/communication-framework.md) - Messaging policies
- [System Announcements](../13-messaging-and-notifications/system-announcements.md) - Annual FERPA notice support

### Payments and Subscriptions

**Organizational Management:**
- [Seat Management](../14-payments-and-subscriptions/seat-management.md) - Student seat allocation
- [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) - Data Processing Agreement templates
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) - Institutional subscription models

### Data Management

**Import/Export:**
- [Export Specs](../20-imports-and-automation/export-specs.md) - Educational record export formats for parent/student requests
- [CSV Specifications](../20-imports-and-automation/csv-specifications.md) - Student data import/export
- [Data Migration](../20-imports-and-automation/data-migration.md) - School transfer and data portability
- [Bulk Ops Jobs](../20-imports-and-automation/bulk-ops-jobs.md) - Automated deletion and retention

### Quality and Operations

**Operational Procedures:**
- [Incident Response](../19-quality-and-operations/incident-response.md) - Data breach response for educational records
- [Observability](../19-quality-and-operations/observability.md) - Monitoring for FERPA compliance issues
- [Release Management](../19-quality-and-operations/release-management.md) - Change management affecting educational records

### Content Style

**UI Text:**
- [Copy Guidelines](../22-content-style-guides/copy-guidelines.md) - Plain language for rights notices
- [Error Messages](../22-content-style-guides/error-messages.md) - FERPA-related error messages
- [Copy Guide - Teachers](../22-content-style-guides/copy-guide-teachers.md) - Teacher messaging about student privacy

### External Regulations

**Regulatory Authorities:**
- **U.S. Department of Education, Family Policy Compliance Office (FPCO)**: FERPA enforcement and guidance
- **State Educational Agencies**: State-specific student privacy requirements
- **FERPA Resources**: [https://studentprivacy.ed.gov/](https://studentprivacy.ed.gov/)
- **FERPA Regulations**: 34 CFR Part 99

---

## Open Questions

Outstanding decisions to be made before build:

### Educational Records Classification

**Scope Questions:**
- Are teacher-created custom assignments "sole possession records" (exempt) or educational records? **Decision needed**: Recommend treating as educational records if stored centrally
- Should platform usage analytics (time spent, pages visited) be classified as educational records? **Decision needed**: Recommend classifying as educational records when tied to individual students
- How do we handle student-to-student messages - are they educational records? **Decision needed**: Messages about educational content are records; social messages may not be

### Parental Access Implementation

**K-12 Parental Access:**
- Should parent accounts be separate feature or view-only access via teacher-provided codes? **Decision needed**: MVP could be view-only codes, Phase 2 adds parent accounts
- How do we verify parent identity before granting access? **Decision needed**: Rely on school's existing verification (parent contacts teacher/admin)
- What level of access do parents get - full dashboard or simplified view? **Decision needed**: Simplified view in MVP, full dashboard option in Phase 2

**Homeschool Context:**
- Do homeschool parents need separate parent accounts or is subscriber-admin access sufficient? **Decision needed**: Subscriber-admin access sufficient (parent IS the school)
- How do we verify homeschool parent identity (they are both school and parent)? **Decision needed**: Account holder verification sufficient

### Higher Education Implementation

**Rights Transfer:**
- How do we handle transition when student turns 18 mid-year? **Decision needed**: Manual flag toggle by subscriber-admin, automated birthday transition in Phase 2
- Should platform automatically transfer rights at age 18 or require manual configuration? **Decision needed**: Manual configuration in MVP (school determines policy)
- How do higher ed institutions configure parent access policies? **Decision needed**: Organization-level settings in subscriber-admin dashboard

**Dependent Student Exception:**
- Should platform support higher ed parent access under dependent student exception? **Decision needed**: Phase 2 feature, requires student consent workflow
- What consent mechanism is needed for eligible student to grant parents access? **Decision needed**: Consent form signed by student, logged in platform

### Amendment Requests

**Process Questions:**
- Who has final authority to approve/deny amendments - MLC or the school? **Decision**: School has final authority, MLC provides tools
- What is the process if parent disagrees with denial and requests hearing? **Decision**: School conducts hearing per their policy, MLC not involved
- Should technical errors (wrong score due to bug) be auto-corrected without formal amendment process? **Decision needed**: Yes, but with notification to school and logging

**Implementation Questions:**
- Should amendment requests be self-service or require admin intervention? **Decision needed**: Admin intervention in MVP, self-service portal in Phase 2
- How do we handle bulk amendment requests (e.g., wrong teacher assigned 30 students to wrong class)? **Decision needed**: Bulk correction tools for subscriber-admin, logged as single event

### Disclosure Logging

**Scope Questions:**
- Should teacher viewing student in their class be logged as disclosure? **Decision**: No - falls under school official exception
- Should every student dashboard view by student themselves be logged? **Decision**: No - not a disclosure (student accessing own records)
- Should parent viewing child's data be logged as disclosure (K-12)? **Decision**: No - parental access is not disclosure under FERPA
- Should exports for backup or migration purposes be logged? **Decision**: Yes - logged as internal disclosure for audit purposes

**Technical Questions:**
- How long should disclosure logs be retained? **Decision needed**: Recommend minimum 3 years, match record retention
- Should disclosure logs be accessible to parents/students on demand? **Decision**: Yes, via school administrator upon request

### Multi-Institutional Scenarios

**Transfers:**
- Should platform facilitate automated transfer of records between MLC-using schools? **Decision needed**: Phase 2 feature, requires school-to-school authorization workflow
- What consent is needed for transfer disclosures? **Decision**: None if covered in annual notice, otherwise reasonable attempt to notify

**Consortia:**
- Should multi-district consortia have shared or separate data? **Decision**: Separate by default with explicit data sharing agreements for any cross-district access
- Who is the "school" for FERPA purposes in a consortium? **Decision**: Each district is separate school, consortium agreement specifies roles

### Data Retention Edge Cases

**Graduation:**
- Should data be automatically deleted when student graduates? **Decision needed**: No - school determines retention, MLC retains per subscription status
- How long after graduation should schools retain educational records from MLC? **Decision**: School exports data for their own retention per state law, MLC follows subscription-based policy

**Student Withdrawal:**
- What happens to student data when student withdraws mid-year? **Decision needed**: Retained as part of school's records, school can delete individual student if needed
- Should withdrawn students be auto-deleted after X days? **Decision**: No automatic deletion, school decides

### Compliance Reporting

**Reports and Dashboards:**
- What metrics should be tracked in FERPA compliance dashboard? **Decision needed**: Access patterns, amendment requests, disclosure counts, rights requests
- Should schools have access to compliance dashboard for their organization? **Decision**: Yes, subscriber-admin dashboard includes FERPA compliance summary
- Who should receive automated compliance reports? **Decision needed**: System admin monthly, subscriber-admin optional enrollment

---

**Document Version**: 1.0  
**Last Updated**: [Current Date]  
**Next Review Date**: [Current Date + 6 months]  
**Owner**: Legal & Compliance Team, Product Team