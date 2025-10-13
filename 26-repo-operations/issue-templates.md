# Issue Templates

This document defines standardized issue templates for managing and maintaining the MLC LMS specifications repository.

## Purpose

This specification enables efficient internal collaboration on the specifications repository by providing:

- **Structured Issue Tracking**: Standardized templates for common spec management tasks
- **Clear Communication**: Consistent format for reporting issues and requesting enhancements
- **Work Prioritization**: Categorized templates help prioritize spec maintenance work
- **Knowledge Capture**: Templates ensure critical context is captured when issues are created
- **Onboarding Support**: New team members can quickly understand how to contribute to spec maintenance

Unlike open-source contribution guidelines, these templates serve the internal team managing this private specification repository. They facilitate coordination among product managers, designers, engineers, and QA working together to keep specs accurate, complete, and consistent.

## Scope

### Included

- **Issue Template Definitions**: Complete templates for common spec repository tasks
- **Template Fields**: Required and optional fields for each template type
- **Template Usage Guidelines**: When to use each template
- **Issue Labels**: Categorization and priority labels for spec issues
- **Workflow States**: Status labels for tracking issue progress
- **Priority Definitions**: How to assign and interpret priority levels

### Excluded

- **Product Feature Requests**: Templates for the actual MLC LMS product (those belong in the product backlog, not this spec repo)
- **Bug Reports for Production System**: Templates for live system bugs (covered in [Incident Response](../19-quality-and-operations/incident-response.md))
- **GitHub Configuration**: Actual `.github/ISSUE_TEMPLATE/` YAML files (this document defines the templates; implementation is separate)
- **Project Management Process**: Broader team workflows, sprint planning, and agile ceremonies
- **Code Review Standards**: Engineering practices for the actual codebase

---

## Issue Template Types

### 1. New Specification Request

**When to Use**: When a new specification document is needed to cover a feature, system, or process not yet documented.

**Template:**

```markdown
## Summary
[Brief 1-2 sentence description of what this spec should cover]

## Motivation
Why is this specification needed?
- [ ] New feature being planned
- [ ] Existing feature lacks documentation
- [ ] Gap identified during implementation
- [ ] Requested by stakeholder: [Name]
- [ ] Other: [Explain]

## Scope
What should this spec include?
- [List key topics]
- [List major sections]

## Related Specifications
Which existing specs relate to this one?
- [Spec 1] - [Brief relationship]
- [Spec 2] - [Brief relationship]

## Target Audience
Who is the primary audience for this spec?
- [ ] Product Managers
- [ ] Designers
- [ ] Engineers
- [ ] QA
- [ ] System Administrators
- [ ] Business Stakeholders

## Success Criteria
What does "complete" look like for this spec?
- [Clear, measurable criteria]

## Priority
- [ ] P0 - Blocking development work (needed this sprint)
- [ ] P1 - High (needed next sprint)
- [ ] P2 - Medium (needed this quarter)
- [ ] P3 - Low (nice to have)

## Proposed Location
Suggested directory and filename: [e.g., `15-analytics-and-reporting/student-progress-reports.md`]

## Additional Context
Links to related materials, previous discussions, or source documents:
- [Links or attachments]
```

**Labels**: `type: new-spec`, `status: proposed`, priority label

---

### 2. Specification Enhancement

**When to Use**: When an existing spec needs additional detail, clarification, or expansion (not fixing errors - see "Spec Error/Correction" template).

**Template:**

```markdown
## Specification
Which spec needs enhancement?
- **File**: [Path to spec, e.g., `08-games-registry-and-authoring/game-model.md`]
- **Section**: [Specific section if applicable]

## Current State
What does the spec currently cover?
[Brief description]

## Proposed Enhancement
What should be added or expanded?
[Detailed description]

## Justification
Why is this enhancement needed?
- [ ] Implementation revealed missing details
- [ ] Cross-functional questions arose
- [ ] Related specs have evolved
- [ ] Stakeholder feedback
- [ ] Other: [Explain]

## Suggested Content
What specifically should be added?
```markdown
[Draft text, examples, or outline]
```

## Impact
Who is affected by this gap?
- [ ] Engineers implementing the feature
- [ ] Designers creating mockups
- [ ] QA writing test cases
- [ ] Product team planning features
- [ ] Other: [Specify]

## Related Documents
Which other specs should be reviewed for consistency?
- [Spec 1]
- [Spec 2]

## Priority
- [ ] P0 - Blocking current work
- [ ] P1 - High (needed this sprint)
- [ ] P2 - Medium (needed next sprint)
- [ ] P3 - Low (continuous improvement)
```

**Labels**: `type: enhancement`, `status: proposed`, priority label, affected area label

---

### 3. Specification Error / Correction

**When to Use**: When a spec contains incorrect information, inconsistencies, or conflicts with other specs.

**Template:**

```markdown
## Specification
Which spec has the error?
- **File**: [Path to spec]
- **Section**: [Specific section]
- **Line Number(s)**: [If applicable]

## Error Description
What is incorrect?
[Clear description of the error]

## Current Text
What does the spec currently say?
```markdown
[Quote the incorrect text]
```

## Correct Information
What should it say instead?
```markdown
[Provide correct text]
```

## Impact
What is the consequence of this error?
- [ ] Could lead to incorrect implementation
- [ ] Conflicts with another spec (specify: [Spec name])
- [ ] Outdated information (no longer accurate)
- [ ] Typo or formatting issue (low impact)
- [ ] Data model inconsistency
- [ ] Other: [Explain]

## Source of Correction
How do you know this is incorrect?
- [ ] Discovered during implementation
- [ ] Conflict with: [Other spec]
- [ ] Updated requirements from stakeholder
- [ ] Correction from original context materials
- [ ] Other: [Explain]

## Related Specs
Do other specs need similar corrections?
- [List any other specs that may have the same error]

## Priority
- [ ] P0 - Critical (blocking work or causing confusion)
- [ ] P1 - High (should be fixed this sprint)
- [ ] P2 - Medium (should be fixed soon)
- [ ] P3 - Low (minor issue, fix when convenient)
```

**Labels**: `type: correction`, `status: reported`, priority label, `impact: [level]`

---

### 4. Cross-Document Consistency Issue

**When to Use**: When multiple specs have conflicting information or when a change in one spec requires updates to related specs.

**Template:**

```markdown
## Affected Specifications
Which specs have inconsistent information?
1. [Spec 1]: [Path]
2. [Spec 2]: [Path]
3. [Spec 3]: [Path] (if applicable)

## Inconsistency Description
What is inconsistent?
[Clear description of the conflict or duplication]

## Spec 1 Says:
```markdown
[Quote from first spec]
```

## Spec 2 Says:
```markdown
[Quote from second spec]
```

## Proposed Resolution
Which version is correct, or what should the consistent approach be?
[Detailed explanation]

## Recommended Actions
What changes are needed?
- [ ] Update Spec 1: [Specific change]
- [ ] Update Spec 2: [Specific change]
- [ ] Create cross-references between specs
- [ ] Consolidate duplicate content
- [ ] Other: [Explain]

## Root Cause
Why did this inconsistency occur?
- [ ] Specs developed independently
- [ ] Requirements changed after initial spec
- [ ] Missing cross-review during creation
- [ ] Parallel editing by different team members
- [ ] Other: [Explain]

## Impact
Who is affected by this inconsistency?
- [ ] Could cause implementation errors
- [ ] Creates confusion for readers
- [ ] Duplicate maintenance burden
- [ ] Other: [Explain]

## Priority
- [ ] P0 - Critical (causing active confusion)
- [ ] P1 - High (should resolve this sprint)
- [ ] P2 - Medium (should resolve soon)
- [ ] P3 - Low (minor inconsistency)
```

**Labels**: `type: consistency`, `status: identified`, priority label, `multiple-specs`

---

### 5. Missing Dependency / Reference

**When to Use**: When a spec references another document that doesn't exist, or when a needed cross-reference is missing.

**Template:**

```markdown
## Specification
Which spec has the missing reference?
- **File**: [Path to spec]
- **Section**: [Specific section]

## Issue Type
- [ ] References a non-existent spec
- [ ] Missing a needed cross-reference to existing spec
- [ ] Broken internal link
- [ ] Missing reference to source material

## Current State
What does the spec currently say or reference?
```markdown
[Quote the text with missing reference]
```

## Expected Reference
What should be referenced?
- **Target Document**: [Expected spec or source document]
- **Reason**: [Why this reference is needed]

## Recommended Action
- [ ] Create the missing spec (use "New Specification Request" template)
- [ ] Add cross-reference to existing spec: [Spec name and path]
- [ ] Fix broken link
- [ ] Add reference to source material in `00-ORIG-CONTEXT/`
- [ ] Other: [Explain]

## Impact
What is the consequence of this missing reference?
- [ ] Readers can't find related information
- [ ] Context is missing for implementation
- [ ] Duplicate content being created
- [ ] Other: [Explain]

## Priority
- [ ] P0 - Blocking work
- [ ] P1 - High (needed this sprint)
- [ ] P2 - Medium (should be fixed)
- [ ] P3 - Low (nice to have)
```

**Labels**: `type: missing-reference`, `status: identified`, priority label

---

### 6. Source Material Integration

**When to Use**: When materials from `00-ORIG-CONTEXT/` need to be reviewed and integrated into specifications.

**Template:**

```markdown
## Source Material
Which file(s) from `00-ORIG-CONTEXT/` need to be integrated?
- **File(s)**: [List files]
- **Type**: [CSV, TXT, PDF]

## Target Specification(s)
Which spec(s) should incorporate this information?
- [Spec 1]: [Path]
- [Spec 2]: [Path]

## Content Summary
What does the source material contain?
[Brief summary of key information]

## Integration Plan
How should this content be incorporated?
- [ ] Add new section to existing spec
- [ ] Enhance existing section with details
- [ ] Create new spec (use "New Specification Request")
- [ ] Add to "Supporting Documents Referenced" section
- [ ] Extract data tables or examples
- [ ] Other: [Explain]

## Key Information to Extract
What specific details are valuable?
- [Detail 1]
- [Detail 2]
- [Detail 3]

## Challenges
Any difficulties with integrating this material?
- [ ] Format conversion needed (CSV → markdown tables)
- [ ] Conflicts with existing specs
- [ ] Outdated information (needs verification)
- [ ] Unclear or ambiguous content
- [ ] Other: [Explain]

## Priority
- [ ] P0 - Critical (needed for current work)
- [ ] P1 - High (needed this sprint)
- [ ] P2 - Medium (should be integrated)
- [ ] P3 - Low (reference only)
```

**Labels**: `type: source-integration`, `status: pending`, priority label, `source-materials`

---

### 7. Specification Review Request

**When to Use**: When a spec is complete and needs review from stakeholders before being marked as stable.

**Template:**

```markdown
## Specification
Which spec needs review?
- **File**: [Path to spec]
- **Status**: [Current status: Draft, In Review, etc.]

## Review Type
- [ ] Initial review (new spec)
- [ ] Major revision review
- [ ] Technical accuracy review
- [ ] Cross-functional review
- [ ] Final approval before marking stable

## Reviewers Needed
Who should review this spec?
- [ ] Product Manager: [Name]
- [ ] Designer: [Name]
- [ ] Engineering Lead: [Name]
- [ ] QA Lead: [Name]
- [ ] System Admin: [Name]
- [ ] Other stakeholder: [Name, Role]

## Review Focus Areas
What should reviewers pay attention to?
- [ ] Technical accuracy
- [ ] Completeness (all sections filled)
- [ ] Consistency with related specs
- [ ] Feasibility of implementation
- [ ] UX implications
- [ ] Security/privacy considerations
- [ ] Data model correctness
- [ ] Other: [Specify]

## Review Deadline
When is review feedback needed?
- **Date**: [YYYY-MM-DD]
- **Reason**: [Why this deadline]

## Key Questions
Specific questions for reviewers:
1. [Question 1]
2. [Question 2]

## Related Specifications
Which related specs should reviewers also check?
- [Spec 1]
- [Spec 2]

## Next Steps
What happens after review?
- [ ] Incorporate feedback and mark as stable
- [ ] Schedule follow-up discussion
- [ ] Create enhancement issues for future work
- [ ] Other: [Explain]
```

**Labels**: `type: review-request`, `status: in-review`, `needs: [stakeholder-type]`

---

## Issue Labels

### Type Labels

| Label | Description | Use Case |
|-------|-------------|----------|
| `type: new-spec` | Request for a new specification document | When a topic needs its own dedicated spec |
| `type: enhancement` | Request to expand or improve existing spec | Adding detail, examples, or clarification |
| `type: correction` | Error or incorrect information in spec | Fixing mistakes, outdated info, or inaccuracies |
| `type: consistency` | Cross-document consistency issue | Conflicts or duplication across multiple specs |
| `type: missing-reference` | Missing or broken cross-references | Links, dependencies, or related docs missing |
| `type: source-integration` | Integrate materials from `00-ORIG-CONTEXT/` | Converting source materials to specs |
| `type: review-request` | Spec needs stakeholder review | Ready for review before marking stable |
| `type: maintenance` | Routine spec maintenance and cleanup | Formatting, organization, minor updates |

### Priority Labels

| Label | Description | SLA | Use Case |
|-------|-------------|-----|----------|
| `priority: P0` | Critical - Blocking work | Resolve within 1 day | Blocking active development or causing confusion |
| `priority: P1` | High - Needed this sprint | Resolve within 1 week | Needed for current sprint work |
| `priority: P2` | Medium - Needed soon | Resolve within 1 month | Needed for upcoming work or continuous improvement |
| `priority: P3` | Low - Nice to have | Resolve when convenient | Minor issues, non-blocking enhancements |

### Status Labels

| Label | Description | Meaning |
|-------|-------------|---------|
| `status: proposed` | Issue created, awaiting triage | New issue needs review and prioritization |
| `status: accepted` | Issue triaged and accepted for work | Team agrees this should be addressed |
| `status: in-progress` | Someone is actively working on this | Work has started |
| `status: in-review` | Changes made, awaiting review | PR open or draft ready for feedback |
| `status: blocked` | Cannot proceed due to dependency | Waiting on another issue, decision, or resource |
| `status: deferred` | Accepted but delayed to future sprint | Not urgent, will address later |
| `status: declined` | Issue will not be addressed | Out of scope, duplicate, or not valuable |
| `status: completed` | Issue resolved and closed | Work done and merged |

### Area Labels

| Label | Description | Maps To |
|-------|-------------|---------|
| `area: foundations` | Foundation specs (vision, glossary, etc.) | `00-foundations/` |
| `area: ux-design` | Design system and UX specs | `01-ux-design-system/` |
| `area: roles-permissions` | Authentication and authorization | `02-roles-and-permissions/` |
| `area: student-experience` | Student-facing features | `03-student-experience/` |
| `area: teacher-experience` | Teacher-facing features | `04-teacher-experience/` |
| `area: admin-experience` | Admin and subscriber features | `05-admin-subscriber-experience/` |
| `area: system-admin` | System administrator tools | `06-system-admin/` |
| `area: content-architecture` | Content models and structures | `07-content-architecture/` |
| `area: games-registry` | Games registry and authoring | `08-games-registry-and-authoring/` |
| `area: sequences-curriculum` | Sequences and curriculum tools | `09-sequences-and-curriculum-tools/` |
| `area: assignments-engine` | Assignment generation and tracking | `10-assignments-engine/` |
| `area: search-discovery` | Search and filtering | `11-search-and-discovery/` |
| `area: gamification` | Badges, points, challenges | `12-gamification/` |
| `area: messaging-notifications` | Communications and notifications | `13-messaging-and-notifications/` |
| `area: payments-subscriptions` | Billing and payments | `14-payments-and-subscriptions/` |
| `area: analytics-reporting` | Analytics, reports, and telemetry | `15-analytics-and-reporting/` |
| `area: accessibility` | Accessibility and inclusion | `16-accessibility-and-inclusion/` |
| `area: integrations` | Third-party integrations | `17-integrations/` |
| `area: architecture` | Technical architecture | `18-architecture/` |
| `area: quality-operations` | QA, testing, and operations | `19-quality-and-operations/` |
| `area: imports-automation` | Data imports and bulk operations | `20-imports-and-automation/` |
| `area: mobile-offline` | Mobile and offline support | `21-mobile-and-offline/` |
| `area: content-style` | Content style guides | `22-content-style-guides/` |
| `area: legal-compliance` | Legal and compliance | `23-legal-and-compliance/` |
| `area: roadmap-phasing` | Roadmap and phasing | `24-roadmap-and-phasing/` |
| `area: research-inspiration` | Research and competitive analysis | `25-research-and-inspiration/` |
| `area: repo-operations` | Repository operations and maintenance | `26-repo-operations/` |
| `area: social-learning` | Social learning features | `27-social-learning/` |
| `area: advanced-assessment` | Advanced assessment features | `28-advanced-assessment/` |
| `area: content-management` | Content management workflows | `29-content-management/` |
| `area: ai-ml` | AI and machine learning features | `30-ai-and-machine-learning/` |
| `area: cross-cutting` | Affects multiple areas | Use when issue spans multiple directories |

### Special Labels

| Label | Description | Use Case |
|-------|-------------|----------|
| `needs: discussion` | Requires team discussion before proceeding | Unclear requirements or multiple approaches |
| `needs: stakeholder-input` | Needs input from product owner or executive | Business decisions or strategic direction |
| `needs: design` | Needs designer review or mockups | UX implications or design decisions |
| `needs: technical-review` | Needs engineering review | Technical feasibility or architecture decisions |
| `good-first-issue` | Good for new team members | Simple, well-defined spec enhancements |
| `documentation` | General documentation work | Meta work about the specs themselves |
| `duplicate` | Duplicate of another issue | Mark and reference original issue |
| `question` | Question about a spec | Seeking clarification |

---

## Workflow and Process

### Creating Issues

1. **Choose the Right Template**: Select the template that best matches your need
2. **Fill Out All Required Fields**: Don't skip sections; "N/A" is better than blank
3. **Assign Labels**: Add appropriate type, priority, area, and status labels
4. **Link Related Issues**: Reference related issues and specs
5. **Add to Project Board**: Assign to appropriate sprint or backlog

### Triaging Issues

**Triage Frequency**: Weekly (or as needed for P0 issues)

**Triage Process**:
1. **Review New Issues**: Check `status: proposed` issues
2. **Validate Completeness**: Ensure all required fields are filled
3. **Assign Priority**: Add appropriate priority label
4. **Assign Area**: Add area label(s)
5. **Assign Owner**: Identify who should address this (PM, Designer, Engineer, etc.)
6. **Update Status**: Change to `status: accepted` or `status: declined`
7. **Add to Sprint**: If P0 or P1, add to current or next sprint

### Working on Issues

1. **Update Status**: Change to `status: in-progress` when starting work
2. **Reference Issue in Commits**: Include issue number in commit messages
3. **Cross-Check Dependencies**: Review related specs for consistency
4. **Update Related Specs**: If changes affect other specs, update them too
5. **Request Review**: When complete, change status to `status: in-review`
6. **Address Feedback**: Incorporate reviewer comments
7. **Close Issue**: Mark `status: completed` when merged

### Review Process

**Review Criteria**:
- All sections filled with meaningful content
- Consistent with related specifications
- Accurate and implementable
- References and cross-links are correct
- "Supporting Documents Referenced" section added (if applicable)
- Proper markdown formatting

**Reviewer Responsibilities**:
- Product Manager: Business requirements, terminology, completeness
- Designer: UX implications, design system consistency
- Engineer: Technical accuracy, feasibility, data models
- QA: Testability, edge cases, validation rules

---

## Behavior and Rules

### Issue Lifecycle Rules

#### Creation Rules
- **One Issue, One Topic**: Don't combine multiple unrelated changes in a single issue
- **Search First**: Check for existing issues before creating a new one
- **Use Templates**: Always use the appropriate template; don't create blank issues
- **Complete Required Fields**: All fields marked "required" must be filled

#### Priority Assignment Rules
- **P0**: Reserved for blocking issues that prevent current work from proceeding
- **P1**: For issues needed within the current sprint or next sprint
- **P2**: For continuous improvement items needed within 1-2 months
- **P3**: For nice-to-have improvements with no timeline pressure

**Priority Escalation**: Issues can be escalated (e.g., P2 → P1) if circumstances change. Document reason for escalation in issue comments.

#### Status Transition Rules

Valid transitions:
```
proposed → accepted → in-progress → in-review → completed
proposed → declined
accepted → deferred
in-progress → blocked → in-progress
blocked → declined
```

Invalid transitions (must justify in comments):
- `proposed` → `completed` (skips review)
- `in-review` → `declined` (should be `completed` or re-opened)

#### Assignment Rules
- **Single Owner**: Each issue should have one assigned owner (person responsible for completion)
- **Multiple Reviewers**: Issues can have multiple reviewers
- **Self-Assignment**: Team members can self-assign issues marked `good-first-issue`
- **Reassignment**: If owner can't complete, reassign to another team member (document reason)

### Quality Standards for Issue Resolution

#### For New Specifications
- All standard spec sections completed (Purpose, Scope, Dependencies, etc.)
- At least 3 cross-references to related specs
- "Supporting Documents Referenced" section if source materials were used
- Telemetry section if user-facing feature
- Permissions section if access control involved

#### For Enhancements
- New content clearly adds value (not just restating existing content)
- Cross-references added where appropriate
- Related specs checked for consistency
- "Supporting Documents Referenced" updated if new sources used

#### For Corrections
- Original error clearly documented in issue
- Correction verified against source materials or stakeholder input
- Related specs checked for same error
- Changelog or comment added to explain correction

#### For Consistency Issues
- All affected specs updated together (not left partially fixed)
- Consistent terminology used across all updates
- Cross-references added to link related specs
- Comment added to each spec noting the consistency fix

---

## Permissions

### Issue Management Permissions

| Role | Can Create | Can Triage | Can Assign | Can Close | Can Delete |
|------|------------|------------|------------|-----------|------------|
| **Product Manager** | Yes | Yes | Yes | Yes | Yes (own issues) |
| **Designer** | Yes | Yes (design area) | Yes | Yes (own issues) | Yes (own issues) |
| **Engineer** | Yes | Yes (technical area) | Yes | Yes (own issues) | Yes (own issues) |
| **QA** | Yes | Yes (quality area) | Yes | Yes (own issues) | Yes (own issues) |
| **Stakeholder (Read-Only)** | Yes | No | No | No | No |

**Issue Ownership**: The person who creates an issue is considered the "reporter". The person assigned to the issue is the "owner" responsible for resolution.

### Label Permissions

| Role | Can Create Labels | Can Edit Labels | Can Apply Labels | Can Remove Labels |
|------|-------------------|-----------------|------------------|-------------------|
| **Repository Admin** | Yes | Yes | Yes | Yes |
| **Product Manager** | Yes | Yes | Yes | Yes |
| **Team Member** | No | No | Yes | Yes (own issues) |

### Triage Permissions

**Who Can Triage**:
- Product Managers (all areas)
- Technical Leads (technical areas)
- Design Leads (design areas)

**Triage Authority**:
- Can change `status: proposed` to `status: accepted` or `status: declined`
- Can assign priority labels
- Can assign area labels
- Can assign issues to team members

---

## Dependencies

### Internal Dependencies

- **All Specification Documents**: Issues reference and affect individual specs across all directories
- **Docs Index** ([docs-index.md](../00-foundations/docs-index.md)): Issues may require updates to the index when new specs are added
- **Glossary** ([glossary.md](../00-foundations/glossary.md)): Issues may involve terminology that needs glossary updates
- **Information Architecture** ([information-architecture.md](../00-foundations/information-architecture.md)): New specs must fit within the established IA

### External Dependencies

- **GitHub Issues**: If this repo uses GitHub, these templates should be implemented as GitHub Issue Templates (`.github/ISSUE_TEMPLATE/` directory)
- **Project Management Tools**: Labels and workflow states should sync with project management tools (e.g., Jira, Linear, GitHub Projects)
- **Source Materials**: `00-ORIG-CONTEXT/` directory contains source materials referenced in integration issues

---

## Open Questions

### Process Questions
- **Automation**: Should we automate issue creation for certain events (e.g., auto-create consistency check issues when related specs are updated)?
- **Integration**: Should issues automatically create checklist items in a project management tool?
- **Notifications**: What notification rules should be configured for issue assignments, mentions, and status changes?

### Template Questions
- **Additional Templates**: Do we need templates for other scenarios (e.g., "Spec Deprecation", "Spec Consolidation", "Terminology Update")?
- **Template Enforcement**: Should we lock down issue creation to require template use (prevent blank issues)?
- **Custom Fields**: Would custom fields (beyond labels) help with tracking (e.g., "Affected Stakeholders", "Implementation Phase")?

### Workflow Questions
- **Review SLA**: What should the expected turnaround time be for spec reviews?
- **Escalation Process**: What's the escalation path if high-priority issues aren't being addressed?
- **Metrics**: Should we track issue resolution metrics (average time to close by type, priority distribution, etc.)?

### Organizational Questions
- **Issue Ownership**: Should issues be assigned based on area (e.g., all `area: payments-subscriptions` → Finance PM) or by availability?
- **Cross-Functional Issues**: Who owns issues that span multiple areas (e.g., consistency issues across 5+ specs)?
- **External Stakeholders**: Can non-team members (e.g., contractors, advisors) create issues? If so, with what permissions?

---

## Related Documentation

- [Docs Index](../00-foundations/docs-index.md) - Complete index of all specification documents
- [Information Architecture](../00-foundations/information-architecture.md) - How specs are organized
- [Glossary](../00-foundations/glossary.md) - Terminology used across specs
- [Release Management](../19-quality-and-operations/release-management.md) - How product releases are managed (different from spec updates)
