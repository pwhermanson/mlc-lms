# Captions and Audio Descriptions

This document will define requirements for captions and audio descriptions in multimedia content.

## Purpose

This specification defines the technical implementation, content authoring workflows, and delivery mechanisms for captions and audio descriptions across all multimedia content in the MLC-LMS platform. The spec ensures that students who are deaf, hard of hearing, blind, or have low vision can fully participate in music learning experiences by providing text alternatives for audio content and audio alternatives for visual-only content.

Captions and audio descriptions enable:
- **Equal access to educational content** - Students with hearing or vision impairments can access the same learning materials as their peers
- **WCAG 2.1 AA compliance** - Meeting Level A requirements for captions (1.2.2) and audio descriptions (1.2.3, 1.2.5)
- **Multi-modal learning support** - Students can engage with content through multiple sensory channels simultaneously
- **Improved comprehension** - Research shows captions benefit all learners, including English language learners and students with processing differences
- **Quiet environment learning** - Students can learn without sound in libraries, shared spaces, or situations where audio is not available
- **Search and indexing** - Captioned content becomes searchable and indexable for improved content discovery

> **Related Documents**: For broader accessibility standards and WCAG compliance, see [Accessibility Standards](../01-ux-design-system/a11y-standards.md) and [WCAG Compliance](wcag-compliance.md). For neurodiverse support features including sensory preferences, see [Neurodiverse Support](neurodiverse-support.md). For game asset requirements and delivery, see [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md).

## Scope

### Included

- **Caption formats and standards** - WebVTT, SRT, and TTML support for maximum compatibility
- **Caption types** - Closed captions (can be toggled), open captions (always visible), subtitles for language translation
- **Audio description types** - Standard audio descriptions, extended audio descriptions for complex visual content
- **Content coverage** - Game Learn stages, instructional videos, teacher training materials, system announcements
- **Caption authoring workflow** - Tools and processes for creating, editing, and synchronizing captions
- **Audio description authoring** - Guidelines and workflow for creating descriptive audio tracks
- **Storage and versioning** - Caption file storage alongside media assets with version control
- **Delivery and synchronization** - Technical implementation for loading and displaying captions in sync with media
- **User preferences** - Caption display settings (size, color, position, background) saved to user profile
- **Quality assurance** - Validation rules for caption timing, spelling, and synchronization accuracy
- **Telemetry** - Usage tracking for caption and audio description features
- **Localization** - Multi-language caption support for international users

### Excluded

- **Screen reader support** - General screen reader compatibility covered in [Accessibility Standards](../01-ux-design-system/a11y-standards.md)
- **Visual accessibility features** - High contrast, color blindness support, and text scaling covered in [WCAG Compliance](wcag-compliance.md)
- **Game audio asset creation** - General audio production workflow covered in [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md)
- **MIDI and keyboard input accessibility** - Covered in [Keyboard and MIDI Access](keyboard-and-midi-access.md)
- **Live captioning** - Real-time captioning for live events (future consideration, not in initial scope)
- **Automatic speech recognition (ASR)** - AI-generated captions (future enhancement, initial captions are human-authored)

## Data model (if applicable)

### Caption Track Entity

Captions are stored as separate assets linked to media files, allowing multiple caption tracks per media item for different languages and purposes.

```typescript
interface CaptionTrack {
  caption_track_id: string;              // Unique identifier: "CAP-G01542-learn-en"
  media_asset_id: string;                // Reference to parent media (game, video, audio)
  media_type: 'game_stage' | 'video' | 'audio' | 'announcement';
  game_id?: string;                      // If media_type is game_stage
  stage_id?: string;                     // Specific stage this caption belongs to
  
  // Caption track metadata
  track_type: 'captions' | 'subtitles' | 'descriptions' | 'chapters';
  language: string;                      // ISO 639-1 code: "en", "es", "fr"
  label: string;                         // Display name: "English", "Español (México)"
  is_default: boolean;                   // Default track for this language
  
  // File storage
  file_format: 'vtt' | 'srt' | 'ttml';   // Caption file format
  file_url: string;                      // CDN URL to caption file
  file_size_bytes: number;               // File size for bandwidth estimation
  
  // Versioning and status
  version: number;                       // Track version, increments on edits
  status: 'draft' | 'review' | 'approved' | 'published' | 'deprecated';
  replaced_by_version?: number;          // Points to newer version if deprecated
  
  // Quality metadata
  caption_count: number;                 // Total number of caption cues
  duration_seconds: number;              // Total duration covered
  completion_percentage: number;         // 100% if entire media is captioned
  auto_generated: boolean;               // True if ASR-generated (future)
  human_reviewed: boolean;               // True if reviewed by human editor
  
  // Accessibility compliance
  wcag_level: 'A' | 'AA' | 'AAA';        // Compliance level achieved
  includes_sound_descriptions: boolean;  // True if non-speech sounds described
  includes_speaker_identification: boolean; // True if speakers identified
  
  // Authoring metadata
  created_by: string;                    // User who created track
  created_at: Date;
  last_modified_by: string;
  last_modified_at: Date;
  reviewed_by?: string;
  reviewed_at?: Date;
}
```

### Audio Description Track Entity

Audio descriptions provide narration of visual content for blind or low-vision users.

```typescript
interface AudioDescriptionTrack {
  audio_description_id: string;          // Unique identifier: "AD-G01542-learn-en"
  media_asset_id: string;                // Reference to parent media
  media_type: 'game_stage' | 'video';
  game_id?: string;
  stage_id?: string;
  
  // Track metadata
  description_type: 'standard' | 'extended'; // Standard fits in pauses, extended may pause media
  language: string;                      // ISO 639-1 code
  label: string;                         // Display name
  is_default: boolean;
  
  // File storage
  audio_file_url: string;                // CDN URL to audio description file
  audio_format: 'mp3' | 'ogg' | 'webm';  // Audio format
  file_size_bytes: number;
  
  // Timing and synchronization
  cue_points: AudioDescriptionCue[];     // Array of timed cues
  requires_media_pause: boolean;         // True for extended descriptions
  
  // Versioning and status
  version: number;
  status: 'draft' | 'review' | 'approved' | 'published' | 'deprecated';
  
  // Quality metadata
  wcag_level: 'A' | 'AA' | 'AAA';
  covers_essential_visual_info: boolean;
  narrator_voice_actor: string;          // For consistency across tracks
  
  // Authoring metadata
  created_by: string;
  created_at: Date;
  last_modified_by: string;
  last_modified_at: Date;
}

interface AudioDescriptionCue {
  cue_id: string;
  start_time_seconds: number;            // When description begins
  end_time_seconds: number;              // When description ends
  description_text: string;              // Script for narrator
  is_essential: boolean;                 // True if describes essential visual content
}
```

### User Caption Preferences

User preferences for caption display are stored in the user's accessibility profile (see [Accessibility Standards](../01-ux-design-system/a11y-standards.md) for complete profile structure).

```typescript
interface CaptionDisplayPreferences {
  userId: string;
  
  // Enable/disable
  captionsEnabled: boolean;              // Master toggle for captions
  audioDescriptionsEnabled: boolean;     // Master toggle for audio descriptions
  preferredCaptionLanguage: string;      // ISO 639-1 code, falls back to UI language
  
  // Visual styling
  fontSize: 'small' | 'medium' | 'large' | 'extra-large'; // Caption text size
  fontFamily: 'default' | 'sans-serif' | 'monospace' | 'serif';
  textColor: string;                     // Hex color code
  backgroundColor: string;               // Hex color with alpha for transparency
  backgroundOpacity: number;             // 0-100, percentage
  edgeStyle: 'none' | 'drop-shadow' | 'raised' | 'depressed' | 'outline';
  
  // Positioning
  position: 'bottom' | 'top' | 'custom';
  customPositionY?: number;              // Percentage from top if position is 'custom'
  
  // Behavior
  autoShow: boolean;                     // Auto-enable captions when audio is muted
  persistentDisplay: boolean;            // Keep captions visible even when no audio
  
  // Saved at
  lastUpdated: Date;
}
```

### Relationships

- **CaptionTrack** belongs to one **Media Asset** (game stage, video, or audio)
- **AudioDescriptionTrack** belongs to one **Media Asset**
- **Media Asset** can have multiple **CaptionTracks** (one per language)
- **Media Asset** can have multiple **AudioDescriptionTracks** (standard and extended, per language)
- **User** has one **CaptionDisplayPreferences** profile
- **Game** stages reference **CaptionTracks** via `stage_id` (see [Game Model](../08-games-registry-and-authoring/game-model.md))

## Behavior and rules

### Caption Content Requirements

**WCAG 2.1 Level A Compliance (Minimum):**
- All pre-recorded audio content MUST have synchronized captions (Success Criterion 1.2.2)
- Captions MUST include all spoken dialogue
- Captions MUST identify speakers when multiple people speak
- Captions MUST describe significant sound effects (e.g., "[piano plays C major scale]", "[correct answer chime]", "[applause]")
- Captions MUST use proper spelling and grammar
- Captions MUST be synchronized within 100ms of audio timing

**WCAG 2.1 Level AA Compliance (Target):**
- All pre-recorded video content MUST have audio descriptions OR all information is already provided in audio track (Success Criterion 1.2.5)
- Audio descriptions MUST describe essential visual information not available in audio
- Extended audio descriptions MUST be provided if pauses in foreground audio are insufficient

**MLC-Specific Caption Standards:**

1. **Musical notation descriptions:**
   - Describe musical elements shown on screen: "[treble clef appears]", "[quarter note on line D]"
   - Use standard musical terminology appropriate to student level
   - Balance between too much detail (overwhelming) and too little (insufficient)

2. **Sound effect descriptions:**
   - Game feedback sounds: "[correct]", "[incorrect]", "[level complete fanfare]"
   - Musical sounds: "[piano plays middle C]", "[ascending C major scale]"
   - Environmental sounds: "[applause]", "[Terry Treble chuckles]"

3. **Speaker identification:**
   - Use consistent naming: "Terry Treble:", "Instructor:", "Student:"
   - Identify speaker on first caption and when speaker changes
   - Use color coding or positioning if multiple simultaneous speakers (advanced feature)

4. **Timing and pacing:**
   - Maximum reading speed: 20 characters per second (160 words per minute)
   - Minimum caption duration: 1 second
   - Maximum caption duration: 7 seconds (split longer captions)
   - Gap between captions: Minimum 0.5 seconds for readability

5. **Text formatting:**
   - Maximum 2 lines per caption for readability
   - Maximum 42 characters per line
   - Use sentence case (not ALL CAPS unless shouting/emphasis)
   - Use italics for emphasis or non-speech sounds: *[music playing]*
   - Use brackets for sound descriptions: [door closes]
   - Use parentheses for speaker identification: (Terry Treble)

### Audio Description Requirements

**When Audio Descriptions Are Required:**
- Video content where visual information is essential to understanding and not conveyed in audio
- Game Learn stages that use visual demonstrations without verbal explanation
- Animated instructional content showing musical notation or keyboard fingering
- Silent game stages where visual-only feedback is provided

**When Audio Descriptions Are NOT Required:**
- Audio-only content (no visual component)
- Video where all visual information is already described in the audio track
- Games with fully narrated instructions that describe all visual elements
- Content where visual elements are purely decorative

**Audio Description Content Standards:**

1. **Describe essential visual content only:**
   - Focus on information necessary for understanding
   - Describe actions, settings, gestures, and expressions
   - Avoid redundancy with existing audio narration

2. **Concise and objective:**
   - Use clear, simple language
   - Avoid interpretation or editorial comments
   - Describe what is seen, not what it might mean

3. **Musical context:**
   - Describe musical notation: "A treble clef with four quarter notes ascending from C to F"
   - Describe hand positions: "Right hand positioned over middle C, fingers curved"
   - Describe visual feedback: "Green checkmark appears, indicating correct answer"

4. **Timing:**
   - **Standard descriptions**: Fit within natural pauses in audio
   - **Extended descriptions**: Pause media playback if needed for longer descriptions
   - Synchronize with visual events (appear/disappear of elements)

### Caption File Format Standards

The platform supports three caption file formats, with WebVTT as the preferred standard:

**1. WebVTT (Web Video Text Tracks) - Preferred**
- W3C standard, best browser support
- Supports styling, positioning, and metadata
- File extension: `.vtt`
- MIME type: `text/vtt`

Example:
```
WEBVTT

00:00:00.000 --> 00:00:03.500
Terry Treble: Welcome to the Grand Staff!

00:00:04.000 --> 00:00:07.800
Today we'll learn the five guide notes.
[piano plays middle C]

00:00:08.200 --> 00:00:11.500
These special notes help you navigate
the entire staff.
```

**2. SRT (SubRip Text) - Legacy Support**
- Widely supported, simple format
- No styling or positioning support
- File extension: `.srt`
- MIME type: `application/x-subrip`

**3. TTML (Timed Text Markup Language) - Advanced**
- XML-based, extensive styling capabilities
- Used for broadcast standards compliance
- File extension: `.ttml`
- MIME type: `application/ttml+xml`

### Caption Authoring Workflow

**Stage 1: Content Creation**
1. Media asset is uploaded (game audio, instructional video)
2. System detects if media requires captions (all audio content)
3. Game or video is marked as "draft" status until captions added
4. Content creator receives notification: "Captions required for publish"

**Stage 2: Caption Creation**
1. Caption editor opens media in authoring tool
2. Two caption creation methods available:
   - **Manual authoring**: Editor listens and types captions with timing
   - **Upload existing**: Editor uploads pre-created .vtt or .srt file
3. Caption authoring tool provides:
   - Waveform visualization for timing accuracy
   - Playback controls (play, pause, rewind, slow-motion)
   - Character count indicators (warn at 42 chars per line)
   - Reading speed calculator (warn if > 20 chars/second)
   - Sound description templates for common game sounds

**Stage 3: Quality Review**
1. Caption track status changes to "review"
2. Reviewer watches media with captions enabled
3. Reviewer checks:
   - Timing accuracy (captions sync with audio within 100ms)
   - Content accuracy (all dialogue and sounds described)
   - Spelling and grammar
   - Reading speed and pacing
   - Proper speaker identification
   - Sound effect descriptions present
4. Reviewer either approves or requests revisions

**Stage 4: Publish**
1. Approved caption track status changes to "published"
2. Caption file uploaded to CDN alongside media asset
3. Game or video status can now be changed to "live"
4. Caption becomes available to all users in the specified language

**Stage 5: Updates and Versioning**
1. If media asset is updated, caption version increments
2. Old caption version marked "deprecated" with reference to new version
3. System alerts editors if caption timing may be affected by media changes

### Caption Display Behavior

**User-Initiated Display:**
- Caption toggle button visible on all media players
- Keyboard shortcut: `C` key toggles captions on/off
- User preference persists across sessions (stored in user profile)
- Caption settings accessible via media player menu or global accessibility settings

**Auto-Enable Scenarios:**
- Captions auto-enable if user has set `captionsEnabled: true` in preferences
- Captions auto-enable if system audio is muted and `autoShow: true` in preferences
- Captions auto-enable if screen reader detected and no preference set (accessibility best practice)

**Caption Rendering:**
- Captions rendered as HTML overlay on video player or game stage
- Default position: bottom-center, 10% from bottom edge
- Default styling: White text with semi-transparent black background
- User custom styling applied if preferences saved
- Captions never obscure essential game controls or interactive elements
- If caption would overlap interactive element, auto-reposition to top or side

**Caption Synchronization:**
- Caption cues loaded from .vtt file before media starts
- Media player `timeupdate` event triggers caption display at correct timing
- If network latency causes media buffering, captions pause until media resumes
- Caption display accounts for playback speed changes (2x speed, slow-motion)

### Audio Description Behavior

**User-Initiated Enable:**
- Audio description toggle in media player settings
- Keyboard shortcut: `D` key toggles audio descriptions on/off
- User preference persists across sessions

**Standard Audio Description Playback:**
- Audio description track loaded as separate audio element
- Description cues synchronized with main media timeline
- Description audio plays during natural pauses in main audio
- Description audio mixed with main audio at appropriate volume levels

**Extended Audio Description Playback:**
- Media playback pauses when extended description cue begins
- Description audio plays in full
- Media playback resumes when description completes
- Visual indicator shows "Audio description playing" during pause
- User can skip extended description with keyboard shortcut

**Volume and Mixing:**
- Audio description volume adjustable independently from main audio
- Default: Description at 100%, main audio ducks to 70% during description
- User can adjust mix levels in audio settings
- Description audio uses center channel for spatial separation if supported

## UX requirements (if applicable)

### Media Player Caption Controls

All media players (game Learn stages, instructional videos, system announcements) must implement consistent caption controls:

**Caption Toggle Button:**
- Icon: "CC" text badge or closed caption symbol (📑)
- Position: Bottom-right corner of media player controls
- States:
  - Off: Gray/inactive, outline icon
  - On: Accent color (blue), filled icon
  - Available: Visible and clickable
  - Unavailable: Hidden or disabled if no captions available
- Tooltip: "Captions (C)" when hovering
- Keyboard: `C` key toggles, `Tab` navigates to button

**Caption Settings Menu:**
- Access: Click gear icon in media controls, select "Caption Settings"
- Options displayed:
  - Language selection (if multiple caption tracks available)
  - Font size (small, medium, large, extra-large)
  - Text color picker
  - Background color picker with opacity slider
  - Edge style dropdown (none, drop shadow, outline, raised, depressed)
  - Position (top, bottom, custom with slider)
  - Preview area showing sample caption with current settings
- Buttons:
  - "Apply" - Saves settings and closes menu
  - "Reset to Defaults" - Restores default styling
  - "Cancel" - Closes menu without saving

**Audio Description Controls:**
- Audio description toggle button next to caption button
- Icon: "AD" text badge or audio description symbol
- Same interaction patterns as caption button
- Additional option in settings: "Pause for extended descriptions" checkbox

### Caption Display Area

**Visual Design:**
- Default style:
  - Font: System sans-serif, 18px (medium)
  - Text color: White (#FFFFFF)
  - Background: Black with 80% opacity (#000000CC)
  - Edge style: Drop shadow (1px black shadow)
  - Padding: 8px horizontal, 4px vertical
  - Border radius: 4px for rounded corners
- Positioning:
  - Default: Centered horizontally, 10% from bottom
  - Never overlaps interactive game elements
  - Adjusts position if essential visual content would be obscured
- Animation:
  - Fade in: 200ms when caption appears
  - Fade out: 200ms when caption disappears
  - No jarring instant appearance/disappearance

**Responsive Behavior:**
- Mobile devices (< 768px width):
  - Font size increases to minimum 16px for readability
  - Caption area maximum width: 90% of screen width
  - Position fixed at bottom to avoid keyboard overlap
- Tablet (768px - 1024px):
  - Font size follows user preference or defaults to 18px
  - Caption area maximum width: 80% of screen width
- Desktop (> 1024px):
  - Font size follows user preference
  - Caption area maximum width: 70% of screen width or 800px

**High Contrast Mode Compatibility:**
- When high contrast mode enabled (see [Neurodiverse Support](neurodiverse-support.md)):
  - Text color: Pure white (#FFFFFF) or pure black (#000000) based on theme
  - Background: Pure black (#000000) or pure white (#FFFFFF) with 100% opacity
  - Edge style: Strong outline (2px) for maximum contrast
  - No gradients or semi-transparent effects

### Caption Authoring Tool UX

**Interface Layout:**
- Left panel (40%): Media player with playback controls
- Right panel (60%): Caption editor timeline and text entry
- Bottom: Transport controls (play, pause, rewind, fast-forward, skip)

**Caption Editor Features:**
- Waveform visualization showing audio amplitude over time
- Caption cues displayed as blocks on timeline
- Click waveform to add new caption cue at that timestamp
- Drag caption block edges to adjust timing
- Double-click caption block to edit text
- Color-coded validation indicators:
  - Green: Caption meets all quality standards
  - Yellow: Warning (long reading speed, long duration)
  - Red: Error (missing sound description, timing overlap)

**Real-Time Feedback:**
- Character counter updates as user types (42 char per line limit)
- Reading speed indicator (green < 20 char/sec, yellow 20-25, red > 25)
- Duration timer for current caption
- Sound description suggestions: "Add [sound] description?" when audio spike detected

**Keyboard Shortcuts for Efficiency:**
- `Space`: Play/pause media
- `Enter`: Create new caption cue at current time
- `Tab`: Move to next caption cue
- `Shift+Tab`: Move to previous caption cue
- `Ctrl+S`: Save caption track
- `Ctrl+Z`: Undo last edit
- `[` and `]`: Adjust start time of current cue by 100ms
- `{` and `}`: Adjust end time of current cue by 100ms

## Telemetry

Caption and audio description usage telemetry helps understand adoption, identify content gaps, and measure accessibility impact.

### Caption Events

```typescript
interface CaptionEvent {
  event_name: 'caption_toggled' | 'caption_language_changed' | 'caption_style_changed' | 'caption_displayed';
  user_id: string;
  session_id: string;
  timestamp: Date;
  
  // Context
  media_type: 'game_stage' | 'video' | 'audio' | 'announcement';
  media_id: string;              // game_id, video_id, etc.
  stage_id?: string;             // If media_type is game_stage
  
  // Event-specific properties
  properties: {
    // For caption_toggled
    captions_enabled?: boolean;
    trigger?: 'user_button' | 'keyboard_shortcut' | 'auto_enable' | 'preference';
    
    // For caption_language_changed
    previous_language?: string;
    new_language?: string;
    
    // For caption_style_changed
    setting_changed?: 'font_size' | 'color' | 'background' | 'position' | 'edge_style';
    new_value?: string;
    
    // For caption_displayed
    caption_cue_id?: string;
    caption_text?: string;         // For content analysis (anonymized in reports)
    display_duration_ms?: number;
  };
}
```

**Event Firing Rules:**
- `caption_toggled`: Fires when user enables or disables captions
- `caption_language_changed`: Fires when user selects different caption language
- `caption_style_changed`: Fires when user modifies caption appearance settings
- `caption_displayed`: Fires when each caption cue is displayed (sampled at 10% for performance)

### Audio Description Events

```typescript
interface AudioDescriptionEvent {
  event_name: 'audio_description_toggled' | 'audio_description_played' | 'extended_description_paused';
  user_id: string;
  session_id: string;
  timestamp: Date;
  
  media_type: 'game_stage' | 'video';
  media_id: string;
  stage_id?: string;
  
  properties: {
    // For audio_description_toggled
    audio_descriptions_enabled?: boolean;
    trigger?: 'user_button' | 'keyboard_shortcut' | 'preference';
    
    // For audio_description_played
    cue_id?: string;
    description_type?: 'standard' | 'extended';
    duration_ms?: number;
    
    // For extended_description_paused
    media_paused?: boolean;
    pause_duration_ms?: number;
    user_skipped?: boolean;        // True if user skipped extended description
  };
}
```

### Caption Authoring Events

```typescript
interface CaptionAuthoringEvent {
  event_name: 'caption_track_created' | 'caption_track_edited' | 'caption_track_published' | 'caption_quality_warning';
  user_id: string;                 // Content editor user ID
  timestamp: Date;
  
  caption_track_id: string;
  media_id: string;
  media_type: 'game_stage' | 'video' | 'audio';
  
  properties: {
    // For caption_track_created
    language?: string;
    file_format?: 'vtt' | 'srt' | 'ttml';
    auto_generated?: boolean;
    
    // For caption_track_edited
    edit_type?: 'timing' | 'text' | 'sound_description' | 'speaker_id';
    cue_count?: number;
    
    // For caption_track_published
    previous_status?: 'draft' | 'review';
    reviewed_by?: string;
    
    // For caption_quality_warning
    warning_type?: 'reading_speed_high' | 'timing_gap' | 'missing_sound_description' | 'spelling_error';
    warning_count?: number;
    cue_id?: string;
  };
}
```

### Metrics and Reporting

**Caption Usage Metrics:**
- Caption enablement rate: % of sessions where captions are enabled
- Caption enablement by user type: Student vs. Teacher vs. Admin
- Caption language distribution: Which languages are most used
- Average caption display duration per session
- Caption style customization rate: % of users who modify default styling

**Audio Description Metrics:**
- Audio description enablement rate
- Extended description skip rate: % of times users skip extended descriptions
- Audio description completion rate: % of content consumed with AD enabled

**Content Coverage Metrics:**
- % of games with captions available (by language)
- % of videos with audio descriptions
- Average caption completion percentage (how much of media is captioned)
- Caption quality score distribution (based on WCAG compliance checks)

**Accessibility Impact Metrics:**
- Session completion rate: Captioned vs. non-captioned content
- Student performance: Users with captions enabled vs. disabled
- User satisfaction: NPS or feedback scores for accessibility features

## Permissions

Caption and audio description management requires specific permissions to ensure content quality and prevent accidental modifications to published tracks.

### Read Permissions

| Role | Permission | Scope |
|------|-----------|-------|
| Student | View published captions | All games and content assigned to student |
| Student | Toggle captions on/off | Own sessions only |
| Student | Modify caption display preferences | Own profile only |
| Teacher | View published captions | All content in their account's catalog |
| Teacher | View caption availability status | For assignment planning (know which games have captions) |
| Admin Subscriber | View published captions | All content in organization's catalog |
| Admin Subscriber | View caption coverage reports | Organization's content usage |
| Content Editor | View all caption tracks | Including draft and review status tracks |
| Content Editor | View caption authoring history | Who created, modified, reviewed tracks |
| System Admin | View all caption data | Full platform visibility, including telemetry |

### Write Permissions

| Role | Permission | Scope |
|------|-----------|-------|
| Student | None | Students cannot author or modify captions |
| Teacher | None | Teachers cannot author captions (future: may request caption corrections) |
| Admin Subscriber | None | Admins cannot author captions |
| Content Editor | Create caption tracks | For games and videos in assigned content domains |
| Content Editor | Edit draft caption tracks | Own tracks or tracks in their domain |
| Content Editor | Submit captions for review | Change status from draft to review |
| Content Reviewer | Edit review-status captions | Captions submitted for review |
| Content Reviewer | Approve or reject caption tracks | Change status from review to published or back to draft |
| Content Reviewer | Request revisions | Add comments and change status to draft with revision notes |
| System Admin | Override any caption track | Emergency fixes for published content (logged in audit trail) |
| System Admin | Delete caption tracks | Remove deprecated or incorrect tracks (soft delete with archive) |

### Workflow State Transitions and Permissions

```typescript
// Caption track status workflow
type CaptionTrackStatus = 'draft' | 'review' | 'approved' | 'published' | 'deprecated';

// Allowed transitions
const statusTransitions = {
  draft: ['review'],                      // Content Editor can submit for review
  review: ['approved', 'draft'],          // Reviewer can approve or send back
  approved: ['published'],                // Reviewer or Admin can publish
  published: ['deprecated'],              // System Admin can deprecate (if replaced)
  deprecated: [],                         // Terminal state, no transitions
};

// Permission requirements for transitions
const transitionPermissions = {
  'draft -> review': ['content_editor', 'content_reviewer', 'system_admin'],
  'review -> approved': ['content_reviewer', 'system_admin'],
  'review -> draft': ['content_reviewer', 'system_admin'],
  'approved -> published': ['content_reviewer', 'system_admin'],
  'published -> deprecated': ['system_admin'],
};
```

### Audit and Compliance

All caption authoring, editing, and approval actions are logged in the audit trail (see [Sysadmin Audit Logs](../06-system-admin/sysadmin-audit-logs.md)):

**Logged Actions:**
- Caption track creation (who, when, for which media)
- Caption track edits (who, when, what changed)
- Status transitions (draft → review → published)
- Reviewer approval/rejection with comments
- System admin overrides (emergency edits to published tracks)
- Caption track deletion or deprecation
- User preference changes (for privacy compliance, users can see their own logs)

**Compliance Reports Available to System Admins:**
- WCAG compliance status by content item (which games/videos meet AA standards)
- Caption coverage percentage by content domain (Staff games, Rhythm games, etc.)
- Caption language availability matrix (which content has Spanish, French, etc. captions)
- Caption authoring velocity (how quickly new content is getting captions)
- Overdue caption requests (content published without captions beyond allowed grace period)

## Dependencies

This specification relies on several other systems and documents:

### System Dependencies

- **Media Player Infrastructure**: HTML5 video/audio players with WebVTT support for caption rendering
- **CDN and Media Delivery**: Caption files stored and delivered via CDN alongside media assets (see [CDN and Media](../17-integrations/cdn-and-media.md))
- **Game Player Shell**: Integration point for displaying captions in game Learn stages (see [Game Player Shell](../03-student-experience/game-player-shell.md))
- **User Profile System**: Storage for caption display preferences (see [Session and Auth States](../02-roles-and-permissions/session-and-auth-states.md))
- **Content Management System**: Workflow for caption authoring, review, and publishing (see [Content Approval Workflows](../29-content-management/content-approval-workflows.md))
- **Asset Pipeline**: Upload, validation, and versioning of caption files (see [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md))

### Specification Dependencies

- **[Accessibility Standards](../01-ux-design-system/a11y-standards.md)**: Broader accessibility requirements including caption preferences in user profile
- **[WCAG Compliance](wcag-compliance.md)**: WCAG 2.1 Level A and AA success criteria for captions and audio descriptions
- **[Game Model](../08-games-registry-and-authoring/game-model.md)**: Game entity structure that references caption tracks for Learn stages
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Analytics event structure for caption telemetry
- **[Localization and Internationalization](../01-ux-design-system/localization-internationalization.md)**: Multi-language caption support and language selection

### Third-Party Dependencies

- **WebVTT Standard (W3C)**: Technical specification for caption file format and browser rendering
- **HTML5 Media Elements API**: Browser support for `<track>` elements and TextTrack API
- **Video.js or similar player library**: If using third-party player for advanced caption features
- **Caption authoring tools**: Integration with tools like Amara, Subtitle Edit, or custom authoring UI

### Future Dependencies (Not in Initial Scope)

- **Automatic Speech Recognition (ASR) Service**: AI-generated caption drafts (future enhancement)
- **Translation Service**: Automated caption translation for multi-language support (future)
- **Live Captioning Service**: Real-time captioning for live events or webinars (future)

## Open questions

The following questions require decisions before full implementation:

### Technical Implementation

1. **Caption file storage location:**
   - Store caption files in same CDN bucket as media assets?
   - Or separate bucket for easier management and versioning?
   - Impact on cache invalidation strategy when captions are updated

2. **Caption caching strategy:**
   - Cache caption files at edge locations for low latency?
   - Cache duration: Same as media assets (long) or shorter for faster updates?
   - How to handle cache invalidation when captions are updated for published games?

3. **Browser compatibility for WebVTT:**
   - Minimum browser versions that fully support WebVTT styling and positioning
   - Fallback strategy for older browsers (plain text captions? SRT format?)
   - Testing required across target browsers (see [Browser Compatibility Matrix](../18-architecture/browser-compatibility-matrix.md))

4. **Audio description mixing in browser:**
   - Use Web Audio API for dynamic mixing of main audio and description audio?
   - Or pre-mix description tracks server-side (easier but less flexible)?
   - Impact on bandwidth if multiple audio tracks loaded simultaneously

### Content Creation and Workflow

5. **Caption authoring tool:**
   - Build custom caption authoring interface in MLC admin panel?
   - Or integrate with existing third-party caption editors (Amara, Subtitle Edit)?
   - Or allow editors to use external tools and upload completed .vtt files?
   - Decision impacts development timeline and ongoing maintenance

6. **Caption quality assurance process:**
   - Who performs QA review: Content reviewers, separate accessibility specialist, or external service?
   - QA checklist and automated validation tools to implement
   - Acceptable error rate before caption track is approved (typos, timing drift)

7. **Audio description voice talent:**
   - In-house narrator for consistency?
   - Or allow game developers to provide their own narration?
   - Voice consistency across content domains (Staff games, Rhythm games, etc.)
   - Budget for professional voice actors vs. synthetic text-to-speech (accessibility concerns with TTS)

### Policy and Compliance

8. **Caption requirement enforcement:**
   - Can games be published to "live" status without captions?
   - Grace period for adding captions to existing content (how long)?
   - Handling for user-generated content or teacher-uploaded videos (who captions those?)

9. **Language prioritization for captions:**
   - Which languages to support first beyond English?
   - Prioritization based on user demographics (Spanish for US market, French for Canada?)
   - Budget and timeline for multi-language caption expansion

10. **Legal compliance verification:**
    - Does platform need third-party accessibility audit (e.g., WebAIM, Deque)?
    - Frequency of compliance audits (annual, before major releases?)
    - Who is responsible for ensuring ongoing WCAG compliance as content is added?

### User Experience

11. **Default caption state for new users:**
    - Captions off by default (user opts in)?
    - Or captions on by default (user opts out)?
    - Data on user preferences from beta testing or competitor analysis

12. **Caption style customization:**
    - How extensive should customization be (full control vs. simple presets)?
    - Balance between user flexibility and development complexity
    - Default styles optimized through user testing

13. **Audio description user education:**
    - Many users may not know what audio descriptions are or how to enable them
    - In-app tutorial or help content explaining audio descriptions?
    - Proactive prompt to enable AD for users who may benefit (screen reader users)?

### Performance and Scalability

14. **Caption file size limits:**
    - Maximum size for caption files to ensure fast loading
    - Impact on bandwidth for mobile users
    - Compression strategies for large caption files (e.g., long videos)

15. **Concurrent caption track loading:**
    - Load all available caption languages up front, or lazy-load on language selection?
    - Impact on initial page load time vs. language switch delay
    - Decision impacts perceived performance

---

**Resolution Process**: These open questions should be addressed through:
1. Technical prototyping and performance testing
2. User research and accessibility testing with users who rely on captions/AD
3. Legal/compliance consultation with accessibility experts
4. Budget and timeline assessment with project management
5. Stakeholder review and sign-off before build phase begins
