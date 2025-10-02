# Non-Goals

## Purpose

This document explicitly defines what the MLC-LMS platform will NOT do in its current phase. These non-goals help maintain focus, prevent scope creep, and ensure resources are allocated to the most important features.

## Content and Curriculum Non-Goals

### Major Graphics Re-skins
- **What we won't do**: Complete visual redesign of existing HTML5 games
- **Why**: Current games are functional and meet learning objectives; visual refreshes are future work unless required for accessibility
- **Future consideration**: Incremental visual improvements may be considered in later phases

### New Game Development
- **What we won't do**: Create entirely new games from scratch
- **Why**: Focus is on platform features, not content creation
- **Future consideration**: Game creation tools may be added for advanced users

### Video Content Creation
- **What we won't do**: Produce instructional videos or multimedia content
- **Why**: Focus is on platform functionality and existing content integration
- **Future consideration**: Video hosting and management features may be added

### Flash-to-HTML5 Migration Tools
- **What we won't do**: Build automated tools to convert legacy Flash games to HTML5
- **Why**: Manual conversion ensures quality and pedagogical integrity; existing games are already functional
- **Current approach**: Games are individually converted with careful attention to learning objectives

### Legacy Method Alignment Tools
- **What we won't do**: Create automated tools to generate curriculum alignments for every teaching method
- **Why**: Manual curation ensures accuracy; focus is on core platform functionality
- **Current approach**: Key method alignments are maintained manually; see [sequence alignment documentation](../09-sequences-and-curriculum-tools/sequence-alignment-to-methods.md)

## Technical Integration Non-Goals

### Deep SIS Integration
- **What we won't do**: Full integration with Student Information Systems
- **Why**: Complex integration requirements vary by institution; provide import/export and email delivery first
- **Future consideration**: LTI or basic SIS sync may be added in Phase 3; defer full LMS/LTI integrations to a later phase

### Mobile Native Apps
- **What we won't do**: Develop iOS or Android native applications
- **Why**: Focus is responsive web and potential PWA; no iOS/Android native builds in this phase
- **Future consideration**: PWA features will provide app-like experience

### Legacy Browser Support
- **What we won't do**: Support browsers older than April 2020 or maintain Flash-based games
- **Why**: Modern browsers provide better security and performance; Flash is deprecated
- **Current approach**: Support recent versions of Chrome, Firefox, Edge, Opera, and Safari; see [browser compatibility matrix](../19-quality-and-operations/browser-device-matrix.md)

### Real-time Collaboration
- **What we won't do**: Live video lessons or real-time collaborative features
- **Why**: Focus is on asynchronous learning and practice
- **Future consideration**: Basic collaboration features may be added later

## Business Model Non-Goals

### Freemium Model
- **What we won't do**: Offer free tier with limited features
- **Why**: Focus on paid subscriptions ensures sustainable development
- **Current approach**: Prelude trial provides evaluation without commitment

### Marketplace for Content
- **What we won't do**: Allow third-party content creators to sell games
- **Why**: Quality control and curriculum alignment are critical; authoring and registry are internal to maintain pedagogy quality and safety
- **Future consideration**: Curated content partnerships may be explored

### White-label Solutions
- **What we won't do**: Provide white-label versions for other companies
- **Why**: Focus on building the best possible MLC experience
- **Future consideration**: API access may enable third-party integrations

### Crypto or Alternative Payments
- **What we won't do**: Support cryptocurrency or alternative payment methods
- **Why**: Use standard processors and school PO workflows first
- **Current approach**: Standard credit card processing and purchase order workflows

### Unbounded Teacher Access to Billing
- **What we won't do**: Allow teachers direct access to billing and payment management
- **Why**: Billing managed by Subscriber-Admin only
- **Current approach**: Teachers can view usage and assignments but cannot modify billing settings

### Free Tier or Freemium Model
- **What we won't do**: Offer permanently free access to games beyond the 30-day trial
- **Why**: Sustainable development requires paid subscriptions; free trial provides adequate evaluation
- **Current approach**: 30-day Prelude trial with 19 student slots; see [pricing structure](../14-payments-and-subscriptions/plans-and-pricing.md)

### Individual Student Subscriptions
- **What we won't do**: Allow students to subscribe directly without teacher/school oversight
- **Why**: Educational context requires teacher guidance and curriculum alignment
- **Current approach**: All subscriptions managed through teacher or institutional accounts

## User Experience Non-Goals

### Social Media Integration
- **What we won't do**: Direct integration with Facebook, Twitter, Instagram
- **Why**: Privacy and safety concerns for student users
- **Current approach**: Internal sharing and achievement features only

### Complex Social Features
- **What we won't do**: Advanced social features like friending, chat, or public profiles
- **Why**: Keep leaderboards class/school/global but defer friending, chat, or public profiles
- **Current approach**: Basic leaderboards and achievement sharing within controlled environments

### Advanced Gamification
- **What we won't do**: Complex RPG-style progression systems or virtual economies
- **Why**: Focus should remain on learning outcomes, not entertainment
- **Current approach**: Simple badges, points, and streaks support motivation; see [gamification system](../12-gamification/badges-system.md)

### Student-to-Student Direct Communication
- **What we won't do**: Enable direct messaging or chat between students
- **Why**: Safety and privacy concerns; focus on teacher-student communication
- **Current approach**: Students can message teachers; see [messaging system](../13-messaging-and-notifications/in-app-notifications.md)

### Customizable Themes
- **What we won't do**: Allow users to completely customize the visual appearance
- **Why**: Consistency and accessibility are more important than personalization
- **Current approach**: Accessibility options and basic preferences only

## Administrative Non-Goals

### Advanced Analytics Dashboard
- **What we won't do**: Complex data visualization or predictive analytics
- **Why**: Focus on core reporting that directly supports teaching
- **Current approach**: Clear, actionable reports for teachers and admins; see [reporting system](../15-analytics-and-reporting/teacher-reports-kpis.md)

### Automated Grade Calculation
- **What we won't do**: Automatic grade calculation or GPA systems
- **Why**: Music education requires qualitative assessment; focus on skill mastery tracking
- **Current approach**: Target score achievement and mastery indicators; see [assignment tracking](../10-assignments-engine/assignment-progress-tracking.md)

### Multi-tenant Architecture
- **What we won't do**: Separate data silos for different institutions
- **Why**: Role-based permissions provide sufficient data isolation
- **Current approach**: Hierarchical permissions within shared platform

### Advanced Workflow Automation
- **What we won't do**: Complex automated assignment generation or grading
- **Why**: Teacher oversight and customization are essential for effective learning
- **Current approach**: AI-assisted tools with teacher approval and editing

### Advanced Experimentation Platform
- **What we won't do**: Full experimentation suite with complex A/B testing infrastructure
- **Why**: Ship feature flags and basic A/B where needed, but defer a full experimentation suite
- **Current approach**: Basic feature flags and simple A/B testing for critical features

## Compliance and Legal Non-Goals

### FERPA Compliance (Phase 1)
- **What we won't do**: Full FERPA compliance implementation in initial release
- **Why**: Focus on core functionality first
- **Future consideration**: FERPA compliance will be added for school deployments; see [compliance documentation](../23-legal-and-compliance/coppa-ferpa-notes.md)

### COPPA Compliance (Phase 1)
- **What we won't do**: Full COPPA compliance implementation in initial release
- **Why**: Focus on core functionality first; current approach handles student data appropriately
- **Future consideration**: Enhanced COPPA compliance will be added for broader school deployments

### Advanced Audit Logging
- **What we won't do**: Comprehensive audit trails for every user action
- **Why**: Focus on essential logging for support and security
- **Current approach**: Key administrative actions and security events

### Data Export/Import Standards
- **What we won't do**: Full compliance with educational data standards (QTI, etc.)
- **Why**: Focus on practical CSV import/export for immediate needs
- **Future consideration**: Standard compliance may be added for enterprise customers

## Performance and Scale Non-Goals

### Global CDN Optimization
- **What we won't do**: Multi-region content delivery optimization
- **Why**: Focus on core functionality and single-region performance
- **Future consideration**: Global CDN will be added as user base grows

### Advanced Caching
- **What we won't do**: Complex caching strategies beyond basic CDN
- **Why**: Focus on application functionality over performance optimization
- **Current approach**: Standard web caching and CDN for static assets

### Microservices Architecture
- **What we won't do**: Break platform into microservices in initial release
- **Why**: Monolithic approach is simpler for initial development
- **Future consideration**: Microservices may be introduced as platform scales

## Platform-Specific Non-Goals

### Multi-Domain Architecture
- **What we won't do**: Maintain separate domains for different platform components (e.g., ComePlayMusic.com, TerryTreble.com)
- **Why**: Focus on unified platform experience; historical multi-domain approach adds complexity
- **Current approach**: Single platform with integrated features; see [platform architecture](../18-architecture/system-overview.md)

### Legacy Flash Game Support
- **What we won't do**: Maintain or support Flash-based games
- **Why**: Flash is deprecated and poses security risks; HTML5 provides better performance
- **Current approach**: All games converted to HTML5; see [game registry](../08-games-registry-and-authoring/games-registry-overview.md)

### Manual Student Page Generation
- **What we won't do**: Create tools to automatically generate method-specific student pages
- **Why**: Focus on automated sequences; manual pages are maintenance burden
- **Current approach**: Automated sequences with method alignment; see [sequence tools](../09-sequences-and-curriculum-tools/sequence-libraries.md)

## What This Means for Development

### Focus Areas
- Core learning functionality and user experience
- Essential administrative and reporting features
- Reliable, scalable platform infrastructure
- Clear, actionable user interfaces

### Success Metrics
- User adoption and engagement
- Learning outcomes and mastery rates
- Teacher efficiency and satisfaction
- Platform stability and performance

### Future Considerations
- Many non-goals may become goals in future phases
- User feedback and usage patterns will inform prioritization
- Technical debt and scalability needs will drive architectural decisions
- Market demands and competitive pressures may influence scope

## Review Process

This document should be reviewed quarterly to:
- Evaluate whether non-goals should become goals
- Assess if current goals are still appropriate
- Consider new non-goals based on platform evolution
- Ensure alignment with business objectives and user needs

