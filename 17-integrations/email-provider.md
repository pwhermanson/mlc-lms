# Email Provider

## Purpose

This specification defines the integration with third-party Email Service Providers (ESPs) for the MLC-LMS platform. While [Email Templates](../13-messaging-and-notifications/email-templates.md) handles template design and content rendering, this spec focuses exclusively on the delivery infrastructure: how emails are sent, tracked, and managed through external email delivery services.

The email provider integration serves as the critical bridge between our application's email generation system and end-user inboxes, handling the complex technical requirements of email deliverability, reputation management, bounce handling, and engagement tracking at scale.

**What this enables:**
- Reliable email delivery to all user roles (students, teachers, admins)
- Professional email deliverability with proper authentication (SPF, DKIM, DMARC)
- Real-time tracking of email lifecycle (sent, delivered, opened, clicked, bounced)
- Webhook-based status updates integrated into our event system
- Scalable email sending infrastructure without managing mail servers
- Compliance with email regulations (CAN-SPAM, GDPR)

**Why it exists:**
- Email delivery is a specialized technical domain requiring dedicated infrastructure
- Maintaining email reputation and deliverability requires expertise and monitoring
- Self-hosting mail servers is complex, expensive, and risky for reputation
- ESP integration provides professional-grade features (analytics, bounce handling, suppression lists)
- Allows platform to focus on core educational functionality rather than email infrastructure

## Scope

**Included**

- Email Service Provider selection criteria and recommendations
- API integration for sending individual and batch emails
- Webhook endpoint implementation for delivery status updates
- Domain configuration and authentication (SPF, DKIM, DMARC records)
- Sending domain strategy and reputation management
- Email sending queue and rate limiting implementation
- Bounce handling and suppression list management
- Provider-specific configuration and credentials management
- Email tracking setup (opens, clicks, unsubscribes)
- Testing and staging environment configuration
- Provider failover and redundancy strategy (future consideration)
- Cost monitoring and optimization strategies
- Provider API error handling and retry logic

**Excluded**

- Email template design and HTML rendering (see [Email Templates](../13-messaging-and-notifications/email-templates.md))
- Notification trigger logic and business rules (see role-specific notification docs)
- User notification preferences and subscription management (see [In-app Notifications](../13-messaging-and-notifications/in-app-notifications.md))
- Email content authoring and copywriting (see [Copy Guidelines](../22-content-style-guides/copy-guidelines.md))
- In-app notification system (see [In-app Notifications](../13-messaging-and-notifications/in-app-notifications.md))
- Legal compliance content requirements (see [Privacy Policy](../23-legal-and-compliance/privacy-policy.md) and [COPPA/FERPA](../23-legal-and-compliance/coppa-ferpa-notes.md))

## Data model (if applicable)

### EmailProviderConfig Entity

Stores ESP configuration and credentials (stored in secure configuration management system).

```typescript
interface EmailProviderConfig {
  // Provider identification
  providerId: string;                  // Unique identifier, e.g., "sendgrid_primary"
  providerName: 'sendgrid' | 'aws_ses' | 'mailgun' | 'postmark' | 'resend';
  environment: 'production' | 'staging' | 'development';
  
  // API credentials (stored encrypted)
  apiKey: string;                      // Provider API key
  apiSecret?: string;                  // Some providers require secret
  region?: string;                     // For AWS SES, e.g., "us-east-1"
  accountId?: string;                  // Provider account identifier
  
  // Sending configuration
  fromEmail: string;                   // Default sender address
  fromName: string;                    // Default sender name
  replyToEmail?: string;               // Reply-to address
  sendingDomain: string;               // Domain for sending, e.g., "notifications.mlc.example.com"
  
  // Rate limiting
  maxSendRate: number;                 // Max emails per second
  dailySendLimit?: number;             // Max emails per day
  burstLimit?: number;                 // Max emails in burst
  
  // Features enabled
  enableOpenTracking: boolean;
  enableClickTracking: boolean;
  enableUnsubscribeTracking: boolean;
  
  // Webhook configuration
  webhookUrl: string;                  // URL for provider to send events
  webhookSecret: string;               // Secret for webhook signature verification
  
  // Status
  isActive: boolean;                   // Whether this config is currently in use
  isPrimary: boolean;                  // Primary provider for this environment
  
  // Audit
  createdAt: Date;
  updatedAt: Date;
  lastTestedAt?: Date;
  testStatus?: 'success' | 'failed';
}
```

### EmailSendRequest Entity

Represents a request to send an email through the provider.

```typescript
interface EmailSendRequest {
  // Identification
  requestId: string;                   // Unique request ID, e.g., "req_abc123"
  templateId: string;                  // Reference to email template
  renderingId: string;                 // Reference to rendered email content
  
  // Recipient
  recipientEmail: string;              // Recipient email address
  recipientName?: string;              // Recipient name for personalization
  recipientId: string;                 // User ID in our system
  
  // Sender
  fromEmail: string;                   // Sender email address
  fromName: string;                    // Sender name
  replyToEmail?: string;               // Reply-to address override
  
  // Content
  subject: string;                     // Email subject
  htmlBody: string;                    // Rendered HTML content
  textBody: string;                    // Plain text version
  
  // Metadata
  emailType: 'transactional' | 'promotional' | 'educational';
  priority: 'high' | 'normal' | 'low';
  tags: string[];                      // Tags for categorization
  
  // Tracking
  customVariables?: Record<string, string>; // Provider-specific variables
  utmParameters?: UTMParameters;
  
  // Provider info
  providerId: string;                  // Which provider config to use
  
  // Status
  status: 'pending' | 'queued' | 'sent' | 'failed' | 'cancelled';
  queuedAt?: Date;
  sentAt?: Date;
  failedAt?: Date;
  
  // Results
  providerMessageId?: string;          // Provider's message ID
  providerResponse?: any;              // Raw provider response
  error?: string;                      // Error message if failed
  retryCount: number;                  // Number of retry attempts
  nextRetryAt?: Date;
}
```

### EmailDeliveryEvent Entity

Tracks email lifecycle events received via webhooks.

```typescript
interface EmailDeliveryEvent {
  // Identification
  eventId: string;                     // Unique event ID
  providerMessageId: string;           // Provider's message ID
  requestId?: string;                  // Our internal request ID (if matched)
  recipientEmail: string;              // Email address (hashed for privacy)
  
  // Event details
  eventType: EmailEventType;
  eventTimestamp: Date;                // When event occurred (provider time)
  receivedAt: Date;                    // When we received the webhook
  
  // Event-specific data
  eventData: {
    // For bounced
    bounceType?: 'hard' | 'soft' | 'undetermined';
    bounceReason?: string;
    smtpCode?: string;
    
    // For opened
    userAgent?: string;
    ipAddress?: string;
    deviceType?: 'desktop' | 'mobile' | 'tablet';
    
    // For clicked
    clickedUrl?: string;
    linkText?: string;
    
    // For spam complaint
    complaintSource?: string;
    
    // For unsubscribed
    unsubscribeMethod?: 'link' | 'list_unsubscribe' | 'marked_as_spam';
  };
  
  // Provider info
  providerId: string;
  rawWebhookPayload: any;              // Full webhook payload for debugging
  
  // Processing
  processed: boolean;
  processedAt?: Date;
  processingError?: string;
}

type EmailEventType = 
  | 'sent'           // Email handed to provider
  | 'delivered'      // Successfully delivered to recipient
  | 'bounced'        // Delivery failed
  | 'deferred'       // Temporarily delayed
  | 'opened'         // Email opened by recipient
  | 'clicked'        // Link clicked in email
  | 'unsubscribed'   // Recipient unsubscribed
  | 'spam_complaint' // Marked as spam
  | 'dropped';       // Dropped by provider (suppression list, etc.)
```

### EmailSuppressionList Entity

Manages list of email addresses that should not receive emails.

```typescript
interface EmailSuppressionList {
  // Identification
  suppressionId: string;
  emailAddress: string;                // Hashed email address
  emailAddressHash: string;            // SHA-256 hash for lookup
  
  // Reason
  suppressionType: 'bounce' | 'complaint' | 'unsubscribe' | 'manual';
  reason: string;                      // Detailed reason
  
  // Scope
  scope: 'global' | 'promotional' | 'educational'; // What to suppress
  
  // Metadata
  addedBy: 'system' | 'user' | 'admin';
  providerId?: string;                 // If provider-specific
  
  // Timestamps
  addedAt: Date;
  expiresAt?: Date;                    // Some suppressions may be temporary
  
  // Bounce-specific
  bounceCount?: number;
  lastBounceAt?: Date;
  
  // Status
  isActive: boolean;
}
```

### EmailSendingQueue Entity

Queue management for batch sends and rate limiting.

```typescript
interface EmailSendingQueue {
  queueId: string;
  batchId?: string;                    // If part of batch send
  
  // Request reference
  requestId: string;
  priority: 'high' | 'normal' | 'low';
  
  // Scheduling
  scheduledFor: Date;                  // When to send
  quietHoursCheck: boolean;            // Respect user quiet hours
  
  // Status
  status: 'pending' | 'processing' | 'completed' | 'failed' | 'cancelled';
  attempts: number;
  lastAttemptAt?: Date;
  nextAttemptAt?: Date;
  
  // Rate limiting
  rateLimitBucket?: string;            // For rate limiting grouping
  
  createdAt: Date;
  updatedAt: Date;
}
```

## Behavior and rules

### Provider Selection and Configuration

#### Recommended Providers

**Primary Recommendation: SendGrid**
- **Pros**: 
  - Comprehensive API and excellent documentation
  - Strong deliverability reputation
  - Robust webhook system
  - Good free tier (100 emails/day, sufficient for development)
  - Email validation API included
  - Built-in template management (though we'll use our own)
  - Marketing automation features for future use
- **Cons**: 
  - Can be more expensive at high volume compared to AWS SES
  - Some advanced features require higher tiers
- **Best for**: Production launch, comprehensive features out of the box

**Alternative: AWS SES (Amazon Simple Email Service)**
- **Pros**: 
  - Very cost-effective at scale ($0.10 per 1,000 emails)
  - Highly scalable infrastructure
  - Integration with AWS ecosystem
  - Dedicated IP options
- **Cons**: 
  - More complex setup and configuration
  - Requires AWS account management
  - Less user-friendly than SendGrid
  - Webhook implementation more complex (requires SNS/SQS)
- **Best for**: High-volume sending when cost is primary concern

**Alternative: Postmark**
- **Pros**: 
  - Focus on transactional email reliability
  - Excellent deliverability rates
  - Simple, clean API
  - Great documentation
- **Cons**: 
  - More expensive than competitors ($15/month for 10K emails)
  - Limited marketing email features
- **Best for**: Mission-critical transactional emails requiring highest reliability

**Alternative: Resend**
- **Pros**: 
  - Modern developer experience
  - React email support built-in
  - Simple pricing ($20/month for 50K emails)
  - Excellent API design
- **Cons**: 
  - Newer provider (less track record)
  - Smaller feature set than established competitors
- **Best for**: Developer-focused teams wanting modern tooling

**Not Recommended:**
- **Mailgun**: API changes and reliability issues in recent years
- **SparkPost**: Declining market position
- **Self-hosted SMTP**: Too complex and risky for deliverability

#### Configuration Strategy

**Development Environment:**
```typescript
{
  providerName: 'sendgrid',
  environment: 'development',
  sendingDomain: 'dev.mlc.example.com',
  fromEmail: 'dev@dev.mlc.example.com',
  enableOpenTracking: false,         // Disable tracking in dev
  enableClickTracking: false,
  maxSendRate: 1,                    // Slow rate for dev
}
```

**Staging Environment:**
```typescript
{
  providerName: 'sendgrid',
  environment: 'staging',
  sendingDomain: 'staging.mlc.example.com',
  fromEmail: 'noreply@staging.mlc.example.com',
  enableOpenTracking: true,
  enableClickTracking: true,
  maxSendRate: 10,
  dailySendLimit: 1000,              // Prevent accidental spam
}
```

**Production Environment:**
```typescript
{
  providerName: 'sendgrid',
  environment: 'production',
  sendingDomain: 'notifications.mlc.example.com',
  fromEmail: 'noreply@musiclearningcommunity.com',
  replyToEmail: 'support@musiclearningcommunity.com',
  enableOpenTracking: true,
  enableClickTracking: true,
  maxSendRate: 100,                  // Start conservative, increase as reputation builds
  dailySendLimit: 50000,
}
```

### Domain Configuration and Authentication

#### DNS Records Setup

Email authentication requires several DNS records to be configured on the sending domain.

**SPF (Sender Policy Framework) Record:**
```dns
Type: TXT
Host: notifications.mlc.example.com
Value: v=spf1 include:sendgrid.net ~all
TTL: 3600
```

**Purpose**: Specifies which mail servers are authorized to send email on behalf of the domain.

**DKIM (DomainKeys Identified Mail) Record:**
```dns
Type: CNAME
Host: em1234._domainkey.notifications.mlc.example.com
Value: em1234.dkim.sendgrid.net
TTL: 3600

Type: CNAME
Host: s1._domainkey.notifications.mlc.example.com
Value: s1.domainkey.u12345678.wl123.sendgrid.net
TTL: 3600
```

**Purpose**: Allows email providers to verify that emails were not tampered with in transit.

**DMARC (Domain-based Message Authentication) Record:**
```dns
Type: TXT
Host: _dmarc.notifications.mlc.example.com
Value: v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@musiclearningcommunity.com; pct=100; adkim=s; aspf=s
TTL: 3600
```

**Purpose**: Tells receiving mail servers what to do if SPF or DKIM checks fail. Provides reporting on email authentication results.

**DMARC Policy Progression:**
- **Phase 1 (Launch)**: `p=none` - Monitor only, don't reject
- **Phase 2 (After 2 weeks)**: `p=quarantine` - Send suspicious emails to spam
- **Phase 3 (After 1 month)**: `p=reject` - Reject unauthenticated emails

**Return-Path (Bounce) Domain:**
```dns
Type: CNAME
Host: bounce.notifications.mlc.example.com
Value: sendgrid.net
TTL: 3600
```

**Purpose**: Handles bounce emails separately from main domain to protect reputation.

#### Domain Reputation Management

**Sending Domain Strategy:**
- **Transactional emails**: `notifications.mlc.example.com`
- **Promotional/Marketing**: `news.mlc.example.com` (future, if needed)
- **System alerts**: `alerts.mlc.example.com` (future, if high volume)

**Rationale**: Separating domains isolates reputation issues. If promotional emails get marked as spam, it doesn't affect critical transactional email deliverability.

**IP Address Strategy:**

**Phase 1: Shared IP (Launch - 3 months)**
- Use provider's shared IP pool
- Benefits from provider's established reputation
- No IP warm-up required
- Cost-effective for low volume

**Phase 2: Dedicated IP (When sending >100K emails/month)**
- Requires IP warm-up process (2-4 weeks)
- Full control over sender reputation
- Better deliverability for high-volume senders
- Requires ongoing reputation monitoring

**IP Warm-up Schedule (if transitioning to dedicated IP):**
```
Day 1:      50 emails
Day 2:      100 emails
Day 3:      500 emails
Day 4:      1,000 emails
Week 2:     5,000 emails/day
Week 3:     10,000 emails/day
Week 4+:    Full volume
```

### API Integration Patterns

#### Sending Individual Emails

**Endpoint**: Provider's send API (e.g., SendGrid `/v3/mail/send`)

**Request Flow:**
```typescript
async function sendEmail(request: EmailSendRequest): Promise<EmailSendResponse> {
  // 1. Validate request
  if (!isValidEmail(request.recipientEmail)) {
    throw new Error('Invalid recipient email');
  }
  
  // 2. Check suppression list
  if (await isEmailSuppressed(request.recipientEmail)) {
    return {
      status: 'suppressed',
      reason: 'Recipient on suppression list'
    };
  }
  
  // 3. Rate limit check
  await rateLimiter.checkLimit(request.providerId);
  
  // 4. Build provider payload
  const payload = buildProviderPayload(request);
  
  // 5. Send to provider
  try {
    const response = await providerClient.send(payload);
    
    // 6. Record send request
    await recordSendRequest({
      ...request,
      status: 'sent',
      providerMessageId: response.messageId,
      sentAt: new Date()
    });
    
    // 7. Emit event
    await emitEvent({
      event_type: 'email_sent',
      properties: {
        request_id: request.requestId,
        provider_message_id: response.messageId,
        recipient_id: request.recipientId
      }
    });
    
    return {
      status: 'sent',
      messageId: response.messageId
    };
    
  } catch (error) {
    // Handle provider errors
    return handleProviderError(error, request);
  }
}
```

**SendGrid API Example:**
```typescript
const sendgridPayload = {
  personalizations: [{
    to: [{
      email: request.recipientEmail,
      name: request.recipientName
    }],
    custom_args: {
      request_id: request.requestId,
      recipient_id: request.recipientId,
      template_id: request.templateId
    }
  }],
  from: {
    email: request.fromEmail,
    name: request.fromName
  },
  reply_to: {
    email: request.replyToEmail || 'support@musiclearningcommunity.com'
  },
  subject: request.subject,
  content: [
    {
      type: 'text/plain',
      value: request.textBody
    },
    {
      type: 'text/html',
      value: request.htmlBody
    }
  ],
  tracking_settings: {
    click_tracking: {
      enable: true,
      enable_text: false
    },
    open_tracking: {
      enable: true,
      substitution_tag: '%open_track%'
    },
    subscription_tracking: {
      enable: true,
      substitution_tag: '%unsubscribe_url%'
    }
  },
  mail_settings: {
    bypass_list_management: {
      enable: request.emailType === 'transactional'
    }
  },
  categories: [
    request.emailType,
    request.templateId.split('_')[1] // Extract category from template ID
  ]
};
```

#### Batch Email Sending

For sending emails to multiple recipients (e.g., weekly digest, broadcast announcements).

**Strategy**: Queue-based processing with rate limiting

```typescript
async function sendEmailBatch(
  batchId: string,
  requests: EmailSendRequest[]
): Promise<BatchSendResult> {
  
  // 1. Add to sending queue
  for (const request of requests) {
    await emailQueue.add({
      queueId: generateId(),
      batchId,
      requestId: request.requestId,
      priority: request.priority,
      scheduledFor: request.scheduledFor || new Date(),
      status: 'pending'
    });
  }
  
  // 2. Background worker processes queue
  // (separate process handles actual sending with rate limiting)
  
  return {
    batchId,
    totalRequests: requests.length,
    status: 'queued'
  };
}
```

**Background Worker Implementation:**
```typescript
// Queue worker (runs continuously)
async function processEmailQueue() {
  while (true) {
    // Get next batch respecting rate limits
    const items = await emailQueue.getNext({
      limit: config.maxSendRate,
      scheduledBefore: new Date()
    });
    
    if (items.length === 0) {
      await sleep(1000); // Wait before checking again
      continue;
    }
    
    // Send emails in parallel (within rate limit)
    const results = await Promise.allSettled(
      items.map(async (item) => {
        const request = await getEmailSendRequest(item.requestId);
        return sendEmail(request);
      })
    );
    
    // Update queue items
    await updateQueueItems(items, results);
    
    // Rate limit: ensure we don't exceed provider limits
    await sleep(1000); // 1 second between batches
  }
}
```

### Webhook Implementation

#### Webhook Endpoint Security

**URL**: `https://api.mlc.example.com/webhooks/email/[provider]`

Example: `https://api.mlc.example.com/webhooks/email/sendgrid`

**Security Requirements:**
1. **Signature Verification**: Verify webhook is from legitimate provider
2. **HTTPS Only**: Never accept webhooks over plain HTTP
3. **Rate Limiting**: Prevent webhook flooding attacks
4. **Idempotency**: Handle duplicate webhook deliveries gracefully

**SendGrid Signature Verification:**
```typescript
function verifyWebhookSignature(
  payload: string,
  signature: string,
  timestamp: string,
  secret: string
): boolean {
  // SendGrid uses HMAC SHA256
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(timestamp + payload)
    .digest('base64');
    
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

**Webhook Handler Implementation:**
```typescript
export async function POST(request: Request) {
  // 1. Extract headers and body
  const signature = request.headers.get('x-twilio-email-event-webhook-signature');
  const timestamp = request.headers.get('x-twilio-email-event-webhook-timestamp');
  const body = await request.text();
  
  // 2. Verify signature
  if (!verifyWebhookSignature(body, signature, timestamp, config.webhookSecret)) {
    return new Response('Invalid signature', { status: 401 });
  }
  
  // 3. Check timestamp (prevent replay attacks)
  const now = Math.floor(Date.now() / 1000);
  if (Math.abs(now - parseInt(timestamp)) > 300) { // 5 minutes
    return new Response('Timestamp too old', { status: 401 });
  }
  
  // 4. Parse events
  const events = JSON.parse(body);
  
  // 5. Process events asynchronously
  // Return 200 immediately to provider, process in background
  await queueWebhookEvents(events);
  
  return new Response('OK', { status: 200 });
}
```

#### Webhook Event Processing

Events should be processed asynchronously to avoid blocking the webhook endpoint.

```typescript
async function processWebhookEvent(event: any) {
  try {
    // 1. Normalize event (provider-specific format → our format)
    const normalizedEvent = normalizeProviderEvent(event);
    
    // 2. Store raw event
    const deliveryEvent = await storeDeliveryEvent(normalizedEvent);
    
    // 3. Update email send request status
    await updateSendRequestStatus(
      normalizedEvent.providerMessageId,
      normalizedEvent.eventType
    );
    
    // 4. Handle specific event types
    switch (normalizedEvent.eventType) {
      case 'bounced':
        await handleBounce(normalizedEvent);
        break;
      case 'spam_complaint':
        await handleSpamComplaint(normalizedEvent);
        break;
      case 'unsubscribed':
        await handleUnsubscribe(normalizedEvent);
        break;
      case 'delivered':
        await handleDelivered(normalizedEvent);
        break;
      case 'opened':
      case 'clicked':
        await handleEngagement(normalizedEvent);
        break;
    }
    
    // 5. Emit platform event
    await emitPlatformEvent(normalizedEvent);
    
    // 6. Mark as processed
    await markEventProcessed(deliveryEvent.eventId);
    
  } catch (error) {
    // Log error but don't throw (webhook endpoint already returned 200)
    logger.error('Webhook event processing failed', { error, event });
  }
}
```

### Bounce Handling

**Hard Bounce Processing:**
```typescript
async function handleHardBounce(event: EmailDeliveryEvent) {
  const emailAddress = event.recipientEmail;
  
  // 1. Add to suppression list
  await addToSuppressionList({
    emailAddress,
    suppressionType: 'bounce',
    reason: event.eventData.bounceReason || 'Hard bounce',
    scope: 'global', // Don't send any emails to this address
    addedBy: 'system'
  });
  
  // 2. Update user record
  await markUserEmailInvalid(emailAddress);
  
  // 3. Notify admin if guardian email (for students)
  const user = await getUserByEmail(emailAddress);
  if (user?.role === 'student' && user.guardianEmail === emailAddress) {
    await notifyAdminInvalidGuardianEmail(user);
  }
  
  // 4. Log for monitoring
  logger.warn('Hard bounce recorded', {
    email: hashEmail(emailAddress),
    reason: event.eventData.bounceReason
  });
}
```

**Soft Bounce Processing:**
```typescript
async function handleSoftBounce(event: EmailDeliveryEvent) {
  const emailAddress = event.recipientEmail;
  
  // 1. Increment bounce counter
  const bounceCount = await incrementBounceCount(emailAddress);
  
  // 2. If soft bounces exceed threshold, treat as hard bounce
  if (bounceCount >= 3) {
    return handleHardBounce(event);
  }
  
  // 3. Schedule retry (provider usually handles this automatically)
  logger.info('Soft bounce recorded', {
    email: hashEmail(emailAddress),
    bounceCount,
    reason: event.eventData.bounceReason
  });
}
```

### Spam Complaint Handling

```typescript
async function handleSpamComplaint(event: EmailDeliveryEvent) {
  const emailAddress = event.recipientEmail;
  
  // 1. Immediately add to suppression list
  await addToSuppressionList({
    emailAddress,
    suppressionType: 'complaint',
    reason: `Spam complaint from ${event.eventData.complaintSource}`,
    scope: 'global', // Suppress all email types
    addedBy: 'system'
  });
  
  // 2. Unsubscribe user from all promotional emails
  const user = await getUserByEmail(emailAddress);
  if (user) {
    await updateUserEmailPreferences(user.id, {
      promotional: false,
      educational: false,
      administrative: true // Keep only critical emails
    });
  }
  
  // 3. Alert system admin (spam complaints hurt reputation)
  await alertSystemAdmin({
    type: 'spam_complaint',
    email: hashEmail(emailAddress),
    templateId: event.eventData.templateId,
    source: event.eventData.complaintSource
  });
  
  // 4. Track spam complaint rate
  await trackMetric('email_spam_complaint_rate', {
    provider: event.providerId,
    template: event.eventData.templateId
  });
  
  // Critical: If spam complaint rate exceeds 0.1%, investigate immediately
  const complaintRate = await getSpamComplaintRate();
  if (complaintRate > 0.001) { // 0.1%
    await triggerIncident({
      severity: 'high',
      message: `Email spam complaint rate ${complaintRate}% exceeds threshold`
    });
  }
}
```

### Rate Limiting Implementation

**Token Bucket Algorithm:**
```typescript
class EmailRateLimiter {
  private buckets: Map<string, RateLimitBucket> = new Map();
  
  async checkLimit(providerId: string): Promise<void> {
    const config = await getProviderConfig(providerId);
    const bucket = this.getBucket(providerId, config.maxSendRate);
    
    // Wait until token available
    await bucket.consume(1);
  }
  
  private getBucket(providerId: string, rate: number): RateLimitBucket {
    if (!this.buckets.has(providerId)) {
      this.buckets.set(providerId, new RateLimitBucket({
        capacity: rate,
        refillRate: rate, // Refill rate per second
        refillInterval: 1000 // 1 second
      }));
    }
    return this.buckets.get(providerId)!;
  }
}
```

**User-Level Rate Limiting:**
```typescript
async function canSendToUser(
  userId: string,
  emailType: string
): Promise<{ allowed: boolean; reason?: string }> {
  
  // Transactional emails: no limit
  if (emailType === 'transactional') {
    return { allowed: true };
  }
  
  // Check daily limit per user
  const today = new Date().toISOString().split('T')[0];
  const key = `email_count:${userId}:${today}`;
  const count = await redis.get(key) || 0;
  
  const maxEmailsPerDay = 5; // Prevent email fatigue
  
  if (count >= maxEmailsPerDay) {
    return {
      allowed: false,
      reason: 'Daily email limit exceeded for user'
    };
  }
  
  // Increment counter
  await redis.incr(key);
  await redis.expire(key, 86400); // Expire after 24 hours
  
  return { allowed: true };
}
```

### Error Handling and Retry Logic

**Provider Error Handling:**
```typescript
async function handleProviderError(
  error: any,
  request: EmailSendRequest
): Promise<EmailSendResponse> {
  
  // Categorize error
  const errorType = categorizeProviderError(error);
  
  switch (errorType) {
    case 'rate_limit':
      // Rate limit exceeded - back off and retry
      return {
        status: 'rate_limited',
        retryAfter: error.retryAfter || 60,
        shouldRetry: true
      };
      
    case 'invalid_request':
      // Bad request - don't retry
      logger.error('Invalid email request', {
        requestId: request.requestId,
        error: error.message
      });
      return {
        status: 'failed',
        error: error.message,
        shouldRetry: false
      };
      
    case 'authentication':
      // Auth error - alert admin immediately
      await alertSystemAdmin({
        type: 'email_auth_error',
        provider: request.providerId,
        error: error.message
      });
      return {
        status: 'failed',
        error: 'Provider authentication failed',
        shouldRetry: false
      };
      
    case 'server_error':
      // Provider server error - retry with backoff
      return {
        status: 'server_error',
        error: error.message,
        shouldRetry: true,
        retryAfter: 30
      };
      
    default:
      // Unknown error - log and retry
      logger.error('Unknown provider error', {
        requestId: request.requestId,
        error
      });
      return {
        status: 'failed',
        error: 'Unknown error',
        shouldRetry: true
      };
  }
}
```

**Retry Strategy:**
```typescript
const retryPolicy = {
  maxRetries: 3,
  backoff: 'exponential', // 1min, 2min, 4min
  retryableErrors: [
    'rate_limit',
    'server_error',
    'timeout',
    'network_error'
  ]
};

async function retryEmailSend(request: EmailSendRequest): Promise<void> {
  const attempt = request.retryCount + 1;
  
  if (attempt > retryPolicy.maxRetries) {
    // Max retries exceeded
    await recordFailedSend(request, 'Max retries exceeded');
    return;
  }
  
  // Calculate backoff delay
  const delayMs = Math.pow(2, attempt) * 60 * 1000; // Exponential: 2^n minutes
  const nextRetryAt = new Date(Date.now() + delayMs);
  
  // Update request
  await updateSendRequest(request.requestId, {
    retryCount: attempt,
    nextRetryAt,
    status: 'pending'
  });
  
  // Re-queue
  await emailQueue.add({
    requestId: request.requestId,
    scheduledFor: nextRetryAt,
    priority: request.priority
  });
}
```

## UX requirements (if applicable)

This integration is backend/infrastructure focused with no direct user-facing UI. However, there are admin-facing monitoring and configuration interfaces.

### System Admin: Email Provider Configuration

**Location**: System Admin → Settings → Email Provider

**Interface Requirements:**

**Provider Configuration Form:**
```
┌─────────────────────────────────────────┐
│ Email Provider Configuration            │
├─────────────────────────────────────────┤
│                                         │
│ Provider:                               │
│ [SendGrid ▼]                            │
│                                         │
│ Environment:                            │
│ [Production ▼]                          │
│                                         │
│ API Key: [••••••••••••••••] [Reveal]    │
│                                         │
│ Sending Domain:                         │
│ [notifications.mlc.example.com]         │
│                                         │
│ From Email:                             │
│ [noreply@musiclearningcommunity.com]    │
│                                         │
│ From Name:                              │
│ [MusicLearningCommunity]                │
│                                         │
│ Reply-To Email:                         │
│ [support@musiclearningcommunity.com]    │
│                                         │
│ ☑ Enable Open Tracking                 │
│ ☑ Enable Click Tracking                │
│                                         │
│ Rate Limit: [100] emails/second         │
│                                         │
│ [Test Configuration]  [Save]            │
└─────────────────────────────────────────┘
```

**DNS Configuration Checklist:**
```
┌─────────────────────────────────────────┐
│ DNS Configuration Status                │
├─────────────────────────────────────────┤
│ ✅ SPF Record      Verified             │
│ ✅ DKIM Record     Verified             │
│ ⚠️  DMARC Record   Policy: none         │
│                    Recommended: quarantine│
│ ✅ Return-Path     Configured           │
│                                         │
│ [Check DNS Records]  [View Instructions]│
└─────────────────────────────────────────┘
```

### System Admin: Email Delivery Monitoring

**Location**: System Admin → Monitoring → Email Delivery

**Dashboard Requirements:**

**Real-time Metrics (Last 24 hours):**
```
┌──────────┬──────────┬──────────┬──────────┐
│  Sent    │Delivered │  Opened  │ Clicked  │
│  1,247   │  1,235   │   687    │   234    │
│  100%    │  99.0%   │  55.6%   │  18.9%   │
└──────────┴──────────┴──────────┴──────────┘

┌──────────┬──────────┬──────────┬──────────┐
│ Bounced  │ Deferred │Complaints│Suppressed│
│    12    │    5     │    1     │    3     │
│  0.96%   │  0.40%   │  0.08%   │  0.24%   │
└──────────┴──────────┴──────────┴──────────┘
```

**Alerts:**
- 🔴 **Critical**: Spam complaint rate >0.1% (requires immediate action)
- ⚠️ **Warning**: Bounce rate >2% (investigate email quality)
- ⚠️ **Warning**: Delivery rate <98% (check provider status)

**Provider Status:**
```
Provider: SendGrid
Status: ● Operational
Last Successful Send: 2 minutes ago
API Latency: 145ms (avg)
Reputation Score: 98/100 ✅
```

### System Admin: Suppression List Management

**Location**: System Admin → Email → Suppression List

**Interface:**
```
┌─────────────────────────────────────────┐
│ Email Suppression List                  │
├─────────────────────────────────────────┤
│ Search: [_________________] [🔍]        │
│                                         │
│ Total Suppressed: 47 addresses         │
│ - Bounces: 32                           │
│ - Complaints: 8                         │
│ - Unsubscribes: 7                       │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ Email (hashed) │ Type │ Date │Action││
│ ├─────────────────────────────────────┤ │
│ │ 7a8f9e...       │Bounce│1/15  │ ⊗  ││
│ │ 3b4c2d...       │Compl │1/14  │ ⊗  ││
│ │ 9e1f0a...       │Unsub │1/13  │ ⊗  ││
│ └─────────────────────────────────────┘ │
│                                         │
│ [Export List]  [Add Manually]           │
└─────────────────────────────────────────┘
```

**Privacy Note**: Email addresses are hashed (SHA-256) in the UI for privacy. Full email visible only when necessary and logged for audit.

## Telemetry

Reference: [Event Model](../15-analytics-and-reporting/event-model.md) for complete event tracking specifications.

### Provider Integration Events

#### Email Provider API Call
```javascript
{
  event_type: "email_provider_api_call",
  properties: {
    provider: "sendgrid",
    endpoint: "/v3/mail/send",
    request_id: "req_abc123",
    http_status: 202,
    latency_ms: 145,
    success: true
  }
}
// Fires: Server-side on every provider API call
```

#### Email Provider Error
```javascript
{
  event_type: "email_provider_error",
  properties: {
    provider: "sendgrid",
    error_type: "rate_limit" | "authentication" | "server_error" | "invalid_request",
    error_message: "Rate limit exceeded",
    request_id: "req_abc123",
    http_status: 429,
    retry_scheduled: true,
    retry_after_seconds: 60
  }
}
// Fires: Server-side when provider API returns error
```

#### Webhook Received
```javascript
{
  event_type: "email_webhook_received",
  properties: {
    provider: "sendgrid",
    event_count: 15,
    signature_valid: true,
    processing_time_ms: 23
  }
}
// Fires: When webhook endpoint receives events from provider
```

### Delivery Lifecycle Events

All delivery events are defined in [Email Templates](../13-messaging-and-notifications/email-templates.md) telemetry section and include:
- `email_sent`
- `email_delivered`
- `email_opened`
- `email_clicked`
- `email_bounced`
- `email_spam_complaint`
- `email_unsubscribed`

These events include `provider_message_id` to correlate with provider systems.

### Suppression List Events

#### Email Address Suppressed
```javascript
{
  event_type: "email_address_suppressed",
  properties: {
    email_hash: "7a8f9e...", // SHA-256 hash
    suppression_type: "bounce" | "complaint" | "unsubscribe" | "manual",
    reason: "Hard bounce - invalid email",
    scope: "global" | "promotional",
    added_by: "system" | "user" | "admin"
  }
}
// Fires: When email added to suppression list
```

#### Email Address Removed from Suppression
```javascript
{
  event_type: "email_address_unsuppressed",
  properties: {
    email_hash: "7a8f9e...",
    suppression_type: "bounce",
    removed_by: "admin",
    reason: "Manual review - false positive"
  }
}
// Fires: When email removed from suppression list (rare)
```

### Monitoring Metrics

**Provider Health Metrics:**
- **API availability**: Percentage of successful API calls (target: >99.9%)
- **API latency**: P50, P95, P99 response times
- **Error rate**: Percentage of API calls resulting in errors
- **Rate limit hits**: Number of times rate limit exceeded

**Deliverability Metrics:**
- **Delivery rate**: delivered / sent (target: >99%)
- **Bounce rate**: bounced / sent (target: <2%)
  - Hard bounce rate (target: <1%)
  - Soft bounce rate (target: <1%)
- **Spam complaint rate**: complaints / delivered (target: <0.1%, critical threshold)
- **Engagement rates**: See [Email Templates](../13-messaging-and-notifications/email-templates.md) telemetry

**Queue Metrics:**
- **Queue depth**: Number of emails waiting to send
- **Average wait time**: Time from queue to send
- **Processing rate**: Emails processed per second
- **Failed sends**: Emails that failed after max retries

## Permissions

Reference: [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) for role definitions.

### Email Provider Configuration

**System Admin:**
- **Configure providers**: Add/edit provider configurations
- **Manage credentials**: Update API keys and secrets
- **Configure domains**: Set up sending domains and DNS
- **View all metrics**: Access complete delivery and engagement metrics
- **Manage suppression list**: Add/remove addresses manually
- **Test configuration**: Send test emails
- **Access webhook logs**: View raw webhook data
- **Export data**: Export delivery logs and metrics

**Subscriber Admin:**
- **No access**: Cannot view or configure email provider settings

**Teacher / Student:**
- **No access**: No visibility into email infrastructure

### Operational Access

**System Admin:**
- **Trigger manual sends**: Send individual emails for testing
- **Retry failed sends**: Manually retry emails that failed
- **View delivery logs**: Access detailed send/delivery logs
- **Monitor queues**: View sending queue status
- **Access provider dashboard**: Direct access to ESP dashboard (if needed)

**Automated System:**
- **Send emails**: Send emails via API based on triggers
- **Process webhooks**: Handle incoming webhook events
- **Update suppression list**: Automatically add bounces/complaints
- **Retry logic**: Automatically retry failed sends

### Data Access and Privacy

**Email Address Visibility:**
- **System Admin**: Can view email addresses (logged for audit)
- **Subscriber Admin**: Can view email addresses in their organization only
- **Teacher**: Can view student email addresses in their classes only
- **Student**: Cannot view other users' email addresses

**Webhook Data:**
- Contains email addresses in plain text (from provider)
- Must be handled securely and logged appropriately
- Email addresses hashed in analytics storage (after 90 days per GDPR)

**Retention Policies:**
- **Email content**: 90 days (then archived)
- **Delivery logs**: 2 years (compliance requirement)
- **Webhook raw data**: 30 days (debugging)
- **Suppression list**: Indefinite (until manually removed or user requests deletion)

## Supporting Documents Referenced

This email provider integration specification draws from the following source documents:

- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Email content and messaging templates
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Email notification text and user messaging
- [AMessagetoParents.pdf](../00-ORIG-CONTEXT/AMessagetoParents.pdf) — Parent communication email examples

## Dependencies

### Core System Dependencies

- **[Email Templates](../13-messaging-and-notifications/email-templates.md)**: Provides rendered HTML/text content for sending
- **[In-app Notifications](../13-messaging-and-notifications/in-app-notifications.md)**: Trigger events that initiate email sends
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Event tracking for delivery lifecycle
- **[Background Jobs](../18-architecture/background-jobs.md)**: Queue processing and retry logic

### Infrastructure Dependencies

- **[Webhooks](../18-architecture/webhooks.md)**: Webhook endpoint implementation patterns
- **[Config and Secrets](../18-architecture/config-and-secrets.md)**: Secure storage of API keys
- **[API Design](../18-architecture/api-design-principles.md)**: Internal API for email sending
- **[Security and Privacy](../18-architecture/security-privacy.md)**: Authentication, encryption, data protection

### Monitoring and Operations

- **[Observability](../19-quality-and-operations/observability.md)**: Logging, metrics, alerting infrastructure
- **[Incident Response](../19-quality-and-operations/incident-response.md)**: Handling email delivery incidents

### Compliance and Legal

- **[Privacy Policy](../23-legal-and-compliance/privacy-policy.md)**: Email data handling policies
- **[GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md)**: Right to erasure, data export
- **[COPPA/FERPA](../23-legal-and-compliance/coppa-ferpa-notes.md)**: Student email restrictions

## Open questions

### Provider Selection

- **Final ESP choice**: Have we committed to SendGrid, or should we evaluate others? **Recommendation**: Start with SendGrid for ease of integration, evaluate AWS SES after 6 months if cost becomes concern.

- **Failover strategy**: Should we configure a backup ESP for redundancy? **Recommendation**: Phase 2 - add backup provider if sending >50K emails/month.

- **Multi-region sending**: Do we need regional ESPs for international users? **Recommendation**: Not initially; evaluate if expanding to EU/Asia markets.

### Domain Configuration

- **Dedicated sending domain**: Should we use `notifications.mlc.example.com` or a completely separate domain? **Recommendation**: Subdomain is sufficient and maintains brand trust.

- **Multiple sending domains**: Separate domains for transactional vs promotional? **Recommendation**: Start with single domain, separate later if promotional volume increases significantly.

- **Dedicated IP timing**: At what volume should we move to dedicated IP? **Recommendation**: When consistently sending >100K emails/month.

### Rate Limiting

- **User-level limits**: What's the right balance for max emails/user/day? **Recommendation**: 5 emails/day for promotional, unlimited transactional.

- **Burst handling**: How to handle sudden spikes (e.g., all teachers send assignments at once)? **Recommendation**: Implement priority queue - high priority transactional, lower priority batch sends.

### Monitoring and Alerting

- **Alert thresholds**: What metrics should trigger immediate alerts? **Recommendation**: 
  - Critical: Spam complaint rate >0.1%, provider API errors >5%
  - Warning: Bounce rate >2%, delivery rate <98%

- **On-call requirements**: Does email delivery require 24/7 monitoring? **Recommendation**: No - business hours monitoring sufficient initially, automated retry handles off-hours issues.

### Cost Optimization

- **Cost ceiling**: At what monthly cost do we reconsider ESP choice? **Recommendation**: If exceeding $500/month, evaluate AWS SES migration.

- **Template vs API sending**: Should we use provider's template system or always send via API? **Recommendation**: Always via API for consistency and control, even if slightly more expensive.

### Testing and Validation

- **Test email accounts**: How to test deliverability across email clients? **Recommendation**: Use Mailtrap for development, Email on Acid for pre-launch testing.

- **Staging environment**: Should staging use same ESP or separate? **Recommendation**: Same ESP but different domain (staging.mlc.example.com).

### Compliance and Privacy

- **Data residency**: Do we need to consider where provider stores email data? **Recommendation**: SendGrid is US-based; acceptable for US market. Evaluate if expanding internationally.

- **Email archival**: Should we archive all sent emails for compliance? **Recommendation**: Store metadata only (subject, recipient, status), not full content. Retain 2 years.

- **Student email restrictions**: Should student emails always go to guardian address if under 13? **Recommendation**: Yes, per COPPA requirements. Validate on registration.

### Future Enhancements

- **Email analytics dashboard**: Build custom analytics or use provider's? **Recommendation**: Use provider dashboard initially, build custom if specific needs emerge.

- **A/B testing coordination**: How to coordinate A/B testing between our template system and provider tracking? **Recommendation**: Track variant in our system, use provider tracking for engagement metrics.

- **SMTP fallback**: Should we maintain SMTP credentials as backup? **Recommendation**: No - adds complexity without significant benefit. Provider API is more reliable than SMTP.
