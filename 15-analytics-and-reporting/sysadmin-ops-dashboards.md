# SysAdmin Ops Dashboards

## Purpose
This specification defines the comprehensive operational dashboards available to system administrators in the MLC-LMS platform. These dashboards provide real-time visibility into platform health, performance metrics, security monitoring, user activity patterns, and business intelligence across all organizations. The dashboards enable proactive system management, capacity planning, incident response, and data-driven platform optimization while maintaining appropriate security and privacy controls.

## Scope
**Included**
- Real-time system health and performance monitoring
- Cross-organizational usage analytics and capacity planning
- Security monitoring and threat detection dashboards
- Business intelligence and revenue analytics
- User behavior patterns and engagement metrics
- System error tracking and incident management
- Database performance and resource utilization
- API usage patterns and rate limiting monitoring
- Content delivery and CDN performance metrics
- Compliance and audit trail monitoring

**Excluded**
- Individual organization-specific KPIs (see [admin-reports-kpis.md](./admin-reports-kpis.md))
- Teacher and student-facing analytics (see [teacher-reports-kpis.md](./teacher-reports-kpis.md))
- Real-time alerting and notification systems (covered in [observability.md](../19-quality-and-operations/observability.md))
- Individual user detailed session data (privacy-protected)

## Data model (if applicable)

### Core Operational Metrics Entities

#### SystemHealthSnapshot
```json
{
  "snapshot_id": "health_1234567890",
  "timestamp": "2025-01-27T14:30:00Z",
  "overall_health_score": 94.2,
  "api_response_time_avg_ms": 145,
  "database_connection_pool_utilization": 67.3,
  "active_sessions": 2847,
  "concurrent_users": 1923,
  "error_rate_percentage": 0.8,
  "uptime_percentage": 99.97,
  "cdn_hit_rate": 89.4,
  "memory_utilization_percentage": 72.1,
  "cpu_utilization_percentage": 58.7,
  "disk_utilization_percentage": 45.2
}
```

#### PlatformUsageMetrics
```json
{
  "metric_id": "usage_1234567890",
  "period_start": "2025-01-27T00:00:00Z",
  "period_end": "2025-01-27T23:59:59Z",
  "total_organizations": 1247,
  "active_organizations": 1189,
  "total_users": 45678,
  "active_users": 28473,
  "total_games_played": 89234,
  "total_assignments_created": 3456,
  "total_api_calls": 2345678,
  "peak_concurrent_users": 1923,
  "avg_session_duration_minutes": 18.7,
  "bounce_rate_percentage": 12.3
}
```

#### SecurityMonitoringData
```json
{
  "security_id": "sec_1234567890",
  "timestamp": "2025-01-27T14:30:00Z",
  "failed_login_attempts": 23,
  "suspicious_activity_flags": 2,
  "rate_limit_violations": 45,
  "blocked_ips": 3,
  "security_scan_results": {
    "vulnerabilities_high": 0,
    "vulnerabilities_medium": 2,
    "vulnerabilities_low": 7
  },
  "ssl_certificate_expiry_days": 45,
  "audit_log_entries": 1234,
  "data_breach_indicators": 0
}
```

#### BusinessIntelligenceMetrics
```json
{
  "bi_id": "bi_1234567890",
  "period_start": "2025-01-01T00:00:00Z",
  "period_end": "2025-01-31T23:59:59Z",
  "total_revenue": 45678.90,
  "monthly_recurring_revenue": 42345.67,
  "new_subscriptions": 23,
  "subscription_cancellations": 8,
  "net_revenue_retention": 94.2,
  "customer_acquisition_cost": 45.67,
  "lifetime_value_avg": 234.56,
  "churn_rate_percentage": 2.1,
  "upgrade_rate_percentage": 12.3,
  "downgrade_rate_percentage": 3.4
}
```

### Dashboard Configuration Schema

#### DashboardWidget
```json
{
  "widget_id": "widget_12345",
  "widget_type": "metric_card|chart|table|alert_panel",
  "title": "API Response Time",
  "description": "Average API response time over last 24 hours",
  "data_source": "system_metrics",
  "refresh_interval_seconds": 30,
  "chart_type": "line|bar|pie|gauge|heatmap",
  "thresholds": {
    "warning": 200,
    "critical": 500
  },
  "position": {
    "row": 1,
    "column": 1,
    "width": 4,
    "height": 2
  }
}
```

## Behavior and rules

### Dashboard Refresh and Data Freshness
- **Real-time Metrics**: System health and performance data refreshed every 30 seconds
- **Business Metrics**: Revenue and subscription data refreshed every 15 minutes
- **Historical Data**: Usage patterns and trends updated hourly
- **Security Metrics**: Security monitoring data refreshed every 60 seconds
- **Caching Strategy**: Frequently accessed metrics cached for 5 minutes maximum

### Data Aggregation Rules
- **Time Windows**: Default to last 24 hours, configurable to 1h, 6h, 24h, 7d, 30d
- **Rolling Averages**: 5-minute, 1-hour, and 24-hour rolling averages for trend analysis
- **Peak Detection**: Automatic identification of usage peaks and performance bottlenecks
- **Anomaly Detection**: Statistical analysis to identify unusual patterns or potential issues

### Access Control and Security
- **System Admin Only**: Dashboards accessible only to users with system admin role
- **Audit Logging**: All dashboard access and data exports logged for security monitoring
- **Data Anonymization**: Cross-organizational insights anonymized to protect privacy
- **Export Restrictions**: Sensitive operational data requires additional approval for export

### Performance Safeguards
- **Query Timeouts**: Dashboard queries timeout after 30 seconds maximum
- **Data Limits**: Large datasets limited to 100,000 records per query
- **Concurrent Users**: Maximum 50 concurrent dashboard users to maintain performance
- **Resource Monitoring**: Dashboard queries monitored for resource consumption

## UX requirements (if applicable)

### Dashboard Layout and Navigation
- **Main Dashboard**: Executive overview with key system health indicators
- **Performance Tab**: Detailed system performance metrics and capacity planning
- **Security Tab**: Security monitoring, threat detection, and compliance metrics
- **Business Tab**: Revenue analytics, subscription health, and growth metrics
- **Users Tab**: User activity patterns, engagement metrics, and behavior analytics
- **Content Tab**: Content delivery performance, game usage, and content analytics

### Key Dashboard States
- **Loading States**: Skeleton placeholders for data loading (max 5 seconds)
- **Empty States**: Clear messaging when no data available with refresh options
- **Error States**: Detailed error messages with retry mechanisms and support contact
- **Alert States**: Prominent alerts for critical system issues requiring immediate attention
- **Maintenance States**: Clear indicators during planned maintenance windows

### Interactive Features
- **Drill-down Capabilities**: Click-through from high-level metrics to detailed views
- **Time Range Selection**: Flexible time range picker with preset options
- **Filter Controls**: Organization, user type, and geographic filtering options
- **Export Functions**: CSV/JSON export for authorized metrics with audit logging
- **Custom Views**: Saveable dashboard configurations for different operational needs

### Accessibility Requirements
- **Screen Reader Support**: All metrics have descriptive text alternatives and ARIA labels
- **Keyboard Navigation**: Full keyboard accessibility for all interactive elements
- **Color Coding**: High contrast color schemes with alternative indicators for color-blind users
- **Responsive Design**: Optimized for large desktop displays (primary) with tablet support
- **Data Tables**: Sortable, filterable tables with proper headers and captions

## Telemetry

### Dashboard Interaction Events
```javascript
// System admin views dashboard
{
  event_type: "sysadmin_dashboard_viewed",
  properties: {
    admin_id: "usr_sysadmin_123",
    dashboard_tab: "performance",
    time_range: "24_hours",
    widgets_viewed: ["api_response_time", "database_utilization", "error_rate"],
    session_duration_seconds: 180
  }
}

// System admin drills down into metric
{
  event_type: "sysadmin_metric_drilldown",
  properties: {
    admin_id: "usr_sysadmin_123",
    metric_type: "api_performance",
    metric_name: "response_time",
    drilldown_level: "endpoint_breakdown",
    time_range: "6_hours"
  }
}

// System admin exports data
{
  event_type: "sysadmin_data_export",
  properties: {
    admin_id: "usr_sysadmin_123",
    export_type: "performance_metrics",
    export_format: "csv",
    record_count: 5000,
    time_range: "7_days",
    filters_applied: ["api_endpoints", "error_codes"]
  }
}
```

### System Performance Events
```javascript
// Dashboard data refresh
{
  event_type: "dashboard_data_refreshed",
  properties: {
    dashboard_type: "system_health",
    refresh_duration_ms: 1250,
    data_points_updated: 45,
    cache_hit_rate: 0.78
  }
}

// Alert threshold triggered
{
  event_type: "system_alert_triggered",
  properties: {
    alert_type: "high_error_rate",
    metric_name: "api_error_rate",
    threshold_value: 5.0,
    actual_value: 7.2,
    severity: "warning",
    affected_organizations: 12
  }
}
```

## Permissions

### System Admin Role Permissions
- **Full Dashboard Access**: Complete access to all operational dashboards and metrics
- **Real-time Monitoring**: Live system health and performance data
- **Cross-Organization Analytics**: Anonymized insights across all organizations
- **Security Monitoring**: Complete security dashboard and threat detection data
- **Business Intelligence**: Full revenue and subscription analytics
- **Data Export**: Export capabilities for all operational metrics with audit logging
- **Alert Management**: Configure and manage system alerts and thresholds

### Data Privacy and Compliance
- **FERPA Compliance**: Educational data properly anonymized in cross-org analytics
- **COPPA Compliance**: No PII exposure for students under 13 in any dashboard
- **Data Retention**: Operational data retained for 2 years for trend analysis
- **Audit Logging**: All dashboard access and data exports logged for compliance
- **Access Controls**: Multi-factor authentication required for sensitive dashboards

### Export and Sharing Restrictions
- **Sensitive Data**: Security and performance data requires additional approval for export
- **External Sharing**: Cross-organization data cannot be shared externally without anonymization
- **Data Classification**: All exported data properly classified and labeled
- **Retention Policies**: Exported data subject to same retention policies as source data

## Dependencies

### Core System Dependencies
- **Event Model**: Dashboard metrics based on events defined in [event-model.md](./event-model.md)
- **Roles and Permissions**: Access control from [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md)
- **System Architecture**: Performance metrics from [system-overview.md](../18-architecture/system-overview.md)
- **Observability**: Monitoring infrastructure from [observability.md](../19-quality-and-operations/observability.md)

### Data Source Dependencies
- **Admin Reports KPIs**: Business intelligence foundation from [admin-reports-kpis.md](./admin-reports-kpis.md)
- **Learning Analytics**: User behavior patterns from [learning-analytics.md](./learning-analytics.md)
- **Billing System**: Revenue and subscription data from payment providers
- **Security Monitoring**: Threat detection and audit logging systems

### Technical Dependencies
- **Time-Series Database**: High-performance storage for operational metrics
- **Real-time Processing**: Stream processing for live dashboard updates
- **Caching Layer**: Redis for frequently accessed dashboard data
- **Export Service**: Secure data export capabilities with proper formatting
- **Alerting System**: Real-time alerting for critical system issues

### Related Documentation
- [System Admin User Index](../06-system-admin/sysadmin-user-index.md) - User management capabilities
- [System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md) - Audit and compliance monitoring
- [Observability](../19-quality-and-operations/observability.md) - System monitoring and alerting
- [Performance and Scalability](../18-architecture/performance-and-scalability.md) - System performance architecture

## Open questions

### Technical Implementation
- **Real-time Processing**: What's the minimum acceptable latency for real-time dashboard updates?
- **Data Storage**: Should we use a dedicated time-series database or extend existing infrastructure?
- **Caching Strategy**: How do we balance data freshness with performance for large datasets?
- **Mobile Access**: Do system admins need mobile access to operational dashboards?

### Security and Compliance
- **Data Classification**: How do we classify and protect different levels of operational data?
- **External Access**: Should external monitoring tools have access to dashboard data?
- **Audit Requirements**: What additional audit logging is needed for dashboard access?
- **Incident Response**: How do dashboards integrate with incident response workflows?

### Business Intelligence
- **Predictive Analytics**: What operational outcomes can we predict from dashboard metrics?
- **Benchmarking**: Should we provide industry benchmarks for system performance?
- **Custom Dashboards**: Should system admins be able to create custom dashboard views?
- **Alert Thresholds**: How do we determine optimal alert thresholds for different metrics?

### Performance and Scale
- **Dashboard Load**: How many concurrent dashboard users can the system support?
- **Data Volume**: What's the expected data volume for operational metrics at scale?
- **Query Performance**: What's the acceptable query response time for complex dashboards?
- **Resource Impact**: How do we minimize the performance impact of dashboard queries?
