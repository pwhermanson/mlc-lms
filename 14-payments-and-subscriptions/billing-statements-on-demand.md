# Billing statements on demand

## Purpose

This document defines the on-demand billing statement functionality for the MLC-LMS platform. It covers the ability for subscribers to generate custom billing statements for any date range, view detailed transaction history, and access downloadable PDF statements for accounting and record-keeping purposes.

The on-demand statement feature enables subscribers to maintain accurate financial records, reconcile billing periods, and provide documentation for internal accounting or audit purposes without waiting for scheduled statement generation.

## Scope

### Included
- Custom date range statement generation
- PDF and CSV export formats
- Email delivery to multiple recipients
- Statement history and archive management
- Integration with existing billing data
- School-specific statement formatting
- PO number inclusion for institutional customers
- Detailed line item breakdowns

### Excluded
- Automated scheduled statement generation (see [billing-plans-and-seats.md](billing-plans-and-seats.md))
- Payment processing details (see [billing-provider-integration.md](billing-provider-integration.md))
- Invoice generation and management (see [billing-plans-and-seats.md](billing-plans-and-seats.md))
- PO management (see [po-and-invoicing-for-schools.md](po-and-invoicing-for-schools.md))

## Statement Generation

### Date Range Selection
- **Custom Ranges**: Select any start and end date for statement period
- **Preset Options**: Quick selection for common periods (current month, last 30 days, current quarter, etc.)
- **Academic Periods**: School-specific periods aligned with academic calendar
- **Fiscal Year**: Support for different fiscal year configurations
- **Maximum Range**: 24-month maximum range for performance and data retention
- **Minimum Range**: 1-day minimum range for specific transaction review

### Statement Content
- **Header Information**: Subscriber name, address, contact information
- **Statement Period**: Clear start and end date range
- **Account Summary**: Current plan, seat count, billing cycle
- **Transaction History**: Detailed list of all charges, credits, and adjustments
- **Line Item Details**: Breakdown by seat additions, plan changes, prorations
- **Payment History**: All payments received during the period
- **Outstanding Balance**: Current amount due if applicable
- **PO Information**: Purchase order numbers for school customers
- **Tax Information**: Tax calculations and exemptions if applicable

### Export Formats

#### PDF Format
- **Professional Layout**: Clean, professional formatting suitable for accounting
- **School Branding**: Include school logo and letterhead for institutional customers
- **Print-Ready**: Optimized for both screen viewing and printing
- **Page Breaks**: Automatic page breaks for multi-page statements
- **Watermarking**: Optional watermark for draft or final status
- **Digital Signature**: Space for digital signatures if required

#### CSV Format
- **Data Export**: Raw data for import into accounting systems
- **Column Headers**: Clear column headers for easy data mapping
- **Date Formats**: Standardized date formats for system compatibility
- **Currency Formatting**: Proper currency formatting for financial systems
- **UTF-8 Encoding**: Support for international characters and symbols

### Email Delivery
- **Multiple Recipients**: Send to up to 10 email addresses per statement
- **Custom Subject Lines**: Personalized subject lines with statement period
- **Email Templates**: Professional email templates with statement summary
- **Delivery Confirmation**: Track email delivery and open rates
- **Attachment Security**: Password-protected PDFs for sensitive financial data
- **Delivery History**: Archive of all email deliveries for audit purposes

## Statement Management

### Statement History
- **Archive System**: Store all generated statements for 7 years
- **Search Functionality**: Search statements by date range, amount, or keywords
- **Quick Access**: Recent statements easily accessible from dashboard
- **Bulk Download**: Download multiple statements as ZIP archive
- **Statement Index**: Chronological index of all generated statements

### Access Controls
- **Role-Based Access**: Different access levels for different user types
- **Download Permissions**: Control who can download statements
- **Email Permissions**: Control who can send statements via email
- **Audit Logging**: Track all statement access and generation
- **IP Restrictions**: Optional IP-based access restrictions

### Data Security
- **Encryption**: All statements encrypted at rest and in transit
- **Access Logging**: Complete audit trail of statement access
- **Data Retention**: Automatic purging of old statements per policy
- **Backup Systems**: Regular backup of statement data
- **Compliance**: Meet financial data protection requirements

## School-Specific Features

### Institutional Formatting
- **School Letterhead**: Custom letterhead with school branding
- **Department Breakdown**: Separate statements by department or program
- **Budget Codes**: Include school budget codes and classifications
- **Approval Workflows**: Multi-level approval for statement generation
- **District Consolidation**: Combine statements from multiple schools

### PO Integration
- **PO References**: Include purchase order numbers on all statements
- **PO Validation**: Verify PO numbers against school systems
- **Approval Status**: Show PO approval status and dates
- **Budget Tracking**: Track spending against budget allocations
- **Compliance Reporting**: Generate compliance reports for school audits

## User Interface

### Statement Generation Interface
- **Date Picker**: Intuitive date range selection tool
- **Preview Mode**: Preview statement before generation
- **Format Selection**: Choose PDF or CSV export format
- **Recipient Management**: Add/remove email recipients
- **Customization Options**: Select which data to include
- **Generation Status**: Real-time status of statement generation

### Statement Dashboard
- **Recent Statements**: Quick access to recently generated statements
- **Generation Queue**: View pending statement generations
- **Usage Statistics**: Track statement generation frequency
- **Storage Usage**: Monitor statement storage usage
- **Quick Actions**: Common actions like regenerate or resend

### Mobile Responsiveness
- **Mobile Viewing**: View statements on mobile devices
- **Touch-Friendly**: Touch-optimized interface for tablets
- **Offline Access**: Download statements for offline viewing
- **Push Notifications**: Notify when statements are ready
- **Mobile Sharing**: Easy sharing of statements via mobile

## Technical Implementation

### Data Sources
- **Billing Database**: Primary source for transaction data
- **User Database**: Subscriber and user information
- **Payment Database**: Payment and transaction history
- **Plan Database**: Subscription plan and pricing information
- **Audit Database**: Change history and audit logs

### Performance Considerations
- **Caching**: Cache frequently accessed statement data
- **Background Processing**: Generate large statements in background
- **Queue Management**: Manage statement generation queue
- **Resource Limits**: Prevent system overload during peak usage
- **Optimization**: Optimize queries for large date ranges

### Integration Points
- **Billing System**: Real-time integration with billing data
- **Email Service**: Integration with email delivery service
- **Storage Service**: Integration with document storage service
- **Notification Service**: Integration with notification system
- **Audit Service**: Integration with audit logging system

## Telemetry

### Statement Events
- `statement_requested` - New statement generation requested
- `statement_generated` - Statement successfully generated
- `statement_downloaded` - Statement downloaded by user
- `statement_emailed` - Statement sent via email
- `statement_failed` - Statement generation failed

### Performance Metrics
- `generation_time` - Time taken to generate statement
- `statement_size` - Size of generated statement
- `download_count` - Number of times statement downloaded
- `email_delivery_time` - Time taken to deliver email
- `error_rate` - Rate of statement generation errors

## Permissions

### Subscriber Admin
- Generate statements for any date range
- Download statements in PDF or CSV format
- Email statements to multiple recipients
- View statement generation history
- Manage statement recipients

### Teacher Admin
- View statements for their organization (if enabled)
- Download statements (if enabled by org policy)
- Cannot email statements to external recipients
- Limited to read-only access

### System Admin
- Generate statements for any subscriber
- Access all statement data and history
- Override statement generation limits
- Manage statement storage and retention
- View system-wide statement analytics

## Dependencies

- [billing-plans-and-seats.md](billing-plans-and-seats.md) - Core billing data and transaction history
- [billing-provider-integration.md](billing-provider-integration.md) - Payment data and transaction details
- [po-and-invoicing-for-schools.md](po-and-invoicing-for-schools.md) - PO data for school customers
- [05-admin-subscriber-experience/billing-statements.md](../05-admin-subscriber-experience/billing-statements.md) - Admin interface for statement management
- [18-architecture/data-model-erd.md](../18-architecture/data-model-erd.md) - Database schema and relationships

## Open Questions

1. **Statement Retention**: What is the required retention period for generated statements?
2. **File Size Limits**: What are the maximum file sizes for PDF and CSV exports?
3. **Concurrent Generation**: How many statements can be generated simultaneously?
4. **International Support**: What additional requirements exist for international customers?
5. **Custom Fields**: What custom fields should be available for statement customization?
6. **API Access**: Should there be API endpoints for programmatic statement generation?
7. **Mobile App**: Should statement generation be available in mobile applications?

## Future Enhancements

### Advanced Features
- **Automated Scheduling**: Schedule regular statement generation
- **Custom Templates**: User-defined statement templates
- **Advanced Filtering**: Filter statements by transaction type or amount
- **Comparison Reports**: Compare statements across different periods
- **Trend Analysis**: Analyze spending trends over time

### Integration Improvements
- **Accounting Software**: Direct integration with QuickBooks, Xero, etc.
- **ERP Systems**: Integration with enterprise resource planning systems
- **Cloud Storage**: Direct upload to Google Drive, Dropbox, etc.
- **API Enhancements**: RESTful API for third-party integrations
- **Webhook Support**: Real-time notifications for statement events
