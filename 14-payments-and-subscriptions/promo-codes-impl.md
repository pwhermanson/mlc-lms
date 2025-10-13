# Promo Codes Implementation

## Purpose

This document defines the technical implementation of the promotional codes system for the MLC-LMS platform. It covers the data models, API endpoints, business logic, validation rules, and integration points required to support promotional code creation, management, redemption, and tracking.

The implementation supports both percentage and fixed-amount discounts, usage limits, expiration dates, and campaign tracking to enable effective marketing campaigns and special offers.

## Scope

### Included
- Data model and database schema for promo codes
- API endpoints for code management and redemption
- Business logic for discount calculation and validation
- Integration with billing and checkout systems
- Usage tracking and analytics
- Error handling and edge cases
- Security and fraud prevention measures

### Excluded
- UI components for code management (see admin interfaces)
- Marketing campaign creation workflows
- Detailed reporting dashboards (see analytics specs)
- Email templates for code delivery

## Data Model

### Core Entities

#### PromoCode
```sql
CREATE TABLE promo_codes (
    code_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(50) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    discount_type ENUM('percent', 'fixed') NOT NULL,
    discount_value DECIMAL(10,2) NOT NULL,
    max_discount_amount DECIMAL(10,2), -- For percentage discounts
    min_order_amount DECIMAL(10,2) DEFAULT 0,
    max_redemptions INTEGER,
    current_redemptions INTEGER DEFAULT 0,
    start_date TIMESTAMP NOT NULL,
    end_date TIMESTAMP,
    is_active BOOLEAN DEFAULT true,
    scope ENUM('global', 'plan_specific', 'org_specific') DEFAULT 'global',
    applicable_plans JSON, -- Array of plan IDs for plan_specific scope
    applicable_orgs JSON, -- Array of org IDs for org_specific scope
    created_by UUID NOT NULL REFERENCES members(member_id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### PromoCodeRedemption
```sql
CREATE TABLE promo_code_redemptions (
    redemption_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code_id UUID NOT NULL REFERENCES promo_codes(code_id),
    subscription_id UUID NOT NULL REFERENCES subscriptions(subscription_id),
    member_id UUID NOT NULL REFERENCES members(member_id),
    discount_amount DECIMAL(10,2) NOT NULL,
    original_amount DECIMAL(10,2) NOT NULL,
    final_amount DECIMAL(10,2) NOT NULL,
    redeemed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    invoice_id UUID REFERENCES invoices(invoice_id),
    metadata JSON -- Additional redemption context
);
```

#### PromoCodeUsage
```sql
CREATE TABLE promo_code_usage (
    usage_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code_id UUID NOT NULL REFERENCES promo_codes(code_id),
    member_id UUID NOT NULL REFERENCES members(member_id),
    action ENUM('viewed', 'applied', 'removed', 'expired') NOT NULL,
    session_id UUID,
    ip_address INET,
    user_agent TEXT,
    occurred_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    metadata JSON
);
```

## API Endpoints

### Code Management (Admin)

#### Create Promo Code
```http
POST /api/v1/admin/promo-codes
Content-Type: application/json

{
  "code": "WELCOME2024",
  "name": "Welcome Discount 2024",
  "description": "20% off first year for new customers",
  "discount_type": "percent",
  "discount_value": 20.00,
  "max_discount_amount": 500.00,
  "min_order_amount": 100.00,
  "max_redemptions": 1000,
  "start_date": "2024-01-01T00:00:00Z",
  "end_date": "2024-12-31T23:59:59Z",
  "scope": "global",
  "applicable_plans": ["solo", "ensemble"]
}
```

#### List Promo Codes
```http
GET /api/v1/admin/promo-codes?page=1&limit=50&status=active&search=welcome
```

#### Update Promo Code
```http
PUT /api/v1/admin/promo-codes/{code_id}
Content-Type: application/json

{
  "is_active": false,
  "end_date": "2024-06-30T23:59:59Z"
}
```

#### Get Promo Code Analytics
```http
GET /api/v1/admin/promo-codes/{code_id}/analytics?start_date=2024-01-01&end_date=2024-12-31
```

### Code Redemption (Public)

#### Validate Promo Code
```http
POST /api/v1/promo-codes/validate
Content-Type: application/json

{
  "code": "WELCOME2024",
  "plan_id": "solo",
  "seat_count": 10,
  "billing_cycle": "annual",
  "member_id": "uuid-here"
}
```

Response:
```json
{
  "valid": true,
  "discount_amount": 95.40,
  "original_amount": 477.00,
  "final_amount": 381.60,
  "discount_percentage": 20.0,
  "expires_at": "2024-12-31T23:59:59Z"
}
```

#### Apply Promo Code to Checkout
```http
POST /api/v1/checkout/apply-promo-code
Content-Type: application/json

{
  "session_id": "checkout-session-uuid",
  "code": "WELCOME2024"
}
```

#### Remove Promo Code from Checkout
```http
DELETE /api/v1/checkout/promo-code/{session_id}
```

## Business Logic

### Code Validation Rules

1. **Code Format Validation**
   - 3-50 characters, alphanumeric and hyphens only
   - Case insensitive matching
   - No consecutive special characters

2. **Availability Validation**
   - Code must be active (`is_active = true`)
   - Current time must be within `start_date` and `end_date` range
   - Redemption count must be less than `max_redemptions` (if set)

3. **Scope Validation**
   - `global`: Available to all customers
   - `plan_specific`: Must match one of the `applicable_plans`
   - `org_specific`: Must match one of the `applicable_orgs`

4. **Order Validation**
   - Order amount must meet `min_order_amount` requirement
   - Customer must not have already used this code (unless configured otherwise)

### Discount Calculation

```python
def calculate_discount(promo_code, order_amount, seat_count, billing_cycle):
    """
    Calculate discount amount based on promo code configuration
    """
    if promo_code.discount_type == 'percent':
        discount_amount = order_amount * (promo_code.discount_value / 100)
        
        # Apply maximum discount limit if set
        if promo_code.max_discount_amount:
            discount_amount = min(discount_amount, promo_code.max_discount_amount)
    else:  # fixed amount
        discount_amount = promo_code.discount_value
    
    # Ensure discount doesn't exceed order amount
    discount_amount = min(discount_amount, order_amount)
    
    return round(discount_amount, 2)
```

### Redemption Process

1. **Pre-redemption Checks**
   - Validate code format and availability
   - Check scope restrictions
   - Verify minimum order amount
   - Confirm user eligibility

2. **Calculate Discount**
   - Apply percentage or fixed discount
   - Enforce maximum discount limits
   - Round to nearest cent

3. **Record Redemption**
   - Create redemption record
   - Update usage counters
   - Log analytics event
   - Apply to checkout session

4. **Post-redemption Actions**
   - Update subscription pricing
   - Generate updated invoice
   - Send confirmation email
   - Track conversion metrics

## Integration Points

### Checkout System Integration

The promo code system integrates with the checkout flow at these points:

1. **Code Entry**: User enters code during checkout
2. **Validation**: Real-time validation with pricing preview
3. **Application**: Code applied to session with updated totals
4. **Removal**: User can remove code and revert pricing
5. **Finalization**: Code locked to subscription during payment

### Billing System Integration

```python
class PromoCodeService:
    def apply_to_subscription(self, subscription_id, code_id):
        """Apply promo code discount to subscription"""
        subscription = self.get_subscription(subscription_id)
        promo_code = self.get_promo_code(code_id)
        
        # Calculate discount
        discount_amount = self.calculate_discount(
            promo_code, 
            subscription.total_amount,
            subscription.seat_count,
            subscription.billing_cycle
        )
        
        # Create adjustment
        adjustment = Adjustment(
            subscription_id=subscription_id,
            type='promo_code',
            amount=-discount_amount,
            description=f"Promo code: {promo_code.code}",
            promo_code_id=code_id
        )
        
        # Update subscription
        subscription.adjustments.append(adjustment)
        subscription.recalculate_totals()
        
        return adjustment
```

### Analytics Integration

Track key metrics for marketing effectiveness:

- **Code Performance**: Redemption rates, conversion rates, revenue impact
- **User Behavior**: Code discovery, application attempts, abandonment
- **Campaign Analysis**: A/B testing, seasonal trends, customer segments

## Error Handling

### Validation Errors

```json
{
  "error": "INVALID_PROMO_CODE",
  "message": "The promo code 'INVALID123' is not valid or has expired",
  "code": "PROMO_CODE_NOT_FOUND"
}
```

### Business Logic Errors

```json
{
  "error": "PROMO_CODE_NOT_APPLICABLE",
  "message": "This promo code is not valid for the Solo plan",
  "code": "SCOPE_VIOLATION",
  "details": {
    "code": "SCHOOL2024",
    "applicable_plans": ["ensemble"]
  }
}
```

### System Errors

```json
{
  "error": "PROMO_CODE_SYSTEM_ERROR",
  "message": "Unable to process promo code at this time",
  "code": "SYSTEM_UNAVAILABLE",
  "retry_after": 30
}
```

## Security Considerations

### Fraud Prevention

1. **Rate Limiting**: Limit code validation attempts per IP/user
2. **Usage Tracking**: Monitor for suspicious redemption patterns
3. **Code Obfuscation**: Avoid predictable code patterns
4. **Audit Logging**: Track all code-related actions

### Data Protection

1. **PII Handling**: Minimize personal data in promo code records
2. **Access Controls**: Restrict code management to authorized admins
3. **Audit Trail**: Log all code modifications and redemptions
4. **Data Retention**: Define retention policies for usage data

## Performance Considerations

### Caching Strategy

- **Code Lookup**: Cache active codes in Redis for fast validation
- **Usage Counts**: Cache redemption counts with TTL
- **Validation Results**: Cache validation results for short periods

### Database Optimization

- **Indexes**: Optimize queries on code, status, and date ranges
- **Partitioning**: Consider partitioning usage tables by date
- **Archiving**: Archive old redemption data to reduce table size

## Monitoring and Alerting

### Key Metrics

- **Redemption Rate**: Percentage of codes that get redeemed
- **Conversion Impact**: Revenue impact of promo codes
- **System Performance**: API response times and error rates
- **Fraud Detection**: Unusual redemption patterns

### Alerts

- **High Error Rate**: API errors exceeding threshold
- **Suspicious Activity**: Unusual redemption patterns
- **Code Expiration**: Codes expiring within 24 hours
- **Usage Limits**: Codes approaching redemption limits

## Testing Strategy

### Unit Tests

- Discount calculation logic
- Validation rule enforcement
- Error handling scenarios
- Edge cases and boundary conditions

### Integration Tests

- API endpoint functionality
- Database operations
- Checkout flow integration
- Billing system integration

### Load Tests

- High-volume code validation
- Concurrent redemption handling
- Database performance under load
- Cache effectiveness

## Supporting Documents Referenced

This promo codes implementation specification draws from the following source documents:

- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Pricing calculations with promotional discounts
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Promotional messaging and checkout flow copy
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Marketing and promotional content

## Deployment Considerations

### Feature Flags

- Enable/disable promo code functionality
- A/B testing for different discount strategies
- Gradual rollout of new features

### Database Migrations

- Schema changes for new fields
- Data migration for existing codes
- Index creation for performance

### Rollback Plan

- Disable promo code functionality
- Revert to previous pricing
- Restore from backup if needed
