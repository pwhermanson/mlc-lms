# Student Messages

## Purpose

The student messaging system enables secure, age-appropriate communication between students and their teachers within the MLC-LMS platform. This system supports the pedagogical goal of maintaining teacher-student relationships while ensuring student safety and privacy compliance. The messaging system facilitates assignment clarification, progress discussions, encouragement, and learning support while maintaining appropriate boundaries and audit trails.

## Scope

### Included
- **Student-to-Teacher Communication**: Direct messaging between students and their assigned teachers
- **Teacher-to-Student Messaging**: Teachers can send messages to individual students or groups
- **Message Threading**: Organized conversation threads with proper context
- **File Attachments**: Support for sharing learning materials, audio recordings, and images
- **Message Status Tracking**: Read receipts, delivery confirmation, and response tracking
- **Privacy Controls**: Age-appropriate content filtering and monitoring
- **Audit Logging**: Complete message history for compliance and safety
- **Notification Integration**: Real-time notifications through the [Communication Framework](../13-messaging-and-notifications/communication-framework.md)

### Excluded
- **Student-to-Student Messaging**: Direct peer-to-peer communication (not included for safety)
- **Parent Communication**: Parent messaging handled through separate [Admin Subscriber Experience](../05-admin-subscriber-experience/) system
- **Public Forums**: Community discussions not included in student messaging
- **Video/Audio Calls**: Real-time communication not included in this spec
- **External Messaging**: Integration with external messaging platforms

## Data Model

### Core Entities

#### Message
```typescript
interface Message {
  id: string;                    // Unique message identifier
  threadId: string;              // Conversation thread identifier
  senderId: string;              // User ID of sender
  senderRole: UserRole;          // Role of sender (Student, Teacher, etc.)
  recipientId: string;           // User ID of primary recipient
  recipientRole: UserRole;       // Role of recipient
  subject: string;               // Message subject line
  content: string;               // Message body content
  contentType: 'text' | 'html';  // Content format
  attachments: Attachment[];     // Associated file attachments
  status: MessageStatus;         // Current message status
  priority: MessagePriority;     // Message priority level
  createdAt: Date;               // Message creation timestamp
  updatedAt: Date;               // Last modification timestamp
  readAt?: Date;                 // When message was read
  repliedAt?: Date;              // When message was replied to
  archivedAt?: Date;             // When message was archived
  metadata: MessageMetadata;     // Additional message data
}
```

#### MessageThread
```typescript
interface MessageThread {
  id: string;                    // Unique thread identifier
  participants: ThreadParticipant[]; // Thread participants
  subject: string;               // Thread subject
  lastMessageId: string;         // ID of most recent message
  lastActivityAt: Date;          // Last activity timestamp
  isActive: boolean;             // Whether thread is active
  isArchived: boolean;           // Whether thread is archived
  createdAt: Date;               // Thread creation timestamp
  updatedAt: Date;               // Last modification timestamp
  metadata: ThreadMetadata;      // Additional thread data
}
```

#### ThreadParticipant
```typescript
interface ThreadParticipant {
  userId: string;                // User identifier
  role: UserRole;                // User role
  joinedAt: Date;                // When user joined thread
  lastReadAt?: Date;             // Last time user read thread
  isActive: boolean;             // Whether user is active in thread
  permissions: ParticipantPermissions; // Thread-specific permissions
}
```

#### Attachment
```typescript
interface Attachment {
  id: string;                    // Unique attachment identifier
  messageId: string;             // Associated message ID
  filename: string;              // Original filename
  displayName: string;           // Display name for UI
  mimeType: string;              // MIME type
  size: number;                  // File size in bytes
  url: string;                   // Secure file URL
  thumbnailUrl?: string;         // Thumbnail URL for images
  uploadedAt: Date;              // Upload timestamp
  metadata: AttachmentMetadata;  // Additional file data
}
```

### Enums and Types

#### MessageStatus
```typescript
enum MessageStatus {
  DRAFT = 'draft',               // Message being composed
  SENT = 'sent',                 // Message sent successfully
  DELIVERED = 'delivered',       // Message delivered to recipient
  READ = 'read',                 // Message read by recipient
  REPLIED = 'replied',           // Message has been replied to
  ARCHIVED = 'archived',         // Message archived
  DELETED = 'deleted'            // Message deleted
}
```

#### MessagePriority
```typescript
enum MessagePriority {
  LOW = 'low',                   // Standard priority
  NORMAL = 'normal',             // Default priority
  HIGH = 'high',                 // Important message
  URGENT = 'urgent'              // Urgent message (teacher only)
}
```

#### MessageMetadata
```typescript
interface MessageMetadata {
  assignmentId?: string;         // Related assignment ID
  gameId?: string;              // Related game ID
  sequenceId?: string;          // Related sequence ID
  tags: string[];               // Message tags for organization
  isEncrypted: boolean;         // Whether message is encrypted
  retentionPolicy: string;      // Data retention policy
  complianceFlags: string[];    // Compliance-related flags
}
```

## Behavior and Rules

### Message Creation and Sending

#### Student Message Rules
- **Recipients**: Students can only message their assigned teachers
- **Content Validation**: All messages filtered for inappropriate content
- **Attachment Limits**: Maximum 3 attachments per message, 10MB total size
- **Character Limits**: Maximum 2000 characters per message
- **Rate Limiting**: Maximum 10 messages per hour per student
- **Draft Saving**: Messages auto-saved as drafts every 30 seconds

#### Teacher Message Rules
- **Recipients**: Teachers can message their assigned students individually or in groups
- **Broadcast Messages**: Teachers can send messages to entire classes
- **Priority Levels**: Teachers can set message priority (students cannot)
- **Attachment Limits**: Maximum 10 attachments per message, 50MB total size
- **Character Limits**: Maximum 5000 characters per message
- **No Rate Limiting**: Teachers have no message rate limits

### Message Threading

#### Thread Creation
- **Automatic Threading**: Messages between same participants automatically grouped
- **Subject Inheritance**: Thread subject inherited from first message
- **Participant Management**: New participants added to existing threads when appropriate
- **Thread Archiving**: Inactive threads archived after 90 days

#### Thread Participation
- **Role-based Access**: Only assigned teachers and students can participate
- **Thread Visibility**: Students see only threads they participate in
- **Teacher Oversight**: Teachers can see all threads involving their students
- **Admin Access**: System admins can access all threads for support

### Content Moderation and Safety

#### Content Filtering
- **Automated Filtering**: AI-powered content analysis for inappropriate material
- **Keyword Blocking**: Blocked list of inappropriate keywords and phrases
- **Image Analysis**: Automated analysis of uploaded images for inappropriate content
- **Audio Content**: Basic audio content analysis for inappropriate material

#### Safety Measures
- **Teacher Monitoring**: All student messages visible to assigned teachers
- **Admin Oversight**: System admins can review flagged content
- **Reporting System**: Students can report inappropriate messages
- **Escalation Process**: Automatic escalation of concerning content to administrators

### Message Status and Lifecycle

#### Status Transitions
```
DRAFT → SENT → DELIVERED → READ → REPLIED
  ↓       ↓        ↓        ↓
ARCHIVED ← ARCHIVED ← ARCHIVED ← ARCHIVED
  ↓
DELETED
```

#### Lifecycle Rules
- **Draft Expiration**: Drafts deleted after 7 days of inactivity
- **Message Retention**: Messages retained for 2 years for compliance
- **Archive Policy**: Read messages archived after 30 days of inactivity
- **Deletion Policy**: Messages can be deleted by sender within 24 hours

## UX Requirements

### Message Interface Layout

#### Desktop Layout
- **Three-Panel Design**: Thread list, message list, message detail
- **Thread Panel**: Left sidebar with conversation threads
- **Message Panel**: Center panel with message list and composition
- **Detail Panel**: Right panel with message details and attachments
- **Responsive Design**: Collapsible panels for smaller screens

#### Mobile Layout
- **Two-Panel Design**: Thread list and message detail (stacked)
- **Touch-Friendly**: Large touch targets and swipe gestures
- **Bottom Navigation**: Quick access to compose and settings
- **Keyboard Optimization**: Optimized for mobile keyboard input

### Message Composition

#### Compose Interface
- **Rich Text Editor**: Basic formatting (bold, italic, lists)
- **Attachment Upload**: Drag-and-drop file upload with progress
- **Draft Auto-Save**: Automatic draft saving with visual indicator
- **Character Counter**: Real-time character count with limits
- **Send Options**: Send now, schedule send, save as draft

#### Message Display
- **Message Bubbles**: Chat-style message display with sender identification
- **Timestamp Display**: Relative timestamps (e.g., "2 hours ago")
- **Read Receipts**: Visual indicators for message status
- **Attachment Preview**: Thumbnail previews for images and documents
- **Message Actions**: Reply, forward, archive, delete options

### Accessibility Features

#### Visual Accessibility
- **High Contrast Mode**: Support for high contrast themes
- **Font Scaling**: Support for system font scaling
- **Color Independence**: Information not conveyed by color alone
- **Focus Indicators**: Clear focus states for keyboard navigation

#### Motor Accessibility
- **Keyboard Navigation**: Full keyboard accessibility
- **Voice Control**: Support for voice commands
- **Switch Control**: Support for assistive devices
- **Large Touch Targets**: Minimum 44px touch targets

#### Cognitive Accessibility
- **Clear Hierarchy**: Obvious visual hierarchy and organization
- **Consistent Patterns**: Predictable interaction patterns
- **Error Prevention**: Clear feedback and confirmation dialogs
- **Help Integration**: Contextual help and guidance

## Telemetry

### Message Events
```typescript
interface MessageEvent {
  eventType: 'message_sent' | 'message_received' | 'message_read' | 'message_replied' | 'message_archived' | 'message_deleted';
  messageId: string;
  threadId: string;
  senderId: string;
  recipientId: string;
  timestamp: Date;
  metadata: {
    messageLength: number;
    attachmentCount: number;
    responseTime?: number; // Time between sent and read
    deviceType: string;
    userAgent: string;
  };
}
```

### Thread Events
```typescript
interface ThreadEvent {
  eventType: 'thread_created' | 'thread_archived' | 'thread_deleted' | 'participant_added' | 'participant_removed';
  threadId: string;
  participantIds: string[];
  timestamp: Date;
  metadata: {
    threadAge: number; // Days since thread creation
    messageCount: number;
    participantCount: number;
  };
}
```

### Performance Metrics
- **Message Delivery Time**: Time from send to delivery
- **Read Response Time**: Time from delivery to read
- **Compose Time**: Time spent composing messages
- **Attachment Upload Time**: Time to upload attachments
- **Search Performance**: Time to search messages and threads

## Permissions

### Student Permissions
- **Send Messages**: Can send messages to assigned teachers
- **Read Messages**: Can read messages in their threads
- **Delete Own Messages**: Can delete their own messages within 24 hours
- **Upload Attachments**: Can upload attachments with size limits
- **Archive Threads**: Can archive their own threads
- **Report Messages**: Can report inappropriate messages

### Teacher Permissions
- **Send Messages**: Can send messages to assigned students
- **Read All Messages**: Can read all messages involving their students
- **Delete Messages**: Can delete any message in their threads
- **Manage Threads**: Can create, archive, and delete threads
- **Moderate Content**: Can flag and moderate student messages
- **Bulk Operations**: Can send messages to multiple students

### Teacher-Admin Permissions
- **All Teacher Permissions**: Inherits all teacher capabilities
- **Cross-Teacher Access**: Can access messages across teachers in their scope
- **Bulk Management**: Can manage threads across their scope
- **Reporting Access**: Can access message reports and analytics

### Subscriber-Admin Permissions
- **All Teacher-Admin Permissions**: Inherits all teacher-admin capabilities
- **Organization-wide Access**: Can access all messages in their organization
- **Compliance Reporting**: Can generate compliance reports
- **Data Export**: Can export message data for compliance

### System Admin Permissions
- **Global Access**: Can access all messages across all organizations
- **Content Moderation**: Can moderate and review all content
- **System Configuration**: Can configure messaging system settings
- **Audit Access**: Can access complete audit trails

## Dependencies

### Core Dependencies
- **[Communication Framework](../13-messaging-and-notifications/communication-framework.md)**: Notification delivery and preferences
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: User role definitions and permissions
- **[Student Dashboard](../03-student-experience/student-dashboard.md)**: Message access and display
- **[Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md)**: Teacher message management

### Technical Dependencies
- **File Storage Service**: Secure file storage for attachments
- **Content Moderation API**: AI-powered content filtering
- **Notification Service**: Real-time notification delivery
- **Search Service**: Message search and indexing
- **Audit Service**: Compliance and audit logging

### External Dependencies
- **Email Service**: Email notifications for message alerts
- **Push Notification Service**: Mobile push notifications
- **CDN**: Content delivery for attachments and media
- **Encryption Service**: Message content encryption

## Integration Points

### Assignment System Integration
- **Assignment Context**: Messages can reference specific assignments
- **Progress Discussion**: Teachers can discuss student progress
- **Clarification Requests**: Students can ask for assignment clarification
- **Feedback Exchange**: Two-way feedback on assignments

### Notification System Integration
- **Real-time Alerts**: Instant notifications for new messages
- **Email Summaries**: Daily/weekly message summaries
- **Push Notifications**: Mobile notifications for urgent messages
- **In-app Notifications**: Dashboard notification indicators

### Analytics Integration
- **Message Analytics**: Track communication patterns and engagement
- **Response Time Metrics**: Measure teacher response times
- **Content Analysis**: Analyze message content for insights
- **Usage Patterns**: Track messaging usage across the platform

## Open Questions

### Content Moderation
- **AI Filtering Accuracy**: What is the acceptable false positive rate for content filtering?
- **Human Review Process**: What is the escalation process for flagged content?
- **Appeal Process**: How do users appeal content moderation decisions?

### Privacy and Compliance
- **Data Retention**: What are the specific data retention requirements for different regions?
- **Parental Access**: Should parents have access to their child's messages?
- **Audit Requirements**: What level of audit logging is required for compliance?

### Technical Implementation
- **Message Encryption**: Should messages be encrypted at rest and in transit?
- **Search Functionality**: What search capabilities should be available?
- **Offline Support**: Should there be offline message composition and reading?

### User Experience
- **Message Templates**: Should there be pre-defined message templates for common scenarios?
- **Auto-responses**: Should there be automated responses for common questions?
- **Message Scheduling**: Should teachers be able to schedule messages for later delivery?

### Performance and Scalability
- **Message Limits**: What are the appropriate limits for message storage and retrieval?
- **Concurrent Users**: How many users can be actively messaging simultaneously?
- **Attachment Handling**: What is the optimal approach for handling large attachments?

