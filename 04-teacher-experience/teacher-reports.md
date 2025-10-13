# Teacher Reports

## Purpose
Provide teachers with actionable visibility into student progress and engagement to inform instruction, celebrate achievements, and intervene early. This page defines the teacher-facing reporting surfaces and their intended outcomes, distinct from admin analytics.

## Scope
Included:
- Progress and mastery views at student, assignment, and game-set levels
- Class/group rollups and cohort comparisons
- Filters (date range, sequence/course, game category, status) and exports
- Achievement artifacts (certificates, badges summaries) for teacher use

Excluded:
- Admin/school-wide analytics (see `15-analytics-and-reporting/admin-reports-kpis.md`)
- System operational dashboards (see `19-quality-and-operations/observability.md`)

## Data model (if applicable)
- Reporting draws from assignment attempts, game session telemetry, and sequence enrollments. Canonical KPI definitions live in `15-analytics-and-reporting/teacher-reports-kpis.md`. This page references, not redefines, those metrics.
- Primary entities referenced:
  - StudentProgressSnapshot (derived): studentId, sequenceId, coverage%, mastery%, lastActiveAt
  - AssignmentOutcome: assignmentId, studentId, status, score, attempts, completedAt
  - GameSessionAggregate: studentId, gameId, totalSessions, avgScore, bestScore, lastPlayedAt

## Behavior and rules
- Default time window: last 30 days; teachers can select custom ranges
- Late/overdue logic mirrors `10-assignments-engine/assignment-progress-tracking.md`
- Retention review flags follow `10-assignments-engine/retention-review-logic.md`
- Visibility:
  - Teachers see their own students and groups only
  - Teacher-Admin additionally sees all teacher cohorts within the subscriber
- Export (CSV): respects current filters and column selections
- Performance safeguards: pagination and server-side aggregation when result sets exceed thresholds

## UX requirements (if applicable)
- Reports hub tabs: Overview, Students, Assignments, Games, Exports
- Overview: headline KPIs (active students, on-track %, assignments due, avg weekly playtime) with trends; link each tile to its detailed tab
- Students: table with name, last active, on-track status, mastery%, sequences; row drill-in opens student detail with sparkline and recent attempts
- Assignments: table with status chips (Not Started, In Progress, Completed, Overdue), completion%, median score; bulk actions: message students, export
- Games: heatmap by game category vs mastery; link to `08-games-registry-and-authoring/game-model.md` for definitions
- Filters: date range, group/class, sequence/course, category, status
- Accessibility: all KPIs have text equivalents; tables are keyboard navigable and support column resize and focus outlines

## Telemetry
- Use canonical events from `15-analytics-and-reporting/event-model.md`:
  - ReportViewed: { reportType, filters, resultCount }
  - ReportExportRequested: { reportType, format, filters }
  - ReportDrilldownOpened: { reportType, entityType, entityId }
- Sampling/PII: no student PII in client logs; IDs only; server correlates

## Permissions
- See `02-roles-and-permissions/roles-matrix.md` and `02-roles-and-permissions/permissions-table.md`.
- Teacher: read reports for assigned students/groups
- Teacher-Admin: read reports across the subscriber tenant
- Subscriber-Admin: not in scope here; see admin analytics

## Supporting Documents Referenced

This teacher reports specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Teacher and Teacher-Admin reporting capabilities and data access
- [Dashboard Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Dashboard%20Chart.xlsx%20-%20Sheet1.csv) — Dashboard and reporting interface specifications
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Reports interface content and messaging

## Dependencies
- KPIs and metric definitions: `15-analytics-and-reporting/teacher-reports-kpis.md`
- Assignment status logic: `10-assignments-engine/assignment-progress-tracking.md`
- Retention flags: `10-assignments-engine/retention-review-logic.md`
- Game taxonomy: `07-content-architecture/game-taxonomy.md`
- Event collection: `15-analytics-and-reporting/event-model.md`

## Open questions
- Export column presets and saved views—do we store per teacher?
- Cohort comparison baseline (class vs grade vs subscriber)—which are in teacher scope?
- Performance budgets for largest classes—acceptable wait thresholds?

