# Teacher Reports KPIs

## Purpose
This specification defines the key performance indicators (KPIs) and metrics available to teachers in the MLC-LMS platform. These metrics provide teachers with actionable insights into student progress, engagement, and learning outcomes to inform instructional decisions, identify students needing support, and celebrate achievements. The KPIs are designed to support data-driven teaching while maintaining focus on pedagogical effectiveness rather than administrative oversight.

## Scope
**Included**
- Student progress and mastery metrics at individual and class levels
- Engagement and participation indicators
- Assignment completion and performance analytics
- Learning sequence progression tracking
- Achievement and milestone recognition metrics
- Comparative analytics (student vs. class, class vs. historical)
- Export and reporting capabilities for teacher use

**Excluded**
- Administrative analytics and school-wide metrics (see [admin-reports-kpis.md](./admin-reports-kpis.md))
- System operational dashboards (see [observability.md](../19-quality-and-operations/observability.md))
- Individual student detailed game session data (available in student reports)
- Cross-organization comparative analytics

## Data model (if applicable)

### Core KPI Entities

#### StudentProgressSnapshot
```json
{
  "student_id": "usr_12345",
  "teacher_id": "usr_67890",
  "sequence_id": "seq_456",
  "snapshot_date": "2025-01-27T00:00:00Z",
  "coverage_percentage": 67.5,
  "mastery_percentage": 82.3,
  "last_active_at": "2025-01-26T15:30:00Z",
  "total_playtime_minutes": 145,
  "assignments_completed": 12,
  "assignments_overdue": 2,
  "current_level": 3,
  "on_track_status": "on_track|at_risk|behind"
}
```

#### AssignmentOutcome
```json
{
  "assignment_id": "asg_789",
  "student_id": "usr_12345",
  "sequence_id": "seq_456",
  "status": "completed|in_progress|not_started|overdue",
  "completion_percentage": 100.0,
  "average_score": 87.5,
  "best_score": 95.0,
  "attempts": 3,
  "time_to_completion_hours": 48.5,
  "assigned_at": "2025-01-20T09:00:00Z",
  "completed_at": "2025-01-22T10:30:00Z",
  "due_date": "2025-01-25T23:59:59Z"
}
```

#### GameSessionAggregate
```json
{
  "student_id": "usr_12345",
  "game_id": "G-01542",
  "game_category": "pitch_aural",
  "total_sessions": 8,
  "average_score": 84.2,
  "best_score": 95.0,
  "mastery_achieved": true,
  "last_played_at": "2025-01-26T14:20:00Z",
  "total_playtime_minutes": 45,
  "concept_tags": ["guide_notes", "treble_staff"]
}
```

### KPI Calculation Models

#### Mastery Rate Calculation
```
Mastery Rate = (Games with Target Score Achieved / Total Games Attempted) × 100
```

#### Engagement Score Calculation
```
Engagement Score = (Active Days in Period / Total Days in Period) × 100
```

#### Progress Velocity Calculation
```
Progress Velocity = (Current Level - Starting Level) / Days Since Enrollment
```

## Behavior and rules

### KPI Calculation Rules
- **Time Windows**: Default to last 30 days, configurable to 7, 14, 30, 60, 90 days, or custom ranges
- **Mastery Definition**: Target score achieved on Quiz stage (as defined in [game-model.md](../08-games-registry-and-authoring/game-model.md))
- **On-Track Status**: Based on assignment completion rate and mastery progression relative to expected pace
- **Data Freshness**: KPIs updated every 15 minutes during business hours, hourly during off-hours
- **Historical Comparisons**: Always compare against same time period in previous month/quarter

### Filtering and Scope Rules
- **Teacher Scope**: Teachers see only their assigned students and groups
- **Teacher-Admin Scope**: Additional access to all teachers and students within their subscriber organization
- **Data Privacy**: No cross-organization data visible, even in aggregated form
- **Retention Flags**: Students marked for retention review highlighted in all relevant KPIs

### Performance Safeguards
- **Pagination**: Large result sets paginated with 50 items per page maximum
- **Caching**: Frequently accessed KPIs cached for 5 minutes
- **Query Limits**: Complex queries limited to 10,000 records maximum
- **Export Limits**: CSV exports limited to 5,000 records per request

## UX requirements (if applicable)

### Dashboard Layout
- **Overview Tab**: Key metrics tiles with trend indicators and drill-down links
- **Students Tab**: Sortable table with student progress summary and individual detail views
- **Assignments Tab**: Assignment status grid with bulk actions and filtering
- **Games Tab**: Game category performance heatmap with mastery indicators
- **Exports Tab**: Custom report builder with saved view management

### Key States and Interactions
- **Loading States**: Skeleton placeholders for data loading (max 3 seconds)
- **Empty States**: Helpful messaging when no data available with action suggestions
- **Error States**: Clear error messages with retry options and support contact
- **Drill-down Flow**: Smooth navigation from high-level metrics to detailed views

### Accessibility Requirements
- **Screen Reader Support**: All KPIs have descriptive text alternatives
- **Keyboard Navigation**: Full keyboard accessibility for all interactive elements
- **Color Contrast**: Minimum 4.5:1 contrast ratio for all text and data visualizations
- **Focus Management**: Clear focus indicators and logical tab order
- **Responsive Design**: Optimized for desktop (primary), tablet, and mobile viewing

## Telemetry

### KPI View Events
```javascript
// Teacher views KPI dashboard
{
  event_type: "kpi_dashboard_viewed",
  properties: {
    teacher_id: "usr_67890",
    time_range: "30_days",
    filters_applied: ["sequence_id", "student_group"],
    kpis_viewed: ["mastery_rate", "engagement_score", "progress_velocity"]
  }
}

// Teacher drills down into specific KPI
{
  event_type: "kpi_drilldown_opened",
  properties: {
    teacher_id: "usr_67890",
    kpi_type: "student_progress",
    entity_id: "usr_12345",
    source_view: "students_table"
  }
}

// Teacher exports KPI data
{
  event_type: "kpi_export_requested",
  properties: {
    teacher_id: "usr_67890",
    export_format: "csv",
    kpi_types: ["mastery_rate", "assignment_completion"],
    record_count: 150,
    filters_applied: ["last_30_days", "sequence_456"]
  }
}
```

### KPI Performance Events
```javascript
// KPI calculation performance
{
  event_type: "kpi_calculation_completed",
  properties: {
    kpi_type: "class_mastery_summary",
    calculation_time_ms: 1250,
    record_count: 45,
    cache_hit: false
  }
}
```

## Permissions

### Teacher Role Permissions
- **Read Access**: All KPIs for assigned students and groups only
- **Export Access**: CSV export of KPI data within their scope
- **Drill-down Access**: Detailed views of individual student progress
- **Historical Access**: KPI trends and comparisons within their scope
- **No Cross-Teacher Access**: Cannot view other teachers' student data

### Teacher-Admin Role Permissions
- **Extended Read Access**: All KPIs for teachers and students within their subscriber organization
- **Cross-Teacher Analytics**: Comparative metrics across teachers in their scope
- **Bulk Export Access**: Organization-wide KPI exports (with appropriate data privacy)
- **Teacher Performance Metrics**: Analytics on teacher effectiveness and student outcomes

### Data Privacy and Compliance
- **FERPA Compliance**: Educational records properly access-controlled and audited
- **COPPA Compliance**: No PII exposure for students under 13
- **Data Retention**: KPI data retained for 7 years for compliance and historical analysis
- **Audit Logging**: All KPI access logged for security and compliance monitoring

## Supporting Documents Referenced

This teacher reports KPIs specification draws from the following source documents:

- [Assignment Play Re-design.docx.txt](../00-ORIG-CONTEXT/Assignment%20Play%20Re-design.docx.txt) — Score display and reporting requirements
- [EXE specs 2012-0913 - Assignment Screen.csv](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Assignment%20Screen.csv) — Assignment score tracking and reporting
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Teacher reporting and visibility permissions

## Dependencies

### Core System Dependencies
- **Event Model**: KPI calculations based on events defined in [event-model.md](./event-model.md)
- **Game Model**: Mastery definitions and scoring aligned with [game-model.md](../08-games-registry-and-authoring/game-model.md)
- **Assignment Engine**: Progress tracking logic from [assignment-progress-tracking.md](../10-assignments-engine/assignment-progress-tracking.md)
- **Roles and Permissions**: Access control from [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md)

### Data Source Dependencies
- **Student Progress Tracking**: Real-time progress data from assignment and game systems
- **Learning Analytics**: Advanced analytics capabilities from [learning-analytics.md](./learning-analytics.md)
- **User Management**: Teacher-student relationships and organizational hierarchy
- **Content Architecture**: Game taxonomy and sequence definitions

### Technical Dependencies
- **Time-Series Database**: High-performance storage for KPI calculations
- **Caching Layer**: Redis for frequently accessed KPI data
- **Export Service**: CSV/JSON export capabilities with proper formatting
- **Real-time Updates**: WebSocket connections for live KPI updates

### Related Documentation
- [Teacher Reports](../04-teacher-experience/teacher-reports.md) - Teacher-facing reporting interface
- [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md) - Progress tracking logic
- [Retention Review Logic](../10-assignments-engine/retention-review-logic.md) - Student retention flags
- [Game Taxonomy](../07-content-architecture/game-taxonomy.md) - Game categorization system

## Open questions

### KPI Definition and Calculation
- **Mastery Thresholds**: Should mastery thresholds be configurable per teacher or standardized?
- **Progress Benchmarks**: What are the expected progress rates for different age groups and skill levels?
- **Engagement Metrics**: How do we define "engaged" vs. "disengaged" students?
- **Comparative Baselines**: What should be the baseline for class vs. individual student comparisons?

### Performance and Scalability
- **Real-time Updates**: What's the acceptable latency for KPI updates during peak usage?
- **Data Volume**: How do we handle KPI calculations for teachers with 100+ students?
- **Caching Strategy**: Should we pre-calculate common KPI combinations?
- **Export Performance**: What's the maximum acceptable time for large KPI exports?

### User Experience and Adoption
- **KPI Onboarding**: How do we help teachers understand and effectively use these metrics?
- **Custom Dashboards**: Should teachers be able to create custom KPI dashboards?
- **Alert Thresholds**: Should teachers be able to set up alerts for specific KPI thresholds?
- **Mobile Optimization**: How do we optimize KPI views for mobile devices?

### Data Privacy and Compliance
- **Cross-Organization Insights**: Can we provide anonymized insights across organizations for research?
- **Parent Access**: Should parents have access to any of these teacher KPIs?
- **Data Portability**: What KPI data should be included in student data exports?
- **Retention Policies**: Should KPI data retention be configurable per organization?
