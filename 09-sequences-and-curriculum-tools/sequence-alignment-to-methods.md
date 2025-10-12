# Sequence Alignment to Methods

## Purpose
Define the standards, framework, and workflow for aligning MLC learning sequences to third-party method books and curriculum series (e.g., Piano Adventures, Hal Leonard Student Piano Library, Alfred's Premier Piano Course). This spec enables teachers who use published method books to seamlessly integrate MLC digital games and assignments that reinforce the concepts taught in their chosen curriculum, maintaining pedagogical continuity and providing precise page-level correlations.

This is a **general framework** document. For a detailed implementation example, see [Piano Adventures Alignment](../07-content-architecture/piano-adventures-alignment.md).

## Scope

### Included
- **Standardized metadata schema** for method-aligned sequences
- **Publisher identification** and book series cataloging requirements
- **Page correlation specifications** (unit numbers, page ranges, lesson titles)
- **Skill mapping taxonomy** for linking method book concepts to MLC games
- **Progression marker standards** for milestones and assessments
- **Validation rules** for ensuring alignment accuracy and completeness
- **Quality assurance workflows** for maintaining alignment accuracy across editions
- **Import/export patterns** for method book content integration
- **Relationship between method-aligned sequences and first-party sequences** (LIFE, SOLF, EVAL)
- **Cross-publisher alignment standards** to ensure consistency

### Excluded
- **Specific method book implementations** (see Piano Adventures as example; other method alignments follow same pattern)
- **Sequence data model details** (see [Sequence Model](sequence-model.md) for core structure)
- **Authoring UI and workflows** (see [Sequence Authoring](sequence-authoring.md) for creation tools)
- **Assignment generation logic** (see [Assignment Generation](../10-assignments-engine/assignment-generation.md))
- **Copyright licensing agreements** with publishers (business operations, not technical spec)
- **Game content creation** (games must already exist in registry before alignment)

## Method alignment fundamentals

### What is method alignment?
Method alignment is the process of mapping MLC's digital learning games to specific pages, units, and concepts within a published method book series. The alignment creates a **bridge** between the physical book curriculum and MLC's interactive learning platform, allowing teachers to assign MLC games that directly support what students are learning in their method books.

### Why method alignment matters
1. **Pedagogical continuity**: Teachers can maintain their preferred teaching method while leveraging MLC's interactive practice tools
2. **Precise reinforcement**: Games are mapped to specific pages, ensuring students practice exactly what they're learning
3. **Adoption acceleration**: Schools and teachers already invested in a method book can adopt MLC without curriculum disruption
4. **Market expansion**: Each method alignment opens MLC to the user base of that method book series
5. **Differentiated instruction**: Teachers can supplement method book lessons with adaptive practice tailored to student needs

### Relationship to first-party sequences
MLC maintains both **first-party sequences** (LIFE, SOLF, EVAL) and **method-aligned sequences**:

- **First-party sequences** represent MLC's proprietary curriculum design and pedagogical approach
- **Method-aligned sequences** adapt MLC content to match external curriculum structures
- Both use the same underlying [Sequence Model](sequence-model.md) data structure
- Method-aligned sequences reference the same game registry as first-party sequences
- Teachers can assign both types simultaneously (e.g., LIFE for comprehensive learning + method alignment for book-specific support)

## Data model (if applicable)

The method-aligned sequence structure extends the core [Sequence Model](sequence-model.md) with specialized metadata fields.

### Method alignment metadata schema

#### Required fields
All method-aligned sequences MUST include these fields to ensure compatibility and searchability:

```json
{
  "sequence_id": "SEQ-FABR-PRIMER",
  "sequence_version": 1,
  "name": "Piano Adventures - Primer Level",
  "visibility": "method_aligned",
  "method_metadata": {
    "publisher": "Faber and Faber",
    "series_name": "Piano Adventures",
    "book_level": "Primer",
    "edition_year": 2015,
    "isbn": "978-1616770181",
    "correlation_date": "2024-01-15",
    "maintained_by": "content_team"
  }
}
```

**Field definitions**:
- `publisher` (string): Official publisher name (e.g., "Faber and Faber", "Hal Leonard", "Alfred Music")
- `series_name` (string): Method book series name (e.g., "Piano Adventures", "Student Piano Library")
- `book_level` (string): Specific level/grade within series (e.g., "Primer", "Book 1", "Level 2A")
- `edition_year` (integer): Publication year of the edition being aligned
- `isbn` (string): ISBN-13 for precise edition identification
- `correlation_date` (date): When this alignment was created or last validated
- `maintained_by` (string): Team or individual responsible for updates

#### Group-level metadata
Each Group (representing a unit or chapter in the method book) includes:

```json
{
  "group_id": "G-UNIT-01",
  "name": "Unit 1: Introduction to Keyboard",
  "group_metadata": {
    "unit_number": 1,
    "page_range": "4-11",
    "estimated_weeks": 2,
    "milestone": "Keyboard geography and hand position"
  }
}
```

#### Assignment-level metadata
Each Assignment (representing specific lesson content) includes:

```json
{
  "assignment_id": "A-UNIT-01-LESSON-01",
  "name": "Lesson 1: Middle C Position",
  "assignment_metadata": {
    "page_range": "4-6",
    "lesson_title": "Middle C Position",
    "concepts_taught": ["middle_c", "hand_position", "finger_numbers"],
    "prerequisite_skills": [],
    "assessment_point": false
  }
}
```

#### Step-level metadata
Each Step (linking to a specific game) includes:

```json
{
  "step_id": "S-01",
  "element_type": "game",
  "element_id": "G-01542",
  "stage": "learn",
  "step_metadata": {
    "page_correlation": "5",
    "concept_tag": "middle_c_identification",
    "skill_statement": "Identify Middle C on keyboard and staff"
  }
}
```

### Sequence code naming convention

Method-aligned sequences follow a standardized naming pattern:

**Format**: `SEQ-[METHOD_CODE]-[LEVEL]`

**Examples**:
- `SEQ-FABR-PRIMER` (Piano Adventures Primer Level)
- `SEQ-HALL-BOOK-1` (Hal Leonard Student Piano Library Book 1)
- `SEQ-ALFR-1A` (Alfred's Premier Piano Course Level 1A)
- `SEQ-TREE-PREP-A` (The Music Tree Prep Level A)

**Method codes** (4-letter identifiers):
- `FABR` - Faber Piano Adventures
- `HALL` - Hal Leonard Student Piano Library
- `ALFR` - Alfred's Premier Piano Course
- `CELE` - Celebrate Piano
- `SUCC` - Succeeding at the Piano
- `TREE` - The Music Tree
- `ARTI` - Artistry at the Piano
- `RCME` - The Royal Conservatory Music Examinations
- `MYCH` - Music For Young Children

New method codes are assigned by Content Editors and registered in the sequence code registry.

## Behavior and rules

### Alignment creation workflow

#### Phase 1: Publisher partnership and content analysis
1. **Legal and licensing**: Establish permission to reference method book content (page numbers, titles)
2. **Content inventory**: Catalog all levels, units, pages, and concepts in the method series
3. **Pedagogical review**: Analyze the method's teaching philosophy, concept progression, and assessment points
4. **Gap analysis**: Identify which MLC games align to method concepts; flag gaps where new games may be needed

#### Phase 2: Metadata specification
1. **Create sequence metadata**: Define publisher, series, level, edition, ISBN
2. **Map unit structure**: Define Groups corresponding to method book units/chapters
3. **Map lesson structure**: Define Assignments corresponding to specific lessons or page ranges
4. **Identify concepts**: Tag each lesson with concept identifiers from MLC's concept taxonomy

#### Phase 3: Game-to-concept mapping
1. **Select games**: For each concept in the method book, identify MLC games that teach or reinforce that skill
2. **Configure steps**: Define game stages (Learn, Play, Quiz, Review) and target scores appropriate for method book context
3. **Set progression**: Ensure games progress in difficulty matching the method book's pedagogical arc
4. **Validate coverage**: Confirm all major concepts in the method book have corresponding MLC game practice

#### Phase 4: Validation and testing
1. **Content validation**: Verify all referenced games exist and are in `live` status
2. **Metadata completeness**: Check that all required fields are populated
3. **Page correlation accuracy**: Confirm page ranges match current edition
4. **Pedagogical review**: Teacher and content expert review for alignment quality
5. **Pilot testing**: Test with real teachers and students using the method book

#### Phase 5: Publication and maintenance
1. **Publish sequence**: Move from draft to published state with appropriate `visibility` setting
2. **Teacher onboarding**: Create documentation explaining how to use method alignment
3. **Monitor usage**: Track completion rates, teacher feedback, and student performance
4. **Edition updates**: When method book publisher releases new editions, validate and update correlations

### Validation rules

Method-aligned sequences must pass validation before publication:

#### Critical errors (block publishing)
- **Missing required metadata**: Any of `publisher`, `series_name`, `book_level`, `edition_year` is null
- **Invalid game references**: Step references a `game_id` that does not exist or is not `live`
- **Orphaned groups or assignments**: Empty groups or assignments with no steps
- **Circular dependencies**: Gating rules that create impossible progression
- **Duplicate IDs**: `group_id`, `assignment_id`, or `step_id` collision within sequence

#### Warnings (allow publishing with acknowledgment)
- **Deprecated games**: Step references a game marked as `deprecated` or `sunset`
- **Missing page correlations**: Assignment lacks `page_range` or `lesson_title`
- **Concept coverage gaps**: Method book introduces concepts (per taxonomy) that have no corresponding MLC games
- **Target score outliers**: Target scores significantly different from game performance baselines
- **Stale correlation date**: `correlation_date` is more than 3 years old

#### Advisory notices (informational)
- **Long assignments**: Assignment contains more than 15 steps (may overwhelm students)
- **No Challenge stages**: Sequence lacks Challenge stages (acceptable but consider for engagement)
- **Edition year mismatch**: Edition year doesn't match most recent publisher edition

### Cross-publisher consistency standards

To ensure method-aligned sequences work consistently across different publishers:

1. **Concept taxonomy alignment**: All method alignments use MLC's standardized concept tags (e.g., `middle_c`, `quarter_note`, `major_pentascale`)
2. **Difficulty tier consistency**: Games at similar points in different method alignments should have comparable difficulty
3. **Stage progression pattern**: All method alignments follow Learn → Play → Quiz → Review stage sequence
4. **Target score standards**: Target scores calibrated to game difficulty, not method book expectations
5. **Metadata schema uniformity**: All method alignments use identical metadata field names and data types

### Relationship to first-party sequences

Method-aligned sequences coexist with first-party sequences:

**Scenario 1: Teacher using method alignment exclusively**
- Assign only the method-aligned sequence (e.g., Piano Adventures Primer)
- Students play games mapped to their current method book pages
- Reports show progress through method book curriculum

**Scenario 2: Teacher using first-party curriculum exclusively**
- Assign LIFE or SOLF sequence
- Students follow MLC's proprietary pedagogical progression
- No method book correlation required

**Scenario 3: Hybrid approach**
- Assign first-party sequence (LIFE) as primary curriculum
- Selectively assign method-aligned assignments to supplement specific method book lessons
- Both sequences progress independently; reports show combined activity

**Scenario 4: Assessment with EVAL**
- Use EVAL sequence for placement testing
- Results inform whether to assign first-party or method-aligned sequence
- Teacher can switch between sequences based on assessment outcomes

## UX requirements (if applicable)

### Teacher experience: Selecting method-aligned sequences

When creating a new assignment, teachers can choose:

**Sequence type selector**:
- Radio button: "MLC Curriculum (LIFE, SOLF)"
- Radio button: "Method Book Alignment"

If "Method Book Alignment" is selected, display:

**Method book dropdown** (grouped by publisher):
```
Faber and Faber
  ├─ Piano Adventures - Primer Level
  ├─ Piano Adventures - Level 1
  ├─ Piano Adventures - Level 2A
  └─ Piano Adventures - Level 2B
Hal Leonard
  ├─ Student Piano Library - Book A
  ├─ Student Piano Library - Book 1
  └─ Student Piano Library - Book 2
Alfred Music
  ├─ Premier Piano Course - Level 1A
  └─ Premier Piano Course - Level 1B
```

**Book information display**:
- Publisher name
- Series name
- Book level
- Edition year
- ISBN (with link to purchase book if not owned)
- Correlation date with freshness indicator (green if < 1 year, yellow if 1-3 years, red if > 3 years)

**Assignment browser**:
- List of units/groups with page ranges
- Expandable to show individual assignments
- Visual indicators: concept tags, estimated duration, assessment points
- Search/filter by page number or concept

### Student experience: Viewing method book correlations

Students see assignments with optional method book context:

**Assignment header** (if method-aligned):
```
Assignment: Lesson 1 - Middle C Position
Piano Adventures Primer Level, Pages 4-6
```

**Optional display** (controlled by teacher preference):
- Show/hide method book correlation in student view
- Some teachers prefer students focus on games without book reference
- Others want students to see explicit connection to their book lessons

### Accessibility

Method-aligned sequence selection must meet [WCAG Compliance](../16-accessibility-and-inclusion/wcag-compliance.md) standards:

- **Screen reader support**: Method book dropdown announces publisher groupings and book levels
- **Keyboard navigation**: Full keyboard access to dropdown and assignment browser
- **Focus indicators**: Clear visual focus on interactive elements
- **Color independence**: Correlation freshness indicators use icons + text, not color alone

## Telemetry

Method-aligned sequence events extend the core telemetry defined in [Event Model](../15-analytics-and-reporting/event-model.md) and [Sequence Model](sequence-model.md).

### Method-specific events

**`method_sequence_assigned`**
- Fired when: Teacher assigns a method-aligned sequence to students
- Properties:
  - `sequence_id`, `sequence_version`
  - `publisher`, `series_name`, `book_level`
  - `student_count`, `org_id`
  - `assignment_mode`: full_sequence | unit_selection | individual_assignments

**`method_page_correlation_viewed`**
- Fired when: Teacher or student views assignment details showing method book page correlation
- Properties:
  - `sequence_id`, `assignment_id`
  - `page_range`, `lesson_title`
  - `viewer_role`: teacher | student

**`method_alignment_validated`**
- Fired when: Validation runs on a method-aligned sequence during authoring or import
- Properties:
  - `sequence_id`, `sequence_version`
  - `publisher`, `series_name`, `book_level`
  - `critical_errors_count`, `warnings_count`, `advisories_count`
  - `validation_result`: pass | pass_with_warnings | fail

**`method_edition_update_triggered`**
- Fired when: Publisher releases new edition and alignment update workflow begins
- Properties:
  - `sequence_id`, `old_edition_year`, `new_edition_year`
  - `publisher`, `series_name`, `book_level`
  - `update_scope`: metadata_only | page_correlations | game_mappings

### Analytics queries for method alignment health

**Adoption rate by publisher**:
- Track which method alignments have highest usage
- Inform prioritization for new method alignments and updates

**Concept coverage heatmap**:
- Identify which method book concepts have strong game coverage vs. gaps
- Guide game development priorities

**Correlation staleness report**:
- Flag method alignments with outdated `correlation_date` or `edition_year`
- Trigger proactive maintenance before teachers encounter issues

**Student performance by method alignment**:
- Compare student outcomes between method-aligned vs. first-party sequences
- Measure effectiveness of different method book integrations

## Permissions

Method-aligned sequence permissions follow the core role-based access control (RBAC) system defined in [Roles Matrix](../02-roles-and-permissions/roles-matrix.md).

### Role capabilities for method-aligned sequences

| Action | Student | Teacher | Teacher-Admin | Subscriber-Admin | Content-Editor | System-Admin |
|--------|---------|---------|---------------|------------------|----------------|--------------|
| View available method alignments | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Assign method-aligned sequences | ❌ | ✅ | ✅ | ✅ | ❌ | ✅ |
| Create method-aligned sequences | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Edit method-aligned sequences | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Publish method-aligned sequences | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Validate page correlations | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Update edition metadata | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| Archive method-aligned sequences | ❌ | ❌ | ❌ | ❌ | ❌ | ✅ |

### Special considerations

**Publisher approval metadata**:
- Method-aligned sequences created with publisher partnership include `publisher_approved: true` in metadata
- Sequences created without formal partnership (fair use page references only) include `publisher_approved: false`
- Approved sequences may be marketed and featured more prominently

**Content-Editor role specialization**:
- Content-Editors assigned to method alignment work typically have expertise in that specific method book series
- Assignment scoped by publisher or series name in role metadata

**Audit requirements**:
- All edits to method-aligned sequences are logged with user ID, timestamp, and change summary
- Page correlation changes require change justification (new edition, error correction, pedagogical improvement)

## Dependencies

### Core data models and systems
- [Sequence Model](sequence-model.md) - Base sequence structure and versioning
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game definitions and stages
- [Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md) - Game availability and health
- [Games Metadata](../08-games-registry-and-authoring/games-metadata.md) - Concept tags and skill taxonomy
- [Learning Elements Overview](../07-content-architecture/learning-elements-overview.md) - Content type definitions

### Sequence management tools
- [Sequence Authoring](sequence-authoring.md) - Authoring interface for creating alignments
- [Sequence Libraries](sequence-libraries.md) - Catalog and discovery of method-aligned sequences
- [Sequence Exports](sequence-exports.md) - Export formats for method alignment data
- [Curriculum Mapping Tools](curriculum-mapping-tools.md) - Tools for analyzing and mapping curricula

### Assignment and delivery
- [Assignment Generation](../10-assignments-engine/assignment-generation.md) - Assignment delivery logic
- [Assignment Model](../07-content-architecture/assignment-model.md) - Assignment structure

### Pedagogy and design
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) - Educational philosophy
- [Piano Adventures Alignment](../07-content-architecture/piano-adventures-alignment.md) - Detailed example implementation
- [Game Taxonomy](../07-content-architecture/game-taxonomy.md) - Concept categorization system

### Import and export
- [CSV Specifications](../20-imports-and-automation/csv-specifications.md) - Bulk import/export schemas
- [Bulk Operations Jobs](../20-imports-and-automation/bulk-ops-jobs.md) - Background processing

### Roles and permissions
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role definitions
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permissions

### Telemetry and monitoring
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry event definitions
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Reporting on method alignment usage
- [Observability](../19-quality-and-operations/observability.md) - System monitoring

### Legal and compliance
- [Copyright and Licensing](../23-legal-and-compliance/copyright-and-licensing.md) - Legal framework for referencing published works

## Open questions

### Publisher partnerships
- **Partnership scope**: Should we prioritize broad method coverage (many publishers, shallow depth) or deep integration (few publishers, comprehensive levels)?
- **Revenue sharing models**: If publishers request revenue share for alignments, how does this affect pricing and which methods we prioritize?
- **Co-marketing agreements**: Should method alignments include publisher branding, logos, and co-marketing materials?

### Metadata and correlation accuracy
- **Edition tracking**: How do we handle method books with multiple editions in circulation? Support multiple editions simultaneously or only latest?
- **Page correlation granularity**: Should correlations reference individual pages, page ranges, or specific exercises within pages?
- **Concept taxonomy evolution**: When MLC's internal concept taxonomy changes, how do we migrate existing method alignments?
- **Multi-book series**: Some methods span companion books (Lesson, Theory, Technique). How do we correlate across companion books?

### Content coverage and gaps
- **Gap prioritization**: When method book teaches concepts MLC lacks games for, should we prioritize new game development or skip those concepts in alignment?
- **Concept depth mismatch**: When method book goes deeper on a concept than MLC games support, how do we handle? (Repeat games, flag gap, suggest free play?)
- **Cross-method game reuse**: Can a single MLC game be used in multiple method alignments with different metadata/targets?

### Student and teacher experience
- **Book ownership verification**: Should system verify students/teachers own the method book before allowing access to alignment?
- **Print integration**: Should we support QR codes or links printed in method books to jump directly to MLC assignments?
- **Teacher customization**: Can teachers edit method-aligned sequences (clone and customize page correlations for their teaching style)?
- **Progress translation**: If a student switches from one method book to another, how do we map progress between alignments?

### Quality assurance and maintenance
- **Update frequency**: How often should we validate method alignments (annually, biannually, on publisher notification)?
- **Community contributions**: Should we allow community (teachers) to submit method alignment corrections or new alignments?
- **Deprecation policy**: When a method book goes out of print, when do we archive the alignment? Keep indefinitely for historical access?
- **Performance benchmarking**: Should we establish baseline performance metrics (completion rates, scores) per method alignment to detect quality issues?

### Technical architecture
- **Sequence versioning on edition updates**: When method book edition changes, should we create new `sequence_version` or new `sequence_id`?
- **Lazy loading for large series**: Some method series have 10+ levels. Should catalog lazy-load alignment details?
- **Caching strategy**: Method metadata rarely changes. What caching TTL is appropriate for method alignment data?
- **API design**: Should method alignments have dedicated API endpoints separate from general sequence endpoints?

### Analytics and reporting
- **ROI tracking**: How do we measure success/ROI of method alignment efforts to inform which methods to align next?
- **Teacher feedback loops**: Should we build in-app feedback mechanisms for teachers to report alignment issues?
- **Competitive analysis**: Should we track which method books competitors support and prioritize accordingly?
