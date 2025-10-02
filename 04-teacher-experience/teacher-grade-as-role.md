# Teacher Grade as Role

## Purpose
Define the Teacher “Grade-as” capability: when and how a Teacher performs grading and administrative actions on student work without being treated as a Student. This avoids polluting student analytics with teacher play sessions while enabling review, override, and completion controls on assignments.

## Scope
Included
- Enter grades and completion statuses for assigned work
- Manually mark items complete, adjust scores, and add feedback
- Preview any game/learning element and assignment flow as a Teacher (no scores recorded)
- Bulk actions for a class: complete, reset, or reassign items

Excluded
- Financial/admin subscription controls (see `../14-payments-and-subscriptions/seat-management.md`)
- Creating new Teachers (see `../02-roles-and-permissions/roles-matrix.md`)

## Data model (if applicable)
Entities
- GradeAction: id, studentId, assignmentItemId, teacherId, actionType [grade|complete|reset|overrideScore|comment], value, comment, createdAt
- GradeAudit: id, gradeActionId, previousValue, newValue, actorId, actorRole, createdAt

Relationships
- A Teacher performs many GradeActions
- Each GradeAction relates to one Assignment Item (see `../10-assignments-engine/assignment-progress-tracking.md`)

## Behavior and rules
Preview vs Grade-as
- Preview: Teacher launches a game or assignment item for inspection; no telemetry contributes to leaderboards or student history. Use when checking content.
- Grade-as: Teacher records outcome against a selected student and item without launching the game; used for rubric-style grading, manual completion, or overrides.

Rules
- Teacher play sessions never write to student score history or global leaderboards (see originals: Teachers “no scores recorded”).
- Only Teacher and Teacher-Admin may perform GradeAction; Subscriber-Admin may view but not grade by default. See `../02-roles-and-permissions/permissions-table.md`.
- Overrides must retain original student score in audit trail; latest effective value is surfaced in reports.
- Manual completion respects retention/review gates (see `../10-assignments-engine/retention-review-logic.md`).
- Bulk actions require explicit confirmation and are limited to the Teacher’s roster.

## UX requirements (if applicable)
Entry points
- Assignment Detail: per-student row actions [Grade, Complete, Reset]
- Item Drawer: quick preview with “Open as Teacher” and “Open as Student (impersonate)” links

States
- Disabled actions when no assignment exists for the student
- Confirmation modals for bulk operations

Accessibility
- Buttons and toggles meet `../01-ux-design-system/a11y-standards.md`
- Clear role context badges: “Teacher Preview” vs “Student (impersonated)”

## Telemetry
Reference `../15-analytics-and-reporting/event-model.md`
- grade_action_performed: {actionType, assignmentItemId, studentId, teacherId, reason}
- grade_override_created: {previousValue, newValue}
- manual_completion_marked: {assignmentItemId, method: teacher}
- preview_opened: {contentId, mode: teacher}

## Permissions
Reference `../02-roles-and-permissions/roles-matrix.md` and original permissions table
- Teacher: read/write GradeAction on own students
- Teacher-Admin: read/write for all students in subscription
- Subscriber-Admin: read-only by default; feature flag can allow write
- Student: no access

## Dependencies
Depends on
- `../10-assignments-engine/assignment-progress-tracking.md`
- `../10-assignments-engine/free-play-vs-assigned.md`
- `../15-analytics-and-reporting/event-model.md`
- `../02-roles-and-permissions/permissions-table.md`

## Open questions
1. Should Subscriber-Admin be allowed to grade with a feature flag, or restricted entirely?
2. Do grade overrides affect retention review triggers, or only reported score?
3. Should impersonation “Open as Student” be enabled here or limited to `../02-roles-and-permissions/impersonation-and-support-access.md`?
4. Bulk reset semantics for partially completed multi-stage games.

