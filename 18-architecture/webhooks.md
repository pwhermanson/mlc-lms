# Webhooks

## Purpose

This document specifies the webhook architecture for the MLC LMS platform, defining how the system receives real-time event notifications from external services (inbound webhooks) and how it delivers event notifications to external integrations (outbound webhooks). Webhooks enable asynchronous, event-driven communication between MLC and external systems, supporting use cases like payment processing, third-party integrations, institutional LMS synchronization, and custom automation workflows.

## Scope

### Included

**Inbound Webhooks (External → MLC)**:
- Payment provider webhooks (Braintree, Stripe) for subscription lifecycle events
- Future external service integrations requiring event delivery to MLC
- Webhook security and signature verification
- Webhook processing and retry logic
- Error handling and monitoring

**Outbound Webhooks (MLC → External)**:
- Event notification delivery to configured external endpoints
- Webhook subscription management and configuration
- Delivery retry logic with exponential backoff
- Security via HMAC signature generation
- Rate limiting and abuse prevention
- Webhook delivery monitoring and diagnostics

**Infrastructure**:
- Webhook endpoint routing and authentication
- Event queuing and async processing (see [Background Jobs](background-jobs.md))
- Idempotency handling for duplicate events
- Webhook testing and debugging tools

### Excluded

- Specific business logic for individual webhook event handlers (see domain-specific docs)
- User interface for webhook configuration (covered in System Admin docs)
- Real-time WebSocket connections (separate specification)
- Polling-based integrations (not webhook-based)
- Detailed payment provider integration specs (see [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md))
- Detailed LTI integration specs (see [LMS LTI Spec](../17-integrations/lms-ltispec-future.md))

---

## Inbound Webhooks (Receiving Events)

Inbound webhooks allow external services to notify MLC of events in real-time, enabling the platform to react immediately to changes in payment status, user provisioning, or other external system state changes.

### Webhook Endpoint Structure

All inbound webhooks follow a consistent URL pattern:

```
POST /api/webhooks/:provider/:event_type?
```

**Examples**:
```
POST /api/webhooks/braintree         # Braintree payment events
POST /api/webhooks/stripe            # Stripe payment events
POST /api/webhooks/lti/:platform_id  # LTI platform events (future)
POST /api/webhooks/custom/:client_id # Custom integration webhooks (future)
```

### Security and Verification

#### Signature Verification

All inbound webhooks must be verified to prevent spoofing and unauthorized access.

**Braintree Signature Verification**:
```typescript
import crypto from 'crypto';
import { BraintreeConfig } from '@/lib/config/braintree';

export async function verifyBraintreeWebhook(
  signature: string,
  payload: string
): Promise<boolean> {
  const expectedSignature = crypto
    .createHmac('sha256', BraintreeConfig.webhookSecret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

**Stripe Signature Verification**:
```typescript
import Stripe from 'stripe';

export async function verifyStripeWebhook(
  payload: string | Buffer,
  signature: string
): Promise<Stripe.Event> {
  const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);
  
  return stripe.webhooks.constructEvent(
    payload,
    signature,
    process.env.STRIPE_WEBHOOK_SECRET
  );
}
```

**Generic HMAC Verification** (for custom integrations):
```typescript
export function verifyHMACSignature(
  payload: string,
  signature: string,
  secret: string,
  algorithm: 'sha256' | 'sha512' = 'sha256'
): boolean {
  const expectedSignature = crypto
    .createHmac(algorithm, secret)
    .update(payload)
    .digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

#### Request Validation

Webhook endpoints validate:
1. **Signature**: HMAC/JWT signature must be valid
2. **Timestamp**: Event timestamp must be within 5 minutes (prevents replay attacks)
3. **Provider ID**: Provider/client must be registered and active
4. **Event ID**: Event ID must be unique (idempotency check)
5. **Payload Schema**: Payload must match expected schema for event type

### Inbound Webhook Processing Pattern

**Standard Processing Flow**:
```typescript
// pages/api/webhooks/[provider]/index.ts
import { withWebhookAuth } from '@/lib/middleware/webhook-auth';
import { WebhookService } from '@/lib/domains/webhooks';

export default withWebhookAuth(async (req, res) => {
  const { provider } = req.query;
  
  // 1. Verify signature (done in middleware)
  // 2. Parse webhook payload
  const event = req.body;
  
  // 3. Check for duplicate (idempotency)
  const isDuplicate = await WebhookService.isDuplicateEvent(
    provider,
    event.id
  );
  
  if (isDuplicate) {
    // Already processed, return success to prevent retries
    return res.status(200).json({ received: true, duplicate: true });
  }
  
  // 4. Log webhook event
  await WebhookService.logInboundEvent({
    provider,
    eventId: event.id,
    eventType: event.type,
    payload: event.data,
    receivedAt: new Date(),
  });
  
  // 5. Enqueue for async processing
  await WebhookService.enqueueEventProcessing(provider, event);
  
  // 6. Return 200 immediately (process async)
  res.status(200).json({ received: true });
  
  // 7. Process event in background
  // (handled by background job queue)
});
```

### Idempotency Handling

Webhooks may be delivered multiple times. MLC ensures idempotent processing:

**Idempotency Strategy**:
```typescript
interface WebhookEventRecord {
  id: string;
  provider: string;
  eventId: string;  // Provider's event ID
  eventType: string;
  payload: any;
  processedAt: Date | null;
  status: 'received' | 'processing' | 'completed' | 'failed';
  retryCount: number;
  errorMessage?: string;
}

// Check for duplicate events
export async function isDuplicateEvent(
  provider: string,
  eventId: string
): Promise<boolean> {
  const existing = await db.webhookEvents.findUnique({
    where: {
      provider_eventId: {
        provider,
        eventId,
      }
    }
  });
  
  return existing !== null;
}
```

**Idempotency Window**: 7 days (events older than 7 days are considered new)

### Error Handling and Retries

**Response Codes**:
- `200 OK`: Event received and queued successfully
- `400 Bad Request`: Invalid payload or malformed request
- `401 Unauthorized`: Missing or invalid signature
- `409 Conflict`: Duplicate event (already processed)
- `500 Internal Server Error`: Processing failure (provider should retry)
- `503 Service Unavailable`: System temporarily unavailable (provider should retry)

**Provider Retry Behavior**:
- Most providers retry failed webhooks with exponential backoff
- Braintree: Retries up to 3 times over 24 hours
- Stripe: Retries up to 3 days with increasing intervals
- MLC must return proper HTTP status codes to control retry behavior

**Internal Retry Logic**:
If async processing fails, MLC retries internally:
```typescript
const RETRY_SCHEDULE = [
  60 * 1000,      // 1 minute
  5 * 60 * 1000,  // 5 minutes
  30 * 60 * 1000, // 30 minutes
  2 * 60 * 60 * 1000, // 2 hours
  12 * 60 * 60 * 1000, // 12 hours
];

export async function retryFailedWebhook(eventId: string) {
  const event = await db.webhookEvents.findUnique({ where: { id: eventId } });
  
  if (event.retryCount >= RETRY_SCHEDULE.length) {
    // Max retries exceeded, mark as failed and alert
    await WebhookService.markAsFailed(eventId, 'Max retries exceeded');
    await AlertService.notifyWebhookFailure(event);
    return;
  }
  
  const delay = RETRY_SCHEDULE[event.retryCount];
  await JobQueue.enqueue('webhook-retry', { eventId }, { delay });
}
```

### Braintree Webhook Events

**Subscription Lifecycle Events**:
```typescript
// Event types handled
const BRAINTREE_EVENTS = {
  // Subscription events
  'subscription_charged_successfully': handleSubscriptionCharged,
  'subscription_charged_unsuccessfully': handleSubscriptionChargeFailed,
  'subscription_canceled': handleSubscriptionCanceled,
  'subscription_expired': handleSubscriptionExpired,
  'subscription_went_active': handleSubscriptionActivated,
  'subscription_went_past_due': handleSubscriptionPastDue,
  'subscription_trial_ended': handleSubscriptionTrialEnded,
  
  // Payment method events
  'payment_method_revoked': handlePaymentMethodRevoked,
  
  // Dispute events
  'dispute_opened': handleDisputeOpened,
  'dispute_won': handleDisputeWon,
  'dispute_lost': handleDisputeLost,
};
```

**Example Handler**:
```typescript
async function handleSubscriptionCharged(event: BraintreeWebhookEvent) {
  const subscription = event.subscription;
  
  // Update subscription status in database
  await SubscriptionService.recordPayment({
    subscriptionId: subscription.id,
    amount: subscription.price,
    status: 'succeeded',
    transactionId: subscription.transactions[0]?.id,
    billingPeriodStart: subscription.billing_period_start_date,
    billingPeriodEnd: subscription.billing_period_end_date,
  });
  
  // Emit internal event for notifications
  await EventBus.emit('subscription.payment_succeeded', {
    subscriptionId: subscription.id,
    amount: subscription.price,
  });
  
  // Send receipt email
  await NotificationService.sendPaymentReceipt(subscription);
}
```

### Stripe Webhook Events

**Subscription Lifecycle Events**:
```typescript
const STRIPE_EVENTS = {
  // Subscription events
  'invoice.payment_succeeded': handleInvoicePaymentSucceeded,
  'invoice.payment_failed': handleInvoicePaymentFailed,
  'customer.subscription.created': handleSubscriptionCreated,
  'customer.subscription.updated': handleSubscriptionUpdated,
  'customer.subscription.deleted': handleSubscriptionDeleted,
  'customer.subscription.trial_will_end': handleTrialWillEnd,
  
  // Payment method events
  'payment_method.attached': handlePaymentMethodAttached,
  'payment_method.detached': handlePaymentMethodDetached,
  'payment_method.updated': handlePaymentMethodUpdated,
  
  // Dispute events
  'charge.dispute.created': handleDisputeCreated,
  'charge.dispute.closed': handleDisputeClosed,
};
```

---

## Outbound Webhooks (Sending Events)

Outbound webhooks enable external systems to receive real-time notifications when events occur in MLC, supporting integrations with institutional systems, analytics platforms, and custom automation workflows.

### Webhook Subscription Management

**Subscription Entity**:
```typescript
interface WebhookSubscription {
  id: string;
  organizationId: string;  // Scope to organization
  name: string;
  url: string;  // Destination endpoint
  events: string[];  // Array of event types to subscribe to
  secret: string;  // For HMAC signature generation
  isActive: boolean;
  headers?: Record<string, string>;  // Custom headers
  retryConfig?: {
    maxRetries: number;
    backoffMultiplier: number;
  };
  createdAt: Date;
  createdBy: string;
  lastDeliveryAt?: Date;
  lastDeliveryStatus?: 'success' | 'failed';
}
```

**Creating a Webhook Subscription** (System Admin only):
```typescript
POST /api/webhooks/subscriptions
{
  "name": "Canvas LMS Integration",
  "url": "https://canvas.school.edu/api/webhooks/mlc",
  "events": [
    "assignment.completed",
    "score.recorded",
    "badge.awarded"
  ],
  "secret": "whsec_abc123..."  // Generated by MLC or provided by client
}
```

### Event Types Available for Subscription

Outbound webhooks can be subscribed to the following event categories:

**Learning Progress Events**:
- `assignment.created`
- `assignment.started`
- `assignment.completed`
- `assignment.overdue`
- `score.recorded`
- `game.completed`
- `sequence.completed`
- `concept.mastered`

**Gamification Events**:
- `badge.awarded`
- `points.earned`
- `streak.achieved`
- `leaderboard.position_changed`

**User Management Events**:
- `student.created`
- `student.activated`
- `student.deactivated`
- `teacher.created`
- `class.created`
- `class.updated`

**Subscription Events**:
- `subscription.created`
- `subscription.renewed`
- `subscription.canceled`
- `subscription.payment_failed`
- `seat.allocated`
- `seat.released`

### Outbound Webhook Payload Format

All outbound webhooks follow a consistent payload structure:

```json
{
  "webhook_id": "wh_abc123xyz",
  "event": "assignment.completed",
  "event_id": "evt_def456",
  "timestamp": "2025-01-27T15:30:01.123Z",
  "organization_id": "org_12345",
  "data": {
    "assignment_id": "asg_67890",
    "student_id": "stu_11111",
    "sequence_id": "seq_22222",
    "completed_at": "2025-01-27T15:30:00Z",
    "score": 92,
    "total_games": 15,
    "passed_games": 14
  },
  "meta": {
    "delivery_attempt": 1,
    "signature": "sha256=abc123..."
  }
}
```

### Security via HMAC Signatures

All outbound webhooks include an HMAC signature for verification:

**Signature Generation**:
```typescript
import crypto from 'crypto';

export function generateWebhookSignature(
  payload: string,
  secret: string
): string {
  return crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
}

// Add signature to request headers
const signature = generateWebhookSignature(
  JSON.stringify(payload),
  subscription.secret
);

headers['X-MLC-Signature'] = `sha256=${signature}`;
headers['X-MLC-Timestamp'] = timestamp.toString();
headers['X-MLC-Event-Type'] = eventType;
```

**Verification Guide for Recipients**:
```typescript
// Example for webhook recipients to verify MLC signatures
function verifyMLCWebhook(
  payload: string,
  signature: string,
  secret: string
): boolean {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(payload)
    .digest('hex');
  
  const receivedSignature = signature.replace('sha256=', '');
  
  return crypto.timingSafeEqual(
    Buffer.from(expectedSignature),
    Buffer.from(receivedSignature)
  );
}
```

### Webhook Delivery and Retry Logic

**Delivery Process**:
```typescript
export async function deliverWebhook(
  subscription: WebhookSubscription,
  event: WebhookEvent
) {
  const payload = {
    webhook_id: generateWebhookId(),
    event: event.type,
    event_id: event.id,
    timestamp: new Date().toISOString(),
    organization_id: subscription.organizationId,
    data: event.data,
    meta: {
      delivery_attempt: 1,
    },
  };
  
  const signature = generateWebhookSignature(
    JSON.stringify(payload),
    subscription.secret
  );
  
  try {
    const response = await fetch(subscription.url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'User-Agent': 'MLC-LMS-Webhooks/1.0',
        'X-MLC-Signature': `sha256=${signature}`,
        'X-MLC-Timestamp': Date.now().toString(),
        'X-MLC-Event-Type': event.type,
        ...subscription.headers,
      },
      body: JSON.stringify(payload),
      timeout: 5000,  // 5 second timeout
    });
    
    if (response.ok) {
      await WebhookService.recordDeliverySuccess(subscription.id, event.id);
    } else {
      await WebhookService.recordDeliveryFailure(
        subscription.id,
        event.id,
        `HTTP ${response.status}: ${response.statusText}`
      );
      // Schedule retry
      await scheduleWebhookRetry(subscription, event, 1);
    }
  } catch (error) {
    await WebhookService.recordDeliveryFailure(
      subscription.id,
      event.id,
      error.message
    );
    // Schedule retry
    await scheduleWebhookRetry(subscription, event, 1);
  }
}
```

**Retry Strategy**:
```typescript
const DEFAULT_RETRY_SCHEDULE = [
  60 * 1000,           // 1 minute
  5 * 60 * 1000,       // 5 minutes
  15 * 60 * 1000,      // 15 minutes
  1 * 60 * 60 * 1000,  // 1 hour
  3 * 60 * 60 * 1000,  // 3 hours
];

async function scheduleWebhookRetry(
  subscription: WebhookSubscription,
  event: WebhookEvent,
  attemptNumber: number
) {
  const maxRetries = subscription.retryConfig?.maxRetries || 5;
  
  if (attemptNumber >= maxRetries) {
    // Max retries exceeded
    await WebhookService.markAsFailed(subscription.id, event.id);
    await AlertService.notifyWebhookFailure(subscription, event);
    return;
  }
  
  const delay = DEFAULT_RETRY_SCHEDULE[attemptNumber - 1] || 
    (3 * 60 * 60 * 1000); // Default to 3 hours for retries beyond schedule
  
  await JobQueue.enqueue(
    'webhook-delivery',
    {
      subscriptionId: subscription.id,
      eventId: event.id,
      attemptNumber: attemptNumber + 1,
    },
    { delay }
  );
}
```

**Retry Behavior**:
- Retries triggered on: HTTP 5xx errors, network timeouts, connection failures
- No retries for: HTTP 4xx errors (except 429 Too Many Requests, 408 Request Timeout)
- Success defined as: HTTP 200-299 response within 5 seconds
- Failed webhooks are logged and reported to System Admins

### Rate Limiting for Outbound Webhooks

To prevent overwhelming recipient systems:

**Rate Limits**:
- Maximum 100 webhook deliveries per second per organization
- Maximum 10,000 webhook deliveries per hour per organization
- Maximum 3 concurrent deliveries to same endpoint

**Backpressure Handling**:
```typescript
export async function canDeliverWebhook(
  subscription: WebhookSubscription
): Promise<boolean> {
  // Check rate limits
  const recentDeliveries = await WebhookService.getRecentDeliveryCount(
    subscription.organizationId,
    60 * 1000  // Last 1 minute
  );
  
  if (recentDeliveries >= 100) {
    return false;  // Rate limit exceeded
  }
  
  // Check concurrent deliveries to same endpoint
  const concurrentDeliveries = await WebhookService.getConcurrentDeliveries(
    subscription.url
  );
  
  if (concurrentDeliveries >= 3) {
    return false;  // Too many concurrent deliveries
  }
  
  return true;
}
```

### Webhook Testing and Debugging

**Webhook Test Endpoint**:
```typescript
POST /api/webhooks/subscriptions/:id/test
{
  "event_type": "assignment.completed",
  "sample_data": {
    // Optional: provide custom test data
  }
}

// Triggers a test webhook delivery with sample data
```

**Webhook Delivery Logs**:
```typescript
GET /api/webhooks/subscriptions/:id/deliveries?limit=50

// Returns recent delivery attempts with status and response
{
  "deliveries": [
    {
      "id": "del_abc123",
      "event_id": "evt_def456",
      "event_type": "assignment.completed",
      "attempted_at": "2025-01-27T15:30:01Z",
      "status": "success",
      "http_status": 200,
      "response_time_ms": 150,
      "retry_count": 0
    },
    {
      "id": "del_xyz789",
      "event_id": "evt_ghi012",
      "event_type": "score.recorded",
      "attempted_at": "2025-01-27T15:29:45Z",
      "status": "failed",
      "http_status": 500,
      "error_message": "Internal Server Error",
      "retry_count": 2,
      "next_retry_at": "2025-01-27T15:44:45Z"
    }
  ]
}
```

### Monitoring and Observability

**Webhook Metrics**:
```typescript
interface WebhookMetrics {
  total_deliveries: number;
  successful_deliveries: number;
  failed_deliveries: number;
  average_response_time_ms: number;
  p95_response_time_ms: number;
  retry_rate: number;
  failure_rate: number;
}

// Track per subscription and globally
GET /api/webhooks/metrics
GET /api/webhooks/subscriptions/:id/metrics
```

**Alerting Conditions**:
- Webhook failure rate > 10% over 1 hour
- Average response time > 3 seconds
- More than 100 failed deliveries in 1 hour
- Subscription has been failing for > 24 hours

---

## Permissions

### Inbound Webhook Access
- **No Authentication Required**: Inbound webhooks use signature verification instead of user auth
- **Provider Registration**: Only registered providers can send webhooks
- **IP Allowlisting**: Optional IP allowlisting for additional security (future)

### Outbound Webhook Management
- **System Admins**: Full access to create, update, delete webhook subscriptions for any organization
- **Subscriber Admins**: Can view and manage webhook subscriptions for their organization
- **Teachers/Students**: No access to webhook configuration
- **Webhook Secret**: Treated as sensitive credential, only shown once at creation

---

## Supporting Documents Referenced

This webhooks specification draws from the following source documents:

- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Billing webhook requirements
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System event webhook specifications

## Dependencies

This specification relies on:
- [System Overview](system-overview.md) - Architecture and deployment context
- [API Design Principles](api-design-principles.md) - RESTful patterns and webhook design patterns
- [Background Jobs](background-jobs.md) - Async processing for webhook delivery and retry
- [Event Model](../15-analytics-and-reporting/event-model.md) - Event taxonomy and telemetry
- [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) - Payment webhook handling
- [Rate Limiting and Abuse](rate-limiting-and-abuse.md) - Rate limiting policies
- [Security & Privacy](security-privacy.md) - Security controls and credential management

---

## Open Questions

### Near-Term Decisions

1. **Webhook Queue**: Should webhook delivery use the same background job queue as other async tasks, or a dedicated webhook delivery queue (for better isolation)?
2. **Event Filtering**: Should webhook subscriptions support filtering by additional criteria (e.g., only scores above 80, only specific students)?
3. **Webhook Signing Algorithm**: Should we support multiple signature algorithms (SHA-256, SHA-512) or standardize on one?
4. **Replay Protection**: Should we enforce timestamp validation on inbound webhooks to prevent replay attacks?
5. **Webhook UI**: Should webhook configuration be exposed to Subscriber Admins in Phase 1, or initially System Admin-only?

### Long-Term Considerations

1. **Webhook Marketplace**: Should we create a catalog of pre-built webhook integrations for common systems (Canvas, Schoology, PowerSchool)?
2. **Event Schema Registry**: As we version events, should we maintain a formal schema registry for webhook consumers?
3. **Webhook Analytics**: Should we provide analytics dashboards showing webhook performance and delivery success rates?
4. **Circuit Breakers**: At what failure threshold should we automatically disable a failing webhook subscription?
5. **Webhook Transformations**: Should we support payload transformation/mapping to match different external system expectations?
6. **Multi-Region Delivery**: For international customers, should webhooks be delivered from regional edge locations for lower latency?

---

## Summary

The MLC LMS webhook architecture provides:

**Inbound Webhooks**:
- Secure reception of payment and subscription events from Braintree and Stripe
- Signature verification and idempotency handling
- Async processing with retry logic
- Extensible for future external integrations

**Outbound Webhooks**:
- Real-time event delivery to external systems
- Flexible subscription management per organization
- HMAC signature security for recipient verification
- Robust retry logic with exponential backoff
- Rate limiting and monitoring

This architecture enables MLC to integrate deeply with institutional systems, payment providers, and custom automation workflows while maintaining security, reliability, and scalability.
