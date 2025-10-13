# In-app Notifications

## Purpose
This specification defines the core in-app notification infrastructure for the MLC-LMS platform. It establishes the technical foundation for delivering timely, relevant, and accessible notifications to all user roles within the application interface. This system ensures users receive important updates about learning progress, system events, and required actions through a consistent, performant notification center and toast notification system.

The in-app notification system serves as the primary real-time communication channel, working in concert with email, push, and desktop notifications to create a comprehensive user notification experience. This specification focuses on the technical delivery mechanism; role-specific notification content and triggers are defined in role experience documents.

## Scope
**Included**
- Core notification data model and storage schema
- Notification center UI component specifications
- Toast notification system (transient in-app alerts)
- Notification delivery pipeline and routing logic
- Read/unread state management and persistence
- Notification grouping, deduplication, and batching rules
- Real-time delivery via WebSocket connections
- Notification badge and counter display logic
- Priority levels and visual hierarchy system
- Accessibility requirements for notification UI
- Performance requirements and offline queuing
- Cross-tab synchronization for multi-window sessions

**Excluded**
- Role-specific notification triggers and content (see [Student Notifications](../03-student-experience/student-notifications.md), [Teacher Notifications](../04-teacher-experience/teacher-notifications.md), [Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md))
- Email notification templates and delivery (see [Email Templates](./email-templates.md))
- Push notification infrastructure (see [Push and Desktop](./push-and-desktop.md))
- System announcement management interface (see [System Announcements](./system-announcements.md))
- Event tracking for notifications (covered in [Event Model](../15-analytics-and-reporting/event-model.md))

## Data model (if applicable)

### Core Notification Entity
```typescript
interface InAppNotification {
  // Identification
  notificationId: string;          // UUID, e.g., "notif_1a2b3c4d5e6f"
  organizationId: string;          // Nullable for system-wide notifications
  
  // Recipient
  userId: string;                  // Target user ID
  userRole: 'student' | 'teacher' | 'teacher_admin' | 'subscriber_admin' | 'system_admin';
  
  // Content
  type: NotificationType;          // See NotificationType enum below
  title: string;                   // Max 120 characters
  body: string;                    // Max 500 characters, supports basic markdown
  icon: string;                    // Icon identifier or emoji
  imageUrl?: string;               // Optional thumbnail/preview image
  
  // Action
  callToAction?: {
    label: string;                 // Button text, e.g., "View Assignment"
    targetRoute: string;           // Internal route path
    targetParams?: Record<string, any>;
  };
  
  // Metadata
  priority: 'blocking' | 'high' | 'normal' | 'low';
  category: 'assignment' | 'challenge' | 'message' | 'system' | 'billing' | 'achievement';
  sourceEventId?: string;          // Links to Event Model event_id for traceability
  groupKey?: string;               // For collapsing similar notifications
  
  // State
  status: 'pending' | 'delivered' | 'read' | 'archived' | 'expired';
  readAt?: Date;                   // ISO 8601 timestamp
  archivedAt?: Date;
  deliveredAt?: Date;
  
  // Timing
  createdAt: Date;                 // ISO 8601 timestamp
  expiresAt?: Date;                // Auto-archive after this date
  
  // Delivery control
  deliveryChannels: ('in_app' | 'email' | 'push' | 'sms')[];
  displayMode: 'toast' | 'center_only' | 'both';
  toastDuration?: number;          // Milliseconds, default 5000
}
```

### NotificationType Enum
```typescript
type NotificationType = 
  // Assignment-related
  | 'assignment_assigned'
  | 'assignment_due_soon'
  | 'assignment_overdue'
  | 'assignment_completed'
  | 'assignment_graded'
  | 'review_scheduled'
  
  // Learning progress
  | 'concept_mastered'
  | 'sequence_completed'
  | 'milestone_achieved'
  | 'placement_result'
  
  // Gamification
  | 'badge_earned'
  | 'challenge_invite'
  | 'challenge_result'
  | 'leaderboard_update'
  | 'streak_at_risk'
  | 'streak_achieved'
  
  // Communication
  | 'message_received'
  | 'message_reply'
  | 'teacher_feedback'
  
  // Administrative
  | 'new_student_enrolled'
  | 'student_needs_attention'
  | 'teacher_performance_alert'
  
  // Billing
  | 'payment_success'
  | 'payment_failed'
  | 'subscription_expiring'
  | 'seat_limit_warning'
  
  // System
  | 'system_announcement'
  | 'maintenance_scheduled'
  | 'feature_update'
  | 'security_alert';
```

### Notification Preferences Entity
```typescript
interface NotificationPreferences {
  userId: string;
  
  // Channel opt-ins (per category)
  preferences: {
    [key in NotificationCategory]: {
      inApp: boolean;              // Always true for critical
      email: boolean;
      push: boolean;
      frequency: 'immediate' | 'hourly' | 'daily' | 'weekly' | 'disabled';
    }
  };
  
  // Quiet hours
  quietHours?: {
    enabled: boolean;
    timezone: string;              // IANA timezone identifier
    startTime: string;             // HH:mm format
    endTime: string;
    daysOfWeek: number[];          // 0=Sunday, 6=Saturday
  };
  
  // Notification limits
  throttling: {
    maxToastsPerSession: number;   // Default: 3
    batchDelay: number;            // Milliseconds, default: 30000
  };
  
  updatedAt: Date;
}
```

### Notification Group Entity
```typescript
interface NotificationGroup {
  groupKey: string;                // Unique identifier for group
  userId: string;
  type: NotificationType;
  count: number;                   // Number of collapsed notifications
  latestNotificationId: string;
  firstOccurredAt: Date;
  lastOccurredAt: Date;
  status: 'active' | 'expanded' | 'dismissed';
}
```

### Real-time Delivery Schema
```typescript
// WebSocket message format
interface NotificationWSMessage {
  action: 'notification_new' | 'notification_updated' | 'notification_deleted' | 'count_update';
  payload: InAppNotification | { notificationId: string } | { unreadCount: number };
  timestamp: Date;
}
```

## Behavior and rules

### Notification Creation and Routing

#### Creation Pipeline
1. **Event trigger**: System event fires (from [Event Model](../15-analytics-and-reporting/event-model.md))
2. **Notification builder**: Evaluates role-specific rules (from role experience docs)
3. **User preference check**: Applies user notification preferences
4. **Quiet hours filter**: Queues or delivers based on quiet hours settings
5. **Deduplication check**: Prevents duplicate notifications within time window
6. **Grouping logic**: Collapses similar notifications using groupKey
7. **Storage**: Persists notification to database
8. **Real-time delivery**: Pushes via WebSocket to active sessions
9. **Badge update**: Increments unread count in header

#### Priority-based Routing
- **Blocking** (`priority: 'blocking'`):
  - Displays as modal overlay requiring acknowledgment
  - Used for critical security alerts, account suspension, mandatory actions
  - Bypasses quiet hours and throttling limits
  - Maximum 1 blocking notification shown at a time
  
- **High** (`priority: 'high'`):
  - Displays as toast notification immediately
  - Used for urgent items: due soon, streak at risk, payment failed
  - Respects quiet hours but shows immediately when hours end
  - Remains in notification center until read
  
- **Normal** (`priority: 'normal'`):
  - Displays as toast if under throttling limit
  - Used for standard updates: assignment completed, badge earned
  - Fully respects quiet hours and throttling
  - Default priority level
  
- **Low** (`priority: 'low'`):
  - Only appears in notification center, no toast
  - Used for informational items: system updates, tips
  - Batched and delivered together to reduce noise

### Notification State Transitions

```
pending → delivered → read → archived
                   ↓
                expired → archived
```

**State rules**:
- `pending`: Created but not yet delivered (queued for quiet hours or batching)
- `delivered`: Pushed to user's active session(s), shown in notification center
- `read`: User clicked notification or visited target route
- `archived`: User manually archived or auto-archived after expiration
- `expired`: Past expiresAt date, automatically moved to archived

**Read state logic**:
- Opening notification center does NOT mark notifications as read
- Clicking notification call-to-action marks as read
- Navigating to target route independently marks related notifications as read
- Bulk "mark all as read" available for each category

### Deduplication and Grouping

#### Deduplication Rules
- Identical notifications within 12-hour window are prevented
- Comparison based on: `userId`, `type`, `groupKey` (if present)
- Updates existing notification instead of creating new one
- Resets `createdAt` timestamp and delivery queue

#### Grouping Rules
Notifications are grouped when:
- Same `groupKey` value (e.g., "assignment_due_G-01542")
- Occurring within 6-hour window
- Same `userId` and `type`

**Grouped notification display**:
- Shows latest notification content
- Displays count badge: "+3 more" or "4 similar notifications"
- Expanding group shows individual notification details
- Reading one does not mark others as read

**Example grouping scenarios**:
- Multiple students completing same assignment → grouped by teacher
- Multiple assignment reminders for same student → grouped
- Sequence of challenge invites → grouped by student

### Throttling and Batching

#### Toast Throttling
- Maximum 3 toast notifications per session by default (user configurable)
- After limit reached, subsequent notifications queue in center only
- Counter badge shows "+5 more notifications" when throttled
- Throttle resets on:
  - User opens and reviews notification center
  - User logs out and back in
  - 4-hour time window passes

#### Batching Rules
- Low-priority notifications batched every 30 minutes
- Normal-priority notifications batched if more than 5 pending
- Batch delivered as single notification: "You have 7 new updates"
- Clicking batch notification opens notification center filtered to new items

### Quiet Hours Handling

**Quiet hours behavior**:
- Configured per-user in notification preferences
- Applies to priority levels: `normal` and `low` only
- `high` priority queues and delivers immediately when quiet hours end
- `blocking` priority always delivers immediately

**Queue management**:
- Queued notifications stored with `status: 'pending'`
- Delivered in batch when quiet hours end
- Maximum 20 queued notifications; older items deduplicated
- Single notification shown: "You have 12 updates from your quiet hours"

### Cross-tab Synchronization

**Multi-window scenarios**:
- WebSocket connection per browser tab
- Notification read in one tab syncs to all tabs within 500ms
- Badge counts synchronized across tabs
- Opening notification center in one tab updates all tabs

**Synchronization events**:
- New notification arrives → all tabs update
- User reads notification → all tabs mark as read
- User archives notification → all tabs remove from center
- User changes preferences → all tabs apply new settings

### Offline Handling

**When user goes offline**:
- WebSocket disconnects, notification queue builds server-side
- On reconnect, client requests notifications since last sync
- Maximum 50 notifications delivered on reconnect
- Older notifications collapsed: "23 notifications from while you were away"

**Offline notification creation**:
- Notifications continue to be created server-side
- Queued for delivery when user reconnects
- Email/push channels continue to function (if enabled)

### Expiration and Cleanup

**Auto-expiration rules**:
- Student assignment notifications: expire 30 days after due date
- Teacher progress alerts: expire 14 days after creation
- Admin billing notifications: expire 90 days after resolution
- System announcements: expire at admin-specified date
- General notifications: expire 60 days after creation

**Cleanup process**:
- Expired notifications moved to `archived` status
- Archived notifications purged after 1 year for compliance
- Critical notifications (security, billing) retained per retention policy
- Bulk cleanup runs daily at 3 AM UTC

## UX requirements (if applicable)

### Notification Center Component

#### Layout and Structure
- **Location**: Accessible from header navigation bell icon
- **Entry point**: Bell icon with unread count badge
- **Panel design**: 
  - Width: 400px on desktop, full-width on mobile
  - Max height: 600px with internal scroll
  - Position: Dropdown panel anchored to bell icon (desktop), slide-up panel (mobile)
  - Z-index: Above main content but below blocking modals

#### Visual Hierarchy
```
┌─────────────────────────────────────┐
│ Notifications                    ⚙  │ ← Header with settings
├─────────────────────────────────────┤
│ [All] [Assignments] [Messages] ...  │ ← Category filters
├─────────────────────────────────────┤
│ ┌─────────────────────────────────┐ │
│ │ 🔵 [Icon] Assignment Due Soon   │ │ ← Unread notification
│ │   Complete "Treble Clef Notes"  │ │
│ │   2 hours ago                   │ │
│ └─────────────────────────────────┘ │
│ ┌─────────────────────────────────┐ │
│ │    [Icon] Badge Earned          │ │ ← Read notification
│ │   You earned "Note Master"!     │ │
│ │   Yesterday                     │ │
│ └─────────────────────────────────┘ │
│ ┌─────────────────────────────────┐ │
│ │    [Icon] +3 Assignment Updates │ │ ← Grouped notifications
│ │   Multiple updates received     │ │
│ │   Today                         │ │
│ └─────────────────────────────────┘ │
├─────────────────────────────────────┤
│      View All  |  Mark All Read     │ ← Footer actions
└─────────────────────────────────────┘
```

#### Notification Card States

**Unread state**:
- Blue indicator dot on left edge
- Bold title text
- Slightly elevated background (subtle shadow)
- Clear visual distinction from read notifications

**Read state**:
- No indicator dot
- Normal weight title text
- Flat background
- Slightly reduced opacity (0.85)

**Hover state** (desktop):
- Subtle background color change
- Cursor changes to pointer
- Action buttons fade in (archive, dismiss)

**Loading state**:
- Skeleton shimmer animation
- Preserves vertical space
- Shows 3-5 placeholder cards

**Empty state**:
- Friendly illustration (Terry Treble character)
- Message: "You're all caught up!"
- Secondary actions: "View archived" or "Adjust settings"

#### Category Filters
- **All**: Shows all notification categories (default)
- **Assignments**: Assignment-related notifications only
- **Challenges**: Gamification and challenge notifications
- **Messages**: Communication and reply notifications
- **System**: System announcements and updates
- **Billing**: (Admin roles only) Payment and subscription notifications

**Filter behavior**:
- Pills or tabs UI pattern
- Active filter highlighted
- Badge count shown per category
- Persists across sessions (saved to preferences)

### Toast Notification Component

#### Toast Appearance
- **Position**: Top-right corner (desktop), top-center (mobile)
- **Width**: 360px (desktop), 90% screen width (mobile)
- **Animation**: Slide in from right (desktop), slide down from top (mobile)
- **Duration**: 5 seconds default, configurable per notification
- **Stacking**: Maximum 3 visible, new ones push older ones down

#### Toast Structure
```
┌───────────────────────────────────┐
│ [Icon] Assignment Due Soon    ✕   │ ← Title with dismiss button
│ Complete "Treble Clef Notes" by   │ ← Body text
│ 3:00 PM today                     │
│ [View Assignment]                 │ ← Call-to-action button
└───────────────────────────────────┘
```

#### Toast Priority Styling
- **Blocking**: Red accent, modal overlay, requires action
- **High**: Orange accent, persistent until action
- **Normal**: Blue accent, auto-dismisses after 5 seconds
- **Low**: Gray accent (but low priority never shows as toast)

#### Toast Interactions
- **Click anywhere**: Navigates to target route and marks as read
- **Dismiss (X)**: Removes toast but keeps in notification center
- **Swipe away** (mobile): Same as dismiss
- **Auto-dismiss**: Fades out after duration, remains in center
- **Pause on hover** (desktop): Prevents auto-dismiss while hovering

### Notification Badge

#### Badge Display
- **Location**: Top-right of bell icon
- **Shape**: Circular badge
- **Color**: Brand primary color (attention-grabbing)
- **Content**: 
  - Shows count up to 99
  - Shows "99+" for counts over 99
  - Hides when count is 0

#### Badge Animation
- **New notification**: Gentle pulse animation (1 cycle)
- **Count update**: Scale animation (0.8x to 1.2x to 1x)
- **First notification**: More prominent animation to draw attention

### Accessibility Requirements

Reference: [Accessibility Standards](../01-ux-design-system/a11y-standards.md)

#### Screen Reader Support
- **ARIA live region**: Polite announcements for new notifications
- **Semantic HTML**: Proper heading hierarchy within notification center
- **Icon labels**: All icons have descriptive `aria-label` attributes
- **State announcements**: "Unread notification" vs "Read notification"
- **Count announcements**: "You have 5 unread notifications"

#### Keyboard Navigation
- **Bell icon**: Focusable, opens center on Enter/Space
- **Within center**: Tab through notifications, category filters, actions
- **Notification cards**: Enter/Space to open, Delete to archive
- **Escape key**: Closes notification center
- **Focus trap**: Focus stays within center when open

#### Visual Accessibility
- **Color contrast**: WCAG AA compliance (4.5:1 minimum)
- **Color independence**: Icons and text convey status, not just color
- **Focus indicators**: Clear 2px outline on focused elements
- **Text scaling**: Supports up to 200% text zoom without breaking layout
- **High contrast mode**: Compatible with Windows high contrast themes

#### Motion and Animation
- **Reduced motion**: Respects `prefers-reduced-motion` media query
- **No auto-play audio**: Notifications never include sound
- **Dismissible animations**: All animations can be dismissed immediately
- **No flashing content**: No elements flash more than 3 times per second

### Responsive Behavior

#### Desktop (≥1024px)
- Notification center as dropdown panel (400px wide)
- Toast notifications in top-right corner
- Hover states and tooltips enabled
- Side-by-side layout for actions

#### Tablet (768px - 1023px)
- Notification center as side panel (360px wide)
- Toast notifications in top-right corner (narrower)
- Touch-optimized tap targets (44px minimum)
- Stacked layout for actions

#### Mobile (<768px)
- Notification center as full-screen slide-up panel
- Toast notifications in top-center (full width minus margin)
- Large touch targets throughout
- Swipe gestures for dismissing toasts and notifications
- Bottom-aligned action buttons for thumb reach

### Performance Requirements

#### Load Performance
- Initial center open: < 200ms to visible content
- Notification list rendering: < 50ms for 50 items
- Badge count update: < 100ms after new notification
- WebSocket message processing: < 50ms

#### Animation Performance
- Toast slide-in: 60fps smooth animation
- Notification center open/close: 60fps
- Badge pulse: No janky frames
- List scroll: Virtual scrolling for 100+ notifications

#### Network Efficiency
- WebSocket connection: Automatic reconnection on disconnect
- Payload compression: Gzip for notification data
- Batching: Combine multiple updates into single message
- Offline queue: Store and forward when reconnected

## Telemetry

Reference: [Event Model](../15-analytics-and-reporting/event-model.md) for complete event tracking specifications.

### Notification Lifecycle Events

#### Notification Created
```javascript
{
  event_type: "notification_created",
  properties: {
    notification_id: "notif_1a2b3c4d",
    notification_type: "assignment_due_soon",
    priority: "high",
    category: "assignment",
    user_id: "usr_12345",
    user_role: "student",
    delivery_channels: ["in_app", "email"],
    source_event_id: "evt_7890abcd"
  }
}
// Fires: Server-side when notification is created
```

#### Notification Delivered
```javascript
{
  event_type: "notification_delivered",
  properties: {
    notification_id: "notif_1a2b3c4d",
    notification_type: "assignment_due_soon",
    delivery_channel: "in_app",
    delivery_method: "websocket",
    latency_ms: 127,
    queued: false
  }
}
// Fires: Client-side when notification arrives via WebSocket
```

### User Interaction Events

#### Notification Center Opened
```javascript
{
  event_type: "notification_center_opened",
  properties: {
    unread_count: 5,
    total_count: 23,
    trigger: "bell_click" | "keyboard_shortcut",
    time_since_last_open_seconds: 3600
  }
}
// Fires: Client-side when user opens notification center
```

#### Notification Viewed
```javascript
{
  event_type: "notification_viewed",
  properties: {
    notification_id: "notif_1a2b3c4d",
    notification_type: "assignment_due_soon",
    priority: "high",
    time_to_view_seconds: 45,        // Time from delivery to view
    viewed_in_center: true,          // vs toast click
    category_filter: "assignments"   // Active filter when viewed
  }
}
// Fires: Client-side when notification becomes visible in viewport
```

#### Notification Clicked
```javascript
{
  event_type: "notification_clicked",
  properties: {
    notification_id: "notif_1a2b3c4d",
    notification_type: "assignment_due_soon",
    click_target: "call_to_action" | "card" | "toast",
    target_route: "/student/assignment-play",
    time_since_delivered_seconds: 125,
    was_read: false
  }
}
// Fires: Client-side when user clicks notification
```

#### Notification Dismissed
```javascript
{
  event_type: "notification_dismissed",
  properties: {
    notification_id: "notif_1a2b3c4d",
    notification_type: "assignment_due_soon",
    dismiss_method: "x_button" | "swipe" | "archive_action",
    time_since_delivered_seconds: 3,
    was_read: false
  }
}
// Fires: Client-side when user dismisses notification
```

#### Notification Archived
```javascript
{
  event_type: "notification_archived",
  properties: {
    notification_id: "notif_1a2b3c4d",
    notification_type: "assignment_due_soon",
    archive_method: "manual" | "auto_expire" | "bulk_action",
    was_read: true,
    days_since_created: 2
  }
}
// Fires: Client or server-side when notification is archived
```

### Preference Events

#### Notification Preference Changed
```javascript
{
  event_type: "notification_preference_changed",
  properties: {
    preference_type: "quiet_hours" | "category_toggle" | "throttling",
    category: "assignments",           // If category-specific
    old_value: "enabled",
    new_value: "disabled",
    changed_from: "settings_panel" | "notification_center"
  }
}
// Fires: Client-side when user modifies preferences
```

### System Performance Events

#### WebSocket Connection
```javascript
{
  event_type: "notification_websocket_status",
  properties: {
    status: "connected" | "disconnected" | "reconnecting",
    reconnect_attempt: 2,            // If reconnecting
    connection_duration_ms: 45000,   // If disconnecting
    error_code: "network_error"      // If error
  }
}
// Fires: Client-side on WebSocket state changes
```

#### Notification Sync
```javascript
{
  event_type: "notification_sync_completed",
  properties: {
    sync_reason: "reconnect" | "page_load" | "manual_refresh",
    notifications_synced: 12,
    sync_duration_ms: 234,
    had_conflicts: false
  }
}
// Fires: Client-side after notification sync completes
```

### Aggregated Analytics Metrics

These events feed into dashboards and reports:

- **Notification engagement rate**: (clicked + viewed) / delivered
- **Time to interaction**: Average time from delivery to first interaction
- **Dismissal rate by type**: Track which notification types are frequently dismissed
- **Quiet hours adoption**: Percentage of users with quiet hours enabled
- **Cross-tab sync performance**: Average sync latency across tabs
- **Delivery success rate**: Percentage of notifications successfully delivered
- **Preference change frequency**: Track how often users adjust settings

## Permissions

Reference: [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md)

### Notification Visibility by Role

#### Student
- **Receive notifications**: Scoped to their own learning activities
- **View notifications**: Personal notification center only
- **Manage preferences**: Own notification settings (categories, quiet hours)
- **Archive notifications**: Own notifications only
- **Cannot see**: Other students' notifications, teacher/admin notifications

#### Teacher
- **Receive notifications**: Student progress, assignments, messages from their students
- **View notifications**: Personal center with student-related alerts
- **Manage preferences**: Own settings plus category priorities
- **Archive notifications**: Own notifications
- **Cannot see**: Other teachers' notifications, billing/admin notifications, student-to-student communications

#### Teacher-Admin
- **Receive notifications**: All teacher capabilities plus teacher performance alerts
- **View notifications**: Aggregated view of teachers in their scope
- **Manage preferences**: Own settings plus organizational defaults
- **Archive notifications**: Own notifications
- **Cannot see**: Billing notifications, student personal notifications outside their scope

#### Subscriber-Admin
- **Receive notifications**: Billing, subscription, system-wide organizational alerts
- **View notifications**: Full organizational view (excluding individual student learning details)
- **Manage preferences**: Own settings plus organization-wide policies
- **Archive notifications**: Own notifications and organization-level announcements
- **Broadcast capability**: Can trigger organization-wide system announcements
- **Cannot see**: Individual student learning progress (unless also Teacher role)

#### System Admin
- **Receive notifications**: Platform-wide system alerts, security notifications
- **View notifications**: All notifications across all organizations (audit purposes)
- **Manage preferences**: Global notification system settings
- **Broadcast capability**: Platform-wide system announcements
- **Override capability**: Can modify notification delivery rules for troubleshooting
- **Special access**: View notification delivery logs and analytics

### Notification Creation Permissions

| Action | Student | Teacher | Teacher-Admin | Subscriber-Admin | System Admin |
|--------|---------|---------|---------------|------------------|--------------|
| Trigger automated notifications | ✓ (via actions) | ✓ (via actions) | ✓ (via actions) | ✓ (via actions) | ✓ |
| Send direct messages (triggers notification) | ✓ (to teachers) | ✓ (to students/admins) | ✓ (in scope) | ✓ (org-wide) | ✓ (platform) |
| Create system announcements | ✗ | ✗ | ✗ | ✓ (org-only) | ✓ (platform) |
| Override notification delivery | ✗ | ✗ | ✗ | ✗ | ✓ |
| Modify notification preferences | ✓ (own) | ✓ (own) | ✓ (own) | ✓ (own + org defaults) | ✓ (all) |

### Privacy and Data Access Rules

- **Notification content**: Scoped to user's permission level, no cross-boundary leakage
- **Notification history**: Users can only view their own historical notifications
- **Analytics access**: Aggregated notification metrics follow same role permissions as user data
- **Audit trail**: System admins can view notification delivery logs for compliance
- **Data retention**: Follows organizational data retention policies (see [Privacy Policy](../23-legal-and-compliance/privacy-policy.md))
- **COPPA compliance**: Student notification content excludes PII in payload
- **FERPA compliance**: Educational records in notifications protected per role boundaries

## Supporting Documents Referenced

This in-app notifications specification draws from the following source documents:

- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform messaging and notification content
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — User interface text for notifications and alerts
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role-specific notification requirements

## Dependencies

### Core System Dependencies

- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Notification triggers based on system events
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: User role definitions and permission scoping
- **[Session and Auth](../02-roles-and-permissions/session-and-auth-states.md)**: User authentication for notification delivery
- **[Assignment Engine](../10-assignments-engine/assignment-generation.md)**: Assignment-related notification triggers
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Learning progress event context

### Role-Specific Notification Specs

- **[Student Notifications](../03-student-experience/student-notifications.md)**: Student-specific notification triggers and content
- **[Teacher Notifications](../04-teacher-experience/teacher-notifications.md)**: Teacher-specific notification triggers and content
- **[Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md)**: Admin-specific notification triggers and content

### Communication Infrastructure

- **[Email Templates](./email-templates.md)**: Email channel for notification delivery
- **[Push and Desktop](./push-and-desktop.md)**: Push notification delivery infrastructure
- **[System Announcements](./system-announcements.md)**: Broadcast announcement management
- **[Communication Framework](./communication-framework.md)**: Overall communication strategy and policies

### Technical Infrastructure

- **WebSocket Server**: Real-time bidirectional communication for live notifications
- **Message Queue**: Reliable notification creation and delivery pipeline (e.g., RabbitMQ, Redis Streams)
- **Database**: PostgreSQL for notification persistence and state management
- **Cache Layer**: Redis for unread counts, online user tracking, and WebSocket session management
- **CDN**: Asset delivery for notification images and icons

### UI Component Dependencies

- **[Design System](../01-ux-design-system/components-overview.md)**: Shared UI components for notification center
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)**: WCAG compliance requirements
- **[Iconography](../01-ux-design-system/iconography.md)**: Icon library for notification types
- **[Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md)**: Mobile/desktop layout adaptation

### Integration Points

- **[Background Jobs](../18-architecture/background-jobs.md)**: Scheduled notification cleanup and digest generation
- **[Webhooks](../18-architecture/webhooks.md)**: External system notification triggers
- **[API Design](../18-architecture/api-design-principles.md)**: REST endpoints for notification CRUD operations
- **[Caching Strategy](../18-architecture/caching-strategy.md)**: Notification count and state caching

## Open questions

### Technical Implementation

- **WebSocket vs Server-Sent Events**: Should we use WebSocket (bidirectional) or SSE (server-to-client) for real-time delivery? WebSocket provides more flexibility but SSE is simpler for one-way push.
- **Storage solution**: PostgreSQL with JSONB for notification payload, or separate notification service with document store? Consider query patterns and scale.
- **Read receipts**: Should we track when user sees notification in viewport (impression tracking) or only when they click? Viewport tracking more accurate but more resource-intensive.
- **Mobile app**: Will we build native mobile apps? If yes, need native push notification service integration (APNs, FCM).

### User Experience

- **Default quiet hours**: Should we have smart defaults for quiet hours based on user age/role (e.g., students: 8 PM - 8 AM, teachers: 6 PM - 7 AM)?
- **Notification sounds**: Should toast notifications have optional sound alerts? If yes, need sound preferences per category and WCAG audio guidance.
- **Digest mode**: Should we offer a digest mode that batches all notifications for delivery at a specific time (e.g., daily 8 AM summary)?
- **Notification preview**: Should email/push notifications include full content or just truncated preview? Privacy vs. convenience trade-off.

### Business Rules

- **Rate limiting**: What are the limits for organization-wide announcements to prevent spam? (e.g., max 5 announcements per week?)
- **Notification priority escalation**: Should overdue assignment notifications escalate from normal → high → critical over time?
- **Parental notifications**: Should parent/guardian accounts receive copies of student notifications? How to handle minor consent?
- **Teacher notification overload**: How to prevent teachers with 100+ students from being overwhelmed by notifications? Need smart batching/aggregation.

### Performance and Scale

- **Notification volume**: What is expected peak notification volume? (estimate: 10K notifications/minute at scale)
- **WebSocket scaling**: How many concurrent WebSocket connections can we support per server? Need load balancing strategy.
- **Database partitioning**: Should we partition notification table by date or organization for query performance?
- **Retention policy**: How long should we keep archived notifications? Balance storage costs vs. audit needs.

### Privacy and Compliance

- **COPPA compliance**: How to handle parental consent for notification delivery to students under 13?
- **GDPR right to erasure**: When user deletes account, do we delete all notifications or anonymize sender/recipient?
- **Cross-border notifications**: Do we need region-specific notification routing for data residency compliance?
- **Notification export**: Should users be able to export their notification history as part of data portability rights?

### Future Features

- **Snooze functionality**: Should users be able to snooze notifications for later (e.g., remind me in 1 hour)?
- **Smart notifications**: Use ML to predict best delivery time based on user activity patterns?
- **Notification templates**: Should admins be able to create custom notification templates for their organization?
- **Multi-language**: How to handle notification content in multiple languages? Translate on delivery or store multiple versions?
