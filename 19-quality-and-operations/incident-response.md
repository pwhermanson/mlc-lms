# Incident Response

## Purpose

This document defines the incident response procedures and protocols for the MusicLearningCommunity (MLC) LMS platform. It establishes:

- **Incident Classification**: Severity levels and incident types to ensure appropriate response
- **Response Procedures**: Step-by-step actions for detecting, triaging, resolving, and recovering from production incidents
- **Roles and Responsibilities**: Clear ownership and escalation paths during incidents
- **Communication Protocols**: Standardized notification procedures for stakeholders
- **Post-Incident Activities**: Learning and improvement processes following incidents
- **Incident Documentation**: Templates and requirements for incident reporting

Given MLC's educational mission and the critical importance of score recording accuracy, incident response procedures must minimize learning disruption while maintaining data integrity and system availability.

## Scope

### Included

- **Incident Classification**: Severity levels, incident types, and response SLAs
- **Detection and Alerting**: How incidents are identified and escalated
- **Response Procedures**: Step-by-step incident handling workflows
- **Incident Roles**: On-call engineers, incident commanders, communication coordinators
- **Communication Templates**: Status page updates, stakeholder notifications, user communications
- **Incident Documentation**: Incident report templates and requirements
- **Post-Mortem Process**: Blameless retrospectives and action item tracking
- **Security Incident Procedures**: Special handling for security/privacy breaches
- **Escalation Paths**: When and how to escalate to leadership or external vendors

### Excluded

- **Monitoring Configuration**: Alerting rules and observability setup (covered in [Observability](observability.md))
- **Rollback Procedures**: Technical steps to revert releases (covered in [Release Management](release-management.md))
- **Security Architecture**: Defense-in-depth and security controls (covered in [Security and Privacy](../18-architecture/security-privacy.md))
- **Performance Testing**: Load testing and performance benchmarks (covered in [Test Strategy](test-strategy.md))
- **Normal Release Issues**: Problems detected before production deployment (covered in [Release Management](release-management.md))

---

## Incident Classification

### Severity Levels

Incident severity determines response urgency, escalation requirements, and communication scope.

#### Severity 1: Critical (Immediate Response Required)

**Definition**: Complete system outage or critical functionality failure affecting all or most users. Immediate data loss risk.

**Response SLA**: 
- **Detection to Acknowledgment**: 5 minutes
- **Acknowledgment to Mitigation**: 30 minutes
- **Mitigation to Resolution**: 4 hours

**Examples**:
- **Score Recording Failures**: Students unable to complete games or scores not saving
- **Authentication/Login Failures**: Users unable to log in or access the platform
- **Payment Processing Errors**: Subscribers unable to purchase or renew subscriptions
- **Database Corruption**: Data integrity issues or data loss detected
- **Complete Site Outage**: Platform unavailable to all users
- **Security Breach**: Active data breach, unauthorized access detected

**Required Actions**:
- Immediate page to on-call engineer (24/7)
- Incident Commander assigned within 10 minutes
- Executive team notified within 30 minutes
- Status page updated: "Major Outage"
- All hands on deck - pull in additional engineers as needed
- Hourly updates to stakeholders until resolved

**Communication Channels**:
- ✅ Status page: Major incident banner
- ✅ Email: All subscribers (via [Email Provider](../17-integrations/email-provider.md))
- ✅ In-app notification: Banner on all pages
- ✅ Slack/Teams: `#incidents-critical` channel
- ✅ SMS: On-call rotation and executive team

#### Severity 2: Major (Urgent Response Required)

**Definition**: Significant degradation of core functionality affecting many users or single critical feature failure. No immediate data loss.

**Response SLA**:
- **Detection to Acknowledgment**: 15 minutes
- **Acknowledgment to Mitigation**: 2 hours
- **Mitigation to Resolution**: 24 hours

**Examples**:
- **Assignment Generation Failures**: Teachers unable to create assignments for students
- **Game Loading Issues**: Specific games or game categories failing to load
- **Report Generation Errors**: Analytics and progress reports not generating
- **Email Delivery Failures**: System emails not sending ([Email Provider](../17-integrations/email-provider.md) outage)
- **Performance Degradation**: API response times >2x baseline (p95 > 1000ms, see [Caching Strategy](../18-architecture/caching-strategy.md))
- **Payment Webhook Failures**: Subscription updates not processing correctly
- **Critical Accessibility Failure**: WCAG AA compliance broken for core features (see [A11y Standards](../01-ux-design-system/a11y-standards.md))

**Required Actions**:
- Page to on-call engineer (business hours) or escalate to secondary (after hours)
- Incident Commander assigned within 30 minutes
- Product team notified within 1 hour
- Status page updated: "Service Degradation"
- Regular updates every 4 hours until resolved

**Communication Channels**:
- ✅ Status page: Degraded service notice
- ✅ In-app notification: Top banner (affected users only)
- ✅ Slack/Teams: `#incidents-major` channel
- ✅ Email: Affected subscribers (if incident persists >4 hours)

#### Severity 3: Moderate (Standard Response)

**Definition**: Non-critical feature failure or isolated user issues. Workaround available. No data loss.

**Response SLA**:
- **Detection to Acknowledgment**: 1 hour (business hours only)
- **Acknowledgment to Mitigation**: 8 hours
- **Mitigation to Resolution**: 72 hours (3 business days)

**Examples**:
- **Minor Game Bugs**: Single game or minor game feature not working correctly
- **UI Display Issues**: Visual bugs or layout issues (not blocking functionality)
- **Search Inconsistencies**: Search results incomplete or incorrect
- **Notification Delays**: In-app notifications delayed by 15+ minutes
- **Non-Critical API Errors**: Sporadic 5xx errors on non-essential endpoints (<1% error rate)
- **Profile Update Failures**: Users unable to update profile fields (non-blocking)
- **Leaderboard Calculation Errors**: Leaderboard rankings incorrect but not affecting game play

**Required Actions**:
- Assigned to on-call engineer during business hours
- No immediate escalation required
- Issue tracked in incident management system
- Updates every 24 hours until resolved
- May be addressed in next scheduled release

**Communication Channels**:
- ✅ Slack/Teams: `#incidents` channel
- ✅ In-app notification: Optional (if widely affecting users)
- ❌ No status page update
- ❌ No proactive email notification

#### Severity 4: Minor (Low Priority)

**Definition**: Cosmetic issues, minor bugs with minimal user impact, or issues affecting <1% of users.

**Response SLA**:
- **Detection to Acknowledgment**: Next business day
- **Resolution**: Best effort (may be deferred to future release)

**Examples**:
- **Cosmetic UI Issues**: Minor visual inconsistencies, color mismatches
- **Documentation Errors**: Help text or tooltips incorrect
- **Low-Traffic Feature Bugs**: Issues in rarely-used features
- **Browser-Specific Quirks**: Minor issues on unsupported browsers (see [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md))
- **Typos**: Copy errors in UI text

**Required Actions**:
- Added to backlog for triage
- No immediate response required
- May be batched with other fixes in future release

**Communication Channels**:
- ✅ Issue tracker only (GitHub/Jira)
- ❌ No incident notification

---

## Incident Types

### System Outages
**Description**: Platform unavailable or inaccessible to users

**Common Causes**:
- Hosting provider outage (Vercel, see [System Overview](../18-architecture/system-overview.md))
- Database connection failures (Neon PostgreSQL)
- CDN failures (Cloudflare, see [CDN and Media](../17-integrations/cdn-and-media.md))
- DDoS attack or traffic spike exceeding rate limits (see [Rate Limiting](../18-architecture/rate-limiting-and-abuse.md))
- Configuration errors in deployment
- SSL/TLS certificate expiration

**Typical Severity**: Severity 1 (if complete outage) or Severity 2 (if partial)

**Response Priority**: Immediate - restore service first, investigate root cause second

### Data Integrity Issues
**Description**: Data corruption, loss, or inconsistency detected

**Common Causes**:
- Database migration errors
- Bug in data processing logic
- Race conditions in concurrent updates
- Failed transaction rollbacks
- Backup restoration errors

**Typical Severity**: Severity 1 (if active data loss) or Severity 2 (if historical data affected)

**Response Priority**: Immediate - stop the bleeding, prevent further data loss, then restore data integrity

**Special Considerations**: 
- **Score Recording**: Any score data loss is Severity 1 due to educational impact
- **Subscription Data**: Payment/billing data issues require immediate financial reconciliation
- **User Accounts**: Account data loss may require password resets and identity verification

### Performance Degradation
**Description**: System responding slowly or intermittently timing out

**Common Causes**:
- Database query performance issues (missing indexes, slow queries)
- Cache invalidation storms
- Memory leaks in application code
- Third-party API latency (payment provider, email service)
- Increased traffic without auto-scaling
- CDN cache misses

**Typical Severity**: Severity 2 (if p95 response time >2x baseline) or Severity 3 (if intermittent)

**Response Priority**: Identify bottleneck, implement cache improvements or database optimization (see [Caching Strategy](../18-architecture/caching-strategy.md))

**Metrics to Monitor** (see [Observability](observability.md)):
- API response times (p50, p95, p99)
- Database query latency
- Cache hit rates
- Error rates by endpoint

### Security Incidents
**Description**: Unauthorized access, data breach, or security vulnerability exploitation

**Common Causes**:
- Credential compromise (password breach, leaked API keys)
- Exploitation of known vulnerability
- Social engineering or phishing attack
- Insider threat
- Third-party vendor breach
- DDoS attack

**Typical Severity**: Severity 1 (active breach) or Severity 2 (vulnerability discovered)

**Response Priority**: Contain threat, preserve evidence, assess scope, notify affected users

**Special Procedures**:
- **Do Not**: Delete logs or evidence
- **Do**: Preserve all logs, take database snapshots, document timeline
- **Legal Notification**: May require breach notification under COPPA, FERPA, GDPR (see [Legal Compliance](../23-legal-and-compliance/coppa-ferpa-notes.md))
- **Forensics**: Engage security consultant if breach confirmed
- **Password Resets**: May require forced password reset for all users

**Escalation**: 
- Immediately notify CEO and legal counsel
- Contact security consultant (if retained)
- Prepare breach notification (if required by law)

### Third-Party Service Failures
**Description**: External dependency outage or degraded performance

**Common Causes**:
- Payment processor outage (Braintree, see [Billing Provider Integration](../14-payments-and-subscriptions/billing-provider-integration.md))
- Email service outage (see [Email Provider](../17-integrations/email-provider.md))
- CDN outage (Cloudflare R2, see [CDN and Media](../17-integrations/cdn-and-media.md))
- Database provider outage (Neon)
- Authentication service issues (NextAuth.js dependencies)

**Typical Severity**: Depends on affected service (Severity 1-3)

**Response Priority**: 
1. Verify third-party status page
2. Implement fallback or graceful degradation if available
3. Communicate expected resolution timeline to users
4. Monitor vendor status and restore service when available

**Communication**: 
- Clearly state that issue is with third-party provider
- Link to vendor status page if public
- Provide workaround if available (e.g., "Payment processing temporarily unavailable. Please try again in 1 hour.")

### Deployment Failures
**Description**: Production deployment causes errors or unexpected behavior

**Common Causes**:
- Untested code path activated in production
- Database migration errors
- Environment configuration mismatch
- Dependency version conflicts
- Cache invalidation issues after deploy

**Typical Severity**: Severity 1-2 (depending on impact)

**Response Priority**: Rollback to previous stable version (see [Release Management](release-management.md))

**Prevention**: 
- Follow deployment checklist (see [Release Management](release-management.md))
- Require staging validation before production
- Implement feature flags for gradual rollout (see [Feature Flags](../06-system-admin/sysadmin-feature-flags.md))
- Monitor error rates and response times for 1 hour post-deploy

---

## Incident Roles and Responsibilities

### On-Call Engineer (Primary Responder)

**Responsibilities**:
- Monitor alerts and incident notifications 24/7 (Severity 1) or business hours (Severity 2-3)
- Acknowledge incidents within SLA timeframe
- Perform initial triage and severity assessment
- Begin immediate mitigation actions
- Escalate to Incident Commander if Severity 1 or unable to resolve within 30 minutes
- Document incident timeline and actions taken

**Rotation**:
- **Weekday Rotation**: Primary on-call (Mon-Fri, 9am-5pm local time)
- **After-Hours Rotation**: Secondary on-call (Mon-Fri, 5pm-9am; Weekends)
- **Rotation Schedule**: Published 2 weeks in advance
- **Handoff**: Daily standup handoff with incident summary

**Tools Required**:
- Access to production monitoring dashboards (see [Observability](observability.md))
- Production database read access (write access requires incident commander approval)
- Deployment and rollback permissions
- Incident management system access
- Communication tools (Slack/Teams, status page admin)

**Escalation Triggers**:
- Unable to identify root cause within 30 minutes
- Incident requires rollback or emergency deployment
- Incident affects >50% of users
- Data loss or corruption detected
- Security incident suspected

### Incident Commander (Severity 1 & 2)

**Responsibilities**:
- Assume overall command of incident response
- Coordinate response team and assign tasks
- Make decisions about rollback, emergency deployments, or vendor escalation
- Approve high-risk actions (database writes, DNS changes, emergency deployments)
- Ensure regular stakeholder updates
- Determine when incident is resolved
- Schedule and facilitate post-mortem

**Assignment**:
- **Severity 1**: Senior engineer or tech lead (assigned within 10 minutes)
- **Severity 2**: On-call engineer may self-assign or escalate to senior engineer

**Authority**:
- Override normal change management processes
- Pull in additional engineers as needed
- Authorize emergency hotfix deployments
- Approve emergency spending (e.g., scaling up infrastructure)

### Communication Coordinator (Severity 1 Only)

**Responsibilities**:
- Update status page with current incident status
- Send stakeholder notifications (email, in-app, SMS)
- Draft user-facing communication (clear, non-technical language)
- Field support inquiries related to incident
- Coordinate with marketing/PR if media attention expected

**Assignment**:
- Product manager, customer success lead, or designated communications person
- Assigned within 30 minutes of Severity 1 incident declaration

**Communication Guidelines**:
- **Be Transparent**: Acknowledge the issue honestly
- **Be Timely**: Update every hour even if no progress ("We're still investigating...")
- **Be Clear**: Use non-technical language
- **Be Realistic**: Don't promise specific timelines unless confident
- **Be Apologetic**: Acknowledge impact on users

### Engineering Support Team (As Needed)

**Responsibilities**:
- Assist incident commander with investigation and remediation
- Provide subject matter expertise (database, API, frontend, specific features)
- Execute mitigation tasks assigned by incident commander
- Document technical details and root cause analysis

**Assignment**:
- Pulled in by incident commander based on affected system
- Expected to respond within 30 minutes if paged

---

## Incident Response Procedures

### Phase 1: Detection and Triage (0-15 minutes)

#### 1.1 Incident Detection

**Automatic Detection** (see [Observability](observability.md)):
- Monitoring alerts trigger (error rate spike, response time degradation)
- Health check failures detected
- Third-party status webhooks fire (e.g., Vercel, Neon, Cloudflare)

**Manual Detection**:
- User reports via support ticket or social media
- Internal team member discovers issue
- Scheduled testing identifies problem

**Actions**:
- Alert sent to on-call engineer (PagerDuty, Opsgenie, or equivalent)
- Incident automatically created in incident management system
- Initial severity assigned based on alert type

#### 1.2 Acknowledgment

**On-Call Engineer Actions** (within SLA):
- Acknowledge alert in monitoring system (stops repeat pages)
- Open incident management record
- Post initial message in `#incidents` Slack channel: "Investigating [brief description]"
- Begin preliminary investigation

**Acknowledgment Template**:
```
🚨 INCIDENT ACKNOWLEDGED
Incident ID: INC-YYYY-MM-DD-NNN
Severity: [1/2/3/4]
Affected System: [e.g., Score Recording, Authentication, Game Player]
Responder: @engineer-name
Status: Investigating
Next Update: [timestamp, +30min]
```

#### 1.3 Severity Assessment and Escalation

**Assessment Checklist**:
- [ ] How many users are affected? (1 user vs. 10% vs. 100%)
- [ ] What functionality is impacted? (critical vs. nice-to-have)
- [ ] Is there active data loss occurring?
- [ ] Is there a workaround available?
- [ ] What is the business impact? (revenue loss, regulatory violation, reputational damage)

**Escalation Decision Matrix**:

| Condition | Action |
|-----------|--------|
| ≥50% users affected + core functionality | Escalate to Severity 1 |
| Data loss detected | Escalate to Severity 1 |
| Security breach suspected | Escalate to Severity 1 |
| Unable to identify cause in 30min | Assign Incident Commander |
| Issue resolved in <15min | Document and close (no escalation needed) |

**Escalation Actions**:
- Update incident severity in incident management system
- Page Incident Commander (if Severity 1 or 2)
- Update Slack channel with severity escalation
- Begin status page update (Severity 1 or 2)

### Phase 2: Investigation and Mitigation (15min - 4 hours)

#### 2.1 Assemble Response Team

**Incident Commander Actions**:
- Review incident details and initial investigation
- Identify required subject matter experts (SMEs)
- Page additional engineers if needed
- Establish incident response Slack channel (e.g., `#incident-2025-01-15-001`)
- Assign Communication Coordinator (Severity 1 only)

**Response Team Kickoff** (Severity 1 only):
- Brief sync call within 15 minutes of Incident Commander assignment
- Incident Commander shares known information
- Assign investigation tasks to team members
- Set next check-in time (30 minutes)

#### 2.2 Investigate Root Cause

**Investigation Checklist**:
- [ ] Review monitoring dashboards (error rates, response times, throughput)
- [ ] Check application logs for error patterns
- [ ] Review recent deployments (last 24 hours)
- [ ] Check third-party service status pages
- [ ] Verify database connectivity and performance
- [ ] Review rate limiting and abuse logs (see [Rate Limiting](../18-architecture/rate-limiting-and-abuse.md))
- [ ] Check CDN status and cache hit rates
- [ ] Review background job queues for failures

**Common Investigation Tools**:
- **Monitoring Dashboards**: See [Observability](observability.md)
- **Log Aggregation**: Application logs, API logs, database logs
- **Error Tracking**: Error stack traces and affected users
- **Database Queries**: Slow query logs, active connections
- **Performance Profiling**: CPU, memory, network usage

**Documentation During Investigation**:
- **Incident Timeline**: Record all observations and actions with timestamps
- **Hypotheses Tested**: Document what you tried and the results
- **Data Points**: Error counts, affected user IDs, error messages, stack traces

#### 2.3 Implement Mitigation

**Mitigation Priority**:
1. **Stop the Bleeding**: Prevent further data loss or user impact
2. **Restore Service**: Get core functionality working (even if degraded)
3. **Full Resolution**: Fix root cause completely

**Mitigation Options** (in order of preference):

**Option A: Rollback** (preferred for deployment-related incidents)
- Follow rollback procedures in [Release Management](release-management.md)
- Requires Incident Commander approval
- Typically resolves issue within 5-15 minutes
- Safe and well-tested process

**Option B: Hotfix Forward** (for bugs in stable code)
- Create emergency PR with fix
- Requires senior engineer code review (expedited)
- Deploy via hotfix process (see [Release Management](release-management.md))
- Higher risk than rollback, but necessary if rollback not viable

**Option C: Configuration Change** (for config-related issues)
- Update environment variables (see [Config and Secrets](../18-architecture/config-and-secrets.md))
- Enable/disable feature flags (see [Feature Flags](../06-system-admin/sysadmin-feature-flags.md))
- Adjust rate limits or caching rules
- Requires Incident Commander approval

**Option D: Vendor Escalation** (for third-party issues)
- Contact vendor support (use priority support line if available)
- Monitor vendor status page for updates
- Implement graceful degradation if possible
- Communicate expected timeline to users

**Option E: Infrastructure Scaling** (for capacity issues)
- Scale up database connections
- Increase serverless function concurrency
- Clear cache and restart services
- May incur additional costs (requires Incident Commander approval)

**Mitigation Approval and Documentation**:
```
🔧 MITIGATION PROPOSED
Incident ID: INC-YYYY-MM-DD-NNN
Proposed Action: [Rollback to v1.2.3 / Deploy hotfix / Enable feature flag]
Risk Level: [Low/Medium/High]
Expected Impact: [Restore score recording / Fix authentication]
Rollback Plan: [If this fails, we will...]
Approval: [Awaiting Incident Commander approval]
```

#### 2.4 Verify Mitigation Success

**Verification Checklist**:
- [ ] Error rate returned to baseline (<1%)
- [ ] Response times within SLA (p95 <500ms API, <2s pages)
- [ ] Affected functionality tested manually
- [ ] User reports of issue stopped
- [ ] Monitoring confirms system stable for 15+ minutes
- [ ] No new related alerts triggered

**Partial Mitigation**:
- If mitigation only partially resolves issue, update incident status to "Mitigated" but keep incident open
- Continue investigating root cause
- Plan for full resolution in next deployment cycle

### Phase 3: Communication (Ongoing)

#### 3.1 Status Page Updates

**Severity 1 Communication Cadence**:
- **0-15 min**: Initial status page update: "Investigating"
- **15-60 min**: Update every 30 minutes with progress
- **60+ min**: Update every 60 minutes until resolved
- **Resolution**: Final update with "Resolved" status and brief explanation

**Status Page Template - Investigating**:
```
🔴 MAJOR OUTAGE - Score Recording Unavailable

Status: Investigating
Last Updated: [timestamp]

We are currently investigating an issue where students are unable to complete games and save scores. Our engineering team is actively working on a resolution.

We will provide an update within 30 minutes.

We apologize for the disruption to your learning.
```

**Status Page Template - Mitigated**:
```
🟡 SERVICE DEGRADED - Score Recording Restored

Status: Monitoring
Last Updated: [timestamp]

The issue with score recording has been resolved. Students can now complete games and scores are saving correctly. We are continuing to monitor the system to ensure stability.

Thank you for your patience.
```

**Status Page Template - Resolved**:
```
🟢 RESOLVED - Score Recording Fully Operational

Status: Resolved
Last Updated: [timestamp]

The score recording issue has been fully resolved. All games are functioning normally and scores are being recorded accurately.

Root Cause: [Brief non-technical explanation]
Resolution: [Brief description of fix]

We will be publishing a detailed post-mortem within 48 hours.

We apologize for the disruption and thank you for your patience.
```

#### 3.2 Stakeholder Notifications

**Internal Stakeholders** (Severity 1 & 2):
- **Engineering Team**: `#incidents` Slack channel (real-time updates)
- **Product Team**: `#product` Slack mention + email summary
- **Executive Team**: Email summary within 30 minutes (Severity 1) or 2 hours (Severity 2)
- **Customer Success**: Email summary + FAQ for responding to user inquiries

**External Stakeholders** (Severity 1 only):
- **All Subscribers**: Email notification if incident persists >1 hour
- **Affected Users**: In-app notification banner
- **Social Media**: Twitter/Facebook post if incident generates social media attention

**Email Template - Subscriber Notification**:
```
Subject: [MLC] Service Disruption - Score Recording [Resolved/In Progress]

Dear Music Learning Community Member,

We experienced a technical issue today between [start time] and [end time] that prevented students from completing games and saving scores. 

[If resolved]:
The issue has been resolved and all systems are operating normally. Any games played during the outage will need to be replayed to record scores.

[If in progress]:
Our engineering team is actively working on a resolution. We will send another update within [timeframe].

We sincerely apologize for any disruption to your teaching and learning. If you have any questions or concerns, please contact support@musiclearningcommunity.com.

Sincerely,
The MLC Team

Technical Details (Optional):
[Brief non-technical explanation of root cause and resolution]
```

#### 3.3 Support Team Coordination

**Support Response Plan**:
- Communication Coordinator drafts FAQ for support team
- Support team monitors ticket volume and common questions
- Support team redirects users to status page for updates
- Support team flags any user reports of new issues (may indicate incomplete mitigation)

**Support FAQ Template**:
```
Q: Why can't I save my scores?
A: We're experiencing a technical issue with score recording. Our engineers are working on a fix. Please check status.musiclearningcommunity.com for updates.

Q: Will I lose my progress?
A: No, your account data and historical scores are safe. You may need to replay any games completed during the outage.

Q: When will this be fixed?
A: We're working as quickly as possible. Check status.musiclearningcommunity.com for the latest estimated resolution time.

Q: Can I get a refund for this downtime?
A: Please contact billing@musiclearningcommunity.com if you'd like to discuss your subscription.
```

### Phase 4: Resolution and Recovery (4-24 hours)

#### 4.1 Confirm Full Resolution

**Resolution Criteria**:
- [ ] Root cause identified and understood
- [ ] Permanent fix deployed (or mitigation verified stable)
- [ ] All affected functionality tested and confirmed working
- [ ] Error rates and response times at baseline for 2+ hours
- [ ] No user reports of ongoing issues
- [ ] Monitoring alerts cleared
- [ ] Incident Commander declares incident resolved

**Resolution Checklist**:
```
✅ INCIDENT RESOLVED
Incident ID: INC-YYYY-MM-DD-NNN
Severity: [1/2/3/4]
Duration: [X hours Y minutes]
Affected Users: [count or percentage]
Root Cause: [Brief technical description]
Resolution: [What was done to fix it]
Preventive Measures: [How we'll prevent recurrence]
Post-Mortem Scheduled: [date/time]
Status: Closed
```

#### 4.2 Post-Incident Communication

**Final Status Page Update**:
- Update status to "Resolved" with green status
- Include brief root cause and resolution summary
- Thank users for patience
- Link to detailed post-mortem (once published)

**Subscriber Email** (Severity 1 only):
- Send resolution email to all subscribers within 4 hours of resolution
- Acknowledge disruption and apologize
- Explain what happened (non-technical terms)
- Describe what was done to fix it
- Outline preventive measures
- Offer support contact for any questions

**Internal Debrief** (Severity 1 & 2):
- Incident Commander sends summary email to engineering and product teams
- Highlights key decisions and actions taken
- Acknowledges team members who contributed
- Schedules post-mortem meeting (within 48 hours for Severity 1, within 1 week for Severity 2)

#### 4.3 Data Reconciliation (If Applicable)

**For Data Integrity Incidents**:
- [ ] Identify scope of affected data (user IDs, date range, data types)
- [ ] Determine if data can be recovered (from backups, logs, or recalculation)
- [ ] Create data recovery plan and get Incident Commander approval
- [ ] Execute data recovery (preferably in off-peak hours)
- [ ] Verify data integrity after recovery
- [ ] Notify affected users if unable to recover data

**Score Recording Incident Example**:
1. Identify students who played games during incident window
2. Check logs to see if score data was recorded but not displayed
3. Reprocess scores from event logs if possible
4. If scores lost, notify affected teachers and students
5. Offer to reset affected assignments or provide manual credit

---

## Incident Documentation

### Incident Report Template

Every Severity 1 and 2 incident requires a formal incident report created during the incident and updated throughout resolution.

**Incident Report Fields**:

```markdown
# Incident Report: INC-YYYY-MM-DD-NNN

## Metadata
- **Incident ID**: INC-YYYY-MM-DD-NNN
- **Severity**: [1/2/3/4]
- **Status**: [Investigating/Mitigated/Resolved/Closed]
- **Detected**: [timestamp]
- **Acknowledged**: [timestamp]
- **Mitigated**: [timestamp]
- **Resolved**: [timestamp]
- **Duration**: [X hours Y minutes]
- **Affected Users**: [count/percentage]
- **On-Call Engineer**: @engineer-name
- **Incident Commander**: @commander-name
- **Communication Coordinator**: @coordinator-name

## Summary
[1-2 sentence description of the incident]

## User Impact
- **Affected Functionality**: [e.g., Score recording, authentication, game loading]
- **Affected Users**: [e.g., All students, subscribers in US region, users of Chrome browser]
- **Impact Description**: [What users experienced]
- **Workaround**: [If available]

## Timeline
All times in UTC.

- `HH:MM` - [Event description]
- `HH:MM` - [Action taken]
- `HH:MM` - [Decision made]
- `HH:MM` - [Mitigation deployed]
- `HH:MM` - [Resolution confirmed]

## Root Cause
[Technical description of what caused the incident]

## Contributing Factors
[What made this incident possible or worse]
- Missing monitoring for [system]
- Lack of automated rollback
- Insufficient testing of [scenario]

## Resolution
[What was done to fix the incident]

## Preventive Measures
[What we'll do to prevent this from happening again]
- [ ] Action item 1 - Assigned: @owner - Due: [date]
- [ ] Action item 2 - Assigned: @owner - Due: [date]

## Lessons Learned
[What went well, what went poorly, what we learned]

## Related Links
- Incident Slack Channel: [link]
- Monitoring Dashboards: [link]
- Post-Mortem: [link - once published]
- Related PRs: [links]
```

### Incident Timeline Documentation

**Real-Time Timeline** (during incident):
- Document every significant event, decision, and action with timestamp
- Record hypotheses tested and results
- Capture error messages, stack traces, user IDs, and other data points
- Document who made key decisions and why

**Timeline Example**:
```
14:23 UTC - Monitoring alert: Score recording API error rate spike to 45%
14:24 UTC - @engineer acknowledged alert, began investigation
14:26 UTC - Checked logs: database connection timeout errors
14:28 UTC - Checked Neon status page: No reported issues
14:30 UTC - Verified other API endpoints responding normally
14:32 UTC - Escalated to Severity 1: 500+ users affected, core functionality down
14:33 UTC - Paged @incident-commander
14:35 UTC - @incident-commander joined, assumed command
14:36 UTC - Updated status page: "Investigating score recording issue"
14:38 UTC - Hypothesis: Recent deployment (v2.5.3) may have introduced bug
14:40 UTC - Decision: Rollback to v2.5.2
14:41 UTC - Rollback initiated
14:46 UTC - Rollback completed, monitoring for recovery
14:48 UTC - Error rate dropped to 2%
14:50 UTC - Manual testing: Score recording working correctly
14:52 UTC - Updated status page: "Issue mitigated, monitoring stability"
15:05 UTC - Error rate at baseline (<1%) for 15 minutes
15:06 UTC - Declared incident resolved
15:10 UTC - Updated status page: "Resolved"
15:30 UTC - Post-incident review scheduled for 2025-01-16 10:00am
```

---

## Post-Mortem Process

### Blameless Post-Mortems

**Philosophy**: Focus on systems and processes, not individuals. Assume everyone acted with good intentions and made the best decisions with the information available at the time.

**Goals**:
- Understand root cause and contributing factors
- Identify preventive measures
- Improve incident response process
- Share learnings across the team

**Anti-Goals**:
- Assigning blame or finding a scapegoat
- Punishing individuals for mistakes
- Covering up or minimizing the incident

### Post-Mortem Meeting

**Scheduling**:
- **Severity 1**: Within 48 hours of resolution
- **Severity 2**: Within 1 week of resolution
- **Severity 3**: Optional (or batched monthly)

**Attendees**:
- Incident Commander (facilitator)
- On-call engineer and all response team members
- Engineering leadership
- Product manager (if user-facing impact)
- Optional: Other engineers interested in learning

**Duration**: 60 minutes

**Agenda**:
1. **Review Incident** (10 min)
   - Incident Commander summarizes incident, impact, and resolution
   - Review incident timeline
   
2. **Root Cause Analysis** (15 min)
   - Technical deep dive into root cause
   - Discuss contributing factors
   - Use "5 Whys" technique to get to underlying issues
   
3. **What Went Well** (10 min)
   - Celebrate effective actions and good decisions
   - Acknowledge team members who contributed
   - Identify processes that worked well
   
4. **What Went Poorly** (15 min)
   - Identify gaps in monitoring, alerting, or processes
   - Discuss confusion or miscommunication
   - Surface bottlenecks or delays in response
   
5. **Action Items** (10 min)
   - Brainstorm preventive measures
   - Prioritize action items (quick wins vs. long-term improvements)
   - Assign owners and due dates
   - Identify action items that should be added to product roadmap

### Post-Mortem Document

**Post-Mortem Template**:

```markdown
# Post-Mortem: [Brief Incident Description]

**Date**: YYYY-MM-DD
**Authors**: @incident-commander, @on-call-engineer
**Status**: Draft / Final
**Severity**: [1/2/3]

---

## Executive Summary
[2-3 sentences summarizing the incident, impact, and resolution for executive audience]

## Incident Overview
- **Detection**: [How incident was detected]
- **Duration**: [X hours Y minutes]
- **Severity**: [1/2/3/4]
- **Affected Users**: [count/percentage]
- **Affected Systems**: [List of affected systems/features]
- **Resolution**: [One-sentence description of fix]

## Timeline
[Detailed timeline with timestamps - copy from incident report]

## Root Cause Analysis

### Primary Root Cause
[Technical description of the underlying cause]

### Contributing Factors
1. [Factor 1 - e.g., Missing test coverage for edge case]
2. [Factor 2 - e.g., Monitoring alert not configured for this scenario]
3. [Factor 3 - e.g., Deployment happened during peak traffic hours]

### Why This Happened (5 Whys)
1. **Why did scores fail to save?** Because the database connection pool was exhausted.
2. **Why was the connection pool exhausted?** Because a new feature created N+1 query problem.
3. **Why didn't we catch the N+1 problem?** Because load testing wasn't performed before deployment.
4. **Why wasn't load testing performed?** Because it's not part of our standard release checklist.
5. **Why isn't it part of the checklist?** Because we haven't prioritized performance testing tools.

## Impact Assessment

### User Impact
- **Students**: Unable to complete games and save scores for 45 minutes
- **Teachers**: Unable to view student progress reports during incident
- **Subscribers**: No impact on billing or account management

### Business Impact
- **Revenue**: No direct revenue impact (no failed payments)
- **Reputation**: ~50 support tickets, some negative social media mentions
- **Trust**: Moderate impact on user trust, mitigated by transparent communication

### Data Impact
- **Data Loss**: ~200 game sessions not recorded (users will need to replay)
- **Data Corruption**: None
- **Data Recovery**: Not possible (game state not preserved)

## Resolution and Recovery

### Immediate Mitigation
[What was done to stop the bleeding]

### Permanent Fix
[What was done to fully resolve the issue]

### Verification
[How we confirmed the fix worked and won't regress]

## What Went Well 👍

1. **Fast Detection**: Monitoring alert fired within 1 minute of incident start
2. **Quick Escalation**: On-call engineer correctly escalated to Severity 1
3. **Effective Rollback**: Rollback executed smoothly within 5 minutes
4. **Clear Communication**: Status page updated regularly, support team well-informed
5. **Team Collaboration**: Engineering team responded quickly and worked well together

## What Went Poorly 👎

1. **Missed in Testing**: N+1 query problem not caught in staging environment
2. **Delayed Diagnosis**: Took 15 minutes to identify database connection issue
3. **No Automated Rollback**: Required manual decision and execution
4. **Alert Fatigue**: Database connection alerts had been firing intermittently (false positives)
5. **Incomplete Runbook**: No documented procedure for this specific incident type

## Action Items

### Immediate (This Week)
- [ ] Add database connection pool monitoring dashboard - @db-engineer - Due: YYYY-MM-DD
- [ ] Update release checklist to require load testing - @release-manager - Due: YYYY-MM-DD
- [ ] Document rollback procedures in runbook - @incident-commander - Due: YYYY-MM-DD

### Short-Term (This Month)
- [ ] Implement automated N+1 query detection in CI - @backend-engineer - Due: YYYY-MM-DD
- [ ] Tune database connection alerts to reduce false positives - @sre-engineer - Due: YYYY-MM-DD
- [ ] Add automated rollback trigger for >10% error rate - @devops-engineer - Due: YYYY-MM-DD

### Long-Term (This Quarter)
- [ ] Invest in comprehensive performance testing suite - @qa-lead - Due: YYYY-MM-DD
- [ ] Implement canary deployments with automated rollback - @platform-team - Due: YYYY-MM-DD
- [ ] Add database query performance monitoring to all endpoints - @backend-team - Due: YYYY-MM-DD

## Lessons Learned

1. **Testing Gap**: Our staging environment doesn't have production-scale data, so performance issues only surface in production
2. **Monitoring Improvement**: Need better visibility into database connection pool utilization
3. **Alert Tuning**: Too many false positive alerts lead to alert fatigue and delayed response
4. **Automation**: Manual rollback decisions add 10-15 minutes to incident resolution
5. **Communication**: Transparent communication and regular updates helped maintain user trust

## Recommendations

1. **Invest in Performance Testing**: Allocate engineering time to build load testing suite
2. **Staging Environment**: Populate staging with production-scale data (anonymized)
3. **Automated Rollback**: Implement automatic rollback on critical error thresholds
4. **Alert Hygiene**: Dedicate time each sprint to tuning alerts and reducing false positives
5. **Runbook Maintenance**: Keep incident runbooks updated with learnings from each incident

---

**Acknowledgments**: Thanks to @engineer1, @engineer2, @engineer3 for their quick response and effective collaboration during this incident.
```

### Post-Mortem Sharing

**Internal Sharing**:
- Post final post-mortem in `#engineering` Slack channel
- Present key learnings at next all-hands or engineering meeting
- Archive post-mortem in shared documentation (wiki, Notion, etc.)

**External Sharing** (Optional):
- For major incidents (Severity 1 with >1 hour downtime), consider publishing public post-mortem
- Remove sensitive technical details (database credentials, internal tool names, etc.)
- Focus on high-level architecture, decision-making, and learnings
- Demonstrates transparency and builds trust with users

---

## Security Incident Procedures

Security incidents require special handling due to legal, regulatory, and reputational implications.

### Security Incident Classification

**Security Incident Types**:
- **Data Breach**: Unauthorized access to user data (PII, scores, account information)
- **Credential Compromise**: User passwords, API keys, or secrets exposed
- **Vulnerability Exploitation**: Attacker successfully exploits security flaw
- **DDoS Attack**: Malicious traffic overwhelming system
- **Insider Threat**: Unauthorized access by employee or contractor
- **Third-Party Breach**: Vendor compromise affecting MLC data

### Security Incident Response

**Immediate Actions** (first 30 minutes):
1. **Contain the Threat**
   - Disable compromised accounts or API keys
   - Block malicious IP addresses (see [Rate Limiting](../18-architecture/rate-limiting-and-abuse.md))
   - Revoke compromised credentials
   - Isolate affected systems if necessary

2. **Preserve Evidence**
   - **DO NOT**: Delete logs, terminate servers, or destroy evidence
   - Take snapshots of databases and filesystems
   - Export relevant logs before they rotate
   - Document all actions taken with timestamps

3. **Assess Scope**
   - Identify what data was accessed or exfiltrated
   - Determine how attacker gained access
   - Check for persistence mechanisms (backdoors, malicious code)
   - Estimate number of affected users

4. **Escalate to Leadership**
   - Immediately notify CEO and legal counsel
   - Brief executive team on initial findings
   - Prepare for potential breach notification (see below)

**Investigation and Recovery** (hours to days):
1. **Forensic Analysis**
   - Review access logs, authentication logs, API logs
   - Identify attack vector and timeline
   - Assess extent of data exfiltration
   - Check for lateral movement within systems

2. **Remediation**
   - Patch exploited vulnerability
   - Force password reset for affected users (if credential breach)
   - Rotate all API keys and secrets (see [Config and Secrets](../18-architecture/config-and-secrets.md))
   - Deploy security updates

3. **Recovery**
   - Restore affected systems from clean backups (if necessary)
   - Verify no backdoors or persistence mechanisms remain
   - Resume normal operations once security confirmed

### Breach Notification Requirements

**Legal Obligations**:
- **COPPA** (Children's Online Privacy Protection Act): Notify FTC if children's data breached
- **FERPA** (Family Educational Rights and Privacy Act): Notify educational institutions if student records breached
- **GDPR** (EU users): Notify users within 72 hours if EU citizen data breached
- **State Laws**: Many US states require breach notification (varies by state)

**Notification Timeline**:
- **Internal Notification**: Immediately (CEO, legal, security team)
- **Regulatory Notification**: As required by law (typically 72 hours)
- **User Notification**: As required by law (typically 30-60 days) or sooner if ethical obligation

**Breach Notification Content**:
- What happened (non-technical description)
- What data was affected
- What we're doing to address it
- What users should do (change password, monitor accounts, etc.)
- How to contact us with questions

For detailed compliance requirements, see [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md) and [GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md).

---

## Escalation Paths

### Technical Escalation

**Level 1: On-Call Engineer**
- Acknowledges and triages all incidents
- Attempts initial resolution
- Escalates if unable to resolve within 30 minutes or if Severity 1

**Level 2: Incident Commander (Senior Engineer/Tech Lead)**
- Assumes command of Severity 1 and complex Severity 2 incidents
- Coordinates response team
- Makes decisions about rollbacks and emergency deployments
- Escalates to CTO if incident requires executive approval

**Level 3: CTO/VP Engineering**
- Involved in all Severity 1 incidents lasting >2 hours
- Approves major decisions (large infrastructure spend, public statements, vendor escalation)
- Coordinates with CEO and executive team
- Authorizes breach notification (if applicable)

### Business Escalation

**Level 1: Product Manager**
- Informed of all Severity 1 and 2 incidents
- Assists with user-facing communication
- Prioritizes post-incident fixes in product roadmap

**Level 2: Head of Customer Success**
- Informed of all Severity 1 incidents
- Coordinates support team response
- Monitors user sentiment and feedback
- May reach out proactively to high-value customers

**Level 3: CEO**
- Informed of all Severity 1 incidents within 30 minutes
- Approves public statements and external communication
- Handles media inquiries (if necessary)
- Authorizes financial decisions (refunds, service credits, etc.)

### Vendor Escalation

**When to Escalate to Vendors**:
- Third-party service outage confirmed (Vercel, Neon, Cloudflare, Braintree)
- Issue suspected to be vendor-related but not confirmed on status page
- Need emergency support or expedited resolution

**Vendor Contact Information** (see [System Overview](../18-architecture/system-overview.md)):
- **Vercel**: Priority support line (if on Pro plan), status.vercel.com
- **Neon**: Support email, neon.tech/docs/introduction/support
- **Cloudflare**: Support portal, cloudflare.com/support
- **Braintree**: Payment support, braintree.com/support
- **Email Provider**: See [Email Provider](../17-integrations/email-provider.md)

**Escalation Template**:
```
Subject: [URGENT] Production Incident - [Brief Description]

Support Team,

We are experiencing a production incident affecting [number] users.

Incident Details:
- Impact: [Description of user impact]
- Start Time: [timestamp UTC]
- Suspected Cause: [Brief description]
- Error Messages: [Copy/paste relevant errors]
- Affected Services: [List of affected services]

We have attempted the following troubleshooting:
- [Action 1]
- [Action 2]

Please prioritize this issue and provide immediate assistance.

Contact: [Your name, email, phone]
Account ID: [Your account ID]
```

---

---

## Supporting Documents Referenced

This incident response specification draws from the following source documents:

- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator incident management tools, monitoring dashboards, and emergency response capabilities informing incident procedures
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional admin tools for incident coordination, communication workflows, and post-incident analysis features
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role definitions informing incident response role assignments and escalation paths based on permission levels

---

## Dependencies

This incident response specification relies on and references:

- **[Observability](observability.md)**: Monitoring, alerting, and dashboards for incident detection
- **[Release Management](release-management.md)**: Rollback procedures and hotfix deployments
- **[Security and Privacy](../18-architecture/security-privacy.md)**: Security controls and breach procedures
- **[Test Strategy](test-strategy.md)**: Testing procedures to prevent incidents
- **[System Overview](../18-architecture/system-overview.md)**: System architecture and component dependencies
- **[Rate Limiting and Abuse](../18-architecture/rate-limiting-and-abuse.md)**: DDoS protection and abuse prevention
- **[Config and Secrets](../18-architecture/config-and-secrets.md)**: Secret rotation after security incidents
- **[Email Provider](../17-integrations/email-provider.md)**: Email notifications during incidents
- **[Feature Flags](../06-system-admin/sysadmin-feature-flags.md)**: Emergency feature disabling
- **[COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md)**: Breach notification requirements

---

## Open Questions

1. **Incident Management Tool**: Which incident management system should we use? (PagerDuty, Opsgenie, VictorOps, custom?)
   - **Consideration**: Cost, integrations with monitoring tools, mobile app quality, escalation policies

2. **Status Page Platform**: Should we use a hosted status page (StatusPage.io, Atlassian Statuspage) or self-hosted?
   - **Consideration**: Third-party status page may also go down, but self-hosted requires maintenance

3. **On-Call Compensation**: How do we compensate on-call engineers for after-hours availability?
   - **Consideration**: Flat stipend, per-incident bonus, comp time, or overtime pay?

4. **Automated Rollback Thresholds**: At what error rate or response time should automatic rollback trigger?
   - **Consideration**: Balance between preventing incidents and avoiding false positive rollbacks

5. **Post-Mortem Enforcement**: Should post-mortems be required for Severity 3 incidents?
   - **Consideration**: Lightweight async post-mortem vs. formal meeting vs. optional

6. **Communication Authority**: Who has authority to send emergency emails to all subscribers?
   - **Consideration**: Incident Commander, CEO, or pre-approved communication coordinator?

7. **Incident Command Training**: Do all engineers need incident commander training or only senior engineers?
   - **Consideration**: Training investment vs. ensuring sufficient coverage

8. **Vendor SLAs**: What are our vendor SLAs and how do they affect our user commitments?
   - **Consideration**: Document vendor commitments and align our SLAs accordingly

9. **Incident Simulation**: How often should we conduct incident response drills (game days)?
   - **Consideration**: Quarterly, bi-annually, or after major architecture changes?

10. **Subscriber Credits**: Under what conditions do we offer service credits or refunds for incidents?
    - **Consideration**: Define clear policy to ensure consistency and fairness

---

## Telemetry

Incident response generates telemetry for operational improvement and post-incident analysis:

### Incident Metrics (tracked per incident)

```typescript
interface IncidentMetrics {
  incident_id: string; // e.g., "INC-2025-01-15-001"
  severity: 1 | 2 | 3 | 4;
  incident_type: 'outage' | 'performance_degradation' | 'data_integrity' | 
                 'security' | 'third_party' | 'deployment_failure';
  affected_system: string; // e.g., "score_recording", "authentication"
  
  // Timing
  detected_at: timestamp;
  acknowledged_at: timestamp;
  mitigated_at: timestamp;
  resolved_at: timestamp;
  
  // Durations (in minutes)
  time_to_acknowledge: number; // detected_at → acknowledged_at
  time_to_mitigate: number; // acknowledged_at → mitigated_at
  time_to_resolve: number; // mitigated_at → resolved_at
  total_duration: number; // detected_at → resolved_at
  
  // Impact
  affected_users_count: number;
  affected_users_percentage: number;
  user_reports_count: number; // support tickets
  
  // Response
  responder: string; // on-call engineer
  incident_commander?: string;
  communication_coordinator?: string;
  additional_responders: string[]; // engineering support team
  
  // Resolution
  resolution_method: 'rollback' | 'hotfix' | 'config_change' | 'vendor_fix' | 'infrastructure_scaling';
  post_mortem_completed: boolean;
  action_items_count: number;
  
  // Metadata
  related_deployment?: string; // version number
  related_incidents?: string[]; // incident IDs
}
```

### Incident Response SLA Tracking

```typescript
interface SLACompliance {
  incident_id: string;
  severity: 1 | 2 | 3 | 4;
  
  // Acknowledgment SLA
  acknowledgment_sla: number; // target in minutes
  actual_acknowledgment_time: number; // actual in minutes
  acknowledgment_sla_met: boolean;
  
  // Mitigation SLA
  mitigation_sla: number; // target in minutes
  actual_mitigation_time: number; // actual in minutes
  mitigation_sla_met: boolean;
  
  // Resolution SLA
  resolution_sla: number; // target in minutes
  actual_resolution_time: number; // actual in minutes
  resolution_sla_met: boolean;
  
  // Overall SLA compliance
  overall_sla_met: boolean;
}
```

### Aggregate Incident Statistics (monthly/quarterly reporting)

```typescript
interface IncidentStatistics {
  period: string; // e.g., "2025-01"
  
  // Volume
  total_incidents: number;
  severity_1_count: number;
  severity_2_count: number;
  severity_3_count: number;
  severity_4_count: number;
  
  // Performance
  mean_time_to_acknowledge: number; // MTTA (minutes)
  mean_time_to_mitigate: number; // MTTM (minutes)
  mean_time_to_resolve: number; // MTTR (minutes)
  
  // SLA Compliance
  sla_compliance_rate: number; // percentage of incidents meeting SLA
  
  // Impact
  total_user_impact_hours: number; // sum of (affected_users * duration)
  total_downtime_minutes: number; // Severity 1 incidents only
  
  // Root Causes (top 5)
  top_root_causes: { cause: string; count: number }[];
  
  // Trends
  month_over_month_change: number; // percentage change from previous period
}
```

These metrics feed into operational dashboards (see [SysAdmin Ops Dashboards](../15-analytics-and-reporting/sysadmin-ops-dashboards.md)) and inform continuous improvement efforts.

---

## Permissions

### Incident Response Roles and Access

| Action | On-Call Engineer | Incident Commander | Communication Coordinator | SysAdmin |
|--------|-----------------|--------------------|-----------------------------|----------|
| View monitoring dashboards | ✅ | ✅ | ❌ | ✅ |
| Acknowledge incidents | ✅ | ✅ | ❌ | ✅ |
| Update incident status | ✅ | ✅ | ❌ | ✅ |
| Approve rollback | ❌ | ✅ | ❌ | ✅ |
| Approve hotfix deployment | ❌ | ✅ | ❌ | ✅ |
| Update status page | ❌ | ✅ | ✅ | ✅ |
| Send subscriber emails | ❌ | ❌ | ✅ | ✅ |
| Access production database (read) | ✅ | ✅ | ❌ | ✅ |
| Access production database (write) | ❌ | ✅ (emergency only) | ❌ | ✅ |
| Rotate API keys/secrets | ❌ | ✅ | ❌ | ✅ |
| Scale infrastructure | ❌ | ✅ | ❌ | ✅ |
| Authorize refunds/credits | ❌ | ❌ | ❌ | ✅ (up to $500) |

**Note**: All production access is logged and auditable (see [Audit Logs](../06-system-admin/sysadmin-audit-logs.md)).

---

*This incident response specification ensures that production incidents are handled efficiently, transparently, and with minimal user impact while continuously improving system reliability.*
