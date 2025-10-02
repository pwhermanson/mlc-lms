# Billing Plans and Seats

## Purpose

This document defines the billing plans, seat management, and payment processing for the MLC-LMS platform. It covers subscription tiers, seat operations, payment processing, and administrative controls for managing billing across different customer types.

Define how plans, seats, proration, and school billing artifacts operate so pricing is predictable, invoices are accurate, and admins can manage capacity at scale. This creates one source of truth for engineering, finance ops, and support.

## Scope

Included
- Plan catalog and seat rules for Prelude, Solo, and Ensemble
- Proration for seat adds and removes
- Pause and reactivation behavior
- Invoicing and statements, including PO fields for schools
- Promo and sponsor code interactions at a high level
- Webhook signals and error handling
- Telemetry and permissions

Excluded
- UI layouts for checkout and billing pages
- Provider specific API details beyond required events
- Detailed dunning emails and copy
- Tax calculation rules

## Data model (if applicable)

Entities
- **Plan**
  - `plan_id`, `name` Prelude | Solo | Ensemble
  - `billing_cycle` monthly | annual
  - `seat_pricing` object with tiers per plan
  - `features` flags
  - `status` active | retired
- **Subscription**
  - `subscription_id`, `member_id`, `plan_id`, `billing_cycle`, `status` active | paused | canceled
  - `seats_purchased`, `seats_in_use`, `effective_seat_price`
  - `period_start`, `period_end`, `next_invoice_at`
  - `po_number?`, `notes?`
- **Adjustment**
  - `adjustment_id`, `type` seat_add | seat_remove | plan_change | pause | reactivate
  - `effective_at`, `proration_amount`, `reason`
- **Invoice**
  - `invoice_id`, `subscription_id`, `period_start`, `period_end`, `line_items[]`, `total`, `status` pending | paid | failed
  - `display_url` for admin access
- **Statement**
  - `statement_id`, `member_id`, `range_start`, `range_end`, `pdf_url`, `emailed_to[]`
- **Code**
  - `code_id`, `code_type` promo | sponsor, `discount_type` percent | fixed, `value`, `start_at`, `end_at`, `max_redemptions`, `scope` plan | org
- **Audit**
  - change history for plan, subscription, and code actions

Relationships
- A Member has one active Subscription
- A Subscription accumulates Adjustments
- Invoices reference a Subscription and reflect Adjustments within the billing period
- Statements summarize one or more Invoices for a date range

## Behavior and rules

Plans
- **Prelude** trial with limited seats and time bound access
  - Seats default to 1 to 10 depending on configuration
  - Auto convert to Solo or Ensemble on acceptance or expiration if enabled
- **Solo** for studios
  - Linear per seat price or light tiering
- **Ensemble** for schools
  - Tiered seat pricing by bands, for example 1 to 50, 51 to 200, 201 plus
  - Annual cycle preferred but allow monthly if configured

Seat operations
- Seat add takes effect immediately unless a future date is specified
- Seat remove takes effect at period end by default
  - Optional immediate remove with credit creation if policy allows
- `seats_in_use` cannot exceed `seats_purchased`
- Blocking rules
  - Prevent seat remove if it would strand active students without reassignment

Proration
- For mid cycle seat add
  - Charge proportional amount for the remaining period based on `effective_seat_price`
- For mid cycle plan change up or down
  - Close out current period with credit or charge
  - Start new plan at change time with proportional charge to next renewal
- For monthly to annual switch
  - Credit unused monthly time against the first annual invoice
- Rounding
  - Round to cents using standard half up

Pause and reactivation
- Pause freezes assignments creation and new logins for students, preserves data
- Reactivation date stored on the Subscription
- No new charges while paused. Resume normal billing on reactivation
- Option to prepay remaining cycle on reactivation if required by plan

Invoices and statements
- Invoice generated on period end or immediate when a synchronous seat add occurs depending on provider features
- Schools can store PO number on Subscription and include it on every invoice
- Statements can be created on demand for any date range and delivered via email to specified recipients
- PDF rendering required for audit records

Promo and sponsor codes
- Promo code reduces charges based on configured scope and schedule
- Sponsor code creates subsidy lines that can zero out or reduce charges
- Codes are validated at checkout and on every invoice generation
- Multiple codes can stack only if configured. Default is single active code

Error handling
- If invoice creation fails, retry with backoff and surface an alert to Subscriber Admin
- If payment fails, enter dunning with retry schedule and email notifications
- If provider webhook signature verification fails, discard and log a high severity event

## Subscription Plans

### Prelude (Trial Plan)
- **Duration**: 30-day evaluation period
- **Seats**: Up to 5 seats for testing
- **Features**: Full platform access with limited usage
- **Billing**: No payment required, no automatic conversion
- **Purpose**: Allow potential customers to evaluate the platform

### Solo (Studio Plan)
- **Target**: Individual music studios and private teachers
- **Pricing**: Per-seat monthly or annual billing
- **Seats**: 1-50 seats with per-seat pricing
- **Features**: Full platform access, basic reporting, email support
- **Billing**: Monthly or annual payment options
- **Scaling**: Linear pricing with volume discounts at higher tiers

### Ensemble (School Plan)
- **Target**: Schools, districts, and large music programs
- **Pricing**: Volume-based pricing with institutional discounts
- **Seats**: 25+ seats with volume pricing tiers
- **Features**: Full platform access, advanced reporting, bulk operations, phone support
- **Billing**: Annual contracts with purchase order support
- **Scaling**: Tiered pricing with significant volume discounts

## Seat Management

### Seat Operations
- **Add Seats**: Increase seat count with prorated billing
- **Remove Seats**: Decrease seat count with prorated credits
- **Seat Transfers**: Move seats between different user types
- **Seat Reassignment**: Change which users occupy seats
- **Seat Suspension**: Temporarily disable seats without billing

### Proration Logic
- **Daily Proration**: Calculate charges based on days remaining in billing cycle
- **Credit Application**: Apply credits for unused seat time
- **Refund Processing**: Handle refunds for seat reductions
- **Billing Alignment**: Align seat changes with billing cycles

### Seat Limits and Caps
- **Soft Caps**: Warnings when approaching seat limits
- **Hard Caps**: Prevent exceeding maximum seat count
- **Overage Billing**: Charge for seats beyond plan limits
- **Automatic Scaling**: Suggest plan upgrades when needed

## Billing Operations

### Payment Processing
- **Payment Methods**: Credit cards, ACH, wire transfers
- **Billing Cycles**: Monthly or annual options
- **Payment Retry**: Automatic retry for failed payments
- **Dunning Management**: Handle failed payment scenarios
- **Receipt Generation**: Automatic receipt delivery

### Invoice Management
- **Invoice Generation**: Automatic monthly/annual invoices
- **Custom Invoicing**: Support for custom billing periods
- **Tax Calculation**: Automatic tax calculation based on location
- **Payment Terms**: Net 30, Net 60, or custom terms
- **Late Fees**: Configurable late payment penalties

### Purchase Order Support
- **PO Number Field**: Capture purchase order numbers
- **PO Validation**: Verify PO numbers with school systems
- **Approval Workflows**: Multi-level approval for large purchases
- **PO Reporting**: Track PO usage and compliance
- **Integration**: Connect with school financial systems

## Administrative Controls

### Subscriber-Admin Capabilities
- **Plan Management**: Upgrade, downgrade, or change billing cycles
- **Seat Management**: Add, remove, or reassign seats
- **Billing History**: View all invoices and payments
- **Payment Methods**: Update credit cards and billing information
- **Usage Monitoring**: Track seat utilization and costs

### Teacher-Admin Capabilities
- **Seat Assignment**: Assign seats to teachers and students
- **Usage Reports**: View seat utilization within their scope
- **Billing Inquiries**: Access to billing information for their area
- **Seat Requests**: Request additional seats from Subscriber-Admin

### System Admin Capabilities
- **Billing Overrides**: Override pricing and billing rules
- **Comp Codes**: Apply complimentary or discounted pricing
- **Refund Processing**: Process refunds and credits
- **Billing Analytics**: System-wide billing and usage analytics
- **Support Tools**: Assist customers with billing issues

## Payment Provider Integration

### Stripe Integration
- **Payment Processing**: Secure payment handling
- **Webhook Support**: Real-time payment status updates
- **Subscription Management**: Automated recurring billing
- **Customer Portal**: Self-service billing management
- **Fraud Protection**: Built-in fraud detection and prevention

### Braintree Integration
- **Payment Processing**: Alternative payment processing
- **Multi-currency**: Support for international customers
- **Advanced Fraud**: Enhanced fraud protection
- **Recurring Billing**: Automated subscription management
- **Reporting**: Detailed payment and transaction reporting

### Webhook Handling
- **Payment Success**: Update subscription status
- **Payment Failure**: Handle failed payment scenarios
- **Subscription Changes**: Process plan and seat changes
- **Cancellation**: Handle subscription cancellations
- **Refund Processing**: Process refunds and credits

## Billing Statements and Reporting

### On-Demand Statements
- **Date Range Selection**: Custom date ranges for statements
- **Statement Generation**: PDF and CSV format options
- **Email Delivery**: Automatic statement delivery
- **Statement History**: Archive of all generated statements
- **Custom Branding**: School-specific statement formatting

### Usage Reporting
- **Seat Utilization**: Track how many seats are actively used
- **Feature Usage**: Monitor which features are being used
- **Cost Analysis**: Break down costs by department or class
- **Trend Analysis**: Track usage and cost trends over time
- **ROI Reporting**: Calculate return on investment metrics

### Financial Reporting
- **Revenue Tracking**: Monitor subscription revenue
- **Churn Analysis**: Track customer retention and churn
- **Growth Metrics**: Monitor customer and revenue growth
- **Forecasting**: Predict future revenue and costs
- **Compliance**: Ensure billing compliance and accuracy

## Pause and Reactivation

### Pause Functionality
- **Pause Reasons**: Track why subscriptions are paused
- **Pause Duration**: Set maximum pause periods
- **Seat Preservation**: Maintain seat assignments during pause
- **Data Retention**: Preserve user data during pause
- **Reactivation Process**: Clear process for reactivating subscriptions

### Reactivation Logic
- **Automatic Reactivation**: Scheduled reactivation dates
- **Manual Reactivation**: Admin-initiated reactivation
- **Billing Resumption**: Resume billing on reactivation
- **Data Restoration**: Restore full access to paused accounts
- **Notification System**: Notify users of reactivation

## Compliance and Security

### Data Protection
- **PCI Compliance**: Secure handling of payment data
- **Data Encryption**: Encrypt sensitive billing information
- **Access Controls**: Restrict access to billing data
- **Audit Logging**: Log all billing-related actions
- **Data Retention**: Comply with data retention requirements

### Financial Compliance
- **Tax Compliance**: Handle tax calculation and reporting
- **Accounting Integration**: Connect with accounting systems
- **Audit Support**: Provide data for financial audits
- **Regulatory Compliance**: Meet financial regulations
- **Reporting Requirements**: Generate required financial reports

## UX requirements (if applicable)

- Admin billing page shows plan, seats purchased, seats in use, next invoice date, PO number, and statement downloads
- Seat change flow explains effective date, proration, and resulting charges before confirmation
- Pause workflow requires a reactivation date. Show what students will experience
- Code entry fields validate format and show the effective discount before saving
- All actions produce clear success or error banners and are reversible when possible

## Telemetry

Events
- `plan_selected`
  - `member_id`, `plan_id`, `billing_cycle`
- `seats_changed`
  - `member_id`, `old_seats`, `new_seats`, `effective_at`, `proration_amount`
- `subscription_paused`
  - `member_id`, `reactivate_at`
- `subscription_reactivated`
  - `member_id`
- `invoice_generated`
  - `invoice_id`, `member_id`, `period_start`, `period_end`, `total`
- `invoice_payment_result`
  - `invoice_id`, `status` paid | failed, `attempt_number`
- `statement_generated`
  - `statement_id`, `member_id`, `range_start`, `range_end`
- `code_applied`
  - `member_id`, `code_id`, `code_type`, `value`, `scope`
- `billing_error`
  - `member_id`, `code`, `message`, `provider_event_id?`

Sampling
- Invoice, payment, and code events are never sampled

## Permissions

- Subscriber Admin
  - View invoices and statements, change seats, apply codes, set PO number, pause or reactivate
- Teacher Admin and Teacher
  - Read only view of plan and seats if enabled by org policy, cannot change billing
- SysAdmin
  - Full access with audit logs and ability to issue refunds or credits
- Content Editors
  - No access to billing

## Dependencies

- Plans and pricing reference data
- Provider integration spec for Stripe or Braintree
- Promo and sponsor code specs
- Statements generation service
- Roles and permissions for access control
- Event model for canonical telemetry properties

## Open questions

- Final price table for Solo and Ensemble seat bands by monthly and annual cycles
- Exact proration formula confirmation and rounding rules if they differ from the above
- Dunning schedule and number of retries before suspension
- Whether immediate seat removal with credit is allowed for schools
- Statement format requirements from finance, including logo usage and address blocks
- Tax handling requirements and jurisdictions we must support

## Future Enhancements

### Advanced Billing Features
- **Usage-Based Billing**: Charge based on actual usage
- **Tiered Pricing**: More granular pricing tiers
- **Promotional Pricing**: Special offers and discounts
- **Loyalty Programs**: Rewards for long-term customers
- **Referral Programs**: Incentives for customer referrals

### Integration Improvements
- **ERP Integration**: Connect with enterprise resource planning systems
- **SIS Integration**: Integrate with student information systems
- **Accounting Software**: Connect with QuickBooks, Xero, etc.
- **CRM Integration**: Connect with customer relationship management systems
- **Analytics Platforms**: Integrate with business intelligence tools
