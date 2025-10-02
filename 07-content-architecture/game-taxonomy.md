# Game Taxonomy

This document defines the comprehensive classification and categorization system for games in the MLC platform. The taxonomy enables systematic organization, discovery, and management of the extensive game library while supporting curriculum alignment and student progression.

## Purpose

The game taxonomy provides a structured framework for:

- **Content Organization**: Systematic categorization of games by musical concepts, difficulty levels, and learning objectives
- **Curriculum Alignment**: Mapping games to educational standards and learning sequences
- **Discovery and Search**: Enabling teachers and students to find appropriate games through multiple classification dimensions
- **Assessment Planning**: Supporting targeted skill development through concept-based game selection
- **Content Management**: Facilitating efficient organization and maintenance of the game library
- **Analytics and Reporting**: Enabling detailed analysis of game usage and effectiveness across different categories

## Scope

### Included
- **Primary Classification System**: Core taxonomy structure with hierarchical categories
- **Concept Mapping**: Musical concept tags and skill area classifications
- **Difficulty Progression**: Level-based organization from beginner to advanced
- **Learning Mode Classification**: Visual, aural, and mixed learning approaches
- **Instrument-Specific Categories**: Piano, general music, and other instrument-focused games
- **Standards Alignment**: Mapping to educational standards and curriculum requirements
- **Metadata Schema**: Comprehensive tagging system for search and filtering

### Excluded
- **Game Mechanics**: Specific gameplay rules and interactions (covered in [Game Model](../08-games-registry-and-authoring/game-model.md))
- **Sequence Logic**: How games combine in learning sequences (covered in [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))
- **Assignment Rules**: How games are presented to students (covered in [Assignment Model](assignment-model.md))

## Data Model

### Primary Classification Hierarchy

The taxonomy follows a hierarchical structure with three main tiers:

#### Tier 1: Music Family Categories
- **Basic Elements (BASE)**: Foundational musical concepts
- **Stage (STAGE)**: Game progression stages
- **Rhythm (RHYTM)**: Rhythmic concepts and patterns
- **Meter (METER)**: Time signatures and rhythmic organization
- **Staff & Pitch (STAFF)**: Notation and pitch recognition
- **Melody (MELOD)**: Melodic patterns and sequences
- **Scale (SCALE)**: Scale patterns and relationships
- **Key Signatures (KYSIG)**: Tonality and key relationships
- **Intervals (INTVL)**: Pitch relationships and interval recognition
- **Keyboard Elements (KEYBD)**: Piano keyboard knowledge
- **Chords (CHORD)**: Harmonic structures and chord recognition
- **Harmonic Progression (HARM)**: Chord progressions and harmonic movement
- **Music Terms (TERM)**: Musical vocabulary and symbols
- **Tonal Memory & Playback (MEMO)**: Aural memory and performance skills

#### Tier 2: Concept Subcategories
Each primary category contains detailed subcategories. For example:

**Rhythm (RHYTM)** includes:
- Pre-Reading concepts (steady beat, long/short)
- Note & Rest Values (whole, half, quarter, eighth, sixteenth notes and rests)
- Syncopation patterns
- Rhythmic Dictation
- Pattern Identification (triplets, dotted rhythms, etc.)

**Staff & Pitch (STAFF)** includes:
- Treble Staff concepts (lines, spaces, ledger lines, pentascales)
- Bass Staff concepts (lines, spaces, ledger lines, pentascales)
- Grand Staff concepts (ledger lines between staves, sharps/flats, key signatures)

#### Tier 3: Specific Concepts
The most granular level includes specific musical concepts. For example:

**Scale (SCALE)** includes:
- Pentascale Major (C, G, D, A, E, B, F#, C#, F, Bb, Eb, Ab, Db, Gb, Cb)
- Pentascale Minor (a, e, b, f#, c#, g#, d#, a#, d, g, c, f, bb, eb, ab)
- Major Scales (all 15 major keys)
- Natural Minor Scales
- Harmonic Minor Scales
- Melodic Minor Scales
- Chromatic Scale
- Whole Tone Scale

### Classification Codes

Each concept is assigned a hierarchical code following the pattern:
`CATEGORY-SUBCATEGORY-SPECIFIC`

Examples:
- `RHYTM-NOTV-QUAR` (Rhythm - Note Values - Quarter Note)
- `STAFF-TREB-TSTF` (Staff - Treble - Treble Staff)
- `SCALE-PENTM-CMAJ` (Scale - Pentascale Major - C Major)
- `CHORD-MAJC-C` (Chord - Major Chord - C)

### Game Metadata Schema

```json
{
  "game_id": "G-01542",
  "primary_category": "Staff",
  "secondary_category": "Grand Staff",
  "concept_tags": [
    "guide_notes",
    "treble_staff", 
    "bass_staff",
    "spatial_relationships"
  ],
  "skill_areas": ["visual", "aural"],
  "difficulty_level": "beginner",
  "musical_concepts": [
    "RHYTM-NOTV-QUAR",
    "STAFF-GRND-LNSP"
  ],
  "learning_mode": "visual",
  "instrument_focus": "piano",
  "prerequisites": ["STAFF-TREB-TSTF", "STAFF-BASS-BSTF"],
  "standards_alignment": [
    "custom:mlc:L1.staff.guide_notes",
    "menc:2a",
    "menc:6c"
  ]
}
```

## Behavior and Rules

### Classification Rules

1. **Single Primary Category**: Each game must be assigned to exactly one primary category
2. **Multiple Concept Tags**: Games can have multiple concept tags for cross-category discovery
3. **Hierarchical Inheritance**: Games inherit characteristics from their category hierarchy
4. **Prerequisite Mapping**: Games can specify prerequisite concepts that should be mastered first
5. **Difficulty Progression**: Games are ordered within categories by difficulty level

### Search and Discovery Rules

1. **Multi-dimensional Filtering**: Users can filter by category, concept, difficulty, instrument, and learning mode
2. **Fuzzy Matching**: Search supports partial matches and related concepts
3. **Cross-category Discovery**: Games appear in search results for related concepts
4. **Prerequisite Awareness**: System suggests prerequisite games when appropriate

### Content Validation

1. **Required Fields**: All games must have primary category, concept tags, and difficulty level
2. **Concept Validation**: All concept tags must reference valid taxonomy codes
3. **Prerequisite Validation**: Prerequisites must exist in the taxonomy
4. **Standards Alignment**: Standards references must be valid and current

## UX Requirements

### Search Interface
- **Category Browser**: Hierarchical tree view for exploring categories
- **Filter Panel**: Multi-select filters for concept, difficulty, instrument, and mode
- **Search Bar**: Text search with autocomplete and suggestion
- **Results Display**: Clear indication of game category and concept alignment

### Game Selection Flow
1. **Category Selection**: User chooses primary category or browses all
2. **Concept Filtering**: User narrows down by specific concepts or difficulty
3. **Game Preview**: User sees game details including concept alignment and prerequisites
4. **Stage Selection**: User chooses specific game stage (Learn, Play, Quiz, Challenge, Review)

### Accessibility
- **Keyboard Navigation**: Full keyboard support for all taxonomy navigation
- **Screen Reader Support**: Proper ARIA labels and semantic structure
- **High Contrast**: Clear visual distinction between categories and concepts
- **Reduced Motion**: Respect user preferences for animation and transitions

## Telemetry

### Taxonomy Interaction Events
- `category_viewed`: When user browses a category
- `concept_filtered`: When user applies concept filters
- `game_discovered`: When user finds a game through taxonomy navigation
- `search_performed`: When user uses text search
- `prerequisite_suggested`: When system suggests prerequisite games

### Analytics Properties
- `category_path`: Full hierarchical path to selected category
- `concept_tags`: Array of concept tags used in filtering
- `search_query`: Text search terms used
- `filter_applied`: Specific filters that were applied
- `discovery_method`: How the game was found (browse, search, filter)

## Permissions

### Student Access
- **Read**: Can browse and search game taxonomy
- **Filter**: Can apply filters to find appropriate games
- **View Details**: Can see game concept alignment and prerequisites

### Teacher Access
- **Read**: Full access to taxonomy and game metadata
- **Search**: Advanced search capabilities with multiple filters
- **Assignment**: Can assign games based on taxonomy categories
- **Analytics**: Can view usage patterns within taxonomy categories

### Content Editor Access
- **Read/Write**: Can modify game taxonomy assignments
- **Create**: Can add new categories and concepts
- **Manage**: Can update concept relationships and hierarchies
- **Validate**: Can ensure taxonomy consistency and accuracy

### System Admin Access
- **Full Control**: Complete taxonomy management capabilities
- **Bulk Operations**: Can perform large-scale taxonomy updates
- **Migration**: Can handle taxonomy restructuring and migrations
- **Analytics**: Full access to taxonomy usage and effectiveness data

## Dependencies

### Internal Dependencies
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Defines game structure and metadata
- **[Learning Elements Overview](learning-elements-overview.md)**: Defines element types and relationships
- **[Search and Discovery](../11-search-and-discovery/)**: Implements taxonomy-based search
- **[Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)**: Uses taxonomy for curriculum alignment

### External Dependencies
- **Content Management System**: Stores taxonomy metadata and relationships
- **Search Engine**: Indexes taxonomy data for fast retrieval
- **Analytics Platform**: Tracks taxonomy usage and effectiveness
- **Standards Database**: Maintains current educational standards references

## Open Questions

### Taxonomy Structure
- Should we add more granular subcategories for complex concepts?
- How should we handle games that span multiple primary categories?
- What is the optimal depth for the hierarchical structure?

### Content Management
- How should we handle taxonomy updates when new concepts are added?
- What is the migration strategy for existing games when taxonomy changes?
- How can we ensure consistency across the large game library?

### User Experience
- What is the optimal balance between detailed categorization and usability?
- How can we improve discovery for teachers unfamiliar with music theory?
- What visual representations would help users understand concept relationships?

### Analytics and Improvement
- How can we use usage data to improve taxonomy organization?
- What metrics indicate effective taxonomy design?
- How should we handle user feedback on taxonomy structure?
