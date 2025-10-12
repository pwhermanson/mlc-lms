# Games Telemetry

## Purpose
Define the game runtime telemetry and instrumentation system that enables game authors, content editors, QA teams, and system administrators to monitor game performance, debug issues, track asset loading, and validate gameplay mechanics. This specification focuses on game-specific telemetry collection, configuration, and debugging capabilities that extend the platform-wide event model with game runtime details.

> **Related Documents**: For platform-wide event tracking, see [Event Model](../15-analytics-and-reporting/event-model.md). For game structure and lifecycle, see [Game Model](game-model.md). For game health monitoring, see [Games Audit and Health](games-audit-and-health.md). For registry-level administrative events, see [Games Registry Overview](games-registry-overview.md).

## Scope
**Included**
- Game runtime instrumentation and event emission requirements
- Per-game telemetry configuration flags and granularity controls
- Asset loading performance tracking and diagnostics
- Device interaction telemetry (MIDI, microphone, keyboard, touch)
- Game error tracking and debugging support
- Performance monitoring for game runtime (FPS, memory, latency)
- Development and QA-specific telemetry modes
- Integration patterns between game runtime and platform event system
- Telemetry for game stage transitions and scoring mechanics

**Excluded**
- Platform-wide event model definitions (covered in [Event Model](../15-analytics-and-reporting/event-model.md))
- Learning analytics and KPI calculations (covered in [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md))
- Registry administrative telemetry (covered in [Games Registry Overview](games-registry-overview.md))
- Sequence and assignment level tracking (covered in [Assignment Progress Tracking](../10-assignments-engine/assignment-progress-tracking.md))

## Data model (if applicable)

### Game Telemetry Configuration
Each game can configure telemetry granularity and emission patterns through telemetry flags defined in the game metadata:

```json
{
  "game_id": "G-01542",
  "telemetry_config": {
    "emit_per_note": false,
    "emit_per_prompt": true,
    "emit_per_interaction": false,
    "emit_performance_metrics": true,
    "emit_device_events": true,
    "emit_error_details": true,
    "sampling_rate": 1.0,
    "debug_mode": false,
    "qa_mode": false
  }
}
```

### Game Runtime Telemetry Schema
Game runtime telemetry extends the core event schema with game-specific properties:

```json
{
  "event_id": "evt_game_1234567890",
  "event_type": "game_runtime_performance",
  "timestamp": "2025-01-27T14:30:45.123Z",
  "user_id": "usr_12345",
  "session_id": "sess_67890",
  "game_context": {
    "game_id": "G-01542",
    "game_version": 3,
    "stage_id": "G-01542-quiz",
    "stage_version": 3,
    "run_id": "run_abc123",
    "assignment_id": "asg_789",
    "sequence_id": "seq_456"
  },
  "runtime_properties": {
    "bundle_url": "https://cdn.mlc/games/G-01542/v3/index.html",
    "bundle_version": "3.0.2",
    "engine_type": "html5|unity|phaser|custom",
    "rendering_mode": "canvas|webgl|svg",
    "audio_context_state": "running|suspended|closed"
  },
  "performance_metrics": {
    "fps_average": 58.4,
    "fps_min": 42.0,
    "memory_used_mb": 45.2,
    "memory_peak_mb": 67.8,
    "latency_ms": 12.5,
    "dropped_frames": 3
  },
  "privacy_level": "standard",
  "retention_days": 90
}
```

### Game Interaction Event Schema
Captures detailed gameplay interactions for debugging and analytics:

```json
{
  "event_type": "game_interaction",
  "game_context": {
    "game_id": "G-01542",
    "stage_id": "G-01542-play",
    "run_id": "run_abc123"
  },
  "interaction_properties": {
    "interaction_type": "note_played|prompt_answered|target_clicked|key_pressed",
    "input_mode": "midi|keyboard|mouse|touch|microphone",
    "correct": true,
    "response_time_ms": 450,
    "prompt_index": 15,
    "total_prompts": 20,
    "hint_shown": false,
    "retry_count": 0
  },
  "device_details": {
    "midi_device_name": "Yamaha USB-MIDI",
    "midi_note": 60,
    "midi_velocity": 87,
    "midi_latency_ms": 8.2
  }
}
```

### Asset Loading Telemetry Schema
Tracks game asset loading performance and issues:

```json
{
  "event_type": "game_asset_loaded",
  "game_context": {
    "game_id": "G-01542",
    "game_version": 3,
    "bundle_url": "https://cdn.mlc/games/G-01542/v3/index.html"
  },
  "asset_properties": {
    "asset_type": "bundle|audio|image|video|data",
    "asset_url": "https://cdn.mlc/games/G-01542/v3/audio-sprite.mp3",
    "asset_size_bytes": 2048576,
    "load_time_ms": 850,
    "cache_hit": true,
    "cdn_region": "us-east-1",
    "http_status": 200,
    "compression_type": "gzip"
  }
}
```

### Game Error Telemetry Schema
Captures runtime errors and exceptions for debugging:

```json
{
  "event_type": "game_error",
  "game_context": {
    "game_id": "G-01542",
    "stage_id": "G-01542-quiz",
    "run_id": "run_abc123"
  },
  "error_properties": {
    "error_code": "MIDI_DEVICE_DISCONNECTED",
    "error_message": "MIDI device lost connection during gameplay",
    "error_category": "device|asset|network|runtime|scoring",
    "severity": "critical|error|warning|info",
    "recoverable": true,
    "recovery_action": "prompt_reconnect",
    "stack_trace": "...",
    "browser_console_errors": ["..."]
  }
}
```

## Behavior and rules

### Telemetry Emission Rules

#### Granularity Control
- **Per-Note Emission**: Disabled by default due to high volume. Enable only for specific games requiring detailed MIDI analysis or aural assessment debugging.
- **Per-Prompt Emission**: Standard granularity for most games. Captures each question/prompt interaction with correctness and timing.
- **Per-Interaction Emission**: Captures all user interactions including navigation, help requests, and UI elements. Used for UX analysis and debugging.
- **Stage Boundary Events**: Always emitted regardless of granularity settings (start, complete, quit, error).

#### Sampling Strategy
- **Production Environment**: 
  - Critical events (start, complete, quit, error): Never sampled (100%)
  - Performance metrics: 10% sampling
  - Interaction events: Configurable per game (default 10%)
  - Asset loading: 10% sampling
- **Development Environment**: All events captured at 100%
- **QA Environment**: All events captured at 100% with extended debug information

#### Batching and Buffering
- **Real-time Events**: Stage start, stage complete, target met, errors sent immediately
- **Buffered Events**: Performance metrics and interactions batched every 30 seconds or 50 events
- **Offline Queue**: Events queued locally during offline play, synced when connection restored (max 500 events)
- **Buffer Overflow**: Oldest non-critical events dropped when buffer exceeds limits

### Device-Specific Telemetry

#### MIDI Device Telemetry
Based on the original system's MIDI implementation challenges and known issues with MIDI latency and device compatibility:

```javascript
// MIDI device connection
{
  event_type: "midi_device_connected",
  properties: {
    device_name: "Yamaha USB-MIDI",
    device_manufacturer: "Yamaha",
    device_inputs: 1,
    device_outputs: 1,
    connection_latency_ms: 45,
    browser: "chrome",
    browser_version: "120.0.0",
    os: "windows"
  }
}

// MIDI input event (when emit_per_note is enabled)
{
  event_type: "midi_note_input",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    note: 60,
    velocity: 87,
    channel: 0,
    timestamp_ms: 1234567890,
    latency_ms: 8.2,
    expected_note: 60,
    correct: true
  }
}

// MIDI device error
{
  event_type: "midi_device_error",
  properties: {
    error_type: "disconnected|timeout|invalid_input|permission_denied",
    device_name: "Yamaha USB-MIDI",
    error_message: "Device disconnected during gameplay",
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    recovery_attempted: true,
    recovery_successful: false
  }
}
```

#### Microphone Telemetry
For aural games requiring pitch detection and audio input:

```javascript
// Microphone permission and initialization
{
  event_type: "microphone_initialized",
  properties: {
    permission_granted: true,
    device_id: "default",
    sample_rate: 48000,
    latency_ms: 15,
    browser: "chrome",
    os: "windows"
  }
}

// Audio input analysis (when emit_per_prompt is enabled)
{
  event_type: "audio_input_analyzed",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-play",
    detected_frequency_hz: 440.0,
    expected_frequency_hz: 440.0,
    confidence_score: 0.95,
    analysis_time_ms: 125,
    correct: true,
    prompt_index: 8
  }
}
```

### Performance Monitoring

#### Frame Rate and Rendering Performance
Critical for ensuring smooth gameplay experience across devices:

```javascript
{
  event_type: "game_performance_snapshot",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-challenge",
    snapshot_interval_seconds: 30,
    fps_average: 58.4,
    fps_min: 42.0,
    fps_max: 60.0,
    dropped_frames: 3,
    rendering_mode: "canvas|webgl",
    memory_used_mb: 45.2,
    memory_peak_mb: 67.8,
    cpu_usage_percent: 35.2
  }
}
```

#### Asset Loading Performance
Based on original system issues with asset loading failures and timeouts:

```javascript
{
  event_type: "game_bundle_load_complete",
  properties: {
    game_id: "G-01542",
    game_version: 3,
    bundle_size_mb: 12.5,
    total_load_time_ms: 3250,
    preload_time_ms: 1200,
    initialization_time_ms: 850,
    cache_hit_rate: 0.85,
    assets_loaded: 45,
    assets_failed: 0,
    timeout_occurred: false
  }
}
```

### Error Tracking and Debugging

#### Game Runtime Errors
Comprehensive error tracking for debugging game issues:

```javascript
// JavaScript runtime error
{
  event_type: "game_runtime_error",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    error_type: "javascript_exception|network_error|asset_load_failure|device_error",
    error_message: "TypeError: Cannot read property 'score' of undefined",
    error_source: "game_bundle",
    file_name: "game-logic.js",
    line_number: 234,
    column_number: 12,
    stack_trace: "...",
    browser: "chrome",
    browser_version: "120.0.0",
    os: "windows",
    recoverable: false
  }
}

// Network connectivity error
{
  event_type: "game_network_error",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    error_type: "score_submission_failed|asset_load_timeout|cdn_unreachable",
    endpoint: "https://api.mlc.com/v1/scores",
    http_status: 0,
    retry_count: 2,
    queued_for_retry: true
  }
}

// Blank screen detection (addressing legacy system issue)
{
  event_type: "game_blank_screen_detected",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-play",
    detection_time_ms: 10000,
    bundle_loaded: true,
    assets_loaded: true,
    canvas_rendered: false,
    console_errors_present: true,
    error_messages: ["..."]
  }
}
```

### Development and QA Telemetry Modes

#### Debug Mode
When `debug_mode: true` is set, additional diagnostic information is captured:

```javascript
{
  event_type: "game_debug_info",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    debug_data: {
      game_state: "playing|paused|completed",
      current_prompt: 15,
      total_prompts: 20,
      score_accumulator: 87,
      mistakes_this_session: 3,
      time_remaining_seconds: 45,
      internal_variables: {...},
      asset_manifest: {...}
    }
  }
}
```

#### QA Mode
When `qa_mode: true` is set, comprehensive validation telemetry is captured:

```javascript
{
  event_type: "game_qa_validation",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    validation_checks: {
      asset_integrity: "pass",
      scoring_logic: "pass",
      target_threshold_math: "pass",
      stage_transition_logic: "pass",
      input_validation: "pass",
      performance_benchmarks: "warn",
      accessibility_features: "pass"
    },
    warnings: [
      "Frame rate dropped below 30 FPS for 2.5 seconds",
      "Asset load time exceeded 5 second target by 1.2 seconds"
    ]
  }
}
```

## UX requirements (if applicable)

### Telemetry Impact on User Experience
- **Performance Budget**: Telemetry collection must not add more than 50ms overhead to game runtime
- **Network Impact**: Event batching must minimize network requests to avoid gameplay interruption
- **Battery Impact**: On mobile devices, telemetry sampling adjusted to conserve battery
- **Offline Graceful Degradation**: Games remain fully playable when telemetry endpoints unreachable

### Developer Experience
- **Console Logging**: When in development environment, key telemetry events logged to browser console
- **Telemetry Dashboard**: Game authors can view real-time telemetry for games they're testing
- **Event Inspector**: Interactive tool for viewing and filtering game telemetry events during QA

## Telemetry

### Game Lifecycle Events

#### Stage Start with Telemetry Context
Emitted when a stage begins, establishing telemetry context for subsequent events:

```javascript
{
  event_type: "game_stage_start",
  properties: {
    game_id: "G-01542",
    game_version: 3,
    stage_id: "G-01542-quiz",
    stage_version: 3,
    stage_type: "quiz",
    run_id: "run_abc123",
    assignment_id: "asg_789",
    sequence_id: "seq_456",
    device_profile: "midi_required",
    input_mode: "midi",
    telemetry_config: {
      emit_per_note: false,
      emit_per_prompt: true,
      sampling_rate: 0.1
    },
    student_context: {
      prior_attempts_this_stage: 2,
      best_score_this_stage: 75,
      time_since_last_attempt_hours: 48
    }
  }
}
```

#### Stage Complete with Performance Metrics
Comprehensive completion event including runtime performance summary:

```javascript
{
  event_type: "game_stage_complete",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    run_id: "run_abc123",
    score: 87,
    max_score: 100,
    passed: true,
    duration_ms: 125000,
    attempts: 2,
    mistake_count: 3,
    performance_summary: {
      fps_average: 58.4,
      fps_min: 42.0,
      memory_peak_mb: 67.8,
      asset_load_time_ms: 3250,
      total_interactions: 20,
      hints_used: 0,
      pauses: 1,
      errors_encountered: 0
    }
  }
}
```

#### Stage Quit with Context
Captures why and when a student exits a stage:

```javascript
{
  event_type: "game_stage_quit",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    run_id: "run_abc123",
    elapsed_ms: 45000,
    progress_percentage: 35,
    score_at_quit: 65,
    reason: "user_exit|timeout|error|browser_close",
    completed_prompts: 7,
    total_prompts: 20,
    performance_at_quit: {
      fps_average: 57.2,
      errors_encountered: 0
    }
  }
}
```

### Score Submission Telemetry

#### Score Submit Attempt
Tracks the submission of scores to the backend, critical for identifying score recording failures (a major issue in the original system):

```javascript
{
  event_type: "score_submit_attempt",
  properties: {
    game_id: "G-01542",
    stage_id: "G-01542-quiz",
    run_id: "run_abc123",
    score: 87,
    max_score: 100,
    passed: true,
    duration_ms: 125000,
    attempts: 2,
    submission_timestamp: "2025-01-27T14:30:45.123Z",
    offline_queued: false,
    retry_count: 0
  }
}

{
  event_type: "score_submit_result",
  properties: {
    run_id: "run_abc123",
    accepted: true,
    latency_ms: 150,
    http_status: 201,
    score_id: "scr_xyz789",
    retry_count: 0,
    offline_queued: false
  }
}

{
  event_type: "score_submit_failed",
  properties: {
    run_id: "run_abc123",
    error_type: "network_error|validation_error|timeout|server_error",
    error_message: "Connection timeout after 30 seconds",
    http_status: 0,
    retry_count: 2,
    will_retry: true,
    queued_for_offline: true
  }
}
```

### Asset Loading Telemetry

#### Bundle Loading Events
Tracks the complete game bundle loading process:

```javascript
{
  event_type: "game_bundle_load_start",
  properties: {
    game_id: "G-01542",
    game_version: 3,
    bundle_url: "https://cdn.mlc/games/G-01542/v3/index.html",
    expected_size_mb: 12.5,
    cache_available: true
  }
}

{
  event_type: "game_asset_loaded",
  properties: {
    game_id: "G-01542",
    asset_type: "audio_sprite",
    asset_url: "https://cdn.mlc/games/G-01542/v3/audio-sprite.mp3",
    asset_size_bytes: 2048576,
    load_time_ms: 850,
    cache_hit: true,
    cdn_region: "us-east-1"
  }
}

{
  event_type: "game_bundle_load_complete",
  properties: {
    game_id: "G-01542",
    total_load_time_ms: 3250,
    assets_loaded: 45,
    assets_failed: 0,
    bundle_initialized: true,
    ready_to_play: true
  }
}
```

### Sampling Configuration
- **Critical Path Events**: Never sampled (stage start, complete, quit, score submission, errors)
- **Performance Metrics**: 10% sampling in production, 100% in development
- **Asset Loading**: 10% sampling in production
- **Interaction Events**: Configurable per game (default 10%)
- **Debug Events**: Only in development and QA environments

## Permissions

### Telemetry Data Access

#### Game Authors and Content Editors
- **Read Access**: Telemetry for games they have created or are assigned to maintain
- **Configuration Access**: Can modify telemetry flags (emit_per_note, emit_per_prompt, sampling_rate) for their games
- **Debug Mode Access**: Can enable debug and QA modes for testing
- **Export Access**: Can export telemetry data for their games

#### QA Team
- **Read Access**: All game telemetry across all games
- **QA Mode Access**: Can enable QA validation telemetry for comprehensive testing
- **Performance Monitoring**: Access to performance dashboards and alerts
- **Error Tracking**: Full access to error logs and debugging information

#### System Administrators
- **Global Access**: All game telemetry across entire platform
- **Configuration Management**: Can modify platform-wide sampling rates and telemetry policies
- **Performance Monitoring**: System-wide performance dashboards and alerts
- **Debug Access**: Can enable enhanced logging for troubleshooting production issues

#### Teachers and Students
- **No Direct Access**: Cannot view raw telemetry data
- **Aggregated Insights**: See processed analytics and KPIs derived from telemetry (see [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md))

## Dependencies

### Core System Dependencies
- **Event Model**: Platform-wide event tracking foundation ([Event Model](../15-analytics-and-reporting/event-model.md))
- **Game Model**: Game structure, stages, and metadata definitions ([Game Model](game-model.md))
- **Games Registry**: Game management and configuration system ([Games Registry Overview](games-registry-overview.md))
- **Browser Compatibility Matrix**: Browser-specific telemetry requirements ([Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md))

### Technical Dependencies
- **Time-Series Database**: Storage for high-volume telemetry data with time-based queries
- **Message Queue**: Reliable event batching and delivery system
- **CDN Integration**: Asset loading performance tracking and CDN health monitoring
- **Error Tracking Service**: Centralized error aggregation and alerting (e.g., Sentry, Rollbar)
- **Performance Monitoring**: Application performance monitoring tools (e.g., New Relic, DataDog)

### Integration Dependencies
- **Game Runtime Libraries**: JavaScript/TypeScript instrumentation libraries embedded in game bundles
- **MIDI Web API**: Browser MIDI device interaction and telemetry
- **Web Audio API**: Microphone input and audio analysis telemetry
- **Local Storage API**: Offline event queue persistence
- **Console Logging API**: Development and debug mode logging

### Related Documentation
- [Event Model](../15-analytics-and-reporting/event-model.md) - Platform-wide event definitions
- [Game Model](game-model.md) - Game structure and telemetry configuration
- [Games Audit and Health](games-audit-and-health.md) - Health monitoring and validation
- [Games Registry Overview](games-registry-overview.md) - Administrative telemetry
- [Assignment Play](../03-student-experience/assignment-play.md) - Student gameplay experience
- [Teacher Reports KPIs](../15-analytics-and-reporting/teacher-reports-kpis.md) - Analytics derived from telemetry

## Open questions

### Telemetry Granularity
- **Optimal Sampling Rates**: What are the optimal sampling rates for different event types to balance insight with cost?
- **Per-Note vs Per-Prompt**: When should per-note telemetry be enabled for MIDI games? What are the storage and processing implications?
- **Custom Telemetry**: Should game authors be able to define custom telemetry events specific to their game mechanics?

### Performance and Scale
- **Event Volume**: What is the expected telemetry event volume at peak usage across all games?
- **Storage Costs**: What is the cost model for telemetry data retention (90 days vs 1 year vs 7 years)?
- **Query Performance**: What query patterns are most common and how do we optimize for them?
- **Real-time Processing**: Which telemetry events need real-time processing vs batch processing?

### Privacy and Compliance
- **Student Privacy**: What telemetry data constitutes PII and requires special handling?
- **Parental Consent**: Do certain telemetry events require explicit parental consent for students under 13?
- **Data Portability**: What telemetry data should be included in student data exports?
- **Cross-Organization Analytics**: Can we aggregate anonymous telemetry across organizations for product insights?

### Development and Debugging
- **Source Maps**: How do we handle source maps for minified game bundles in production error tracking?
- **Remote Debugging**: Should we support remote debugging capabilities for production game issues?
- **Telemetry Testing**: How do QA teams validate that telemetry is correctly implemented in new games?
- **Telemetry Documentation**: What documentation and examples do game authors need to properly instrument their games?

### Device-Specific Challenges
- **MIDI Latency**: What telemetry is needed to diagnose and resolve MIDI latency issues across different devices and browsers?
- **Microphone Quality**: How do we track audio input quality and provide feedback to users about their microphone setup?
- **Touch Input**: What touch-specific telemetry is needed for mobile and tablet gameplay?
- **Offline Mode**: How do we handle telemetry synchronization when students play games offline and then reconnect?

### Error Recovery
- **Automatic Recovery**: Which errors should trigger automatic recovery attempts vs surfacing to the user?
- **Error Notifications**: Should game authors be notified in real-time when their games experience errors?
- **Error Thresholds**: At what error rate should a game be automatically flagged for review or disabled?
- **Blank Screen Detection**: What is the optimal timeout for detecting blank screen issues without false positives?
