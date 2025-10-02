# Assignment Play Interface Redesign

## Overview
Redesigned interface for students to play assigned games and track their progress, addressing critical usability issues identified in the original MLC system and implementing a streamlined two-tier game selection experience.

## Description
Modernized student experience for playing assignments, including improved navigation, progress tracking, score display, and seamless integration with the learning management system. This redesign addresses the highest priority improvement needed for the MLC platform, as efficient student game playing is essential to the website's value proposition.

## Core Design Principles

### Two-Tier Game Selection Structure
The redesigned Assignment Play interface implements a clear two-tier structure that separates game selection from stage selection:

1. **Screen 1 - Game Selection**: Students choose either default (thumbnail) or list view to select a game
2. **Screen 2 - Stage Selection**: Upon game selection, students choose from available stages (Learn, Play, Quiz, Challenge)
3. **Screen 3 - Full-Screen Gameplay**: Game/stage is presented in a full-screen experience with header and footer removed
4. **Return Navigation**: Upon game completion, "Quit" returns students to Screen 2 (stage selection)

### Game-Centric Display
- **Primary Focus**: Display games, not game/stages, as the main selection unit
- **Stage Organization**: Each game shows its available stages in a secondary selection interface
- **Thumbnail Optimization**: Very small thumbnails to maximize games visible "above the fold" (24-36 games instead of 12)
- **Review Stage Handling**: Review stage need not be presented in the main interface, as it's just Quiz repetition

## Addressing Historical Pain Points

Based on feedback from the original system, the Assignment Play interface specifically addresses these key issues:

### Navigation and User Experience
- **Easy Dashboard Return**: Clear "Return to Dashboard" button provides quick navigation back to check scores and progress
- **Game ID Display**: Game identifiers are prominently displayed for easy reference and support purposes
- **Reduced Visual Clutter**: Header elements are minimized during gameplay to maximize screen real estate for the learning experience

### Progress Visibility
- **At-a-Glance Progress**: Students can see their completion status and scores without navigating away from the assignment
- **Score Recording Confirmation**: Clear feedback ensures students know their progress is being tracked
- **Next Assignment Preview**: "Similar Courses" section shows upcoming assignments in the student's sequence

### Technical Reliability
- **Score Recording Integrity**: Robust score submission ensures no progress is lost due to technical issues
- **Offline Resilience**: Cached content and queued submissions maintain functionality during connectivity issues
- **Error Recovery**: Clear error messages and recovery options prevent students from getting stuck

## Assignment Play Interface Layout

### Header Section
- **Assignment Information**: Clear display of assignment name, sequence, and level
- **Game ID Display**: Prominent Game ID for easy reference and support
- **Return to Dashboard**: Quick navigation button for checking scores and progress
- **Next Up Button**: Focus on the next required step in the assignment
- **Progress Indicator**: Visual representation of overall assignment completion

### Game Selection Panel
- **Two-Tier Structure**: 
  - **Tier 1**: Game selection within the assignment
  - **Tier 2**: Stage selection (Learn, Play, Quiz, Challenge, Review) within selected game
- **Game Information**: Display Game ID, Game Name, Skill Level, Skill Area, and availability
- **Progress Pills**: Visual indicators showing completion status for each game
- **Locked States**: Clear indication when prerequisites are not yet met

### Stage Selection Interface
- **Stage Cards**: Visual cards for each available stage (Learn, Play, Quiz, Challenge, Review)
- **Stage States**: Clear indication of locked, available, in progress, or complete states
- **Target Scores**: Display target scores for each stage when relevant
- **Score History**: Show previous attempts and best scores for motivation

### Full-Screen Gameplay Experience
- **Chrome-less Mode**: Game takes up whole screen with header and footer removed
- **Minimal Overlay**: Essential controls only (pause, exit, help)
- **Back Navigation**: Clear back button if student decides not to play
- **Return Flow**: "Quit" button automatically returns to stage selection screen

## Score Recording and Display

### Real-Time Score Tracking
- **Score Recording**: All scores are recorded consistently whether played through Assignments or All Games
- **Progress Indicators**: Clear visual feedback showing completion status and mastery levels
- **Score History**: Students can see their progress at a glance with clear completion indicators
- **Score Recording Confirmation**: Clear feedback ensures students know their progress is being tracked

### Score Display Format
Based on the original system requirements:
- **Target Score**: Clear indication of required score for mastery
- **Times Played**: Track of attempts made
- **Best Score**: Highest score achieved
- **Last Score**: Most recent attempt score
- **Completion Status**: Visual indicators for completed, in progress, or not started

## Integration with All Games Library

The Assignment Play interface integrates with the broader [All Games Library](../03-student-experience/all-games-library.md) through a two-tier selection structure:

### Free Play Reconciliation
The system reconciles scores from the All Games library with Assignment requirements:
- If a student has already played a game in free play mode with a qualifying score, the Assignment Play interface can mark that stage as complete
- Teachers can configure whether fresh attempts are required regardless of prior free play scores
- This integration ensures students don't repeat work unnecessarily while maintaining assessment integrity

### Consistent Score Recording
- All scores are recorded consistently whether played through Assignments or All Games
- Students can see their progress at a glance with clear completion indicators
- Score recording status is clearly communicated to prevent confusion about progress tracking

## Pedagogical Approach

The Assignment Play interface implements the core pedagogical principles outlined in [Pedagogy Principles](../00-foundations/pedagogy-principles.md) through structured, progressive learning:

### Stage-Based Learning Progression
Each game follows the established Learn → Play → Quiz → Review sequence:
- **Learn Stage**: Interactive tutorial with guided exploration, no scoring pressure
- **Play Stage**: Practice mode with immediate feedback and encouragement
- **Quiz Stage**: Formal assessment determining mastery and progression eligibility
- **Challenge Stage**: Optional advanced practice with competitive elements
- **Review Stage**: Spaced repetition for long-term retention

### Mastery-Based Progression
- Students must demonstrate mastery (typically 80-85% on Quiz stage) before advancing
- Failed attempts provide learning opportunities rather than punitive consequences
- Teachers can adjust mastery thresholds based on individual student needs
- Review stages ensure long-term retention of mastered concepts

## Technical Requirements

### Performance Optimization
- **Loading Speed**: Minimize initial page load time for game discovery
- **Thumbnail Loading**: Efficient thumbnail loading and caching
- **Responsive Design**: Optimized for various screen sizes and devices
- **Browser Compatibility**: Support for modern browsers with graceful degradation

### Data Integration
- **Game Registry**: Integration with [Games Registry](../08-games-registry-and-authoring/games-registry-overview.md) for real-time game availability
- **Search Indexing**: Connection to [Search Architecture](../18-architecture/search-architecture.md) for efficient game discovery
- **Progress Tracking**: Integration with [Assignment Play](../03-student-experience/assignment-play.md) for seamless gameplay experience

## Accessibility Features

### Visual Accessibility
- **High Contrast**: Support for high contrast mode
- **Large Text**: Scalable text sizes
- **Color Independence**: Information not conveyed by color alone
- **Focus Indicators**: Clear focus states for keyboard navigation

### Cognitive Accessibility
- **Clear Hierarchy**: Obvious visual hierarchy
- **Consistent Patterns**: Predictable interaction patterns
- **Error Prevention**: Clear feedback and confirmation
- **Help Integration**: Contextual help and guidance

### Motor Accessibility
- **Large Touch Targets**: Minimum 44px touch targets
- **Keyboard Navigation**: Full keyboard accessibility
- **Voice Control**: Support for voice commands
- **Switch Control**: Support for assistive devices

## Future Enhancements

### Advanced Features
- **Personalized Recommendations**: AI-driven game suggestions based on student progress
- **Favorites System**: Allow students to bookmark frequently played games
- **Progress Indicators**: Visual representation of game completion and mastery
- **Social Features**: Integration with [Social Learning](../27-social-learning/social-learning-framework.md) capabilities

### Analytics Integration
- **Usage Tracking**: Monitor game selection patterns and popular choices
- **Performance Metrics**: Track loading times and user engagement
- **Search Analytics**: Analyze search patterns to improve discovery
- **A/B Testing**: Test different layouts and features for optimization
