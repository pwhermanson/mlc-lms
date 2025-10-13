# Email Templates

## Purpose

This specification defines the email template system for the MLC-LMS platform, establishing comprehensive standards for all email communications sent to users. Email serves as a critical notification delivery channel, reaching users outside the application to drive re-engagement, convey important information, and maintain communication when in-app channels are unavailable.

The email template system provides a centralized, maintainable library of HTML email templates that ensure consistent branding, accessibility, deliverability, and personalization across all user roles and notification types. This system bridges the gap between system-generated events and user email inboxes, supporting the full spectrum of transactional, educational, and administrative communications.

Email templates work in coordination with [In-app Notifications](./in-app-notifications.md) to provide a multi-channel communication experience, with templates serving as the rendering layer for email delivery while notification content and triggers are defined in role-specific documentation.

## Scope

**Included**
- HTML email template structure and responsive design patterns
- Template library organized by category and user role
- Variable substitution and personalization system
- Transactional vs promotional email categorization
- Email-specific content rendering and formatting
- Mobile-first responsive email layouts (desktop, tablet, mobile clients)
- Plain-text fallback generation for all templates
- Email accessibility standards (WCAG 2.1 AA for email)
- Template versioning and update management
- Email preview and testing workflows
- Deliverability best practices (SPF, DKIM, DMARC compliance)
- Unsubscribe mechanisms and preference management
- Localization hooks for multi-language support (future)
- Template inheritance and composition patterns
- Email footer and header standardization

**Excluded**
- Notification trigger logic and business rules (see role-specific notification docs: [Student Notifications](../03-student-experience/student-notifications.md), [Teacher Notifications](../04-teacher-experience/teacher-notifications.md), [Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md))
- Email service provider integration and delivery infrastructure (see [Email Provider](../17-integrations/email-provider.md))
- In-app notification rendering (see [In-app Notifications](./in-app-notifications.md))
- Push notification templates (see [Push and Desktop](./push-and-desktop.md))
- Direct messaging content (see [Student Messages](../03-student-experience/student-messages.md), [Teacher Messaging](../04-teacher-experience/teacher-messaging.md))
- System announcement management (see [System Announcements](./system-announcements.md))
- User notification preferences UI (defined in role experience docs)

## Data model (if applicable)

### EmailTemplate Entity

```typescript
interface EmailTemplate {
  // Identification
  templateId: string;              // Unique identifier, e.g., "tmpl_assignment_due_student"
  templateName: string;            // Human-readable name
  category: EmailCategory;         // Template category
  notificationType?: NotificationType; // Linked to notification type from in-app system
  
  // Versioning
  version: string;                 // Semantic version, e.g., "1.2.3"
  status: 'draft' | 'active' | 'deprecated' | 'archived';
  isDefault: boolean;              // Whether this is the default for category
  parentTemplateId?: string;       // For template inheritance
  
  // Content
  subject: string;                 // Email subject line with variable placeholders
  preheader?: string;              // Preview text (max 100 chars)
  htmlBody: string;                // Full HTML template
  textBody: string;                // Plain-text version
  
  // Metadata
  targetRole: UserRole[];          // Applicable user roles
  language: string;                // ISO 639-1 code, e.g., "en"
  tags: string[];                  // Searchable tags
  
  // Design
  layoutType: 'single-column' | 'two-column' | 'hero' | 'digest';
  primaryColor?: string;           // Override brand color if needed
  includeHeader: boolean;          // Show MLC header with logo
  includeFooter: boolean;          // Show standard footer
  
  // Personalization
  requiredVariables: TemplateVariable[];
  optionalVariables: TemplateVariable[];
  
  // Behavior
  emailType: 'transactional' | 'promotional' | 'educational';
  allowUnsubscribe: boolean;       // Show unsubscribe link
  priority: 'high' | 'normal' | 'low';
  sendingDomain: string;           // Domain for sending (affects deliverability)
  
  // Tracking
  enableOpenTracking: boolean;
  enableClickTracking: boolean;
  utmParameters?: UTMParameters;   // For campaign tracking
  
  // Audit
  createdBy: string;
  createdAt: Date;
  updatedBy: string;
  updatedAt: Date;
  publishedAt?: Date;
  archivedAt?: Date;
}
```

### TemplateVariable

```typescript
interface TemplateVariable {
  key: string;                     // Variable name, e.g., "student_name"
  displayName: string;             // Human-readable name
  description: string;             // What this variable represents
  dataType: 'string' | 'number' | 'date' | 'boolean' | 'url' | 'html';
  format?: string;                 // Format string for dates/numbers
  defaultValue?: string;           // Fallback if data missing
  example: string;                 // Example value for preview
  isRequired: boolean;
  maxLength?: number;
  validation?: string;             // Regex pattern for validation
}
```

### EmailCategory Enum

```typescript
type EmailCategory = 
  // Authentication & Onboarding
  | 'welcome'
  | 'account_verification'
  | 'password_reset'
  | 'account_created'
  
  // Learning & Progress
  | 'assignment_notification'
  | 'progress_update'
  | 'achievement'
  | 'milestone'
  | 'sequence_completion'
  
  // Engagement & Reminders
  | 'activity_reminder'
  | 'inactivity_nudge'
  | 'streak_alert'
  | 'challenge_invitation'
  
  // Communication
  | 'message_received'
  | 'message_reply'
  | 'teacher_feedback'
  
  // Administrative
  | 'subscription_update'
  | 'billing_notification'
  | 'user_management'
  | 'report_ready'
  
  // System
  | 'system_announcement'
  | 'maintenance_notice'
  | 'security_alert'
  
  // Promotional
  | 'feature_announcement'
  | 'tips_and_tricks'
  | 'newsletter';
```

### UTMParameters

```typescript
interface UTMParameters {
  source: string;                  // utm_source (e.g., "mlc_platform")
  medium: string;                  // utm_medium (e.g., "email")
  campaign: string;                // utm_campaign (e.g., "assignment_reminder_2025q1")
  term?: string;                   // utm_term
  content?: string;                // utm_content (for A/B testing)
}
```

### EmailRendering Entity

```typescript
interface EmailRendering {
  renderingId: string;             // Unique rendering instance
  templateId: string;              // Source template
  recipientId: string;             // User receiving email
  variables: Record<string, any>;  // Actual variable values
  
  // Output
  renderedSubject: string;         // Subject with variables substituted
  renderedHtml: string;            // HTML with variables substituted
  renderedText: string;            // Plain text with variables substituted
  
  // Metadata
  renderedAt: Date;
  sentAt?: Date;
  deliveredAt?: Date;
  openedAt?: Date;
  clickedAt?: Date;
  bouncedAt?: Date;
  
  // Tracking
  messageId: string;               // Email provider message ID
  deliveryStatus: 'pending' | 'sent' | 'delivered' | 'bounced' | 'failed';
  bounceReason?: string;
  
  // Engagement
  openCount: number;
  clickCount: number;
  clickedLinks: string[];          // URLs clicked
}
```

### Template Inheritance Structure

```typescript
interface TemplateInheritance {
  baseTemplateId: string;          // Parent template
  childTemplateId: string;         // Derived template
  inheritedSections: ('header' | 'footer' | 'styles' | 'layout')[];
  overriddenSections: Record<string, string>;
}
```

## Behavior and rules

### Template Rendering Pipeline

#### 1. Template Selection
- System selects appropriate template based on:
  - Notification type (from in-app notification system)
  - Target user role
  - User language preference (defaults to "en")
  - Template status: only `active` templates used
- Falls back to default template if specific template unavailable
- Logs warning if no appropriate template found

#### 2. Variable Resolution
```javascript
// Variable resolution order
1. User data (from user profile)
2. Event data (from triggering event)
3. System data (current date, platform name, etc.)
4. Template default values
5. Global fallback: "[Data Not Available]"
```

**Required vs Optional Variables**:
- **Required**: Rendering fails if missing, email not sent
- **Optional**: Uses default value or omits section if missing
- Validation performed before rendering begins

#### 3. Personalization Rules

**Subject Line Personalization**:
```
"{{student_name}}, your assignment is due {{due_date_relative}}"
→ "Sarah, your assignment is due in 2 hours"
```

**Content Personalization**:
- First name in greeting (required)
- Role-specific content blocks
- Conditional sections based on user attributes
- Dynamic CTAs based on user state

**Date/Time Formatting**:
- All dates formatted in user's timezone
- Relative dates when appropriate ("in 2 hours", "tomorrow", "next week")
- Absolute dates for distant future ("March 15, 2025")

#### 4. HTML Generation

**Responsive Design**:
- Mobile-first approach, tested on iOS Mail, Gmail, Outlook
- Fluid layouts using percentage widths
- Breakpoint at 600px for mobile vs desktop
- Simplified layouts for narrow viewports

**Inline CSS**:
- All CSS inlined for email client compatibility
- No external stylesheets
- Limited CSS properties (email client safe)
- Tested against Email on Acid / Litmus standards

**Image Handling**:
- All images served from CDN with HTTPS
- Alt text required for all images
- Fallback background colors for blocked images
- Retina-ready images (@2x versions)

#### 5. Plain-text Generation

**Auto-generation Rules**:
- HTML-to-text conversion with formatting preservation
- Preserve headings with markdown-style notation
- Convert links to readable format: "Link text: https://..."
- Preserve bullet points and numbered lists
- Remove decorative elements and spacers

**Manual Override**:
- Authors can provide custom plain-text version
- Useful for complex layouts that don't convert well
- Must include same information as HTML version

#### 6. Accessibility Requirements

Reference: [Accessibility Standards](../01-ux-design-system/a11y-standards.md)

**Email-Specific Accessibility**:
- Semantic HTML with proper heading hierarchy
- Alt text for all meaningful images
- Color contrast ratio ≥4.5:1 for text
- Font size minimum 14px (16px preferred)
- Sufficient line height (1.5x minimum)
- Clickable areas minimum 44x44px
- Clear focus indicators for links
- Descriptive link text (no "click here")

**Screen Reader Optimization**:
- `role` attributes for email structure
- `aria-label` for icon buttons
- Hidden spacer images (`role="presentation"`)
- Logical reading order in source

### Template Versioning and Updates

#### Version Management
- **Semantic versioning**: Major.Minor.Patch
  - **Major**: Breaking changes to required variables
  - **Minor**: New optional variables or sections
  - **Patch**: Bug fixes, copy tweaks, styling updates
  
- **Deployment rules**:
  - Only one `active` version per template ID
  - Previous versions marked `deprecated` or `archived`
  - Deprecation period: 30 days before archival
  - Rollback capability to previous version

#### A/B Testing Support
```typescript
interface TemplateVariant {
  variantId: string;
  baseTemplateId: string;
  variantName: string;              // e.g., "Subject Line Test A"
  trafficAllocation: number;        // Percentage (0-100)
  overrides: {
    subject?: string;
    htmlBody?: string;
    textBody?: string;
  };
  testStartDate: Date;
  testEndDate?: Date;
  winnerDeclaredAt?: Date;
}
```

**A/B testing rules**:
- Maximum 3 variants per template
- Minimum sample size: 100 sends per variant
- Test duration: 7-14 days recommended
- Automatic winner selection based on engagement metrics

### Transactional vs Promotional Classification

#### Transactional Emails
**Characteristics**:
- Triggered by user action or system event
- Contain user-specific, time-sensitive information
- Cannot be unsubscribed from (required by service)
- High sending priority and deliverability

**Examples**:
- Password reset
- Assignment due reminders
- Account verification
- Payment receipts
- Security alerts

**Regulations**:
- CAN-SPAM compliant (physical address required)
- Unsubscribe not required but preferences link included
- No promotional content allowed
- Must be directly related to triggering action

#### Promotional Emails
**Characteristics**:
- Marketing or engagement-focused
- Not required for platform function
- Must include unsubscribe link
- Lower priority in sending queue

**Examples**:
- Feature announcements
- Newsletter
- Tips and tricks
- Re-engagement campaigns
- Seasonal promotions

**Regulations**:
- CAN-SPAM Act full compliance
- Clear unsubscribe mechanism
- Honor opt-out within 10 business days
- Clearly identify as marketing content

### Deliverability Best Practices

#### Technical Requirements
- **SPF Record**: Publish SPF record for sending domain
- **DKIM Signing**: Sign all emails with DKIM
- **DMARC Policy**: Implement DMARC with monitoring
- **Return-Path**: Configure valid return-path domain
- **List-Unsubscribe Header**: Include RFC 8058 compliant header

#### Content Rules to Avoid Spam Filters
- **Subject lines**:
  - Avoid ALL CAPS
  - Limit exclamation marks (max 1)
  - No misleading subject lines
  - Keep under 60 characters
  
- **Body content**:
  - Maintain text-to-image ratio >60% text
  - Avoid spam trigger words ("free", "act now", etc.)
  - Include proper unsubscribe link
  - Use valid from address with real domain
  
- **HTML structure**:
  - Valid HTML5 markup
  - No broken tags or excessive nesting
  - Avoid hidden text (white text on white background)
  - No form elements in email body

#### Bounce Handling
- **Hard bounces**: Invalid email address, permanently undeliverable
  - Mark email as invalid after first hard bounce
  - Remove from send list automatically
  - Notify admin if guardian email bounces (students)

- **Soft bounces**: Temporary delivery failure (mailbox full, server down)
  - Retry up to 3 times over 72 hours
  - Mark as hard bounce if still failing
  - Log all bounce reasons for analysis

#### Reputation Management
- **Sending limits**:
  - Warm-up new sending domains gradually
  - Monitor sending reputation scores
  - Throttle sends to avoid rate limiting
  
- **Engagement tracking**:
  - Monitor open rates by template
  - Track complaint rates (must stay <0.1%)
  - Identify inactive recipients
  - Implement re-engagement campaigns before removing

### Unsubscribe and Preference Management

#### One-Click Unsubscribe
- **RFC 8058 List-Unsubscribe Header**:
  ```
  List-Unsubscribe: <https://mlc.example.com/unsubscribe?token=abc123>
  List-Unsubscribe-Post: List-Unsubscribe=One-Click
  ```
- **Unsubscribe button** in email footer
- **Confirmation page** with option to resubscribe
- **No login required** for unsubscribe action

#### Preference Center
- **Granular controls** by email category:
  - Assignments and learning
  - Achievements and gamification
  - Administrative and billing
  - Tips and feature announcements
  
- **Frequency options**:
  - Immediate (real-time)
  - Daily digest
  - Weekly summary
  - Off (except transactional)
  
- **Quiet hours** configuration:
  - Respect in-app quiet hours (see [In-app Notifications](./in-app-notifications.md))
  - Email-specific quiet hours if different
  - Batch emails outside quiet hours

#### Regulatory Compliance
- Honor unsubscribe requests within **10 business days** (CAN-SPAM)
- Retain unsubscribe records for **5 years**
- Never re-subscribe users without explicit opt-in
- Provide clear privacy policy link in all emails
- Include physical mailing address in footer (required by CAN-SPAM)

## UX requirements (if applicable)

### Email Design System

#### Layout Patterns

**Single-Column Layout** (Most common)
```
┌────────────────────────────────┐
│  Header: Logo + Brand          │
├────────────────────────────────┤
│  Hero Image (optional)         │
├────────────────────────────────┤
│  Primary Message               │
│  Body content                  │
│  Supporting details            │
├────────────────────────────────┤
│  Call-to-Action Button         │
├────────────────────────────────┤
│  Secondary Information         │
├────────────────────────────────┤
│  Footer: Links + Unsubscribe   │
└────────────────────────────────┘
```

**Two-Column Layout** (Digest/Summary emails)
```
┌────────────────────────────────┐
│  Header: Logo + Brand          │
├────────────────────────────────┤
│ ┌─────────────┬──────────────┐ │
│ │  Content    │  Content     │ │
│ │  Block 1    │  Block 2     │ │
│ └─────────────┴──────────────┘ │
│ ┌─────────────┬──────────────┐ │
│ │  Content    │  Content     │ │
│ │  Block 3    │  Block 4     │ │
│ └─────────────┴──────────────┘ │
├────────────────────────────────┤
│  Footer                        │
└────────────────────────────────┘
```

**Hero Layout** (Announcements, achievements)
```
┌────────────────────────────────┐
│  Full-Width Hero Image         │
│  Overlay Text + CTA            │
├────────────────────────────────┤
│  Body Content                  │
│  Supporting information        │
├────────────────────────────────┤
│  Secondary CTA                 │
├────────────────────────────────┤
│  Footer                        │
└────────────────────────────────┘
```

#### Email Header Component

**Standard Header** (all emails):
- MLC logo (200px width, left-aligned)
- Optional tagline: "Playground for Lifetime Musicians™"
- Background: Brand color or white
- Height: 80px desktop, 60px mobile
- Linked to platform dashboard

**Variation by role**:
- Students: Playful header with Terry Treble character
- Teachers: Professional header with logo only
- Admins: Streamlined header with organization name

#### Email Footer Component

**Required Elements** (all emails):
- Company name: "MusicLearningCommunity.com LLC"
- Physical address (CAN-SPAM requirement)
- Copyright notice: "© 2025 MusicLearningCommunity.com"
- Unsubscribe link (if promotional)
- Privacy policy link
- Help center link

**Optional Elements**:
- Social media icons (if applicable)
- Contact information
- Preference center link
- View in browser link

#### Typography Standards

**Font Stack**:
```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 
             Helvetica, Arial, sans-serif, 'Apple Color Emoji';
```

**Font Sizes**:
- **Heading 1**: 28px (24px mobile)
- **Heading 2**: 22px (20px mobile)
- **Heading 3**: 18px (16px mobile)
- **Body text**: 16px (14px minimum)
- **Small text**: 14px (footer, disclaimers)
- **Button text**: 16px (bold)

**Line Height**:
- Body text: 1.5x (24px for 16px font)
- Headings: 1.3x
- Button text: 1x (centered vertically)

#### Color Palette

Reference: Brand colors from [UX Design System](../01-ux-design-system/components-overview.md)

**Primary Colors**:
- **Brand Primary**: #0066CC (links, primary CTA)
- **Brand Secondary**: #FF6B35 (accent, highlights)
- **Success**: #28A745 (positive messages)
- **Warning**: #FFC107 (alerts, caution)
- **Error**: #DC3545 (errors, urgent)
- **Neutral**: #6C757D (secondary text)

**Background Colors**:
- **Email background**: #F4F4F4 (light gray)
- **Content container**: #FFFFFF (white)
- **Footer background**: #333333 (dark gray)

**Text Colors**:
- **Primary text**: #212529 (near black)
- **Secondary text**: #6C757D (gray)
- **Link text**: #0066CC (brand primary)
- **Footer text**: #FFFFFF (white on dark)

#### Call-to-Action (CTA) Buttons

**Primary CTA Button**:
```html
<table role="presentation" cellspacing="0" cellpadding="0" border="0">
  <tr>
    <td style="border-radius: 4px; background: #0066CC;">
      <a href="{{cta_url}}" style="display: inline-block; 
         padding: 14px 24px; font-size: 16px; color: #ffffff; 
         text-decoration: none; font-weight: bold;">
        {{cta_text}}
      </a>
    </td>
  </tr>
</table>
```

**Button Specifications**:
- Minimum width: 200px
- Height: 44px (minimum touch target)
- Border radius: 4px
- Padding: 14px vertical, 24px horizontal
- Font weight: Bold (600-700)
- Text transform: None (sentence case)
- Margin: 20px above/below

**Button Hierarchy**:
- **Primary**: Solid background, brand color
- **Secondary**: Outlined, transparent background
- **Tertiary**: Text link, underlined on hover

#### Responsive Breakpoints

**Desktop (≥600px)**:
- Email width: 600px
- Container padding: 40px
- Two-column layouts enabled
- Larger images and text

**Mobile (<600px)**:
- Email width: 100% (with max-width: 600px)
- Container padding: 20px
- Single-column layouts only
- Stacked content blocks
- Larger touch targets (44x44px minimum)

**Responsive Techniques**:
```html
<!-- Fluid tables -->
<table width="100%" style="max-width: 600px;">

<!-- Responsive images -->
<img src="image.jpg" width="100%" style="max-width: 600px; height: auto;">

<!-- Media queries -->
<style>
  @media only screen and (max-width: 600px) {
    .mobile-hide { display: none !important; }
    .mobile-full { width: 100% !important; }
    .mobile-padding { padding: 20px !important; }
  }
</style>
```

### Template Examples by Category

#### Welcome Email (New Student)

**Subject**: `Welcome to MusicLearningCommunity, {{student_name}}! 🎵`

**Content Structure**:
- Hero image: Terry Treble welcoming graphic
- Personalized greeting
- Brief platform overview (3-4 sentences)
- "Start Your First Game" CTA button
- Quick tips section (3 bullet points)
- Help resources link
- Footer with support contact

**Key Variables**:
- `{{student_name}}`: Student first name
- `{{teacher_name}}`: Assigned teacher name
- `{{first_game_url}}`: Link to first recommended game
- `{{help_center_url}}`: Help center link

#### Assignment Due Reminder (Student)

**Subject**: `{{student_name}}, your assignment "{{assignment_name}}" is due {{due_date_relative}}`

**Content Structure**:
- Icon: Clock or calendar graphic
- Personalized greeting
- Assignment details (name, due date, description)
- Progress indicator (if partially complete)
- "Continue Assignment" CTA button
- Estimated time to complete
- Footer

**Key Variables**:
- `{{student_name}}`: Student first name
- `{{assignment_name}}`: Assignment title
- `{{due_date_relative}}`: "in 2 hours", "tomorrow"
- `{{due_date_absolute}}`: "March 15, 2025 at 3:00 PM"
- `{{progress_percentage}}`: 60 (optional)
- `{{assignment_url}}`: Link to assignment

#### Progress Report (Teacher)

**Subject**: `Weekly Progress Report: {{class_name}}`

**Content Structure**:
- Header with date range
- Summary statistics (cards or table)
  - Total assignments completed
  - Average class progress
  - Students needing attention
- Top performers section (with celebration icons)
- Students needing attention section (with support icons)
- "View Full Report" CTA button
- Footer

**Key Variables**:
- `{{class_name}}`: Class or group name
- `{{start_date}}` / `{{end_date}}`: Report period
- `{{total_assignments_completed}}`: Number
- `{{average_progress}}`: Percentage
- `{{top_students}}`: Array of student objects
- `{{attention_students}}`: Array of student objects
- `{{report_url}}`: Link to full report

#### Payment Confirmation (Admin)

**Subject**: `Payment Received - MusicLearningCommunity Subscription`

**Content Structure**:
- Confirmation icon (green checkmark)
- "Payment Successful" heading
- Transaction details (table):
  - Date
  - Amount
  - Payment method
  - Transaction ID
- Subscription details
  - Plan name
  - Billing cycle
  - Next billing date
  - Number of seats
- "View Invoice" CTA button
- "Manage Subscription" secondary link
- Footer

**Key Variables**:
- `{{organization_name}}`: Subscriber organization
- `{{payment_date}}`: Transaction date
- `{{amount}}`: Dollar amount
- `{{payment_method}}`: Last 4 digits of card
- `{{transaction_id}}`: Reference number
- `{{plan_name}}`: Subscription plan
- `{{next_billing_date}}`: Next charge date
- `{{invoice_url}}`: PDF invoice link
- `{{subscription_url}}`: Subscription management link

### Email Accessibility Checklist

#### Visual Accessibility
- [ ] Minimum font size 14px for body text
- [ ] Color contrast ratio ≥4.5:1 for all text
- [ ] Information not conveyed by color alone
- [ ] Alt text provided for all meaningful images
- [ ] Decorative images have empty alt text (`alt=""`)
- [ ] Images have fallback background colors
- [ ] Sufficient spacing between interactive elements

#### Structural Accessibility
- [ ] Semantic HTML5 elements used
- [ ] Proper heading hierarchy (h1 → h2 → h3)
- [ ] Tables used only for tabular data (with proper markup)
- [ ] Links have descriptive text (not "click here")
- [ ] ARIA landmarks for major sections
- [ ] Logical reading order in source code

#### Cognitive Accessibility
- [ ] Clear, concise subject line
- [ ] Content organized with headings and lists
- [ ] Primary action is obvious and prominent
- [ ] Consistent layout and navigation
- [ ] No flashing or animated GIFs

#### Screen Reader Optimization
- [ ] `role="presentation"` on layout tables
- [ ] `role="article"` on main content container
- [ ] `aria-label` for icon-only buttons
- [ ] Hidden spacer images and cells
- [ ] Language attribute: `<html lang="en">`

## Telemetry

Reference: [Event Model](../15-analytics-and-reporting/event-model.md) for complete event tracking specifications.

### Email Lifecycle Events

#### Email Template Rendered
```javascript
{
  event_type: "email_template_rendered",
  properties: {
    template_id: "tmpl_assignment_due_student",
    template_version: "1.2.3",
    template_category: "assignment_notification",
    recipient_id: "usr_12345",
    recipient_role: "student",
    notification_id: "notif_abc123",  // Links to in-app notification
    variables_count: 8,
    rendering_duration_ms: 45,
    has_personalization: true
  }
}
// Fires: Server-side when email HTML is generated
```

#### Email Sent
```javascript
{
  event_type: "email_sent",
  properties: {
    template_id: "tmpl_assignment_due_student",
    recipient_id: "usr_12345",
    recipient_email: "student@example.com",  // Hashed for privacy
    subject: "Sarah, your assignment is due in 2 hours",
    email_type: "transactional",
    priority: "high",
    sending_domain: "notifications.mlc.example.com",
    provider_message_id: "msg_xyz789",
    sent_at: "2025-01-15T14:30:00Z"
  }
}
// Fires: Server-side after email handed to provider
```

#### Email Delivered
```javascript
{
  event_type: "email_delivered",
  properties: {
    template_id: "tmpl_assignment_due_student",
    recipient_id: "usr_12345",
    provider_message_id: "msg_xyz789",
    delivery_time_ms: 1250,
    smtp_response: "250 OK",
    delivered_at: "2025-01-15T14:30:01Z"
  }
}
// Fires: Webhook from email provider on successful delivery
```

#### Email Opened
```javascript
{
  event_type: "email_opened",
  properties: {
    template_id: "tmpl_assignment_due_student",
    recipient_id: "usr_12345",
    provider_message_id: "msg_xyz789",
    time_to_open_seconds: 120,  // Time from delivery to open
    client: "iOS Mail",          // Email client detected
    device_type: "mobile",
    opened_at: "2025-01-15T14:32:01Z",
    is_unique_open: true
  }
}
// Fires: Email provider tracking pixel loaded
```

#### Email Clicked
```javascript
{
  event_type: "email_clicked",
  properties: {
    template_id: "tmpl_assignment_due_student",
    recipient_id: "usr_12345",
    provider_message_id: "msg_xyz789",
    link_url: "https://mlc.example.com/assignment/123",
    link_text: "Continue Assignment",
    link_position: "primary_cta",
    time_to_click_seconds: 135,
    clicked_at: "2025-01-15T14:32:15Z",
    is_unique_click: true
  }
}
// Fires: User clicks tracked link in email
```

#### Email Bounced
```javascript
{
  event_type: "email_bounced",
  properties: {
    template_id: "tmpl_assignment_due_student",
    recipient_id: "usr_12345",
    recipient_email: "student@example.com",  // Hashed
    provider_message_id: "msg_xyz789",
    bounce_type: "hard" | "soft",
    bounce_reason: "mailbox_full" | "invalid_address" | "blocked",
    smtp_code: "550",
    bounced_at: "2025-01-15T14:30:02Z"
  }
}
// Fires: Webhook from email provider on bounce
```

#### Email Spam Complaint
```javascript
{
  event_type: "email_spam_complaint",
  properties: {
    template_id: "tmpl_assignment_due_student",
    recipient_id: "usr_12345",
    provider_message_id: "msg_xyz789",
    complaint_source: "outlook" | "gmail" | "yahoo",
    complained_at: "2025-01-15T15:00:00Z"
  }
}
// Fires: Webhook from email provider on spam complaint
```

#### Email Unsubscribed
```javascript
{
  event_type: "email_unsubscribed",
  properties: {
    template_id: "tmpl_assignment_due_student",
    recipient_id: "usr_12345",
    unsubscribe_method: "one_click" | "preference_center" | "reply",
    unsubscribe_category: "all" | "assignment_notification",
    unsubscribed_at: "2025-01-15T16:00:00Z"
  }
}
// Fires: User unsubscribes via email link or preference center
```

### Template Management Events

#### Template Created
```javascript
{
  event_type: "email_template_created",
  properties: {
    template_id: "tmpl_new_feature_announcement",
    template_name: "New Feature Announcement",
    category: "feature_announcement",
    created_by: "usr_admin_123",
    version: "1.0.0",
    status: "draft"
  }
}
// Fires: Admin creates new template in CMS
```

#### Template Published
```javascript
{
  event_type: "email_template_published",
  properties: {
    template_id: "tmpl_new_feature_announcement",
    previous_status: "draft",
    new_status: "active",
    version: "1.0.0",
    published_by: "usr_admin_123",
    published_at: "2025-01-15T10:00:00Z"
  }
}
// Fires: Template moved from draft to active
```

#### Template A/B Test Started
```javascript
{
  event_type: "email_template_ab_test_started",
  properties: {
    test_id: "test_subject_line_001",
    template_id: "tmpl_assignment_due_student",
    variant_count: 2,
    variants: [
      { variant_id: "var_a", traffic: 50 },
      { variant_id: "var_b", traffic: 50 }
    ],
    test_metric: "open_rate",
    test_duration_days: 7,
    started_at: "2025-01-15T00:00:00Z"
  }
}
// Fires: A/B test begins
```

### Aggregated Email Metrics

These metrics feed into dashboards and reports:

#### Deliverability Metrics
- **Delivery rate**: delivered / sent (target: >99%)
- **Bounce rate**: bounced / sent (target: <2%)
  - Hard bounce rate (target: <1%)
  - Soft bounce rate (target: <1%)
- **Spam complaint rate**: complaints / delivered (target: <0.1%)
- **Unsubscribe rate**: unsubscribes / delivered (by category)

#### Engagement Metrics
- **Open rate**: unique opens / delivered (by template category)
- **Click rate**: unique clicks / delivered (by template category)
- **Click-to-open rate**: unique clicks / unique opens
- **Time to open**: average time from delivery to first open
- **Time to click**: average time from delivery to first click

#### Template Performance
- **Open rate by template**: Compare templates within category
- **Click rate by template**: Identify high-performing templates
- **Unsubscribe rate by template**: Identify problematic templates
- **A/B test results**: Winner determination based on engagement

#### User Engagement Segments
- **Highly engaged**: Opens >50% of emails, clicks >20%
- **Moderately engaged**: Opens 20-50%, clicks 5-20%
- **Inactive**: Opens <20%, clicks <5%
- **Disengaged**: No opens in 30 days

## Permissions

Reference: [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) and [Permissions Table](../02-roles-and-permissions/permissions-table.md)

### Email Template Management Permissions

#### System Admin
- **Create templates**: Create new email templates
- **Edit templates**: Modify all templates (draft, active, deprecated)
- **Publish templates**: Move templates from draft to active
- **Deprecate templates**: Mark templates as deprecated
- **Archive templates**: Remove templates from active use
- **Delete templates**: Permanently delete archived templates
- **Version control**: Create new versions, rollback changes
- **A/B testing**: Create and manage A/B test variants
- **View analytics**: Access all email performance metrics
- **Export templates**: Download template HTML/data
- **Import templates**: Upload template packages

#### Subscriber Admin
- **View templates**: Read-only access to active templates
- **Preview templates**: Generate preview with sample data
- **View organization analytics**: Email metrics for their organization only
- **Cannot modify**: No edit/publish permissions

#### Teacher / Teacher-Admin / Student
- **No access**: Cannot view or manage email templates
- **Receive emails**: Can receive emails per notification preferences
- **Manage preferences**: Can adjust email notification settings
- **Unsubscribe**: Can opt out of promotional emails

### Email Sending Permissions

#### Automated System
- **Transactional emails**: Send all transactional emails based on triggers
- **Promotional emails**: Send promotional emails to users with opt-in
- **Respect preferences**: Honor user notification preferences
- **Throttle sends**: Enforce rate limits and quiet hours

#### System Admin
- **Manual send**: Trigger individual email sends for testing
- **Broadcast**: Send platform-wide announcements
- **Override preferences**: Send critical emails regardless of preferences (security only)
- **Retry failed sends**: Manually retry failed deliveries

#### Subscriber Admin
- **Organization broadcasts**: Send emails to all users in their organization
- **Cannot override**: Must respect user unsubscribe preferences
- **Preview before send**: Must preview emails before broadcast

### Email Data Access Permissions

#### System Admin
- **All email data**: Access all sent emails, delivery logs, engagement metrics
- **User email addresses**: View user email addresses across organizations
- **Bounce logs**: Access bounce and complaint data
- **Audit trails**: View complete email sending audit logs

#### Subscriber Admin
- **Organization data**: Email metrics and logs for their organization only
- **Aggregated metrics**: Cannot see individual user email addresses (privacy)
- **Delivery status**: Can see if emails to users in their org were delivered
- **Cannot export**: Cannot export raw email address lists

#### Teacher / Teacher-Admin
- **No email data access**: Cannot view email delivery logs or metrics
- **Own preferences**: Can view/edit their own email preferences only

#### Student
- **No email data access**: Cannot view system email data
- **Own preferences**: Can manage their own notification settings
- **Guardian emails**: Students cannot view guardian email addresses

### Privacy and Data Protection Rules

- **Email addresses**: Treated as PII, access restricted per GDPR/FERPA
- **Content retention**: Email content retained per retention policy (default: 2 years)
- **Audit logs**: Email sending audit logs retained for 5 years (compliance)
- **Anonymization**: Email addresses anonymized in analytics after 90 days
- **Right to erasure**: User can request deletion of email data (GDPR)
- **Data export**: User can request export of their email history (GDPR)
- **COPPA compliance**: Student emails sent to guardian address if under 13
- **FERPA compliance**: Educational content in emails protected per role boundaries

## Supporting Documents Referenced

This email templates specification draws from the following source documents:

- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform messaging and email communication content
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — User interface text and messaging guidelines
- [AMessagetoParents.pdf](../00-ORIG-CONTEXT/AMessagetoParents.pdf) — Parent communication messaging and tone examples

## Dependencies

### Core System Dependencies

- **[In-app Notifications](./in-app-notifications.md)**: Notification types and triggers that generate emails
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: System events that trigger email notifications
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: User role definitions for email personalization
- **[Session and Auth](../02-roles-and-permissions/session-and-auth-states.md)**: User authentication for email linking

### Role-Specific Notification Sources

- **[Student Notifications](../03-student-experience/student-notifications.md)**: Student email triggers and content requirements
- **[Teacher Notifications](../04-teacher-experience/teacher-notifications.md)**: Teacher email triggers and content
- **[Admin Notifications](../05-admin-subscriber-experience/admin-notifications.md)**: Admin email triggers and content

### Messaging System Integration

- **[Student Messages](../03-student-experience/student-messages.md)**: Direct message email notifications
- **[Teacher Messaging](../04-teacher-experience/teacher-messaging.md)**: Teacher message email notifications
- **[System Announcements](./system-announcements.md)**: Broadcast email announcements
- **[Communication Framework](./communication-framework.md)**: Overall communication policies

### Technical Infrastructure

- **[Email Provider Integration](../17-integrations/email-provider.md)**: ESP integration (SendGrid, AWS SES, etc.)
- **[Background Jobs](../18-architecture/background-jobs.md)**: Scheduled email digest generation
- **[API Design](../18-architecture/api-design-principles.md)**: Email template CRUD API endpoints
- **[CDN](../17-integrations/cdn-and-media.md)**: Image hosting for email assets
- **[Webhooks](../18-architecture/webhooks.md)**: Email provider webhook handling

### UI and Design Dependencies

- **[Design System](../01-ux-design-system/components-overview.md)**: Brand colors, typography, iconography
- **[Screen Text and Copy](../01-ux-design-system/screen-text-and-copy.md)**: Writing guidelines for email copy
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)**: WCAG compliance for email
- **[Localization](../01-ux-design-system/localization-internationalization.md)**: Multi-language email support (future)

### Content and Compliance

- **[Copy Guidelines](../22-content-style-guides/copy-guidelines.md)**: Editorial standards for email content
- **[Privacy Policy](../23-legal-and-compliance/privacy-policy.md)**: Privacy requirements for email communications
- **[COPPA/FERPA Compliance](../23-legal-and-compliance/coppa-ferpa-notes.md)**: Student email restrictions
- **[GDPR Compliance](../23-legal-and-compliance/gdpr-compliance.md)**: EU user email requirements (if applicable)

## Open questions

### Technical Implementation

- **Email Service Provider**: Which ESP should we use? Options: SendGrid (strong API, good docs), AWS SES (cost-effective, scalable), Mailgun (good deliverability). Recommendation: Start with SendGrid for ease of integration, evaluate AWS SES for cost at scale.

- **Template Engine**: Which templating system? Options: Handlebars (widely adopted, good logic support), Liquid (Shopify's engine, simpler), MJML (email-specific, great responsive output). Recommendation: MJML for responsive email generation, compiled to HTML.

- **Template Storage**: Store templates in database or version-controlled files? Database allows dynamic editing, files enable version control and code review. Recommendation: Hybrid approach - source in Git, compiled versions in database.

- **Image Hosting**: Self-host email images on CDN or use ESP image hosting? Self-hosting gives control, ESP hosting simplifies inline images. Recommendation: CDN for production images, ESP for dynamic/generated images.

### User Experience

- **Preview Text**: Should we auto-generate preheader text from email body, or require manual entry? Auto-generation is easier, manual gives better control. Recommendation: Default to auto-generation with manual override option.

- **Dark Mode**: Should we support email dark mode (iOS Mail, Outlook)? Requires duplicate color schemes and media queries. Recommendation: Phase 2 - implement for high-engagement templates first.

- **Digest Frequency**: What default digest frequencies should we offer? Daily and weekly are standard. Recommendation: Daily digest at 8 AM user local time, weekly on Sunday evening.

- **Email Footers**: How much content in footer? Minimal (just required links) or expanded (help, social, etc.)? Recommendation: Minimal for transactional, expanded for promotional/newsletters.

### Content and Personalization

- **Subject Line Length**: Optimal subject line length? Research shows 41-50 characters ideal for mobile. Recommendation: 50 character soft limit, 70 hard limit with warning.

- **Emoji in Subject Lines**: Allow emoji in subject lines for student emails? Can improve engagement but may look unprofessional. Recommendation: Allow for student/parent emails, restrict for teacher/admin.

- **Dynamic Content Blocks**: How granular should template personalization be? Balance between flexibility and complexity. Recommendation: Start with role-based content blocks, expand to behavioral targeting in Phase 2.

- **Video in Email**: Should we support embedded video or animated GIFs? Limited email client support but high engagement. Recommendation: Static thumbnail with link to video (universal support).

### Deliverability and Compliance

- **Sending Domain**: Use subdomain (notifications.mlc.example.com) or main domain? Subdomain isolates reputation but may reduce trust. Recommendation: Subdomain for transactional, main domain for promotional after reputation established.

- **IP Reputation**: Dedicated or shared IP address? Dedicated gives control but requires warming. Recommendation: Start with ESP shared IPs, move to dedicated when sending 100K+ emails/month.

- **Double Opt-In**: Require double opt-in for promotional emails? More compliant but reduces list size. Recommendation: Single opt-in at account creation, confirmation email (not double opt-in).

- **Re-engagement Cadence**: How often to re-engage inactive users before suppressing? Balance between giving chances and respecting disinterest. Recommendation: 2 re-engagement campaigns over 60 days, then suppress from promotional emails.

### Localization and Internationalization

- **Translation Workflow**: How to manage multi-language email templates? Need translation management system. Recommendation: Phase 2 - start with English only, add Spanish in Q3 2025 if demand warrants.

- **Cultural Adaptation**: Should templates be culturally adapted beyond translation? Different imagery, holidays, etc. Recommendation: Evaluate after first international market entry.

- **Timezone Handling**: Send emails in user's local timezone or UTC? Local timezone improves engagement. Recommendation: Convert all send times to user's timezone (from user profile).

- **Date Formatting**: Use ISO dates, local formats, or relative dates? Consistency vs cultural norms. Recommendation: Relative dates for near-term ("tomorrow"), localized format for far future ("March 15, 2025").

### Testing and Quality Assurance

- **Email Testing Tools**: Which tool for cross-client testing? Options: Litmus (comprehensive, expensive), Email on Acid (good value), Mailtrap (affordable, limited). Recommendation: Email on Acid for pre-launch testing, spot-check with Litmus quarterly.

- **Automated Testing**: How to automate email rendering tests? Prevent regressions. Recommendation: Automated screenshot comparison for critical templates (assignment reminders, welcome emails).

- **A/B Test Duration**: How long to run A/B tests? Balance statistical significance and speed. Recommendation: Minimum 7 days or 1000 sends per variant, whichever is longer.

- **Preview Recipients**: Who should receive template previews before launch? Internal testing group. Recommendation: Create "Email QA" group with system admin and willing volunteers from each role.

### Performance and Scalability

- **Send Rate Limits**: What are appropriate sending rate limits? Prevent overwhelming users and ESP. Recommendation: 100 emails/second to ESP, max 5 emails/user/day (except transactional).

- **Batch Size**: How many emails in each batch send? Affects memory and performance. Recommendation: 1000 emails per batch job, stagger batches 30 seconds apart.

- **Retry Policy**: How many retries for failed sends? Balance between persistence and resource usage. Recommendation: Soft fails: 3 retries over 24 hours. Hard fails: No retry, log and alert.

- **Archive Strategy**: When to archive old sent emails? Database growth management. Recommendation: Archive sent emails after 90 days to cold storage, purge after 2 years (except compliance-flagged).
