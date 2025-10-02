# Teacher Dashboard

## Purpose

The teacher dashboard provides educators with a comprehensive overview of their students' progress, quick access to essential tools, and alerts for students who need attention. It serves as the central command center for managing assignments, tracking progress, and communicating with students and parents.

### Focused value
- A single-screen triage and action hub where teachers can: identify who needs help now, take the next best action in one click, and confirm impact via lightweight KPIs. Other teacher pages go deeper (builder, reports, messaging); the dashboard optimizes for speed and prioritization.

## Dashboard Layout

### Header Section
- **Teacher Name**: Personalized greeting with role indicator
- **Class Selector**: Switch between different classes (if applicable)
- **Quick Actions**: Most common tasks (Create Assignment, View Reports, etc.)
- **Notifications**: Alerts for students needing attention

### Main Content Areas

#### Class Overview
- **Student Roster**: Grid or list view of all students
- **Progress Indicators**: Visual progress bars for each student
- **Status Alerts**: Students who need attention or assistance
- **Recent Activity**: Latest student submissions and progress

#### Quick Stats
- **Total Students**: Number of active students
- **Assignments Active**: Currently assigned activities
- **Completion Rate**: Overall class completion percentage
- **Average Score**: Class performance metrics

#### Prioritization and layout rules
- Place "Students Needing Attention" and "Overdue Assignments" above the fold, left-aligned on desktop and first in mobile stacks.
- Show at most 5 alerts by default with a "View all" link to detailed lists.
- Quick Actions (Create Assignment, Send Message, View Reports) are sticky in the header for rapid follow-up.

#### Alerts and Notifications
- **Students Struggling**: Students below performance thresholds
- **Overdue Assignments**: Students with late submissions
- **New Submissions**: Recently completed work to review
- **System Notifications**: Platform updates and announcements

## Student Roster

### Student Cards
- **Student Name**: Clear identification with avatar
- **Progress Bar**: Visual progress through current assignment
- **Status Indicator**: Active, Struggling, Completed, Overdue
- **Last Activity**: When student last accessed the platform
- **Quick Actions**: View progress, send message, adjust assignment

### Student Status Indicators
- **Green**: On track and performing well
- **Yellow**: Some concerns, monitor closely
- **Red**: Struggling, needs immediate attention
- **Gray**: Inactive or not started

### Bulk Actions
- **Select All**: Choose multiple students for bulk operations
- **Mass Assignment**: Assign same activity to multiple students
- **Send Message**: Communicate with selected students
- **Export Data**: Download progress reports

## Assignment Management

### Active Assignments
- **Assignment List**: Currently assigned activities
- **Student Progress**: Individual completion status
- **Due Dates**: When assignments are due
- **Modification Options**: Edit, extend, or cancel assignments

### Quick Assignment Creation
- **From Sequence**: Create assignment from pre-built sequences
- **Custom Assignment**: Build custom assignment from scratch
- **Duplicate**: Copy existing assignment for new students
- **Template Library**: Use saved assignment templates

### Assignment Status
- **Draft**: Not yet assigned to students
- **Active**: Currently assigned and in progress
- **Completed**: All students have finished
- **Archived**: Historical assignments for reference

## Progress Tracking

### Class Performance
- **Overall Progress**: Class-wide completion rates
- **Mastery Levels**: Number of students at each mastery level
- **Time on Task**: Average time spent on assignments
- **Score Distribution**: Range of scores across the class

### Individual Student Details
- **Detailed Progress**: Game-by-game completion status
- **Score History**: Performance over time
- **Learning Patterns**: When and how student practices
- **Areas of Strength**: Concepts student excels at
- **Areas for Improvement**: Concepts needing more practice

### Comparative Analysis
- **Class Averages**: How individual students compare to class
- **Progress Trends**: Improvement or decline over time
- **Peer Comparison**: Anonymous comparison with classmates
- **Benchmark Data**: Performance against grade-level standards

## Role and permissions

- The dashboard is visible to users with the Teacher role; surface area and actions reflect permission scopes. See: [roles matrix](../02-roles-and-permissions/roles-matrix.md) and [permissions table](../02-roles-and-permissions/permissions-table.md).
- Multi-teacher subscriber contexts inherit from member/subscriber admin where applicable; teacher-only contexts restrict to owned classes.
- Impersonation and support access must display a banner and disable destructive actions per [impersonation policy](../02-roles-and-permissions/impersonation-and-support-access.md).

## Navigation to adjacent teacher pages

- Build or modify work: [Assignment Builder](./assignment-builder.md)
- Analyze progress: [Teacher Reports](./teacher-reports.md)
- Communicate: [Teacher Messaging](./teacher-messaging.md) and [In‑app notifications](../13-messaging-and-notifications/in-app-notifications.md)

## Telemetry and events

Instrument the dashboard with events following the [event model](../15-analytics-and-reporting/event-model.md). At minimum:
- `dashboard_viewed` with context: class_id, student_count, open_alert_count
- `dashboard_quick_action_clicked` with action_type (create_assignment | view_reports | send_message)
- `dashboard_student_row_clicked` with student_id, alert_types
- `dashboard_filter_changed` with filter_name, filter_value

Use these to populate teacher KPIs in [learning analytics](../15-analytics-and-reporting/learning-analytics.md).

## Data sources and calculations

- Completion rate and average score definitions align with [assignment progress tracking](../10-assignments-engine/assignment-progress-tracking.md) and [teacher reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md).
- Alert thresholds (e.g., "Struggling") should reuse classification from [retention review logic](../10-assignments-engine/retention-review-logic.md).
- Notifications mirror the delivery rules in [communication framework](../13-messaging-and-notifications/communication-framework.md); do not re-implement.

## Legacy parity notes (for migration)

Based on historic specs in `00-ORIG-CONTEXT`:
- Maintain quick access patterns observed in "Teacher Resource Center" (language choice, manage student users, get score reports) while mapping to modern equivalents: profile/settings, member/student management, and reports.
- Ensure a visible route to Student Resource Center analogs (student view) only where permissions allow; otherwise hide.
- Preserve sponsor/program awareness via announcements rather than dashboard chrome (see legacy entries in "Master Tables – Dashboards").

## Communication Tools

### Student Messages
- **Inbox**: Messages from students
- **Compose**: Send messages to individual or multiple students
- **Templates**: Pre-written messages for common situations
- **Message History**: Archive of all communications

### Parent Communication
- **Progress Reports**: Automated progress summaries
- **Parent Messages**: Direct communication with parents
- **Achievement Sharing**: Share student successes
- **Concern Alerts**: Notify parents of issues

### Announcements
- **Class Announcements**: Broadcast messages to all students
- **Assignment Reminders**: Automated reminders for due dates
- **System Updates**: Platform changes and improvements
- **Celebration Messages**: Recognize class achievements

## Reporting and Analytics

### Quick Reports
- **Student Summary**: Individual student performance overview
- **Class Progress**: Class-wide performance metrics
- **Assignment Analysis**: How well assignments are working
- **Time Tracking**: How much time students spend practicing

### Detailed Analytics
- **Learning Outcomes**: Mastery rates by concept
- **Engagement Metrics**: Student participation and activity
- **Performance Trends**: Improvement over time
- **Comparative Data**: How class compares to other classes

### Export Options
- **CSV Export**: Download data for external analysis
- **PDF Reports**: Professional reports for administrators
- **Parent Reports**: Shareable progress summaries
- **Administrative Reports**: Data for school administrators

## Settings and Preferences

### Class Configuration
- **Class Settings**: Name, description, and preferences
- **Student Management**: Add, remove, or modify student accounts
- **Assignment Defaults**: Default settings for new assignments
- **Notification Preferences**: What alerts to receive

### Personal Settings
- **Profile Management**: Update personal information
- **Password Security**: Change password and security settings
- **Accessibility Options**: Customize interface for needs
- **Integration Settings**: Connect with external tools

### System Preferences
- **Display Options**: How information is presented
- **Data Retention**: How long to keep historical data
- **Backup Settings**: Data backup and recovery options
- **Privacy Settings**: Control data sharing and visibility

## Responsive Design

### Mobile Layout
- **Stacked Cards**: Vertical layout for small screens
- **Touch-friendly**: Large buttons and touch targets
- **Swipe Actions**: Natural mobile interaction patterns
- **Bottom Navigation**: Quick access to main functions

### Tablet Layout
- **Grid Layout**: Multiple cards visible at once
- **Sidebar Navigation**: Persistent navigation panel
- **Split View**: Content and navigation side-by-side
- **Touch and Mouse**: Support both input methods

### Desktop Layout
- **Multi-column**: Efficient use of screen space
- **Hover States**: Rich hover interactions
- **Keyboard Shortcuts**: Power user features
- **Detailed Views**: More information visible at once

## Accessibility Features

### Visual Accessibility
- **High Contrast**: Support for high contrast mode
- **Large Text**: Scalable text sizes
- **Color Independence**: Information not conveyed by color alone
- **Focus Indicators**: Clear focus states for keyboard navigation

### Cognitive Accessibility
- **Clear Hierarchy**: Obvious visual hierarchy
- **Consistent Patterns**: Predictable interaction patterns
- **Error Prevention**: Clear feedback and confirmation
- **Help Integration**: Contextual help and guidance

### Motor Accessibility
- **Large Touch Targets**: Minimum 44px touch targets
- **Keyboard Navigation**: Full keyboard accessibility
- **Voice Control**: Support for voice commands
- **Switch Control**: Support for assistive devices

## Performance Considerations

### Loading States
- **Skeleton Screens**: Show content structure while loading
- **Progressive Loading**: Load most important content first
- **Caching**: Cache frequently accessed data
- **Offline Support**: Basic functionality without internet

### Optimization
- **Image Optimization**: Compressed thumbnails and assets
- **Lazy Loading**: Load content as needed
- **CDN**: Fast content delivery
- **Minimal JavaScript**: Efficient client-side code

## Future Enhancements

### AI Integration
- **Student Insights**: AI-powered analysis of student performance
- **Assignment Recommendations**: Suggest optimal assignments
- **Intervention Alerts**: Early warning for struggling students
- **Personalized Learning**: Adapt content to individual needs

### Advanced Analytics
- **Predictive Analytics**: Forecast student performance
- **Learning Patterns**: Identify effective teaching strategies
- **Intervention Strategies**: Suggest interventions for struggling students
- **Success Metrics**: Comprehensive success tracking

### Collaboration Features
- **Teacher Collaboration**: Share assignments and strategies
- **Professional Development**: Access to training resources
- **Community Features**: Connect with other educators
- **Best Practices**: Share and discover effective teaching methods

