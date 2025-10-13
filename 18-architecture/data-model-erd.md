# Data Model ERD

This document defines the entity relationship diagram and canonical data model for the MLC-LMS platform. It serves as the single source of truth for understanding how entities relate across all domains, enabling consistent implementation across features, reports, and integrations.

## Purpose

This specification provides a comprehensive view of the platform's data architecture, enabling:

- **Unified Understanding**: A single reference for all entities, relationships, and data flows across the entire system
- **Implementation Consistency**: Ensures database schema, API contracts, and application logic align with the canonical model
- **Integration Planning**: Clarifies data dependencies for external integrations (billing, analytics, LMS connectors)
- **Migration Strategy**: Documents the path from legacy MLC 4.0 structure to the modern relational model
- **Query Optimization**: Informs indexing, caching, and query patterns by making relationships explicit
- **Reporting Foundation**: Enables accurate cross-domain analytics and KPI calculations

This ERD complements detailed entity specifications in individual feature documents by showing how they interconnect.

## Scope

### Included
- **Core Entity Definitions**: All primary entities with identifiers, key fields, and types
- **Relationship Mapping**: Foreign keys, join tables, and cardinality (1:1, 1:N, M:N)
- **Domain Organization**: How entities group into bounded contexts (Identity, Subscriptions, Content, Learning, etc.)
- **Identifier Patterns**: Naming conventions and ID generation strategies
- **Versioning Strategy**: How content versions and historical data are maintained
- **Legacy Migration**: Mapping from MLC 4.0 tables to MLC 5.0 entities
- **Key Indexes**: Critical indexes for performance and data integrity

### Excluded
- **Detailed Field Specifications**: Full schemas are in domain-specific docs (e.g., [Game Model](../08-games-registry-and-authoring/game-model.md), [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))
- **Business Logic**: State transitions, validation rules, and workflows (covered in feature specs)
- **API Endpoint Design**: REST contracts and request/response formats (see [REST Endpoints](rest-endpoints.md))
- **Physical Database Design**: Partitioning, sharding, replication strategies (implementation-specific)
- **Migration Scripts**: Actual SQL/ETL code for data migration (operational documentation)

## Data model (if applicable)

### Domain Organization

The data model is organized into seven bounded contexts, each owning specific entities:

1. **Identity & Access**: Users, roles, sessions, permissions
2. **Subscription & Billing**: Organizations, subscriptions, seats, payments, codes
3. **User Profiles**: Students, teachers, admins, classes, groups
4. **Content**: Games, sequences, groups, assignments (definitions)
5. **Learning Progress**: Student assignments, steps, scores, attempts, mastery
6. **Gamification**: Badges, achievements, challenges, leaderboards
7. **Communication**: Messages, notifications, announcements

### Core Entities Overview

#### Identity & Access Domain

**User**
- Primary entity for all platform users
- `user_id` (UUID, primary key)
- `username` (unique, indexed)
- `email` (unique, indexed)
- `password_hash` (bcrypt)
- `role` (enum: subscriber_admin, teacher_admin, teacher, student, generic_student, system_admin)
- `status` (enum: active, suspended, archived)
- `created_at`, `updated_at`, `last_login_at`
- **Related to**: Session (1:N), UserProfile (1:1), AuditLog (1:N)

**Session**
- Tracks active user sessions for authentication
- `session_id` (UUID, primary key)
- `user_id` (FK → User)
- `token` (JWT, indexed)
- `ip_address`, `user_agent`
- `created_at`, `expires_at`
- **Related to**: User (N:1)

**Role & Permission** (future enhancement)
- Granular permission system for advanced RBAC
- Currently permissions are role-based enums
- See [Permissions Table](../02-roles-and-permissions/permissions-table.md) for detailed matrix

**AuditLog**
- Tracks significant user actions for compliance and security
- `log_id` (UUID, primary key)
- `user_id` (FK → User)
- `action_type` (enum: login, password_change, role_change, data_access, etc.)
- `entity_type`, `entity_id`
- `ip_address`, `timestamp`
- `metadata` (JSONB)
- **Related to**: User (N:1)

#### Subscription & Billing Domain

**Organization**
- Top-level subscriber entity (school, studio, institution)
- `org_id` (UUID, primary key)
- `org_name`
- `org_type` (enum: solo, ensemble, school_district)
- `owner_user_id` (FK → User, the Subscriber-Admin)
- `status` (enum: active, trial, suspended, cancelled)
- `created_at`, `updated_at`
- **Related to**: User (N:1 owner), Subscription (1:N), PromoCode (M:N), SponsorCode (M:N)

**Subscription**
- Represents a billing subscription
- `subscription_id` (UUID, primary key)
- `org_id` (FK → Organization)
- `plan_type` (enum: prelude, solo, ensemble)
- `billing_cycle` (enum: monthly, annual)
- `seat_count` (integer)
- `base_price_cents`, `per_seat_price_cents`
- `status` (enum: trial, active, paused, cancelled)
- `trial_ends_at`, `current_period_start`, `current_period_end`
- `created_at`, `updated_at`, `cancelled_at`
- **Related to**: Organization (N:1), Payment (1:N), Invoice (1:N), Seat (1:N)
- See [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) for pricing logic

**Seat**
- Represents an allocated seat within a subscription
- `seat_id` (UUID, primary key)
- `subscription_id` (FK → Subscription)
- `assigned_user_id` (FK → User, nullable)
- `seat_type` (enum: student, teacher, admin)
- `status` (enum: vacant, assigned, suspended)
- `assigned_at`, `created_at`
- **Related to**: Subscription (N:1), User (N:1 optional)
- See [Seat Management](../14-payments-and-subscriptions/seat-management.md)

**Payment**
- Records payment transactions via Braintree
- `payment_id` (UUID, primary key)
- `subscription_id` (FK → Subscription)
- `amount_cents`, `currency`
- `payment_method` (enum: credit_card, paypal, invoice, po)
- `braintree_transaction_id`
- `status` (enum: pending, completed, failed, refunded)
- `processed_at`, `created_at`
- **Related to**: Subscription (N:1), Invoice (N:1)

**Invoice**
- Generated billing statements
- `invoice_id` (UUID, primary key)
- `subscription_id` (FK → Subscription)
- `invoice_number` (unique, human-readable)
- `amount_cents`, `tax_cents`, `total_cents`
- `period_start`, `period_end`
- `due_date`, `paid_at`
- `status` (enum: draft, sent, paid, overdue, void)
- `pdf_url` (Cloudflare R2 reference)
- `created_at`
- **Related to**: Subscription (N:1), Payment (1:1 optional)

**PromoCode**
- Promotional discount codes
- `promo_code_id` (UUID, primary key)
- `code` (string, unique, indexed)
- `code_name`
- `discount_type` (enum: percentage, fixed_amount, free_seats, free_trial_extension)
- `discount_value`
- `max_uses`, `current_uses`
- `valid_from`, `valid_until`
- `status` (enum: active, expired, disabled)
- **Related to**: Organization (M:N via PromoCodeUsage)
- See [Promo Codes](../14-payments-and-subscriptions/promo-codes.md)

**SponsorCode**
- Sponsor codes for grant programs
- `sponsor_code_id` (UUID, primary key)
- `code` (string, unique, indexed, format: `{member_number}A`)
- `sponsor_org_id` (FK → Organization)
- `sponsored_org_id` (FK → Organization, nullable until assigned)
- `assigned_at`
- `status` (enum: available, assigned, expired)
- **Related to**: Organization (N:1 sponsor, N:1 sponsored)
- See [Sponsor Codes](../14-payments-and-subscriptions/sponsor-codes.md)

#### User Profiles Domain

**StudentProfile**
- Extended profile for students
- `student_id` (UUID, primary key)
- `user_id` (FK → User)
- `org_id` (FK → Organization)
- `assigned_teacher_id` (FK → TeacherProfile, nullable)
- `grade_level` (enum: primary_1a, level_1, level_2, etc.)
- `preferred_sequence_id` (FK → Sequence, defaults to LIFE)
- `display_name`, `avatar_url`
- `settings` (JSONB: notifications, accessibility, game preferences)
- `created_at`, `updated_at`
- **Related to**: User (1:1), TeacherProfile (N:1), Organization (N:1), Class (M:N)

**TeacherProfile**
- Extended profile for teachers
- `teacher_id` (UUID, primary key)
- `user_id` (FK → User)
- `org_id` (FK → Organization)
- `can_create_students` (boolean)
- `can_manage_classes` (boolean)
- `display_name`
- `created_at`, `updated_at`
- **Related to**: User (1:1), Organization (N:1), StudentProfile (1:N), Class (1:N)

**AdminProfile**
- Extended profile for Subscriber-Admin and Teacher-Admin
- `admin_id` (UUID, primary key)
- `user_id` (FK → User)
- `org_id` (FK → Organization)
- `admin_type` (enum: subscriber_admin, teacher_admin)
- `display_name`
- `created_at`, `updated_at`
- **Related to**: User (1:1), Organization (N:1)

**Class**
- Grouping of students under a teacher
- `class_id` (UUID, primary key)
- `teacher_id` (FK → TeacherProfile)
- `org_id` (FK → Organization)
- `class_name`
- `grade_level`
- `is_generic` (boolean, for shared login classes)
- `max_students`
- `created_at`, `updated_at`
- **Related to**: TeacherProfile (N:1), Organization (N:1), StudentProfile (M:N via ClassEnrollment)
- See [Group Model](../07-content-architecture/group-model.md)

**ClassEnrollment** (join table)
- `enrollment_id` (UUID, primary key)
- `class_id` (FK → Class)
- `student_id` (FK → StudentProfile)
- `enrolled_at`, `status` (enum: active, archived)

#### Content Domain

**Game**
- Master game definition
- `game_id` (string, primary key, format: `G-{sequential}`, e.g., `G-01542`)
- `game_version` (integer, default 1)
- `slug` (string, unique, indexed)
- `name`
- `status` (enum: draft, live, deprecated)
- `replaced_by_game_id` (FK → Game, nullable)
- `instrument` (enum: piano, general_music, choir)
- `mode` (enum: visual, aural, mixed)
- `device_profile` (enum: any, midi_required, mic_optional)
- `keyboard_mode` (enum: keyboard, non_keyboard)
- `level` (enum: beginner, early_intermediate, intermediate, advanced)
- `difficulty` (integer 1-10)
- `concept_tags` (array of strings, indexed for search)
- `category` (string: Staff, Intervals, Rhythm, etc.)
- `standards` (array of strings: custom:mlc:L1.staff.guide_notes)
- `pedagogical_notes` (text)
- `assets_url` (Cloudflare R2 reference)
- `created_at`, `updated_at`, `published_at`
- **Related to**: GameStage (1:N), Sequence (M:N via SequenceStep), Assignment (M:N via AssignmentStep)
- See [Game Model](../08-games-registry-and-authoring/game-model.md)

**GameStage**
- Individual stages within a game (Learn, Play, Quiz, Challenge, Review)
- `stage_id` (string, primary key, format: `G-{game}-{type}`, e.g., `G-01542-quiz`)
- `game_id` (FK → Game)
- `stage_type` (enum: learn, play, quiz, challenge, review)
- `stage_version` (integer, default 1)
- `target_score` (integer, nullable)
- `pass_threshold` (integer, nullable)
- `time_limit_seconds` (integer, nullable)
- `enabled` (boolean)
- `created_at`, `updated_at`
- **Related to**: Game (N:1), SequenceStep (1:N), AssignmentStep (1:N), StudentAttempt (1:N)

**Sequence**
- Ordered curriculum pathway
- `sequence_id` (string, primary key, format: `SEQ-{CODE}`, e.g., `SEQ-LIFE`)
- `sequence_version` (integer, default 1)
- `name` (e.g., "Lifetime Musician")
- `visibility` (enum: first_party, method_aligned, custom)
- `org_id` (FK → Organization, nullable, only for custom sequences)
- `instrument` (enum: piano, general_music)
- `metadata` (JSONB: description, target_age, estimated_hours, prerequisites)
- `status` (enum: draft, published, archived)
- `created_at`, `updated_at`, `published_at`
- **Related to**: SequenceGroup (1:N), Organization (N:1 optional)
- See [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)

**SequenceGroup**
- Logical chapter or unit within a sequence
- `group_id` (UUID, primary key)
- `sequence_id` (FK → Sequence)
- `sequence_version` (integer)
- `group_code` (string, e.g., "G-01", "G-02")
- `name` (e.g., "Foundations", "Primary Level 1A")
- `order_index` (integer)
- `metadata` (JSONB)
- **Related to**: Sequence (N:1), SequenceAssignment (1:N)

**SequenceAssignment**
- Assignment definition within a group (template, not student-specific)
- `assignment_def_id` (UUID, primary key)
- `group_id` (FK → SequenceGroup)
- `assignment_code` (string, e.g., "A-01", "A-02")
- `name` (e.g., "Assignment 1", "Skill Check 5")
- `order_index` (integer)
- `metadata` (JSONB)
- **Related to**: SequenceGroup (N:1), SequenceStep (1:N)

**SequenceStep**
- Individual learning element reference within an assignment template
- `step_def_id` (UUID, primary key)
- `assignment_def_id` (FK → SequenceAssignment)
- `order_index` (integer)
- `element_type` (enum: GAM, VID, AUD, TXT, RWD)
- `element_id` (polymorphic: game_id, video_id, etc.)
- `stage_id` (FK → GameStage, nullable, only for GAM type)
- `gating_rules` (JSONB: require_previous_steps, min_attempts, etc.)
- `target_rules` (JSONB: pass_threshold, time_limit, etc.)
- `optional` (boolean)
- **Related to**: SequenceAssignment (N:1), GameStage (N:1 optional)

**LearningElement** (future: non-game elements)
- Videos, audio, text, rewards
- `element_id` (string, primary key, format: `VID-INS-2345-E`)
- `element_type` (enum: VID, AUD, TXT, RWD)
- `name`, `description`
- `content_url` (Cloudflare R2 reference)
- `metadata` (JSONB)
- `status` (enum: draft, live, deprecated)
- **Note**: Currently games are primary; videos/audio/text are future enhancements

#### Learning Progress Domain

**StudentAssignment**
- Instance of an assignment assigned to a specific student
- `assignment_id` (UUID, primary key)
- `student_id` (FK → StudentProfile)
- `sequence_id` (FK → Sequence)
- `sequence_version` (integer, pinned at creation)
- `group_id` (FK → SequenceGroup)
- `assignment_def_id` (FK → SequenceAssignment, source template)
- `name`, `description`
- `status` (enum: open, in_progress, complete, archived)
- `assigned_by_user_id` (FK → User)
- `due_at` (timestamp, nullable)
- `completed_at` (timestamp, nullable)
- `policy_snapshot` (JSONB: frozen rules at creation)
- `created_at`, `updated_at`
- **Related to**: StudentProfile (N:1), Sequence (N:1), SequenceAssignment (N:1), AssignmentStep (1:N)
- See [Assignment Model](../07-content-architecture/assignment-model.md)

**AssignmentStep**
- Individual step within a student's assignment
- `step_id` (UUID, primary key)
- `assignment_id` (FK → StudentAssignment)
- `step_def_id` (FK → SequenceStep, source template)
- `order_index` (integer)
- `element_type` (enum: GAM, VID, AUD, TXT, RWD)
- `element_id` (polymorphic)
- `stage_id` (FK → GameStage, nullable)
- `state` (enum: locked, available, in_progress, complete, skipped)
- `target_score`, `pass_threshold` (from policy_snapshot)
- `best_score` (integer, nullable)
- `attempts_count` (integer, default 0)
- `time_spent_seconds` (integer, default 0)
- `first_attempt_at`, `completed_at`
- **Related to**: StudentAssignment (N:1), GameStage (N:1 optional), StudentAttempt (1:N)

**StudentAttempt**
- Individual game play attempt
- `attempt_id` (UUID, primary key)
- `student_id` (FK → StudentProfile)
- `step_id` (FK → AssignmentStep, nullable if free play)
- `game_id` (FK → Game)
- `stage_id` (FK → GameStage)
- `score` (integer)
- `max_possible_score` (integer)
- `passed` (boolean)
- `duration_ms` (integer)
- `play_type` (enum: assigned, free_play, review, challenge)
- `session_data` (JSONB: detailed game session metrics)
- `created_at` (timestamp of play)
- **Related to**: StudentProfile (N:1), GameStage (N:1), AssignmentStep (N:1 optional)
- See [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md)

**StudentMastery**
- Tracks concept mastery and review scheduling
- `mastery_id` (UUID, primary key)
- `student_id` (FK → StudentProfile)
- `game_id` (FK → Game)
- `concept_tag` (string, indexed)
- `mastery_level` (integer 0-100)
- `last_played_at`
- `next_review_at` (timestamp, for spaced repetition)
- `total_attempts`, `successful_attempts`
- **Related to**: StudentProfile (N:1), Game (N:1)
- See [Retention Review Logic](../10-assignments-engine/retention-review-logic.md)

#### Gamification Domain

**Badge**
- Achievement badges
- `badge_id` (UUID, primary key)
- `badge_code` (string, unique)
- `name`, `description`
- `icon_url` (Cloudflare R2 reference)
- `criteria` (JSONB: rules for earning)
- `rarity` (enum: common, uncommon, rare, epic, legendary)
- `points_value` (integer)
- **Related to**: StudentBadge (1:N)
- See [Badges System](../12-gamification/badges-system.md)

**StudentBadge**
- Earned badges for students
- `student_badge_id` (UUID, primary key)
- `student_id` (FK → StudentProfile)
- `badge_id` (FK → Badge)
- `earned_at` (timestamp)
- `context` (JSONB: which game/assignment triggered)
- **Related to**: StudentProfile (N:1), Badge (N:1)

**Challenge**
- Time-based competitive events
- `challenge_id` (UUID, primary key)
- `game_id` (FK → Game)
- `stage_id` (FK → GameStage)
- `name`, `description`
- `start_at`, `end_at`
- `status` (enum: upcoming, active, ended)
- **Related to**: Game (N:1), GameStage (N:1), ChallengeLeaderboard (1:N)

**ChallengeLeaderboard**
- Leaderboard entries for challenges
- `entry_id` (UUID, primary key)
- `challenge_id` (FK → Challenge)
- `student_id` (FK → StudentProfile)
- `score` (integer)
- `rank` (integer)
- `achieved_at` (timestamp)
- **Related to**: Challenge (N:1), StudentProfile (N:1)
- See [Challenge Boards](../12-gamification/challenge-boards.md)

**PointsLog**
- Tracks points earned for gamification
- `log_id` (UUID, primary key)
- `student_id` (FK → StudentProfile)
- `points` (integer)
- `reason` (enum: game_complete, badge_earned, streak, challenge)
- `reference_id` (UUID, polymorphic: attempt_id, badge_id, etc.)
- `earned_at` (timestamp)
- **Related to**: StudentProfile (N:1)
- See [Points and Streaks](../12-gamification/points-and-streaks.md)

#### Communication Domain

**Message**
- Teacher-student messages
- `message_id` (UUID, primary key)
- `from_user_id` (FK → User)
- `to_user_id` (FK → User)
- `subject`, `body` (text)
- `read_at` (timestamp, nullable)
- `created_at`
- **Related to**: User (N:1 sender), User (N:1 recipient)
- See [Teacher Messaging](../04-teacher-experience/teacher-messaging.md)

**Notification**
- In-app notifications
- `notification_id` (UUID, primary key)
- `user_id` (FK → User)
- `type` (enum: assignment_due, badge_earned, message_received, etc.)
- `title`, `body`
- `action_url` (nullable)
- `read_at` (timestamp, nullable)
- `created_at`
- **Related to**: User (N:1)
- See [In-App Notifications](../13-messaging-and-notifications/in-app-notifications.md)

**SystemAnnouncement**
- Platform-wide announcements
- `announcement_id` (UUID, primary key)
- `title`, `content` (markdown)
- `target_roles` (array: student, teacher, admin)
- `priority` (enum: info, warning, critical)
- `publish_at`, `expire_at`
- `status` (enum: draft, published, expired)
- **Related to**: User (M:N via AnnouncementViews)
- See [System Announcements](../13-messaging-and-notifications/system-announcements.md)

### Entity Relationship Diagram

**High-Level Domain Relationships**:

```
┌─────────────────────┐
│  Identity & Access  │
│  ┌──────┐           │
│  │ User │           │
│  └──┬───┘           │
└─────┼───────────────┘
      │
      │ owns/belongs to
      ▼
┌─────────────────────┐
│ Subscription/Org    │
│  ┌──────────────┐   │
│  │ Organization │   │
│  └──────┬───────┘   │
│         │           │
│  ┌──────▼────────┐  │
│  │ Subscription  │  │
│  └───────────────┘  │
└─────────────────────┘
      │
      │ has seats/members
      ▼
┌─────────────────────┐
│   User Profiles     │
│  ┌─────────┐        │
│  │ Student │        │
│  │ Teacher │        │
│  │  Admin  │        │
│  └─────────┘        │
└─────────────────────┘
      │
      │ assigned to/creates
      ▼
┌─────────────────────────┐
│  Learning Progress      │
│  ┌────────────────────┐ │
│  │ StudentAssignment  │ │
│  └─────────┬──────────┘ │
│            │             │
│  ┌─────────▼──────────┐ │
│  │  AssignmentStep    │ │
│  └─────────┬──────────┘ │
│            │             │
│  ┌─────────▼──────────┐ │
│  │  StudentAttempt    │ │
│  └────────────────────┘ │
└─────────────────────────┘
      │
      │ references
      ▼
┌─────────────────────┐
│      Content        │
│  ┌──────────┐       │
│  │ Sequence │       │
│  └────┬─────┘       │
│       │             │
│  ┌────▼──────┐      │
│  │   Game    │      │
│  └────┬──────┘      │
│       │             │
│  ┌────▼──────────┐  │
│  │  GameStage    │  │
│  └───────────────┘  │
└─────────────────────┘
```

**Detailed ERD (Key Relationships)**:

```
User ────────┬─── 1:1 ───> StudentProfile
             │
             ├─── 1:1 ───> TeacherProfile
             │
             ├─── 1:1 ───> AdminProfile
             │
             ├─── 1:N ───> Session
             │
             └─── 1:N ───> AuditLog

Organization ─── 1:N ───> Subscription
             │
             ├─── 1:N ───> StudentProfile
             │
             ├─── 1:N ───> TeacherProfile
             │
             ├─── M:N ───> PromoCode (via PromoCodeUsage)
             │
             └─── M:N ───> SponsorCode (sponsor & sponsored)

Subscription ─── 1:N ───> Seat
             │
             ├─── 1:N ───> Payment
             │
             └─── 1:N ───> Invoice

TeacherProfile ── 1:N ───> StudentProfile (assigned students)
               │
               ├─── 1:N ───> Class
               │
               └─── 1:N ───> StudentAssignment (creator)

Class ────────┬─── M:N ───> StudentProfile (via ClassEnrollment)
             │
             └─── N:1 ───> TeacherProfile

Sequence ─────┬─── 1:N ───> SequenceGroup
             │
             └─── M:N ───> Game (via SequenceStep → GameStage)

SequenceGroup ── 1:N ───> SequenceAssignment

SequenceAssignment ── 1:N ───> SequenceStep

SequenceStep ─────┬─── N:1 ───> GameStage (if element_type = GAM)
                 │
                 └─── N:1 ───> LearningElement (if other types)

Game ─────────┬─── 1:N ───> GameStage
             │
             ├─── M:N ───> Sequence (via SequenceStep)
             │
             └─── 1:N ───> StudentAttempt

GameStage ────┬─── 1:N ───> StudentAttempt
             │
             ├─── 1:N ───> AssignmentStep
             │
             └─── N:1 ───> Game

StudentProfile ─── 1:N ───> StudentAssignment
               │
               ├─── 1:N ───> StudentAttempt
               │
               ├─── 1:N ───> StudentBadge
               │
               ├─── 1:N ───> StudentMastery
               │
               ├─── M:N ───> Class (via ClassEnrollment)
               │
               └─── N:1 ───> TeacherProfile (assigned teacher)

StudentAssignment ─── 1:N ───> AssignmentStep

AssignmentStep ────── 1:N ───> StudentAttempt

StudentAttempt ───────┬─── N:1 ───> StudentProfile
                     │
                     ├─── N:1 ───> GameStage
                     │
                     └─── N:1 ───> AssignmentStep (nullable, only if assigned)

Badge ───────────┬─── 1:N ───> StudentBadge
                 │
                 └─── M:N ───> StudentProfile (via StudentBadge)

Challenge ────────┬─── N:1 ───> Game
                 │
                 ├─── N:1 ───> GameStage
                 │
                 └─── 1:N ───> ChallengeLeaderboard

ChallengeLeaderboard ─── N:1 ───> StudentProfile

Message ──────────┬─── N:1 ───> User (sender)
                 │
                 └─── N:1 ───> User (recipient)

Notification ─────── N:1 ───> User

SystemAnnouncement ── M:N ───> User (via AnnouncementViews)
```

### Identifier Conventions

**Primary Key Patterns**:
- **UUID (v4)**: Default for most entities (assignments, attempts, users)
  - Example: `a3bb189e-8bf9-3888-9912-ace4e6543002`
  - Use case: Internal IDs, high write-volume entities, distributed generation
  
- **Prefixed Sequential Strings**: For user-facing content entities
  - Games: `G-01542` (legacy numeric games were GAM-0010, GAM-0020)
  - Sequences: `SEQ-LIFE`, `SEQ-ALFP` (code-based)
  - Stages: `G-01542-quiz` (game_id + stage_type)
  - Elements: `VID-INS-2345-E` (type-subtype-number-language)

**Foreign Key Naming**:
- Convention: `{entity}_id` (e.g., `user_id`, `game_id`, `subscription_id`)
- Composite FKs: Use multiple columns (e.g., `sequence_id` + `sequence_version`)

**Indexes**:
- Primary keys: Automatic clustered index
- Foreign keys: Non-clustered index on all FK columns
- Usernames/emails: Unique index
- Search fields: GIN index on `concept_tags`, `category` (PostgreSQL full-text)
- Temporal queries: Index on `created_at`, `updated_at`, `due_at`, `next_review_at`

### Versioning Strategy

**Content Versioning**:
- **Games**: `game_version` increments on breaking changes; old versions archived but queryable
- **Sequences**: `sequence_version` increments; assignments pin version at creation time
- **Pinning**: StudentAssignment stores `sequence_id` + `sequence_version` to ensure grading stability

**Historical Data**:
- **Soft Deletes**: Use `status` field (active → archived) rather than hard deletes
- **Audit Trail**: AuditLog table captures significant mutations
- **Retention**: 13 months after subscription cancellation (FERPA compliance)

### Legacy Migration Mapping (MLC 4.0 → 5.0)

**MLC 4.0 Table → MLC 5.0 Entity**:
- `tbl_members` → `User`, `Organization`, `Subscription`
- `tbl_students` → `User` (role=student), `StudentProfile`
- `tbl_teachers` → `User` (role=teacher), `TeacherProfile`
- `tbl_games` → `Game`, `GameStage`
- `tbl_sequences` → `Sequence`, `SequenceGroup`, `SequenceAssignment`, `SequenceStep`
- `tbl_assignments` → `StudentAssignment`, `AssignmentStep`
- `tbl_scores` → `StudentAttempt`
- `tbl_promo_codes` → `PromoCode`
- `tbl_sponsor_codes` → `SponsorCode`

**Key Changes**:
- Normalized member hierarchy (subscriber/teacher/student split)
- Explicit versioning for content
- Separated assignment definitions (templates) from student instances
- Stage-level tracking (not just game-level)
- Richer metadata using JSONB fields

## Behavior and rules

**State Transitions**:

**Subscription Status Flow**:
```
[new] → trial → active → paused → cancelled
                    ↓
                 suspended (non-payment)
                    ↓
                 active (resumed)
```

**Assignment Status Flow**:
```
open → in_progress → complete
  ↓
archived (teacher action)
```

**Step State Flow**:
```
locked → available → in_progress → complete
                           ↓
                        skipped (optional steps)
```

**Game Status Flow**:
```
draft → live → deprecated (replaced_by_game_id set)
```

**Gating Rules**:
- **Sequential Gating**: Step N locked until Step N-1 is complete
- **Prerequisite Gating**: Step requires specific prior steps (non-sequential)
- **Attempt Gating**: Minimum attempts required before unlock
- **Score Gating**: Minimum score threshold on prerequisite step

**Validation Rules**:
- Usernames: Unique within organization, 3-30 characters
- Seat Allocation: Cannot exceed subscription seat_count
- Assignment Due Dates: Must be future date at creation
- Game Stages: Quiz stage required; Challenge optional
- Sequence Steps: order_index must be unique within assignment

**Cascade Delete Policies**:
- User deleted → soft delete (archive status), retain StudentAttempt for analytics
- Organization cancelled → subscriptions archived, users suspended, data retained 13 months
- Game deprecated → StudentAssignment still references via pinned version
- Class deleted → ClassEnrollment archived, StudentProfile intact

## UX requirements (if applicable)

**Data Display Patterns**:

**Student Dashboard**:
- Query: Recent assignments with step completion percentage
- Join: StudentAssignment ← AssignmentStep ← StudentAttempt
- Performance: Cache aggregated completion % on StudentAssignment

**Teacher Reports**:
- Query: Student progress across all assignments in a class
- Join: Class → ClassEnrollment → StudentProfile → StudentAssignment → AssignmentStep
- Performance: Materialized view for class-level rollups

**Admin Billing**:
- Query: Subscription details with seat utilization
- Join: Organization → Subscription → Seat (count assigned vs. total)
- Performance: Index on Seat.status, cache utilization %

**Search & Discovery**:
- Query: Find games by concept tags, level, difficulty
- Index: GIN index on Game.concept_tags (PostgreSQL array search)
- Filter: Teacher Quick Find uses concept_tags + level + difficulty

**Assignment Player**:
- Query: Fetch assignment with all steps and game metadata
- Join: StudentAssignment → AssignmentStep → GameStage → Game
- Performance: Single query with nested JSON aggregation (PostgreSQL json_agg)

**Data Privacy**:
- **PII Fields**: User.email, StudentProfile.display_name (encrypted at rest)
- **FERPA Compliance**: No student data shared across organizations
- **Audit**: All PII access logged in AuditLog
- **Retention**: 13-month retention after subscription end, then hard delete

## Telemetry

**Database Events** (via triggers or application layer):

**User Events**:
- `user.created`, `user.login`, `user.password_changed`
- Properties: user_id, role, org_id, ip_address, timestamp

**Subscription Events**:
- `subscription.created`, `subscription.upgraded`, `subscription.paused`, `subscription.cancelled`
- Properties: subscription_id, org_id, plan_type, seat_count, amount_cents

**Learning Events**:
- `assignment.created`, `assignment.started`, `assignment.completed`
- `step.unlocked`, `step.started`, `step.completed`
- `attempt.started`, `attempt.completed`, `attempt.scored`
- Properties: student_id, assignment_id, step_id, game_id, stage_id, score, duration_ms

**Gamification Events**:
- `badge.earned`, `challenge.joined`, `leaderboard.updated`
- Properties: student_id, badge_id, challenge_id, rank, points

**System Events**:
- `game.published`, `sequence.published`, `content.updated`
- Properties: game_id, sequence_id, version, admin_user_id

See [Event Model](../15-analytics-and-reporting/event-model.md) for complete event taxonomy.

**Query Performance Monitoring**:
- Track slow queries (>500ms) via PostgreSQL slow query log
- Monitor N+1 queries in application layer (ORM query analysis)
- Alert on connection pool exhaustion

## Permissions

**Data Access by Role**:

**System Admin**:
- Read/Write: All entities
- Special: AuditLog read-only, no delete on historical data

**Subscriber-Admin**:
- Read/Write: Organization, Subscription, all users in org, Seat allocation
- Read-only: StudentAssignment, StudentAttempt (their org)
- No access: Other organizations' data

**Teacher-Admin** (Ensemble only):
- Read/Write: TeacherProfile, StudentProfile, Class (their org scope)
- Read-only: Organization, Subscription (their org), StudentAssignment, StudentAttempt
- No access: Billing details, Payment, Invoice

**Teacher**:
- Read/Write: StudentProfile (assigned to them), Class (theirs), StudentAssignment (theirs)
- Read-only: StudentAttempt (their students), Game, Sequence
- No access: Organization-level data, other teachers' students

**Student**:
- Read/Write: StudentProfile (own), StudentAssignment (assigned to them), StudentAttempt (own)
- Read-only: Game, GameStage, Sequence (public content)
- No access: Other students' data, billing, admin functions

**Permission Enforcement**:
- **Row-Level Security (RLS)**: PostgreSQL policies filter queries by org_id, user_id
- **API Middleware**: Role-based checks before controller execution
- **Frontend Guards**: UI hides/disables features based on role

See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for complete permission grid.

## Supporting Documents Referenced

This data model ERD specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game and content data structure
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence data model
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User and role data structure
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription and seat data model
- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Billing data model

## Dependencies

**Internal Dependencies**:
- **Feature Specs**: Detailed entity definitions in domain docs
  - [Game Model](../08-games-registry-and-authoring/game-model.md)
  - [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)
  - [Assignment Model](../07-content-architecture/assignment-model.md)
  - [Group Model](../07-content-architecture/group-model.md)
  - [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md)
  - [Event Model](../15-analytics-and-reporting/event-model.md)
- **Architecture Specs**:
  - [System Overview](system-overview.md)
  - [Service Boundaries](service-boundaries.md)
  - [REST Endpoints](rest-endpoints.md)

**External Dependencies**:
- **Database**: Neon PostgreSQL (serverless, connection pooling)
- **ORM**: Prisma or Drizzle ORM for type-safe queries
- **Storage**: Cloudflare R2 for game assets, avatars, invoices (references stored as URLs)
- **CDN**: Cloudflare CDN for asset delivery
- **Payments**: Braintree API (transaction IDs stored in Payment table)
- **Email**: Email provider (message content not stored, only notification records)

**Data Import/Export**:
- CSV schemas for bulk operations (see [CSV Specifications](../20-imports-and-automation/csv-specifications.md))
- API endpoints for external LMS integration (see [LTI Spec](../17-integrations/lms-ltispec-future.md))

## Open questions

1. **Sharding Strategy**: At what organization size do we need to shard the database? Current plan assumes single-region PostgreSQL. Future: multi-region replication for global customers?

2. **Event Sourcing**: Should critical state transitions (subscription changes, assignment completion) use event sourcing for audit trail? Current plan: append-only AuditLog table.

3. **Real-Time Sync**: Do we need WebSocket subscriptions for real-time leaderboard updates, or is polling sufficient? Current plan: polling every 30s for challenges.

4. **GDPR Right to Erasure**: How do we handle hard delete requests while maintaining referential integrity for analytics? Current plan: anonymize PII fields, retain aggregated stats.

5. **Multi-Tenancy**: Should we use schema-per-org for data isolation, or rely on RLS? Current plan: RLS with org_id filtering (simpler, scales to 10K+ orgs).

6. **Game Asset Versioning**: Should game assets (images, audio) be versioned separately from Game entity? Current plan: single assets_url per game_version, stored in R2 with versioned paths.

7. **Materialized Views**: Which reports benefit from materialized views vs. real-time aggregation? Candidates: class progress rollups, leaderboard rankings, seat utilization.

8. **Soft Delete Cascade**: When a Teacher is archived, should their StudentAssignments be reassigned or archived? Current plan: archive assignments, require admin to reassign students to new teacher first.
