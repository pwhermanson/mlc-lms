# Promo Codes

## Purpose

This document defines the promotional code management interface and workflows for Subscriber-Admins within the MLC-LMS platform. It covers the administrative tools, user experience flows, and business processes that enable subscribers to create, manage, and track promotional codes for their organizations.

The system supports both Solo and Ensemble subscription models, allowing Subscriber-Admins to create codes for customer acquisition, retention campaigns, and special offers while maintaining proper oversight and control over promotional activities.

## Scope

### Included
- Administrative interface for promo code creation and management
- Code distribution and sharing workflows
- Usage tracking and analytics for subscriber-created codes
- Integration with subscriber dashboard and billing systems
- User experience flows for code application and redemption
- Reporting and performance metrics

### Excluded
- Technical implementation details (see [Promo Codes Implementation](../14-payments-and-subscriptions/promo-codes-impl.md))
- System-wide promo code management (see [System Admin Tools](../06-system-admin/sysadmin-billing-tools.md))
- Marketing campaign creation workflows
- Email template management

## Administrative Interface

### Code Management Dashboard

#### Overview Section
- **Active Codes**: Count of currently active promotional codes
- **Total Redemptions**: Aggregate usage across all subscriber codes
- **Revenue Impact**: Total savings provided to customers
- **Recent Activity**: Latest code creations, redemptions, and modifications
- **Performance Summary**: Top-performing codes and their metrics

#### Code Creation Workflow

##### Basic Information
- **Code Name**: Descriptive name for internal tracking (e.g., "Back to School 2024")
- **Promo Code**: Customer-facing code (e.g., "SCHOOL2024")
- **Description**: Internal notes about code purpose and target audience
- **Category**: Code type (acquisition, retention, seasonal, referral, etc.)

##### Discount Configuration
- **Discount Type**: Percentage or fixed amount
- **Discount Value**: Numerical value for the discount
- **Maximum Discount**: Cap for percentage discounts (optional)
- **Minimum Order**: Required purchase amount to use code
- **Applicable Plans**: Solo, Ensemble, or both

##### Usage Controls
- **Redemption Limit**: Maximum number of times code can be used
- **User Limit**: One per customer or unlimited use
- **Start Date**: When code becomes active
- **End Date**: When code expires (optional)
- **Scope**: Global or organization-specific

##### Advanced Settings
- **Auto-Generate Code**: System creates unique code or manual entry
- **Code Format**: Customizable format (e.g., "ORG-YYYY-####")
- **Notification Settings**: Email alerts for redemptions and expiration
- **Approval Required**: Whether code needs approval before activation

### Code List View

#### Display Columns
- **Code**: The actual promotional code
- **Name**: Internal descriptive name
- **Type**: Discount type and value
- **Status**: Active, expired, paused, or draft
- **Redemptions**: Current usage vs. limit
- **Created**: Date and creator
- **Expires**: Expiration date
- **Actions**: Edit, pause, duplicate, or delete

#### Filtering and Search
- **Status Filter**: Show active, expired, or all codes
- **Date Range**: Filter by creation or expiration date
- **Code Type**: Filter by discount type or category
- **Search**: Find codes by name, code, or description
- **Sort Options**: By creation date, usage, expiration, or name

#### Bulk Operations
- **Bulk Edit**: Modify multiple codes simultaneously
- **Bulk Pause**: Temporarily disable multiple codes
- **Bulk Delete**: Remove multiple codes (with confirmation)
- **Export**: Download code list and usage data

### Code Detail View

#### Code Information
- **Basic Details**: All code configuration settings
- **Usage Statistics**: Redemption count, unique users, revenue impact
- **Performance Metrics**: Conversion rate, average order value
- **Timeline**: Creation, modifications, and key events

#### Redemption History
- **Recent Redemptions**: Latest code uses with customer details
- **Usage Patterns**: When and how often code is used
- **Customer Insights**: Who is using the code and why
- **Geographic Data**: Where redemptions are coming from

#### Analytics Dashboard
- **Redemption Trends**: Usage over time with charts
- **Customer Segmentation**: Who responds to the code
- **Revenue Attribution**: Financial impact of the code
- **A/B Testing Results**: Performance comparison with other codes

## User Experience Flows

### Code Creation Process

#### Step 1: Planning
- **Campaign Goals**: Define objectives (acquisition, retention, revenue)
- **Target Audience**: Identify who should receive the code
- **Timing Strategy**: When to launch and expire the code
- **Success Metrics**: How to measure code effectiveness

#### Step 2: Configuration
- **Discount Setup**: Choose type and value based on campaign goals
- **Usage Limits**: Set appropriate redemption and user limits
- **Timing**: Define start and end dates for the campaign
- **Scope**: Determine who can use the code

#### Step 3: Testing
- **Validation**: Test code format and discount calculation
- **Preview**: See how code appears in checkout process
- **Sandbox Testing**: Verify code works in test environment
- **Approval**: Submit for review if required

#### Step 4: Launch
- **Activation**: Enable code for customer use
- **Distribution**: Share code through appropriate channels
- **Monitoring**: Track early usage and performance
- **Adjustments**: Make changes based on initial results

### Code Distribution Workflows

#### Email Campaigns
- **Template Selection**: Choose appropriate email template
- **Code Integration**: Embed code in email content
- **Personalization**: Customize message for recipient
- **Tracking**: Monitor email open and click rates

#### Website Integration
- **Banner Placement**: Add promotional banners to relevant pages
- **Checkout Integration**: Ensure code entry works smoothly
- **Landing Pages**: Create dedicated pages for specific codes
- **Mobile Optimization**: Ensure codes work on all devices

#### Social Media Sharing
- **Content Creation**: Develop shareable promotional content
- **Platform Optimization**: Adapt content for different social platforms
- **Tracking Links**: Monitor social media traffic and conversions
- **Viral Mechanics**: Encourage organic sharing of codes

### Code Monitoring and Management

#### Real-Time Monitoring
- **Usage Alerts**: Notifications when codes are used
- **Performance Dashboards**: Live view of code effectiveness
- **Error Tracking**: Monitor failed redemption attempts
- **System Health**: Ensure code system is functioning properly

#### Regular Review Process
- **Weekly Reviews**: Check code performance and usage
- **Monthly Analysis**: Comprehensive performance evaluation
- **Quarterly Planning**: Plan future promotional campaigns
- **Annual Assessment**: Evaluate overall promotional strategy

#### Optimization Workflows
- **A/B Testing**: Compare different code strategies
- **Performance Tuning**: Adjust codes based on results
- **Customer Feedback**: Incorporate user input into improvements
- **Best Practices**: Document successful strategies

## Integration with Subscriber Systems

### Dashboard Integration

#### Quick Actions
- **Create New Code**: One-click access to code creation
- **View Active Codes**: Quick overview of current promotions
- **Check Performance**: Real-time metrics on code effectiveness
- **Manage Expiring Codes**: Alerts and actions for expiring codes

#### Widgets and Metrics
- **Code Performance Widget**: Key metrics on dashboard
- **Recent Activity Feed**: Latest code-related actions
- **Usage Trends Chart**: Visual representation of code performance
- **Revenue Impact Summary**: Financial impact of promotional codes

### Billing Integration

#### Cost Tracking
- **Discount Impact**: Track total discounts provided
- **Revenue Attribution**: Measure revenue generated by codes
- **ROI Calculation**: Return on investment for promotional activities
- **Budget Management**: Monitor promotional spending against budget

#### Invoice Integration
- **Discount Display**: Show applied discounts on invoices
- **Code Attribution**: Link discounts to specific codes
- **Tax Handling**: Proper tax calculation with discounts
- **Refund Processing**: Handle code-related refunds appropriately

### User Management Integration

#### Customer Segmentation
- **Code User Profiles**: Identify customers who use promotional codes
- **Behavioral Analysis**: Understand promotional customer patterns
- **Lifetime Value**: Track long-term value of promotional customers
- **Retention Analysis**: Measure retention rates for promotional customers

#### Communication Tools
- **Targeted Messaging**: Send codes to specific customer segments
- **Follow-up Campaigns**: Engage customers after code redemption
- **Loyalty Programs**: Integrate codes with customer loyalty initiatives
- **Feedback Collection**: Gather input on promotional experiences

## Business Rules and Constraints

### Code Creation Guidelines

#### Naming Conventions
- **Code Format**: Consistent, memorable format (e.g., "ORG-YYYY-####")
- **Character Limits**: 3-50 characters, alphanumeric and hyphens only
- **Uniqueness**: Ensure codes don't conflict with existing ones
- **Branding**: Include organization identifier when appropriate

#### Value Limits
- **Percentage Discounts**: 5% to 50% off subscription costs
- **Fixed Discounts**: $5 to $500 off total order
- **Maximum Caps**: Prevent excessive discounts on high-value plans
- **Minimum Orders**: Require reasonable minimum purchase amounts

#### Usage Controls
- **Redemption Limits**: Set appropriate maximum usage counts
- **User Restrictions**: One per customer or unlimited use
- **Time Windows**: Reasonable start and end dates
- **Scope Limitations**: Appropriate targeting for code audience

## Success Metrics and KPIs

### Code Performance Metrics

#### Usage Metrics
- **Redemption Rate**: Percentage of distributed codes that get used
- **Unique Users**: Number of distinct customers using codes
- **Repeat Usage**: Customers using multiple codes
- **Geographic Distribution**: Where codes are being used

#### Financial Metrics
- **Revenue Impact**: Total revenue generated by promotional codes
- **Average Discount**: Typical discount amount per redemption
- **Customer Acquisition Cost**: Cost to acquire customers through codes
- **Lifetime Value**: Long-term value of promotional customers

#### Conversion Metrics
- **Conversion Rate**: Percentage of code users who complete purchase
- **Abandonment Rate**: Percentage of users who don't complete purchase
- **Upsell Rate**: Percentage of users who upgrade after using code
- **Retention Rate**: Long-term retention of promotional customers

## Related Documentation

- [Promo Codes Implementation](../14-payments-and-subscriptions/promo-codes-impl.md) - Technical implementation details
- [Subscriber Dashboard](subscriber-dashboard.md) - Main administrative interface
- [Billing Statements](billing-statements.md) - Financial reporting and invoicing
- [Member Management](member-management.md) - User and organization management
- [System Admin Billing Tools](../06-system-admin/sysadmin-billing-tools.md) - System-wide billing management


