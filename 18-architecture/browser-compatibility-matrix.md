# Browser Compatibility Matrix

## Overview
Comprehensive browser and device compatibility specifications for the platform.

## Description
Detailed compatibility requirements and testing specifications for various browsers, operating systems, and devices, ensuring optimal user experience across all supported platforms.

---

## Purpose

This document defines the **architectural requirements** for browser compatibility across the MLC LMS platform. It specifies:

- Which browsers and versions are officially supported
- Critical browser APIs and features required for core functionality
- Feature detection and progressive enhancement strategies
- Polyfill and fallback approaches for legacy browser support
- Browser-specific implementation considerations

**Related documents:**
- For QA/testing requirements, see [Browser Device Matrix](../19-quality-and-operations/browser-device-matrix.md)
- For responsive design behaviors, see [Responsive Behaviors](../21-mobile-and-offline/responsive-behaviors.md)
- For overall technology choices, see [Technology Stack](../00-foundations/techstack.md)

---

## Scope

### Included in This Specification

- Official browser support matrix with minimum versions
- Required browser features and APIs for core functionality
- Critical APIs for music education features (Web Audio, Web MIDI)
- Feature detection and capability testing strategies
- Polyfill requirements and fallback behaviors
- Browser-specific quirks and handling strategies
- Mobile browser considerations (iOS Safari, Chrome Mobile, etc.)
- Performance considerations per browser

### Excluded from This Specification

- QA test plans and device testing procedures (see [Browser Device Matrix](../19-quality-and-operations/browser-device-matrix.md))
- Responsive design breakpoints and layouts (see [Responsive Behaviors](../21-mobile-and-offline/responsive-behaviors.md))
- Detailed implementation of frontend components (see [Components Overview](../01-ux-design-system/components-overview.md))
- Accessibility standards (see [A11y Standards](../01-ux-design-system/a11y-standards.md))

---

## Supported Browsers and Platforms

### Desktop Browsers

Based on analysis of legacy MLC compatibility requirements and modern web standards:

| Browser | Minimum Version | Game Support | MIDI Support | Notes |
|---------|----------------|--------------|--------------|-------|
| **Chrome** | 90+ | ✅ Full | ✅ Full | Primary development target |
| **Edge** (Chromium) | 90+ | ✅ Full | ✅ Full | Chromium-based, equivalent to Chrome |
| **Firefox** | 88+ | ✅ Full | ✅ Full | Web MIDI requires user permission |
| **Safari** | 14+ | ✅ Full | ⚠️ Limited | Web MIDI available in Safari 14.1+ |
| **Opera** | 76+ | ✅ Full | ✅ Full | Chromium-based |

**Version Support Policy:**
- Support last 2 major versions for evergreen browsers
- Test against current stable and previous major release
- Monitor usage analytics quarterly to adjust support matrix

### Mobile Browsers

| Platform | Browser | Minimum Version | Game Support | MIDI Support | Notes |
|----------|---------|----------------|--------------|--------------|-------|
| **iOS** | Safari | iOS 14+ | ✅ Full | ⚠️ Limited | Web MIDI iOS 14.3+, requires physical MIDI adapter |
| **iOS** | Chrome | iOS 14+ | ✅ Full | ⚠️ Limited | Uses WebKit engine (same as Safari) |
| **Android** | Chrome | 90+ | ✅ Full | ✅ Full | Primary Android target |
| **Android** | Firefox | 88+ | ✅ Full | ✅ Full | Alternative Android browser |
| **Android** | Samsung Internet | 14+ | ✅ Full | ⚠️ Limited | Chromium-based with modifications |

**Mobile-Specific Considerations:**
- iOS browsers all use WebKit rendering engine (per App Store requirements)
- MIDI support on mobile requires USB or Bluetooth MIDI adapters
- Touch interactions require specific event handling (see [Responsive Behaviors](../21-mobile-and-offline/responsive-behaviors.md))
- Mobile Safari has specific audio context resume requirements

### Operating Systems

| OS | Minimum Version | Support Level | Notes |
|----|----------------|---------------|-------|
| **Windows** | Windows 10 | Full | Primary desktop platform |
| **macOS** | macOS 10.15 (Catalina) | Full | Full Safari and Chrome support |
| **ChromeOS** | Current - 2 versions | Full | Chromebook education market |
| **iOS** | iOS 14+ | Full | iPad and iPhone support |
| **Android** | Android 10+ | Full | Tablet and phone support |

### Explicitly Unsupported

The following browsers are **not supported** due to technical limitations or security concerns:

- ❌ Internet Explorer (all versions) - End of life, lacks modern web standards
- ❌ Opera Mini - Proxy-based rendering incompatible with interactive games
- ❌ UC Browser - Security concerns and non-standard behaviors
- ❌ Older Edge (EdgeHTML) - Deprecated, replaced by Chromium Edge
- ❌ iOS < 14 - Lacks critical Web APIs and security features
- ❌ Android < 10 - Insufficient Web Audio and performance capabilities

---

## Required Browser Features and APIs

### Core Web Platform Features

These features are **required** for the platform to function:

#### JavaScript and DOM
- **ECMAScript 2020 (ES11)** support minimum
  - Optional chaining (`?.`)
  - Nullish coalescing (`??`)
  - Promise.allSettled()
  - BigInt (for precise timing calculations)
- **Async/await** support
- **ES Modules** (import/export)
- **Service Workers** (for PWA features, optional but recommended)

#### CSS Features
- **CSS Grid** - Layout system for responsive design
- **CSS Flexbox** - Component layout
- **CSS Custom Properties (Variables)** - Theming system
- **CSS Transforms** - Animations and transitions
- **Media Queries** - Responsive breakpoints

#### Web APIs (General)
- **Fetch API** - Network requests
- **WebSocket API** - Real-time updates (future)
- **Local Storage / Session Storage** - Client-side state persistence
- **IndexedDB** - Offline data storage (future)
- **Clipboard API** - Copy/paste functionality
- **Notifications API** - Push notifications (optional)

### Music-Specific APIs

Critical for music education functionality:

#### Web Audio API
**Status:** Required for all game functionality

- **AudioContext** - Audio processing and synthesis
- **Audio routing and effects** - Reverb, dynamics, filtering
- **Precise timing** - Scheduling note playback
- **Audio buffer management** - Loading and playing audio files
- **Audio analysis** - Pitch detection and visualization

**Browser Support:**
- Chrome/Edge/Opera: Full support
- Firefox: Full support
- Safari: Full support (requires user gesture to resume AudioContext)

**Implementation Notes:**
- Must handle `AudioContext.state === 'suspended'` on page load (especially Safari)
- User interaction required to resume audio context
- See [Music Library Integration](../17-integrations/midi-support.md) for detailed implementation

#### Web MIDI API
**Status:** Required for MIDI keyboard input features

- **MIDI input detection** - Recognize connected MIDI devices
- **MIDI message handling** - Note on/off, velocity, control changes
- **Device enumeration** - List available MIDI inputs/outputs
- **Hot-plugging** - Detect device connection/disconnection

**Browser Support:**
- Chrome/Edge/Opera: Full support
- Firefox: Requires `dom.webmidi.enabled` flag (enabled by default since 108)
- Safari: Supported in Safari 14.1+ (macOS/iOS)

**Implementation Notes:**
- Requires HTTPS (secure context)
- User must grant permission to access MIDI devices
- Implement graceful fallback for browsers/devices without MIDI support
- See [MIDI Support](../17-integrations/midi-support.md) for detailed architecture

**Fallback Strategy:**
When Web MIDI is unavailable:
- Allow on-screen keyboard input
- Support computer keyboard as piano keyboard
- Display message: "Connect a MIDI keyboard for the best experience"

### Performance and Graphics APIs

#### Canvas API
**Status:** Required for game rendering

- **2D Context** - Core game graphics
- **Canvas transformations** - Rotation, scaling, translation
- **Image rendering** - Sprite display
- **Text rendering** - Score display and labels

**Note:** If Phaser.js game wrapper is implemented (backburner item), it will handle Canvas rendering internally.

#### WebGL (Optional Enhancement)
**Status:** Optional for advanced visual effects

- **WebGL 1.0** minimum
- Used for particle effects, advanced transitions
- Graceful degradation to Canvas 2D if unavailable

#### RequestAnimationFrame
**Status:** Required for smooth animations

- 60fps target for game loops
- Handles variable frame rates automatically
- Supported in all target browsers

---

## Feature Detection Strategy

### Detection Approach

The platform uses **feature detection**, not browser sniffing, to determine capabilities:

```javascript
// Example: Web Audio detection
const hasWebAudio = !!(window.AudioContext || window.webkitAudioContext);

// Example: Web MIDI detection
const hasWebMIDI = !!navigator.requestMIDIAccess;

// Example: Service Worker support
const hasSW = 'serviceWorker' in navigator;
```

### Capability Flags

Maintain a runtime capability object:

```javascript
// Centralized capability detection (pseudo-code)
const BrowserCapabilities = {
  audio: {
    webAudio: detectWebAudio(),
    webMIDI: detectWebMIDI(),
    audioContext: null, // Initialized on user interaction
  },
  storage: {
    localStorage: detectLocalStorage(),
    indexedDB: detectIndexedDB(),
  },
  features: {
    serviceWorker: detectServiceWorker(),
    webGL: detectWebGL(),
    clipboard: detectClipboard(),
  },
  platform: {
    isMobile: detectMobile(),
    isIOS: detectIOS(),
    isTouchDevice: detectTouch(),
  }
};
```

### Progressive Enhancement Pattern

1. **Core functionality first** - Basic HTML/CSS/JS that works everywhere
2. **Detect enhanced features** - Check for Web Audio, Web MIDI, etc.
3. **Enable enhanced experiences** - Activate features when available
4. **Provide fallbacks** - Alternative UI/UX when features unavailable

**Example:**
- If Web MIDI unavailable → Show on-screen keyboard
- If Web Audio unavailable → Show error message (cannot proceed without audio)
- If WebGL unavailable → Fall back to Canvas 2D rendering

---

## Polyfills and Compatibility Shims

### Polyfill Strategy

The platform uses **selective polyfills** to support target browsers:

#### Core Polyfills (Included by Default)

Since Next.js handles most modern JavaScript polyfills automatically, minimal additional polyfills needed:

- **Web Audio API Shims**
  - Safari AudioContext resume handling
  - Vendor prefix normalization (`webkitAudioContext` → `AudioContext`)
  
- **Fetch API** (if targeting older browsers)
  - Already included in Next.js polyfills for older browsers

#### Optional Polyfills (Load on Demand)

- **Web MIDI API Polyfill**
  - **WebMIDIAPIShim** - Provides Flash-based fallback (deprecated approach)
  - **Recommendation:** Do not polyfill; use feature detection and graceful degradation
  
- **Intersection Observer**
  - For lazy loading images and components
  - Next.js provides automatic polyfill

#### Polyfill Loading Strategy

```javascript
// Pseudo-code: Dynamic polyfill loading
async function ensurePolyfills() {
  const polyfills = [];
  
  // Only load if feature missing
  if (!window.IntersectionObserver) {
    polyfills.push(import('intersection-observer'));
  }
  
  await Promise.all(polyfills);
}
```

### Browser-Specific Shims

#### Safari-Specific Fixes

**Audio Context Resume:**
Safari requires user interaction before AudioContext can play audio:

```javascript
// Pseudo-code: Safari AudioContext initialization
function initAudioContext() {
  const AudioContextClass = window.AudioContext || window.webkitAudioContext;
  const audioContext = new AudioContextClass();
  
  // Safari requires resume() after user interaction
  if (audioContext.state === 'suspended') {
    document.addEventListener('click', () => {
      audioContext.resume();
    }, { once: true });
  }
  
  return audioContext;
}
```

**iOS Viewport Handling:**
iOS Safari has specific viewport behavior that requires meta tag configuration:

```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
```

Note: Disable zoom for game interactions, but ensure accessibility alternatives.

#### Firefox-Specific Considerations

**Web MIDI Permission:**
Firefox requires explicit user permission for MIDI access. Implement clear UI prompts:

```javascript
// Pseudo-code: MIDI permission handling
async function requestMIDIAccess() {
  try {
    const midiAccess = await navigator.requestMIDIAccess();
    return midiAccess;
  } catch (error) {
    // User denied permission or MIDI not supported
    showMIDIUnavailableMessage();
    return null;
  }
}
```

---

## Browser-Specific Implementation Notes

### Chrome/Edge (Chromium)

**Status:** Primary development target

- **Strengths:**
  - Full support for all required APIs
  - Best developer tools
  - Consistent behavior across Windows/Mac/Linux
  - Excellent performance

- **Considerations:**
  - Chromium engine updates rapidly; test regularly
  - Memory usage can be high with many audio buffers

### Firefox

**Status:** Fully supported

- **Strengths:**
  - Strong privacy features
  - Excellent developer tools
  - Good Web Audio performance

- **Considerations:**
  - Web MIDI requires user permission (show clear prompt)
  - Slightly different audio timing characteristics vs. Chromium

### Safari (macOS/iOS)

**Status:** Fully supported with caveats

- **Strengths:**
  - Required for iOS users
  - Good Web Audio performance
  - Increasing Web MIDI support

- **Considerations:**
  - **AudioContext autoplay restrictions** - Requires user gesture
  - **Web MIDI limited on iOS** - Requires physical MIDI adapter
  - **iOS uses WebKit for all browsers** - Chrome/Firefox on iOS = Safari engine
  - **Different touch behavior** - Requires iOS-specific event handling
  - **Viewport quirks** - Address bar shows/hides affecting viewport height
  - **Local Storage limits** - Stricter limits in Private Browsing mode

**Safari-Specific Testing Requirements:**
- Test audio initialization on every page load
- Test MIDI device detection on both macOS and iOS
- Test touch interactions on iPad and iPhone
- Verify viewport behavior with address bar transitions

### Mobile Browsers

**General Mobile Considerations:**

- **Touch events** vs. mouse events
- **Viewport management** - Address bars, notches, safe areas
- **Performance constraints** - Lower CPU/GPU than desktop
- **Network variability** - Cellular connections, offline scenarios
- **Battery consumption** - Audio processing impact

**iOS Safari Specific:**
- AudioContext must be created/resumed after user gesture
- No true background audio processing
- Limited concurrent audio contexts
- Web MIDI requires iOS 14.3+ and physical adapter

**Android Chrome Specific:**
- Generally similar to desktop Chrome
- Web MIDI works with USB OTG adapters and Bluetooth MIDI
- Better background audio support than iOS

---

## Performance Considerations by Browser

### Audio Performance

**Chrome/Edge:**
- Can handle 50+ simultaneous audio sources
- Low-latency MIDI input processing
- Recommended for advanced music production features

**Firefox:**
- Similar performance to Chrome for most use cases
- Slightly higher latency in some MIDI scenarios

**Safari:**
- Good performance for typical educational scenarios (10-20 audio sources)
- Higher latency on older iOS devices
- Limit simultaneous audio sources on mobile (< 16 recommended)

### Rendering Performance

**Target Frame Rates:**
- **Desktop:** 60fps for game rendering
- **Mobile:** 30fps minimum, 60fps target on newer devices

**Browser Considerations:**
- Chrome/Edge: Best Canvas/WebGL performance
- Firefox: Comparable Canvas performance, slightly slower WebGL
- Safari: Good performance, optimize for iOS devices (A12+ chip recommended)

### Memory Management

**Audio Buffer Limits:**
- Desktop: Up to 500MB audio data loaded
- Mobile: Limit to 100MB, lazy-load as needed

**Browser-Specific Limits:**
- Safari: More aggressive memory management, may purge cached audio
- Chrome: Higher limits but monitor memory usage
- Mobile: Always assume constrained memory environment

---

## Unsupported Browser Handling

### Detection and Messaging

When a user accesses the platform with an unsupported browser:

**Detection Method:**
```javascript
// Pseudo-code: Browser support check
function isBrowserSupported() {
  // Check for required features
  const hasRequiredFeatures = 
    !!window.AudioContext && 
    !!window.fetch &&
    !!window.Promise &&
    !!document.querySelector;
    
  return hasRequiredFeatures;
}
```

**User Messaging:**

Display clear, actionable message:

```
Your browser is not supported for Music Learning Community.

For the best experience, please use:
- Chrome 90 or newer
- Firefox 88 or newer  
- Safari 14 or newer
- Microsoft Edge 90 or newer

Need help? [Contact Support]
```

**Graceful Degradation:**
- Allow users to view limited content (marketing pages, documentation)
- Block access to interactive game features that require Web Audio
- Provide download links to supported browsers

### Enterprise and School Environments

**Common Scenarios:**
- Schools may use older managed browsers
- Enterprises may restrict browser updates
- Chromebooks often have automatic updates

**Approach:**
- Communicate minimum browser requirements to school IT departments
- Provide [Browser Compatibility page](../19-quality-and-operations/browser-device-matrix.md) for school administrators
- Offer technical support for environment validation
- Document in onboarding materials

---

## Testing and Validation

### Browser Testing Strategy

**Development Testing:**
- Primary: Latest Chrome (daily development)
- Secondary: Latest Firefox, Safari (weekly verification)
- Mobile: iOS Safari, Android Chrome (per feature)

**Pre-Release Testing:**
- Full test suite across all supported browsers
- Minimum two versions per browser (current + previous)
- Mobile testing on physical devices (iOS and Android)

**Automated Testing:**
- Cross-browser testing via Playwright (see [Test Strategy](../19-quality-and-operations/test-strategy.md))
- Visual regression testing for rendering differences
- Audio/MIDI capability detection tests

**Manual Testing Focus:**
- Audio playback quality and latency
- MIDI device detection and input handling
- Touch interaction responsiveness (mobile)
- Performance under load (multiple audio sources)

For detailed test plans and QA procedures, see:
- [Browser Device Matrix](../19-quality-and-operations/browser-device-matrix.md)
- [QA Checklists](../19-quality-and-operations/qa-checklists.md)
- [Test Strategy](../19-quality-and-operations/test-strategy.md)

---

## Version Policy and Updates

### Browser Version Support Timeline

**Evergreen Browser Policy:**
- Support latest stable version + 1 previous major version
- Example: If Chrome 110 is current, support Chrome 109+

**Review Cycle:**
- **Quarterly review** of browser version support
- **Analytics monitoring** of actual user browser versions
- **Adjust minimum versions** based on usage data (<2% usage = consider deprecation)

### Communication of Changes

When dropping support for older browser versions:
1. **90-day advance notice** to users
2. **Email notifications** to affected users (if identifiable)
3. **In-app banner warnings** when accessing from deprecated browser
4. **Update documentation** and support pages

### Emergency Support Updates

If a critical browser bug affects the platform:
1. **Identify affected browsers and versions**
2. **Implement hotfix or workaround**
3. **Communicate to users via system announcements**
4. **Document in incident log** (see [Incident Response](../19-quality-and-operations/incident-response.md))

---

## Supporting Documents Referenced

This browser compatibility matrix specification draws from the following source documents:

- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Legacy browser compatibility requirements and minimum version specifications from MLC 4.0
- [MLC_LearningUI_wireframes_201210.pdf](../00-ORIG-CONTEXT/MLC_LearningUI_wireframes_201210.pdf) — 2012 UI wireframes showing browser-specific layout considerations and responsive design requirements
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game implementation requirements informing Web Audio API, Canvas, and MIDI support specifications

---

## Dependencies

This specification relies on:

- **[Technology Stack](../00-foundations/techstack.md)** - Overall technology choices and frameworks
- **[System Overview](system-overview.md)** - Architecture and deployment model
- **[MIDI Support](../17-integrations/midi-support.md)** - Detailed Web MIDI implementation
- **[Browser Device Matrix](../19-quality-and-operations/browser-device-matrix.md)** - QA testing requirements
- **[Test Strategy](../19-quality-and-operations/test-strategy.md)** - Automated testing approach
- **[Responsive Behaviors](../21-mobile-and-offline/responsive-behaviors.md)** - Mobile/tablet specific behaviors

This specification informs:

- **Frontend component implementation** - Polyfill requirements
- **QA test plans** - Which browsers to test
- **User onboarding documentation** - System requirements
- **Support and troubleshooting** - Browser-specific issues

---

## Open Questions

1. **Safari 13 Support?**
   - Decision: No - Safari 14+ provides critical Web MIDI support
   - Risk: Some older macOS/iOS users (< 5% estimated)
   - Mitigation: Clear messaging about minimum iOS 14 requirement

2. **WebGL Requirement?**
   - Decision: Optional enhancement, not required
   - Rationale: Canvas 2D sufficient for current game graphics
   - Future: May require for advanced visual effects (Phaser integration)

3. **Service Worker Requirement?**
   - Decision: Optional for PWA features
   - Status: Not Phase 1 requirement
   - See: [PWA Specifications](../21-mobile-and-offline/pwa-specifications.md)

4. **Chromebook Offline Support?**
   - Priority: High (education market)
   - Dependencies: Service Workers, IndexedDB
   - See: [PWA Specifications](../21-mobile-and-offline/pwa-specifications.md)

---

## Related Documentation

- [Technology Stack](../00-foundations/techstack.md) - Overall technology choices
- [System Overview](system-overview.md) - Architecture overview
- [Browser Device Matrix](../19-quality-and-operations/browser-device-matrix.md) - QA testing matrix
- [Test Strategy](../19-quality-and-operations/test-strategy.md) - Testing approach
- [MIDI Support](../17-integrations/midi-support.md) - Web MIDI implementation
- [Responsive Behaviors](../21-mobile-and-offline/responsive-behaviors.md) - Mobile considerations
- [A11y Standards](../01-ux-design-system/a11y-standards.md) - Accessibility requirements
- [PWA Specifications](../21-mobile-and-offline/pwa-specifications.md) - Progressive web app features