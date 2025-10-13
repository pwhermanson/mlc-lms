# Config and Secrets Management

## Purpose

This document defines the configuration and secrets management strategy for the MLC LMS platform. It establishes patterns for storing, accessing, and managing environment-specific configuration values and sensitive credentials across development, staging, and production environments while maintaining security, type safety, and developer experience.

## Scope

**Included:**
- Configuration management philosophy and patterns
- Secret storage and access strategies
- Environment variable organization and naming conventions
- Secret rotation and lifecycle management
- Configuration validation and type safety
- Development vs. production configuration strategies
- Secret injection and runtime access patterns
- Configuration documentation and discovery
- Credentials management for third-party integrations
- Feature flags and environment-specific behavior
- Configuration versioning and change management
- Emergency secret rotation procedures

**Excluded:**
- Security architecture and controls (see [Security and Privacy](security-privacy.md))
- Specific integration API specifications (see [Integrations](../17-integrations/))
- Infrastructure deployment automation (see [Release Management](../19-quality-and-operations/release-management.md))
- Application monitoring and observability (see [Observability](../19-quality-and-operations/observability.md))
- Rate limiting configuration (see [Rate Limiting and Abuse](rate-limiting-and-abuse.md))

---

## Configuration Philosophy

### Core Principles

1. **12-Factor App Methodology**: Configuration lives in the environment, not in code
2. **Secrets Never Committed**: No credentials, API keys, or sensitive data in source control
3. **Type Safety**: Configuration is validated at startup with TypeScript types
4. **Environment Parity**: Dev/staging/production use the same configuration structure
5. **Fail Fast**: Application refuses to start with missing or invalid configuration
6. **Least Privilege**: Each service/component accesses only the secrets it needs
7. **Rotation-Ready**: Secrets can be rotated without code changes or full deployments
8. **Audit Trail**: Configuration changes are logged and tracked

### Configuration Hierarchy

The platform uses a layered configuration approach:

```
┌─────────────────────────────────────────────────────────────────────┐
│                  CONFIGURATION HIERARCHY                             │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 1: Environment Variables (Vercel/Local)                       │
│  - Runtime configuration injected by platform                        │
│  - Highest priority, overrides all other layers                      │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 2: Secret Manager (Vercel Environment Variables)              │
│  - Encrypted at rest, accessed via platform API                      │
│  - Used for sensitive credentials and API keys                       │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 3: Environment-Specific Config Files (.env.local, etc.)       │
│  - Local development overrides                                       │
│  - Gitignored, developer-specific settings                           │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 4: Default Config (lib/config/defaults.ts)                    │
│  - Non-sensitive defaults                                            │
│  - Committed to repository                                           │
│  - Feature flags, timeouts, pagination limits                        │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Environment Variable Organization

### Naming Conventions

All environment variables follow consistent naming patterns:

```typescript
// Service/Provider Prefixes
DATABASE_URL              // Database connection strings
NEXTAUTH_*                // Authentication configuration
R2_*                      // Cloudflare R2 object storage
STRIPE_*                  // Stripe payment provider
BRAINTREE_*               // Braintree payment provider
SENDGRID_*                // SendGrid email provider
AWS_SES_*                 // AWS SES email provider

// Environment Suffixes
*_URL                     // HTTP endpoints
*_KEY                     // API keys
*_SECRET                  // Secret keys, passwords
*_ID                      // Account/application identifiers
*_REGION                  // Geographic regions
*_TOKEN                   // Access tokens
```

### Required Environment Variables

#### Core Application

```bash
# Application Identity
NODE_ENV=development|staging|production
NEXT_PUBLIC_APP_URL=https://app.musiclearningcommunity.com
NEXT_PUBLIC_API_URL=https://app.musiclearningcommunity.com/api
VERCEL_ENV=development|preview|production  # Auto-set by Vercel

# Next.js Configuration
NEXTAUTH_URL=https://app.musiclearningcommunity.com
NEXTAUTH_SECRET=<64-char-random-string>
```

#### Database

```bash
# Neon PostgreSQL
DATABASE_URL=postgresql://user:password@host:5432/dbname
DATABASE_POOL_MAX=10
DATABASE_CONNECTION_TIMEOUT=30000  # milliseconds
```

#### Object Storage

```bash
# Cloudflare R2
R2_ACCOUNT_ID=<cloudflare-account-id>
R2_ACCESS_KEY_ID=<r2-access-key>
R2_SECRET_ACCESS_KEY=<r2-secret-key>
R2_BUCKET_NAME=mlc-assets-production
R2_PUBLIC_URL=https://assets.musiclearningcommunity.com
```

#### Payment Processing

```bash
# Stripe (Primary)
STRIPE_PUBLIC_KEY=pk_live_...
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Braintree (Secondary/Legacy)
BRAINTREE_MERCHANT_ID=<merchant-id>
BRAINTREE_PUBLIC_KEY=<public-key>
BRAINTREE_PRIVATE_KEY=<private-key>
BRAINTREE_ENVIRONMENT=sandbox|production
```

#### Email Provider

```bash
# SendGrid
SENDGRID_API_KEY=SG....
SENDGRID_FROM_EMAIL=notifications@musiclearningcommunity.com
SENDGRID_FROM_NAME=Music Learning Community

# AWS SES (Backup)
AWS_SES_ACCESS_KEY_ID=<access-key>
AWS_SES_SECRET_ACCESS_KEY=<secret-key>
AWS_SES_REGION=us-east-1
```

#### Feature Flags

```bash
# Feature Toggles (non-sensitive, can be public)
NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING=true
NEXT_PUBLIC_FEATURE_AI_CONTENT=false
NEXT_PUBLIC_FEATURE_MIDI_SUPPORT=true
```

#### Monitoring and Observability

```bash
# Error Tracking (Future)
SENTRY_DSN=https://...@sentry.io/...
SENTRY_ENVIRONMENT=production

# Analytics (Future)
ANALYTICS_WRITE_KEY=<segment-write-key>
```

### Optional/Advanced Variables

```bash
# Rate Limiting
RATE_LIMIT_MAX_REQUESTS=100
RATE_LIMIT_WINDOW_MS=60000

# Session Configuration
SESSION_MAX_AGE=86400  # 24 hours in seconds
SESSION_UPDATE_AGE=3600  # 1 hour

# CORS Configuration
CORS_ALLOWED_ORIGINS=https://app.musiclearningcommunity.com,https://admin.musiclearningcommunity.com

# Background Jobs (Future)
REDIS_URL=redis://...
QUEUE_CONCURRENCY=5
```

---

## Secret Storage Strategy

### Vercel Environment Variables

Primary secret storage for deployed environments:

```typescript
// Accessed via process.env
const config = {
  database: {
    url: process.env.DATABASE_URL,
  },
  stripe: {
    secretKey: process.env.STRIPE_SECRET_KEY,
  },
};
```

**Storage Locations by Environment:**
- **Production**: Vercel Environment Variables (Production scope)
- **Staging/Preview**: Vercel Environment Variables (Preview scope)
- **Development**: Local `.env.local` file (gitignored)

### Access Control

**Vercel Environment Variables Support:**
- Encryption at rest
- Scoped access (Production/Preview/Development)
- Team member access control via Vercel dashboard
- Audit logs for configuration changes
- Automatic injection into serverless functions

### Secret Classification

Secrets are classified by sensitivity level:

```typescript
enum SecretClassification {
  CRITICAL = 'critical',     // Database passwords, payment API keys
  HIGH = 'high',             // Email API keys, OAuth secrets
  MEDIUM = 'medium',         // CDN tokens, monitoring keys
  LOW = 'low',               // Feature flags, non-sensitive URLs
}
```

**Access Patterns by Classification:**
- **CRITICAL**: Stored encrypted, accessed only at runtime, never logged
- **HIGH**: Stored encrypted, accessed at runtime, redacted in logs
- **MEDIUM**: Stored encrypted, may be cached temporarily
- **LOW**: Can be exposed to client (NEXT_PUBLIC_* prefix)

---

## Configuration Type Safety

### Typed Configuration Module

Centralized configuration with TypeScript validation:

```typescript
// lib/config/index.ts
import { z } from 'zod';

// Configuration schema with validation
const configSchema = z.object({
  app: z.object({
    env: z.enum(['development', 'staging', 'production']),
    url: z.string().url(),
    apiUrl: z.string().url(),
  }),
  database: z.object({
    url: z.string().min(1),
    poolMax: z.number().int().positive().default(10),
    connectionTimeout: z.number().int().positive().default(30000),
  }),
  auth: z.object({
    nextAuthUrl: z.string().url(),
    nextAuthSecret: z.string().min(32),
  }),
  storage: z.object({
    r2AccountId: z.string().min(1),
    r2AccessKeyId: z.string().min(1),
    r2SecretAccessKey: z.string().min(1),
    r2BucketName: z.string().min(1),
    r2PublicUrl: z.string().url(),
  }),
  payments: z.object({
    stripe: z.object({
      publicKey: z.string().startsWith('pk_'),
      secretKey: z.string().startsWith('sk_'),
      webhookSecret: z.string().startsWith('whsec_'),
    }),
    braintree: z.object({
      merchantId: z.string().min(1),
      publicKey: z.string().min(1),
      privateKey: z.string().min(1),
      environment: z.enum(['sandbox', 'production']),
    }).optional(),
  }),
  email: z.object({
    sendgrid: z.object({
      apiKey: z.string().startsWith('SG.'),
      fromEmail: z.string().email(),
      fromName: z.string().min(1),
    }),
  }),
  features: z.object({
    socialLearning: z.boolean().default(false),
    aiContent: z.boolean().default(false),
    midiSupport: z.boolean().default(true),
  }),
});

export type Config = z.infer<typeof configSchema>;

// Load and validate configuration
function loadConfig(): Config {
  const rawConfig = {
    app: {
      env: process.env.NODE_ENV,
      url: process.env.NEXT_PUBLIC_APP_URL,
      apiUrl: process.env.NEXT_PUBLIC_API_URL,
    },
    database: {
      url: process.env.DATABASE_URL,
      poolMax: process.env.DATABASE_POOL_MAX 
        ? parseInt(process.env.DATABASE_POOL_MAX, 10) 
        : undefined,
      connectionTimeout: process.env.DATABASE_CONNECTION_TIMEOUT
        ? parseInt(process.env.DATABASE_CONNECTION_TIMEOUT, 10)
        : undefined,
    },
    auth: {
      nextAuthUrl: process.env.NEXTAUTH_URL,
      nextAuthSecret: process.env.NEXTAUTH_SECRET,
    },
    storage: {
      r2AccountId: process.env.R2_ACCOUNT_ID,
      r2AccessKeyId: process.env.R2_ACCESS_KEY_ID,
      r2SecretAccessKey: process.env.R2_SECRET_ACCESS_KEY,
      r2BucketName: process.env.R2_BUCKET_NAME,
      r2PublicUrl: process.env.R2_PUBLIC_URL,
    },
    payments: {
      stripe: {
        publicKey: process.env.STRIPE_PUBLIC_KEY,
        secretKey: process.env.STRIPE_SECRET_KEY,
        webhookSecret: process.env.STRIPE_WEBHOOK_SECRET,
      },
      braintree: process.env.BRAINTREE_MERCHANT_ID ? {
        merchantId: process.env.BRAINTREE_MERCHANT_ID,
        publicKey: process.env.BRAINTREE_PUBLIC_KEY,
        privateKey: process.env.BRAINTREE_PRIVATE_KEY,
        environment: process.env.BRAINTREE_ENVIRONMENT as 'sandbox' | 'production',
      } : undefined,
    },
    email: {
      sendgrid: {
        apiKey: process.env.SENDGRID_API_KEY,
        fromEmail: process.env.SENDGRID_FROM_EMAIL,
        fromName: process.env.SENDGRID_FROM_NAME,
      },
    },
    features: {
      socialLearning: process.env.NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING === 'true',
      aiContent: process.env.NEXT_PUBLIC_FEATURE_AI_CONTENT === 'true',
      midiSupport: process.env.NEXT_PUBLIC_FEATURE_MIDI_SUPPORT !== 'false',
    },
  };

  try {
    return configSchema.parse(rawConfig);
  } catch (error) {
    if (error instanceof z.ZodError) {
      console.error('Configuration validation failed:');
      console.error(JSON.stringify(error.errors, null, 2));
      throw new Error('Invalid configuration. Check environment variables.');
    }
    throw error;
  }
}

// Singleton configuration instance
export const config = loadConfig();
```

### Usage in Application Code

```typescript
// Import typed config instead of accessing process.env directly
import { config } from '@/lib/config';

// Type-safe access with autocomplete
const stripeKey = config.payments.stripe.secretKey;
const isDevelopment = config.app.env === 'development';

// Feature flag checks
if (config.features.socialLearning) {
  // Enable social learning features
}
```

---

## Environment-Specific Configuration

### Development Environment

**Local Configuration File: `.env.local`** (gitignored)

```bash
# Development overrides
NODE_ENV=development
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# Local database
DATABASE_URL=postgresql://postgres:password@localhost:5432/mlc_dev

# Test/sandbox credentials
STRIPE_PUBLIC_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
BRAINTREE_ENVIRONMENT=sandbox

# Development email (capture emails locally)
SENDGRID_API_KEY=SG.test...
```

**Development Setup:**
1. Copy `.env.example` to `.env.local`
2. Fill in test/sandbox credentials
3. Never commit `.env.local` to git
4. Use local Postgres via Docker Compose

### Staging/Preview Environment

**Vercel Preview Deployments:**
- Automatically deployed for pull requests
- Uses "Preview" scoped environment variables
- Connected to staging database
- Uses sandbox/test API credentials

```bash
# Staging configuration (Vercel Dashboard)
NODE_ENV=staging
NEXT_PUBLIC_APP_URL=https://mlc-lms-pr-123.vercel.app
DATABASE_URL=postgresql://...staging-db...
STRIPE_SECRET_KEY=sk_test_...  # Test mode
BRAINTREE_ENVIRONMENT=sandbox
```

### Production Environment

**Vercel Production Deployment:**
- Deployed from `main` branch
- Uses "Production" scoped environment variables
- Connected to production database
- Uses live API credentials

```bash
# Production configuration (Vercel Dashboard)
NODE_ENV=production
NEXT_PUBLIC_APP_URL=https://app.musiclearningcommunity.com
DATABASE_URL=postgresql://...production-db...
STRIPE_SECRET_KEY=sk_live_...  # Live mode
BRAINTREE_ENVIRONMENT=production
```

---

## Secret Rotation

### Rotation Strategy

**Rotation Schedule:**
- **Critical Secrets**: Rotate quarterly (every 90 days)
- **High Secrets**: Rotate semi-annually (every 180 days)
- **Medium Secrets**: Rotate annually
- **Emergency Rotation**: Immediate upon suspected compromise

### Rotation Procedure

**Standard Rotation Process:**

1. **Generate New Secret**
   - Create new secret in provider dashboard
   - Test new secret in staging environment
   - Document new secret in secure location

2. **Update Configuration**
   - Add new secret to Vercel environment variables
   - Deploy updated configuration (no code changes)
   - Verify application health after deployment

3. **Deprecate Old Secret**
   - Monitor for any lingering usage of old secret
   - After verification period (24-48 hours), delete old secret
   - Update internal documentation

4. **Audit and Document**
   - Log rotation event in audit system
   - Update rotation schedule
   - Notify relevant team members

**Example: Rotating Database Password**

```bash
# 1. Generate new password in Neon dashboard
NEW_PASSWORD=<new-secure-password>

# 2. Update DATABASE_URL in Vercel
#    Old: postgresql://user:old_pass@host/db
#    New: postgresql://user:new_pass@host/db

# 3. Redeploy application (Vercel auto-redeploys on env change)
vercel --prod

# 4. Verify health
curl https://app.musiclearningcommunity.com/api/health

# 5. Delete old password from Neon dashboard after 48 hours
```

### Emergency Rotation

**Compromise Response:**

If a secret is suspected to be compromised:

1. **Immediately Rotate**: Generate and deploy new secret within 1 hour
2. **Revoke Old Secret**: Immediately invalidate compromised secret
3. **Audit Access**: Review access logs for unauthorized usage
4. **Notify Stakeholders**: Inform security team and relevant parties
5. **Document Incident**: Create incident report
6. **Review Process**: Identify how compromise occurred

---

## Access Patterns

### Server-Side Access

Secrets are only accessed in server-side code (API routes, getServerSideProps):

```typescript
// pages/api/payments/charge.ts
import { config } from '@/lib/config';
import Stripe from 'stripe';

export default async function handler(req, res) {
  // Secret accessed securely on server-side
  const stripe = new Stripe(config.payments.stripe.secretKey, {
    apiVersion: '2023-10-16',
  });
  
  // Process payment...
}
```

### Client-Side Configuration

Only non-sensitive config exposed to client via `NEXT_PUBLIC_*` prefix:

```typescript
// components/FeatureToggle.tsx
export function FeatureToggle({ children }) {
  // Safe to access in browser
  const socialEnabled = process.env.NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING === 'true';
  
  if (!socialEnabled) return null;
  return <>{children}</>;
}
```

### API Client Configuration

External API clients initialized with secrets at runtime:

```typescript
// lib/clients/stripe.ts
import Stripe from 'stripe';
import { config } from '@/lib/config';

// Singleton client initialized with secret
export const stripeClient = new Stripe(config.payments.stripe.secretKey, {
  apiVersion: '2023-10-16',
  typescript: true,
});

// Usage elsewhere
import { stripeClient } from '@/lib/clients/stripe';
const customer = await stripeClient.customers.create({...});
```

---

## Configuration Documentation

### .env.example Template

Committed to repository as a template:

```bash
# .env.example
# Copy to .env.local and fill in actual values

# Application
NODE_ENV=development
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:3000/api

# Authentication
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=<generate-with-openssl-rand-base64-32>

# Database
DATABASE_URL=postgresql://postgres:password@localhost:5432/mlc_dev

# Object Storage (Cloudflare R2)
R2_ACCOUNT_ID=<your-account-id>
R2_ACCESS_KEY_ID=<your-access-key>
R2_SECRET_ACCESS_KEY=<your-secret-key>
R2_BUCKET_NAME=mlc-assets-dev
R2_PUBLIC_URL=http://localhost:9000

# Payments (use test keys)
STRIPE_PUBLIC_KEY=pk_test_...
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# Email (use sandbox account)
SENDGRID_API_KEY=SG...
SENDGRID_FROM_EMAIL=dev@musiclearningcommunity.com
SENDGRID_FROM_NAME=MLC Dev

# Feature Flags
NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING=false
NEXT_PUBLIC_FEATURE_AI_CONTENT=false
NEXT_PUBLIC_FEATURE_MIDI_SUPPORT=true
```

### Configuration Registry

Maintain a configuration registry document:

| Variable | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `DATABASE_URL` | Secret | Yes | - | PostgreSQL connection string |
| `NEXTAUTH_SECRET` | Secret | Yes | - | JWT signing secret (min 32 chars) |
| `STRIPE_SECRET_KEY` | Secret | Yes | - | Stripe API secret key |
| `R2_BUCKET_NAME` | Config | Yes | - | Cloudflare R2 bucket name |
| `NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING` | Feature Flag | No | `false` | Enable social learning features |

---

## Validation and Startup Checks

### Application Startup Validation

The application validates configuration at startup and refuses to start if critical config is missing:

```typescript
// lib/config/validate.ts
import { config } from './index';

export function validateConfig() {
  const errors: string[] = [];

  // Critical checks
  if (!config.database.url) {
    errors.push('DATABASE_URL is required');
  }

  if (!config.auth.nextAuthSecret || config.auth.nextAuthSecret.length < 32) {
    errors.push('NEXTAUTH_SECRET must be at least 32 characters');
  }

  if (config.app.env === 'production') {
    // Production-specific validations
    if (config.payments.stripe.secretKey.startsWith('sk_test_')) {
      errors.push('Production environment must use live Stripe keys');
    }

    if (!config.app.url.startsWith('https://')) {
      errors.push('Production URL must use HTTPS');
    }
  }

  if (errors.length > 0) {
    console.error('Configuration validation failed:');
    errors.forEach(err => console.error(`  - ${err}`));
    throw new Error('Invalid configuration');
  }

  console.log('✓ Configuration validated successfully');
}

// Call during app initialization
validateConfig();
```

### Health Check Endpoint

Configuration health check endpoint (non-sensitive):

```typescript
// pages/api/health/config.ts
import { config } from '@/lib/config';

export default function handler(req, res) {
  const status = {
    environment: config.app.env,
    features: config.features,
    integrations: {
      database: !!config.database.url,
      storage: !!config.storage.r2BucketName,
      payments: !!config.payments.stripe.secretKey,
      email: !!config.email.sendgrid.apiKey,
    },
    timestamp: new Date().toISOString(),
  };

  res.status(200).json(status);
}
```

---

## Integration-Specific Configuration

### Database Configuration

See [Data Model ERD](data-model-erd.md) for schema details.

```typescript
// lib/db/connection.ts
import { Pool } from 'pg';
import { config } from '@/lib/config';

export const pool = new Pool({
  connectionString: config.database.url,
  max: config.database.poolMax,
  connectionTimeoutMillis: config.database.connectionTimeout,
  ssl: config.app.env === 'production' ? { rejectUnauthorized: false } : undefined,
});
```

### Object Storage Configuration

See [CDN and Media](../17-integrations/cdn-and-media.md) for detailed integration.

```typescript
// lib/storage/r2.ts
import { S3Client } from '@aws-sdk/client-s3';
import { config } from '@/lib/config';

export const r2Client = new S3Client({
  region: 'auto',
  endpoint: `https://${config.storage.r2AccountId}.r2.cloudflarestorage.com`,
  credentials: {
    accessKeyId: config.storage.r2AccessKeyId,
    secretAccessKey: config.storage.r2SecretAccessKey,
  },
});
```

### Payment Provider Configuration

See [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md) for details.

```typescript
// lib/payments/stripe.ts
import Stripe from 'stripe';
import { config } from '@/lib/config';

export const stripe = new Stripe(config.payments.stripe.secretKey, {
  apiVersion: '2023-10-16',
  typescript: true,
  maxNetworkRetries: 2,
});
```

### Email Provider Configuration

See [Email Provider](../17-integrations/email-provider.md) for detailed integration.

```typescript
// lib/email/sendgrid.ts
import sgMail from '@sendgrid/mail';
import { config } from '@/lib/config';

sgMail.setApiKey(config.email.sendgrid.apiKey);

export const sendEmail = async (to: string, subject: string, html: string) => {
  return sgMail.send({
    to,
    from: {
      email: config.email.sendgrid.fromEmail,
      name: config.email.sendgrid.fromName,
    },
    subject,
    html,
  });
};
```

---

## Feature Flags

### Feature Flag Management

Feature flags enable gradual rollout and A/B testing:

```typescript
// lib/features/flags.ts
import { config } from '@/lib/config';

export const features = {
  socialLearning: config.features.socialLearning,
  aiContent: config.features.aiContent,
  midiSupport: config.features.midiSupport,
} as const;

export function isFeatureEnabled(feature: keyof typeof features): boolean {
  return features[feature] === true;
}

// Usage
import { isFeatureEnabled } from '@/lib/features/flags';

if (isFeatureEnabled('socialLearning')) {
  // Show social features
}
```

### Environment-Specific Feature Flags

```bash
# Development: Enable all experimental features
NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING=true
NEXT_PUBLIC_FEATURE_AI_CONTENT=true

# Staging: Test features before production
NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING=true
NEXT_PUBLIC_FEATURE_AI_CONTENT=false

# Production: Only stable features
NEXT_PUBLIC_FEATURE_SOCIAL_LEARNING=false
NEXT_PUBLIC_FEATURE_AI_CONTENT=false
```

---

## Security Best Practices

### Never Log Secrets

```typescript
// BAD: Logs entire config including secrets
console.log('Config:', config);

// GOOD: Log non-sensitive info only
console.log('Environment:', config.app.env);
console.log('Database connected:', !!config.database.url);
```

### Redact Secrets in Error Messages

```typescript
// lib/utils/error.ts
export function sanitizeError(error: Error): Error {
  // Remove secrets from error messages
  let message = error.message;
  
  // Redact common secret patterns
  message = message.replace(/sk_live_\w+/g, 'sk_live_[REDACTED]');
  message = message.replace(/sk_test_\w+/g, 'sk_test_[REDACTED]');
  message = message.replace(/SG\.\w+/g, 'SG.[REDACTED]');
  message = message.replace(/postgresql:\/\/[^@]+@/g, 'postgresql://[REDACTED]@');
  
  return new Error(message);
}
```

### Avoid Secret Exposure in Client Code

```typescript
// BAD: Never expose secrets to client
const MyComponent = () => {
  const apiKey = process.env.STRIPE_SECRET_KEY; // ❌ Exposed to browser!
  return <div>{apiKey}</div>;
};

// GOOD: Keep secrets server-side
// pages/api/payment.ts
export default async function handler(req, res) {
  const apiKey = config.payments.stripe.secretKey; // ✅ Server-only
  // ... use apiKey safely
}
```

---

## Development Workflow

### Local Development Setup

```bash
# 1. Clone repository
git clone https://github.com/mlc/mlc-lms.git
cd mlc-lms

# 2. Copy environment template
cp .env.example .env.local

# 3. Fill in local/test credentials
# Edit .env.local with your test API keys

# 4. Start development server
npm run dev

# 5. Application validates config and starts
# ✓ Configuration validated successfully
# Ready on http://localhost:3000
```

### Adding New Configuration

**Process for adding new config:**

1. **Update Schema**: Add to `lib/config/index.ts` schema
2. **Add to .env.example**: Document new variable
3. **Update Vercel**: Add to staging and production environments
4. **Document**: Update configuration registry
5. **Deploy**: Merge and deploy changes

```typescript
// Example: Adding new monitoring configuration
const configSchema = z.object({
  // ... existing schema
  monitoring: z.object({
    sentryDsn: z.string().url().optional(),
    sentryEnvironment: z.string(),
  }).optional(),
});

// Update .env.example
// SENTRY_DSN=https://...@sentry.io/...
// SENTRY_ENVIRONMENT=production
```

---

## Troubleshooting

### Common Configuration Issues

**Problem**: Application fails to start with "Invalid configuration"
**Solution**: 
- Check console output for specific missing variables
- Verify `.env.local` exists and is populated
- Ensure no typos in variable names

**Problem**: "Database connection failed"
**Solution**:
- Verify `DATABASE_URL` is correct
- Check database is running (local) or accessible (remote)
- Test connection string with psql: `psql $DATABASE_URL`

**Problem**: "Stripe API key invalid"
**Solution**:
- Verify key starts with `sk_test_` (dev) or `sk_live_` (prod)
- Check for whitespace or newlines in key
- Regenerate key in Stripe dashboard if needed

**Problem**: Feature flag not working
**Solution**:
- Ensure variable has `NEXT_PUBLIC_` prefix
- Restart development server after changing .env.local
- Clear browser cache for client-side features

### Configuration Debugging

```typescript
// lib/config/debug.ts
export function debugConfig() {
  if (config.app.env !== 'development') {
    return; // Only debug in development
  }

  console.log('Configuration Debug:');
  console.log('  Environment:', config.app.env);
  console.log('  App URL:', config.app.url);
  console.log('  Database connected:', !!config.database.url);
  console.log('  Stripe configured:', !!config.payments.stripe.secretKey);
  console.log('  Email configured:', !!config.email.sendgrid.apiKey);
  console.log('  Features:', config.features);
}
```

---

## Future Considerations

### Secret Manager Migration

**Current State**: Vercel Environment Variables (sufficient for current scale)

**Future State**: Dedicated secret manager (AWS Secrets Manager, HashiCorp Vault)

**Migration Triggers**:
- Team size > 10 developers requiring granular access control
- Multi-cloud deployment requiring centralized secret management
- Compliance requirements for secret rotation automation
- Need for dynamic secret generation

### Configuration as Code

**Future Enhancement**: Manage configuration via Infrastructure as Code (Terraform, Pulumi)

```hcl
# terraform/environments/production/config.tf
resource "vercel_project_environment_variable" "stripe_secret" {
  project_id = vercel_project.mlc_lms.id
  key        = "STRIPE_SECRET_KEY"
  value      = var.stripe_secret_key
  target     = ["production"]
  sensitive  = true
}
```

### Multi-Tenant Configuration

**Future Consideration**: Support multi-tenant configuration per subscriber

```typescript
// Future: Tenant-specific overrides
interface TenantConfig {
  tenantId: string;
  branding: {
    primaryColor: string;
    logoUrl: string;
  };
  features: {
    socialLearning: boolean;
    customDomain: string;
  };
}
```

---

## Supporting Documents Referenced

This configuration and secrets management specification draws from the following source documents:

- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator configuration management tools and environment variable management interfaces
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional admin tools for feature flag management and configuration control
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription tier configurations and plan-specific feature flag settings
- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Pricing tier specifications informing tier-based configuration and feature availability settings

---

## Related Documentation

- **Security Implementation**: [Security and Privacy](security-privacy.md)
- **System Architecture**: [System Overview](system-overview.md)
- **Database Connection**: [Data Model ERD](data-model-erd.md)
- **External Integrations**: [Integrations Overview](../17-integrations/)
- **Deployment Process**: [Release Management](../19-quality-and-operations/release-management.md)
- **Feature Flags**: [Feature Flags](../06-system-admin/sysadmin-feature-flags.md)
