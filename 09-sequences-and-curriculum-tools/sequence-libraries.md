# Sequence Libraries

## Purpose
Catalog of first-party and method-aligned Sequences available for assignment and AI generation.

This spec defines the organizational structure, discovery mechanisms, and lifecycle management for all sequence libraries in the system. For the underlying sequence data model and versioning rules, see [Sequence Model](sequence-model.md). For authoring tools, see [Sequence Authoring](sequence-authoring.md).

## Library buckets
- First-party core
  - LIFE - Lifetime Musician
  - SOLF - Solfege
  - EVAL - Evaluation
- Method-aligned
  - Publisher and method sequences mapped to skills, units, pages, and explicit Game stages
- State or district curricula
- Custom teacher sequences

### Library visibility and access
Each sequence library has a `visibility` attribute that controls discoverability and assignment permissions:

- **first_party**: Visible to all subscribers. Can be assigned by Teachers, Teacher-Admins, and Subscriber-Admins.
- **method_aligned**: Visible when enabled at the organization or account level. Controlled via feature flags or entitlements.
- **custom**: Scoped to specific organizations. Created by Teachers or Admins within their org and not visible outside.
- **archived**: Retained for historical reporting but hidden from new assignment workflows.

See [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) for full role capabilities and [Permissions Table](../02-roles-and-permissions/permissions-table.md) for granular permission definitions.

## First-party sequence details

### LIFE - Lifetime Musician
The flagship first-party curriculum designed for comprehensive music education. Organized by proficiency levels from Primary 1A through Level 6+.

**Structure example from legacy system**:
- Primary Level 1A
  - Assignment 1: Songbirds High and Low, Storm Chasers 1, Smiley and Friends 1, Faster-Slower-Same 1, Tommy Tiger's 2's & 3's
  - Assignment 2: Hand Prints, Low Middle High, Finger Finder, Smiley & Friends-Rhythm 1, Song Birds 1
  - Assignment 3: Steady As She Goes, Pick the Pattern Loud Soft, Meteor Match 1 A, Falling Symbols 1, Song Birds 2
  - Each game appears with Learn, Play, Quiz stages; some include Challenge stage

**Pedagogical approach**: Balanced progression through keyboard elements, pitch, rhythm, scales, intervals, and music symbols. Emphasizes both aural (listening) and visual (reading) skill development. See [Pedagogy Principles](../00-foundations/pedagogy-principles.md) for the educational philosophy.

### SOLF - Solfege
Specialized sequence focusing on ear training and solfege syllable recognition. Uses pre-staff notation initially, transitioning to staff-based exercises.

**Core concept areas**: Do-Re-Mi patterns, pitch matching, melodic dictation, interval recognition in solfege context.

### EVAL - Evaluation
Diagnostic sequence for student placement testing. Shorter, targeted games that assess proficiency across key skill areas.

**Usage pattern**: Typically assigned once at onboarding or when re-evaluating student level. Results inform which LIFE or method-aligned sequence assignment is appropriate.

## Method-aligned sequence libraries

### Hal Leonard Student Piano Library - exemplars
These PDFs show the exact correlation pattern we will mirror in the AI importer and authoring UI: Unit, page range, Category, Game, Skills/Concepts, and stage targets.

- Book A - Pages 4 to 79. Early skills include high vs low, steady beat, finger numbers, groups of 2 and 3 black keys, and quarter vs half notes. Lists per-row Learn, Play, Quiz target values.
- Book B - Pages 4 to 79. Introduces first staff work, guide notes, middle C pentascale, and simple aural patterns with per-stage targets.
- Book 1 - Intro plus Units 1 to 5. Adds rhythm identification, staff birds playback, guide notes, and beginner pentascales with explicit targets for each stage.
- Book 2 - Units 1 to 5. Expands to bass and treble pentascales, rhythm writers, interval work, and pattern reading with MIDI variants.
- Book 3 - Units 1 to 5. Broader interval sets, key signatures, more complex rhythm tasks, and ledger line reading.
- Book 4 - Units 1 to 5. Adds triplets, 3-8 and 6-8 meters, more ledger lines, and triads with inversions, plus timed note-reading games.
- Book 5 - Units 1 to 6. Key signatures through multiple sharps and flats, chord quality identification, relative minors, dictation, and advanced rhythm writers.

### Other method publishers (legacy system supported)
The original MLC-4.0 system supported alignments to the following method book series:

- **ALFR** - Alfred's Premier Piano Course
- **ARTI** - Artistry at the Piano
- **CELE** - Celebrate Piano
- **FABR** - Faber Piano Adventures
- **MYCH** - Music For Young Children
- **SUCC** - Succeeding at the Piano
- **TREE** - The Music Tree
- **RCME** - The Royal Conservatory Examinations

Each method-aligned sequence followed the structure of the original book: levels, units, page ranges, and mapped specific games and stages to lesson concepts. For details on correlation metadata and how these sequences are structured, see [Sequence Alignment to Methods](sequence-alignment-to-methods.md).

### Correlation pattern
Method-aligned sequences typically include:
- **Level/Book identifier**: e.g., "Level 1A" or "Book 2"
- **Unit or chapter**: Logical grouping within a book level
- **Page range**: Specific pages in the physical or digital book
- **Concept coverage**: Skills or learning goals addressed (e.g., "Treble C pentascale", "Dotted quarter-eighth rhythm")
- **Game mappings**: Which games align to the concept, and at which stages (Learn, Play, Quiz, Challenge)
- **Target scores**: Expected mastery levels per stage to mirror the book's pedagogical intent

See [Piano Adventures Alignment](../07-content-architecture/piano-adventures-alignment.md) for a detailed example of method book correlation.

## Metadata per sequence
Each sequence library entry includes structured metadata to support discovery, filtering, assignment, and reporting:

### Core identifiers
- `sequence_id`: Canonical stable identifier (e.g., "SEQ-LIFE", "SEQ-HALL-BOOK-1")
- `name`: Display name (e.g., "Lifetime Musician", "Hal Leonard Book 1")
- `version`: Integer version number for tracking content changes
- `visibility`: Enum (`first_party`, `method_aligned`, `custom`, `archived`)
- `instrument`: Target instrument (e.g., "piano", "keyboard")

### Source and alignment metadata
- `source_reference`: For method-aligned sequences
  - `publisher`: Name of the method book publisher
  - `series_name`: The method book series title
  - `book_level`: Level or book identifier (e.g., "Book 1A", "Level 2")
  - `edition_year`: Publication year for version tracking
  - `page_mapping`: Structured map of units/chapters to page ranges
- `org_id`: Present for custom sequences to scope visibility

### Pedagogical metadata
- `concept_coverage`: Array of concept tags (e.g., `["intervals", "rhythm", "key_signatures"]`)
- `estimated_hours`: Expected time to complete the full sequence
- `target_age`: Recommended age range (e.g., "8-14", "Adult")
- `prerequisite_skills`: Required prior knowledge or mastery
- `skill_mix`: Categorization like "visual", "aural", "mixed", "performance"

### Assignment permissions
- `who_can_assign`: Array of role codes (`Teacher`, `Teacher-Admin`, `Subscriber-Admin`, `System-Admin`)
- `entitlement_required`: Optional flag indicating if special subscription or feature access is needed for method-aligned sequences

For the full schema and versioning behavior, see [Sequence Model](sequence-model.md).

## Sequence discovery and browsing

Teachers and admins browse sequence libraries via filtered views:

### Filtering and search
- **By visibility**: First-party, method-aligned, custom (org-scoped), archived
- **By publisher**: For method-aligned sequences, filter by Alfred, Faber, Hal Leonard, etc.
- **By level**: Filter by target proficiency (e.g., "Primary", "Level 1", "Intermediate")
- **By concept**: Search or filter by tagged concepts like "intervals", "key signatures", "rhythm"
- **By instrument**: Currently piano-focused, but extensible to other instruments

### Sequence preview
Before assigning, teachers can:
- View sequence structure: Groups > Assignments > Steps
- See estimated completion time and target age range
- Review concept coverage and prerequisite requirements
- Check assignment permissions and entitlements

For search indexing and faceted filtering implementation, see [Search and Discovery](../11-search-and-discovery/search-indexing.md).

## Sequence lifecycle management

### Creation and publishing
- **First-party sequences**: Created by content editors, reviewed by pedagogical team, published by System-Admin.
- **Method-aligned sequences**: Imported via [Sequence Importer](sequence-importer.md) or manually authored, then published with publisher approval metadata.
- **Custom sequences**: Teachers or Admins create within their organization using [Sequence Authoring](sequence-authoring.md) tools, no global publish step required.

### Versioning and updates
When a sequence is updated:
1. Increment `sequence_version` integer
2. Existing assignments remain pinned to prior version for grading stability
3. New assignments reference the latest version
4. Teachers receive notifications of available updates and can opt to migrate pinned assignments
5. Migration tools show diffs and preserve student progress where possible

See [Sequence Model](sequence-model.md) for detailed versioning rules and [Content Versioning](../07-content-architecture/content-versioning.md) for broader content change management patterns.

### Archiving and deprecation
Sequences that are no longer actively maintained or have been superseded:
- Set `visibility = archived`
- Remain accessible for historical assignments and reporting
- Hidden from new assignment workflows and teacher browsing
- Cannot be assigned to new students
- Archived sequences can be unarchived if needed (e.g., publisher re-releases a method book)

## Import and export
- **JSON format**: Full sequence structure including groups, assignments, steps, metadata, and targets. Suitable for programmatic creation, backup, and version control.
- **CSV format**: Flattened representation for bulk editing in spreadsheet tools. Typically used for method-aligned sequences where units and page mappings are managed in tabular form.
- **Import validation**: All imports validate against sequence schema, check for missing game references, and flag deprecated content.
- **Export use cases**: Backup, migration between environments, sharing custom sequences across organizations (with permission), and analysis.

For detailed CSV specifications, see [CSV Specifications](../20-imports-and-automation/csv-specifications.md). For AI-assisted sequence generation from method books, see [AI Sequence Generator](ai-sequence-generator.md) (note: this feature is on hold per current planning).

## QA gates and health checks

Before a sequence is published or updated, automated and manual checks ensure content quality:

### Coverage checks
- **Baseline skills per level**: Ensure that foundational concepts (e.g., staff reading, basic rhythm) are addressed before advancing to more complex topics.
- **Concept progression**: Validate that prerequisite concepts appear before dependent concepts (e.g., note names before intervals).
- **Balanced skill mix**: Check that sequences include a reasonable distribution of aural, visual, and performance-based games.

### Content health checks
- **Missing game references**: Detect if a sequence step references a `game_id` or `stage` that does not exist in the [Games Registry](../08-games-registry-and-authoring/games-registry-overview.md).
- **Deprecated content warnings**: Flag games marked as `deprecated` or `sunset` in the registry. See [Games Stages Policy](../08-games-registry-and-authoring/games-stages-policy.md) for stage lifecycle rules.
- **Asset availability**: Verify that all referenced game bundles, media, and localized assets are accessible via the [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md).
- **Broken stage mappings**: Ensure each game step specifies valid stages (Learn, Play, Quiz, Challenge, Review) as defined in [Game Model](../08-games-registry-and-authoring/game-model.md).

### Manual review gates
- **Pedagogical review**: First-party sequences undergo review by education experts to confirm alignment with [Pedagogy Principles](../00-foundations/pedagogy-principles.md).
- **Publisher approval**: Method-aligned sequences require sign-off from the publisher or rights holder before publication.
- **Compliance checks**: Ensure sequences meet accessibility standards ([WCAG Compliance](../16-accessibility-and-inclusion/wcag-compliance.md)) and legal requirements ([COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md)).

### Health monitoring dashboards
System-Admins and content editors access dashboards showing:
- Sequences with missing or deprecated content
- Completion rates and average time-to-mastery per sequence (feed into [Admin Reports](../05-admin-subscriber-experience/admin-reports.md))
- Student feedback and teacher flags for sequences needing revision
- Usage analytics to identify underutilized or problematic sequences

For operational monitoring and alerting, see [Observability](../19-quality-and-operations/observability.md).

## Telemetry
Sequence library interactions generate events for analytics and operational monitoring:

- `sequence_library_browsed`: Fired when a user accesses the sequence library view
  - Properties: `user_role`, `filter_applied`, `results_count`
- `sequence_previewed`: Fired when a user opens sequence details before assigning
  - Properties: `sequence_id`, `sequence_version`, `visibility`, `user_role`
- `sequence_published`: Fired when a sequence is published or updated
  - Properties: `sequence_id`, `sequence_version`, `visibility`, `publisher_id`, `change_type` (new, update, archive)
- `sequence_assigned`: Fired when a sequence is assigned to students
  - Properties: `sequence_id`, `sequence_version`, `student_count`, `org_id`, `assigner_role`
- `sequence_version_migrated`: Fired when an existing assignment is migrated to a new sequence version
  - Properties: `sequence_id`, `from_version`, `to_version`, `assignment_id`, `student_count`

For canonical telemetry definitions and sampling rules, see [Event Model](../15-analytics-and-reporting/event-model.md).

## Open questions
- **Multi-instrument support**: How should sequences be structured and filtered when expanding beyond piano (e.g., guitar, voice)?
- **Sequence recommendations**: Should the system auto-recommend sequences based on student proficiency, prior assignments, or evaluation results?
- **Custom sequence sharing**: Allow teachers to share custom sequences publicly (with moderation) or within a district/region?
- **Localization**: How should method-aligned sequences handle multi-language editions of the same book series? See [Localization and Internationalization](../01-ux-design-system/localization-internationalization.md).
- **Adaptive sequences**: Support for dynamically adjusting sequence content based on real-time student performance and learning analytics?
- **Cross-sequence dependencies**: How to represent sequences that build on others (e.g., SOLF extending LIFE foundations)?
- **Sequence cloning**: Allow teachers to clone and modify first-party or method-aligned sequences as a starting point for custom sequences?
- **Publisher partnerships**: Define process and metadata requirements for onboarding new method book publishers.

## Dependencies
- [Sequence Model](sequence-model.md) - Core data structures, versioning, and progression rules
- [Sequence Authoring](sequence-authoring.md) - Tools for creating and editing sequences
- [Sequence Alignment to Methods](sequence-alignment-to-methods.md) - Method book correlation patterns
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game and stage definitions
- [Games Registry](../08-games-registry-and-authoring/games-registry-overview.md) - Content availability and health
- [Games Stages Policy](../08-games-registry-and-authoring/games-stages-policy.md) - Stage lifecycle rules
- [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md) - Asset availability checks
- [Assignment Generation](../10-assignments-engine/assignment-generation.md) - How assignments are created from sequences
- [Search and Discovery](../11-search-and-discovery/search-indexing.md) - Sequence indexing and filtering
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Role capabilities for sequence management
- [Permissions Table](../02-roles-and-permissions/permissions-table.md) - Granular permissions
- [Event Model](../15-analytics-and-reporting/event-model.md) - Telemetry definitions
- [CSV Specifications](../20-imports-and-automation/csv-specifications.md) - Import/export formats
- [Content Versioning](../07-content-architecture/content-versioning.md) - Version control patterns
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) - Educational philosophy
- [Piano Adventures Alignment](../07-content-architecture/piano-adventures-alignment.md) - Example method correlation
- [WCAG Compliance](../16-accessibility-and-inclusion/wcag-compliance.md) - Accessibility standards
- [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md) - Legal compliance
- [Observability](../19-quality-and-operations/observability.md) - System monitoring
- [Admin Reports](../05-admin-subscriber-experience/admin-reports.md) - Reporting dashboards
