# Security and Privacy Architecture

## Purpose

This document defines the technical security architecture and privacy engineering practices for the MLC LMS platform. It establishes defense-in-depth security layers, data protection mechanisms, secure development practices, and privacy-by-design principles that protect user data and ensure system integrity.

## Scope

**Included:**
- Security architecture layers and defense-in-depth strategy
- Data protection (encryption, classification, minimization)
- Application security controls (XSS, CSRF, injection prevention)
- Infrastructure security (network, TLS, certificates, WAF)
- Vulnerability management and security testing
- Security monitoring, logging, and alerting
- Incident response and breach procedures
- Privacy engineering (anonymization, retention, erasure)
- Third-party security and vendor management
- Secure development lifecycle practices

**Excluded:**
- Authentication flows and session management (see [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md))
- User impersonation and support access (see [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md))
- Legal privacy policies and terms (see [Privacy Policy](../23-legal-and-compliance/privacy-policy.md))
- Compliance frameworks (see [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md), [GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md))
- Rate limiting and abuse prevention (see [Rate Limiting and Abuse](rate-limiting-and-abuse.md))
- Configuration and secrets management (see [Config and Secrets](config-and-secrets.md))
- API design conventions (see [API Design Principles](api-design-principles.md))

---

## Security Architecture

### Defense-in-Depth Strategy

The MLC LMS implements multiple layers of security controls to ensure comprehensive protection:

```
┌─────────────────────────────────────────────────────────────────────┐
│                      SECURITY LAYER MODEL                            │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 7: User & Application Security                                │
│  - User authentication (MFA, password policies)                      │
│  - Role-based access control (RBAC)                                  │
│  - Session management and timeouts                                   │
│  - Security awareness training                                       │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 6: Application Security                                       │
│  - Input validation & sanitization                                   │
│  - XSS/CSRF protection                                               │
│  - SQL injection prevention (parameterized queries)                  │
│  - Secure coding standards                                           │
│  - Security headers (CSP, HSTS, X-Frame-Options)                     │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 5: Data Security                                              │
│  - Encryption at rest (database, storage)                            │
│  - Encryption in transit (TLS 1.3)                                   │
│  - PII data classification and handling                              │
│  - Data minimization and retention policies                          │
│  - Secure key management                                             │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 4: API & Service Security                                     │
│  - API authentication (JWT tokens)                                   │
│  - API authorization and scoping                                     │
│  - Rate limiting and throttling                                      │
│  - Request validation and type safety                                │
│  - API security testing                                              │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 3: Network Security                                           │
│  - Web Application Firewall (WAF)                                    │
│  - DDoS protection (Cloudflare)                                      │
│  - Network isolation and segmentation                                │
│  - Secure database connections (connection pooling)                  │
│  - IP allowlisting for admin functions                               │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 2: Platform & Infrastructure Security                         │
│  - Vercel platform security controls                                 │
│  - Serverless function isolation                                     │
│  - Environment variable protection                                   │
│  - Automated security patching                                       │
│  - Infrastructure as Code (IaC) security                             │
├─────────────────────────────────────────────────────────────────────┤
│ Layer 1: Physical & Vendor Security                                 │
│  - Cloud provider security (Vercel, Neon, Cloudflare)               │
│  - SOC 2 / ISO 27001 certified vendors                               │
│  - Physical data center security                                     │
│  - Vendor security assessments                                       │
│  - Supply chain security                                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Security Principles

1. **Zero Trust**: Never trust, always verify - authenticate and authorize every request
2. **Least Privilege**: Users and services have minimum necessary permissions
3. **Separation of Duties**: Critical operations require multiple roles/approvals
4. **Defense in Depth**: Multiple overlapping security controls at each layer
5. **Privacy by Design**: Privacy controls built into architecture from the start
6. **Secure by Default**: Systems configured securely out of the box
7. **Fail Securely**: System failures default to secure/locked state
8. **Security as Code**: Security controls defined, versioned, and automated

---

## Data Protection

### Data Classification

All data is classified by sensitivity level to determine appropriate security controls:

| Classification | Definition | Examples | Security Requirements |
|----------------|------------|----------|----------------------|
| **Public** | Information intended for public access | Marketing content, public help docs | Standard encryption in transit |
| **Internal** | Non-sensitive business data | System configuration, non-PII metadata | Encryption in transit, access control |
| **Confidential** | Sensitive business data | Subscription details, usage analytics | Encryption at rest and in transit, audit logging |
| **Restricted** | Regulated/protected data | Student PII (COPPA), educational records (FERPA), payment data (PCI) | Strong encryption, access controls, audit logging, data minimization |

### Encryption Standards

#### Encryption at Rest

**Database Encryption (Neon PostgreSQL):**
- **Algorithm**: AES-256-GCM
- **Scope**: All database tables and backups
- **Key Management**: Neon-managed encryption keys with automatic rotation
- **Column-level Encryption**: Additional encryption for highly sensitive fields (payment tokens, SSNs if stored)

**Object Storage Encryption (Cloudflare R2):**
- **Algorithm**: AES-256
- **Scope**: All stored objects (game assets, audio files, user uploads)
- **Key Management**: Cloudflare-managed keys
- **Access Control**: Pre-signed URLs with time-limited access

**Application-Level Encryption:**
- **Sensitive Fields**: Additional encryption layer for PII fields (email, phone, address)
- **Library**: Node.js `crypto` module with `aes-256-gcm`
- **Key Storage**: Environment variables in Vercel (encrypted at rest)
- **Key Rotation**: Annual rotation schedule with backward compatibility

#### Encryption in Transit

**TLS Configuration:**
- **Protocol**: TLS 1.3 (minimum TLS 1.2)
- **Cipher Suites**: Modern ciphers only (no weak/deprecated ciphers)
- **Certificate Authority**: Let's Encrypt (auto-renewed by Vercel/Cloudflare)
- **HSTS**: Strict-Transport-Security header with 1-year max-age
- **Certificate Pinning**: Not implemented (conflicts with auto-renewal)

**Internal Service Communication:**
- **Database Connections**: TLS-encrypted connections to Neon PostgreSQL
- **API Requests**: HTTPS for all external service calls (Braintree, email provider)
- **CDN to Origin**: TLS 1.3 between Cloudflare CDN and Vercel origin

### Data Minimization

Following privacy-by-design principles, we collect and retain only necessary data:

**Collection Principles:**
- **Purpose Limitation**: Collect data only for specific, legitimate purposes
- **Minimal Data**: Collect minimum data needed to fulfill purpose
- **User Consent**: Obtain explicit consent for data collection (especially for students)
- **Transparency**: Clearly communicate what data is collected and why

**Student Data (Special Protection):**
- **No Behavioral Tracking**: No third-party analytics or advertising trackers on student pages
- **Anonymous Student Option**: Generic Student accounts with no PII collection
- **Parental Consent**: Verifiable parental consent for students under 13 (COPPA)
- **Educational Purpose Only**: Student data used exclusively for educational purposes

**Data Retention:**
- **Active Users**: Data retained while subscription is active
- **Inactive Subscriptions**: Data retained for 13 months after cancellation
- **Deletion Schedule**: Automated deletion of expired data via background jobs
- **Backup Retention**: Encrypted backups retained for 90 days, then permanently deleted

### Data Anonymization

**Anonymization Techniques:**
- **De-identification**: Remove direct identifiers (names, emails) from analytics data
- **Pseudonymization**: Replace identifiers with pseudonyms for internal analytics
- **Aggregation**: Report data at aggregate level (school/district statistics)
- **K-Anonymity**: Ensure individuals cannot be re-identified from aggregated data

**Implementation:**
- **Analytics Events**: Strip PII before sending to analytics systems
- **Reporting**: Generate reports with anonymized/aggregated data only
- **Research Data**: Apply strong anonymization for any research use
- **Testing Data**: Use synthetic/anonymized data in development/staging environments

---

## Application Security

### Input Validation and Sanitization

**Server-Side Validation (Primary Defense):**
- **Validation Library**: Zod for TypeScript-first schema validation
- **Validation Scope**: All API route inputs (body, query params, path params)
- **Type Safety**: Strict TypeScript types generated from Zod schemas
- **Rejection Policy**: Invalid inputs rejected with 400 Bad Request

**Validation Rules:**
```typescript
// Example validation schema
const createStudentSchema = z.object({
  username: z.string().min(3).max(50).regex(/^[a-zA-Z0-9_-]+$/),
  email: z.string().email().optional(),
  firstName: z.string().min(1).max(100),
  lastName: z.string().min(1).max(100),
  dateOfBirth: z.string().datetime().optional(),
  parentalConsent: z.boolean()
});
```

**Sanitization:**
- **HTML Sanitization**: Use DOMPurify for any user-generated HTML content
- **SQL Injection Prevention**: Parameterized queries via Prisma ORM (no raw SQL)
- **Command Injection Prevention**: No shell command execution with user input
- **Path Traversal Prevention**: Validate file paths, use allowlists for file access

**Client-Side Validation (UX Enhancement):**
- **Immediate Feedback**: Client-side validation for better UX
- **Same Rules**: Client validation mirrors server validation (shared schemas)
- **Not a Security Control**: Always validate on server regardless of client validation

### Cross-Site Scripting (XSS) Protection

**Automatic Protections:**
- **React Escaping**: React automatically escapes JSX expressions
- **Content Security Policy**: Strict CSP headers to prevent inline scripts
- **DOM-based XSS**: Avoid dangerous patterns like `dangerouslySetInnerHTML`

**Content Security Policy (CSP):**
```http
Content-Security-Policy: 
  default-src 'self';
  script-src 'self' 'unsafe-inline' https://trusted-cdn.com;
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self' data:;
  connect-src 'self' https://api.musiclearningcommunity.com;
  media-src 'self' https://media.musiclearningcommunity.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
```

**User-Generated Content:**
- **Rich Text**: Use sanitized Markdown (marked + DOMPurify) instead of raw HTML
- **Game Titles/Descriptions**: Plain text only, HTML-escaped on render
- **Student Names**: Plain text, validated against character allowlist
- **No User Uploads of Executable Content**: Images/audio only, validated by MIME type

### Cross-Site Request Forgery (CSRF) Protection

**Token-Based Protection:**
- **CSRF Tokens**: NextAuth.js automatically provides CSRF protection
- **Double Submit Cookie**: CSRF token in cookie and request header
- **Validation**: Server validates token on all state-changing operations (POST, PUT, PATCH, DELETE)

**Same-Site Cookie Protection:**
```http
Set-Cookie: sessionToken=...; SameSite=Lax; Secure; HttpOnly
```

**Additional Safeguards:**
- **Origin Header Check**: Validate `Origin` and `Referer` headers for state-changing requests
- **Custom Headers**: Require custom header (e.g., `X-Requested-With: XMLHttpRequest`) for API calls
- **GET Requests Safe**: Ensure GET requests never modify state

### Security Headers

**Comprehensive Security Headers:**
```http
# Prevent clickjacking
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'none'

# XSS protection (legacy browsers)
X-XSS-Protection: 1; mode=block

# Prevent MIME sniffing
X-Content-Type-Options: nosniff

# Referrer policy
Referrer-Policy: strict-origin-when-cross-origin

# HTTPS enforcement
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload

# Permissions policy (restrict browser features)
Permissions-Policy: geolocation=(), microphone=(), camera=()
```

**Next.js Configuration:**
```typescript
// next.config.js
const securityHeaders = [
  { key: 'X-Frame-Options', value: 'DENY' },
  { key: 'X-Content-Type-Options', value: 'nosniff' },
  { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
  { key: 'Strict-Transport-Security', value: 'max-age=31536000; includeSubDomains' }
];
```

---

## Infrastructure Security

### Network Security

**Cloudflare WAF (Web Application Firewall):**
- **Managed Rulesets**: Enable Cloudflare Managed Ruleset + OWASP Core Ruleset
- **Custom Rules**: Block suspicious patterns (SQL injection, XSS attempts, scanner traffic)
- **Rate Limiting**: IP-based rate limiting for login, signup, API endpoints
- **Bot Detection**: Challenge/block malicious bots, allow legitimate crawlers
- **GeoBlocking**: Optional blocking of high-risk countries (configurable by admin)

**DDoS Protection:**
- **Cloudflare DDoS Protection**: Automatic DDoS mitigation at edge
- **Vercel Platform**: Built-in DDoS protection for serverless functions
- **Rate Limiting**: Application-level rate limiting as secondary defense

**Database Security:**
- **Network Isolation**: Neon database not publicly accessible (connection pooling via private endpoint)
- **IP Allowlisting**: Optional IP allowlist for database access (admin operations)
- **Connection Pooling**: Secure connection pooling prevents connection exhaustion attacks
- **Automatic SSL/TLS**: All database connections encrypted

### TLS/HTTPS Configuration

**Certificate Management:**
- **Automatic Provisioning**: Vercel automatically provisions Let's Encrypt certificates
- **Auto-Renewal**: Certificates auto-renewed 30 days before expiry
- **Wildcard Certificates**: Support for subdomains (*.musiclearningcommunity.com)
- **Certificate Monitoring**: Alerts if certificate renewal fails

**HTTPS Enforcement:**
- **Redirect HTTP to HTTPS**: Automatic redirect for all HTTP requests
- **HSTS Preloading**: Submit domain to HSTS preload list (Chrome, Firefox, Safari)
- **Secure Cookies**: All cookies marked `Secure` (HTTPS-only)

---

## Vulnerability Management

### Dependency Security

**Automated Scanning:**
- **npm audit**: Run `npm audit` on every build (fail build on high/critical vulnerabilities)
- **Dependabot**: GitHub Dependabot enabled for automatic dependency updates
- **Snyk**: Optional integration for advanced vulnerability scanning
- **License Compliance**: Scan for incompatible open-source licenses

**Update Policy:**
- **Critical Vulnerabilities**: Patch within 24 hours
- **High Vulnerabilities**: Patch within 7 days
- **Medium Vulnerabilities**: Patch within 30 days
- **Low Vulnerabilities**: Patch in next scheduled maintenance window

**Dependency Management:**
- **Lock Files**: Commit `package-lock.json` to ensure reproducible builds
- **Minimal Dependencies**: Audit and remove unnecessary dependencies
- **Trusted Sources**: Only use packages from npm registry, verify package integrity
- **Version Pinning**: Pin major versions, allow minor/patch updates

### Security Testing

**Automated Testing:**
- **Static Analysis**: ESLint with security plugins (`eslint-plugin-security`)
- **Code Scanning**: GitHub Advanced Security / CodeQL for vulnerability detection
- **Secret Scanning**: GitHub Secret Scanning to prevent credential leaks
- **Container Scanning**: Scan Docker images (if used) for vulnerabilities

**Manual Testing:**
- **Code Review**: Security-focused code review for high-risk changes
- **Penetration Testing**: Annual third-party penetration testing
- **Vulnerability Disclosure**: Responsible disclosure program for security researchers
- **Bug Bounty**: Consider bug bounty program post-MVP (HackerOne, Bugcrowd)

### Security Updates

**Patching Schedule:**
- **Security Patches**: Applied within SLA based on severity
- **Platform Updates**: Vercel/Next.js updates applied quarterly (or as needed)
- **Database Updates**: Neon manages PostgreSQL updates automatically
- **Infrastructure Updates**: Cloudflare/R2 managed by vendor

**Emergency Patches:**
- **Critical Vulnerabilities**: Emergency patch procedure with expedited testing
- **Zero-Day Exploits**: Immediate mitigation (WAF rules, access restrictions) while patch is developed
- **Vendor Advisories**: Monitor security advisories from all vendors

---

## Security Monitoring and Logging

### Security Event Logging

**Logged Security Events:**
- **Authentication**: Login attempts (success/failure), MFA challenges, password resets, account lockouts
- **Authorization**: Permission denials, unauthorized access attempts, privilege escalations
- **Data Access**: Access to sensitive data (student PII, payment data)
- **Data Modification**: Creation, update, deletion of critical records
- **Administrative Actions**: User creation/deletion, permission changes, system configuration changes
- **Impersonation**: All impersonation sessions (start/end), actions taken while impersonated
- **API Access**: Failed API requests, rate limit violations, suspicious patterns

**Log Structure:**
```typescript
interface SecurityLogEntry {
  timestamp: string;        // ISO 8601 timestamp
  eventType: string;        // e.g., "auth.login.failure"
  severity: "low" | "medium" | "high" | "critical";
  userId?: string;          // User involved (if applicable)
  actorId?: string;         // Actor performing action (for impersonation)
  ipAddress: string;        // Source IP
  userAgent: string;        // Browser/client info
  resource?: string;        // Resource accessed
  action?: string;          // Action attempted
  result: "success" | "failure" | "blocked";
  details: Record<string, any>; // Additional context
  requestId: string;        // Trace request across logs
}
```

**Log Storage:**
- **Short-term**: Vercel logs (7-day retention on Pro plan)
- **Long-term**: External log aggregation service (e.g., Datadog, Logtail, Better Stack)
- **Security Logs**: Minimum 1-year retention for audit/compliance
- **PII in Logs**: Redact PII from logs (emails, names, etc.)

### Anomaly Detection

**Automated Detection:**
- **Failed Login Attempts**: Alert on multiple failed logins from same IP or user
- **Unusual Access Patterns**: Detect access from new locations, unusual times, abnormal data access
- **Privilege Escalation**: Alert on permission changes, role modifications
- **Data Exfiltration**: Monitor for large data exports, unusual API usage
- **Brute Force Attacks**: Detect password guessing, credential stuffing

**Alerting Rules:**
- **Immediate Alerts** (Critical): Account compromise, data breach, privilege escalation
- **Hourly Digest** (High): Multiple failed logins, suspicious access patterns
- **Daily Digest** (Medium): Unusual activity, warning-level events
- **Weekly Report** (Low): General security metrics, audit log summaries

### Security Dashboards

**System Admin Security Dashboard:**
- **Recent Security Events**: Last 100 security events with filtering
- **Active Sessions**: Real-time view of active user sessions
- **Failed Login Attempts**: Chart of failed logins by hour/day
- **Blocked Requests**: WAF blocks, rate limit violations
- **Impersonation Activity**: All impersonation sessions (active and recent)
- **Anomaly Alerts**: Security alerts requiring investigation

---

## Incident Response

### Security Incident Types

| Type | Definition | Examples | Response Time |
|------|------------|----------|---------------|
| **P1 - Critical** | Active security breach or data exposure | Database breach, credential leak, active attack | Immediate (< 1 hour) |
| **P2 - High** | Potential security compromise | Suspected account takeover, vulnerability exploitation attempt | < 4 hours |
| **P3 - Medium** | Security weakness detected | Non-critical vulnerability discovered, configuration issue | < 24 hours |
| **P4 - Low** | Security concern or question | Security policy question, low-risk finding | < 5 business days |

### Incident Response Procedure

**Detection Phase:**
1. **Alert Received**: Security monitoring alert or user report
2. **Initial Triage**: Assess severity, determine incident type
3. **Escalation**: Notify incident response team based on severity

**Containment Phase:**
4. **Isolate Affected Systems**: Disable compromised accounts, block malicious IPs, restrict access
5. **Preserve Evidence**: Capture logs, system snapshots for forensic analysis
6. **Stop Attack**: Implement immediate mitigations (WAF rules, rate limits, access restrictions)

**Investigation Phase:**
7. **Root Cause Analysis**: Determine how breach occurred, what vulnerabilities were exploited
8. **Impact Assessment**: Identify affected users, compromised data, extent of breach
9. **Evidence Collection**: Gather logs, screenshots, system state for analysis

**Remediation Phase:**
10. **Fix Vulnerabilities**: Patch exploited vulnerabilities, update configurations
11. **Strengthen Controls**: Add additional security controls to prevent recurrence
12. **Restore Services**: Return systems to normal operation once secured

**Communication Phase:**
13. **Internal Communication**: Update stakeholders on incident status and resolution
14. **User Notification**: Notify affected users if PII was compromised (required by law)
15. **Regulatory Notification**: Report breach to authorities if required (FERPA, GDPR)
16. **Public Disclosure**: Transparency report if appropriate

**Recovery Phase:**
17. **Monitor for Recurrence**: Enhanced monitoring for similar attacks
18. **Post-Incident Review**: Document lessons learned, update procedures
19. **Security Improvements**: Implement preventive measures based on findings

### Breach Notification Requirements

**Legal Requirements:**
- **FERPA**: Notify affected students/parents if educational records compromised
- **COPPA**: Notify FTC and parents if data of children under 13 compromised
- **GDPR**: Notify supervisory authority within 72 hours if EU users affected
- **State Laws**: Comply with state breach notification laws (varies by state)

**Notification Timeline:**
- **Initial Assessment**: Determine if breach meets notification threshold (< 24 hours)
- **Regulatory Notification**: File required reports with authorities (< 72 hours)
- **User Notification**: Notify affected users via email (< 7 days)
- **Public Disclosure**: Optional transparency report after investigation complete

**Notification Content:**
- **What Happened**: Clear description of the incident
- **What Data Was Affected**: Specific data types compromised
- **What We're Doing**: Steps taken to address the breach
- **What You Should Do**: Recommended actions for affected users
- **How to Contact Us**: Support contact for questions

---

## Privacy Engineering

### Privacy by Design

**Core Privacy Principles:**
1. **Proactive not Reactive**: Build privacy into system design, not as add-on
2. **Privacy as Default**: Systems configured for maximum privacy out of the box
3. **Privacy Embedded**: Privacy controls integrated into architecture
4. **Full Functionality**: Privacy without sacrificing functionality (positive-sum)
5. **End-to-End Security**: Protect data throughout entire lifecycle
6. **Visibility and Transparency**: Users understand what data is collected and why
7. **Respect for Privacy**: User-centric design with privacy as priority

**Implementation:**
- **Data Minimization**: Collect only necessary data (e.g., generic students have no PII)
- **Purpose Limitation**: Use data only for stated educational purposes
- **Consent Management**: Clear consent flows for data collection
- **User Control**: Users can view, export, delete their data
- **Transparency**: Privacy policy in plain language, clear data practices

### Data Subject Rights

**GDPR Rights (for EU users):**
- **Right to Access**: Users can request copy of their data
- **Right to Rectification**: Users can correct inaccurate data
- **Right to Erasure**: Users can request deletion of their data ("right to be forgotten")
- **Right to Portability**: Users can export their data in machine-readable format
- **Right to Object**: Users can object to certain data processing
- **Right to Restrict**: Users can request temporary restriction of processing

**Implementation:**
- **Data Export**: Self-service export feature for users (JSON format)
- **Data Deletion**: Automated deletion workflow with confirmation
- **Account Dashboard**: User-facing dashboard to view/manage personal data
- **Retention Schedule**: Automated deletion based on retention policies
- **Parent Rights**: Parents can exercise rights on behalf of children under 13 (COPPA)

### Retention and Deletion

**Retention Periods:**
- **Active Subscriptions**: Data retained as long as subscription is active
- **Cancelled Subscriptions**: Data retained for 13 months post-cancellation
- **Support Data**: Support tickets and logs retained for 3 years
- **Security Logs**: Security/audit logs retained for 1 year (minimum)
- **Backups**: Encrypted backups retained for 90 days, then deleted

**Automated Deletion:**
- **Background Jobs**: Daily job to identify and delete expired data
- **Soft Delete**: Initial soft delete (marked as deleted, hidden from users)
- **Hard Delete**: Permanent deletion 30 days after soft delete (grace period)
- **Cascading Deletion**: Delete related records (assignments, scores, sessions)
- **Backup Cleanup**: Remove deleted users from backup rotation

**Manual Deletion (Right to Erasure):**
- **User Request**: User submits deletion request via dashboard or support
- **Identity Verification**: Verify user identity before processing request
- **Deletion Exceptions**: Retain data if legally required (e.g., financial records for tax purposes)
- **Confirmation**: Email confirmation once deletion is complete
- **Timeline**: Complete deletion within 30 days of request

---

## Third-Party Security

### Vendor Security Assessment

**Vendor Selection Criteria:**
- **Certifications**: SOC 2 Type II, ISO 27001, PCI DSS (for payment processors)
- **Security Documentation**: Published security whitepapers, compliance reports
- **Data Processing Agreement**: Sign DPA with clear data handling terms
- **Incident Response**: Vendor has clear incident response and breach notification procedures
- **Data Residency**: Understand where data is stored and processed
- **Right to Audit**: Contract includes right to audit vendor security practices

**Current Vendors:**
- **Vercel** (Hosting): SOC 2 Type II certified, ISO 27001 certified
- **Neon** (Database): SOC 2 Type II certified, PostgreSQL with encryption
- **Cloudflare** (CDN/Security): SOC 2, ISO 27001, GDPR compliant
- **Braintree** (Payments): PCI DSS Level 1 certified (PayPal subsidiary)
- **Email Provider** (TBD): Must be SOC 2 certified, support DKIM/SPF/DMARC

### Integration Security

**API Security:**
- **API Keys**: Store API keys in Vercel environment variables (encrypted)
- **Key Rotation**: Rotate API keys annually or immediately if compromised
- **Least Privilege**: API keys scoped to minimum necessary permissions
- **No Hardcoded Secrets**: Never commit secrets to Git repository

**Webhook Security:**
- **Signature Verification**: Verify webhook signatures (HMAC) from external services
- **IP Allowlisting**: Restrict webhooks to known IPs when possible
- **Replay Protection**: Validate webhook timestamp, reject old requests
- **Idempotency**: Handle duplicate webhooks gracefully

**Third-Party Scripts:**
- **Subresource Integrity**: Use SRI hashes for third-party scripts
- **CSP**: Allowlist third-party script sources in Content Security Policy
- **Minimal Scripts**: Avoid unnecessary third-party scripts (analytics, tracking)
- **Student Pages**: No third-party scripts on student-facing pages (privacy protection)

### Supply Chain Security

**Dependency Management:**
- **Trusted Sources**: Only use packages from npm registry
- **Package Verification**: Verify package integrity via checksums
- **Maintainer Reputation**: Prefer packages from trusted maintainers with active communities
- **Typosquatting Protection**: Carefully verify package names

**Build Pipeline Security:**
- **Secure CI/CD**: GitHub Actions with minimal permissions
- **Build Reproducibility**: Deterministic builds with locked dependencies
- **Artifact Signing**: Sign build artifacts (future consideration)
- **Access Control**: Limit who can merge to production branches

---

## Secure Development Lifecycle

### Secure Coding Standards

**Code Review Guidelines:**
- **Security Checklist**: Review for common vulnerabilities (OWASP Top 10)
- **Input Validation**: Verify all inputs are validated server-side
- **Authentication/Authorization**: Confirm proper access controls
- **Sensitive Data**: Check for PII handling, encryption, secure storage
- **Logging**: Ensure no PII in logs, proper error handling

**Prohibited Practices:**
- **No Raw SQL**: Use Prisma ORM, avoid raw SQL queries
- **No eval()**: Never use `eval()` or `Function()` constructor with user input
- **No Dangerous HTML**: Avoid `dangerouslySetInnerHTML` without sanitization
- **No Secrets in Code**: Never commit API keys, passwords, tokens to Git

**Recommended Libraries:**
- **Validation**: Zod for schema validation
- **Sanitization**: DOMPurify for HTML sanitization
- **Encryption**: Node.js `crypto` module (not custom crypto)
- **JWT**: `jsonwebtoken` or NextAuth.js for token handling
- **Password Hashing**: bcrypt or Argon2 (never plain text or MD5/SHA1)

### Security Training

**Developer Training:**
- **Onboarding**: Security training for new developers (OWASP Top 10, secure coding)
- **Annual Training**: Yearly refresher on security best practices
- **Incident Learnings**: Share learnings from security incidents
- **Secure Code Examples**: Provide secure code snippets and patterns

**Security Awareness:**
- **Phishing Awareness**: Training to recognize phishing attempts
- **Password Security**: Use password manager, unique passwords, MFA
- **Social Engineering**: Awareness of social engineering tactics
- **Reporting**: Clear process to report security concerns

---

## Telemetry

### Security Events

Tracked via [Event Model](../15-analytics-and-reporting/event-model.md):

- `security.vulnerability.detected` - Security vulnerability discovered
- `security.vulnerability.patched` - Vulnerability patched/remediated
- `security.incident.reported` - Security incident reported
- `security.incident.resolved` - Security incident resolved
- `security.audit.started` - Security audit or review started
- `security.audit.completed` - Security audit completed
- `security.breach.detected` - Data breach detected
- `security.breach.contained` - Breach contained
- `security.test.completed` - Security test (pentest, scan) completed
- `privacy.data.exported` - User data export requested/completed
- `privacy.data.deleted` - User data deletion requested/completed
- `privacy.consent.given` - User consent provided
- `privacy.consent.withdrawn` - User consent withdrawn

**Event Properties:**
- `severity`: low | medium | high | critical
- `incident_type`: vulnerability | breach | misconfiguration | attack
- `affected_users_count`: Number of users impacted
- `data_types_affected`: Types of data compromised
- `resolution_time_hours`: Time to resolve incident
- `root_cause`: Brief description of root cause

---

## Permissions

Security and privacy management is restricted to System Admin role:

- **View Security Logs**: System Admin only
- **Manage Security Settings**: System Admin only
- **Review Security Incidents**: System Admin only
- **Access Audit Logs**: System Admin only
- **Configure Security Controls**: System Admin only

Users have rights to their own data:
- **View Personal Data**: All users can view their own data
- **Export Personal Data**: All users can export their own data
- **Delete Account**: All users can request account deletion
- **Parent/Guardian Rights**: Parents can exercise rights on behalf of children under 13

See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for complete permission specifications.

---

## Supporting Documents Referenced

This security and privacy architecture specification draws from the following source documents:

- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Comprehensive role specifications informing access control, permissions model, and security boundaries for each user type
- [Blayzer note USER ROLES.docx.txt](../00-ORIG-CONTEXT/Blayzer%20note%20USER%20ROLES.docx.txt) — Additional role-based access control requirements and security considerations for role hierarchies
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator security tools, audit logging, and impersonation access controls
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional system admin security features including security monitoring and incident management
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription and seat management specifications informing data retention policies, PII handling for billing, and subscriber data protection

---

## Dependencies

### Internal Dependencies
- [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md) - Authentication and session security
- [Impersonation and Support Access](../02-roles-and-permissions/impersonation-and-support-access.md) - Support access controls
- [Rate Limiting and Abuse](rate-limiting-and-abuse.md) - API rate limiting and abuse prevention
- [Config and Secrets](config-and-secrets.md) - Secret management and configuration
- [API Design Principles](api-design-principles.md) - API security patterns
- [Caching Strategy](caching-strategy.md) - Cache security considerations
- [Webhooks](webhooks.md) - Webhook security
- [Data Model ERD](data-model-erd.md) - Data model and sensitive fields
- [Privacy Policy](../23-legal-and-compliance/privacy-policy.md) - Legal privacy policy
- [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md) - Compliance requirements

### External Dependencies
- **Vercel Platform Security**: Platform-level security controls and DDoS protection
- **Cloudflare WAF**: Web application firewall and attack mitigation
- **Neon Security**: Database encryption, backups, and access controls
- **Braintree Security**: PCI-compliant payment processing
- **Email Provider Security**: Secure email delivery and DKIM/SPF/DMARC
- **Secret Scanning**: GitHub Secret Scanning for credential leak detection
- **Dependency Scanning**: Dependabot and npm audit for vulnerability detection

### Monitoring/Logging Services (Future)
- **Log Aggregation**: Datadog, Logtail, or Better Stack for centralized logging
- **Security Monitoring**: SIEM solution for security event correlation
- **Uptime Monitoring**: StatusPage or similar for service availability
- **Error Tracking**: Sentry or similar for application error monitoring

---

## Open Questions

### Advanced Security Features
- Should we implement certificate pinning for mobile apps (if developed)?
- What level of security analytics should we provide to admins?
- Should we implement automated threat intelligence feeds?
- Do we need a dedicated security information and event management (SIEM) system?

### Privacy Enhancements
- Should we offer additional privacy options (e.g., fully anonymous student accounts)?
- How should we handle international data transfer (EU to US)?
- Should we implement differential privacy for aggregate reporting?
- What additional transparency features would benefit users?

### Compliance and Auditing
- Should we pursue SOC 2 Type II certification?
- What additional compliance frameworks should we target (ISO 27001, NIST)?
- How frequently should we conduct third-party penetration testing?
- Should we establish a bug bounty program?

### Incident Response
- Do we need a dedicated security operations center (SOC)?
- Should we establish relationships with external security forensics firms?
- What level of cyber insurance should we carry?
- How should we handle coordinated disclosure with security researchers?
