# Caching Strategy

## Purpose

This document defines the comprehensive caching strategy for the MLC LMS platform, specifying how data is cached across multiple layers to optimize performance, reduce latency, and minimize infrastructure costs. The caching strategy balances freshness requirements with performance goals, ensuring students experience fast game loading, teachers see up-to-date progress data, and the system scales efficiently under load.

A well-designed caching strategy is critical for delivering the sub-3-second Time to Interactive (TTI) and sub-500ms API response times required for a responsive learning experience while operating within serverless cost constraints.

## Scope

### Included

**Caching Layers and Infrastructure**:
- CDN/Edge caching (Cloudflare)
- Application-level caching (Next.js)
- Database query caching (Neon PostgreSQL)
- Client-side caching (React Query/SWR)
- Session and authentication caching

**Cache Policies by Data Type**:
- Static content caching (games, sequences, public pages)
- User-specific data caching (assignments, progress)
- Dynamic content caching (leaderboards, notifications)
- API response caching
- Asset caching (images, audio, game files)

**Cache Management**:
- Cache key patterns and namespacing
- Time-to-Live (TTL) policies per content type
- Cache invalidation strategies
- Cache warming and pre-loading
- Conditional requests (ETags, Last-Modified)
- Stale-while-revalidate patterns

**Performance and Monitoring**:
- Cache hit rate targets and monitoring
- Performance impact measurements
- Cost optimization through caching
- Cache debugging and troubleshooting

### Excluded

- Specific API endpoint implementations (see [REST Endpoints](rest-endpoints.md))
- Database schema and indexing details (see [Data Model ERD](data-model-erd.md))
- CDN configuration specifics (see [CDN and Media](../17-integrations/cdn-and-media.md))
- Rate limiting policies (see [Rate Limiting and Abuse](rate-limiting-and-abuse.md))
- Background job processing (see [Background Jobs](background-jobs.md))

---

## Cache Architecture Overview

### Four-Layer Caching Model

The MLC LMS implements a hierarchical caching strategy with four distinct layers, each optimized for different data characteristics and access patterns:

```
┌──────────────────────────────────────────────────────────────┐
│ Layer 1: Browser/Client Cache                                │
│ - React Query/SWR in-memory cache                            │
│ - Browser HTTP cache (Cache-Control headers)                 │
│ - Service Worker cache (future PWA)                          │
│ - TTL: 5 seconds - 5 minutes                                 │
└───────────────────────────┬──────────────────────────────────┘
                            │ Cache MISS
                            ▼
┌──────────────────────────────────────────────────────────────┐
│ Layer 2: CDN/Edge Cache (Cloudflare)                         │
│ - Static assets (images, CSS, JS)                            │
│ - Public pages (marketing, documentation)                    │
│ - Game assets (HTML5 files, sprites, audio)                  │
│ - TTL: 1 hour - 1 year                                       │
└───────────────────────────┬──────────────────────────────────┘
                            │ Cache MISS
                            ▼
┌──────────────────────────────────────────────────────────────┐
│ Layer 3: Application Cache (Next.js/Vercel)                  │
│ - API responses (user-specific data)                         │
│ - SSR page fragments                                          │
│ - Computed aggregations                                       │
│ - TTL: 30 seconds - 60 minutes                               │
└───────────────────────────┬──────────────────────────────────┘
                            │ Cache MISS
                            ▼
┌──────────────────────────────────────────────────────────────┐
│ Layer 4: Database Cache (Neon PostgreSQL)                    │
│ - Query result caching (automatic)                           │
│ - Connection pooling                                          │
│ - Prepared statement caching                                  │
└──────────────────────────────────────────────────────────────┘
```

**Cache Selection by Content Type**:

| Content Type | Layer 1 (Client) | Layer 2 (CDN) | Layer 3 (App) | Layer 4 (DB) |
|--------------|------------------|---------------|---------------|--------------|
| Game Assets | No | **Yes** (1 year) | No | N/A |
| Game Metadata | **Yes** (5 min) | **Yes** (1 hour) | **Yes** (1 hour) | Auto |
| Student Progress | **Yes** (30 sec) | No (private) | **Yes** (5 min) | Auto |
| Assignments | **Yes** (1 min) | No (private) | **Yes** (5 min) | Auto |
| Leaderboards | **Yes** (30 sec) | No | **Yes** (30 sec) | Auto |
| User Profile | **Yes** (5 min) | No (private) | **Yes** (15 min) | Auto |
| Public Pages | **Yes** (1 hour) | **Yes** (1 hour) | No (SSG) | N/A |

---

## Layer 1: Client-Side Caching

### React Query / SWR In-Memory Cache

**Purpose**: Minimize redundant API calls by caching query results in the browser's JavaScript memory.

**Implementation**: Use React Query or SWR as the data fetching library with built-in caching.

**Configuration Example** (React Query):
```typescript
// lib/react-query-config.ts
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes - data considered fresh
      cacheTime: 10 * 60 * 1000, // 10 minutes - keep in cache
      refetchOnWindowFocus: true, // Refetch when tab regains focus
      refetchOnReconnect: true, // Refetch on network reconnect
      retry: 2, // Retry failed requests twice
    },
  },
});
```

**Cache Keys and Invalidation**:
```typescript
// Query keys follow hierarchical pattern
const queryKeys = {
  students: ['students'] as const,
  student: (id: string) => ['students', id] as const,
  studentAssignments: (id: string) => ['students', id, 'assignments'] as const,
  
  games: ['games'] as const,
  game: (id: string) => ['games', id] as const,
  
  assignments: ['assignments'] as const,
  assignment: (id: string) => ['assignments', id] as const,
};

// Invalidate specific caches after mutations
await queryClient.invalidateQueries({ queryKey: queryKeys.student(studentId) });
await queryClient.invalidateQueries({ queryKey: queryKeys.studentAssignments(studentId) });
```

**Stale-While-Revalidate Pattern**:
```typescript
// Student dashboard - show cached data immediately, refetch in background
const { data: assignments, isLoading } = useQuery({
  queryKey: queryKeys.studentAssignments(studentId),
  queryFn: () => fetchStudentAssignments(studentId),
  staleTime: 30 * 1000, // Consider fresh for 30 seconds
  refetchInterval: 60 * 1000, // Auto-refetch every 60 seconds
});
```

### Browser HTTP Cache

**Purpose**: Leverage native browser caching for static assets and API responses.

**Cache-Control Headers** (set by API routes):
```typescript
// Public static content
res.setHeader('Cache-Control', 'public, max-age=31536000, immutable'); // 1 year

// User-specific API responses
res.setHeader('Cache-Control', 'private, max-age=300, must-revalidate'); // 5 minutes

// Real-time data (no cache)
res.setHeader('Cache-Control', 'no-cache, no-store, must-revalidate');
```

---

## Layer 2: CDN/Edge Caching (Cloudflare)

### Static Asset Caching

**Purpose**: Deliver static files (images, CSS, JS, game assets) from edge locations worldwide for minimal latency.

**Cached Content**:
- Next.js static assets (`/_next/static/*`) → 1 year TTL
- Game HTML5 files, sprites, audio → 1 year TTL (versioned URLs)
- Images, icons, badges → 1 year TTL (content-addressed)
- CSS, JavaScript bundles → 1 year TTL (hashed filenames)
- Fonts, WOFF2 files → 1 year TTL

**Cache Key Pattern**:
```
https://cdn.mlc-lms.com/games/G-01542/v3/game.html
                                      ^^
                                    version in URL for cache busting
```

**Cloudflare Cache Rules**:
```yaml
# cloudflare-cache-rules.yml
rules:
  - match: "/_next/static/*"
    cache_ttl: 31536000  # 1 year
    browser_ttl: 31536000
    
  - match: "/games/**/*"
    cache_ttl: 31536000  # 1 year (versioned paths)
    browser_ttl: 31536000
    
  - match: "/api/*"
    cache_ttl: 0  # Never cache API routes at CDN (private data)
    
  - match: "/assets/images/*"
    cache_ttl: 86400  # 24 hours
    browser_ttl: 86400
```

**Cache Invalidation**: When games are updated, new version URLs are generated automatically, ensuring old versions remain cached while new versions fetch fresh.

### Public Page Caching

**Marketing and Static Pages**:
- Home page → 1 hour TTL
- About, Help, Contact → 1 hour TTL
- Documentation pages → 1 hour TTL

**Implementation**: Next.js Static Site Generation (SSG) with Incremental Static Regeneration (ISR).

```typescript
// pages/index.tsx (Marketing home page)
export async function getStaticProps() {
  return {
    props: { /* data */ },
    revalidate: 3600, // Regenerate page every 1 hour
  };
}
```

---

## Layer 3: Application-Level Caching (Next.js)

### API Response Caching

**Purpose**: Cache computed data and aggregations at the application layer to reduce database load.

**Cache Store Options**:
1. **In-Memory Cache** (Node.js Map) - for single-instance development
2. **Vercel Edge Config** - for serverless edge caching (future)
3. **Redis** (Upstash) - for distributed caching (future when needed)

**Initial Implementation** (In-Memory Cache):
```typescript
// lib/cache/api-cache.ts
import { LRUCache } from 'lru-cache';

const apiCache = new LRUCache<string, any>({
  max: 500, // Maximum 500 entries
  ttl: 5 * 60 * 1000, // 5 minutes default TTL
  updateAgeOnGet: true, // Reset TTL on access
  allowStale: false,
});

export async function cachedFetch<T>(
  cacheKey: string,
  fetcher: () => Promise<T>,
  ttl?: number
): Promise<T> {
  // Check cache first
  const cached = apiCache.get(cacheKey);
  if (cached) return cached as T;
  
  // Cache miss - fetch data
  const data = await fetcher();
  apiCache.set(cacheKey, data, { ttl: ttl ?? 5 * 60 * 1000 });
  return data;
}
```

**Usage in API Routes**:
```typescript
// pages/api/games/[gameId].ts
import { cachedFetch } from '@/lib/cache/api-cache';

export default async function handler(req, res) {
  const { gameId } = req.query;
  
  const game = await cachedFetch(
    `game:${gameId}`,
    () => db.game.findUnique({ where: { id: gameId } }),
    60 * 60 * 1000 // 1 hour TTL for game metadata
  );
  
  res.setHeader('Cache-Control', 'public, max-age=3600, s-maxage=3600');
  res.json({ data: game });
}
```

### Cached Data Types and TTLs

**Content (Read-Heavy, Rarely Updated)**:
- Game metadata → 1 hour TTL
- Sequence definitions → 1 hour TTL
- Badge definitions → 1 hour TTL
- Taxonomy and categories → 24 hours TTL

**User Data (Moderately Dynamic)**:
- Student profile → 15 minutes TTL
- Teacher profile → 15 minutes TTL
- User preferences → 15 minutes TTL

**Progress and Analytics (Dynamic, Tolerate Slight Staleness)**:
- Student assignment list → 5 minutes TTL
- Class progress summary → 5 minutes TTL
- Leaderboard rankings → 30 seconds TTL

**Real-Time (No Cache or Very Short)**:
- Current assignment step → No cache (query fresh)
- Live notifications count → 10 seconds TTL
- Active session data → No cache

### Cache Invalidation Patterns

**Time-Based Expiration** (default): Cache entries expire after TTL.

**Event-Based Invalidation**: Invalidate specific cache keys on mutations.

```typescript
// After updating a game
await apiCache.delete(`game:${gameId}`);
await apiCache.delete(`games:list`); // Invalidate list cache

// After student completes assignment
await apiCache.delete(`student:${studentId}:assignments`);
await apiCache.delete(`teacher:${teacherId}:class-progress`);
```

**Tag-Based Invalidation** (future with Redis):
```typescript
// Tag caches for batch invalidation
await cache.set(`game:${gameId}`, data, { tags: ['games', `game:${gameId}`] });
await cache.invalidateByTag('games'); // Invalidate all game caches
```

---

## Layer 4: Database Query Caching (Neon PostgreSQL)

### Automatic Query Result Caching

**Neon PostgreSQL Features**:
- Automatic query plan caching for repeated queries
- Connection pooling reduces connection overhead
- Prepared statement caching
- Internal buffer cache for frequently accessed pages

**Optimization Strategy**:
- Use prepared statements with parameterized queries (prevents SQL injection and enables caching)
- Index frequently queried fields (see [Data Model ERD](data-model-erd.md))
- Avoid SELECT * queries (reduces data transfer and parsing)

**Connection Pooling** (via Prisma or pg-pool):
```typescript
// lib/db.ts
import { Pool } from 'pg';

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20, // Maximum 20 concurrent connections
  idleTimeoutMillis: 30000, // Close idle connections after 30s
  connectionTimeoutMillis: 2000, // Fail fast if pool exhausted
});

export { pool };
```

**Query Optimization**:
```typescript
// BAD: Uncached, inefficient query
const students = await db.query('SELECT * FROM students WHERE org_id = $1', [orgId]);

// GOOD: Indexed field, prepared statement, specific columns
const students = await db.query(
  'SELECT id, first_name, last_name, email FROM students WHERE org_id = $1 AND status = $2',
  [orgId, 'active']
);
```

---

## Conditional Requests and ETags

### ETag Implementation

**Purpose**: Enable conditional requests to reduce bandwidth and server load for unchanged resources.

**ETag Generation**:
```typescript
// lib/cache/etag.ts
import crypto from 'crypto';

export function generateETag(data: any): string {
  const hash = crypto
    .createHash('sha256')
    .update(JSON.stringify(data))
    .digest('hex');
  return `"${hash.substring(0, 16)}"`;
}
```

**API Route with ETag Support**:
```typescript
// pages/api/games/[gameId].ts
import { generateETag } from '@/lib/cache/etag';

export default async function handler(req, res) {
  const game = await fetchGame(req.query.gameId);
  const etag = generateETag(game);
  
  res.setHeader('ETag', etag);
  res.setHeader('Cache-Control', 'public, max-age=3600');
  
  // Check if client already has latest version
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end(); // Not Modified
  }
  
  res.json({ data: game });
}
```

### Last-Modified Headers

**Alternative to ETags** for time-based caching:
```typescript
res.setHeader('Last-Modified', game.updatedAt.toUTCString());

if (req.headers['if-modified-since']) {
  const clientDate = new Date(req.headers['if-modified-since']);
  if (game.updatedAt <= clientDate) {
    return res.status(304).end();
  }
}
```

---

## Cache Key Patterns and Namespacing

### Standardized Cache Key Format

**Pattern**: `{domain}:{entity}:{id}:{scope}`

**Examples**:
- `game:G-01542` → Single game metadata
- `game:G-01542:stages` → Game stages
- `student:stu_123:assignments` → Student's assignments
- `teacher:tch_456:class:cls_789:progress` → Class progress for teacher
- `org:org_001:students` → All students in organization
- `leaderboard:challenge:chal_999` → Challenge leaderboard

**Benefits**:
- Predictable key structure for debugging
- Easy wildcard invalidation (e.g., invalidate all `student:stu_123:*`)
- Namespace isolation prevents key collisions

### Cache Versioning

**Global Cache Version** for breaking changes:
```typescript
const CACHE_VERSION = 'v2';
const cacheKey = `${CACHE_VERSION}:game:${gameId}`;
```

When data model changes require cache bust, increment `CACHE_VERSION`.

---

## Performance Targets and Monitoring

### Cache Hit Rate Targets

**By Layer**:
- CDN Cache: > 95% hit rate for static assets
- Application Cache: > 80% hit rate for API responses
- Client Cache: > 90% hit rate for repeated UI queries

**Measurement**:
```typescript
// Instrument cache with hit/miss tracking
let cacheHits = 0;
let cacheMisses = 0;

function getCacheHitRate() {
  const total = cacheHits + cacheMisses;
  return total > 0 ? (cacheHits / total) * 100 : 0;
}
```

### Performance Metrics

**API Response Time** (with caching):
- Cached response (hit): < 50ms (p95)
- Uncached response (miss): < 500ms (p95)

**Page Load Time**:
- First Contentful Paint (FCP): < 1.5s
- Time to Interactive (TTI): < 3s
- Largest Contentful Paint (LCP): < 2.5s

See [System Overview](system-overview.md) for comprehensive performance targets.

### Cache Size and Memory Limits

**Client Cache** (React Query):
- Default: 50MB memory limit
- Automatic garbage collection when limit exceeded

**Application Cache** (In-Memory):
- LRU eviction policy (max 500 entries initially)
- Monitor heap usage via Node.js metrics

**Future Redis Cache**:
- Max 1GB cache size
- TTL-based eviction
- Persistence to disk for warm restarts

---

## Cache Warming Strategies

### Pre-Loading Critical Data

**On Application Start**:
```typescript
// lib/cache/warming.ts
export async function warmCache() {
  // Pre-load popular games
  const popularGames = await db.game.findMany({
    where: { status: 'live' },
    orderBy: { playCount: 'desc' },
    take: 50,
  });
  
  popularGames.forEach(game => {
    apiCache.set(`game:${game.id}`, game, { ttl: 60 * 60 * 1000 });
  });
  
  // Pre-load badge definitions
  const badges = await db.badge.findMany();
  apiCache.set('badges:all', badges, { ttl: 24 * 60 * 60 * 1000 });
}
```

**Scheduled Warm-Up** (via background job):
```typescript
// Run cache warming every 6 hours
cron.schedule('0 */6 * * *', async () => {
  await warmCache();
  console.log('Cache warmed successfully');
});
```

### Predictive Pre-Fetching

**Client-Side** (React Query):
```typescript
// Pre-fetch next assignment when current one is opened
const queryClient = useQueryClient();

queryClient.prefetchQuery({
  queryKey: queryKeys.assignment(nextAssignmentId),
  queryFn: () => fetchAssignment(nextAssignmentId),
});
```

---

## Cache Invalidation Strategies

### Invalidation Triggers

**1. Time-Based Expiration** (default for all caches)
- TTL expires, cache entry removed automatically

**2. Mutation-Based Invalidation**
- After POST/PATCH/DELETE operations, invalidate affected caches

```typescript
// After updating student profile
await updateStudentProfile(studentId, updates);
await invalidateCache([
  `student:${studentId}`,
  `student:${studentId}:assignments`,
  `teacher:${teacherId}:students`,
]);
```

**3. Event-Based Invalidation**
- Listen to application events and invalidate related caches

```typescript
// Event emitter pattern
eventBus.on('game.published', async (event) => {
  await invalidateCache([
    `game:${event.gameId}`,
    'games:list',
    `sequence:${event.sequenceId}:games`,
  ]);
});
```

**4. Manual Invalidation** (System Admin)
- Admin panel to manually purge specific caches
- Useful for debugging or emergency cache clearing

### Stale-While-Revalidate

**Pattern**: Serve stale cache while fetching fresh data in background.

```typescript
// API route with stale-while-revalidate
res.setHeader('Cache-Control', 'public, max-age=300, stale-while-revalidate=600');
// Serve cached version for 5 minutes, revalidate in background for up to 10 minutes
```

**React Query Implementation**:
```typescript
useQuery({
  queryKey: ['games'],
  queryFn: fetchGames,
  staleTime: 5 * 60 * 1000, // 5 minutes
  cacheTime: 10 * 60 * 1000, // 10 minutes
  refetchInterval: 5 * 60 * 1000, // Background refetch every 5 minutes
});
```

---

## Caching Best Practices

### Do's

✅ **Cache static content aggressively** (games, sequences, badges) with long TTLs
✅ **Use versioned URLs** for cache busting (e.g., `/games/G-01542/v3/game.html`)
✅ **Set appropriate Cache-Control headers** for public vs. private data
✅ **Implement ETags** for conditional requests on frequently accessed resources
✅ **Monitor cache hit rates** and optimize based on metrics
✅ **Use hierarchical cache keys** for easy invalidation (e.g., `student:123:*`)
✅ **Pre-load critical data** (popular games, badge definitions) on startup
✅ **Invalidate related caches** after mutations to maintain consistency

### Don'ts

❌ **Don't cache sensitive data at CDN layer** (use `Cache-Control: private` for user-specific data)
❌ **Don't cache error responses** (always fetch fresh to detect recovery)
❌ **Don't use overly long TTLs** for dynamic data (progress, leaderboards) - staleness hurts UX
❌ **Don't cache authenticated API responses at CDN** (risk leaking private data)
❌ **Don't forget to set `Vary` headers** when response varies by request header (e.g., `Vary: Accept-Language`)
❌ **Don't cache CSRF tokens or nonces** (security risk)
❌ **Don't ignore cache size limits** - implement LRU eviction to prevent memory bloat

---

## Cost Optimization Through Caching

### Infrastructure Cost Savings

**Reduced Database Queries**:
- Without cache: 10,000 requests/day → 10,000 DB queries
- With 80% hit rate: 10,000 requests/day → 2,000 DB queries
- **Cost Reduction**: 80% fewer database round-trips

**Reduced Serverless Function Invocations**:
- CDN cache hit → No function invocation
- 95% CDN hit rate for assets → 95% fewer function calls for static content

**Bandwidth Savings**:
- ETag / 304 Not Modified responses → No data transfer
- Estimated 30-40% bandwidth reduction for repeat visitors

### Monitoring Cost Impact

**Track Metrics**:
```typescript
const metrics = {
  dbQueriesAvoided: cacheHits, // Queries saved by cache
  bandwidthSaved: notModifiedResponses * avgResponseSize,
  functionInvocationsSaved: cdnHits,
};
```

See [System Overview](system-overview.md) for cost estimates at various scales.

---

## Debugging and Troubleshooting

### Cache Headers for Debugging

**Add debug headers in development**:
```typescript
if (process.env.NODE_ENV === 'development') {
  res.setHeader('X-Cache-Status', cacheHit ? 'HIT' : 'MISS');
  res.setHeader('X-Cache-Key', cacheKey);
  res.setHeader('X-Cache-TTL', ttl.toString());
}
```

### Cache Inspection Tools

**Browser DevTools**:
- Network tab → Check Cache-Control, ETag headers
- Application tab → Inspect Cache Storage (Service Worker)

**Cloudflare Dashboard**:
- Analytics → Caching tab → Cache hit rate, bandwidth saved
- Cache Rules → Review and adjust TTLs

**Application Logging**:
```typescript
logger.debug('Cache operation', {
  operation: 'get',
  cacheKey: key,
  hit: !!cached,
  ttl: cached?.ttl,
});
```

### Common Issues and Solutions

**Issue**: Cache hit rate lower than expected
- **Solution**: Check TTLs, ensure cache keys are consistent, investigate query parameter variations

**Issue**: Stale data served to users
- **Solution**: Reduce TTLs, implement event-based invalidation, use stale-while-revalidate

**Issue**: Memory pressure from cache
- **Solution**: Implement LRU eviction, reduce max cache size, migrate to Redis

**Issue**: Cache inconsistency after deployments
- **Solution**: Bump global cache version, implement cache warming on deploy

---

## Supporting Documents Referenced

This caching strategy specification draws from the following source documents:

- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — Admin panel requirements informing cache invalidation controls and system monitoring dashboards
- [System Admin Re-design B.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design%20B.docx.txt) — Additional system admin tool specifications for cache management features
- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser compatibility considerations informing CDN caching strategies and HTTP header requirements
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Public page content specifications informing static content caching policies and TTL configurations

---

## Future Enhancements

### Short-Term (Phase 2)

1. **Upstash Redis Integration**:
   - Distributed cache for multi-region deployments
   - Persistent cache across serverless function restarts
   - Pub/sub for cache invalidation across instances

2. **Cache Analytics Dashboard**:
   - Real-time cache hit rate visualization
   - Cache key usage heatmap
   - Performance impact analysis

3. **Advanced Invalidation**:
   - Tag-based batch invalidation
   - Dependency tracking for automatic invalidation

### Long-Term (Phase 3)

1. **Service Worker Caching** (PWA):
   - Offline-first game playing
   - Background sync for progress data
   - Pre-cache critical assets

2. **GraphQL Cache Normalization**:
   - If migrating to GraphQL, implement Apollo Client normalized cache
   - Automatic cache updates on mutations

3. **Edge Compute Functions**:
   - Move cache logic to Cloudflare Workers
   - Ultra-low latency cache hits at edge

---

## Related Documentation

**Architecture**:
- [System Overview](system-overview.md) - Overall architecture and performance targets
- [API Design Principles](api-design-principles.md) - Cache-Control headers and conditional requests
- [Service Boundaries](service-boundaries.md) - Domain boundaries and data ownership
- [Data Model ERD](data-model-erd.md) - Database schema and indexing for query performance
- [Background Jobs](background-jobs.md) - Cache warming via scheduled jobs

**Infrastructure**:
- [CDN and Media](../17-integrations/cdn-and-media.md) - Cloudflare R2 and CDN configuration
- [Rate Limiting and Abuse](rate-limiting-and-abuse.md) - Rate limiting integration with caching
- [Security & Privacy](security-privacy.md) - Caching sensitive data safely

**Operations**:
- [Observability](../19-quality-and-operations/observability.md) - Cache metrics and monitoring
- [Performance and Scalability](performance-and-scalability.md) - Performance optimization strategies

---

## Open Questions

### Near-Term Decisions

1. **Redis vs. Vercel Edge Config**: Which distributed cache should we adopt when in-memory cache becomes insufficient?
2. **Cache Invalidation Pub/Sub**: How do we coordinate cache invalidation across multiple serverless function instances?
3. **Service Worker Strategy**: When do we implement Service Worker caching for offline support?
4. **Cache Key Versioning**: Should we version cache keys per deployment to force cache refresh on releases?

### Long-Term Considerations

1. **Multi-Region Caching**: As the platform goes global, how do we handle cache consistency across regions?
2. **Real-Time Cache Updates**: Should we implement WebSocket-based cache invalidation for real-time features like leaderboards?
3. **Personalized Cache**: How do we cache personalized content (recommended games, adaptive learning paths) efficiently?
4. **Cache Warming Automation**: Can we use ML to predict which data to pre-load based on usage patterns?

---

## Summary

The MLC LMS caching strategy implements a four-layer hierarchical cache system optimized for the platform's unique characteristics:

- **Static content** (games, assets) cached aggressively at CDN with 1-year TTLs
- **Dynamic content** (progress, assignments) cached at application layer with 5-minute TTLs
- **User-specific data** cached with `private` headers to prevent CDN caching
- **Real-time data** (leaderboards, notifications) cached briefly (30 seconds) or not at all

**Key Benefits**:
- **Performance**: Sub-500ms API responses, sub-3s page loads
- **Cost Efficiency**: 80% reduction in database queries, 95% CDN hit rate for assets
- **Scalability**: Handles traffic spikes without database load
- **User Experience**: Fast game loading, responsive dashboards

By balancing freshness requirements with performance goals, the caching strategy ensures students experience instant game launches, teachers see timely progress updates, and the system scales cost-effectively from hundreds to hundreds of thousands of users.
