# Competency-Based Assessment

## Overview
Competency-based assessment and portfolio management specifications for skill-based learning evaluation.

## Purpose

While [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md) measures advancement through structured curricula, competency-based assessment focuses on demonstrating mastery of discrete musical skills regardless of the learning path taken. This specification enables:

- **Standards-Based Progression**: Students advance when they demonstrate competency, not based on seat time or assignment completion
- **Flexible Learning Paths**: Multiple routes to demonstrating the same competency
- **Evidence-Based Mastery**: Portfolio of artifacts proving skill achievement
- **Credential Management**: Formal recognition of competencies through badges, certificates, and external alignment
- **Gap Analysis**: Identifying specific skill deficiencies and recommending targeted interventions
- **Transfer Credit**: Recognizing competencies earned outside the MLC system

This system complements the existing assignment-based progression model by providing an alternative assessment framework aligned with competency-based education (CBE) principles and music education standards (National Core Arts Standards, MENC).

## Scope

### Included
- **Competency Framework**: Hierarchical skill taxonomy aligned with music education standards
- **Evidence Collection**: Portfolio management for capturing proof of competency achievement
- **Skill Mapping**: Relationship between learning elements (games, sequences) and competencies
- **Competency Tracking**: Progress monitoring at skill level (independent of assignments)
- **Adaptive Assessment**: Diagnostic tools for identifying competency gaps and strengths
- **Certification Management**: Issuing formal credentials for competency achievement
- **Standards Alignment**: Mapping competencies to external standards (MENC, Piano Adventures, RCM)
- **Transfer Credit**: Evaluating and recognizing prior learning and external certifications
- **Competency-Based Reporting**: Progress reports organized by skill domains rather than assignments

### Excluded
- **Assignment-based progress** (covered in [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))
- **General game scoring mechanics** (covered in [Game Model](../08-games-registry-and-authoring/game-model.md))
- **Placement testing workflows** (covered in [Sequence Placements and EVAL](../09-sequences-and-curriculum-tools/sequence-placements-eval.md))
- **Badge UI and reward mechanics** (covered in [Badges System](../12-gamification/badges-system.md))
- **Teacher reporting dashboards** (covered in [Teacher Reports](../04-teacher-experience/teacher-reports.md))

## Description
Detailed competency-based assessment framework including skill mapping, portfolio management, competency tracking, adaptive assessment algorithms, and certification management for skill-based learning outcomes.

## Key Concepts

### Competency vs. Progress
- **Assignment Progress**: Measures completion of structured learning sequences (e.g., "75% through Piano Adventures Level 2A")
- **Competency Achievement**: Measures demonstration of specific skills (e.g., "Can accurately identify all intervals from 2nds through octaves")

**Example**: A student might complete 50% of a rhythm assignment but already demonstrate full competency in "identifying quarter and half note values"—in a competency-based model, they would receive credit and advance, potentially skipping redundant content.

### Competency Hierarchy

The MLC competency framework organizes skills into a four-level hierarchy:

1. **Domains**: Broad skill categories (e.g., Rhythm & Meter, Pitch & Melody)
2. **Competencies**: Specific, measurable skills (e.g., "Identify note values and rests")
3. **Sub-competencies**: Granular skill components (e.g., "Identify quarter notes and rests")
4. **Evidence Requirements**: Specific demonstrations needed to prove mastery

### Standards Alignment

All competencies map to one or more external standards:
- **National Core Arts Standards** (NCAS) for music education
- **MENC National Standards** (K-4, 5-8, 9-12)
- **Piano method curricula** (Piano Adventures, Alfred, Hal Leonard)
- **Royal Conservatory of Music** (RCM) examination requirements
- **State music education standards** (where applicable)

### Portfolio Evidence

Competency achievement requires documented evidence:
- **Performance recordings**: Audio/video of student playing
- **Assessment scores**: Passing scores on diagnostic games
- **Teacher observations**: Structured rubric evaluations
- **Project artifacts**: Compositions, arrangements, analyses
- **External credentials**: Certificates from recitals, competitions, RCM exams

## Data Model

### Competency Definition

```typescript
interface Competency {
  competency_id: string;              // Unique identifier (e.g., "COMP-RHYTM-0025")
  code: string;                       // Short code (e.g., "RHYTM-NOTEVALS-QH")
  name: string;                       // Display name
  description: string;                // Full description of the skill
  
  // Hierarchy
  domain_id: string;                  // Parent domain (e.g., "RHYTM")
  parent_competency_id?: string;      // Parent competency (if sub-competency)
  sub_competencies: string[];         // Child competencies
  
  // Standards alignment
  standards_alignment: {
    standard_type: 'NCAS' | 'MENC' | 'RCM' | 'METHOD' | 'STATE';
    standard_id: string;
    alignment_notes?: string;
  }[];
  
  // Difficulty and sequencing
  level: 'primary' | '1' | '2' | '3' | '4' | '5' | '6';
  difficulty_score: number;           // 0-100 for adaptive sequencing
  prerequisite_competencies: string[]; // Must achieve before this
  
  // Evidence requirements
  evidence_requirements: EvidenceRequirement[];
  min_evidence_count: number;         // Minimum pieces of evidence needed
  
  // Metadata
  skill_types: ('aural' | 'visual' | 'kinesthetic' | 'theoretical')[];
  concept_tags: string[];             // Related musical concepts
  active: boolean;                    // Whether currently assessed
  version: number;                    // For competency evolution
}

interface EvidenceRequirement {
  evidence_type: 'game_score' | 'performance' | 'teacher_observation' | 'project' | 'external_credential';
  requirement_details: {
    // For game_score
    game_ids?: string[];              // Which games can provide evidence
    min_score?: number;               // Minimum passing score
    required_consistency?: boolean;   // Must pass multiple times
    
    // For performance
    rubric_id?: string;               // Which rubric to use
    min_rubric_score?: number;        // Minimum score on rubric
    
    // For teacher_observation
    observation_criteria?: string[];  // What teacher must observe
    
    // For project
    project_type?: string;            // Type of project (composition, analysis, etc.)
    project_requirements?: string[];  // Specific requirements
    
    // For external_credential
    credential_types?: string[];      // Accepted external credentials
  };
  weight: number;                     // How much this evidence counts (0-1)
}
```

### Competency Domain

```typescript
interface CompetencyDomain {
  domain_id: string;                  // Unique identifier
  code: string;                       // Short code (e.g., "RHYTM", "PITCH")
  name: string;                       // Display name
  description: string;
  
  // Visual identity
  icon: string;                       // Icon identifier
  color: string;                      // Theme color for UI
  
  // Standards
  standards_alignment: {
    standard_type: string;
    standard_section: string;
  }[];
  
  // Ordering
  sequence_order: number;             // Display order in lists
  
  // Competencies in this domain
  competency_ids: string[];           // All competencies in this domain
}
```

### Student Competency Achievement

```typescript
interface StudentCompetencyAchievement {
  achievement_id: string;
  student_id: string;
  competency_id: string;
  organization_id: string;
  
  // Achievement status
  status: 'not_started' | 'in_progress' | 'achieved' | 'certified' | 'expired';
  achievement_date?: Date;            // When competency was first achieved
  certification_date?: Date;          // When formally certified
  expiration_date?: Date;             // If competency expires (e.g., time-sensitive skills)
  
  // Evidence collected
  evidence_items: EvidenceItem[];
  evidence_score: number;             // 0-100, how strong is the evidence
  confidence_level: 'low' | 'moderate' | 'high' | 'verified';
  
  // Performance metrics
  first_evidence_date?: Date;         // When student first attempted this competency
  time_to_achieve_ms?: number;        // Time from first attempt to achievement
  attempts_required: number;          // How many attempts needed
  
  // Maintenance tracking
  last_demonstrated_date?: Date;      // Most recent evidence
  maintenance_status: 'current' | 'needs_review' | 'needs_reassessment';
  
  // Teacher involvement
  teacher_verified: boolean;
  verifying_teacher_id?: string;
  verification_notes?: string;
  
  // Certification
  certificate_issued: boolean;
  certificate_id?: string;
  certificate_url?: string;
}

interface EvidenceItem {
  evidence_id: string;
  evidence_type: 'game_score' | 'performance' | 'teacher_observation' | 'project' | 'external_credential';
  collected_date: Date;
  
  // Evidence details
  source_id: string;                  // ID of game, assignment, project, etc.
  source_context: {
    // For game_score
    game_id?: string;
    stage?: string;
    score?: number;
    attempt_number?: number;
    
    // For performance
    recording_url?: string;
    rubric_scores?: Record<string, number>;
    teacher_comments?: string;
    
    // For teacher_observation
    observation_date?: Date;
    observation_criteria_met?: string[];
    rubric_score?: number;
    
    // For project
    project_url?: string;
    project_description?: string;
    project_type?: string;
    
    // For external_credential
    credential_type?: string;
    issuing_organization?: string;
    credential_url?: string;
    credential_date?: Date;
  };
  
  // Quality indicators
  confidence_score: number;           // 0-100, how strong is this evidence
  weight: number;                     // How much this contributes to achievement
  
  // Validation
  validated: boolean;
  validated_by?: string;              // Teacher or admin who validated
  validated_date?: Date;
}
```

### Competency Portfolio

```typescript
interface CompetencyPortfolio {
  portfolio_id: string;
  student_id: string;
  organization_id: string;
  
  // Portfolio metadata
  created_date: Date;
  last_updated: Date;
  portfolio_status: 'active' | 'archived' | 'exported';
  
  // Achievement summary
  total_competencies_available: number;
  competencies_achieved: number;
  competencies_in_progress: number;
  competencies_not_started: number;
  
  // Domain breakdown
  domain_summary: {
    domain_id: string;
    competencies_in_domain: number;
    achieved_in_domain: number;
    proficiency_level: number;        // 0-100
  }[];
  
  // Evidence summary
  total_evidence_items: number;
  evidence_by_type: Record<string, number>;
  
  // Performance indicators
  average_time_to_competency_ms: number;
  average_attempts_per_competency: number;
  teacher_verification_rate: number;  // % of competencies teacher-verified
  
  // Credentials
  certificates_earned: string[];
  external_credentials: string[];
  
  // Sharing and export
  public_url?: string;                // If portfolio is shared publicly
  export_format_preference?: string;  // PDF, JSON, etc.
  last_exported_date?: Date;
}
```

### Competency Map (Learning Element → Competency)

```typescript
interface CompetencyMap {
  map_id: string;
  
  // What provides evidence
  source_type: 'game' | 'sequence' | 'assignment' | 'project';
  source_id: string;
  
  // What competencies it addresses
  competency_mappings: {
    competency_id: string;
    evidence_type: string;
    evidence_strength: 'weak' | 'moderate' | 'strong';
    evidence_weight: number;          // 0-1
    notes?: string;
  }[];
  
  // Metadata
  created_by: string;                 // Who created this mapping
  verified: boolean;
  last_reviewed_date?: Date;
}
```

## Behavior and Rules

### Competency Achievement Criteria

A competency is marked as "achieved" when:

1. **Sufficient Evidence**: Minimum number of evidence items collected
2. **Evidence Quality**: Combined evidence score meets threshold (typically 75+)
3. **Confidence Level**: Evidence confidence is at least "moderate"
4. **Teacher Verification** (optional): Teacher has reviewed and approved (if required by organization)
5. **Prerequisite Competencies**: All prerequisite competencies are achieved

**Achievement Calculation**:
```pseudo
achievement_score = sum(evidence_weight * evidence_confidence) / sum(evidence_weight)

if (achievement_score >= 75 
    AND evidence_count >= min_evidence_count
    AND confidence_level >= 'moderate'
    AND all_prerequisites_met) {
  status = 'achieved'
}
```

### Evidence Weighting and Confidence

Different evidence types carry different weights and confidence levels:

**Evidence Type Weights** (default):
- **Game scores**: 0.4 (reliable but limited context)
- **Performance recordings**: 0.8 (rich evidence but interpretation needed)
- **Teacher observations**: 0.9 (expert evaluation but subjective)
- **Projects**: 0.7 (demonstrates application but variable quality)
- **External credentials**: 1.0 (verified by third party)

**Confidence Levels**:
- **Low** (0-59%): Single piece of weak evidence or inconsistent performance
- **Moderate** (60-79%): Multiple pieces of evidence with some consistency
- **High** (80-94%): Strong, consistent evidence across multiple contexts
- **Verified** (95-100%): Teacher-verified and/or externally credentialed

### Competency Maintenance and Decay

Some competencies require ongoing practice to maintain:

**Maintenance Rules**:
1. **Static Competencies**: Theoretical knowledge (e.g., "Identify treble clef notes") - no decay
2. **Performance Competencies**: Skills requiring practice (e.g., "Play C Major scale") - decay after 90 days without evidence
3. **Maintenance Assessment**: If competency shows decay indicators, status changes to "needs_review"
4. **Re-assessment**: Student must provide new evidence to restore "achieved" status

**Decay Indicators**:
- No evidence of competency use in last 90 days
- Significant struggles on related competencies that depend on this one
- Teacher flags competency for re-assessment

### Adaptive Competency Assessment

The system can generate personalized assessment sequences to efficiently evaluate competency achievement:

**Adaptive Assessment Algorithm**:
1. **Start with diagnostic**: Administer broad assessment covering multiple competencies
2. **Identify strong areas**: Competencies where student shows immediate mastery
3. **Identify weak areas**: Competencies where student struggles
4. **Narrow focus**: Administer targeted assessments for unclear competencies
5. **Confirm boundaries**: Test edge cases to precisely identify competency ceiling
6. **Generate report**: Competency profile showing achieved, in-progress, and not-yet-attempted

This is similar to the EVAL sequence but focused on competency demonstration rather than placement.

### Competency-Based Progression

Organizations can enable competency-based progression where students:

1. **Advance by competency**: Move forward when competency is achieved, regardless of assignment completion
2. **Skip redundant content**: If a student demonstrates a competency, related games can be marked optional
3. **Fill gaps**: If assessment reveals missing competencies, targeted content is automatically assigned
4. **Set own pace**: Students work at their own speed, advancing when ready

**Progression Rules**:
```pseudo
if (competency_achieved) {
  // Skip future content that only addresses this competency
  mark_redundant_steps_optional(competency_id)
  
  // Unlock content that required this competency
  unlock_dependent_content(competency_id)
  
  // Check if sequence milestone reached
  if (all_required_competencies_for_level_achieved) {
    advance_to_next_level()
  }
}
```

### Certification Management

When a defined set of competencies is achieved, the system can issue formal certificates:

**Certificate Types**:
1. **Domain Certificates**: All competencies in a domain achieved (e.g., "Rhythm & Meter Mastery")
2. **Level Certificates**: All required competencies for a level achieved (e.g., "Level 2 Completion")
3. **Method Certificates**: Competencies aligned with external method completed (e.g., "Piano Adventures 2A")
4. **Custom Certificates**: Teacher-defined competency sets

**Certification Process**:
1. **Auto-detection**: System identifies when certificate criteria are met
2. **Teacher notification**: Teacher receives notification for review
3. **Teacher approval**: Teacher reviews evidence and approves certificate (or requests more evidence)
4. **Certificate generation**: System generates PDF certificate with student name, competencies, date
5. **Portfolio inclusion**: Certificate added to student's competency portfolio
6. **Public sharing**: Student can share certificate via public URL or download

### Gap Analysis and Recommendations

The system continuously analyzes competency gaps and recommends interventions:

**Gap Analysis**:
- **Identify missing competencies**: Compare student's achieved competencies to expected for their level
- **Identify weak competencies**: Competencies with low evidence scores or inconsistent performance
- **Identify prerequisite gaps**: Missing foundational competencies blocking advancement
- **Identify decay**: Previously achieved competencies showing maintenance concerns

**Recommendations**:
- **Targeted assignments**: Specific games/sequences that address identified gaps
- **Remediation sequences**: Custom learning paths for struggling competencies
- **Practice schedules**: Spaced repetition for competency maintenance
- **Teacher interventions**: Flag students needing one-on-one instruction

## UX Requirements

### Student Competency Dashboard

**Competency Overview**:
- Visual competency map showing domains and achievement status
- Progress indicators for each domain (e.g., "Rhythm: 12/18 competencies achieved")
- Recently achieved competencies with celebration animations
- Next competencies to work on with recommendations

**Competency Detail View**:
- Competency name and description
- Why this competency matters (real-world application)
- Current status and evidence collected
- What's needed to achieve this competency
- Recommended games/activities to build this skill
- Related competencies (prerequisites, dependents)

**Portfolio View**:
- Grid or timeline view of all competencies
- Filter by domain, level, status
- Evidence artifacts (recordings, scores, projects)
- Certificates earned
- Share portfolio button (generates public URL)

### Teacher Competency Tools

**Class Competency Overview**:
- Heatmap: students (rows) x competencies (columns), colored by achievement
- Filter by domain, level, or specific competencies
- Identify class-wide gaps (competencies where many students struggle)
- Export competency reports for students or class

**Student Competency Detail**:
- Full competency portfolio view
- Evidence review and validation tools
- Override/manual competency marking (with justification)
- Add teacher observation evidence
- Issue certificates
- Generate parent-friendly competency report

**Competency Mapping Tools**:
- View which games/sequences map to which competencies
- Add new competency mappings for custom content
- Review and validate AI-suggested mappings
- Adjust evidence weights and requirements

### Admin Competency Configuration

**Competency Framework Management**:
- CRUD operations for competencies and domains
- Standards alignment management
- Evidence requirement configuration
- Certificate template management

**Competency Analytics**:
- Which competencies are most difficult (lowest achievement rates)
- Which competencies have insufficient evidence sources (need more mapped games)
- Time-to-competency averages for benchmarking
- Competency achievement trends over time

## Telemetry

### Competency Events

```typescript
// When student provides evidence for a competency
event: 'competency_evidence_collected'
properties: {
  student_id: string
  competency_id: string
  evidence_type: string
  evidence_id: string
  evidence_score: number
  confidence_level: string
  source_id: string
  source_type: string
}

// When competency status changes
event: 'competency_status_changed'
properties: {
  student_id: string
  competency_id: string
  previous_status: string
  new_status: string
  achievement_score: number
  evidence_count: number
  confidence_level: string
  teacher_verified: boolean
}

// When certificate is issued
event: 'competency_certificate_issued'
properties: {
  student_id: string
  certificate_id: string
  certificate_type: string
  competencies_included: string[]
  issued_by: string
  issued_date: Date
}

// When competency gap is identified
event: 'competency_gap_detected'
properties: {
  student_id: string
  competency_id: string
  gap_type: 'missing' | 'weak' | 'prerequisite' | 'decay'
  gap_severity: 'low' | 'moderate' | 'high'
  recommended_intervention: string
}

// When teacher verifies competency
event: 'competency_teacher_verified'
properties: {
  student_id: string
  competency_id: string
  teacher_id: string
  verification_method: 'observation' | 'performance_review' | 'project_review'
  confidence_level: string
  notes?: string
}
```

### Sampling
- Competency status changes: **100%** (never sampled, critical for student records)
- Evidence collection: **100%** (audit trail required)
- Certificate issuance: **100%** (important milestones)
- Gap detection: **100%** (drives interventions)
- Teacher verification: **100%** (quality assurance)

## Permissions

### Student Access
- **Read**: Can view own competency portfolio and achievement status
- **Evidence submission**: Can submit project artifacts and external credentials for review
- **Portfolio sharing**: Can generate and share public portfolio URL (if organization allows)

### Teacher Access
- **Read**: Can view competency portfolios for assigned students
- **Evidence validation**: Can approve/reject submitted evidence
- **Teacher observations**: Can add teacher observation evidence
- **Manual override**: Can mark competencies as achieved with justification
- **Certificate issuance**: Can approve and issue certificates
- **Competency mapping**: Can create/edit competency mappings for custom content (if permitted)

### Admin Access
- **Read**: Can view all competency data for their organization
- **Configuration**: Can configure competency framework and evidence requirements
- **Certificate management**: Can create certificate templates and approve issuance
- **Analytics**: Can access organization-wide competency analytics
- **Competency framework editing**: Can add/modify competencies and domains (if permitted by system admin)

### System Admin Access
- **Full access**: Can manage master competency framework
- **Standards alignment**: Can update standards mappings
- **Global analytics**: Can view competency data across all organizations
- **Framework versioning**: Can publish new versions of competency framework

## Supporting Documents Referenced

This competency-based assessment specification draws from the following source documents:

- [Level1CurriculumMap.xls - Sheet1.csv](../00-ORIG-CONTEXT/Level1CurriculumMap.xls%20-%20Sheet1.csv) — Comprehensive skill taxonomy including aural, visual, writing, and performing skills with game stages and MENC standards alignment
- [Help Eval Sequence.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Help%20Sequence/Help%20Eval%20Sequence.xlsx%20-%20Sheet1.csv) — Evaluation curriculum structure with skill domain assessment across Primary through Level 5
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence specifications with competency frameworks including domains, levels, and skill classifications
- [Assignment Play Re-design.docx.txt](../00-ORIG-CONTEXT/Assignment%20Play%20Re-design.docx.txt) — Score tracking and progress visibility requirements for competency evidence collection

## Dependencies

### Internal Dependencies
- **[Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md)**: Competency evidence often comes from assignment completion
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Game scores are primary evidence source
- **[Sequence Placements and EVAL](../09-sequences-and-curriculum-tools/sequence-placements-eval.md)**: EVAL can serve as initial competency diagnostic
- **[Badges System](../12-gamification/badges-system.md)**: Badges can be tied to competency achievement
- **[Teacher Reports](../04-teacher-experience/teacher-reports.md)**: Competency data feeds into teacher dashboards
- **[Student Profile](../03-student-experience/student-profile.md)**: Competency achievements displayed in profile
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Telemetry infrastructure

### External Dependencies
- **Standards databases**: NCAS, MENC, RCM standards for alignment
- **PDF generation**: For certificate creation
- **Cloud storage**: For portfolio artifacts (recordings, projects)
- **Public sharing service**: For portfolio URLs

## Open Questions

### Framework Design
- Should competencies be granular (many small skills) or broader (fewer comprehensive skills)?
- How often should the competency framework be updated to reflect evolving music education standards?
- Should organizations be able to customize the core competency framework or only add extensions?

### Evidence and Assessment
- What is the optimal number of evidence items required per competency (balance thoroughness vs. burden)?
- Should game scores count equally across all games, or should certain games carry more weight?
- How should the system handle subjective teacher observations vs. objective game scores?
- Should students be able to challenge a "not achieved" determination and request re-assessment?

### Competency Progression
- Should competency-based progression be optional or required for all organizations?
- If a student achieves a competency through free play, should they still complete assigned content for that competency?
- How should the system balance structured sequences (linear) with competency-based progression (non-linear)?

### Certification and Credentials
- Who should have authority to issue certificates: only teachers, or also system-admins based on data?
- Should certificates expire, and if so, under what conditions?
- How should the system integrate with external credentialing systems (RCM, ABRSM, etc.)?
- What information should be included on certificates to make them meaningful and verifiable?

### Portfolio Management
- What file size limits should be imposed on portfolio artifacts (recordings, projects)?
- Should portfolios be exportable in standardized formats (e.g., IMS ePortfolio, Open Badges)?
- How long should portfolio evidence be retained after student account deactivation?
- Should there be privacy controls on what evidence is visible in shared portfolios?

### Maintenance and Decay
- Which competencies should be subject to maintenance requirements vs. considered permanent?
- What is the appropriate decay timeline (currently 90 days)?
- Should maintenance requirements vary by competency difficulty or student level?

### Standards Alignment
- How should the system handle conflicting or evolving standards from different organizations?
- Should competency mappings to standards be visible to students/parents, or only teachers?
- How can the system support international standards (e.g., ABRSM for UK, AMEB for Australia)?
