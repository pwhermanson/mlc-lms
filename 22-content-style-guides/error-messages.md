# Error Messages

The definitive, comprehensive guide for all error messaging across the MLC-LMS platform. This document establishes universal standards, patterns, and best practices for communicating errors to users across all roles, contexts, and channels.

## Purpose

This specification exists to ensure that every error message—whether technical, validation-related, or user-generated—communicates clearly, maintains user confidence, and provides actionable guidance toward resolution. Error messages are critical touchpoints that can either:

- **Build trust** by showing the system is reliable and transparent
- **Reduce frustration** by explaining what happened in plain language
- **Enable recovery** by providing clear next steps
- **Support learning** by contextualizing errors within the educational mission
- **Maintain engagement** by avoiding panic-inducing or shame-based language

Poor error messages create support burden, user abandonment, and loss of trust. Well-crafted error messages turn potentially negative experiences into opportunities for guidance and confidence-building.

### Goals
1. **Consistency**: All errors follow predictable patterns regardless of where they occur
2. **Clarity**: Users immediately understand what happened and why it matters
3. **Actionability**: Every error provides concrete steps toward resolution
4. **Appropriate Tone**: Messages match the severity and context of the error
5. **Accessibility**: Errors are perceivable and understandable by all users
6. **Localization-Ready**: Structure supports translation without losing meaning

## Scope

### Included
- **All user-facing error messages** across web, mobile, email, and in-app notifications
- **Role-specific error patterns** for Students, Teachers, Admins, System Admins, and Guests
- **Error categories**: Technical errors, validation errors, permission errors, data errors, network errors
- **Context-specific guidance**: Errors during gameplay, assignment creation, billing, imports, etc.
- **Fallback patterns**: When detailed error information is unavailable
- **Error logging requirements**: What to log for support and debugging
- **Visual presentation standards**: Layout, icons, colors, and emphasis
- **Accessibility requirements**: Screen reader support, keyboard navigation, color independence
- **Multi-channel delivery**: How errors appear in UI, email, SMS, API responses
- **Localization structure**: How to maintain clarity across languages

### Excluded
- **Success messages and confirmations** (see [Screen Text and Copy](../01-ux-design-system/screen-text-and-copy.md))
- **Validation feedback during input** (covered in component specifications)
- **General UI copy** (see role-specific copy guides: [Students](./copy-guide-students.md), [Teachers](./copy-guide-teachers.md), [Admins](./copy-guide-admins.md))
- **Marketing or promotional messaging**
- **Legal disclaimers** (see [Legal and Compliance](../23-legal-and-compliance/privacy-policy.md))

## Universal Error Message Structure

Every error message across the platform follows this hierarchical structure, with components shown/hidden based on severity and context:

### Primary Structure (Always Present)

```
[ICON/INDICATOR] [HEADLINE]
[Description]
[Action(s)]
[Reassurance]
```

### Component Definitions

#### 1. Icon/Indicator
**Purpose**: Immediate visual signal of error type and severity

**Standards**:
- **Critical/Blocking**: Red ⛔ or ❌ icon
- **Warning/Recoverable**: Yellow/Orange ⚠️ icon  
- **Informational/Temporary**: Blue ℹ️ icon
- **Validation/Input**: Inline red text or border highlight

**Accessibility**: Icon must have text alternative; do not rely on color alone

#### 2. Headline
**Purpose**: One-sentence summary of what went wrong

**Standards**:
- **Length**: 5-10 words maximum
- **Structure**: What happened, not why (details come next)
- **Tone**: Calm and factual, not alarmist
- **Avoid**: Technical jargon, error codes in headline

**Examples**:
- ✅ "Unable to save assignment"
- ✅ "Game did not load"
- ✅ "Payment method declined"
- ❌ "Error 500: Internal server exception"
- ❌ "FATAL: Database connection failed"
- ❌ "You made a mistake"

#### 3. Description
**Purpose**: Brief explanation of why it happened and what it means

**Standards**:
- **Length**: 1-2 sentences (3 max for complex errors)
- **Language**: Plain, role-appropriate terms (see role-specific sections)
- **Context**: Help user understand impact
- **Optional**: Technical details in expandable section for advanced users

**Examples**:
- ✅ "Your internet connection was interrupted. Don't worry—your progress is saved."
- ✅ "This student already has an active assignment for this sequence."
- ✅ "The CSV file contains formatting errors in rows 5, 12, and 18."
- ❌ "Network timeout exception in session handler"
- ❌ "Something went wrong" (too vague)

#### 4. Action(s)
**Purpose**: Clear next steps user can take to resolve or work around the error

**Standards**:
- **Format**: Bulleted list or numbered steps if multiple actions
- **Language**: Action verbs, imperative mood ("Try...", "Check...", "Contact...")
- **Priority**: List most likely solution first
- **Specificity**: Provide concrete steps, not vague suggestions

**Examples**:
- ✅ "• Refresh the page and try again"
- ✅ "• Check your internet connection"
- ✅ "• If the problem continues, contact support"
- ✅ "1. Remove the duplicate student from row 12"
- ✅ "2. Re-upload the corrected CSV file"
- ❌ "Please fix the problem" (not actionable)
- ❌ "Try something else" (not specific)

#### 5. Reassurance
**Purpose**: Reduce anxiety and maintain trust

**Standards**:
- **When to include**: Data loss concerns, game/assignment interruptions, payment issues
- **When to omit**: Minor validation errors, expected workflow blocks
- **Tone**: Confident and calming
- **Focus**: What IS protected/saved, not what might be lost

**Examples**:
- ✅ "Your progress is saved and will appear when you reconnect."
- ✅ "No payment has been processed. You can try again safely."
- ✅ "Your students' scores are protected and accessible."
- ❌ "We hope this doesn't happen again"
- ❌ "Sorry for the inconvenience" (overused, insincere)

### Optional Components (Contextual)

#### Support Reference Code
**When**: Technical errors that may require support intervention  
**Format**: "Reference code: ERR-2024-03-15-A7B3" (timestamp-based, readable)  
**Placement**: Small text at bottom of error message

#### Technical Details (Expandable)
**When**: Power users, developers, or admins need deeper information  
**Format**: Collapsible "Show technical details" section  
**Content**: Stack traces, API responses, SQL errors (sanitized for security)

#### Related Help Articles
**When**: Error is common or requires education  
**Format**: "Learn more: [Article title]" as clickable link  
**Placement**: After actions, before reassurance

## Error Categories and Patterns

### Category 1: Technical/System Errors

**Definition**: Errors caused by infrastructure, services, or system failures beyond user control

**Common Types**:
- Server errors (5xx)
- Database connection failures
- Third-party service outages (payment processor, CDN, email provider)
- Timeout errors
- Memory/resource exhaustion

**Tone**: Apologetic but confident; emphasize system reliability and temporary nature

**Pattern Template**:
```
[HEADLINE: Service temporarily unavailable]
[DESCRIPTION: We're experiencing a brief technical issue. This doesn't happen often, and our team is already working on it.]
[ACTIONS:
• Please try again in a few minutes
• If urgent, check our status page: status.musiclearningcommunity.com
• Your work is saved and will be available when service resumes]
[REASSURANCE: All student progress and account data are protected.]
```

**Examples by Context**:

**Game Loading Failure**:
```
⚠️ Game didn't load

The game couldn't connect to our servers. This sometimes happens with internet connections.

• Refresh the page and try again
• Check that you're connected to the internet
• Try a different browser if the problem continues

Don't worry—your scores and progress are saved!
```

**Assignment Save Failure (Teacher)**:
```
❌ Unable to save assignment

A temporary server issue prevented saving. This is rare and usually resolves quickly.

• Click "Save" again in a moment
• If it fails again, copy your assignment details and contact support
• Reference code: ERR-ASGN-20240315-4B2C

Student data is protected. No changes have been lost.
```

**Payment Processing Error (Admin)**:
```
⚠️ Payment could not be processed

The payment provider is temporarily unavailable. Your payment method was not charged.

• Try again in 10-15 minutes
• Check payment provider status: stripe.com/status
• Contact billing support if urgent: billing@musiclearningcommunity.com

Your subscription remains active during this grace period.
```

### Category 2: Network/Connectivity Errors

**Definition**: Errors caused by poor/lost internet connection or client-side network issues

**Tone**: Reassuring and practical; acknowledge this is often outside platform control

**Pattern Template**:
```
[HEADLINE: Connection lost]
[DESCRIPTION: Your internet connection was interrupted.]
[ACTIONS:
• Check your internet connection
• Refresh the page when reconnected
• Your progress is saved automatically]
[REASSURANCE: No work has been lost.]
```

**Examples by Context**:

**Mid-Game Disconnect (Student)**:
```
ℹ️ Connection lost

You got disconnected from the internet while playing.

• Check your wifi or internet connection
• When you're back online, refresh this page
• Your last score was saved—you won't lose your progress!

Tip: If this happens a lot, ask an adult to check your internet connection.
```

**Report Loading Timeout (Teacher)**:
```
⚠️ Report timed out

The report is taking longer than expected to load. This can happen with slow connections or very large class rosters.

• Refresh the page to try again
• Filter by date range or student group to reduce load time
• Download the CSV export for faster access to data

Your student data is safe and accessible.
```

**Bulk Import Timeout (Admin)**:
```
⚠️ Import timed out

The CSV upload timed out before completing. This can happen with large files or slow connections.

• Ensure stable internet connection
• Split large CSV files into batches of 500 rows or fewer
• Try uploading during off-peak hours for better performance

No partial data was imported. Re-upload when ready.
```

### Category 3: Validation/Input Errors

**Definition**: Errors caused by user input that doesn't meet system requirements

**Tone**: Instructional and helpful, never blaming

**Pattern Template**:
```
[HEADLINE: [Field name] needs correction]
[DESCRIPTION: What's wrong with the input and why it's required]
[ACTIONS: Specific correction needed]
[REASSURANCE: Omit unless data loss risk]
```

**Examples by Context**:

**Username Validation (Teacher creating student)**:
```
⚠️ Username already exists

The username "sarah_chen" is already taken. Usernames must be unique across the system.

• Try "sarah_chen_2024" or "s_chen_studio"
• Add your studio name to make it unique
• Check that you didn't already create this student

All other student information is saved—just update the username.
```

**Assignment Date Validation (Teacher)**:
```
⚠️ End date must be after start date

The due date (March 10) is before the start date (March 15).

• Change the due date to March 16 or later
• Or change the start date to March 9 or earlier

Your assignment details are saved. Just fix the dates and save again.
```

**CSV Format Error (Admin)**:
```
❌ CSV file has formatting errors

3 errors found in student_upload.csv:
• Row 5: Email format invalid ("sarah.email" needs @domain)
• Row 12: Duplicate username "mike_t"
• Row 18: Grade level missing (required field)

• Download the error report to see all issues
• Fix the errors in your spreadsheet
• Re-upload the corrected file

No data was imported. Fix errors and try again.
```

**Password Requirements (All Users)**:
```
⚠️ Password doesn't meet requirements

Your password must:
• Be at least 8 characters long
• Include at least one number
• Include at least one uppercase letter

Try a stronger password like: "Music2024!" (don't use this exact one!)
```

### Category 4: Permission/Access Errors

**Definition**: Errors when users attempt actions they don't have permission to perform

**Tone**: Informative and non-judgmental; explain permissions clearly

**Pattern Template**:
```
[HEADLINE: You don't have permission for this action]
[DESCRIPTION: What permission/role is required and why]
[ACTIONS: Who to contact or how to request access]
[REASSURANCE: Omit]
```

**Examples by Context**:

**Teacher Attempting Admin Function**:
```
🔒 Teacher-Admin permission required

Bulk student imports require Teacher-Admin or Subscriber-Admin permissions.

• Contact your school's subscriber admin to request this permission
• Or ask them to perform the import on your behalf
• See all teacher permissions: [Link to help article]
```

**Student Attempting Locked Game**:
```
🔒 Complete previous games to unlock

This game is locked until you complete "Rhythm Factory Level 2."

• Go back to your assignment and finish the earlier games
• Games unlock in order to help you learn step-by-step

Your teacher chose this sequence to help you learn best!
```

**Subscriber Attempting System Admin Function**:
```
🔒 System Administrator access required

This action is restricted to MLC system administrators for security and data protection.

• If you believe you need this access, contact support: support@musiclearningcommunity.com
• Include your organization name and reason for request

Reference: [Link to roles documentation]
```

### Category 5: Data/Content Errors

**Definition**: Errors related to missing, corrupt, or inaccessible data

**Tone**: Transparent and helpful; provide workarounds when possible

**Pattern Template**:
```
[HEADLINE: Content not found / Content unavailable]
[DESCRIPTION: What's missing and likely cause]
[ACTIONS: Workarounds or recovery steps]
[REASSURANCE: When appropriate]
```

**Examples by Context**:

**Game Asset Missing (Student)**:
```
⚠️ Game graphics didn't load

Some images in this game didn't load. You can still play, but it might look different.

• Refresh the page to reload the images
• If it keeps happening, try clearing your browser cache
• The game will still record your score correctly

Your progress is safe!
```

**Student Record Not Found (Teacher)**:
```
❌ Student not found

The student "Sarah Chen" could not be found in your roster. This student may have been moved to another teacher or removed.

• Check with your subscriber admin if this is unexpected
• View your complete current roster: [Link]
• Search for the student by username instead of name

If this is an error, contact support with reference code: STU-404-20240315
```

**Report Data Incomplete (Admin)**:
```
⚠️ Report partially unavailable

Score data is incomplete for March 1-5 due to a system migration. Reports for other dates are accurate.

• Download reports for dates after March 6 for complete data
• Contact support if you need the missing data reconstructed
• Reference: MIGRATION-2024-Q1

All current and future data is recording correctly.
```

### Category 6: Business Logic/Workflow Errors

**Definition**: Errors when actions violate business rules (not technical failures)

**Tone**: Explanatory and instructional; help users understand why the rule exists

**Pattern Template**:
```
[HEADLINE: Action not allowed]
[DESCRIPTION: What rule was violated and why it exists]
[ACTIONS: Alternative approaches]
[REASSURANCE: Optional]
```

**Examples by Context**:

**Exceeding Seat Limit (Subscriber)**:
```
⚠️ Seat limit reached

You've used all 20 student seats in your subscription. To add more students, you'll need to add seats.

• Add seats to your subscription: [Link to billing]
• Or remove inactive students to free up seats
• Contact us for school/district pricing

Your existing students can continue playing without interruption.
```

**Assigning Expired Subscription (Teacher)**:
```
❌ Cannot create assignment—subscription expired

Your organization's subscription expired on March 1. Students cannot play games until subscription is renewed.

• Contact your subscriber admin to renew: [Admin contact info]
• Or direct them to billing management: [Link]

Existing student data and scores are preserved and will be accessible when subscription renews.
```

**Duplicate Sequence Assignment (Teacher)**:
```
⚠️ Student already has this sequence

Michael Torres is already assigned "Piano Adventures Level 1." Students can only have one sequence active at a time.

• Replace the existing assignment with a new one
• Or assign a different sequence instead
• Or wait until Michael completes the current sequence

View Michael's current assignment progress: [Link]
```

### Category 7: Quota/Rate Limit Errors

**Definition**: Errors when users exceed system limits designed to prevent abuse

**Tone**: Apologetic but firm; explain the limit's purpose

**Pattern Template**:
```
[HEADLINE: Limit reached]
[DESCRIPTION: What limit and why it exists]
[ACTIONS: When they can try again or how to request increase]
[REASSURANCE: Their work is saved]
```

**Examples by Context**:

**API Rate Limit (Integration User)**:
```
⚠️ API rate limit exceeded

You've exceeded the limit of 100 requests per minute. This limit prevents system overload.

• Wait 60 seconds before making more requests
• Implement exponential backoff in your integration
• Contact support for higher limits: api-support@musiclearningcommunity.com

Your data is safe. Retry your request in a moment.
```

**Bulk Message Limit (Teacher)**:
```
⚠️ Message limit reached

You can send up to 5 bulk messages per day to prevent spam. You've reached today's limit.

• Your limit resets at midnight
• Or send individual messages to specific students
• Contact support if you have an urgent need

Your previously sent messages were delivered successfully.
```

**Export Size Limit (Admin)**:
```
⚠️ Export too large

This export would exceed 10,000 rows. Large exports can cause performance issues.

• Filter by date range to reduce export size (e.g., one month at a time)
• Or export by building/grade level separately
• Contact support for full data extraction: support@musiclearningcommunity.com

Apply filters and try again.
```

## Role-Specific Error Messaging

### Students (Ages 6-18)

**Key Principles**:
- **Never shame or blame**: "Let's try again" not "You did this wrong"
- **Simple language**: Age-appropriate vocabulary
- **Visual cues**: Emoji or icons to reinforce meaning
- **Maintain game illusion**: Avoid technical jargon
- **Preserve confidence**: Emphasize that they didn't lose progress

**Vocabulary Guidelines**:
- ✅ "Oops!", "Uh-oh", "Hmm..."
- ✅ "Let's try again", "One more time"
- ✅ "Check that...", "Make sure..."
- ❌ "Error", "Failed", "Invalid", "Denied"
- ❌ Technical terms without explanation

**Example Error Rewrite**:

**Before (too technical)**:
```
ERROR: Session timeout. Authentication token expired.
```

**After (student-appropriate)**:
```
😊 Time to log in again

You've been away for a while. For safety, we logged you out automatically.

• Log in again with your username and password
• If you forgot your password, ask your teacher

Your scores and progress are all saved!
```

**Age-Specific Adjustments**:

**Ages 6-8**: Extra simple, very visual
```
😊 Oops! Something happened

The game stopped. That's okay!

• Click here to try again: [Big Reload Button]
• Ask your teacher or parent for help

You didn't lose anything!
```

**Ages 9-12**: Slightly more detail
```
⚠️ Game didn't save your score

Your internet connection got unplugged. The game couldn't send your score.

• Check your internet connection
• Play the game again when you're back online
• Your other scores are safe!
```

**Ages 13-18**: More autonomy and detail
```
⚠️ Connection lost during gameplay

Your score couldn't be saved because the connection was interrupted.

• Verify your internet connection is stable
• Replay the game to record your score
• Contact your teacher if this keeps happening

Previous scores are unaffected.
```

### Teachers

**Key Principles**:
- **Respect expertise**: Don't talk down or over-explain
- **Save time**: Front-load the solution
- **Protect student impact**: Always clarify if students are affected
- **Provide context**: Link to relevant help documentation
- **Enable self-service**: Give tools to diagnose and fix

**Example Structure**:
```
[Problem + Impact on Students]
[Why this happened - technical context if helpful]
[Fix steps - numbered if multiple]
[Link to help article for background]
```

**Example Error**:
```
❌ Unable to assign sequence

This sequence contains games that are no longer active in the system. Students would encounter broken links.

• Choose a different sequence from the library
• Or contact support to restore archived games: support@musiclearningcommunity.com
• See currently active sequences: [Link to game library]

Reference: Sequence ID #287 (Piano Adventures 2015 Edition)
```

**When Error Affects Students Already**:
```
⚠️ 8 students unable to access assignment

A game in "Level 2 Rhythm Assignment" was moved to draft status, breaking the assignment for students who haven't completed it yet.

Immediate action needed:
1. Click here to view affected students: [Link]
2. Assign them a replacement game: [Suggested alternatives]
3. Or remove the broken game from the assignment

Students see "This game is temporarily unavailable" and cannot proceed until you take action.
```

### Administrators/Subscribers

**Key Principles**:
- **Show organizational impact**: How many users/teachers/students affected
- **Provide admin tools**: Links to relevant admin dashboards
- **Include billing context**: When errors relate to payment/subscription
- **Enable delegation**: Show which role needs to fix (if not admin)
- **Offer escalation**: When to contact support

**Example Error**:
```
❌ Subscription payment failed

Your automatic payment for March 2024 was declined by your bank.

Impact:
• Subscription remains active until March 15 (grace period)
• 45 students and 3 teachers can continue using the system
• After grace period, access will be suspended

Action needed:
1. Update payment method: [Link to billing]
2. Or contact your bank to authorize the charge
3. Or contact billing support: billing@musiclearningcommunity.com

Reference: Invoice #2024-03-001 • Amount: $244.00
```

**Multi-User Impact Example**:
```
⚠️ Teacher import partially failed

20 of 25 teacher accounts created successfully. 5 failed due to duplicate email addresses.

• Download error report: [CSV with failed rows]
• Correct the email addresses
• Re-upload only the corrected rows

Successfully created teachers have been notified and can log in immediately.
```

### System Administrators

**Key Principles**:
- **Include technical details**: Stack traces, error codes, system info
- **Show system-wide impact**: How many orgs/users affected
- **Provide diagnostic tools**: Links to logs, metrics, admin panels
- **Indicate urgency level**: Critical/High/Medium/Low
- **Enable incident response**: Reference codes, timestamps, affected services

**Example Error**:
```
🚨 CRITICAL: Payment processor webhook failures

Stripe webhook endpoint has failed 47 times in the last 15 minutes. Subscription renewals are not processing.

Impact:
• 12 organizations with renewals due today
• Estimated revenue at risk: $3,240
• Users are not yet aware (grace period active)

Immediate actions:
1. Check Stripe dashboard for webhook status: [Link]
2. Verify webhook endpoint is responding: [Health check URL]
3. Review recent deploys for breaking changes
4. Escalate to on-call engineer if not resolved in 30 min

Technical details:
• Error: 502 Bad Gateway
• Endpoint: /api/webhooks/stripe
• First failure: 2024-03-15 14:23:18 UTC
• Logs: [Datadog link]
• Runbook: [Confluence link]

Reference: INC-20240315-001
```

## Contextual Error Scenarios

### During Gameplay

**Special Considerations**:
- **Minimize disruption**: Keep student in flow state if possible
- **Preserve progress**: Explicitly state that scores/progress are saved
- **Enable quick recovery**: One-click reload/retry
- **Visual consistency**: Match game UI (don't break immersion with system-style errors)

**Example**:
```
[Game UI styled error overlay]

😅 Oops! Audio didn't load

The music for this game needs to load again.

[Reload Game Button]  [Skip This Game Button]

Your score so far is saved!
```

### During Assignment Creation (Teacher)

**Special Considerations**:
- **Save work in progress**: Auto-save drafts
- **Prevent loss**: Warn before navigating away
- **Show validation early**: Check as user builds, not just at submission

**Example (Inline Validation)**:
```
Assignment Builder > Sequence Selection

⚠️ This sequence is archived
"Piano Adventures 2015 Edition" is no longer maintained. Students may encounter outdated content.

Recommended: Use "Piano Adventures 2020 Edition" instead [Switch to this sequence]
Or continue with archived sequence [Continue anyway]
```

### During Bulk Operations (Admin)

**Special Considerations**:
- **Confirm before execution**: Show preview of impact
- **Allow cancel mid-process**: For long-running operations
- **Provide detailed results**: Which succeeded, which failed, why
- **Enable selective retry**: Retry only failed items

**Example**:
```
Bulk Student Import Results

✅ 245 students imported successfully
❌ 12 students failed (see details below)

Failures:
Row 67: Duplicate username "alex_smith" [Edit and retry]
Row 89: Invalid grade level "13" (must be K-12) [Edit and retry]
Row 103: Missing required field "teacher_id" [Edit and retry]
[+ 9 more errors] [Download error report CSV]

[Retry Failed Rows] [Start New Import] [Return to Dashboard]
```

### During Payment/Billing

**Special Considerations**:
- **Never lose payment data**: Save attempted payment details securely
- **Provide payment status**: Clearly indicate if money was charged
- **Give alternatives**: Other payment methods if one fails
- **Reassure access**: Subscription remains active during grace period

**Example**:
```
💳 Payment method needs updating

Your credit card ending in 4532 expires this month. Update your payment method to avoid service interruption.

• Update card information: [Link to secure billing portal]
• Or switch to annual invoice payment: [Link]

Your subscription remains active. Update by March 31 to prevent disruption.
```

### During System Maintenance

**Special Considerations**:
- **Announce in advance**: Use system announcements, email, in-app notifications
- **Show countdown**: "Maintenance in 2 hours"
- **Provide time window**: Exact start and end times in user's timezone
- **Offer alternatives**: What features remain available

**Example (Advance Warning)**:
```
📅 Scheduled maintenance: Sunday, March 17, 2-4 AM EST

We'll be upgrading our servers to improve performance. Here's what to expect:

During maintenance:
• Students can play games normally
• Teachers cannot create new assignments or view reports
• No access to admin/billing functions

After maintenance:
• Faster report loading
• Improved bulk import speed
• All features restored

Schedule lessons/assignments around this window, or proceed normally—student play is unaffected.

[Add to calendar] [Learn more]
```

### During Migration/Data Import

**Special Considerations**:
- **Set expectations**: How long will it take
- **Show progress**: Real-time updates when possible
- **Allow pause/resume**: For very large operations
- **Validate before committing**: Dry-run option for admins

**Example**:
```
🔄 Importing 1,247 students...

Progress: ████████░░░░ 672 of 1,247 (54%)
Elapsed time: 3 minutes
Estimated remaining: 3 minutes

✅ 672 imported successfully
⚠️ 8 warnings (non-blocking, see details)
❌ 2 failed (will need correction)

[View Details] [Pause Import] [Cancel Import]

You can leave this page—we'll email you when complete.
```

## Visual Presentation Standards

### Error UI Components

#### Modal/Dialog Errors (Blocking)
**When to use**: Critical errors requiring immediate attention before continuing

**Structure**:
```
[Overlay backdrop - dims content behind]

┌─────────────────────────────────────┐
│  [Icon] [Headline]                  │
│                                      │
│  [Description text]                  │
│                                      │
│  [Bulleted actions]                  │
│                                      │
│  [Reassurance text]                  │
│                                      │
│  [Primary Button] [Secondary Button] │
└─────────────────────────────────────┘
```

**Example**:
```
┌─────────────────────────────────────┐
│  ❌ Payment method declined          │
│                                      │
│  Your bank declined this payment.    │
│  You have not been charged.          │
│                                      │
│  • Update payment method             │
│  • Try a different card              │
│  • Contact your bank                 │
│                                      │
│  Your subscription remains active.   │
│                                      │
│  [Update Payment] [Try Again]        │
└─────────────────────────────────────┘
```

#### Toast/Snackbar Errors (Non-Blocking)
**When to use**: Minor errors that don't prevent continued use

**Structure**:
```
[Slide in from top or bottom, auto-dismiss after 5-7 seconds]
[Icon] [Brief message] [Optional action link] [X dismiss]
```

**Example**:
```
⚠️ Unable to send message. Try again. [Retry] ✕
```

#### Inline Form Errors (Field-Level)
**When to use**: Validation errors on specific form fields

**Structure**:
```
[Field Label]
[Input field with red border]
🔴 [Error message directly below field]
```

**Example**:
```
Email Address
[sarah.student.com        ]
🔴 Email must include "@" symbol
```

#### Banner Errors (Page-Level)
**When to use**: Persistent errors affecting entire page/section

**Structure**:
```
┌─────────────────────────────────────────────────────┐
│ [Icon] [Message] [Action button] [Dismiss X]        │
└─────────────────────────────────────────────────────┘
[Rest of page content below]
```

**Example**:
```
┌─────────────────────────────────────────────────────┐
│ ⚠️ Your subscription will expire in 3 days.         │
│ [Renew Subscription] ✕                              │
└─────────────────────────────────────────────────────┘
```

### Color Standards

**Critical/Blocking Errors**:
- Background: `#FEE` (light red)
- Border: `#E53E3E` (red-600)
- Icon: `#C53030` (red-700)
- Text: `#742A2A` (red-900)

**Warning/Recoverable**:
- Background: `#FEF5E7` (light yellow)
- Border: `#DD6B20` (orange-600)
- Icon: `#C05621` (orange-700)
- Text: `#7C2D12` (orange-900)

**Informational**:
- Background: `#EBF8FF` (light blue)
- Border: `#3182CE` (blue-600)
- Icon: `#2B6CB0` (blue-700)
- Text: `#2C5282` (blue-900)

**Accessibility**:
- All color combinations must meet WCAG AA contrast ratio (4.5:1 minimum)
- Do not rely on color alone—always include icon and text
- Support high-contrast mode and dark mode variants

### Icon Standards

**Error Icons**:
- ❌ or ⛔: Critical/blocking errors
- ⚠️: Warnings
- ℹ️: Informational
- 🔒: Permission/access denied
- 💳: Payment/billing
- 🔄: Loading/processing
- ✅: Success (for comparison/resolution)

**Icon Sizing**:
- Modal headers: 24px × 24px
- Inline errors: 16px × 16px
- Toast notifications: 20px × 20px
- Banner alerts: 20px × 20px

## Accessibility Requirements

### Screen Reader Support

**ARIA Labels**:
```html
<div role="alert" aria-live="assertive" aria-atomic="true">
  <span class="sr-only">Error:</span>
  <h3>Unable to save assignment</h3>
  <p>A temporary server issue prevented saving...</p>
</div>
```

**Key Attributes**:
- `role="alert"`: Announces to screen readers immediately
- `aria-live="assertive"`: Interrupts current reading for critical errors
- `aria-live="polite"`: Waits for pause for non-critical errors
- `aria-atomic="true"`: Reads entire error message, not just changes
- Include hidden `<span class="sr-only">Error:</span>` prefix

### Keyboard Navigation

- **Focus management**: Move focus to error message when it appears
- **Dismissible**: Allow ESC key to close modal errors
- **Actionable**: All error action buttons must be keyboard accessible
- **Tab order**: Logical tab sequence through error UI elements

### Visual Accessibility

- **Text size**: Minimum 14px, scalable with browser zoom
- **Contrast**: Meet WCAG AA (4.5:1) or AAA (7:1) for critical errors
- **Color independence**: Information conveyed through icon + text, not color alone
- **Focus indicators**: Visible focus outline on interactive elements
- **Animation**: Respect `prefers-reduced-motion` setting

### Cognitive Accessibility

- **Simple language**: Grade 6-8 reading level when possible
- **Consistent structure**: Same pattern every time
- **Numbered steps**: For multi-step resolution
- **No time pressure**: Don't auto-dismiss critical errors
- **Visual hierarchy**: Clear headlines, short paragraphs, bullet points

## Multi-Channel Delivery

### In-App (Web/Mobile)

**Format**: Full structure with modal/banner/toast as appropriate

**Example**:
```
[Modal overlay]
❌ Unable to load student roster

The system couldn't retrieve your student list. This is usually a temporary connection issue.

• Refresh the page to try again
• Check your internet connection
• If problem persists, contact support

Your student data is safe and will load when connection is restored.

[Refresh Page] [Contact Support]
```

### Email

**Format**: Text-based with HTML formatting, include logo and support footer

**Example**:
```
Subject: Action Required: Payment Method Declined

Hi Ms. Johnson,

Your automatic payment for March 2024 was declined by your bank.

What this means:
• Your subscription remains active until March 15 (grace period)
• Your 45 students can continue playing normally
• After March 15, access will be suspended

What to do:
1. Update your payment method: [Update Payment Button]
2. Or contact your bank to authorize the charge ($244.00)
3. Or contact our billing team: billing@musiclearningcommunity.com

If you've already updated your payment method, please disregard this message.

Questions? Reply to this email or call (314) 555-0100.

Best regards,
MLC Billing Team

[Footer with logo, links, unsubscribe]
```

### SMS (Urgent Only)

**Format**: Ultra-brief, text-only, include link

**Example**:
```
MLC Alert: Your subscription will expire in 3 days. Renew now to avoid service interruption: [short link]
```

### API Response

**Format**: Structured JSON with consistent error schema

**Example**:
```json
{
  "error": {
    "code": "VALIDATION_FAILED",
    "message": "Student username already exists",
    "details": "The username 'sarah_chen' is already registered in the system. Usernames must be unique.",
    "field": "username",
    "suggested_action": "Try 'sarah_chen_2024' or append your studio name",
    "documentation_url": "https://api.musiclearningcommunity.com/docs/errors/duplicate-username",
    "request_id": "req_7B3A9F2E",
    "timestamp": "2024-03-15T14:23:18Z"
  },
  "status": 400
}
```

**API Error Code Conventions**:
- `400-499`: Client errors (user can fix)
- `500-599`: Server errors (system issue)
- Specific codes: `VALIDATION_FAILED`, `UNAUTHORIZED`, `NOT_FOUND`, `RATE_LIMIT_EXCEEDED`, `INTERNAL_ERROR`

### Push Notifications (Mobile App)

**Format**: Brief title + body, include action

**Example**:
```
Title: Assignment Error
Body: Unable to load your assignment. Tap to retry.
Action: [Open App]
```

## Telemetry and Logging

### What to Log

Every error that reaches a user should be logged with:

**Required Fields**:
- `error_id`: Unique identifier (e.g., `ERR-2024-03-15-A7B3`)
- `timestamp`: ISO 8601 format
- `user_id`: Who encountered the error (or `null` if guest)
- `user_role`: Student, Teacher, Admin, SysAdmin, Guest
- `error_category`: Technical, Network, Validation, Permission, Data, Business Logic, Rate Limit
- `error_severity`: Critical, High, Medium, Low, Info
- `error_code`: Machine-readable code (e.g., `DATABASE_CONNECTION_FAILED`)
- `error_message_displayed`: Exact text shown to user
- `url`: Page/endpoint where error occurred
- `user_agent`: Browser/device info
- `session_id`: Current session identifier

**Optional but Recommended**:
- `stack_trace`: Technical details (sanitized, no PII)
- `affected_entity_type`: Game, Assignment, Student, etc.
- `affected_entity_id`: Specific ID of affected entity
- `recovery_action_taken`: What user did (dismissed, retried, contacted support)
- `resolution_time`: How long until error resolved (if tracked)

**Example Log Entry** (JSON):
```json
{
  "error_id": "ERR-2024-03-15-A7B3",
  "timestamp": "2024-03-15T14:23:18.492Z",
  "user_id": "usr_789456",
  "user_role": "Teacher",
  "error_category": "Technical",
  "error_severity": "High",
  "error_code": "ASSIGNMENT_SAVE_FAILED",
  "error_message_displayed": "Unable to save assignment. A temporary server issue prevented saving. This is rare and usually resolves quickly.",
  "url": "/teacher/assignments/create",
  "user_agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)...",
  "session_id": "sess_1234567890",
  "stack_trace": "[sanitized stack trace]",
  "affected_entity_type": "Assignment",
  "affected_entity_id": null,
  "recovery_action_taken": "user_dismissed",
  "additional_context": {
    "assignment_type": "sequence",
    "sequence_id": "seq_piano_adventures_l1",
    "student_count": 12
  }
}
```

### Error Tracking Integration

**Tools**:
- Sentry, Rollbar, or similar for error aggregation
- Datadog, New Relic for system monitoring
- Google Analytics for user-facing error events
- Internal database for support reference lookup

**Alert Thresholds**:
- Critical errors: Alert immediately
- High errors: Alert if >10 occurrences in 5 minutes
- Medium errors: Daily digest
- Low/Info errors: Weekly summary

### Support Reference Lookup

System admins and support staff should be able to:
1. Look up error by reference code (e.g., `ERR-2024-03-15-A7B3`)
2. See full error context, including user, session, and technical details
3. View user's recent activity before error
4. Track if error was resolved and how
5. Link to related errors (same root cause)

## Localization Considerations

### Structure for Translation

**Translatable**:
- Headlines
- Descriptions
- Action button labels
- Reassurance text

**Non-Translatable** (Keep in English or localize separately):
- Error codes (e.g., `ERR-2024-03-15-A7B3`)
- Technical field names in developer errors
- URLs and email addresses
- Reference IDs

**Translation Keys Structure**:
```
error.category.specific_error.component

Examples:
error.technical.game_load_failed.headline
error.technical.game_load_failed.description
error.technical.game_load_failed.action_1
error.validation.duplicate_username.headline
```

### Cultural Adaptations

**Tone Variations**:
- Some cultures prefer more formal language even for students
- Some prefer more apologetic tone for system errors
- Date/time format varies by locale

**Example** (US vs. UK):
- US: "Check your internet connection"
- UK: "Check your internet connexion" (or "Check you're connected to the internet")

**Example** (US vs. Japan):
- US: "Oops! Something went wrong."
- JP: "申し訳ございません。エラーが発生しました。" (More formal apology)

### Variable Placeholders

Use named placeholders for dynamic content to support different grammar structures:

**Good**:
```
"Unable to save assignment for {student_count} students"
```

**Bad** (doesn't work in all languages):
```
"Unable to save assignment for " + student_count + " students"
```

**Translation Note**: Provide context about variables:
```
error.assignment.save_failed.description
"Unable to save assignment for {student_count} students"
Context: student_count is a number (1-1000)
```

## Fallback Patterns

When detailed error information is unavailable (e.g., catastrophic failure, network issues), use these generic fallback patterns:

### Generic Technical Error (All Users)

```
⚠️ Something went wrong

We encountered an unexpected issue. This doesn't happen often.

• Refresh the page and try again
• If the problem continues, contact support
• Reference code: [auto-generated code]

Your data is safe and will be available when the system recovers.

[Refresh Page] [Contact Support]
```

### Generic Network Error (All Users)

```
ℹ️ Connection lost

Your internet connection was interrupted.

• Check your internet connection
• Refresh the page when reconnected

Your progress is saved automatically.

[Refresh Page]
```

### Generic Validation Error (All Users)

```
⚠️ Please check your information

Some of the information you entered needs correction.

• Review the fields marked in red
• Make the needed corrections
• Try saving again

Your other information is saved.
```

### When Error Logging Fails

If the error logging system itself fails, store minimal info locally and retry:

**Browser Local Storage**:
```json
{
  "failed_errors": [
    {
      "timestamp": "2024-03-15T14:23:18Z",
      "error_code": "UNKNOWN",
      "user_message": "Something went wrong",
      "retry_count": 0
    }
  ]
}
```

Background process retries sending to logging system every 5 minutes, maximum 10 retries.

## Testing and Quality Assurance

### Error Message Checklist

Before deploying any new error message, verify:

- [ ] Follows universal structure (Headline → Description → Actions → Reassurance)
- [ ] Uses role-appropriate language and tone
- [ ] Provides specific, actionable next steps
- [ ] Includes reassurance when data loss is a concern
- [ ] Has proper icon and color coding for severity
- [ ] Meets accessibility requirements (ARIA, contrast, screen reader)
- [ ] Is localization-ready (uses translation keys, named variables)
- [ ] Logs complete error context for support/debugging
- [ ] Works on mobile and desktop
- [ ] Has been reviewed by content designer or writer
- [ ] Has been tested with actual users in role
- [ ] Edge cases handled (very long text, multiple simultaneous errors)

### A/B Testing Opportunities

Test different error message approaches to optimize:

1. **Headline tone**: Apologetic ("We're sorry") vs. matter-of-fact ("Unable to load")
2. **Action button labels**: "Try Again" vs. "Reload Page" vs. "Refresh"
3. **Reassurance placement**: At top vs. at bottom
4. **Level of technical detail**: Brief vs. detailed explanations
5. **Icon types**: Emoji vs. system icons vs. custom illustrations

### User Testing Scenarios

Test error messages with real users:

1. **Student (age 8)**: Can they understand what happened and what to do?
2. **Teacher (new)**: Do they feel confident resolving the error?
3. **Teacher (experienced)**: Do they have enough detail to diagnose?
4. **Admin**: Can they assess organizational impact and take action?
5. **Screen reader user**: Does the error make sense without visual cues?

## Permissions

### Who Can View Errors
- **All users**: See errors relevant to their actions and permissions
- **Teachers**: See errors for their students and classes
- **Admins**: See errors for their organization
- **System Admins**: See all errors across all organizations

### Who Can Configure Error Messages
- **Content Designers**: Edit error message text and templates
- **Product Managers**: Approve new error patterns
- **Developers**: Implement error handling logic
- **System Admins**: Configure error logging and alerting thresholds

### Who Can Access Error Logs
- **System Admins**: Full access to all error logs
- **Support Staff**: Access with user permission or for support tickets
- **Organization Admins**: Access to errors within their organization only
- **Individual Users**: Can request their own error history

## Supporting Documents Referenced

This error messaging specification draws from the following source documents:

- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Legacy UI messaging examples including error states and validation feedback
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — Administrative error scenarios and system-level alerts
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Permission-based error contexts and role-specific access restrictions
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform tone and voice standards that inform error messaging approach

## Dependencies

This specification relies on and connects with:

- **[Screen Text and Copy Guidelines](../01-ux-design-system/screen-text-and-copy.md)**: Platform-wide messaging standards that error messages must align with
- **[Copy Guide Students](./copy-guide-students.md)**: Student-specific tone, language, and examples for error messages
- **[Copy Guide Teachers](./copy-guide-teachers.md)**: Teacher-specific tone, language, and examples for error messages
- **[Copy Guide Admins](./copy-guide-admins.md)**: Administrator-specific tone, language, and examples for error messages
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)**: WCAG compliance, screen reader support, keyboard navigation requirements
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Error logging events and telemetry structure
- **[Observability](../19-quality-and-operations/observability.md)**: Error monitoring, alerting, and incident response integration
- **[Incident Response](../19-quality-and-operations/incident-response.md)**: How errors escalate to incidents and system admin response
- **[Localization and Internationalization](../01-ux-design-system/localization-internationalization.md)**: Translation structure and cultural adaptation
- **[API Design Principles](../18-architecture/api-design-principles.md)**: API error response format and status codes
- **[Security and Privacy](../18-architecture/security-privacy.md)**: Sanitizing error messages to prevent information disclosure

## Open Questions

1. **Dynamic Error Severity Adjustment**: Should errors auto-escalate severity based on frequency? For example, if 100 users hit the same "minor" error in 5 minutes, should it auto-promote to "critical" and alert on-call team?

2. **User-Initiated Error Reporting**: Should we allow users to add context to errors they encounter ("This happened after I clicked X")? Would this improve support quality or create noise?

3. **Proactive Error Prevention**: Should we show warnings before actions likely to cause errors? For example: "Your internet connection is slow. Saving large assignments may timeout. Continue anyway?"

4. **Error Recovery Automation**: For certain error types (especially technical errors), should the system automatically retry in the background and only show error if all retries fail?

5. **Child Data Privacy in Error Logs**: How do we balance detailed error logging for support with COPPA/FERPA requirements not to log children's personal information? Should student-related errors use anonymized identifiers in logs?

6. **Multilingual Error Support**: Should users be able to toggle error language independently of interface language? (E.g., Spanish-speaking teacher wants Spanish UI but English error messages for tech support)

7. **Error Message Personalization**: Should error messages adapt based on user's past behavior? For example, show "Try clearing your cache" only to users who haven't done it recently, vs. "Contact support" to those who've tried common fixes?

8. **Gamification of Error Resilience**: For students, should we reward resilient behavior ("You encountered an error but kept practicing! +10 Persistence Points")? Or does this trivialize frustrating experiences?
