# Admin Reports KPIs

## Purpose
This specification defines the key performance indicators (KPIs) and metrics available to administrative users in the MLC-LMS platform. These metrics provide subscriber administrators and system administrators with comprehensive insights into organizational performance, subscription health, user engagement, and business intelligence to support strategic decision-making, resource allocation, and platform optimization. The KPIs are designed to provide both high-level oversight and detailed operational insights while maintaining appropriate data privacy and access controls.

## Scope
**Included**
- Organization-wide performance and engagement metrics
- Subscription health and utilization analytics
- User growth, retention, and churn analysis
- Revenue and billing performance indicators
- System usage patterns and capacity planning metrics
- Teacher effectiveness and student outcome analytics
- Cross-organization benchmarking and industry insights
- Administrative action tracking and audit metrics
- Export and reporting capabilities for administrative use

**Excluded**
- Individual student detailed game session data (available in teacher reports)
- Real-time system operational dashboards (see [observability.md](../19-quality-and-operations/observability.md))
- Teacher-specific classroom management metrics (see [teacher-reports-kpis.md](./teacher-reports-kpis.md))
- System-level technical performance metrics (covered in architecture docs)

## Data model (if applicable)

### Core Administrative KPI Entities

#### OrganizationHealthSnapshot
```json
{
  "organization_id": "org_54321",
  "snapshot_date": "2025-01-27T00:00:00Z",
  "subscription_tier": "ensemble",
  "total_seats": 150,
  "active_seats": 127,
  "utilization_rate": 84.7,
  "active_students": 120,
  "active_teachers": 7,
  "total_playtime_hours": 245.5,
  "avg_session_duration_minutes": 18.3,
  "retention_rate_30d": 89.2,
  "churn_rate_30d": 2.1,
  "revenue_mrr": 299.95,
  "health_score": 87.5
}
```

#### SubscriptionAnalytics
```json
{
  "organization_id": "org_54321",
  "subscription_id": "sub_789",
  "tier": "ensemble",
  "start_date": "2024-06-15T00:00:00Z",
  "current_period_start": "2025-01-15T00:00:00Z",
  "current_period_end": "2025-02-14T23:59:59Z",
  "base_seats": 20,
  "additional_seats": 130,
  "total_seats": 150,
  "seat_utilization_rate": 84.7,
  "monthly_revenue": 299.95,
  "annual_revenue": 3599.40,
  "payment_status": "current",
  "days_since_last_payment": 12,
  "upgrade_history": [
    {
      "date": "2024-08-15T00:00:00Z",
      "from_tier": "solo",
      "to_tier": "ensemble",
      "seat_change": 15
    }
  ]
}
```

#### UserEngagementMetrics
```json
{
  "organization_id": "org_54321",
  "period_start": "2025-01-01T00:00:00Z",
  "period_end": "2025-01-31T23:59:59Z",
  "total_active_users": 127,
  "new_users": 8,
  "returning_users": 119,
  "daily_active_users_avg": 45.2,
  "weekly_active_users_avg": 89.7,
  "monthly_active_users": 127,
  "avg_sessions_per_user": 12.3,
  "avg_session_duration_minutes": 18.3,
  "total_playtime_hours": 245.5,
  "engagement_score": 78.4,
  "retention_rates": {
    "day_1": 95.2,
    "day_7": 87.3,
    "day_30": 76.8,
    "day_90": 68.9
  }
}
```

#### LearningOutcomeAnalytics
```json
{
  "organization_id": "org_54321",
  "period_start": "2025-01-01T00:00:00Z",
  "period_end": "2025-01-31T23:59:59Z",
  "total_games_played": 15420,
  "total_mastery_achievements": 8920,
  "overall_mastery_rate": 57.8,
  "avg_score_across_games": 78.4,
  "concept_mastery_breakdown": {
    "pitch_aural": 62.3,
    "rhythm": 71.2,
    "melody": 58.9,
    "chords": 45.6,
    "scales": 52.1
  },
  "sequence_progression": {
    "lifetime_musician": 89.2,
    "solfege": 67.8,
    "evaluation": 45.3
  },
  "teacher_effectiveness_scores": {
    "avg_student_mastery_rate": 57.8,
    "avg_assignment_completion": 84.3,
    "avg_student_engagement": 78.4
  }
}
```

### KPI Calculation Models

#### Utilization Rate Calculation
```
Utilization Rate = (Active Seats / Total Seats) × 100
```

#### Health Score Calculation
```
Health Score = (Utilization Rate × 0.3) + (Retention Rate × 0.3) + (Engagement Score × 0.2) + (Payment Health × 0.2)
```

#### Churn Rate Calculation
```
Churn Rate = (Users Lost in Period / Users at Start of Period) × 100
```

#### Revenue per User Calculation
```
Revenue per User = Monthly Revenue / Active Users
```

## Behavior and rules

### KPI Calculation Rules
- **Time Windows**: Default to last 30 days, configurable to 7, 14, 30, 60, 90 days, or custom ranges
- **Data Freshness**: KPIs updated every 15 minutes during business hours, hourly during off-hours
- **Historical Comparisons**: Always compare against same time period in previous month/quarter/year
- **Rolling Averages**: 7-day and 30-day rolling averages for trend analysis
- **Seasonal Adjustments**: Account for academic calendar and holiday patterns

### Access Control Rules
- **Subscriber-Admin**: Full access to their organization's KPIs only
- **System-Admin**: Global access to all organizations with anonymized cross-org insights
- **Data Privacy**: No cross-organization data visible to subscriber admins
- **Audit Logging**: All KPI access logged for security and compliance monitoring

### Performance Safeguards
- **Pagination**: Large result sets paginated with 100 items per page maximum
- **Caching**: Frequently accessed KPIs cached for 10 minutes
- **Query Limits**: Complex queries limited to 50,000 records maximum
- **Export Limits**: CSV exports limited to 25,000 records per request
- **Rate Limiting**: Maximum 100 KPI requests per minute per user

## UX requirements (if applicable)

### Dashboard Layout
- **Overview Tab**: Executive summary with key metrics tiles and trend indicators
- **Subscriptions Tab**: Subscription health, utilization, and revenue analytics
- **Users Tab**: User growth, engagement, and retention metrics with drill-down capabilities
- **Learning Tab**: Learning outcomes, mastery rates, and teacher effectiveness analytics
- **Financial Tab**: Revenue, billing, and subscription performance metrics
- **Exports Tab**: Custom report builder with saved view management and scheduled reports

### Key States and Interactions
- **Loading States**: Skeleton placeholders for data loading (max 5 seconds)
- **Empty States**: Helpful messaging when no data available with action suggestions
- **Error States**: Clear error messages with retry options and support contact
- **Drill-down Flow**: Smooth navigation from high-level metrics to detailed organizational views
- **Comparison Views**: Side-by-side period comparisons and benchmark overlays

### Accessibility Requirements
- **Screen Reader Support**: All KPIs have descriptive text alternatives and ARIA labels
- **Keyboard Navigation**: Full keyboard accessibility for all interactive elements
- **Color Contrast**: Minimum 4.5:1 contrast ratio for all text and data visualizations
- **Focus Management**: Clear focus indicators and logical tab order
- **Responsive Design**: Optimized for desktop (primary), tablet, and mobile viewing
- **Data Tables**: Sortable, filterable tables with proper headers and captions

## Telemetry

### Administrative KPI Events
```javascript
// Admin views KPI dashboard
{
  event_type: "admin_kpi_dashboard_viewed",
  properties: {
    admin_id: "usr_12345",
    organization_id: "org_54321",
    time_range: "30_days",
    filters_applied: ["subscription_tier", "user_type"],
    kpis_viewed: ["utilization_rate", "retention_rate", "revenue_mrr"]
  }
}

// Admin drills down into specific KPI
{
  event_type: "admin_kpi_drilldown_opened",
  properties: {
    admin_id: "usr_12345",
    organization_id: "org_54321",
    kpi_type: "user_engagement",
    entity_id: "org_54321",
    source_view: "overview_dashboard"
  }
}

// Admin exports KPI data
{
  event_type: "admin_kpi_export_requested",
  properties: {
    admin_id: "usr_12345",
    organization_id: "org_54321",
    export_format: "csv",
    kpi_types: ["subscription_health", "user_engagement"],
    record_count: 1250,
    filters_applied: ["last_30_days", "ensemble_tier"]
  }
}
```

### System Performance Events
```javascript
// KPI calculation performance
{
  event_type: "admin_kpi_calculation_completed",
  properties: {
    kpi_type: "organization_health_summary",
    calculation_time_ms: 3250,
    record_count: 150,
    cache_hit: false,
    organization_count: 1
  }
}
```

## Permissions

### Subscriber-Admin Role Permissions
- **Organization Access**: Full KPI access for their organization only
- **Subscription Analytics**: Complete subscription health and utilization metrics
- **User Analytics**: User engagement, growth, and retention within their organization
- **Learning Analytics**: Learning outcomes and teacher effectiveness for their organization
- **Financial Analytics**: Revenue and billing metrics for their subscriptions
- **Export Access**: Full data export capabilities for their organization
- **No Cross-Organization Access**: Cannot view other organizations' data

### System-Admin Role Permissions
- **Global Access**: All KPIs across all organizations
- **Cross-Organization Analytics**: Anonymized insights and benchmarking data
- **System-Wide Metrics**: Platform health, usage patterns, and capacity planning
- **Financial Analytics**: Complete revenue and billing analytics across all organizations
- **Audit Analytics**: Administrative action tracking and compliance monitoring
- **Full Export**: Complete system-wide data export capabilities
- **Anonymized Insights**: Research and development insights with privacy protection

### Data Privacy and Compliance
- **FERPA Compliance**: Educational records properly access-controlled and audited
- **COPPA Compliance**: No PII exposure for students under 13
- **Data Retention**: KPI data retained for 7 years for compliance and historical analysis
- **Audit Logging**: All KPI access logged for security and compliance monitoring
- **Cross-Organization Privacy**: No individual organization data visible in cross-org analytics

## Dependencies

### Core System Dependencies
- **Event Model**: KPI calculations based on events defined in [event-model.md](./event-model.md)
- **Roles and Permissions**: Access control from [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md)
- **Subscription Management**: Billing and seat management from [seat-management.md](../14-payments-and-subscriptions/seat-management.md)
- **User Management**: Organization hierarchy and user relationships

### Data Source Dependencies
- **Teacher Reports KPIs**: Learning analytics foundation from [teacher-reports-kpis.md](./teacher-reports-kpis.md)
- **Learning Analytics**: Advanced analytics capabilities from [learning-analytics.md](./learning-analytics.md)
- **Assignment Engine**: Progress tracking and learning outcomes
- **Billing System**: Revenue and subscription data from payment providers

### Technical Dependencies
- **Time-Series Database**: High-performance storage for KPI calculations
- **Caching Layer**: Redis for frequently accessed KPI data
- **Export Service**: CSV/JSON export capabilities with proper formatting
- **Real-time Updates**: WebSocket connections for live KPI updates
- **Data Warehouse**: Complex analytics and historical trend analysis

### Related Documentation
- [Subscriber Dashboard](../05-admin-subscriber-experience/subscriber-dashboard.md) - Administrative interface
- [Member Management](../05-admin-subscriber-experience/member-management.md) - User management capabilities
- [Billing Statements](../05-admin-subscriber-experience/billing-statements.md) - Financial reporting
- [Sysadmin Operations Dashboards](./sysadmin-ops-dashboards.md) - System-level operational metrics

## Open questions

### KPI Definition and Business Intelligence
- **Benchmarking Data**: Should we provide industry benchmarks for subscription health and engagement?
- **Predictive Analytics**: What business outcomes can we predict from KPI patterns?
- **Custom Metrics**: Should organizations be able to define custom KPIs for their specific needs?
- **Alert Thresholds**: Should admins be able to set up automated alerts for specific KPI thresholds?

### Performance and Scalability
- **Real-time Updates**: What's the acceptable latency for KPI updates during peak usage?
- **Data Volume**: How do we handle KPI calculations for organizations with 1000+ users?
- **Caching Strategy**: Should we pre-calculate common KPI combinations across organizations?
- **Export Performance**: What's the maximum acceptable time for large KPI exports?

### User Experience and Adoption
- **KPI Onboarding**: How do we help admins understand and effectively use these metrics?
- **Custom Dashboards**: Should admins be able to create custom KPI dashboards?
- **Mobile Optimization**: How do we optimize KPI views for mobile administrative access?
- **Report Scheduling**: Should admins be able to schedule automated KPI reports?

### Data Privacy and Compliance
- **Cross-Organization Insights**: What anonymized insights can we provide for research and development?
- **Data Portability**: What KPI data should be included in organization data exports?
- **Retention Policies**: Should KPI data retention be configurable per organization?
- **Audit Requirements**: What additional audit logging is needed for administrative KPI access?
