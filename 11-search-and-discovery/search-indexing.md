# Search Indexing

This document defines how content is indexed for search across the MLC learning management system, enabling efficient discovery of games, sequences, and educational content.

## Purpose
The search indexing system enables users to quickly find relevant music learning games and content through multiple search dimensions. It supports the core user workflows of game discovery, assignment creation, and curriculum navigation by providing fast, accurate search results across the extensive content library.

## Scope
**Included:**
- Music learning games (450+ games across multiple levels and categories)
- Learning sequences (Lifetime Musician, Solfege, Evaluation, and method-specific sequences)
- Game metadata (levels, skill areas, concepts, difficulty, target scores)
- User-generated content (assignments, custom sequences)
- Search result ranking and relevance scoring
- Multi-language content indexing (English, Solfege variants)

**Excluded:**
- Real-time game play data
- User score history (indexed separately in analytics)
- Administrative system content
- Payment and subscription data

## Data model (if applicable)

### Core Indexed Entities

**Games**
- `game_id`: Unique identifier (e.g., GAM-0010)
- `game_name`: Display name (e.g., "60 Second Club 1")
- `levels`: Array of applicable levels (P, 1, 2, 3, 4, 5, 6)
- `skill_area`: Primary category (PITCH, RHYTM, INTVL, CHORD, SCALE, etc.)
- `skill_category`: Sub-category (Visual, Aural, Pattern Playing, etc.)
- `skill_details`: Array of specific concepts taught
- `description`: Learning objective text
- `target_scores`: Object with scores for each stage (Learn, Play, Quiz, Challenge, Review)
- `capabilities`: Object indicating MIDI support, timing, solfege variants
- `sequence_positions`: Object mapping to sequence assignments

**Sequences**
- `sequence_id`: Unique identifier (LIFE, SOL, ALFR, etc.)
- `sequence_name`: Display name
- `sequence_type`: Auto-assignable, manual, evaluation
- `levels`: Array of covered levels
- `game_assignments`: Array of game positions within sequence

**Search Metadata**
- `searchable_text`: Combined searchable content
- `tags`: Array of searchable tags
- `difficulty_score`: Numeric difficulty rating
- `popularity_score`: Usage-based ranking factor
- `last_updated`: Index freshness timestamp

## Behavior and rules

### Indexing Rules
1. **Content Extraction**: Extract searchable text from game names, descriptions, skill areas, and concept details
2. **Multi-language Support**: Index both English and Solfege variants of content
3. **Hierarchical Indexing**: Index content at both broad category and specific concept levels
4. **Real-time Updates**: Re-index content when games are updated or new content is added
5. **Incremental Updates**: Only re-index changed content to maintain performance

### Search Behavior
1. **Fuzzy Matching**: Support partial matches and typos in search queries
2. **Faceted Search**: Enable filtering by level, skill area, sequence type, and capabilities
3. **Relevance Ranking**: Prioritize results based on:
   - Exact name matches
   - Skill area relevance
   - Level appropriateness
   - User's learning progress
   - Popularity and usage patterns
4. **Auto-complete**: Provide real-time suggestions as users type
5. **Search History**: Track and suggest recent searches per user

### Content Availability Rules
1. **Permission-based Filtering**: Only index content accessible to the user's subscription level
2. **Sequence Context**: Show games in context of assigned sequences when applicable
3. **Level Appropriateness**: Weight results based on user's current level and progress
4. **Language Preferences**: Respect user's language selection (English vs. Solfege)

## UX requirements (if applicable)

### Search Interface Layout
- **Search Bar**: Prominent, accessible search input with clear placeholder text
- **Filter Panel**: Collapsible sidebar with level, skill area, and sequence filters
- **Results Display**: Grid/list toggle with game thumbnails, names, and key metadata
- **Pagination**: Load more functionality for large result sets
- **Sort Options**: By relevance, name, level, difficulty, popularity

### Key States
- **Empty State**: Helpful guidance when no results found
- **Loading State**: Clear indication of search in progress
- **Error State**: Graceful handling of search failures
- **No Results**: Suggestions for alternative searches

### Accessibility Notes
- Full keyboard navigation support
- Screen reader compatible result announcements
- High contrast mode support
- Focus management during search interactions

## Telemetry

### Search Events
- `search_initiated`: User starts typing in search box
- `search_executed`: Search query submitted
- `search_result_clicked`: User clicks on search result
- `search_filter_applied`: User applies faceted filter
- `search_suggestion_clicked`: User clicks auto-complete suggestion
- `search_no_results`: Search returns empty results

### Event Properties
- `query_text`: Search query string
- `result_count`: Number of results returned
- `filters_applied`: Array of active filters
- `result_position`: Position of clicked result
- `search_duration`: Time from query to results display
- `user_level`: Current user's learning level
- `subscription_type`: User's subscription level

## Permissions

### Read Access
- **Students**: Can search games assigned to their level and accessible content
- **Teachers**: Can search all games and sequences for assignment creation
- **Admins**: Full search access across all content
- **System**: Full access for indexing and maintenance

### Write Access
- **Content Authors**: Can trigger re-indexing of their content
- **System Admins**: Can manage search index configuration
- **Automated Systems**: Can update popularity scores and usage metrics

### Override Capabilities
- **System Admins**: Can force re-indexing of specific content
- **Support Staff**: Can access search logs for troubleshooting

## Dependencies

### Content Management
- **Game Registry**: [games-registry-overview.md](../08-games-registry-and-authoring/games-registry-overview.md) for game metadata
- **Sequence Management**: [sequence-model.md](../09-sequences-and-curriculum-tools/sequence-model.md) for sequence data
- **Content Versioning**: [content-versioning.md](../07-content-architecture/content-versioning.md) for content updates

### Search Infrastructure
- **Search Architecture**: [search-architecture.md](../18-architecture/search-architecture.md) for technical implementation
- **Caching Strategy**: [caching-strategy.md](../18-architecture/caching-strategy.md) for performance optimization
- **Event Model**: [event-model.md](../15-analytics-and-reporting/event-model.md) for telemetry integration

### User Context
- **Roles and Permissions**: [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) for access control
- **User Progress**: Student learning progress for personalized results

## Open questions

1. **Search Performance**: What are the acceptable response times for different search scenarios?
2. **Index Freshness**: How frequently should the search index be updated for optimal performance vs. freshness?
3. **Personalization**: Should search results be personalized based on individual learning progress and preferences?
4. **Search Analytics**: What additional metrics should be tracked to improve search relevance?
5. **Content Relationships**: How should related games and concepts be surfaced in search results?
6. **Mobile Optimization**: What are the specific requirements for mobile search interfaces?
7. **Offline Search**: Should search functionality work when users are offline or have limited connectivity?
