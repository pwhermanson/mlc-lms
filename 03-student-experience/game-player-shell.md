# Game Player Shell

## Purpose
Define the technical interface and runtime environment that hosts individual music learning games within the MLC platform. The Game Player Shell provides a standardized, chrome-less container that ensures consistent gameplay experience, score recording, accessibility compliance, and seamless integration with both Assignment Play and All Games Library workflows.

This specification enables the core pedagogical experience by providing a reliable, accessible, and performant environment for students to engage with Learn, Play, Quiz, Challenge, and Review stages of music learning games.

## Scope
**Included**
- Chrome-less full-screen game runtime environment
- Stage loading, execution, and lifecycle management
- Score recording and progress tracking integration
- Device input handling (keyboard, MIDI, microphone, touch)
- Accessibility compliance and assistive technology support
- Performance optimization and asset management
- Error handling and recovery mechanisms
- Telemetry and analytics integration
- Integration with Assignment Play and All Games workflows

**Excluded**
- Individual game mechanics and content (handled by game runtime)
- Assignment creation and management (covered by Assignment Play)
- Game registry and authoring tools (covered by Games Registry)
- User authentication and role management (covered by Auth system)
- Billing and subscription logic (covered by Payments system)

## Data model (if applicable)
### Core Entities

**Game Player Runtime State**
```json
{
  "session_id": "session_abc123",
  "game_id": "G-01542",
  "stage_id": "G-01542-quiz",
  "stage_type": "quiz",
  "assignment_id": "assign_456",
  "student_id": "student_789",
  "device_profile": "midi_required",
  "input_mode": "midi",
  "connectivity_state": "online",
  "started_at": "2025-01-20T10:30:00Z",
  "current_score": 0,
  "max_score": 100,
  "attempts": 1,
  "duration_ms": 45000,
  "paused": false,
  "submitted": false
}
```

**Player Context**
```json
{
  "device_profile": "any|midi_required|mic_optional|mic_required",
  "input_mode": "keyboard|midi|mic|touch",
  "connectivity_state": "online|offline_lite|offline",
  "browser_capabilities": {
    "midi_supported": true,
    "mic_supported": true,
    "webgl_supported": true,
    "audio_context_supported": true
  },
  "accessibility_preferences": {
    "reduced_motion": false,
    "high_contrast": false,
    "screen_reader": false,
    "sound_off": false
  }
}
```

**Stage Asset Bundle**
```json
{
  "stage_id": "G-01542-quiz",
  "bundle_url": "https://cdn.mlc/games/G-01542/v3/quiz.html",
  "version": "3.2.1",
  "size_mb": 12.5,
  "dependencies": [
    "https://cdn.mlc/audio/G-01542-sprite.mp3",
    "https://cdn.mlc/fonts/music-symbols.woff2"
  ],
  "requirements": {
    "minimum_resolution": "1024x768",
    "audio_formats": ["mp3", "ogg"],
    "loading_timeout_seconds": 30
  }
}
```

### Relationships
- **Game Player Shell** hosts **Game Stages** from the Game Model
- **Runtime State** tracks progress for **Student** and **Assignment** contexts
- **Asset Bundles** are loaded from the **Games Registry** CDN
- **Score Records** are written to the **Analytics** system upon completion

## Behavior and rules

### Stage Lifecycle Management
1. **Pre-load Phase**
   - Validate device capabilities against stage requirements
   - Check MIDI/microphone permissions if required
   - Pre-load asset bundle and dependencies
   - Initialize input handlers and accessibility features

2. **Launch Phase**
   - Enter chrome-less full-screen mode
   - Hide global navigation and header/footer elements
   - Display minimal overlay controls (pause, exit, help)
   - Initialize game runtime with player context

3. **Execution Phase**
   - Monitor input events and device connectivity
   - Track score, attempts, and duration in real-time
   - Handle pause/resume functionality
   - Manage error states and recovery options

4. **Completion Phase**
   - Capture final score and performance metrics
   - Submit score to backend with retry logic
   - Display result panel with clear next actions
   - Prepare for stage transition or exit

### Chrome-less Mode Requirements
Based on the original system's "All Games" redesign requirements:

- **Full Viewport Usage**: Games occupy entire browser viewport during play
- **Header/Footer Removal**: Global navigation elements are completely hidden
- **Minimal Overlay Controls**: Only essential controls (pause, exit, help) are visible
- **Return Navigation**: Clear path back to previous context (Assignment or All Games)
- **Focus Management**: Proper keyboard navigation and screen reader support

### Input Handling and Device Support
- **Keyboard Input**: Full keyboard accessibility for all interactive elements
- **MIDI Support**: Real-time MIDI input processing with device pairing and fallbacks
- **Microphone Support**: Audio input capture with permission handling and fallbacks
- **Touch Support**: Responsive touch interactions for tablet and mobile devices
- **Input Mode Switching**: Seamless transition between input methods during gameplay

### Error Handling and Recovery
- **Asset Loading Failures**: Graceful degradation with retry options and fallback content
- **Device Connection Issues**: Clear messaging and alternative input methods
- **Network Connectivity**: Offline mode support with score queuing
- **Runtime Errors**: User-friendly error messages with recovery actions
- **Timeout Handling**: Automatic pause and resume for extended inactivity

### Score Recording and Validation
- **Real-time Tracking**: Continuous score monitoring during gameplay
- **Validation Rules**: Score validation against stage targets and thresholds
- **Submission Logic**: Reliable score submission with retry mechanisms
- **Offline Queuing**: Score storage and submission when connectivity is restored
- **Audit Trail**: Complete logging of score changes and submission attempts

## UX requirements (if applicable)

### Layout and Visual Design
- **Full-Screen Canvas**: Games utilize entire viewport without browser chrome
- **Responsive Design**: Adapt to various screen sizes and orientations
- **High Contrast Support**: Compliance with accessibility color contrast requirements
- **Reduced Motion**: Respect user preferences for motion and animation
- **Consistent Branding**: Minimal Terry Treble branding elements during gameplay

### Overlay Controls
- **Pause Button**: Prominent pause/resume control accessible via keyboard
- **Exit Button**: Clear exit option with confirmation for unsaved progress
- **Help Button**: Context-sensitive help and instructions
- **Progress Indicator**: Visual progress tracking for current stage
- **Score Display**: Real-time score display (when appropriate for stage type)

### Accessibility Requirements
- **Keyboard Navigation**: Full keyboard operability for all controls
- **Screen Reader Support**: Proper ARIA labels and announcements
- **Focus Management**: Clear focus indicators and logical tab order
- **High Contrast Mode**: Support for high contrast color schemes
- **Text Scaling**: Respect browser text scaling preferences
- **Audio Descriptions**: Support for audio description tracks when available

### Performance Requirements
- **Loading Time**: Target <2 seconds to interactive on modern devices
- **Frame Rate**: Maintain 60fps for smooth gameplay experience
- **Memory Usage**: Efficient memory management for extended play sessions
- **Battery Optimization**: Minimize battery drain on mobile devices
- **Network Efficiency**: Optimized asset loading and caching strategies

### Mobile and Touch Considerations
- **Touch Targets**: Minimum 44px touch targets for mobile interaction
- **Gesture Support**: Standard touch gestures (pinch, zoom, swipe) where appropriate
- **Orientation Handling**: Smooth rotation between portrait and landscape
- **Performance**: Optimized rendering for mobile GPUs and processors

## Telemetry

### Core Gameplay Events
- `game_stage_launch`
  - Properties: `game_id`, `stage_id`, `assignment_id?`, `device_profile`, `input_mode`, `bundle_version`
- `game_stage_ready`
  - Properties: `load_time_ms`, `asset_cache_hit`, `browser_capabilities`
- `game_stage_pause`
  - Properties: `elapsed_ms`, `reason` (user_action|system|timeout)
- `game_stage_resume`
  - Properties: `pause_duration_ms`, `total_elapsed_ms`
- `game_stage_complete`
  - Properties: `score`, `max_score`, `passed`, `duration_ms`, `attempts`, `mistake_count?`
- `game_stage_quit`
  - Properties: `elapsed_ms`, `reason` (user_exit|completion|error|timeout)
- `game_stage_error`
  - Properties: `error_code`, `error_message`, `context`, `recoverable`

### Input and Device Events
- `midi_device_connected`
  - Properties: `device_name`, `device_id`, `latency_ms`
- `midi_device_disconnected`
  - Properties: `device_name`, `duration_connected_ms`
- `microphone_permission_requested`
  - Properties: `stage_id`, `permission_granted`
- `input_mode_changed`
  - Properties: `from_mode`, `to_mode`, `reason`

### Performance and Quality Events
- `asset_load_start`
  - Properties: `asset_url`, `asset_type`, `priority`
- `asset_load_complete`
  - Properties: `asset_url`, `load_time_ms`, `size_bytes`, `cache_hit`
- `performance_metric`
  - Properties: `metric_name`, `value`, `unit`, `timestamp`
- `battery_level_change`
  - Properties: `level_percent`, `charging`, `device_type`

### Score and Progress Events
- `score_update`
  - Properties: `current_score`, `max_score`, `progress_percent`
- `score_submit_attempt`
  - Properties: `stage_id`, `score`, `attempt_number`
- `score_submit_success`
  - Properties: `stage_id`, `latency_ms`, `server_response`
- `score_submit_failure`
  - Properties: `stage_id`, `error_code`, `retry_attempt`

### Sampling Strategy
- **Critical Events**: Game completion, score submission, and errors are never sampled
- **Performance Events**: Asset loading and performance metrics sampled at 10% in production
- **Input Events**: MIDI and microphone events sampled at 25% in production
- **Debug Events**: Development and testing events sampled at 100% in non-production

## Permissions

### Student Access
- **Read and Execute**: Can launch and play assigned game stages
- **Score Recording**: Can submit scores for completed stages
- **Progress Tracking**: Can view personal progress and completion status
- **Device Access**: Can use available input devices (MIDI, microphone) with proper permissions

### Teacher and Teacher-Admin Access
- **Override Controls**: Can modify stage targets and gating rules for their classes
- **Progress Monitoring**: Can view student progress and scores in real-time
- **Device Management**: Can configure device requirements and fallbacks for their students
- **Debug Access**: Can access diagnostic information when supporting students

### Subscriber-Admin Access
- **Class Management**: Can configure game access and device policies for their organization
- **Reporting Access**: Can view aggregated progress and performance data
- **Billing Integration**: Can access usage metrics for billing and seat management

### System-Admin Access
- **Full Control**: Can access all game player shell functionality and diagnostics
- **Debug Overlays**: Can enable diagnostic overlays and performance monitoring
- **Emergency Controls**: Can pause or terminate game sessions if needed
- **Audit Access**: Can view complete audit logs and telemetry data

## Dependencies

### Core System Dependencies
- **Game Model**: Stage definitions, targets, and lifecycle management
- **Assignment Play**: Integration with assignment workflow and progression logic
- **All Games Library**: Free play mode integration and score reconciliation
- **Games Registry**: Asset management and health monitoring
- **Analytics System**: Score recording and progress tracking

### Technical Dependencies
- **Design System**: UI components, accessibility standards, and responsive breakpoints
- **Authentication System**: User context and session management
- **CDN Infrastructure**: Asset delivery and caching
- **Device APIs**: MIDI, Web Audio, and microphone access
- **Offline Support**: Local storage and sync capabilities

### Integration Points
- **Assignment Play**: See [Assignment Play](assignment-play.md) for student workflow integration
- **All Games Library**: See [All Games Library](all-games-library.md) for free play integration
- **Game Model**: See [Game Model](../08-games-registry-and-authoring/game-model.md) for stage definitions
- **Accessibility Standards**: See [A11y Standards](../01-ux-design-system/a11y-standards.md) for compliance requirements
- **Responsive Design**: See [Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md) for layout specifications

## Open questions

### Technical Implementation
- Should the game player shell support multiple concurrent game instances or enforce single-instance mode?
- What is the optimal strategy for handling large asset bundles on slow connections?
- How should we handle browser compatibility for older devices while maintaining performance?

### User Experience
- Should there be a global pause/resume capability that works across all game types?
- What level of customization should teachers have over the chrome-less interface?
- How should we handle games that require specific browser features not available on the user's device?

### Performance and Scalability
- What are the minimum system requirements for optimal game performance?
- How should we handle memory management for extended play sessions?
- What caching strategy provides the best balance of performance and storage efficiency?

### Accessibility and Inclusion
- Should we provide alternative input methods for students with motor disabilities?
- How can we ensure games remain accessible when played in full-screen mode?
- What level of customization should be available for students with specific learning needs?

### Integration and Workflow
- How should the game player shell handle transitions between different assignment contexts?
- What happens when a student's assignment changes while they're in the middle of a game?
- Should there be a way to bookmark or save progress within a game session?
