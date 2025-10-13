# Content Versioning

This document defines the comprehensive content versioning and update management system for the MLC platform. It establishes how learning elements, games, sequences, and educational content are versioned, updated, and maintained across the system.

## Purpose

The content versioning system enables:

- **Content Evolution**: Systematic management of content updates, improvements, and corrections
- **Curriculum Continuity**: Ensuring learning sequences remain functional during content updates
- **Quality Assurance**: Controlled deployment of content changes with rollback capabilities
- **Multi-version Support**: Maintaining different content versions for different user groups or regions
- **Audit Trail**: Complete history of content changes for compliance and analysis
- **Collaborative Editing**: Support for multiple content creators working on the same materials
- **A/B Testing**: Ability to test different content versions with different user segments

## Scope

### Included
- **Learning Element Versioning**: Games, videos, audio, text, and reward elements
- **Sequence Versioning**: Learning sequences and curriculum structures
- **Content Asset Management**: Multimedia files, images, and interactive content
- **Metadata Versioning**: Tags, descriptions, and classification data
- **Translation Management**: Multi-language content versions
- **Deployment Control**: Staged rollout and rollback capabilities
- **Version Dependencies**: Managing relationships between different content versions

### Excluded
- **User Data Versioning**: Student progress and personal information (covered in [data-model-erd](../18-architecture/data-model-erd.md))
- **System Configuration**: Platform settings and feature flags (covered in [config-and-secrets](../18-architecture/config-and-secrets.md))
- **Assignment Logic**: How content is presented to students (covered in [assignment-model](assignment-model.md))

## Data Model

### Core Versioning Entities

#### Content Version
```json
{
  "version_id": "v1.2.3",
  "content_id": "GAM-0950",
  "content_type": "game",
  "version_number": "1.2.3",
  "status": "published",
  "created_at": "2024-01-15T10:30:00Z",
  "created_by": "user_123",
  "change_summary": "Updated graphics and improved accessibility",
  "breaking_changes": false,
  "compatibility": {
    "min_platform_version": "5.0.0",
    "max_platform_version": null
  }
}
```

#### Content Item
```json
{
  "content_id": "GAM-0950",
  "content_type": "game",
  "current_version": "v1.2.3",
  "latest_version": "v1.2.3",
  "created_at": "2023-06-01T09:00:00Z",
  "created_by": "user_456",
  "status": "active",
  "version_history": ["v1.0.0", "v1.1.0", "v1.2.0", "v1.2.3"]
}
```

#### Version Dependencies
```json
{
  "version_id": "v1.2.3",
  "content_id": "GAM-0950",
  "dependencies": [
    {
      "content_id": "VID-INS-2005",
      "version_requirement": ">=v2.1.0",
      "dependency_type": "prerequisite"
    },
    {
      "content_id": "TXT-LAB-3015",
      "version_requirement": ">=v1.0.0",
      "dependency_type": "reference"
    }
  ]
}
```

### Version Numbering Scheme

#### Semantic Versioning
- **Major Version (X.0.0)**: Breaking changes that affect sequence compatibility
- **Minor Version (X.Y.0)**: New features or significant improvements
- **Patch Version (X.Y.Z)**: Bug fixes and minor improvements

#### Content-Specific Versioning
- **Games**: GAM-XXXX-vY.Z format (e.g., GAM-0950-v1.2)
- **Sequences**: SEQ-XXXX-vY.Z format (e.g., LIFE-000010-v2.1)
- **Multimedia**: MEDIA-XXXX-vY.Z format (e.g., VID-INS-2005-v3.0)

### Version States

#### Content Version States
1. **Draft**: Initial creation, not yet ready for review
2. **Review**: Submitted for quality assurance and approval
3. **Approved**: Passed review, ready for staging
4. **Staged**: Deployed to staging environment for testing
5. **Published**: Live in production environment
6. **Deprecated**: Marked for removal, no longer recommended
7. **Archived**: Removed from active use, historical only

#### Content Item States
1. **Active**: Currently available to users
2. **Maintenance**: Temporarily unavailable for updates
3. **Retired**: No longer available to new users
4. **Deleted**: Removed from system (soft delete)

## Behavior and Rules

### Version Creation Rules

#### Automatic Versioning
- **Auto-increment**: Patch versions increment automatically on content updates
- **Manual Promotion**: Minor and major versions require explicit promotion
- **Breaking Change Detection**: System identifies potential breaking changes
- **Dependency Validation**: Ensures all dependencies are satisfied

#### Content Update Workflow
1. **Content Modification**: Author makes changes to content
2. **Version Creation**: System creates new patch version automatically
3. **Validation**: Automated checks for syntax, dependencies, and compatibility
4. **Review Process**: Content goes through quality assurance review
5. **Staging Deployment**: Approved content deployed to staging environment
6. **Production Deployment**: Final approval and deployment to production

### Version Compatibility Rules

#### Backward Compatibility
- **Patch Versions**: Must maintain full backward compatibility
- **Minor Versions**: Should maintain API compatibility, may add features
- **Major Versions**: May include breaking changes with migration path

#### Sequence Compatibility
- **Sequence Preservation**: Existing sequences continue to work with new versions
- **Dependency Updates**: Automatic updates to compatible newer versions
- **Migration Support**: Tools to update sequences to use new versions

### Content Deployment Rules

#### Staged Rollout
1. **Internal Testing**: Content tested by development team
2. **Beta Testing**: Limited release to selected teachers/students
3. **Gradual Rollout**: Phased deployment to increasing user percentage
4. **Full Deployment**: Complete rollout to all users

#### Rollback Procedures
- **Automatic Rollback**: System automatically reverts on critical errors
- **Manual Rollback**: Admin can manually revert to previous version
- **Selective Rollback**: Rollback specific content while keeping others

## UX Requirements

### Content Management Interface

#### Version History View
- **Timeline Display**: Chronological view of all versions
- **Change Highlights**: Visual indicators of what changed between versions
- **Comparison Tools**: Side-by-side comparison of different versions
- **Rollback Controls**: Easy reversion to previous versions

#### Content Editor Integration
- **Version Awareness**: Editor shows current version and pending changes
- **Change Tracking**: Real-time indication of modifications
- **Collaboration Features**: Multi-user editing with conflict resolution
- **Preview Mode**: Test content changes before publishing

### Student and Teacher Experience

#### Seamless Updates
- **Transparent Updates**: Content updates appear without user intervention
- **Progress Preservation**: Student progress maintained across version changes
- **Notification System**: Optional notifications about significant content updates
- **Version Information**: Access to content version information when needed

#### Accessibility Considerations
- **Screen Reader Support**: Version information accessible to assistive technologies
- **Keyboard Navigation**: Full keyboard support for version management interfaces
- **High Contrast**: Clear visual distinction between different version states
- **Reduced Motion**: Respect user preferences for animations and transitions

## Telemetry

### Version Management Events
- **version_created**: When a new content version is created
- **version_published**: When a version is deployed to production
- **version_rolled_back**: When a version is reverted to previous state
- **content_updated**: When content is modified and versioned
- **dependency_updated**: When content dependencies are modified

### Analytics Properties
- **content_id**: Identifier of the content being versioned
- **version_number**: Specific version number
- **change_type**: Type of change (major, minor, patch)
- **breaking_changes**: Whether the version includes breaking changes
- **deployment_method**: How the version was deployed (staged, immediate, etc.)
- **rollback_reason**: Reason for any rollbacks that occurred

### Performance Metrics
- **deployment_time**: Time taken to deploy new versions
- **rollback_frequency**: How often versions are rolled back
- **version_adoption**: How quickly new versions are adopted by users
- **compatibility_issues**: Frequency of compatibility problems

## Permissions

### Content Creator Access
- **Create Versions**: Can create new versions of content they own
- **Edit Drafts**: Can modify content in draft state
- **Submit for Review**: Can submit versions for quality assurance
- **View History**: Can see version history for their content

### Content Reviewer Access
- **Review Versions**: Can approve or reject content versions
- **Quality Control**: Can test content in staging environment
- **Approve Deployment**: Can approve content for production deployment
- **View All History**: Can see version history for all content

### System Admin Access
- **Full Version Control**: Complete control over all content versions
- **Deployment Management**: Can deploy, rollback, and manage all versions
- **Emergency Actions**: Can take emergency actions to protect system stability
- **Analytics Access**: Full access to version management analytics

### Teacher Access
- **Version Information**: Can see version information for assigned content
- **Update Notifications**: Can receive notifications about content updates
- **Version Selection**: Can choose specific versions for custom sequences
- **Feedback Submission**: Can provide feedback on content versions

## Supporting Documents Referenced

This content versioning specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Original game organization and structure specifications
- [Games Loaded.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Sheet1.csv) — Current game library inventory and status tracking
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Game import specifications and metadata structure
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence creation specifications including content dependencies

## Dependencies

### Internal Dependencies
- **[Learning Elements Overview](learning-elements-overview.md)**: Defines content types that need versioning
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Defines game-specific versioning requirements
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: Defines sequence versioning and compatibility
- **[Assignment Engine](../10-assignments-engine/assignment-generation.md)**: Uses versioned content for student assignments

### External Dependencies
- **Content Management System**: Stores and serves versioned content
- **CDN**: Distributes content versions efficiently
- **Database**: Stores version metadata and relationships
- **File Storage**: Manages versioned content assets
- **Deployment Pipeline**: Automated deployment and rollback systems

## Open Questions

### Version Strategy
- How should we handle content that spans multiple learning elements?
- What is the optimal versioning strategy for multimedia assets?
- How can we minimize disruption during content updates?

### Technical Implementation
- What is the best approach for handling large file versioning?
- How should we implement real-time collaboration on content versions?
- What caching strategy will optimize versioned content delivery?

### User Experience
- How can we make version management transparent to end users?
- What level of version control should teachers have access to?
- How should we handle content that requires user re-training?

### Quality Assurance
- What automated testing should be performed on new versions?
- How can we ensure content quality across different versions?
- What is the optimal review process for content updates?
