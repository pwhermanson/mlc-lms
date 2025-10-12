# Push and Desktop Notifications

## Purpose

This specification defines the push notification and desktop notification infrastructure for the MLC-LMS platform. Push notifications enable the platform to reach users on their mobile devices (iOS and Android) even when the app is not actively running, while desktop notifications deliver system-level alerts on desktop operating systems (Windows, macOS, Linux) when users have their browser or app open or minimized.

These notification channels extend the platform's communication reach beyond in-app notifications and email, enabling timely alerts for time-sensitive events (assignment due soon, streak at risk, teacher feedback) that drive re-engagement and ensure users don't miss critical updates. Push and desktop notifications are particularly effective for just-in-time learning reminders and urgent communications.

This system works in concert with [In-app Notifications](./in-app-notifications.md) and [Email Templates](./email-templates.md) to create a comprehensive multi-channel notification strategy. While in-app notifications reach active users and email provides detailed content, push/desktop notifications serve as the bridge—alerting users when they're away from the platform and encouraging them to return for important actions.

## Scope

**Included**
- Native mobile push notification infrastructure (iOS APNs, Android FCM/GCM)
- Web push notification system (browser-based push via Service Workers)
- Desktop operating system notifications (Windows Action Center, macOS Notification Center, Linux)
- Device registration and token management
- Push notification payload structure and formatting
- Opt-in permission flows and consent management
- Silent push notifications for background data sync
- Push notification badge management (app icon badge counts)
- Rich push notifications (images, actions, custom layouts)
- Notification grouping and threading (iOS notification stacks, Android bundling)
- Deep linking from push notifications to specific app screens
- Push notification delivery tracking and receipts
- Token refresh and expiration handling
- Multi-device management per user
- Platform-specific notification behaviors and limitations
- Push notification testing and preview tools

**Excluded**
- In-app notification center and toast notifications (see [In-app Notifications](./in-app-notifications.md))
- Email notification delivery (see [Email Templates](./email-templates.md))
- SMS notifications (future consideration)
- Notification content and triggers by role (see [Student Notifications](../03-student-experience/student-notifications.md), [Teacher Notifications](../04-teacher-experience/teacher-notifications.md), [Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md))
- System announcement content management (see [System Announcements](./system-announcements.md))
- User notification preference UI (defined in role experience docs)
- Event tracking schema (see [Event Model](../15-analytics-and-reporting/event-model.md))

## Data model (if applicable)

### Device Registration Entity

```typescript
interface DeviceRegistration {
  // Identification
  deviceId: string;                // UUID, e.g., "dev_1a2b3c4d5e6f"
  userId: string;                  // Owner of device
  organizationId: string;          // Nullable for personal accounts
  
  // Device Information
  deviceType: 'ios' | 'android' | 'web' | 'desktop';
  platform: string;                // "iOS", "Android", "Chrome", "Firefox", "Safari", "Windows", "macOS", "Linux"
  platformVersion: string;         // OS or browser version
  deviceModel?: string;            // "iPhone 14 Pro", "Pixel 7", etc. (mobile only)
  deviceName?: string;             // User-assigned name, e.g., "Sarah's iPhone"
  
  // Push Token
  pushToken: string;               // Platform-specific push token/endpoint
  pushTokenType: 'apns' | 'fcm' | 'web_push' | 'desktop';
  tokenProvider?: string;          // "Firebase", "OneSignal", etc. if using service
  vapidPublicKey?: string;         // For web push (VAPID)
  
  // Registration Status
  status: 'active' | 'inactive' | 'expired' | 'revoked';
  isValid: boolean;                // Token validated with push provider
  lastValidatedAt?: Date;
  
  // Capabilities
  supportsRichContent: boolean;    // Images, actions, etc.
  supportsNotificationActions: boolean;
  supportsSilentPush: boolean;
  maxPayloadSize: number;          // Bytes
  
  // Settings
  notificationsEnabled: boolean;   // Device-level opt-in
  badgeEnabled: boolean;           // Show badge count on app icon
  soundEnabled: boolean;           // Play sound for notifications
  alertsEnabled: boolean;          // Show banner/alert
  
  // Metadata
  appVersion?: string;             // App version if native app
  appBuildNumber?: string;
  sdkVersion?: string;             // SDK version for analytics
  timezone: string;                // IANA timezone
  locale: string;                  // "en-US", "es-MX", etc.
  
  // Audit
  registeredAt: Date;
  lastUsedAt: Date;                // Last successful push delivery
  expiresAt?: Date;                // Token expiration (platform-dependent)
  revokedAt?: Date;
  deactivatedAt?: Date;
  
  // Tracking
  totalNotificationsSent: number;
  totalNotificationsDelivered: number;
  totalNotificationsOpened: number;
  lastDeliveryStatus?: DeliveryStatus;
}
```

### PushNotification Entity

```typescript
interface PushNotification {
  // Identification
  pushId: string;                  // UUID for this push send
  notificationId: string;          // Links to in-app notification
  userId: string;
  deviceIds: string[];             // Target devices (may send to multiple)
  
  // Content
  title: string;                   // Max 65 chars (platform limitation)
  body: string;                    // Max 240 chars (platform limitation)
  subtitle?: string;               // iOS only, max 65 chars
  badge?: number;                  // Badge count for app icon
  sound?: string;                  // Sound file name or "default"
  icon?: string;                   // URL to notification icon
  image?: string;                  // URL to rich notification image
  
  // Behavior
  priority: 'high' | 'normal' | 'low';
  timeToLive: number;              // Seconds to keep if device offline
  collapseKey?: string;            // Group related notifications
  threadId?: string;               // iOS notification thread
  tag?: string;                    // Android notification tag
  channelId?: string;              // Android notification channel
  
  // Actions
  callToAction?: {
    defaultAction: {               // Tapping notification body
      type: 'deep_link' | 'url';
      target: string;
      params?: Record<string, any>;
    };
    actions?: NotificationAction[]; // Action buttons (iOS 10+, Android)
  };
  
  // Data Payload
  data?: Record<string, string>;   // Custom key-value pairs (silent push)
  isSilent: boolean;               // Silent/background push
  contentAvailable?: boolean;      // iOS background fetch trigger
  mutableContent?: boolean;        // iOS notification service extension
  
  // Platform-Specific
  platformConfig?: {
    ios?: IOSConfig;
    android?: AndroidConfig;
    web?: WebPushConfig;
  };
  
  // Delivery
  status: 'pending' | 'queued' | 'sent' | 'delivered' | 'failed' | 'expired';
  scheduledFor?: Date;             // Scheduled delivery time
  sentAt?: Date;
  deliveredAt?: Date;
  openedAt?: Date;
  failedAt?: Date;
  failureReason?: string;
  
  // Tracking
  deliveryReceipts: DeviceDeliveryReceipt[];
  
  // Audit
  createdAt: Date;
  expiresAt: Date;
}
```

### NotificationAction

```typescript
interface NotificationAction {
  actionId: string;                // Unique action identifier
  title: string;                   // Button text, max 25 chars
  icon?: string;                   // Icon identifier (Android)
  requiresAuth?: boolean;          // Requires device unlock (iOS)
  foreground?: boolean;            // Brings app to foreground
  destructive?: boolean;           // Red text (iOS)
  
  action: {
    type: 'deep_link' | 'url' | 'dismiss' | 'callback';
    target?: string;
    params?: Record<string, any>;
  };
}
```

### Platform-Specific Configs

```typescript
interface IOSConfig {
  apnsId?: string;                 // APNs message ID
  apnsPriority?: 'high' | 'normal';
  apnsExpiration?: Date;
  apnsCollapseId?: string;
  apnsCategory?: string;           // Notification category for actions
  apnsThreadId?: string;
  interruptionLevel?: 'passive' | 'active' | 'time-sensitive' | 'critical'; // iOS 15+
  relevanceScore?: number;         // 0.0 to 1.0, for notification summary (iOS 15+)
  targetContentId?: string;        // For Live Activities (iOS 16+)
}

interface AndroidConfig {
  channelId: string;               // Required for Android 8.0+
  color?: string;                  // Notification color (hex)
  smallIcon?: string;              // Small icon resource name
  largeIcon?: string;              // Large icon URL
  vibrate?: number[];              // Vibration pattern in milliseconds
  ledColor?: string;               // LED color (hex)
  priority?: 'min' | 'low' | 'default' | 'high' | 'max';
  visibility?: 'private' | 'public' | 'secret';
  autoCancel?: boolean;            // Auto-dismiss on tap
  ongoing?: boolean;               // Cannot be dismissed
  group?: string;                  // Notification group key
  groupSummary?: boolean;          // Is this the summary notification
  ticker?: string;                 // Status bar text
  style?: 'bigtext' | 'bigpicture' | 'inbox' | 'messaging';
}

interface WebPushConfig {
  badge?: string;                  // URL to badge icon
  dir?: 'auto' | 'ltr' | 'rtl';
  lang?: string;
  requireInteraction?: boolean;    // Notification stays visible
  renotify?: boolean;              // Alert again if replacing existing
  silent?: boolean;
  vibrate?: number[];
  timestamp?: number;              // Milliseconds since epoch
}
```

### DeviceDeliveryReceipt

```typescript
interface DeviceDeliveryReceipt {
  deviceId: string;
  pushId: string;
  
  status: 'queued' | 'sent' | 'delivered' | 'opened' | 'failed' | 'expired';
  sentToProviderAt?: Date;         // Sent to APNs/FCM
  deliveredToDeviceAt?: Date;      // Confirmed delivery
  openedAt?: Date;                 // User tapped notification
  dismissedAt?: Date;
  
  failureReason?: string;
  providerErrorCode?: string;
  providerMessageId?: string;
  
  // Response times
  providerLatencyMs?: number;      // Time to send to APNs/FCM
  deliveryLatencyMs?: number;      // Total time to device confirmation
  
  // Actions taken
  actionTaken?: string;            // Which action button was tapped
  
  createdAt: Date;
}
```

### NotificationChannel Entity (Android)

```typescript
interface NotificationChannel {
  channelId: string;               // Unique identifier
  name: string;                    // User-visible name
  description: string;             // Channel description
  importance: 'min' | 'low' | 'default' | 'high' | 'max';
  
  // Sound and vibration
  sound?: string;                  // Sound file name
  vibrationPattern?: number[];
  enableLights?: boolean;
  lightColor?: string;
  
  // Behavior
  showBadge?: boolean;
  allowBubbles?: boolean;          // Android 11+
  
  // Metadata
  createdAt: Date;
  updatedAt: Date;
}
```

### PushSubscription Entity (Web Push)

```typescript
interface PushSubscription {
  subscriptionId: string;
  userId: string;
  deviceId: string;
  
  // Web Push Subscription
  endpoint: string;                // Push service endpoint URL
  keys: {
    p256dh: string;                // Public key for encryption
    auth: string;                  // Auth secret
  };
  
  // Browser Info
  browser: string;
  browserVersion: string;
  userAgent: string;
  
  // Status
  isActive: boolean;
  lastTestedAt?: Date;
  
  // Audit
  subscribedAt: Date;
  unsubscribedAt?: Date;
  expiresAt?: Date;
}
```

## Behavior and rules

### Device Registration Flow

#### Initial Registration

**Mobile App Registration (iOS/Android)**:
1. User launches app for first time
2. App requests notification permission from OS
3. If granted:
   - App obtains push token from APNs (iOS) or FCM (Android)
   - App sends token + device info to MLC backend
   - Backend stores `DeviceRegistration` record with `status: 'active'`
   - Backend validates token with push provider
   - Confirmation sent to app
4. If denied:
   - App stores local flag (permission denied)
   - User can later enable in Settings → Notifications
   - App re-requests token when permission granted

**Web Push Registration (Browser)**:
1. User visits MLC web app (must be HTTPS)
2. Service Worker registered in background
3. User triggers notification opt-in (e.g., clicks "Enable Notifications" button)
4. Browser displays permission prompt
5. If granted:
   - Service Worker subscribes to push service (browser-specific endpoint)
   - Subscription object contains endpoint + encryption keys
   - Subscription sent to MLC backend
   - Backend creates `PushSubscription` and `DeviceRegistration`
6. If denied:
   - Store local flag, show dismissible banner with option to enable later
   - Permission can be reset in browser settings

**Desktop Notification Registration (Native App)**:
1. Desktop app requests system notification permission
2. If granted:
   - Desktop app generates local notification identifier
   - Registers with platform-specific notification API
   - Sends registration to MLC backend
3. Notifications shown via OS notification center (Windows Action Center, macOS Notification Center, etc.)

#### Token Refresh and Validation

**Token Expiration Rules**:
- **iOS APNs tokens**: Don't expire but can be invalidated (app uninstall, OS reinstall)
- **Android FCM tokens**: May refresh periodically; app must handle `onTokenRefresh` callback
- **Web push subscriptions**: May expire based on browser/push service; typically no fixed expiration

**Validation Process**:
- Backend validates tokens weekly by sending silent test push
- If token invalid (410 Gone, 404 Not Found from provider):
  - Mark `DeviceRegistration.status = 'expired'`
  - Remove device from active push list
  - Device will re-register on next app launch
- If soft failure (503 Service Unavailable):
  - Retry up to 3 times
  - Don't mark as expired unless all retries fail

**Token Rotation**:
- When new token received for existing device:
  - Update `pushToken` in `DeviceRegistration`
  - Mark previous token as `status: 'revoked'` (keep for audit)
  - Log token rotation event for security monitoring

#### Multi-Device Management

**User with Multiple Devices**:
- Each device gets separate `DeviceRegistration` record
- Notifications can be sent to:
  - All active devices (default for high-priority)
  - Most recently used device only (for low-priority)
  - User-selected primary device (preference setting)
  - Specific device type only (e.g., mobile only, desktop only)

**Device Deactivation**:
- User can deactivate specific devices in Settings → Devices
- Deactivated devices marked `status: 'inactive'`
- Backend does not send push to inactive devices
- Device can be reactivated by user or automatic on next app launch

**Device Cleanup**:
- Devices not used for 90 days automatically deactivated
- Devices with expired tokens purged after 180 days
- User notified via email before purging

### Push Notification Delivery Pipeline

#### 1. Notification Creation and Routing

```
Event Trigger → Notification Builder → Channel Router → Push Composer → Provider Gateway → Device
```

**Trigger Sources**:
- System events (from [Event Model](../15-analytics-and-reporting/event-model.md))
- In-app notification creation (from [In-app Notifications](./in-app-notifications.md))
- Direct push-only events (silent data sync, badge updates)

**Channel Selection Logic**:
```javascript
// Pseudo-code for channel routing
function determineChannels(notification, userPreferences) {
  const channels = ['in_app']; // Always include in-app
  
  // Check user preferences per category
  if (userPreferences.push_enabled && notification.category in userPreferences.push_categories) {
    // Check if user online in-app
    if (isUserOnline(notification.userId)) {
      // High priority always sends push
      if (notification.priority === 'high' || notification.priority === 'blocking') {
        channels.push('push');
      }
      // Normal priority: skip push if user active in last 5 minutes
      else if (lastActivityAge(notification.userId) > 5 * 60) {
        channels.push('push');
      }
    } else {
      // User offline, always send push (unless quiet hours)
      if (!isInQuietHours(notification.userId, notification.priority)) {
        channels.push('push');
      }
    }
  }
  
  // Email follows separate rules (see Email Templates)
  if (userPreferences.email_enabled) {
    channels.push('email');
  }
  
  return channels;
}
```

**Priority-Based Delivery**:
- **Blocking** (`priority: 'blocking'`):
  - Sends to all active devices immediately
  - Uses high-priority platform flags (APNs priority 10, FCM high priority)
  - Bypasses quiet hours
  - Plays sound and vibration even if device on silent (platform permitting)
  - Displays as heads-up notification (Android) or banner (iOS)
  
- **High** (`priority: 'high'`):
  - Sends to most recently used device first
  - If not delivered within 5 minutes, sends to all devices
  - Uses high-priority platform flags
  - Respects quiet hours but queues for delivery immediately after
  - Plays default sound/vibration
  
- **Normal** (`priority: 'normal'`):
  - Sends to most recently used device only
  - Uses normal priority platform flags (may be throttled by OS)
  - Fully respects quiet hours
  - Plays sound/vibration per user settings
  - May be grouped with other notifications by OS
  
- **Low** (`priority: 'low'`):
  - No push notification sent (in-app and email only)
  - Exception: Badge-only silent push to update app icon count

#### 2. Push Composition

**Content Truncation**:
- Title: Max 65 characters (enforced, truncates with "...")
- Body: Max 240 characters (enforced, truncates with "...")
- Subtitle (iOS): Max 65 characters
- Longer content available by opening notification (deep link to in-app)

**Rich Content Preparation**:
- Images: Resize to max 1024x1024, compress to <500KB
- Icons: Resize to platform-specific dimensions (iOS 40x40@3x, Android 24x24dp)
- Upload to CDN with short TTL (7 days)
- Include fallback for devices that don't support rich content

**Localization**:
- Use device locale from `DeviceRegistration.locale`
- Fall back to user profile language preference
- Fall back to English if no translation available
- Store localized strings in notification content service

**Deep Link Construction**:
```javascript
// Example deep links
const deepLinks = {
  assignment: `mlc://assignment/${assignmentId}`,
  game: `mlc://game/${gameId}`,
  message: `mlc://messages/${threadId}`,
  dashboard: `mlc://dashboard`,
  // Web fallback URLs
  assignment_web: `https://mlc.example.com/student/assignment-play?id=${assignmentId}`,
  // ...
};
```

#### 3. Provider Gateway Delivery

**iOS APNs Delivery**:
```json
{
  "aps": {
    "alert": {
      "title": "Assignment Due Soon",
      "body": "Complete 'Treble Clef Notes' by 3:00 PM",
      "subtitle": "Your Daily Practice"
    },
    "badge": 3,
    "sound": "default",
    "thread-id": "assignments",
    "category": "ASSIGNMENT_REMINDER",
    "interruption-level": "time-sensitive",
    "relevance-score": 0.8
  },
  "data": {
    "notification_id": "notif_1a2b3c4d",
    "assignment_id": "asg_789",
    "deep_link": "mlc://assignment/asg_789"
  }
}
```

**Android FCM Delivery**:
```json
{
  "message": {
    "token": "fcm_token_here",
    "notification": {
      "title": "Assignment Due Soon",
      "body": "Complete 'Treble Clef Notes' by 3:00 PM",
      "image": "https://cdn.mlc.example.com/images/assignment_reminder.jpg"
    },
    "android": {
      "priority": "high",
      "notification": {
        "channel_id": "assignments",
        "color": "#0066CC",
        "icon": "ic_assignment",
        "sound": "default",
        "tag": "assignment_asg_789",
        "click_action": "ASSIGNMENT_OPEN"
      }
    },
    "data": {
      "notification_id": "notif_1a2b3c4d",
      "assignment_id": "asg_789",
      "deep_link": "mlc://assignment/asg_789"
    }
  }
}
```

**Web Push Delivery**:
```javascript
// Service Worker receives:
self.addEventListener('push', (event) => {
  const payload = event.data.json();
  
  const options = {
    body: payload.body,
    icon: payload.icon || '/images/logo-192.png',
    badge: '/images/badge-96.png',
    image: payload.image,
    data: payload.data,
    tag: payload.tag,
    requireInteraction: payload.priority === 'high',
    actions: [
      { action: 'open', title: 'View', icon: '/images/open.png' },
      { action: 'dismiss', title: 'Dismiss', icon: '/images/dismiss.png' }
    ]
  };
  
  event.waitUntil(
    self.registration.showNotification(payload.title, options)
  );
});
```

#### 4. Delivery Confirmation and Tracking

**Receipt Tracking**:
- **APNs**: No delivery confirmation (fire-and-forget)
  - Use APNs feedback service for token invalidation
  - Track open rate via deep link tracking
  
- **FCM**: Optional delivery receipts
  - Enable FCM Analytics for delivery data
  - Webhook notifications for delivery status
  
- **Web Push**: No native delivery confirmation
  - Track notification display in Service Worker
  - Track open/dismiss actions

**Retry Logic**:
- If provider returns 5xx error: Retry up to 3 times with exponential backoff (1s, 5s, 25s)
- If provider returns 4xx error: Log and fail (bad token, invalid payload)
- If provider returns 410 Gone: Mark device token expired, don't retry
- If no response after 30s: Timeout and retry

**Fallback Mechanism**:
- If push fails to all devices: Send email notification (if enabled)
- If push delivery delayed >15 minutes: Send email for high-priority notifications
- In-app notification always created regardless of push/email status

### Silent Push and Background Sync

**Silent Push Use Cases**:
- Badge count updates (increment app icon badge without visible notification)
- Content pre-fetch (download assignment data in background)
- Token refresh triggers
- Real-time data sync (messages, leaderboard updates)

**Silent Push Payload**:
```json
{
  "aps": {
    "content-available": 1,
    "badge": 5
  },
  "data": {
    "sync_type": "assignments",
    "sync_timestamp": "2025-01-15T14:30:00Z"
  }
}
```

**Platform Limitations**:
- **iOS**: Silent push wakes app in background for max 30 seconds
  - Subject to iOS background throttling (may be delayed)
  - Not guaranteed to be delivered
- **Android**: Background data sync via FCM data messages
  - More reliable than iOS
  - Can trigger WorkManager for longer tasks
- **Web**: Background Sync API (separate from push notifications)

**Badge Management**:
- Backend maintains badge count per user
- Increment on new notification, decrement on read
- Periodic badge sync (daily) to correct drift
- Push badge-only update when count changes (silent push)

### Notification Grouping and Threading

**iOS Notification Stacks** (iOS 12+):
```json
{
  "aps": {
    "alert": { "title": "3 new assignments", "body": "Tap to view" },
    "thread-id": "assignments",
    "summary-arg": "assignments",
    "summary-arg-count": 3
  }
}
```

**Android Notification Bundles**:
```json
{
  "android": {
    "notification": {
      "tag": "assignment_group",
      "group": "assignments",
      "group_summary": false
    }
  }
}
```

**Grouping Rules**:
- Group by `category` (assignments, challenges, messages)
- Maximum 5 notifications per group before auto-stacking
- Summary notification shows count: "You have 5 new assignments"
- Expanding stack shows individual notifications
- Tapping summary opens category-filtered notification center

### Quiet Hours and Delivery Scheduling

**Quiet Hours Application**:
- Push notifications respect user quiet hours (from [In-app Notifications](./in-app-notifications.md))
- Quiet hours applied differently by priority:
  - **Blocking**: Always delivered immediately (security, critical)
  - **High**: Queued, delivered immediately when quiet hours end
  - **Normal/Low**: Queued, delivered in batch when quiet hours end

**Scheduled Delivery**:
- Notifications can be scheduled for future delivery
- Use case: Reminder notifications scheduled based on user timezone
- Example: "Assignment due tomorrow" sent at 8 AM user local time
- Scheduled notifications stored with `scheduledFor` timestamp
- Background job checks for due scheduled notifications every minute

**Delivery Batching**:
- Normal-priority notifications during quiet hours batched together
- Delivered as single notification: "You have 7 updates"
- Tapping opens notification center with new items
- Reduces notification fatigue while ensuring nothing is missed

### Permission Management and Opt-In

**Initial Opt-In Flow**:
1. **Native Apps**: Permission prompt on first launch or when feature requires it
2. **Web**: User-triggered prompt from clear UI affordance (button/banner)
3. **Best Practice**: Explain value before showing system permission prompt

**Permission States**:
- **Not requested**: User hasn't been asked yet
- **Granted**: User accepted permission
- **Denied**: User rejected permission
- **Provisional** (iOS 12+): App can send silent notifications, user sees in Notification Center but not as banner/sound

**Re-requesting Permission**:
- If denied, cannot re-prompt via API (OS limitation)
- Show in-app UI explaining how to enable in device settings
- Provide deep link to app settings: `Settings → Notifications → MLC`
- Track permission status and show contextual prompts (e.g., before assignment due)

**Granular Opt-In by Category**:
- Backend manages category preferences (assignments, challenges, messages, etc.)
- Device permission is all-or-nothing (OS level)
- If device permission granted but category disabled in app:
  - Don't send push for that category
  - Still send in-app and email notifications (if enabled)

**Opt-Out Flow**:
- User can disable push in app Settings → Notifications
- Backend marks all user devices as `notificationsEnabled: false`
- User can re-enable without needing to re-register devices
- Permanent opt-out: User can revoke device permission at OS level

### Rich Push Notifications

**Rich Notification Support**:
- **iOS 10+**: Notification Service Extension for rich content
  - Images, GIFs, audio, video
  - Custom UI via Notification Content Extension
  - Max 4 action buttons
  
- **Android 7.0+**: BigPictureStyle, BigTextStyle, MessagingStyle
  - Inline reply
  - Custom actions
  - Media playback controls

**Image Attachments**:
```javascript
// iOS
{
  "aps": {
    "alert": { "title": "Badge Earned!", "body": "You earned Note Master" },
    "mutable-content": 1
  },
  "media-url": "https://cdn.mlc.example.com/badges/note_master.png"
}
```

**Notification Actions**:
```javascript
// Example: Assignment reminder with actions
const actions = [
  {
    actionId: 'start_assignment',
    title: 'Start Now',
    foreground: true,
    action: { type: 'deep_link', target: 'mlc://assignment/asg_789' }
  },
  {
    actionId: 'snooze_1h',
    title: 'Remind in 1 hour',
    foreground: false,
    action: { type: 'callback', target: '/api/notifications/snooze' }
  },
  {
    actionId: 'dismiss',
    title: 'Dismiss',
    destructive: true,
    action: { type: 'dismiss' }
  }
];
```

**Platform-Specific Rich Features**:
- **iOS**: Live Activities (iOS 16.1+) for real-time updates (e.g., challenge progress)
- **Android**: Bubbles (Android 11+) for floating chat heads
- **Web**: Notification actions with service worker handlers

### Error Handling and Resilience

**Common Push Errors**:
| Error | Code | Cause | Resolution |
|-------|------|-------|------------|
| Invalid token | 400/404 | Token expired or revoked | Mark device expired, remove from list |
| Bad certificate | 403 | APNs cert invalid | Alert engineering, rotate cert |
| Payload too large | 413 | Payload >4KB (APNs) or >1MB (FCM) | Truncate content, remove images |
| Rate limit | 429 | Too many requests | Implement exponential backoff, queue |
| Service unavailable | 503 | Provider outage | Retry with backoff, monitor status page |
| Unregistered | 410 | Device uninstalled app | Mark device expired |

**Monitoring and Alerting**:
- Track delivery success rate per platform (target: >98%)
- Alert if success rate drops below 95% for 5 minutes
- Monitor provider latency (target: <500ms P95)
- Track token expiration rate (should be <1% per week)
- Daily report of failed pushes by error code

**Disaster Recovery**:
- If APNs/FCM completely unavailable:
  - Queue notifications for retry (max 24 hours)
  - Send email notifications immediately as fallback
  - Display in-app notifications for active users
  - Show system banner: "Push notifications temporarily unavailable"
- If notification database unavailable:
  - Write to backup queue (Redis/RabbitMQ)
  - Process queue when database restored
  - No notifications lost (at-least-once delivery)

## UX requirements (if applicable)

### Permission Request UI

#### Native App Permission Prompt (Pre-Permission Primer)

**Context**: Before showing OS permission dialog, show in-app explanation

```
┌─────────────────────────────────────┐
│  [Bell Icon]                        │
│                                     │
│  Stay On Track with Reminders       │
│                                     │
│  Get notified about:                │
│  • Assignments due soon             │
│  • Teacher feedback                 │
│  • Challenges from friends          │
│  • Practice streak milestones       │
│                                     │
│  You can change this anytime in     │
│  Settings.                          │
│                                     │
│  [Enable Notifications] [Not Now]   │
└─────────────────────────────────────┘
```

**Flow**:
1. Show primer screen when user completes first assignment or after 3 days active use
2. If user taps "Enable Notifications": Show OS permission dialog
3. If user taps "Not Now": Dismiss, don't ask again for 7 days
4. If user grants: Immediate confirmation toast "You're all set!"
5. If user denies: Show subtle banner with link to enable in Settings

#### Web Push Permission Prompt

**Approach**: Never use auto-prompt on page load (poor UX, low conversion)

**Contextual Prompt** (Example: After first game completed):
```
┌─────────────────────────────────────┐
│  🎉 Great job on your first game!   │
│                                     │
│  Want to stay motivated?            │
│  Get reminders when it's time to    │
│  practice your next assignment.     │
│                                     │
│  [Sure, notify me] [No thanks]     │
└─────────────────────────────────────┘
```

**Persistent Opt-In Affordance** (Settings page):
```
Notifications
┌─────────────────────────────────────┐
│ 🔔 Browser Notifications  [Disabled]│
│                                     │
│ Get alerts even when MLC isn't      │
│ open. Your browser will ask for     │
│ permission.                         │
│                                     │
│ [Enable Browser Notifications]     │
└─────────────────────────────────────┘
```

#### Permission Denied Recovery

**In-App Banner** (if permission previously denied):
```
┌─────────────────────────────────────┐
│ ⚠️ Notifications are turned off     │
│                                     │
│ You might miss important reminders. │
│ [Show me how to enable] [Dismiss]   │
└─────────────────────────────────────┘
```

**Settings Instructions** (after tapping "Show me how to enable"):
```
How to Enable Notifications

iOS:
1. Open Settings app
2. Scroll to MLC
3. Tap Notifications
4. Turn on Allow Notifications

Android:
1. Open Settings
2. Tap Apps & notifications
3. Find MLC
4. Tap Notifications
5. Toggle on

[Open Settings] [Done]
```

### Notification Management Interface

Reference: User Settings screens in role-specific docs

#### Device Management UI

**Settings → Notifications → Devices**
```
Your Devices
┌─────────────────────────────────────┐
│ 📱 Sarah's iPhone           [Active]│
│    iPhone 14 Pro • iOS 17.2         │
│    Last used: Today at 2:30 PM      │
│    [Remove Device]                  │
├─────────────────────────────────────┤
│ 💻 MacBook Pro           [Inactive] │
│    Chrome Browser • macOS 14        │
│    Last used: 3 days ago            │
│    [Reactivate]                     │
├─────────────────────────────────────┤
│ 🌐 Chrome on Windows      [Active]  │
│    Web Push • Windows 11            │
│    Last used: Yesterday at 7:15 PM  │
│    [Remove Device]                  │
└─────────────────────────────────────┘

[Add Device]
```

#### Push Notification Preferences

**Settings → Notifications → Push Notifications**
```
Push Notifications                [🔔 On]

Receive push notifications even when 
MLC isn't open.

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Categories

☑️ Assignments & Practice
   New assignments, due dates, reviews

☑️ Challenges & Achievements  
   Challenge invites, badges earned

☑️ Messages
   Teacher feedback, replies

☑️ System Announcements
   Important updates

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Delivery

Quiet Hours             [8:00 PM - 7:00 AM]
Don't send notifications during quiet hours
(urgent notifications still delivered)

Only notify on:
○ All my devices
● My most recently used device
○ Mobile devices only

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

[Test Notification]
```

### Push Notification Display Examples

#### Mobile Push Notification (iOS)

**Assignment Reminder** (High Priority):
```
┌─────────────────────────────────────┐
│ MusicLearningCommunity  now     [×] │
├─────────────────────────────────────┤
│ 🎵 Assignment Due Soon              │
│ Complete "Treble Clef Notes" by    │
│ 3:00 PM today                       │
│ [VIEW] [SNOOZE 1H]                  │
└─────────────────────────────────────┘
```

**Badge Earned** (Normal Priority, Rich):
```
┌─────────────────────────────────────┐
│ MusicLearningCommunity  2h ago  [×] │
├─────────────────────────────────────┤
│ [Badge Image: Note Master]          │
│                                     │
│ 🏆 Badge Earned!                    │
│ You earned "Note Master" for        │
│ mastering 50 notes!                 │
│ [VIEW]                              │
└─────────────────────────────────────┘
```

**Grouped Notifications** (iOS Stack):
```
┌─────────────────────────────────────┐
│ MusicLearningCommunity          [×] │
├─────────────────────────────────────┤
│ 📚 3 updates                        │
│ You have 3 new assignments          │
│                                     │
│ [Tap to expand...]                  │
└─────────────────────────────────────┘
```

#### Desktop Notification (Windows Action Center)

```
┌──────────────────────────────────────────┐
│ MusicLearningCommunity                   │
├──────────────────────────────────────────┤
│ Assignment Due Soon                      │
│ Complete "Treble Clef Notes" by 3:00 PM │
│ today                                    │
└──────────────────────────────────────────┘
```

#### Web Push Notification (Browser)

```
┌──────────────────────────────────────────┐
│ MusicLearningCommunity              [×]  │
│ [MLC Logo Icon]                          │
│ Assignment Due Soon                      │
│ Complete "Treble Clef Notes" by 3:00 PM │
│ [VIEW ASSIGNMENT] [DISMISS]              │
└──────────────────────────────────────────┘
```

### Accessibility Considerations

Reference: [Accessibility Standards](../01-ux-design-system/a11y-standards.md)

**Push Notification Accessibility**:
- **Screen readers**: Notification content read aloud when received (OS native behavior)
- **VoiceOver/TalkBack**: Notification actions are focusable and labeled
- **Text size**: Notifications respect device text size settings
- **Reduce motion**: No animated GIFs in notifications if user has reduce motion enabled
- **Sound**: Option to disable sound but keep visual notifications
- **Vibration**: Customizable vibration patterns for different categories

**Permission UI Accessibility**:
- High contrast mode support for permission prompts
- Keyboard navigation for web permission prompts
- Clear focus indicators on buttons
- Descriptive labels for screen readers

## Telemetry

Reference: [Event Model](../15-analytics-and-reporting/event-model.md) for complete event tracking specifications.

### Device Registration Events

#### Device Registered
```javascript
{
  event_type: "push_device_registered",
  properties: {
    device_id: "dev_1a2b3c4d",
    device_type: "ios",
    platform: "iOS",
    platform_version: "17.2",
    device_model: "iPhone 14 Pro",
    user_id: "usr_12345",
    permission_status: "granted",
    registration_method: "app_launch" | "settings_toggle"
  }
}
// Fires: Client-side when device successfully registers
```

#### Device Permission Changed
```javascript
{
  event_type: "push_permission_changed",
  properties: {
    device_id: "dev_1a2b3c4d",
    old_status: "not_determined",
    new_status: "granted" | "denied" | "provisional",
    changed_from: "permission_prompt" | "os_settings",
    user_id: "usr_12345"
  }
}
// Fires: Client-side when permission status changes
```

#### Device Token Refreshed
```javascript
{
  event_type: "push_token_refreshed",
  properties: {
    device_id: "dev_1a2b3c4d",
    device_type: "android",
    token_age_days: 45,
    refresh_reason: "fcm_rotation" | "manual_refresh",
    user_id: "usr_12345"
  }
}
// Fires: Client-side when push token rotates
```

### Push Notification Lifecycle Events

#### Push Notification Sent
```javascript
{
  event_type: "push_notification_sent",
  properties: {
    push_id: "push_1a2b3c4d",
    notification_id: "notif_xyz789",
    user_id: "usr_12345",
    device_ids: ["dev_1a2b3c4d", "dev_5e6f7g8h"],
    device_count: 2,
    notification_type: "assignment_due_soon",
    priority: "high",
    platform: "ios" | "android" | "web",
    has_image: true,
    has_actions: true,
    provider: "apns" | "fcm" | "web_push"
  }
}
// Fires: Server-side when push sent to provider
```

#### Push Notification Delivered
```javascript
{
  event_type: "push_notification_delivered",
  properties: {
    push_id: "push_1a2b3c4d",
    device_id: "dev_1a2b3c4d",
    notification_id: "notif_xyz789",
    delivery_latency_ms: 1250,
    provider_message_id: "apns_msg_123",
    delivered_at: "2025-01-15T14:30:01Z"
  }
}
// Fires: Webhook from push provider (FCM only; APNs doesn't provide)
```

#### Push Notification Opened
```javascript
{
  event_type: "push_notification_opened",
  properties: {
    push_id: "push_1a2b3c4d",
    device_id: "dev_1a2b3c4d",
    notification_id: "notif_xyz789",
    user_id: "usr_12345",
    time_to_open_seconds: 125,
    action_taken: "default" | "action_button_1",
    deep_link_target: "mlc://assignment/asg_789",
    app_state_on_open: "foreground" | "background" | "not_running"
  }
}
// Fires: Client-side when user taps notification
```

#### Push Notification Dismissed
```javascript
{
  event_type: "push_notification_dismissed",
  properties: {
    push_id: "push_1a2b3c4d",
    device_id: "dev_1a2b3c4d",
    notification_id: "notif_xyz789",
    dismiss_method: "swipe" | "clear_all" | "timeout",
    time_since_delivered_seconds: 300
  }
}
// Fires: Client-side when notification cleared without opening
```

#### Push Notification Failed
```javascript
{
  event_type: "push_notification_failed",
  properties: {
    push_id: "push_1a2b3c4d",
    device_id: "dev_1a2b3c4d",
    notification_id: "notif_xyz789",
    failure_reason: "invalid_token" | "payload_too_large" | "provider_error",
    provider_error_code: "410",
    provider_error_message: "Unregistered",
    will_retry: false
  }
}
// Fires: Server-side when push delivery fails
```

### Push Preference Events

#### Push Category Toggled
```javascript
{
  event_type: "push_category_toggled",
  properties: {
    user_id: "usr_12345",
    category: "assignments",
    old_value: true,
    new_value: false,
    changed_from: "settings_page" | "notification_action"
  }
}
// Fires: Client-side when user changes category preference
```

#### Quiet Hours Updated
```javascript
{
  event_type: "push_quiet_hours_updated",
  properties: {
    user_id: "usr_12345",
    enabled: true,
    start_time: "20:00",
    end_time: "07:00",
    timezone: "America/New_York",
    days_of_week: [0,1,2,3,4,5,6]
  }
}
// Fires: Client-side when quiet hours changed
```

### Aggregated Push Metrics

These metrics feed into dashboards and reports:

#### Delivery Metrics
- **Delivery rate**: delivered / sent (target: >98% for FCM, unknown for APNs)
- **Failure rate**: failed / sent (target: <2%)
  - By failure reason (invalid token, provider error, etc.)
- **Token expiration rate**: expired tokens / active tokens (target: <1% per week)

#### Engagement Metrics
- **Open rate**: opened / delivered (by notification type)
- **Time to open**: average time from delivery to open
- **Dismiss rate**: dismissed / delivered (track annoyance)
- **Action click rate**: action clicks / opened (for notifications with buttons)

#### Platform Performance
- **Provider latency**: time to send to APNs/FCM (target: <500ms P95)
- **Delivery latency**: time from send to device receipt (FCM only)
- **Permission grant rate**: granted / prompted (target: >60%)
- **Device registration success rate**: registered / attempted (target: >99%)

#### User Segmentation
- **Push enabled users**: % of active users with push enabled
- **Multi-device users**: users with 2+ registered devices
- **Platform distribution**: iOS vs Android vs Web vs Desktop
- **Permission denied rate**: % of users who denied permission

## Permissions

Reference: [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md)

### Push Notification Sending Permissions

#### Automated System
- **Send transactional push**: Send all transactional push notifications based on triggers
- **Send promotional push**: Send promotional push to users with opt-in
- **Respect preferences**: Honor user push notification preferences and quiet hours
- **Throttle sends**: Enforce rate limits per user and overall system

#### System Admin
- **Manual send**: Trigger individual push sends for testing
- **Broadcast push**: Send platform-wide push announcements
- **Override preferences**: Send critical push regardless of preferences (security only)
- **View all devices**: Access device registration data across all users
- **Revoke tokens**: Manually mark device tokens as expired
- **Test push**: Send test push to own devices or designated test accounts

#### Subscriber Admin
- **Organization broadcasts**: Send push to all users in their organization
- **Cannot override**: Must respect user opt-out preferences
- **View org devices**: See device count and stats for their organization (aggregated only)
- **Cannot access**: Individual user device tokens or push logs

#### Teacher / Teacher-Admin
- **No push sending**: Cannot send push notifications directly
- **Triggered push**: Push notifications sent automatically based on their actions (e.g., grading assignment triggers student push)
- **Cannot access**: Push infrastructure or device data

#### Student
- **Receive push**: Can receive push notifications per their preferences
- **Manage own devices**: View and manage their registered devices
- **Cannot send**: Cannot send push notifications to others
- **Cannot access**: Other users' device data

### Device Management Permissions

#### User (Any Role)
- **Register own devices**: Register devices they own for push notifications
- **View own devices**: See list of their registered devices
- **Remove own devices**: Deactivate or remove their devices
- **Manage preferences**: Configure push preferences per category
- **Cannot access**: Other users' device data

#### System Admin
- **View all devices**: Access complete device registry
- **Remove any device**: Deactivate user devices (support purposes)
- **Export device data**: Download device registration reports
- **Token management**: Manually refresh or revoke tokens

#### Subscriber Admin
- **View org stats**: See aggregate device stats for their organization
- **Cannot access**: Individual device tokens or user device lists

### Privacy and Data Protection Rules

- **Device tokens**: Treated as sensitive data, encrypted at rest
- **Device information**: Model, OS version stored for compatibility, not shared
- **Push content**: Encrypted in transit (HTTPS, APNs/FCM native encryption)
- **Content retention**: Push notification content retained per retention policy (default: 90 days)
- **Delivery logs**: Push delivery logs retained for 1 year (compliance)
- **Token rotation**: Automatic token refresh tracked for security
- **Right to erasure**: User can request deletion of device data (GDPR)
- **COPPA compliance**: Students under 13 require parental consent for push notifications (verify with guardian email)
- **FERPA compliance**: Educational content in push protected per role boundaries

## Dependencies

### Core System Dependencies

- **[In-app Notifications](./in-app-notifications.md)**: Push notifications triggered from in-app notification creation
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: System events that trigger push notifications
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: User role definitions for push targeting
- **[Session and Auth](../02-roles-and-permissions/session-and-auth-states.md)**: User authentication for device registration

### Role-Specific Notification Sources

- **[Student Notifications](../03-student-experience/student-notifications.md)**: Student push triggers and content
- **[Teacher Notifications](../04-teacher-experience/teacher-notifications.md)**: Teacher push triggers and content
- **[Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md)**: Admin push triggers and content

### Communication Infrastructure

- **[Email Templates](./email-templates.md)**: Fallback channel when push fails
- **[System Announcements](./system-announcements.md)**: Broadcast push announcements
- **[Communication Framework](./communication-framework.md)**: Overall communication policies

### Technical Infrastructure

- **iOS APNs** (Apple Push Notification service): Native iOS push delivery
- **Android FCM** (Firebase Cloud Messaging): Native Android push delivery
- **Web Push API**: Browser-based push notifications via Service Workers
- **VAPID** (Voluntary Application Server Identification): Web push authentication
- **Service Worker**: Web push notification display and action handling
- **CDN**: Image hosting for rich push notification media
- **Message Queue**: Reliable push notification queuing (Redis, RabbitMQ)
- **Database**: PostgreSQL for device registration and push logs
- **Cache Layer**: Redis for active device lookups and badge counts

### Mobile and Web Dependencies

- **[PWA Specifications](../21-mobile-and-offline/pwa-specifications.md)**: Web push requirements for PWA
- **[Responsive Behaviors](../21-mobile-and-offline/responsive-behaviors.md)**: Mobile-optimized notification settings UI
- **Native Mobile Apps** (future): iOS and Android app implementations

### Compliance and Legal

- **[Privacy Policy](../23-legal-and-compliance/privacy-policy.md)**: Push notification privacy disclosures
- **[COPPA/FERPA Compliance](../23-legal-and-compliance/coppa-ferpa-notes.md)**: Student push notification restrictions
- **[GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md)**: EU user push requirements (if applicable)

## Open questions

### Technical Implementation

- **Push Service Provider**: Build in-house or use third-party service?
  - **In-house**: Direct APNs/FCM integration. Pros: Full control, no per-notification cost. Cons: More engineering effort, maintain certificates.
  - **Third-party**: OneSignal, Pusher, Airship. Pros: Easier setup, analytics included, cross-platform. Cons: Ongoing cost, vendor lock-in.
  - **Recommendation**: Start with direct APNs/FCM integration for Phase 1 (lower cost, platform learning). Evaluate third-party if scaling issues or need advanced features (A/B testing, segmentation).

- **Certificate Management**: How to securely store and rotate APNs certificates?
  - APNs certificates expire annually
  - Need secure key storage (AWS Secrets Manager, HashiCorp Vault)
  - Automated rotation process with alerts
  - **Recommendation**: Use APNs authentication token (JWT) instead of certificates (doesn't expire)

- **Web Push VAPID Keys**: Generate once or per-user? How to store securely?
  - Single set of VAPID keys for entire platform (typical approach)
  - Store private key in secrets manager, never expose
  - Public key included in web app config
  - **Recommendation**: Single VAPID key pair, stored in AWS Secrets Manager, rotated annually

- **Token Storage**: Store push tokens in main database or separate service?
  - Main database simpler but increases load
  - Separate service (Redis) faster for lookups
  - **Recommendation**: Store registrations in PostgreSQL, cache active tokens in Redis with 7-day TTL

### User Experience

- **Push Frequency Limits**: How many push notifications per day before users annoyed?
  - Research suggests 3-5 per day max for non-critical notifications
  - High-priority (assignment due soon) exempt from limits
  - **Recommendation**: Max 5 normal-priority push per day, 10 high-priority per week, unlimited blocking
  - **Open**: Should we enforce daily limits or just warn users in settings?

- **Default Push Settings**: Which categories enabled by default?
  - All enabled risks overwhelming users
  - Too few enabled reduces engagement
  - **Recommendation**: Students: Assignments + Messages enabled by default. Teachers: Student progress + Messages. Admins: Billing + System.
  - **Open**: Should we use smart defaults based on user behavior after 1 week?

- **Notification Sounds**: Use system default or custom sounds?
  - Custom sounds improve brand recognition
  - Requires packaging sounds in native apps (~100KB per sound)
  - **Recommendation**: System default for Phase 1, custom sounds in Phase 2 if user testing shows value
  - **Open**: If custom sounds, different sounds per category or single brand sound?

- **Badge Count Sync**: How to keep badge count accurate across devices?
  - Badge can drift if notifications read on one device but not updated on others
  - Silent push can sync badge but requires background processing
  - **Recommendation**: Daily badge sync via silent push, immediate sync on notification read via WebSocket if available
  - **Open**: What if user has 100+ unread notifications? Cap badge at 99 or show "99+"?

### Content and Localization

- **Push Notification Length**: How much content to include?
  - Longer content requires expanding notification
  - iOS limit: 178 chars visible in collapsed notification
  - Android limit: ~40 chars in status bar, ~200 in expanded
  - **Recommendation**: Title max 50 chars, body max 150 chars, full content available in-app
  - **Open**: Should we A/B test shorter vs longer body text for open rates?

- **Emoji in Push**: Use emoji in push notification content?
  - Can improve engagement and emotional connection
  - May appear unprofessional for some audiences (teachers/admins)
  - **Recommendation**: Use emoji for student push notifications only, not for teachers/admins
  - **Open**: Which emoji are universally understood and appropriate for education context?

- **Localization Strategy**: Translate push notifications or only support English?
  - Translation adds complexity (manage strings, hire translators)
  - Large non-English markets may require localization
  - **Recommendation**: Phase 1 English only, Phase 2 add Spanish if >10% users prefer Spanish
  - **Open**: How to detect user language preference? Device locale or user profile setting?

### Privacy and Compliance

- **COPPA Compliance**: How to handle push for students under 13?
  - COPPA requires parental consent for push notifications to minors
  - Options: (1) Require guardian approval before enabling push, (2) Send push to guardian device instead
  - **Recommendation**: Option 1 - Show in-app message to student: "Ask your parent to enable notifications in your account settings"
  - **Open**: How to verify guardian approval? Email confirmation or in-app switch in guardian account?

- **Parental Controls**: Should guardians be able to control student push settings?
  - Guardians may want to disable certain categories (e.g., challenges) but not others (assignments)
  - Adds complexity to settings UI and permissions
  - **Recommendation**: Phase 2 feature, start with student controls only in Phase 1
  - **Open**: If implemented, do guardian settings override student preferences or complement them?

- **Data Retention**: How long to keep push notification logs?
  - Longer retention useful for debugging and compliance
  - Increases storage costs and GDPR risk (right to erasure)
  - **Recommendation**: Push content logs: 90 days. Delivery receipts: 1 year. Aggregated analytics: indefinite.
  - **Open**: Should we anonymize user_id in logs after 90 days?

- **Opt-Out Tracking**: Track users who opt out and never re-prompt?
  - Respecting user choice vs. encouraging re-engagement
  - **Recommendation**: Track opt-out timestamp, don't re-prompt via UI for 180 days
  - **Open**: What about contextual prompts (e.g., "Enable notifications to never miss a due date") - are these acceptable or annoying?

### Performance and Scalability

- **Push Sending Rate**: How many pushes can we send per second?
  - APNs: Unlimited with HTTP/2 (no official limit)
  - FCM: 1 million messages per second (typical account limit)
  - **Recommendation**: Start with 100 push/second, scale up based on user base
  - **Open**: At what user count do we need dedicated APNs connection pool?

- **Retry Strategy**: How aggressively to retry failed pushes?
  - Too many retries waste resources
  - Too few retries miss deliveries due to transient errors
  - **Recommendation**: 3 retries with exponential backoff (1s, 5s, 25s) for 5xx errors only
  - **Open**: Should we retry indefinitely for high-priority notifications?

- **Silent Push Frequency**: How often to send silent pushes for badge sync?
  - iOS limits background push wakeups (not documented but ~2-3 per hour)
  - Too frequent drains battery and annoys users
  - **Recommendation**: Badge sync once per day at user's typical wake time
  - **Open**: Is daily enough or should we sync on each notification read via WebSocket?

- **Notification Expiration**: How long to keep undelivered pushes in queue?
  - APNs: Default 30 days if not specified
  - FCM: Default 4 weeks
  - Older notifications may be irrelevant (e.g., "assignment due in 1 hour" sent yesterday)
  - **Recommendation**: High-priority: 24 hours. Normal: 7 days. Low: 3 days.
  - **Open**: Should expiration vary by notification type (time-sensitive vs. informational)?
