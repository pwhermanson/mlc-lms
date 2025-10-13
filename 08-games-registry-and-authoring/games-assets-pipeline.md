# Games Assets Pipeline

## Purpose

This specification defines the complete lifecycle for game assets from creation through upload, validation, storage, deployment, and delivery to ensure that every playable game stage has reliable, optimized, and browser-compatible assets available at runtime. The assets pipeline transforms raw content files into production-ready bundles served via CDN with appropriate versioning, caching, and fallback strategies.

> **Related Documents**: For the game entity structure that references these assets, see [Game Model](game-model.md). For publish validation that includes asset checks, see [Games CRUD](games-crud.md). For metadata fields describing asset requirements, see [Games Metadata](games-metadata.md). For stage-specific asset policies, see [Games Stages Policy](games-stages-policy.md).

The assets pipeline enables:
- **Reliable game loading**: Students experience fast, consistent access to game content without broken asset references
- **Version control**: Multiple versions of game assets coexist without conflicts, supporting gradual rollout and rollback
- **Content updates**: Editors can upload new assets without breaking live games or disrupting active student sessions
- **Performance optimization**: Assets are automatically optimized, compressed, and distributed via CDN for global low-latency access
- **Quality assurance**: Automated validation catches missing assets, broken references, incompatible formats, and performance issues before publish
- **Browser compatibility**: Asset bundles include format fallbacks and polyfills to support the platform's minimum browser requirements
- **Audit and compliance**: Complete tracking of who uploaded what assets when, with ability to trace asset provenance

Without a structured assets pipeline, the platform risks broken games from missing files, slow load times from unoptimized assets, version conflicts between metadata and bundles, and inability to update content safely at scale.

## Scope

**Included**
- Asset type definitions (HTML5 bundles, thumbnails, audio sprites, videos, fonts, images)
- Upload workflow for content creators and editors
- Storage structure and naming conventions for asset organization
- Validation rules for file formats, sizes, dimensions, and quality
- Asset processing pipeline (optimization, compression, transcoding, thumbnail generation)
- CDN deployment strategy and cache invalidation
- Versioning and rollback mechanisms
- Asset references in game metadata and database schema
- Fallback and degraded mode asset variants
- Health checks and smoke tests for deployed assets
- Telemetry for asset upload, processing, and delivery
- Permissions for asset management operations

**Excluded**
- Game runtime logic and scoring mechanics (covered by individual game implementations)
- Player shell UI and chrome (see [Game Player Shell](../03-student-experience/game-player-shell.md))
- CRUD operations on game metadata records (see [Games CRUD](games-crud.md))
- Sequence and assignment generation logic (see [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md))
- Bulk game import workflows (see [Games Importer](games-importer.md))
- Legacy Flash to HTML5 game migration strategy (historical, covered separately)

## Data model (if applicable)

### Asset Types and Structures

Game assets are organized into several categories, each with specific requirements and purposes:

#### 1. HTML5 Game Bundles

**Description**: Self-contained HTML5 applications that render game stages. Each bundle includes HTML, JavaScript, CSS, and embedded or referenced media assets.

**Structure**:
```
html5_bundle/
├── index.html          # Entry point, must be present
├── js/
│   ├── game.min.js     # Minified game logic
│   └── vendor.min.js   # Third-party libraries
├── css/
│   └── game.min.css    # Minified styles
├── assets/
│   ├── sprites/        # Image sprites
│   ├── audio/          # Audio files (if not using shared sprite)
│   └── fonts/          # Custom web fonts (if needed)
└── manifest.json       # Bundle metadata and dependencies
```

**Requirements**:
- Entry point: `index.html` must exist at bundle root
- Size limit: 50 MB uncompressed per bundle (enforced at upload)
- Format: Standard HTML5, ES6+ JavaScript (transpiled for compatibility)
- Dependencies: All external resources must be declared in `manifest.json`
- Communication: Game must implement MLC Player Bridge API for score submission and events
- Responsive: Must support minimum resolution 1024x768, recommended 1920x1080
- Browser support: Compatible with Chromium 114+, Safari 17+, Firefox 115+ (see [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md))

**Field in Game Model**: `assets.html5_bundle_url`  
**Format**: `https://cdn.mlc/games/{game_id}/v{game_version}/{stage_type}/index.html`  
**Example**: `https://cdn.mlc/games/G-01542/v3/play/index.html`

#### 2. Thumbnail Images

**Description**: Visual preview images displayed in game catalogs, assignment builders, and All Games browsing views.

**Requirements**:
- Dimensions: 400x300 pixels (4:3 aspect ratio) recommended, 800x600 for retina displays
- Format: PNG (with transparency) or JPEG (if no transparency needed)
- File size: Max 200 KB per thumbnail
- Content: Should represent game concept visually, not contain text-heavy content
- Accessibility: Thumbnail filename should be descriptive for asset management

**Field in Game Model**: `assets.thumbnail_url`  
**Format**: `https://cdn.mlc/thumbs/{game_id}.png` or `{game_id}@2x.png` for retina  
**Example**: `https://cdn.mlc/thumbs/G-01542.png`

**Fallback**: If no custom thumbnail provided, system generates placeholder with game name and category icon.

#### 3. Audio Sprites

**Description**: Consolidated audio file containing multiple short sound effects or music clips used across game stages, accessed via timing markers to reduce HTTP requests.

**Requirements**:
- Format: MP3 (primary), OGG (fallback for older Firefox)
- Bit rate: 128 kbps for effects, 192 kbps for music
- Sample rate: 44.1 kHz standard
- File size: Max 5 MB per sprite
- Sprite map: JSON manifest with timing markers must accompany audio file

**Field in Game Model**: `assets.audio_sprite_url`  
**Format**: `https://cdn.mlc/audio/{game_id}-sprite-v{version}.mp3`  
**Example**: `https://cdn.mlc/audio/G-01542-sprite-v2.mp3`

**Sprite Map Example**:
```json
{
  "version": 2,
  "sprite": {
    "correct_chime": { "start": 0, "end": 1.2 },
    "incorrect_buzz": { "start": 1.2, "end": 2.5 },
    "completion_fanfare": { "start": 2.5, "end": 6.8 },
    "background_loop": { "start": 6.8, "end": 30.0, "loop": true }
  }
}
```

#### 4. Video Assets (Optional)

**Description**: Tutorial videos, instructional content, or animated introductions embedded in Learn stages.

**Requirements**:
- Format: MP4 (H.264 codec), WebM (VP9 codec) for browser compatibility
- Resolution: 1280x720 (720p) recommended, 1920x1080 (1080p) for high-quality content
- Frame rate: 30 fps standard
- File size: Max 20 MB per video (use streaming for longer content)
- Captions: WebVTT captions file required for accessibility
- Duration: Keep under 3 minutes for engagement (tutorial videos)

**Field in Game Model**: `assets.video_url` (optional field, not all games use video)  
**Format**: `https://cdn.mlc/video/{game_id}-{video_id}.mp4`

#### 5. Fonts (Optional)

**Description**: Custom web fonts for games requiring specific notation or stylized text rendering.

**Requirements**:
- Format: WOFF2 (primary), WOFF (fallback)
- License: Must have web embedding license, document in game metadata
- Subset: Include only glyphs needed by game to minimize file size
- Font loading: Use `font-display: swap` to prevent text flash during load

**Storage**: Fonts typically embedded in HTML5 bundle `/assets/fonts/` directory

### Asset Storage Schema

Assets are stored in a structured hierarchy on object storage (e.g., AWS S3, Google Cloud Storage) and served via CDN:

```
/games/{game_id}/
  /v{game_version}/
    /learn/
      index.html
      /js/
      /css/
      /assets/
      manifest.json
    /play/
      (same structure)
    /quiz/
      (same structure)
    /challenge/
      (same structure)
    /review/
      (same structure)
  /v{previous_version}/
    (preserved for rollback)
    
/thumbs/
  {game_id}.png
  {game_id}@2x.png
  
/audio/
  {game_id}-sprite-v{version}.mp3
  {game_id}-sprite-v{version}.ogg
  {game_id}-sprite-v{version}.json
  
/video/
  {game_id}-{video_id}.mp4
  {game_id}-{video_id}.webm
  {game_id}-{video_id}.vtt
```

**Naming Conventions**:
- Game ID format: `G-XXXXX` (stable identifier, never changes)
- Version suffix: `v{integer}` increments on breaking changes
- Stage subdirectories: `learn`, `play`, `quiz`, `challenge`, `review` match stage types
- Retina suffix: `@2x` for high-resolution image variants
- Sprite version: `-v{integer}` for audio sprite versioning independent of game version

### Asset Metadata in Database

Game records reference assets via URL fields defined in the [Game Model](game-model.md#core-fields):

```json
{
  "game_id": "G-01542",
  "game_version": 3,
  "assets": {
    "html5_bundle_url": "https://cdn.mlc/games/G-01542/v3/play/index.html",
    "thumbnail_url": "https://cdn.mlc/thumbs/G-01542.png",
    "audio_sprite_url": "https://cdn.mlc/audio/G-01542-sprite-v2.mp3",
    "video_url": null,
    "midi_supported": true,
    "mic_supported": false,
    "asset_requirements": {
      "minimum_resolution": "1024x768",
      "audio_formats": ["mp3", "ogg"],
      "bundle_size_limit_mb": 50,
      "loading_timeout_seconds": 30
    }
  },
  "health": {
    "last_qc_pass": "2025-09-20T12:01:00Z",
    "browser_support": ["chromium_114plus", "safari_17plus", "ios_17plus"],
    "known_limitations": ["safari_low_latency_midi_off"],
    "degraded_modes": ["fallback_keyboard_input"]
  }
}
```

**Asset Health Signals**:
- `last_qc_pass`: Timestamp of last successful asset validation and smoke test
- `browser_support`: Array of supported browser+version combinations
- `known_limitations`: Documented issues or constraints (e.g., MIDI latency in Safari)
- `degraded_modes`: Available fallback options if primary features unavailable

### Asset Versioning Strategy

Assets follow semantic versioning aligned with game versions but with independent granularity for performance and flexibility:

**Game Version Increments** (breaking changes):
- When stage structure changes (e.g., new stage added, stage removed)
- When scoring logic fundamentally changes
- When asset bundle API contract changes
- Format: `v1`, `v2`, `v3` (integer increments)

**Asset Micro-Versions** (non-breaking updates):
- When fixing typos in game text
- When optimizing asset file sizes without functional change
- When updating audio quality or image compression
- Format: Timestamp or hash suffix in CDN path (transparent to game record)

**CDN Cache Keys**:
- Primary: `https://cdn.mlc/games/G-01542/v3/play/index.html` (versioned URL, long TTL)
- Immutable: Once deployed, v3 assets never change; new content becomes v4
- Cache TTL: 1 year for versioned URLs, 5 minutes for thumbnails (can update without version bump)

## Behavior and rules

### Asset Upload Workflow

Content editors and system administrators follow a structured workflow to upload game assets:

#### Step 1: Prepare Assets Locally

**Content Creator Actions**:
1. Develop game using HTML5, JavaScript, and approved frameworks
2. Test game locally ensuring MLC Player Bridge API integration
3. Organize assets into required structure (see Storage Schema above)
4. Create or update `manifest.json` with dependencies and metadata
5. Generate thumbnail image (400x300 PNG or JPEG)
6. Prepare audio sprite and timing map if using shared audio
7. Run local validation checks (linter, size limits, format compliance)

**Validation Checks (Local)**:
- HTML5 bundle includes `index.html` at root
- JavaScript is minified and transpiled for target browsers
- All image assets optimized (PNG crush, JPEG quality 85%)
- Audio files compressed at appropriate bit rates
- No hardcoded absolute paths (all references relative)
- Player Bridge API calls present for score submission
- Manifest accurately lists all external dependencies

#### Step 2: Upload via Admin Panel

**Entry Point**: Games Registry → Edit Game → Assets Tab → Upload Bundle

**Upload Interface**:
- **Stage Selector**: Dropdown to choose which stage to upload (Learn, Play, Quiz, Challenge, Review)
- **Bundle Upload**: Drag-and-drop zone or file picker for ZIP archive containing game bundle
- **Thumbnail Upload**: Separate upload for thumbnail image (optional, can reuse existing)
- **Audio Sprite Upload**: Optional upload for shared audio sprite (if not embedded in bundle)
- **Validation Toggles**:
  - ✅ "Run full validation before upload" (recommended, catches errors early)
  - ✅ "Generate retina thumbnail automatically" (creates @2x variant)
  - ✅ "Test in sandbox environment" (deploys to test CDN first)

**Upload Process**:
1. User selects stage and uploads ZIP file
2. System extracts ZIP to temporary storage
3. Server-side validation runs (see Validation Rules below)
4. If validation passes, assets processed through optimization pipeline
5. Processed assets uploaded to object storage with versioned paths
6. CDN cache warmed with new assets
7. Database updated with new asset URLs
8. Success confirmation shown with preview link

**Error Handling**:
- Validation failures show detailed error report with line numbers and file paths
- Partial upload not committed; atomic transaction ensures consistency
- User can download validation report and retry upload after fixes

#### Step 3: Asset Processing Pipeline

After upload passes validation, assets enter automated processing:

**Processing Stages**:

1. **Extraction**: Unzip bundle, verify structure matches expected layout
2. **Optimization**:
   - Images: PNG crush, JPEG optimization (85% quality), WebP variants generated
   - JavaScript: Additional minification pass, dead code elimination, source map generation
   - CSS: Minification, unused rule removal, critical CSS extraction
   - HTML: Minification, inline critical CSS, preload hints injected
3. **Transcoding**:
   - Audio: Generate fallback formats (MP3 → OGG, AAC where needed)
   - Video: Transcode to multiple resolutions (720p, 480p) and formats (MP4, WebM)
4. **Thumbnail Generation**: Automatic variants at 1x, 2x, 3x resolutions
5. **Hashing**: Generate content hashes for cache busting and integrity checks
6. **Manifest Update**: Augment manifest with asset hashes, sizes, and metadata
7. **Bundle Packaging**: Create final immutable bundle with versioned references

**Processing Telemetry**:
```json
{
  "event": "asset_processing_complete",
  "game_id": "G-01542",
  "stage_type": "play",
  "game_version": 3,
  "upload_timestamp": "2025-10-12T14:23:45Z",
  "editor_id": "usr_12345",
  "original_size_mb": 15.7,
  "optimized_size_mb": 8.3,
  "compression_ratio": 0.53,
  "processing_duration_seconds": 12.4
}
```

#### Step 4: Deployment to CDN

**CDN Strategy**:
- **Primary CDN**: Cloudflare / AWS CloudFront / Google Cloud CDN (configurable)
- **Edge Locations**: Global distribution for low-latency access
- **Cache Policy**:
  - Versioned URLs (`/v3/`): Cache-Control `public, max-age=31536000, immutable`
  - Thumbnails: Cache-Control `public, max-age=300` (5 minutes, allows updates)
- **Compression**: Automatic Brotli / Gzip compression at edge
- **Security**: Signed URLs for draft games (not yet published), public URLs for live games

**Deployment Steps**:
1. Upload processed assets to object storage (S3, GCS, etc.)
2. Set appropriate ACLs and metadata (content-type, cache-control)
3. Trigger CDN cache invalidation for updated paths (if replacing existing version)
4. Warm CDN cache by pre-fetching to major edge locations
5. Update database record with final CDN URLs
6. Run smoke test against deployed assets (see Validation Rules below)

**Rollback Strategy**:
- Previous versions remain accessible at `/v{previous_version}/` paths
- Rollback operation updates database to point to previous version URLs
- No CDN changes needed; simply update database pointers
- Students mid-session continue using loaded assets; new sessions use rolled-back version

### Validation Rules

Assets undergo multiple validation stages to ensure quality and compatibility:

#### Upload Validation (Synchronous, Blocks Upload)

**Bundle Structure Validation**:
- `index.html` exists at bundle root
- Required subdirectories present (`js/`, `css/`, `assets/`)
- No files outside allowed structure (e.g., no root-level scripts except `index.html`)
- `manifest.json` present and valid JSON

**File Format Validation**:
- HTML: Valid HTML5, no deprecated tags, proper DOCTYPE
- JavaScript: Syntax valid, no `eval()` or `Function()` constructor (security)
- CSS: Syntax valid, no `@import` (performance issue)
- Images: PNG, JPEG, WebP, SVG only; no BMP, TIFF, or exotic formats
- Audio: MP3, OGG, WAV, AAC only; correct headers and metadata
- Video: MP4, WebM only; valid codec declarations

**Size Limits Validation**:
- Total bundle: < 50 MB uncompressed
- Single file: < 10 MB (except video, which can be 20 MB)
- Thumbnail: < 200 KB
- Audio sprite: < 5 MB

**Security Validation**:
- No executable files (.exe, .app, .sh, .bat)
- No server-side scripts (PHP, Python, Ruby)
- No external script inclusions from non-whitelisted domains
- No inline event handlers (`onclick`, `onerror`, etc.) in HTML
- No `javascript:` URLs

#### Publish Validation (Pre-Publication, Blocks Going Live)

**Functional Validation**:
- Smoke test: Load game in headless browser, verify renders without errors
- Player Bridge API: Verify game implements required methods (`submitScore`, `reportProgress`)
- Stage completeness: All enabled stages have corresponding asset bundles
- Loading performance: Game loads within 30 seconds on simulated 3G connection

**Cross-Browser Validation**:
- Test game in minimum supported browser versions (Chromium 114, Safari 17, Firefox 115)
- Verify no console errors or warnings
- Check responsive layout at minimum resolution (1024x768)
- Test MIDI input if `midi_supported = true`
- Test microphone input if `mic_supported = true`

**Accessibility Validation**:
- All interactive elements keyboard accessible
- Images have alt text (or marked decorative)
- Color contrast ratios meet WCAG AA standards
- Screen reader announcements tested (if game claims accessibility support)

**Asset Reachability Validation**:
- All URLs in game record return HTTP 200 status
- CDN endpoints respond within 500ms threshold
- No 404 errors for referenced assets
- Content-Type headers correct for each asset type

**Failure Responses**:
- Validation failures generate detailed report with:
  - Error code and severity (blocker vs warning)
  - File path and line number where issue detected
  - Suggested fix or remediation steps
  - Link to documentation for specific validation rule
- Blockers prevent publish; warnings logged but allow publish with confirmation

### Asset Updates and Hotfixes

**Non-Breaking Asset Updates** (no version increment):
- Fix typos in game text
- Optimize asset file sizes
- Update thumbnail image
- Refresh audio quality
- Process: Upload new assets, run validation, replace at same version path (for non-versioned assets like thumbnails) or increment micro-version (transparent to game record)

**Breaking Asset Updates** (version increment required):
- Change game structure or stage configuration
- Modify scoring logic or mastery criteria
- Update Player Bridge API contract
- Process: Create new game version, upload assets to new versioned path, update game record to point to new version

**Hotfix Workflow**:
1. Identify critical issue (broken asset, game crash, security vulnerability)
2. SysAdmin creates hotfix branch in asset repository
3. Fix applied and tested locally
4. Emergency upload bypasses normal review (requires SysAdmin approval)
5. Assets deployed to CDN with same version path (replaces existing)
6. Cache invalidation triggered immediately
7. Smoke test re-run to verify fix
8. Post-deployment audit log entry created

**Grandfathering Policy**:
- Students mid-assignment continue using loaded assets until completion
- New assignment instances use updated assets
- No forced reload for active sessions (prevents disruption)

### Asset Health Monitoring

**Continuous Monitoring**:
- CDN health checks every 5 minutes for live game assets
- Automated smoke tests run daily against all live games
- Browser compatibility tests run weekly with latest browser versions
- Performance regression tests check load times monthly

**Health Metrics**:
- Asset availability: % uptime for asset URLs
- Load time: P50, P95, P99 latency for asset delivery
- Error rate: % of failed asset requests (404, 500, timeout)
- Browser compatibility: % of supported browsers passing smoke tests

**Alert Thresholds**:
- Critical: Asset availability < 99.5% over 5-minute window
- Warning: Load time P95 > 3 seconds for HTML5 bundle
- Info: New browser version released, compatibility unknown

**Health Dashboard**:
- Real-time view of asset health per game
- Historical trends and performance graphs
- Broken asset report with remediation recommendations
- Browser compatibility matrix with pass/fail per game

## UX requirements (if applicable)

### Asset Upload Interface

**Admin Panel: Games Registry → Edit Game → Assets Tab**

**Layout**:
- **Header Section**:
  - Game name and ID prominently displayed
  - Current game version badge
  - Last updated timestamp and editor name
- **Stage Selector Tabs**: Horizontal tabs for Learn, Play, Quiz, Challenge, Review
  - Active stage highlighted
  - Badge showing "✓ Uploaded" or "⚠ Missing" for each stage
- **Current Assets Panel** (per stage):
  - Preview thumbnail of current bundle (auto-generated screenshot)
  - Asset URLs with "Copy" button
  - File sizes and upload timestamps
  - "Download Bundle" button to retrieve current assets
  - "View in Player" button to test game in modal

**Upload Section**:
- **Bundle Upload Zone**:
  - Large drag-and-drop target (400px tall)
  - File picker button: "Choose ZIP File"
  - Supported formats label: "ZIP archives up to 50 MB"
  - Real-time upload progress bar with % complete and estimated time remaining
- **Thumbnail Upload Zone** (optional):
  - Smaller drag-and-drop target (200px tall)
  - Preview of current thumbnail
  - Auto-generation checkbox: "Generate thumbnail from bundle screenshot"
- **Audio Sprite Upload Zone** (optional, if game uses shared audio):
  - Drag-and-drop for MP3/OGG file
  - Sprite map JSON upload or inline editor
- **Validation Options**:
  - Checkbox: "Run full validation before upload" (checked by default)
  - Checkbox: "Deploy to test environment first" (recommended for major updates)
  - Checkbox: "Generate retina variants" (checked by default)

**Upload Flow**:
1. User drags ZIP file to upload zone or clicks file picker
2. File size and format validated client-side immediately
3. If valid, upload starts with progress bar
4. Server-side validation runs (shows loading spinner)
5. Validation results displayed:
   - Success: Green checkmark, "Assets uploaded successfully"
   - Warnings: Yellow warning icon, expandable list of non-blocking issues
   - Errors: Red X, detailed error report with fix suggestions
6. If successful, assets processed (loading spinner with status updates)
7. Completion: Preview updated, CDN URLs shown, "View in Player" enabled

**Error States**:
- **File Too Large**: "Bundle size 67.3 MB exceeds limit of 50 MB. Please optimize assets and try again."
- **Invalid Structure**: "Missing required file: index.html. Ensure bundle root contains entry point."
- **Validation Failed**: Expandable error list with file paths, line numbers, and remediation steps
- **Upload Interrupted**: "Upload failed due to network error. Please try again."

**Accessibility**:
- Drag-and-drop zone keyboard accessible (Enter/Space to open file picker)
- Upload progress announced to screen readers
- Error messages have `role="alert"` for immediate announcement
- Validation results navigable by keyboard with arrow keys
- "View in Player" opens accessible modal with focus trap

### Asset Preview and Testing

**Preview Modal** (triggered by "View in Player" button):
- Full-screen modal with close button (Escape key)
- Game loaded in iframe with Player Shell chrome
- Simulated student view (assignment context)
- Developer tools panel (collapsible, SysAdmin only):
  - Console log output
  - Network requests panel
  - Performance metrics (load time, FPS)
  - Player Bridge API call log
- "Test Controls" toolbar:
  - Dropdown: Select stage to test (if multi-stage game)
  - Checkbox: "Enable debug mode" (shows hitboxes, timing markers)
  - Button: "Reload game" (clears cache, fresh load)
  - Button: "Download logs" (exports console and network logs)

**Catalog Preview** (thumbnail display):
- Thumbnail shown in catalog cards with lazy loading
- Hover: Enlarge thumbnail with smooth transition
- Alt text: Game name for accessibility
- Fallback: Generated placeholder if custom thumbnail missing

## Telemetry

### Asset Pipeline Events

All events follow schema defined in [Event Model](../15-analytics-and-reporting/event-model.md#event-core-schema).

**Asset Upload Started**  
Event: `asset_upload_started`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_type": "play",
  "game_version": 3,
  "editor_id": "usr_12345",
  "file_size_mb": 15.7,
  "upload_method": "drag_drop|file_picker"
}
```
Fires: When user initiates upload

**Asset Upload Completed**  
Event: `asset_upload_completed`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_type": "play",
  "game_version": 3,
  "editor_id": "usr_12345",
  "upload_duration_seconds": 8.3,
  "file_size_mb": 15.7,
  "validation_passed": true
}
```
Fires: When upload finishes and validation completes

**Asset Validation Failed**  
Event: `asset_validation_failed`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_type": "play",
  "editor_id": "usr_12345",
  "validation_errors": [
    {"type": "missing_file", "file": "index.html", "severity": "error"},
    {"type": "size_limit", "file": "bundle.js", "size_mb": 12.3, "limit_mb": 10, "severity": "error"}
  ],
  "validation_warnings": [
    {"type": "unoptimized_image", "file": "assets/bg.png", "size_kb": 450, "recommended_kb": 200}
  ]
}
```
Fires: When validation identifies blocking errors or warnings

**Asset Processing Completed**  
Event: `asset_processing_complete`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_type": "play",
  "game_version": 3,
  "editor_id": "usr_12345",
  "original_size_mb": 15.7,
  "optimized_size_mb": 8.3,
  "compression_ratio": 0.53,
  "processing_duration_seconds": 12.4,
  "optimizations_applied": ["image_compression", "js_minification", "css_minification"]
}
```
Fires: After assets processed through optimization pipeline

**Asset Deployed to CDN**  
Event: `asset_deployed_cdn`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_type": "play",
  "game_version": 3,
  "cdn_url": "https://cdn.mlc/games/G-01542/v3/play/index.html",
  "deployment_duration_seconds": 3.7,
  "cache_warmed": true
}
```
Fires: When assets successfully deployed to CDN

**Asset Load Time** (student-facing)  
Event: `asset_load_time`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_id": "G-01542-play",
  "student_id": "stu_67890",
  "load_duration_ms": 1450,
  "cdn_latency_ms": 120,
  "asset_count": 23,
  "total_size_mb": 8.3,
  "browser": "chrome_120",
  "device_type": "desktop",
  "connection_type": "wifi"
}
```
Fires: When student loads game, bundle fully loaded

**Asset Load Error** (student-facing)  
Event: `asset_load_error`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_id": "G-01542-play",
  "student_id": "stu_67890",
  "error_type": "404|timeout|cors|parse_error",
  "failed_url": "https://cdn.mlc/games/G-01542/v3/play/assets/sprite.png",
  "retry_count": 2,
  "browser": "safari_17",
  "device_type": "ipad"
}
```
Fires: When asset load fails for student

**Asset Hotfix Deployed**  
Event: `asset_hotfix_deployed`  
Properties:
```json
{
  "game_id": "G-01542",
  "stage_type": "play",
  "game_version": 3,
  "editor_id": "usr_12345",
  "hotfix_reason": "Critical bug fix: scoring calculation error",
  "bypass_validation": true,
  "approval_required": true,
  "approver_id": "usr_admin"
}
```
Fires: When emergency hotfix deployed outside normal workflow

### Sampling

- Asset upload and processing events: Never sampled (100% capture for audit)
- Validation events: Never sampled
- CDN deployment events: Never sampled
- Asset load time (student-facing): 10% sampling in production
- Asset load errors: Never sampled (100% capture for reliability monitoring)

## Permissions

### Asset Read Permissions

**Students**:
- ✅ Load and play published game assets via CDN (public URLs for live games)
- ❌ Cannot view draft game assets (requires signed URLs or authentication)
- ❌ Cannot access asset management interface

**Teachers**:
- ✅ Load and play published game assets for preview and testing
- ❌ Cannot view draft game assets unless granted preview permission
- ❌ Cannot upload or modify assets

**Content Editors**:
- ✅ View all game assets (draft and live)
- ✅ Download current asset bundles for reference or local testing
- ✅ Preview games in test environment before publish

**System Admin**:
- ✅ Full read access to all assets (including deprecated and historical versions)
- ✅ Access to asset processing logs and error reports
- ✅ View CDN metrics and health dashboards

### Asset Write Permissions

**Students**:
- ❌ No write access

**Teachers**:
- ❌ No write access

**Content Editors**:
- ✅ Upload new assets for draft games
- ✅ Replace assets on draft games
- ❌ Cannot modify live game assets directly (must create new version or request hotfix)
- ❌ Cannot deploy to CDN (automatic after validation)

**Content Editors with Publish Permission**:
- ✅ All Content Editor permissions
- ✅ Publish draft game assets (triggers CDN deployment)
- ✅ Update thumbnails on live games (non-breaking change)
- ❌ Cannot deploy hotfixes without SysAdmin approval

**System Admin**:
- ✅ All asset write permissions
- ✅ Deploy hotfixes to live games (with approval workflow)
- ✅ Rollback to previous asset versions
- ✅ Manually trigger CDN cache invalidation
- ✅ Override validation rules (with confirmation and audit log)

### Asset Management Operations Permissions

**Upload Asset Bundle**:
- Content Editors: ✅ Draft games only
- SysAdmin: ✅ All games

**Publish Assets (Draft → Live)**:
- Content Editors with Publish Permission: ✅ Yes
- SysAdmin: ✅ Yes
- Content Editors without Publish Permission: ❌ No

**Deploy Hotfix**:
- SysAdmin: ✅ Yes (with approval)
- All others: ❌ No

**Rollback Assets**:
- SysAdmin: ✅ Yes
- All others: ❌ No

**Delete Assets**:
- SysAdmin: ✅ Yes (soft delete, preserves history)
- All others: ❌ No

**Access Asset Logs**:
- Content Editors: ✅ Own uploads only
- SysAdmin: ✅ All uploads

## Supporting Documents Referenced

This games assets pipeline specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Original game asset organization and structure
- [Games Loaded.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Sheet1.csv) — Game library showing asset references and availability
- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser compatibility requirements affecting asset formats

## Dependencies

### Internal Dependencies

- **[Game Model](game-model.md)**: Asset URLs stored in game entity, health signals reference asset status
- **[Games CRUD](games-crud.md)**: Asset validation enforced during create and publish operations
- **[Games Metadata](games-metadata.md)**: Asset requirements defined in metadata fields
- **[Games Stages Policy](games-stages-policy.md)**: Stage enablement requires corresponding asset bundles
- **[Games Registry Overview](games-registry-overview.md)**: Admin UI for asset upload and management
- **[Game Player Shell](../03-student-experience/game-player-shell.md)**: Loads and renders HTML5 bundles
- **[Permissions Table](../02-roles-and-permissions/permissions-table.md)**: Permission checks for asset operations
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Asset telemetry follows canonical event schema
- **[Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md)**: Asset bundles must support minimum browser versions

### External Dependencies

- **Object Storage Service**: AWS S3, Google Cloud Storage, or Azure Blob Storage for asset persistence
- **CDN Provider**: Cloudflare, AWS CloudFront, or Google Cloud CDN for global asset delivery
- **Image Optimization Service**: ImageMagick, Sharp, or cloud-based service for thumbnail generation and optimization
- **Video Transcoding Service**: FFmpeg, AWS Elemental MediaConvert, or similar for video processing
- **Asset Processing Queue**: Background job system (e.g., Redis Queue, AWS SQS, Google Pub/Sub) for async processing
- **Headless Browser**: Puppeteer, Playwright, or BrowserStack for smoke testing and screenshot generation
- **Virus Scanning Service**: ClamAV or cloud-based antivirus for uploaded file scanning (security)

### Third-Party Libraries

Games may use third-party JavaScript libraries, which must be:
- Declared in `manifest.json` with version numbers
- Whitelisted by platform (security review for popular libraries)
- Bundled with game (no external CDN links to avoid availability issues)
- Licensed appropriately for commercial use

**Common Whitelisted Libraries**:
- Phaser.js (game framework)
- Tone.js (audio synthesis and scheduling)
- PixiJS (2D rendering engine)
- Howler.js (audio library)
- MIDI.js (MIDI playback)

## Open questions

### Asset Storage Cost Optimization

- **Question**: Should we implement automatic archival of old game versions to reduce storage costs?
- **Context**: Keeping all historical versions indefinitely increases object storage costs
- **Options**:
  - A) Keep all versions forever (simplest, highest cost)
  - B) Archive versions older than 6 months to cold storage (reduced cost, slower retrieval)
  - C) Delete deprecated game assets after 2 years if no active references (lowest cost, potential data loss)
- **Current Assumption**: Option A (keep all versions), reassess if storage costs exceed threshold
- **Decision Needed By**: Phase 2

### Video Streaming vs Embedded

- **Question**: Should video assets be served via streaming protocol (HLS, DASH) or embedded directly in bundles?
- **Context**: Large video files increase bundle size and load time; streaming reduces initial load but requires infrastructure
- **Options**:
  - A) Embed videos in bundles (simple, works offline, but slow initial load)
  - B) Stream videos via separate service (fast initial load, requires streaming infrastructure)
  - C) Hybrid: Embed short videos (<2 min), stream long videos (best of both worlds, complexity)
- **Trade-offs**: Streaming infrastructure adds operational complexity but significantly improves performance for video-heavy games
- **Decision Needed By**: Phase 1 beta (if video games planned for launch)

### Asset Approval Workflow

- **Question**: Should asset uploads require explicit approval before deployment, or trust Content Editors to publish directly?
- **Context**: Approval workflow ensures quality but slows velocity; automated validation reduces need for manual review
- **Options**:
  - A) No approval required, automated validation only (fast, risk of quality issues)
  - B) Peer review required (one Content Editor reviews another's upload before publish)
  - C) QA approval required for all uploads (highest quality, slowest)
  - D) Risk-based: New games require approval, updates to existing games auto-approved
- **Current Assumption**: Option A with strong automated validation; manual spot checks by QA
- **Decision Needed By**: Phase 1 alpha

### Fallback Asset Strategy

- **Question**: Should we maintain lightweight fallback assets for slow connections or older browsers?
- **Context**: Students on 3G connections or older devices may struggle with large bundles
- **Options**:
  - A) Single asset bundle optimized for modern browsers (simplest, may exclude some users)
  - B) Adaptive loading: Detect connection speed and serve optimized bundle variant (best UX, complexity)
  - C) Graceful degradation: Games detect capabilities and disable non-essential features (middle ground)
- **Trade-offs**: Adaptive loading provides best experience but requires bandwidth detection and multiple bundle variants
- **Decision Needed By**: Phase 1 beta

### Asset Versioning Granularity

- **Question**: Should we version assets at game level (one version for all stages) or stage level (independent versioning per stage)?
- **Context**: Stage-level versioning allows targeted updates but increases complexity
- **Options**:
  - A) Game-level versioning: All stages share same version number (simple, atomic updates)
  - B) Stage-level versioning: Each stage independently versioned (flexible, allows hotfixes per stage)
  - C) Hybrid: Major versions at game level, minor versions at stage level
- **Current Assumption**: Option A (game-level versioning) for simplicity, can migrate to B if needed
- **Decision Needed By**: Phase 1 alpha

### CDN Failover Strategy

- **Question**: Should we configure multi-CDN failover for critical asset availability?
- **Context**: Single CDN outage could prevent students from loading games
- **Options**:
  - A) Single CDN with high SLA (99.9%+), accept rare outages (lowest cost, some risk)
  - B) Multi-CDN with automatic failover (highest availability, doubled cost and complexity)
  - C) Origin fallback: If CDN fails, serve from origin storage (middle ground, slower but available)
- **Trade-offs**: Multi-CDN provides best availability but significantly increases operational complexity and cost
- **Decision Needed By**: Phase 2 (post-launch, based on reliability requirements)

### Asset Localization

- **Question**: How should we handle localized assets (text, audio, images) for international markets?
- **Context**: Future internationalization may require translated game content
- **Options**:
  - A) Separate bundles per language (e.g., `/games/G-01542/v3/en/`, `/games/G-01542/v3/es/`)
  - B) Single bundle with locale detection and dynamic content loading
  - C) Text-only localization in external files, all other assets shared
- **Current Assumption**: Not addressing localization in Phase 1; will revisit for international expansion
- **Decision Needed By**: Phase 3 (internationalization phase)
