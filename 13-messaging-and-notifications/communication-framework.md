# Communication Framework

## Purpose

This specification defines the overarching communication and notification framework for the MLC-LMS platform, establishing the strategic foundation for how the system communicates with users across multiple channels. The framework serves as the orchestration layer that coordinates in-app notifications, email, push notifications, and future channels (SMS, desktop) to create a cohesive, user-centered communication experience.

The communication framework is the "brain" of the notification system—it defines when to communicate, through which channels, at what frequency, and with what priority. While individual channel specifications ([In-app Notifications](./in-app-notifications.md), [Email Templates](./email-templates.md), [Push and Desktop](./push-and-desktop.md)) define how messages are delivered through specific channels, this framework defines the rules, policies, and decision logic that govern the entire communication lifecycle.

This system balances the platform's need to inform, engage, and alert users with the user's need to control the volume, timing, and nature of communications they receive. By establishing clear principles, preference structures, and channel coordination rules, the framework prevents notification fatigue while ensuring critical information reaches users reliably.

## Scope

**Included**
- Communication philosophy and design principles
- Multi-channel communication strategy and orchestration
- User notification preferences and consent management
- Preference center interface and experience
- Channel selection logic and priority routing
- Communication frequency limits and throttling rules
- Quiet hours and do-not-disturb policies
- Notification lifecycle orchestration (trigger → routing → delivery → tracking)
- Cross-channel coordination and fallback strategies
- Communication governance and rate limiting
- Emergency communication escalation paths
- Communication analytics and health monitoring
- Notification opt-in and opt-out workflows
- Digest and batching strategies
- Communication personalization framework
- Channel-specific adaptation rules

**Excluded**
- Channel-specific technical delivery (see [In-app Notifications](./in-app-notifications.md), [Email Templates](./email-templates.md), [Push and Desktop](./push-and-desktop.md))
- System announcement authoring interface (see [System Announcements](./system-announcements.md))
- Event definitions and telemetry schema (see [Event Model](../15-analytics-and-reporting/event-model.md))
- Role-specific notification content and triggers (see [Student Notifications](../03-student-experience/student-notifications.md), [Teacher Notifications](../04-teacher-experience/teacher-notifications.md), [Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md))
- Direct messaging between users (see [Student Messages](../03-student-experience/student-messages.md), [Teacher Messaging](../04-teacher-experience/teacher-messaging.md))

## Communication Philosophy and Principles

### Guiding Principles

**1. User Empowerment**
- Users have full control over what communications they receive and how
- Preferences are granular, easy to understand, and immediately effective
- Opt-out is always honored (except critical security/system communications)
- Users can pause communications temporarily without losing their settings

**2. Relevance Over Volume**
- Only send communications that provide clear value to the recipient
- Personalize content and timing based on user context and behavior
- Quality over quantity—fewer, more meaningful communications
- Respect user attention as a limited resource

**3. Right Channel, Right Time**
- Choose delivery channel based on urgency, content type, and user preference
- Coordinate across channels to avoid redundancy and noise
- Respect quiet hours and user availability patterns
- Provide consistency of information across all channels

**4. Progressive Disclosure**
- Start with minimal interruption (notification center)
- Escalate to more intrusive channels only when justified
- Critical information can escalate across channels over time if unacknowledged
- Give users control over escalation paths

**5. Transparency and Trust**
- Clear explanation of why each communication was sent
- Easy access to preference management from any communication
- Honest about data usage and communication practices
- Build trust through consistent, respectful communication

**6. Accessibility First**
- All communications accessible to users with disabilities
- Support for screen readers, keyboard navigation, and assistive technology
- Clear, plain language in all communications
- Multiple ways to access the same information

### Communication Categories

All platform communications fall into these strategic categories:

**Transactional** (Cannot opt out)
- Assignment creation and due date notifications
- Grade and feedback notifications
- Account security alerts
- Password reset confirmations
- Payment receipts and billing notifications
- Terms of service updates requiring acknowledgment

**Educational** (Can customize frequency)
- Learning progress updates
- Achievement and badge notifications
- Practice reminders and streak alerts
- Review and retention prompts
- Teacher feedback and encouragement

**Engagement** (Can opt out)
- Challenge invitations from peers
- Leaderboard updates
- Feature tips and best practices
- Community highlights
- Motivational nudges

**Administrative** (Can customize by role)
- Student enrollment notifications (teachers)
- Student performance alerts (teachers)
- Billing and subscription updates (admins)
- Organization announcements
- System maintenance notices

**Promotional** (Must opt in)
- New feature announcements
- Platform updates and enhancements
- Tips and tricks newsletters
- Seasonal campaigns
- Special offers

## Data model (if applicable)

### UserCommunicationPreferences Entity

```typescript
interface UserCommunicationPreferences {
  // Identification
  userId: string;
  organizationId: string;
  
  // Global Settings
  globalPreferences: {
    communicationsEnabled: boolean;     // Master switch (except critical)
    preferredLanguage: string;          // ISO 639-1 code
    timezone: string;                   // IANA timezone identifier
    emailAddress: string;               // Primary email
    guardianEmailAddress?: string;      // For students <13 (COPPA)
    phoneNumber?: string;               // For SMS (future)
  };
  
  // Category Preferences
  categoryPreferences: {
    [key in CommunicationCategory]: CategoryPreference;
  };
  
  // Quiet Hours
  quietHours: QuietHoursConfig;
  
  // Channel Preferences
  channelPreferences: {
    inApp: ChannelPreference;
    email: ChannelPreference;
    push: ChannelPreference;
    desktop: ChannelPreference;
    sms: ChannelPreference;            // Future
  };
  
  // Digest Preferences
  digestPreferences: DigestConfig;
  
  // Frequency Limits
  frequencyLimits: {
    maxDailyEmails: number;            // User-configurable
    maxDailyPush: number;
    maxWeeklyDigests: number;
    respectSystemLimits: boolean;      // Override system defaults
  };
  
  // Metadata
  lastUpdatedAt: Date;
  lastUpdatedBy: 'user' | 'system' | 'admin';
  preferencesVersion: number;
}
```

### CategoryPreference Interface

```typescript
interface CategoryPreference {
  enabled: boolean;                    // Master switch for category
  channels: ('in_app' | 'email' | 'push' | 'desktop' | 'sms')[];
  frequency: 'immediate' | 'hourly' | 'daily' | 'weekly' | 'disabled';
  priority: 'high' | 'normal' | 'low'; // How to treat within category
  customRules?: {
    minScoreToNotify?: number;         // E.g., only notify for scores >80%
    daysBeforeDue?: number;            // E.g., only assignment reminders 1 day before
  };
}
```

### QuietHoursConfig Interface

```typescript
interface QuietHoursConfig {
  enabled: boolean;
  schedule: QuietHoursPeriod[];
  exemptPriorities: ('critical' | 'high')[];  // What bypasses quiet hours
  queueForLater: boolean;              // Queue or drop messages during quiet hours
  timezone: string;                    // IANA timezone (defaults to user timezone)
}

interface QuietHoursPeriod {
  daysOfWeek: number[];                // 0=Sunday, 6=Saturday
  startTime: string;                   // HH:mm format (24-hour)
  endTime: string;                     // HH:mm format
  label?: string;                      // User-assigned label, e.g., "Sleep hours"
}
```

### ChannelPreference Interface

```typescript
interface ChannelPreference {
  enabled: boolean;
  categories: CommunicationCategory[];  // Which categories allowed on this channel
  devicePreference?: 'all' | 'primary' | 'mobile_only' | 'desktop_only';
}
```

### DigestConfig Interface

```typescript
interface DigestConfig {
  enabled: boolean;
  frequency: 'daily' | 'weekly' | 'monthly';
  deliveryDay?: number;                // For weekly (0-6) or monthly (1-31)
  deliveryTime: string;                // HH:mm format
  categories: CommunicationCategory[]; // What to include in digest
  minItemsToSend: number;              // Don't send if fewer items
  includeCategories: {
    [key in CommunicationCategory]?: boolean;
  };
}
```

### CommunicationCategory Enum

```typescript
type CommunicationCategory =
  // Core Learning
  | 'assignments'
  | 'progress'
  | 'achievements'
  | 'reviews'
  
  // Social & Engagement
  | 'challenges'
  | 'messages'
  | 'leaderboards'
  
  // Teacher/Admin
  | 'student_alerts'
  | 'billing'
  | 'administrative'
  
  // Platform
  | 'system'
  | 'security'
  | 'feature_updates'
  | 'tips_and_tricks'
  | 'newsletters';
```

### CommunicationRequest Entity

```typescript
interface CommunicationRequest {
  // Identification
  requestId: string;                   // UUID
  sourceEventId?: string;              // Links to triggering event
  
  // Content
  templateId: string;                  // Template to use for rendering
  category: CommunicationCategory;
  priority: 'critical' | 'high' | 'normal' | 'low';
  
  // Recipient
  recipientUserId: string;
  recipientRole: UserRole;
  organizationId: string;
  
  // Channel Selection
  requestedChannels: ('in_app' | 'email' | 'push' | 'desktop' | 'sms')[];
  requiredChannel?: 'in_app' | 'email'; // Must deliver via this channel
  
  // Scheduling
  scheduledFor?: Date;                 // Deliver at specific time
  expiresAt?: Date;                    // Don't deliver after this time
  
  // Personalization
  variables: Record<string, any>;      // Data for template rendering
  locale?: string;                     // Override user locale
  
  // Routing Hints
  bypassQuietHours: boolean;
  bypassRateLimits: boolean;
  allowBatching: boolean;
  
  // Lifecycle
  status: 'pending' | 'routing' | 'scheduled' | 'delivered' | 'failed' | 'expired';
  createdAt: Date;
  processedAt?: Date;
}
```

### CommunicationDelivery Entity

```typescript
interface CommunicationDelivery {
  // Identification
  deliveryId: string;                  // UUID
  requestId: string;                   // Parent request
  userId: string;
  
  // Channel-specific IDs
  inAppNotificationId?: string;
  emailMessageId?: string;
  pushNotificationId?: string;
  
  // Delivery Status
  channelsDelivered: {
    inApp?: DeliveryStatus;
    email?: DeliveryStatus;
    push?: DeliveryStatus;
    desktop?: DeliveryStatus;
  };
  
  // Engagement
  firstEngagedAt?: Date;
  firstEngagedChannel?: string;
  engagementType?: 'viewed' | 'clicked' | 'dismissed';
  
  // Timing
  requestedAt: Date;
  routedAt?: Date;
  deliveredAt?: Date;
  
  // Metadata
  wasBatched: boolean;
  batchId?: string;
  fallbackChannelUsed?: string;
}

interface DeliveryStatus {
  status: 'pending' | 'sent' | 'delivered' | 'failed' | 'skipped';
  attemptCount: number;
  lastAttemptAt?: Date;
  failureReason?: string;
  deliveredAt?: Date;
}
```

### CommunicationRateLimit Entity

```typescript
interface CommunicationRateLimit {
  // Identification
  userId: string;
  category: CommunicationCategory;
  channel: string;
  
  // Limits
  windowStart: Date;
  windowEnd: Date;
  maxAllowed: number;
  currentCount: number;
  
  // Tracking
  communicationIds: string[];          // Recent communications in window
  lastResetAt: Date;
  nextResetAt: Date;
}
```

## Behavior and rules

### Communication Lifecycle and Orchestration

The communication system follows a consistent lifecycle from trigger to delivery:

```
Event/Trigger → Communication Request → Routing & Channel Selection
     ↓
Preference Check → Rate Limiting → Quiet Hours Check
     ↓
Channel-Specific Delivery → Tracking → Engagement Analysis
```

#### 1. Communication Request Creation

**Trigger Sources**:
- System events (from [Event Model](../15-analytics-and-reporting/event-model.md))
- User actions (assignment completion, badge earned, etc.)
- Scheduled communications (daily digest, weekly summary)
- Admin-initiated broadcasts ([System Announcements](./system-announcements.md))
- API-triggered communications (external integrations)

**Request Validation**:
- Verify recipient exists and is active
- Confirm category and priority are valid
- Check that at least one delivery channel is possible
- Ensure required personalization variables are present
- Validate scheduling constraints (future date, before expiration)

**Example Request Creation**:
```javascript
// Assignment due reminder request
const request = {
  requestId: uuid(),
  sourceEventId: "evt_assignment_due_soon",
  templateId: "tmpl_assignment_reminder",
  category: "assignments",
  priority: "high",
  recipientUserId: "usr_student_123",
  recipientRole: "student",
  requestedChannels: ["in_app", "email", "push"],
  scheduledFor: calculateOptimalDeliveryTime(user),
  variables: {
    student_name: "Sarah",
    assignment_name: "Treble Clef Notes",
    due_date_relative: "in 2 hours",
    due_date_absolute: "2025-01-15T15:00:00Z"
  },
  bypassQuietHours: false,
  bypassRateLimits: false,
  allowBatching: true
};
```

#### 2. Routing and Channel Selection

**Channel Selection Algorithm**:

```javascript
function selectDeliveryChannels(request, userPreferences) {
  const selectedChannels = [];
  
  // Step 1: Check if communication globally disabled
  if (!userPreferences.globalPreferences.communicationsEnabled 
      && request.priority !== 'critical') {
    return [];  // User has disabled all communications
  }
  
  // Step 2: Check category preferences
  const categoryPref = userPreferences.categoryPreferences[request.category];
  if (!categoryPref.enabled) {
    // Category disabled, but allow critical
    if (request.priority === 'critical') {
      return ['in_app', 'email'];  // Critical always delivered
    }
    return [];
  }
  
  // Step 3: Apply frequency rules
  if (isOverFrequencyLimit(request, categoryPref)) {
    // Exceeded frequency, batch for later
    if (request.allowBatching) {
      addToBatchQueue(request);
      return [];
    }
    // Don't batch, downgrade to in-app only
    return ['in_app'];
  }
  
  // Step 4: Select channels from user preferences
  for (const channel of categoryPref.channels) {
    // Check if channel enabled
    const channelPref = userPreferences.channelPreferences[channel];
    if (!channelPref.enabled) continue;
    
    // Check if category allowed on this channel
    if (!channelPref.categories.includes(request.category)) continue;
    
    // Channel is valid
    selectedChannels.push(channel);
  }
  
  // Step 5: Ensure at least in-app for important communications
  if (selectedChannels.length === 0 && request.priority !== 'low') {
    selectedChannels.push('in_app');  // Always deliver to notification center
  }
  
  // Step 6: Respect required channel
  if (request.requiredChannel && !selectedChannels.includes(request.requiredChannel)) {
    selectedChannels.push(request.requiredChannel);
  }
  
  return selectedChannels;
}
```

**Priority-Based Channel Override**:
- **Critical**: Always in-app + email, regardless of preferences (security/account issues)
- **High**: Respects preferences but suggests in-app + email + push if available
- **Normal**: Fully respects user preferences
- **Low**: In-app only, unless user explicitly enabled other channels for category

#### 3. Rate Limiting and Throttling

**System-Wide Rate Limits** (Default, can be customized by org):

| Channel | Per Hour | Per Day | Per Week |
|---------|----------|---------|----------|
| In-app | Unlimited | Unlimited | Unlimited |
| Email | 5 | 10 | 50 |
| Push | 10 | 20 | 100 |
| Desktop | 10 | 20 | 100 |

**Category-Specific Limits**:
- **Assignments**: Max 5 per day (due reminders, grading)
- **Progress**: Max 3 per day (achievements, milestones)
- **Challenges**: Max 10 per day (invitations, results)
- **Messages**: Max 20 per day (teacher feedback, replies)
- **Administrative**: Max 3 per day
- **System**: Max 2 per day (maintenance, security)

**Rate Limit Enforcement**:

```javascript
function checkRateLimit(request, userId, channel) {
  const limit = getRateLimitForCategory(request.category, channel);
  const window = getCurrentRateLimitWindow(channel);  // Hour, day, or week
  
  const currentCount = countCommunicationsInWindow(
    userId, 
    request.category, 
    channel, 
    window
  );
  
  if (currentCount >= limit.maxAllowed) {
    // Rate limit exceeded
    if (request.bypassRateLimits && request.priority === 'critical') {
      // Allow critical communications to bypass
      return { allowed: true, bypassed: true };
    }
    
    // Check if batching is allowed
    if (request.allowBatching) {
      addToBatchQueue(request, userId, channel);
      return { allowed: false, reason: 'rate_limit_batch' };
    }
    
    // Drop communication
    logRateLimitExceeded(request, userId, channel);
    return { allowed: false, reason: 'rate_limit_exceeded' };
  }
  
  // Within limits, proceed
  incrementRateLimitCounter(userId, request.category, channel);
  return { allowed: true };
}
```

**User-Configurable Limits**:
- Users can set stricter limits than system defaults
- Options: "Frequent" (default), "Moderate" (50% reduction), "Minimal" (80% reduction)
- Cannot increase limits beyond system maximum for their plan

#### 4. Quiet Hours Handling

**Quiet Hours Logic**:

```javascript
function isInQuietHours(userId, currentTime) {
  const preferences = getUserPreferences(userId);
  
  if (!preferences.quietHours.enabled) {
    return false;
  }
  
  const userTime = convertToUserTimezone(currentTime, preferences.globalPreferences.timezone);
  const dayOfWeek = userTime.getDay();
  
  for (const period of preferences.quietHours.schedule) {
    if (!period.daysOfWeek.includes(dayOfWeek)) continue;
    
    const start = parseTime(period.startTime);
    const end = parseTime(period.endTime);
    
    if (isTimeBetween(userTime, start, end)) {
      return true;  // Currently in quiet hours
    }
  }
  
  return false;
}

function handleQuietHours(request, userId) {
  if (!isInQuietHours(userId, new Date())) {
    return { proceed: true };  // Not in quiet hours, deliver normally
  }
  
  const preferences = getUserPreferences(userId);
  
  // Check if priority is exempt
  if (preferences.quietHours.exemptPriorities.includes(request.priority)) {
    return { proceed: true, bypassed: true };
  }
  
  // Queue for delivery after quiet hours
  if (preferences.quietHours.queueForLater) {
    const nextAvailableTime = calculateNextDeliveryTime(userId);
    scheduleForLater(request, nextAvailableTime);
    return { proceed: false, reason: 'quiet_hours_queued' };
  }
  
  // Drop communication (user chose not to queue)
  return { proceed: false, reason: 'quiet_hours_dropped' };
}
```

**Default Quiet Hours** (Suggested, user-configurable):
- **Students**: 9 PM - 7 AM local time (school nights)
- **Teachers**: 8 PM - 6 AM local time
- **Admins**: No default quiet hours (can configure)
- **All**: Weekends can have extended quiet hours if desired

**Quiet Hours End Batching**:
- When quiet hours end, queued communications delivered in batch
- Maximum 20 queued items delivered at once
- If >20 items, send digest notification: "You have 25 updates from quiet hours"
- User opens notification center to see all

#### 5. Digest and Batching Strategy

**Digest Configuration**:

```javascript
interface DigestConfiguration {
  // Frequency
  daily: {
    enabled: boolean;
    deliveryTime: "08:00";         // User's local time
    categories: ["progress", "achievements", "tips_and_tricks"];
    minItemsToSend: 3;
  };
  
  weekly: {
    enabled: boolean;
    deliveryDay: 0;                // Sunday
    deliveryTime: "18:00";
    categories: ["progress", "achievements", "challenges"];
    minItemsToSend: 5;
  };
  
  monthly: {
    enabled: boolean;
    deliveryDay: 1;                // First of month
    deliveryTime: "09:00";
    categories: ["progress", "newsletters"];
    minItemsToSend: 10;
  };
}
```

**Batching Rules**:

1. **Time-based batching**:
   - Low-priority notifications batched every 30 minutes
   - Normal-priority notifications batched if >5 pending
   - High-priority notifications never batched

2. **Category batching**:
   - Similar notifications grouped (e.g., multiple assignment completions)
   - Batch title: "3 assignments completed"
   - Batch delivery as single notification with expandable list

3. **Digest Generation**:
   - Background job runs at scheduled digest times
   - Collects all communications since last digest
   - Filters by user digest preferences
   - Renders email template with summary of items
   - Marks individual items as "included in digest"

**Digest Email Structure**:
```
Subject: Your Daily MLC Summary - 5 Updates

---
Hi Sarah,

Here's what happened today:

ASSIGNMENTS (2)
• Assignment "Treble Clef Notes" completed - Score: 95%
• New assignment "Bass Clef Practice" assigned - Due: Tomorrow

ACHIEVEMENTS (1)
• Badge earned: "Note Master" for mastering 50 notes

TIPS & TRICKS (2)
• New feature: Assignment reminders now available
• Teaching tip: How to use the practice scheduler

[View All in Dashboard]
---
```

#### 6. Channel Fallback and Escalation

**Fallback Strategy**:

```javascript
function handleDeliveryFailure(request, failedChannel, userId) {
  const fallbackOrder = {
    email: ['push', 'in_app'],
    push: ['email', 'in_app'],
    desktop: ['push', 'in_app'],
    in_app: []  // No fallback for in-app
  };
  
  const fallbacks = fallbackOrder[failedChannel];
  
  for (const fallbackChannel of fallbacks) {
    // Check if fallback channel is enabled for user
    const preferences = getUserPreferences(userId);
    if (preferences.channelPreferences[fallbackChannel].enabled) {
      // Attempt delivery via fallback
      const result = deliverViaChannel(request, fallbackChannel, userId);
      if (result.success) {
        logFallbackSuccess(request, failedChannel, fallbackChannel);
        return { delivered: true, channel: fallbackChannel };
      }
    }
  }
  
  // All fallbacks failed, ensure in-app delivery
  if (failedChannel !== 'in_app') {
    deliverViaChannel(request, 'in_app', userId);
  }
  
  return { delivered: false, reason: 'all_channels_failed' };
}
```

**Time-Based Escalation** (For critical communications):

```javascript
function setupEscalationPath(request, userId) {
  if (request.priority !== 'critical' && request.priority !== 'high') {
    return;  // Only critical/high priority escalate
  }
  
  const escalationSchedule = [
    { delay: 0, channels: ['in_app'] },
    { delay: 15 * 60, channels: ['in_app', 'push'] },      // +15 min
    { delay: 60 * 60, channels: ['in_app', 'push', 'email'] },  // +1 hour
    { delay: 4 * 60 * 60, channels: ['in_app', 'push', 'email'] }  // +4 hours (resend)
  ];
  
  for (const step of escalationSchedule) {
    scheduleEscalationStep(request, userId, step.delay, step.channels);
  }
  
  // Escalation stops when user engages with any channel
  onUserEngagement(request.requestId, () => {
    cancelEscalation(request.requestId);
  });
}
```

**Example Escalation Path**:
- **T+0**: Security alert delivered to notification center
- **T+15 min**: If not viewed, send push notification
- **T+1 hour**: If still not viewed, send email
- **T+4 hours**: Resend all channels if still unacknowledged
- **Stop**: User clicks any notification, escalation cancelled

### Multi-Channel Coordination

**Channel Synchronization Rules**:

1. **Content Consistency**:
   - Same core message across all channels
   - Channel-appropriate formatting (truncation for push, full text for email)
   - Deep links from external channels (email, push) to in-app content

2. **Engagement Tracking**:
   - Track first engagement across all channels
   - Mark communication as "engaged" when user interacts via any channel
   - Don't resend via other channels once engaged (unless escalation policy requires)

3. **Cross-Channel Deduplication**:
   - If user engages in-app, don't send redundant email
   - Exception: Email sent for record-keeping regardless of in-app engagement (receipts, confirmations)

4. **Channel-Specific Timing**:
   - In-app: Immediate delivery
   - Push: Immediate or batched based on priority
   - Email: Batched for normal-priority, immediate for high/critical
   - Desktop: Same as push

**Coordination Example**:
```
Assignment Due Reminder:
├─ In-app: Delivered immediately to notification center
├─ Push: Sent 2 hours before due (if user not recently active)
└─ Email: Sent 24 hours before due (backup reminder)

If user clicks in-app notification:
├─ Push: Cancelled (no longer needed)
└─ Email: Still sent (provides written record)
```

### Preference Management Workflows

#### First-Time User Preference Setup

**Onboarding Flow**:
1. User completes account creation
2. System creates default preferences based on role:
   - Students: Assignments, achievements, messages enabled
   - Teachers: Student alerts, messages, administrative enabled
   - Admins: Billing, administrative, system enabled
3. Show preference primer during first session:
   - "Would you like to receive push notifications?"
   - "When should we send your daily digest?"
   - "Set quiet hours to avoid interruptions"
4. User can skip and configure later

**Default Preferences by Role**:

```javascript
const defaultPreferences = {
  student: {
    categories: {
      assignments: { enabled: true, channels: ['in_app', 'email'], frequency: 'immediate' },
      progress: { enabled: true, channels: ['in_app'], frequency: 'daily' },
      achievements: { enabled: true, channels: ['in_app', 'push'], frequency: 'immediate' },
      challenges: { enabled: true, channels: ['in_app', 'push'], frequency: 'immediate' },
      messages: { enabled: true, channels: ['in_app', 'email', 'push'], frequency: 'immediate' },
      tips_and_tricks: { enabled: false, channels: [], frequency: 'disabled' }
    },
    quietHours: {
      enabled: true,
      schedule: [{ daysOfWeek: [0,1,2,3,4,5,6], startTime: "21:00", endTime: "07:00" }]
    },
    digest: { enabled: false }  // Off by default, can enable
  },
  
  teacher: {
    categories: {
      student_alerts: { enabled: true, channels: ['in_app', 'email'], frequency: 'immediate' },
      messages: { enabled: true, channels: ['in_app', 'email', 'push'], frequency: 'immediate' },
      administrative: { enabled: true, channels: ['in_app', 'email'], frequency: 'immediate' },
      tips_and_tricks: { enabled: true, channels: ['email'], frequency: 'weekly' }
    },
    quietHours: {
      enabled: true,
      schedule: [{ daysOfWeek: [0,1,2,3,4,5,6], startTime: "20:00", endTime: "06:00" }]
    },
    digest: { enabled: true, frequency: 'daily', deliveryTime: "07:00" }
  }
  
  // ... other roles
};
```

#### Preference Change Flow

**User-Initiated Changes**:
1. User navigates to Settings → Notifications
2. User modifies preferences (category toggles, channel selection, quiet hours)
3. Changes saved immediately (no "Save" button required)
4. Confirmation toast: "Preferences updated"
5. Changes effective immediately for new communications

**System-Initiated Changes** (Rare, with user notification):
- New category added to platform → Default to off, notify user of option
- Channel deprecated (e.g., SMS sunsetted) → Notify user, suggest alternative
- Organization policy change → Notify admins, explain impact

#### Preference Import/Export

**Use Cases**:
- Teacher sets up preferences, wants to recommend to students
- Admin wants to set organization-wide defaults
- User switches devices, wants to restore preferences

**Export Format** (JSON):
```json
{
  "version": "1.0",
  "userId": "usr_123",
  "exportedAt": "2025-01-15T10:00:00Z",
  "preferences": {
    "categories": { ... },
    "quietHours": { ... },
    "channelPreferences": { ... }
  }
}
```

### Consent and Opt-In Management

**COPPA Compliance** (Students under 13):
- Student communications sent to guardian email by default
- Guardian must opt in to email communications for student
- In-app notifications allowed (no PII in notification payload)
- Push notifications require guardian consent via guardian account

**GDPR Compliance** (EU users):
- Explicit consent required for promotional communications
- Transactional communications allowed based on legitimate interest
- Clear unsubscribe mechanism in every email
- Right to export communication history
- Right to erasure (anonymize user in communication logs)

**CAN-SPAM Compliance** (Email):
- Physical address in email footer
- Clear "Unsubscribe" link in promotional emails
- Honor unsubscribe within 10 business days
- No deceptive subject lines or headers

**Opt-In Workflows**:

```javascript
// Email opt-in for promotional communications
function requestEmailOptIn(userId, category) {
  sendEmail({
    to: getUserEmail(userId),
    subject: "Confirm your MLC newsletter subscription",
    body: `
      Click below to confirm you want to receive ${category} emails.
      We'll send you helpful tips and updates about MLC.
      
      [Confirm Subscription]
      
      Didn't request this? You can safely ignore this email.
    `,
    trackingType: 'opt_in_confirmation'
  });
  
  // User clicks confirmation link
  onConfirmation(() => {
    updatePreferences(userId, {
      categoryPreferences: {
        [category]: { enabled: true, channels: ['email'] }
      }
    });
    
    sendConfirmation(userId, `You're now subscribed to ${category} emails.`);
  });
}
```

### Communication Health Monitoring

**System Health Metrics**:
- **Delivery success rate**: % of communications successfully delivered per channel
- **User engagement rate**: % of communications user interacts with
- **Opt-out rate**: % of users opting out of categories over time
- **Rate limit hit rate**: % of communications dropped due to rate limiting
- **Quiet hours queue depth**: Number of communications queued during quiet hours

**Alerting Thresholds**:
- Email delivery success rate <95% → Alert engineering
- Push delivery success rate <90% → Alert engineering
- Opt-out rate >5% in category → Alert product team
- Rate limit hit rate >10% → Review limits, possibly too restrictive
- Quiet hours queue depth >500 per user → Review quiet hours strategy

**User-Specific Health Monitoring**:
- Track users with high dismiss rates (>80%) → Reduce communication frequency
- Track users with zero engagement (never clicks) → Offer preference review
- Track users who frequently change preferences → May indicate confusion, offer help

## UX requirements (if applicable)

### Preference Center Interface

The preference center is the central control panel where users manage all communication settings.

#### Preference Center Layout

```
┌─────────────────────────────────────────────────────────────┐
│  Settings → Notifications                                   │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  📬 Email Notifications                              [ON]   │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  Primary email: sarah@example.com             [Change]      │
│                                                              │
│  🔔 Push Notifications                             [ON]     │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│  Manage 2 devices                           [View Devices]  │
│                                                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                              │
│  Notification Categories                                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  📚 Assignments & Learning                    [ON]   │  │
│  │  New assignments, due dates, grades, reviews          │  │
│  │  Channels: ☑ In-app  ☑ Email  ☑ Push                │  │
│  │  Frequency: ● Immediate  ○ Daily digest              │  │
│  │  [Advanced Settings ▼]                               │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  🏆 Achievements & Progress                   [ON]   │  │
│  │  Badges, milestones, streaks                          │  │
│  │  Channels: ☑ In-app  ☐ Email  ☑ Push                │  │
│  │  Frequency: ● Immediate  ○ Daily digest              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  🎯 Challenges & Competitions                 [ON]   │  │
│  │  Challenge invites, leaderboard updates               │  │
│  │  Channels: ☑ In-app  ☐ Email  ☑ Push                │  │
│  │  Frequency: ● Immediate  ○ Daily digest              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  💬 Messages & Feedback                       [ON]   │  │
│  │  Teacher messages, replies, feedback                  │  │
│  │  Channels: ☑ In-app  ☑ Email  ☑ Push                │  │
│  │  Frequency: ● Immediate  ○ Daily digest              │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  💡 Tips & Tricks                            [OFF]  │  │
│  │  Feature tips, best practices, suggestions            │  │
│  │  Channels: ☐ In-app  ☐ Email  ☐ Push                │  │
│  │  [Enable to receive helpful tips]                    │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                              │
│  ⏰ Quiet Hours                                  [ON]       │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Don't send notifications during:                    │  │
│  │  Every day from 9:00 PM to 7:00 AM                   │  │
│  │  (Urgent notifications still delivered)              │  │
│  │  [Edit Schedule]                                     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  📊 Daily Digest                                 [OFF]      │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  Receive one daily email summary instead of          │  │
│  │  individual notifications.                            │  │
│  │  [Enable Daily Digest]                               │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                              │
│  🛠️ Advanced                                                │
│  • Pause all notifications for: [1 week ▼]     [Pause]    │
│  • Export preferences                         [Export]     │
│  • Reset to defaults                          [Reset]      │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

#### Category Card Expanded (Advanced Settings)

```
┌──────────────────────────────────────────────────────┐
│  📚 Assignments & Learning                    [ON]   │
│  New assignments, due dates, grades, reviews          │
│                                                        │
│  Channels:                                            │
│  ☑ In-app notifications                              │
│  ☑ Email                                             │
│  ☑ Push notifications                                │
│  ☐ Desktop notifications                             │
│                                                        │
│  Frequency:                                           │
│  ● Immediate (as they happen)                        │
│  ○ Hourly summary                                    │
│  ○ Daily digest (8:00 AM)                           │
│                                                        │
│  Custom Rules:                                        │
│  ☑ Only remind me 24 hours before due date          │
│  ☐ Only notify for scores above 90%                 │
│                                                        │
│  [Save] [Cancel]                                     │
└──────────────────────────────────────────────────────┘
```

#### Quiet Hours Configuration

```
┌──────────────────────────────────────────────────────┐
│  ⏰ Quiet Hours Schedule                              │
├──────────────────────────────────────────────────────┤
│                                                        │
│  ☑ Enable quiet hours                                │
│                                                        │
│  Schedule 1: Sleep Hours                             │
│  ┌────────────────────────────────────────────────┐  │
│  │  Days: ☑ Sun ☑ Mon ☑ Tue ☑ Wed ☑ Thu ☑ Fri ☑ Sat│  │
│  │  From: [9:00 PM ▼]  To: [7:00 AM ▼]            │  │
│  │  [Remove Schedule]                              │  │
│  └────────────────────────────────────────────────┘  │
│                                                        │
│  [+ Add Another Schedule]                            │
│                                                        │
│  What should happen during quiet hours?              │
│  ● Queue notifications for later                     │
│  ○ Don't send notifications at all                   │
│                                                        │
│  Allow these priorities during quiet hours:          │
│  ☑ Critical (security, account issues)               │
│  ☑ High (urgent assignments, important updates)      │
│  ☐ Normal (standard notifications)                   │
│                                                        │
│  [Save] [Cancel]                                     │
└──────────────────────────────────────────────────────┘
```

#### Digest Configuration

```
┌──────────────────────────────────────────────────────┐
│  📊 Notification Digest Settings                      │
├──────────────────────────────────────────────────────┤
│                                                        │
│  ☑ Enable daily digest                               │
│                                                        │
│  Delivery time: [8:00 AM ▼]                          │
│  (Your local time: America/Chicago)                  │
│                                                        │
│  Include in digest:                                   │
│  ☑ Progress updates                                   │
│  ☑ Achievements and badges                           │
│  ☑ Challenge results                                  │
│  ☑ Tips and tricks                                    │
│  ☐ Messages (always sent immediately)                │
│  ☐ Assignments (always sent immediately)             │
│                                                        │
│  Minimum items to send digest: [3  ▼]                │
│  (Don't send if fewer than 3 items)                  │
│                                                        │
│  ○ Enable weekly digest instead                      │
│    Delivery: [Sunday ▼] at [6:00 PM ▼]              │
│                                                        │
│  [Preview Example] [Save] [Cancel]                   │
└──────────────────────────────────────────────────────┘
```

### In-Email Preference Management

Every email includes quick preference management links in footer:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
You received this email because you're subscribed to 
Assignments & Learning notifications.

Manage preferences: All | Assignments | Unsubscribe

MusicLearningCommunity.com LLC
123 Music Lane, Wildwood, MO 63011
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Quick Actions**:
- **Manage preferences**: Opens full preference center
- **Assignments**: Toggles only assignment category
- **Unsubscribe**: One-click unsubscribe with confirmation page

### Notification Pause Feature

**Use Case**: User going on vacation, wants temporary break from notifications

```
┌──────────────────────────────────────────────────────┐
│  Pause All Notifications                              │
├──────────────────────────────────────────────────────┤
│                                                        │
│  Take a break from notifications without losing      │
│  your settings.                                       │
│                                                        │
│  Pause for:                                           │
│  ○ 1 day                                             │
│  ○ 1 week                                            │
│  ● 2 weeks                                           │
│  ○ 1 month                                           │
│  ○ Custom dates                                      │
│                                                        │
│  Start: [Today ▼]  End: [Jan 30, 2025 ▼]           │
│                                                        │
│  During the pause:                                    │
│  • You won't receive any email or push notifications │
│  • In-app notifications will be queued               │
│  • Critical security alerts will still be sent       │
│  • Your preferences will be restored automatically   │
│                                                        │
│  [Pause Notifications] [Cancel]                      │
└──────────────────────────────────────────────────────┘
```

### Accessibility Requirements

Reference: [Accessibility Standards](../01-ux-design-system/a11y-standards.md)

**Preference Center Accessibility**:
- Keyboard navigation: Tab through all controls
- Screen reader: Clear labels for all toggles and options
- Color contrast: WCAG AA compliance (4.5:1 minimum)
- Focus indicators: Clear 2px outline on focused elements
- ARIA labels: Descriptive labels for all interactive elements
- Status announcements: Screen reader announces when preferences saved

**Mobile Optimization**:
- Touch targets minimum 44x44px
- Single-column layout on narrow screens
- Expandable sections to reduce scrolling
- Sticky save button at bottom of screen

## Telemetry

Reference: [Event Model](../15-analytics-and-reporting/event-model.md) for complete event tracking specifications.

### Preference Management Events

#### Preferences Updated
```javascript
{
  event_type: "communication_preferences_updated",
  properties: {
    user_id: "usr_12345",
    user_role: "student",
    updated_by: "user" | "system" | "admin",
    changes: {
      category: "assignments",
      field: "channels",
      old_value: ["in_app", "email"],
      new_value: ["in_app", "email", "push"]
    },
    update_trigger: "settings_page" | "email_footer" | "onboarding"
  }
}
// Fires: Client-side when user changes preferences
```

#### Quiet Hours Configured
```javascript
{
  event_type: "quiet_hours_configured",
  properties: {
    user_id: "usr_12345",
    enabled: true,
    schedule_count: 1,
    total_quiet_hours_per_week: 70,
    exempt_priorities: ["critical", "high"]
  }
}
// Fires: Client-side when quiet hours set up
```

#### Digest Enabled
```javascript
{
  event_type: "digest_enabled",
  properties: {
    user_id: "usr_12345",
    frequency: "daily",
    delivery_time: "08:00",
    categories: ["progress", "achievements", "tips_and_tricks"],
    min_items: 3
  }
}
// Fires: Client-side when user enables digest
```

### Communication Delivery Events

#### Communication Routed
```javascript
{
  event_type: "communication_routed",
  properties: {
    request_id: "req_1a2b3c4d",
    user_id: "usr_12345",
    category: "assignments",
    priority: "high",
    selected_channels: ["in_app", "email", "push"],
    dropped_channels: [],
    reason_dropped: null,
    rate_limited: false,
    in_quiet_hours: false,
    batched: false
  }
}
// Fires: Server-side after routing decision
```

#### Communication Rate Limited
```javascript
{
  event_type: "communication_rate_limited",
  properties: {
    request_id: "req_1a2b3c4d",
    user_id: "usr_12345",
    category: "progress",
    channel: "email",
    limit_type: "daily",
    current_count: 10,
    max_allowed: 10,
    action_taken: "batched" | "dropped"
  }
}
// Fires: Server-side when rate limit hit
```

#### Communication Queued for Quiet Hours
```javascript
{
  event_type: "communication_queued_quiet_hours",
  properties: {
    request_id: "req_1a2b3c4d",
    user_id: "usr_12345",
    category: "achievements",
    queued_at: "2025-01-15T22:30:00Z",
    scheduled_delivery: "2025-01-16T07:00:00Z",
    queue_depth: 5
  }
}
// Fires: Server-side when communication queued
```

### User Engagement Events

#### Communication Engaged
```javascript
{
  event_type: "communication_engaged",
  properties: {
    delivery_id: "del_1a2b3c4d",
    user_id: "usr_12345",
    category: "assignments",
    first_engaged_channel: "in_app",
    engagement_type: "viewed" | "clicked" | "dismissed",
    time_to_engage_seconds: 125,
    other_channels_delivered: ["email", "push"]
  }
}
// Fires: Client-side when user first engages
```

### Aggregated Metrics

**Communication Health Metrics**:
- **Cross-channel engagement rate**: % of users who engage via any channel
- **Preferred channel by category**: Which channel users engage with most
- **Rate limit hit frequency**: How often rate limits are reached
- **Quiet hours effectiveness**: % of communications successfully queued
- **Digest adoption rate**: % of users with digest enabled

**Preference Change Patterns**:
- **Opt-out velocity**: How quickly users opt out after sign-up
- **Category popularity**: Which categories most enabled/disabled
- **Preference change frequency**: Average changes per user per month
- **Default vs custom**: % of users who customize vs keep defaults

## Permissions

Reference: [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md)

### User Permissions

#### All Users
- **Manage own preferences**: Full control over personal communication settings
- **View communication history**: Access to own notification history
- **Pause notifications**: Temporarily pause for vacation/break
- **Export preferences**: Download preference configuration
- **Cannot**: Modify other users' preferences

#### Guardian (for students <13)
- **Manage student email**: Set guardian email for student communications
- **Opt in/out**: Control email and push notification consent for student
- **View communications**: See what communications student receives
- **Cannot**: Access student's in-app notification center without student login

### Administrator Permissions

#### Subscriber-Admin
- **Set organizational defaults**: Configure default preferences for new users in organization
- **View aggregate stats**: Communication delivery and engagement metrics for organization
- **Cannot**: View individual user preferences or communication history

#### System Admin
- **View all preferences**: Access to user preferences for support purposes
- **Modify user preferences**: Override for troubleshooting (with audit trail)
- **Configure system limits**: Set platform-wide rate limits and policies
- **Access communication logs**: Full delivery and engagement logs
- **Broadcast communications**: Send platform-wide announcements

### Privacy and Compliance

- **Communication content**: Encrypted in transit, stored securely at rest
- **Preference data**: Treated as PII, protected under GDPR/COPPA
- **Communication logs**: Retained per retention policy (90 days operational, 1 year archived)
- **Right to erasure**: User can request deletion of communication history
- **Data export**: User can export full communication history and preferences
- **COPPA**: Guardian consent required for email/push to students <13
- **FERPA**: Educational communications protected per role boundaries

## Supporting Documents Referenced

This communication framework specification draws from the following source documents:

- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform communication content and user-facing messaging about notifications
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — User interface text for notification settings and preferences
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role-specific communication requirements and preferences

## Dependencies

### Core Communication Channels

- **[In-app Notifications](./in-app-notifications.md)**: In-app delivery mechanism (notification center, toasts)
- **[Email Templates](./email-templates.md)**: Email delivery channel and template system
- **[Push and Desktop](./push-and-desktop.md)**: Push and desktop notification delivery
- **[System Announcements](./system-announcements.md)**: Admin-authored broadcast communications

### System Foundation

- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Event triggers for communications
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: User role definitions for default preferences
- **[Session and Auth](../02-roles-and-permissions/session-and-auth-states.md)**: User authentication for preference management

### Role-Specific Notification Specs

- **[Student Notifications](../03-student-experience/student-notifications.md)**: Student notification triggers and content
- **[Teacher Notifications](../04-teacher-experience/teacher-notifications.md)**: Teacher notification triggers and content
- **[Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md)**: Admin notification triggers and content

### User Experience

- **[Student Settings](../03-student-experience/student-settings.md)**: Student preference center interface
- **[Teacher Settings](../04-teacher-experience/teacher-settings.md)**: Teacher preference center interface
- **[Design System](../01-ux-design-system/components-overview.md)**: UI components for preference center
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)**: WCAG compliance for preference UI

### Technical Infrastructure

- **[Background Jobs](../18-architecture/background-jobs.md)**: Scheduled digest generation, quiet hours queue processing
- **[API Design](../18-architecture/api-design-principles.md)**: REST endpoints for preference CRUD operations
- **[Caching Strategy](../18-architecture/caching-strategy.md)**: Preference caching for performance

### Analytics and Reporting

- **[Admin Reports KPIs](../15-analytics-and-reporting/admin-reports-kpis.md)**: Communication health metrics in admin dashboards
- **[System Admin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md)**: Platform-wide communication monitoring

### Compliance

- **[Privacy Policy](../23-legal-and-compliance/privacy-policy.md)**: Privacy requirements for communication practices
- **[COPPA/FERPA Compliance](../23-legal-and-compliance/coppa-ferpa-notes.md)**: Student communication restrictions
- **[GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md)**: EU user communication requirements (if applicable)

## Open questions

### User Experience and Adoption

- **Default preferences**: Should new users start with all categories enabled or gradually opt-in? Research suggests starting with core categories enabled (assignments, messages) improves engagement vs. requiring explicit opt-in.
  - **Recommendation**: Enable essential categories by default (assignments, messages, security), keep optional categories (tips, newsletters) disabled
  - **Open**: Need to A/B test default-enabled vs opt-in for achievement/progress notifications

- **Preference complexity**: How granular should preferences be? Balance between control and simplicity.
  - Current design: Category-level with channel selection and frequency options
  - **Open**: Should we add per-notification-type controls (e.g., separate control for "assignment due" vs "assignment graded")?

- **Smart defaults**: Should system learn user preferences over time and suggest optimizations?
  - **Recommendation**: Phase 2 - track engagement patterns, suggest "You dismiss 80% of challenge notifications, want to disable?"
  - **Open**: What engagement threshold triggers suggested preference changes?

### Rate Limiting and Throttling

- **Limit strictness**: Are current rate limits too restrictive or too permissive?
  - Proposed: Email 10/day, Push 20/day
  - **Open**: Should limits vary by user role? Teachers may need higher limits than students.

- **Rate limit overrides**: Who can bypass rate limits and under what circumstances?
  - Currently: Critical priority bypasses all limits
  - **Open**: Should admins be able to set custom limits per organization?

- **Rate limit user visibility**: Should users see when they're approaching rate limits?
  - **Recommendation**: Show warning in preference center: "You're receiving 8 emails per day (limit: 10)"
  - **Open**: Do warnings reduce user satisfaction or increase control perception?

### Digest Implementation

- **Digest content quality**: How to ensure digest emails provide value and aren't just noise?
  - **Recommendation**: Use smart summarization, highlight most important items first
  - **Open**: Should digest include AI-generated summary of key points?

- **Digest timing optimization**: Fixed time or smart delivery based on user activity?
  - Currently: User-selected time (8 AM default)
  - **Open**: Phase 2 - learn optimal delivery time from user open patterns?

- **Mixed digest frequency**: Should users be able to set different digest frequencies per category?
  - Example: Daily digest for progress, weekly for tips and tricks
  - **Recommendation**: Phase 2 - start with single digest frequency, add per-category in future

### Quiet Hours

- **Quiet hours defaults**: Should we set age-appropriate default quiet hours?
  - Students: 9 PM - 7 AM (school schedule)
  - Teachers: 8 PM - 6 AM
  - **Open**: Regional differences? Different schedules for different countries?

- **Quiet hours vs. Do Not Disturb**: Should we integrate with device-level Do Not Disturb?
  - iOS/Android support DND modes
  - **Recommendation**: Phase 2 - respect device DND in addition to our quiet hours
  - **Open**: What if device DND conflicts with our quiet hours settings?

### Channel Coordination

- **Cross-channel redundancy**: When is redundancy helpful vs. annoying?
  - Example: Assignment due soon sent via in-app + push + email
  - **Recommendation**: Critical/high priority -> all channels, normal -> user preference
  - **Open**: Should we automatically reduce channels if user consistently ignores one?

- **Channel fallback delays**: How quickly to fallback if primary channel fails?
  - Currently: Immediate fallback for email failure
  - **Open**: Should we wait for push notification to be opened before sending email fallback?

### Privacy and Compliance

- **Communication data retention**: How long to keep communication delivery logs?
  - Proposed: 90 days operational, 1 year archived
  - **Open**: Does this meet audit requirements for educational records (FERPA)?

- **Guardian consent workflow**: Best way to obtain guardian consent for student communications?
  - **Recommendation**: Guardian account receives email requesting opt-in for student push/email
  - **Open**: What if guardian doesn't respond? Default to opt-out (conservative) or opt-in (better engagement)?

- **Right to explanation**: Should users be able to see why they received each communication?
  - **Recommendation**: Include "Why did I get this?" link in all communications
  - **Open**: How detailed should explanations be? Risk overwhelming users with technical details?

### Performance and Scale

- **Preference caching strategy**: How to cache preferences for performance without stale data?
  - **Recommendation**: Cache in Redis with 5-minute TTL, invalidate on update
  - **Open**: At what user scale do we need distributed cache?

- **Rate limit tracking**: Store in database or in-memory cache?
  - **Recommendation**: Redis for real-time tracking, sync to database periodically
  - **Open**: Risk of data loss if Redis crashes? Acceptable trade-off for performance?

- **Quiet hours processing**: How to efficiently process queued communications at quiet hours end?
  - Proposed: Batch process every minute, deliver max 20 per user at once
  - **Open**: What if 10,000 users' quiet hours end at 7 AM simultaneously?
