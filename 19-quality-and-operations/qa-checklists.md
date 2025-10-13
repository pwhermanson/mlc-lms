# QA Checklists

## Purpose

This document provides comprehensive, actionable QA test checklists for validating the MusicLearningCommunity (MLC) LMS platform before each release. These checklists translate the overall [Test Strategy](test-strategy.md) into specific, step-by-step test cases that QA engineers execute to ensure quality across all features, user roles, browsers, and devices.

The QA checklists serve multiple purposes:
- **Pre-Release Validation**: Comprehensive test execution before production deployment
- **Smoke Testing**: Rapid validation immediately after deployment to any environment
- **Regression Testing**: Verification that existing functionality remains intact after changes
- **Feature-Specific Testing**: Detailed test scenarios for new or modified features
- **Browser/Device Validation**: Systematic testing across the [Browser Device Matrix](browser-device-matrix.md)
- **Music-Specific Validation**: Specialized tests for MIDI, audio, and notation features

> **Context**: These checklists address historical pain points from the legacy MLC system, particularly around score recording reliability (critical for student motivation), browser compatibility issues, MIDI device support, and the complexity of testing educational game flows. The checklists emphasize verification of data integrity, accessibility for special needs students, and multi-device consistency.

## Scope

### Included
- **Smoke Test Checklists**: Quick validation of critical functionality post-deployment
- **Regression Test Checklists**: Core feature validation for each release
- **Role-Based Test Scenarios**: Detailed test cases for Student, Teacher, Admin, and System Admin workflows
- **Music-Specific Test Cases**: MIDI keyboard, audio playback, notation rendering validation
- **Browser/Device Validation Checklists**: Platform-specific testing procedures
- **Integration Test Scenarios**: Payment processing, email delivery, CSV import/export validation
- **Accessibility Test Checklists**: WCAG compliance and assistive technology validation
- **Pre-Release Sign-Off Checklist**: Final validation before production deployment

### Excluded
- **Overall Testing Strategy**: Philosophy, tools, and automation priorities (see [Test Strategy](test-strategy.md))
- **Browser/Device Support Matrix**: Specific browser versions and device configurations (see [Browser Device Matrix](browser-device-matrix.md))
- **Automated Test Implementation**: Unit, integration, and E2E test code (see [Testing Requirements](testing-requirements.md))
- **Game-Specific Test Plans**: Individual game QA procedures (see [Games Audit and Health](../08-games-registry-and-authoring/games-audit-and-health.md))
- **Incident Response Procedures**: Production issue handling (see [Incident Response](incident-response.md))
- **Release Deployment Process**: Deployment steps and rollback procedures (see [Release Management](release-management.md))

## Smoke Test Checklist

**Purpose**: Rapid validation (< 15 minutes) immediately after deployment to staging or production. Focuses on critical user paths to ensure the system is fundamentally operational.

**When to Execute**: 
- Immediately after deployment to staging
- Immediately after production deployment
- After emergency hotfix deployment
- After infrastructure changes (database migration, CDN update, etc.)

### Critical Path Validation

#### Student Login and Assignment Access
- [ ] Navigate to login page - page loads without errors
- [ ] Log in as test student (username: `qa_student_01`, environment-specific password)
- [ ] Dashboard loads with assigned games visible
- [ ] Click on first assigned game - assignment play page loads
- [ ] View game stages - Learn, Play, Quiz stages display correctly
- [ ] Click on Learn stage - game loads in player shell
- [ ] Complete Learn stage - score records (check database or student progress view)
- [ ] Return to dashboard - progress indicator updates

**Expected Results**:
- All pages load within 3 seconds
- No console errors in browser DevTools
- Student can navigate full assignment flow
- Scores persist to database and display correctly

#### Teacher Login and Assignment Creation
- [ ] Navigate to login page
- [ ] Log in as test teacher (username: `qa_teacher_01`, environment-specific password)
- [ ] Dashboard loads with class overview visible
- [ ] Navigate to Assignment Builder
- [ ] Select a sequence from library (e.g., "Lifetime Musician - Level 1")
- [ ] Assign to test student (`qa_student_01`)
- [ ] Set due date (tomorrow)
- [ ] Save and publish assignment
- [ ] Verify assignment appears in teacher's assignment list
- [ ] Impersonate or verify as student - assignment appears on student dashboard

**Expected Results**:
- Assignment builder loads all sequences
- Assignment saves successfully
- Assignment immediately visible to assigned student
- No errors in console or UI

#### Admin Login and Member Management
- [ ] Navigate to login page
- [ ] Log in as test admin (username: `qa_admin_01`, environment-specific password)
- [ ] Dashboard loads with subscription status visible
- [ ] Navigate to Member Management
- [ ] View student list - students display correctly
- [ ] Click on a student - student profile opens
- [ ] View student's assignment history
- [ ] Return to dashboard

**Expected Results**:
- Admin can access all management screens
- Student data displays correctly
- No permission errors

#### Payment System Validation (Staging Only)
- [ ] Log in as admin without active subscription
- [ ] Navigate to billing/subscription page
- [ ] View available plans
- [ ] Select plan (Solo or Ensemble)
- [ ] Add payment method (use Braintree test card: 4111 1111 1111 1111)
- [ ] Complete purchase
- [ ] Verify subscription status updates to "Active"
- [ ] Verify receipt email sent (check Mailtrap or test email inbox)

**Expected Results**:
- Payment form loads without errors
- Braintree sandbox processes test payment successfully
- Subscription activates immediately
- Email sends with correct details

#### System Critical Services
- [ ] CDN asset loading - game thumbnails and assets load from Cloudflare R2
- [ ] API health endpoint - `/api/health` returns 200 OK
- [ ] Database connectivity - queries execute successfully (evidenced by data loading)
- [ ] Background jobs - check job dashboard or logs for recent completions
- [ ] Email delivery - test email sends successfully (use admin "Send Test Email" feature)

**Expected Results**:
- All external services responding
- No service degradation alerts
- Recent background jobs completed successfully

### Smoke Test Sign-Off

**Deployment ID**: _______________  
**Environment**: [ ] Staging [ ] Production  
**Tester**: _______________  
**Date/Time**: _______________  
**Result**: [ ] PASS [ ] FAIL  

**If FAIL, describe critical issues**:
_______________________________________________

**Rollback Decision**: [ ] Continue [ ] Rollback immediately

---

## Regression Test Checklist

**Purpose**: Comprehensive validation that existing functionality remains intact after code changes. Execute before each release to production.

**When to Execute**:
- Before every production release
- After major feature additions
- After framework or dependency upgrades
- After database schema changes

**Estimated Time**: 2-4 hours for full regression suite

### Student Experience Regression

#### Authentication and Profile
- [ ] Student can log in with username and password
- [ ] Student can log in with email and password (if applicable)
- [ ] "Remember Me" checkbox persists session correctly
- [ ] "Forgot Password" flow sends reset email and allows password change
- [ ] Student can log out successfully
- [ ] Session expires after configured timeout (verify with DevTools)
- [ ] Student profile page displays correct information (name, avatar, badges)
- [ ] Student can update profile settings (avatar, display name, notification preferences)
- [ ] Parent message system loads and displays messages from teacher

#### Dashboard and Navigation
- [ ] Dashboard loads with current assignments in "Up Next" section
- [ ] Completed assignments move to "Completed" section
- [ ] Badge display shows earned badges correctly
- [ ] Leaderboard displays (if student has opted in)
- [ ] Navigation menu expands/collapses correctly
- [ ] "All Games" library is accessible from navigation
- [ ] Help menu loads and displays appropriate content
- [ ] Student can navigate between dashboard, assignments, all games, profile, and settings

#### Assignment Play Flow
- [ ] Assignment play page loads with all assigned games visible
- [ ] Stage selector displays Learn, Play, Quiz, Review, Challenge stages appropriately
- [ ] Locked stages display lock icon and cannot be accessed
- [ ] Available stages can be clicked and load
- [ ] Game loads in chromeless player shell
- [ ] Game audio plays correctly
- [ ] Game responds to mouse input correctly
- [ ] Game responds to touch input correctly (on tablet/mobile)
- [ ] Game responds to keyboard input correctly
- [ ] "Pause" button pauses game and displays pause overlay
- [ ] "Quit" button returns to assignment page (with confirmation if mid-game)
- [ ] Help button opens contextual help overlay
- [ ] Game completion triggers score submission
- [ ] Score displays on result screen
- [ ] "Next Up" button advances to next required stage
- [ ] Progress indicators update in real-time
- [ ] Quiz stage requires passing score to unlock next assignment
- [ ] Review stage only unlocks after passing Quiz
- [ ] Assignment marked "Complete" when all required stages finished

#### MIDI Keyboard Integration
- [ ] Plug in MIDI keyboard - system detects device and displays "MIDI Connected" notification
- [ ] Start MIDI-required game - game prompts for MIDI if not connected
- [ ] Play notes on MIDI keyboard - game responds with visual feedback
- [ ] Game accurately detects correct notes
- [ ] Game accurately detects incorrect notes
- [ ] Game records timing of note presses
- [ ] Score calculation includes MIDI input accuracy
- [ ] Disconnect MIDI keyboard mid-game - game displays reconnection prompt
- [ ] Reconnect MIDI keyboard - game resumes normally
- [ ] Multiple MIDI devices connected - game allows device selection

#### All Games Library
- [ ] All Games page loads with game grid
- [ ] Search filter works (search by game name or concept)
- [ ] Category filter works (filter by Music Theory, Ear Training, Sight Reading, etc.)
- [ ] Difficulty filter works (filter by Beginner, Intermediate, Advanced)
- [ ] Free Play mode allows playing any unlocked game
- [ ] Free Play scores record separately from assignment scores
- [ ] Games previously completed in assignments show completion status
- [ ] Game thumbnails load from CDN correctly
- [ ] Clicking game opens in player shell (similar to assignment play)

#### Gamification Features
- [ ] Badge earned notification displays when criteria met
- [ ] Badge appears in student profile badge collection
- [ ] Point total increments correctly after game completion
- [ ] Streak counter increments for consecutive daily activity
- [ ] Streak resets if student doesn't log in for configured period
- [ ] Leaderboard updates with current scores
- [ ] Leaderboard respects student opt-in/opt-out preference
- [ ] Challenge board displays active challenges
- [ ] Challenge completion awards appropriate badges/points

### Teacher Experience Regression

#### Authentication and Dashboard
- [ ] Teacher can log in with credentials
- [ ] Teacher dashboard loads with class overview
- [ ] Student progress summary displays correctly
- [ ] Recent activity feed shows latest student completions
- [ ] Quick stats (average scores, completion rates) display accurately
- [ ] Navigation to Assignment Builder, Reports, Messaging, Settings works

#### Assignment Builder
- [ ] Assignment Builder page loads
- [ ] Sequence library displays all available curricula
- [ ] Search and filter work in sequence library
- [ ] Select sequence - game list displays for selected sequence
- [ ] Customize assignment - can add/remove games from sequence
- [ ] Customize assignment - can reorder games
- [ ] Customize assignment - can set due date with date picker
- [ ] Student selector displays all students in teacher's classes
- [ ] Can select individual students
- [ ] Can select entire class/group
- [ ] "Save as Draft" saves assignment without publishing
- [ ] "Publish" makes assignment immediately visible to students
- [ ] Published assignment appears in assignment list
- [ ] Can edit existing assignment (with warning about student progress)
- [ ] Can duplicate existing assignment
- [ ] Can delete assignment (with confirmation)
- [ ] Can archive completed assignments

#### Progress Monitoring and Reports
- [ ] Class dashboard shows all students with progress indicators
- [ ] Click on student - individual student detail page loads
- [ ] Student detail shows assignment history with scores
- [ ] Score history chart displays correctly
- [ ] Can drill down to see individual game attempts
- [ ] Can filter by date range
- [ ] Can filter by assignment
- [ ] Export to PDF generates formatted report
- [ ] Export to CSV generates spreadsheet with accurate data
- [ ] Reports respect student privacy settings
- [ ] Struggling student identification algorithm highlights students needing attention

#### Messaging
- [ ] Teacher messaging page loads
- [ ] Can compose message to individual student
- [ ] Can compose message to parent (appears in parent message tab)
- [ ] Can compose message to entire class
- [ ] Message sends successfully
- [ ] Student receives message notification
- [ ] Parent receives email notification (if configured)
- [ ] Message history displays sent messages
- [ ] Can reply to received messages
- [ ] Message search works correctly

#### Teacher Settings
- [ ] Teacher can update profile information
- [ ] Teacher can upload profile photo
- [ ] Teacher can configure notification preferences (email, in-app)
- [ ] Teacher can set default assignment settings (due dates, difficulty)
- [ ] Teacher can manage class/group definitions
- [ ] Settings save successfully and persist on next login

### Admin/Subscriber Experience Regression

#### Member Management
- [ ] Member Management page loads with student list
- [ ] Can search students by name, email, or username
- [ ] Can filter students by class, grade, or status
- [ ] Student list pagination works correctly
- [ ] Click on student - student profile opens
- [ ] Can edit student information (name, email, grade level)
- [ ] Can reset student password
- [ ] Can deactivate/reactivate student account
- [ ] Can view student's assignment history
- [ ] Can manually assign/unassign students to teachers

#### Bulk Operations
- [ ] CSV student import page loads
- [ ] Upload sample CSV file - file uploads successfully
- [ ] Import preview displays with student data
- [ ] Validation errors display clearly with line numbers
- [ ] Can correct errors and re-upload
- [ ] Confirm import - students created successfully
- [ ] Can bulk update students via CSV (update existing records)
- [ ] Can bulk assign students to classes
- [ ] Can export student list to CSV
- [ ] Export includes all selected fields

#### Billing and Subscription Management
- [ ] Subscription status page displays current plan details
- [ ] Displays seat count (total, used, available)
- [ ] Displays billing cycle and next payment date
- [ ] Can add additional seats
- [ ] Can upgrade plan (Solo to Ensemble)
- [ ] Payment method update form works
- [ ] Can view billing history
- [ ] Can download past invoices as PDF
- [ ] Promo code entry works (applies discount correctly)
- [ ] Sponsor code entry works (applies credits correctly)
- [ ] Subscription cancellation flow works (with confirmation and retention offer)
- [ ] Paused subscription shows correct status and reactivation option

#### Admin Reports
- [ ] Admin reports dashboard loads
- [ ] School-wide progress report generates correctly
- [ ] Usage report shows accurate student activity
- [ ] Teacher performance report displays correctly
- [ ] Can filter reports by date range, class, or teacher
- [ ] Can export reports to PDF and CSV
- [ ] Reports load within reasonable time (< 10 seconds for 100 students)

### System Admin Experience Regression

#### User Search and Management
- [ ] System admin dashboard loads
- [ ] User search works by email, username, or user ID
- [ ] Search results display relevant users
- [ ] Click on user - user profile loads with all details
- [ ] Can view user's subscription status
- [ ] Can view user's assignment history
- [ ] Can view user's billing history
- [ ] Can manually adjust user roles/permissions
- [ ] Can impersonate user (with audit log entry)
- [ ] Impersonation mode displays banner clearly
- [ ] Can exit impersonation mode
- [ ] Impersonation actions logged in audit trail

#### Content Management
- [ ] Game registry page loads with all games
- [ ] Can search and filter games
- [ ] Can edit game metadata
- [ ] Can change game status (Draft, Published, Archived)
- [ ] Can upload new game assets
- [ ] Can create new game entry
- [ ] Sequence library management page loads
- [ ] Can edit sequence definitions
- [ ] Can create new sequences
- [ ] Can assign games to sequences with ordering

#### Audit Logs and Monitoring
- [ ] Audit log page loads
- [ ] Can filter by user, action type, or date range
- [ ] Log entries display with timestamp, user, action, and details
- [ ] Can export audit logs to CSV
- [ ] Can view detailed log entry (JSON payload)
- [ ] Audit logs capture all security-sensitive actions (login, permission changes, impersonation)

#### Feature Flags and Configuration
- [ ] Feature flags page loads
- [ ] Can view all feature flags with current status
- [ ] Can toggle feature flag on/off
- [ ] Flag changes take effect immediately (verify in separate browser session)
- [ ] Can set percentage rollout for gradual feature release
- [ ] Can target feature flags by user role or subscription plan
- [ ] Configuration page loads with all environment variables
- [ ] Can view (but not edit) sensitive config values (masked)

### Integration and Cross-Role Regression

#### Teacher Creates Assignment → Student Completes → Teacher Views Results
- [ ] Teacher creates new assignment and assigns to test student
- [ ] Student logs in and sees new assignment on dashboard
- [ ] Student completes assignment stages
- [ ] Student's progress updates in real-time on teacher's dashboard
- [ ] Teacher views student's detailed scores and attempt history
- [ ] Scores match what student saw on completion screen
- [ ] Teacher sends congratulatory message
- [ ] Student receives message notification

#### Admin Imports Students → Assigns to Teacher → Students Can Log In
- [ ] Admin uploads CSV with new students
- [ ] Import completes successfully
- [ ] Admin assigns students to teacher's class
- [ ] Students appear in teacher's student list
- [ ] Students receive welcome email with login credentials
- [ ] Students can log in with provided credentials
- [ ] Students see teacher's assignments on their dashboard

#### Payment Processing → Seat Allocation → Student Assignment
- [ ] Admin views subscription status (0 available seats)
- [ ] Admin purchases additional 10 seats via Braintree
- [ ] Payment processes successfully
- [ ] Subscription immediately shows 10 available seats
- [ ] Admin assigns students to newly purchased seats
- [ ] Students can log in and access full platform
- [ ] Billing statement reflects new charges correctly

### Browser and Device Validation

**Note**: Full browser/device matrix testing is detailed in [Browser Device Matrix](browser-device-matrix.md). This section covers critical browser-specific validations for regression testing.

#### Desktop Browser Validation
- [ ] Chrome (latest) - All critical flows work, MIDI support functional, audio playback clear
- [ ] Firefox (latest) - All critical flows work, MIDI support functional, audio playback clear
- [ ] Safari (macOS, latest) - All critical flows work, MIDI support functional, audio playback clear
- [ ] Edge (latest) - All critical flows work, MIDI support functional, audio playback clear

#### Mobile Browser Validation
- [ ] Chrome Android (tablet, latest) - Touch input works, games render correctly, audio playback works
- [ ] Safari iOS (iPad, latest) - Touch input works, games render correctly, audio playback works
- [ ] Safari iOS (iPhone, latest) - Responsive layout adapts, core flows functional

#### Critical Browser-Specific Tests
- [ ] Safari - Web Audio API works correctly (some games use audio synthesis)
- [ ] Safari - MIDI over Bluetooth works (iOS 13+ feature)
- [ ] Firefox - MIDI device detection works (requires explicit permissions)
- [ ] Chrome Android - Touch event handling works in games
- [ ] Safari iOS - No audio autoplay restrictions (verify audio plays on user interaction)

### Regression Test Sign-Off

**Release Version**: _______________  
**Test Cycle Dates**: _______________ to _______________  
**Primary Tester**: _______________  
**Additional Testers**: _______________  

**Test Results Summary**:
- Total Test Cases: _____
- Passed: _____
- Failed: _____
- Blocked: _____
- Not Applicable: _____

**Critical Issues Found**: (List any blocking issues)
1. _______________________________________________
2. _______________________________________________
3. _______________________________________________

**Go/No-Go Decision**: [ ] GO - Approve for production [ ] NO-GO - Block release

**Sign-Off**: _______________ (Name and Date)

---

## Feature-Specific Test Scenarios

### MIDI Keyboard Testing

**Purpose**: Validate MIDI keyboard integration across browsers and devices. MIDI functionality is critical for piano learning and a key differentiator for MLC.

**Test Devices**:
- USB MIDI keyboard (e.g., Yamaha P-45, Casio Privia)
- Bluetooth MIDI keyboard (iOS/macOS support)
- MIDI controller with velocity sensitivity

#### MIDI Device Detection
- [ ] Connect USB MIDI keyboard - Browser prompts for MIDI access permission
- [ ] Grant permission - System displays "MIDI Connected" notification with device name
- [ ] Disconnect MIDI keyboard - System displays "MIDI Disconnected" notification
- [ ] Reconnect MIDI keyboard - System automatically reconnects without re-prompting
- [ ] Connect multiple MIDI devices - System allows selection of active device
- [ ] No MIDI device connected - Games requiring MIDI display "Connect MIDI Keyboard" prompt

#### MIDI Input Accuracy
- [ ] Start MIDI-enabled game
- [ ] Press C4 on MIDI keyboard - Game recognizes and responds to C4
- [ ] Press all keys from C3 to C6 - Game recognizes all pitches correctly
- [ ] Press key softly (low velocity) - Game responds with appropriate visual feedback
- [ ] Press key hard (high velocity) - Game responds with appropriate visual feedback
- [ ] Hold key for sustained note - Game recognizes note duration
- [ ] Press chord (multiple simultaneous notes) - Game recognizes all notes
- [ ] Rapid note sequence (16th notes at 120 BPM) - Game captures all notes without dropping

#### MIDI Latency and Timing
- [ ] Press key on MIDI keyboard - Visual feedback appears within 100ms
- [ ] Press key on MIDI keyboard - Audio feedback plays within 100ms (if game plays back)
- [ ] Rhythm game - Game accurately measures timing of key presses
- [ ] Timing feedback displays (early, on-time, late) based on actual note timing
- [ ] Score calculation includes timing accuracy

#### MIDI Browser Compatibility
- [ ] Chrome Windows - MIDI works via Web MIDI API
- [ ] Chrome macOS - MIDI works via Web MIDI API
- [ ] Chrome Android - MIDI over USB works (with USB-C adapter)
- [ ] Firefox Windows - MIDI works (may require explicit permission)
- [ ] Firefox macOS - MIDI works (may require explicit permission)
- [ ] Safari macOS - MIDI works via Web MIDI API
- [ ] Safari iOS - MIDI over Bluetooth works (iOS 13+)
- [ ] Edge Windows - MIDI works via Web MIDI API

#### MIDI Error Handling
- [ ] Disconnect MIDI keyboard during game - Game pauses and displays reconnection prompt
- [ ] Reconnect MIDI keyboard - Game resumes without data loss
- [ ] Browser denies MIDI permission - Game displays helpful error message with instructions
- [ ] MIDI device malfunction (unplug cable partially) - Game handles gracefully without crashing

### Audio Playback Testing

**Purpose**: Validate audio playback quality and consistency across browsers and devices. Audio is essential for music education games.

#### Audio Quality and Playback
- [ ] Start game with audio - Audio plays clearly without distortion
- [ ] Audio volume control - Slider adjusts volume from 0% to 100%
- [ ] Mute button - Mutes all audio immediately
- [ ] Background music - Plays consistently during game
- [ ] Sound effects - Play correctly on game events (correct answer, wrong answer, completion)
- [ ] Multiple simultaneous sounds - Game can play background music + sound effects without clipping
- [ ] Audio sprite loading - Audio assets load from CDN without delays
- [ ] Audio preloading - Audio ready before game starts (no delay on first sound)

#### Audio Browser Compatibility
- [ ] Chrome - Audio plays immediately without user interaction requirement (if policy allows)
- [ ] Firefox - Audio plays immediately without user interaction requirement (if policy allows)
- [ ] Safari desktop - Audio requires user interaction to start (verify prompt displays)
- [ ] Safari iOS - Audio requires user interaction to start (verify prompt displays)
- [ ] Chrome Android - Audio plays clearly without distortion
- [ ] Audio format support - Games use compatible formats (MP3, WebM, AAC) for all browsers

#### Audio Recording (Aural Games)
- [ ] Start game requiring microphone - Browser prompts for microphone access
- [ ] Grant permission - Game displays "Microphone Ready" indicator
- [ ] Sing or play note - Game captures audio and analyzes pitch
- [ ] Game provides feedback on pitch accuracy (in tune, sharp, flat)
- [ ] Recording stops cleanly on game completion
- [ ] Deny microphone permission - Game displays helpful error message

### Responsive Design and Mobile Testing

**Purpose**: Validate responsive layouts and touch interactions on mobile and tablet devices. Ensures consistent experience across device sizes.

**Test Devices**:
- iPhone 13/14 (iOS, portrait and landscape)
- iPad Air/Pro (iOS, portrait and landscape)
- Samsung Galaxy S21/S22 (Android, portrait and landscape)
- Samsung Galaxy Tab (Android, tablet)

#### Responsive Layout Validation
- [ ] Dashboard - Layouts adapt correctly at breakpoints (320px, 768px, 1024px, 1440px)
- [ ] Navigation menu - Collapses to hamburger menu on mobile
- [ ] Assignment play page - Stage selector stacks vertically on mobile
- [ ] Game player - Scales to fit viewport without horizontal scrolling
- [ ] Teacher reports - Tables scroll horizontally or reflow on narrow screens
- [ ] Admin member management - Data tables responsive or scrollable
- [ ] Login/signup forms - Inputs and buttons sized appropriately for touch

#### Touch Interaction Validation
- [ ] Tap buttons - All buttons respond to tap events (visual feedback)
- [ ] Swipe gestures - Supported where appropriate (image galleries, card stacks)
- [ ] Pinch to zoom - Disabled on game canvas (prevents accidental zoom during play)
- [ ] Drag and drop - Works with touch input (e.g., drag music notes in notation games)
- [ ] Long press - Triggers context menus where applicable
- [ ] Double tap - No unintended zoom behavior on game elements

#### Mobile-Specific Features
- [ ] Portrait orientation - Layouts work correctly in portrait mode
- [ ] Landscape orientation - Layouts work correctly in landscape mode (especially for game play)
- [ ] Orientation change - Game pauses on orientation change, resumes correctly
- [ ] iOS safe area - Content respects notch and home indicator on iPhone X+
- [ ] Android back button - Returns to previous page appropriately
- [ ] iOS swipe back gesture - Returns to previous page appropriately

### Accessibility Testing

**Purpose**: Validate WCAG 2.1 AA compliance and assistive technology support. Ensures platform is usable by students with special needs, particularly those on the autism spectrum (a key user group for MLC).

#### Keyboard Navigation
- [ ] Tab navigation - All interactive elements reachable via Tab key
- [ ] Tab order - Focus order follows logical reading order
- [ ] Shift+Tab - Reverse tab order works correctly
- [ ] Enter/Space - Activates buttons and links
- [ ] Escape - Closes modals and overlays
- [ ] Arrow keys - Navigate within menus, lists, and game controls where appropriate
- [ ] Skip to content link - Allows skipping navigation and going directly to main content
- [ ] Focus visible - Clear focus indicator on all interactive elements

#### Screen Reader Support
- [ ] VoiceOver (macOS/iOS) - All content and controls announced correctly
- [ ] NVDA (Windows) - All content and controls announced correctly
- [ ] JAWS (Windows) - All content and controls announced correctly
- [ ] ARIA labels - Form inputs, buttons, and regions have appropriate labels
- [ ] ARIA live regions - Dynamic content changes announced (e.g., score updates, notifications)
- [ ] Heading structure - Proper heading hierarchy (H1 → H2 → H3)
- [ ] Alt text - All images have descriptive alt text
- [ ] Form validation errors - Announced by screen readers and linked to fields

#### Visual Accessibility
- [ ] Color contrast - Text meets WCAG AA contrast ratios (4.5:1 for normal text, 3:1 for large text)
- [ ] Color contrast - Interactive elements meet WCAG AA contrast ratios
- [ ] Color not sole indicator - Information conveyed with color also conveyed with text/icons
- [ ] Font size - Text scales correctly with browser zoom (up to 200%)
- [ ] High contrast mode - Site remains usable in Windows High Contrast Mode
- [ ] Focus indicators - Visible on all interactive elements (not removed by CSS)
- [ ] Motion - Animations respect prefers-reduced-motion setting

#### Cognitive Accessibility (Critical for MLC)
- [ ] Simple language - Instructions use age-appropriate, clear language
- [ ] Consistent navigation - Navigation structure consistent across pages
- [ ] Clear error messages - Errors explain what went wrong and how to fix
- [ ] Visual feedback - All actions provide clear visual confirmation
- [ ] Non-threatening design - Games use calm colors, friendly characters, positive reinforcement
- [ ] No time pressure - Games don't impose strict time limits (or allow disabling timers)
- [ ] Progress indicators - Students always know where they are and what's next

### Payment and Billing Testing

**Purpose**: Validate payment processing integration with Braintree across different plans, payment methods, and edge cases. Payment reliability is critical for business operations.

**Environment**: Always test payments in **Staging** with Braintree sandbox mode. Never use real payment information.

#### Subscription Purchase Flow
- [ ] View pricing page - All plans display correctly (Solo, Ensemble, Institution)
- [ ] Select Solo plan (1 student) - Checkout page loads
- [ ] Enter Braintree test card (4111 1111 1111 1111, exp 12/25, CVV 123)
- [ ] Enter billing address
- [ ] Apply promo code (if applicable) - Discount applies correctly
- [ ] Submit payment - Payment processes successfully
- [ ] Verify subscription status changes to "Active"
- [ ] Verify seat count reflects purchased plan
- [ ] Verify receipt email sent to admin email
- [ ] Verify billing statement includes transaction

#### Plan Upgrades and Downgrades
- [ ] Current plan: Solo (1 student) - Upgrade to Ensemble (5 students)
- [ ] Verify prorated charge calculated correctly
- [ ] Process upgrade - Subscription updates immediately
- [ ] Verify seat count increases to 5
- [ ] Downgrade from Ensemble to Solo (with appropriate warnings)
- [ ] Verify proration credit applied
- [ ] Verify students currently assigned to seats (warnings if > new seat count)

#### Additional Seat Purchase
- [ ] View subscription status - Shows seat usage (e.g., 5 used / 5 total)
- [ ] Click "Add Seats"
- [ ] Select quantity (e.g., add 3 seats)
- [ ] Process payment
- [ ] Verify seat count updates immediately (5 → 8 total)
- [ ] Verify billing statement includes seat purchase
- [ ] Verify prorated charge for partial billing cycle

#### Payment Methods
- [ ] Credit card - Visa (4111 1111 1111 1111)
- [ ] Credit card - Mastercard (5555 5555 5555 4444)
- [ ] Credit card - Discover (6011 1111 1111 1117)
- [ ] Credit card - American Express (3782 822463 10005)
- [ ] PayPal - Connect PayPal account and process payment (sandbox)
- [ ] Bank account - ACH payment (if supported, sandbox)

#### Payment Error Handling
- [ ] Declined card (Braintree test: 4000 0000 0000 0002) - Error message displays clearly
- [ ] Expired card - Validation error displays
- [ ] Invalid CVV - Validation error displays
- [ ] Network timeout - User-friendly error message with retry option
- [ ] Duplicate transaction prevention - Verify double-submit doesn't charge twice

#### Billing Management
- [ ] View billing history - All transactions display with dates and amounts
- [ ] Download invoice as PDF - PDF generates with correct formatting
- [ ] Update payment method - New card saves successfully
- [ ] Remove payment method - Old card removed (if no active subscription)
- [ ] Auto-renewal - Verify subscription renews automatically at end of billing cycle (check logs/scheduled jobs)
- [ ] Failed auto-renewal - Verify retry logic and notification emails

#### Promo Codes and Sponsor Codes
- [ ] Apply valid promo code (e.g., "SUMMER2025") - Discount applies (e.g., 20% off)
- [ ] Apply expired promo code - Error message displays
- [ ] Apply invalid promo code - Error message displays
- [ ] Apply sponsor code (e.g., "TEACHER-JOHN") - Credits apply to account
- [ ] Verify promo/sponsor code usage tracked in admin dashboard
- [ ] Verify promo code applies to first payment only (if one-time code)
- [ ] Verify ongoing promo code applies to all renewals (if recurring code)

### CSV Import/Export Testing

**Purpose**: Validate bulk operations via CSV upload/download for students, teachers, courses, and games. CSV operations are critical for school deployments with hundreds of students.

#### Student CSV Import
- [ ] Download sample student CSV template
- [ ] Prepare test CSV with 10 students (valid data)
- [ ] Upload CSV to Member Management → Import Students
- [ ] Verify preview displays all 10 students correctly
- [ ] Confirm import - All students created successfully
- [ ] Verify students appear in student list
- [ ] Verify welcome emails sent (if configured)
- [ ] Verify students can log in with generated credentials

#### Student CSV Import - Error Handling
- [ ] Upload CSV with missing required field (e.g., email) - Validation error on specific line
- [ ] Upload CSV with duplicate email - Error message indicates duplicate
- [ ] Upload CSV with invalid email format - Validation error displays
- [ ] Upload CSV with 500 students - Import completes within 30 seconds
- [ ] Upload CSV with incorrect encoding (UTF-8 required) - Displays encoding error or converts correctly
- [ ] Upload CSV with extra columns - Import ignores extra columns or handles gracefully

#### Student CSV Update (Bulk Edit)
- [ ] Download existing student list as CSV
- [ ] Edit student data (e.g., change grade levels)
- [ ] Upload modified CSV with "Update Existing" option
- [ ] Verify preview shows changes
- [ ] Confirm update - Student records updated successfully
- [ ] Verify updates reflected in student profiles

#### Teacher and Admin CSV Import
- [ ] Upload CSV with teacher accounts
- [ ] Verify teachers created with correct role permissions
- [ ] Verify teachers can log in
- [ ] Upload CSV with admin accounts
- [ ] Verify admins created with correct permissions

#### Course and Assignment CSV Import
- [ ] Upload CSV defining courses
- [ ] Verify courses created with correct metadata
- [ ] Upload CSV defining assignments (linked to courses and students)
- [ ] Verify assignments created and assigned correctly
- [ ] Students see assigned courses/assignments on dashboard

#### CSV Export
- [ ] Export student list to CSV - File downloads with correct data
- [ ] Open exported CSV in Excel/Google Sheets - Data displays correctly
- [ ] Export assignment report to CSV - File includes all expected columns
- [ ] Export billing data to CSV - File includes transactions and calculations
- [ ] Verify exported CSV encoding is UTF-8 (supports international characters)
- [ ] Verify exported CSV includes headers row

---

## Pre-Release Validation Checklist

**Purpose**: Final comprehensive validation before production deployment. This checklist must be completed and signed off before any production release.

**Release Version**: _______________  
**Scheduled Deployment Date**: _______________  
**Release Manager**: _______________  

### Code and Build Validation
- [ ] All automated tests passing in CI/CD pipeline
- [ ] Code review completed for all changes in this release
- [ ] Release notes drafted and reviewed
- [ ] Database migration scripts reviewed and tested in staging
- [ ] Environment variables and secrets configured for production
- [ ] Feature flags configured correctly for production rollout
- [ ] Build artifacts generated and tagged with release version
- [ ] Security scan completed (no critical or high vulnerabilities)

### Staging Environment Validation
- [ ] Staging environment updated to release candidate build
- [ ] Database migration executed successfully in staging
- [ ] Smoke test checklist completed in staging (100% pass)
- [ ] Regression test checklist completed in staging (>95% pass, no critical failures)
- [ ] Browser/device matrix validation completed (all priority 1 combinations tested)
- [ ] Performance testing completed (load times, API response times within SLA)
- [ ] Security testing completed (authentication, authorization, data encryption)

### User Acceptance Testing (UAT)
- [ ] UAT environment provided to stakeholders (or staging used for UAT)
- [ ] Key stakeholders tested critical flows (Product Owner, Lead Teacher, Admin user)
- [ ] UAT feedback collected and reviewed
- [ ] Critical UAT issues resolved
- [ ] Minor UAT issues documented and prioritized for future releases
- [ ] UAT sign-off received from Product Owner

### Documentation and Communication
- [ ] User-facing documentation updated (if applicable)
- [ ] Internal documentation updated (runbooks, troubleshooting guides)
- [ ] Support team briefed on new features and changes
- [ ] Customer communication drafted (if user-facing changes)
- [ ] Downtime notification sent (if maintenance window required)
- [ ] Rollback plan documented and reviewed

### Production Readiness
- [ ] Production database backup completed within last 24 hours
- [ ] Monitoring and alerting configured for new features
- [ ] Error tracking configured (Sentry or equivalent)
- [ ] CDN cache invalidation plan ready (if assets changed)
- [ ] On-call engineer assigned and available
- [ ] Deployment time scheduled (prefer low-traffic periods)

### Deployment Validation
- [ ] Deployment executed according to release procedure
- [ ] Database migration completed successfully (if applicable)
- [ ] Application deployment verified (version number, build timestamp)
- [ ] Smoke test checklist executed in production (100% pass)
- [ ] Critical user flows validated in production
- [ ] Monitoring dashboards checked (no error spikes)
- [ ] Error tracking checked (no new critical errors)
- [ ] CDN asset loading validated
- [ ] Email delivery validated (test email sent)

### Post-Deployment Monitoring
- [ ] Monitor error rates for first 1 hour (< 0.1% error rate)
- [ ] Monitor API response times for first 1 hour (< 500ms p95)
- [ ] Monitor user activity (no drop in login rate or game completions)
- [ ] Check support channels (no spike in support tickets)
- [ ] Monitor payment processing (if payment-related changes)
- [ ] Verify background jobs running correctly

### Sign-Off

**Release Manager**: _______________ (Signature and Date)  
**Product Owner**: _______________ (Signature and Date)  
**QA Lead**: _______________ (Signature and Date)  
**Engineering Lead**: _______________ (Signature and Date)  

**Release Status**: [ ] APPROVED FOR PRODUCTION [ ] POSTPONED (see issues)

**Rollback Decision** (if issues found post-deployment):  
[ ] Continue monitoring (minor issues)  
[ ] Hotfix required (critical issues, fast fix available)  
[ ] Rollback to previous version (critical issues, no fast fix)

---

---

## Supporting Documents Referenced

This QA checklists specification draws from the following source documents:

- [Browser Chart.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Browser%20Chart.xlsx%20-%20Sheet1.csv) — Browser compatibility requirements informing cross-browser validation checklists
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game implementation specifications informing MIDI keyboard testing, audio playback validation, and game flow checklists
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — User role definitions informing role-based test scenarios and permission validation checklists for each user type
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — System administrator functionality informing admin workflow checklists and feature validation procedures
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Subscription and seat management specifications informing billing, payment, and subscription checklist scenarios
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Public page content specifications informing content display validation and accessibility checklists

---

## Dependencies

This document relies on and references:
- [Test Strategy](test-strategy.md) - Overall testing approach, tools, and automation priorities
- [Browser Device Matrix](browser-device-matrix.md) - Detailed browser/OS/device support matrix
- [Testing Requirements](testing-requirements.md) - Automated test infrastructure and standards
- [Games Audit and Health](../08-games-registry-and-authoring/games-audit-and-health.md) - Game-specific QA procedures
- [Event Model](../15-analytics-and-reporting/event-model.md) - Analytics tracking validation
- [Roles Matrix](../02-roles-and-permissions/roles-matrix.md) - Permission validation by role
- [Assignment Play](../03-student-experience/assignment-play.md) - Student play flow specifications
- [Assignment Builder](../04-teacher-experience/assignment-builder.md) - Teacher assignment creation flow

## Open Questions

- **Automated Checklist Execution**: Should we build a web-based QA checklist tool that allows testers to check off items digitally and automatically generates test reports?
- **Visual Regression Testing**: Should we integrate Percy or Chromatic for automated visual regression testing, and which screens/components should be included?
- **Performance Benchmarks**: What are the specific performance SLA thresholds (page load time, API response time) that must be validated in each release?
- **Accessibility Testing Frequency**: Should full accessibility testing be conducted for every release, or only when UI changes are made?
- **Mobile Device Lab**: Should we invest in a physical device lab (BrowserStack Device Lab) or continue with cloud-based testing (BrowserStack, Sauce Labs)?
