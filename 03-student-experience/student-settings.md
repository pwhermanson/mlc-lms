# Student Settings

## Purpose
Enable students to personalize their learning experience and manage their account safely. Settings focus on accessibility, device/audio, language, and notification preferences to reduce friction in gameplay and improve focus, while keeping personally identifiable information minimal and compliant with student privacy guidance.

## Scope
Included:
- Profile basics: display name (read-only if managed by teacher), avatar selection from a safe, curated set, timezone.
- Account security: password change (when allowed), sign-out of other devices.
- Accessibility: text size, high-contrast theme, motion/animation reduction, captions for instructional audio where available.
- Audio/MIDI: sound on/off, sound effects volume, input device selection and test for MIDI keyboards.
- Language: UI language selection from supported locales.
- Notifications: in-app badges/toasts and email toggle (if email exists for student account; typically off/absent).
- Data and privacy: view score history link, export request (if permitted), clear local cache on this device.

Excluded:
- Payment/subscription controls (subscriber/admin only). See `05-admin-subscriber-experience/subscriber-dashboard.md`.
- Teacher or assignment configuration. See `03-student-experience/student-assignments.md` and `10-assignments-engine/assignment-generation.md`.

## Data model (if applicable)
Primary entity: `StudentSettings` keyed by `studentId`.
- displayName (string; usually derived from roster; optional student-editable flag)
- avatarId (string from curated catalog)
- timezone (IANA string)
- accessibility: { textScale (1.0–1.5), highContrast (bool), reduceMotion (bool), captionsPreferred (bool) }
- audio: { masterMuted (bool), sfxVolume (0–100), midiInputId (string|null) }
- locale (BCP-47 string)
- notifications: { inAppEnabled (bool), emailEnabled (bool) }
- security: { lastPasswordChangeAt (ts), sessionsCount (int) }

Related references:
- Scores are not stored in settings; link via `studentId` to analytics. See `15-analytics-and-reporting/event-model.md`.
- Avatar catalog metadata lives with assets. See `08-games-registry-and-authoring/games-assets-pipeline.md`.

## Behavior and rules
- Profile name editability: default read-only for students; teacher-admin may permit edits per policy. See `02-roles-and-permissions/roles-matrix.md`.
- Password change: allowed only when student has independent credentials; disabled for generic/shared students. See `00-ORIG-CONTEXT/USER ROLES detail.docx.txt` and `02-roles-and-permissions/session-and-auth-states.md`.
- Accessibility settings apply immediately app-wide and persist per student device session; when signed in on another device, server values hydrate on login.
- Reduce Motion disables non-essential animations in `03-student-experience/game-player-shell.md` and transition effects.
- MIDI selection: show available inputs; provide a test note; selection persisted. See `17-integrations/midi-support.md`.
- Locale changes rehydrate strings and date/time formatting. See `01-ux-design-system/localization-internationalization.md`.
- Notification email toggle: off if no student email on file; students generally lack email per privacy policy. See `23-legal-and-compliance/privacy-policy.md`.
- Session management: allow “Sign out all devices” to revoke tokens except current after confirmation. See `02-roles-and-permissions/session-and-auth-states.md`.

Validation:
- textScale clamped to supported steps; volume numeric with increments of 5; timezone and locale validated against supported lists.

## UX requirements (if applicable)
Layout
- Sections: Profile, Accessibility, Audio & MIDI, Language, Notifications, Security & Privacy.
- Mobile-first; responsive per `01-ux-design-system/responsive-breakpoints.md`.

Key states
- Disabled controls with helper text when role/policy disallows edits (e.g., password for generic students per `00-ORIG-CONTEXT/USER ROLES detail.docx.txt`).
- Live previews: text size slider previews, high-contrast toggle immediate theme swap.
- MIDI test button with success/failure feedback.

Accessibility
- All controls keyboard-navigable and labeled; respect OS/browser prefers-reduced-motion. See `01-ux-design-system/a11y-standards.md` and `16-accessibility-and-inclusion/wcag-compliance.md`.

Cross-links from Student Dashboard
- Provide entry points from `03-student-experience/student-dashboard.md` for quick access to Accessibility and Audio sections.

## Telemetry
Emit events (see `15-analytics-and-reporting/event-model.md`):
- SettingsViewed { section }
- SettingChanged { key, oldValueHash, newValueHash }
- PasswordChanged { method }
- SignOutAllDevices { sessionsRevoked }
- MidiDeviceTested { deviceId, success }

## Permissions
- Student may read and update most personal settings.
- Teacher and teacher-admin may reset certain student settings (e.g., locale, accessibility) for support; password resets via account recovery flow, not via this page. See `02-roles-and-permissions/permissions-table.md`.
- Subscriber/admin have no direct access here in student UI; administrative overrides live elsewhere. See `06-system-admin/user-permissions.md`.

## Dependencies
- Localization framework and message catalogs: `01-ux-design-system/localization-internationalization.md`.
- Theming and contrast tokens: `01-ux-design-system/a11y-standards.md`.
- Game shell for animation flags: `03-student-experience/game-player-shell.md`.
- MIDI integration: `17-integrations/midi-support.md`.
- Session/auth: `02-roles-and-permissions/session-and-auth-states.md`.
- Privacy policy constraints: `23-legal-and-compliance/privacy-policy.md` and COPPA notes in `00-ORIG-CONTEXT/Screeens Text.docx.txt`.

## Open questions
- Should avatar uploads be allowed, or only curated picks? (Current: curated only.)
- Are teacher-enforced defaults for accessibility needed (lockable per class)?
- Do we support per-device overrides vs. global settings for audio/MIDI?
- Is student email ever present, or always absent under current policy?

