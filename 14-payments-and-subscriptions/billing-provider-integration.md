# Billing Provider Integration

## Purpose

This specification defines the integration between the MLC-LMS platform and external billing providers (Stripe and Braintree) for payment processing, subscription management, and financial operations. It enables secure, reliable, and scalable billing operations that support the platform's three-tier pricing model (Prelude, Solo, Ensemble) and handles the complex requirements of educational institutions.

The integration provides a unified interface for payment processing, subscription lifecycle management, webhook handling, and financial reporting while maintaining PCI compliance and supporting both individual music teachers and large educational institutions.

## Scope

### Included
- Payment processor integration (Stripe and Braintree)
- Subscription lifecycle management (create, update, cancel, pause, reactivate)
- Payment method management (credit cards, PayPal, ACH, wire transfers)
- Webhook handling for real-time payment status updates
- Invoice generation and payment processing
- Refund and credit processing
- Dunning management for failed payments
- Purchase order support for educational institutions
- Multi-currency support for international customers
- Fraud detection and prevention
- PCI compliance and security measures
- Financial reporting and reconciliation

### Excluded
- Detailed billing calculations and proration logic (see [billing-plans-and-seats.md](billing-plans-and-seats.md))
- Pricing structure and plan definitions (see [plans-and-pricing.md](plans-and-pricing.md))
- Seat management operations (see [seat-management.md](seat-management.md))
- Checkout and trial flows (see [checkout-and-trials.md](checkout-and-trials.md))
- Promotional code processing (see [promo-codes.md](promo-codes.md))
- User interface design and layouts
- Tax calculation and compliance (handled by providers)

## Data Model

### Core Entities

#### Payment Provider
- `provider_id` (UUID, Primary Key)
- `provider_name` (enum: stripe, braintree)
- `provider_type` (enum: primary, secondary, test)
- `api_credentials` (encrypted JSON)
- `webhook_secret` (encrypted string)
- `is_active` (boolean)
- `supported_features` (JSON array)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### Payment Method
- `payment_method_id` (UUID, Primary Key)
- `customer_id` (UUID, Foreign Key)
- `provider_payment_method_id` (string, provider-specific)
- `provider_name` (enum: stripe, braintree)
- `payment_type` (enum: credit_card, paypal, ach, wire_transfer)
- `card_type` (enum: visa, mastercard, amex, discover, nullable)
- `last_four_digits` (string, nullable)
- `expiration_month` (integer, nullable)
- `expiration_year` (integer, nullable)
- `billing_address` (JSON)
- `is_default` (boolean)
- `is_active` (boolean)
- `created_at` (timestamp)
- `updated_at` (timestamp)

#### Payment Transaction
- `transaction_id` (UUID, Primary Key)
- `subscription_id` (UUID, Foreign Key)
- `payment_method_id` (UUID, Foreign Key)
- `provider_transaction_id` (string)
- `provider_name` (enum: stripe, braintree)
- `transaction_type` (enum: payment, refund, credit, chargeback)
- `amount` (decimal)
- `currency` (string, ISO 4217)
- `status` (enum: pending, succeeded, failed, canceled, disputed)
- `failure_reason` (string, nullable)
- `failure_code` (string, nullable)
- `processed_at` (timestamp, nullable)
- `created_at` (timestamp)

#### Webhook Event
- `webhook_id` (UUID, Primary Key)
- `provider_name` (enum: stripe, braintree)
- `event_id` (string, provider-specific)
- `event_type` (string)
- `event_data` (JSON)
- `processed` (boolean)
- `processed_at` (timestamp, nullable)
- `retry_count` (integer)
- `error_message` (string, nullable)
- `created_at` (timestamp)

#### Provider Configuration
- `config_id` (UUID, Primary Key)
- `provider_name` (enum: stripe, braintree)
- `environment` (enum: sandbox, production)
- `api_key` (encrypted string)
- `secret_key` (encrypted string)
- `webhook_endpoint` (string)
- `webhook_secret` (encrypted string)
- `supported_currencies` (JSON array)
- `supported_payment_types` (JSON array)
- `fraud_protection_enabled` (boolean)
- `pci_compliance_level` (string)
- `created_at` (timestamp)
- `updated_at` (timestamp)

### Relationships
- One customer can have multiple payment methods
- One payment method can be used for multiple transactions
- One subscription can have multiple payment transactions
- One webhook event belongs to one provider
- One provider can have multiple configurations (sandbox/production)

## Behavior and Rules

### Payment Provider Selection

#### Primary Provider (Braintree)
- **Default Provider**: Braintree serves as the primary payment processor
- **Historical Requirement**: Based on existing MLC platform integration
- **Supported Cards**: Discover, American Express, MasterCard, Visa
- **PayPal Integration**: Native PayPal payment support
- **Educational Focus**: Optimized for educational institution billing
- **Purchase Order Support**: Enhanced PO processing capabilities

#### Secondary Provider (Stripe)
- **Backup Provider**: Stripe serves as secondary/backup processor
- **Modern Features**: Advanced fraud protection and analytics
- **International Support**: Better multi-currency and international support
- **Developer Experience**: Superior API and webhook handling
- **Scalability**: Better handling of high-volume transactions

#### Provider Selection Logic
- **New Subscriptions**: Use primary provider (Braintree) by default
- **Failed Payments**: Retry with secondary provider (Stripe) if configured
- **International Customers**: Use Stripe for non-US customers
- **High-Value Transactions**: Use primary provider for transactions >$10,000
- **Provider Outage**: Automatic failover to secondary provider

### Payment Processing Rules

#### Credit Card Processing
- **Supported Cards**: Visa, MasterCard, American Express, Discover
- **Address Verification**: Required for all credit card transactions
- **CVV Verification**: Required for all credit card transactions
- **3D Secure**: Enabled for international transactions
- **Fraud Detection**: Real-time fraud scoring and blocking
- **PCI Compliance**: All card data handled via tokenization

#### PayPal Processing
- **PayPal Express**: One-click PayPal payments
- **PayPal Pro**: Credit card processing through PayPal
- **PayPal Subscriptions**: Recurring billing through PayPal
- **International Support**: Multi-currency PayPal support
- **Account Verification**: PayPal account verification required

#### ACH and Wire Transfer Processing
- **ACH Processing**: Bank account verification required
- **Wire Transfers**: Manual processing for large amounts
- **Educational Institutions**: Preferred for annual billing
- **Verification Process**: Bank account ownership verification
- **Processing Time**: 3-5 business days for ACH, 1-2 days for wire

### Subscription Lifecycle Management

#### Subscription Creation
- **Payment Method Required**: Valid payment method must be on file
- **Immediate Billing**: First payment processed immediately
- **Trial Conversion**: Seamless conversion from trial to paid
- **Plan Validation**: Ensure plan and seat count are valid
- **Tax Calculation**: Automatic tax calculation based on location

#### Subscription Updates
- **Plan Changes**: Upgrade/downgrade with proration
- **Seat Changes**: Add/remove seats with proration
- **Billing Cycle Changes**: Monthly to annual conversion
- **Payment Method Updates**: Change payment methods
- **Pause/Resume**: Suspend and reactivate subscriptions

#### Subscription Cancellation
- **Immediate Cancellation**: Stop billing immediately
- **End of Period**: Cancel at end of current billing period
- **Refund Processing**: Automatic refund calculation
- **Data Retention**: Preserve user data per retention policy
- **Reactivation**: Allow reactivation within grace period

### Webhook Handling

#### Stripe Webhooks
- **Event Types**: `customer.created`, `customer.updated`, `customer.deleted`
- **Payment Events**: `payment_intent.succeeded`, `payment_intent.payment_failed`
- **Subscription Events**: `customer.subscription.created`, `customer.subscription.updated`, `customer.subscription.deleted`
- **Invoice Events**: `invoice.created`, `invoice.payment_succeeded`, `invoice.payment_failed`
- **Dispute Events**: `charge.dispute.created`, `charge.dispute.updated`

#### Braintree Webhooks
- **Event Types**: `customer_created`, `customer_updated`, `customer_deleted`
- **Payment Events**: `transaction_settled`, `transaction_settlement_declined`
- **Subscription Events**: `subscription_charged_successfully`, `subscription_charged_unsuccessfully`
- **Dispute Events**: `dispute_opened`, `dispute_lost`, `dispute_won`

#### Webhook Processing Rules
- **Idempotency**: Prevent duplicate processing of webhook events
- **Retry Logic**: Exponential backoff for failed webhook processing
- **Error Handling**: Log and alert on webhook processing failures
- **Signature Verification**: Verify webhook signatures for security
- **Event Ordering**: Process events in chronological order

### Error Handling and Recovery

#### Payment Failures
- **Retry Logic**: Automatic retry with exponential backoff
- **Dunning Management**: Escalating email notifications
- **Grace Period**: 30-day grace period before suspension
- **Manual Intervention**: Admin tools for payment recovery
- **Provider Fallback**: Switch to secondary provider on repeated failures

#### System Failures
- **Provider Outage**: Automatic failover to secondary provider
- **API Rate Limits**: Implement rate limiting and queuing
- **Data Consistency**: Ensure data consistency across providers
- **Audit Logging**: Comprehensive logging of all operations
- **Monitoring**: Real-time monitoring and alerting

### Security and Compliance

#### PCI Compliance
- **Data Handling**: No storage of sensitive card data
- **Tokenization**: Use provider tokenization for card data
- **Encryption**: Encrypt all sensitive data at rest and in transit
- **Access Controls**: Role-based access to payment data
- **Audit Trails**: Complete audit trail of all payment operations

#### Fraud Prevention
- **Real-time Scoring**: Provider fraud detection services
- **Velocity Checks**: Monitor transaction frequency and amounts
- **Geolocation**: Flag transactions from unusual locations
- **Device Fingerprinting**: Track and analyze device characteristics
- **Machine Learning**: Use ML models for fraud detection

## UX Requirements

### Payment Method Management
- **Add Payment Method**: Secure form for adding credit cards
- **Update Payment Method**: Easy updating of existing methods
- **Default Payment Method**: Set and change default payment method
- **Payment History**: View all payment transactions
- **Receipt Access**: Download receipts and invoices

### Subscription Management
- **Plan Overview**: Clear display of current plan and pricing
- **Billing History**: Complete history of all charges and payments
- **Upgrade/Downgrade**: Easy plan changes with cost preview
- **Seat Management**: Add/remove seats with cost calculation
- **Pause/Resume**: Suspend and reactivate subscriptions

### Error Handling
- **Clear Error Messages**: User-friendly error messages
- **Retry Options**: Easy retry for failed payments
- **Support Access**: Direct access to billing support
- **Status Indicators**: Clear indication of payment status
- **Progress Tracking**: Show progress for long-running operations

### Accessibility
- **Screen Reader Support**: Full accessibility for payment forms
- **Keyboard Navigation**: Complete keyboard accessibility
- **High Contrast**: Sufficient color contrast for all elements
- **Error Announcements**: Screen reader announcements for errors
- **Form Validation**: Clear validation messages and indicators

## Telemetry

### Payment Events
- `payment_initiated`
  - `transaction_id`, `customer_id`, `amount`, `currency`, `payment_type`
- `payment_succeeded`
  - `transaction_id`, `processing_time`, `provider_name`, `provider_transaction_id`
- `payment_failed`
  - `transaction_id`, `failure_reason`, `failure_code`, `retry_count`
- `payment_refunded`
  - `transaction_id`, `refund_amount`, `refund_reason`, `refund_method`

### Subscription Events
- `subscription_created`
  - `subscription_id`, `customer_id`, `plan_id`, `billing_cycle`, `amount`
- `subscription_updated`
  - `subscription_id`, `change_type`, `old_value`, `new_value`, `effective_date`
- `subscription_canceled`
  - `subscription_id`, `cancelation_reason`, `cancelation_date`, `refund_amount`
- `subscription_paused`
  - `subscription_id`, `pause_reason`, `reactivation_date`

### Webhook Events
- `webhook_received`
  - `webhook_id`, `provider_name`, `event_type`, `event_id`
- `webhook_processed`
  - `webhook_id`, `processing_time`, `success`, `error_message`
- `webhook_failed`
  - `webhook_id`, `retry_count`, `error_message`, `next_retry_at`

### Provider Events
- `provider_switched`
  - `customer_id`, `from_provider`, `to_provider`, `switch_reason`
- `provider_outage`
  - `provider_name`, `outage_start`, `outage_end`, `affected_customers`
- `fraud_detected`
  - `transaction_id`, `fraud_score`, `fraud_reasons`, `action_taken`

### Financial Events
- `invoice_generated`
  - `invoice_id`, `subscription_id`, `amount`, `due_date`, `status`
- `payment_collected`
  - `invoice_id`, `amount_collected`, `collection_date`, `payment_method`
- `refund_processed`
  - `refund_id`, `original_transaction_id`, `refund_amount`, `refund_reason`

## Permissions

### Public Access
- **Payment Forms**: Access to secure payment forms
- **Payment Status**: View payment status and history
- **Receipt Access**: Download receipts and invoices

### Authenticated Users
- **Payment Methods**: Manage their own payment methods
- **Billing History**: View their own billing history
- **Subscription Changes**: Modify their own subscription
- **Receipt Downloads**: Download their own receipts

### Subscriber Administrators
- **Organization Billing**: Manage billing for their organization
- **Payment Methods**: Manage organization payment methods
- **Billing History**: View organization billing history
- **Subscription Management**: Full subscription management
- **Purchase Orders**: Manage PO numbers and billing

### System Administrators
- **All Billing Data**: Access to all billing and payment data
- **Provider Management**: Configure payment providers
- **Refund Processing**: Process refunds and credits
- **Fraud Management**: Manage fraud detection and prevention
- **Audit Access**: Access to all audit logs and reports

### Finance Team
- **Financial Reports**: Access to financial reporting
- **Reconciliation**: Access to reconciliation tools
- **Tax Reporting**: Access to tax-related data
- **Compliance**: Access to compliance reports

## Dependencies

### Internal Dependencies
- [billing-plans-and-seats.md](billing-plans-and-seats.md) - Billing calculations and proration
- [plans-and-pricing.md](plans-and-pricing.md) - Pricing structure and plan definitions
- [seat-management.md](seat-management.md) - Seat allocation and management
- [checkout-and-trials.md](checkout-and-trials.md) - Checkout and trial flows
- [promo-codes.md](promo-codes.md) - Promotional code processing
- [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) - User permissions
- [event-model.md](../15-analytics-and-reporting/event-model.md) - Event tracking

### External Dependencies
- **Stripe API**: Payment processing and subscription management
- **Braintree API**: Payment processing and subscription management
- **PayPal API**: PayPal payment processing
- **Banking APIs**: ACH and wire transfer processing
- **Tax Services**: Tax calculation and compliance
- **Fraud Detection**: Third-party fraud detection services

### Infrastructure Dependencies
- **Database**: PostgreSQL for transaction storage
- **Message Queue**: Redis for webhook processing
- **Monitoring**: Application performance monitoring
- **Logging**: Centralized logging system
- **Security**: Encryption and key management
- **Backup**: Data backup and recovery systems

## Open Questions

1. **Provider Migration Strategy**: What is the timeline and approach for migrating from Braintree to Stripe as the primary provider?

2. **International Expansion**: How will the system handle international customers with different currencies and payment methods?

3. **Fraud Detection Thresholds**: What are the specific fraud detection rules and thresholds for different customer segments?

4. **Webhook Reliability**: What is the acceptable webhook failure rate and how should the system handle webhook outages?

5. **PCI Compliance Scope**: What specific PCI compliance requirements must be met for the MLC-LMS platform?

6. **Payment Method Limits**: Are there limits on the number of payment methods a customer can have on file?

7. **Refund Policy**: What is the refund policy for different subscription types and customer segments?

8. **Provider Failover**: How quickly should the system failover to the secondary provider during outages?

9. **Audit Requirements**: What specific audit requirements must be met for financial compliance?

10. **Performance Requirements**: What are the performance requirements for payment processing and webhook handling?
