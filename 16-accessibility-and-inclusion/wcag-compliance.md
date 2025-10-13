# WCAG 2.1 AA Compliance

## Overview
Comprehensive Web Content Accessibility Guidelines (WCAG) 2.1 AA compliance specifications for the MLC-LMS platform.

## Purpose

This specification provides technical implementation requirements for achieving and maintaining WCAG 2.1 Level AA conformance across the MLC-LMS platform. It serves as the authoritative reference for developers, QA engineers, and content creators to ensure that all features, interfaces, and content meet internationally recognized accessibility standards.

WCAG 2.1 AA compliance is critical for:
- **Legal compliance** with accessibility regulations (ADA, Section 508, international equivalents)
- **Educational access** ensuring all students can participate regardless of disability
- **Market access** meeting requirements for institutional and school district procurement
- **Ethical responsibility** providing equal access to music education for all learners

This document complements the broader [Accessibility Standards](../01-ux-design-system/a11y-standards.md) which covers business rules, UX requirements, and platform-wide accessibility strategy.

## Description
Detailed implementation requirements for meeting WCAG 2.1 AA standards, including technical specifications, testing procedures, and validation criteria to ensure the platform is accessible to users with disabilities.

## Scope

### Included
- **WCAG 2.1 Level A and AA Success Criteria** - All applicable success criteria implementation
- **Technical Implementation Guidelines** - HTML, CSS, JavaScript, ARIA best practices
- **Testing Procedures** - Automated and manual testing requirements
- **Validation Tools** - Recommended tools and testing protocols
- **Documentation Requirements** - Accessibility conformance statements and reports
- **Content Authoring Standards** - Requirements for games, sequences, and learning materials
- **Remediation Procedures** - Processes for addressing accessibility issues
- **Browser and Assistive Technology Support** - Compatibility requirements

### Excluded
- **Level AAA Requirements** - Beyond scope unless specifically beneficial
- **Physical Hardware Accessibility** - MIDI keyboard hardware specifications (covered in [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md))
- **Pedagogy and Learning Design** - Covered in [Pedagogy Principles](../00-foundations/pedagogy-principles.md)
- **Neurodiverse-Specific Features** - Covered in [Neurodiverse Support](../16-accessibility-and-inclusion/neurodiverse-support.md)

## WCAG 2.1 Level A Conformance

### Perceivable - Information and user interface components must be presentable to users in ways they can perceive

#### 1.1 Text Alternatives

**1.1.1 Non-text Content (Level A)**
- All images must have appropriate alt text describing content and function
- Decorative images must use `alt=""` or CSS backgrounds
- Complex images (diagrams, charts) require extended descriptions via `aria-describedby` or adjacent text
- Music notation images require descriptive text alternatives
- Form inputs must have associated labels
- Icons must have text equivalents via `aria-label` or `sr-only` text

**Implementation:**
```html
<!-- Image with meaningful content -->
<img src="treble-clef.png" alt="Treble clef showing G note on the second line">

<!-- Decorative image -->
<img src="decoration.png" alt="" role="presentation">

<!-- Icon button -->
<button aria-label="Play audio">
  <i class="icon-play" aria-hidden="true"></i>
</button>

<!-- Complex diagram with extended description -->
<img src="staff-notation.png" alt="Musical staff notation" aria-describedby="staff-desc">
<div id="staff-desc">
  A five-line staff showing notes C, D, E, F, G ascending from the first space to the top line.
</div>
```

#### 1.2 Time-based Media

**1.2.1 Audio-only and Video-only (Level A)**
- Pre-recorded audio-only content must have text transcripts
- Pre-recorded video-only content must have text descriptions or audio tracks

**1.2.2 Captions (Level A)**
- All pre-recorded audio content in synchronized media must have captions
- Applies to instructional videos, game audio, musical examples with spoken instruction

**1.2.3 Audio Description or Media Alternative (Level A)**
- Pre-recorded video content must have audio descriptions or full text alternatives
- Musical demonstration videos require descriptions of visual elements

**Implementation:**
- Use WebVTT format for captions
- Provide transcript downloads
- Ensure caption synchronization accuracy within 200ms
- See [Captions and Audio Descriptions](../16-accessibility-and-inclusion/captions-and-audio-descriptions.md) for detailed requirements

#### 1.3 Adaptable

**1.3.1 Info and Relationships (Level A)**
- Use semantic HTML to convey structure and relationships
- Headings must follow logical hierarchy (h1, h2, h3, etc.)
- Lists must use `<ul>`, `<ol>`, `<dl>` elements
- Tables must use proper markup with `<th>` headers and scope attributes
- Form labels must be programmatically associated with inputs
- ARIA landmarks must identify page regions

**Implementation:**
```html
<!-- Proper heading hierarchy -->
<h1>Student Dashboard</h1>
<h2>Current Assignments</h2>
<h3>Assignment: Treble Clef Notes</h3>

<!-- Semantic list -->
<ul aria-label="Assignment stages">
  <li>Learn (Completed)</li>
  <li>Play (In Progress)</li>
  <li>Quiz (Not Started)</li>
</ul>

<!-- Data table with headers -->
<table>
  <caption>Student Progress Report</caption>
  <thead>
    <tr>
      <th scope="col">Game Name</th>
      <th scope="col">Stage</th>
      <th scope="col">Score</th>
      <th scope="col">Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Note Names</th>
      <td>Quiz</td>
      <td>95%</td>
      <td>2025-10-10</td>
    </tr>
  </tbody>
</table>

<!-- Landmark regions -->
<header role="banner">
  <nav aria-label="Main navigation">...</nav>
</header>
<main role="main">...</main>
<aside role="complementary">...</aside>
<footer role="contentinfo">...</footer>
```

**1.3.2 Meaningful Sequence (Level A)**
- DOM order must match visual presentation order
- Reading order must be logical when CSS is disabled
- Tab order must follow logical flow
- Flexbox/Grid layouts must maintain logical source order

**1.3.3 Sensory Characteristics (Level A)**
- Instructions must not rely solely on sensory characteristics (shape, size, color, visual location, sound)
- Combine visual indicators with text labels
- Example: "Click the green 'Start' button" not "Click the green button"

#### 1.4 Distinguishable

**1.4.1 Use of Color (Level A)**
- Color must not be the only visual means of conveying information
- Combine color with text, icons, patterns, or other visual cues
- Error messages must use text + icons, not just red color
- Required fields must use text label + asterisk, not just red outline

**1.4.2 Audio Control (Level A)**
- Auto-playing audio lasting more than 3 seconds must have pause/stop control
- Background music must be controllable independently of game audio
- Volume controls must be keyboard accessible

**Implementation:**
```html
<!-- Audio controls -->
<audio id="background-music" src="music.mp3">
  <button aria-label="Play background music">Play</button>
  <button aria-label="Pause background music">Pause</button>
  <input type="range" min="0" max="100" aria-label="Volume control">
</audio>
```

## WCAG 2.1 Level AA Conformance

### Perceivable (Continued)

#### 1.2 Time-based Media (AA)

**1.2.4 Captions (Live) (Level AA)**
- Live audio content must have captions
- Applies to live instructor sessions, webinars, live demonstrations

**1.2.5 Audio Description (Level AA)**
- Pre-recorded video content must have audio descriptions
- All videos with visual-only information must have narrated descriptions

#### 1.3 Adaptable (AA)

**1.3.4 Orientation (Level AA)**
- Content must not be restricted to a single display orientation (portrait or landscape)
- Support both orientations unless specific orientation is essential
- Allow rotation for tablets and mobile devices

**1.3.5 Identify Input Purpose (Level AA)**
- Input fields collecting user information must use appropriate autocomplete attributes
- Enables autofill and assists users with cognitive disabilities

**Implementation:**
```html
<form>
  <label for="email">Email Address</label>
  <input type="email" id="email" name="email" autocomplete="email">
  
  <label for="name">Full Name</label>
  <input type="text" id="name" name="name" autocomplete="name">
  
  <label for="phone">Phone Number</label>
  <input type="tel" id="phone" name="phone" autocomplete="tel">
</form>
```

#### 1.4 Distinguishable (AA)

**1.4.3 Contrast (Minimum) (Level AA)**
- Text contrast ratio must be at least 4.5:1 for normal text
- Large text (18pt+, or 14pt+ bold) must be at least 3:1
- Graphical objects and UI components must be at least 3:1 against adjacent colors

**Color Contrast Requirements:**
- Normal text: 4.5:1 minimum
- Large text (≥18pt or ≥14pt bold): 3:1 minimum
- UI components: 3:1 minimum
- Graphical objects: 3:1 minimum
- Incidental text (decorative, disabled) exempt

**Approved Color Combinations:**
```css
/* High contrast text combinations */
--text-primary: #1a1a1a;      /* on white: 16.1:1 ✓ */
--text-secondary: #4a4a4a;    /* on white: 9.7:1 ✓ */
--link-color: #0066cc;        /* on white: 7.5:1 ✓ */
--error-text: #c70000;        /* on white: 6.2:1 ✓ */
--success-text: #006600;      /* on white: 6.4:1 ✓ */

/* Button backgrounds */
--button-primary: #0066cc;    /* white text: 7.5:1 ✓ */
--button-secondary: #5a5a5a;  /* white text: 8.6:1 ✓ */
```

**Testing Tools:**
- WebAIM Contrast Checker
- Chrome DevTools Contrast Ratio inspector
- Automated testing in CI/CD pipeline

**1.4.4 Resize Text (Level AA)**
- Text must be resizable up to 200% without loss of content or functionality
- Page must remain usable when browser zoom is set to 200%
- No horizontal scrolling at 200% zoom (width ≤ 320px equivalent)
- Layout must adapt responsively

**Implementation:**
```css
/* Use relative units for font sizing */
body {
  font-size: 16px; /* base size */
}

h1 { font-size: 2rem; }     /* 32px */
h2 { font-size: 1.5rem; }   /* 24px */
p { font-size: 1rem; }       /* 16px */

/* Responsive containers */
.container {
  max-width: 100%;
  padding: 0 1rem;
}
```

**1.4.5 Images of Text (Level AA)**
- Avoid images of text except for logos or where essential
- Use HTML text with CSS styling instead of images
- Music notation may be exempted as essential
- Mathematical notation should use MathML when possible

**1.4.10 Reflow (Level AA)**
- Content must reflow to single column at 320px width without horizontal scrolling
- Equivalent to 400% zoom on 1280px viewport
- Exceptions: data tables, complex diagrams, toolbars requiring two dimensions

**1.4.11 Non-text Contrast (Level AA)**
- User interface components and graphical objects must have 3:1 contrast ratio
- Includes form inputs, buttons, focus indicators, data visualizations
- Inactive/disabled components exempt

**Examples:**
```css
/* Form inputs with sufficient contrast */
input, select, textarea {
  border: 2px solid #767676; /* 4.6:1 against white ✓ */
}

/* Focus indicators */
*:focus {
  outline: 3px solid #0066cc; /* 7.5:1 against white ✓ */
  outline-offset: 2px;
}

/* Chart/graph elements */
.data-bar {
  fill: #0066cc; /* 7.5:1 minimum ✓ */
}
```

**1.4.12 Text Spacing (Level AA)**
- Content must be readable when users override text spacing:
  - Line height: at least 1.5× font size
  - Paragraph spacing: at least 2× font size
  - Letter spacing: at least 0.12× font size
  - Word spacing: at least 0.16× font size
- No loss of content or functionality when these are applied

**Implementation:**
```css
/* Support user stylesheet overrides */
body {
  line-height: 1.5;
  letter-spacing: normal; /* allow user override */
}

p {
  margin-bottom: 2em; /* 2× font size */
}
```

**1.4.13 Content on Hover or Focus (Level AA)**
- Content triggered by hover or focus must be:
  - **Dismissible**: Can be dismissed without moving pointer/focus (ESC key)
  - **Hoverable**: Pointer can move over additional content without it disappearing
  - **Persistent**: Remains visible until dismissed or no longer relevant

**Example - Tooltip Implementation:**
```html
<button aria-describedby="tooltip1">Help</button>
<div role="tooltip" id="tooltip1" class="tooltip">
  This button provides contextual help
</div>

<script>
// Tooltip must:
// - Appear on hover and focus
// - Dismiss on ESC key
// - Remain visible when hovering over tooltip itself
// - Disappear when focus moves away
</script>
```

### Operable - User interface components and navigation must be operable

#### 2.1 Keyboard Accessible

**2.1.1 Keyboard (Level A)**
- All functionality must be operable via keyboard interface
- No keyboard traps (users can navigate away from all components)
- Standard keyboard shortcuts: Tab, Shift+Tab, Arrow keys, Enter, Space, Escape
- Music game controls must be keyboard accessible

**2.1.2 No Keyboard Trap (Level A)**
- Keyboard focus must not be trapped in any component
- If focus is restricted (e.g., modal dialogs), provide documented escape method
- Standard escape: ESC key or instructions provided

**2.1.4 Character Key Shortcuts (Level A)**
- If single-character shortcuts exist, must provide:
  - Turn off mechanism, OR
  - Remap mechanism, OR
  - Active only when component has focus

#### 2.2 Enough Time

**2.2.1 Timing Adjustable (Level A)**
- If time limits exist, provide ability to:
  - Turn off timing, OR
  - Adjust timing (at least 10× default), OR
  - Extend before timeout (20 seconds warning, simple action to extend)
- Applies to game timers, session timeouts, timed quizzes

**Implementation:**
```javascript
// Provide timeout warning and extension
function showTimeoutWarning() {
  const warning = document.getElementById('timeout-warning');
  warning.hidden = false;
  
  const extendButton = document.getElementById('extend-session');
  extendButton.addEventListener('click', () => {
    extendSession(600000); // 10 more minutes
    warning.hidden = true;
  });
}

// Show warning 20 seconds before timeout
setTimeout(showTimeoutWarning, sessionDuration - 20000);
```

**2.2.2 Pause, Stop, Hide (Level A)**
- Moving, blinking, scrolling, or auto-updating content must be pausable
- Applies to:
  - Auto-scrolling assignment lists
  - Animated Terry Treble mascot
  - Auto-advancing carousels
  - Real-time score updates

#### 2.3 Seizures and Physical Reactions

**2.3.1 Three Flashes or Below Threshold (Level A)**
- Content must not flash more than 3 times per second
- Applies to all animations, transitions, game effects
- Strict requirement for safety

**Testing:**
- Use Photosensitive Epilepsy Analysis Tool (PEAT)
- Review all animations frame-by-frame
- Ensure no rapid color changes or strobing effects

#### 2.4 Navigable

**2.4.1 Bypass Blocks (Level A)**
- Provide "Skip to main content" link at top of page
- Allow keyboard users to skip repetitive navigation
- Multiple skip links if page has multiple sections

**Implementation:**
```html
<body>
  <a href="#main-content" class="skip-link">Skip to main content</a>
  <a href="#navigation" class="skip-link">Skip to navigation</a>
  
  <header>
    <nav id="navigation">...</nav>
  </header>
  
  <main id="main-content" tabindex="-1">
    <!-- Main content -->
  </main>
</body>

<style>
.skip-link {
  position: absolute;
  top: -40px;
  left: 0;
  background: #000;
  color: #fff;
  padding: 8px;
  text-decoration: none;
  z-index: 100;
}

.skip-link:focus {
  top: 0;
}
</style>
```

**2.4.2 Page Titled (Level A)**
- All pages must have descriptive, unique titles
- Format: "Specific Page Name | Section | MLC-LMS"
- Example: "Assignment: Treble Clef Notes | Student Dashboard | MLC-LMS"

**2.4.3 Focus Order (Level A)**
- Focus order must be logical and intuitive
- Tab order follows visual layout and reading order
- Modal dialogs trap focus appropriately

**2.4.4 Link Purpose (Level A)**
- Link purpose must be determinable from link text alone or link text + programmatic context
- Avoid "click here" or "read more" without context
- Use descriptive link text

**Good Examples:**
```html
<a href="/games/treble-clef">Play Treble Clef Note Names Game</a>
<a href="/reports/student-123">View Alex Martinez's Progress Report</a>
```

**Bad Examples:**
```html
<a href="/games/123">Click here</a> <!-- Ambiguous -->
<a href="/reports/456">Read more</a> <!-- No context -->
```

**2.4.5 Multiple Ways (Level AA)**
- Provide multiple ways to locate pages within the site:
  - Main navigation menu
  - Search functionality
  - Sitemap
  - Breadcrumb trails
- Exception: Pages in a defined process (e.g., checkout)

**2.4.6 Headings and Labels (Level AA)**
- Headings and labels must be descriptive
- Headings describe topic or purpose
- Form labels describe purpose of input

**2.4.7 Focus Visible (Level AA)**
- Keyboard focus indicator must be visible
- Minimum 2px outline or equivalent visual indicator
- Contrast ratio of 3:1 against adjacent colors

**Implementation:**
```css
/* Visible focus indicator for all interactive elements */
a:focus, button:focus, input:focus, select:focus, textarea:focus {
  outline: 3px solid #0066cc;
  outline-offset: 2px;
}

/* High contrast focus for better visibility */
.high-contrast a:focus,
.high-contrast button:focus {
  outline: 4px solid #000;
  outline-offset: 3px;
}
```

#### 2.5 Input Modalities

**2.5.1 Pointer Gestures (Level A)**
- All multipoint or path-based gestures must have single-pointer alternative
- Pinch-zoom alternative: +/- buttons
- Swipe alternative: Previous/Next buttons
- Drag alternative: Move buttons

**2.5.2 Pointer Cancellation (Level A)**
- For single-pointer activation:
  - Down-event should not trigger action
  - Action occurs on up-event
  - Provides abort mechanism (move pointer away before releasing)
- Prevents accidental activation

**2.5.3 Label in Name (Level A)**
- Visible text label must be included in accessible name
- Speech input users can activate by speaking visible label
- Example: Button shows "Play" → accessible name includes "Play"

**2.5.4 Motion Actuation (Level A)**
- Functionality triggered by device motion must also be operable via UI components
- Must provide setting to disable motion actuation
- Example: Shake to shuffle → also provide shuffle button

### Understandable - Information and operation of user interface must be understandable

#### 3.1 Readable

**3.1.1 Language of Page (Level A)**
- Page language must be programmatically identified
- Use `lang` attribute on `<html>` element

**Implementation:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>MLC-LMS</title>
</head>
```

**3.1.2 Language of Parts (Level AA)**
- Language changes within page must be identified
- Use `lang` attribute on containing element

**Example:**
```html
<p>The solfege system uses <span lang="it">do, re, mi, fa, sol, la, ti, do</span> to represent notes.</p>
```

#### 3.2 Predictable

**3.2.1 On Focus (Level A)**
- Focus must not trigger context changes
- Context changes: opening new windows, moving focus, changing content significantly
- Exceptions must be clearly announced in advance

**3.2.2 On Input (Level A)**
- Changing setting of UI component must not automatically cause context change
- Unless user is advised beforehand
- Example: Form should not auto-submit when last field is filled

**3.2.3 Consistent Navigation (Level AA)**
- Navigation mechanisms repeated on multiple pages must be in consistent order
- Main menu, breadcrumbs, footer links maintain same relative order

**3.2.4 Consistent Identification (Level AA)**
- Components with same functionality must be identified consistently
- "Play" button icon and label consistent across all games
- "Help" icon consistent across all pages

#### 3.3 Input Assistance

**3.3.1 Error Identification (Level A)**
- Input errors must be identified in text
- Specific error described to user
- Error location clearly indicated

**Implementation:**
```html
<form>
  <label for="email">Email Address *</label>
  <input type="email" id="email" aria-describedby="email-error" aria-invalid="true">
  <div id="email-error" role="alert" class="error">
    Error: Please enter a valid email address in the format name@domain.com
  </div>
</form>
```

**3.3.2 Labels or Instructions (Level A)**
- Labels or instructions provided when input required
- Required fields clearly marked
- Expected format described (e.g., "MM/DD/YYYY")

**3.3.3 Error Suggestion (Level AA)**
- If input error detected and correction known, provide suggestions
- Example: "Email must include @ symbol. Did you mean: john@example.com?"

**3.3.4 Error Prevention (Legal, Financial, Data) (Level AA)**
- For legal commitments, financial transactions, data deletion:
  - Reversible, OR
  - Checked (data validated), OR
  - Confirmed (confirmation page before final submission)
- Applies to: subscriptions, billing, student data deletion

**Implementation:**
```html
<!-- Confirmation dialog before deletion -->
<dialog role="dialog" aria-labelledby="confirm-title" aria-describedby="confirm-desc">
  <h2 id="confirm-title">Confirm Student Deletion</h2>
  <p id="confirm-desc">
    Are you sure you want to delete student Alex Martinez? This action cannot be undone.
    All assignments, scores, and progress data will be permanently deleted.
  </p>
  <button onclick="confirmDelete()">Yes, Delete Student</button>
  <button onclick="cancelDelete()" autofocus>Cancel</button>
</dialog>
```

### Robust - Content must be robust enough to be interpreted by a wide variety of user agents, including assistive technologies

#### 4.1 Compatible

**4.1.1 Parsing (Level A) - DEPRECATED in WCAG 2.2**
- Valid HTML no longer strictly required in WCAG 2.2
- However, maintain valid HTML as best practice
- Validate with W3C Markup Validator

**4.1.2 Name, Role, Value (Level A)**
- All UI components must have programmatically determined:
  - **Name**: Accessible name (label)
  - **Role**: What it is (button, link, checkbox, etc.)
  - **Value**: Current state/value
  - **State**: Checked, expanded, disabled, etc.

**Implementation:**
```html
<!-- Button with name and role -->
<button>Play Game</button>

<!-- Custom checkbox with ARIA -->
<div role="checkbox" 
     aria-checked="false" 
     aria-labelledby="label1"
     tabindex="0">
</div>
<span id="label1">Enable background music</span>

<!-- Expandable section -->
<button aria-expanded="false" aria-controls="details1">
  More Details
</button>
<div id="details1" hidden>
  <!-- Additional content -->
</div>

<!-- Progress indicator -->
<div role="progressbar" 
     aria-valuenow="75" 
     aria-valuemin="0" 
     aria-valuemax="100"
     aria-label="Assignment completion">
  75% Complete
</div>
```

**4.1.3 Status Messages (Level AA)**
- Status messages must be programmatically determined through role or properties
- Users can be informed without receiving focus
- Use ARIA live regions

**Implementation:**
```html
<!-- Success message -->
<div role="status" aria-live="polite" aria-atomic="true">
  Assignment saved successfully
</div>

<!-- Error message - assertive for immediate attention -->
<div role="alert" aria-live="assertive" aria-atomic="true">
  Unable to save. Please try again.
</div>

<!-- Progress update -->
<div aria-live="polite" aria-atomic="false">
  <span>Completed 3 of 10 questions</span>
</div>
```

## Assistive Technology Support

### Screen Readers
**Minimum Support:**
- NVDA 2021+ (Windows, free)
- JAWS 2021+ (Windows, commercial)
- VoiceOver (macOS, iOS, built-in)
- TalkBack (Android, built-in)

**Testing Requirements:**
- Test all critical user flows with at least 2 screen readers
- Verify all interactive elements are announced correctly
- Test form completion and error handling
- Verify game instructions are accessible

### Browser Compatibility
**Accessibility API Support:**
Per [Browser Chart](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv), support modern accessibility APIs in:
- Chrome 90+ (PC, Mac, Chrome OS, Android)
- Firefox 88+ (PC, Mac, Android)
- Safari 14+ (Mac, iOS)
- Edge 90+ (PC, Mac)

**Testing Matrix:**
- Screen reader + browser combinations:
  - NVDA + Chrome/Firefox (Windows)
  - JAWS + Chrome/Edge (Windows)
  - VoiceOver + Safari (macOS, iOS)
  - TalkBack + Chrome (Android)

### Keyboard Navigation Patterns
**Standard Patterns:**
- **Tab**: Move forward through interactive elements
- **Shift + Tab**: Move backward
- **Enter**: Activate links and buttons
- **Space**: Activate buttons, toggle checkboxes
- **Arrow Keys**: Navigate within components (radio groups, tabs, menus)
- **Escape**: Close dialogs, dismiss popups
- **Home/End**: Jump to first/last item in lists
- **Page Up/Down**: Scroll large content areas

**Music Game Specific:**
- **Letter keys (A-G)**: Play corresponding notes (when game has focus)
- **Number keys (1-5)**: Select answer options
- **Arrow keys**: Navigate staff notation, adjust note positions
- See [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md) for full MIDI keyboard specifications

## Testing and Validation

### Automated Testing Tools

**Required Tools:**
1. **axe DevTools** (Browser extension)
   - Run on every page during development
   - Integrate into CI/CD pipeline
   - Must pass with 0 violations before deployment

2. **WAVE** (WebAIM)
   - Secondary validation tool
   - Manual review of flagged items

3. **Lighthouse Accessibility Audit** (Chrome DevTools)
   - Score must be 95+ for all pages
   - Review flagged items and false positives

4. **Pa11y** (Command line)
   - Automated testing in CI/CD
   - Fail builds with critical violations

**CI/CD Integration:**
```yaml
# Example GitHub Actions workflow
- name: Run Accessibility Tests
  run: |
    npm run test:a11y
    pa11y-ci --threshold 10
```

### Manual Testing Procedures

**Keyboard Testing:**
1. Disconnect mouse/trackpad
2. Navigate entire page using only keyboard
3. Verify all interactive elements reachable
4. Verify focus indicator always visible
5. Verify logical tab order
6. Test all keyboard shortcuts
7. Verify no keyboard traps

**Screen Reader Testing:**
1. Navigate page with screen reader
2. Verify all content announced correctly
3. Test form completion end-to-end
4. Verify error messages announced
5. Test dynamic content updates (ARIA live regions)
6. Verify image alt text appropriate
7. Test game instructions and feedback

**Visual Testing:**
1. Zoom to 200% - verify no horizontal scroll
2. Test with Windows High Contrast Mode
3. Verify color contrast with Color Contrast Analyzer
4. Test with reduced motion settings enabled
5. Test with custom stylesheets (text spacing overrides)

**Mobile Accessibility Testing:**
1. Test with mobile screen readers (VoiceOver, TalkBack)
2. Verify touch targets minimum 44px × 44px
3. Test portrait and landscape orientations
4. Verify pinch-zoom enabled
5. Test with mobile magnification tools

### Conformance Reporting

**Accessibility Conformance Report:**
- Use VPAT (Voluntary Product Accessibility Template) format
- Document conformance level for each success criterion
- List known issues and remediation plans
- Update quarterly or with major releases

**Issue Tracking:**
- Tag accessibility issues with severity:
  - **Critical**: Blocks access for users (immediate fix)
  - **High**: Significant barrier (fix in sprint)
  - **Medium**: Inconvenience (fix in release cycle)
  - **Low**: Enhancement (backlog)

**Sample Issue Template:**
```markdown
## Accessibility Issue: [Brief Description]

**WCAG Success Criterion**: [e.g., 2.4.7 Focus Visible (Level AA)]
**Severity**: [Critical/High/Medium/Low]
**Location**: [Page/Component URL or name]
**Assistive Technology**: [Screen reader, keyboard, etc.]

**Description**: [Detailed description of the issue]

**Steps to Reproduce**:
1. [Step 1]
2. [Step 2]

**Expected Behavior**: [What should happen]
**Actual Behavior**: [What currently happens]

**Proposed Solution**: [How to fix]
**Related Issues**: [Links to related issues]
```

## Content Authoring Guidelines

### Game Accessibility Requirements

All games must meet accessibility standards before publication:

**Visual Elements:**
- Color contrast ratios verified
- Text alternatives for all graphics
- Music notation includes descriptive text
- Visual feedback accompanied by audio or text cues

**Audio Elements:**
- Captions for all spoken instructions
- Visual alternatives for audio-only cues
- Volume controls accessible via keyboard
- Support for mono audio output

**Interaction:**
- Keyboard accessible controls
- Touch targets minimum 44px
- No time limits or adjustable/extendable timers
- Clear error messages and correction guidance

**Game Metadata:**
Required accessibility metadata for each game:
```typescript
{
  gameId: string,
  accessibilityFeatures: {
    keyboardAccessible: boolean,
    screenReaderCompatible: boolean,
    captionsAvailable: boolean,
    audioDescriptions: boolean,
    colorIndependent: boolean,
    reducedMotionMode: boolean,
    highContrastMode: boolean,
    midiKeyboardSupport: boolean
  },
  wcagLevel: 'A' | 'AA' | 'AAA',
  knownIssues: string[],
  remediationPlan: string
}
```

### Document Accessibility

**PDFs and Downloadable Content:**
- All PDFs must be tagged for accessibility
- Include document structure (headings, lists, tables)
- Provide alt text for images
- Ensure reading order is correct
- Test with Adobe Acrobat Accessibility Checker

**Video Content:**
- Closed captions required for all videos
- Transcripts provided
- Audio descriptions for visual-only content
- Player controls keyboard accessible
- See [Captions and Audio Descriptions](../16-accessibility-and-inclusion/captions-and-audio-descriptions.md)

## Remediation and Maintenance

### Issue Prioritization

**Priority Levels:**
1. **P0 - Critical**: Blocker for users with disabilities
   - Fix immediately, hotfix deployment
   - Examples: Inaccessible login, keyboard trap, missing form labels

2. **P1 - High**: Significant barrier
   - Fix within current sprint
   - Examples: Poor color contrast, missing alt text on key images

3. **P2 - Medium**: Inconvenience but workarounds exist
   - Fix in current release cycle
   - Examples: Inconsistent heading hierarchy, vague link text

4. **P3 - Low**: Enhancement, doesn't block access
   - Backlog, address in future release
   - Examples: Additional keyboard shortcuts, enhanced ARIA labels

### Continuous Monitoring

**Automated Monitoring:**
- Run accessibility scans on every pull request
- Nightly full-site scans
- Alert developers of new violations
- Track metrics over time

**Quarterly Audits:**
- Comprehensive manual audit every quarter
- External audit annually
- Update conformance documentation
- Review and update remediation plans

**User Feedback:**
- Provide accessibility feedback mechanism
- Email: accessibility@musiclearningcommunity.com
- Monitor support tickets for accessibility issues
- Incorporate user feedback into remediation priorities

## Training and Documentation

### Developer Training

**Required Training:**
- WCAG 2.1 overview and principles
- Semantic HTML and ARIA best practices
- Keyboard navigation patterns
- Screen reader testing basics
- Accessibility testing tools

**Ongoing Education:**
- Monthly accessibility tips and updates
- Share accessibility testing results
- Code review focus on accessibility
- Attend accessibility conferences/webinars

### Designer Training

**Required Knowledge:**
- Color contrast requirements
- Focus indicator design
- Touch target sizing
- Text spacing and readability
- Accessible form design

### Content Creator Training

**Required Skills:**
- Writing effective alt text
- Creating accessible PDFs
- Captioning video content
- Writing clear instructions
- Avoiding sensory-only references

## Supporting Documents Referenced

This WCAG 2.1 AA compliance specification draws from the following source documents:

- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser and operating system compatibility requirements for accessibility API support
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — User interface content, MIDI keyboard accessibility, and special needs support documentation

## Dependencies

### Core Dependencies
- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)** - Platform-wide accessibility strategy, data models, and UX requirements
- **[Neurodiverse Support](../16-accessibility-and-inclusion/neurodiverse-support.md)** - Cognitive accessibility features and special needs support
- **[Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md)** - Alternative input methods and keyboard navigation specifications
- **[Captions and Audio Descriptions](../16-accessibility-and-inclusion/captions-and-audio-descriptions.md)** - Multimedia accessibility requirements

### Technical Dependencies
- **[Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md)** - Browser support requirements for accessibility APIs
- **[Component Library](../01-ux-design-system/components-overview.md)** - Accessible component implementations
- **[Design System](../01-ux-design-system/responsive-breakpoints.md)** - Responsive design requirements

### Testing Dependencies
- **[Test Strategy](../19-quality-and-operations/test-strategy.md)** - Overall testing approach including accessibility testing
- **[QA Checklists](../19-quality-and-operations/qa-checklists.md)** - Quality assurance procedures

## Open Questions

### Technical Questions
1. **MIDI Keyboard Accessibility**: How to announce MIDI keyboard input to screen readers in a way that doesn't interfere with musical feedback?
2. **Music Notation**: What is the best approach for making complex music notation accessible to screen readers? MusicXML? Custom ARIA descriptions?
3. **Real-time Audio Games**: How to provide synchronized captions for games with dynamically generated audio?
4. **Touch Gestures**: What are the most effective single-pointer alternatives for music notation interaction?

### Implementation Questions
1. **Performance vs. Accessibility**: How to balance animation performance with reduced-motion accessibility requirements?
2. **Third-party Content**: How to ensure third-party embedded content (if any) meets WCAG standards?
3. **Legacy Content**: What is the prioritization for remediating accessibility issues in existing games vs. new development?
4. **Automated Testing Coverage**: What percentage of WCAG criteria can be covered by automated tests vs. manual testing?

### Policy Questions
1. **Conformance Target**: Should we target WCAG 2.2 (newer) or maintain 2.1 for broader support?
2. **AAA Criteria**: Are there specific Level AAA criteria worth implementing for competitive advantage?
3. **Certification**: Should we pursue formal WCAG certification or third-party accessibility audit?
4. **Public Reporting**: Should we publish our VPAT and accessibility conformance report publicly?

## Related Documentation

**Accessibility:**
- [Accessibility Standards](../01-ux-design-system/a11y-standards.md) - Platform accessibility strategy
- [Neurodiverse Support](../16-accessibility-and-inclusion/neurodiverse-support.md) - Cognitive and special needs accessibility
- [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md) - Input methods
- [Captions and Audio Descriptions](../16-accessibility-and-inclusion/captions-and-audio-descriptions.md) - Multimedia accessibility

**Technical:**
- [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md) - Browser support
- [Components Overview](../01-ux-design-system/components-overview.md) - Accessible components
- [Test Strategy](../19-quality-and-operations/test-strategy.md) - Testing approach
- [QA Checklists](../19-quality-and-operations/qa-checklists.md) - Quality procedures

**Legal:**
- [Privacy Policy](../23-legal-and-compliance/privacy-policy.md) - Privacy considerations
- [COPPA/FERPA Notes](../23-legal-and-compliance/coppa-ferpa-notes.md) - Compliance requirements
