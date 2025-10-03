# Event Model

## Purpose
This specification defines the comprehensive event tracking system for the MLC-LMS platform, enabling detailed analytics, learning insights, and system monitoring. The event model captures user interactions, learning progress, system performance, and business metrics to support data-driven decision making across all user roles and system components.

## Scope
**Included**
- Event taxonomy and categorization system
- Standardized event properties and data schemas
- Real-time and batch event processing requirements
- Integration with existing game model and scoring systems
- Privacy-compliant data collection and retention policies
- Role-based event access and filtering
- Performance monitoring and system health events

**Excluded**
- Specific analytics dashboard implementations (covered in separate KPI specs)
- Data warehouse schema design (covered in architecture docs)
- Third-party analytics tool configurations
- Real-time alerting and notification systems

## Data model (if applicable)

### Event Core Schema
```json
{
  "event_id": "evt_1234567890abcdef",
  "event_type": "game_stage_complete",
  "timestamp": "2025-01-27T14:30:45.123Z",
  "user_id": "usr_12345",
  "session_id": "sess_67890abcdef",
  "organization_id": "org_54321",
  "device_info": {
    "user_agent": "Mozilla/5.0...",
    "platform": "desktop",
    "browser": "chrome",
    "version": "120.0.0.0"
  },
  "context": {
    "page_url": "/student/assignment-play",
    "referrer": "/student/dashboard",
    "assignment_id": "asg_789",
    "sequence_id": "seq_456"
  },
  "properties": {
    "game_id": "G-01542",
    "stage_id": "G-01542-quiz",
    "score": 87,
    "max_score": 100,
    "passed": true,
    "duration_ms": 125000,
    "attempts": 2
  },
  "privacy_level": "standard",
  "retention_days": 2555
}
```

### Event Categories

#### User Interaction Events
- **Authentication**: `user_login`, `user_logout`, `session_timeout`, `password_reset`
- **Navigation**: `page_view`, `page_exit`, `menu_click`, `breadcrumb_click`
- **Content Access**: `game_launch`, `sequence_view`, `assignment_open`, `resource_download`
- **Communication**: `message_sent`, `message_read`, `notification_received`

#### Learning Progress Events
- **Gameplay**: `game_stage_start`, `game_stage_complete`, `game_stage_quit`, `target_met`
- **Assignments**: `assignment_started`, `assignment_completed`, `assignment_overdue`
- **Mastery**: `concept_mastered`, `review_scheduled`, `retention_check`
- **Challenges**: `challenge_joined`, `challenge_completed`, `leaderboard_update`

#### System Performance Events
- **Performance**: `page_load_time`, `api_response_time`, `asset_load_time`
- **Errors**: `javascript_error`, `api_error`, `game_error`, `network_error`
- **Health**: `service_health_check`, `database_connection`, `cdn_status`

#### Business Intelligence Events
- **Subscription**: `subscription_created`, `subscription_updated`, `payment_processed`
- **Usage**: `seat_utilization`, `feature_adoption`, `retention_metrics`
- **Support**: `support_ticket_created`, `support_ticket_resolved`

### Event Properties Schema

#### Standard Properties (All Events)
```json
{
  "event_id": "string (UUID)",
  "event_type": "string (camelCase)",
  "timestamp": "ISO 8601 datetime",
  "user_id": "string (nullable for anonymous events)",
  "session_id": "string (nullable for system events)",
  "organization_id": "string (nullable for system events)",
  "device_info": {
    "user_agent": "string",
    "platform": "desktop|mobile|tablet",
    "browser": "string",
    "version": "string",
    "screen_resolution": "string",
    "timezone": "string"
  },
  "context": {
    "page_url": "string",
    "referrer": "string (nullable)",
    "assignment_id": "string (nullable)",
    "sequence_id": "string (nullable)",
    "class_id": "string (nullable)",
    "teacher_id": "string (nullable)"
  },
  "privacy_level": "standard|sensitive|pii",
  "retention_days": "integer"
}
```

#### Game-Specific Properties
```json
{
  "game_id": "string (G-XXXXX format)",
  "stage_id": "string (G-XXXXX-stage format)",
  "stage_type": "learn|play|quiz|challenge|review",
  "score": "integer (0-100)",
  "max_score": "integer",
  "passed": "boolean",
  "duration_ms": "integer",
  "attempts": "integer",
  "mistake_count": "integer (nullable)",
  "input_mode": "mouse|keyboard|midi|touch",
  "device_profile": "any|midi_required|mic_optional|mic_required"
}
```

#### Learning Analytics Properties
```json
{
  "concept_tags": ["string"],
  "difficulty_level": "integer (1-10)",
  "mastery_status": "not_started|in_progress|mastered|reviewed",
  "time_to_mastery_ms": "integer (nullable)",
  "retention_score": "integer (nullable)",
  "adaptive_difficulty": "integer (nullable)",
  "learning_style": "visual|aural|kinesthetic|mixed"
}
```

## Behavior and rules

### Event Collection Rules
- **Sampling**: System performance events sampled at 10% in production, learning events never sampled
- **Batching**: Events batched every 30 seconds or 100 events, whichever comes first
- **Retry Logic**: Failed event submissions retried up to 3 times with exponential backoff
- **Validation**: All events validated against schema before storage
- **Deduplication**: Duplicate events (same event_id) rejected within 24-hour window

### Privacy and Compliance
- **COPPA Compliance**: No PII collected for users under 13 without parental consent
- **FERPA Compliance**: Educational records properly anonymized and access-controlled
- **Data Retention**: Standard events retained for 7 years, sensitive events for 3 years
- **Anonymization**: User IDs hashed for cross-organization analytics
- **Consent Tracking**: User consent status tracked per event category

### Event Processing Pipeline
1. **Collection**: Events captured at source (browser, mobile app, server)
2. **Validation**: Schema validation and privacy compliance checks
3. **Enrichment**: Add contextual data (user role, subscription level, etc.)
4. **Routing**: Route to appropriate processing streams (real-time, batch, alerts)
5. **Storage**: Store in time-series database with appropriate partitioning
6. **Aggregation**: Pre-compute common metrics and KPIs
7. **Access**: Role-based access controls for event data retrieval

## UX requirements (if applicable)

### Event Collection UX
- **Performance Impact**: Event collection must not impact user experience (< 50ms overhead)
- **Offline Support**: Events queued locally when offline, synced when connection restored
- **Consent Management**: Clear opt-in/opt-out controls for analytics data collection
- **Transparency**: Users can view their event history and data usage

### Analytics Dashboard Requirements
- **Real-time Updates**: Critical metrics update within 5 seconds
- **Role-based Views**: Different data views for different user roles
- **Export Capabilities**: CSV/JSON export for authorized users
- **Drill-down Navigation**: From high-level metrics to detailed event streams

## Telemetry

### Event Types and Triggers

#### User Authentication Events
```javascript
// User login
{
  event_type: "user_login",
  properties: {
    login_method: "password|sso|oauth",
    mfa_used: boolean,
    session_duration_estimate: integer
  }
}

// User logout
{
  event_type: "user_logout",
  properties: {
    session_duration_ms: integer,
    logout_reason: "user_initiated|timeout|forced"
  }
}
```

#### Gameplay Events
```javascript
// Game stage start
{
  event_type: "game_stage_start",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    stage_type: "quiz",
    assignment_id: "asg_789",
    sequence_id: "seq_456",
    device_profile: "midi_required",
    input_mode: "midi"
  }
}

// Game stage completion
{
  event_type: "game_stage_complete",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    score: 87,
    max_score: 100,
    passed: true,
    duration_ms: 125000,
    attempts: 2,
    mistake_count: 3,
    mastery_achieved: true
  }
}

// Target achievement
{
  event_type: "target_met",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    target_score: 85,
    actual_score: 87,
    time_to_target_ms: 125000,
    attempts_to_target: 2
  }
}
```

#### Learning Progress Events
```javascript
// Concept mastery
{
  event_type: "concept_mastered",
  properties: {
    game_id: "G-01542",
    concept_tags: ["guide_notes", "treble_staff"],
    mastery_level: "quiz_passed",
    time_to_mastery_ms: 125000,
    total_attempts: 2,
    learning_sequence_position: 15
  }
}

// Review scheduling
{
  event_type: "review_scheduled",
  properties: {
    game_id: "G-01542",
    sequence_id: "seq_456",
    review_date: "2025-02-03T14:30:00Z",
    offset_days: 7,
    retention_confidence: 0.85
  }
}
```

#### System Performance Events
```javascript
// Page load performance
{
  event_type: "page_load_time",
  properties: {
    page_url: "/student/dashboard",
    load_time_ms: 1250,
    dom_ready_ms: 800,
    first_paint_ms: 600,
    largest_contentful_paint_ms: 1100,
    cumulative_layout_shift: 0.05
  }
}

// API performance
{
  event_type: "api_response_time",
  properties: {
    endpoint: "/api/v1/assignments",
    method: "GET",
    status_code: 200,
    response_time_ms: 150,
    payload_size_bytes: 2048
  }
}
```

### Event Sampling and Filtering
- **Critical Events**: Never sampled (user actions, learning progress, errors)
- **Performance Events**: 10% sampling in production, 100% in development
- **Debug Events**: Only collected in development and staging environments
- **High-Volume Events**: Page views and mouse movements sampled at 1%

## Permissions

### Event Data Access by Role

#### Subscriber-Admin
- **Full Organization Access**: All events for their organization
- **Aggregated Analytics**: Cross-teacher and cross-student insights
- **Billing Events**: Complete subscription and payment event history
- **Export Capabilities**: Full data export for their organization

#### Teacher-Admin
- **Scope-Limited Access**: Events for teachers and students under their management
- **Class Analytics**: Aggregated data for their assigned classes
- **Teacher Performance**: Events related to teacher activities in their scope
- **Limited Export**: Data export limited to their scope

#### Teacher
- **Student Events**: Learning progress and game events for their students
- **Assignment Events**: Events related to assignments they create
- **Personal Events**: Their own interaction events
- **No Cross-Teacher Access**: Cannot see other teachers' student data

#### Student
- **Personal Events Only**: Their own learning progress and interaction events
- **Privacy-Protected**: No access to other students' data
- **Learning Insights**: Personal analytics and progress tracking
- **Export Own Data**: Can export their personal learning data

#### System Admin
- **Global Access**: All events across all organizations
- **System Events**: Platform performance and health events
- **Audit Events**: Security and compliance event monitoring
- **Full Export**: Complete system-wide data export capabilities

### Event Data Retention and Deletion
- **Standard Events**: 7-year retention for compliance and analytics
- **Sensitive Events**: 3-year retention with enhanced security
- **PII Events**: Immediate anonymization, 1-year retention
- **System Events**: 1-year retention for operational monitoring
- **Automatic Deletion**: Events automatically purged after retention period

## Dependencies

### Core System Dependencies
- **Game Model**: Event properties must align with game and stage definitions
- **Roles and Permissions**: Event access controlled by role-based permissions
- **User Management**: User and organization context for event attribution
- **Assignment Engine**: Learning progress events tied to assignment system

### Technical Dependencies
- **Time-Series Database**: High-performance storage for event data
- **Message Queue**: Reliable event processing and batching
- **CDN Integration**: Asset loading and performance event collection
- **Analytics Pipeline**: Real-time and batch processing infrastructure

### Integration Dependencies
- **Learning Analytics Framework**: Advanced analytics and xAPI implementation
- **Reporting Systems**: KPI dashboards and report generation
- **Notification System**: Event-driven alerts and communications
- **Audit Logging**: Security and compliance event tracking

### Related Documentation
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game and stage definitions
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Permission and access control
- [Learning Analytics Framework](./learning-analytics.md) - Advanced analytics capabilities
- [Teacher Reports KPIs](./teacher-reports-kpis.md) - Teacher-facing analytics
- [Admin Reports KPIs](./admin-reports-kpis.md) - Administrative analytics

## Open questions

### Technical Implementation
- **Event Storage**: Should we use a dedicated time-series database or extend existing PostgreSQL?
- **Real-time Processing**: What's the minimum acceptable latency for real-time analytics?
- **Data Warehouse**: Do we need a separate data warehouse for complex analytics queries?
- **Mobile Events**: How do we handle event collection in offline mobile scenarios?

### Privacy and Compliance
- **Cross-Organization Analytics**: Can we provide anonymized insights across organizations?
- **Parental Consent**: How do we handle event collection for students under 13?
- **Data Portability**: What's the minimum data export format for user data portability?
- **Retention Policies**: Should retention periods be configurable per organization?

### Business Intelligence
- **Predictive Analytics**: What learning outcomes can we predict from event patterns?
- **A/B Testing**: How do we integrate event tracking with feature flag experiments?
- **Custom Metrics**: Should organizations be able to define custom event types?
- **Third-party Integration**: What events should be shared with external analytics tools?

### Performance and Scale
- **Event Volume**: What's the expected event volume at peak usage?
- **Processing Capacity**: How many events per second can our system handle?
- **Storage Costs**: What's the cost model for long-term event data retention?
- **Query Performance**: What's the acceptable query response time for analytics dashboards?
