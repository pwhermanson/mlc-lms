# xAPI (Experience API) Implementation

## Overview
Experience API (xAPI) implementation specifications for advanced learning analytics and interoperability.

## Description
Comprehensive xAPI implementation framework for tracking learning experiences, enabling advanced analytics, and supporting interoperability with other learning systems and tools.

## Purpose

This specification defines the technical implementation of xAPI (Experience API / Tin Can API) within the MLC-LMS platform. xAPI is a standardized specification for learning technology that enables the collection and sharing of learning activity data in a consistent format. 

**Why xAPI for MLC**: 
- **Learning Record Portability**: Student learning records can be exported and used in other educational systems
- **Research and Analytics**: Enables educational research and cross-institutional learning analytics studies
- **Third-party Integration**: Allows MLC data to flow into institutional Learning Record Stores (LRS) or analytics platforms
- **Future-proofing**: Positions MLC as a standards-compliant educational tool that integrates into broader learning ecosystems
- **Advanced Analytics Foundation**: Provides standardized data layer for predictive analytics and machine learning (see [Learning Analytics Framework](../15-analytics-and-reporting/learning-analytics.md))

**xAPI vs Internal Events**: MLC maintains two parallel tracking systems:
- **Internal Event Model** ([event-model.md](../15-analytics-and-reporting/event-model.md)): Optimized for real-time platform operations, detailed debugging, and MLC-specific features
- **xAPI Statements**: Standardized format for external consumption, research, and interoperability

This document focuses on the technical implementation of xAPI generation, storage, and export. For xAPI statement schemas and analytics use cases, see [Learning Analytics Framework](../15-analytics-and-reporting/learning-analytics.md).

## Scope

### Included

**xAPI Statement Generation**:
- Real-time statement generation from learning events
- Statement transformation from internal event model to xAPI format
- Compliance with xAPI 1.0.3 specification
- Support for standard ADL verbs and MLC-specific verb extensions
- Context preservation (assignment, sequence, class hierarchies)
- Attachments support for MIDI recordings or performance artifacts (future)

**Learning Record Store (LRS) Integration**:
- Internal LRS implementation for statement storage
- External LRS connection capabilities (Learning Locker, Watershed, institutional LRS)
- Statement forwarding and sync between internal and external LRS
- Query API for statement retrieval
- Authentication and authorization for LRS access
- Batch statement submission and retry logic

**xAPI Endpoints and API**:
- REST API endpoints compliant with xAPI specification
- Statement POST/PUT/GET endpoints
- Query interface with filtering and pagination
- About endpoint for LRS metadata
- State API for activity state tracking (optional)
- Agent Profile API for learner preference storage (optional)

**Statement Management**:
- Statement validation against xAPI schema
- Duplicate detection and idempotency
- Statement voiding and correction workflows
- Conflict resolution for statement updates
- Long-term statement storage and archival
- Statement export formats (JSON, CSV for analysis)

**Privacy and Compliance**:
- PII anonymization in cross-organization statements
- FERPA/COPPA-compliant statement generation
- Consent management for xAPI data sharing
- Configurable statement retention policies
- Right to deletion support for student data

**Configuration and Administration**:
- System Admin interface for LRS configuration
- Per-organization xAPI settings (enable/disable, external LRS URLs)
- Statement generation rules and filters
- Monitoring and diagnostics for statement pipeline
- Error handling and retry management

### Excluded

**Advanced Analytics Dashboards**: xAPI provides raw data; analytics visualization covered in [Learning Analytics Framework](../15-analytics-and-reporting/learning-analytics.md) and KPI documents.

**Predictive Models**: Machine learning models that consume xAPI data are specified in [Learning Analytics Framework](../15-analytics-and-reporting/learning-analytics.md).

**Real-time Dashboards**: While xAPI statements are generated in real-time, dashboard implementations use internal event streams for lower latency.

**Caliper Analytics**: IMS Caliper is an alternative learning analytics standard. This spec focuses solely on xAPI; Caliper support is out of scope.

**CMI5 Profile**: CMI5 is an xAPI profile for launched content (successor to SCORM). While related to [SCORM Compliance](./scorm-compliance.md), CMI5 implementation is deferred to future phases.

**Video Interaction Tracking**: The xAPI Video Profile for detailed video interaction tracking is out of scope for initial implementation.

**Game Mechanics Profile**: While applicable to MLC's game-based learning, implementing the xAPI Serious Games Profile is optional and not required for core functionality.

## Data model (if applicable)

### xAPI Statement Core Structure

For detailed xAPI statement schemas and examples specific to MLC learning activities, see [Learning Analytics Framework - xAPI Statement Schema](../15-analytics-and-reporting/learning-analytics.md#xapi-statement-schema).

### Internal xAPI Statement Storage

#### xapi_statements Table
Primary storage for all generated xAPI statements.

```sql
CREATE TABLE xapi_statements (
  id UUID PRIMARY KEY,
  statement_id UUID UNIQUE NOT NULL, -- xAPI statement ID
  actor_id VARCHAR(255) NOT NULL, -- Actor identifier (user_id)
  verb_id VARCHAR(255) NOT NULL, -- Verb IRI
  object_id VARCHAR(255) NOT NULL, -- Activity/object IRI
  object_type VARCHAR(50) NOT NULL, -- Activity, Agent, SubStatement
  
  statement_json JSONB NOT NULL, -- Full xAPI statement
  
  timestamp TIMESTAMPTZ NOT NULL,
  stored TIMESTAMPTZ DEFAULT NOW(),
  
  organization_id VARCHAR(100), -- MLC organization
  context_registration UUID, -- xAPI registration/session ID
  
  voided BOOLEAN DEFAULT FALSE,
  voiding_statement_id UUID, -- References another statement if voided
  
  privacy_level VARCHAR(50) DEFAULT 'standard', -- standard, anonymized, pii
  
  -- Indexes for common queries
  INDEX idx_actor (actor_id),
  INDEX idx_verb (verb_id),
  INDEX idx_object (object_id),
  INDEX idx_timestamp (timestamp DESC),
  INDEX idx_organization (organization_id),
  INDEX idx_registration (context_registration),
  INDEX idx_statement_json USING gin(statement_json)
);
```

#### xapi_lrs_configurations Table
Stores LRS connection settings for internal and external LRS instances.

```sql
CREATE TABLE xapi_lrs_configurations (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  lrs_type VARCHAR(50) NOT NULL, -- internal, external
  
  organization_id VARCHAR(100), -- NULL for global/internal LRS
  
  endpoint_url VARCHAR(500), -- Base URL for xAPI endpoints
  auth_type VARCHAR(50), -- basic, oauth, api_key
  auth_credentials_encrypted TEXT, -- Encrypted auth details
  
  forward_statements BOOLEAN DEFAULT FALSE, -- Auto-forward to this LRS
  
  status VARCHAR(50) DEFAULT 'active', -- active, suspended, error
  last_sync_at TIMESTAMPTZ,
  last_error TEXT,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  
  INDEX idx_organization (organization_id),
  INDEX idx_status (status)
);
```

#### xapi_statement_queue Table
Queue for async statement processing and external LRS forwarding.

```sql
CREATE TABLE xapi_statement_queue (
  id UUID PRIMARY KEY,
  statement_id UUID NOT NULL REFERENCES xapi_statements(id),
  lrs_configuration_id UUID REFERENCES xapi_lrs_configurations(id),
  
  operation VARCHAR(50) NOT NULL, -- store, forward, void
  
  status VARCHAR(50) DEFAULT 'pending', -- pending, processing, success, failed
  retry_count INTEGER DEFAULT 0,
  max_retries INTEGER DEFAULT 5,
  
  error_message TEXT,
  
  queued_at TIMESTAMPTZ DEFAULT NOW(),
  processed_at TIMESTAMPTZ,
  
  INDEX idx_status_queued (status, queued_at),
  INDEX idx_statement (statement_id)
);
```

#### xapi_verb_registry Table
Registry of standard and custom xAPI verbs used by MLC.

```sql
CREATE TABLE xapi_verb_registry (
  id UUID PRIMARY KEY,
  verb_iri VARCHAR(500) UNIQUE NOT NULL, -- Full IRI
  verb_type VARCHAR(50) NOT NULL, -- standard_adl, mlc_custom
  
  display_en_us VARCHAR(255) NOT NULL,
  display_translations JSONB, -- Other language displays
  
  description TEXT,
  usage_notes TEXT,
  
  created_at TIMESTAMPTZ DEFAULT NOW(),
  
  INDEX idx_verb_iri (verb_iri)
);
```

### xAPI Activity Types

MLC uses standard xAPI activity types plus custom extensions:

**Standard Activity Types**:
- `http://adlnet.gov/expapi/activities/assessment` - Quiz and Review stages
- `http://adlnet.gov/expapi/activities/lesson` - Learn stages
- `http://adlnet.gov/expapi/activities/simulation` - Play stages
- `http://adlnet.gov/expapi/activities/course` - Sequences
- `http://adlnet.gov/expapi/activities/module` - Assignments

**MLC Custom Activity Types**:
- `https://musiclearningcommunity.com/xapi/activity-types/music-game` - Music learning game
- `https://musiclearningcommunity.com/xapi/activity-types/game-stage` - Game stage (learn/play/quiz/review)
- `https://musiclearningcommunity.com/xapi/activity-types/challenge` - Challenge competition
- `https://musiclearningcommunity.com/xapi/activity-types/sequence` - Learning sequence

### MLC-Specific xAPI Verb Registry

For the complete list of MLC xAPI verbs and their usage, see [Learning Analytics Framework - MLC-Specific xAPI Verbs](../15-analytics-and-reporting/learning-analytics.md#mlc-specific-xapi-verbs).

**Standard ADL Verbs Used**:
- `http://adlnet.gov/expapi/verbs/completed` - Quiz/Review completion
- `http://adlnet.gov/expapi/verbs/passed` - Successful quiz (target score met)
- `http://adlnet.gov/expapi/verbs/failed` - Quiz below target
- `http://adlnet.gov/expapi/verbs/attempted` - Game stage started
- `http://adlnet.gov/expapi/verbs/experienced` - Play stage usage
- `http://adlnet.gov/expapi/verbs/scored` - Score achieved

**MLC Custom Verbs**:
- `https://musiclearningcommunity.com/xapi/verbs/mastered` - Concept mastery achieved
- `https://musiclearningcommunity.com/xapi/verbs/reviewed` - Retention review completed
- `https://musiclearningcommunity.com/xapi/verbs/practiced` - Practice (Play) stage usage
- `https://musiclearningcommunity.com/xapi/verbs/challenged` - Challenge accepted/completed
- `https://musiclearningcommunity.com/xapi/verbs/explored` - Free play game selection

## Behavior and rules

### Statement Generation Pipeline

**1. Internal Event Capture**
- Learning events captured via standard [Event Model](../15-analytics-and-reporting/event-model.md)
- Events include: `game_stage_complete`, `assignment_completed`, `concept_mastered`, etc.
- Event stored in internal event database first (source of truth)

**2. Statement Transformation**
- Background job processes internal events and generates xAPI statements
- Transformation logic maps internal event properties to xAPI format:
  - `event_type` → `verb` (using verb registry)
  - `game_id`, `stage_id` → `object` (activity)
  - `user_id` → `actor` (agent)
  - `assignment_id`, `sequence_id` → `context` (grouping/parent)
  - `score`, `duration_ms`, etc. → `result`
- Apply privacy rules: anonymize if cross-organization analytics
- Generate unique `statement_id` (UUID v4)
- Add MLC-specific extensions to preserve full fidelity

**3. Statement Validation**
- Validate against xAPI 1.0.3 JSON schema
- Required fields check: `id`, `actor`, `verb`, `object`, `timestamp`
- IRI format validation for verbs and activity types
- Timestamp format validation (ISO 8601)
- Score range validation (scaled 0-1, raw within min/max)
- Reject invalid statements, log error for debugging

**4. Statement Storage**
- Insert into `xapi_statements` table
- Full statement stored as JSONB for flexibility
- Indexed fields extracted for fast queries
- Duplicate detection: reject if `statement_id` already exists
- Return 200 OK with stored statement

**5. Statement Forwarding (if configured)**
- If organization has external LRS configured with `forward_statements: true`
- Create entry in `xapi_statement_queue` with status `pending`
- Background worker picks up queue entries
- POST statement to external LRS via xAPI API
- Handle success (200/204) or retry on failure

**6. Statement Availability**
- Statement immediately queryable via internal xAPI API
- Available for analytics processing (feeding into predictive models)
- Exportable via admin interface or bulk API

### Statement Transformation Rules

**Event-to-Verb Mapping**:
```javascript
const EVENT_TO_VERB_MAP = {
  'game_stage_complete': {
    quiz: 'http://adlnet.gov/expapi/verbs/completed',
    review: 'https://musiclearningcommunity.com/xapi/verbs/reviewed',
    play: 'https://musiclearningcommunity.com/xapi/verbs/practiced',
    learn: 'http://adlnet.gov/expapi/verbs/experienced'
  },
  'target_met': 'http://adlnet.gov/expapi/verbs/passed',
  'concept_mastered': 'https://musiclearningcommunity.com/xapi/verbs/mastered',
  'assignment_completed': 'http://adlnet.gov/expapi/verbs/completed',
  'challenge_joined': 'https://musiclearningcommunity.com/xapi/verbs/challenged'
};
```

**Activity ID Construction**:
- Games: `https://musiclearningcommunity.com/games/{game_id}`
- Game Stages: `https://musiclearningcommunity.com/games/{game_id}/{stage_type}`
- Sequences: `https://musiclearningcommunity.com/sequences/{sequence_id}`
- Assignments: `https://musiclearningcommunity.com/assignments/{assignment_id}`

**Actor Identification**:
- Use stable MLC `user_id` as actor identifier
- Format: `{"account": {"homePage": "https://musiclearningcommunity.com", "name": "usr_12345"}}`
- Never use email or name in actor (privacy)
- For anonymized statements: hash user_id with organization salt

**Context Building**:
- **Parent**: Immediate parent activity (game for stage, sequence for game)
- **Grouping**: Higher-level activities (assignment, sequence, class)
- **Registration**: Unique per assignment attempt or session
- **Instructor**: Teacher `user_id` if applicable
- **Extensions**: MLC-specific context (assignment_id, sequence_id, class_id, teacher_id)

**Result Calculation**:
- **score.raw**: Actual numeric score (0-100 for MLC)
- **score.max**: Maximum possible score (100)
- **score.min**: Minimum score (0)
- **score.scaled**: Normalized 0-1 (raw / max)
- **success**: Boolean, true if `passed` target score
- **completion**: Boolean, true if stage/activity completed
- **duration**: ISO 8601 duration format (PT2M5S for 2 minutes 5 seconds)
- **extensions**: Additional MLC data (attempts, mistakes, input_mode, etc.)

### Statement Validation Rules

**Required Field Validation**:
- `id`: Must be UUID format
- `actor`: Must have either `mbox`, `mbox_sha1sum`, `openid`, or `account`
- `verb`: Must have `id` (IRI) and `display` (language map)
- `object`: Must have `id` and `objectType`
- `timestamp`: Must be valid ISO 8601 datetime

**IRI Format Validation**:
- All verb and activity type IRIs must be valid absolute URLs
- MLC custom IRIs must use `https://musiclearningcommunity.com/xapi/` namespace
- No relative URLs or invalid characters

**Score Validation**:
- If `score.scaled` present, must be 0-1
- If `score.raw` present, must be between `score.min` and `score.max`
- If `success` true, `score.raw` should meet or exceed documented target score

**Timestamp Validation**:
- Must not be in the future (allow 5-minute clock skew tolerance)
- Must be within reasonable past range (not older than account creation)
- Use ISO 8601 format with timezone (UTC preferred)

### External LRS Integration Rules

**Authentication Methods**:
- **Basic Auth**: Username/password for xAPI endpoints (most common)
- **OAuth 2.0**: Client credentials or bearer token (if LRS supports)
- **API Key**: Custom header authentication (vendor-specific)

**Statement Forwarding Behavior**:
- **Real-time Forwarding**: Queue immediately after internal storage
- **Batch Forwarding**: Collect statements, send in batches every 5 minutes (up to 100 statements per batch)
- **Selective Forwarding**: Filter by event type, user role, or organization policy
- **Retry Logic**: Exponential backoff - 1 min, 5 min, 15 min, 1 hour, 4 hours (max 5 attempts)
- **Error Handling**: Log failure, alert admin if error rate exceeds 5%

**LRS Compatibility**:
- Detect LRS platform via About endpoint
- Apply platform-specific quirks or workarounds if needed
- Fallback to strict xAPI 1.0.3 spec if platform unknown

**Statement Consistency**:
- Statements forwarded to external LRS must match internal storage exactly
- Use same `statement_id` for consistency
- If external LRS rejects, log but don't delete internal statement

### Privacy and Anonymization Rules

**Cross-Organization Statements**:
- When generating statements for cross-organization analytics or research:
  - Replace `user_id` with HMAC hash: `HMAC-SHA256(user_id, global_salt)`
  - Omit `organization_id` from context
  - Remove any PII from extensions (names, emails)
- Store both identified and anonymized versions if needed

**COPPA Compliance** (users under 13):
- Require parental consent before generating xAPI statements
- If consent not granted: generate statements but mark `privacy_level: restricted`
- Restricted statements never forwarded to external LRS
- Restricted statements excluded from cross-organization analytics

**FERPA Compliance**:
- Student learning records classified as educational records
- xAPI statements with PII require same protections as internal data
- Consent required before sharing statements with third parties
- Right to access: students/parents can request all xAPI statements about them

**Statement Deletion**:
- If user exercises right to deletion:
  - Mark statements as `voided` with voiding statement
  - Optionally purge after grace period
  - Notify external LRS to void or delete (if supported)

### Query API Rules

**GET /statements Endpoint**:
- Filter parameters: `agent`, `verb`, `activity`, `registration`, `since`, `until`, `limit`
- Return statements in reverse chronological order (most recent first)
- Pagination via `limit` (max 100) and `more` token
- Consistent results: query must return deterministic set

**Permissions**:
- Students: can query only their own statements
- Teachers: can query statements for their assigned students
- Subscriber-Admin: can query all statements for their organization
- System-Admin: can query all statements globally
- Researchers (future): can query anonymized statements with IRB approval

**Performance**:
- Use database indexes on `actor_id`, `verb_id`, `object_id`, `timestamp`
- Cache common queries (e.g., recent statements for a student)
- Limit query complexity to prevent expensive joins

## UX requirements (if applicable)

### System Admin xAPI Configuration

**LRS Management Interface**:
- "xAPI / Learning Record Stores" section in System Admin panel
- List of configured LRS instances (internal + external)
- "Add External LRS" button to configure third-party LRS integration

**LRS Configuration Form**:
- **LRS Name**: Display name (e.g., "District LRS", "Watershed Analytics")
- **Endpoint URL**: Base xAPI endpoint (e.g., `https://lrs.example.com/xapi/`)
- **Authentication**: Type selection (Basic, OAuth, API Key) with credential inputs
- **Organization**: Select organization or "All Organizations"
- **Forward Statements**: Toggle to enable auto-forwarding
- **Test Connection**: Button to validate credentials and endpoint

**LRS Status Dashboard**:
- Connection status indicator (green/yellow/red)
- Last successful sync timestamp
- Statement queue depth
- Error count in last 24 hours
- "View Logs" link to detailed sync history

**Statement Export Interface**:
- Date range picker
- Filter by organization, user, verb, activity
- Export format: JSON (xAPI native), CSV (for analysis)
- "Download" button generates export file
- Large exports queued and emailed when ready

### Teacher xAPI Data Visibility

**Student Profile xAPI Tab** (optional):
- "Learning Records" tab in student profile
- Timeline view of major learning milestones (mastery, completions)
- Filter by activity type, date range
- "Export Student Records" button (generates xAPI JSON)
- Privacy notice: explains what data is collected and shared

### Student/Parent Data Access

**My Learning Data Page** (FERPA compliance):
- Student/parent can view all xAPI statements about them
- Plain-language explanations of what each statement means
- "Download My Data" button (JSON format)
- "Delete My Data" request form (triggers voiding process)

### Accessibility

- All xAPI configuration interfaces meet WCAG 2.1 AA standards
- Statement timeline views accessible via keyboard navigation
- Screen reader support for statement descriptions
- High contrast mode for status indicators

## Telemetry

### xAPI System Events

**`xapi_statement_generated`**
- **When**: xAPI statement successfully generated from internal event
- **Properties**:
  - `statement_id`: xAPI statement ID
  - `verb_iri`: Verb IRI used
  - `activity_type`: Activity type
  - `user_id`: Actor MLC user ID
  - `organization_id`: Organization
  - `transformation_duration_ms`: Time to transform event to statement

**`xapi_statement_forwarded`**
- **When**: Statement successfully forwarded to external LRS
- **Properties**:
  - `statement_id`: xAPI statement ID
  - `lrs_configuration_id`: Target LRS
  - `endpoint_url`: External LRS URL
  - `response_time_ms`: HTTP request duration
  - `retry_count`: Number of attempts (1 if first try succeeded)

**`xapi_statement_forward_failed`**
- **When**: Statement forwarding fails after all retries
- **Properties**:
  - `statement_id`: xAPI statement ID
  - `lrs_configuration_id`: Target LRS
  - `error_type`: `auth_error`, `network_error`, `server_error`, `validation_error`
  - `error_message`: Detailed error
  - `total_attempts`: Number of retry attempts

**`xapi_query_executed`**
- **When**: xAPI statement query executed (GET /statements)
- **Properties**:
  - `user_id`: Requesting user
  - `filters`: Query parameters used
  - `result_count`: Number of statements returned
  - `query_duration_ms`: Query execution time
  - `cache_hit`: Boolean, true if served from cache

**`xapi_lrs_connection_test`**
- **When**: Admin tests external LRS connection
- **Properties**:
  - `lrs_configuration_id`: Target LRS
  - `success`: Boolean
  - `response_time_ms`: Connection test duration
  - `error_message`: If failed, error details
  - `tested_by`: System Admin user ID

## Permissions

Access control for xAPI features follows the platform's [Roles Matrix](../02-roles-and-permissions/roles-matrix.md).

### xAPI Configuration and Management

**Configure External LRS**: System Admin only
**View xAPI Configuration**: System Admin, Subscriber-Admin (own organization)
**Test LRS Connection**: System Admin, Subscriber-Admin (own organization)
**View Statement Logs**: System Admin (all), Subscriber-Admin (own org), Teacher-Admin (own scope)
**Export Statements**: System Admin (all), Subscriber-Admin (own org), Teacher (own students)

### Statement Querying (xAPI API)

**Query Own Statements**: Student (own learning records)
**Query Student Statements**: Teacher (assigned students), Teacher-Admin (scope), Subscriber-Admin (organization)
**Query All Statements**: System Admin only
**Anonymized Query Access**: Researcher role (future), requires IRB approval

### Statement Generation Control

**Enable/Disable xAPI per Organization**: System Admin, Subscriber-Admin
**Opt Out of xAPI Tracking**: Student/parent can request (privacy setting)
**Void Statements**: System Admin only (for corrections or deletions)

### External LRS Forwarding

**Enable Statement Forwarding**: System Admin, Subscriber-Admin (own organization)
**Configure Forwarding Rules**: System Admin, Subscriber-Admin (own organization)
**Retry Failed Forwards**: System Admin, Subscriber-Admin (own organization)

## Dependencies

### Core System Dependencies

**Event Model** ([event-model.md](../15-analytics-and-reporting/event-model.md)):
- xAPI statements generated from internal events
- Event properties mapped to xAPI statement fields
- Event sampling rules determine which events generate statements

**Learning Analytics Framework** ([learning-analytics.md](../15-analytics-and-reporting/learning-analytics.md)):
- Defines xAPI statement schemas and examples for MLC activities
- Specifies xAPI verb registry and custom extensions
- Documents analytics use cases for xAPI data

**Game Model** ([game-model.md](../08-games-registry-and-authoring/game-model.md)):
- Game metadata (name, description, concepts) used in activity definitions
- Stage structure mapped to xAPI activity types
- Scoring rules determine statement result values

**Assignment Model** ([assignment-model.md](../07-content-architecture/assignment-model.md)):
- Assignment context included in xAPI statements
- Assignment completion triggers xAPI statement generation
- Assignment hierarchy (grouping) reflected in statement context

**Roles and Permissions** ([roles-matrix.md](../02-roles-and-permissions/roles-matrix.md)):
- Role-based access to xAPI query API
- Permission checks for statement export and configuration
- Teacher-student relationships determine query scope

### Technical Infrastructure Dependencies

**Background Job Queue** ([background-jobs.md](../18-architecture/background-jobs.md)):
- Async statement generation from events
- Statement forwarding to external LRS
- Batch export processing

**API Endpoints** ([rest-endpoints.md](../18-architecture/rest-endpoints.md)):
- xAPI-compliant REST API implementation
- Statement POST/GET endpoints
- Query interface with filtering

**Database**:
- PostgreSQL with JSONB support for statement storage
- Indexes on common query fields
- Foreign keys to users, games, assignments

**Caching Layer** ([caching-strategy.md](../18-architecture/caching-strategy.md)):
- Cache recent statements for faster queries
- Cache verb registry to avoid repeated lookups
- Cache external LRS credentials (encrypted)

**Security and Privacy** ([security-privacy.md](../18-architecture/security-privacy.md)):
- Statement anonymization for cross-organization analytics
- Encryption of external LRS credentials
- HTTPS enforcement for all xAPI endpoints
- COPPA/FERPA compliance in statement generation

### External Dependencies

**xAPI Specification 1.0.3** (ADL / IEEE):
- Full compliance with xAPI standard
- Statement format, validation rules, API endpoints

**External Learning Record Stores**:
- Optional integration with third-party LRS platforms
- Common platforms: Learning Locker, Watershed, Veracity LRS, institutional LRS

**xAPI Profiles**:
- xAPI Video Profile (future, for video interaction tracking)
- xAPI Serious Games Profile (optional, for game mechanics)
- cmi5 Profile (future, for launched content compatibility)

### Related Documentation

- [Learning Analytics Framework](../15-analytics-and-reporting/learning-analytics.md) - xAPI statement schemas and analytics use cases
- [Event Model](../15-analytics-and-reporting/event-model.md) - Internal event tracking system
- [SCORM Compliance](./scorm-compliance.md) - Related standards for content packaging
- [LMS LTI Integration](./lms-ltispec-future.md) - LMS integration (mentions xAPI as complementary)
- [API Design Principles](../18-architecture/api-design-principles.md) - REST API design for xAPI endpoints

## Open questions

### LRS Strategy

**Internal vs. External LRS**: Should MLC host its own full-featured LRS, or just generate statements and rely on external LRS integrations?
- **Internal LRS**: Full control, faster queries, no dependency on third parties
- **External Only**: Lower operational overhead, leverage specialized LRS platforms
- **Recommendation**: Implement lightweight internal LRS for basic storage and queries; support external LRS forwarding for advanced analytics

**LRS Platform Selection** (if internal): If we build an internal LRS, which open-source platform should we use as a foundation?
- **Learning Locker**: Full-featured, but requires MongoDB
- **Veracity LRS**: Lightweight, but less mature
- **Custom Implementation**: Full control, tailored to MLC needs, but development overhead
- **Recommendation**: Custom lightweight implementation using PostgreSQL JSONB; sufficient for MLC needs

### Statement Generation

**Real-time vs. Batch**: Should xAPI statements be generated in real-time (immediately after events) or in batch (every few minutes)?
- **Real-time**: Lower latency, better for live analytics, but higher resource usage
- **Batch**: More efficient, easier to manage, but delayed availability
- **Recommendation**: Hybrid - generate statements in real-time but queue for validation and forwarding in small batches (every 30 seconds)

**Statement Volume**: What percentage of internal events should generate xAPI statements?
- **All Events**: Complete record, but high volume and storage costs
- **Learning Events Only**: Focus on educational activities, omit UI interactions
- **Recommendation**: Generate statements for all learning progress events (game completions, mastery, assignments); omit page views, clicks, and system events unless needed for research

**Sampling for Free Trial Users**: Should Prelude (free trial) users generate xAPI statements?
- **Yes**: Complete data for trial evaluation and research
- **No**: Reduce costs and complexity for non-paying users
- **Recommendation**: Generate statements for trial users, but don't forward to external LRS; allow export if user converts to paid subscription

### Privacy and Compliance

**Anonymization Strategy**: How should we anonymize statements for cross-organization analytics while preserving utility?
- **Hash User IDs**: HMAC hash of user_id with salt (prevents re-identification)
- **Aggregate Only**: Only provide aggregated metrics, never individual statements
- **Consent-Based**: Only include in cross-org analytics if user explicitly consents
- **Recommendation**: Use hashed user IDs for anonymization; require opt-in consent for cross-organization research use

**Right to Deletion**: If a user requests data deletion, should we void xAPI statements or completely purge them?
- **Void Only**: Mark statements as voided (per xAPI spec), preserves referential integrity
- **Full Purge**: Delete statements entirely, ensures no traces remain
- **Recommendation**: Void statements initially (90-day grace period); full purge after grace period or if user explicitly requests immediate deletion

**COPPA and Under-13 Users**: Should we generate xAPI statements for users under 13, given stricter COPPA requirements?
- **Yes, with Consent**: Generate if parental consent obtained
- **No Statements**: Disable xAPI for under-13 users entirely
- **Restricted Mode**: Generate statements but never share externally or use in analytics
- **Recommendation**: Restricted mode - generate and store internally, but mark as `privacy_level: restricted` and exclude from exports/analytics without parental consent

### External LRS Integration

**Multiple LRS per Organization**: Should organizations be able to configure multiple external LRS instances?
- **Single LRS**: Simpler configuration, lower overhead
- **Multiple LRS**: Flexibility for organizations with multiple analytics platforms
- **Recommendation**: Support single primary LRS initially; add multi-LRS support if customer demand warrants

**LRS Credential Management**: How should we securely store and manage LRS authentication credentials?
- **Encrypted in Database**: Standard approach, credentials encrypted at rest
- **External Secrets Manager**: Use AWS Secrets Manager, Azure Key Vault, etc.
- **Recommendation**: Encrypt credentials in database using application-level encryption; migrate to external secrets manager as platform matures

**Statement Forwarding Failure Handling**: If external LRS is down for an extended period, how long should we retry?
- **Finite Retries**: Give up after 5 attempts over 24 hours
- **Indefinite Queue**: Keep retrying until successful
- **Time-based Expiration**: Retry for up to 7 days, then give up
- **Recommendation**: Retry for 48 hours with exponential backoff; after 48 hours, alert admin and optionally export failed statements to file

### Query API and Performance

**Query Performance at Scale**: How do we ensure fast xAPI queries as statement volume grows?
- **Database Indexes**: Standard indexes on actor, verb, activity, timestamp
- **Partitioning**: Partition statements table by month or organization
- **Materialized Views**: Pre-compute common aggregations
- **Recommendation**: Start with indexes; add partitioning when statement table exceeds 10 million rows; use materialized views for common admin dashboards

**Query Complexity Limits**: Should we limit the complexity of xAPI queries to prevent expensive operations?
- **No Limits**: Full flexibility, but risk of slow queries
- **Rate Limiting**: Limit query frequency per user
- **Complexity Scoring**: Reject queries that would scan too many rows
- **Recommendation**: Implement rate limiting (10 queries per minute per user); add complexity scoring if abuse detected

**Statement Attachments**: Should we support xAPI attachments for storing artifacts like MIDI recordings or student compositions?
- **Yes**: Rich learning records, but significant storage costs
- **No**: Keep statements lightweight, store artifacts separately
- **Recommendation**: Phase 2 feature - support attachments for specific use cases (student performances, compositions), with size limits (5 MB max per attachment)

### Technical Implementation

**xAPI Library**: Should we use an existing xAPI library or build custom?
- **Existing Library** (e.g., TinCanPHP, xAPI.js): Faster implementation, community support
- **Custom Implementation**: Full control, tailored to MLC needs
- **Recommendation**: Use existing library for statement validation and formatting; build custom transformation and storage logic

**Statement Validation Strictness**: How strict should we be in validating statements before storage?
- **Strict**: Reject any statement that doesn't perfectly match xAPI spec
- **Lenient**: Accept statements with minor issues, log warnings
- **Recommendation**: Strict validation for all statements; better to fail early than store invalid data

**API Versioning**: Should we version the xAPI API separately from the main MLC API?
- **Separate Versioning**: Aligns with xAPI spec versioning (1.0.3)
- **Unified Versioning**: Consistent with MLC platform versioning
- **Recommendation**: Use xAPI spec versioning for xAPI endpoints (`/xapi/v1/statements`); clearly document xAPI 1.0.3 compliance

### Future Enhancements (Out of Scope for Initial Release)

**cmi5 Profile**: Implement cmi5 profile for launched content compatibility (successor to SCORM).

**xAPI Cohort Queries**: Specialized queries for cohort analysis and educational research.

**Real-time xAPI Streams**: WebSocket or SSE streams for real-time statement notifications.

**xAPI Dashboards**: Purpose-built dashboards for exploring xAPI data (distinct from standard MLC reports).

**Multi-Tenant LRS**: Allow multiple organizations to share internal LRS while maintaining data isolation.

**Statement Versioning**: Support for statement correction and versioning beyond simple voiding.

---

## Implementation Notes

**Phase 1 (Core xAPI)**:
- Internal lightweight LRS implementation (PostgreSQL JSONB storage)
- Statement generation for major learning events (game completions, mastery, assignments)
- Basic xAPI API (POST/GET statements, query with filtering)
- System Admin configuration interface for LRS settings

**Phase 2 (External Integration)**:
- External LRS connection and statement forwarding
- Batch export capabilities (JSON, CSV)
- Enhanced privacy controls and anonymization
- Student/parent data access interface (FERPA compliance)

**Phase 3 (Advanced Features)**:
- Statement attachments for artifacts
- Real-time statement streaming
- Advanced cohort queries for research
- cmi5 profile support

**Testing Requirements**:
- xAPI conformance testing using ADL test suite
- Statement validation against xAPI 1.0.3 schema
- Load testing for statement generation (1000 statements/second target)
- External LRS integration testing (Learning Locker, Watershed, institutional LRS)
- Privacy compliance testing (anonymization, consent workflows)

**Documentation Deliverables**:
- xAPI Developer Guide (for future integrations)
- xAPI Admin Configuration Guide
- xAPI Statement Reference (MLC-specific verbs and extensions)
- External LRS Integration Guide (for IT administrators)
- xAPI Data Dictionary (for researchers and analysts)

**Related Specifications**:
- [Learning Analytics Framework](../15-analytics-and-reporting/learning-analytics.md) - xAPI statement schemas and use cases
- [Event Model](../15-analytics-and-reporting/event-model.md) - Internal event system
- [SCORM Compliance](./scorm-compliance.md) - Related content packaging standard
- [LMS LTI Integration](./lms-ltispec-future.md) - LMS integration (complementary to xAPI)