# MIDI Support

## Purpose

This specification defines the technical integration of MIDI (Musical Instrument Digital Interface) hardware support into the MLC-LMS platform. MIDI keyboard integration is a core pedagogical feature that enables students to use real piano keyboards as input devices for music learning games, providing authentic kinesthetic learning experiences and preparing students for real-world piano performance.

**Why this integration exists:**
- Enables authentic music learning through physical piano keyboard interaction
- Bridges the gap between on-screen learning and real instrument practice
- Provides kinesthetic reinforcement for visual and aural music concepts
- Supports pedagogical goals of developing proper finger technique and muscle memory
- Differentiates MLC from software-only competitors by leveraging real hardware

**Technical Goals:**
- Seamless detection and connection of MIDI devices across all supported platforms
- Low-latency MIDI input processing (<50ms) for responsive gameplay
- Automatic device management with minimal user configuration
- Graceful fallback strategies when MIDI hardware is unavailable
- Platform-specific optimizations for web browsers and mobile devices

## Scope

### Included

**Web MIDI API Integration (Primary Implementation)**
- Web MIDI API initialization and lifecycle management
- WebMIDI.js library integration for cross-browser abstraction
- MIDI device detection, enumeration, and hot-plug support
- MIDI input event processing (note on, note off, velocity)
- Real-time event routing to active game instances
- Connection state monitoring and recovery
- Latency measurement and optimization
- Browser permission handling and user consent

**Platform-Specific Implementations**
- **Desktop (PC, Mac, Chrome OS)**: Web MIDI API via Chrome/Chromium browsers
- **Android**: Web MIDI API via Chrome browser with USB and Bluetooth MIDI
- **iOS**: Core MIDI framework via custom MusicLearningCommunity.com app (see iOS Custom App section)

**Device Support**
- USB MIDI class-compliant keyboards (plug-and-play)
- Bluetooth LE MIDI devices across supported platforms
- Multi-device handling (automatic selection of most recently active device)
- Device profile persistence and reconnection preferences

**Technical Infrastructure**
- MIDI device registry and session management
- Performance monitoring and diagnostics
- Error handling and recovery mechanisms
- Browser compatibility detection and polyfills
- MIDI API feature detection and graceful degradation

### Excluded

**Out of Scope for This Integration**
- **User Experience and Accessibility**: See [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md) for UX flows, visual indicators, connection screens, and accessibility requirements
- **Game-Specific MIDI Logic**: Individual game implementations handle MIDI note interpretation according to their pedagogical goals
- **MIDI Output**: Sending MIDI signals to external synthesizers or devices (listen-only mode)
- **Advanced MIDI Messages**: Program changes, control changes, pitch bend, aftertouch, SysEx (note on/off only)
- **MIDI Recording and Playback**: Performance capture and replay (future content authoring feature)
- **MIDI Sequencing**: Multi-track MIDI composition or arrangement
- **Custom MIDI Mapping**: User-configurable key mappings beyond standard note numbers
- **Non-Standard MIDI Devices**: Drum pads, wind controllers, guitar-to-MIDI converters (piano keyboards only)

## Data model (if applicable)

**Note**: Data models for MIDI device state, session context, and pairing records are defined in [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md). This section focuses on technical implementation models.

### MIDI API Configuration

```typescript
interface MIDIAPIConfig {
  enabled: boolean; // Platform-wide MIDI feature flag
  sysex: false; // Never enable system exclusive messages
  software: false; // Ignore software synthesizers, hardware only
  preferredProtocol: 'midi1'; // MIDI 1.0 protocol
  initTimeoutMs: 5000; // Timeout for MIDI API initialization
  reconnectAttemptsMax: 3; // Max automatic reconnection attempts
  reconnectDelayMs: 2000; // Delay between reconnection attempts
  latencyWarningThresholdMs: 100; // Show warning above this latency
  latencyCriticalThresholdMs: 200; // Block gameplay above this latency
}
```

### MIDI Input Event

```typescript
interface MIDIInputEvent {
  eventId: string; // Unique event identifier
  timestamp: DOMHighResTimeStamp; // High-resolution timestamp from browser
  deviceId: string; // Web MIDI API device identifier
  messageType: 'noteon' | 'noteoff';
  channel: number; // MIDI channel (1-16, typically channel 1)
  noteNumber: number; // MIDI note number (0-127, typically 21-108 for piano)
  velocity: number; // Note velocity (0-127, 0 = note off)
  processingLatencyMs?: number; // Measured latency from hardware to app
}
```

### MIDI Device Capability

```typescript
interface MIDIDeviceCapability {
  deviceId: string;
  supportsNoteOn: boolean; // Always true for keyboards
  supportsNoteOff: boolean; // Always true for keyboards
  supportsVelocity: boolean; // Detected on first note event
  noteRange: { min: number; max: number }; // Detected range (typically 21-108)
  averageLatencyMs: number; // Running average of input latency
  inputChannels: number[]; // Active MIDI channels (typically [1])
  connectionType: 'usb' | 'bluetooth' | 'unknown';
  manufacturerId?: string; // MIDI manufacturer ID if available
}
```

### MIDI Connection State

```typescript
interface MIDIConnectionState {
  apiAvailable: boolean; // Web MIDI API available in browser
  apiInitialized: boolean; // API successfully initialized
  permissionGranted: boolean; // User granted MIDI permissions
  permissionState: 'granted' | 'denied' | 'prompt' | 'unsupported';
  activeDeviceId?: string; // Currently active MIDI device
  availableDevices: string[]; // IDs of all connected devices
  lastStateChange: Date;
  errorState?: {
    code: string;
    message: string;
    recoverable: boolean;
  };
}
```

### Relationships

- **Game Session** → **MIDIConnectionState** (1:1, tracks MIDI availability during gameplay)
- **MIDIInputEvent** → **Game Stage** (many:1, events routed to active stage)
- **MIDIDeviceCapability** → **MIDIPairingRecord** (1:1, technical capabilities of paired device)

## Behavior and rules

### Web MIDI API Initialization

**Startup Sequence**

1. **Feature Detection Phase** (on app load)
   ```typescript
   if (navigator.requestMIDIAccess) {
     // Web MIDI API available, proceed with initialization
   } else {
     // API not available, log platform/browser info for diagnostics
     // Set midiSupported = false in session context
   }
   ```

2. **Permission Request Phase**
   - Request MIDI access: `navigator.requestMIDIAccess({ sysex: false, software: false })`
   - Browser displays native permission prompt (one-time per origin)
   - If granted: Proceed to device enumeration
   - If denied: Log denial, set permissionState = 'denied', show user-friendly message
   - If error: Capture error code and message for diagnostics

3. **Device Enumeration Phase**
   - Iterate through `midiAccess.inputs` to find all MIDI input devices
   - Filter out software synthesizers (check device name patterns)
   - Store device list with IDs, names, manufacturers, connection states
   - Attach event listeners to each input device: `device.onmidimessage = handleMIDIMessage`
   - Register global state change listener: `midiAccess.onstatechange = handleStateChange`

4. **Hot-Plug Monitoring Phase**
   - Listen for `statechange` events on MIDIAccess object
   - Detect device connections (`state === 'connected'`)
   - Detect device disconnections (`state === 'disconnected'`)
   - Update available device list dynamically
   - Notify UI components to update MIDI status indicators

**Browser Compatibility Detection**

Platform-specific initialization logic:

```typescript
const platformMIDISupport = {
  // Full Web MIDI API support
  desktop_chrome: { api: 'webmidi', version: 'chrome >= 43' },
  desktop_edge: { api: 'webmidi', version: 'edge >= 79' },
  chromeos: { api: 'webmidi', version: 'chrome >= 43' },
  android_chrome: { api: 'webmidi', version: 'chrome >= 43' },
  
  // No Web MIDI API support
  ios_safari: { api: 'none', fallback: 'custom_app' },
  desktop_firefox: { api: 'none', fallback: 'keyboard_mouse' },
  desktop_safari: { api: 'none', fallback: 'keyboard_mouse' },
};
```

**Permission Handling**

- **First Visit**: Browser shows permission prompt automatically on first `requestMIDIAccess()`
- **Permission Granted**: Store `permissionState = 'granted'` in session, proceed with MIDI
- **Permission Denied**: Show in-app message: "MIDI access was blocked. To enable, click the lock icon in your browser's address bar and allow MIDI access."
- **Permission Revoked**: Detect via failed `requestMIDIAccess()`, prompt user to re-grant
- **No Prompt Option**: If browser doesn't support prompts (rare), assume granted and proceed

### MIDI Event Processing

**Event Flow**

1. **Hardware Input**: Student presses/releases key on MIDI keyboard
2. **Browser Reception**: Web MIDI API receives MIDI message with timestamp
3. **Event Parsing**: Extract message type, channel, note number, velocity
4. **Event Filtering**: 
   - Accept: Note On (0x90-0x9F), Note Off (0x80-0x8F)
   - Ignore: All other MIDI message types (control change, program change, etc.)
5. **Latency Measurement**: Calculate `processingLatencyMs = Date.now() - event.timestamp`
6. **Event Routing**: Send parsed event to active game stage's MIDI handler
7. **Telemetry**: Sample and log event for analytics (see Telemetry section)

**Message Parsing**

```typescript
function parseMIDIMessage(event: WebMidi.MIDIMessageEvent): MIDIInputEvent | null {
  const [status, note, velocity] = event.data;
  const messageType = status & 0xF0; // High nibble
  const channel = (status & 0x0F) + 1; // Low nibble, 1-indexed
  
  if (messageType === 0x90 && velocity > 0) {
    // Note On
    return {
      eventId: generateEventId(),
      timestamp: event.timeStamp,
      deviceId: event.target.id,
      messageType: 'noteon',
      channel,
      noteNumber: note,
      velocity,
      processingLatencyMs: Date.now() - event.timeStamp,
    };
  } else if (messageType === 0x80 || (messageType === 0x90 && velocity === 0)) {
    // Note Off (explicit or velocity 0)
    return {
      eventId: generateEventId(),
      timestamp: event.timeStamp,
      deviceId: event.target.id,
      messageType: 'noteoff',
      channel,
      noteNumber: note,
      velocity: 0,
      processingLatencyMs: Date.now() - event.timeStamp,
    };
  }
  
  return null; // Ignore all other message types
}
```

**Multi-Device Handling**

- If multiple MIDI devices are connected simultaneously:
  1. Accept input from **all connected devices** (don't filter by device ID)
  2. Route all events to the active game stage
  3. Track most recently active device for preference persistence
  4. Display all connected device names in MIDI status UI
- Rationale: Allows teacher and student keyboards in same room, or switching between devices mid-session

**Latency Monitoring**

- Measure latency for every MIDI event: `latency = currentTime - eventTimestamp`
- Maintain rolling average of last 20 events per device
- Display latency in MIDI test interface (see UX spec)
- Show warning if `averageLatencyMs > 100ms`
- Block game launch if `averageLatencyMs > 200ms` (configurable by system admin)
- Log high-latency events to telemetry for support diagnostics

### Connection State Management

**State Transitions**

```
[Not Available] → [Available] → [Initializing] → [Ready] → [Active]
       ↓              ↓             ↓               ↓          ↓
       ↓              ↓             ↓               ↓      [Disconnected]
       ↓              ↓             ↓               ↓          ↓
       └──────────────┴─────────────┴───────────────┴──────[Error]
                                    ↓
                              [Reconnecting]
```

**State Definitions:**
- **Not Available**: Web MIDI API not supported in current browser/platform
- **Available**: API exists but not yet initialized
- **Initializing**: Requesting permissions and enumerating devices
- **Ready**: MIDI system initialized, no devices connected
- **Active**: At least one MIDI device connected and ready for input
- **Disconnected**: Previously active device has disconnected
- **Reconnecting**: Attempting to re-establish connection to known device
- **Error**: Unrecoverable error state (permission denied, API failure)

**State Change Rules:**
- Transition to **Initializing** when user navigates to MIDI-enabled game
- Transition to **Ready** when initialization succeeds but no devices detected
- Transition to **Active** when first MIDI device connects
- Transition to **Disconnected** when active device disconnects during gameplay
- Transition to **Reconnecting** automatically after 2-second delay
- Transition to **Error** on permission denial or critical API failure
- Remain in **Not Available** permanently if browser doesn't support Web MIDI API

**Automatic Reconnection Logic**

When device disconnects during active gameplay:

1. **Immediate Response** (0ms)
   - Pause active game automatically
   - Set connectionState = 'Disconnected'
   - Display disconnection overlay (see UX spec)
   - Start 30-second countdown timer

2. **Reconnection Attempts** (every 2 seconds)
   - Poll `midiAccess.inputs` for matching device ID
   - Check device `state === 'connected'`
   - Attempt to re-attach event listener
   - Increment attempt counter (max 3 attempts)

3. **Successful Reconnection**
   - Set connectionState = 'Active'
   - Resume game from paused state
   - Show success message: "MIDI keyboard reconnected"
   - Reset attempt counter

4. **Failed Reconnection** (after 3 attempts or 30 seconds)
   - Offer fallback input options (keyboard, mouse, touch)
   - Allow student to switch input mode or exit game
   - Preserve game progress and score

### Platform-Specific Implementation

**Desktop (PC, Mac, Chrome OS) - Web MIDI API**

- **Browser Requirement**: Chrome 43+ or Chromium-based browsers (Edge 79+, Opera 30+)
- **Connection Types**: USB MIDI (class-compliant), Bluetooth MIDI
- **Driver Requirements**: Most modern MIDI keyboards are class-compliant (no driver needed)
- **Performance**: Low latency (~10-50ms typical)
- **Known Issues**:
  - Firefox: No Web MIDI API support (show fallback message)
  - Safari: No Web MIDI API support (show fallback message)
  - Older Chrome (<43): Upgrade browser prompt

**Android - Web MIDI API**

- **Browser Requirement**: Chrome 43+ (must be Chrome, not other browsers)
- **Connection Types**: USB MIDI (via USB OTG adapter), Bluetooth MIDI
- **OS Requirement**: Android 6.0+ (Marshmallow) for full MIDI support
- **Performance**: Moderate latency (~30-80ms typical, varies by device)
- **Known Issues**:
  - USB OTG cable required for USB MIDI keyboards
  - Bluetooth latency can be higher than USB
  - Some older Android devices have inconsistent MIDI support
  - Power consumption: MIDI may drain battery faster

**iOS - Custom App (Core MIDI Framework)**

- **Implementation**: Custom MusicLearningCommunity.com app (separate from web platform)
- **Download**: Free from Apple App Store (search "MusicLearningCommunity")
- **Technology**: Core MIDI framework (native iOS MIDI support)
- **Connection Types**: USB MIDI (via Camera Connection Kit), Bluetooth MIDI
- **Performance**: Low latency (~10-40ms typical)
- **Rationale**: Safari and iOS browsers do not support Web MIDI API
- **Integration**: Custom app embeds web views for game content + native MIDI layer
- **Maintenance**: Separate codebase requires iOS-specific updates

**iOS Custom App Architecture**

```
┌─────────────────────────────────────────┐
│  MusicLearningCommunity.com iOS App     │
├─────────────────────────────────────────┤
│  Native Swift/Objective-C Layer         │
│  - Core MIDI Framework                  │
│  - Device enumeration and monitoring    │
│  - MIDI input event capture             │
│  - JavaScriptCore bridge                │
├─────────────────────────────────────────┤
│  WKWebView (Embedded Web Content)       │
│  - MLC-LMS web pages and games          │
│  - JavaScript receives MIDI events      │
│  - Standard game logic (unchanged)      │
└─────────────────────────────────────────┘
```

**Bridge Communication**: Native layer sends MIDI events to JavaScript via `evaluateJavaScript()`:

```swift
// Swift code in native layer
let midiEvent = """
  window.dispatchEvent(new CustomEvent('midiinput', {
    detail: {
      noteNumber: \(noteNumber),
      velocity: \(velocity),
      messageType: '\(messageType)',
      timestamp: \(timestamp)
    }
  }));
"""
webView.evaluateJavaScript(midiEvent)
```

JavaScript in web content listens for these custom events:

```typescript
// JavaScript code in web content
window.addEventListener('midiinput', (event: CustomEvent) => {
  const midiEvent = event.detail;
  routeToActiveGame(midiEvent);
});
```

### Error Handling and Recovery

**Common Error Scenarios**

1. **Web MIDI API Not Available**
   - **Detection**: `navigator.requestMIDIAccess === undefined`
   - **User Message**: "MIDI keyboards are not supported in this browser. Please use Chrome, Edge, or our iOS app."
   - **Fallback**: Offer on-screen keyboard or computer keyboard input
   - **Telemetry**: Log browser and platform for compatibility tracking

2. **Permission Denied**
   - **Detection**: `requestMIDIAccess()` throws `SecurityError`
   - **User Message**: "MIDI access was blocked. Click the lock icon in your browser's address bar and allow MIDI access."
   - **Retry**: Provide "Request Permission Again" button
   - **Telemetry**: Log denial for support follow-up

3. **Device Connection Failed**
   - **Detection**: Device in `midiAccess.inputs` but `state !== 'connected'`
   - **User Message**: "MIDI keyboard detected but not connected. Check USB cable or Bluetooth pairing."
   - **Retry**: Provide "Refresh Connection" button (re-initialize MIDI API)
   - **Telemetry**: Log device ID and error state

4. **High Latency Detected**
   - **Detection**: `averageLatencyMs > 100ms` over last 20 events
   - **User Message**: "MIDI keyboard latency is high (Xms). Try closing other browser tabs or restarting your browser."
   - **Fallback**: Continue gameplay with warning, or block if >200ms
   - **Telemetry**: Log latency measurements and system info

5. **Device Disconnected During Gameplay**
   - **Detection**: `statechange` event with `state === 'disconnected'` while game active
   - **Auto-Response**: Pause game immediately, start reconnection attempts
   - **User Message**: See disconnection overlay in UX spec
   - **Recovery**: Auto-resume if reconnected within 30 seconds
   - **Telemetry**: Log disconnection timing and context

**Browser Refresh Recommendation**

Some MIDI connection issues (especially device not detected after hot-plug) can be resolved by refreshing the browser. Provide clear prompts:

- **Trigger**: Device expected but not appearing in `midiAccess.inputs` after 5 seconds
- **Message**: "MIDI keyboard not detected. Try refreshing your browser."
- **Action Button**: "Refresh and Reconnect" (reload page)
- **Persistence**: Use sessionStorage to preserve game context across refresh
- **Telemetry**: Track refresh success rate for connection issues

### Performance Optimization

**Latency Reduction Strategies**

1. **Direct Event Listeners**: Attach `onmidimessage` directly to device, avoid event bubbling
2. **Minimal Processing**: Parse MIDI messages immediately, defer telemetry logging
3. **Batch Processing**: If multiple notes arrive simultaneously, process as single event batch
4. **Disable Software Synths**: Set `software: false` in MIDI API options to ignore virtual devices
5. **Optimize Game Handlers**: Games should handle MIDI events in <5ms to avoid cumulative latency

**Memory Management**

- Detach event listeners when game unmounts: `device.onmidimessage = null`
- Clear device state when navigating away from MIDI-enabled game
- Avoid storing large event history arrays (keep last 20 events only)
- Dispose of MIDI access object on app unmount

**Thread Safety**

- Web MIDI API events fire on main JavaScript thread (no worker threads)
- Ensure game render loop doesn't block MIDI event processing
- Use `requestAnimationFrame()` for visual updates, handle MIDI events immediately

## UX requirements (if applicable)

**Note**: Detailed UX requirements, including connection screens, status indicators, disconnection overlays, and MIDI test interfaces, are fully specified in [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md). 

This section covers **technical UX integration points** only.

### JavaScript API for UI Components

UI components should use the following API to integrate with MIDI system:

```typescript
// Check if MIDI is available on current platform
const midiAvailable = MIDISystem.isAvailable();

// Get current connection state
const connectionState = MIDISystem.getConnectionState();

// Subscribe to state changes
MIDISystem.onStateChange((newState: MIDIConnectionState) => {
  updateMIDIStatusIndicator(newState);
});

// Request MIDI initialization (call when game requires MIDI)
await MIDISystem.initialize();

// Get list of connected devices
const devices = MIDISystem.getConnectedDevices();

// Register MIDI event handler for active game
MIDISystem.registerEventHandler((event: MIDIInputEvent) => {
  handleGameMIDIInput(event);
});

// Unregister handler when game unmounts
MIDISystem.unregisterEventHandler();

// Measure current latency
const latency = MIDISystem.getAverageLatency();
```

### Visual Status Indicators (Technical Requirements)

- Update status indicator within 100ms of state change
- Use CSS animations for pulsing/active states (GPU-accelerated)
- Display latency measurement refreshed every 2 seconds
- Show device name truncated to 20 characters max for UI fit
- Handle multiple devices: show count "(2 devices)" if >1 connected

### Accessibility Integration

- Announce MIDI state changes to screen readers via ARIA live regions
- Ensure MIDI status text is readable by assistive technologies
- Provide keyboard-accessible controls for MIDI test interface
- Support high contrast mode for MIDI indicators

## Telemetry

**Note**: Detailed telemetry events are specified in [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md). This section covers **additional technical telemetry** for integration diagnostics.

### Technical Diagnostic Events

**API Initialization**
- `midi_api_initialization_attempted`
  - Properties: `platform`, `browser`, `browser_version`, `user_agent`, `timestamp`
  - Sampling: 100%

- `midi_api_initialization_succeeded`
  - Properties: `initialization_duration_ms`, `devices_detected_count`, `platform`, `browser`
  - Sampling: 100%

- `midi_api_initialization_failed`
  - Properties: `error_code`, `error_message`, `platform`, `browser`, `permission_state`
  - Sampling: 100%

**Permission Events**
- `midi_permission_requested`
  - Properties: `platform`, `browser`, `is_first_request`, `timestamp`
  - Sampling: 100%

- `midi_permission_granted`
  - Properties: `platform`, `browser`, `response_time_ms`
  - Sampling: 100%

- `midi_permission_denied`
  - Properties: `platform`, `browser`, `is_permanent_denial`, `timestamp`
  - Sampling: 100% (critical for support)

**Device Detection**
- `midi_device_enumeration_completed`
  - Properties: `device_count`, `device_names[]`, `device_ids[]`, `platform`, `browser`, `enumeration_duration_ms`
  - Sampling: 100%

- `midi_device_hot_plug_detected`
  - Properties: `device_id`, `device_name`, `connection_type`, `time_since_page_load_ms`
  - Sampling: 100%

**Performance Metrics**
- `midi_event_processing_latency`
  - Properties: `latency_ms`, `device_id`, `note_number`, `game_id?`, `stage_id?`
  - Sampling: 5% (high volume)

- `midi_event_dropped`
  - Properties: `reason` (handler_not_registered|game_not_active|parsing_error), `device_id`, `raw_message`
  - Sampling: 100% (indicates bugs)

**Error and Recovery**
- `midi_connection_error`
  - Properties: `error_type`, `error_message`, `device_id?`, `state_before_error`, `recovery_attempted`
  - Sampling: 100%

- `midi_browser_refresh_recommended`
  - Properties: `reason`, `device_id?`, `user_action` (refreshed|dismissed)
  - Sampling: 100%

- `midi_automatic_reconnection_attempted`
  - Properties: `attempt_number` (1-3), `device_id`, `time_since_disconnect_ms`
  - Sampling: 100%

### Sampling Strategy

- **Critical Events**: API initialization, permissions, device detection, errors = 100%
- **High-Volume Events**: Individual MIDI note events = 5% sampling
- **Performance Metrics**: Latency measurements = 5-10% sampling
- **Recovery Events**: Reconnection attempts, browser refresh prompts = 100%

## Permissions

**Note**: User-facing permissions (student, teacher, admin access to MIDI features) are defined in [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md).

This section covers **technical system permissions** only.

### Browser Permissions

**MIDI Access Permission** (standard browser permission)
- **Required for**: Web MIDI API functionality
- **Scope**: Origin-based (granted per domain)
- **Persistence**: Permanent until revoked by user
- **User Control**: Revocable via browser settings → Site Permissions → MIDI
- **Fallback if Denied**: Show instruction to re-grant, offer fallback input methods

### System-Admin Technical Controls

- **Enable/Disable Web MIDI API**: Global feature flag to disable MIDI system-wide
- **Latency Thresholds**: Configure warning and blocking thresholds (default 100ms / 200ms)
- **Reconnection Settings**: Configure max attempts and delay intervals
- **Sampling Rates**: Adjust telemetry sampling for performance events
- **Diagnostic Mode**: Enable verbose logging for support troubleshooting

### Content Security Policy (CSP)

MIDI support requires appropriate CSP headers:

```
Content-Security-Policy: 
  midi-src 'self';  /* Allow Web MIDI API */
```

Ensure CSP policies allow MIDI access on production domains.

## Dependencies

### Core System Dependencies

**User Experience Specification**
- **[Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md)**: Complete UX flows, connection screens, status indicators, accessibility requirements, user-facing telemetry events, and permissions model

**Content and Game Architecture**
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: `device_profile` field defines MIDI requirements per game stage (`midi_required`, `midi_optional`, `any`)
- **[Game Player Shell](../03-student-experience/game-player-shell.md)**: Integration point for MIDI event routing to active games, status display
- **[Assignment Play](../03-student-experience/assignment-play.md)**: Pre-game MIDI availability checks before launching MIDI-required games

**Data and Analytics**
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Standard telemetry event structure and properties

### Technical Dependencies

**JavaScript Libraries**

- **WebMIDI.js** (recommended): Cross-browser abstraction library for Web MIDI API
  - Repository: https://github.com/djipco/webmidi
  - Version: 3.x (MIT license)
  - Benefits: Simplifies device management, event parsing, error handling
  - Already included in [Technology Stack](../00-foundations/techstack.md) as recommended library

Alternatively, use **native Web MIDI API** directly (no external dependencies):
  - Available in Chrome/Chromium 43+
  - More control, but requires manual handling of browser differences
  - Recommended for production to reduce bundle size if no other WebMIDI.js features needed

**Browser APIs**

- **Web MIDI API**: 
  - Standard: https://www.w3.org/TR/webmidi/
  - Browser support: Chrome 43+, Edge 79+, Opera 30+ (no Firefox or Safari)
  - Permissions API: Browser native prompts for MIDI access

- **High Resolution Time API**: For accurate latency measurements
  - `performance.now()` or `DOMHighResTimeStamp`

- **Custom Events API**: For iOS app bridge communication
  - `CustomEvent` and `window.dispatchEvent()`

**Platform-Specific Dependencies**

- **iOS Custom App**:
  - Core MIDI framework (native iOS)
  - WKWebView for embedding web content
  - JavaScriptCore for native-to-web communication
  - Swift/Objective-C development environment

- **Android**:
  - Chrome browser 43+ installed on device
  - Android 6.0+ (Marshmallow) for full MIDI support
  - USB OTG support for USB MIDI connections

**Infrastructure Dependencies**

- **Session Storage**: Preserve MIDI context across browser refreshes
- **Feature Flags**: System-admin controls for enabling/disabling MIDI globally
- **Telemetry Pipeline**: Event logging and analytics (see [Event Model](../15-analytics-and-reporting/event-model.md))

### Content Dependencies

**Support Documentation** (to be created)
- **MIDI Setup Guide**: Platform-specific connection instructions (Windows, Mac, Chrome OS, Android, iOS)
- **Compatible MIDI Keyboards List**: Tested and recommended devices with price ranges ($50-$200)
- **MIDI Troubleshooting Guide**: Common connection issues and solutions
- **Teacher MIDI Guide**: How to configure MIDI requirements and fallback options

**Hardware Recommendations**
- Entry-level MIDI keyboards: 25-49 keys, USB powered ($50-$100)
- Mid-range MIDI keyboards: 61-88 keys, velocity-sensitive, USB/Bluetooth ($100-$300)
- Professional MIDI keyboards: 88 weighted keys, full velocity range ($300+)

All recommendations should specify:
- USB and/or Bluetooth connectivity
- Class-compliant (no driver required)
- Compatible with Windows, Mac, Chrome OS, iOS (via Camera Connection Kit), Android (via USB OTG)

## Open questions

### Technical Implementation

**Web MIDI API vs. WebMIDI.js Library**
- Should we use WebMIDI.js library (simpler API, extra dependency) or native Web MIDI API (more control, no dependencies)?
- How much bundle size does WebMIDI.js add (~50KB minified)?
- Does WebMIDI.js provide meaningful error handling improvements over native API?
- **Recommendation**: Start with native API for MVP, consider WebMIDI.js if complexity grows

**iOS Custom App vs. Web Platform Parity**
- Should we invest in bringing Web MIDI API support to iOS Safari (requires browser vendor changes), or continue maintaining custom app?
- How do we ensure feature parity between web platform and iOS app as new features are added?
- What is the long-term maintenance strategy and cost for the iOS app?
- Should we explore PWA + Bluetooth Web MIDI as alternative to custom app (limited support)?

**Latency Thresholds**
- What is the acceptable latency threshold for different game types (rhythm games vs. note identification)?
- Should we implement automatic latency compensation, or require manual calibration per device?
- Should latency thresholds be configurable per-game rather than globally?
- How do we account for Bluetooth latency variability (30-80ms range)?

**Device Compatibility**
- Should we maintain a device compatibility database (whitelist/blacklist) based on user reports?
- How do we handle MIDI devices that send non-standard messages or have unusual note ranges?
- Should we support MIDI devices with fewer than 25 keys (mini keyboards)?
- What is the minimum requirement for velocity sensitivity (some cheap keyboards don't support)?

### Platform-Specific Questions

**Android Platform**
- How do we detect USB OTG support on Android devices?
- Should we provide in-app guidance for purchasing USB OTG adapters?
- How do we handle Android devices with poor MIDI performance (<Android 8.0)?

**iOS Custom App**
- Should the iOS app be a native wrapper around web content, or a hybrid approach with native UI?
- How do we handle app store updates when only web content changes (no native code)?
- Should we implement in-app browser or use WKWebView exclusively?
- What's the impact of iOS app review process on rapid iteration?

### Error Handling

**Permission Denial Recovery**
- Should we implement a "request permission again" flow after initial denial?
- How many times should we prompt for permissions before giving up?
- Should we track permission denial patterns to identify confused users vs. intentional blocks?

**Connection Reliability**
- What is the root cause of "device detected but no input received" errors?
- Should we implement a "MIDI health check" on game launch to preemptively detect issues?
- How do we handle intermittent Bluetooth connections (signal drops)?

### Performance and Optimization

**Event Processing**
- Should we batch MIDI events if multiple notes arrive within the same animation frame?
- How do we prevent MIDI event processing from blocking game render loops?
- Should we implement a MIDI event queue with priority levels?
- What's the maximum sustainable event rate (notes per second) before performance degrades?

**Telemetry Overhead**
- Does telemetry logging (5% sampling) introduce measurable latency?
- Should we use a separate worker thread for telemetry to avoid blocking game logic?
- How do we balance diagnostic data collection with performance impact?

### Future Enhancements

**Advanced MIDI Features** (explicitly out of scope now, but worth documenting)
- Should we support velocity-sensitive games that use velocity data for scoring?
- Should we support pedal input (sustain pedal = MIDI CC 64)?
- Should we support multi-touch (polyphonic) games that require simultaneous notes?
- Should we implement MIDI recording for teacher assessment and review?

**Multi-User Scenarios**
- How do we handle multiple students with MIDI keyboards in the same classroom?
- Should we implement "MIDI device claiming" to prevent cross-talk between students?
- Should we support teacher-student duet modes (two MIDI keyboards simultaneously)?

### Support and Training

**Diagnostics and Support**
- Should we implement remote diagnostics to help support staff troubleshoot MIDI issues?
- Should we auto-capture device logs and system info when MIDI errors occur?
- How do we provide teachers with enough information to troubleshoot student MIDI issues?
- Should we implement a "MIDI report card" showing connection quality metrics per student?

**Teacher Training**
- How do we train teachers on the pedagogical benefits of MIDI hardware vs. fallback options?
- Should we provide in-platform tutorials for setting up MIDI requirements per game?
- What level of technical MIDI support should teachers be expected to provide vs. referring to documentation?

### Cross-Cutting Concerns

**Data Privacy**
- What privacy considerations exist for tracking MIDI device names and serial numbers?
- Should we anonymize device IDs in telemetry data?
- Should we allow students to opt out of MIDI device tracking?

**Accessibility**
- How do we ensure MIDI integration doesn't exclude students without access to MIDI hardware?
- Should we provide loaner MIDI keyboards for schools that can't afford them?
- How do we balance pedagogical benefits of MIDI with equity concerns?
