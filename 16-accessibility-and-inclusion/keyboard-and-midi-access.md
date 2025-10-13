# Keyboard and MIDI Access

## Purpose

This specification defines two critical input modalities for the MLC-LMS platform:

1. **Keyboard Navigation (Accessibility)**: Ensuring all functionality is accessible via computer keyboard input (arrow keys, Tab, Enter, etc.) for users who cannot or prefer not to use a mouse or touch interface, meeting WCAG 2.1 AA compliance requirements.

2. **MIDI Keyboard Hardware (Musical Input)**: Enabling authentic music learning experiences through external MIDI piano keyboards connected via USB or Bluetooth, allowing students to respond to games using real piano keys rather than on-screen controls.

These distinct input methods work together to create an inclusive, pedagogically effective learning environment. While keyboard navigation supports accessibility for all users, MIDI keyboard integration provides the authentic musical experience that enhances learning outcomes, particularly for kinesthetic learners and students preparing for real-world piano performance.

## Scope

### Included

**Keyboard Navigation (Accessibility)**
- Full keyboard operability across all platform interfaces (Student, Teacher, Admin)
- Standard keyboard shortcuts and navigation patterns (Tab, Shift+Tab, Enter, Space, Arrows)
- Focus management and visible focus indicators
- Skip links and landmark navigation for screen reader users
- Keyboard-accessible modal dialogs, dropdowns, and complex widgets
- Game-specific keyboard controls when MIDI is not required or available
- Configurable keyboard shortcuts for power users

**MIDI Keyboard Hardware (Musical Input)**
- USB MIDI keyboard detection and pairing (PC, Mac, Chrome OS, Android)
- Bluetooth MIDI keyboard support across all platforms
- iOS MIDI support via custom MusicLearningCommunity.com app
- MIDI device status indicators and connection diagnostics
- Automatic MIDI input routing to active game stages
- MIDI latency monitoring and optimization
- Fallback strategies for MIDI-required games when device is unavailable
- Teacher-configurable MIDI fallback options
- Multi-device MIDI input handling (prefer most recently active)

### Excluded

**Out of Scope**
- **MIDI Output**: Sending MIDI signals to external devices or synthesizers (listen-only mode)
- **Advanced MIDI Features**: Program changes, control changes, pitch bend, aftertouch (note on/off only)
- **MIDI Recording**: Capturing MIDI performances for playback (covered in future content authoring specs)
- **Custom Key Mappings**: Remapping keyboard keys beyond standard shortcuts (future enhancement)
- **Gamepad/Controller Input**: Alternative input devices beyond keyboard and MIDI (future consideration)
- **Voice Control**: Voice-based navigation and input (see [Neurodiverse Support](neurodiverse-support.md))
- **Switch Control**: Single-switch accessibility (see [WCAG Compliance](wcag-compliance.md))

## Data model (if applicable)

### Keyboard Navigation State
```typescript
interface KeyboardNavigationPreferences {
  userId: string;
  shortcutsEnabled: boolean;
  skipLinksVisible: 'always' | 'on_focus' | 'never';
  focusIndicatorStyle: 'default' | 'high_contrast' | 'custom';
  customShortcuts: Record<string, string>; // Future enhancement
}
```

### MIDI Device Registration
```typescript
interface MIDIDeviceState {
  deviceId: string; // Browser-assigned MIDI device identifier
  deviceName: string; // Manufacturer and model (e.g., "Yamaha P-125")
  connectionType: 'usb' | 'bluetooth' | 'unknown';
  connectionStatus: 'connected' | 'disconnected' | 'error';
  lastConnectedAt: Date;
  lastDisconnectedAt?: Date;
  latencyMs: number; // Measured input latency
  inputChannel: number; // MIDI channel (typically 1)
  noteRange: { min: number; max: number }; // MIDI note numbers (e.g., 21-108)
  supportsVelocity: boolean;
}
```

### MIDI Session Context
```typescript
interface MIDISessionContext {
  sessionId: string;
  studentId: string;
  gameId: string;
  stageId: string;
  deviceProfile: 'any' | 'midi_required' | 'midi_optional' | 'mic_required' | 'mic_optional';
  activeDeviceId?: string;
  fallbackMode: 'keyboard' | 'mouse' | 'touch' | 'none';
  midiEnabled: boolean;
  inputMode: 'midi' | 'keyboard' | 'mouse' | 'touch';
  lastInputTimestamp: Date;
}
```

### MIDI Device Pairing Record
```typescript
interface MIDIPairingRecord {
  pairingId: string;
  studentId: string;
  deviceId: string;
  deviceName: string;
  platform: 'windows' | 'macos' | 'chromeos' | 'android' | 'ios';
  browser: string;
  pairedAt: Date;
  lastUsedAt: Date;
  totalSessionCount: number;
  averageLatencyMs: number;
  isPreferred: boolean; // User's preferred device if multiple available
}
```

### Relationships
- **Student** → **KeyboardNavigationPreferences** (1:1, optional)
- **Student** → **MIDIPairingRecord** (1:many, student may have multiple MIDI devices)
- **Game Stage** → **Device Profile** (1:1, defines input requirements)
- **Game Session** → **MIDISessionContext** (1:1, tracks active input mode)
- **MIDIDeviceState** → **MIDIPairingRecord** (many:many, device can be used by multiple students)

## Behavior and rules

### Keyboard Navigation Standards

**Focus Management**
- **Visible Focus Indicators**: All interactive elements must have clear, visible focus indicators with minimum 2px outline and 3:1 contrast ratio against background
- **Logical Tab Order**: Tab order follows visual reading order (left-to-right, top-to-bottom in English/Western locales)
- **Focus Trap in Modals**: When modal dialogs open, focus moves to first interactive element and remains within modal until dismissed
- **Skip Links**: "Skip to main content" and "Skip to navigation" links available at top of page, visible on keyboard focus
- **Focus Restoration**: When dialogs close or navigation occurs, focus returns to triggering element or logical next element

**Keyboard Shortcuts**
- **Global Shortcuts** (available platform-wide):
  - `Tab` / `Shift+Tab`: Navigate forward/backward through interactive elements
  - `Enter`: Activate buttons, links, and primary actions
  - `Space`: Toggle checkboxes, activate buttons, scroll page
  - `Escape`: Close modals, cancel operations, exit full-screen
  - `Arrow Keys`: Navigate within menus, lists, and game controls
  - `/` (forward slash): Focus search input (if available on page)
  - `?` (shift + /): Display keyboard shortcuts help (future enhancement)

- **Game Player Shortcuts** (during gameplay):
  - `Escape`: Pause game and show menu
  - `Space`: Pause/resume game (when safe to do so)
  - `P`: Pause/resume (alternative to Space)
  - `H`: Show help overlay
  - `Q`: Quit game (with confirmation if unsaved progress)
  - Number keys or arrow keys for game-specific responses (when MIDI not required)

- **Teacher/Admin Shortcuts** (power user efficiency):
  - `Ctrl/Cmd + K`: Quick search/command palette (future enhancement)
  - `Ctrl/Cmd + S`: Save current form or assignment
  - `Ctrl/Cmd + Enter`: Submit form or send message

**Navigation Patterns**
- **Landmark Regions**: Proper use of HTML5 semantic elements (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`) and ARIA landmarks
- **Heading Hierarchy**: Logical heading structure (h1 → h2 → h3) for screen reader navigation
- **List Navigation**: Arrow keys to navigate lists, Enter to select, Escape to close
- **Menu Navigation**: Arrow keys for menu items, Tab to exit menu
- **Data Tables**: Arrow keys for cell navigation, Ctrl+Home for first cell, Ctrl+End for last cell

### MIDI Keyboard Integration

**Platform-Specific Support**
- **PC, Mac, Chrome OS, Android**: Web MIDI API via Chrome/Chromium browser (version 43+, Chromium 114+ recommended)
  - Automatic detection when MIDI device is connected before page load
  - Hot-plug detection for devices connected after page load (with browser refresh recommendation)
  - Background MIDI monitoring when user is on platform
  
- **iOS (iPhone, iPad)**: Custom MusicLearningCommunity.com app required
  - Free download from Apple App Store
  - Provides both MIDI support and full platform functionality
  - Uses Core MIDI framework for native iOS MIDI support
  - Supports USB MIDI (via Camera Connection Kit) and Bluetooth MIDI

**Connection and Pairing Flow**

1. **Device Detection Phase**
   - Platform initializes Web MIDI API (or Core MIDI on iOS)
   - Scan for available MIDI input devices
   - Display MIDI status indicator in game player shell header
   - Log available devices to browser console for diagnostics

2. **Pre-Game Validation Phase**
   - When student launches a game stage, check `device_profile` requirement:
     - `midi_required`: Must detect connected MIDI device before allowing stage to start
     - `midi_optional`: MIDI preferred but allow alternative input (keyboard/mouse/touch)
     - `any`: No MIDI requirement, standard input methods available
   - If `midi_required` and no device detected:
     - Display MIDI connection prompt with troubleshooting steps
     - Show device compatibility information (USB/Bluetooth)
     - Offer teacher-configured fallback option if defined
     - Provide "Continue Anyway" option if teacher allows override (configurable)

3. **Active Connection Phase**
   - Monitor MIDI input events in real-time during gameplay
   - Route MIDI note on/off events to active game stage
   - Track input latency and display warning if >100ms
   - Detect device disconnection and pause game with reconnection prompt
   - Allow seamless switching between MIDI and alternative input mid-game

4. **Disconnection and Recovery**
   - If MIDI device disconnects during gameplay:
     - Automatically pause game
     - Display reconnection prompt with countdown timer
     - Allow switch to alternative input method
     - Resume game when device reconnected or alternative input selected
   - Preserve game state and score during disconnection events

**MIDI Input Processing**
- **Note Range**: Support MIDI notes 21-108 (standard 88-key piano range)
- **Velocity Sensitivity**: Capture velocity data (0-127) when device supports it
- **Note Duration**: Track note on/off events for duration-sensitive games
- **Multiple Note Support**: Handle polyphonic input (multiple keys pressed simultaneously)
- **Ignore Non-Note Events**: Filter out program changes, pitch bend, control changes (not used in current games)

**MIDI Device Status Indicators**

Visual indicators displayed in game player shell or pre-game screen:
- ✅ **Connected**: Green indicator, device name shown, ready for input
- ⚠️ **Not Detected**: Yellow indicator, shows "No MIDI device found" with help link
- ❌ **Disconnected**: Red indicator, shows "MIDI device disconnected" with reconnection prompt
- 🔄 **Reconnecting**: Blue indicator, attempting to re-establish connection
- ℹ️ **Optional**: Gray indicator, shows "MIDI device optional for this game"

**Fallback Strategies**

For `midi_required` games when MIDI device is unavailable:
1. **Teacher-Configured Fallback**: Teacher sets preferred alternative for specific game/student
   - Options: `keyboard` (computer keyboard keys), `mouse` (on-screen piano), `touch` (mobile touchscreen), `none` (no fallback, must use MIDI)
2. **On-Screen Piano Keyboard**: Visual piano keyboard with mouse/touch interaction
   - Displays standard piano key layout matching required note range
   - Mouse click or touch to trigger notes
   - Computer keyboard keys mapped to piano keys (QWERTY mapping for chromatic scale)
3. **Skip with Note**: Allow student to skip MIDI-required game with recorded note in transcript
   - Indicates "Skipped: MIDI device required" in score history
   - Does not count toward mastery or sequence completion
   - Teacher can see skip reason in student reports

**Browser Refresh Recommendations**

If MIDI connection issues occur during gameplay:
- Display friendly prompt: "Having trouble with your MIDI keyboard? Try refreshing your browser."
- Provide one-click "Refresh and Reconnect" button that reloads page
- Preserve game session context across refresh (via sessionStorage)
- Auto-resume game at same stage after successful reconnection

### Gating Rules

**Game Launch Gating**
- If `device_profile = midi_required`:
  - Check for connected MIDI device before allowing stage to start
  - Block game launch if no device detected (unless teacher-configured fallback allows override)
  - Display MIDI connection instructions and troubleshooting steps
  - Provide link to MIDI setup guide and compatible device list

- If `device_profile = midi_optional`:
  - Prefer MIDI input if device detected
  - Allow game launch with alternative input if MIDI not available
  - Show status message: "Playing without MIDI keyboard" (non-blocking)

- If `device_profile = any`:
  - No MIDI requirement or validation
  - MIDI input still routed if device is connected (seamless enhancement)

**Input Mode Validation**
- Student cannot change input mode once game stage has started scoring
- Input mode locked in at `game_stage_start` event
- Exception: Disconnection events allow fallback input mode switch with teacher permission

## UX requirements (if applicable)

### Keyboard Navigation Interface

**Focus Indicators**
- **Default Style**: 2px solid outline in primary brand color with 3:1 contrast minimum
- **High Contrast Mode**: 3px solid outline in high-contrast color (yellow or cyan on dark backgrounds)
- **Focus Ring Offset**: 2px offset from element boundary for visual clarity
- **Focus within Complex Widgets**: Visible focus indicator on active child elements (e.g., selected menu item, active tab)

**Skip Links**
- **Position**: First focusable element at top of page, before header
- **Visibility**: Hidden by default, visible when focused via keyboard
- **Styling**: High-contrast button with clear label ("Skip to main content")
- **Targets**: Jump to `<main>` element or primary content area

**Keyboard Shortcuts Help**
- **Access Method**: Press `?` (shift + /) to display help overlay (future enhancement)
- **Content**: List of available shortcuts organized by context (Global, Game Player, Teacher, Admin)
- **Visibility**: Accessible via Help menu and Settings
- **Customization**: Allow users to view and print shortcuts reference (future)

### MIDI Keyboard Interface

**Pre-Game Connection Screen**

When launching a MIDI-required or MIDI-optional game:

```
┌─────────────────────────────────────────────┐
│  🎹 MIDI Keyboard Setup                      │
├─────────────────────────────────────────────┤
│                                              │
│  ✅ MIDI Keyboard Connected                  │
│     Yamaha P-125 (USB)                       │
│                                              │
│  [Test Connection]  [Continue to Game]       │
│                                              │
│  Having trouble?                             │
│  → View MIDI setup guide                     │
│  → Check compatible devices                  │
│  → Play without MIDI (if allowed)            │
│                                              │
└─────────────────────────────────────────────┘
```

If no MIDI device detected for `midi_required` game:

```
┌─────────────────────────────────────────────┐
│  ⚠️ MIDI Keyboard Required                   │
├─────────────────────────────────────────────┤
│                                              │
│  This game requires a MIDI keyboard.         │
│  Please connect your keyboard and refresh.   │
│                                              │
│  [Refresh Browser]                           │
│                                              │
│  Don't have a MIDI keyboard?                 │
│  → View compatible keyboards ($50-$200)      │
│  → Use on-screen keyboard (if allowed)       │
│  → Contact your teacher for alternatives     │
│                                              │
└─────────────────────────────────────────────┘
```

**In-Game MIDI Status Indicator**

Displayed in minimal overlay controls during gameplay:

- **Connected**: Small green MIDI icon (🎹) in top-right corner
- **Active Input**: Icon pulses or brightens when MIDI note detected
- **Latency Warning**: Orange indicator if latency >100ms, tooltip shows measured latency
- **Disconnected**: Red X over MIDI icon, game auto-pauses with reconnection prompt

**MIDI Disconnection Overlay**

When MIDI device disconnects during gameplay:

```
┌─────────────────────────────────────────────┐
│  🔌 MIDI Keyboard Disconnected               │
├─────────────────────────────────────────────┤
│                                              │
│  Your MIDI keyboard was disconnected.        │
│  Your progress has been saved.               │
│                                              │
│  [Reconnect and Resume]                      │
│  [Use On-Screen Keyboard]                   │
│  [Exit Game]                                 │
│                                              │
│  Reconnecting in 30 seconds...               │
│                                              │
└─────────────────────────────────────────────┘
```

**MIDI Test Interface**

Accessible from Settings or pre-game screen, allows student to test MIDI connection:

```
┌─────────────────────────────────────────────┐
│  🎹 Test Your MIDI Keyboard                  │
├─────────────────────────────────────────────┤
│                                              │
│  Device: Yamaha P-125 (USB)                  │
│  Status: ✅ Connected                         │
│  Latency: 45ms (Good)                        │
│                                              │
│  Play any key on your keyboard...            │
│                                              │
│  ┌────────────────────────────────────┐     │
│  │  [Piano Keyboard Visualization]    │     │
│  │  [Shows pressed keys in real-time] │     │
│  └────────────────────────────────────┘     │
│                                              │
│  Last Note: C4 (Middle C)                    │
│  Velocity: 87 / 127                          │
│                                              │
│  [Close]  [Troubleshooting Guide]            │
│                                              │
└─────────────────────────────────────────────┘
```

**Mobile and Tablet Considerations**

- **iOS Devices**: Prominent link to download MusicLearningCommunity.com app from App Store
- **Android Devices**: Require Chrome browser for Web MIDI API support
- **Touch Optimization**: Larger touch targets (minimum 44px) for MIDI setup controls
- **Orientation**: MIDI connection screen works in both portrait and landscape

### Accessibility Requirements

**Keyboard Navigation Accessibility**
- All MIDI setup interfaces must be fully keyboard accessible
- MIDI connection status announced to screen readers (ARIA live regions)
- Keyboard focus clearly visible throughout MIDI pairing flow
- Error messages and troubleshooting steps accessible to assistive technologies

**Visual Accessibility**
- MIDI status indicators use color + icon + text (not color alone)
- High contrast support for MIDI connection screens
- Text scaling support (up to 200% zoom) for MIDI setup instructions
- Clear visual hierarchy in MIDI troubleshooting steps

**Cognitive Accessibility**
- Simple, clear language in MIDI setup instructions
- Step-by-step troubleshooting guidance
- Visual progress indicators during MIDI connection process
- Consistent MIDI status indicators across all games

## Telemetry

### Keyboard Navigation Events

**Focus Tracking**
- `keyboard_navigation_shortcut_used`
  - Properties: `shortcut_key`, `context` (game|dashboard|assignment_builder), `user_role`, `timestamp`
  - Sampling: 10% in production

- `skip_link_activated`
  - Properties: `skip_target` (main_content|navigation), `page_url`, `user_role`
  - Sampling: 100% (indicates accessibility usage)

- `keyboard_help_viewed`
  - Properties: `user_role`, `page_context`, `timestamp`
  - Sampling: 100% (feature engagement metric)

### MIDI Device Events

**Connection and Pairing**
- `midi_device_detected`
  - Properties: `device_id`, `device_name`, `connection_type` (usb|bluetooth), `platform`, `browser`, `timestamp`
  - Sampling: 100% (critical for device compatibility tracking)

- `midi_device_connected`
  - Properties: `device_id`, `device_name`, `connection_type`, `pairing_duration_ms`, `is_first_pairing`, `student_id`
  - Sampling: 100%

- `midi_device_disconnected`
  - Properties: `device_id`, `device_name`, `connection_duration_ms`, `during_gameplay`, `game_id?`, `stage_id?`
  - Sampling: 100% (critical for reliability tracking)

- `midi_device_reconnected`
  - Properties: `device_id`, `reconnection_duration_ms`, `game_id?`, `stage_id?`
  - Sampling: 100%

**Game Integration**
- `midi_game_launch_attempt`
  - Properties: `game_id`, `stage_id`, `device_profile`, `midi_available`, `proceed_allowed`, `fallback_mode?`
  - Sampling: 100%

- `midi_game_launch_blocked`
  - Properties: `game_id`, `stage_id`, `device_profile`, `reason` (no_device|no_fallback|permission_denied)
  - Sampling: 100% (critical for user experience issues)

- `midi_input_detected`
  - Properties: `game_id`, `stage_id`, `note_number`, `velocity`, `latency_ms`
  - Sampling: 5% (high volume, sampled for latency monitoring)

- `midi_fallback_activated`
  - Properties: `game_id`, `stage_id`, `fallback_mode` (keyboard|mouse|touch), `reason` (no_device|disconnection|user_choice)
  - Sampling: 100%

**Performance and Quality**
- `midi_latency_measured`
  - Properties: `device_id`, `latency_ms`, `game_id?`, `stage_id?`, `measurement_method`
  - Sampling: 25% (periodic monitoring)

- `midi_latency_warning_shown`
  - Properties: `device_id`, `latency_ms`, `game_id`, `stage_id`, `threshold_exceeded`
  - Sampling: 100% (indicates performance problems)

**Troubleshooting and Support**
- `midi_setup_guide_viewed`
  - Properties: `context` (pre_game|settings|help), `device_detected`, `student_id`
  - Sampling: 100% (support engagement tracking)

- `midi_test_interface_used`
  - Properties: `student_id`, `device_id?`, `test_duration_ms`, `notes_detected_count`
  - Sampling: 100%

- `midi_browser_refresh_prompted`
  - Properties: `game_id?`, `stage_id?`, `reason` (connection_issue|reconnection_failed), `user_action` (refreshed|dismissed)
  - Sampling: 100%

### Sampling Strategy
- **Critical Events**: Device connection/disconnection, game launch blocking, fallback activation = 100% (never sampled)
- **Performance Events**: Latency measurements, input detection = 5-25% sampling
- **Support Events**: Setup guide views, test interface usage = 100% (indicates user needs help)
- **Keyboard Navigation**: Shortcut usage = 10% sampling, skip links = 100%

## Permissions

### Student Access
- **Keyboard Navigation**: Full keyboard access to all student-facing features (dashboard, assignments, games, messages)
- **MIDI Device Pairing**: Can connect and test MIDI devices in their own session
- **MIDI Preferences**: Can set preferred MIDI device if multiple devices available
- **Fallback Options**: Can use teacher-configured fallback input methods for MIDI games
- **Test Interface**: Can access MIDI test interface to verify connection

### Teacher and Teacher-Admin Access
- **Student Keyboard Preferences**: Can view but not modify student keyboard navigation preferences
- **MIDI Fallback Configuration**: Can configure fallback options for MIDI-required games on per-class or per-student basis
  - Options: Allow on-screen keyboard, allow computer keyboard mapping, allow skip, require MIDI only
- **MIDI Device Reports**: Can view which students have MIDI devices connected and usage statistics
- **Game Device Requirements**: Can view device_profile for any game before assigning
- **Override MIDI Requirements**: Can temporarily exempt specific students from MIDI-required games (with documented reason)

### Subscriber-Admin Access
- **Organization Keyboard Standards**: Can set default keyboard navigation preferences for all users in organization
- **MIDI Policy Configuration**: Can set organization-wide policies for MIDI usage
  - Require MIDI devices for all students (hardware requirement)
  - Allow all fallback options (flexible approach)
  - Custom configuration per grade level or program
- **MIDI Device Inventory**: Can view aggregate MIDI device usage and compatibility across organization
- **Purchase Recommendations**: Can access reports on MIDI device compatibility and student needs for purchasing decisions

### System-Admin Access
- **Platform Keyboard Shortcuts**: Can define and modify global keyboard shortcuts
- **MIDI Device Compatibility**: Can view all detected MIDI devices, browsers, platforms for compatibility analysis
- **MIDI API Diagnostics**: Can access Web MIDI API logs and diagnostic data
- **Fallback Policy Management**: Can create and modify fallback policies available to teachers and admins
- **Emergency MIDI Override**: Can disable MIDI requirements system-wide if critical compatibility issues arise

## Supporting Documents Referenced

This keyboard and MIDI access specification draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — MIDI game requirements and keyboard interaction
- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser compatibility for keyboard and MIDI access
- [All Games Redesign.docx.txt](../00-ORIG-CONTEXT/All%20Games%20Redesign.docx.txt) — Keyboard navigation for game selection

## Dependencies

### Core System Dependencies

**Keyboard Navigation**
- **[WCAG Compliance](wcag-compliance.md)**: Keyboard navigation standards and testing requirements
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)**: Focus management, ARIA implementation, skip links
- **[Neurodiverse Support](neurodiverse-support.md)**: Simplified navigation patterns for cognitive accessibility
- **[Design System Components](../01-ux-design-system/components-overview.md)**: Keyboard-accessible UI components
- **[Game Player Shell](../03-student-experience/game-player-shell.md)**: Keyboard controls during gameplay

**MIDI Keyboard Integration**
- **[MIDI Support Integration](../17-integrations/midi-support.md)**: Technical implementation of Web MIDI API and Core MIDI (iOS)
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Device profile requirements (`midi_required`, `midi_optional`, etc.)
- **[Game Player Shell](../03-student-experience/game-player-shell.md)**: MIDI input routing and device status display
- **[Assignment Play](../03-student-experience/assignment-play.md)**: MIDI availability checking before game launch
- **[All Games Library](../03-student-experience/all-games-library.md)**: MIDI device indicators in game catalog

### Technical Dependencies

**Browser and Platform**
- **Web MIDI API**: Chrome 43+, Chromium 114+ recommended (PC, Mac, Chrome OS, Android)
- **Core MIDI Framework**: iOS via custom MusicLearningCommunity.com app
- **Permissions API**: Browser microphone and MIDI device permissions
- **Session Storage**: Preserving MIDI context across browser refreshes

**Hardware and Connectivity**
- **USB MIDI Devices**: Standard USB MIDI class-compliant keyboards
- **Bluetooth MIDI**: Bluetooth LE MIDI devices (supported on modern platforms)
- **iOS Camera Connection Kit**: USB MIDI connection on iOS devices (Lightning or USB-C adapter)

### Content Dependencies

**Documentation and Support**
- **MIDI Setup Guide**: Step-by-step instructions for connecting MIDI keyboards on each platform
- **Compatible Devices List**: Tested and recommended MIDI keyboards with price ranges
- **Troubleshooting Knowledge Base**: Common MIDI connection issues and solutions
- **Keyboard Shortcuts Reference**: Printable guide of all platform keyboard shortcuts

## Open questions

### Keyboard Navigation

**Technical Implementation**
- Should we implement a command palette (Cmd/Ctrl+K) for power users to quickly access any function via keyboard?
- How do we handle keyboard shortcuts that conflict with browser defaults or assistive technology shortcuts?
- Should we allow user customization of keyboard shortcuts, and if so, how do we prevent conflicts?

**User Experience**
- What is the optimal balance between providing comprehensive keyboard shortcuts vs. overwhelming users with too many shortcuts to remember?
- Should we provide visual hints for available keyboard shortcuts (e.g., showing shortcut keys in tooltips or menus)?
- How do we educate users about available keyboard shortcuts without being intrusive?

### MIDI Keyboard Integration

**Technical Implementation**
- What is the acceptable latency threshold for MIDI input in musical gameplay? Current draft: warn at >100ms, but is this appropriate for all game types?
- Should we implement automatic latency compensation, or require manual calibration?
- How do we handle MIDI devices that send non-standard messages or have unusual note ranges?

**Device Compatibility**
- What is the minimum hardware specification for recommended MIDI keyboards (e.g., must support velocity, minimum key count)?
- Should we maintain a blacklist of incompatible devices based on user reports?
- How do we handle MIDI devices that work intermittently or have platform-specific issues?

**Fallback Strategies**
- What is the pedagogical impact of allowing on-screen keyboard fallbacks vs. requiring authentic MIDI hardware?
- Should teachers be required to provide justification when disabling MIDI requirements for students?
- How do we prevent students from deliberately avoiding MIDI hardware to use "easier" on-screen alternatives?

**iOS Custom App**
- Should we invest in bringing Web MIDI API support to iOS Safari, or continue maintaining the custom app?
- How do we ensure feature parity between the web platform and iOS app regarding MIDI functionality?
- What is the long-term maintenance strategy for the custom iOS app?

**User Experience**
- Should we auto-pause games when MIDI device disconnects, or allow brief reconnection grace period?
- How do we handle multiple MIDI devices connected simultaneously (e.g., student and teacher both have keyboards)?
- Should we remember MIDI device preferences per-game, per-student, or globally?

### Cross-Cutting Concerns

**Data and Analytics**
- How do we use MIDI device usage data to inform game design and device_profile requirements?
- Should we provide teachers with reports on which students consistently lack MIDI devices for purchasing decisions?
- What privacy considerations exist for tracking MIDI device names and connection patterns?

**Support and Training**
- What level of MIDI technical support should teachers be expected to provide vs. referring to platform documentation?
- Should we implement remote diagnostics to help support staff troubleshoot MIDI connection issues?
- How do we train teachers on the pedagogical benefits of MIDI hardware vs. fallback options?
