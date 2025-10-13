# Rate Limiting and Abuse Prevention

## Purpose

This document defines the comprehensive rate limiting and abuse prevention strategies for the MLC LMS platform. It establishes the policies, mechanisms, and technical implementations that protect the platform from malicious attacks, accidental overuse, and resource exhaustion while ensuring fair access for legitimate users. The rate limiting system is designed to scale with different subscription tiers and user roles, providing appropriate access levels for each context.

## Scope

**Included:**
- API rate limiting policies per role and subscription tier
- Request throttling mechanisms and strategies
- Abuse detection patterns and prevention
- DDoS protection and mitigation strategies
- Bot detection and CAPTCHA integration
- Rate limit response formats and HTTP headers
- Quota management and enforcement
- Monitoring, alerting, and incident response
- User notification and communication strategies
- Implementation patterns and code examples

**Excluded:**
- Background job concurrency limits (see [Background Jobs](background-jobs.md))
- Webhook delivery rate limits (see [Webhooks](webhooks.md))
- Database connection pooling (see [System Overview](system-overview.md))
- General security architecture (see [Security and Privacy](security-privacy.md))
- Network-level DDoS protection (see [System Overview](system-overview.md) - Cloudflare)
- API design conventions (see [API Design Principles](api-design-principles.md))

---

## Rate Limiting Overview

### Rate Limiting Strategy

The MLC LMS implements a **multi-layered rate limiting approach** that protects the platform at different levels:

```
┌─────────────────────────────────────────────────────────────────────┐
│                    RATE LIMITING LAYERS                              │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 1: Network/Infrastructure Level                               │
│  - Cloudflare DDoS protection                                        │
│  - Web Application Firewall (WAF)                                    │
│  - Geographic blocking (if needed)                                   │
│  - IP reputation filtering                                           │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 2: Application Gateway Level                                  │
│  - Per-IP rate limits (coarse-grained)                               │
│  - Request size limits                                               │
│  - Connection limits                                                 │
│  - Bandwidth throttling                                              │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 3: API Route Level                                            │
│  - Per-user/session rate limits                                      │
│  - Per-endpoint specific limits                                      │
│  - Tier-based quotas (Prelude/Solo/Ensemble)                        │
│  - Role-based limits (Student/Teacher/Admin)                         │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 4: Resource-Specific Level                                    │
│  - Expensive operation throttling                                    │
│  - Bulk operation concurrency limits                                 │
│  - Report generation quotas                                          │
│  - Search query complexity limits                                    │
└─────────────────────────────────────────────────────────────────────┘
```

### Rate Limiting Principles

1. **Fair Access**: Legitimate users should never hit rate limits during normal usage
2. **Progressive Degradation**: Soft limits warn users before hard limits block them
3. **Transparency**: Users receive clear feedback about limits and quotas
4. **Tier Appropriate**: Rate limits scale with subscription tier and user role
5. **Context Aware**: Different limits apply to different operations (read vs. write)
6. **Recoverable**: Rate limits reset automatically; users can resume normal operations

---

## API Rate Limits

### Rate Limit Tiers by Role and Subscription

| User Type | Subscription Plan | General Requests (per hour) | Write Operations (per hour) | Burst Allowance | Notes |
|-----------|------------------|---------------------------|---------------------------|----------------|-------|
| **Unauthenticated** | N/A | 100 | 0 | 20 | Public endpoints only |
| **Student** | Prelude (Trial) | 500 | 50 | 50 | Normal student gameplay |
| **Student** | Solo/Ensemble | 1,000 | 100 | 100 | Full access |
| **Teacher** | Solo | 2,000 | 200 | 200 | Assignment creation, progress checks |
| **Teacher** | Ensemble | 3,000 | 300 | 300 | Higher limits for larger classes |
| **Teacher-Admin** | Ensemble | 5,000 | 500 | 500 | Bulk operations, reporting |
| **Subscriber-Admin** | Solo | 2,000 | 200 | 200 | Similar to Teacher |
| **Subscriber-Admin** | Ensemble | 5,000 | 500 | 500 | Bulk operations, account management |
| **System Admin** | Internal | 10,000 | 1,000 | 1,000 | Administrative operations |
| **Guest** | N/A | 50 | 0 | 10 | Heavily restricted |

**Rate Limit Windows:**
- **Per Hour**: Rolling 60-minute window
- **Per Minute**: Rolling 60-second window for burst detection
- **Per Day**: Rolling 24-hour window for absolute quotas

**Burst Allowance**: Temporary allowance above hourly rate to accommodate short spikes in legitimate usage.

### Rate Limits by Endpoint Category

Different endpoints have different cost profiles and require specific rate limits:

#### Read Operations (GET requests)

| Endpoint Category | Requests per Minute | Requests per Hour | Notes |
|------------------|-------------------|------------------|-------|
| `/api/auth/*` | 10 | 100 | Authentication endpoints |
| `/api/students/:id` | 60 | 1,000 | Single student data |
| `/api/students` (list) | 20 | 500 | Student list/search |
| `/api/assignments/:id` | 60 | 1,000 | Single assignment |
| `/api/assignments` (list) | 30 | 800 | Assignment list |
| `/api/games/:id` | 60 | 2,000 | Single game metadata |
| `/api/games` (list/search) | 30 | 1,000 | Game catalog browsing |
| `/api/progress/*` | 40 | 1,000 | Progress tracking |
| `/api/reports/*` | 10 | 100 | Report generation (expensive) |
| `/api/search/*` | 20 | 500 | Search operations |

#### Write Operations (POST, PUT, PATCH, DELETE)

| Endpoint Category | Requests per Minute | Requests per Hour | Notes |
|------------------|-------------------|------------------|-------|
| `/api/auth/login` | 5 | 30 | Login attempts (security) |
| `/api/auth/register` | 3 | 20 | New account creation |
| `/api/students` (create) | 10 | 100 | Student creation |
| `/api/students/:id` (update) | 20 | 200 | Student updates |
| `/api/assignments` (create) | 10 | 100 | Assignment creation |
| `/api/assignments/:id` (update) | 20 | 200 | Assignment updates |
| `/api/progress/record` | 60 | 2,000 | Game progress recording (high volume) |
| `/api/bulk/*` | 5 | 20 | Bulk operations (very expensive) |
| `/api/admin/*` (write) | 10 | 100 | Admin modifications |

**Special Considerations:**
- **Progress Recording**: Higher limits as students generate many progress events during gameplay
- **Bulk Operations**: Severely limited due to resource intensity
- **Authentication**: Strict limits to prevent brute-force attacks
- **Reports**: Limited due to database and computation cost

### Rate Limit Implementation

#### Rate Limit Middleware

All API routes pass through rate limiting middleware:

```typescript
// middleware/rateLimit.ts
import { Ratelimit } from "@upstash/ratelimit";
import { Redis } from "@upstash/redis";
import { NextRequest, NextResponse } from "next/server";

// Initialize Redis for rate limiting
const redis = Redis.fromEnv();

// Define rate limiters for different tiers
const rateLimiters = {
  unauthenticated: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(100, "1h"),
    prefix: "ratelimit:unauth",
    analytics: true,
  }),
  
  student: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(1000, "1h"),
    prefix: "ratelimit:student",
    analytics: true,
  }),
  
  teacher: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(3000, "1h"),
    prefix: "ratelimit:teacher",
    analytics: true,
  }),
  
  admin: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5000, "1h"),
    prefix: "ratelimit:admin",
    analytics: true,
  }),
  
  sysadmin: new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10000, "1h"),
    prefix: "ratelimit:sysadmin",
    analytics: true,
  }),
};

// Define endpoint-specific limiters
const endpointLimiters = {
  "POST:/api/auth/login": new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5, "1m"),
    prefix: "ratelimit:login",
  }),
  
  "POST:/api/auth/register": new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(3, "1m"),
    prefix: "ratelimit:register",
  }),
  
  "POST:/api/bulk/*": new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(5, "1h"),
    prefix: "ratelimit:bulk",
  }),
  
  "GET:/api/reports/*": new Ratelimit({
    redis,
    limiter: Ratelimit.slidingWindow(10, "1h"),
    prefix: "ratelimit:reports",
  }),
};

export async function rateLimitMiddleware(req: NextRequest) {
  const { user, session } = await getAuthContext(req);
  
  // Determine rate limiter based on user role
  const limiter = getRateLimiterForUser(user);
  
  // Check endpoint-specific limits first
  const endpointLimiter = getEndpointLimiter(req.method, req.url);
  if (endpointLimiter) {
    const endpointResult = await endpointLimiter.limit(
      getIdentifier(req, user)
    );
    if (!endpointResult.success) {
      return rateLimitResponse(endpointResult);
    }
  }
  
  // Check general rate limit
  const identifier = getIdentifier(req, user);
  const result = await limiter.limit(identifier);
  
  if (!result.success) {
    return rateLimitResponse(result);
  }
  
  // Add rate limit headers to response
  const response = NextResponse.next();
  addRateLimitHeaders(response, result);
  
  return response;
}

function getIdentifier(req: NextRequest, user?: User): string {
  if (user) {
    return `user:${user.id}`;
  }
  
  // Fall back to IP address for unauthenticated requests
  const ip = req.ip || 
    req.headers.get("x-forwarded-for")?.split(",")[0] ||
    req.headers.get("x-real-ip") ||
    "unknown";
  
  return `ip:${ip}`;
}

function getRateLimiterForUser(user?: User): Ratelimit {
  if (!user) {
    return rateLimiters.unauthenticated;
  }
  
  switch (user.role) {
    case "SYSTEM_ADMIN":
      return rateLimiters.sysadmin;
    case "SUBSCRIBER_ADMIN":
    case "TEACHER_ADMIN":
      return rateLimiters.admin;
    case "TEACHER":
      return rateLimiters.teacher;
    case "STUDENT":
      return rateLimiters.student;
    default:
      return rateLimiters.student;
  }
}

function rateLimitResponse(result: RatelimitResult): NextResponse {
  return NextResponse.json(
    {
      error: "RATE_LIMIT_EXCEEDED",
      message: "You have exceeded your API request quota. Please try again later.",
      retryAfter: result.reset - Date.now(),
      limit: result.limit,
      remaining: result.remaining,
      reset: result.reset,
    },
    {
      status: 429,
      headers: {
        "X-RateLimit-Limit": result.limit.toString(),
        "X-RateLimit-Remaining": "0",
        "X-RateLimit-Reset": result.reset.toString(),
        "Retry-After": Math.ceil((result.reset - Date.now()) / 1000).toString(),
      },
    }
  );
}

function addRateLimitHeaders(
  response: NextResponse,
  result: RatelimitResult
): void {
  response.headers.set("X-RateLimit-Limit", result.limit.toString());
  response.headers.set("X-RateLimit-Remaining", result.remaining.toString());
  response.headers.set("X-RateLimit-Reset", result.reset.toString());
}
```

#### Rate Limit Headers

All API responses include rate limit information in standard headers:

```
HTTP/1.1 200 OK
X-RateLimit-Limit: 1000              # Total requests allowed in window
X-RateLimit-Remaining: 847           # Requests remaining in window
X-RateLimit-Reset: 1738176000000     # Unix timestamp when limit resets
X-RateLimit-Policy: user:1000/1h     # Policy applied (user-facing description)
```

When rate limit is exceeded:

```
HTTP/1.1 429 Too Many Requests
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1738176000000
Retry-After: 3600                    # Seconds until retry is allowed

{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "You have exceeded your API request quota. Please try again later.",
  "retryAfter": 3600000,            # Milliseconds
  "limit": 1000,
  "remaining": 0,
  "reset": 1738176000000,
  "policy": {
    "type": "user",
    "window": "1h",
    "limit": 1000
  }
}
```

---

## Abuse Prevention Mechanisms

### Abuse Detection Patterns

The platform monitors for suspicious activity patterns that indicate abuse:

#### Pattern 1: Brute Force Attacks

**Detection:**
- Multiple failed login attempts from same IP or user
- Password reset requests exceeding threshold
- Sequential user enumeration attempts

**Thresholds:**
- 5 failed login attempts in 5 minutes → Temporary account lock (15 minutes)
- 10 failed login attempts in 1 hour → Extended lock (1 hour) + notification
- 20 failed login attempts in 24 hours → Account disabled + manual review required

**Response:**
```typescript
interface BruteForceResponse {
  error: "ACCOUNT_LOCKED";
  message: "Your account has been temporarily locked due to multiple failed login attempts.";
  lockedUntil: "2025-01-27T15:30:00Z";
  support: "contact@musiclearningcommunity.com";
}
```

**Mitigation:**
- Progressive delay on repeated failed attempts
- CAPTCHA requirement after 3 failed attempts
- Email notification to account owner
- IP-based temporary blocking

#### Pattern 2: Content Scraping / Automated Access

**Detection:**
- High request velocity from single source
- Sequential access to all resources in category
- Identical User-Agent across many requests
- Missing or suspicious browser fingerprints
- Lack of typical user interaction patterns (no mouse movement, instant clicks)

**Thresholds:**
- 100+ requests per minute from single IP
- Accessing 50+ game records sequentially without interaction
- Download of 100+ assets in short timeframe

**Response:**
- Progressive rate limiting (increasing delays)
- CAPTCHA challenge
- Temporary IP block (24 hours)
- Email notification to System Admins

#### Pattern 3: Resource Exhaustion Attacks

**Detection:**
- Repeated requests for expensive operations (reports, bulk exports)
- Large payload submissions exceeding normal limits
- Concurrent bulk operations from single user
- Repeated cancellation and resubmission of long-running jobs

**Thresholds:**
- 10+ concurrent bulk operations
- 20+ report generation requests per hour
- Payload size exceeding 10MB (when 1MB is typical)

**Response:**
- Reject requests with HTTP 429
- Queue operations with extended delays
- Notification to account admin
- Possible manual intervention for persistent abuse

#### Pattern 4: Account Sharing / Credential Abuse

**Detection:**
- Single account accessed from multiple IPs simultaneously
- Geographic impossibility (logins from distant locations within short timeframe)
- Concurrent sessions exceeding normal patterns
- Device fingerprint mismatches

**Thresholds:**
- 3+ concurrent sessions from different IPs
- Logins from 2+ countries within 1 hour
- 10+ unique device fingerprints per account per week

**Response:**
- Force re-authentication
- Session invalidation
- Email notification to account owner
- Account review if pattern persists

**Note:** Some account sharing is expected (e.g., teacher accessing from home and school), so thresholds are permissive and focus on extreme abuse.

#### Pattern 5: Payment Fraud

**Detection:**
- Multiple declined payment attempts
- Rapid subscription creation and cancellation
- Use of stolen credit card patterns (detected by Braintree)
- Promo code abuse (multiple redemptions)

**Thresholds:**
- 3+ declined payments within 24 hours
- 5+ trial accounts from same email/IP
- 10+ promo code attempts with different codes

**Response:**
- Block new subscription creation from IP/email
- Flag account for manual review
- Report to payment processor
- Notify fraud detection system

### CAPTCHA Integration

**When CAPTCHA Is Required:**
- After 3 failed login attempts
- Before bulk operations (CSV import, mass assignments)
- When abuse patterns are detected
- For unauthenticated sensitive operations (password reset, registration)

**CAPTCHA Provider:**
- **Primary**: hCaptcha (privacy-focused, WCAG compliant)
- **Fallback**: Google reCAPTCHA v3 (invisible, score-based)

**Implementation:**
```typescript
// CAPTCHA verification middleware
export async function verifyCaptcha(
  req: NextRequest,
  action: string
): Promise<boolean> {
  const captchaToken = req.headers.get("X-Captcha-Token");
  
  if (!captchaToken) {
    return false;
  }
  
  const response = await fetch(
    "https://hcaptcha.com/siteverify",
    {
      method: "POST",
      headers: { "Content-Type": "application/x-www-form-urlencoded" },
      body: new URLSearchParams({
        secret: process.env.HCAPTCHA_SECRET_KEY!,
        response: captchaToken,
        sitekey: process.env.HCAPTCHA_SITE_KEY!,
      }),
    }
  );
  
  const data = await response.json();
  
  if (data.success) {
    // Log successful CAPTCHA verification
    await logCaptchaVerification(req, action, true);
    return true;
  }
  
  await logCaptchaVerification(req, action, false);
  return false;
}

// CAPTCHA requirement response
function captchaRequiredResponse(): NextResponse {
  return NextResponse.json(
    {
      error: "CAPTCHA_REQUIRED",
      message: "Please complete the CAPTCHA verification to continue.",
      captcha: {
        provider: "hcaptcha",
        siteKey: process.env.NEXT_PUBLIC_HCAPTCHA_SITE_KEY,
      },
    },
    { status: 403 }
  );
}
```

### Bot Detection

**Bot Detection Signals:**
- Missing or suspicious User-Agent headers
- Lack of Accept-Language or other typical browser headers
- No JavaScript execution (for web requests)
- Unusual request timing patterns (too fast, too consistent)
- Missing browser fingerprints (canvas, WebGL, fonts)
- Inconsistent header combinations

**Bot Classification:**
```typescript
enum BotType {
  LEGITIMATE = "legitimate",        // Google, Bing crawlers (allowed)
  SUSPICIOUS = "suspicious",        // Unknown bots (rate limited)
  MALICIOUS = "malicious",          // Known bad actors (blocked)
}

async function classifyBot(req: NextRequest): Promise<BotType> {
  const userAgent = req.headers.get("user-agent") || "";
  
  // Check legitimate bot list
  const legitimateBots = [
    "Googlebot",
    "Bingbot",
    "Slackbot",
    "facebookexternalhit",
  ];
  
  if (legitimateBots.some(bot => userAgent.includes(bot))) {
    return BotType.LEGITIMATE;
  }
  
  // Check malicious bot list (maintained in Redis)
  const isMalicious = await redis.sismember(
    "bots:malicious",
    userAgent
  );
  
  if (isMalicious) {
    return BotType.MALICIOUS;
  }
  
  // Check for bot-like behavior
  const botScore = await calculateBotScore(req);
  
  if (botScore > 0.8) {
    return BotType.MALICIOUS;
  } else if (botScore > 0.5) {
    return BotType.SUSPICIOUS;
  }
  
  return BotType.LEGITIMATE;
}

async function calculateBotScore(req: NextRequest): Promise<number> {
  let score = 0;
  
  // Missing typical headers
  if (!req.headers.get("accept-language")) score += 0.2;
  if (!req.headers.get("accept-encoding")) score += 0.2;
  if (!req.headers.get("referer") && req.method === "POST") score += 0.1;
  
  // Suspicious User-Agent
  const ua = req.headers.get("user-agent") || "";
  if (ua.length < 20) score += 0.3;
  if (!ua.includes("Mozilla")) score += 0.2;
  
  // Check request frequency
  const ip = req.ip || "unknown";
  const recentRequests = await redis.incr(`bot-detect:${ip}:1m`);
  await redis.expire(`bot-detect:${ip}:1m`, 60);
  
  if (recentRequests > 60) score += 0.3;  // More than 1 req/sec
  
  return Math.min(score, 1.0);
}
```

---

## DDoS Protection

### Network-Level Protection (Cloudflare)

The primary DDoS protection is provided by Cloudflare, which handles:
- Volumetric attacks (UDP floods, ICMP floods)
- Protocol attacks (SYN floods, fragmented packet attacks)
- Application-layer attacks (HTTP floods, Slowloris)

**Cloudflare Configuration:**
- **Challenge Mode**: Enabled for suspicious traffic
- **Under Attack Mode**: Manual activation during active DDoS
- **Rate Limiting Rules**: Custom rules for known attack patterns
- **IP Reputation**: Automatic blocking of known malicious IPs
- **Geographic Restrictions**: Optional blocking of specific countries (if needed)

See [System Overview](system-overview.md) for infrastructure details.

### Application-Level Protection

The application implements additional protections:

#### Connection Limits

```typescript
// Vercel serverless function configuration
export const config = {
  maxDuration: 30,  // Max 30 seconds per request
  memory: 1024,     // 1GB memory limit
};

// Connection pooling limits (Neon PostgreSQL)
const dbConfig = {
  max: 20,          // Max 20 concurrent connections
  idleTimeout: 30000,  // 30 second idle timeout
  connectionTimeout: 5000,  // 5 second connection timeout
};
```

#### Request Size Limits

```typescript
// Body parser limits
export const bodyParserConfig = {
  json: {
    limit: "1mb",     // 1MB for JSON payloads
  },
  urlencoded: {
    limit: "1mb",
    extended: true,
  },
  raw: {
    limit: "10mb",    // 10MB for file uploads
    type: "application/octet-stream",
  },
};

// Specific endpoint overrides
const uploadLimits = {
  "/api/admin/bulk-import": "50mb",   // CSV bulk imports
  "/api/games/assets": "100mb",       // Game asset uploads
};
```

#### Slowloris Protection

```typescript
// Request timeout enforcement
export async function requestTimeoutMiddleware(
  req: NextRequest
): Promise<NextResponse> {
  const timeout = 30000;  // 30 second timeout
  
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  try {
    const response = await handleRequest(req, controller.signal);
    clearTimeout(timeoutId);
    return response;
  } catch (error) {
    if (error.name === "AbortError") {
      return NextResponse.json(
        {
          error: "REQUEST_TIMEOUT",
          message: "Request exceeded maximum processing time",
        },
        { status: 408 }
      );
    }
    throw error;
  }
}
```

---

## Monitoring and Alerting

### Rate Limit Metrics

The platform tracks rate limiting metrics for monitoring and analysis:

**Metrics Collected:**
- Total requests per minute/hour/day
- Rate limit hits (by endpoint, by user role, by subscription tier)
- Average request rate per user
- Peak request rates
- Rate limit violations (abuse attempts)
- CAPTCHA challenges issued
- CAPTCHA success/failure rates
- Bot detection results

**Storage:**
- Real-time metrics in Redis (short-term, 7 days)
- Aggregated metrics in PostgreSQL (long-term, 1 year)
- Analytics events sent to analytics platform (see [Event Model](../15-analytics-and-reporting/event-model.md))

**Event Schema:**
```typescript
interface RateLimitEvent {
  eventType: "rate_limit.hit" | "rate_limit.exceeded";
  timestamp: string;
  userId?: string;
  sessionId?: string;
  ip: string;
  endpoint: string;
  method: string;
  userRole?: string;
  subscriptionTier?: string;
  limit: number;
  remaining: number;
  reset: number;
  violationType?: "soft_limit" | "hard_limit" | "abuse";
}
```

### Alerting Thresholds

System Admins receive alerts when abuse patterns are detected:

| Alert Type | Threshold | Severity | Recipients |
|-----------|-----------|----------|-----------|
| High rate limit hits | 100+ unique users hitting limits per hour | Warning | DevOps team |
| Single user persistent abuse | Same user exceeds limit 10+ times in 1 hour | High | Security team |
| DDoS suspected | 10,000+ requests per minute from unknown IPs | Critical | DevOps + Security |
| Bot detection spike | 50+ bot detections per minute | Warning | DevOps team |
| Brute force attack | 50+ failed login attempts per minute | High | Security team |
| Payment fraud pattern | 10+ declined payments from same IP | High | Finance + Security |

**Alert Channels:**
- PagerDuty (critical alerts, 24/7)
- Slack #security-alerts (all security events)
- Email (daily digest of warnings)

### Dashboard and Reporting

**Rate Limit Dashboard** (System Admin view):
- Real-time request rate graph
- Rate limit hit rate by endpoint
- Top users by request volume
- Geographic distribution of requests
- Bot detection statistics
- Active CAPTCHA challenges

**Reports Available:**
- Daily rate limit summary
- Weekly abuse pattern report
- Monthly API usage by subscription tier
- Quarterly security incident summary

See [System Admin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md) for dashboard specifications.

---

## User Communication

### Rate Limit Notifications

**In-App Notification** (when user approaches limit):
```
⚠️ API Usage Warning

You have used 80% of your hourly API request quota (800 of 1,000 requests).

Your quota will reset in 22 minutes. If you continue to exceed limits, your 
requests may be temporarily throttled.

Need higher limits? Contact support or upgrade your subscription plan.
```

**Email Notification** (when user repeatedly exceeds limits):
```
Subject: MLC API Rate Limit Exceeded

Hello [Name],

We noticed your account has repeatedly exceeded API rate limits over the 
past 24 hours. This may indicate:

- An application or script making excessive requests
- Unusual usage patterns
- A potential security issue

Your current plan allows 1,000 API requests per hour. If you need higher 
limits, please consider upgrading to an Ensemble plan or contact our support 
team to discuss your use case.

API Usage Summary (Last 24 Hours):
- Total Requests: 45,200
- Rate Limit Hits: 23
- Average Requests/Hour: 1,883

If you did not initiate this activity, please secure your account immediately.

Best regards,
MusicLearningCommunity Security Team
```

### Developer Communication

For external API consumers (future):

**API Documentation:**
- Clear documentation of all rate limits
- Best practices for staying within limits
- Retry strategies and exponential backoff examples
- How to monitor usage via headers
- How to request limit increases

**Developer Portal** (future):
- Real-time API usage dashboard
- Historical usage graphs
- Alert setup for approaching limits
- Webhook notifications for limit events

---

## Special Cases and Exemptions

### System Admin Exemptions

System Admins have elevated rate limits but are still subject to monitoring:
- **Rate Limits**: 10x normal user limits
- **Monitoring**: All requests logged for audit
- **Alerts**: Notify if System Admin exceeds even elevated limits (potential compromised account)

### Bulk Operations

Bulk operations (CSV imports, mass assignments) have special handling:
- **Concurrency**: Max 10 concurrent bulk jobs per user (see [Background Jobs](background-jobs.md))
- **Rate Limits**: Max 20 bulk job submissions per hour
- **Queue Priority**: Bulk jobs have lower priority than interactive requests
- **Progress Tracking**: Real-time progress updates prevent repeated submissions

### Free Trial (Prelude Plan)

Trial accounts have more restrictive limits to prevent abuse:
- **API Requests**: 50% of paid plan limits
- **Bulk Operations**: Disabled or limited to small batches
- **Reports**: Limited to 5 per day
- **Duration**: 14-day time limit with automatic expiration

---

## Supporting Documents Referenced

This rate limiting and abuse prevention specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role definitions informing role-based rate limit tiers and permission-appropriate API quotas
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator monitoring tools and security dashboards for abuse detection and rate limit management
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional admin security features including anomaly detection and incident response capabilities
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription tier specifications informing usage quotas and rate limits by plan type (Prelude/Solo/Ensemble)

---

## Implementation References

### Related Specifications

- [API Design Principles](api-design-principles.md) - API conventions and patterns
- [REST Endpoints](rest-endpoints.md) - Specific endpoint rate limit notes
- [Background Jobs](background-jobs.md) - Job concurrency and rate limits
- [Webhooks](webhooks.md) - Webhook delivery rate limits
- [Security and Privacy](security-privacy.md) - General security architecture
- [System Overview](system-overview.md) - Infrastructure and DDoS protection

### Technology Stack

- **Rate Limiting**: Upstash Redis + @upstash/ratelimit SDK
- **DDoS Protection**: Cloudflare WAF and rate limiting
- **Bot Detection**: Custom scoring + hCaptcha
- **Monitoring**: Custom analytics + event tracking
- **Alerting**: PagerDty + Slack webhooks

### Future Enhancements

**Planned:**
- Adaptive rate limiting based on system load
- Machine learning-based abuse detection
- Distributed rate limiting across multiple regions
- GraphQL-specific rate limiting (by query complexity)
- WebSocket connection rate limits

**Considerations:**
- Dynamic rate limit adjustment based on subscription usage patterns
- Credits system for burst usage beyond limits
- API key management for external integrations
- White-label rate limiting for institutional deployments

---

## Summary

The MLC LMS rate limiting and abuse prevention system provides:

**Multi-Layered Protection:**
- Network-level DDoS protection via Cloudflare
- Application-level rate limiting via Redis
- Endpoint-specific limits for expensive operations
- Role and subscription-tier appropriate quotas

**Abuse Prevention:**
- Brute force attack detection and mitigation
- Content scraping prevention
- Resource exhaustion protection
- Bot detection and CAPTCHA challenges
- Payment fraud pattern detection

**User Experience:**
- Transparent rate limit communication via HTTP headers
- Progressive warnings before hard limits
- Fair access for legitimate users
- Clear upgrade paths for higher limits

**Operational Excellence:**
- Comprehensive monitoring and alerting
- Real-time dashboards for System Admins
- Automated incident response
- Regular security reporting

This architecture ensures the MLC platform remains available, performant, and secure for all legitimate users while effectively preventing abuse and malicious activity.
