# Teacher Challenges

## Purpose
Define the teacher-facing experience for creating, managing, and reviewing time-bound challenges that motivate students via friendly competition. Distinct from general gamification and global leaderboards, this page focuses on class- and group-scoped challenges teachers configure and track.

## Scope
Included:
- Teacher flows to create/edit/archive a challenge (class-, group-, or individual-student scope)
- Challenge configuration: game(s)/stage(s), scoring metric, duration, eligibility, tiebreakers
- Visibility: class-only leaderboards vs. global boards (link out), anonymization options
- Monitoring: live leaderboard, participation, basic fairness/anti-cheat guardrails

Excluded (covered elsewhere):
- Platform-wide challenge mechanics and global boards (see `../12-gamification/challenge-boards.md`)
- Core game stages, scoring semantics, and retention logic (see `../10-assignments-engine/assignment-progress-tracking.md` and game specs)
- Student play UX (see `../03-student-experience/assignment-play.md`)

## Data model (if applicable)
Entities (conceptual):
- Challenge
  - `challengeId` (UUID)
  - `title`, `description`
  - `scope` (class | group | individual)
  - `ownerTeacherId`, `organizationId`
  - `eligibility` (students/groups included)
  - `gameIds` and optional `stageFilter` (e.g., CHALLENGE, QUIZ)
  - `scoringMetric` (highest_score | fastest_time | most_correct | total_points_window)
  - `startAt`, `endAt`, `timezone`
  - `visibility` (class_only | org | global_link)
  - `status` (draft | active | completed | archived)

- ChallengeLeaderboardEntry
  - `challengeId`, `studentId`
  - `rank`, `score`, `tiebreaker` (first_to_reach | earliest_submission | least_attempts)
  - `lastUpdatedAt`

## Behavior and rules
State machine and rules:
- Draft → Active (requires: at least one eligible student, valid date window, selected metric, at least one game)
- Active → Completed (when `now >= endAt`) or manual close by teacher
- Completed → Archived (teacher action)

Validations:
- Time window must be ≥ 15 minutes, ≤ 30 days by default (configurable per policy)
- Scope must include at least one student; students must have access to the selected game(s)
- If metric is highest_score with CHALLENGE stage, ensure selected games support unbounded/high-score mode (legacy “Challenge” stage exists in several titles per original context)

Leaderboards:
- Update on qualifying play events; show top N (default 20) with “View all”
- Tie rules apply consistently to displayed ranks and awards
- Option to anonymize names (initials or displayId) when `visibility != class_only`

## UX requirements (if applicable)
Key screens (teacher):
- Create Challenge: form with scope selector, game/stage picker, metric, window, visibility, tie-breaker
- Challenge Overview: live leaderboard, participation count, filter by group, export results
- Edit Challenge: permitted while Draft; limited edits while Active (endAt extend/shorten, visibility)

Accessibility/Copy:
- Use consistent terminology with Teacher Dashboard and Assignment Builder
- Respect `a11y` guidelines (see `../01-ux-design-system/a11y-standards.md`) and copy tone (see `../01-ux-design-system/screen-text-and-copy.md`)

## Telemetry
Instrument per [event model](../15-analytics-and-reporting/event-model.md):
- `challenge.created` { challengeId, scope, metric, startAt, endAt, gameIds }
- `challenge.published` { challengeId, eligibleCount }
- `challenge.viewed` { challengeId, teacherId }
- `challenge.leaderboard.viewed` { challengeId, page, pageSize }
- `challenge.updated` { challengeId, changed_fields }
- `challenge.closed` { challengeId, reason }
Student-side events (emitted elsewhere, consumed here): `game_session.completed`, `score.submitted` with challenge attribution

## Permissions
Access:
- Teacher: create/manage challenges for owned classes/groups
- Teacher-Admin: org-wide within subscriber account
- Subscriber Admin: read-only unless explicitly granted (see `../06-system-admin/user-permissions.md`)
- Impersonation: banner and disabled destructive actions (see `../02-roles-and-permissions/impersonation-and-support-access.md`)

## Supporting Documents Referenced

This teacher challenges specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Teacher role challenge creation and management capabilities
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Challenge game specifications and competitive gameplay requirements
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Challenge interface content and messaging

## Dependencies
- Game registry for stage availability and scoring semantics (see `../08-games-registry-and-authoring/game-model.md`)
- Assignment/Play events pipeline for scores (see `../10-assignments-engine/assignment-progress-tracking.md`)
- Challenge Boards for platform-wide patterns and global views (see `../12-gamification/challenge-boards.md`)
- Reports for downstream analysis (see `../15-analytics-and-reporting/teacher-reports-kpis.md`)

## Open questions
- Do we allow multi-game composite challenges or one-game-per-challenge only in v1?
- Can teachers opt-in to org/global visibility or must challenges remain class-only in v1?
- What anti-cheat signals are required for score acceptance (e.g., max rate, device checks)?
- Export format defaults (CSV fields) and retention duration for completed challenges.

