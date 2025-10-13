# Games Audit and Health

## Purpose

This specification defines the comprehensive audit and health monitoring system for games in the MLC platform, enabling Content Editors, QA teams, and System Administrators to track game quality, identify issues, monitor resolution progress, and maintain high standards across the game library.

The audit and health system ensures:
- **Proactive Issue Detection**: Automated health checks identify problems before students encounter them
- **Quality Assurance Tracking**: Systematic QA review workflows maintain library standards
- **Historical Audit Trails**: Complete change history enables forensic analysis and accountability
- **Resolution Management**: Clear workflows track issues from detection through remediation
- **Compliance Verification**: Health dashboards demonstrate quality metrics for stakeholders
- **Preventive Maintenance**: Trend analysis identifies recurring problems requiring systemic fixes

> **Legacy Context**: The original MLC system tracked game health through spreadsheet-based problem reporting, achieving an 88% playability rate. Common issues included "Blank Screen" failures, MIDI compatibility problems, timing inconsistencies, and asset loading errors. This new system automates health monitoring while preserving the rigor of manual QA review.

> **Related Documents**: For game data structures, see [Game Model](game-model.md). For registry workflows, see [Games Registry Overview](games-registry-overview.md). For CRUD operations and versioning, see [Games CRUD](games-crud.md). For broader system audit capabilities, see [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md).

## Scope

**Included**
- Automated health check systems for games and stages
- Manual QA review workflows and checklists
- Problem tracking and issue management
- Health status visualization and dashboards
- Historical audit trail maintenance
- Resolution workflow management
- Health metrics and reporting
- Browser and device compatibility tracking
- Asset integrity verification
- Performance monitoring and budgets
- Remediation priority scoring

**Excluded**
- Player runtime error logging (see [Event Model](../15-analytics-and-reporting/event-model.md))
- General system audit logs (see [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md))
- Content editing operations (see [Games CRUD](games-crud.md))
- Asset pipeline management (see [Games Assets Pipeline](games-assets-pipeline.md))
- Gameplay telemetry and analytics (see [Event Model](../15-analytics-and-reporting/event-model.md))

## Data model (if applicable)

### Key Entities

**Game Health Record**: Aggregated health status for a complete game  
**Stage Health Record**: Individual health status for each stage  
**Health Check Result**: Outcome of a specific automated check  
**QA Review**: Manual quality assurance review session  
**Problem Report**: Documented issue requiring resolution  
**Audit Log Entry**: Historical record of game changes and events  
**Health Metric**: Time-series data for trend analysis

### Game Health Record Fields

```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "health_status": "warn",
  "health_score": 75,
  "last_health_check": "2025-01-27T14:30:45Z",
  "checks_passed": 12,
  "checks_failed": 2,
  "checks_warned": 1,
  "failed_checks": [
    {
      "check_id": "asset_bundle_size",
      "severity": "warn",
      "message": "Bundle size 52MB exceeds recommended 50MB limit"
    },
    {
      "check_id": "safari_compatibility",
      "severity": "error",
      "message": "Game fails to load on Safari 17.2"
    }
  ],
  "qa_review_status": "approved",
  "qa_reviewer": "usr_67890",
  "qa_review_date": "2025-01-25T10:00:00Z",
  "qa_notes": "Approved with minor performance warning",
  "known_issues_count": 1,
  "open_problems_count": 1,
  "resolved_problems_count": 4
}
```

### Health Check Types

**Asset Health Checks**:
- `asset_bundle_reachable`: HTML5 bundle URL returns HTTP 200
- `asset_bundle_size`: Bundle size within budget (recommended <50MB, max 100MB)
- `asset_bundle_integrity`: Bundle checksum matches expected value
- `thumbnail_reachable`: Thumbnail image URL accessible
- `audio_sprite_reachable`: Audio sprite URL accessible (if applicable)
- `asset_cache_headers`: Proper CDN cache headers present

**Browser Compatibility Checks**:
- `chromium_compatibility`: Game loads and runs on Chromium 114+
- `safari_compatibility`: Game loads and runs on Safari 17+
- `firefox_compatibility`: Game loads and runs on Firefox 115+
- `ios_compatibility`: Game loads and runs on iOS 17+
- `android_compatibility`: Game loads and runs on Android 13+

**Device Requirement Checks**:
- `midi_device_detection`: MIDI devices detected when required
- `mic_device_detection`: Microphone access granted when required
- `keyboard_input_functional`: Keyboard input registers correctly
- `touch_input_functional`: Touch input works on mobile devices

**Performance Checks**:
- `load_time_acceptable`: Game loads within 30 seconds on standard connection
- `memory_usage_acceptable`: Peak memory usage < 500MB
- `frame_rate_stable`: Maintains 30+ FPS during gameplay
- `audio_latency_acceptable`: Audio response latency < 100ms

**Content Integrity Checks**:
- `required_stages_enabled`: Learn, Play, Quiz stages enabled for live games
- `stage_targets_defined`: All enabled stages have valid target scores
- `concept_tags_present`: At least 1 concept tag defined
- `metadata_complete`: Required metadata fields populated

**Pedagogical Quality Checks** (Manual QA):
- `instructions_clear`: Learn stage instructions understandable by target age
- `feedback_appropriate`: Play stage feedback constructive and encouraging
- `difficulty_accurate`: Stated difficulty matches actual gameplay challenge
- `content_accuracy`: Musical content is pedagogically correct
- `accessibility_verified`: Accessibility features function as claimed

### Problem Report Fields

```json
{
  "problem_id": "prob_12345",
  "game_id": "G-01542",
  "stage_id": "G-01542-quiz",
  "problem_type": "blank_screen",
  "severity": "critical",
  "status": "in_progress",
  "title": "Quiz stage shows blank screen on Safari 17.2",
  "description": "Students report blank screen when loading Quiz stage on Safari desktop. Issue does not occur on Chrome or Firefox.",
  "reported_by": "usr_teacher_456",
  "reported_date": "2025-01-20T14:30:00Z",
  "assigned_to": "usr_content_editor_789",
  "assigned_date": "2025-01-21T09:00:00Z",
  "priority": "high",
  "impact_students_count": 45,
  "impact_assignments_count": 12,
  "resolution_target_date": "2025-01-28T00:00:00Z",
  "resolution_status": "in_progress",
  "resolution_notes": "Identified missing Safari-specific WebGL fallback. Fix in development.",
  "resolved_date": null,
  "resolved_by": null,
  "verification_status": "pending",
  "related_problems": ["prob_12340", "prob_12341"],
  "tags": ["safari", "webgl", "stage_quiz", "asset_loading"]
}
```

### Health Status Values

- `pass`: All health checks passed, no known issues, QA approved
- `warn`: Minor issues present but game is playable, may impact some users
- `fail`: Critical issues prevent game from being playable for significant user base
- `unknown`: Health checks not yet run or incomplete
- `pending_review`: Awaiting manual QA review
- `deprecated`: Game deprecated, health monitoring disabled

### Problem Severity Levels

- `critical`: Prevents game from loading or causes data loss
- `high`: Major functionality broken, significant user impact
- `medium`: Feature degraded, workaround available
- `low`: Minor cosmetic issue, no functional impact
- `info`: Informational notice, no action required

### Problem Status Workflow

```
reported → assigned → in_progress → resolved → verified → closed
                  ↓                       ↓
              deferred              reopened
                                        ↓
                                  in_progress
```

## Behavior and rules

### Automated Health Check Execution

**Trigger Points**:
- **On game publish**: Full health check suite runs before status changes to `live`
- **On asset upload**: Asset-specific checks run when bundle or thumbnail updated
- **Scheduled daily**: All live games checked once per 24 hours
- **On-demand**: Content Editors or SysAdmin can manually trigger checks
- **Post-deployment**: All games checked after platform updates

**Check Execution Rules**:
- Health checks run asynchronously to avoid blocking publish workflow
- Failed checks block publish unless SysAdmin overrides with documented reason
- Checks timeout after 60 seconds to prevent hanging
- Browser compatibility checks run in headless browser environments
- Check results cached for 6 hours for non-critical queries

**Health Score Calculation**:
```
health_score = (checks_passed / total_checks) * 100
- Deduct 10 points per open critical problem
- Deduct 5 points per open high-severity problem
- Deduct 2 points per warning
- Minimum score: 0, Maximum score: 100
```

**Health Status Assignment**:
- `pass`: health_score >= 95, no critical or high problems
- `warn`: health_score 70-94, or 1+ medium problems
- `fail`: health_score < 70, or 1+ critical problems
- `pending_review`: health_score >= 80 but no QA review completed

### Manual QA Review Workflow

**QA Review Checklist** (per game):
1. **Functional Testing**:
   - [ ] Learn stage loads and displays content correctly
   - [ ] Play stage interactive elements respond to input
   - [ ] Quiz stage scoring calculates accurately
   - [ ] Challenge stage (if enabled) functions as designed
   - [ ] Review stage triggers appropriately after Quiz completion
   
2. **Cross-Browser Testing**:
   - [ ] Game tested on Chrome 114+ (desktop)
   - [ ] Game tested on Safari 17+ (desktop)
   - [ ] Game tested on Safari iOS 17+ (mobile)
   - [ ] Game tested on Chrome Android 13+ (mobile)
   
3. **Device Requirement Testing**:
   - [ ] MIDI keyboard pairing tested (if midi_required = true)
   - [ ] Microphone detection tested (if mic_required = true)
   - [ ] Touch input tested on tablet devices
   - [ ] Keyboard shortcuts functional
   
4. **Pedagogical Review**:
   - [ ] Instructions clear and age-appropriate
   - [ ] Musical content pedagogically accurate
   - [ ] Difficulty rating matches actual gameplay
   - [ ] Feedback messages constructive and encouraging
   - [ ] Concept alignment matches metadata tags
   
5. **Accessibility Review**:
   - [ ] Color contrast meets WCAG AA standards
   - [ ] Text readable at standard sizes
   - [ ] Keyboard navigation functional
   - [ ] Screen reader labels appropriate
   - [ ] Reduced motion mode available (if applicable)
   
6. **Performance Review**:
   - [ ] Game loads within 30 seconds on standard connection
   - [ ] No frame rate drops during normal gameplay
   - [ ] Audio playback smooth and synchronized
   - [ ] Memory usage reasonable (< 500MB peak)

**QA Review Outcomes**:
- **Approved**: Game meets all quality standards, publish allowed
- **Approved with Warnings**: Minor issues noted but not blocking
- **Rejected**: Critical issues require fixes before publish
- **Needs Revision**: Content or design changes recommended

**QA Review Assignment**:
- New games automatically assigned to QA queue
- Content Editors cannot QA review their own games
- At least one QA reviewer must approve before publish
- SysAdmin can override QA rejection with documented justification

### Problem Tracking and Resolution

**Problem Reporting Sources**:
- **Automated health checks**: System-detected issues create problem reports automatically
- **QA reviews**: Reviewers can file problems during manual testing
- **Teacher feedback**: Teachers report problems via "Report Issue" button on game detail
- **Student support**: Support team can file problems on behalf of students
- **System monitoring**: Runtime errors trigger problem reports

**Priority Assignment Algorithm**:
```
priority = critical if:
  - severity = critical AND impact_students_count > 50
  - Game completely unplayable
  
priority = high if:
  - severity = high AND impact_students_count > 20
  - Major functionality broken
  - Published < 30 days ago (new game regression)
  
priority = medium if:
  - severity = medium OR impact_students_count 5-20
  - Feature degraded but workaround available
  
priority = low if:
  - severity = low AND impact_students_count < 5
  - Cosmetic issue or minor inconvenience
```

**Resolution Workflow**:
1. **Report**: Problem created with description, severity, impact
2. **Triage**: Content Lead reviews and assigns priority and owner
3. **Investigation**: Assigned editor investigates root cause
4. **Fix**: Editor implements fix and deploys updated game version
5. **Verification**: Original reporter or QA team verifies fix
6. **Close**: Problem marked closed, resolution documented

**Resolution SLA Targets** (based on priority):
- Critical: 24 hours
- High: 7 days
- Medium: 30 days
- Low: 90 days or next planned update

**Escalation Policy**:
- Overdue critical problems escalate to Content Lead after 48 hours
- Overdue high problems escalate after 14 days
- Problems reopened 3+ times escalate to SysAdmin for systemic review

### Audit Trail Maintenance

**Audit Events Captured**:
- `game_health_check_run`: Automated health check execution
- `game_health_status_changed`: Health status transition
- `game_qa_review_started`: QA review session initiated
- `game_qa_review_completed`: QA review outcome recorded
- `game_problem_reported`: New problem filed
- `game_problem_assigned`: Problem assigned to owner
- `game_problem_status_changed`: Problem workflow transition
- `game_problem_resolved`: Problem marked resolved
- `game_problem_closed`: Problem verified and closed
- `game_health_override`: SysAdmin override of health check failure

**Audit Log Retention**:
- Game-level audit logs retained indefinitely
- Health check results retained for 2 years
- QA review records retained permanently
- Problem reports retained permanently (even after game deprecated)

**Audit Log Access**:
- Content Editors: Read access to games they manage
- QA Reviewers: Read access to all games and reviews
- SysAdmin: Full read/write access to all audit logs
- Teachers: No direct audit access (can view problem status only)

### Health Dashboards and Reporting

**Registry Health Overview Dashboard**:
- Total games by health status (pass/warn/fail/pending)
- Trend chart: health scores over last 90 days
- Top 10 games with most open problems
- Recent health check failures
- QA review queue depth
- Average problem resolution time by priority

**Game Detail Health View**:
- Current health status and score
- Failed/warned checks with details and remediation steps
- Open problems with priority and assignment
- Resolved problems history
- QA review status and notes
- Recent audit log entries

**Problem Management Dashboard**:
- Open problems by priority and age
- Problems approaching SLA deadline
- Most frequently reported problems (pattern detection)
- Problem resolution metrics (time-to-close by severity)
- Top problem reporters and resolvers

## UX requirements (if applicable)

### Health Status Indicators

**Catalog List View**:
- Health badge in "Health" column: Green check (pass), Yellow warning (warn), Red X (fail), Gray clock (pending)
- Tooltip on hover: "Health Score: 85 - 2 warnings, QA approved 2025-01-25"
- Sortable by health status and health score

**Game Detail - Health Tab**:

**Section 1: Health Summary Card**:
- Large health status icon with color coding
- Health score as percentage with trend indicator (↑ improving, ↓ declining, → stable)
- Last check timestamp and "Run Checks Now" button
- Quick stats: X checks passed, Y warnings, Z failures

**Section 2: Health Checks Table**:
- Columns: Check Name, Category (Asset/Browser/Device/Performance/Content), Status (pass/warn/fail), Message, Last Run
- Row colors: Green (pass), Yellow (warn), Red (fail)
- Expandable rows show detailed check output
- Filter by category and status
- "Export Report" button for PDF/CSV download

**Section 3: QA Review Panel**:
- QA review status badge: Approved, Approved with Warnings, Rejected, Pending, Not Required
- Reviewer name and date
- QA notes displayed in read-only text area
- Link to full QA checklist results
- "Request Re-Review" button (Content Editors only)

**Section 4: Open Problems List**:
- Card-based layout: Title, Severity badge, Priority badge, Status, Assigned to, Days open
- Click card to view problem detail modal
- Filter by severity, priority, status
- "Report New Problem" button

**Section 5: Audit Log**:
- Chronological list of health-related events
- Expandable entries show full event details
- Filter by event type, date range, user
- Paginated, default 50 entries per page

### Problem Tracking Interface

**Problem Report Form**:
- Title (required, max 100 chars)
- Description (required, rich text editor)
- Problem Type dropdown (blank_screen, midi_failure, audio_issue, scoring_error, asset_missing, performance, etc.)
- Severity radio buttons (Critical, High, Medium, Low)
- Impact fields: Estimated students affected, Assignments affected
- Attach screenshot or screen recording
- Tags (autocomplete from existing tags)
- "Submit Problem" button

**Problem Detail Modal**:
- Header: Title, Severity badge, Priority badge, Status badge
- Tabs:
  - **Details**: Description, game/stage context, reporter, dates, impact metrics
  - **Activity**: Status transitions, comments, assignments, resolution notes
  - **Related**: Linked problems, related health checks, related audit events
  - **Verification**: Verification checklist, tester notes, reopening history
- Action buttons (context-dependent by role and status):
  - "Assign to Me", "Change Priority", "Change Status", "Add Comment", "Link Related Problem", "Close Problem"

**QA Review Interface**:

**QA Review Dashboard** (QA Reviewers only):
- Queue of games awaiting review, sorted by publish date
- Columns: Game Name, Category, Submitted by, Submitted date, Priority (new games = high)
- "Start Review" button launches review session

**QA Review Session**:
- Left panel: Game player embedded (full-screen play mode)
- Right panel: QA checklist with checkboxes
- Bottom panel: Notes text area, severity indicators, pass/fail/warn selectors per checklist item
- Top action bar: "Approve", "Approve with Warnings", "Reject", "Save Progress", "Cancel Review"
- Keyboard shortcuts: Space = toggle checkbox, Cmd+Enter = submit review

### Health Report Export

**Export Formats**:
- PDF: Formatted health report with charts and check details
- CSV: Tabular health check results for analysis
- JSON: Full health data for API integration

**Export Customization**:
- Date range filter
- Include/exclude specific check categories
- Include/exclude audit log
- Include/exclude problem history

### Accessibility

- All health status indicators have text equivalents and ARIA labels
- Color is not the sole indicator of health status (icons + text)
- Health check results table keyboard navigable
- Problem report form fully keyboard accessible
- Screen reader announces health status changes
- High contrast mode supported for all health visualizations

## Telemetry

### Health Check Events

**Health Check Run**  
Event: `game_health_check_run`  
Properties:
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "check_trigger": "scheduled|manual|pre_publish|post_deployment",
  "checks_total": 15,
  "checks_passed": 12,
  "checks_failed": 2,
  "checks_warned": 1,
  "duration_ms": 8500,
  "health_score_before": 85,
  "health_score_after": 75,
  "health_status_before": "pass",
  "health_status_after": "warn"
}
```
Fires: After health check suite completes  
Sampling: Never sampled (100% capture for audit compliance)

**Health Check Failed**  
Event: `game_health_check_failed`  
Properties:
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "check_id": "safari_compatibility",
  "check_category": "browser",
  "severity": "error",
  "error_message": "Game fails to load on Safari 17.2",
  "error_details": "TypeError: WebGL context creation failed"
}
```
Fires: When individual health check fails  
Sampling: Never sampled

**Health Status Changed**  
Event: `game_health_status_changed`  
Properties:
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "status_from": "pass",
  "status_to": "warn",
  "score_from": 95,
  "score_to": 75,
  "trigger": "health_check|problem_reported|problem_resolved"
}
```
Fires: When health status transitions  
Sampling: Never sampled

### QA Review Events

**QA Review Started**  
Event: `game_qa_review_started`  
Properties:
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "reviewer_id": "usr_67890",
  "review_type": "initial|re_review|post_fix"
}
```
Fires: When QA reviewer opens review session  
Sampling: Never sampled

**QA Review Completed**  
Event: `game_qa_review_completed`  
Properties:
```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "reviewer_id": "usr_67890",
  "outcome": "approved|approved_with_warnings|rejected",
  "duration_minutes": 45,
  "checklist_items_total": 25,
  "checklist_items_passed": 23,
  "checklist_items_failed": 2,
  "notes_length": 250
}
```
Fires: When QA reviewer submits review  
Sampling: Never sampled

### Problem Tracking Events

**Problem Reported**  
Event: `game_problem_reported`  
Properties:
```json
{
  "problem_id": "prob_12345",
  "game_id": "G-01542",
  "stage_id": "G-01542-quiz",
  "problem_type": "blank_screen",
  "severity": "critical",
  "reporter_id": "usr_teacher_456",
  "reporter_role": "teacher",
  "source": "manual|automated|support_ticket",
  "impact_students": 45,
  "impact_assignments": 12
}
```
Fires: When new problem reported  
Sampling: Never sampled

**Problem Resolved**  
Event: `game_problem_resolved`  
Properties:
```json
{
  "problem_id": "prob_12345",
  "game_id": "G-01542",
  "resolver_id": "usr_content_789",
  "resolution_time_hours": 18,
  "sla_met": true,
  "resolution_method": "game_fix|asset_update|configuration|workaround"
}
```
Fires: When problem marked resolved  
Sampling: Never sampled

### Health Dashboard Events

**Health Dashboard Viewed**  
Event: `game_health_dashboard_viewed`  
Properties:
```json
{
  "user_id": "usr_67890",
  "user_role": "content_editor",
  "view_type": "registry_overview|game_detail|problem_dashboard",
  "filters_applied": ["status:warn", "category:browser"]
}
```
Fires: When user opens health dashboard  
Sampling: 10% (analytics only)

## Permissions

### Health Monitoring Access

**Read Health Status**:
- Content Editors: ✅ All games
- QA Reviewers: ✅ All games
- SysAdmin: ✅ All games
- Teachers: ✅ Live games only (basic health badge, no details)
- Students: ❌ No access

**Run Health Checks**:
- Content Editors: ✅ Games they manage
- QA Reviewers: ✅ All games
- SysAdmin: ✅ All games
- All others: ❌ No

**View Audit Logs**:
- Content Editors: ✅ Games they manage
- QA Reviewers: ✅ All games
- SysAdmin: ✅ All games with full detail
- All others: ❌ No

### QA Review Permissions

**Conduct QA Reviews**:
- QA Reviewers: ✅ Yes (cannot review own authored games)
- SysAdmin: ✅ Yes (can review any game)
- Content Editors: ❌ No (cannot self-review)

**Override Failed Health Checks**:
- SysAdmin: ✅ Yes (with mandatory documentation)
- All others: ❌ No

**Approve Games with Warnings**:
- QA Reviewers: ✅ Yes
- Content Lead: ✅ Yes
- SysAdmin: ✅ Yes

### Problem Tracking Permissions

**Report Problems**:
- Content Editors: ✅ Yes
- QA Reviewers: ✅ Yes
- SysAdmin: ✅ Yes
- Teachers: ✅ Yes (via "Report Issue" button)
- Students: ❌ No (must report via teacher or support)

**Assign Problems**:
- Content Lead: ✅ Yes (any problem)
- SysAdmin: ✅ Yes (any problem)
- Content Editors: ✅ Yes (problems on games they manage)
- All others: ❌ No

**Resolve Problems**:
- Assigned owner: ✅ Yes
- Content Lead: ✅ Yes (any problem)
- SysAdmin: ✅ Yes (any problem)
- All others: ❌ No

**Close Problems**:
- Original reporter: ✅ Yes (verify and close)
- QA Reviewer: ✅ Yes (verify and close)
- SysAdmin: ✅ Yes (any problem)

## Supporting Documents Referenced

This games audit and health specification draws from the following source documents:

- [Games Loaded.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Sheet1.csv) — Current game library inventory showing status and health indicators
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game data structure for validation and audit requirements
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System admin tools requirements including content management

## Dependencies

### Internal Dependencies

- **[Game Model](game-model.md)**: Health fields are part of game schema, checks validate model constraints
- **[Games Registry Overview](games-registry-overview.md)**: Health status displayed in catalog, health checks integrated into publish workflow
- **[Games CRUD](games-crud.md)**: Health checks run on create/update operations, audit logs track all CRUD events
- **[Games Metadata](games-metadata.md)**: Metadata completeness validated by health checks
- **[Games Stages Policy](games-stages-policy.md)**: Stage configuration validated by health checks
- **[Games Assets Pipeline](games-assets-pipeline.md)**: Asset health checks validate bundle integrity and reachability
- **[Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md)**: Browser health checks reference platform-wide support matrix
- **[QA Checklists](../19-quality-and-operations/qa-checklists.md)**: QA review checklists follow platform-wide QA standards
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Health telemetry events follow canonical event schema
- **[Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)**: Health audit events feed into system-wide audit infrastructure
- **[Permissions Table](../02-roles-and-permissions/permissions-table.md)**: Permission checks enforce role-based access to health features

### External Dependencies

- **Health Check Service**: Microservice executes automated health checks asynchronously
- **Browser Testing Grid**: Headless browser environments (BrowserStack, Sauce Labs) for compatibility checks
- **Asset CDN**: Validates asset reachability and performance
- **Problem Tracking Database**: Stores problem reports separate from main game database for historical retention
- **Audit Log Database**: Immutable audit trail storage with long-term retention
- **Notification Service**: Alerts stakeholders when critical health checks fail or problems escalate

### Integration Points

- **Publish Workflow**: Health checks gate publish operation, must pass before draft → live transition
- **Import Pipeline**: Bulk-imported games queued for health checks post-import
- **Deployment Pipeline**: Platform updates trigger full health check sweep across all games
- **Support Ticketing**: Problem reports can be created from support tickets, linked bidirectionally
- **Analytics Dashboard**: Health metrics feed into Content Lead and SysAdmin dashboards

## Open questions

### Health Check Frequency and Performance

- **Question**: Should health checks run continuously or on a fixed schedule?  
- **Current Assumption**: Daily scheduled checks for live games, on-demand for others
- **Alternative**: Continuous monitoring with intelligent check throttling based on game stability
- **Trade-offs**: Continuous monitoring provides faster issue detection but increases infrastructure cost
- **Decision Needed By**: Phase 1 alpha

### Problem Priority Algorithm Tuning

- **Question**: Should problem priority be auto-calculated or manually assigned?  
- **Current Assumption**: Auto-calculated based on severity + impact, manually adjustable
- **Alternative**: Fully manual priority assignment by Content Lead
- **Trade-offs**: Automation ensures consistency but may miss contextual factors
- **Decision Needed By**: Phase 1 beta

### Health Score Weighting

- **Question**: Should all health checks be weighted equally in score calculation?  
- **Current Assumption**: Equal weighting, with deductions for open problems by severity
- **Alternative**: Weight critical checks (browser compatibility, asset loading) higher than minor checks (bundle size, performance)
- **Trade-offs**: Weighted scoring better reflects actual playability but adds complexity
- **Decision Needed By**: Phase 1 beta

### QA Review Requirement Threshold

- **Question**: Should all games require QA review before publish, or only new/significantly changed games?  
- **Current Assumption**: All games require QA review before initial publish, minor updates skip review
- **Alternative**: Risk-based review (e.g., only review if automated checks fail)
- **Trade-offs**: Universal review ensures quality but creates bottleneck; risk-based is faster but may miss issues
- **Decision Needed By**: Phase 1 alpha

### Problem Resolution SLA Enforcement

- **Question**: Should problem resolution SLAs be enforced with automatic escalations, or serve as guidelines?  
- **Current Assumption**: Automatic escalation after SLA deadline, but no blocking enforcement
- **Alternative**: Block game edits until critical problems resolved
- **Trade-offs**: Strict enforcement ensures accountability but may create workflow friction
- **Decision Needed By**: Phase 2

### Historical Health Trend Retention

- **Question**: How long should detailed health trend data be retained for analysis?  
- **Current Assumption**: 2 years of detailed check results, summary metrics indefinitely
- **Alternative**: 5 years detailed, indefinite summary for long-term quality research
- **Trade-offs**: Longer retention enables better trend analysis but increases storage costs
- **Decision Needed By**: Phase 2

### Teacher Problem Reporting Workflow

- **Question**: Should teachers report problems directly into the system, or via support intermediary?  
- **Current Assumption**: Teachers report directly via "Report Issue" button, auto-creates problem record
- **Alternative**: Teachers submit to support, support team triages and creates formal problem reports
- **Trade-offs**: Direct reporting is faster but may create noise; mediated reporting filters issues but adds delay
- **Decision Needed By**: Phase 1 beta

### Health Override Justification Requirements

- **Question**: What level of documentation should be required for SysAdmin health check overrides?  
- **Current Assumption**: Mandatory text justification field, minimum 50 characters
- **Alternative**: Formal override request with approval workflow
- **Trade-offs**: Lightweight documentation is fast but may not capture enough context for audit
- **Decision Needed By**: Phase 1 beta
