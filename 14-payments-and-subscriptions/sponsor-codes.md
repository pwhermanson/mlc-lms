# Sponsor Codes System

## Overview
Sponsor code functionality for supporting non-profit educational organizations through the Christine Hermanson Grant Program, enabling Ensemble Members to provide free or discounted access to educational institutions.

## Description
System for managing sponsor codes that enable Ensemble Members to support non-profit schools and educational institutions with discounted or free access to the platform. Unlike promotional codes which are marketing tools, sponsor codes are relationship-based codes that create connections between Ensemble Members and educational institutions, facilitating philanthropic support for music education.

## Christine Hermanson Grant Program

### Program Foundation
The Christine Hermanson Grant Program was established in honor of the platform's founder, Christine Hermanson, who dedicated her occupational life to music education, particularly early learners. As a nationally-recognized pioneer in music education technology, her teaching philosophy included the conviction that anyone could learn, make, and enjoy music—that music is an integral part of every individual's experience.

### Program Purpose
The program encourages early learners in music education by providing schools with free subscriptions to MusicLearningCommunity.com for General Music classes through a simple sponsor program. This reflects Christine's discovery that computer technology could be very effective, particularly in learning music literacy.

### How It Works
- **Automatic Sponsorship**: Anyone who subscribes to the ENSEMBLE level (including a school) automatically becomes a sponsor
- **Free Solo Subscription**: Sponsors can provide a free SOLO subscription (of 5 seats) to be used for General Music
- **Ongoing Support**: As long as the ENSEMBLE subscription is active, the free sponsored subscription remains active
- **Unique Sponsor Code**: Each new Ensemble Member receives a unique "sponsor code" for assignment

## Sponsor Code Types and Structure

### Code Format
- **Pattern**: `{member_number}A` (e.g., "11238A")
- **Member Number**: Based on the Ensemble Member's unique identifier
- **Suffix**: "A" indicates this is a sponsor code (vs. promotional codes)
- **Case Insensitive**: Codes work regardless of case when entered

### Code Assignment Options
1. **Direct Assignment**: Sponsor provides code to a school of their choice
2. **Banked Assignment**: Sponsor "banks" the code with MusicLearningCommunity.com for administrative assignment
3. **Administrative Assignment**: Site administrators assign banked codes to eligible organizations

### Discount Types
- **Free Subscription**: Complete waiver of subscription fees (most common)
- **Percentage Discount**: Partial discount on subscription costs
- **Fixed Amount**: Specific dollar amount reduction

## User Experience Flows

### For Ensemble Members (Sponsors)

#### Code Generation
- **Automatic Creation**: Sponsor code generated upon Ensemble subscription activation
- **Dashboard Access**: Code appears in member dashboard with management options
- **Assignment Choice**: Option to assign directly or bank for administrative use
- **Status Tracking**: Monitor whether code has been used and by which organization

#### Code Management
- **View Active Codes**: See all generated sponsor codes and their status
- **Assignment Details**: View which organization is assigned to each code
- **Usage History**: Track when and how codes have been used
- **Revoke/Reassign**: Ability to revoke unused codes or reassign them

### For Educational Institutions (Sponsees)

#### Code Discovery
- **Direct Assignment**: Receive sponsor code from sponsoring Ensemble Member
- **Administrative Assignment**: Receive code through MusicLearningCommunity.com administrative process
- **Application Process**: Submit code during subscription signup or upgrade

#### Code Application
- **Checkout Integration**: Enter sponsor code during subscription process
- **Real-time Validation**: Instant feedback on code validity and benefits
- **Discount Application**: Automatic application of free or discounted pricing
- **Confirmation**: Clear display of sponsored subscription details

### For Administrators

#### Code Management
- **Bank Management**: View and assign banked sponsor codes
- **Organization Verification**: Confirm educational institution eligibility
- **Assignment Tracking**: Monitor code assignments and usage
- **Program Analytics**: Track program effectiveness and impact

## Business Rules and Eligibility

### Sponsor Eligibility
- **Active Ensemble Member**: Must have active Ensemble subscription
- **Good Standing**: Account must be in good standing with no billing issues
- **Completed Onboarding**: Must have completed account setup process
- **One Code Per Member**: Each Ensemble Member receives one sponsor code

### Sponsee Eligibility
- **Educational Institution**: Must be a recognized educational organization
- **Non-profit Status**: Preference given to non-profit educational institutions
- **General Music Focus**: Primary use should be for General Music classes
- **Valid Contact**: Must provide valid institutional contact information

### Code Usage Rules
- **Single Use**: Each sponsor code can typically be used once
- **Organization Specific**: Codes are assigned to specific organizations
- **Time Limited**: Codes may have expiration dates (typically 1 year)
- **Plan Specific**: Codes may be limited to specific subscription plans

## Integration with Subscription System

### Checkout Process
- **Code Entry**: Prominent field for entering sponsor code during checkout
- **Validation**: Real-time validation of code format and eligibility
- **Discount Calculation**: Automatic calculation and application of discounts
- **Pricing Display**: Clear display of original price, discount, and final amount

### Billing Integration
- **Invoice Generation**: Discounts properly reflected on billing statements
- **Payment Processing**: Codes applied before payment authorization
- **Adjustment Tracking**: Detailed tracking of sponsor code adjustments
- **Audit Trail**: Complete record of all sponsor code activities

### Subscription Management
- **Seat Allocation**: Proper allocation of sponsored seats
- **Renewal Handling**: Automatic renewal of sponsored subscriptions
- **Upgrade Options**: Ability to upgrade sponsored subscriptions
- **Cancellation Rules**: Special handling for sponsored subscription cancellations

## Program Benefits and Impact

### For Sponsors
- **Philanthropic Impact**: Direct support of music education in schools
- **Community Building**: Connection with educational institutions
- **Tax Benefits**: Potential tax advantages for educational donations
- **Brand Recognition**: Credit for sponsorship in program materials

### For Educational Institutions
- **Free Access**: No-cost access to comprehensive music learning platform
- **Curriculum Support**: Alignment with music education standards
- **Student Engagement**: Interactive games that enhance learning
- **Teacher Resources**: Access to teacher tools and reporting features

### For the Platform
- **Educational Mission**: Fulfillment of founder's educational vision
- **Market Expansion**: Increased adoption in educational institutions
- **Community Impact**: Support for music education worldwide
- **Brand Value**: Enhanced reputation as education-focused platform

## Success Metrics and Reporting

### Program Effectiveness
- **Code Generation Rate**: Number of sponsor codes created per month
- **Usage Rate**: Percentage of codes that get used by institutions
- **Educational Impact**: Number of students served through sponsored subscriptions
- **Geographic Reach**: Distribution of sponsored institutions across regions

### Sponsor Engagement
- **Assignment Rate**: Percentage of codes that get assigned to institutions
- **Direct vs. Banked**: Ratio of direct assignments to banked codes
- **Renewal Impact**: Effect on sponsor subscription renewals
- **Satisfaction Scores**: Sponsor feedback on program experience

### Educational Outcomes
- **Student Participation**: Number of students using sponsored access
- **Teacher Adoption**: Level of teacher engagement with platform
- **Learning Progress**: Measurable improvement in student outcomes
- **Institutional Retention**: Long-term adoption by educational institutions

## Supporting Documents Referenced

This sponsor codes specification draws from the following source documents:

- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Seat allocation for sponsored educational organizations
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Ensemble member role and sponsorship capabilities
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Christine Hermanson Grant Program messaging

## Future Enhancements

### Program Expansion
- **Multiple Codes**: Allow multiple sponsor codes per Ensemble Member
- **Tiered Sponsorship**: Different levels of sponsorship support
- **Recurring Sponsorship**: Ongoing sponsorship commitments
- **Corporate Sponsorship**: Business and corporate sponsorship options

### Enhanced Features
- **Impact Reporting**: Detailed reports on educational impact
- **Success Stories**: Sharing of program success stories
- **Recognition Programs**: Enhanced recognition for sponsors
- **Partnership Development**: Deeper partnerships with educational institutions

### Technology Improvements
- **Automated Matching**: AI-powered matching of sponsors and institutions
- **Mobile Integration**: Enhanced mobile experience for code management
- **Analytics Dashboard**: Comprehensive analytics for program management
- **API Integration**: Enhanced integration with school management systems

## Related Documentation

- [Sponsor Codes Implementation](../14-payments-and-subscriptions/sponsor-codes-impl.md) - Technical implementation details
- [Promotional Codes](../14-payments-and-subscriptions/promo-codes.md) - Marketing discount codes
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) - Subscription plan details
- [Seat Management](../14-payments-and-subscriptions/seat-management.md) - Student seat management
- [Company History](../00-foundations/company-history.md) - Christine Hermanson Grant Program history
- [Founder Biography](../00-foundations/founder-biography.md) - Christine Hermanson's legacy
