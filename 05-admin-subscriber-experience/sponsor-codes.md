# Sponsor Codes

## Purpose

This document defines the sponsor code management interface and workflows for Subscriber-Admins within the MLC-LMS platform. It covers the administrative tools, user experience flows, and business processes that enable subscribers to manage sponsor codes through the Christine Hermanson Grant Program, facilitating philanthropic support for music education.

Unlike promotional codes which are marketing tools, sponsor codes are relationship-based codes that create connections between Ensemble Members and educational institutions, enabling direct support of music education in schools.

## Scope

### Included
- Administrative interface for sponsor code management and assignment
- Christine Hermanson Grant Program workflow integration
- Sponsor-sponsee relationship tracking and management
- Integration with subscriber dashboard and billing systems
- User experience flows for code assignment and redemption
- Reporting and impact metrics for sponsored subscriptions

### Excluded
- Technical implementation details (see [Sponsor Codes Implementation](../14-payments-and-subscriptions/sponsor-codes-impl.md))
- System-wide sponsor code management (see [System Admin Tools](../06-system-admin/sysadmin-billing-tools.md))
- Christine Hermanson Grant Program policy details (see [Company History](../00-foundations/company-history.md))
- Email template management for sponsor notifications

## Christine Hermanson Grant Program Context

### Program Foundation
The Christine Hermanson Grant Program was established in honor of the platform's founder, Christine Hermanson, who dedicated her occupational life to music education, particularly early learners. As a nationally-recognized pioneer in music education technology, her teaching philosophy included the conviction that anyone could learn, make, and enjoy music—that music is an integral part of every individual's experience.

### Program Purpose
The program encourages early learners in music education by providing schools with free subscriptions to MusicLearningCommunity.com for General Music classes through a simple sponsor program. This reflects Christine's discovery that computer technology could be very effective, particularly in learning music literacy.

### How It Works
- **Automatic Sponsorship**: Anyone who subscribes to the ENSEMBLE level (including a school) automatically becomes a sponsor
- **Free Solo Subscription**: Sponsors can provide a free SOLO subscription (of 5 seats) to be used for General Music
- **Ongoing Support**: As long as the ENSEMBLE subscription is active, the free sponsored subscription remains active
- **Unique Sponsor Code**: Each new Ensemble Member receives a unique "sponsor code" for assignment

## Administrative Interface

### Sponsor Code Management Dashboard

#### Overview Section
- **My Sponsor Code**: Display of current sponsor code and status
- **Assignment Status**: Whether code has been assigned to an organization
- **Sponsored Organization**: Details of the currently sponsored institution
- **Program Impact**: Summary of educational impact through sponsorship
- **Code History**: Previous sponsor codes and their usage

#### Code Information Display
- **Sponsor Code**: The unique code (format: {member_number}A)
- **Generation Date**: When the code was created
- **Expiration Date**: When the code expires (typically 1 year)
- **Status**: Pending, Active, Used, Expired, or Revoked
- **Assignment Type**: Direct assignment or banked for administrative use

#### Assignment Management

##### Direct Assignment
When a sponsor has identified a specific educational institution to support:

- **Organization Search**: Find and select eligible educational institutions
- **Contact Information**: Enter sponsee contact details
- **Assignment Notes**: Add context about the sponsorship relationship
- **Confirmation Process**: Verify assignment before activation
- **Notification Settings**: Choose notification preferences for both parties

##### Banked Assignment
When a sponsor chooses to let administrators assign their code:

- **Bank Submission**: Submit code to the sponsor bank for administrative assignment
- **Priority Setting**: Indicate preferred types of organizations (if any)
- **Assignment Tracking**: Monitor when and to whom the code gets assigned
- **Impact Reporting**: Receive updates on the sponsored organization's progress

### Assignment Workflow

#### Pre-Assignment Validation
- **Eligibility Check**: Verify organization meets educational institution criteria
- **Non-profit Verification**: Confirm non-profit status when applicable
- **Contact Validation**: Ensure valid institutional contact information
- **Code Status**: Confirm sponsor code is active and available

#### Assignment Process
1. **Organization Selection**: Choose from eligible educational institutions
2. **Contact Details**: Enter sponsee contact information
3. **Assignment Confirmation**: Review and confirm assignment details
4. **Activation**: Activate the sponsor code for the assigned organization
5. **Notifications**: Send confirmation to both sponsor and sponsee

#### Post-Assignment Management
- **Status Monitoring**: Track whether the code has been used
- **Usage Details**: View when and how the code was redeemed
- **Impact Tracking**: Monitor the sponsored organization's engagement
- **Reassignment Options**: Ability to reassign if code goes unused

## User Experience Flows

### For Subscriber-Admins (Sponsors)

#### Code Generation and Management
- **Automatic Creation**: Sponsor code appears on dashboard upon Ensemble subscription
- **Code Details**: View code information, expiration, and usage status
- **Assignment Options**: Choose between direct assignment or banking the code
- **Status Updates**: Receive notifications about code status changes

#### Direct Assignment Process
1. **Access Code Management**: Navigate to sponsor code section of dashboard
2. **Select Assignment Type**: Choose "Assign to Organization"
3. **Search Organizations**: Find eligible educational institutions
4. **Enter Details**: Provide sponsee contact information
5. **Confirm Assignment**: Review and confirm assignment details
6. **Activate Code**: Make code available for the assigned organization

#### Banked Assignment Process
1. **Access Code Management**: Navigate to sponsor code section of dashboard
2. **Select Banking Option**: Choose "Bank for Administrative Assignment"
3. **Set Preferences**: Indicate any preferences for assignment (optional)
4. **Submit to Bank**: Submit code for administrative assignment
5. **Track Assignment**: Monitor when code gets assigned by administrators

### For Educational Institutions (Sponsees)

#### Code Discovery and Application
- **Code Receipt**: Receive sponsor code from sponsoring Ensemble Member
- **Checkout Integration**: Enter code during subscription signup process
- **Validation Feedback**: Receive confirmation of code validity and benefits
- **Subscription Activation**: Automatic application of sponsored pricing

#### Sponsored Subscription Management
- **Subscription Details**: View sponsored subscription information
- **Usage Tracking**: Monitor student engagement and progress
- **Teacher Resources**: Access to teacher tools and reporting features
- **Renewal Process**: Automatic renewal as long as sponsor remains active

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

### Checkout Process Integration
- **Code Entry Field**: Prominent field for entering sponsor code during checkout
- **Real-time Validation**: Instant feedback on code validity and benefits
- **Discount Application**: Automatic application of free or discounted pricing
- **Pricing Display**: Clear display of original price, discount, and final amount

### Billing Integration
- **Invoice Generation**: Discounts properly reflected on billing statements
- **Payment Processing**: Codes applied before payment authorization
- **Adjustment Tracking**: Detailed tracking of sponsor code adjustments
- **Audit Trail**: Complete record of all sponsor code activities

### Subscription Management
- **Seat Allocation**: Proper allocation of sponsored seats (typically 5 for Solo)
- **Renewal Handling**: Automatic renewal of sponsored subscriptions
- **Upgrade Options**: Ability to upgrade sponsored subscriptions
- **Cancellation Rules**: Special handling for sponsored subscription cancellations

## Reporting and Analytics

### Sponsor Impact Metrics
- **Code Usage Rate**: Whether assigned codes get used by institutions
- **Educational Impact**: Number of students served through sponsored access
- **Geographic Reach**: Distribution of sponsored institutions across regions
- **Program Effectiveness**: Long-term engagement of sponsored organizations

### Assignment Analytics
- **Direct vs. Banked**: Ratio of direct assignments to banked codes
- **Assignment Success**: Percentage of assigned codes that get used
- **Time to Assignment**: How quickly banked codes get assigned
- **Administrative Efficiency**: Effectiveness of administrative assignment process

### Educational Outcomes
- **Student Participation**: Number of students using sponsored access
- **Teacher Adoption**: Level of teacher engagement with platform
- **Learning Progress**: Measurable improvement in student outcomes
- **Institutional Retention**: Long-term adoption by educational institutions

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

## Administrative Tools and Features

### Code Management Tools
- **Code Status Updates**: Change code status (active, expired, revoked)
- **Assignment Modification**: Update assignment details or reassign codes
- **Expiration Management**: Handle code expiration and renewal
- **Usage Monitoring**: Track code usage and impact

### Reporting Tools
- **Impact Reports**: Generate reports on educational impact
- **Usage Analytics**: Detailed analytics on code usage patterns
- **Program Effectiveness**: Measure program success and areas for improvement
- **Export Capabilities**: Export data for external analysis

### Communication Tools
- **Notification System**: Automated notifications for code status changes
- **Assignment Confirmations**: Confirm assignments with both parties
- **Impact Updates**: Regular updates on sponsored organization progress
- **Program Communications**: Share program news and success stories

## Supporting Documents Referenced

This sponsor codes specification draws from the following source documents:

- [MLC Tiered Pricing 2020.xlsx files](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Full%20Calc.csv) — Pricing models and sponsored subscription specifications
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Subscriber-Admin and System Admin sponsor code management capabilities
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Sponsored seat allocation and subscription management

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
- [Promotional Codes](promo-codes.md) - Marketing discount codes for comparison
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) - Subscription plan details
- [Seat Management](../14-payments-and-subscriptions/seat-management.md) - Student seat management
- [Company History](../00-foundations/company-history.md) - Christine Hermanson Grant Program history
- [Founder Biography](../00-foundations/founder-biography.md) - Christine Hermanson's legacy
- [Subscriber Dashboard](subscriber-dashboard.md) - Main dashboard interface
- [Member Management](member-management.md) - Member management tools

