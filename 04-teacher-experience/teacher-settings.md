# Teacher Settings

## Purpose
What this spec enables and why it exists.

- Provide a single place for Teachers to manage account and classroom-related preferences without needing Subscriber-Admin access.
- Streamline setup for day one teaching: identity, public profile (Teacher Finder), notification preferences, and assignment defaults.
- Reduce support load by enabling self-serve changes for common tasks (username/password, notification toggles).


## Scope
What is included and what is excluded in this spec.

Included
- Profile basics: display name, username change, password change.
- Public directory options: Teacher Finder participation, instruments taught, website link. (from 00-ORIG-CONTEXT “USER ROLES detail” and MLC specs)
- Teaching preferences: default named sequence, default starting level, recess play override toggle for classroom use.
- Notifications: opt-in/out email summaries; per-type toggles reference canonical notification docs.
- Privacy controls related to the public profile and discoverability.

Excluded
- Billing/subscription changes (Subscriber-Admin only; see `05-admin-subscriber-experience` docs).
- Creating or managing Teachers (Teacher-Admin/Subscriber-Admin flows).
- Bulk student operations (see `05-admin-subscriber-experience/student-bulk-operations.md`).


## Data model (if applicable)
Key entities, fields, IDs, and relationships.

TeacherSettings (owned by `Teacher` user; viewable/editable by `Teacher`, `Teacher-Admin`, `Subscriber-Admin` per permissions)
- id: UUID
- teacherId: FK → User(Teacher)
- displayName: string
- username: string (change flow with uniqueness validation)
- teacherFinderParticipation: boolean
- instrumentsTaught: string[]
- websiteUrl: string
- defaultNamedSequenceCode: string (e.g., "LIFE" or other supported codes)
- defaultStartingLevel: integer (0–7 internal group levels per specs)
- recessOverrideAllowed: boolean (allows student recess play outside sequence; see constraints below)
- notificationPrefs: object
  - weeklyEmailToParents: boolean
  - assignmentDueReminders: boolean
  - teacherDigestEmail: boolean
- audit
  - updatedByUserId: FK
  - updatedAt: timestamp

Relationships
- TeacherSettings 1:1 Teacher
- References event taxonomy for changes; see `15-analytics-and-reporting/event-model.md`.


## Behavior and rules
State transitions, gating rules, and validations.

General
- Only authenticated Teachers (or elevated roles) may edit these settings for a Teacher.
- Username changes require uniqueness across all users; enforce reserved/blocked names.

Teacher Finder
- If `teacherFinderParticipation = true`, `displayName` and at least one `instrumentsTaught` must be present.
- Public profile shows city and email per program policy; abide by privacy rules from `23-legal-and-compliance/privacy-policy.md`.

Teaching preferences
- `defaultNamedSequenceCode` must be one of configured sequences (see `09-sequences-and-curriculum-tools/sequence-model.md`).
- `defaultStartingLevel` uses internal group levels (0–7) consistent with MLC specs.
- `recessOverrideAllowed` may be disabled by policy for schools; where allowed, it affects student recess only and does not bypass quiz target requirements. See context in `00-ORIG-CONTEXT` Assignment Progression.

Notifications
- All notification toggles map to canonical definitions in `13-messaging-and-notifications/in-app-notifications.md` and `13-messaging-and-notifications/email-templates.md`.
- Parent email features must comply with COPPA/FERPA; no student emails are stored (see `23-legal-and-compliance/ferpa-compliance.md`).

Validation
- `websiteUrl` must be https/http; max length 2000.
- `instrumentsTaught` entries are from a curated list with “Other” text option; max 10 entries.


## UX requirements (if applicable)
Layouts, key states, accessibility notes.

Layouts
- Section 1: Profile & Security — display name, username change, password change.
- Section 2: Public Profile — Teacher Finder toggle, instruments multi-select, website URL.
- Section 3: Teaching Preferences — default sequence selector, default starting level (0–7), recess override toggle with helper text.
- Section 4: Notifications — per-type toggles with short descriptions and links to canonical templates/specs.

States
- Success and error toasts after save; disabled Save until dirty and valid.
- Inline field errors for uniqueness (username) and URL validation.

Accessibility
- All controls operable via keyboard; use native labels; visible focus; meets `01-ux-design-system/a11y-standards.md`.

Links
- Deep links to: `09-sequences-and-curriculum-tools/sequence-model.md`, `13-messaging-and-notifications/communication-framework.md`.


## Telemetry
Events, properties, and where they fire.

Reference: `15-analytics-and-reporting/event-model.md`
- teacher_settings_viewed: { teacherId }
- teacher_settings_updated: { teacherId, changedFields[] }
- teacher_finder_toggled: { teacherId, enabled }
- teacher_notification_prefs_updated: { teacherId, prefs }


## Permissions
Who can read, write, publish, or override.

- Read: Teacher (self), Teacher-Admin, Subscriber-Admin, Content Admin.
- Write: Teacher (self). Teacher-Admin/Subscriber-Admin may edit for their Teachers when policy allows.
- Username changes by Teacher are permitted; bulk or forced changes require elevated roles.
- Recess override may be locked by Subscriber-Admin policy for schools.


## Supporting Documents Referenced

This teacher settings specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Teacher and Teacher-Admin settings access and configuration permissions
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Teacher settings interface content and messaging
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Settings screen labels, descriptions, and help text

## Dependencies
Other specs or systems this relies on.

- Roles and permissions: `02-roles-and-permissions/roles-matrix.md`, `02-roles-and-permissions/permissions-table.md`.
- Sequences and internal levels: `09-sequences-and-curriculum-tools/sequence-model.md`.
- Notifications and templates: `13-messaging-and-notifications/*`.
- Privacy/compliance: `23-legal-and-compliance/ferpa-compliance.md`, `23-legal-and-compliance/privacy-policy.md`.


## Open questions
Outstanding decisions to make before build.

- Exact default set of sequences exposed to Teachers vs. school-controlled list?
- Allowed instrument taxonomy for Teacher Finder; is free text enabled?
- School policy flags that lock `recessOverrideAllowed` and notification defaults.
- Do we surface public profile preview for Teacher Finder before enable?


