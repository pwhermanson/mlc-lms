# Browser Device Matrix

This document defines the QA testing matrix for browsers, operating systems, and devices that must be verified before each release.

## Purpose

This specification establishes the comprehensive testing matrix for quality assurance, defining which browser/OS/device combinations must be validated to ensure the platform delivers a consistent, functional experience across all supported environments. This matrix guides QA test planning, regression testing priorities, and release sign-off decisions.

Unlike the [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md) which defines which platforms are architecturally *supported*, this document focuses on practical QA execution: which combinations to test, test priorities, known issues, and testing frequency.

## Scope

### Included
- **Core Test Matrix** - Browser/OS/device combinations requiring pre-release testing
- **Test Priority Levels** - Critical, high, medium, and low priority test combinations
- **Feature Coverage by Platform** - Which features (games, MIDI, video, audio) must work on each platform
- **Testing Frequency** - How often each combination should be tested (per release, monthly, quarterly)
- **Known Issues and Limitations** - Documented platform-specific issues and workarounds
- **Regression Test Requirements** - Which tests must be re-run for each platform
- **Device-Specific Testing** - Physical device testing requirements vs. emulator/simulator testing

### Excluded
- **Architectural Support Decisions** - Covered in [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md)
- **Responsive Design Breakpoints** - Covered in [Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md)
- **Overall Test Strategy** - Covered in [Test Strategy](test-strategy.md)
- **Specific Test Cases** - Covered in [QA Checklists](qa-checklists.md)
- **Automated Testing Infrastructure** - Covered in [Testing Requirements](testing-requirements.md)

## Data model

### Test Matrix Configuration
```typescript
interface BrowserDeviceTestMatrix {
  matrixId: string;
  version: string;
  effectiveDate: Date;
  testCombinations: TestCombination[];
  testCycles: TestCycle[];
}

interface TestCombination {
  combinationId: string;
  priority: 'critical' | 'high' | 'medium' | 'low';
  browser: BrowserSpec;
  operatingSystem: OSSpec;
  deviceType: 'desktop' | 'laptop' | 'tablet' | 'smartphone';
  deviceModel?: string; // Specific device model if applicable
  featureSupport: FeatureSupport;
  testingMethod: 'physical-device' | 'emulator' | 'browser-stack' | 'virtual-machine';
  testingFrequency: 'per-release' | 'weekly' | 'monthly' | 'quarterly';
  knownIssues: KnownIssue[];
}

interface BrowserSpec {
  name: 'Chrome' | 'Firefox' | 'Safari' | 'Edge' | 'Opera' | 'MLC-App';
  version: string; // "latest", "latest-1", "latest-2", or specific version
  engine?: string; // "Chromium", "Gecko", "WebKit"
}

interface OSSpec {
  name: 'Windows' | 'macOS' | 'Chrome OS' | 'iOS' | 'Android';
  version: string; // "11", "10.15+", "latest", etc.
  architecture?: 'x64' | 'ARM64' | 'x86';
}

interface FeatureSupport {
  games: boolean; // HTML5 game playback
  midi: boolean; // MIDI keyboard support
  audio: boolean; // Audio playback and recording
  video: boolean; // Video playback
  webgl: boolean; // WebGL for advanced graphics
  touchEvents: boolean; // Touch interaction support
  accelerometer: boolean; // Device motion support
  camera: boolean; // Camera access for recording
  microphone: boolean; // Microphone access for recording
}

interface KnownIssue {
  issueId: string;
  severity: 'blocker' | 'critical' | 'major' | 'minor' | 'cosmetic';
  description: string;
  workaround?: string;
  affectedFeatures: string[];
  trackingLink?: string; // Issue tracker link
  plannedResolution?: Date;
}

interface TestCycle {
  cycleId: string;
  releaseVersion: string;
  startDate: Date;
  endDate: Date;
  testedCombinations: TestedCombination[];
  signOffStatus: 'pending' | 'approved' | 'rejected';
}

interface TestedCombination {
  combinationId: string;
  testDate: Date;
  tester: string;
  status: 'pass' | 'pass-with-issues' | 'fail' | 'blocked' | 'not-tested';
  testResults: TestResult[];
  notes?: string;
}

interface TestResult {
  testCaseId: string;
  testName: string;
  status: 'pass' | 'fail' | 'blocked' | 'skipped';
  defectsFound: string[]; // Links to bug tracker
  screenshots?: string[];
  executionTime: number; // seconds
}
```

## Behavior and rules

### Core Test Matrix

Based on the legacy MLC system browser chart and modern web standards, the following browser/OS/device combinations define the QA test matrix:

#### Critical Priority (Must Pass Before Release)

**Desktop - Windows**
- Chrome (latest) + Windows 11
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Physical device, per-release
- Chrome (latest) + Windows 10
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Virtual machine, per-release
- Edge (latest) + Windows 11
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Physical device, per-release

**Desktop - macOS**
- Chrome (latest) + macOS (latest)
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Physical device, per-release
- Safari (latest) + macOS (latest)
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Physical device, per-release

**Mobile - iOS**
- Safari (latest) + iOS (latest) - iPhone
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: Physical device, per-release
- Safari (latest) + iOS (latest) - iPad
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: Physical device, per-release

**Mobile - Android**
- Chrome (latest) + Android (latest) - Samsung/Google devices
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: Physical device, per-release

#### High Priority (Should Pass Before Release)

**Desktop - Windows**
- Firefox (latest) + Windows 11
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓
  - Testing: Virtual machine, per-release
- Chrome (latest-1) + Windows 10/11
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: BrowserStack, monthly

**Desktop - macOS**
- Firefox (latest) + macOS (latest)
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓
  - Testing: Physical device, monthly
- Safari (latest-1) + macOS (latest-1)
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Physical device, monthly

**Desktop - Chrome OS**
- Chrome (latest) + Chrome OS (latest)
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Physical device (Chromebook), monthly

**Mobile - iOS**
- Chrome (latest) + iOS (latest) - iPhone
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: Physical device, monthly

**Mobile - Android**
- Chrome (latest) + Android (latest) - Various manufacturers
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: Physical device, monthly

#### Medium Priority (Nice to Have Before Release)

**Desktop - Windows**
- Opera (latest) + Windows 10/11
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓
  - Testing: BrowserStack, quarterly
- Edge (latest-1) + Windows 10
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓
  - Testing: Virtual machine, quarterly

**Desktop - macOS**
- Opera (latest) + macOS (latest)
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓
  - Testing: BrowserStack, quarterly

**Mobile - iOS**
- Safari + iOS (latest-1) - iPhone/iPad
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: Physical device, quarterly

**Mobile - Android**
- Firefox (latest) + Android (latest)
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: BrowserStack, quarterly
- Chrome (latest) + Android (older versions) - 2-3 years old
  - Features: Games ✓, MIDI ✓, Audio ✓, Video ✓, WebGL ✓, Touch ✓
  - Testing: BrowserStack, quarterly

#### Low Priority (Opportunistic Testing)

**Desktop - Windows**
- Chrome/Edge/Firefox (latest-2, latest-3) + Windows 10/11
  - Features: Games ✓, MIDI varies, Audio ✓, Video ✓, WebGL ✓
  - Testing: BrowserStack, as-needed

**Desktop - Linux**
- Chrome/Firefox (latest) + Ubuntu (latest LTS)
  - Features: Games ✓, MIDI ✗, Audio ✓, Video ✓, WebGL ✓
  - Testing: BrowserStack, as-needed
  - Note: Not officially supported, but opportunistic testing for open-source users

### Testing Frequency Rules

**Per-Release Testing** (Critical Priority)
- Required before any production deployment
- All critical priority combinations must pass
- Blocking issues must be resolved or documented workarounds provided
- Sign-off required from QA lead

**Monthly Testing** (High Priority)
- Conducted on first Monday of each month
- Focuses on high-priority combinations
- Regression testing of recently fixed issues
- Performance benchmarking across platforms

**Quarterly Testing** (Medium Priority)
- Conducted at end of each quarter
- Comprehensive testing of medium-priority combinations
- Full regression test suite
- Browser version updates and compatibility checks

**As-Needed Testing** (Low Priority)
- Conducted when user reports issues
- New browser version releases
- Special feature testing
- Community-reported compatibility issues

### Feature Coverage Requirements

**Games (HTML5 Game Playback)**
- Must work: All critical and high priority combinations
- Should work: All medium priority combinations
- Best effort: Low priority combinations
- Test criteria: Game loads, responds to input, saves progress, audio works

**MIDI Support**
- Must work: Chrome (all platforms), Edge (Windows), Safari (macOS)
- Known limitations: Firefox (no Web MIDI API support), Safari (iOS - hardware limitation)
- Test criteria: MIDI device detected, note input captured, latency < 50ms

**Audio Playback**
- Must work: All critical and high priority combinations
- Test criteria: Audio plays without stuttering, volume controls work, no sync issues

**Video Playback**
- Must work: All critical and high priority combinations
- Test criteria: Video plays smoothly, controls work, no buffering issues

**Touch Interactions**
- Must work: All mobile and tablet combinations
- Test criteria: Tap, swipe, pinch-to-zoom work correctly, no ghost touches

### Known Issues and Limitations

**Safari (iOS) - MIDI Support**
- Issue: Web MIDI API not supported on iOS Safari
- Severity: Major (known platform limitation)
- Workaround: Direct students to use desktop/Android for MIDI keyboard exercises
- Status: Platform limitation, no planned resolution

**Firefox - Web MIDI API**
- Issue: Firefox does not support Web MIDI API
- Severity: Major (known platform limitation)
- Workaround: Recommend Chrome/Edge for MIDI keyboard exercises
- Status: Firefox policy decision, no planned support

**Android - Audio Latency**
- Issue: Some Android devices have high audio latency (>100ms)
- Severity: Minor to Major (device-dependent)
- Workaround: Recommend devices with Android 10+ and low-latency audio hardware
- Status: Hardware-dependent, ongoing optimization

**Safari (macOS) - WebGL Performance**
- Issue: Some complex games have lower frame rates on Safari vs Chrome
- Severity: Minor (performance, not functionality)
- Workaround: Recommend Chrome for best performance
- Status: Ongoing optimization

**Older Browsers - Modern JavaScript Features**
- Issue: Browsers older than 2 years may lack modern JavaScript features
- Severity: Critical (functionality breaking)
- Workaround: Display upgrade prompt to users
- Status: Browsers older than supported versions are not officially supported

### Regression Testing Requirements

**After Bug Fixes**
- Test on the originally reported browser/OS combination
- Test on all critical priority combinations
- Test on related browser engines (e.g., fix in Chrome → test in Edge)

**After New Features**
- Test new feature on all critical and high priority combinations
- Verify existing features still work (smoke test)
- Document any new platform-specific behavior

**After Third-Party Updates**
- Browser version updates: Test within 1 week of release
- OS version updates: Test within 2 weeks of release
- Third-party library updates: Full regression test on all critical combinations

### Test Environment Requirements

**Physical Device Testing**
- Required for: All critical mobile combinations, primary desktop combinations
- Devices needed:
  - Desktop: 1x Windows 11 PC, 1x macOS (latest) laptop
  - Mobile: 1x iPhone (latest iOS), 1x iPad (latest iOS), 2x Android devices (Samsung, Google)
  - Chromebook: 1x Chromebook (latest Chrome OS)

**Virtual/Emulator Testing**
- Acceptable for: High and medium priority desktop combinations
- Tools: VirtualBox, VMware, Xcode Simulator, Android Emulator

**Cloud Testing (BrowserStack/Sauce Labs)**
- Acceptable for: Medium and low priority combinations
- Use for: Older browser versions, less common OS versions, quick smoke tests

## UX requirements

### Visual Regression Testing
- Screenshot capture for all critical UI states across all critical priority combinations
- Automated visual diff detection
- Manual review for layout and rendering issues
- Document platform-specific rendering differences (e.g., font rendering, colors)

### Performance Benchmarks
- Game load time: < 3 seconds on all critical combinations
- Page load time: < 2 seconds on all combinations
- Interactive time: < 1 second on all combinations
- MIDI latency: < 50ms on supported combinations
- Audio latency: < 100ms on all combinations

### Accessibility Testing by Platform
- Screen reader compatibility: Test on VoiceOver (iOS/macOS), TalkBack (Android), JAWS/NVDA (Windows)
- Keyboard navigation: Test on all desktop combinations
- Touch accessibility: Test on all mobile combinations
- High contrast mode: Test on Windows, macOS, iOS

## Telemetry

### Browser/Device Usage Analytics
Track actual user device/browser combinations to inform testing priorities:

```typescript
interface DeviceUsageEvent {
  eventType: 'session_start';
  userId: string;
  sessionId: string;
  timestamp: Date;
  device: {
    browser: string;
    browserVersion: string;
    operatingSystem: string;
    osVersion: string;
    deviceType: 'desktop' | 'mobile' | 'tablet';
    deviceModel?: string;
    screenResolution: string;
    touchSupport: boolean;
  };
  features: {
    webgl: boolean;
    webmidi: boolean;
    webAudio: boolean;
    serviceWorker: boolean;
    localStorage: boolean;
  };
}
```

### Test Execution Tracking
```typescript
interface TestExecutionEvent {
  testCycleId: string;
  combinationId: string;
  testCaseId: string;
  tester: string;
  executionTime: number;
  status: 'pass' | 'fail' | 'blocked' | 'skipped';
  timestamp: Date;
}
```

### Issue Discovery Tracking
```typescript
interface IssueDiscoveryEvent {
  issueId: string;
  combinationId: string;
  discoveryMethod: 'automated-test' | 'manual-test' | 'user-report';
  severity: 'blocker' | 'critical' | 'major' | 'minor' | 'cosmetic';
  timestamp: Date;
}
```

### Analytics Use Cases
- Identify which browser/OS combinations are most used by actual users
- Adjust test priority based on real usage patterns
- Track platform-specific issue trends
- Measure test coverage effectiveness
- Estimate testing effort for releases

## Permissions

### Test Matrix Management
- **QA Engineers**: Can execute tests, report results, create defects
- **QA Leads**: Can modify test priorities, approve sign-offs, update known issues
- **Product Managers**: Can view test status, approve release sign-offs
- **Engineering Leads**: Can view test results, provide technical input on known issues
- **System Admins**: Can modify test matrix configuration, manage test environments

### Test Environment Access
- **QA Engineers**: Access to all test devices and cloud testing platforms
- **Developers**: Access to test devices for debugging, read-only access to test results
- **Product/Design**: Access to test devices for review, read-only test results

---

## Supporting Documents Referenced

This browser device matrix specification draws from the following source documents:

- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Legacy MLC 4.0 browser compatibility requirements and minimum version specifications that inform the testing matrix priorities
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game device requirements (MIDI keyboard, microphone, touch) informing feature support matrix by platform
- [MLC_LearningUI_wireframes_201210.pdf](../00-ORIG-CONTEXT/MLC_LearningUI_wireframes_201210.pdf) — 2012 UI wireframes showing responsive design considerations and browser-specific layout requirements for testing validation

---

## Dependencies

### Internal Dependencies
- **[Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md)** - Defines which platforms are architecturally supported
- **[Responsive Breakpoints](../01-ux-design-system/responsive-breakpoints.md)** - Defines responsive behavior that must be tested
- **[Test Strategy](test-strategy.md)** - Overall testing approach and methodology
- **[QA Checklists](qa-checklists.md)** - Specific test cases to execute
- **[Testing Requirements](testing-requirements.md)** - Automated testing infrastructure
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)** - Accessibility requirements to test

### External Dependencies
- **Browser Vendors**: Browser update schedules affect testing frequency
- **OS Vendors**: OS update schedules affect testing requirements
- **Device Manufacturers**: Device availability affects physical device testing
- **Cloud Testing Providers**: BrowserStack/Sauce Labs for automated cross-browser testing
- **CI/CD Pipeline**: Automated test execution triggers

### Technical Dependencies
- **Web Standards**: HTML5, CSS3, JavaScript ES2020+, Web MIDI API, Web Audio API
- **Testing Tools**: Selenium, Playwright, BrowserStack, Percy (visual regression)
- **Device Labs**: Physical device inventory for testing
- **Issue Tracking**: Jira/GitHub Issues for defect management

## Open questions

### Testing Strategy Questions
- **Automation Coverage**: What percentage of the test matrix can be automated vs. manual?
- **Cloud Testing**: Which cloud testing platform (BrowserStack, Sauce Labs, LambdaTest) provides best coverage?
- **Test Data Management**: How to maintain consistent test data across all test environments?
- **Test Execution Time**: How to optimize test execution time while maintaining coverage?

### Device Coverage Questions
- **Minimum Supported Versions**: How far back should browser/OS version support extend?
- **Device Diversity**: Should testing include low-end devices to test performance limits?
- **Physical vs. Virtual**: Which tests absolutely require physical devices vs. emulators?
- **Device Lab Management**: How to maintain and update the physical device lab?

### Priority and Frequency Questions
- **Dynamic Prioritization**: Should test priorities adjust based on real user analytics?
- **Regression Scope**: What triggers a full regression test vs. targeted smoke test?
- **Release Gating**: Which test failures should block releases vs. be documented as known issues?
- **Continuous Testing**: Should high-priority combinations be tested continuously (nightly builds)?

### Platform-Specific Questions
- **iOS Testing**: How to test on iOS devices given limited debugging tools?
- **Android Fragmentation**: How many Android device models are sufficient for coverage?
- **MIDI Support**: How to test MIDI functionality consistently across platforms?
- **Performance Baselines**: What are acceptable performance benchmarks for each platform?

### Process Questions
- **Sign-Off Process**: Who must approve test results before release?
- **Issue Triage**: How to prioritize and assign platform-specific issues?
- **Communication**: How to communicate known platform issues to users and support team?
- **Feedback Loop**: How to incorporate user-reported platform issues into the test matrix?
