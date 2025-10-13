# LMS LTI Spec Future

## Purpose

This specification enables Learning Tools Interoperability (LTI) integration between MLC-LMS and institutional Learning Management Systems (Canvas, Moodle, Blackboard, Schoology, D2L Brightspace, etc.). LTI provides a standardized way for schools and districts to embed MLC learning activities within their existing LMS infrastructure, enabling:

- **Seamless Single Sign-On**: Students and teachers access MLC content directly from their LMS without separate login credentials
- **Grade Passback**: Student scores from MLC games and assignments automatically sync to LMS gradebooks
- **Deep Linking**: Teachers select and embed specific MLC games, sequences, or assignments directly into LMS course content
- **Roster Sync**: Student enrollments and teacher assignments flow automatically from the LMS to MLC
- **Institutional Integration**: Schools leverage their existing LMS investment while accessing MLC's specialized music literacy instruction

This integration supports MLC's vision to serve schools and districts at scale by meeting institutions where they already work, reducing administrative overhead, and providing teachers with a unified experience across their digital learning ecosystem.

**Why This Matters**: Many schools using MLC operate within district-mandated LMS platforms. LTI compliance removes adoption friction, increases teacher engagement, and positions MLC as a best-in-class educational tool that respects institutional technology ecosystems.

## Scope

### Included

**LTI 1.3 (LTI Advantage) Implementation**:
- LTI 1.3 Core specification for tool launch and authentication
- Deep Linking 2.0 for content item selection and placement
- Assignment and Grade Services (AGS) for score sync to LMS gradebook
- Names and Role Provisioning Services (NRPS) for roster synchronization
- OAuth 2.0 and JWT-based security model
- Platform registration and configuration management

**Integration Features**:
- Single Sign-On (SSO) launch from LMS to MLC
- User provisioning and role mapping between LMS and MLC roles
- Context (course/class) mapping to MLC organizations
- Real-time and batch grade passback
- Assignment linking and deep content selection
- Secure platform registration and key management
- Support for multiple LMS platforms with unified configuration

**Administrative Capabilities**:
- LTI platform registration interface for System Admins
- Connection status monitoring and diagnostics
- Grade sync audit logs and error handling
- Configuration per organization or globally
- Support for multiple simultaneous LMS integrations per organization

**User Experience**:
- Transparent LMS-to-MLC launch experience
- Contextual navigation back to LMS
- Grade visibility in both systems
- Teacher content selection via LTI Deep Linking

### Excluded

**LTI 1.1 (Legacy)**: The older LTI 1.1 standard will not be supported due to security limitations and lack of modern features. Schools requiring LTI must use LMS platforms supporting LTI 1.3.

**Real-time Communication**: LTI does not provide real-time push notifications. Grade updates and roster changes are pull-based or scheduled sync operations.

**Full Data Migration**: LTI does not replace CSV import capabilities or handle historical data migration. It operates prospectively from integration date forward.

**LMS-Specific Features**: Custom integrations tailored to individual LMS platform proprietary APIs are excluded. This spec covers the standard LTI protocol only.

**Content Hosting in LMS**: MLC content remains hosted on MLC infrastructure. LTI provides launch and grade sync, not content embedding or SCORM packaging (see [SCORM Compliance](./scorm-compliance.md) for alternative).

**Billing Integration**: LTI does not handle subscription management, billing, or seat allocation. These remain managed through MLC's native billing system (see [Payments and Subscriptions](../14-payments-and-subscriptions/)).

## Data model (if applicable)

### Key Entities

#### LTI Platform Registration
Represents a registered LMS platform that can launch MLC tools.

- **Platform ID**: Unique identifier for the LMS platform instance (e.g., `lti_platform_12345`)
- **Platform Name**: Display name (e.g., "District Central High Canvas")
- **Issuer**: LTI issuer identifier (URL from LMS)
- **Client ID**: OAuth 2.0 client ID assigned by the LMS
- **Deployment ID**: LTI deployment identifier (one platform can have multiple deployments)
- **Auth URL**: Platform's OIDC authentication endpoint
- **Token URL**: Platform's OAuth 2.0 token endpoint
- **JWKS URL**: Platform's JSON Web Key Set endpoint for signature verification
- **Organization ID**: MLC organization this platform connects to
- **Status**: Active, Suspended, Archived
- **Created At**: Registration timestamp
- **Last Verified**: Last successful handshake timestamp
- **Configuration**: JSON object with platform-specific settings

#### LTI Launch Session
Represents a user's authenticated launch from LMS into MLC.

- **Launch ID**: Unique identifier for this launch session
- **Platform ID**: Associated LTI platform
- **User ID**: MLC user ID (provisioned or matched)
- **LTI User ID**: User identifier from LMS (stable, opaque)
- **Context ID**: LMS course/context identifier
- **Organization ID**: MLC organization
- **Resource Link ID**: Identifier for the specific LMS placement/link
- **Roles**: Array of LTI roles (e.g., `Instructor`, `Learner`, `Administrator`)
- **Launch Type**: `resource_link`, `deep_linking`, `data_privacy`
- **Created At**: Launch timestamp
- **Expires At**: Session expiration timestamp
- **Return URL**: LMS URL to return user when done
- **Custom Parameters**: Key-value pairs from LMS launch
- **Locale**: User's preferred language from LMS

#### LTI Resource Link
Represents a placement of MLC content within an LMS course.

- **Resource Link ID**: Unique identifier from LMS
- **Platform ID**: Associated LTI platform
- **Context ID**: LMS course/context identifier
- **Organization ID**: MLC organization
- **Content Type**: `game`, `sequence`, `assignment`, `all_games_library`
- **Content ID**: MLC game ID, sequence ID, or assignment ID
- **Title**: Display title in LMS
- **Description**: Optional description
- **Gradable**: Boolean flag if this resource reports grades
- **AGS Lineitem ID**: LTI AGS lineitem identifier for grade sync
- **Max Score**: Maximum score for this resource (default 100)
- **Created At**: Link creation timestamp
- **Created By**: MLC user who created the link (teacher)
- **Configuration**: JSON object with resource-specific settings

#### LTI User Mapping
Maps LMS user identities to MLC user accounts.

- **Mapping ID**: Unique identifier
- **Platform ID**: Associated LTI platform
- **LTI User ID**: User identifier from LMS (stable, opaque)
- **MLC User ID**: MLC user account ID
- **LTI Roles**: Array of roles from LMS
- **MLC Role**: Mapped MLC role (Teacher, Student, etc.)
- **Context ID**: Optional - specific to a course context
- **Email**: User's email from LMS (if provided)
- **Name**: User's full name from LMS
- **First Seen**: Initial mapping timestamp
- **Last Launch**: Most recent launch timestamp
- **Provisioning Method**: `auto`, `manual`, `matched`

#### LTI Grade Sync Record
Tracks grade passback operations to LMS.

- **Sync ID**: Unique identifier
- **Platform ID**: Associated LTI platform
- **Resource Link ID**: Associated resource link
- **AGS Lineitem ID**: LTI AGS lineitem identifier
- **User ID**: MLC user ID
- **LTI User ID**: LMS user identifier
- **Score**: Numeric score to sync
- **Max Score**: Maximum possible score
- **Activity Progress**: `Initialized`, `Started`, `InProgress`, `Submitted`, `Completed`
- **Grading Progress**: `FullyGraded`, `Pending`, `PendingManual`, `Failed`, `NotReady`
- **Comment**: Optional comment to include with grade
- **Timestamp**: When score was achieved in MLC
- **Sync Status**: `pending`, `in_progress`, `success`, `failed`, `retry`
- **Sync Attempted At**: Last sync attempt timestamp
- **Sync Completed At**: Successful sync timestamp
- **Error Message**: If sync failed, error details
- **Retry Count**: Number of sync attempts

### Relationships

- **LTI Platform Registration** → **Organization** (one-to-one or one-to-many)
- **LTI Launch Session** → **LTI Platform Registration** (many-to-one)
- **LTI Launch Session** → **User** (many-to-one)
- **LTI Resource Link** → **LTI Platform Registration** (many-to-one)
- **LTI Resource Link** → **Game** or **Sequence** or **Assignment** (polymorphic)
- **LTI User Mapping** → **LTI Platform Registration** (many-to-one)
- **LTI User Mapping** → **User** (many-to-one)
- **LTI Grade Sync Record** → **LTI Resource Link** (many-to-one)
- **LTI Grade Sync Record** → **User** (many-to-one)

### Database Indexes

**Critical Performance Indexes**:
- `lti_launch_sessions` (platform_id, lti_user_id) for rapid user lookup
- `lti_user_mappings` (platform_id, lti_user_id) for identity resolution
- `lti_resource_links` (platform_id, resource_link_id) for content routing
- `lti_grade_sync_records` (sync_status, sync_attempted_at) for queue processing
- `lti_grade_sync_records` (user_id, resource_link_id) for user grade history

## Behavior and rules

### LTI Launch Flow

**1. LMS Initiates Launch**
- User clicks MLC link in their LMS course
- LMS generates LTI 1.3 launch request (OIDC login initiation)
- LMS redirects user to MLC's LTI login endpoint with state parameter

**2. MLC Validates Platform**
- MLC receives login initiation request
- MLC looks up LTI Platform Registration by issuer and client_id
- If platform not found or suspended: display error, log security event
- MLC generates authentication request and redirects to platform's auth URL

**3. Platform Authenticates User**
- LMS authenticates user (if not already authenticated)
- LMS generates signed JWT with launch claims (user, context, roles, resource)
- LMS redirects user back to MLC's launch endpoint with signed JWT

**4. MLC Processes Launch**
- MLC validates JWT signature using platform's JWKS
- MLC verifies required claims: `iss`, `aud`, `sub`, `nonce`, `exp`, `iat`
- MLC extracts user identity (`sub`), roles, context, resource link
- MLC creates or updates LTI User Mapping
- MLC provisions or matches user to existing MLC account

**5. User Provisioning and Role Mapping**
- Extract user info: `sub` (required), `name`, `email`, `given_name`, `family_name`
- Map LTI roles to MLC roles:
  - `http://purl.imsglobal.org/vocab/lis/v2/membership#Instructor` → Teacher
  - `http://purl.imsglobal.org/vocab/lis/v2/membership#Learner` → Student
  - `http://purl.imsglobal.org/vocab/lis/v2/membership#Administrator` → Teacher-Admin or Subscriber-Admin
  - `http://purl.imsglobal.org/vocab/lis/v2/membership#TeachingAssistant` → Teacher (limited permissions)
- Create MLC user account if first launch (auto-provisioning)
- Update user profile with latest info from LMS (name, email)
- Respect privacy settings: student age, COPPA/FERPA compliance

**6. Context and Organization Mapping**
- Extract context: `context.id`, `context.title`, `context.label`
- Map LMS context to MLC organization or class
- If first launch for this context: create organization or prompt admin for mapping
- Associate user with context/organization
- Apply seat allocation rules based on subscription plan

**7. Resource Routing**
- Extract resource link: `resource_link.id`, `resource_link.title`, `custom` params
- Lookup LTI Resource Link in database
- If deep linking request: launch Deep Linking interface (teacher selects content)
- If resource link exists: route user to specified game, sequence, or assignment
- If generic launch (no resource link): route to MLC dashboard

**8. Session Creation**
- Create MLC session linked to LTI launch
- Store launch context: return_url, platform_id, resource_link_id
- Set appropriate session timeout (aligned with LMS session duration)
- Log launch event to analytics

**9. User Lands in MLC**
- User sees MLC content in full browser or iframe (depending on LMS)
- MLC displays contextual navigation (breadcrumbs showing LMS course)
- "Return to [LMS]" button visible using launch return_url
- User interacts with MLC games, assignments, sequences

### Deep Linking Flow (Teacher Content Selection)

**1. Teacher Initiates Deep Linking**
- Teacher clicks "Add MLC Content" or similar in LMS course editor
- LMS launches MLC with `message_type: LtiDeepLinkingRequest`
- Launch includes: `deep_linking_settings`, `accept_types`, `accept_presentation_targets`

**2. MLC Presents Content Picker**
- MLC authenticates teacher via LTI launch
- MLC displays content selection interface:
  - Browse games by category, level, concept
  - Browse sequences (Lifetime Musician, Solfege, method-aligned)
  - Create assignment on-the-fly
  - Select "All Games Library" for open exploration
- Filters and search enabled
- Preview capabilities for games

**3. Teacher Configures Content**
- Teacher selects one or more items
- For each item, teacher configures:
  - Display title (pre-filled, editable)
  - Description (optional)
  - Gradable (yes/no)
  - Max score (default 100)
  - Target score (optional custom parameter)
- Teacher clicks "Add to Course" or "Embed"

**4. MLC Generates Deep Link Response**
- MLC creates LTI Resource Link entries in database
- MLC generates signed JWT with deep linking response:
  - `content_items`: Array of selected content with URLs, titles, metadata
  - `ltiResourceLink` for each item with custom parameters
- MLC redirects teacher back to LMS with response JWT

**5. LMS Places Content**
- LMS receives deep linking response
- LMS creates course content items (links, assignments, modules)
- LMS stores resource link IDs for future launches
- Teacher sees content embedded in course structure

### Grade Passback (Assignment and Grade Services)

**1. Grade Sync Trigger**
- Student completes MLC game stage (Quiz or Review)
- Student completes full assignment
- Teacher manually triggers grade recalculation
- Scheduled batch job runs (every 15 minutes)

**2. Score Calculation**
- MLC determines final score based on:
  - Quiz stage score (primary)
  - Review stage score (if applicable)
  - Target score achievement
  - Assignment completion percentage
- Score normalized to resource link max_score (default 100)

**3. Grade Sync Queue**
- MLC creates LTI Grade Sync Record with status `pending`
- Record includes: user, resource link, score, activity/grading progress
- Queue processor picks up pending records (FIFO, with retry logic)

**4. AGS Token Request**
- MLC requests OAuth 2.0 access token from platform token endpoint
- Includes: `grant_type: client_credentials`, `scope: https://purl.imsglobal.org/spec/lti-ags/scope/score`
- Platform returns access token (cached for reuse, respects expires_in)

**5. Score Submission**
- MLC calls platform's AGS score endpoint: `POST /lineitems/{lineitem_id}/scores`
- Request body includes:
  ```json
  {
    "userId": "lti_user_id",
    "scoreGiven": 87,
    "scoreMaximum": 100,
    "activityProgress": "Completed",
    "gradingProgress": "FullyGraded",
    "timestamp": "2025-10-12T14:30:00Z",
    "comment": "Quiz: 87/100 (Target: 80)"
  }
  ```
- Include authentication header: `Authorization: Bearer {access_token}`

**6. Platform Response Handling**
- **Success (200/201)**: Update sync record status to `success`, log success event
- **Client Error (400/401/403/404)**: Mark as `failed`, log error, alert admin if resource misconfigured
- **Server Error (500/503)**: Mark as `retry`, increment retry count, requeue with exponential backoff
- **Timeout**: Mark as `retry`, requeue
- Max retries: 5 attempts over 24 hours, then mark `failed` and alert

**7. Grade Visibility**
- Student sees grade in LMS gradebook (timing depends on LMS)
- Student still sees score details in MLC (MLC is source of truth)
- Teacher sees grade in both LMS and MLC reporting
- Grade sync audit trail available to admins

### Roster Sync (Names and Role Provisioning Services)

**1. Sync Trigger**
- Scheduled job runs (e.g., daily at 2:00 AM)
- Manual "Sync Roster" action by Teacher-Admin or System Admin
- On-demand when accessing LTI-linked class for first time

**2. NRPS Token Request**
- MLC requests OAuth 2.0 access token with NRPS scope
- Scope: `https://purl.imsglobal.org/spec/lti-nrps/scope/contextmembership.readonly`

**3. Fetch Context Membership**
- MLC calls NRPS endpoint: `GET /context/{context_id}/memberships`
- Response includes array of members with:
  - `user_id`: Stable LMS user identifier
  - `name`, `email`, `given_name`, `family_name`
  - `roles`: Array of LTI roles
  - `status`: `Active`, `Inactive`, `Deleted`
  - `lis_person_sourcedid`: Optional institutional ID

**4. Sync Operations**
- **New Users**: Auto-provision MLC accounts with role mapping
- **Existing Users**: Update name/email if changed
- **Role Changes**: Update MLC role if LTI role changed (with admin approval if sensitive)
- **Dropped Students**: Mark as inactive in MLC (don't delete, preserve history)
- **Added Students**: Provision and assign to context class
- **Teacher Changes**: Update class assignments

**5. Conflict Resolution**
- If email conflict: log warning, don't overwrite manually-entered email
- If role conflict (e.g., user is Teacher in MLC but Learner in LMS): alert admin, require manual resolution
- If user exists in multiple LMS contexts with different roles: use highest permission level
- If user manually created in MLC before LTI launch: create mapping, don't duplicate

**6. Audit and Reporting**
- Log all roster sync operations with timestamps
- Report to Teacher-Admin: X users added, Y updated, Z inactivated
- Alert on anomalies: mass deletion, unexpected role changes

### Security and Validation Rules

**JWT Signature Validation**:
- All incoming JWTs must be validated using platform's JWKS
- Reject if signature invalid, expired, or from unknown issuer
- Verify `aud` matches MLC's client ID
- Verify `nonce` is unique and not reused (prevent replay attacks)

**HTTPS Required**:
- All LTI endpoints must be served over HTTPS
- Reject any non-HTTPS launch or callback URLs

**State Parameter**:
- Use cryptographically random state parameter in OIDC flow
- Verify state on callback to prevent CSRF attacks

**Token Expiration**:
- Respect `exp` claim in JWTs, reject expired tokens
- OAuth access tokens must be cached and reused until near expiration
- Implement token refresh logic if supported by platform

**Privacy and Data Minimization**:
- Only request NRPS data when necessary (not on every launch)
- Store only required user data (ID, name, email, roles)
- Respect "do not share email" flags from LMS
- Comply with COPPA/FERPA: if student under 13, limit data collection

**Rate Limiting**:
- Apply rate limits to LTI endpoints to prevent abuse
- Throttle grade sync operations (max 100 scores/minute per platform)
- Queue roster sync operations to avoid overwhelming LMS APIs

**Error Handling**:
- Display user-friendly error messages (don't expose internal details)
- Log detailed errors for admin diagnostics
- Provide clear resolution steps for common issues

### Assignment Completion and Grade Finalization Rules

**Single Game Resource**:
- Grade based on Quiz or Review stage score (whichever is primary for that game)
- Sync grade immediately upon stage completion
- `activityProgress: Completed`, `gradingProgress: FullyGraded`

**Sequence Resource**:
- Grade based on average score across all games in sequence
- Only sync when full sequence completed
- `activityProgress: InProgress` until all games done
- Allow teacher override to sync partial completion

**Custom Assignment Resource**:
- Grade based on assignment completion rules (see [Assignment Model](../07-content-architecture/assignment-model.md))
- Sync when assignment marked complete or all required games done
- Include weighted scoring if teacher configured weights

**Multiple Attempts**:
- Support grade replacement (always send most recent score)
- Track attempt history in MLC, sync only final/best score to LMS
- Allow teacher policy: "best score" vs "most recent" vs "average"

**Late Submissions**:
- Include timestamp in grade passback
- Let LMS apply late penalties based on its policies
- MLC does not enforce due dates from LMS (future enhancement)

## UX requirements (if applicable)

### LTI Launch Experience

**Seamless Transition**:
- User clicks MLC link in LMS → immediately redirected to MLC content (no intermediate login screens if trust established)
- Loading indicator during LTI handshake (< 2 seconds for 95th percentile)
- User lands directly on intended resource (game, sequence, assignment) without further navigation

**Contextual Navigation**:
- Display breadcrumb: "[LMS Name] / [Course Name] / [Content Name]"
- Prominent "Return to [LMS Name]" button in header (uses LTI return_url)
- MLC navigation sidebar available but contextualized to LTI session
- Disable or hide features not applicable to LTI users (e.g., standalone subscription management)

**Iframe vs Full Window**:
- Support both iframe embed and full window launch
- Detect iframe context, adjust UI accordingly (e.g., hide redundant headers)
- Provide "Open in New Window" option if iframe too constraining
- Ensure full functionality in iframe (no iframe-busting)

**Session Continuity**:
- If user navigates within MLC, maintain LTI context
- Allow "breadcrumb" navigation back through MLC structure
- On session timeout, redirect to LMS return_url (don't leave user stranded)
- Warn before session expiration (2 minutes prior)

### Deep Linking Content Picker

**Teacher Interface**:
- Full-screen modal or dedicated page for content selection
- Three-tab interface: "Games", "Sequences", "Assignments"
- Left sidebar filters: Category, Level, Concept, Method Alignment
- Main area: Grid view of content items with thumbnails, titles, descriptions
- Search bar with instant filtering
- Preview button for each item (launches game in preview mode)

**Item Selection**:
- Checkboxes for multi-select
- Single-select mode if LMS only supports one item at a time
- Selected items summary panel (bottom or right sidebar)
- Remove button for each selected item

**Configuration Per Item**:
- Title input (pre-filled with default, editable)
- Description textarea (optional)
- "Make this gradable" toggle
- Max score input (default 100, visible if gradable)
- Target score input (optional, stored as custom parameter)

**Submission**:
- Prominent "Add to Course" button (primary CTA)
- "Cancel" button returns to LMS without changes
- Confirmation: "Adding [N] items to your course..."
- Auto-close and redirect to LMS on success

**Accessibility**:
- Keyboard navigable (tab order, arrow keys for grid)
- Screen reader friendly (ARIA labels, live regions for updates)
- High contrast mode support
- Focus management (return focus to trigger on cancel)

### Grade Visibility

**Student View**:
- In MLC: Full score details, breakdown by stage, history
- In LMS: Single synced grade (per resource link) in gradebook
- No MLC-branded gradebook in LMS (uses native LMS gradebook)
- Clear indication in MLC if grade is synced to LMS: "✓ Grade synced to [LMS Name]"

**Teacher View**:
- In MLC: Full analytics, all student scores, assignment completion
- In LMS: Gradebook shows MLC-synced scores with timestamp
- Teacher can click grade in LMS to launch into MLC for details (deep link back)
- Grade sync status indicator: "Last synced: 5 minutes ago" or "Sync pending"

**Sync Status Indicators**:
- Real-time status: Syncing (spinner), Synced (checkmark), Error (warning icon)
- Tooltip on hover: detailed sync info or error message
- Retry button for failed syncs (teacher/admin only)

### Admin Configuration Interface

**Platform Registration (System Admin)**:
- "LTI Integrations" section in System Admin panel
- "Add New LMS Platform" button launches wizard
- Wizard steps:
  1. Platform Details: Name, Issuer URL, Client ID, Deployment ID
  2. Endpoints: Auth URL, Token URL, JWKS URL (auto-detect option)
  3. Organization Mapping: Select MLC organization to link
  4. Review and Register
- Display registration keys to copy into LMS configuration
- Test connection button (performs handshake validation)

**Platform Management List View**:
- Table columns: Platform Name, Organization, Status, Last Verified, Actions
- Status badge: Active (green), Suspended (yellow), Error (red)
- Actions: Edit, Test Connection, View Audit Log, Suspend, Delete
- Filter by organization, status
- Search by platform name

**Connection Diagnostics**:
- "Test Connection" runs through OIDC handshake without user
- Displays results: JWT validation, endpoint reachability, JWKS fetch
- Error details with remediation steps
- Export diagnostic report (for support tickets)

**Grade Sync Monitor**:
- Dashboard widget: grades synced today, failed syncs, pending queue depth
- Detailed log: filterable by platform, user, resource, status, date range
- Retry failed syncs in bulk
- Alert thresholds: notify if failed syncs exceed X%

### Accessibility

**WCAG 2.1 AA Compliance**:
- All LTI-specific UI meets accessibility standards (see [A11y Standards](../01-ux-design-system/a11y-standards.md))
- Content picker keyboard navigable
- Error messages announced to screen readers
- Loading states communicated via ARIA live regions

**Keyboard Support**:
- Tab order: Filter controls → Content grid → Selected items → Action buttons
- Arrow keys navigate content grid
- Enter/Space to select items
- Escape to cancel

**Visual Design**:
- High contrast mode for all LTI UI
- Focus indicators meet 3:1 contrast ratio
- Color not sole indicator of state (use icons + text)

## Telemetry

### LTI Events

All LTI events follow the platform's [Event Model](../15-analytics-and-reporting/event-model.md) structure.

#### Launch Events

**`lti_launch_initiated`**
- **When**: User clicks MLC link in LMS, before authentication
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `lti_message_type`: `LtiResourceLinkRequest`, `LtiDeepLinkingRequest`, etc.
  - `issuer`: Platform issuer URL
  - `client_id`: Platform client ID
  - `user_agent`: Browser user agent
  - `referrer`: LMS URL where launch originated

**`lti_launch_completed`**
- **When**: Successful LTI launch, user lands in MLC
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `launch_id`: Unique launch session ID
  - `user_id`: MLC user ID
  - `lti_user_id`: LMS user identifier
  - `context_id`: LMS context/course ID
  - `resource_link_id`: Resource link ID (if applicable)
  - `roles`: Array of LTI roles
  - `provisioning_method`: `auto`, `manual`, `matched`
  - `new_user`: Boolean, true if user auto-provisioned
  - `duration_ms`: Time from initiation to completion

**`lti_launch_failed`**
- **When**: LTI launch fails due to validation or error
- **Properties**:
  - `platform_id`: LTI Platform ID (if identifiable)
  - `error_type`: `invalid_jwt`, `platform_not_found`, `expired_token`, `missing_claims`, `unknown`
  - `error_message`: Detailed error description
  - `issuer`: Platform issuer URL
  - `user_agent`: Browser user agent

#### Deep Linking Events

**`lti_deep_linking_opened`**
- **When**: Teacher opens MLC content picker via deep linking
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `user_id`: MLC user ID (teacher)
  - `context_id`: LMS context ID
  - `accept_types`: Content types LMS accepts
  - `accept_presentation_targets`: iframe, window, etc.

**`lti_deep_linking_content_selected`**
- **When**: Teacher selects content item in picker
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `user_id`: MLC user ID (teacher)
  - `content_type`: `game`, `sequence`, `assignment`, `all_games`
  - `content_id`: MLC content identifier
  - `gradable`: Boolean

**`lti_deep_linking_completed`**
- **When**: Teacher submits content selection, returns to LMS
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `user_id`: MLC user ID (teacher)
  - `context_id`: LMS context ID
  - `items_count`: Number of items selected
  - `items`: Array of selected content types and IDs

#### Grade Sync Events

**`lti_grade_sync_queued`**
- **When**: Grade sync record created and added to queue
- **Properties**:
  - `sync_id`: Grade sync record ID
  - `platform_id`: LTI Platform ID
  - `user_id`: MLC user ID
  - `resource_link_id`: Resource link ID
  - `score`: Score to sync
  - `max_score`: Maximum score
  - `trigger`: `game_completion`, `assignment_completion`, `manual`, `batch`

**`lti_grade_sync_success`**
- **When**: Grade successfully synced to LMS
- **Properties**:
  - `sync_id`: Grade sync record ID
  - `platform_id`: LTI Platform ID
  - `user_id`: MLC user ID
  - `resource_link_id`: Resource link ID
  - `score`: Synced score
  - `duration_ms`: Time from queue to completion
  - `attempt_count`: Number of attempts (usually 1)

**`lti_grade_sync_failed`**
- **When**: Grade sync fails after retry attempts
- **Properties**:
  - `sync_id`: Grade sync record ID
  - `platform_id`: LTI Platform ID
  - `user_id`: MLC user ID
  - `resource_link_id`: Resource link ID
  - `error_type`: `unauthorized`, `not_found`, `server_error`, `timeout`
  - `error_message`: Detailed error
  - `attempt_count`: Total attempts made
  - `will_retry`: Boolean, true if will retry

#### Roster Sync Events

**`lti_roster_sync_started`**
- **When**: NRPS roster sync job begins
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `context_id`: LMS context ID
  - `organization_id`: MLC organization ID
  - `trigger`: `scheduled`, `manual`, `on_demand`

**`lti_roster_sync_completed`**
- **When**: Roster sync finishes successfully
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `context_id`: LMS context ID
  - `organization_id`: MLC organization ID
  - `users_added`: Count of new users provisioned
  - `users_updated`: Count of existing users updated
  - `users_inactivated`: Count of users marked inactive
  - `errors_count`: Count of errors encountered
  - `duration_ms`: Sync duration

**`lti_roster_sync_failed`**
- **When**: Roster sync fails
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `context_id`: LMS context ID
  - `error_type`: `token_error`, `endpoint_error`, `data_error`
  - `error_message`: Detailed error

#### Admin Events

**`lti_platform_registered`**
- **When**: New LMS platform registered
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `platform_name`: Display name
  - `organization_id`: MLC organization ID
  - `registered_by`: System Admin user ID

**`lti_platform_test_connection`**
- **When**: Admin tests platform connection
- **Properties**:
  - `platform_id`: LTI Platform ID
  - `success`: Boolean
  - `error_message`: If failed, error details
  - `tested_by`: System Admin user ID

## Permissions

Access control for LTI features follows the platform's [Roles Matrix](../02-roles-and-permissions/roles-matrix.md).

### Platform Registration and Configuration

**Create LTI Platform Registration**: System Admin only
**Edit LTI Platform Registration**: System Admin only
**Delete LTI Platform Registration**: System Admin only
**Test LTI Connection**: System Admin, Subscriber-Admin (own organization only)
**View LTI Configuration**: System Admin, Subscriber-Admin (own organization only), Teacher-Admin (read-only)

### User Provisioning and Mapping

**Auto-provision Users via LTI**: Enabled by default, configurable per platform
**Manually Map LTI User to MLC User**: System Admin, Subscriber-Admin (own organization only)
**View LTI User Mappings**: System Admin, Subscriber-Admin (own organization only), Teacher-Admin (read-only, own scope)
**Override Role Mapping**: System Admin only (security-sensitive)

### Content Linking (Deep Linking)

**Use Deep Linking to Add Content**: Teacher, Teacher-Admin, Subscriber-Admin (if also Teacher role)
**Configure Resource Link Settings**: User who created the link (teacher), Teacher-Admin (same organization), System Admin
**Delete Resource Link**: User who created the link, Teacher-Admin (same organization), System Admin

### Grade Sync

**Trigger Grade Sync**: Automatic on game/assignment completion (system)
**Manually Trigger Grade Sync**: Teacher (own students), Teacher-Admin (own scope), System Admin
**Retry Failed Grade Sync**: Teacher (own students), Teacher-Admin (own scope), System Admin
**View Grade Sync Logs**: Teacher (own students), Teacher-Admin (own scope), Subscriber-Admin (own organization), System Admin
**Override Grade Sync**: System Admin only (for troubleshooting)

### Roster Sync

**Trigger Roster Sync**: System Admin, Subscriber-Admin (own organization), Teacher-Admin (own scope)
**View Roster Sync Logs**: Teacher-Admin (own scope), Subscriber-Admin (own organization), System Admin
**Approve Role Changes**: Subscriber-Admin (own organization), System Admin

### LTI Launches

**Launch MLC via LTI**: Any user with valid LTI credentials from registered platform
**View Launch History**: User (own history), Teacher (own students), Teacher-Admin (own scope), Subscriber-Admin (own organization), System Admin

## Supporting Documents Referenced

This LMS LTI specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role mapping for LMS integration
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — LMS integration messaging and workflows

## Dependencies

### Internal Dependencies

**Authentication System** ([Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md)):
- LTI launches create authenticated MLC sessions
- User provisioning integrates with MLC user account system
- Role mapping depends on MLC role hierarchy

**User and Organization Model** ([Roles Matrix](../02-roles-and-permissions/roles-matrix.md)):
- LTI users mapped to MLC roles
- LMS contexts mapped to MLC organizations or classes
- Subscription seat allocation applies to LTI-provisioned users

**Assignment Model** ([Assignment Model](../07-content-architecture/assignment-model.md)):
- LTI resource links can reference MLC assignments
- Assignment completion triggers grade sync
- Teacher assignment builder used in deep linking content picker

**Game Registry** ([Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md)):
- Deep linking allows selection of individual games
- Game metadata (name, description, thumbnail) used in content picker
- Game scores drive grade passback

**Sequence Model** ([Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)):
- LTI resource links can reference sequences
- Sequence completion percentage used for grading
- Sequences available in deep linking picker

**Event Model** ([Event Model](../15-analytics-and-reporting/event-model.md)):
- All LTI interactions tracked via standard event system
- Grade sync events logged for audit trail
- Launch analytics inform platform health monitoring

**API Design** ([API Design Principles](../18-architecture/api-design-principles.md)):
- LTI endpoints follow platform API conventions
- RESTful design for LTI admin interfaces
- OAuth 2.0 token management for AGS/NRPS

**Security and Privacy** ([Security Privacy](../18-architecture/security-privacy.md)):
- JWT validation and cryptographic verification
- HTTPS enforcement
- Data privacy compliance (COPPA/FERPA)

### External Dependencies

**LTI 1.3 Specification** (IMS Global / 1EdTech):
- LTI 1.3 Core: Authentication, launch protocol
- LTI Deep Linking 2.0: Content selection and placement
- LTI Assignment and Grade Services (AGS): Grade passback
- LTI Names and Role Provisioning Services (NRPS): Roster sync

**OAuth 2.0 and JWT**:
- OAuth 2.0 client credentials flow for AGS/NRPS tokens
- JSON Web Tokens (JWT) for signed launch messages
- JWKS (JSON Web Key Set) for signature verification

**LMS Platforms**:
- Canvas, Moodle, Blackboard, D2L Brightspace, Schoology, etc.
- Each platform must support LTI 1.3 (Advantage)
- Platform-specific quirks and configuration documented separately

**SSL/TLS Certificates**:
- Valid HTTPS certificate required for all LTI endpoints
- Certificate must be trusted by LMS platforms

**Background Job Queue** ([Background Jobs](../18-architecture/background-jobs.md)):
- Grade sync operations processed asynchronously
- Roster sync jobs scheduled via queue
- Retry logic for failed operations

**Caching Layer** ([Caching Strategy](../18-architecture/caching-strategy.md)):
- OAuth tokens cached until near expiration
- JWKS cached with TTL (24 hours)
- Platform configuration cached

**Database**:
- Store LTI platform registrations, user mappings, resource links
- Index on foreign keys for performance
- Transactional integrity for roster sync operations

## Open questions

### Technical Decisions Needed

**LTI 1.1 Support**: Should we provide a limited LTI 1.1 fallback for legacy LMS platforms, or strictly require LTI 1.3? 
- **Trade-off**: LTI 1.1 has security limitations (shared secrets, no JWT) but would expand compatibility with older installations.
- **Recommendation**: Start with LTI 1.3 only; add LTI 1.1 only if customer demand warrants.

**Grade Sync Timing**: Should grade sync be real-time (on game completion) or batched (every 15 minutes)?
- **Real-time**: Better UX for students, but higher API load on LMS platforms.
- **Batched**: Reduces API calls, but delays grade visibility.
- **Recommendation**: Hybrid approach - real-time for Quiz/Review completion, batched for partial progress updates.

**User Matching Strategy**: When LTI user has same email as existing MLC user, should we auto-match or require manual confirmation?
- **Auto-match**: Seamless UX, but risk of incorrect match if email shared or typo.
- **Manual confirmation**: Safer, but adds friction.
- **Recommendation**: Auto-match if email verified in both systems, else prompt admin for confirmation.

**Context Mapping**: Should one LMS course map to one MLC organization, one class, or allow flexible mapping?
- **One-to-one (course to organization)**: Simple, but may not fit all school structures.
- **One-to-many (course to multiple classes)**: Flexible, but complex admin UX.
- **Recommendation**: Default one course to one class, with admin override to map to organization or multiple classes.

**Deep Linking Limits**: Should we limit the number of items teachers can select at once (e.g., max 10)?
- **No limit**: Maximum flexibility, but could overwhelm LMS or cause UI issues.
- **Limit**: Safer, encourages focused assignments.
- **Recommendation**: Suggest limit of 10 items per deep linking session; make configurable.

### Product and UX Questions

**Iframe vs Full Window**: Should LTI launches default to iframe embed or full window?
- **Iframe**: Keeps user in LMS context, better navigation continuity.
- **Full window**: More screen space, better for complex games.
- **Recommendation**: Detect LMS preference from launch parameters; default to iframe with "Open in New Window" option.

**Return to LMS Behavior**: When should "Return to LMS" button appear, and where should it go?
- Always visible in header vs. only on completion screens.
- Return to launch page vs. course home vs. gradebook.
- **Recommendation**: Always visible in header, return to LTI return_url (provided by LMS).

**Grade Sync Failure Notifications**: Should students/teachers be notified when grade sync fails?
- **Notify immediately**: Increases awareness, but may cause alarm if transient issue.
- **Don't notify**: Reduces noise, but users may wonder why grade not in LMS.
- **Recommendation**: Don't notify on first failure (will retry), notify teacher if still failing after 24 hours.

**Roster Sync Frequency**: How often should roster sync run?
- **Real-time (on launch)**: Always up to date, but performance overhead.
- **Daily**: Reasonable freshness, low overhead.
- **Manual only**: Full control, but risk of stale data.
- **Recommendation**: Daily scheduled sync at off-peak hours (2 AM), plus manual trigger option.

### Policy and Business Questions

**Seat Allocation for LTI Users**: Do LTI-provisioned users count against subscription seats? How are seats allocated?
- **Count against seats**: Standard billing model, but may surprise admins if LMS roster is large.
- **Separate LTI seat pool**: Clearer, but complex billing.
- **Recommendation**: Count against seats; provide clear roster sync preview before auto-provisioning; alert admin if approaching seat limit.

**LTI as Standalone vs Add-on**: Should LTI integration be available to all subscription tiers, or only Ensemble and above?
- **All tiers**: Maximum accessibility, but Solo users unlikely to need LTI.
- **Ensemble only**: Aligns with school use case, simpler to support.
- **Recommendation**: Available for Ensemble tier and above; Prelude (free trial) excluded due to limited provisioning.

**Data Retention for LTI Users**: If school stops LTI integration, what happens to student data?
- **Delete immediately**: Clean, but loses history.
- **Retain per standard policy**: Consistent with non-LTI users, preserves learning history.
- **Recommendation**: Retain per standard data retention policy (13 months after subscription ends); provide export option.

**Grade Override by Teacher**: If teacher manually adjusts grade in MLC, should that override LTI-synced grade in LMS?
- **Allow override**: Teacher control, but may confuse if LMS grade differs.
- **Sync override back to LMS**: Consistent, but requires AGS write permission.
- **Recommendation**: Allow override in MLC, sync updated grade to LMS with "Teacher adjusted" comment.

### Platform-Specific Considerations

**Canvas Deep Linking Quirks**: Canvas has specific behavior for content item types (e.g., `ltiResourceLink` vs `link`). How do we handle?
- **Recommendation**: Test with Canvas sandbox, document Canvas-specific configuration in implementation guide.

**Moodle Grade Scale Mapping**: Moodle uses different grade scale formats (points, percentage, letter). How do we map MLC scores?
- **Recommendation**: Always send numeric score + max score; let Moodle apply its scale.

**Blackboard Token Expiration**: Blackboard tokens may expire faster than other platforms. How do we handle?
- **Recommendation**: Implement aggressive token refresh logic; cache tokens conservatively (50% of expires_in).

**D2L Context Roles**: D2L has extensive context role types. Do we need to map all of them?
- **Recommendation**: Map common roles; treat unmapped roles as generic "Learner" with alert to admin.

### Future Enhancements (Out of Scope for Initial Release)

**LTI 1.1 Support**: Add LTI 1.1 for legacy LMS platforms (if customer demand exists).

**Due Date Sync**: Pull assignment due dates from LMS, enforce in MLC.

**Proctoring Integration**: LTI Proctoring Services for high-stakes assessments (future).

**Caliper Analytics**: Send learning analytics via IMS Caliper specification (alternative to xAPI).

**LTI Names Only (No Roster Sync)**: Support platforms that provide names in launch but not NRPS.

**Multi-Tenancy**: Support multiple LMS platforms per organization (e.g., Canvas for K-8, Schoology for 9-12).

**White-label LTI**: Custom branding for LTI launches (per organization).

---

## Implementation Notes

**Phase 1 (MVP)**:
- LTI 1.3 Core launch and authentication
- User auto-provisioning and role mapping
- Single game resource links (no deep linking)
- Grade passback for Quiz stage scores
- System Admin platform registration UI

**Phase 2 (Enhanced)**:
- Deep Linking 2.0 for content selection
- Sequence and assignment resource links
- Roster sync via NRPS
- Grade sync for assignments
- Teacher-Admin and Subscriber-Admin configuration access

**Phase 3 (Advanced)**:
- Real-time grade sync with batched fallback
- Advanced conflict resolution for roster sync
- Platform-specific optimizations (Canvas, Moodle, Blackboard)
- Comprehensive diagnostics and monitoring dashboard
- White-label and multi-tenancy support (if business case warrants)

**Testing Requirements**:
- LTI Certification Suite (IMS Global) for compliance
- Integration testing with Canvas, Moodle, Blackboard sandbox environments
- Load testing for grade sync queue (1000 scores/minute)
- Security audit of JWT validation and OAuth flows
- User acceptance testing with pilot schools

**Documentation Deliverables**:
- LTI Admin Setup Guide (for System Admins)
- LMS Configuration Guides (Canvas, Moodle, Blackboard)
- Teacher User Guide for Deep Linking
- API Reference for LTI Endpoints
- Troubleshooting Guide for Common Issues

**Related Specifications**:
- [SSO Integration](./sso-integration.md) - For direct SAML/OAuth SSO without LTI
- [xAPI Implementation](./xapi-implementation.md) - For detailed learning analytics
- [SCORM Compliance](./scorm-compliance.md) - For packaged content export
- [API Design Principles](../18-architecture/api-design-principles.md) - For LTI endpoint design
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - For LTI session creation
