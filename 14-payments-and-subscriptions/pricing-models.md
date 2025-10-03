# Pricing Models and Tiers

## Overview
Comprehensive pricing strategy and subscription tier specifications for all customer segments. This document defines the mathematical models, pricing formulas, and business logic that govern how subscription costs are calculated across all tiers and scenarios.

## Description
Detailed pricing structure for Solo, Ensemble, and Prelude subscription tiers, including seat management, billing cycles, promotional pricing, and special offers for educational institutions. The pricing models are designed to support volume-based discounts, encourage long-term commitments through annual billing, and provide clear cost predictability for different customer segments.

## Pricing Model Architecture

### Core Pricing Principles
- **Volume-Based Discounts**: Per-seat costs decrease as total seat count increases
- **Annual Commitment Rewards**: 20% savings for annual billing vs monthly
- **Educational Institution Focus**: Specialized pricing for schools and districts
- **Transparent Pricing**: Clear, predictable costs with no hidden fees
- **Scalable Growth**: Pricing structure supports growth from 5 to 75,000+ seats

### Mathematical Foundation

#### Solo Plan Pricing Formula
```
Monthly Cost = $7.95 + (Additional Seats × $0.80)
Annual Cost = $95.40 + (Additional Seats × $9.60)
Maximum Seats = 19
```

#### Ensemble Plan Pricing Formula
The Ensemble plan uses a tiered pricing structure with declining per-seat costs:

```
Base Cost + (Seats in Tier × Tier Rate) = Total Monthly Cost
```

Where each tier has:
- **Base Cost**: Fixed monthly cost for the tier
- **Tier Rate**: Per-seat cost for seats within that tier
- **Seat Range**: Minimum and maximum seats for the tier

## Detailed Pricing Tiers

### Prelude Plan (Free Trial)
**Target Market**: Prospective customers evaluating the platform
**Duration**: 14 days (2 weeks)
**Seat Limit**: 19 seats maximum
**Cost**: $0.00 (completely free)
**Features**: Full platform access with all features enabled
**Conversion**: Must convert to Solo or Ensemble before expiration
**Purpose**: Allow potential customers to fully evaluate the platform with realistic seat counts

### Solo Plan (Studio Plan)
**Target Market**: Individual music studios, private teachers, small music schools
**Pricing Model**: Fixed base cost + per-seat pricing
**Seat Range**: 5-19 seats
**Billing Options**: Monthly or Annual

#### Solo Plan Pricing Structure
| Seat Count | Monthly Cost | Per Seat (Monthly) | Annual Cost | Per Seat (Annual) | Effective Annual Savings |
|------------|--------------|-------------------|-------------|-------------------|-------------------------|
| 5 | $7.95 | $1.59 | $95.40 | $19.08 | 20% |
| 10 | $11.95 | $1.20 | $143.40 | $14.34 | 20% |
| 15 | $15.95 | $1.06 | $191.40 | $12.76 | 20% |
| 19 | $19.15 | $1.01 | $229.80 | $12.09 | 20% |

**Features Included**:
- Full platform access
- Basic reporting and analytics
- Email support
- Student progress tracking
- Assignment creation and management
- Up to 19 seats maximum

### Ensemble Plan (School Plan)
**Target Market**: Schools, school districts, large music programs
**Pricing Model**: Tiered volume-based pricing with significant discounts at scale
**Seat Range**: 20+ seats (unlimited)
**Billing Options**: Monthly or Annual (Annual preferred for educational institutions)

#### Ensemble Plan Tier Structure

| Tier | Seat Range | Monthly Base | Per Additional Seat | Annual Base | Per Additional Seat (Annual) | Effective Per-Seat Cost (Monthly) | Effective Per-Seat Cost (Annual) |
|------|------------|--------------|-------------------|-------------|----------------------------|-----------------------------------|----------------------------------|
| 1 | 20-119 | $19.95 | $0.20 | $239.40 | $2.40 | $0.998 | $0.599 |
| 2 | 120-239 | $39.95 | $0.18 | $479.40 | $2.16 | $0.333 | $0.200 |
| 3 | 240-499 | $61.55 | $0.16 | $738.60 | $1.92 | $0.256 | $0.154 |
| 4 | 500-999 | $103.15 | $0.12 | $1,237.80 | $1.44 | $0.206 | $0.124 |
| 5 | 1,000-2,499 | $163.15 | $0.10 | $1,957.80 | $1.20 | $0.163 | $0.098 |
| 6 | 2,500-4,199 | $313.15 | $0.08 | $3,757.80 | $0.96 | $0.125 | $0.075 |
| 7 | 4,200-5,999 | $449.15 | $0.06 | $5,389.80 | $0.72 | $0.107 | $0.064 |
| 8 | 6,000+ | $557.15 | $0.05 | $6,685.80 | $0.60 | $0.093 | $0.056 |

#### Ensemble Plan Calculation Examples

**Example 1: 50 Seats (Tier 1)**
- Monthly: $19.95 + (30 × $0.20) = $25.95
- Annual: $239.40 + (30 × $2.40) = $311.40
- Per-seat cost: $0.52 monthly, $0.62 annual

**Example 2: 200 Seats (Tier 2)**
- Monthly: $39.95 + (80 × $0.18) = $54.35
- Annual: $479.40 + (80 × $2.16) = $652.20
- Per-seat cost: $0.27 monthly, $0.33 annual

**Example 3: 1,000 Seats (Tier 5)**
- Monthly: $163.15 + (0 × $0.10) = $163.15
- Annual: $1,957.80 + (0 × $1.20) = $1,957.80
- Per-seat cost: $0.16 monthly, $0.20 annual

**Example 4: 5,000 Seats (Tier 8)**
- Monthly: $557.15 + (0 × $0.05) = $557.15
- Annual: $6,685.80 + (0 × $0.60) = $6,685.80
- Per-seat cost: $0.11 monthly, $0.13 annual

## Volume Discount Analysis

### Discount Progression
The Ensemble plan demonstrates significant volume discounts as seat count increases:

| Seat Count | Monthly Per-Seat | Annual Per-Seat | Discount from Base |
|------------|------------------|-----------------|-------------------|
| 20 | $0.998 | $0.599 | 0% |
| 100 | $0.360 | $0.216 | 64% |
| 500 | $0.206 | $0.124 | 79% |
| 1,000 | $0.163 | $0.098 | 84% |
| 5,000 | $0.111 | $0.067 | 89% |
| 10,000 | $0.076 | $0.046 | 92% |
| 25,000 | $0.060 | $0.036 | 94% |
| 50,000 | $0.053 | $0.032 | 95% |
| 75,000 | $0.053 | $0.032 | 95% |

### Cost Efficiency Thresholds
- **Break-even with Solo**: 25 seats (Solo max is 19, so 20+ requires Ensemble)
- **Significant savings**: 100+ seats (per-seat cost drops below $0.40)
- **Maximum efficiency**: 6,000+ seats (per-seat cost stabilizes at $0.11)

## Billing Model Specifications

### Monthly Billing
- **Billing Cycle**: Calendar month
- **Payment Due**: First day of each month
- **Proration**: Mid-cycle changes prorated to month end
- **Seat Changes**: Can add/remove seats with immediate proration
- **Cancellation**: Can cancel at month end

### Annual Billing
- **Billing Cycle**: 12-month period from start date
- **Payment Due**: Start of annual period
- **Discount**: 20% savings vs monthly equivalent
- **Seat Changes**: Can add seats anytime, prorated to year end
- **Seat Removal**: Typically takes effect at year end
- **Cancellation**: Can cancel at year end (no mid-year cancellation)

### Proration Calculations
**Monthly Proration Formula**:
```
Prorated Amount = (Monthly Rate × Days Remaining) / Days in Month
```

**Annual Proration Formula**:
```
Prorated Amount = (Annual Rate × Days Remaining) / 365
```

## Educational Institution Pricing

### School District Considerations
- **Volume Discounts**: Automatic application of Ensemble tier pricing
- **Purchase Order Support**: Annual billing with PO numbers
- **Academic Year Alignment**: Billing cycles aligned with school calendar
- **Summer Pause Options**: Optional billing suspension during summer months
- **Implementation Support**: Dedicated onboarding and training

### Custom Enterprise Pricing
For very large implementations (typically 10,000+ seats):
- **Custom Tiers**: Negotiated pricing below standard Tier 8 rates
- **Dedicated Support**: Priority support and account management
- **Custom Features**: Tailored feature sets and integrations
- **Contract Terms**: Multi-year agreements with additional discounts

## Pricing Model Validation

### Business Rules
1. **Seat Limits**: Solo plan cannot exceed 19 seats
2. **Plan Upgrades**: Solo to Ensemble upgrades allowed anytime
3. **Plan Downgrades**: Only allowed at billing period end
4. **Trial Conversion**: Must convert before 14-day expiration
5. **Billing Consistency**: Annual rates always 20% less than monthly

### Calculation Accuracy
- **Rounding**: All calculations rounded to nearest cent
- **Tax Handling**: Pricing excludes applicable taxes
- **Currency**: All pricing in USD
- **Precision**: Calculations maintain 4 decimal places internally

## Competitive Analysis Context

### Market Positioning
- **Solo Plan**: Competitive with individual music lesson management tools
- **Ensemble Plan**: Positioned between basic LMS and enterprise solutions
- **Volume Pricing**: More aggressive discounts than most educational software
- **Annual Discounts**: Standard 20% discount aligns with industry norms

### Value Proposition
- **Transparent Pricing**: No hidden fees or complex calculations
- **Scalable Growth**: Pricing supports growth from 5 to 75,000+ seats
- **Educational Focus**: Specialized features and pricing for music education
- **Flexible Billing**: Monthly and annual options with easy transitions

## Implementation Considerations

### Technical Requirements
- **Pricing Engine**: Real-time calculation of costs based on seat count
- **Billing Integration**: Seamless integration with payment processors
- **Proration Logic**: Accurate mid-cycle billing calculations
- **Audit Trail**: Complete history of pricing changes and calculations

### Business Intelligence
- **Usage Analytics**: Track pricing model effectiveness
- **Conversion Metrics**: Monitor trial-to-paid conversion rates
- **Churn Analysis**: Identify pricing-related cancellation patterns
- **Revenue Forecasting**: Predict revenue based on pricing models

### Telemetry Integration
Pricing model calculations integrate with the platform's telemetry system. For detailed event specifications and data collection requirements, see the telemetry section in [plans-and-pricing.md](plans-and-pricing.md), which defines events such as:
- `pricing_calculated` - When pricing calculations are performed
- `plan_comparison_viewed` - When users view pricing comparisons
- `pricing_calculator_used` - When users interact with pricing calculators

## Future Pricing Considerations

### Potential Enhancements
- **International Pricing**: Currency and regional adjustments
- **Promotional Pricing**: Time-limited discounts and offers
- **Feature-Based Pricing**: Additional costs for premium features
- **Usage-Based Pricing**: Pay-per-use models for specific features

### Market Adaptations
- **Competitive Response**: Adjustments based on competitor pricing
- **Customer Feedback**: Pricing model refinements based on user input
- **Market Expansion**: New pricing tiers for different market segments
- **Technology Costs**: Adjustments based on infrastructure and development costs

## Dependencies

### Related Specifications
- [plans-and-pricing.md](plans-and-pricing.md) - High-level plan definitions and features
- [billing-plans-and-seats.md](billing-plans-and-seats.md) - Billing operations and seat management
- [seat-management.md](seat-management.md) - Detailed seat management operations
- [checkout-and-trials.md](checkout-and-trials.md) - Checkout flows and trial management
- [billing-provider-integration.md](billing-provider-integration.md) - Payment processing integration
- [seat-pricing-calculator.md](seat-pricing-calculator.md) - Pricing calculation tools and interfaces

### Technical Dependencies
- **Pricing Engine**: Real-time calculation system for cost computations
- **Billing System**: Integration with payment processors and invoicing
- **Database**: Storage for pricing rules, calculations, and historical data
- **Audit System**: Logging and tracking of pricing model changes
- **Analytics Platform**: Data collection and analysis for pricing effectiveness
