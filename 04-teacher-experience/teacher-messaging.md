# Teacher Messaging

## Purpose
Enable teachers to communicate with students and guardians directly within the platform, coordinating learning activity, sharing feedback, and reducing reliance on external email threads. Messaging supports 1:1 and small group conversations tied to classes, sequences, and assignments, with delivery surfaces in-app and optional email/push per user preferences. This page focuses on the teacher-facing experience and rules; implementation primitives are referenced from canonical docs rather than duplicated here.

## Scope
Included
- Compose and send messages from teachers to: individual students, multiple students, entire class/group, and guardians (where provided)
- Threaded conversations with read receipts and basic rich text (bold, italic, links)
- Attachments: links to games, assignments, certificates, or external URLs
- Notification fan-out to in-app notifications and optional email based on user preferences
- Filtering and search within teacher inbox (by student, class, unread)

Excluded
- Student-to-student direct messaging (not supported)
- Cross-organization messaging
- Advanced templating beyond what exists in email templates (link below)

## Data model (if applicable)
Reference: `../15-analytics-and-reporting/event-model.md` (events), `../13-messaging-and-notifications/in-app-notifications.md` (delivery), `../13-messaging-and-notifications/email-templates.md` (templates)

Core entities (conceptual)
- MessageThread: `threadId`, `subject`, `createdAt`, `lastMessageAt`, `ownerRole`
- Message: `messageId`, `threadId`, `authorId`, `authorRole`, `body`, `renderedBody`, `attachments[]`, `createdAt`
- Participant: `userId`, `role`, `threadId`, `notificationPrefsSnapshot`
- RecipientGroup (virtual): class/group identifier to expand into participants at send-time

Relationships
- MessageThread has many Messages
- MessageThread has many Participants (teacher required; students/guardians optional)
- Attachments reference assignment IDs, sequence IDs, or URLs (do not inline content)

## Behavior and rules
Composition
- Author must be `Teacher` or `Teacher-Admin` for the subscription. See `../02-roles-and-permissions/permissions-table.md`.
- Allowed recipients: students the teacher owns or is assigned to; guardians linked to those students; whole class/group via expansion.
- Validation: non-empty body OR at least one attachment; max 5 attachments; total size limits delegated to email subsystem when applicable.

Threading
- First send creates `MessageThread`. Subsequent replies append to the same thread. Subject is editable only on first message.
- Read receipts tracked per participant. Teacher sees per-recipient read states.

Notifications
- On send, enqueue in-app notification per participant. Email fan-out occurs only if the participant has email enabled (see `../13-messaging-and-notifications/communication-framework.md`).
- Students never receive email to personal addresses unless provided by guardian and allowed by policy; prefer guardian email when present. See `../23-legal-and-compliance/coppa-ferpa-notes.md`.

Privacy and safety
- Teachers cannot message students outside their roster. Teachers cannot see student personal emails if policy hides them.
- Messages are stored with audit visibility for `Subscriber-Admin` and `Teacher-Admin` per policy; student inboxes show only teacher-directed messages.

Rate limits and abuse controls
- Soft limit: 200 recipients per send. Larger sends require using announcements tooling (see `../13-messaging-and-notifications/system-announcements.md`).
- Spam protections and throttles follow `../18-architecture/rate-limiting-and-abuse.md`.

## UX requirements (if applicable)
Teacher Inbox
- Left panel: folders (All, Unread, Starred, Classes), search box
- Main list: thread rows with subject, first line, unread badge, last activity time
- Detail view: thread with messages, composer at bottom, recipients pill list

Composer
- Rich text: bold, italic, link insertion; attachment picker (assignment, sequence, URL)
- Recipient selector: individuals and groups with inline chips; keyboard accessible
- Send, Save draft, and Preview (especially for email rendering)

Accessibility
- All controls reachable via keyboard; announce send status to screen readers
- Color contrast meets `../01-ux-design-system/a11y-standards.md`

## Telemetry
Reference: `../15-analytics-and-reporting/event-model.md`
- teacher_message_composed: { has_attachments, recipient_count }
- teacher_message_sent: { thread_id, message_id, recipient_count, via_email, via_inapp }
- teacher_message_read: { thread_id, message_id, reader_role }
- teacher_message_reply: { thread_id, message_id }
- teacher_message_delivery_failed: { medium, reason }

## Permissions
Reference: `../02-roles-and-permissions/roles-matrix.md`, `../02-roles-and-permissions/permissions-table.md`
- Teacher, Teacher-Admin: compose, send, read, and delete their own messages; view read receipts
- Subscriber-Admin: read-only audit access when enabled by org policy
- Student, Guardian: read and reply within threads where they are participants; cannot create new threads to teachers unless explicitly enabled (future toggle)

## Supporting Documents Referenced

This teacher messaging specification draws from the following source documents:

- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Teacher messaging interface content and communication templates
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Teacher role messaging capabilities and communication permissions
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Messaging screen text and communication workflows

## Dependencies
- In-app delivery: `../13-messaging-and-notifications/in-app-notifications.md`
- Email delivery and templates: `../13-messaging-and-notifications/email-templates.md`, `../17-integrations/email-provider.md`
- Roles and access: `../02-roles-and-permissions/*`
- Abuse controls: `../18-architecture/rate-limiting-and-abuse.md`

## Open questions
- Do we allow guardians to initiate threads to teachers? Default proposal: off.
- Should student replies be limited to canned responses for younger cohorts? Potential A/B.
- Attachment size limits for email vs in-app link cards—finalize with provider.
- Retention policy for message content and audit logs.

