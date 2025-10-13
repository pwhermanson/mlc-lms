# Plans and Pricing

## Purpose

This specification defines the subscription plans, pricing tiers, and pricing structure for the MLC-LMS platform. It establishes the foundation for all billing operations by defining what plans are available, how pricing is calculated, and what features are included in each tier.

The pricing structure supports three distinct customer segments: individual music studios (Solo), educational institutions (Ensemble), and evaluation users (Prelude), with volume-based pricing that scales appropriately for each market.

## Scope

### Included
- Complete pricing structure for all subscription plans
- Seat-based pricing tiers and volume discounts
- Monthly and annual billing options with discount structures
- Plan features and limitations for each tier
- Pricing calculation rules and formulas
- Special pricing for educational institutions
- Trial and evaluation pricing

### Excluded
- Detailed billing operations and payment processing (see [billing-plans-and-seats.md](billing-plans-and-seats.md))
- Seat management operations (see [seat-management.md](seat-management.md))
- Promotional codes and discounts (see [promo-codes.md](promo-codes.md))
- Checkout and trial flows (see [checkout-and-trials.md](checkout-and-trials.md))
- Invoice generation and statements (see [billing-statements-on-demand.md](billing-statements-on-demand.md))

## Subscription Plans

### Prelude Plan (Free Trial)
**Target**: Prospective customers evaluating the platform
**Duration**: 14 days (2 weeks)
**Seats**: Up to 19 seats included
**Features**: Full platform access with all features enabled
**Billing**: No payment required, no automatic conversion
**Purpose**: Allow potential customers to fully evaluate the platform with a realistic number of seats

### Solo Plan (Studio Plan)
**Target**: Individual music studios, private teachers, and small music schools
**Base Pricing**: $7.95/month for 5 seats
**Additional Seats**: $0.80 per seat per month (up to 19 seats maximum)
**Annual Pricing**: $95.40/year for 5 seats ($19.08 per seat)
**Additional Annual Seats**: $9.60 per seat per year
**Features**: 
- Full platform access
- Basic reporting and analytics
- Email support
- Student progress tracking
- Assignment creation and management

### Ensemble Plan (School Plan)
**Target**: Schools, school districts, and large music programs
**Pricing Structure**: Volume-based tiered pricing with significant discounts at scale

#### Ensemble Pricing Tiers

| Seat Range | Monthly Base | Per Additional Seat | Annual Base | Per Additional Seat (Annual) | Effective Per-Seat Cost |
|------------|--------------|-------------------|-------------|----------------------------|------------------------|
| 20-119 | $19.95 | $0.20 | $239.40 | $2.40 | $11.97 |
| 120-239 | $39.95 | $0.18 | $479.40 | $2.16 | $4.00 |
| 240-499 | $61.55 | $0.16 | $738.60 | $1.92 | $3.08 |
| 500-999 | $103.15 | $0.12 | $1,237.80 | $1.44 | $2.48 |
| 1,000-2,499 | $163.15 | $0.10 | $1,957.80 | $1.20 | $1.96 |
| 2,500-4,199 | $313.15 | $0.08 | $3,757.80 | $0.96 | $1.50 |
| 4,200-5,999 | $449.15 | $0.06 | $5,389.80 | $0.72 | $1.28 |
| 6,000+ | $557.15 | $0.05 | $6,685.80 | $0.60 | $1.11 |

**Features**:
- Full platform access
- Advanced reporting and analytics
- Bulk operations and management tools
- Phone and email support
- Purchase order support
- Custom billing and invoicing
- District-level administration
- Advanced student progress tracking
- Custom assignment libraries

## Pricing Models

### Volume Discount Structure
The Ensemble plan uses a declining per-seat pricing model where the cost per seat decreases as the total number of seats increases. This encourages larger institutions to adopt the platform while maintaining profitability.

### Annual vs Monthly Pricing
- **Annual Discount**: Approximately 20% savings compared to monthly billing
- **Annual Billing**: Preferred for Ensemble plans, especially for educational institutions
- **Monthly Billing**: Available for all plans but at higher per-seat costs

### Seat Management
- **Seat Addition**: Can be added at any time with prorated billing
- **Seat Removal**: Typically takes effect at the end of the billing period
- **Seat Reassignment**: Seats can be reassigned between users without additional cost
- **Seat Limits**: Solo plan capped at 19 seats; Ensemble plan has no upper limit

## Educational Institution Pricing

### School District Pricing
For very large implementations (typically 1,000+ seats), custom pricing may be available with additional discounts beyond the standard Ensemble tiers.

### Purchase Order Support
- Schools can use purchase orders for annual billing
- PO numbers are captured and included on all invoices
- Special invoicing formats available for school accounting systems

### Special Considerations
- **Academic Year Billing**: Align billing cycles with academic calendars
- **Summer Pause**: Option to pause billing during summer months
- **Teacher Training**: Included training and onboarding support
- **Implementation Support**: Dedicated support for large deployments

## Pricing Calculation Examples

### Solo Plan Examples
- **5 seats monthly**: $7.95/month ($1.59 per seat)
- **10 seats monthly**: $7.95 + (5 × $0.80) = $11.95/month ($1.20 per seat)
- **15 seats monthly**: $7.95 + (10 × $0.80) = $15.95/month ($1.06 per seat)

### Ensemble Plan Examples
- **50 seats monthly**: $19.95 + (30 × $0.20) = $25.95/month ($0.52 per seat)
- **100 seats monthly**: $39.95 + (0 × $0.18) = $39.95/month ($0.40 per seat)
- **500 seats monthly**: $103.15 + (0 × $0.12) = $103.15/month ($0.21 per seat)
- **1,000 seats monthly**: $163.15 + (0 × $0.10) = $163.15/month ($0.16 per seat)

### Annual Pricing Examples
- **50 seats annual**: $239.40 + (30 × $2.40) = $311.40/year ($6.23 per seat)
- **100 seats annual**: $479.40 + (0 × $2.16) = $479.40/year ($4.79 per seat)
- **500 seats annual**: $1,237.80 + (0 × $1.44) = $1,237.80/year ($2.48 per seat)

## Plan Features Comparison

| Feature | Prelude | Solo | Ensemble |
|---------|---------|------|----------|
| Platform Access | Full | Full | Full |
| Seat Limit | 19 | 19 | Unlimited |
| Duration | 14 days | Ongoing | Ongoing |
| Reporting | Basic | Basic | Advanced |
| Support | Email | Email | Phone + Email |
| Bulk Operations | No | No | Yes |
| Purchase Orders | No | No | Yes |
| Custom Billing | No | No | Yes |
| District Admin | No | No | Yes |
| API Access | No | Limited | Full |

## Pricing Rules and Validations

### Seat Limits
- **Prelude**: Maximum 19 seats
- **Solo**: Maximum 19 seats (must upgrade to Ensemble for more)
- **Ensemble**: No upper limit

### Billing Cycle Rules
- **Monthly Billing**: Can be changed to annual at any time
- **Annual Billing**: Cannot be changed to monthly mid-cycle
- **Proration**: Applied for mid-cycle changes

### Plan Upgrade/Downgrade Rules
- **Upgrade**: Can upgrade from Solo to Ensemble at any time
- **Downgrade**: Can only downgrade at the end of billing period
- **Prelude**: Must convert to Solo or Ensemble before expiration

## UX Requirements

### Pricing Display
- Clear pricing tiers with per-seat costs prominently displayed
- Interactive calculator showing total cost based on seat count
- Comparison table highlighting differences between plans
- Annual vs monthly savings clearly shown

### Plan Selection
- Visual indicators for recommended plans based on seat count
- Feature comparison matrix
- Clear call-to-action buttons for each plan
- Trial signup prominently featured

### Checkout Process
- Seat count selector with real-time price updates
- Billing cycle selection (monthly/annual)
- Clear summary of charges before payment
- Promotional code entry field

## Telemetry

### Pricing Events
- `pricing_calculated`
  - `plan_id`, `seat_count`, `billing_cycle`, `total_cost`, `per_seat_cost`
- `plan_comparison_viewed`
  - `viewed_plans[]`, `seat_count`, `user_type`
- `pricing_calculator_used`
  - `seat_count`, `billing_cycle`, `calculated_cost`

### Plan Selection Events
- `plan_selected`
  - `plan_id`, `seat_count`, `billing_cycle`, `total_cost`
- `trial_started`
  - `plan_id`, `seat_count`, `trial_duration`
- `plan_upgraded`
  - `from_plan_id`, `to_plan_id`, `seat_count`, `upgrade_reason`

## Permissions

### Public Access
- View pricing information and plan details
- Use pricing calculator
- Start free trial

### Authenticated Users
- View current plan and pricing
- Access billing history
- Change billing cycles (if allowed)

### Subscriber Admins
- View and modify subscription details
- Change seat counts
- Upgrade/downgrade plans
- Access detailed billing information

### System Admins
- Override pricing rules
- Apply custom pricing
- Manage promotional pricing
- Access all billing data

## Supporting Documents Referenced

This plans and pricing specification draws from the following source documents:

- [MLC Tiered Pricing 2020.xlsx - Chart.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Chart.csv) — Pricing tier visualization and structure
- [MLC Tiered Pricing 2020.xlsx - Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Calc.csv) — Pricing calculation formulas and seat-based pricing
- [MLC Tiered Pricing 2020.xlsx - Full Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Full%20Calc.csv) — Complete pricing model calculations
- [Screeens Text.docx.txt](../00-ORIG-CONTEXT/Screeens%20Text.docx.txt) — Pricing page copy and plan descriptions
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Seat allocation and management specifications

## Dependencies

- [billing-plans-and-seats.md](billing-plans-and-seats.md) - Detailed billing operations
- [seat-management.md](seat-management.md) - Seat management operations
- [checkout-and-trials.md](checkout-and-trials.md) - Checkout and trial flows
- [promo-codes.md](promo-codes.md) - Promotional pricing
- [billing-provider-integration.md](billing-provider-integration.md) - Payment processing
- [seat-pricing-calculator.md](seat-pricing-calculator.md) - Pricing calculation tools

## Open Questions

1. **Custom Enterprise Pricing**: What criteria determine eligibility for custom enterprise pricing beyond standard Ensemble tiers?

2. **International Pricing**: How will pricing be adjusted for different international markets and currencies?

3. **Mid-Cycle Plan Changes**: What are the exact proration rules for mid-cycle plan changes, especially for annual to monthly transitions?

4. **Seat Overage**: How should the system handle cases where seat usage exceeds purchased seats?

5. **Trial Conversion**: What is the optimal trial duration and conversion strategy for different customer segments?

6. **Volume Discounts**: Are there additional volume discounts available for very large implementations (10,000+ seats)?

7. **Educational Discounts**: Should there be additional discounts for qualifying educational institutions beyond the standard Ensemble pricing?

8. **Promotional Pricing**: How will promotional pricing integrate with the base pricing structure defined here?
