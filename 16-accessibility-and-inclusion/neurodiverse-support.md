# Neurodiverse Support

## Purpose

This specification documents the design principles, features, and pedagogical approaches that make the MLC-LMS platform particularly effective for neurodiverse learners, including students with autism spectrum disorders, ADHD, dyslexia, sensory processing differences, and other learning variations. 

MLC has **incontrovertible evidence** of success with neurodiverse learners, particularly students with autism spectrum disorders. As documented in the platform's history, "the simple and engaging game presentation turned out to be very supportive of the learning efforts of children with special needs, particularly those diagnosed as autistic. Not only did they enjoy the activity of playing the music games, noticeable improvement in other areas of their experience came as a result of that activity."

This spec ensures that MLC 5.0 preserves and enhances the proven design patterns that have made the platform successful with neurodiverse learners while adding new features that provide even greater support.

## Scope

### Included

- **Proven Design Principles** - Core patterns that have demonstrated success with neurodiverse learners
- **Cognitive Accessibility Features** - Reduced stimuli modes, predictable layouts, clear navigation
- **Sensory Preferences** - Visual, audio, and motion reduction settings
- **Executive Function Support** - Clear task structure, progress indicators, achievement markers
- **Anxiety Reduction** - Non-threatening design, no time pressure, positive reinforcement
- **Self-Regulation Tools** - Pacing controls, break reminders, mastery-based progression
- **Teacher Guidance** - Best practices for supporting neurodiverse students
- **Parent Resources** - Information about neurodiverse support features
- **Preference Persistence** - User settings saved across sessions

### Excluded

- **Medical Diagnosis** - Platform does not diagnose or assess learning differences
- **Therapeutic Interventions** - MLC is educational, not a therapeutic tool
- **Individualized Education Plans (IEPs)** - Specific IEP implementation (though platform can support IEP goals)
- **Clinical Data Collection** - No collection of medical or diagnostic information
- **Third-Party Assistive Technology** - External tools covered in [Accessibility Standards](../01-ux-design-system/a11y-standards.md)

## Historical Foundation

### Documented Success

MLC's effectiveness with neurodiverse learners emerged organically from its foundational design principles:

**Original Documentation (2020):**
> "We have incontrovertible evidence that playing the music learning games is of benefit to children with special needs. In many cases the result of playing the games is improvement in areas other than music. The games are intentionally designed to be simple, enjoyable, and non-threatening. Each game teaches a very simple concept, and the sequencing takes the student from basics to more advanced concepts in a step-by-step process. The special needs student may benefit from achieving success with concept mastery, or only from playing a game to completion."

**Key Historical Outcomes:**
- Students with autism spectrum disorders showed particular affinity for the platform
- Improvements observed not only in music skills but in other areas of experience
- Simple, engaging presentation reduced anxiety and supported learning efforts
- Success rate high enough to be promoted as a core platform strength
- Teachers specifically sought out MLC for their neurodiverse students

### Why It Works

The platform's effectiveness stems from alignment with neurodiversity-affirming design principles:

1. **One Concept at a Time** - Reduces cognitive overload by focusing on single, specific learning objectives
2. **Predictable Structure** - Consistent staged approach (Learn → Play → Quiz → Challenge → Review) provides security through routine
3. **Clear Success Criteria** - Quantitative scoring provides objective, unambiguous feedback
4. **No Time Pressure** - Self-paced learning accommodates processing differences
5. **Immediate Feedback** - Reduces uncertainty and supports self-correction
6. **Mastery-Based Progression** - Students advance when ready, not on arbitrary timelines
7. **Visual and Aural Reinforcement** - Multi-sensory approach supports diverse processing styles
8. **Simple, Non-Threatening Graphics** - Reduces sensory overwhelm and anxiety
9. **Achievable Success** - Appropriately challenging tasks build confidence through accomplishment
10. **Consistent Patterns** - Predictable interactions reduce cognitive load and anxiety

## Data Model

### Neurodiverse Preferences Profile

```typescript
interface NeurodiversePreferences {
  userId: string;
  profileName: string; // e.g., "Reduced Stimuli", "High Contrast", "Minimal Audio"
  
  // Visual preferences
  visualSettings: {
    reducedStimuli: boolean; // Simplified graphics, reduced animations
    highContrast: boolean;
    monochromeMode: boolean; // Option to reduce color variety
    animationLevel: 'none' | 'minimal' | 'standard' | 'full';
    backgroundComplexity: 'simple' | 'standard' | 'detailed';
    fontSize: 'standard' | 'large' | 'extra-large';
  };
  
  // Sensory preferences
  sensorySettings: {
    soundEffects: 'off' | 'minimal' | 'standard';
    backgroundMusic: boolean;
    visualEffects: 'off' | 'minimal' | 'standard';
    reducedMotion: boolean;
    flashingContent: 'remove' | 'reduce' | 'allow';
  };
  
  // Cognitive support
  cognitiveSettings: {
    extraProcessingTime: boolean; // Extended timeouts before auto-advance
    simplifiedInstructions: boolean; // More concise, clear language
    stepByStepGuidance: boolean; // Additional scaffolding
    progressReminders: boolean; // Visual progress indicators
    achievementNotifications: 'immediate' | 'session-end' | 'weekly';
    errorHandling: 'gentle' | 'standard'; // Softer error messages
  };
  
  // Executive function support
  executiveSupport: {
    breakReminders: boolean; // Suggest breaks after set intervals
    breakInterval: number; // Minutes between break reminders (default: 20)
    sessionGoals: boolean; // Display session objectives at start
    nextStepPrompts: boolean; // Clear "what's next" guidance
    completionChecklists: boolean; // Visual task completion indicators
  };
  
  // Social/emotional settings
  socialSettings: {
    leaderboardParticipation: 'opt-in' | 'opt-out';
    competitiveFeatures: boolean; // Challenge mode, comparisons
    socialSharing: boolean; // Badge sharing, achievements
    encouragementLevel: 'minimal' | 'standard' | 'frequent';
  };
  
  // Applied automatically
  autoApply: boolean; // Apply preferences without confirmation
  teacherOverride: boolean; // Allow teacher to modify settings
  
  // Metadata
  createdBy: 'student' | 'teacher' | 'parent' | 'admin';
  lastModified: Date;
  effectiveDate: Date;
}
```

### Neurodiverse-Friendly Game Configuration

```typescript
interface GameNeurodiverseConfig {
  gameId: string;
  
  // Cognitive load indicators
  cognitiveLoad: 'low' | 'medium' | 'high';
  conceptCount: number; // Number of simultaneous concepts
  stimuliLevel: 'minimal' | 'moderate' | 'complex';
  
  // Sensory profile
  visualComplexity: 'simple' | 'moderate' | 'complex';
  auditoryDemands: 'low' | 'medium' | 'high';
  animationIntensity: 'static' | 'subtle' | 'animated' | 'highly-animated';
  
  // Executive function requirements
  taskSwitching: boolean; // Requires switching between tasks
  sequentialSteps: number; // Number of steps to completion
  workingMemoryLoad: 'low' | 'medium' | 'high';
  
  // Support features available
  supportFeatures: {
    reducedStimuliMode: boolean;
    simplifiedInstructions: boolean;
    additionalGuidance: boolean;
    practiceMode: boolean; // Extra practice without scoring
  };
}
```

## Behavior and Rules

### Preference Application

**Automatic Application:**
- Neurodiverse preferences apply automatically when user logs in
- Settings persist across sessions and devices
- Changes take effect immediately without page refresh
- Default to most supportive settings for flagged accounts

**Progressive Disclosure:**
- Advanced preferences hidden until needed
- Teacher can enable/disable specific option groups
- Preset profiles available: "Minimal Stimuli", "High Structure", "Sensory Friendly", "Executive Support"

**Inheritance and Override:**
- Teacher-Admin can set default preferences for groups of students
- Individual student settings override group defaults
- Teachers can temporarily override for specific assignments
- System Admin can set organization-wide baseline preferences

### Visual Presentation Rules

**Reduced Stimuli Mode:**
When enabled:
- Remove decorative animations and background elements
- Use solid, muted background colors instead of patterns
- Reduce color palette to 3-4 core colors
- Eliminate particle effects, transitions, and flourishes
- Display static mascot (Terry Treble) instead of animated
- Simplified button designs with clear labels
- Increased white space between interactive elements

**Predictable Layout:**
- Consistent element placement across all game stages
- Game controls always in same position
- Progress indicators in fixed location (top right)
- Navigation always in same area (bottom)
- No unexpected popups or modal interruptions
- Clear visual hierarchy with headers and sections

### Audio Presentation Rules

**Sensory-Friendly Audio:**
When sensory-friendly audio enabled:
- Background music disabled by default
- Sound effects at 50% standard volume
- No overlapping audio sources
- Optional visual indicators for all audio cues
- Mono audio output (no stereo panning effects)
- Adjustable audio balance per source type

**Audio Alternatives:**
- All audio content has text equivalent
- Musical notation displayed when notes played
- Visual feedback for rhythm and timing
- Captions for spoken instructions

### Cognitive Support Rules

**Processing Time Extensions:**
When extra processing time enabled:
- Auto-advance delays increased by 50%
- No time limits on quiz questions
- Extend timeout warnings from 30s to 60s
- Add "I need more time" button to pause auto-progression
- Allow unlimited review of instructions before starting

**Clear Task Structure:**
- Display numbered steps for multi-step tasks
- Show progress through current game (e.g., "Question 3 of 10")
- Provide "What's Next" preview after each game stage
- Visual checklist for assignment completion
- Clear exit points with save confirmation

### Executive Function Support

**Break Management:**
When break reminders enabled:
- Suggest breaks every [configurable] minutes (default: 20)
- Show elapsed practice time in corner of screen
- Gentle prompt: "You've been practicing for 20 minutes. Would you like to take a break?"
- Save progress automatically before break
- Resume exactly where student left off

**Goal Setting and Tracking:**
- Display session goal at start (e.g., "Complete 3 Quiz stages today")
- Visual progress bar toward session goal
- Celebrate completion with clear achievement message
- Option to set daily/weekly goals
- Teacher can view goal completion rates

**Organization Support:**
- Clear assignment queue with prioritization
- "Next Assignment" prominently displayed
- Completed work clearly marked with checkmarks
- "Resume" option for partially completed assignments
- Visual separation between different assignment types

### Anxiety Reduction Features

**Non-Threatening Feedback:**
When gentle error handling enabled:
- Replace "Incorrect" with "Not quite, try again"
- Remove red X symbols, use neutral "↻ Try Again" button
- No penalty for wrong answers in Learn and Play stages
- Multiple attempts allowed before providing correct answer
- Encouraging messages: "You're learning!" "Keep going!"

**Mastery-Based Progression:**
- No comparisons to other students (unless opted-in)
- Progress measured against own past performance
- Target scores clearly displayed and achievable
- Option to repeat games without affecting score history
- "Practice Mode" available for stress-free exploration

**Positive Reinforcement:**
- Immediate visual feedback for correct responses
- Accumulating achievements throughout session
- End-of-session summary of accomplishments
- Badge awards for effort, not just perfection
- Teacher commendation system for encouragement

## UX Requirements

### Student Interface Adaptations

**Dashboard (Neurodiverse-Optimized):**
```
┌─────────────────────────────────────────────────────┐
│  [🎵 MLC Logo]           Hi, [Student Name]  [Help] │
│                                                      │
│  Your Next Assignment:                              │
│  ┌─────────────────────────────────────────────┐   │
│  │  ⬚ Learn: Note Names on Treble Clef         │   │
│  │     Simple • 5 minutes • Ready to start     │   │
│  │                    [▶ Start Learning]        │   │
│  └─────────────────────────────────────────────┘   │
│                                                      │
│  Today's Progress: ▓▓▓░░  (3 of 5 goals)           │
│                                                      │
│  ✅ Completed Today:                                │
│  • Play: Rhythm Factory                            │
│  • Quiz: Half Notes and Whole Notes                │
│  • Review: Treble Clef Line Notes                  │
│                                                      │
│  [🎮 All Games]  [📊 My Progress]  [⚙️ Settings]    │
└─────────────────────────────────────────────────────┘
```

**Key Features:**
- Single primary action prominently displayed
- Clear, simple language
- Visual progress indicators
- Calm, uncluttered layout
- Consistent element positioning
- High contrast text
- Large touch/click targets (min 44px)

**Game Player Interface (Reduced Stimuli):**
```
┌─────────────────────────────────────────────────────┐
│ [← Back]    Learn: Note Names     [Progress: 3/10] │
├─────────────────────────────────────────────────────┤
│                                                      │
│            [Simple, Clean Background]               │
│                                                      │
│       ♪  Click the note that is played  ♪          │
│                                                      │
│         ┌────┬────┬────┬────┬────┬────┐            │
│         │ C  │ D  │ E  │ F  │ G  │ A  │            │
│         └────┴────┴────┴────┴────┴────┘            │
│                                                      │
│              [🔊 Play Again]                        │
│                                                      │
├─────────────────────────────────────────────────────┤
│  ⏱️ Practice time: 8 min         [Need Help?]      │
└─────────────────────────────────────────────────────┘
```

**Key Features:**
- Minimal visual distractions
- Clear, large instruction text
- Simple, high-contrast buttons
- One interaction at a time
- Progress visible but not distracting
- Help always accessible
- Consistent layout across all games

### Teacher Interface - Neurodiverse Support Tools

**Student Profile - Neurodiverse Settings:**
```
┌─────────────────────────────────────────────────────┐
│  Student: Alex Martinez                             │
│  Neurodiverse Support Profile: ✓ Active             │
├─────────────────────────────────────────────────────┤
│  Current Profile: "Sensory Friendly" (Custom)       │
│                                                      │
│  Quick Settings:                                     │
│  ☑ Reduced stimuli mode                            │
│  ☑ Extended processing time                        │
│  ☑ Break reminders (every 20 min)                  │
│  ☐ Simplified instructions                         │
│  ☑ Gentle error handling                           │
│  ☐ Disable competitive features                    │
│                                                      │
│  [Customize Profile...]  [Load Preset]             │
│                                                      │
│  Performance Notes:                                 │
│  • Completes 15% more games with break reminders   │
│  • Shows preference for visual over audio games    │
│  • Best performance: mornings before 11am          │
│                                                      │
│  [View Detailed Analytics]                          │
└─────────────────────────────────────────────────────┘
```

**Assignment Builder - Neurodiverse Filters:**
- Filter games by cognitive load level
- Filter by sensory profile (visual/audio intensity)
- View games by executive function requirements
- See which games have reduced-stimuli mode
- Preview game in student's configured display mode
- Suggest appropriate games based on student profile

### Preset Profiles

**Minimal Stimuli Profile:**
- Simple backgrounds and graphics
- No animations
- Minimal sound effects
- High contrast
- Extra white space
- Large buttons

**High Structure Profile:**
- Session goals displayed
- Step-by-step progress
- Break reminders
- Clear next steps
- Completion checklists
- Detailed progress tracking

**Sensory Friendly Profile:**
- Reduced motion
- Lower audio volume
- No background music
- No flashing content
- Muted color palette
- Visual alternatives for audio

**Executive Support Profile:**
- Break reminders
- Session goals
- Progress indicators
- Next step prompts
- Organization tools
- Time awareness features

**Combination Profiles:**
Teachers can select multiple profile features or create fully custom configurations.

## Telemetry

### Neurodiverse Support Events

Track effectiveness and usage patterns to continuously improve support:

```typescript
// Profile activation
{
  event: 'neurodiverse_profile_activated',
  userId: string,
  profileType: 'minimal-stimuli' | 'high-structure' | 'sensory-friendly' | 'executive-support' | 'custom',
  appliedBy: 'student' | 'teacher' | 'parent' | 'admin',
  timestamp: Date
}

// Setting changes
{
  event: 'neurodiverse_setting_changed',
  userId: string,
  settingCategory: 'visual' | 'sensory' | 'cognitive' | 'executive' | 'social',
  settingName: string,
  oldValue: any,
  newValue: any,
  changedBy: 'student' | 'teacher' | 'parent',
  timestamp: Date
}

// Break interactions
{
  event: 'break_reminder_response',
  userId: string,
  action: 'break-taken' | 'break-dismissed' | 'break-snoozed',
  sessionDuration: number, // minutes
  gamesCompleted: number,
  timestamp: Date
}

// Support feature usage
{
  event: 'support_feature_used',
  userId: string,
  feature: 'reduced-stimuli' | 'extended-time' | 'gentle-errors' | 'practice-mode' | 'extra-guidance',
  gameId: string,
  gameStage: string,
  timestamp: Date
}

// Completion rates with/without support
{
  event: 'game_completion',
  userId: string,
  gameId: string,
  gameStage: string,
  neurodiverseProfileActive: boolean,
  supportFeaturesUsed: string[],
  completionTime: number, // seconds
  score: number,
  attemptsRequired: number,
  timestamp: Date
}

// Success patterns
{
  event: 'mastery_achieved',
  userId: string,
  gameId: string,
  neurodiverseProfileActive: boolean,
  supportFeaturesUsed: string[],
  daysToMastery: number,
  attemptsRequired: number,
  timestamp: Date
}
```

### Analytics Queries

**For Teachers:**
- How does student perform with vs. without neurodiverse support features?
- Which support features are most used by this student?
- What time of day shows best performance?
- Are break reminders being utilized?
- Completion rates by game complexity level

**For System Admins:**
- What percentage of students use neurodiverse support features?
- Which features are most commonly enabled?
- Correlation between support features and completion rates
- Effectiveness of different preset profiles
- Feature usage trends over time

**For Product Development:**
- Which games present highest cognitive load for neurodiverse learners?
- Where do students with support profiles get stuck?
- What game characteristics correlate with success?
- Feature adoption and abandonment rates
- A/B testing of support feature variations

## Permissions

### Student-Level Permissions

**Own Profile:**
- View all neurodiverse support settings
- Modify own preferences (if age-appropriate, typically 13+)
- Enable/disable break reminders
- Choose preset profiles
- Reset to teacher-configured defaults

**Younger Students (< 13):**
- View current settings
- Request changes (requires teacher approval)
- Toggle break reminders only
- Cannot disable teacher-mandated settings

### Teacher Permissions

**For Their Students:**
- View and modify all neurodiverse support settings
- Create and apply custom profiles
- Set mandatory settings (student cannot override)
- View analytics on support feature effectiveness
- Export student preference profiles
- Clone settings from one student to another

**Cannot:**
- Access medical/diagnostic information
- Override parent-set restrictions
- Share student profiles with other teachers (without parent consent)

### Teacher-Admin Permissions

**Organization-Wide:**
- Set default neurodiverse profiles for school/studio
- Create organization preset profiles
- View aggregated (anonymized) effectiveness data
- Mandate minimum support settings
- Configure which features teachers can modify

### Parent/Guardian Permissions

**For Their Child:**
- View and modify all neurodiverse support settings
- Set restrictions on student self-modification
- Enable/disable data collection for support analytics
- Request consultation with teacher on settings
- Export preference profile for IEP documentation

### System Admin Permissions

**Platform-Wide:**
- Configure available neurodiverse support features
- Create system preset profiles
- View aggregated usage and effectiveness analytics
- A/B test new support features
- Configure default settings for new accounts
- Access audit logs of setting changes

## Dependencies

### Core Dependencies

**Accessibility Standards:**
- [Accessibility Standards](../01-ux-design-system/a11y-standards.md) - Foundation accessibility requirements
- [WCAG Compliance](../16-accessibility-and-inclusion/wcag-compliance.md) - Technical accessibility standards
- [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md) - Alternative input methods

**Pedagogy:**
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) - Core learning principles that support neurodiverse learners
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game structure and staging
- [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) - Learning pathways

**User Experience:**
- [Student Dashboard](../03-student-experience/student-dashboard.md) - Primary student interface
- [Game Player Shell](../03-student-experience/game-player-shell.md) - Game presentation layer
- [Assignment Play](../03-student-experience/assignment-play.md) - Assignment workflow

**Teacher Tools:**
- [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) - Teacher overview
- [Assignment Builder](../04-teacher-experience/assignment-builder.md) - Assignment creation with neurodiverse considerations
- [Teacher Reports](../04-teacher-experience/teacher-reports.md) - Student progress analytics

**Content:**
- [Game Metadata](../08-games-registry-and-authoring/games-metadata.md) - Cognitive load and sensory profile data
- [Content Versioning](../07-content-architecture/content-versioning.md) - Support for simplified vs. standard versions

### Technical Dependencies

**Frontend:**
- CSS custom properties for theme switching (reduced stimuli, high contrast)
- Animation controls (prefers-reduced-motion CSS media query)
- Local storage for preference persistence
- React context for global accessibility state

**Backend:**
- User preferences service
- Profile management API
- Analytics event pipeline
- Feature flag system for gradual rollout

**Data:**
- User preferences table
- Profile templates table
- Support feature usage events
- Game neurodiverse metadata

## Implementation Guidelines

### Design Principles

**Neurodiversity-Affirming Approach:**
1. **Support, don't "fix"** - Frame features as support options, not corrections
2. **User agency** - Empower users to choose what works for them
3. **Avoid stigma** - Use neutral, respectful language (not "special needs mode")
4. **Celebrate differences** - Recognize neurodiversity as natural variation
5. **Evidence-based** - Ground decisions in research and user feedback

**Universal Design for Learning (UDL):**
- Multiple means of representation (visual, auditory, kinesthetic)
- Multiple means of engagement (motivational variety)
- Multiple means of expression (demonstrate learning in various ways)
- Benefits all students, essential for neurodiverse learners

### Content Creation Guidelines

**When Designing New Games:**

1. **Simplicity First:**
   - Teach one specific concept per game
   - Minimize extraneous visual elements
   - Use clear, concise instructions
   - Provide example before practice

2. **Predictable Structure:**
   - Consistent layout across game stages
   - Standard control placement
   - Familiar interaction patterns
   - Clear visual hierarchy

3. **Sensory Considerations:**
   - Avoid rapid flashing (seizure risk)
   - Limit animation speed and complexity
   - Provide volume controls for audio
   - Use high-contrast color combinations
   - Test with reduced stimuli mode enabled

4. **Cognitive Load Management:**
   - Limit simultaneous information sources
   - Use chunking for complex concepts
   - Provide scaffolding for multi-step tasks
   - Allow processing time before auto-advance

5. **Error Handling:**
   - Gentle, encouraging error messages
   - Clear path to correct response
   - No penalties for mistakes in learning stages
   - Positive reinforcement for efforts

**Metadata Requirements:**
All games must include neurodiverse metadata:
- Cognitive load level (low/medium/high)
- Visual complexity rating
- Auditory demands level
- Executive function requirements
- Reduced stimuli mode availability
- Recommended support features

### Testing Requirements

**Neurodiverse User Testing:**
- Include neurodiverse users in all usability testing
- Test with various support profiles enabled
- Validate reduced stimuli mode for all games
- Test with screen readers and keyboard navigation
- Verify predictable, consistent interactions

**Performance Testing:**
- Ensure support features don't impact performance
- Test preference loading and application speed
- Verify smooth transitions between display modes
- Test cross-device preference synchronization

**Regression Testing:**
- Automated tests for preference application
- Visual regression tests for reduced stimuli mode
- Accessibility audit for all game templates
- Cross-browser testing with support features enabled

## Teacher Guidance

### Best Practices for Supporting Neurodiverse Students

**Understanding Individual Needs:**
- Every neurodiverse student is unique
- Observe which features help each student
- Be willing to adjust and experiment
- Communicate with parents about settings
- Document what works for future reference

**Starting Configuration:**
1. Begin with a preset profile (sensory-friendly or high-structure)
2. Observe student response for 2-3 sessions
3. Adjust based on performance and feedback
4. Fine-tune individual settings as needed
5. Review and adjust periodically

**Communication Strategies:**
- Explain features in simple, positive terms
- Emphasize student choice and agency
- Celebrate successes and progress
- Be patient with adjustment period
- Maintain regular check-ins

**Progress Monitoring:**
- Review analytics comparing with/without support features
- Note which games are most successful
- Track completion rates over time
- Document effective settings for future students
- Share insights with parents

**When to Adjust Settings:**
- Student shows frustration or avoidance
- Completion rates declining
- Student requests changes
- Moving to more/less complex content
- Changes in student's needs or environment

### Recommended Settings by Profile

**Autism Spectrum Disorder (ASD):**
- ✓ Reduced stimuli mode
- ✓ Predictable, consistent layouts
- ✓ Extended processing time
- ✓ Clear task structure and progression
- ✓ Break reminders
- Consider: Gentle error handling, minimal audio

**ADHD:**
- ✓ Break reminders (every 15-20 min)
- ✓ Session goals and progress tracking
- ✓ Clear next steps
- ✓ Immediate feedback
- ✓ Achievement notifications
- Consider: Reduced background animations, simplified navigation

**Dyslexia:**
- ✓ Extra-large text
- ✓ High contrast
- ✓ Extended reading time
- ✓ Audio alternatives for text
- ✓ Simplified instructions
- Consider: Dyslexia-friendly fonts (if available)

**Sensory Processing Disorder:**
- ✓ Reduced stimuli mode
- ✓ Minimal sound effects
- ✓ No background music
- ✓ Reduced motion
- ✓ Simple color palette
- Consider: Ability to control all sensory inputs independently

**Anxiety Disorders:**
- ✓ Gentle error handling
- ✓ No competitive features
- ✓ Practice mode availability
- ✓ Clear success criteria
- ✓ Positive reinforcement
- ✓ No time pressure
- Consider: Private progress (no leaderboards)

**Multiple Co-occurring Conditions:**
Start with most restrictive settings, gradually enable features as appropriate.

## Parent Resources

### Understanding Neurodiverse Support Features

**Information for Parents:**
MLC-LMS includes research-based features that support neurodiverse learners. These features can be customized by your child's teacher to match their individual needs.

**Why MLC Works for Neurodiverse Learners:**
1. **One concept at a time** - Reduces overwhelm
2. **Clear, predictable structure** - Reduces anxiety
3. **Self-paced learning** - No pressure to keep up
4. **Immediate feedback** - Clear understanding of progress
5. **Achievable goals** - Builds confidence
6. **Non-threatening design** - Reduces learning anxiety

**Historical Success:**
> "The simple and engaging game presentation turned out to be very supportive of the learning efforts of children with special needs, particularly those diagnosed as autistic. Not only did they enjoy the activity of playing the music games, noticeable improvement in other areas of their experience came as a result of that activity."

**How to Support Your Child:**
- Review teacher-configured settings together
- Encourage use of break reminders
- Celebrate achievements, not just scores
- Maintain consistent practice schedule
- Communicate with teacher about what works
- Let your child explore support options

**Accessing Settings:**
Parents can view and modify neurodiverse support settings through the parent/guardian portal. Work with your child's teacher to find the optimal configuration.

## Open Questions

### Research and Validation

1. **Longitudinal Studies** - Can we track long-term outcomes for students using neurodiverse support features?
2. **Efficacy Measurement** - What metrics best demonstrate effectiveness of support features?
3. **Comparative Analysis** - How do completion rates compare with/without support across different neurodivergent profiles?
4. **Feature Optimization** - Which combinations of support features are most effective?
5. **Generalization** - Do skills learned with support features transfer to other contexts?

### Feature Development

1. **AI Personalization** - Can machine learning suggest optimal support settings based on performance patterns?
2. **Adaptive Difficulty** - Should games automatically adjust complexity based on neurodiverse profile?
3. **Social Features** - How to include neurodiverse students in social/competitive features safely?
4. **Parent Dashboard** - What level of detail should parents see in neurodiverse support analytics?
5. **Progress Sharing** - How to support IEP documentation and reporting needs?

### Content Questions

1. **Game Categorization** - How to categorize games by suitability for different neurodivergent profiles?
2. **Alternative Versions** - Should some games have distinct neurodiverse-optimized versions?
3. **New Game Types** - Are there game mechanics particularly suited to neurodiverse learners?
4. **Sensory Profiles** - Should we create detailed sensory profiles for all games?
5. **Scaffolding Levels** - How many levels of support should be available per game?

### User Experience

1. **Profile Naming** - What terminology is most respectful and clear?
2. **Student Control** - At what age should students manage their own settings?
3. **Preset Profiles** - How many presets are helpful vs. overwhelming?
4. **Mode Switching** - Should students be able to temporarily disable support for challenge?
5. **Visual Indicators** - How to show which support features are active without stigma?

### Privacy and Ethics

1. **Data Sensitivity** - Should neurodiverse support data be treated as sensitive/medical information?
2. **Disclosure** - Should students be able to hide their use of support features from peers?
3. **Research Consent** - What consent is needed to use support feature data for research?
4. **Cross-Teacher Sharing** - Can neurodiverse profiles be shared when students change teachers?
5. **Third-Party Access** - Should IEP coordinators have read access to support settings?

## Related Documentation

**Accessibility:**
- [Accessibility Standards](../01-ux-design-system/a11y-standards.md) - Platform-wide accessibility requirements
- [WCAG Compliance](../16-accessibility-and-inclusion/wcag-compliance.md) - Technical compliance standards
- [Keyboard and MIDI Access](../16-accessibility-and-inclusion/keyboard-and-midi-access.md) - Alternative input methods
- [Captions and Audio Descriptions](../16-accessibility-and-inclusion/captions-and-audio-descriptions.md) - Multimedia accessibility

**Pedagogy:**
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) - Learning science foundation
- [Personas](../00-foundations/personas.md) - User profiles including neurodiverse learners
- [Legacy Systems](../00-foundations/legacy-systems.md) - Historical neurodiverse support success

**User Experience:**
- [Student Dashboard](../03-student-experience/student-dashboard.md) - Student interface
- [Game Player Shell](../03-student-experience/game-player-shell.md) - Game presentation
- [Student Settings](../03-student-experience/student-settings.md) - User preferences

**Teacher Tools:**
- [Teacher Dashboard](../04-teacher-experience/teacher-dashboard.md) - Teacher interface
- [Assignment Builder](../04-teacher-experience/assignment-builder.md) - Creating neurodiverse-friendly assignments
- [Teacher Reports](../04-teacher-experience/teacher-reports.md) - Student analytics

**Content:**
- [Game Model](../08-games-registry-and-authoring/game-model.md) - Game structure
- [Games Metadata](../08-games-registry-and-authoring/games-metadata.md) - Neurodiverse metadata
- [Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md) - Learning pathways
