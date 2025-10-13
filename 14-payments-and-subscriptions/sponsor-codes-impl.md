# Sponsor Codes Implementation

## Purpose

This document defines the technical implementation of the sponsor codes system for the MLC-LMS platform. The sponsor codes system enables Ensemble Members to support non-profit educational organizations through the Christine Hermanson Grant Program by providing discounted or free access to the platform.

Unlike promotional codes which are marketing tools, sponsor codes are relationship-based codes that create connections between Ensemble Members and educational institutions, facilitating philanthropic support for music education.

## Scope

### Included
- Data model and database schema for sponsor codes
- API endpoints for sponsor code management and assignment
- Business logic for sponsor code generation and validation
- Integration with subscription and billing systems
- Sponsor-sponsee relationship tracking
- Usage analytics and reporting
- Error handling and edge cases
- Security and access controls

### Excluded
- UI components for sponsor code management (see admin interfaces)
- Christine Hermanson Grant Program policy details (see program documentation)
- Detailed reporting dashboards (see analytics specs)
- Email templates for sponsor notifications

## Data Model

### Core Entities

#### SponsorCode
```sql
CREATE TABLE sponsor_codes (
    sponsor_code_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    code VARCHAR(50) UNIQUE NOT NULL,
    sponsor_member_id UUID NOT NULL REFERENCES members(member_id),
    sponsor_name VARCHAR(255) NOT NULL,
    status ENUM('pending', 'active', 'used', 'expired', 'revoked') DEFAULT 'pending',
    max_uses INTEGER DEFAULT 1,
    current_uses INTEGER DEFAULT 0,
    discount_type ENUM('percent', 'fixed', 'free') NOT NULL,
    discount_value DECIMAL(10,2),
    applicable_plans JSON, -- Array of plan IDs that can use this code
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    activated_at TIMESTAMP,
    expires_at TIMESTAMP,
    last_used_at TIMESTAMP,
    metadata JSON -- Additional sponsor context
);
```

#### SponsorCodeAssignment
```sql
CREATE TABLE sponsor_code_assignments (
    assignment_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sponsor_code_id UUID NOT NULL REFERENCES sponsor_codes(sponsor_code_id),
    sponsee_org_id UUID NOT NULL REFERENCES organizations(org_id),
    sponsee_contact_name VARCHAR(255),
    sponsee_contact_email VARCHAR(255),
    assigned_by UUID NOT NULL REFERENCES members(member_id),
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    status ENUM('assigned', 'used', 'expired', 'revoked') DEFAULT 'assigned',
    notes TEXT,
    metadata JSON
);
```

#### SponsorCodeUsage
```sql
CREATE TABLE sponsor_code_usage (
    usage_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    sponsor_code_id UUID NOT NULL REFERENCES sponsor_codes(sponsor_code_id),
    subscription_id UUID NOT NULL REFERENCES subscriptions(subscription_id),
    sponsee_org_id UUID NOT NULL REFERENCES organizations(org_id),
    discount_amount DECIMAL(10,2) NOT NULL,
    original_amount DECIMAL(10,2) NOT NULL,
    final_amount DECIMAL(10,2) NOT NULL,
    used_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    used_by UUID NOT NULL REFERENCES members(member_id),
    invoice_id UUID REFERENCES invoices(invoice_id),
    metadata JSON
);
```

## API Endpoints

### Sponsor Code Management (Ensemble Members)

#### Generate Sponsor Code
```http
POST /api/v1/members/{member_id}/sponsor-codes
Content-Type: application/json

{
  "discount_type": "free",
  "discount_value": null,
  "max_uses": 1,
  "expires_at": "2025-12-31T23:59:59Z",
  "applicable_plans": ["ensemble", "solo"]
}
```

Response:
```json
{
  "sponsor_code_id": "uuid-here",
  "code": "11238A",
  "status": "pending",
  "sponsor_name": "Joe Smith",
  "expires_at": "2025-12-31T23:59:59Z",
  "dashboard_url": "/members/dashboard/sponsor-codes/11238A"
}
```

#### List My Sponsor Codes
```http
GET /api/v1/members/{member_id}/sponsor-codes?status=active&page=1&limit=20
```

#### Get Sponsor Code Details
```http
GET /api/v1/members/{member_id}/sponsor-codes/{sponsor_code_id}
```

#### Revoke Sponsor Code
```http
DELETE /api/v1/members/{member_id}/sponsor-codes/{sponsor_code_id}
```

### Sponsor Code Assignment (Admin)

#### Assign Sponsor Code to Organization
```http
POST /api/v1/admin/sponsor-codes/{sponsor_code_id}/assign
Content-Type: application/json

{
  "sponsee_org_id": "org-uuid-here",
  "sponsee_contact_name": "Sally Peterson",
  "sponsee_contact_email": "sally@school.edu",
  "notes": "Assigned for fall semester program"
}
```

#### List Sponsor Code Assignments
```http
GET /api/v1/admin/sponsor-codes/assignments?sponsor_code_id={id}&status=assigned
```

#### Revoke Assignment
```http
DELETE /api/v1/admin/sponsor-codes/assignments/{assignment_id}
```

### Sponsor Code Redemption (Public)

#### Validate Sponsor Code
```http
POST /api/v1/sponsor-codes/validate
Content-Type: application/json

{
  "code": "11238A",
  "org_id": "org-uuid-here",
  "plan_id": "ensemble",
  "seat_count": 25
}
```

Response:
```json
{
  "valid": true,
  "sponsor_name": "Joe Smith",
  "discount_type": "free",
  "discount_amount": 0.00,
  "original_amount": 2500.00,
  "final_amount": 0.00,
  "expires_at": "2025-12-31T23:59:59Z",
  "sponsee_org_name": "Pine Valley School"
}
```

#### Apply Sponsor Code to Checkout
```http
POST /api/v1/checkout/apply-sponsor-code
Content-Type: application/json

{
  "session_id": "checkout-session-uuid",
  "code": "11238A",
  "org_id": "org-uuid-here"
}
```

## Business Logic

### Sponsor Code Generation Rules

1. **Automatic Generation**
   - Generated when Ensemble Member subscribes: `{member_number}A`
   - Code becomes available only after member becomes Ensemble Member
   - Default expiration: 1 year from generation
   - Default max uses: 1 (single organization)

2. **Code Format Validation**
   - Pattern: `{member_number}A` (e.g., "11238A")
   - Member number must be valid and active
   - Must be unique across all sponsor codes
   - Case insensitive matching

3. **Sponsor Eligibility**
   - Must be active Ensemble Member
   - Account must be in good standing
   - No outstanding billing issues
   - Must have completed onboarding

### Assignment Process

1. **Pre-assignment Checks**
   - Verify sponsor code is active and unused
   - Confirm organization is eligible (non-profit, educational)
   - Validate contact information
   - Check assignment permissions

2. **Assignment Creation**
   - Create assignment record
   - Update sponsor code status to 'active'
   - Send notification to sponsor
   - Send notification to sponsee contact

3. **Assignment Management**
   - Track assignment status
   - Allow reassignment if unused
   - Handle expiration and revocation
   - Maintain audit trail

### Redemption Process

1. **Pre-redemption Validation**
   - Verify code format and existence
   - Check code is active and not expired
   - Confirm organization is assigned to code
   - Validate subscription plan eligibility

2. **Discount Calculation**
   ```python
   def calculate_sponsor_discount(sponsor_code, subscription_amount):
       """
       Calculate discount based on sponsor code configuration
       """
       if sponsor_code.discount_type == 'free':
           return subscription_amount
       elif sponsor_code.discount_type == 'percent':
           return subscription_amount * (sponsor_code.discount_value / 100)
       elif sponsor_code.discount_type == 'fixed':
           return min(sponsor_code.discount_value, subscription_amount)
       
       return 0.00
   ```

3. **Redemption Recording**
   - Create usage record
   - Update sponsor code usage count
   - Mark code as used if max uses reached
   - Apply discount to subscription
   - Send confirmation notifications

### Sponsor Bank Management

When sponsor codes are not used within the specified timeframe, they enter the "sponsor bank" for administrative assignment:

1. **Bank Entry Criteria**
   - Code unused for 90 days after generation
   - Sponsor has not manually assigned code
   - Code not expired

2. **Bank Management**
   - Site administrators can view available codes
   - Manual assignment to eligible organizations
   - Priority given to high-need educational institutions
   - Tracking of bank usage and effectiveness

## Integration Points

### Member Dashboard Integration

Sponsor codes appear on Ensemble Member dashboards with:

- **Code Display**: Show active sponsor code
- **Usage Status**: Track if code has been used
- **Assignment Info**: Display sponsee organization details
- **Management Actions**: Revoke, extend, or reassign codes

### Subscription System Integration

```python
class SponsorCodeService:
    def apply_to_subscription(self, subscription_id, sponsor_code_id, org_id):
        """Apply sponsor code discount to subscription"""
        subscription = self.get_subscription(subscription_id)
        sponsor_code = self.get_sponsor_code(sponsor_code_id)
        
        # Validate assignment
        if not self.is_code_assigned_to_org(sponsor_code_id, org_id):
            raise ValidationError("Code not assigned to this organization")
        
        # Calculate discount
        discount_amount = self.calculate_sponsor_discount(
            sponsor_code, 
            subscription.total_amount
        )
        
        # Create adjustment
        adjustment = Adjustment(
            subscription_id=subscription_id,
            type='sponsor_code',
            amount=-discount_amount,
            description=f"Sponsor code: {sponsor_code.code} (Sponsored by {sponsor_code.sponsor_name})",
            sponsor_code_id=sponsor_code_id
        )
        
        # Update subscription
        subscription.adjustments.append(adjustment)
        subscription.recalculate_totals()
        
        return adjustment
```

### Analytics Integration

Track key metrics for program effectiveness:

- **Sponsor Engagement**: Code generation and usage rates
- **Educational Impact**: Number of organizations supported
- **Financial Impact**: Total value of sponsored subscriptions
- **Program Growth**: Trends in sponsor participation

## Error Handling

### Validation Errors

```json
{
  "error": "INVALID_SPONSOR_CODE",
  "message": "The sponsor code 'INVALID123' is not valid",
  "code": "SPONSOR_CODE_NOT_FOUND"
}
```

### Assignment Errors

```json
{
  "error": "CODE_NOT_ASSIGNED",
  "message": "This sponsor code is not assigned to your organization",
  "code": "ASSIGNMENT_MISMATCH",
  "details": {
    "code": "11238A",
    "assigned_org": "Pine Valley School"
  }
}
```

### Business Logic Errors

```json
{
  "error": "SPONSOR_CODE_EXPIRED",
  "message": "This sponsor code has expired and is no longer valid",
  "code": "CODE_EXPIRED",
  "expired_at": "2024-12-31T23:59:59Z"
}
```

## Security Considerations

### Access Controls

1. **Sponsor Code Management**: Only Ensemble Members can generate and manage their own codes
2. **Assignment Management**: Only system administrators can assign codes to organizations
3. **Redemption**: Only assigned organizations can use sponsor codes
4. **Audit Access**: Full audit trail for all sponsor code activities

### Data Protection

1. **PII Handling**: Minimize personal data in sponsor code records
2. **Organization Data**: Protect sponsee organization information
3. **Audit Logging**: Log all sponsor code activities
4. **Data Retention**: Define retention policies for usage data

## Performance Considerations

### Caching Strategy

- **Active Codes**: Cache active sponsor codes in Redis
- **Assignment Lookups**: Cache code-to-organization mappings
- **Usage Counts**: Cache usage statistics with TTL

### Database Optimization

- **Indexes**: Optimize queries on code, status, and assignment fields
- **Partitioning**: Consider partitioning usage tables by date
- **Archiving**: Archive old usage data to reduce table size

## Monitoring and Alerting

### Key Metrics

- **Code Generation Rate**: New sponsor codes created per month
- **Usage Rate**: Percentage of codes that get used
- **Program Impact**: Number of organizations supported
- **System Performance**: API response times and error rates

### Alerts

- **High Error Rate**: API errors exceeding threshold
- **Code Expiration**: Codes expiring within 7 days
- **Bank Management**: Codes entering sponsor bank
- **Unusual Activity**: Suspicious code usage patterns

## Testing Strategy

### Unit Tests

- Sponsor code generation logic
- Assignment validation rules
- Discount calculation algorithms
- Error handling scenarios

### Integration Tests

- API endpoint functionality
- Database operations
- Subscription system integration
- Notification system integration

### Load Tests

- High-volume code validation
- Concurrent assignment handling
- Database performance under load
- Cache effectiveness

## Supporting Documents Referenced

This sponsor codes implementation specification draws from the following source documents:

- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Pricing calculations for sponsored subscriptions
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Seat allocation for sponsored organizations
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Ensemble member sponsorship permissions

## Deployment Considerations

### Feature Flags

- Enable/disable sponsor code functionality
- A/B testing for different discount strategies
- Gradual rollout of new features

### Database Migrations

- Schema changes for new fields
- Data migration for existing codes
- Index creation for performance

### Rollback Plan

- Disable sponsor code functionality
- Revert to standard pricing
- Restore from backup if needed
