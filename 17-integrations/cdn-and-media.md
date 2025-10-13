# CDN and Media

## Purpose

This specification defines the Content Delivery Network (CDN) infrastructure and media delivery architecture for the MLC LMS platform. It establishes how static assets, media files, and dynamic content are stored, cached, secured, and delivered globally with optimal performance and reliability.

> **Related Documents**: For game-specific asset management and upload workflows, see [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md). For caching strategy details, see [Caching Strategy](../18-architecture/caching-strategy.md). For security policies, see [Security and Privacy](../18-architecture/security-privacy.md). For technology choices, see [Technology Stack](../00-foundations/techstack.md).

The CDN and media infrastructure enables:
- **Global low-latency access**: Students and teachers worldwide experience fast page loads and media playback regardless of geographic location
- **Scalable content delivery**: Platform automatically handles traffic spikes during peak usage times without performance degradation
- **Cost optimization**: Leveraging Cloudflare R2's zero-egress-fee model significantly reduces bandwidth costs compared to traditional object storage
- **Security and access control**: Signed URLs, hotlink protection, and DDoS mitigation protect content and prevent unauthorized access
- **Reliability and uptime**: Multi-region redundancy and automatic failover ensure content availability even during outages
- **Performance optimization**: Edge caching, compression, and image optimization reduce load times and bandwidth consumption
- **Development velocity**: Separate CDN configurations for dev, staging, and production environments enable safe testing and deployment

Without a robust CDN infrastructure, the platform would face slow load times for international users, high bandwidth costs, vulnerability to traffic spikes and attacks, and operational complexity managing content distribution.

## Scope

**Included**
- CDN provider selection and configuration (Cloudflare-specific implementation)
- Cloudflare R2 object storage integration and bucket organization
- CDN caching policies and TTL strategies per asset type
- Edge location configuration and geographic distribution
- Compression and optimization settings (Brotli, Gzip, image formats)
- Security policies: signed URLs, hotlink protection, DDoS mitigation, SSL/TLS configuration
- Multi-environment CDN setup (development, staging, production)
- Asset URL structure and domain strategy
- Cache invalidation and purging workflows
- Media delivery for platform-wide assets: images, videos, audio, fonts, documents, user-generated content
- Performance monitoring and CDN analytics
- Cost optimization strategies and budget management
- Disaster recovery and failover procedures

**Excluded**
- Game asset upload workflows and validation (see [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md))
- Game-specific asset types and processing (covered in Games Assets Pipeline)
- Player shell UI and game rendering (see [Game Player Shell](../03-student-experience/game-player-shell.md))
- Video streaming protocols and live streaming (future consideration, not Phase 1)
- Application-level caching (Redis, in-memory) (see [Caching Strategy](../18-architecture/caching-strategy.md))
- API response caching (separate concern from static asset CDN)

## Data model (if applicable)

### CDN Asset Organization

Assets are stored in Cloudflare R2 buckets and served via Cloudflare CDN with a structured hierarchy to support versioning, multi-tenancy, and efficient cache management.

#### R2 Bucket Structure

```
mlc-production/
├── games/                          # Game assets (see Games Assets Pipeline)
│   └── {game_id}/
│       └── v{version}/
│           └── {stage_type}/
├── static/                         # Platform static assets
│   ├── images/
│   │   ├── branding/               # Logos, icons, brand assets
│   │   ├── ui/                     # UI elements, backgrounds
│   │   └── content/                # Content images (tutorials, help)
│   ├── fonts/
│   │   └── {font_family}/          # Web fonts (WOFF2, WOFF)
│   ├── audio/
│   │   ├── sfx/                    # Sound effects (UI clicks, notifications)
│   │   └── music/                  # Background music
│   └── video/
│       ├── tutorials/              # Tutorial and help videos
│       └── marketing/              # Landing page and promo videos
├── user-content/                   # User-generated content
│   ├── avatars/
│   │   └── {user_id}.{ext}
│   ├── uploads/
│   │   └── {tenant_id}/
│   │       └── {upload_id}/
│   └── exports/                    # Generated reports, CSV exports
│       └── {tenant_id}/
│           └── {export_id}.{ext}
├── documents/                      # Policy docs, terms, guides
│   ├── legal/
│   └── help/
└── temp/                           # Temporary files (auto-expire after 7 days)
    └── {temp_id}/
```

**Bucket Naming Conventions**:
- Production: `mlc-production`
- Staging: `mlc-staging`
- Development: `mlc-dev`
- Backups: `mlc-backups`

**Path Naming Rules**:
- Lowercase alphanumeric and hyphens only
- No spaces or special characters
- Version suffixes for immutable content: `v{integer}`
- Date stamps for archival: `YYYY-MM-DD`
- Hash suffixes for cache busting: `-{hash8}` (8-character hash)

#### CDN Domain Strategy

**Production Domains**:
- Primary CDN: `cdn.musiclearningcommunity.com` (or `cdn.mlc.com` when domain acquired)
- R2 Direct: `assets.mlc-production.r2.dev` (fallback, not public-facing)
- Legacy Support: `media.musiclearningcommunity.com` (if migrating from existing system)

**Environment Domains**:
- Staging: `cdn-staging.musiclearningcommunity.com`
- Development: `cdn-dev.musiclearningcommunity.com` (or use R2 direct URLs)

**Asset URL Format**:
```
https://cdn.musiclearningcommunity.com/{asset_category}/{path}

Examples:
https://cdn.musiclearningcommunity.com/games/G-01542/v3/play/index.html
https://cdn.musiclearningcommunity.com/static/images/branding/logo.svg
https://cdn.musiclearningcommunity.com/user-content/avatars/usr_12345.jpg
https://cdn.musiclearningcommunity.com/documents/legal/privacy-policy.pdf
```

**URL Signing for Private Content**:
- User-generated content (avatars, uploads) uses signed URLs with expiration
- Draft game assets use signed URLs until published
- Exports and reports use time-limited signed URLs (24-hour expiry)
- Format: `https://cdn.mlc.com/{path}?signature={hmac}&expires={timestamp}`

#### Asset Metadata

Assets stored in R2 include custom metadata for efficient management:

```json
{
  "x-amz-meta-asset-type": "image|video|audio|font|document|game|user-content",
  "x-amz-meta-uploaded-by": "usr_12345",
  "x-amz-meta-uploaded-at": "2025-10-12T14:23:45Z",
  "x-amz-meta-tenant-id": "tenant_abc" (if multi-tenant),
  "x-amz-meta-public": "true|false",
  "x-amz-meta-cache-ttl": "31536000|3600|300" (seconds),
  "x-amz-meta-content-hash": "sha256:abc123...",
  "content-type": "image/jpeg",
  "content-disposition": "inline|attachment; filename=\"{name}\"",
  "cache-control": "public, max-age=31536000, immutable"
}
```

**Standard Metadata Fields** (R2 native):
- `content-type`: MIME type (critical for browser rendering)
- `content-length`: File size in bytes
- `etag`: Content hash (auto-generated by R2)
- `last-modified`: Last upload/modification timestamp
- `cache-control`: HTTP caching directives

**Custom Metadata Fields** (via `x-amz-meta-` prefix):
- `asset-type`: Categorization for analytics and billing
- `uploaded-by`: User ID for audit trails
- `tenant-id`: Multi-tenant isolation (if applicable)
- `public`: Boolean flag for access control
- `cache-ttl`: Recommended TTL for CDN edge caching

### CDN Cache Policies

Different asset types have distinct caching requirements based on mutability, size, and access patterns.

#### Cache TTL Strategy

| Asset Type | Cache-Control | TTL | Rationale |
|------------|---------------|-----|-----------|
| Game bundles (versioned) | `public, max-age=31536000, immutable` | 1 year | Immutable versioned paths never change |
| Static assets (versioned) | `public, max-age=31536000, immutable` | 1 year | Images, fonts, CSS/JS with hash in filename |
| Game thumbnails | `public, max-age=86400` | 1 day | May update occasionally without version bump |
| UI images | `public, max-age=604800` | 1 week | Logos, icons, UI elements change infrequently |
| User avatars | `public, max-age=3600` | 1 hour | Users may update profile pictures |
| Documents (legal, help) | `public, max-age=300` | 5 minutes | May need urgent updates (e.g., policy changes) |
| User exports/reports | `private, max-age=0, no-cache` | No cache | Sensitive, one-time download |
| Video files | `public, max-age=2592000` | 30 days | Large files, expensive to re-fetch |
| Audio files | `public, max-age=2592000` | 30 days | Music and sound effects rarely change |

**Immutable Content Strategy**:
- Append content hash to filename: `app-{hash}.js`, `logo-{hash}.svg`
- Use versioned paths: `/v3/`, `/v4/`
- Once deployed, content at that URL never changes
- Allows aggressive caching (1 year TTL) without stale content risk
- Cache invalidation unnecessary; new versions use new URLs

**Dynamic Content Caching**:
- API responses: Not cached at CDN (application handles caching)
- Personalized pages: Not cached at CDN (server-side rendering)
- HTML pages: `max-age=60` (1 minute) for static marketing pages

#### Edge Location Configuration

Cloudflare provides 300+ global edge locations. No explicit configuration needed, but understanding distribution is critical:

**Priority Regions** (based on expected user base):
1. **North America**: US (coast-to-coast coverage), Canada
2. **Europe**: UK, Germany, France, Netherlands
3. **Asia-Pacific**: Australia, Japan, South Korea, Singapore
4. **Other**: South America (Brazil), Middle East, Africa (limited initially)

**Cloudflare Automatic Routing**:
- Anycast network automatically routes requests to nearest edge
- Tiered caching: Edge → Regional → Origin
- Smart routing around outages and congestion

**Custom Configuration**:
- Cloudflare Argo: Intelligent routing for optimal performance (optional, paid)
- Cloudflare Load Balancing: Geo-steering and failover (future consideration)

## Behavior and rules

### Asset Upload and Storage

#### Uploading to R2

**Direct Upload (Admin/System)**:
1. Application backend generates pre-signed R2 upload URL with short expiry (15 minutes)
2. Client uploads file directly to R2 using pre-signed URL (avoids proxying large files through app server)
3. R2 returns success with asset URL and metadata
4. Application backend records asset URL in database
5. CDN cache warmed automatically on first request

**Upload Workflow**:
```
User → Frontend → Backend API (generates pre-signed URL)
          ↓
       R2 Upload (direct, bypasses app server)
          ↓
    R2 Success → Backend (save URL to DB) → CDN Warm
```

**Pre-Signed URL Generation** (Backend):
```javascript
// Example: Generate pre-signed upload URL
const uploadUrl = await r2.generatePresignedUploadUrl({
  bucket: 'mlc-production',
  key: `user-content/avatars/${userId}.jpg`,
  expiresIn: 900, // 15 minutes
  contentType: 'image/jpeg',
  maxSize: 5242880, // 5 MB
  metadata: {
    'uploaded-by': userId,
    'uploaded-at': new Date().toISOString(),
    'public': 'true'
  }
});
```

**Upload Validation**:
- File size limits enforced at R2 upload (not backend)
- Content-Type validation via pre-signed URL restrictions
- Virus scanning for user uploads (ClamAV or cloud service)
- Image dimension validation (if applicable)

#### Asset Delivery via CDN

**Request Flow**:
```
User Browser → Cloudflare Edge (nearest location)
                  ↓
            Edge Cache Hit? 
                  ↓ No
         Regional Cache Hit?
                  ↓ No
         R2 Origin (fetch asset)
                  ↓
         Cache at Edge & Regional
                  ↓
         Deliver to User
```

**Cache Key Structure**:
- Default: Full URL including query string
- Custom: Ignore specific query params (e.g., analytics tracking params)
- Example: `cdn.mlc.com/games/G-01542/v3/play/index.html` (cache key)

**Cache Hit Behavior**:
1. **Edge Hit**: Asset served from edge cache (1-5ms latency)
2. **Regional Hit**: Asset served from regional cache (10-30ms latency)
3. **Origin Hit**: Asset fetched from R2, cached at edge and regional (50-200ms latency)

**Cache Miss Handling**:
- Cloudflare automatically fetches from R2 on miss
- Concurrent requests coalesced (only one origin fetch for multiple simultaneous requests)
- Stale-while-revalidate: Serve stale cache while fetching fresh content (optional)

#### Cache Invalidation and Purging

**Automated Invalidation** (triggered by system):
- New game version published: Purge `/games/{game_id}/*` (if replacing same version)
- Logo updated: Purge `/static/images/branding/logo.svg`
- Legal document updated: Purge `/documents/legal/*`

**Manual Invalidation** (SysAdmin):
- Cloudflare Dashboard or API
- Purge by URL, tag, or entire domain
- Purge confirms in 30 seconds globally

**Invalidation Strategies**:
- **Single URL Purge**: Fastest, most targeted
- **Tag-Based Purge**: Purge all assets with specific tag (e.g., `game-G-01542`)
- **Prefix Purge**: Purge all assets under path (e.g., `/games/G-01542/*`)
- **Full Purge**: Purge entire domain cache (emergency only, impacts performance)

**Purge API Example**:
```javascript
// Purge specific asset
await cloudflare.zones.purgeCache({
  zoneId: 'abc123',
  files: [
    'https://cdn.mlc.com/static/images/branding/logo.svg'
  ]
});

// Purge by tag
await cloudflare.zones.purgeCache({
  zoneId: 'abc123',
  tags: ['game-G-01542']
});
```

**Cache Tag Assignment** (via Cloudflare Workers or Page Rules):
- Tag assets at upload: `Cache-Tag: game-G-01542, stage-play, version-v3`
- Enables granular cache invalidation without knowing all asset URLs

### Security and Access Control

#### Signed URLs for Private Content

**Use Cases**:
- User-generated content (avatars, uploads) 
- Draft game assets (not yet published)
- Reports and exports (time-limited access)
- Admin-only documents

**Signed URL Generation** (Backend):
```javascript
// Example: Generate signed URL with 24-hour expiry
const signedUrl = await r2.generatePresignedUrl({
  bucket: 'mlc-production',
  key: 'user-content/exports/tenant_abc/report_12345.csv',
  expiresIn: 86400, // 24 hours
  method: 'GET'
});
// Returns: https://cdn.mlc.com/user-content/exports/tenant_abc/report_12345.csv?signature=...&expires=1728756000
```

**Signature Validation**:
- Cloudflare validates signature at edge (via R2 integration)
- Expired signatures return HTTP 403
- Invalid signatures return HTTP 403
- No origin request if signature invalid (prevents abuse)

#### Hotlink Protection

**Purpose**: Prevent other websites from embedding MLC assets and consuming bandwidth.

**Implementation** (Cloudflare Page Rules or Workers):
```javascript
// Cloudflare Worker: Hotlink protection
export default {
  async fetch(request, env) {
    const referer = request.headers.get('Referer');
    const allowedDomains = [
      'musiclearningcommunity.com',
      'mlc.com',
      'localhost:3000' // Dev environment
    ];
    
    // Allow direct access (no referer) for RSS readers, etc.
    if (!referer) return fetch(request);
    
    const refererDomain = new URL(referer).hostname;
    if (!allowedDomains.some(domain => refererDomain.includes(domain))) {
      return new Response('Hotlinking not allowed', { status: 403 });
    }
    
    return fetch(request);
  }
};
```

**Exceptions**:
- Allow no-referer requests (direct links, privacy extensions)
- Whitelist specific domains (e.g., partner sites, embedded content)
- Marketing assets may be intentionally embeddable

#### DDoS Mitigation and Rate Limiting

**Cloudflare Automatic Protection**:
- Layer 3/4 DDoS mitigation (network/transport layer attacks)
- Layer 7 DDoS mitigation (HTTP floods, slowloris)
- Automatically enabled, no configuration needed

**Rate Limiting** (Cloudflare Rate Limiting Rules):
```yaml
# Example: Limit asset download abuse
Rule: "Excessive Asset Downloads"
Condition: 
  - Path matches: /user-content/*
  - Requests: > 100 per 1 minute per IP
Action: Block for 10 minutes
```

**Bot Management**:
- Cloudflare Bot Management (optional, paid feature)
- Distinguishes good bots (search engines) from bad bots (scrapers, attackers)
- Challenge suspicious traffic with CAPTCHA

#### SSL/TLS Configuration

**Certificate Management**:
- Cloudflare Universal SSL (free, auto-renewing)
- Custom SSL certificate (optional, if required for branding)
- R2 origin supports TLS 1.3 (encrypted origin connection)

**Encryption Modes**:
- **Full (Strict)**: End-to-end encryption, Cloudflare validates R2 certificate ✅ Recommended
- **Full**: End-to-end encryption, no certificate validation
- **Flexible**: Cloudflare → Browser encrypted, Cloudflare → Origin unencrypted ❌ Not recommended

**TLS Settings**:
- Minimum TLS version: TLS 1.2 (for compatibility)
- TLS 1.3 preferred (faster handshake)
- HTTP/2 and HTTP/3 (QUIC) enabled for performance

**HTTPS Redirect**:
- Automatic HTTP → HTTPS redirect enabled
- HSTS (HTTP Strict Transport Security) header: `max-age=31536000; includeSubDomains; preload`

### Performance Optimization

#### Compression

**Brotli Compression** (Cloudflare automatic):
- Enabled for text assets: HTML, CSS, JS, JSON, SVG, XML
- Compression level: Automatic (Cloudflare optimizes)
- Fallback to Gzip if browser doesn't support Brotli

**Gzip Compression** (Cloudflare automatic):
- Enabled for all text assets
- Compression level: 6 (balance of speed and size)
- Pre-compressed files (`.gz` suffix) served directly if available

**Non-Compressible Assets**:
- Already compressed: JPEG, PNG, MP4, MP3 (no further compression)
- Small files (<1 KB): Compression overhead not worth it
- Binary formats: `.woff2`, `.wasm` (already optimized)

#### Image Optimization

**Cloudflare Image Resizing** (optional, paid feature):
- On-the-fly image resizing: `?width=400&height=300`
- Format conversion: Automatic WebP for supported browsers
- Quality optimization: Smart compression balancing quality and size

**Manual Image Optimization** (at upload):
- PNG: PNG crush, convert to WebP variants
- JPEG: Quality 85% (Visually lossless, significant size reduction)
- SVG: Minify and remove metadata
- Generate responsive variants: `@1x`, `@2x`, `@3x`

**Lazy Loading**:
- Frontend implements lazy loading for images below fold
- Uses `loading="lazy"` attribute (native browser support)
- Intersection Observer API for legacy browsers

#### Video Optimization

**Video Compression** (at upload):
- Codec: H.264 (broad compatibility) or H.265 (better compression, limited support)
- Resolution: 720p standard, 1080p for high-quality tutorials
- Bit rate: Variable bit rate (VBR) targeting 2-4 Mbps for 720p

**Streaming Preparation** (future):
- HLS (HTTP Live Streaming) for adaptive bitrate
- DASH (Dynamic Adaptive Streaming over HTTP) for cross-platform
- Segment videos into chunks for smooth playback and scrubbing

**Cloudflare Stream** (future consideration, paid):
- Managed video hosting and streaming
- Automatic transcoding and adaptive bitrate
- Built-in player with analytics
- Avoids R2 egress fees for video

#### Font Optimization

**Font Formats**:
- **WOFF2**: Primary format (best compression, broad support)
- **WOFF**: Fallback for older browsers
- **TTF/OTF**: Not served (convert to WOFF2)

**Font Loading Strategy**:
```css
@font-face {
  font-family: 'CustomFont';
  src: url('https://cdn.mlc.com/static/fonts/custom/font.woff2') format('woff2'),
       url('https://cdn.mlc.com/static/fonts/custom/font.woff') format('woff');
  font-display: swap; /* Show fallback font immediately, swap when loaded */
}
```

**Font Subsetting**:
- Include only glyphs needed (Latin alphabet, numbers, punctuation)
- Reduces file size 50-70%
- Separate font files for extended character sets if needed

**Preloading Critical Fonts**:
```html
<link rel="preload" href="https://cdn.mlc.com/static/fonts/custom/font.woff2" as="font" type="font/woff2" crossorigin>
```

### Multi-Environment Configuration

#### Development Environment

**Purpose**: Local development and testing without affecting production CDN.

**Configuration**:
- R2 Bucket: `mlc-dev`
- CDN Domain: `cdn-dev.musiclearningcommunity.com` or direct R2 URLs
- Cache TTL: Short (5 minutes) to facilitate rapid iteration
- Access: Public or IP-restricted
- Cost: Minimal (low traffic, aggressive cache purging)

**Developer Workflow**:
1. Upload assets to `mlc-dev` bucket via local scripts or admin panel
2. Assets served via dev CDN domain or R2 direct URLs
3. Test asset loading in local Next.js development server
4. Purge dev CDN cache frequently during development

#### Staging Environment

**Purpose**: Pre-production testing with production-like CDN configuration.

**Configuration**:
- R2 Bucket: `mlc-staging`
- CDN Domain: `cdn-staging.musiclearningcommunity.com`
- Cache TTL: Same as production (validate caching behavior)
- Access: IP-restricted or password-protected (not public)
- Cost: Low (limited QA traffic)

**QA Workflow**:
1. Deploy assets to staging via CI/CD pipeline
2. QA team tests asset loading, performance, and cache behavior
3. Validate CDN settings (cache headers, compression, security)
4. Approve for production deployment

#### Production Environment

**Purpose**: Live platform serving real users.

**Configuration**:
- R2 Bucket: `mlc-production`
- CDN Domain: `cdn.musiclearningcommunity.com`
- Cache TTL: Aggressive (per asset type, see Cache TTL Strategy)
- Access: Public for published assets, signed URLs for private assets
- Cost: Optimized (R2 zero-egress fees, Cloudflare free tier)

**Deployment Workflow**:
1. Assets uploaded to production via automated deployment (CI/CD)
2. Cache warm triggered for critical assets (homepage, popular games)
3. Monitor CDN analytics for errors or performance issues
4. Rollback capability via versioned asset paths

### Disaster Recovery and Failover

#### R2 Redundancy

**Cloudflare R2 Built-In Redundancy**:
- Multi-region replication (automatic, no configuration)
- Durability: 99.999999999% (11 nines)
- Availability: 99.9% SLA (paid plans)

**Backup Strategy**:
- Automated daily backups to separate R2 bucket: `mlc-backups`
- Backup retention: 30 days rolling
- Critical assets (games, sequences) backed up to secondary storage (AWS S3 or Google Cloud Storage)

#### CDN Failover

**Cloudflare Uptime** (historical):
- 99.99%+ uptime (rare outages)
- Global Anycast network provides automatic failover

**Fallback Strategy** (if Cloudflare CDN unavailable):
- Option 1: Serve assets directly from R2 origin (slower, but available)
- Option 2: Multi-CDN setup (future): AWS CloudFront as secondary CDN
- Option 3: Static asset bundle deployed with Vercel app (critical assets only)

**Disaster Recovery Testing**:
- Quarterly DR drills: Simulate CDN outage, validate fallback
- Monitor CDN health and switch to fallback automatically (future)

## UX requirements (if applicable)

### Admin Asset Management Interface

**Entry Point**: System Admin → Content Management → CDN Assets

**Asset Browser**:
- **Bucket Selector**: Dropdown to switch between buckets (dev, staging, production)
- **Folder Tree**: Expandable tree view of asset hierarchy
- **Asset List**: Table view with columns:
  - Thumbnail/Icon
  - Asset Name
  - Type (image, video, audio, document, etc.)
  - Size (KB, MB)
  - Last Modified
  - CDN URL (copy button)
  - Actions (download, delete, purge cache)
- **Search/Filter**: Search by filename, filter by type or date range

**Upload Interface**:
- **Drag-and-Drop Zone**: Bulk upload multiple files
- **Folder Selection**: Choose destination folder
- **Metadata Fields**:
  - Public/Private toggle
  - Cache TTL override (optional)
  - Custom tags for cache purging
- **Upload Progress**: Real-time progress bar with % complete
- **Upload Confirmation**: List of uploaded assets with CDN URLs

**Cache Management**:
- **Purge Cache Button**: Purge specific asset or folder
- **Purge Confirmation**: "Are you sure? This will affect all users."
- **Purge Status**: Success/failure notification with purge propagation time estimate

**Accessibility**:
- Keyboard navigation for folder tree and asset list
- Screen reader announcements for upload progress and purge actions
- High contrast mode support

### CDN Performance Dashboard

**Entry Point**: System Admin → Analytics → CDN Performance

**Key Metrics**:
- **Bandwidth Usage**: Graph of bandwidth over time (GB per day/week/month)
- **Request Count**: Total requests and cache hit ratio (edge/regional/origin)
- **Latency**: P50, P95, P99 latency by region
- **Error Rate**: 4xx and 5xx errors by asset type and URL
- **Cache Hit Ratio**: Percentage of requests served from edge cache
- **Top Assets**: Most requested assets (by request count and bandwidth)
- **Cost Estimate**: Projected monthly cost based on usage

**Visualizations**:
- Line charts for bandwidth and request trends
- Pie chart for cache hit ratio breakdown
- World map showing request distribution by region
- Table of top assets with drill-down capability

**Alerts** (future):
- High bandwidth spike (>50% above baseline)
- Cache hit ratio drop (<80%)
- Elevated error rate (>1%)
- Cost threshold exceeded

## Telemetry

### CDN Events

All events follow schema defined in [Event Model](../15-analytics-and-reporting/event-model.md#event-core-schema).

**Asset Uploaded to R2**  
Event: `cdn_asset_uploaded`  
Properties:
```json
{
  "asset_url": "https://cdn.mlc.com/user-content/avatars/usr_12345.jpg",
  "asset_type": "image",
  "file_size_bytes": 245760,
  "uploaded_by": "usr_12345",
  "bucket": "mlc-production",
  "public": true,
  "cache_ttl": 3600
}
```
Fires: When asset successfully uploaded to R2

**Asset Served from CDN**  
Event: `cdn_asset_served`  
Properties:
```json
{
  "asset_url": "https://cdn.mlc.com/games/G-01542/v3/play/index.html",
  "cache_status": "HIT|MISS|EXPIRED",
  "latency_ms": 45,
  "region": "us-east",
  "user_agent": "Mozilla/5.0...",
  "referer": "https://app.musiclearningcommunity.com/play/assignment/12345"
}
```
Fires: On every CDN asset request (sampled at 1% for performance)

**Cache Purged**  
Event: `cdn_cache_purged`  
Properties:
```json
{
  "purge_type": "single_url|tag|prefix|full",
  "purge_target": "/games/G-01542/*",
  "purged_by": "usr_admin",
  "reason": "New game version published",
  "affected_urls": 37
}
```
Fires: When cache purge triggered

**CDN Error**  
Event: `cdn_error`  
Properties:
```json
{
  "asset_url": "https://cdn.mlc.com/missing/asset.jpg",
  "error_code": "404|500|timeout",
  "error_message": "Asset not found",
  "region": "eu-west",
  "user_id": "stu_67890" (if logged in)
}
```
Fires: When CDN asset request fails (never sampled, 100% capture)

### Sampling

- Asset uploads: Never sampled (100% capture for audit)
- Asset served: 1% sampling (high volume, performance impact)
- Cache purges: Never sampled (100% capture for audit)
- CDN errors: Never sampled (100% capture for reliability monitoring)

## Permissions

### CDN Asset Read Permissions

**Students**:
- ✅ Access public assets (games, images, videos)
- ✅ Access own private assets (avatar, uploads) via signed URLs
- ❌ Cannot access other users' private assets
- ❌ Cannot access admin-only assets

**Teachers**:
- ✅ All Student permissions
- ✅ Access preview/draft game assets (with permission)
- ❌ Cannot access subscriber-specific exports or reports

**Content Editors**:
- ✅ All Teacher permissions
- ✅ View all assets in admin interface
- ✅ Download assets for editing

**System Admin**:
- ✅ Full read access to all assets (public and private)
- ✅ Access CDN analytics and performance dashboard
- ✅ View R2 bucket contents and metadata

### CDN Asset Write Permissions

**Students**:
- ✅ Upload own avatar (size and format restricted)
- ❌ Cannot upload other asset types

**Teachers**:
- ❌ No write access to CDN assets

**Content Editors**:
- ✅ Upload assets to designated folders (e.g., `/static/images/`, `/documents/`)
- ✅ Replace existing assets (creates new version, preserves old)
- ❌ Cannot delete assets
- ❌ Cannot upload to production without approval

**System Admin**:
- ✅ Upload assets to any folder
- ✅ Delete assets (soft delete, preserves in backups)
- ✅ Purge CDN cache (manual invalidation)
- ✅ Modify R2 bucket configuration

### CDN Management Operations

**Upload Asset**:
- Students: ✅ Own avatar only
- Content Editors: ✅ Designated folders
- SysAdmin: ✅ All folders

**Delete Asset**:
- Students: ❌ No
- Content Editors: ❌ No
- SysAdmin: ✅ Yes (with confirmation)

**Purge Cache**:
- Students: ❌ No
- Content Editors: ❌ No
- SysAdmin: ✅ Yes

**Access CDN Analytics**:
- Students: ❌ No
- Content Editors: ✅ Limited (own uploads)
- SysAdmin: ✅ Full access

## Supporting Documents Referenced

This CDN and media specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game asset structure and delivery requirements
- [Games Loaded.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Sheet1.csv) — Game asset inventory and media references
- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser compatibility for media formats

## Dependencies

### Internal Dependencies

- **[Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md)**: Game-specific asset upload and validation workflows rely on CDN infrastructure
- **[Game Player Shell](../03-student-experience/game-player-shell.md)**: Loads game bundles from CDN
- **[Technology Stack](../00-foundations/techstack.md)**: Defines Cloudflare R2 and CDN as core infrastructure choices
- **[Caching Strategy](../18-architecture/caching-strategy.md)**: Application-level caching complements CDN edge caching
- **[Security and Privacy](../18-architecture/security-privacy.md)**: Signed URLs, hotlink protection, and DDoS mitigation policies
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: CDN telemetry follows canonical event schema
- **[Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md)**: CDN compression and format support aligned with browser requirements

### External Dependencies

- **Cloudflare CDN**: Global edge network for content delivery
- **Cloudflare R2**: Object storage with S3-compatible API
- **Cloudflare Workers**: Optional edge computing for custom logic (hotlink protection, signed URLs)
- **Vercel**: Application hosting and deployment, integrates with Cloudflare CDN
- **R2 S3-Compatible API**: Enables migration to AWS S3 or other S3-compatible storage
- **Virus Scanning Service**: ClamAV or cloud-based antivirus for user uploads (optional)

### Third-Party Libraries

- **AWS SDK for JavaScript (S3 Client)**: Interface with R2 via S3-compatible API
- **Cloudflare API Client**: Manage CDN cache purging and analytics
- **Sharp (Node.js)**: Image optimization and thumbnail generation (server-side)

## Open questions

### Multi-CDN Failover Strategy

- **Question**: Should we implement multi-CDN failover (e.g., Cloudflare primary, AWS CloudFront secondary) for critical asset availability?
- **Context**: Single CDN outage (rare but possible) could prevent students from loading games and content
- **Options**:
  - A) Single CDN with high SLA (99.9%+), accept rare outages (simplest, lowest cost)
  - B) Multi-CDN with automatic failover via DNS or load balancer (highest availability, doubled cost and complexity)
  - C) Origin fallback: If CDN fails, serve assets directly from R2 origin (middle ground, slower but available)
- **Trade-offs**: Multi-CDN provides best availability but significantly increases operational complexity and cost
- **Current Assumption**: Option A (single CDN) for Phase 1, reassess based on uptime requirements and user impact
- **Decision Needed By**: Phase 2 (post-launch, based on reliability data)

### Video Streaming Infrastructure

- **Question**: Should we implement HLS/DASH streaming for video content, or continue serving MP4 files directly?
- **Context**: Large video files increase CDN bandwidth costs and slow initial load; streaming enables adaptive bitrate and better UX
- **Options**:
  - A) Serve MP4 files directly from R2 via CDN (simplest, works for short videos)
  - B) Cloudflare Stream (managed video hosting and streaming, paid feature)
  - C) Self-hosted HLS/DASH transcoding and streaming (most control, highest complexity)
- **Trade-offs**: Streaming provides better UX for long videos but requires infrastructure investment
- **Current Assumption**: Option A for Phase 1 (videos under 5 minutes), Option B for Phase 2 if video content expands
- **Decision Needed By**: Phase 1 beta (if tutorial videos planned for launch)

### Asset Localization and Internationalization

- **Question**: How should we structure CDN assets for multi-language support (future internationalization)?
- **Context**: Future expansion to non-English markets will require localized images, videos, and audio
- **Options**:
  - A) Separate folders per language: `/static/images/en/`, `/static/images/es/`
  - B) Locale detection at CDN edge (Cloudflare Workers) serves appropriate variant
  - C) Single CDN structure, application handles localization logic
- **Current Assumption**: Not addressing internationalization in Phase 1; will revisit for Phase 3 expansion
- **Decision Needed By**: Phase 3 (internationalization phase)

### CDN Cost Optimization and Budget Thresholds

- **Question**: Should we implement automated cost controls and alerting for CDN bandwidth usage?
- **Context**: Unexpected traffic spikes or abuse could lead to high bandwidth costs (though R2 eliminates egress fees)
- **Options**:
  - A) No automated controls, monitor usage manually (simplest, risk of surprise costs)
  - B) Cost alerts at thresholds (e.g., $100/month), notify SysAdmin (middle ground)
  - C) Rate limiting and throttling at high usage thresholds (prevents abuse, may impact legitimate users)
- **Current Assumption**: Option B (cost alerts) with Option C (throttling) as emergency measure
- **Decision Needed By**: Phase 1 launch (before production traffic)

### Asset Versioning and Archival Policy

- **Question**: Should we automatically archive or delete old asset versions to reduce storage costs?
- **Context**: Keeping all historical versions indefinitely increases R2 storage costs; versioned URLs make old assets rarely accessed
- **Options**:
  - A) Keep all versions forever (simplest, highest cost, best for rollback)
  - B) Archive versions older than 6 months to cold storage (reduced cost, slower retrieval)
  - C) Delete versions older than 2 years if no active references (lowest cost, potential data loss)
- **Current Assumption**: Option A for Phase 1, reassess if storage costs exceed $500/month
- **Decision Needed By**: Phase 2 (post-launch, based on storage growth trends)
