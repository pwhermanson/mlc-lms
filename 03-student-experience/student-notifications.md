# Student Notifications

## Purpose
Enable students to reliably receive timely, relevant updates that help them complete assignments, stay engaged, and understand progress without creating noise or distraction. Student-facing notifications focus on learning actions and progress; implementation details for channels and templates live in `13-messaging-and-notifications/`.

## Scope
- In scope: notifications visible to students within the app (notification center, toasts), optional push/desktop delivery opt-ins, and preference controls specifically for students.
- Out of scope: teacher/admin notification details, system-wide broadcast mechanics, and channel infrastructure. See `13-messaging-and-notifications/in-app-notifications.md`, `13-messaging-and-notifications/push-and-desktop.md`, and `13-messaging-and-notifications/system-announcements.md`.

## Data model (if applicable)
Student-visible notifications reference existing events and entities rather than redefining them.
- Core references: Assignment, Game, Sequence, Message Thread, and Course context.
- Required fields (consumer view): notificationId, createdAt, type (assignment_due, assignment_assigned, streak_lost, challenge_invite, message_reply, system_announcement_ref), title, body, icon, callToAction { label, targetRoute }, readAt, archivedAt, priority.
- Source mapping: eventId from `15-analytics-and-reporting/event-model.md` is stored for traceability but not shown to the user.

## Behavior and rules
High-level student experience rules; channel-specific delivery details are in messaging specs.
- Creation triggers (examples):
  - Assignment assigned/updated/graded; due soon/overdue reminders (ties to `10-assignments-engine/assignment-progress-tracking.md`).
  - Sequence milestone achieved or placement result posted (`09-sequences-and-curriculum-tools/sequence-placements-eval.md`).
  - Challenge invites and outcomes (`12-gamification/challenge-boards.md`).
  - Message replies in student threads (`03-student-experience/student-messages.md`).
  - System announcements targeted to students (high-level reference only; details in `13-messaging-and-notifications/system-announcements.md`).
- Deduplication and frequency: collapse identical reminders within 12 hours; show latest with a counter ("+2 updates").
- Priority: blocking (rare, e.g., account action) > important (due soon, streak risk) > informational (milestones). Only blocking may interrupt focus; others queue in center.
- Quiet hours: respect student preference for Do Not Disturb; important items queue for next allowed window.
- Read/Archive: opening the target route marks as read; student may archive to hide from center.
- Throttling: max 3 new notifications shown per session; remainder appear as a count badge.

## UX requirements (if applicable)
- Entry points: header bell badge and an "Updates" section on `03-student-experience/student-dashboard.md`.
- Notification center list: newest first, grouped by category with filters: All, Assignments, Challenges, Messages, System.
- Minimal toast: non-blocking, 5s auto-dismiss, accessible with Pause/Resume for screen readers.
- States: empty (friendly copy with links to assignments), loading shimmer, error retry.
- Accessibility: comply with `01-ux-design-system/a11y-standards.md`; ensure focus is not stolen on toast; provide ARIA live region polite; keyboard dismissal and tab order preserved.

## Telemetry
Log student interactions for learning analytics without duplicating event schemas.
- Events (names illustrative; see `15-analytics-and-reporting/event-model.md`):
  - NotificationShown { notificationId, type, priority }
  - NotificationClicked { notificationId, targetRoute }
  - NotificationArchived { notificationId }
  - NotificationPreferenceChanged { setting, value }
- Use these to evaluate reminder effectiveness and tune thresholds.

## Permissions
- Visibility: students see only notifications scoped to their enrollment and identity. Enforcement follows `02-roles-and-permissions/roles-matrix.md`.
- Preferences: students can toggle categories (Assignments, Challenges, Messages, System) and opt into push/desktop per device (details in `13-messaging-and-notifications/push-and-desktop.md`).

## Dependencies
- Channel and delivery: `13-messaging-and-notifications/in-app-notifications.md`, `13-messaging-and-notifications/push-and-desktop.md`.
- Announcement governance: `13-messaging-and-notifications/system-announcements.md`.
- Communication settings and templates: `13-messaging-and-notifications/communication-framework.md`.
- Assignments logic: `10-assignments-engine/assignment-progress-tracking.md` and `10-assignments-engine/retention-review-logic.md`.

## Open questions
- What are the default quiet hours for minors vs adults? (policy input)
- Should overdue reminders escalate priority after N days? If yes, define N.
- Do we surface a weekly digest option within student UI or only via email (future)?
