Here’s the complete `00-foundations/docs-index.md` file, ready to paste into Cursor.

```markdown
# Docs Index

Welcome. This is the entry point for the MusicLearningCommunity LMS specifications. Start with Foundations, then drill into the area you are working on.

## Getting started
- [Executive Summary](EXECUTIVE-SUMMARY.md)
- [Product Vision](00-foundations/product-vision.md)
- [Pedagogy Principles](00-foundations/pedagogy-principles.md)
- [Information Architecture](00-foundations/information-architecture.md)

## Quick Reference
- **System Overview**: MusicLearningCommunity.com is a subscription-based music education platform with hundreds of interactive games for music theory, aural skills, and sight-reading
- **Target Users**: Private music teachers, schools (general music, band, choir), homeschool families, and students ages 3-87
- **Core Philosophy**: "Lifetime Musician" - developing students who can read, write, think and create in the universal language of music
- **Key Features**: Game-based learning with Learn/Play/Quiz/Challenge/Review stages, MIDI keyboard support, automated sequencing, real-time scoring

---

## Foundations
- [Glossary](00-foundations/glossary.md) | Owner: PM | Status: Stable
- [Personas](00-foundations/personas.md) | Owner: PM | Status: Stable
- [Non-goals](00-foundations/non-goals.md) | Owner: PM | Status: Stable
- [Company History](00-foundations/company-history.md) | Owner: PM | Status: Stable
- [Founder Biography](00-foundations/founder-biography.md) | Owner: PM | Status: Stable
- [Legacy Systems](00-foundations/legacy-systems.md) | Owner: PM | Status: Stable

## UX and Components
- [Design Language](01-ux-design-system/design-language.md) | Owner: Design | Status: Draft
- [Tokens](01-ux-design-system/tokens.md) | Owner: Design | Status: Draft
- [Components Overview](01-ux-design-system/components-overview.md) | Owner: Design | Status: Stable
- [Iconography](01-ux-design-system/iconography.md) | Owner: Design | Status: Draft
- [Responsive Breakpoints](01-ux-design-system/responsive-breakpoints.md) | Owner: Design | Status: Draft
- [Screen Text and Copy](01-ux-design-system/screen-text-and-copy.md) | Owner: Design | Status: Draft
- [Localization and Internationalization](01-ux-design-system/localization-internationalization.md) | Owner: PM | Status: Draft
- [Accessibility Standards](01-ux-design-system/a11y-standards.md) | Owner: Design | Status: Stable

## Roles and Access
- [Roles Matrix](02-roles-and-permissions/roles-matrix.md) | Owner: PM | Status: Stable
- [Permissions Table](02-roles-and-permissions/permissions-table.md) | Owner: PM | Status: Draft
- [Session and Auth States](02-roles-and-permissions/session-and-auth-states.md) | Owner: Eng | Status: Draft
- [Impersonation and Support Access](02-roles-and-permissions/impersonation-and-support-access.md) | Owner: Eng | Status: Draft
- [Account States](02-roles-and-permissions/account-states.md) | Owner: Eng | Status: Draft

**Role Hierarchy**: Subscriber-Admin (full access) → Teacher-Admin (educational management) → Teacher (student management) → Student (game access)

## Student Experience
- [Student Dashboard](03-student-experience/student-dashboard.md) | Owner: Design | Status: Draft
- [Student Assignments](03-student-experience/student-assignments.md) | Owner: Design | Status: Draft
- [Assignment Play](03-student-experience/assignment-play.md) | Owner: Design | Status: Draft
- [Assignment Play Redesign](03-student-experience/assignment-play-redesign.md) | Owner: Design | Status: Draft
- [Game Player Shell](03-student-experience/game-player-shell.md) | Owner: Eng | Status: Draft
- [All Games Library](03-student-experience/all-games-library.md) | Owner: Design | Status: Draft
- [All Games Redesign](03-student-experience/all-games-redesign.md) | Owner: Design | Status: Draft
- [Leaderboards](03-student-experience/leaderboards.md) | Owner: Design | Status: Draft
- [Student Profile](03-student-experience/student-profile.md) | Owner: Design | Status: Draft
- [Student Messages](03-student-experience/student-messages.md) | Owner: PM | Status: Draft
- [Student Settings](03-student-experience/student-settings.md) | Owner: Eng | Status: Draft
- [Student Notifications](03-student-experience/student-notifications.md) | Owner: Eng | Status: Draft

## Teacher Experience
- [Teacher Dashboard](04-teacher-experience/teacher-dashboard.md) | Owner: Design | Status: Draft
- [Assignment Builder](04-teacher-experience/assignment-builder.md) | Owner: Design | Status: Draft
- [Teacher Reports](04-teacher-experience/teacher-reports.md) | Owner: PM | Status: Draft
- [Teacher Messaging](04-teacher-experience/teacher-messaging.md) | Owner: PM | Status: Draft
- [Teacher Challenges](04-teacher-experience/teacher-challenges.md) | Owner: PM | Status: Draft
- [Teacher Grade as Role](04-teacher-experience/teacher-grade-as-role.md) | Owner: PM | Status: Draft
- [Teacher Settings](04-teacher-experience/teacher-settings.md) | Owner: Eng | Status: Draft
- [Teacher Notifications](04-teacher-experience/teacher-notifications.md) | Owner: Eng | Status: Draft

## Admin and School Ops
- [Subscriber Dashboard](05-admin-subscriber-experience/subscriber-dashboard.md) | Owner: PM | Status: Draft
- [Member Management](05-admin-subscriber-experience/member-management.md) | Owner: Eng | Status: Draft
- [Student Bulk Operations](05-admin-subscriber-experience/student-bulk-operations.md) | Owner: Eng | Status: Draft
- [Mass Assignments](05-admin-subscriber-experience/mass-assignments.md) | Owner: Eng | Status: Draft
- [Billing Plans and Seats](05-admin-subscriber-experience/billing-plans-and-seats.md) | Owner: PM | Status: Draft
- [Billing Statements](05-admin-subscriber-experience/billing-statements.md) | Owner: PM | Status: Draft
- [Promo Codes](05-admin-subscriber-experience/promo-codes.md) | Owner: PM | Status: Draft
- [Sponsor Codes](05-admin-subscriber-experience/sponsor-codes.md) | Owner: PM | Status: Draft
- [School Hierarchy View](05-admin-subscriber-experience/school-hierarchy-view.md) | Owner: Eng | Status: Draft
- [Admin Reports](05-admin-subscriber-experience/admin-reports.md) | Owner: PM | Status: Draft
- [Admin Notifications](05-admin-subscriber-experience/admin-notifications.md) | Owner: Eng | Status: Draft

## System Admin
- [SysAdmin User Index](06-system-admin/sysadmin-user-index.md) | Owner: Eng | Status: Draft
- [SysAdmin Member Profile](06-system-admin/sysadmin-member-profile.md) | Owner: Eng | Status: Draft
- [SysAdmin Billing Tools](06-system-admin/sysadmin-billing-tools.md) | Owner: Eng | Status: Draft
- [SysAdmin Content Tools](06-system-admin/sysadmin-content-tools.md) | Owner: Eng | Status: Draft
- [SysAdmin Audit Logs](06-system-admin/sysadmin-audit-logs.md) | Owner: Eng | Status: Draft
- [SysAdmin Feature Flags](06-system-admin/sysadmin-feature-flags.md) | Owner: Eng | Status: Draft
- [User Permissions](06-system-admin/user-permissions.md) | Owner: Eng | Status: Draft

## Content Architecture
- [Learning Elements Overview](07-content-architecture/learning-elements-overview.md) | Owner: PM | Status: Draft
- [Game Taxonomy](07-content-architecture/game-taxonomy.md) | Owner: PM | Status: Draft
- [Assignment Model](07-content-architecture/assignment-model.md) | Owner: PM | Status: Draft
- [Group Model](07-content-architecture/group-model.md) | Owner: PM | Status: Draft
- [Content Versioning](07-content-architecture/content-versioning.md) | Owner: Eng | Status: Draft
- [Game Model](07-content-architecture/game-model.md) | Owner: PM | Status: Draft
- [Game Catalog](07-content-architecture/game-catalog.md) | Owner: PM | Status: Draft
- [Piano Adventures Alignment](07-content-architecture/piano-adventures-alignment.md) | Owner: PM | Status: Draft

## Games Registry and Authoring
- [Game Model](08-games-registry-and-authoring/game-model.md) | Owner: Eng | Status: Draft
- [Games Registry Overview](08-games-registry-and-authoring/games-registry-overview.md) | Owner: Eng | Status: Draft
- [Games CRUD](08-games-registry-and-authoring/games-crud.md) | Owner: Eng | Status: Draft
- [Games Stages Policy](08-games-registry-and-authoring/games-stages-policy.md) | Owner: PM | Status: Draft
- [Games Metadata](08-games-registry-and-authoring/games-metadata.md) | Owner: PM | Status: Draft
- [Games Assets Pipeline](08-games-registry-and-authoring/games-assets-pipeline.md) | Owner: Eng | Status: Draft
- [Games Importer](08-games-registry-and-authoring/games-importer.md) | Owner: Eng | Status: Draft
- [Games Audit and Health](08-games-registry-and-authoring/games-audit-and-health.md) | Owner: QA | Status: Draft
- [Games Telemetry](08-games-registry-and-authoring/games-telemetry.md) | Owner: Eng | Status: Draft

**Game Structure**: Each game has Learn (tutor) → Play (practice) → Quiz (mastery) → Challenge (competition) → Review (retention) stages

## Sequences and Curriculum Tools
- [Sequence Model](09-sequences-and-curriculum-tools/sequence-model.md) | Owner: PM | Status: Draft
- [Sequence Libraries](09-sequences-and-curriculum-tools/sequence-libraries.md) | Owner: PM | Status: Draft
- [Sequence Authoring](09-sequences-and-curriculum-tools/sequence-authoring.md) | Owner: Design | Status: Draft
- [Sequence Alignment to Methods](09-sequences-and-curriculum-tools/sequence-alignment-to-methods.md) | Owner: PM | Status: Draft
- [Sequence Placements and EVAL](09-sequences-and-curriculum-tools/sequence-placements-eval.md) | Owner: PM | Status: Draft
- [Sequence Exports](09-sequences-and-curriculum-tools/sequence-exports.md) | Owner: Eng | Status: Draft
- [AI Sequence Generator](09-sequences-and-curriculum-tools/ai-sequence-generator.md) | Owner: Eng | Status: Draft
- [Curriculum Mapping Tools](09-sequences-and-curriculum-tools/curriculum-mapping-tools.md) | Owner: PM | Status: Draft

**Available Sequences**: Lifetime Musician (default), Solfege (voice students), Evaluation (assessment), plus method-specific sequences (Alfred, Faber, etc.)

## Assignments Engine
- [Assignment Generation](10-assignments-engine/assignment-generation.md) | Owner: Eng | Status: Draft
- [Assignment Progress Tracking](10-assignments-engine/assignment-progress-tracking.md) | Owner: Eng | Status: Draft
- [Retention and Review Logic](10-assignments-engine/retention-review-logic.md) | Owner: PM | Status: Draft
- [Free Play vs Assigned](10-assignments-engine/free-play-vs-assigned.md) | Owner: PM | Status: Draft

## Search and Discovery
- [Search Indexing](11-search-and-discovery/search-indexing.md) | Owner: Eng | Status: Draft
- [Filters and Facets](11-search-and-discovery/filters-and-facets.md) | Owner: PM | Status: Draft
- [Teacher Quick Find](11-search-and-discovery/teacher-quick-find.md) | Owner: PM | Status: Draft

## Gamification
- [Badges System](12-gamification/badges-system.md) | Owner: PM | Status: Draft
- [Points and Streaks](12-gamification/points-and-streaks.md) | Owner: PM | Status: Draft
- [Challenge Boards](12-gamification/challenge-boards.md) | Owner: PM | Status: Draft
- [Achievements Sharing](12-gamification/achievements-sharing.md) | Owner: PM | Status: Draft

## Messaging and Notifications
- [In-app Notifications](13-messaging-and-notifications/in-app-notifications.md) | Owner: Eng | Status: Draft
- [Email Templates](13-messaging-and-notifications/email-templates.md) | Owner: PM | Status: Draft
- [Push and Desktop](13-messaging-and-notifications/push-and-desktop.md) | Owner: Eng | Status: Draft
- [System Announcements](13-messaging-and-notifications/system-announcements.md) | Owner: PM | Status: Draft
- [Communication Framework](13-messaging-and-notifications/communication-framework.md) | Owner: PM | Status: Draft

## Payments and Subscriptions
- [Plans and Pricing](14-payments-and-subscriptions/plans-and-pricing.md) | Owner: PM | Status: Draft
- [Checkout and Trials](14-payments-and-subscriptions/checkout-and-trials.md) | Owner: Eng | Status: Draft
- [Seat Management](14-payments-and-subscriptions/seat-management.md) | Owner: PM | Status: Draft
- [Billing Provider Integration](14-payments-and-subscriptions/billing-provider-integration.md) | Owner: Eng | Status: Draft
- [PO and Invoicing for Schools](14-payments-and-subscriptions/po-and-invoicing-for-schools.md) | Owner: PM | Status: Draft
- [Pause and Reactivation](14-payments-and-subscriptions/pause-and-reactivation.md) | Owner: PM | Status: Draft
- [Billing Statements on Demand](14-payments-and-subscriptions/billing-statements-on-demand.md) | Owner: PM | Status: Draft
- [Promo Codes Implementation](14-payments-and-subscriptions/promo-codes-impl.md) | Owner: Eng | Status: Draft
- [Sponsor Codes Implementation](14-payments-and-subscriptions/sponsor-codes-impl.md) | Owner: Eng | Status: Draft
- [Billing Plans and Seats](14-payments-and-subscriptions/billing-plans-and-seats.md) | Owner: PM | Status: Draft
- [Pricing Models](14-payments-and-subscriptions/pricing-models.md) | Owner: PM | Status: Draft
- [Seat Pricing Calculator](14-payments-and-subscriptions/seat-pricing-calculator.md) | Owner: PM | Status: Draft
- [Promo Codes](14-payments-and-subscriptions/promo-codes.md) | Owner: PM | Status: Draft
- [Sponsor Codes](14-payments-and-subscriptions/sponsor-codes.md) | Owner: PM | Status: Draft

**Subscription Tiers**: Prelude (free trial, 19 seats), Solo ($7.95/month, 5-19 seats), Ensemble ($19.95/month, 20+ seats)

## Analytics and Reporting
- [Event Model](15-analytics-and-reporting/event-model.md) | Owner: Eng | Status: Draft
- [Teacher Reports KPIs](15-analytics-and-reporting/teacher-reports-kpis.md) | Owner: PM | Status: Draft
- [Admin Reports KPIs](15-analytics-and-reporting/admin-reports-kpis.md) | Owner: PM | Status: Draft
- [SysAdmin Ops Dashboards](15-analytics-and-reporting/sysadmin-ops-dashboards.md) | Owner: Eng | Status: Draft
- [Learning Analytics](15-analytics-and-reporting/learning-analytics.md) | Owner: PM | Status: Draft

## Accessibility and Inclusion
- [Neurodiverse Support](16-accessibility-and-inclusion/neurodiverse-support.md) | Owner: Design | Status: Draft
- [Captions and Audio Descriptions](16-accessibility-and-inclusion/captions-and-audio-descriptions.md) | Owner: PM | Status: Draft
- [Keyboard and MIDI Access](16-accessibility-and-inclusion/keyboard-and-midi-access.md) | Owner: Eng | Status: Draft
- [WCAG Compliance](16-accessibility-and-inclusion/wcag-compliance.md) | Owner: Design | Status: Draft

## Integrations
- [MIDI Support](17-integrations/midi-support.md) | Owner: Eng | Status: Draft
- [LMS LTI Spec Future](17-integrations/lms-ltispec-future.md) | Owner: Eng | Status: Draft
- [Email Provider](17-integrations/email-provider.md) | Owner: Eng | Status: Draft
- [CDN and Media](17-integrations/cdn-and-media.md) | Owner: Eng | Status: Draft
- [SCORM Compliance](17-integrations/scorm-compliance.md) | Owner: Eng | Status: Draft
- [SSO Integration](17-integrations/sso-integration.md) | Owner: Eng | Status: Draft
- [xAPI Implementation](17-integrations/xapi-implementation.md) | Owner: Eng | Status: Draft

## Architecture
- [System Overview](18-architecture/system-overview.md) | Owner: Eng | Status: Draft
- [Service Boundaries](18-architecture/service-boundaries.md) | Owner: Eng | Status: Draft
- [Data Model ERD](18-architecture/data-model-erd.md) | Owner: Eng | Status: Draft
- [API Design Principles](18-architecture/api-design-principles.md) | Owner: Eng | Status: Draft
- [REST Endpoints](18-architecture/rest-endpoints.md) | Owner: Eng | Status: Draft
- [Webhooks](18-architecture/webhooks.md) | Owner: Eng | Status: Draft
- [Background Jobs](18-architecture/background-jobs.md) | Owner: Eng | Status: Draft
- [Search Architecture](18-architecture/search-architecture.md) | Owner: Eng | Status: Draft
- [Caching Strategy](18-architecture/caching-strategy.md) | Owner: Eng | Status: Draft
- [Security and Privacy](18-architecture/security-privacy.md) | Owner: Eng | Status: Draft
- [Rate Limiting and Abuse](18-architecture/rate-limiting-and-abuse.md) | Owner: Eng | Status: Draft
- [Config and Secrets](18-architecture/config-and-secrets.md) | Owner: Eng | Status: Draft
- [Browser Compatibility Matrix](18-architecture/browser-compatibility-matrix.md) | Owner: Eng | Status: Draft
- [Performance and Scalability](18-architecture/performance-and-scalability.md) | Owner: Eng | Status: Draft

## Quality and Operations
- [Test Strategy](19-quality-and-operations/test-strategy.md) | Owner: QA | Status: Draft
- [Browser Device Matrix](19-quality-and-operations/browser-device-matrix.md) | Owner: QA | Status: Draft
- [QA Checklists](19-quality-and-operations/qa-checklists.md) | Owner: QA | Status: Draft
- [Release Management](19-quality-and-operations/release-management.md) | Owner: Eng | Status: Draft
- [Observability](19-quality-and-operations/observability.md) | Owner: Eng | Status: Draft
- [Incident Response](19-quality-and-operations/incident-response.md) | Owner: Eng | Status: Draft
- [Testing Requirements](19-quality-and-operations/testing-requirements.md) | Owner: QA | Status: Draft

## Imports and Automation
- [CSV Spec Students](20-imports-and-automation/csv-spec-students.md) | Owner: Eng | Status: Draft
- [CSV Spec Teachers](20-imports-and-automation/csv-spec-teachers.md) | Owner: Eng | Status: Draft
- [CSV Spec Courses and Assignments](20-imports-and-automation/csv-spec-courses-assignments.md) | Owner: Eng | Status: Draft
- [CSV Spec Games](20-imports-and-automation/csv-spec-games.md) | Owner: Eng | Status: Draft
- [Bulk Ops Jobs](20-imports-and-automation/bulk-ops-jobs.md) | Owner: Eng | Status: Draft
- [Export Specs](20-imports-and-automation/export-specs.md) | Owner: Eng | Status: Draft
- [CSV Specifications](20-imports-and-automation/csv-specifications.md) | Owner: Eng | Status: Draft
- [Data Migration](20-imports-and-automation/data-migration.md) | Owner: Eng | Status: Draft

## Mobile and Offline
- [Responsive Behaviors](21-mobile-and-offline/responsive-behaviors.md) | Owner: Design | Status: Draft
- [PWA Considerations](21-mobile-and-offline/pwa-considerations.md) | Owner: Eng | Status: Draft
- [PWA Specifications](21-mobile-and-offline/pwa-specifications.md) | Owner: Eng | Status: Draft

## Content Style Guides
- [Copy Guide Students](22-content-style-guides/copy-guide-students.md) | Owner: Design | Status: Draft
- [Copy Guide Teachers](22-content-style-guides/copy-guide-teachers.md) | Owner: Design | Status: Draft
- [Copy Guide Admins](22-content-style-guides/copy-guide-admins.md) | Owner: Design | Status: Draft
- [Error Messages](22-content-style-guides/error-messages.md) | Owner: PM | Status: Draft
- [Copy Guidelines](22-content-style-guides/copy-guidelines.md) | Owner: Design | Status: Draft
- [Terry Treble Branding](22-content-style-guides/terry-treble-branding.md) | Owner: Design | Status: Draft

## Legal and Compliance
- [Privacy Policy Outline](23-legal-and-compliance/privacy-policy-outline.md) | Owner: Legal | Status: Draft
- [Terms of Service Outline](23-legal-and-compliance/tos-outline.md) | Owner: Legal | Status: Draft
- [COPPA and FERPA Notes](23-legal-and-compliance/coppa-ferpa-notes.md) | Owner: Legal | Status: Draft
- [Copyright and Licensing](23-legal-and-compliance/copyright-and-licensing.md) | Owner: Legal | Status: Draft
- [Privacy Policy](23-legal-and-compliance/privacy-policy.md) | Owner: Legal | Status: Draft
- [Terms and Conditions](23-legal-and-compliance/terms-conditions.md) | Owner: Legal | Status: Draft
- [GDPR Compliance](23-legal-and-compliance/gdpr-compliance.md) | Owner: Legal | Status: Draft
- [FERPA Compliance](23-legal-and-compliance/ferpa-compliance.md) | Owner: Legal | Status: Draft

## Roadmap and Phasing
- [Phase 1 Scope](24-roadmap-and-phasing/phase-1-scope.md) | Owner: PM | Status: Draft
- [Phase 2 Scope](24-roadmap-and-phasing/phase-2-scope.md) | Owner: PM | Status: Draft
- [Phase 3 Scope](24-roadmap-and-phasing/phase-3-scope.md) | Owner: PM | Status: Draft
- [Measures of Success](24-roadmap-and-phasing/measures-of-success.md) | Owner: PM | Status: Draft

## Research and Inspiration
- [Wireframes 2012 Notes](25-research-and-inspiration/wireframes-2012-notes.md) | Owner: Design | Status: Draft
- [Competitor Scan 2025](25-research-and-inspiration/competitor-scan-2025.md) | Owner: PM | Status: Draft
- [Competitor Analysis](25-research-and-inspiration/competitor-analysis.md) | Owner: PM | Status: Draft
- [User Feedback](25-research-and-inspiration/user-feedback.md) | Owner: PM | Status: Draft

## Repo Operations
- [Issue Templates](26-repo-operations/issue-templates.md) | Owner: PM | Status: Draft
- [Docs Index](00-foundations/docs-index.md) | Owner: PM | Status: Stable

## Social Learning
- [Social Learning Framework](27-social-learning/social-learning-framework.md) | Owner: PM | Status: Draft
- [Community Features](27-social-learning/community-features.md) | Owner: PM | Status: Draft

## Advanced Assessment
- [Proctoring Security](28-advanced-assessment/proctoring-security.md) | Owner: Eng | Status: Draft
- [Competency Assessment](28-advanced-assessment/competency-assessment.md) | Owner: PM | Status: Draft

## Content Management
- [Advanced Versioning](29-content-management/advanced-versioning.md) | Owner: Eng | Status: Draft
- [Content Approval Workflows](29-content-management/content-approval-workflows.md) | Owner: PM | Status: Draft

## AI and Machine Learning
- [AI Content Generation](30-ai-and-machine-learning/ai-content-generation.md) | Owner: Eng | Status: Draft
- [Predictive Analytics](30-ai-and-machine-learning/predictive-analytics.md) | Owner: PM | Status: Draft

---

## Historical Context and Legacy Information

### System Evolution
- **Founded**: 2006 by Christine Hermanson, pioneer in music education technology
- **Original Platform**: Adobe Flash-based games with cloud delivery
- **Current Status**: Complete redesign with modern web technologies
- **Global Reach**: 60,000+ students served across 40+ countries

### Key Historical Features
- **Terry Treble**: Mascot character created by Paul Hermanson
- **MIDI Support**: USB and Bluetooth keyboard integration since inception
- **Challenge Games**: Global leaderboards with top 20 worldwide scores
- **Special Needs Support**: Proven effectiveness with autistic students
- **Method Alignment**: Student Pages for Alfred, Faber, and other popular methods

### Legacy Documentation
- **Original Site Content**: See `00-ORIG-CONTEXT/MLC Site Content.docx.txt`
- **User Role Details**: See `00-ORIG-CONTEXT/USER ROLES detail.docx.txt`
- **Historical Sequences**: See `00-ORIG-CONTEXT/MLC SeqCreate 2020.xlsx` files
- **Pricing Evolution**: See `00-ORIG-CONTEXT/MLC Tiered Pricing 2020.xlsx` files

### Technical Requirements
- **Browser Support**: Chrome, Firefox, Edge, Opera, Safari (post-April 2020)
- **Operating Systems**: PC, macOS, Chrome OS, Android, iOS
- **MIDI Support**: USB and Bluetooth keyboard connectivity
- **Screen Resolution**: Minimum 800x600 (full-screen mode recommended)
- **Internet**: Reliable connection required for game play
- **Mobile**: Native iOS app available for MIDI support

---

### Conventions
- Status values: Draft, In Review, Stable, Deprecated
- Filenames use kebab-case
- One spec per file
- Each spec begins with Purpose, Scope, Non-goals, Dependencies, Open Questions

### Changelog
- 2025-01-27: Added hyperlinks to all file references, added missing files from new directories (27-30), updated status indicators, and enhanced with historical context, technical requirements, and legacy documentation references
- 2025-10-01: Initial index created
```
