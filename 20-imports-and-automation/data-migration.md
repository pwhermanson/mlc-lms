# Data Migration Specifications

## Overview
Comprehensive data migration strategy and implementation specifications for transitioning existing MLC 4.0 subscriber organizations, user accounts, content, and learning progress data to the new MLC 5.0 platform architecture. This specification ensures zero data loss, minimal downtime, and seamless continuity of service for active subscribers during the platform modernization.

## Purpose

This specification provides the operational blueprint for migrating all production data from the legacy MLC 4.0 Flash-based system to the modern MLC 5.0 HTML5/JavaScript platform. The migration is critical to:

- **Preserve Business Continuity**: Maintain all active subscriptions, billing relationships, and payment histories without interruption
- **Protect Learning Progress**: Retain all student scores, assignment completions, mastery achievements, and historical activity data
- **Maintain User Trust**: Ensure teachers and students can log in with existing credentials and continue their work immediately post-migration
- **Enable Modern Features**: Transform legacy data structures into the new relational model to unlock enhanced reporting, analytics, and integration capabilities
- **Comply with Regulations**: Maintain audit trails and data integrity for FERPA/COPPA compliance requirements
- **Support Phased Rollout**: Allow gradual migration of organizations with rollback capability if issues are detected

This specification is narrowly focused on the technical migration process itself. For ongoing CSV import workflows (post-migration), see [CSV Spec Students](./csv-spec-students.md), [CSV Spec Teachers](./csv-spec-teachers.md), and [Bulk Ops Jobs](./bulk-ops-jobs.md).

## Scope

### Included

**Migration Strategy and Phases**:
- Phased migration approach (pilot, staged rollout, final cutover)
- Pre-migration data analysis and cleanup procedures
- Migration validation and acceptance criteria
- Rollback and recovery procedures

**Data Entity Mapping**:
- MLC 4.0 to MLC 5.0 entity mapping for all core tables
- Data transformation rules and normalization procedures
- Default value assignment for new MLC 5.0 fields
- Orphaned and invalid data handling strategies

**Migration Execution**:
- ETL (Extract, Transform, Load) pipeline architecture
- Batch processing approach and parallelization strategy
- Progress monitoring and error reporting
- Downtime minimization and maintenance window planning

**Validation and Testing**:
- Pre-migration data quality checks
- Post-migration validation queries and data integrity tests
- Sample organization validation procedures
- User acceptance testing criteria

**Specific Entity Migrations**:
- User accounts (Members, Teachers, Students)
- Organizations and subscriptions (billing, seats, codes)
- Content library (games, sequences, groups, assignments)
- Learning progress (scores, attempts, mastery, achievements)
- Historical data (audit logs, activity logs, payment records)

**Special Cases and Edge Conditions**:
- Duplicate username resolution
- Invalid or corrupted data handling
- Incomplete subscription data repair
- Legacy data format conversions

### Excluded

- **Ongoing CSV Import Operations**: Post-migration bulk import workflows for new students/teachers (see [Bulk Ops Jobs](./bulk-ops-jobs.md))
- **CSV Format Specifications**: Detailed field specifications for ongoing imports (see [CSV Spec Students](./csv-spec-students.md), [CSV Spec Teachers](./csv-spec-teachers.md), etc.)
- **New Platform Features**: Feature development and enhancement work beyond migration (see respective feature specs)
- **Content Authoring**: New game creation or sequence authoring processes (see [Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md))
- **Infrastructure Setup**: Server provisioning, database configuration, deployment architecture (see [System Overview](../18-architecture/system-overview.md))
- **API Development**: REST endpoint implementation details (see [REST Endpoints](../18-architecture/rest-endpoints.md))

---

## Migration Strategy and Phasing

### Migration Approach Philosophy

The MLC 4.0 to 5.0 migration follows a **phased, reversible, and validated** approach to minimize risk while maintaining service continuity. The strategy prioritizes:

1. **Zero Data Loss**: All data is preserved, even if temporarily unused in MLC 5.0
2. **Minimal Downtime**: Most migration work occurs in parallel to production with brief cutover windows
3. **Pilot Validation**: Small cohort testing before full production rollout
4. **Rollback Capability**: Ability to revert to MLC 4.0 if critical issues are discovered
5. **User Communication**: Proactive notification and documentation for subscribers

### Phase 1: Analysis and Preparation (Pre-Migration)

**Objectives**:
- Analyze MLC 4.0 production database for data quality issues
- Identify orphaned records, duplicates, and data integrity problems
- Create reference snapshots and baseline metrics
- Develop and test ETL scripts in staging environment

**Key Activities**:

1. **Data Profiling**:
   - Count all entities (users, games, sequences, scores, subscriptions)
   - Identify null/missing critical fields
   - Detect duplicate usernames, email addresses
   - Find orphaned records (students without organizations, assignments without teachers)
   - Analyze character encoding issues and special characters

2. **Test Data Creation**:
   - Export anonymized sample data from production (3-5 organizations of varying sizes)
   - Create synthetic edge case data (very large orgs, complex hierarchies, unusual characters)
   - Document expected outcomes for validation testing

3. **Mapping Verification**:
   - Create detailed field-level mapping documentation (see Data Entity Mapping section)
   - Identify MLC 4.0 fields that have no MLC 5.0 equivalent (archive for reference)
   - Define transformation rules for format conversions

4. **Stakeholder Preparation**:
   - Identify pilot organization candidates (willing to test and provide feedback)
   - Prepare communication templates for subscribers
   - Create rollback communication plan

**Success Criteria**:
- ETL scripts successfully migrate 100% of test data with zero critical errors
- All validation queries pass on staging environment
- Pilot organizations identified and briefed

**Estimated Duration**: 2-3 weeks

---

### Phase 2: Pilot Migration (Limited Production)

**Objectives**:
- Migrate 3-5 real production organizations to MLC 5.0
- Validate migration accuracy with real users
- Identify unforeseen edge cases and issues
- Refine ETL scripts based on pilot learnings

**Key Activities**:

1. **Pilot Organization Selection**:
   - Select diverse organization types:
     - 1 small Solo subscriber (5-10 students)
     - 1 medium Ensemble school (50-100 students)
     - 1 large school district (500+ students)
     - 1 organization with complex sequence assignments
     - 1 organization with extensive historical data (2+ years)

2. **Pilot Migration Execution**:
   - Schedule maintenance window with pilot organizations (2-4 hours)
   - Set MLC 4.0 organizations to read-only mode
   - Execute migration ETL pipeline
   - Validate data completeness
   - Enable MLC 5.0 access for pilot users
   - Monitor first 48 hours of usage closely

3. **User Acceptance Testing**:
   - Pilot users verify:
     - Login with existing credentials works
     - Student rosters match expectations
     - Assignment history is preserved
     - Scores and progress are accurate
     - Subscription/billing information is correct
   - Collect feedback on any discrepancies or issues

4. **Iteration and Refinement**:
   - Fix any data issues identified during pilot
   - Update ETL scripts to handle newly discovered edge cases
   - Re-migrate pilot organizations if significant fixes are needed

**Success Criteria**:
- All pilot organizations successfully migrated with <1% data discrepancy rate
- Pilot users confirm core functionality works correctly
- No critical bugs or data loss incidents
- Rollback not required for any pilot organization

**Estimated Duration**: 1-2 weeks

---

### Phase 3: Staged Rollout (Production Migration)

**Objectives**:
- Migrate remaining production organizations in batches
- Monitor system performance and stability under increasing load
- Provide support for organizations as they transition

**Key Activities**:

1. **Batch Planning**:
   - Group organizations by size and activity level:
     - **Batch 1**: Inactive/trial organizations (low risk)
     - **Batch 2**: Small active organizations (5-20 students)
     - **Batch 3**: Medium organizations (20-100 students)
     - **Batch 4**: Large organizations (100-500 students)
     - **Batch 5**: Enterprise organizations (500+ students)
   - Schedule batch migrations 3-5 days apart to allow monitoring between batches

2. **Communication**:
   - Email subscribers 1 week before their scheduled migration
   - Provide FAQ document addressing common questions
   - Offer optional training webinar for new MLC 5.0 features
   - Send confirmation email with login instructions after migration

3. **Batch Migration Execution** (per batch):
   - Set organizations to read-only in MLC 4.0
   - Execute ETL pipeline for batch
   - Run validation queries
   - Enable MLC 5.0 access
   - Monitor system metrics (database load, API response times, error rates)
   - Provide 24-hour priority support for migrated organizations

4. **Issue Resolution**:
   - Triage and fix data discrepancies within 24 hours
   - Re-run migration for specific organizations if needed
   - Document issues and resolutions for future batches

**Success Criteria**:
- All organizations successfully migrated with <0.5% data discrepancy rate
- No system performance degradation as user load increases
- Support ticket volume within acceptable range (<5% of migrated users)
- No rollbacks required

**Estimated Duration**: 3-4 weeks

---

### Phase 4: Final Cutover and Decommissioning

**Objectives**:
- Complete migration of all remaining organizations
- Decommission MLC 4.0 system gracefully
- Establish MLC 5.0 as the sole production platform

**Key Activities**:

1. **Final Migration**:
   - Migrate any remaining organizations (stragglers, special cases)
   - Archive MLC 4.0 database as read-only backup

2. **MLC 4.0 Decommissioning**:
   - Set MLC 4.0 to read-only with redirect message to MLC 5.0
   - Maintain MLC 4.0 availability for 30 days post-cutover as safety net
   - After 30 days, redirect all MLC 4.0 URLs to MLC 5.0 equivalents
   - Archive MLC 4.0 codebase and database for compliance (retain 7 years)

3. **Post-Migration Support**:
   - Provide enhanced support for first 30 days post-cutover
   - Monitor for late-emerging issues
   - Collect feedback on migration experience

4. **Retrospective**:
   - Document lessons learned
   - Calculate actual vs. expected data discrepancy rates
   - Identify process improvements for future migrations

**Success Criteria**:
- 100% of organizations migrated
- MLC 4.0 gracefully decommissioned without service interruption
- Post-migration support ticket volume returns to baseline within 30 days

**Estimated Duration**: 1-2 weeks

---

## Data Entity Mapping: MLC 4.0 to MLC 5.0

This section maps legacy MLC 4.0 database tables and fields to the new MLC 5.0 data model. For detailed MLC 5.0 entity definitions, see [Data Model ERD](../18-architecture/data-model-erd.md).

### Legacy System Context

MLC 4.0 utilized a monolithic database structure with the following characteristics:
- **Flat table design** with heavy denormalization
- **Member-centric hierarchy**: All data organized under Member records
- **Numeric ID system**: Sequential integer IDs with prefixes (e.g., GAM-0950-2)
- **Limited referential integrity**: Foreign keys not consistently enforced
- **Mixed data types**: Inconsistent use of dates, booleans, and enums

For comprehensive legacy system documentation, see [Legacy Systems Overview](../00-foundations/legacy-systems.md).

---

### 1. User Accounts and Identity Migration

#### 1.1 Member (Subscriber Admin) Migration

**MLC 4.0 Source**: `tbl_members`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `member_number` | Organization | `legacy_member_number` | Store as reference; generate new UUID `org_id` |
| `member_number` | User | `legacy_member_id` | Store as reference; generate new UUID `user_id` |
| `last_name` | User | `last_name` | Direct copy, trim whitespace |
| `first_name` | User | `first_name` | Direct copy, trim whitespace |
| `email_1` | User | `email` | Direct copy, validate format, force lowercase |
| `email_2` | Organization | `billing_email` | Copy to org record if different from primary |
| `phone_1` | User | `phone` | Direct copy, normalize format |
| `school_name` | Organization | `org_name` | Copy to org record; use "{first_name} {last_name} Studio" if blank |
| `address_1`, `address_2`, `city`, `state`, `zip`, `country` | Organization | `address_line_1`, `address_line_2`, `city`, `state_province`, `postal_code`, `country` | Direct copy to billing address |
| `subscription_level` | Subscription | `plan_type` | Map: "Prelude" → `prelude`, "Solo" → `solo`, "Ensemble" → `ensemble` |
| `max_users` | Subscription | `seat_count` | Direct copy |
| `start_date` | Subscription | `created_at` | Parse date format, convert to ISO 8601 |
| `end_date` | Subscription | `current_period_end` | Parse date, set to null if subscription is cancelled |
| `suspend_period_start`, `suspend_period_restart` | Subscription | `paused_at`, `pause_resumes_at` | Copy if status is paused |
| `sponsor_code` | Organization | N/A | Link to SponsorCode via join table (see Promo/Sponsor Codes migration) |
| `sponsored_code` | Organization | N/A | Link to SponsorCode as sponsor (see Promo/Sponsor Codes migration) |
| `comments` | Organization | `admin_notes` | Direct copy |
| (none) | User | `role` | Set to `subscriber_admin` |
| (none) | User | `status` | Set to `active` if subscription active, else `suspended` |
| (none) | User | `username` | Generate from email prefix or `{LastName}{FirstName}{org_id}` pattern |
| (none) | User | `password_hash` | **Critical**: Migrate existing password hashes if compatible, else force password reset on first login |

**Special Handling**:
- **Username Generation**: If MLC 4.0 member has no username, generate using email prefix (before @) or name-based pattern
- **Password Migration**: If MLC 4.0 password hashing is incompatible with MLC 5.0 (bcrypt), flag account for password reset on first login
- **Member Groups**: If `member_group_number` is present, create Organization hierarchy relationship (parent-child orgs)

---

#### 1.2 Teacher Migration

**MLC 4.0 Source**: `tbl_teachers` (or teacher records within member structures)

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `teacher_id` | User | `legacy_teacher_id` | Store as reference; generate new UUID `user_id` |
| `teacher_real_name` | User | Parse into `first_name`, `last_name` | Split on comma or last space; handle "Lastname, Firstname" and "Firstname Lastname" formats |
| `teacher_username` | User | `username` | Direct copy, force lowercase, validate uniqueness within org |
| `teacher_password` | User | `password_hash` | Migrate hash if compatible, else flag for reset |
| `teacher_email` | User | `email` | Copy if present, else generate placeholder: `{username}@{org_domain}` or null |
| `member_number` | Organization | `org_id` | Link teacher to parent organization via foreign key |
| (none) | User | `role` | Set to `teacher` (or `teacher_admin` if indicated by permissions) |
| (none) | User | `status` | Set to `active` if org is active, else `suspended` |
| (none) | Seat | `seat_id`, `assigned_user_id` | Create Seat record linking teacher to subscription |

**Special Handling**:
- **Name Parsing**: Handle various formats: "Doe, Jane" vs "Jane Doe" vs "Jane Marie Doe"
- **Email Placeholder**: If no email provided, allow null or generate placeholder (clarify with business)
- **Teacher-Admin Role**: Check MLC 4.0 permissions table to determine if teacher should be elevated to `teacher_admin`
- **Orphaned Teachers**: If teacher's member_number doesn't exist, create placeholder organization or flag for manual review

---

#### 1.3 Student Migration

**MLC 4.0 Source**: `tbl_students`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `student_id` | User | `legacy_student_id` | Store as reference; generate new UUID `user_id` |
| `student_real_name` | User | Parse into `first_name`, `last_name` | Split on comma or last space |
| `student_username` | User | `username` | Direct copy, force lowercase, validate uniqueness within org |
| `student_password` | User | `password_hash` | Migrate hash if compatible, else flag for reset |
| `student_email` | User | `email` | Copy if present, else null (students rarely have emails) |
| `active_status` | User | `status` | Map: "Y" → `active`, blank/N → `suspended` |
| `teacher_id` | TeacherStudent | `teacher_id`, `student_id` | Create join table record linking student to teacher |
| `member_number` | Organization | `org_id` | Link student to parent organization via foreign key |
| `grade_level` | StudentProfile | `grade_level` | Direct copy |
| `class_name` | StudentProfile | `class_name` | Direct copy |
| `student_type` | User | `role` | Map: "regular" → `student`, "generic" → `generic_student` |
| `external_id` | User | `external_student_id` | Direct copy (SIS ID) |
| `date_of_birth` | User | `date_of_birth` | Parse date, handle various formats |
| `notes` | StudentProfile | `admin_notes` | Direct copy |
| (none) | Seat | `seat_id`, `assigned_user_id` | Create Seat record linking student to subscription |

**Special Handling**:
- **Duplicate Usernames**: If username already exists in org, append suffix: `SmithJohn17502` → `SmithJohn17502A`, `SmithJohn17502B`, etc.
- **Orphaned Students**: If teacher_id or member_number is invalid, link student directly to org without teacher assignment
- **Generic Students**: Generic student accounts (shared accounts for General Music classes) do not consume seats; flag appropriately

---

### 2. Subscription and Billing Migration

#### 2.1 Subscription Records

**MLC 4.0 Source**: Fields within `tbl_members`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `member_number` | Subscription | `legacy_member_number` | Store as reference; generate new UUID `subscription_id` |
| `subscription_level` | Subscription | `plan_type` | Map: "Prelude" → `prelude`, "Solo" → `solo`, "Ensemble" → `ensemble` |
| `billing_cycle` | Subscription | `billing_cycle` | Infer from payment history: recurring monthly → `monthly`, annual → `annual` |
| `max_users` | Subscription | `seat_count` | Direct copy |
| `start_date` | Subscription | `created_at` | Parse date, convert to ISO 8601 timestamp |
| `end_date` | Subscription | `current_period_end` | Parse date; if in past and still active, set to current date + billing cycle |
| `suspend_period_start` | Subscription | `paused_at` | Parse date if present |
| `suspend_period_restart` | Subscription | `pause_resumes_at` | Parse date if present |
| (none) | Subscription | `status` | Calculate: `trial` if in trial period, `active` if current, `paused` if suspended, `cancelled` if end_date passed |
| (none) | Subscription | `base_price_cents`, `per_seat_price_cents` | Calculate from plan_type using current pricing model (see [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md)) |

**Special Handling**:
- **Status Calculation**: Determine current status based on dates and payment history
- **Trial Period Detection**: If `start_date` is within 30 days and subscription level is "Prelude", mark as trial
- **Expired Subscriptions**: Organizations with `end_date` in past but no cancellation record may be auto-suspended; flag for review

---

#### 2.2 Payment History

**MLC 4.0 Source**: `tbl_payments` (if exists) or Braintree transaction history

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `payment_id` | Payment | `legacy_payment_id` | Store as reference; generate new UUID `payment_id` |
| `braintree_transaction_id` | Payment | `braintree_transaction_id` | Direct copy |
| `member_number` | Payment | `subscription_id` | Link to migrated Subscription via member_number lookup |
| `amount` | Payment | `amount_cents` | Convert to cents (multiply by 100) |
| `payment_date` | Payment | `processed_at` | Parse date, convert to ISO 8601 timestamp |
| `payment_method` | Payment | `payment_method` | Map to enum: credit card, paypal, invoice, po |
| `status` | Payment | `status` | Map to enum: pending, completed, failed, refunded |

**Special Handling**:
- **Historical Data**: Migrate all payment history for audit and reporting purposes
- **Currency**: If currency is not USD, store original currency and convert to cents appropriately

---

#### 2.3 Promo Codes and Sponsor Codes

**MLC 4.0 Source**: `tbl_promo_codes`, `tbl_sponsor_codes`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `promo_code` | PromoCode | `code` | Direct copy, force uppercase |
| `promo_code_name` | PromoCode | `name` | Direct copy |
| `num_students` | PromoCode | `max_students` | Direct copy |
| `num_days` | PromoCode | `duration_days` | Direct copy |
| `discount_percent` | PromoCode | `discount_percent` | Direct copy |
| `start_date`, `end_date` | PromoCode | `valid_from`, `valid_until` | Parse dates |
| `comments` | PromoCode | `description` | Direct copy |

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `sponsor_code` | SponsorCode | `code` | Direct copy (format: {member_number}A) |
| `sponsor_member_number` | SponsorCode | `sponsor_org_id` | Lookup Organization by legacy member_number |
| `sponsored_member_number` | OrganizationSponsor | `sponsored_org_id`, `sponsor_code_id` | Create join table record linking sponsored org to sponsor code |
| `assigned_date` | OrganizationSponsor | `assigned_at` | Parse date |

**Special Handling**:
- **Active Codes Only**: Only migrate codes with `end_date` in future or no end date
- **Sponsor Relationships**: Create OrganizationSponsor join table records to link sponsors to sponsored organizations

---

### 3. Content Library Migration

#### 3.1 Games Registry

**MLC 4.0 Source**: `tbl_games`, `tbl_learning_elements`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `game_id` (e.g., `GAM-0950-2`) | Game | `legacy_game_id` | Store as reference; generate new UUID `game_id` |
| `game_number` (e.g., `0950`) | Game | `game_number` | Direct copy |
| `game_name` | Game | `title` | Direct copy, trim whitespace |
| `game_description` | Game | `description` | Direct copy |
| `concept_category` | Game | `primary_concept` | Map to taxonomy (see [Game Taxonomy](../07-content-architecture/game-taxonomy.md)) |
| `concept_tags` | Game | `concept_tags` | Parse comma-separated list, map to taxonomy |
| `game_level` | Game | `difficulty_level` | Map: "P" → `primary_1a`, "1" → `level_1`, etc. |
| `midi_capable` | Game | `requires_midi` | Map: "M" flag → `true` |
| `solfege_game` | Game | `is_solfege` | Map: "S" flag → `true` |
| `stage` (e.g., `LEARN`, `PLAY`, `QUIZ`) | GameStage | `stage_type` | Create GameStage records for each stage; map to enum |
| `asset_path` | Game | `swf_path` (legacy), `html5_path` | Preserve SWF path; html5_path initially null (requires conversion) |
| `active_status` | Game | `status` | Map: "A" → `published`, blank → `draft` |

**Special Handling**:
- **Game Stages**: Each MLC 4.0 game with multiple stages (LEARN, PLAY, QUIZ, CHAL, REV1) requires multiple GameStage records in MLC 5.0
- **Asset Migration**: SWF files are NOT converted during migration; html5_path remains null until games are converted
- **Learning Element Codes**: Preserve legacy element number format (GAM-0950-2) for backward compatibility and reference
- **Deprecated Games**: Games marked inactive in MLC 4.0 are migrated with status=`archived`

For detailed game model structure, see [Game Model](../08-games-registry-and-authoring/game-model.md).

---

#### 3.2 Named Sequences and Curriculum

**MLC 4.0 Source**: `tbl_named_sequences`, `tbl_sequence_definitions`, sequence-specific tables (e.g., `LIFE-000010`)

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `sequence_code` (e.g., `LIFE`, `ALFR`) | Sequence | `legacy_sequence_code` | Store as reference; generate new UUID `sequence_id` |
| `sequence_name` | Sequence | `title` | Direct copy |
| `level_title` | SequenceLevel | `level_title` | Create SequenceLevel records |
| `unit_title` | SequenceUnit | `unit_title` | Create SequenceUnit records nested under levels |
| `seq_table_number` (e.g., `LIFE-000010`) | SequenceUnit | `legacy_unit_id` | Store as reference |
| `seq_order` | SequenceStep | `order_index` | Preserve order within each unit |
| `element_number` | SequenceStep | `game_id` or `content_id` | Lookup Game or other content by legacy element number |
| `element_type` | SequenceStep | `content_type` | Map: "GAM" → `game`, "VID" → `video`, "TXT" → `text`, "AUD" → `audio` |
| `stage` | SequenceStep | `stage_type` | Map to enum: `learn`, `play`, `quiz`, `challenge`, `review` |

**Special Handling**:
- **Nested Hierarchy**: MLC 4.0 sequences have Level → Unit → Assignment structure; MLC 5.0 uses Sequence → Level → Unit → Steps
- **Cross-References**: Sequence steps reference games and other content; ensure all referenced content is migrated first
- **Method-Specific Sequences**: Piano method sequences (Alfred, Faber, Hal Leonard) must preserve page/unit references for correlation

For detailed sequence model structure, see [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md).

---

### 4. Learning Progress and Activity Migration

#### 4.1 Student Assignments

**MLC 4.0 Source**: `tbl_student_assignments`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `assignment_id` | StudentAssignment | `legacy_assignment_id` | Store as reference; generate new UUID `assignment_id` |
| `student_id` | StudentAssignment | `student_id` | Lookup User by legacy_student_id |
| `teacher_id` | StudentAssignment | `assigned_by_user_id` | Lookup User by legacy_teacher_id |
| `sequence_code`, `level`, `unit` | StudentAssignment | `sequence_unit_id` | Lookup SequenceUnit by legacy identifiers |
| `assigned_date` | StudentAssignment | `assigned_at` | Parse date, convert to timestamp |
| `completed_date` | StudentAssignment | `completed_at` | Parse date if present |
| `status` | StudentAssignment | `status` | Map to enum: `not_started`, `in_progress`, `completed` |

**Special Handling**:
- **In-Progress Assignments**: Preserve current step/position within assignment
- **Completion Tracking**: Migrate completion timestamps for historical reporting

---

#### 4.2 Game Scores and Attempts

**MLC 4.0 Source**: `tbl_student_scores`, `tbl_game_attempts`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `score_id` | GameAttempt | `legacy_score_id` | Store as reference; generate new UUID `attempt_id` |
| `student_id` | GameAttempt | `student_id` | Lookup User by legacy_student_id |
| `game_id` | GameAttempt | `game_id` | Lookup Game by legacy_game_id |
| `stage` | GameAttempt | `stage_type` | Map to enum |
| `score` | GameAttempt | `score` | Direct copy |
| `target_score` | GameAttempt | `target_score` | Copy from game definition if not present |
| `mastery_achieved` | GameAttempt | `is_mastery` | Map: green checkmark → `true` |
| `attempt_date` | GameAttempt | `completed_at` | Parse timestamp |
| `time_spent_seconds` | GameAttempt | `duration_seconds` | Direct copy if available |

**Special Handling**:
- **High Volume**: Student scores table is typically the largest table; use batched processing
- **Historical Data**: Migrate all attempts to preserve learning trajectory and analytics
- **Mastery Calculation**: If mastery flag is missing, recalculate based on score >= target_score

---

#### 4.3 Badges and Achievements

**MLC 4.0 Source**: `tbl_student_badges`, `tbl_achievements`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `badge_id` | StudentBadge | `legacy_badge_id` | Store as reference; generate new UUID |
| `student_id` | StudentBadge | `student_id` | Lookup User |
| `badge_type` | StudentBadge | `badge_id` | Lookup Badge definition by type |
| `earned_date` | StudentBadge | `earned_at` | Parse timestamp |

**Special Handling**:
- **Badge Definitions**: Ensure badge definitions are migrated to MLC 5.0 BadgeDefinition table first
- **Legacy Badges**: If MLC 4.0 badge types don't exist in MLC 5.0, create new definitions or map to closest equivalent

---

### 5. Messaging and Communication Migration

**MLC 4.0 Source**: `tbl_messages`, `tbl_announcements`

| MLC 4.0 Field | MLC 5.0 Entity | MLC 5.0 Field | Transformation Rule |
|---------------|----------------|---------------|---------------------|
| `message_id` | Message | `legacy_message_id` | Store as reference; generate new UUID |
| `sender_id` | Message | `sender_user_id` | Lookup User |
| `recipient_id` | Message | `recipient_user_id` | Lookup User |
| `message_text` | Message | `body` | Direct copy, sanitize HTML |
| `sent_date` | Message | `sent_at` | Parse timestamp |
| `read_date` | Message | `read_at` | Parse timestamp if present |

**Special Handling**:
- **Message History**: Migrate last 6 months of messages; archive older messages for reference if needed
- **System Announcements**: Migrate active announcements only; expired announcements can be archived

---

## ETL Pipeline Architecture

### Pipeline Overview

The migration utilizes a custom ETL (Extract, Transform, Load) pipeline built with Python and SQL scripts. The pipeline is designed for:
- **Idempotency**: Can be re-run safely; duplicate records are detected and skipped
- **Parallelization**: Independent entity types can be migrated in parallel
- **Progress Tracking**: Real-time monitoring and logging
- **Error Isolation**: Failed records are logged and skipped; valid records continue processing

### ETL Execution Order

Entities must be migrated in dependency order to satisfy foreign key constraints:

**Order 1: Foundation Entities** (no dependencies)
1. Badge definitions
2. Promo codes
3. Sponsor codes
4. Game definitions and stages
5. Sequence definitions (named sequences)

**Order 2: Organizational Entities** (depend on foundation)
6. Organizations (from members)
7. Subscriptions (linked to organizations)
8. Seats (linked to subscriptions)

**Order 3: User Accounts** (depend on organizations)
9. Users: Subscriber Admins
10. Users: Teachers
11. Users: Students
12. Seat assignments (link users to seats)
13. Teacher-Student relationships (join table)

**Order 4: Content and Assignments** (depend on users and content)
14. Sequence steps (link games to sequences)
15. Student assignments (link students to sequence units)

**Order 5: Activity and Progress** (depend on everything)
16. Game attempts and scores
17. Assignment progress (steps completed)
18. Student badges and achievements

**Order 6: Communication and Metadata** (lowest priority)
19. Messages and announcements
20. Audit logs and activity logs
21. Payment history (already in Braintree; optional)

### ETL Script Structure

Each entity type has a dedicated ETL script following this pattern:

```python
# Example: migrate_teachers.py

import logging
from database import mlc4_connection, mlc5_connection
from utils import generate_uuid, normalize_string, parse_legacy_date

logger = logging.getLogger(__name__)

def migrate_teachers(batch_size=1000):
    """Migrate all teachers from MLC 4.0 to MLC 5.0."""
    
    logger.info("Starting teacher migration...")
    
    # Extract: Fetch teachers from MLC 4.0
    query = """
        SELECT teacher_id, member_number, teacher_real_name, 
               teacher_username, teacher_password_hash, teacher_email,
               active_status, created_date
        FROM tbl_teachers
        WHERE deleted_at IS NULL
        ORDER BY teacher_id
    """
    
    mlc4_cursor = mlc4_connection.cursor()
    mlc4_cursor.execute(query)
    
    migrated_count = 0
    error_count = 0
    
    while True:
        batch = mlc4_cursor.fetchmany(batch_size)
        if not batch:
            break
            
        for teacher in batch:
            try:
                # Transform: Parse and normalize data
                user_id = generate_uuid()
                first_name, last_name = parse_name(teacher['teacher_real_name'])
                org_id = lookup_org_by_member_number(teacher['member_number'])
                
                if not org_id:
                    logger.warning(f"Orphaned teacher {teacher['teacher_id']}: member_number {teacher['member_number']} not found")
                    continue
                
                # Load: Insert into MLC 5.0
                mlc5_connection.execute("""
                    INSERT INTO users (
                        user_id, username, email, password_hash,
                        first_name, last_name, role, status,
                        org_id, legacy_teacher_id, created_at
                    ) VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
                """, (
                    user_id,
                    normalize_string(teacher['teacher_username']),
                    teacher['teacher_email'],
                    teacher['teacher_password_hash'],
                    first_name,
                    last_name,
                    'teacher',
                    'active' if teacher['active_status'] == 'Y' else 'suspended',
                    org_id,
                    teacher['teacher_id'],
                    parse_legacy_date(teacher['created_date'])
                ))
                
                migrated_count += 1
                
            except Exception as e:
                logger.error(f"Failed to migrate teacher {teacher['teacher_id']}: {e}")
                error_count += 1
                
    logger.info(f"Teacher migration complete: {migrated_count} migrated, {error_count} errors")
    return migrated_count, error_count
```

### Progress Monitoring

Migration progress is tracked via:
- **Real-time console logging**: Shows current entity, progress percentage, and errors
- **Database migration_status table**: Tracks completion of each entity type with timestamps
- **Error log file**: Detailed error messages for failed records
- **Metrics dashboard**: Grafana dashboard showing migration progress across all batches

---

## Validation and Testing

### Pre-Migration Validation

Before executing migration on production data, validate:

1. **Data Quality Checks**:
   - No critical fields are NULL in required columns
   - All foreign key references are valid (e.g., every student's teacher_id exists)
   - Date fields are in valid ranges (not year 1900 or 2099)
   - Usernames are unique within each organization

2. **Test Data Migration**:
   - Run full ETL pipeline on staging database with test data
   - Verify all expected records are migrated
   - Check data types, formats, and values match expectations

3. **Script Dry-Run**:
   - Execute ETL scripts in validation-only mode (no writes)
   - Output data transformation results for manual review
   - Identify potential errors before committing changes

### Post-Migration Validation Queries

After migration, run comprehensive validation SQL queries:

#### Count Reconciliation

```sql
-- Verify user counts match between systems
SELECT 
    'Users' AS entity,
    (SELECT COUNT(*) FROM mlc4.tbl_members) + 
    (SELECT COUNT(*) FROM mlc4.tbl_teachers) + 
    (SELECT COUNT(*) FROM mlc4.tbl_students) AS mlc4_count,
    (SELECT COUNT(*) FROM mlc5.users) AS mlc5_count,
    mlc5_count - mlc4_count AS difference
UNION ALL
SELECT 
    'Organizations' AS entity,
    (SELECT COUNT(DISTINCT member_number) FROM mlc4.tbl_members) AS mlc4_count,
    (SELECT COUNT(*) FROM mlc5.organizations) AS mlc5_count,
    mlc5_count - mlc4_count AS difference;
-- Expected: difference = 0 for all entities
```

#### Data Integrity Checks

```sql
-- Check for orphaned students (students without valid org)
SELECT COUNT(*) AS orphaned_students
FROM mlc5.users u
LEFT JOIN mlc5.organizations o ON u.org_id = o.org_id
WHERE u.role = 'student' AND o.org_id IS NULL;
-- Expected: 0

-- Check for missing game references in sequence steps
SELECT COUNT(*) AS missing_games
FROM mlc5.sequence_steps ss
LEFT JOIN mlc5.games g ON ss.game_id = g.game_id
WHERE ss.content_type = 'game' AND g.game_id IS NULL;
-- Expected: 0
```

#### Sample Data Spot Checks

```sql
-- Verify specific organization migrated correctly
SELECT 
    o.org_name,
    s.plan_type,
    s.seat_count,
    COUNT(u.user_id) AS user_count
FROM mlc5.organizations o
JOIN mlc5.subscriptions s ON o.org_id = s.org_id
LEFT JOIN mlc5.users u ON o.org_id = u.org_id
WHERE o.legacy_member_number = 17502  -- Replace with pilot org member number
GROUP BY o.org_name, s.plan_type, s.seat_count;
-- Manually compare to MLC 4.0 data
```

### User Acceptance Testing Checklist

Pilot organization users should validate:

- [ ] **Login**: Can log in with existing username and password
- [ ] **Dashboard**: Dashboard displays correctly with accurate user info
- [ ] **Student Roster**: All students appear in roster with correct names
- [ ] **Teacher Assignments**: Students are assigned to correct teachers
- [ ] **Sequence Assignments**: Student assignments match MLC 4.0 assignments
- [ ] **Score History**: Student scores and progress are accurate
- [ ] **Badges**: Earned badges are displayed correctly
- [ ] **Subscription Info**: Billing information and subscription level are accurate
- [ ] **Game Play**: Students can launch and play games successfully
- [ ] **Assignment Completion**: Completing a game updates progress correctly

---

## Rollback Strategy

### When to Rollback

Rollback to MLC 4.0 is triggered if:
- **Critical Data Loss**: >1% of records are missing or corrupted after migration
- **System Instability**: MLC 5.0 platform experiences critical bugs preventing core functionality
- **User Outcry**: >20% of migrated organizations report critical issues within 48 hours
- **Security Incident**: Data breach or unauthorized access discovered post-migration

### Rollback Procedure

**Immediate Actions** (within 1 hour):
1. Set MLC 5.0 to read-only maintenance mode with status message
2. Redirect all traffic back to MLC 4.0
3. Re-enable MLC 4.0 write access
4. Notify affected organizations via email and in-app message

**Data Reconciliation** (within 24 hours):
1. Identify any data created in MLC 5.0 during migration window (new scores, messages, etc.)
2. Manually export and import this data back to MLC 4.0
3. Validate data consistency between systems

**Post-Rollback Analysis**:
1. Conduct incident retrospective to identify root cause
2. Fix critical issues in MLC 5.0 before attempting re-migration
3. Communicate revised migration timeline to subscribers

**Prevention Measures**:
- Maintain MLC 4.0 in read-only mode during migration to minimize reconciliation effort
- Implement feature flags to disable writes in MLC 5.0 if issues detected
- Schedule migrations during low-usage windows (weekends, evenings)

---

## Special Cases and Edge Conditions

### 1. Duplicate Usernames

**Problem**: Multiple students with same name in organization (e.g., two "John Smith" students)

**MLC 4.0 Behavior**: Appends letter suffix: `SmithJohn17502`, `SmithJohn17502A`, `SmithJohn17502B`

**Migration Handling**:
- Preserve existing suffixed usernames as-is
- If new duplicates are created during migration, follow same letter-suffix pattern
- Validate username uniqueness within org_id scope before insert

### 2. Invalid or Corrupted Data

**Problem**: Records with missing critical fields, invalid foreign keys, or corrupted text

**Migration Handling**:
- **Missing Critical Fields**: Log error and skip record; flag for manual review
- **Invalid Foreign Keys**: Attempt to resolve via fuzzy matching (e.g., member_number with leading zeros); if unresolvable, create placeholder parent record or skip
- **Corrupted Text**: Attempt UTF-8 encoding fix; if fails, store as base64 and flag for manual review
- **Invalid Dates**: If date is clearly invalid (year 1900, 2099), set to null and log warning

### 3. Orphaned Records

**Problem**: Child records reference parent records that don't exist (e.g., student with invalid teacher_id)

**Migration Handling**:
- **Orphaned Students**: Link directly to organization without teacher assignment; flag for manual teacher assignment post-migration
- **Orphaned Scores**: If student_id or game_id is invalid, skip record and log error (scores cannot exist without context)
- **Orphaned Assignments**: Skip and log error; assignments require valid student and sequence

### 4. Large Organizations

**Problem**: Organizations with 1000+ students may cause memory or timeout issues

**Migration Handling**:
- Use batched processing (500-1000 records per batch)
- Migrate large organizations during dedicated maintenance windows
- Monitor database connection pool and memory usage during processing
- Implement progress checkpoints to resume if interrupted

### 5. Inactive/Archived Organizations

**Problem**: Organizations with cancelled subscriptions or no recent activity

**Migration Strategy**:
- **Recently Inactive** (<6 months): Migrate fully; users may return
- **Long-Term Inactive** (>1 year): Migrate with status=`archived`; data accessible but not active
- **Deactivated Accounts**: Migrate with status=`suspended`; can be reactivated on request

### 6. Password Hash Compatibility

**Problem**: MLC 4.0 password hashing algorithm may differ from MLC 5.0 (bcrypt)

**Migration Handling**:
- If hashes are compatible: Migrate directly
- If incompatible: Migrate user accounts but flag `requires_password_reset = true`
- Send email to users on first login prompting password reset
- Provide "Forgot Password" flow for seamless transition

### 7. Character Encoding Issues

**Problem**: Special characters, accents, or non-English names may have encoding issues

**Migration Handling**:
- Ensure ETL pipeline uses UTF-8 encoding throughout
- Test with sample data containing accents (José, François, Müller)
- Handle emoji and special characters gracefully (strip or preserve based on field)

---

## Migration Timeline and Resource Requirements

### Estimated Timeline

| Phase | Duration | Key Milestones |
|-------|----------|----------------|
| Phase 1: Analysis & Prep | 2-3 weeks | ETL scripts complete, test data migrated, pilot orgs identified |
| Phase 2: Pilot Migration | 1-2 weeks | 3-5 orgs migrated, UAT complete, issues resolved |
| Phase 3: Staged Rollout | 3-4 weeks | All production orgs migrated in batches |
| Phase 4: Final Cutover | 1-2 weeks | MLC 4.0 decommissioned, MLC 5.0 sole production |
| **Total** | **7-11 weeks** | **100% migration complete** |

### Resource Requirements

**Engineering Team**:
- **Backend Developer** (full-time): ETL script development, validation query writing
- **Database Administrator** (50% time): Schema migration, query optimization, backup management
- **QA Engineer** (50% time): Validation testing, UAT coordination, bug triage
- **DevOps Engineer** (25% time): Infrastructure setup, monitoring, deployment automation

**Business Team**:
- **Product Manager** (25% time): Stakeholder communication, rollback decision authority
- **Customer Success** (50% time): Pilot org coordination, user training, support ticket triage

**Tools and Infrastructure**:
- **Staging Environment**: Full MLC 5.0 deployment for testing
- **Migration Server**: Dedicated server for running ETL scripts (8 CPU, 32GB RAM recommended)
- **Monitoring**: Grafana dashboard for real-time progress tracking
- **Communication**: Email templates, status page, support documentation

---

## Supporting Documents Referenced

This data migration specification draws from the following source documents:

- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — MLC 4.0 system overview and business context informing migration scope and requirements
- [MLC History Draft 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20History%20Draft%202020.docx.txt) — Historical context and legacy system evolution informing backward compatibility requirements
- [Mass Upload.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Mass%20Upload.xlsx%20-%20Sheet1.csv) — Legacy student import format informing user data migration mapping
- [Course import.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Course%20import.xlsx%20-%20Sheet1.csv) — Legacy course/assignment import format informing curriculum data migration structure
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Legacy game registry format informing content library migration mapping
- [Couurse ID numbers.xlsx - Sheet 1 - tbl_contents.csv](../00-ORIG-CONTEXT/Couurse%20ID%20numbers.xlsx%20-%20Sheet%201%20-%20tbl_contents.csv) — Legacy course ID numbering scheme informing ID preservation strategy
- [AlphaMasterList2020.xlsx - Master.csv](../00-ORIG-CONTEXT/AlphaMasterList2020.xlsx%20-%20Master.csv) — Legacy sequence/assignment master list informing curriculum structure migration
- [MLC SeqCreate 2020.xlsx - ALPHA LIST .csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20ALPHA%20LIST%20.csv) — Legacy sequence creation specifications informing sequence versioning and structure preservation
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Legacy subscription and seat management specifications informing subscription data migration
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Legacy user role definitions informing role migration and permission mapping

---

## Related Documentation

- [Legacy Systems Overview](../00-foundations/legacy-systems.md) - Detailed MLC 4.0 architecture and history
- [Data Model ERD](../18-architecture/data-model-erd.md) - Complete MLC 5.0 entity relationship diagram
- [CSV Spec Students](./csv-spec-students.md) - Post-migration CSV import format for students
- [CSV Spec Teachers](./csv-spec-teachers.md) - Post-migration CSV import format for teachers
- [Bulk Ops Jobs](./bulk-ops-jobs.md) - Ongoing bulk operation workflows
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game entity structure and fields
- [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) - Sequence curriculum structure
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) - Subscription pricing logic
- [Seat Management](../14-payments-and-subscriptions/seat-management.md) - Seat allocation and tracking
