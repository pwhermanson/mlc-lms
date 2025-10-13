# Phase 2 Scope

This document defines the exact scope, deliverables, and implementation requirements for **Phase 2: First Game Integration** of the MLC LMS rebuild.

> **Context**: See [initial-project-phases.md](../00-foundations/initial-project-phases.md) for strategic overview and phasing philosophy. Phase 2 builds directly on the foundation established in [phase-1-scope.md](./phase-1-scope.md).

## Purpose

Phase 2 delivers the core pedagogical experience by integrating the first educational game into the platform and creating a functional student learning workflow. This phase validates the game container architecture, establishes the progress tracking system, and demonstrates end-to-end student gameplay with score recording.

**What this enables:**
- Students can discover and play music learning games within the platform
- Game progress and scores are accurately tracked and displayed
- Game assets are efficiently stored and delivered via CDN
- Students can see their learning progress on an enhanced dashboard
- Foundation for Assignment Play and full curriculum integration in Phase 3+

**Why it exists:**
- **Value delivery**: Creates the first real educational experience for students
- **Technical validation**: Proves the game container architecture works with real games
- **Risk mitigation**: Identifies integration challenges early with a single game before scaling
- **User feedback**: Enables real testing of the student learning experience
- **Foundation building**: Establishes patterns for integrating remaining 200+ games

**Success means**: Real students can discover a game, play through all stages (Learn, Play, Quiz), see their scores recorded, and view progress on their dashboard without critical bugs.

---

## Scope

### What IS Included in Phase 2

#### 1. Game Container System
- **Chrome-less game player shell**
  - Full-screen game viewport (removes header/footer during play)
  - Minimal overlay controls (pause, exit, help)
  - Proper focus management and accessibility
  - Loading states and error handling
- **Stage lifecycle management**
  - Pre-load validation (device capabilities check)
  - Launch phase (enter full-screen mode)
  - Execution phase (monitor input and score)
  - Completion phase (capture results and display next actions)
- **Input handling foundation**
  - Keyboard input (full accessibility)
  - Touch support for tablets
  - Device compatibility checking
  - Graceful degradation for unsupported features

#### 2. First Game Integration
- **Game selection criteria**
  - Choose simplest existing game for integration (non-MIDI, visual-only)
  - Ideally a Staff game or Note Reading game from Level 1
  - Must have all standard stages (Learn, Play, Quiz)
  - Must be browser-compatible (HTML5, no Flash)
- **Game adaptation**
  - Convert game to work within new container system
  - Integrate with platform authentication and session
  - Connect to score recording API
  - Implement proper stage transitions
- **Stage implementation**
  - Learn stage: Tutorial/introduction mode
  - Play stage: Practice with feedback
  - Quiz stage: Assessment with target score
  - Review stage: Retention check (scheduled for later)
  - Challenge stage: Optional competitive mode (deferred to Phase 3)

#### 3. File Storage Infrastructure
- **Cloudflare R2 object storage**
  - R2 bucket setup (dev, staging, production)
  - Bucket structure for game assets (`/games/{game_id}/v{version}/`)
  - Asset organization by stage type
  - Metadata tagging for cache management
- **CDN configuration**
  - Cloudflare CDN integration with R2
  - Custom domain setup (`cdn.musiclearningcommunity.com`)
  - Cache policies by asset type (immutable versioned content)
  - Compression (Brotli, Gzip) for text assets
- **Asset upload workflow**
  - Pre-signed URL generation for secure uploads
  - Direct upload to R2 (bypasses app server)
  - Asset validation (size, format, virus scan)
  - CDN cache warming after upload

#### 4. Student Progress Tracking
- **Score recording system**
  - Score submission API endpoint
  - Database schema for student scores
  - Best score tracking per stage
  - Full attempt history preservation
- **Progress state management**
  - Stage completion tracking (Learn, Play, Quiz)
  - Mastery calculation (target score achievement)
  - Time spent and attempt count tracking
  - Session-based progress persistence
- **Data model**
  - Student progress table: `student_id`, `game_id`, `stage_id`, `score`, `passed`, `completed_at`
  - Scores table: `score_id`, `student_id`, `stage_id`, `score`, `max_score`, `duration_ms`, `attempts`, `submitted_at`
  - Indexes for efficient querying by student and game

#### 5. Enhanced Student Dashboard
- **Progress visualization**
  - "My Progress" section showing recently played games
  - Score display with pass/fail indicators
  - Visual progress bars or completion badges
  - "Continue Learning" CTA for incomplete games
- **Game discovery**
  - Featured game spotlight (the integrated game)
  - "All Games" preview/link (full library in Phase 3)
  - Simple game browsing by category
  - Search foundation (basic text search)
- **Dashboard layout**
  - Welcome message with student name
  - Quick stats: games played, average score, achievements earned
  - Recent activity feed
  - Next up: recommended next game or assignment

#### 6. Basic "All Games" Library (Single Game)
- **Game selection interface**
  - Two-tier navigation: Select Game → Select Stage
  - Game card with thumbnail, name, and description
  - Stage selector: Learn, Play, Quiz buttons
  - Clear indication of completion status
- **Game information display**
  - Game ID (e.g., `G-01542`)
  - Game name and description
  - Skill level (Beginner, Intermediate, etc.)
  - Skill area (Staff, Rhythm, Melody, etc.)
  - Visual vs. Aural indicator
  - Target score and pass threshold
- **Navigation flow**
  - Dashboard → All Games → Game Details → Stage Selection → Game Play → Results → Dashboard

#### 7. Game Telemetry and Events
- **Core gameplay events** (see [event-model.md](../15-analytics-and-reporting/event-model.md))
  - `game_stage_launch`: When student starts a stage
  - `game_stage_complete`: When student finishes a stage
  - `game_stage_quit`: When student exits without completing
  - `score_submit`: When score is recorded
  - `target_met`: When pass threshold is achieved
- **Asset loading events**
  - `asset_load_start`: Asset fetch begins
  - `asset_load_complete`: Asset successfully loaded
  - `asset_load_error`: Asset fails to load
- **Performance metrics**
  - Game load time (target: <2 seconds to interactive)
  - Frame rate monitoring (target: 60fps)
  - Memory usage tracking

#### 8. API Endpoints
- `GET /api/games` - List available games (initially returns one game)
- `GET /api/games/:gameId` - Get game details and stages
- `GET /api/games/:gameId/stages/:stageId` - Get stage-specific data
- `POST /api/play/start` - Initialize game session
- `POST /api/play/score` - Submit score after completion
- `GET /api/student/progress` - Get student progress overview
- `GET /api/student/scores/:gameId` - Get student scores for specific game

#### 9. Database Schema Extensions
**Games table** (minimal, registry in Phase 3+):
```
games
  - game_id: varchar(50) PK
  - slug: varchar(100) unique
  - name: varchar(255)
  - description: text
  - category: varchar(50)
  - level: varchar(50)
  - mode: enum(visual, aural, mixed)
  - status: enum(draft, live, deprecated)
  - created_at: timestamp
  - updated_at: timestamp
```

**Game Stages table**:
```
game_stages
  - stage_id: varchar(100) PK (e.g., G-01542-quiz)
  - game_id: varchar(50) FK -> games.game_id
  - stage_type: enum(learn, play, quiz, challenge, review)
  - target_score: integer
  - pass_threshold: integer
  - time_limit_seconds: integer nullable
  - bundle_url: varchar(500)
  - enabled: boolean
```

**Student Scores table**:
```
student_scores
  - score_id: uuid PK
  - student_id: uuid FK -> users.id
  - stage_id: varchar(100) FK -> game_stages.stage_id
  - score: integer
  - max_score: integer
  - passed: boolean
  - duration_ms: integer
  - attempts: integer
  - device_profile: varchar(50)
  - submitted_at: timestamp
```

**Student Progress table** (summary view):
```
student_progress
  - progress_id: uuid PK
  - student_id: uuid FK -> users.id
  - game_id: varchar(50) FK -> games.game_id
  - learn_completed: boolean
  - play_completed: boolean
  - quiz_completed: boolean
  - quiz_best_score: integer nullable
  - quiz_passed: boolean
  - last_played_at: timestamp
  - total_time_spent_ms: bigint
  - updated_at: timestamp
```

**Indexes**:
- `student_scores.student_id, submitted_at` (for student history)
- `student_scores.stage_id, score DESC` (for leaderboards)
- `student_progress.student_id, last_played_at DESC` (for dashboard)

#### 10. Testing Coverage
- **Unit tests**
  - Score calculation and validation
  - Progress state transitions
  - API endpoint logic
- **Integration tests**
  - Game launch and completion flow
  - Score submission and retrieval
  - CDN asset loading
- **E2E tests**
  - Full student journey: login → find game → play → see score
  - Game completion and progress update
  - Dashboard displays correct progress

---

### What is EXCLUDED from Phase 2

**NOT in Phase 2** (deferred to later phases):

- ❌ Assignment creation and assignment play (Phase 3)
- ❌ Teacher assignment builder and reports (Phase 3)
- ❌ Sequences and curriculum mapping (Phase 3)
- ❌ Multiple game integration (Phase 3: add 5-10 more games)
- ❌ MIDI keyboard support (Phase 3+)
- ❌ Microphone input for aural games (Phase 3+)
- ❌ Challenge boards and competitive features (Phase 3)
- ❌ Badges, points, streaks, and gamification (Phase 3)
- ❌ Leaderboards (Phase 3)
- ❌ Student messaging and notifications (Phase 3)
- ❌ Full games registry and authoring tools (Phase 4+)
- ❌ Game importer and bulk import (Phase 4+)
- ❌ Payment integration (Phase 4+)
- ❌ Admin subscriber management (Phase 4+)
- ❌ Advanced admin tools (Phase 4+)
- ❌ Full search and filtering (Phase 3)
- ❌ Social features and collaboration (Phase 4+)
- ❌ Mobile app (PWA features in Phase 4+)

**Rationale:** Phase 2 focuses exclusively on proving the game integration architecture and student learning experience with a single game. Additional games, teacher features, and advanced functionality build on this foundation in Phases 3-4 after the core experience is validated.

---

## Data model (if applicable)

See **Database Schema Extensions** in the Scope section above for complete entity definitions.

### Key Relationships

#### Student → Student Progress
- One-to-many relationship
- Each student has progress records for games they've played
- Progress summary updated on each game completion

#### Student Progress → Scores
- One-to-many relationship
- Each progress record links to many score attempts
- Best score and most recent attempt are denormalized for performance

#### Game → Game Stages
- One-to-many relationship
- Each game has 3-5 stages (Learn, Play, Quiz, Challenge, Review)
- Stage order and gating rules defined at stage level

#### Game Stages → Student Scores
- One-to-many relationship
- Each stage accumulates scores from all students
- Used for leaderboards and analytics

### Foreign Key Constraints
- `game_stages.game_id` references `games.game_id` ON DELETE CASCADE
- `student_scores.student_id` references `users.id` ON DELETE CASCADE
- `student_scores.stage_id` references `game_stages.stage_id` ON DELETE CASCADE
- `student_progress.student_id` references `users.id` ON DELETE CASCADE
- `student_progress.game_id` references `games.game_id` ON DELETE CASCADE

---

## Behavior and rules

### Game Discovery and Selection Flow

#### Student navigates to "All Games"
1. Dashboard displays "Play Games" or "All Games" button
2. Click navigates to `/student/games` route
3. Initially displays single featured game (Phase 2 MVP)
4. Game card shows: thumbnail, name, description, skill level, completion status

#### Student selects game
1. Click game card navigates to `/student/games/:gameId`
2. Game details page displays:
   - Game information (ID, name, description, category, level)
   - Stage buttons (Learn, Play, Quiz)
   - Student's best scores for each stage
   - Progress indicators (completed stages highlighted)
   - "Start" or "Continue" CTA for next stage

#### Student selects stage
1. Click stage button (e.g., "Play")
2. System checks prerequisites:
   - Learn stage: Always available
   - Play stage: Available after attempting Learn
   - Quiz stage: Available after attempting Play
3. If prerequisites not met, display friendly message: "Complete [Learn] first"
4. If prerequisites met, navigate to game player: `/student/play/:gameId/:stageId`

### Game Play Flow

#### Game initialization
1. **Pre-load phase**
   - Display loading screen with game thumbnail
   - Fetch game bundle from CDN: `https://cdn.mlc.com/games/{gameId}/v{version}/{stageType}/index.html`
   - Validate device capabilities (screen size, browser features)
   - Initialize game session: `POST /api/play/start` returns `session_id`
2. **Launch phase**
   - Enter chrome-less full-screen mode (hide header/footer)
   - Display minimal overlay controls (pause, exit)
   - Initialize game runtime with student context
   - Fire telemetry: `game_stage_launch`

#### During gameplay
1. **Execution monitoring**
   - Track time elapsed (duration_ms)
   - Monitor score updates from game
   - Handle pause/resume if student clicks pause button
   - Handle exit confirmation if student clicks exit button
2. **Score tracking**
   - Game reports score events via postMessage API
   - Player shell tracks current_score and max_score
   - No server communication during play (offline-capable)

#### Game completion
1. **Capture results**
   - Game signals completion via postMessage: `{ event: 'complete', score: 85, max_score: 100 }`
   - Player shell captures final score and duration
2. **Submit score**
   - POST `/api/play/score` with: `{ session_id, stage_id, score, max_score, duration_ms, attempts: 1 }`
   - Server validates score and writes to `student_scores` table
   - Server updates `student_progress` summary
   - Fire telemetry: `game_stage_complete`, `score_submit`
3. **Display results**
   - Exit full-screen mode, restore header/footer
   - Display result panel:
     - Score achieved: "85 / 100"
     - Pass/fail indicator: "Great job! You passed!" or "Try again to reach 80"
     - Next action buttons: "Try Again", "Next Stage", "Return to Dashboard"
   - If target score met, fire telemetry: `target_met`

### Score Recording Rules

#### Score validation
- **Score range**: 0 ≤ score ≤ max_score
- **Stage match**: score must be for a valid stage_id
- **Session validity**: session_id must match active session
- **Duplicate prevention**: Only one score per session_id
- **Timestamp**: submitted_at set to server time (prevent client-side tampering)

#### Score persistence
- **Best score tracking**: Update student_progress.quiz_best_score if new score is higher
- **Attempt counting**: Increment attempts in student_progress
- **History preservation**: All scores saved to student_scores for analytics
- **Mastery flag**: Set quiz_passed = true if score ≥ pass_threshold

#### Score reconciliation (free play vs. assignments)
- Phase 2: All plays are "free play" (no assignments yet)
- Phase 3: Assignment engine will check for prior free play scores
- Policy: Free play scores can count toward assignment completion (configurable by teacher)

### Progress State Transitions

#### Learn Stage
- **Initial state**: Always available
- **In progress**: After launch, before completion
- **Complete**: After any score submission (no pass threshold)
- **Effect**: Unlocks Play stage

#### Play Stage
- **Initial state**: Locked until Learn attempted
- **Available**: After Learn completion
- **In progress**: After launch, before completion
- **Complete**: After score submission ≥ target_score (encouragement score, not required)
- **Effect**: Unlocks Quiz stage

#### Quiz Stage
- **Initial state**: Locked until Play attempted
- **Available**: After Play completion
- **In progress**: After launch, before completion
- **Complete**: After score submission ≥ pass_threshold
- **Mastery**: quiz_passed flag set to true when pass_threshold met
- **Effect**: Game marked as complete on dashboard

#### Challenge Stage (deferred to Phase 3)
- Always optional, never gates progression

#### Review Stage (deferred to Phase 3)
- Scheduled by assignments engine after time delay

### Asset Loading and Caching

#### Asset delivery flow
1. Browser requests: `https://cdn.mlc.com/games/G-01542/v3/play/index.html`
2. Cloudflare edge cache check:
   - **Cache hit**: Serve immediately (1-5ms latency)
   - **Cache miss**: Fetch from R2 origin, cache at edge, serve (50-200ms latency)
3. Browser loads game bundle (HTML, JS, CSS, images, audio)
4. Game initializes and connects to parent window (postMessage API)

#### Cache policies
- **Game bundles** (versioned): `Cache-Control: public, max-age=31536000, immutable` (1 year TTL)
- **Game thumbnails**: `Cache-Control: public, max-age=86400` (1 day TTL)
- **Audio assets**: `Cache-Control: public, max-age=2592000` (30 days TTL)

#### Cache invalidation
- New game version deployed: New URL path (`/v4/` instead of `/v3/`)
- No manual cache purging required for versioned assets
- Thumbnail updates: Manual purge via Cloudflare API

---

## UX requirements (if applicable)

### Game Player Shell Layout

#### Full-screen mode (during gameplay)
- **Viewport**: Game occupies entire browser viewport
- **Chrome removal**: Hide global header, footer, navigation
- **Overlay controls**: Minimal floating controls in corner
  - Pause button (icon: pause symbol)
  - Exit button (icon: X, requires confirmation)
  - Help button (icon: ?, context-sensitive)
- **Focus management**: Game iframe receives focus, keyboard events captured
- **Accessibility**: Skip to controls link, announce state changes

#### Results panel (after completion)
- **Header**: "Quiz Complete!" or "Try Again"
- **Score display**: Large text, e.g., "85 / 100"
- **Pass indicator**: Checkmark ✓ or "Passed!" if above threshold, else "Try again to reach 80"
- **Action buttons**: Row of 3 buttons
  - "Try Again" (restart same stage)
  - "Next Stage" (navigate to Play → Quiz, or Dashboard if Quiz complete)
  - "Return to Dashboard"
- **Progress update**: "Your progress has been saved" confirmation message

### Student Dashboard Layout

#### Welcome section
- **Greeting**: "Welcome back, [FirstName]!"
- **Quick stats**: Cards or pills showing:
  - Games Played: 1
  - Best Score: 85
  - Achievements: 0 (future)

#### My Progress section
- **Header**: "My Progress"
- **Progress cards**: One card per played game
  - Game thumbnail
  - Game name
  - Stage completion indicators: Learn ✓, Play ✓, Quiz ⏱️
  - Best score for Quiz
  - "Continue" button if incomplete, "Review" button if complete

#### Featured Game section
- **Header**: "Featured Game" or "Try This Game"
- **Large game card**:
  - Game thumbnail (larger than progress cards)
  - Game name and description
  - Skill level and category
  - "Start Playing" CTA button

#### Navigation
- **Sidebar or top nav** (consistent with Phase 1):
  - Dashboard (active)
  - My Assignments (placeholder, Phase 3)
  - All Games
  - Leaderboards (placeholder, Phase 3)
  - Profile
  - Logout

### All Games Library (Single Game MVP)

#### Game list view
- **Header**: "All Games" with count: "(1 game)"
- **Filter controls** (placeholder): Level, Category, Skill (disabled for now)
- **Game grid**: Single game card
  - Thumbnail
  - Game name
  - Skill level badge
  - Skill area badge
  - Completion status: "Not started" or "In progress" or "Complete ✓"
  - Click navigates to game details

#### Game details view
- **Breadcrumb**: All Games > [Game Name]
- **Game header**:
  - Game thumbnail (medium size)
  - Game name (large)
  - Game ID (small, subtle)
  - Skill level and skill area
  - Visual/Aural indicator
- **Description**: Text block explaining what the game teaches
- **Stages section**: Cards or buttons for each stage
  - Learn: "Introduction and Tutorial"
    - Button: "Start Learn" or "Replay Learn" if completed
  - Play: "Practice Mode"
    - Button: "Start Play" or "Replay Play" if completed
    - Locked indicator if Learn not completed
  - Quiz: "Assessment Mode"
    - Button: "Start Quiz" or "Retry Quiz" if completed
    - Locked indicator if Play not completed
    - Best score displayed if completed
- **Action buttons**: "Start [Next Stage]" primary CTA, "Back to All Games" secondary

### Accessibility Requirements

See [a11y-standards.md](../01-ux-design-system/a11y-standards.md) for complete requirements. Phase 2 specific:

- ✅ Game player shell has proper focus management
- ✅ Overlay controls are keyboard accessible
- ✅ Results panel announced to screen readers
- ✅ Progress updates announced (`aria-live="polite"`)
- ✅ Stage completion visually indicated and announced
- ✅ Color contrast meets WCAG AA for all text
- ✅ Game thumbnails have descriptive alt text
- ⚠️ Full WCAG 2.1 Level AA audit deferred to Phase 3

### Mobile Considerations

See [responsive-breakpoints.md](../01-ux-design-system/responsive-breakpoints.md) for detailed specifications. Phase 2 targets:

- **Mobile gameplay**: Games designed for 375px width minimum
- **Touch controls**: Touch targets ≥44px for overlay buttons
- **Orientation**: Support both portrait and landscape modes
- **Performance**: Target 60fps on modern mobile devices (iOS 14+, Android 10+)
- **Network**: Optimize asset loading for 3G connections (target <5MB bundle size)

---

## Telemetry

See [event-model.md](../15-analytics-and-reporting/event-model.md) for complete event specifications. Phase 2 implements core gameplay telemetry:

### Events Tracked

#### `game_stage_launch`
**When:** Student starts a game stage  
**Where:** Client-side when game player initializes  
**Properties:**
- `game_id`: Game identifier (e.g., "G-01542")
- `stage_id`: Stage identifier (e.g., "G-01542-quiz")
- `stage_type`: "learn" | "play" | "quiz"
- `student_id`: UUID of student
- `session_id`: Game session identifier
- `device_profile`: "desktop" | "tablet" | "mobile"
- `input_mode`: "keyboard" | "touch"
- `timestamp`: ISO 8601 timestamp

#### `game_stage_ready`
**When:** Game bundle loaded and ready to play  
**Where:** Client-side after asset loading complete  
**Properties:**
- `game_id`, `stage_id`, `session_id`
- `load_time_ms`: Time from launch to ready (target: <2000ms)
- `asset_cache_hit`: Boolean (was bundle served from cache?)
- `bundle_size_bytes`: Size of loaded bundle
- `timestamp`: ISO 8601 timestamp

#### `game_stage_complete`
**When:** Student completes a game stage  
**Where:** Server-side after score submission  
**Properties:**
- `game_id`, `stage_id`, `student_id`, `session_id`
- `score`: Score achieved (e.g., 85)
- `max_score`: Maximum possible score (e.g., 100)
- `passed`: Boolean (score ≥ pass_threshold?)
- `duration_ms`: Time spent playing (milliseconds)
- `attempts`: Number of attempts (1 for first try)
- `timestamp`: ISO 8601 timestamp

#### `game_stage_quit`
**When:** Student exits game without completing  
**Where:** Client-side when exit button clicked  
**Properties:**
- `game_id`, `stage_id`, `student_id`, `session_id`
- `elapsed_ms`: Time spent before quitting
- `reason`: "user_exit" | "timeout" | "error"
- `current_score`: Score at time of exit (if available)
- `timestamp`: ISO 8601 timestamp

#### `score_submit`
**When:** Score is submitted to server  
**Where:** Server-side when POST /api/play/score processes request  
**Properties:**
- `score_id`: UUID of score record
- `stage_id`, `student_id`, `session_id`
- `score`, `max_score`, `passed`
- `latency_ms`: Time to process submission
- `timestamp`: ISO 8601 timestamp

#### `target_met`
**When:** Student achieves target score for first time  
**Where:** Server-side when pass_threshold first exceeded  
**Properties:**
- `game_id`, `stage_id`, `student_id`
- `score`: Passing score
- `pass_threshold`: Target score (e.g., 80)
- `attempts_to_pass`: Number of attempts before passing
- `timestamp`: ISO 8601 timestamp

#### `asset_load_complete`
**When:** Game asset finishes loading  
**Where:** Client-side during game bundle load  
**Properties:**
- `asset_url`: CDN URL of asset
- `asset_type`: "bundle" | "audio" | "image"
- `load_time_ms`: Time to load asset
- `size_bytes`: Asset size
- `cache_hit`: Boolean (served from browser or CDN cache?)
- `timestamp`: ISO 8601 timestamp

#### `cdn_error`
**When:** CDN asset fails to load  
**Where:** Client-side if asset fetch fails  
**Properties:**
- `asset_url`: Failed asset URL
- `error_code`: HTTP status code (404, 500, etc.)
- `error_message`: Browser error message
- `game_id`, `stage_id` (if applicable)
- `timestamp`: ISO 8601 timestamp

### Implementation

**Phase 2 telemetry storage:**
- Events written to dedicated `analytics_events` table
- Audit logs for critical events (score submission, completion)
- Console logging in development for debugging
- No external analytics platform in Phase 2 (deferred to Phase 3)

**Event batching:**
- Client-side events batched and sent every 10 seconds or on page unload
- Critical events (score submission) sent immediately
- Failed events stored in localStorage and retried

**Sampling:**
- Game completion and score events: 100% (never sampled)
- Asset loading events: 10% sampling in production (high volume)
- Performance metrics: 25% sampling

---

## Permissions

See [roles-matrix.md](../02-roles-and-permissions/roles-matrix.md) for complete permission specifications. Phase 2 implements minimal game access permissions:

### Roles in Phase 2

#### Students
- **Read**: Can view available games and their own progress
- **Play**: Can launch and play any "live" game
- **Submit Scores**: Can submit scores for completed stages
- **View Progress**: Can see their own scores and completion status

#### Teachers
- **All Student permissions**: Teachers can play games like students
- **View Student Progress**: Can see progress for students in their classes (foundation for Phase 3 reports)
- **No game management**: Cannot create/edit games (Phase 4+)

#### System Admin
- **All Teacher permissions**
- **Game Management**: Can create, edit, and publish games
- **View All Progress**: Can see all student progress
- **CDN Management**: Can upload game assets and manage CDN

### Permission Checks

**Route protection:**
- `/student/games` - Authenticated students only
- `/student/play/:gameId/:stageId` - Authenticated students, game must be "live" status
- `/teacher/reports` - Teachers only (placeholder for Phase 3)
- `/admin/games` - System Admin only

**API endpoints:**
- `GET /api/games` - Authenticated users (returns games based on role and status)
- `POST /api/play/start` - Students only
- `POST /api/play/score` - Students only, must own session_id
- `GET /api/student/progress` - Students can read own, teachers can read their students, admins can read all

**Game visibility:**
- **Live games**: Visible to all authenticated users
- **Draft games**: Visible to System Admin only
- **Deprecated games**: Hidden from game lists but playable if student has prior progress

---

## Supporting Documents Referenced

This Phase 2 scope specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game stage definitions (Learn, Play, Quiz, Challenge, Review), scoring mechanics, and player shell requirements
- [Games Loaded.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Sheet1.csv) — Existing game catalog with IDs, categories, and asset structures
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Game metadata and categorization for integration planning

## Dependencies

### Internal Dependencies

**Foundational documents:**
- [initial-project-phases.md](../00-foundations/initial-project-phases.md) - Strategic context and phasing philosophy
- [phase-1-scope.md](./phase-1-scope.md) - Phase 1 foundation (authentication, database, deployment)
- [product-vision.md](../00-foundations/product-vision.md) - Overall product goals and principles
- [pedagogy-principles.md](../00-foundations/pedagogy-principles.md) - Learning design principles

**Game and content:**
- [game-model.md](../08-games-registry-and-authoring/game-model.md) - Game structure, stages, and scoring
- [game-player-shell.md](../03-student-experience/game-player-shell.md) - Player runtime requirements
- [assignment-model.md](../07-content-architecture/assignment-model.md) - Foundation for Phase 3 assignments
- [game-taxonomy.md](../07-content-architecture/game-taxonomy.md) - Game categorization

**Architecture:**
- [system-overview.md](../18-architecture/system-overview.md) - High-level architecture
- [data-model-erd.md](../18-architecture/data-model-erd.md) - Complete database schema
- [cdn-and-media.md](../17-integrations/cdn-and-media.md) - Cloudflare R2 and CDN setup
- [caching-strategy.md](../18-architecture/caching-strategy.md) - Cache policies
- [api-design-principles.md](../18-architecture/api-design-principles.md) - API conventions

**UX and design:**
- [responsive-breakpoints.md](../01-ux-design-system/responsive-breakpoints.md) - Screen size specifications
- [a11y-standards.md](../01-ux-design-system/a11y-standards.md) - Accessibility requirements
- [components-overview.md](../01-ux-design-system/components-overview.md) - Reusable component library

**Operations:**
- [test-strategy.md](../19-quality-and-operations/test-strategy.md) - Testing approach
- [release-management.md](../19-quality-and-operations/release-management.md) - Deployment process
- [observability.md](../19-quality-and-operations/observability.md) - Monitoring and logging

### External Dependencies

**Required services (in addition to Phase 1):**
- **Cloudflare R2** - Object storage for game assets
- **Cloudflare CDN** - Global content delivery network
- **Cloudflare Workers** (optional) - Edge computing for custom logic

**Development tools (in addition to Phase 1):**
- **Sharp** - Image optimization for thumbnails (Node.js library)
- **AWS SDK for JavaScript** - R2 integration (S3-compatible API)

**Game runtime (depends on selected game):**
- **HTML5 Canvas** - If game uses canvas rendering
- **Phaser 3** (optional) - If game uses Phaser framework
- **Web Audio API** - For game sound effects and music

**NOT in Phase 2** (deferred):
- ❌ MIDI Web API (Phase 3+ for MIDI keyboard games)
- ❌ Web Audio input (Phase 3+ for microphone/singing games)
- ❌ External analytics platform (PostHog, Mixpanel - Phase 3)
- ❌ Error monitoring (Sentry - Phase 3)
- ❌ Advanced CDN features (image resizing, video streaming - Phase 4)

### Phase 3 Blockers

Phase 3 (Assignments Engine and Multiple Games) cannot begin until Phase 2 delivers:
- ✅ Working game player shell with chrome-less full-screen mode
- ✅ Score recording system with database persistence
- ✅ Student progress tracking and dashboard display
- ✅ CDN infrastructure with at least one game deployed
- ✅ Game stage lifecycle (Learn, Play, Quiz) functional
- ✅ Telemetry events for gameplay and performance

**Critical path:** Game player shell architecture must support multiple games without modification. Phase 2 design should anticipate Phase 3 needs for assignment generation and sequence integration.

---

## Open questions

### To Resolve Before Build

**1. Which Game to Integrate First?**
- **Question:** Which existing game should be selected for Phase 2 integration?
- **Criteria:**
  - Simple mechanics (no MIDI, no microphone required)
  - Browser-compatible (HTML5, no Flash)
  - Complete stages (Learn, Play, Quiz)
  - Representative of typical game structure
  - Level 1 or Beginner difficulty
- **Candidates:**
  - Grand Staff Guide Notes (Staff category, visual)
  - Rhythm Reading Simple Meters (Rhythm category, visual)
  - Note Name Recognition (Note Reading category, visual)
- **Recommendation:** Grand Staff Guide Notes A (G-01542) - foundational staff reading, pure visual, no audio complexity
- **Owner:** Product Lead + Tech Lead
- **Deadline:** Before Phase 2 kickoff

**2. Game Bundle Format**
- **Question:** What format should game bundles use for the player shell?
- **Options:**
  - A) Single HTML file with embedded assets (simplest, limited size)
  - B) HTML + separate JS/CSS/assets (standard web app)
  - C) iframe sandboxing with postMessage API (security, isolation)
- **Recommendation:** Option C (iframe + postMessage) for security and flexibility
- **Owner:** Tech Lead
- **Deadline:** Before Phase 2 development starts

**3. Score Submission Retry Logic**
- **Question:** How should the system handle score submission failures (network errors, server downtime)?
- **Options:**
  - A) Fail immediately, student must replay stage
  - B) Queue in localStorage, retry on next page load
  - C) Automatic retry with exponential backoff (3 attempts)
- **Recommendation:** Option C (automatic retry) + Option B (localStorage fallback) for reliability
- **Owner:** Tech Lead
- **Dependencies:** [api-design-principles.md](../18-architecture/api-design-principles.md)
- **Deadline:** Before Phase 2 development starts

**4. CDN Asset Upload Process**
- **Question:** How are game assets uploaded to R2 in Phase 2?
- **Options:**
  - A) Manual upload via Cloudflare dashboard (simplest, no automation)
  - B) Admin panel in MLC app (user-friendly, requires development)
  - C) CLI tool / GitHub Actions automation (developer-friendly, scalable)
- **Recommendation:** Option A for Phase 2 (one game only), Option C for Phase 3 (bulk import)
- **Owner:** DevOps Lead
- **Deadline:** Before first game deployment

**5. Game Session Timeout**
- **Question:** Should game sessions have a maximum duration before auto-save and exit?
- **Options:**
  - A) No timeout (session lasts until completion or manual exit)
  - B) 30-minute timeout with warning
  - C) 60-minute timeout with warning
- **Recommendation:** Option A for Phase 2 (simple games, short duration), revisit in Phase 3
- **Owner:** Product Lead
- **Deadline:** Before Phase 2 development starts

**6. Free Play vs. Assignment Scoring**
- **Question:** Should free play scores be stored differently than assignment scores in preparation for Phase 3?
- **Options:**
  - A) Same table, add `assignment_id` nullable field
  - B) Separate tables: `freeplay_scores` and `assignment_scores`
  - C) Single table, `context_type` enum field
- **Recommendation:** Option A (simple, flexible, supports Phase 3 assignment reconciliation)
- **Owner:** Tech Lead
- **Deadline:** Before database schema design

### Future Considerations (Not Blocking Phase 2)

**7. Game Performance Benchmarking**
- Establish performance baselines for load time, frame rate, memory usage
- **Timing:** Phase 2 testing and validation
- **Reference:** [observability.md](../19-quality-and-operations/observability.md)

**8. Game Accessibility Compliance**
- Full WCAG 2.1 Level AA audit for game player shell
- **Timing:** Phase 3 (after multiple games integrated)
- **Reference:** [a11y-standards.md](../01-ux-design-system/a11y-standards.md)

**9. CDN Cost Monitoring**
- Set up alerts for unexpected bandwidth spikes
- **Timing:** Phase 2 deployment to production
- **Reference:** [cdn-and-media.md](../17-integrations/cdn-and-media.md)

**10. Game Asset Versioning Strategy**
- Define process for updating game assets without breaking active sessions
- **Timing:** Phase 3 (when multiple games and updates needed)
- **Reference:** [content-versioning.md](../07-content-architecture/content-versioning.md)

---

## Phase 2 Timeline

**Total duration:** 2 weeks (10 business days)

### Week 1: Game Container and CDN Infrastructure
- **Days 1-2:** CDN and R2 setup (Cloudflare configuration, bucket creation, domain setup)
- **Days 3-5:** Game player shell development (full-screen mode, postMessage API, score capture)

### Week 2: Game Integration and Dashboard
- **Days 6-7:** First game integration (adapt game to container, test stages)
- **Days 8-9:** Student dashboard enhancements (progress display, game discovery)
- **Day 10:** E2E testing, bug fixes, deployment

**Milestone:** By end of Week 2, real students can log in, find a game, play through Learn/Play/Quiz stages, see scores recorded, and view progress on their dashboard.

---

## Success Criteria

Phase 2 is complete when:

✅ **Technical:**
- Game player shell renders games in chrome-less full-screen mode
- PostMessage API enables game-to-platform communication
- Score submission API accepts and validates scores
- Database stores student progress and scores correctly
- CDN delivers game assets with <2 second load time
- All tests pass (unit, integration, E2E)

✅ **Functional:**
- Students can discover the integrated game on dashboard
- Students can navigate to game details page
- Students can select and launch Learn stage
- Students can complete Learn and unlock Play stage
- Students can complete Play and unlock Quiz stage
- Students can complete Quiz and see score recorded
- Students can see progress on dashboard after completion
- Students can replay stages and improve scores

✅ **Quality:**
- Game loads in <2 seconds on 3G connection
- Game runs at 60fps on modern desktop browsers
- Score submission succeeds with retry logic on failure
- Game player shell is keyboard accessible
- Progress display updates correctly after score submission
- No console errors or warnings in production

✅ **User Experience:**
- Clear visual progress indicators on dashboard
- Intuitive navigation between dashboard, game library, and gameplay
- Friendly results panel after stage completion
- Responsive design works on desktop, tablet, and mobile

✅ **Documentation:**
- Game integration guide documented
- CDN asset upload process documented
- Score submission API documented
- Database schema and migrations documented

**Phase 2 does NOT require:**
- ❌ Multiple games (one game is sufficient)
- ❌ Assignment creation or sequencing
- ❌ Teacher reports or analytics
- ❌ MIDI or microphone support
- ❌ Gamification features (badges, points, leaderboards)
- ❌ Full WCAG 2.1 AA compliance (basic a11y only)

---

## Next Phase

Upon successful completion of Phase 2, proceed to:
- **[phase-3-scope.md](./phase-3-scope.md)** - Assignments Engine and Multiple Games

Phase 3 will build on this foundation to deliver:
- Assignment creation from sequences
- Assignment play workflow with gating and progression
- Integration of 5-10 additional games
- Teacher assignment builder and basic reports
- Enhanced student dashboard with assignments view
- Challenge boards and basic gamification

---

## Related Documentation

**Foundational context:**
- [initial-project-phases.md](../00-foundations/initial-project-phases.md) - Strategic overview and phasing philosophy
- [phase-1-scope.md](./phase-1-scope.md) - Phase 1 foundation (authentication, database, deployment)
- [product-vision.md](../00-foundations/product-vision.md) - Overall product vision and goals
- [pedagogy-principles.md](../00-foundations/pedagogy-principles.md) - Learning design principles

**Game architecture:**
- [game-model.md](../08-games-registry-and-authoring/game-model.md) - Game structure and stage definitions
- [game-player-shell.md](../03-student-experience/game-player-shell.md) - Player runtime specifications
- [games-registry-overview.md](../08-games-registry-and-authoring/games-registry-overview.md) - Game catalog management

**Content and curriculum:**
- [assignment-model.md](../07-content-architecture/assignment-model.md) - Assignment structure (foundation for Phase 3)
- [sequence-model.md](../09-sequences-and-curriculum-tools/sequence-model.md) - Curriculum sequencing (Phase 3)
- [game-taxonomy.md](../07-content-architecture/game-taxonomy.md) - Game categorization

**Student experience:**
- [student-dashboard.md](../03-student-experience/student-dashboard.md) - Dashboard design
- [all-games-library.md](../03-student-experience/all-games-library.md) - Game discovery interface
- [assignment-play.md](../03-student-experience/assignment-play.md) - Assignment workflow (Phase 3)

**Infrastructure:**
- [cdn-and-media.md](../17-integrations/cdn-and-media.md) - Cloudflare R2 and CDN specifications
- [caching-strategy.md](../18-architecture/caching-strategy.md) - Cache policies
- [data-model-erd.md](../18-architecture/data-model-erd.md) - Complete database schema

**Analytics:**
- [event-model.md](../15-analytics-and-reporting/event-model.md) - Telemetry event specifications
- [teacher-reports-kpis.md](../15-analytics-and-reporting/teacher-reports-kpis.md) - Reporting foundation (Phase 3)

---

**Document Status:** ✅ Complete and ready for Phase 2 implementation  
**Last Updated:** 2025-10-13  
**Owner:** Product & Engineering  
**Reviewers:** Tech Lead, UX Lead, Product Manager
