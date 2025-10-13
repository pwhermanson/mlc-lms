# Glossary

## Purpose

This document provides canonical definitions for key terms and concepts used throughout the MLC-LMS platform. These definitions ensure consistent understanding across all documentation, development, and user interactions.

> **Related Documents**: For detailed information about user roles and permissions, see [Roles and Permissions](../02-roles-and-permissions/). For pedagogical principles and educational philosophy, see [Pedagogy Principles](pedagogy-principles.md). For system architecture details, see [System Overview](../18-architecture/system-overview.md).

## Core Learning Terms

### Learning Element (Element)
The atomic unit of content in the platform. Can be a Game stage, Text, Video, Audio, or Reward. Learning Elements are combined to create Assignments and Sequences. Each element has a unique identifier (e.g., GAM-0950-2 for Game 950, Stage 2).

### Game
A concept-focused set of interactive stages designed to teach specific musical skills. Each Game includes Learn, Play, Quiz, and Review stages. Some Games also include an optional Challenge stage. Games are designed to be simple, non-threatening, and fun while teaching very specific musical concepts. Review has its own score record.

### Stage
A specific phase within a Game:
- **Learn**: Introduction and exploration of new concepts (originally called "tutor stage")
- **Play**: Practice and reinforcement through interactive gameplay  
- **Quiz**: Assessment of understanding with mastery targets (quantitative assessment)
- **Challenge**: Optional advanced practice for motivated learners (some games have no score limit for worldwide competition)
- **Review**: Spaced repetition for long-term retention (scored separately)

### Tutor Stage
**Legacy term** - The original name for the first phase of a Game, now called "Learn". This terminology may still appear in legacy code, documentation, or historical references. **Tutor = Learn** in all contexts.

### Assignment
An ordered set of Learning Elements delivered to a student; part of a Group in a Sequence. Assignments are created from Sequences and can be customized by teachers.

### Group
A collection of Assignments within a Sequence. Groups help organize content by topic, difficulty, or curriculum unit.

### Sequence
An ordered set of Groups aligned to a curriculum or method book; can be first-party or method-aligned. Sequences form a complete learning pathway.

### Slot
A position within a Sequence where a Learning Element is placed. Slots maintain the order and structure of learning progression.

### Concept
The specific musical skill a Game teaches, e.g., rhythm value, interval, staff note, key signature. Games are organized by Level and Group covering aural and visual Pitch, Rhythm, Melody, Chords, Scales, Memory Playing, Intervals, Staff, Symbols, Keyboard Elements, Pattern Playing, and Musical Terms.

### Challenge
Optional competitive Game stage; not present in every Game. Allows students to compete with others worldwide with unlimited scoring potential. High scores are displayed on global leaderboards.

### Review
A repeat of Quiz scheduled later for retention; tracked separately in reports. Helps ensure long-term mastery of concepts through spaced repetition.

### Lifetime Musician
The fundamental goal of the platform - an active participant in music-making who can read, write, think and create in the universal language of music, continuing to learn new music independently for a lifetime of music enjoyment.

### Terry Treble
The platform's mascot character, created by Paul Hermanson, used in branding and as a friendly guide for students throughout their learning journey.

## User and Role Terms

### Member
A paying account that owns seats; typically a Subscriber-Admin for a studio or school. Members can have multiple roles (e.g., a Teacher can also be a Student).

### User
The technical account that authenticates a Member. A User can be associated with multiple Members in different contexts.

### Subscriber-Admin
The role that owns subscription, seats, billing, and teacher management. Has full administrative control over the organization.

### Teacher-Admin
Optional school role managing teachers and classes within a larger organization. Reports to Subscriber-Admin.

### Teacher
Role that manages students, builds assignments, and reads reports. Can be assigned to specific classes or grades.

### Student
Plays Games, completes Assignments, and tracks their learning progress through the platform.

### Student Resource Center (Dashboard)
Student hub for assignments, badges, and messages. Provides access to My Assignments, All Games, Scores History, and messaging with teachers.

### Generic Student
A special student account type often used for General Music classes where multiple students share the same username and password. Scores are recorded but are not individually meaningful since multiple students use the same credentials.

### Regular Student
A standard student account with unique username and password, allowing individual score tracking and participation in Challenge games with global competition.

### System Admin (SysAdmin)
Internal operator role for troubleshooting, billing overrides, and audits without accessing member passwords. Platform administrators who support the system at scale with global visibility and audit capabilities.

## Content and Assessment Terms

### Target Score
The minimum score required to demonstrate mastery of a specific Game stage. Target scores are set per stage and concept difficulty.

### Mastery
Achieving the target score on a Quiz for a given Game. Demonstrated understanding of a concept as measured by assessment performance.

### Review Score
A separate score recorded for Review stages to track long-term retention and spaced repetition effectiveness.

### Free Play
Student-initiated gameplay outside of assigned sequences. Free-play scores can count toward mastery when aligned with assigned content.

### Badge
Achievement marker for motivation; appears on dashboards and leaderboards.

### Sequence Version
A specific version of a Sequence that preserves grading history and ensures consistency across assignments.

### EVAL (Evaluation Sequence)
Diagnostic sequence for placement and gap discovery. Consists of selected Quiz stages at each level representing various concept types, used to evaluate student proficiency and determine appropriate starting level.

### LIFE (Lifetime Musician)
Default long-form curriculum sequence designed by Christine Hermanson to present concepts logically by level, generally following widely used pedagogical models. The cornerstone sequence for developing Lifetime Musicians.

### SOLF (Solfege)
Aural skills sequence emphasizing fixed-do work (do, re, mi, fa, sol, la, ti). Based on the Lifetime Musician sequence but replaces lettered notes with solfege syllables, particularly helpful for voice students.

### Student Pages
Archived curriculum mapping documents that correlate individual games to specific pages in popular piano methods (e.g., Alfred, Faber, Celebrate Piano). Used for manual assignment when automated sequences don't match specific method books.

## Technical Terms

### Game ID
Canonical identifier for a Game across all stages and contexts.

### Stage ID
Unique identifier for a specific stage within a Game.

### Element ID
Unique identifier for any Learning Element (Game, Text, Video, Audio, Reward).

### Assignment ID
Unique identifier for a specific Assignment instance.

### Sequence ID
Unique identifier for a named Sequence.

### Member ID
Unique identifier for a platform Member.

### User ID
Unique identifier for a platform User account.

### MIDI
Musical Instrument Digital Interface - optional keyboard input for certain Games. Supports USB and Bluetooth connections for enhanced kinesthetic learning. All games with on-screen keyboards can be played using MIDI keyboards.

### Smartboard
Interactive whiteboard technology used in classroom settings to engage entire classes with the learning games, particularly effective for General Music classes.

### iSwifter
Alternative browser app for iPad users to access the platform, as iPads don't support Flash Player natively.

## Business and Subscription Terms

### Plans
Pricing tiers: Prelude trial, Solo, Ensemble with seat scaling.

### Solo Plan
Paid subscription tier for individual studios with per-seat pricing.

### Ensemble Plan
Paid subscription tier for schools with volume pricing and advanced administrative features.

### Prelude
Time-limited trial plan for evaluation purposes.

### Seat (Slot)
A licensed student position within a plan. A user license that allows one Member to access the platform. Seats are managed by Subscriber-Admins.

### Proration
Billing adjustment when seats are added or removed mid-billing cycle.

### Purchase Order (PO)
School billing field captured for invoicing and statements. School-specific billing arrangement that allows payment via institutional purchase orders.

### Promo Code
Marketing discount code.

### Sponsor Code
Grant-style subsidy code with reporting. Generated for Ensemble subscribers to sponsor free Solo subscriptions for schools, particularly for General Music classes. Part of the Christine Hermanson Grant Program.

### Christine Hermanson Grant Program
Program established in honor of the platform's founder to encourage early learners in music education by providing free subscriptions to schools for General Music classes through sponsor codes.

### Statement
On-demand billing statement for a date range, often for schools.

## Platform Features

### All Games Library
Library for exploring any Game by category, level, or skill; scores persist to history. Student-accessible catalog of all available Games, organized by category and searchable by name. Features both thumbnail and list views with filtering by Level, Category, Visual/Aural, and Skill type.

### Leaderboard
Global high score display showing the top 20 students worldwide for Challenge games, encouraging friendly competition and motivation.

### Assignment Builder
Teacher tool for creating custom assignments by selecting and ordering Learning Elements.

### Bulk Operations
Administrative tools for managing multiple students, teachers, or content items simultaneously.

### CSV Import/Export
Bulk upload utilities for students, teachers, assignments, or content. File-based tools for bulk data management, particularly useful for school deployments.

### Impersonation
System Admin capability to safely view the platform as another user for support purposes.

### Audit Logs
Comprehensive logging of user actions, system changes, and administrative operations for compliance and debugging.

## Educational and Pedagogical Terms

### Sight-Reading
The ability to read and perform music at first sight, a core skill developed through the platform's sequential game progression.

### Aural Skills
Ear training abilities including pitch recognition, interval identification, and melodic dictation, heavily emphasized in the platform's curriculum.

### Music Literacy
The ability to read, write, and understand musical notation and concepts - the primary educational goal of the platform.

### Differentiated Learning
Educational approach supported by the platform allowing students at multiple skill levels to access appropriate content simultaneously.

### Spaced Repetition
Learning technique implemented through Review stages to reinforce concepts at increasing intervals for long-term retention.

### Mastery Learning
Educational philosophy where students must demonstrate proficiency (target score achievement) before advancing to new concepts.

## Platform History and Context

### Christine Hermanson
Founder and Director of MusicLearningCommunity.com, nationally recognized pioneer in music education technology, creator of the Lifetime Musician curriculum and MTNA National Symposium on Technology in Music.

### MTNA
Music Teachers National Association - professional organization for music teachers, with which the platform has strong historical connections through Christine Hermanson's leadership.

### Flash Player
Original technology platform for the learning games, requiring browser plugins for game functionality (being phased out in favor of modern web technologies).

## Acronyms

- **MLC**: MusicLearningCommunity
- **LMS**: Learning Management System
- **LTI**: Learning Tools Interoperability (future integration)
- **SIS**: Student Information System (future integration)
- **PO**: Purchase Order
- **CSV**: Comma-Separated Values
- **API**: Application Programming Interface
- **CDN**: Content Delivery Network
- **MIDI**: Musical Instrument Digital Interface
- **PWA**: Progressive Web App
- **WCAG**: Web Content Accessibility Guidelines
- **MTNA**: Music Teachers National Association
- **COPPA**: Children's Online Privacy Protection Act
- **SSL**: Secure Socket Layer (encryption technology)

## Supporting Documents Referenced

This glossary draws from the following source documents:

- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game stages and learning element definitions
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Terminology, feature descriptions, and user-facing language
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role definitions and user type descriptions
- [MLC History Draft 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20History%20Draft%202020.docx.txt) — Historical context and Lifetime Musician philosophy
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence terminology and structure
- [MLC SeqCreate 2020.xlsx - LIFE SEQ.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20LIFE%20SEQ.csv) — LIFE, SOLF, EVAL sequence definitions
- [Help Life Sequence.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Help%20Sequence/Help%20Life%20Sequence.xlsx%20-%20Sheet1.csv) — LIFE sequence details
- [Help Solf Sequemce.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Help%20Sequence/Help%20Solf%20Sequemce.xlsx%20-%20Sheet1.csv) — SOLF sequence details
- [Help Eval Sequence.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Help%20Sequence/Help%20Eval%20Sequence.xlsx%20-%20Sheet1.csv) — EVAL sequence details
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Seat, slot, and subscription tier definitions
- [MLC Tiered Pricing 2020.xlsx - Full Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Full%20Calc.csv) — Pricing tier terminology
- [EXE specs 2012-0913.xlsx - Elements.csv](../00-ORIG-CONTEXT/exe-specs/EXE%20specs%202012-0913.xlsx%20-%20Elements.csv) — Learning element specifications
- [AlphaMasterList2020.xlsx - Master.csv](../00-ORIG-CONTEXT/AlphaMasterList2020.xlsx%20-%20Master.csv) — Game taxonomy and concept definitions
- [pianoAdventures.doc.txt](../00-ORIG-CONTEXT/pianoAdventures.doc.txt) — Method book and Student Pages terminology
