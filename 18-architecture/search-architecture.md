# Search Architecture

## Purpose

This document defines the technical architecture for search and content discovery within the MLC LMS platform. It specifies the search engine technology, query processing pipeline, performance optimizations, and integration patterns that enable efficient discovery of 450+ music learning games, sequences, and educational content at scale.

## Scope

**Included:**
- Search engine technology selection and justification
- Query processing pipeline architecture
- Index structure and management strategies
- Performance optimization techniques (caching, query optimization)
- Scalability design and capacity planning
- Search API architecture and integration patterns
- Real-time indexing and synchronization
- Search result ranking algorithms
- Monitoring and observability for search operations

**Excluded:**
- Content indexing data model and rules (see [Search Indexing](../11-search-and-discovery/search-indexing.md))
- Filter and facet UI/UX design (see [Filters and Facets](../11-search-and-discovery/filters-and-facets.md))
- Teacher-specific search workflows (see [Teacher Quick Find](../11-search-and-discovery/teacher-quick-find.md))
- Database schema and data model (see [Data Model ERD](data-model-erd.md))
- General API design patterns (see [API Design Principles](api-design-principles.md))

---

## Search Technology Stack

### Primary Search Engine: PostgreSQL Full-Text Search

**Technology Choice**: PostgreSQL native full-text search capabilities with `tsvector` and `tsquery` types.

**Rationale**:
1. **Simplicity**: Leverages existing Neon PostgreSQL infrastructure without additional services
2. **Consistency**: Single source of truth with no synchronization lag between database and search index
3. **Cost-Efficiency**: No additional search service licensing or hosting costs
4. **Scale Appropriateness**: Adequate for 450+ games with predictable query patterns
5. **Development Velocity**: Faster iteration without managing separate search infrastructure
6. **ACID Guarantees**: Search index updates are transactional with data changes

**Limitations Acknowledged**:
- Less flexible than dedicated search engines (Elasticsearch, Algolia) for complex linguistic analysis
- Limited support for advanced features like ML-based ranking, geospatial search
- Scalability ceiling at very high query volumes (acceptable for current scale)

**Future Migration Path**: If search requirements grow beyond PostgreSQL capabilities (e.g., >10K queries/second, advanced ML ranking), architecture supports migration to Elasticsearch or Algolia with minimal API surface changes.

### Complementary Technologies

**Caching Layer**: Redis-compatible cache (Upstash Redis) for frequently accessed search results
- Cache search results for popular queries (TTL: 5 minutes)
- Cache facet counts and filter combinations (TTL: 15 minutes)
- Invalidate cache on content updates

**CDN Edge Caching**: Cloudflare caching for static search assets
- Auto-complete dictionaries
- Popular search term lists
- Search UI components and assets

---

## Search Architecture Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│                     CLIENT APPLICATIONS                              │
│  - Student Dashboard  - Teacher Quick Find  - Admin Content Tools   │
└────────────────────────────┬─────────────────────────────────────────┘
                             │
                             │ HTTPS/REST
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    NEXT.JS API ROUTES                                │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │         /api/search/* - Search Endpoints                      │  │
│  │  - /api/search/games       - Game search                      │  │
│  │  - /api/search/sequences   - Sequence search                  │  │
│  │  - /api/search/autocomplete - Auto-complete suggestions       │  │
│  │  - /api/search/facets      - Facet counts                     │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                             │                                         │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │         SEARCH SERVICE LAYER                                  │  │
│  │  - Query parsing & validation                                 │  │
│  │  - Filter application logic                                   │  │
│  │  - Ranking algorithm execution                                │  │
│  │  - Result formatting & pagination                             │  │
│  │  - Permission filtering                                       │  │
│  └──────────┬────────────────────────────────────┬───────────────┘  │
└─────────────┼────────────────────────────────────┼───────────────────┘
              │                                    │
              ▼                                    ▼
┌──────────────────────────┐       ┌──────────────────────────┐
│   REDIS CACHE            │       │   NEON POSTGRESQL        │
│   (Upstash Redis)        │       │                          │
│                          │       │  ┌────────────────────┐  │
│  - Query result cache    │       │  │  Search Indices    │  │
│  - Facet count cache     │       │  │  - games_fts_idx   │  │
│  - Autocomplete cache    │       │  │  - sequences_fts   │  │
│  - Popular queries       │       │  │  - skill_gin_idx   │  │
│                          │       │  └────────────────────┘  │
│  TTL: 5-15 minutes       │       │                          │
└──────────────────────────┘       │  ┌────────────────────┐  │
                                   │  │  Base Tables       │  │
                                   │  │  - games           │  │
                                   │  │  - sequences       │  │
                                   │  │  - game_metadata   │  │
                                   │  └────────────────────┘  │
                                   └──────────────────────────┘
```

---

## Query Processing Pipeline

### 1. Query Reception and Validation

**API Endpoint**: `GET /api/search/games`

**Query Parameters**:
```typescript
interface SearchQueryParams {
  q?: string;                    // Search query text
  level?: string[];              // Filter by level (P, 1-6)
  skillArea?: string[];          // Filter by skill area
  difficulty?: { min?: number; max?: number };
  capabilities?: string[];       // MIDI, solfege variants
  page?: number;                 // Pagination
  limit?: number;                // Results per page (max 100)
  sort?: string;                 // Sort field
  order?: 'asc' | 'desc';        // Sort direction
}
```

**Validation Steps**:
1. Parse and sanitize query parameters
2. Validate filter combinations (e.g., level values are valid)
3. Apply rate limiting per user role (see [Rate Limiting and Abuse](rate-limiting-and-abuse.md))
4. Extract user context (role, subscription level, student level)
5. Check cache for identical query (cache key includes user context)

### 2. Permission Filtering

**Access Control Integration**:
- Extract user role from session (see [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md))
- Apply subscription-level content filtering
- Filter games by user's accessible levels and sequences
- Apply content visibility rules (draft, published, archived)

**Permission Query Fragment**:
```sql
-- Applied to all search queries
AND games.status = 'published'
AND games.subscription_tier <= user.subscription_tier
AND (games.levels && user.accessible_levels OR user.role = 'admin')
```

### 3. Full-Text Search Query Construction

**PostgreSQL Full-Text Search Implementation**:

**Index Structure**:
```sql
-- Games full-text search index
CREATE INDEX games_fts_idx ON games 
USING GIN(to_tsvector('english', 
  coalesce(game_name, '') || ' ' ||
  coalesce(description, '') || ' ' ||
  coalesce(skill_details_text, '')
));

-- Skill area and metadata GIN index for fast filtering
CREATE INDEX games_metadata_gin_idx ON games 
USING GIN(skill_area, levels, capabilities);

-- Composite index for common filter combinations
CREATE INDEX games_level_skill_idx ON games (level, skill_area, difficulty_rating);
```

**Query Translation**:
```typescript
// Text query: "rhythm quarter notes"
// Translated to PostgreSQL:
const tsquery = `to_tsquery('english', 'rhythm:* & quarter:* & notes:*')`;

const searchQuery = `
  SELECT 
    games.*,
    ts_rank(games.searchable_tsvector, to_tsquery('english', $1)) AS relevance_score
  FROM games
  WHERE 
    games.searchable_tsvector @@ to_tsquery('english', $1)
    AND games.status = 'published'
    AND ($2::text[] IS NULL OR games.levels && $2::text[])
    AND ($3::text[] IS NULL OR games.skill_area = ANY($3))
  ORDER BY relevance_score DESC, games.popularity_score DESC
  LIMIT $4 OFFSET $5
`;
```

**Query Optimization Techniques**:
1. **Prefix Matching**: Use `:*` suffix for partial word matches (e.g., `rhythm:*` matches "rhythm", "rhythmic")
2. **Phrase Boosting**: Exact phrase matches receive higher relevance scores
3. **Stemming**: Automatic word stemming (e.g., "playing" matches "play")
4. **Stop Words**: Common words (a, the, is) automatically filtered
5. **Index-Only Scans**: Leverage covering indexes to avoid table lookups

### 4. Filter Application

**Multi-Dimensional Filtering**:

**Filter Logic**:
- Filters within same dimension use OR logic (level:1 OR level:2)
- Filters across dimensions use AND logic (level:1 AND skill_area:RHYTM)

**SQL Implementation**:
```sql
-- Level filter (OR logic within dimension)
AND (levels && ARRAY['1', '2']::text[])

-- Skill area filter (OR logic within dimension)
AND skill_area IN ('RHYTM', 'STAFF')

-- Difficulty range filter
AND difficulty_rating BETWEEN 3 AND 7

-- Capabilities filter (all required capabilities must be present)
AND capabilities @> ARRAY['midi_support', 'solfege_variant']::text[]
```

**Facet Count Calculation**:
```sql
-- Calculate facet counts for current query
SELECT 
  skill_area,
  COUNT(*) as count
FROM games
WHERE <existing_filters_except_skill_area>
GROUP BY skill_area;
```

### 5. Ranking and Relevance Scoring

**Composite Ranking Algorithm**:

```typescript
// Relevance score = weighted sum of multiple factors
const relevanceScore = 
  (0.4 * textRelevanceScore) +      // PostgreSQL ts_rank score
  (0.2 * levelAppropriateness) +    // Matches user's level
  (0.15 * popularityScore) +        // Usage frequency
  (0.15 * recentUsageBoost) +       // Recently played by user
  (0.1 * sequenceContextBoost);     // Part of user's current sequence
```

**Ranking Factors**:

1. **Text Relevance** (40% weight):
   - PostgreSQL `ts_rank()` score
   - Exact name matches boosted by 2x
   - Description matches boosted by 1.5x

2. **Level Appropriateness** (20% weight):
   - Exact level match: score = 1.0
   - Adjacent level: score = 0.7
   - Other levels: score = 0.3

3. **Popularity Score** (15% weight):
   - Based on global play counts and effectiveness metrics
   - Normalized to 0-1 scale
   - Updated daily via background job

4. **Recent Usage Boost** (15% weight):
   - User's recently played games boosted
   - Exponential decay: boost = exp(-days_since_play / 7)

5. **Sequence Context Boost** (10% weight):
   - Games in user's active sequences boosted
   - Next recommended game in sequence: score = 1.0
   - Other games in sequence: score = 0.5

**SQL Implementation**:
```sql
SELECT
  games.*,
  (
    0.4 * ts_rank(searchable_tsvector, query) +
    0.2 * CASE 
      WHEN $user_level = ANY(levels) THEN 1.0
      WHEN $user_level - 1 = ANY(levels) OR $user_level + 1 = ANY(levels) THEN 0.7
      ELSE 0.3
    END +
    0.15 * (popularity_score / 100.0) +
    0.15 * COALESCE(exp(-(EXTRACT(EPOCH FROM (NOW() - last_played_by_user)) / 86400) / 7), 0) +
    0.1 * COALESCE(sequence_position_boost, 0)
  ) AS final_relevance_score
FROM games
ORDER BY final_relevance_score DESC;
```

### 6. Result Formatting and Response

**Response Structure** (follows [API Design Principles](api-design-principles.md)):
```json
{
  "data": [
    {
      "gameId": "GAM-0010",
      "gameName": "60 Second Club 1",
      "skillArea": "RHYTM",
      "level": "1",
      "difficultyRating": 4,
      "thumbnailUrl": "https://cdn.mlc.com/games/GAM-0010/thumb.webp",
      "concepts": ["quarter_notes", "time_signatures"],
      "targetScores": {"learn": 80, "play": 85, "quiz": 90},
      "relevanceScore": 0.95,
      "popularityScore": 87,
      "capabilities": ["midi_support"]
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3,
    "hasNext": true,
    "hasPrevious": false
  },
  "facets": {
    "skillArea": [
      {"value": "RHYTM", "label": "Rhythm", "count": 23},
      {"value": "STAFF", "label": "Staff Reading", "count": 15}
    ],
    "level": [
      {"value": "1", "label": "Level 1", "count": 18},
      {"value": "2", "label": "Level 2", "count": 20}
    ]
  },
  "meta": {
    "queryTime": 45,
    "cached": false,
    "timestamp": "2025-01-27T10:00:00Z"
  }
}
```

---

## Caching Strategy

### Multi-Layer Cache Architecture

**Layer 1: Application Memory Cache**
- In-memory LRU cache for frequently accessed search metadata
- Cache size: 100MB per instance
- TTL: 5 minutes
- Cached items: Auto-complete dictionaries, popular queries, skill taxonomies

**Layer 2: Redis Distributed Cache**
- Shared cache across all Next.js instances (Upstash Redis)
- TTL: 5-15 minutes based on content type
- Cache keys include user context hash for permission-aware caching

**Layer 3: CDN Edge Cache**
- Cloudflare edge caching for static search assets
- TTL: 1 hour to 24 hours
- Cached items: Search UI components, filter metadata, static taxonomies

### Cache Key Strategy

```typescript
// Cache key structure
const cacheKey = generateSearchCacheKey({
  queryType: 'game_search',
  queryText: normalizeQuery(params.q),
  filters: sortFilters(params.filters),
  userContext: hashUserContext(user.role, user.subscriptionTier),
  page: params.page,
  limit: params.limit
});

// Example: "search:games:rhythm:level=1,2:skill=RHYTM:role=teacher:sub=premium:p1:l20"
```

### Cache Invalidation

**Trigger Events**:
1. **Content Updates**: Game or sequence modified/created
   - Invalidate all search caches for affected content
   - Invalidate facet count caches
   - Use PostgreSQL NOTIFY/LISTEN for real-time invalidation

2. **User Context Changes**: Subscription level change, role change
   - Invalidate user-specific cached results
   - Use user session ID to target invalidation

3. **Scheduled Invalidation**: Popularity scores updated (daily)
   - Invalidate all cached results with popularity-based ranking
   - Execute during low-traffic periods (2 AM UTC)

**Invalidation Implementation**:
```typescript
// PostgreSQL trigger for content updates
CREATE OR REPLACE FUNCTION notify_search_cache_invalidation()
RETURNS TRIGGER AS $$
BEGIN
  PERFORM pg_notify('search_cache_invalidate', json_build_object(
    'entity_type', TG_TABLE_NAME,
    'entity_id', NEW.id,
    'action', TG_OP
  )::text);
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER games_search_cache_invalidate
AFTER INSERT OR UPDATE OR DELETE ON games
FOR EACH ROW EXECUTE FUNCTION notify_search_cache_invalidation();
```

**Cache Warming**:
- Pre-populate cache with top 100 most common search queries
- Execute during deployment and after cache clears
- Background job runs every 4 hours to refresh popular queries

---

## Real-Time Indexing

### Synchronous Index Updates

**On Content Create/Update**:
```typescript
// Within transaction
async function createGame(gameData: GameInput) {
  const client = await db.getClient();
  try {
    await client.query('BEGIN');
    
    // Insert game
    const game = await client.query(`
      INSERT INTO games (game_name, description, skill_details, ...)
      VALUES ($1, $2, $3, ...)
      RETURNING *
    `, [gameData.name, gameData.description, ...]);
    
    // Update full-text search vector (automatic via generated column)
    // No separate action needed - searchable_tsvector is GENERATED column
    
    await client.query('COMMIT');
    
    // Async: invalidate search cache
    await invalidateSearchCache({ gameId: game.id });
    
    return game;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}
```

**Generated Column for FTS**:
```sql
-- Automatic full-text search vector generation
ALTER TABLE games ADD COLUMN searchable_tsvector tsvector
GENERATED ALWAYS AS (
  to_tsvector('english',
    coalesce(game_name, '') || ' ' ||
    coalesce(description, '') || ' ' ||
    coalesce(skill_details_text, '') || ' ' ||
    coalesce(array_to_string(concept_tags, ' '), '')
  )
) STORED;
```

### Asynchronous Index Optimization

**Background Jobs** (see [Background Jobs](background-jobs.md)):

1. **Index Maintenance** (daily at 2 AM UTC):
   - `VACUUM ANALYZE` on games and sequences tables
   - `REINDEX` GIN indices if fragmentation detected
   - Update index statistics for query planner

2. **Popularity Score Updates** (daily at 3 AM UTC):
   - Recalculate game popularity scores based on recent play data
   - Update denormalized popularity_score column
   - Trigger search cache invalidation

3. **Synonym Dictionary Updates** (weekly):
   - Update PostgreSQL text search dictionaries with new terms
   - Add commonly searched but misspelled terms

---

## Auto-Complete Implementation

### Endpoint: `GET /api/search/autocomplete`

**Query Parameters**:
```typescript
interface AutoCompleteParams {
  q: string;           // Partial query (min 2 chars)
  limit?: number;      // Max suggestions (default 10, max 20)
  types?: string[];    // ['game', 'skill', 'concept']
}
```

**Implementation Strategy**:

**1. Prefix Matching with GIN Index**:
```sql
-- Trigram index for fast prefix matching
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE INDEX games_name_trgm_idx ON games 
USING GIN(game_name gin_trgm_ops);

-- Autocomplete query
SELECT DISTINCT
  game_name AS suggestion,
  'game' AS type,
  similarity(game_name, $1) AS score
FROM games
WHERE 
  game_name % $1  -- Trigram similarity operator
  AND status = 'published'
ORDER BY score DESC, game_name
LIMIT 10;
```

**2. Multi-Source Suggestions**:

Combine suggestions from:
- **Game names**: Direct prefix matches
- **Skill areas**: Category and subcategory names
- **Concept tags**: Musical concepts (e.g., "quarter notes", "treble clef")
- **Search history**: User's recent searches
- **Popular queries**: Most common searches globally

**Response Structure**:
```json
{
  "suggestions": [
    {
      "text": "Rhythm Reading Level 1",
      "type": "game",
      "gameId": "GAM-0045",
      "score": 0.95
    },
    {
      "text": "Rhythm",
      "type": "skill_area",
      "count": 45
    },
    {
      "text": "rhythm quarter notes",
      "type": "recent_search"
    }
  ],
  "meta": {
    "queryTime": 12,
    "cached": true
  }
}
```

**Caching**:
- Cache autocomplete results for 15 minutes
- Cache key: `autocomplete:{query}:{userRole}`
- Pre-populate cache with top 500 search prefixes

---

## Performance Optimization

### Query Performance Targets

| Query Type | Target Response Time | Max Response Time |
|------------|---------------------|-------------------|
| Simple text search | < 50ms | 100ms |
| Text + single filter | < 75ms | 150ms |
| Text + multiple filters | < 100ms | 200ms |
| Autocomplete | < 20ms | 50ms |
| Facet count calculation | < 50ms | 100ms |

### Optimization Techniques

**1. Index Optimization**:
- Use GIN indices for full-text search and array containment
- Create composite indices for common filter combinations
- Partial indices for published/active content only
- Regular `ANALYZE` to keep statistics current

**2. Query Optimization**:
```sql
-- Use EXPLAIN ANALYZE to validate query plans
EXPLAIN (ANALYZE, BUFFERS) 
SELECT ... FROM games WHERE ...;

-- Ensure index scans (not sequential scans)
-- Target: Index Scan or Bitmap Index Scan
```

**3. Connection Pooling**:
- Use Neon's connection pooling (Prisma with pgBouncer)
- Pool size: 20 connections for search queries
- Statement timeout: 5 seconds for search queries

**4. Result Pagination**:
- Use cursor-based pagination for deep pages (page > 10)
- Limit max page depth to prevent expensive OFFSET queries
- Cache total count separately to avoid COUNT(*) on every query

**5. Denormalization**:
- Store pre-computed popularity scores
- Cache frequently accessed game metadata
- Maintain materialized view for complex facet calculations (if needed)

### Monitoring and Alerting

**Key Metrics** (see [Observability](../19-quality-and-operations/observability.md)):

1. **Query Performance**:
   - P50, P95, P99 response times
   - Slow query log (> 200ms)
   - Query plan changes

2. **Cache Hit Rates**:
   - Redis cache hit rate (target: > 70%)
   - CDN cache hit rate (target: > 85%)

3. **Index Health**:
   - Index bloat percentage
   - Unused indices
   - Missing indices suggestions

4. **Error Rates**:
   - Search query errors
   - Timeout errors
   - Permission errors

**Alerting Thresholds**:
- P95 response time > 200ms for 5 minutes → Page on-call
- Error rate > 1% for 5 minutes → Alert team
- Cache hit rate < 50% for 10 minutes → Investigate
- Database connection pool exhaustion → Immediate alert

---

## Scalability Considerations

### Current Scale

- **Content Volume**: ~450 games, ~50 sequences
- **Query Volume**: Estimated 100-500 queries/minute at launch
- **User Base**: Projected 5,000 active users in first year
- **Data Growth**: ~50-100 new games per year

### Scaling Strategy

**Vertical Scaling (Current Approach)**:
- Neon PostgreSQL auto-scales compute based on load
- Increase database instance size as query volume grows
- PostgreSQL FTS adequate for projected scale (next 3-5 years)

**Horizontal Scaling (Future, if needed)**:

**Phase 1: Read Replicas** (> 1,000 queries/minute):
- Add Neon read replicas for search queries
- Route all search reads to replicas
- Write operations remain on primary

**Phase 2: Dedicated Search Service** (> 10,000 queries/minute):
- Migrate to Elasticsearch or Algolia
- Maintain PostgreSQL as source of truth
- Asynchronous index synchronization
- API surface remains unchanged (internal implementation swap)

**Phase 3: Distributed Search** (> 100,000 queries/minute):
- Elasticsearch cluster with multiple nodes
- Shard by content type (games, sequences) or user segment
- Geographic distribution for global users

### Capacity Planning

**Database Sizing**:
```
Current: Neon Starter (0.5 GB RAM, shared CPU)
Year 1: Neon Pro (2 GB RAM, 1 vCPU) - ~$50/month
Year 2: Neon Scale (4 GB RAM, 2 vCPU) - ~$150/month
Year 3+: Evaluate dedicated search service
```

**Cache Sizing**:
```
Current: Upstash Redis 256 MB - $10/month
Year 1: Upstash Redis 1 GB - $30/month
Year 2+: Scale based on hit rate metrics
```

---

## Integration Patterns

### Search API Service Layer

**Module Structure**:
```
/lib/services/search/
  ├── searchService.ts           # Main search orchestration
  ├── queryBuilder.ts            # SQL query construction
  ├── rankingEngine.ts           # Relevance scoring
  ├── filterEngine.ts            # Filter application
  ├── permissionFilter.ts        # Access control
  ├── cacheManager.ts            # Cache operations
  └── types.ts                   # TypeScript interfaces
```

**Service Interface**:
```typescript
interface SearchService {
  searchGames(params: GameSearchParams, userContext: UserContext): Promise<SearchResult<Game>>;
  searchSequences(params: SequenceSearchParams, userContext: UserContext): Promise<SearchResult<Sequence>>;
  autocomplete(params: AutoCompleteParams, userContext: UserContext): Promise<AutoCompleteResult>;
  getFacetCounts(params: FacetParams, userContext: UserContext): Promise<FacetCounts>;
  invalidateCache(params: CacheInvalidationParams): Promise<void>;
}
```

### Integration with Content Domain

**Domain Boundary** (see [Service Boundaries](service-boundaries.md)):
- Search service consumes Content domain data
- Content domain publishes events on content changes
- Search service subscribes to content events for cache invalidation

**Event-Driven Updates**:
```typescript
// Content domain publishes event
contentEventBus.publish({
  type: 'GAME_UPDATED',
  payload: { gameId: 'GAM-0010', fields: ['description', 'skillArea'] }
});

// Search service subscribes
contentEventBus.subscribe('GAME_UPDATED', async (event) => {
  await searchCacheManager.invalidate({ gameId: event.payload.gameId });
  await searchIndexManager.reindexGame(event.payload.gameId);
});
```

### Integration with Analytics

**Search Telemetry** (see [Event Model](../15-analytics-and-reporting/event-model.md)):

Emit events for all search interactions:
```typescript
await analytics.track({
  event: 'search_executed',
  userId: user.id,
  properties: {
    query: params.q,
    filters: params.filters,
    resultCount: results.total,
    queryTime: results.meta.queryTime,
    cached: results.meta.cached,
    page: params.page
  }
});
```

**Search Analytics Dashboard**:
- Most searched terms
- Zero-result queries (opportunities for content)
- Average result count per query
- Click-through rates by position
- Search abandonment rate

---

## Security Considerations

### Query Injection Prevention

**Parameterized Queries**:
```typescript
// NEVER: Direct string interpolation
// const query = `SELECT * FROM games WHERE name LIKE '%${userInput}%'`;

// ALWAYS: Parameterized queries
const query = `SELECT * FROM games WHERE name ILIKE $1`;
const results = await db.query(query, [`%${sanitizedInput}%`]);
```

**Input Sanitization**:
- Escape special PostgreSQL FTS characters: `&`, `|`, `!`, `(`, `)`, `:`, `*`
- Limit query length (max 200 characters)
- Validate filter values against allowed enums
- Rate limit by IP and user ID

### Permission-Based Filtering

**Enforced at Query Level**:
```typescript
async function searchGames(params: SearchParams, user: User) {
  const permissionFilter = buildPermissionFilter(user);
  
  const query = `
    SELECT * FROM games
    WHERE searchable_tsvector @@ to_tsquery($1)
      AND ${permissionFilter.sql}  -- Enforced in SQL
  `;
  
  return db.query(query, [params.q, ...permissionFilter.params]);
}
```

**Never Rely on Client-Side Filtering**:
- All permission checks executed server-side
- No client-side filtering of search results
- Unauthorized content never leaves the database

### Rate Limiting

**Per-User Limits** (see [Rate Limiting and Abuse](rate-limiting-and-abuse.md)):
- Anonymous users: 20 queries/minute
- Authenticated students: 60 queries/minute
- Teachers/admins: 120 queries/minute
- API integrations: 300 queries/minute

**Implementation**:
- Use Redis-based rate limiter (sliding window)
- Return `429 Too Many Requests` with `Retry-After` header
- Log excessive query patterns for abuse detection

---

## Open Questions

1. **Search Service Migration**: At what query volume threshold should we migrate from PostgreSQL FTS to Elasticsearch/Algolia?
2. **ML-Based Ranking**: Should we invest in machine learning models for personalized search ranking based on user behavior?
3. **Fuzzy Matching**: What level of fuzzy matching should be supported for typo tolerance (current: minimal via stemming)?
4. **Multi-Language Search**: How should we handle multi-language content search beyond English and Solfege variants?
5. **Offline Search**: Should we implement client-side search for offline/PWA scenarios with pre-downloaded indices?
6. **Search Analytics**: What additional search metrics should we track to improve relevance and user experience?
7. **Voice Search**: Should we support voice search on mobile devices, and how would it integrate with the current architecture?
8. **Semantic Search**: Should we explore semantic/vector search for concept-based discovery (e.g., "games similar to X")?

---

## Supporting Documents Referenced

This search architecture specification draws from the following source documents:

- [All Games Redesign.docx.txt](../00-ORIG-CONTEXT/All%20Games%20Redesign.docx.txt) — Game search interface requirements
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game searchable metadata
- [EXE specs 2012-0913 - Category Screen.csv](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Category%20Screen.csv) — Category and search filtering specifications

## Dependencies

### Infrastructure Dependencies
- **[System Overview](system-overview.md)**: Overall architecture and technology stack
- **[Data Model ERD](data-model-erd.md)**: Database schema and relationships
- **[Caching Strategy](caching-strategy.md)**: Cache layer implementation details
- **[API Design Principles](api-design-principles.md)**: API patterns and conventions

### Feature Dependencies
- **[Search Indexing](../11-search-and-discovery/search-indexing.md)**: Content indexing rules and data model
- **[Filters and Facets](../11-search-and-discovery/filters-and-facets.md)**: Filter UI and facet calculations
- **[Teacher Quick Find](../11-search-and-discovery/teacher-quick-find.md)**: Teacher-specific search workflows

### Security Dependencies
- **[Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md)**: Authentication context
- **[Roles Matrix](../02-roles-and-permissions/roles-matrix.md)**: Permission-based access control
- **[Rate Limiting and Abuse](rate-limiting-and-abuse.md)**: Query rate limiting

### Operational Dependencies
- **[Background Jobs](background-jobs.md)**: Index maintenance and optimization jobs
- **[Observability](../19-quality-and-operations/observability.md)**: Monitoring and alerting
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Search telemetry and analytics
