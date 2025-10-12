# System Announcements

## Purpose

This specification defines the system announcement authoring, management, and scheduling infrastructure for the MLC-LMS platform. System announcements enable administrators to create and broadcast important communications to targeted groups of users across multiple delivery channels (in-app, email, push notifications).

The announcement system serves as the centralized content management and scheduling interface for broadcast communications, ranging from feature updates and scheduled maintenance notices to emergency alerts and organizational news. Unlike individual notifications triggered by user actions, system announcements are manually authored communications that require careful targeting, scheduling, and delivery coordination.

This system provides Subscriber-Admins and System Admins with the tools to craft, preview, schedule, and track broadcast messages while ensuring appropriate audience segmentation, delivery timing, and multi-channel coordination. The announcement management interface balances ease of authoring with governance controls to prevent announcement fatigue and maintain communication quality across the platform.

## Scope

**Included**
- Announcement authoring interface and rich text editor
- Audience targeting and segmentation rules
- Announcement scheduling and publishing workflow
- Draft, review, and approval states for announcements
- Announcement templates and reusable content blocks
- Organization-level vs platform-wide announcement scoping
- Multi-channel delivery coordination (in-app, email, push)
- Urgency levels and priority classification
- Display duration and expiration rules
- Announcement preview and testing tools
- Announcement history and archiving
- Recipient tracking and engagement metrics
- Emergency/critical announcement override capabilities
- Recurring announcement scheduling (maintenance windows, newsletters)
- Announcement categories and classification taxonomy

**Excluded**
- In-app notification delivery infrastructure (see [In-app Notifications](./in-app-notifications.md))
- Email template rendering and delivery (see [Email Templates](./email-templates.md))
- Push notification infrastructure (see [Push and Desktop](./push-and-desktop.md))
- User notification preferences and opt-in management (defined in role experience docs)
- Individual user-to-user messaging (see [Student Messages](../03-student-experience/student-messages.md), [Teacher Messaging](../04-teacher-experience/teacher-messaging.md))
- Automated system-generated notifications (see [Event Model](../15-analytics-and-reporting/event-model.md))
- Role-specific notification triggers (see role notification docs)

## Data model (if applicable)

### Announcement Entity

```typescript
interface SystemAnnouncement {
  // Identification
  announcementId: string;          // UUID, e.g., "ann_1a2b3c4d5e6f"
  organizationId?: string;         // Null for platform-wide announcements
  
  // Content
  title: string;                   // Max 120 characters, required
  body: string;                    // Rich text HTML, max 5000 characters
  summary?: string;                // Plain text preview, max 200 characters
  category: AnnouncementCategory;
  
  // Visual Design
  bannerImage?: string;            // CDN URL for hero/banner image
  iconType?: string;               // Icon identifier for announcement type
  accentColor?: string;            // Hex color for accent elements
  layoutTemplate: 'basic' | 'hero' | 'urgent' | 'newsletter';
  
  // Targeting
  audience: AudienceSegment;
  targetRoles: UserRole[];         // Which role types see this
  targetOrganizations?: string[];  // For platform admins targeting specific orgs
  excludeUserIds?: string[];       // Opt-out or override exclusions
  
  // Call to Action
  callToAction?: {
    label: string;                 // Button text, e.g., "Learn More"
    targetUrl: string;             // Internal route or external URL
    opensInNewTab: boolean;
  };
  
  // Scheduling
  status: AnnouncementStatus;
  publishAt?: Date;                // Scheduled publish time (ISO 8601)
  expiresAt?: Date;                // Auto-archive after this date
  timezone: string;                // IANA timezone for scheduling
  
  // Priority and Urgency
  priority: 'critical' | 'high' | 'normal' | 'low';
  isUrgent: boolean;               // Shows as urgent banner/blocking modal
  dismissible: boolean;            // Can users dismiss or must acknowledge?
  requiresAcknowledgment: boolean; // User must click "I Understand" or similar
  
  // Delivery Channels
  deliveryChannels: {
    inApp: boolean;                // Show in notification center and/or banner
    email: boolean;                // Send via email
    push: boolean;                 // Send push notification
    desktop: boolean;              // Desktop OS notification
  };
  inAppDisplayMode: 'banner' | 'modal' | 'notification_center' | 'all';
  
  // Versioning and Approval
  version: number;                 // Increments with each edit
  createdBy: string;               // User ID of author
  lastEditedBy: string;
  approvedBy?: string;             // User ID of approver
  approvedAt?: Date;
  
  // Audit Trail
  createdAt: Date;
  updatedAt: Date;
  publishedAt?: Date;
  archivedAt?: Date;
  
  // Analytics
  recipientCount: number;          // Total users targeted
  deliveredCount: number;          // Successfully delivered
  viewedCount: number;             // Users who viewed
  clickedCount: number;            // Users who clicked CTA
  acknowledgedCount: number;       // Users who acknowledged (if required)
  
  // Metadata
  tags: string[];                  // Searchable tags for organization
  relatedAnnouncementIds?: string[]; // Link to related announcements
  externalReferenceId?: string;    // For integration with external systems
}
```

### AnnouncementCategory Enum

```typescript
type AnnouncementCategory =
  // System & Technical
  | 'feature_update'
  | 'maintenance_scheduled'
  | 'maintenance_emergency'
  | 'service_disruption'
  | 'security_alert'
  | 'bug_fix'
  
  // Educational & Content
  | 'new_content'
  | 'curriculum_update'
  | 'teaching_tip'
  | 'resource_available'
  
  // Community & Events
  | 'event_announcement'
  | 'contest_challenge'
  | 'achievement_highlight'
  | 'newsletter'
  
  // Administrative
  | 'policy_change'
  | 'billing_update'
  | 'account_action_required'
  | 'organization_news'
  
  // Compliance & Legal
  | 'privacy_update'
  | 'terms_update'
  | 'compliance_notice';
```

### AnnouncementStatus Enum

```typescript
type AnnouncementStatus =
  | 'draft'              // Being authored, not visible to users
  | 'pending_approval'   // Submitted for review (org policy dependent)
  | 'approved'           // Approved but not yet published
  | 'scheduled'          // Scheduled for future publication
  | 'published'          // Currently active and visible
  | 'expired'            // Past expiration date, auto-archived
  | 'archived'           // Manually archived, no longer visible
  | 'cancelled';         // Cancelled before publication
```

### AudienceSegment Interface

```typescript
interface AudienceSegment {
  // Scope Level
  scope: 'platform' | 'organization' | 'custom';
  
  // Role Targeting
  roles: UserRole[];               // 'student', 'teacher', etc.
  
  // Organization Targeting (for platform admins)
  organizationIds?: string[];
  organizationTypes?: ('solo' | 'ensemble')[];
  subscriptionPlans?: ('prelude' | 'solo' | 'ensemble')[];
  
  // User Attributes
  userAttributes?: {
    gradeLevel?: string[];         // For students
    courseAssignments?: string[];  // Students in specific sequences
    lastActivityDate?: {           // Activity-based targeting
      operator: 'before' | 'after' | 'between';
      date: Date | Date[];
    };
    accountAge?: {                 // New vs existing users
      operator: 'less_than' | 'greater_than';
      days: number;
    };
  };
  
  // Geographic Targeting (future)
  regions?: string[];              // ISO country codes
  timezones?: string[];            // IANA timezone identifiers
  
  // Estimated Reach
  estimatedRecipientCount?: number; // Calculated on save
}
```

### AnnouncementRecipient Entity

```typescript
interface AnnouncementRecipient {
  recipientId: string;             // UUID
  announcementId: string;
  userId: string;
  
  // Delivery Status
  deliveryStatus: {
    inApp: 'pending' | 'delivered' | 'failed' | null;
    email: 'pending' | 'sent' | 'delivered' | 'bounced' | null;
    push: 'pending' | 'sent' | 'delivered' | 'failed' | null;
    desktop: 'pending' | 'sent' | 'delivered' | 'failed' | null;
  };
  
  // Engagement
  viewedAt?: Date;                 // First time user viewed
  viewCount: number;               // Total times viewed
  clickedAt?: Date;                // First CTA click
  acknowledgedAt?: Date;           // User acknowledged (if required)
  dismissedAt?: Date;              // User dismissed announcement
  
  // Timing
  targetedAt: Date;                // When user was added to recipient list
  firstDeliveredAt?: Date;         // First successful delivery on any channel
  
  // Metadata
  deliveryAttempts: number;
  lastDeliveryAttemptAt?: Date;
}
```

### AnnouncementTemplate Entity

```typescript
interface AnnouncementTemplate {
  templateId: string;              // UUID
  templateName: string;
  category: AnnouncementCategory;
  
  // Template Content
  titleTemplate: string;           // With variable placeholders
  bodyTemplate: string;            // Rich text with placeholders
  summaryTemplate?: string;
  
  // Default Settings
  defaultPriority: 'critical' | 'high' | 'normal' | 'low';
  defaultChannels: {
    inApp: boolean;
    email: boolean;
    push: boolean;
    desktop: boolean;
  };
  defaultLayoutTemplate: string;
  
  // Variables
  variables: TemplateVariable[];
  
  // Metadata
  createdBy: string;
  createdAt: Date;
  usageCount: number;
  isActive: boolean;
}

interface TemplateVariable {
  key: string;                     // e.g., "maintenance_start_time"
  displayName: string;
  dataType: 'string' | 'date' | 'number' | 'url';
  defaultValue?: string;
  isRequired: boolean;
  example: string;
}
```

## Behavior and rules

### Announcement Authoring Workflow

#### Draft Creation
1. **Initiate**: Admin clicks "Create Announcement" in admin dashboard
2. **Select template** (optional): Choose from predefined templates or start blank
3. **Author content**:
   - Enter title (required, 120 char max)
   - Compose body using rich text editor (5000 char max)
   - Add optional summary for preview text
   - Upload optional banner image
4. **Configure targeting**:
   - Select audience scope (platform/organization/custom)
   - Define target roles
   - Apply user attribute filters if custom
   - View estimated recipient count
5. **Set delivery options**:
   - Choose delivery channels (in-app, email, push, desktop)
   - Select in-app display mode
   - Configure priority and urgency
   - Set dismissibility and acknowledgment requirement
6. **Schedule publication**:
   - Publish immediately or schedule for future
   - Set expiration date (optional)
   - Select timezone for scheduling
7. **Save as draft**: Announcement saved with `status: 'draft'`

#### Preview and Testing
- **Preview mode**: Shows announcement as it will appear in each channel
  - In-app: Renders banner/modal/notification
  - Email: Generates email template preview
  - Push: Shows notification preview with character limits
- **Test delivery**: Send test to specific users (admin themselves or test accounts)
- **Recipient count calculation**: Real-time estimate of targeted users
- **Validation**: System checks for required fields, image sizes, character limits

#### Approval Workflow (Organization Policy Dependent)

**When approval required** (org setting):
1. Author submits draft → `status: 'pending_approval'`
2. Designated approver receives notification
3. Approver reviews content, targeting, timing
4. Approver can:
   - **Approve**: → `status: 'approved'`
   - **Request changes**: Returns to draft with comments
   - **Reject**: → `status: 'archived'` with rejection reason
5. Once approved, author can publish or schedule

**When approval not required** (default for most orgs):
- Draft can be immediately published by author

#### Publication Rules

**Immediate Publication**:
- Status changes from `draft` or `approved` to `published`
- `publishedAt` timestamp set to current time
- Delivery begins immediately based on channel configuration
- Announcement becomes visible to targeted users

**Scheduled Publication**:
- Status changes to `scheduled`
- `publishAt` timestamp set to future date/time
- Background job monitors scheduled announcements
- At scheduled time:
  - Status changes to `published`
  - Delivery begins
  - `publishedAt` timestamp set

**Publication Validation**:
- Required fields: title, body, audience, at least one delivery channel
- Title length: 1-120 characters
- Body length: 1-5000 characters
- Banner image: max 2MB, formats: JPG, PNG, WebP
- Valid expiration date: must be future date if set
- Recipient count: must be >0 (at least one user targeted)

### Audience Targeting Rules

#### Scope Levels

**Platform-wide** (System Admin only):
- Announcement visible to all users across all organizations
- Typically used for: platform maintenance, critical security updates, major feature releases
- Examples: "Scheduled Maintenance: January 15, 2025", "New Mobile App Available"

**Organization-level** (Subscriber-Admin and System Admin):
- Announcement visible only to users within specific organization
- Scoped to single organization or multiple (platform admin)
- Typically used for: org-specific news, policy changes, local events
- Examples: "Welcome New Students!", "Spring Recital Registration Open"

**Custom** (Advanced targeting):
- Fine-grained targeting based on user attributes
- Combine role, activity, and account criteria
- Typically used for: targeted engagement campaigns, feature adoption, re-activation
- Examples: "Inactive Student Reminder", "New Teacher Resources Available"

#### Role Targeting Logic

**Single role**:
- Announcement delivered only to users with specified role
- Example: Target only "student" role for student-specific feature

**Multiple roles**:
- Announcement delivered to users with ANY of specified roles (OR logic)
- Example: Target "teacher" AND "teacher_admin" for teaching resource announcement

**All roles**:
- Announcement delivered to all user types
- Example: Platform-wide maintenance notice

#### Exclusion Rules

- **Individual exclusions**: Specific user IDs can be excluded (opt-out, testing)
- **Inactive users**: Optionally exclude users with no activity in X days
- **Unsubscribed users**: Email channel respects email opt-out preferences
- **Device limitations**: Push notifications only sent to users with registered devices

### Priority and Urgency Handling

#### Priority Levels

**Critical** (`priority: 'critical'`):
- Security alerts, account suspension, immediate action required
- In-app: Blocking modal, cannot be dismissed without acknowledgment
- Email: Sent immediately, bypasses batching
- Push: High-priority push, delivered immediately
- Examples: "Security Alert: Password Reset Required", "Account Locked"

**High** (`priority: 'high'`):
- Important but non-blocking announcements
- In-app: Prominent banner at top of page, dismissible
- Email: Sent within 15 minutes, priority queue
- Push: Normal priority, delivered quickly
- Examples: "Scheduled Maintenance in 1 Hour", "New Feature Available"

**Normal** (`priority: 'normal'`):
- Standard announcements, informational updates
- In-app: Notification center entry, optional banner
- Email: Batched with other notifications (if digest enabled)
- Push: Normal priority, respects quiet hours
- Examples: "New Games Added", "Monthly Newsletter"

**Low** (`priority: 'low'`):
- Tips, suggestions, non-urgent information
- In-app: Notification center only, no banner
- Email: Batched, sent during optimal engagement times
- Push: Not sent (typically)
- Examples: "Teaching Tip of the Week", "Did You Know?"

#### Urgency Flag

**When `isUrgent: true`**:
- Overrides normal display logic
- Shows as urgent banner with distinct styling (red/orange accent)
- May include countdown timer if time-sensitive
- Used for: emergency maintenance, critical bugs, urgent action needed
- Examples: "Emergency Maintenance in Progress", "Critical Bug Fix Required"

### Delivery Channel Coordination

#### Multi-channel Delivery Strategy

**In-app + Email** (most common):
- In-app notification delivered immediately when user next logs in
- Email sent to provide offline access to content
- User can engage with either channel
- Engagement tracked separately per channel

**In-app + Email + Push**:
- Push notification alerts user to log in
- In-app shows full content when user arrives
- Email provides backup/reference
- Push includes deep link to relevant in-app screen

**In-app only**:
- Used for non-critical, in-context announcements
- Examples: Feature tips, inline help, contextual guidance

**Email only**:
- Used for newsletters, digests, detailed content
- Examples: Monthly roundup, detailed policy changes

#### Channel-specific Adaptations

**In-app**:
- Full rich text rendering
- Images and media embedded
- Interactive elements (buttons, links)
- Persistent until dismissed or expired

**Email**:
- Rendered using email template (see [Email Templates](./email-templates.md))
- Subject line auto-generated from announcement title
- Plain-text fallback generated
- Unsubscribe footer included (if promotional)

**Push**:
- Title truncated to platform limits (iOS: 65 chars, Android: 50 chars)
- Body truncated (iOS: 178 chars, Android: 240 chars)
- Single action button (maps to CTA)
- Deep link to in-app announcement view

**Desktop**:
- Native OS notification format
- Brief title and summary only
- Click opens application to full announcement

### Expiration and Archival

#### Auto-expiration Rules

**Time-based expiration**:
- Announcement automatically archived when `expiresAt` date reached
- Status changes: `published` → `expired` → `archived`
- No longer visible to users in any channel
- Viewable in announcement history by admins

**Default expiration by category**:
- Feature updates: 30 days
- Maintenance notices: 7 days after maintenance window
- Security alerts: 90 days
- Newsletters: 90 days
- Policy changes: No expiration (persistent)

**Manual expiration**:
- Admin can manually archive published announcement at any time
- Immediately removes from all user views
- Useful for outdated information or corrections

#### Announcement History

**Archived announcements**:
- Retained in database for audit/compliance
- Viewable by admins in "Announcement History"
- Searchable by date, category, author, status
- Analytics and engagement metrics preserved
- Can be duplicated/cloned for reuse

**Retention policy**:
- Archived announcements retained for 3 years
- After 3 years, anonymized (user engagement data removed)
- Announcement content retained indefinitely for compliance

### Recurring Announcements

#### Scheduled Recurrence

**Use cases**:
- Weekly newsletter
- Monthly maintenance windows
- Quarterly policy reminders
- Seasonal event notifications

**Recurrence configuration**:
```typescript
interface RecurrenceConfig {
  enabled: boolean;
  frequency: 'daily' | 'weekly' | 'monthly' | 'quarterly' | 'yearly';
  interval: number;              // Every X days/weeks/months
  daysOfWeek?: number[];         // For weekly (0=Sunday, 6=Saturday)
  dayOfMonth?: number;           // For monthly (1-31)
  endDate?: Date;                // Stop recurrence after this date
  occurrences?: number;          // Or stop after X occurrences
}
```

**Recurrence behavior**:
- System creates new announcement instance for each occurrence
- Each instance has unique `announcementId`
- Each instance tracks its own engagement metrics
- Original announcement marked as "recurring template"
- Recurring announcements can be edited:
  - **This occurrence only**: Edit single instance
  - **This and future occurrences**: Update template and future instances
  - **All occurrences**: Update template and all instances (historical and future)

### Acknowledgment and Dismissal

#### Dismissible Announcements

**When `dismissible: true`** (default):
- User can close/dismiss announcement
- Dismissed announcement no longer shown to that user
- User can view in notification history if needed
- Dismissal tracked for analytics

**When `dismissible: false`**:
- User cannot dismiss announcement
- Remains visible until expiration or manual archive
- Used for critical, persistent announcements
- Examples: Urgent account action, policy acceptance

#### Acknowledgment Required

**When `requiresAcknowledgment: true`**:
- User must explicitly acknowledge announcement
- Modal/banner includes "I Understand" or "Acknowledge" button
- User cannot proceed without acknowledging (blocking)
- Acknowledgment timestamp tracked
- Used for: Critical security updates, policy changes, mandatory training

**Acknowledgment workflow**:
1. User sees announcement (modal or banner)
2. User reads content
3. User clicks "Acknowledge" button
4. `acknowledgedAt` timestamp recorded
5. Announcement dismissed
6. User can proceed with platform use

**Compliance reporting**:
- Admin can view list of users who have/haven't acknowledged
- Export acknowledgment report (CSV)
- Useful for audit trail, policy compliance

## UX requirements (if applicable)

### Announcement Authoring Interface

#### Layout Structure (Admin Dashboard)

```
┌─────────────────────────────────────────────────────────────┐
│  System Announcements                          [Create New]  │
├─────────────────────────────────────────────────────────────┤
│  Filters: [All Status ▼] [All Categories ▼] [Search...]     │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Active Announcements (3)                                    │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🔴 Maintenance Scheduled: Jan 15        [Edit] [⋮]  │   │
│  │ Published • Expires in 2 days • 45 views            │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🎉 New Games Available!                 [Edit] [⋮]  │   │
│  │ Published • 235 views • 89% engagement              │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 📘 Teaching Tips Newsletter             [Edit] [⋮]  │   │
│  │ Scheduled for Jan 10 at 8:00 AM                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                               │
│  Drafts (2)                                                  │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ Contest Announcement                    [Edit] [⋮]  │   │
│  │ Draft • Last edited 2 hours ago                     │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                               │
│  [View Archived] [View Analytics]                           │
└─────────────────────────────────────────────────────────────┘
```

#### Announcement Editor (Create/Edit View)

```
┌─────────────────────────────────────────────────────────────┐
│  ← Back to Announcements              [Preview] [Save Draft] │
│                                              [Publish Now ▼]  │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  📋 Basic Information                                        │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Title *                                                │ │
│  │ [New Feature: Assignment Reminders                  ] │ │
│  │ 120 character limit (42 remaining)                    │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Category *                                             │ │
│  │ [Feature Update              ▼]                       │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Summary (optional)                                     │ │
│  │ [Brief preview shown in notifications...            ] │ │
│  │ 200 character limit (158 remaining)                   │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Body Content *                                         │ │
│  │ [B] [I] [Link] [List] [Image] [Video]                │ │
│  │ ┌─────────────────────────────────────────────────┐  │ │
│  │ │ We're excited to announce assignment reminders!  │  │ │
│  │ │                                                   │  │ │
│  │ │ You'll now receive notifications when...         │  │ │
│  │ └─────────────────────────────────────────────────┘  │ │
│  │ 5000 character limit (4,827 remaining)                │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Banner Image (optional)                               │ │
│  │ [📁 Upload Image] or [Drag & Drop]                   │ │
│  │ Max 2MB • JPG, PNG, WebP • Recommended: 1200x400px   │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  🎯 Audience Targeting                                       │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Scope                                                  │ │
│  │ ● Organization (This organization only)               │ │
│  │ ○ Platform-wide (All organizations)                   │ │
│  │ ○ Custom (Advanced targeting)                         │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Target Roles *                                         │ │
│  │ ☑ Students    ☑ Teachers    ☑ Teacher-Admins         │ │
│  │ ☑ Admins      ☐ System Admins                        │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
│  📊 Estimated Recipients: ~1,247 users                      │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  📬 Delivery Channels                                        │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ ☑ In-app notification                                 │ │
│  │   Display as: ● Banner  ○ Modal  ○ Center only       │ │
│  │                                                        │ │
│  │ ☑ Email notification                                  │ │
│  │                                                        │ │
│  │ ☐ Push notification (mobile devices)                 │ │
│  │ ☐ Desktop notification                                │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ⚙️ Options                                                  │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Priority: [Normal        ▼]                           │ │
│  │ ☐ Mark as urgent                                      │ │
│  │ ☑ Allow users to dismiss                             │ │
│  │ ☐ Require acknowledgment                              │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ Call to Action (optional)                             │ │
│  │ Button Label: [Learn More                          ]  │ │
│  │ Target URL:   [/help/assignment-reminders          ]  │ │
│  │ ☐ Open in new tab                                     │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  📅 Scheduling                                               │
│  ┌───────────────────────────────────────────────────────┐ │
│  │ ● Publish immediately                                 │ │
│  │ ○ Schedule for later                                  │ │
│  │   Date: [Jan 10, 2025 ▼]  Time: [08:00 AM ▼]        │ │
│  │   Timezone: [America/Chicago ▼]                      │ │
│  │                                                        │ │
│  │ Expiration (optional)                                 │ │
│  │ Expires on: [Jan 30, 2025 ▼]                         │ │
│  │ ☐ Set up recurring announcement                       │ │
│  └───────────────────────────────────────────────────────┘ │
│                                                               │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  [Cancel]    [Save Draft]    [Preview]    [Publish Now ▼]   │
│                                           └─ Schedule Publish │
│                                              └─ Send Test     │
└─────────────────────────────────────────────────────────────┘
```

#### Preview Mode

**Multi-channel Preview**:
- Tabs for each delivery channel: "In-app" | "Email" | "Push" | "Desktop"
- Real-time preview updates as content is edited
- Shows how announcement appears on different devices/platforms
- Includes character truncation for push notifications
- Preview reflects targeting (shows "Will be sent to X users")

**Preview Views**:

**In-app Banner Preview**:
```
┌─────────────────────────────────────────────────────────────┐
│  🎉 New Feature: Assignment Reminders                    ✕  │
│  You'll now receive notifications when assignments are       │
│  due soon. Never miss a deadline!         [Learn More]      │
└─────────────────────────────────────────────────────────────┘
```

**In-app Modal Preview**:
```
        ┌───────────────────────────────────┐
        │  🎉 New Feature                ✕  │
        ├───────────────────────────────────┤
        │  Assignment Reminders             │
        │                                   │
        │  We're excited to announce...     │
        │  You'll now receive...            │
        │                                   │
        │  [Learn More]      [Got It]       │
        └───────────────────────────────────┘
```

**Email Preview**:
- Shows full email template with header, body, footer
- Subject line preview
- Mobile vs desktop view toggle

**Push Notification Preview**:
```
┌─────────────────────────────────────┐
│  MusicLearningCommunity             │
│  New Feature: Assignment Reminders  │
│  You'll now receive notifications   │
│  when assignments are due soon.     │
└─────────────────────────────────────┘
```

#### Test Delivery

**Send Test Announcement**:
1. Click "Send Test" button in editor
2. Modal appears:
   ```
   ┌───────────────────────────────────────┐
   │  Send Test Announcement               │
   ├───────────────────────────────────────┤
   │  Test Recipients:                     │
   │  [Yourself (admin@example.com)     ]  │
   │  [+ Add Test Recipient]               │
   │                                       │
   │  Channels:                            │
   │  ☑ In-app  ☑ Email  ☐ Push  ☐ Desktop│
   │                                       │
   │  [Cancel]           [Send Test]       │
   └───────────────────────────────────────┘
   ```
3. Test announcement delivered to specified recipients
4. Test recipients see "[TEST]" prefix in notification
5. Test delivery does not affect live recipient counts

### Announcement Display (User-facing)

#### In-app Banner (Normal Priority)

```
┌─────────────────────────────────────────────────────────────┐
│  MusicLearningCommunity Dashboard                           │
├─────────────────────────────────────────────────────────────┤
│  ℹ️ New Games Available! We've added 12 new rhythm games   ✕│
│  to Level 2. Check them out today!        [Play Now]        │
├─────────────────────────────────────────────────────────────┤
│  Student Dashboard Content...                               │
```

#### In-app Banner (Urgent Priority)

```
┌─────────────────────────────────────────────────────────────┐
│  ⚠️ SCHEDULED MAINTENANCE: January 15, 3:00-5:00 PM CST    ✕│
│  The platform will be unavailable for scheduled maintenance. │
│  Please save your work.                   [Learn More]       │
├─────────────────────────────────────────────────────────────┤
│  (Rest of page content...)                                  │
```

#### Blocking Modal (Critical Priority)

```
        ┌─────────────────────────────────────┐
        │  🔐 Security Alert                  │
        ├─────────────────────────────────────┤
        │  For your security, we require      │
        │  all users to reset their           │
        │  passwords within the next 7 days.  │
        │                                     │
        │  This is due to a recent security   │
        │  update to protect your account.    │
        │                                     │
        │  [Reset Password Now]  [Remind Me]  │
        └─────────────────────────────────────┘
                (Page content dimmed behind modal)
```

#### Notification Center Entry

```
┌─────────────────────────────────────────────────────────────┐
│  Notifications                                          ⚙    │
├─────────────────────────────────────────────────────────────┤
│  [All] [Assignments] [System] [Messages]                    │
├─────────────────────────────────────────────────────────────┤
│  System (3 new)                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 🔵 📣 New Feature: Assignment Reminders             │   │
│  │   You'll now receive notifications when...          │   │
│  │   2 hours ago                        [Learn More]   │   │
│  └─────────────────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │    📘 Teaching Tips Newsletter                      │   │
│  │   This month's tips include...                      │   │
│  │   Yesterday                          [Read More]    │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### Accessibility Requirements

Reference: [Accessibility Standards](../01-ux-design-system/a11y-standards.md)

#### Screen Reader Support
- **ARIA landmarks**: Announcement banner has `role="region"` with `aria-label="System Announcement"`
- **Live regions**: Urgent announcements use `aria-live="assertive"`, normal use `aria-live="polite"`
- **Dismissible**: Dismiss button has `aria-label="Dismiss announcement"`
- **Acknowledgment**: Modal has proper focus trap and `aria-modal="true"`

#### Keyboard Navigation
- **Banner**: Focus moves to banner on page load (if critical/urgent)
- **Dismiss**: Escape key dismisses banner (if dismissible)
- **CTA button**: Fully keyboard accessible
- **Modal**: Tab cycles through elements, Escape closes (if dismissible)

#### Visual Accessibility
- **Color contrast**: 4.5:1 minimum for text, 3:1 for interactive elements
- **Color independence**: Urgency conveyed by icon and text, not just color
- **Focus indicators**: Clear 2px outline on focused elements
- **Text scaling**: Supports 200% zoom without breaking layout

### Responsive Behavior

#### Desktop (≥1024px)
- Full-width banner across top of page
- Modal centered on screen (500px width)
- Rich content rendering with images

#### Tablet (768px - 1023px)
- Full-width banner, stacked content
- Modal 90% screen width
- Touch-optimized buttons (44px height)

#### Mobile (<768px)
- Full-width banner, condensed layout
- Modal full-screen slide-up
- Large touch targets throughout
- Truncated text with "Read More" expansion

## Telemetry

Reference: [Event Model](../15-analytics-and-reporting/event-model.md) for complete event tracking specifications.

### Announcement Lifecycle Events

#### Announcement Created
```javascript
{
  event_type: "announcement_created",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    category: "feature_update",
    status: "draft",
    created_by: "usr_admin_123",
    organization_id: "org_456",
    target_roles: ["student", "teacher"],
    estimated_recipients: 1247,
    delivery_channels: ["in_app", "email"]
  }
}
// Fires: Server-side when announcement draft saved
```

#### Announcement Published
```javascript
{
  event_type: "announcement_published",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    category: "feature_update",
    priority: "normal",
    publish_type: "immediate" | "scheduled",
    scheduled_for: "2025-01-10T08:00:00Z",  // If scheduled
    target_recipient_count: 1247,
    delivery_channels: ["in_app", "email"],
    has_expiration: true,
    expires_at: "2025-01-30T23:59:59Z",
    published_by: "usr_admin_123"
  }
}
// Fires: Server-side when announcement goes live
```

### User Engagement Events

#### Announcement Viewed
```javascript
{
  event_type: "announcement_viewed",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    announcement_category: "feature_update",
    user_id: "usr_12345",
    user_role: "student",
    view_channel: "in_app_banner" | "in_app_modal" | "notification_center" | "email",
    time_to_view_seconds: 45,      // Time from delivery to view
    is_first_view: true
  }
}
// Fires: Client-side when announcement becomes visible in viewport
```

#### Announcement Clicked
```javascript
{
  event_type: "announcement_clicked",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    announcement_category: "feature_update",
    user_id: "usr_12345",
    click_target: "cta_button" | "banner" | "link",
    target_url: "/help/assignment-reminders",
    time_since_viewed_seconds: 12,
    channel: "in_app_banner"
  }
}
// Fires: Client-side when user clicks announcement CTA or link
```

#### Announcement Dismissed
```javascript
{
  event_type: "announcement_dismissed",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    announcement_category: "feature_update",
    user_id: "usr_12345",
    time_to_dismiss_seconds: 8,
    was_viewed: true,
    was_clicked: false,
    dismiss_method: "x_button" | "escape_key" | "outside_click"
  }
}
// Fires: Client-side when user dismisses announcement
```

#### Announcement Acknowledged
```javascript
{
  event_type: "announcement_acknowledged",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    announcement_category: "security_alert",
    user_id: "usr_12345",
    time_to_acknowledge_seconds: 45,
    acknowledgment_method: "button_click" | "checkbox_confirm"
  }
}
// Fires: Client-side when user acknowledges announcement (if required)
```

### Admin Management Events

#### Announcement Edited
```javascript
{
  event_type: "announcement_edited",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    edited_by: "usr_admin_123",
    previous_status: "draft",
    new_status: "draft",
    fields_changed: ["title", "body", "target_roles"],
    version: 2
  }
}
// Fires: Server-side when announcement is edited
```

#### Announcement Approved
```javascript
{
  event_type: "announcement_approved",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    approved_by: "usr_admin_456",
    submitted_by: "usr_admin_123",
    approval_duration_seconds: 1200,
    had_requested_changes: false
  }
}
// Fires: Server-side when announcement approved (if approval required)
```

#### Announcement Archived
```javascript
{
  event_type: "announcement_archived",
  properties: {
    announcement_id: "ann_1a2b3c4d",
    archive_method: "manual" | "auto_expire" | "cancelled",
    archived_by: "usr_admin_123",  // If manual
    days_active: 15,
    total_views: 1089,
    total_clicks: 234,
    engagement_rate: 21.5  // Percentage
  }
}
// Fires: Server-side when announcement archived
```

### Aggregated Analytics Metrics

These metrics feed into dashboards and reports:

#### Announcement Performance
- **View rate**: views / total_recipients
- **Click-through rate**: clicks / views
- **Engagement rate**: (views + clicks) / total_recipients
- **Dismissal rate**: dismissals / views
- **Acknowledgment rate**: acknowledgments / total_recipients (if required)
- **Time to engage**: Average time from delivery to first interaction

#### Channel Performance
- **In-app view rate**: in_app_views / delivered
- **Email open rate**: email_opens / emails_sent
- **Push open rate**: push_opens / push_delivered
- **Channel preference**: Which channel users engage with first

#### Category Analytics
- **Performance by category**: Engagement rates by announcement category
- **Optimal timing**: Best publish times by day/hour
- **Audience response**: Engagement rates by role, organization, user segment

#### Admin Activity
- **Announcement frequency**: Announcements published per week/month
- **Author productivity**: Announcements created/published by admin
- **Draft abandonment**: Drafts created but never published
- **Approval bottlenecks**: Average approval time, approval rejection rate

## Permissions

Reference: [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md)

### Announcement Management Permissions

#### System Admin
- **Create announcements**: Platform-wide or organization-specific
- **Edit announcements**: All announcements (any status, any organization)
- **Publish announcements**: Immediate or scheduled, platform-wide or targeted
- **Archive announcements**: Any announcement regardless of author
- **View analytics**: All announcement engagement metrics across platform
- **Manage templates**: Create, edit, delete announcement templates
- **Override targeting**: Bypass audience rules, exclude specific users
- **Emergency broadcast**: Send critical announcements without approval
- **Approval workflow**: Approve organization announcements (if org policy requires)

#### Subscriber-Admin
- **Create announcements**: Organization-level only (cannot create platform-wide)
- **Edit announcements**: Own organization's announcements (draft, scheduled)
- **Publish announcements**: Organization-level, immediate or scheduled
- **Archive announcements**: Own organization's announcements
- **View analytics**: Organization-level announcement metrics only
- **Manage templates**: Create organization-specific templates
- **Target audience**: All roles within organization
- **Approval workflow**: 
  - Can submit announcements for approval (if org policy enabled)
  - Can designate approvers within organization
- **Cannot**: 
  - Create platform-wide announcements
  - View/edit other organizations' announcements
  - Override platform-level targeting rules

#### Teacher-Admin
- **View announcements**: Can view published announcements targeted to their role
- **Cannot create/edit**: No authoring or management capabilities
- **Can provide feedback**: Can report issues or suggest announcements to Subscriber-Admin

#### Teacher
- **View announcements**: Can view published announcements targeted to their role
- **Cannot create/edit**: No authoring or management capabilities
- **Engagement tracking**: Their views/clicks tracked for analytics

#### Student
- **View announcements**: Can view published announcements targeted to their role
- **Cannot create/edit**: No authoring or management capabilities
- **Engagement tracking**: Their views/clicks tracked for analytics
- **Guardian visibility**: Guardian accounts may see student-targeted announcements

### Announcement Visibility Rules

#### Published Announcement Visibility
- **User role match**: User must have one of target roles specified in announcement
- **Organization scope**: User must belong to targeted organization(s)
- **Attribute filters**: User must match custom targeting criteria (if specified)
- **Exclusion list**: User must not be on individual exclusion list
- **Active status**: Announcement must have `status: 'published'`
- **Not expired**: Current date must be before `expiresAt` (if set)

#### Draft/Scheduled Announcement Visibility
- **System Admin**: Can view all drafts and scheduled announcements across platform
- **Subscriber-Admin**: Can view own organization's drafts and scheduled announcements
- **Other roles**: Cannot view draft or scheduled announcements

#### Archived Announcement Visibility
- **System Admin**: Can view all archived announcements with full history
- **Subscriber-Admin**: Can view own organization's archived announcements
- **Other roles**: Cannot view archived announcements
- **Recipient users**: Can view archived announcements they received in notification history

### Approval Workflow Permissions

**Organizations with approval enabled** (org setting):

**Announcement Author** (Subscriber-Admin or designated user):
- Create draft announcement
- Submit for approval → `status: 'pending_approval'`
- Edit after rejection → Returns to `status: 'draft'`
- Cannot self-approve

**Approver** (designated Subscriber-Admin or System Admin):
- View pending announcements
- Approve → `status: 'approved'`, author can publish
- Reject with comments → Returns to `status: 'draft'`, author notified
- Request changes → Returns to `status: 'draft'` with change requests

**System Admin override**:
- Can bypass approval workflow for critical announcements
- Can approve any organization's announcements
- Bypass requires justification (audit trail)

## Dependencies

### Core System Dependencies

- **[In-app Notifications](./in-app-notifications.md)**: In-app delivery channel for announcements (notification center, banners, modals)
- **[Email Templates](./email-templates.md)**: Email delivery channel for announcements (template rendering, sending)
- **[Push and Desktop](./push-and-desktop.md)**: Push and desktop notification delivery channels
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Event tracking for announcement engagement
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: User role definitions for targeting and permissions
- **[Session and Auth](../02-roles-and-permissions/session-and-auth-states.md)**: User authentication for delivery and engagement tracking

### Role-Specific Notification Integration

- **[Student Notifications](../03-student-experience/student-notifications.md)**: How announcements appear to students
- **[Teacher Notifications](../04-teacher-experience/teacher-notifications.md)**: How announcements appear to teachers
- **[Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md)**: How announcements appear to admins

### Admin and Management Tools

- **[Subscriber Dashboard](../05-admin-subscriber-experience/subscriber-dashboard.md)**: Admin dashboard housing announcement management interface
- **[System Admin Content Tools](../06-system-admin/sysadmin-content-tools.md)**: Platform admin tools for managing announcements
- **[System Admin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)**: Audit trail for announcement creation, editing, publishing

### Content and Design

- **[Design System](../01-ux-design-system/components-overview.md)**: UI components for announcement display and authoring
- **[Screen Text and Copy](../01-ux-design-system/screen-text-and-copy.md)**: Writing guidelines for announcement content
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)**: WCAG compliance for announcement UI
- **[Iconography](../01-ux-design-system/iconography.md)**: Icons for announcement categories and priorities

### Technical Infrastructure

- **[Background Jobs](../18-architecture/background-jobs.md)**: Scheduled publication, expiration, recurring announcements
- **[API Design](../18-architecture/api-design-principles.md)**: REST endpoints for announcement CRUD operations
- **[CDN](../17-integrations/cdn-and-media.md)**: Banner image hosting and delivery
- **[Caching Strategy](../18-architecture/caching-strategy.md)**: Announcement caching for performance

### Analytics and Reporting

- **[Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md)**: Announcement engagement metrics in admin dashboards
- **[System Admin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md)**: Platform-wide announcement analytics

### Compliance

- **[Privacy Policy](../23-legal-and-compliance/privacy-policy.md)**: Privacy requirements for user targeting and tracking
- **[COPPA/FERPA Compliance](../23-legal-and-compliance/coppa-ferpa-notes.md)**: Student communication restrictions
- **[GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md)**: EU user communication requirements (if applicable)

## Open questions

### Approval Workflow Design

- **Default approval policy**: Should organizations have approval enabled by default, or opt-in? Recommendation: Opt-in for organizations with >100 users, off by default for smaller orgs.

- **Multi-level approval**: Should we support multi-level approval (e.g., draft → Teacher-Admin review → Subscriber-Admin approval)? Recommendation: Phase 2 feature - start with single approver model.

- **Approval timeout**: Should pending approvals auto-expire after X days? What happens to expired approvals? Recommendation: 7-day timeout, returns to draft with notification.

- **Approval delegation**: Can approvers delegate approval authority to others? Recommendation: Yes, via org settings, with audit trail.

### Content Authoring Tools

- **Rich text editor**: Which editor should we use? Options: Quill (lightweight, customizable), Trix (simple, accessible), TinyMCE (feature-rich, heavy). Recommendation: Quill for balance of features and performance.

- **Media library**: Should we provide a media library for reusable images/assets? Or upload per-announcement? Recommendation: Phase 2 - start with per-announcement upload, add library based on usage patterns.

- **AI assistance**: Should we offer AI-powered content suggestions (headline optimization, grammar check, tone analysis)? Recommendation: Phase 3 - evaluate after platform launch.

- **Collaborative editing**: Should multiple admins be able to collaborate on draft announcements in real-time? Recommendation: Phase 2 - start with single-author model, add collaboration if requested.

### Audience Targeting Complexity

- **Custom segments**: How complex should custom targeting be? Support for AND/OR logic, nested conditions? Recommendation: Start with simple AND logic (user must match ALL conditions), add OR logic in Phase 2.

- **Saved segments**: Should admins be able to save/reuse audience segments (e.g., "Active Teachers," "Inactive Students")? Recommendation: Yes, include in Phase 1 - will reduce repetitive targeting setup.

- **Dynamic segments**: Should segments auto-update based on changing user attributes (e.g., "Students with overdue assignments" updates daily)? Recommendation: Phase 2 - complex implementation, evaluate demand first.

- **A/B testing**: Should we support A/B testing for announcement content/targeting? Recommendation: Phase 3 - valuable for platform admins, but lower priority than core features.

### Delivery Timing Optimization

- **Smart scheduling**: Should system suggest optimal delivery times based on user activity patterns? Recommendation: Phase 2 - requires activity analytics baseline, nice-to-have.

- **Timezone handling**: For platform-wide announcements, send at same local time for all users (e.g., 8 AM in each timezone) or same absolute time? Recommendation: Admin configurable, default to same absolute time.

- **Rate limiting**: Should we limit how many announcements an organization can send per day/week to prevent fatigue? Recommendation: Yes, soft limit of 3 per week with warning, hard limit configurable by platform admin.

- **Quiet hours**: Should announcements respect user notification quiet hours? Recommendation: Yes for normal/low priority, no for high/critical priority.

### Recurring Announcements

- **Recurrence editing**: When editing a recurring announcement, should we always ask "this occurrence only" vs "all future"? Recommendation: Yes, with clear modal explaining impact.

- **Recurrence exceptions**: Can admin skip a single occurrence of recurring announcement without canceling series? Recommendation: Phase 2 - useful but adds complexity.

- **Dynamic content in recurrence**: Should recurring announcements support variables that update each occurrence (e.g., "This week's tip: {{tip_of_week}}")? Recommendation: Phase 3 - requires integration with content management system.

### Engagement and Analytics

- **Read tracking**: How do we accurately track "view" vs "skim"? Require minimum time in viewport? Recommendation: Announcement counts as "viewed" if visible in viewport for 3+ seconds.

- **Engagement benchmarks**: Should we show admins how their announcement engagement compares to platform average? Recommendation: Yes, helpful for admins to gauge effectiveness.

- **User feedback**: Should users be able to provide feedback on announcements (helpful/not helpful, report issue)? Recommendation: Phase 2 - add "Was this helpful?" button with thumbs up/down.

- **Survey integration**: Should announcements support embedded surveys/polls? Recommendation: Phase 3 - complex feature, evaluate demand first.

### Emergency and Critical Announcements

- **Override mechanism**: What's the process for sending emergency announcements that bypass all normal rules? Recommendation: System Admin only, requires incident ticket number, bypasses approval/scheduling.

- **Escalation path**: If critical announcement dismissed, should we re-show after X time or via other channels? Recommendation: Yes, re-show critical announcements every 4 hours until acknowledged or issue resolved.

- **Incident tracking**: Should emergency announcements link to incident status page? Recommendation: Yes, include optional status page URL field for maintenance/incident announcements.

- **Multi-language**: For platform-wide critical announcements, should we auto-translate to user's language? Recommendation: Phase 2 - start with English only, evaluate international user base.

### Privacy and Compliance

- **User opt-out**: Can users opt out of certain announcement categories (e.g., newsletters yes, maintenance no)? Recommendation: Yes, promotional/newsletter announcements respect opt-out, system/critical do not.

- **Data retention**: How long should we retain announcement engagement data? Balance analytics needs with privacy. Recommendation: Active announcements: indefinite, archived announcements: anonymize after 1 year, purge after 3 years.

- **GDPR right to erasure**: When user deletes account, delete their announcement engagement data? Recommendation: Yes, anonymize user_id in engagement records, keep aggregated metrics.

- **Student targeting**: Should we allow targeting students directly, or only via teachers/guardians? COPPA implications. Recommendation: Allow student targeting for educational content, but all email goes to guardian address for students <13.

### Performance and Scale

- **Delivery batching**: For large organizations (1000+ users), batch announcement delivery or send all at once? Recommendation: Batch in groups of 500, stagger by 30 seconds to avoid overwhelming infrastructure.

- **Delivery retries**: How many times to retry failed deliveries? Exponential backoff? Recommendation: 3 retries over 24 hours, exponential backoff (1 hour, 4 hours, 19 hours).

- **Caching strategy**: Cache published announcements or fetch from DB each time? Recommendation: Cache active announcements in Redis for 5 minutes, invalidate on edit/archive.

- **Database partitioning**: Partition announcement tables by date or organization? Recommendation: Partition by created_at date (monthly partitions) for query performance at scale.
