# Checkout and Trials

## Purpose

This specification defines the checkout process and trial management system for the MLC-LMS platform. It enables prospective customers to evaluate the platform through free trials and convert to paid subscriptions through a streamlined checkout experience. The system supports the platform's three-tier pricing model (Prelude, Solo, Ensemble) and provides seamless transitions between trial and paid states.

The checkout and trial system is critical for customer acquisition, conversion optimization, and providing a frictionless path from evaluation to subscription. It integrates with the platform's seat-based pricing model and supports both individual music teachers and educational institutions.

## Scope

### Included
- Free trial registration and management (Prelude plan)
- Checkout flow for Solo and Ensemble plan subscriptions
- Trial-to-paid conversion processes
- Payment method collection and validation
- Seat count selection and pricing calculations
- Billing cycle selection (monthly/annual)
- Promotional code application
- Trial expiration and conversion reminders
- Account setup and user onboarding
- Integration with payment processing systems

### Excluded
- Detailed billing operations (see [billing-plans-and-seats.md](billing-plans-and-seats.md))
- Seat management after subscription (see [seat-management.md](seat-management.md))
- Promotional code creation and management (see [promo-codes.md](promo-codes.md))
- Payment processing integration details (see [billing-provider-integration.md](billing-provider-integration.md))
- User account management beyond initial setup (see [member-management.md](../05-admin-subscriber-experience/member-management.md))

## Data Model

### Core Entities

#### Trial Account
- `trial_id` (UUID, Primary Key)
- `email` (string, unique)
- `plan_type` (enum: prelude)
- `seat_count` (integer, 1-19)
- `start_date` (timestamp)
- `expiration_date` (timestamp)
- `status` (enum: active, expired, converted, cancelled)
- `conversion_plan` (enum: solo, ensemble, nullable)
- `created_at` (timestamp)
- `last_accessed_at` (timestamp, nullable)
- `metadata` (JSON, for additional trial data)

#### Checkout Session
- `session_id` (UUID, Primary Key)
- `user_id` (UUID, Foreign Key, nullable)
- `trial_id` (UUID, Foreign Key, nullable)
- `plan_type` (enum: solo, ensemble)
- `seat_count` (integer)
- `billing_cycle` (enum: monthly, annual)
- `total_amount` (decimal)
- `promo_code` (string, nullable)
- `discount_amount` (decimal, nullable)
- `status` (enum: pending, completed, failed, abandoned)
- `payment_method_id` (string, nullable)
- `created_at` (timestamp)
- `completed_at` (timestamp, nullable)
- `expires_at` (timestamp)

#### Trial Conversion
- `conversion_id` (UUID, Primary Key)
- `trial_id` (UUID, Foreign Key)
- `subscription_id` (UUID, Foreign Key)
- `conversion_plan` (enum: solo, ensemble)
- `conversion_date` (timestamp)
- `conversion_method` (enum: checkout, admin, support)
- `retention_duration` (integer, days)
- `metadata` (JSON, for conversion analytics)

### Relationships
- One trial account can have one checkout session
- One checkout session belongs to one user (nullable for guest checkout)
- One trial account can have one conversion record
- One conversion record belongs to one subscription

## Behavior and Rules

### Trial Management Rules

#### Prelude Plan (Free Trial)
- **Duration**: 14 days (2 weeks) from activation
- **Seat Limit**: Maximum 19 seats
- **Features**: Full platform access with all features enabled
- **No Payment Required**: No credit card information collected
- **Auto-Expiration**: Trials automatically expire after 14 days
- **Conversion Required**: Must convert to Solo or Ensemble before expiration
- **One Trial Per Email**: Each email address can only have one trial
- **Immediate Access**: Full platform access granted upon registration

#### Trial Registration Process
1. **Email Collection**: Valid email address required
2. **Plan Selection**: Prelude plan automatically selected
3. **Seat Count**: User selects 1-19 seats (default: 5)
4. **Account Creation**: Trial account created immediately
5. **Access Granted**: Full platform access provided
6. **Welcome Email**: Confirmation and onboarding information sent

#### Trial Conversion Rules
- **Conversion Window**: Can convert at any time during trial
- **Seat Count**: Can change seat count during conversion
- **Plan Selection**: Can choose Solo or Ensemble plan
- **Billing Cycle**: Can select monthly or annual billing
- **Immediate Access**: No interruption in platform access
- **Data Retention**: All trial data and progress preserved

### Checkout Process Rules

#### Solo Plan Checkout
- **Minimum Seats**: 5 seats required
- **Maximum Seats**: 19 seats maximum
- **Billing Options**: Monthly or annual
- **Payment Required**: Credit card or PayPal required
- **Immediate Activation**: Access granted upon successful payment
- **Proration**: Not applicable for new subscriptions

#### Ensemble Plan Checkout
- **Minimum Seats**: 20 seats required
- **Maximum Seats**: No upper limit
- **Billing Options**: Monthly or annual (annual recommended)
- **Payment Required**: Credit card, PayPal, or purchase order
- **Volume Pricing**: Automatic tier-based pricing applied
- **Immediate Activation**: Access granted upon successful payment
- **Purchase Orders**: Special handling for educational institutions

#### Checkout Flow States
```
[Plan Selection] → [Seat Count] → [Billing Cycle] → [Payment Method] → [Review] → [Payment] → [Confirmation]
```

#### Payment Validation Rules
- **Credit Card**: Valid card number, expiration, and CVV required
- **Address Verification**: Billing address validation
- **Payment Authorization**: Real-time payment authorization
- **Failure Handling**: Clear error messages and retry options
- **Security**: PCI-compliant payment processing

### Trial Expiration and Conversion

#### Expiration Process
1. **Warning Notifications**: 7 days, 3 days, 1 day before expiration
2. **Grace Period**: 24-hour grace period after expiration
3. **Access Suspension**: Platform access suspended after grace period
4. **Data Retention**: User data retained for 30 days
5. **Conversion Reminders**: Ongoing conversion prompts

#### Conversion Process
1. **Plan Selection**: Choose Solo or Ensemble plan
2. **Seat Count**: Confirm or modify seat count
3. **Billing Cycle**: Select monthly or annual billing
4. **Payment Collection**: Credit card or PayPal information
5. **Account Migration**: Trial account converted to subscription
6. **Access Continuation**: Seamless access without interruption

## UX Requirements

### Trial Registration Flow

#### Landing Page
- **Clear Value Proposition**: "Start your 14-day free trial"
- **Feature Highlights**: Key platform benefits and capabilities
- **Social Proof**: Testimonials and user success stories
- **Pricing Preview**: Brief overview of Solo and Ensemble plans
- **Call-to-Action**: Prominent "Start Free Trial" button

#### Registration Form
- **Email Input**: Valid email address with real-time validation
- **Seat Count Selector**: Dropdown or slider for 1-19 seats
- **Plan Information**: Clear explanation of trial features
- **Terms Acceptance**: Checkbox for terms and privacy policy
- **Submit Button**: "Start My Free Trial" with loading state

#### Welcome Experience
- **Confirmation Message**: "Your trial is now active"
- **Onboarding Tour**: Guided tour of key platform features
- **Quick Setup**: Essential account configuration steps
- **Support Access**: Contact information and help resources

### Checkout Flow

#### Plan Selection
- **Plan Comparison**: Side-by-side Solo vs Ensemble comparison
- **Feature Matrix**: Detailed feature comparison table
- **Pricing Calculator**: Interactive seat count and cost calculator
- **Recommendations**: Suggested plan based on seat count
- **Trial Conversion**: Special handling for trial-to-paid conversion

#### Seat Count Selection
- **Interactive Calculator**: Real-time cost calculation
- **Usage Guidance**: Help text for determining seat needs
- **Plan Limits**: Clear indication of Solo (19 max) vs Ensemble (unlimited)
- **Bulk Discounts**: Visual representation of Ensemble volume discounts
- **Validation**: Real-time validation of seat count limits

#### Billing Cycle Selection
- **Monthly vs Annual**: Clear comparison with savings highlighted
- **Cost Breakdown**: Detailed pricing breakdown for selected cycle
- **Savings Calculator**: Annual savings amount prominently displayed
- **Recommendation**: Suggested billing cycle based on plan type
- **Flexibility**: Easy switching between monthly and annual

#### Payment Method Collection
- **Credit Card Form**: Secure, PCI-compliant payment form
- **Payment Options**: Credit card, PayPal, and purchase order options
- **Address Collection**: Billing address for tax and compliance
- **Security Indicators**: SSL and security badges
- **Error Handling**: Clear, actionable error messages

#### Review and Confirmation
- **Order Summary**: Complete breakdown of charges
- **Billing Details**: Payment method and billing cycle
- **Terms Review**: Links to terms and privacy policy
- **Final Confirmation**: "Complete Purchase" button
- **Processing State**: Loading indicator during payment processing

### Trial Management Interface

#### Trial Dashboard
- **Trial Status**: Days remaining and expiration date
- **Usage Statistics**: Seat usage and platform activity
- **Conversion Options**: Prominent upgrade buttons
- **Support Access**: Help and contact information
- **Account Settings**: Basic account configuration

#### Conversion Prompts
- **In-App Notifications**: Non-intrusive conversion reminders
- **Email Campaigns**: Automated conversion email sequences
- **Progress Tracking**: Visual representation of trial usage
- **Feature Highlights**: Showcase of premium features
- **Success Stories**: Customer testimonials and case studies

### Accessibility Requirements

#### Form Accessibility
- **Label Association**: All form fields properly labeled
- **Error Announcements**: Screen reader announcements for validation errors
- **Keyboard Navigation**: Full keyboard accessibility for all form elements
- **Focus Management**: Logical tab order and focus indicators
- **High Contrast**: Sufficient color contrast for all text and elements

#### Payment Security
- **Visual Indicators**: Clear security badges and SSL indicators
- **Trust Signals**: Payment processor logos and security certifications
- **Error Prevention**: Clear validation and confirmation steps
- **Help Text**: Contextual help for complex form fields
- **Progress Indicators**: Clear indication of checkout progress

## Telemetry

### Trial Events

#### Trial Registration
- `trial_started`
  - `trial_id`, `email`, `seat_count`, `referral_source`
- `trial_activated`
  - `trial_id`, `activation_time`, `user_agent`
- `trial_accessed`
  - `trial_id`, `access_type`, `session_duration`

#### Trial Usage
- `trial_feature_used`
  - `trial_id`, `feature_name`, `usage_duration`
- `trial_seat_utilized`
  - `trial_id`, `seats_used`, `utilization_percentage`
- `trial_progress_made`
  - `trial_id`, `progress_metric`, `completion_percentage`

#### Trial Conversion
- `trial_conversion_started`
  - `trial_id`, `target_plan`, `seat_count`
- `trial_converted`
  - `trial_id`, `subscription_id`, `conversion_plan`, `retention_days`
- `trial_expired`
  - `trial_id`, `expiration_reason`, `final_usage_stats`

### Checkout Events

#### Checkout Initiation
- `checkout_started`
  - `session_id`, `plan_type`, `seat_count`, `billing_cycle`
- `checkout_plan_selected`
  - `session_id`, `plan_type`, `seat_count`, `selection_reason`
- `checkout_billing_cycle_selected`
  - `session_id`, `billing_cycle`, `savings_amount`

#### Payment Processing
- `payment_method_added`
  - `session_id`, `payment_type`, `card_type` (if applicable)
- `payment_attempted`
  - `session_id`, `amount`, `payment_method`
- `payment_succeeded`
  - `session_id`, `subscription_id`, `processing_time`
- `payment_failed`
  - `session_id`, `error_type`, `error_message`

#### Checkout Completion
- `checkout_completed`
  - `session_id`, `subscription_id`, `total_amount`, `completion_time`
- `checkout_abandoned`
  - `session_id`, `abandonment_stage`, `time_spent`
- `checkout_retried`
  - `session_id`, `retry_count`, `previous_failure_reason`

### Conversion Analytics
- Trial-to-paid conversion rates by plan type
- Average time to conversion
- Conversion rates by trial usage patterns
- Abandonment rates by checkout stage
- Payment method preferences by customer segment

## Permissions

### Public Access
- **Trial Registration**: Can start free trial with email
- **Pricing Information**: Can view plans and pricing
- **Checkout Process**: Can complete checkout for new subscriptions
- **Support Access**: Can access help and contact information

### Trial Users
- **Platform Access**: Full access to platform features during trial
- **Account Management**: Can modify basic account settings
- **Conversion**: Can convert trial to paid subscription
- **Support**: Can access trial-specific support resources

### Authenticated Users
- **Checkout Access**: Can complete checkout for subscription changes
- **Payment Management**: Can update payment methods
- **Billing History**: Can view past transactions
- **Account Settings**: Can modify subscription settings

### Subscriber Administrators
- **Trial Management**: Can manage trial accounts for their organization
- **Bulk Operations**: Can perform bulk trial conversions
- **Payment Override**: Can override payment requirements for special cases
- **Subscription Management**: Full control over subscription settings

### System Administrators
- **Trial Override**: Can extend or modify trial periods
- **Payment Override**: Can override payment failures
- **Analytics Access**: Full access to trial and conversion analytics
- **Support Operations**: Can assist with trial and checkout issues

## Supporting Documents Referenced

This checkout and trials specification draws from the following source documents:

- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Checkout flow copy and trial messaging
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Registration and subscription flow content
- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Trial and subscription pricing calculations

## Dependencies

### Internal Dependencies
- [plans-and-pricing.md](plans-and-pricing.md) - Pricing structure and plan definitions
- [seat-management.md](seat-management.md) - Seat allocation and management
- [billing-plans-and-seats.md](billing-plans-and-seats.md) - Billing calculations
- [promo-codes.md](promo-codes.md) - Promotional code validation
- [member-management.md](../05-admin-subscriber-experience/member-management.md) - User account creation
- [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) - User role assignments

### External Dependencies
- **Payment Processor**: Stripe, PayPal, or similar for payment processing
- **Email Service**: For trial notifications and conversion campaigns
- **Analytics Platform**: For conversion tracking and user behavior analysis
- **Security Services**: For fraud detection and payment security
- **Compliance Tools**: For PCI compliance and data protection

### Integration Points
- **User Management API**: For account creation and user setup
- **Billing API**: For subscription creation and payment processing
- **Notification Service**: For email campaigns and user communications
- **Analytics Engine**: For conversion tracking and optimization
- **Security Services**: For payment validation and fraud prevention

## Open Questions

1. **Trial Extension Policy**: Should there be a mechanism for extending trials beyond 14 days, and under what circumstances?

2. **Guest Checkout**: Should the system support guest checkout without account creation, and how would this integrate with trial management?

3. **Trial Conversion Incentives**: What promotional offers or incentives should be available for trial-to-paid conversions?

4. **Payment Method Validation**: What level of payment method validation should be performed before allowing trial conversion?

5. **Trial Data Migration**: How should trial data and user progress be handled during conversion to ensure continuity?

6. **Multi-Currency Support**: How should the checkout process handle different currencies for international customers?

7. **Trial Usage Limits**: Should there be usage limits during trials to encourage conversion, and if so, what should they be?

8. **Conversion Attribution**: How should trial conversions be attributed to marketing campaigns and referral sources?

9. **Trial Analytics Depth**: What level of detailed analytics should be provided for trial usage patterns and conversion optimization?

10. **Emergency Trial Access**: What procedures should be in place for providing emergency access during trial system outages?
