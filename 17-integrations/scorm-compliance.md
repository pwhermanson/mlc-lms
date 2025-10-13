# SCORM Compliance Specifications

## Overview
SCORM 2004 and SCORM 1.2 compliance specifications for content interoperability and standards compliance.

## Description
Detailed SCORM compliance requirements for ensuring content can be imported from and exported to other learning management systems, maintaining compatibility with industry standards.

## Purpose

This specification defines SCORM (Shareable Content Object Reference Model) compliance capabilities for the MLC-LMS platform. SCORM enables packaging and distribution of MLC learning content as standardized, portable modules that can be imported into third-party Learning Management Systems (LMS) that support SCORM standards.

**Why SCORM for MLC**:
- **Content Portability**: Enable schools to use MLC content within their existing LMS infrastructure without requiring live integration
- **Offline Distribution**: Package content for environments with limited or no internet connectivity
- **Institutional Requirements**: Meet procurement requirements for institutions that mandate SCORM-compliant educational content
- **Legacy System Compatibility**: Support older LMS platforms that may not support modern standards like LTI 1.3 or xAPI
- **Third-party Marketplaces**: Enable distribution of MLC content through educational content marketplaces and repositories

**SCORM vs. Other Integration Approaches**:
- **SCORM**: Packaged content exported as ZIP files, self-contained, supports offline use, static snapshots
- **LTI** (see [LMS LTI Integration](./lms-ltispec-future.md)): Live embedding of MLC platform, real-time grade sync, dynamic content updates
- **xAPI** (see [xAPI Implementation](./xapi-implementation.md)): Learning analytics data exchange, not content packaging, complements both SCORM and LTI

**Key Limitation**: SCORM packages are static snapshots of content at a point in time. Dynamic features like real-time progress tracking, teacher messaging, live leaderboards, and personalized recommendations require live LTI integration or native MLC platform use.

## Scope

### Included

**SCORM Package Generation**:
- Automated creation of SCORM 1.2 and SCORM 2004 (3rd/4th Edition) packages
- Package individual games, sequences, or assignments
- Manifest file (imsmanifest.xml) generation with metadata
- Content organization and file structure per SCORM specification
- Sequencing rules and navigation for SCORM 2004 (Simple Sequencing)
- Launch data and initialization parameters
- Completion and scoring data model implementation

**Content Packaging**:
- HTML5 game wrappers for SCORM API communication
- Asset bundling (game files, audio, graphics, fonts)
- Accessibility metadata and alternative content
- Localization support for multi-language packages
- Resource file manifest and checksums
- Compression and ZIP archive creation

**SCORM API Implementation**:
- JavaScript API adapter for SCORM 1.2 and 2004
- Communication with LMS via SCORM API
- Data model elements (cmi.core, cmi.interactions, cmi.objectives)
- Score reporting and completion tracking
- Suspend data for progress preservation
- Error handling and diagnostic logging

**Export Workflows**:
- Teacher and Admin interfaces for SCORM package generation
- Configuration options (SCORM version, scope, packaging options)
- Background job processing for package generation
- Download delivery and expiration management
- Package validation and testing tools

**Metadata and Cataloging**:
- IEEE LOM (Learning Object Metadata) for content description
- Dublin Core metadata elements
- Educational metadata (difficulty, age range, learning objectives)
- Technical requirements and compatibility notes
- Rights and licensing information

### Excluded

**Real-time Features**: Live teacher messaging, real-time leaderboards, dynamic content updates, personalized recommendations (require live platform or LTI).

**CMI5 Profile**: CMI5 (xAPI profile for launched content) is the modern successor to SCORM. CMI5 support is out of scope for initial SCORM implementation but noted for future consideration.

**SCORM Cloud Hosting**: MLC will not host SCORM packages in third-party SCORM repositories or learning object repositories. Packages are generated on-demand for customer download.

**SCORM 1.1 and Earlier**: Legacy SCORM versions prior to 1.2 are not supported due to limited adoption and significant limitations.

**AICC Standard**: Alternative standard to SCORM, not widely used, excluded from scope.

**Content Authoring Tools**: MLC is not a SCORM authoring tool. SCORM packages are generated from existing MLC content (games, sequences), not created from scratch.

**LMS-Specific Customizations**: While SCORM is a standard, some LMS platforms have vendor-specific extensions or quirks. Platform-specific workarounds are out of scope unless critical for major platforms (Canvas, Moodle, Blackboard).

## Data model (if applicable)

### SCORM Package Entity

Represents a generated SCORM package available for download.

**Fields**:
- `package_id`: UUID for the SCORM package
- `content_type`: Enum: `game` | `sequence` | `assignment` | `game_collection`
- `content_id`: Reference to source content (game ID, sequence ID, assignment ID)
- `scorm_version`: Enum: `scorm_1_2` | `scorm_2004_3rd_edition` | `scorm_2004_4th_edition`
- `package_scope`: Enum: `single_game` | `sequence_full` | `assignment` | `custom_collection`
- `generated_by_user_id`: User who requested package generation
- `organization_id`: Organization context for package
- `requested_at`: Timestamp of package request
- `status`: Enum: `queued` | `generating` | `completed` | `failed` | `expired`
- `package_name`: Display name for the package
- `package_description`: Description text for package metadata
- `include_review_stages`: Boolean - include review/retention stages in package
- `include_instructional_content`: Boolean - include learn stages and instructional videos
- `scoring_mode`: Enum: `quiz_only` | `average_all_stages` | `cumulative`
- `file_url`: Pre-signed URL for package download (24-hour expiration)
- `file_size_bytes`: Size of generated ZIP package
- `file_checksum_md5`: MD5 hash for package integrity verification
- `expires_at`: Download link expiration timestamp
- `error_message`: Error details if status is `failed`
- `metadata`: JSON object with additional packaging options

### SCORM Manifest Structure

The `imsmanifest.xml` file is the core descriptor for every SCORM package. MLC generates manifests conforming to SCORM specifications.

**Key Manifest Elements**:

**Metadata Section**:
```xml
<metadata>
  <schema>ADL SCORM</schema>
  <schemaversion>2004 4th Edition</schemaversion>
  <lom>
    <general>
      <title><langstring xml:lang="en">MLC - Sight Reading: Treble Clef Level 1</langstring></title>
      <description><langstring xml:lang="en">Master treble clef note reading through interactive games</langstring></description>
      <keyword><langstring xml:lang="en">music literacy</langstring></keyword>
      <keyword><langstring xml:lang="en">sight reading</langstring></keyword>
    </general>
    <lifecycle>
      <version>1.0</version>
      <contribute>
        <role><source>LOMv1.0</source><value>publisher</value></role>
        <entity>BEGIN:VCARD FN:MusicLearningCommunity END:VCARD</entity>
      </contribute>
    </lifecycle>
    <educational>
      <learningresourcetype><source>LOMv1.0</source><value>exercise</value></learningresourcetype>
      <interactivitylevel><source>LOMv1.0</source><value>medium</value></interactivitylevel>
      <difficulty><source>LOMv1.0</source><value>easy</value></difficulty>
      <typicallearningtime><datetime>PT00H30M00S</datetime></typicallearningtime>
    </educational>
  </lom>
</metadata>
```

**Organizations Section (Sequencing)**:
```xml
<organizations default="org_mlc_sequence_001">
  <organization identifier="org_mlc_sequence_001">
    <title>Sight Reading Sequence</title>
    <item identifier="item_assignment_01" identifierref="res_assignment_01">
      <title>Assignment 1: Treble Clef Lines</title>
      <item identifier="item_game_learn_01" identifierref="res_game_learn_01">
        <title>Learn: Note Names</title>
      </item>
      <item identifier="item_game_quiz_01" identifierref="res_game_quiz_01">
        <title>Quiz: Note Reading</title>
        <imsss:sequencing>
          <imsss:controlMode choice="true" flow="true" />
          <imsss:sequencingRules>
            <imsss:preConditionRule>
              <imsss:ruleConditions conditionCombination="all">
                <imsss:ruleCondition condition="completed" referencedObjective="obj_game_learn_01" />
              </imsss:ruleConditions>
              <imsss:ruleAction action="disabled" />
            </imsss:preConditionRule>
          </imsss:sequencingRules>
        </imsss:sequencing>
      </item>
    </item>
  </organization>
</organizations>
```

**Resources Section**:
```xml
<resources>
  <resource identifier="res_game_quiz_01" type="webcontent" href="games/game_01542/index.html" adlcp:scormtype="sco">
    <file href="games/game_01542/index.html" />
    <file href="games/game_01542/game.js" />
    <file href="games/game_01542/styles.css" />
    <file href="shared/scorm_api_wrapper.js" />
    <file href="assets/audio/piano_c4.mp3" />
    <file href="assets/images/staff_treble.svg" />
    <dependency identifierref="res_shared_assets" />
  </resource>
  <resource identifier="res_shared_assets" type="webcontent" adlcp:scormtype="asset">
    <file href="shared/mlc_core.js" />
    <file href="shared/styles_common.css" />
  </resource>
</resources>
```

### SCORM API Data Model (CMI Elements)

MLC games packaged for SCORM communicate with the LMS using standardized data model elements.

**SCORM 1.2 Data Model**:
- `cmi.core.lesson_status`: `not attempted`, `incomplete`, `completed`, `passed`, `failed`
- `cmi.core.score.raw`: Numeric score (0-100)
- `cmi.core.score.min`: Minimum score (0)
- `cmi.core.score.max`: Maximum score (100)
- `cmi.core.session_time`: Time spent in session (HH:MM:SS format)
- `cmi.core.total_time`: Cumulative time across all sessions
- `cmi.core.exit`: `time-out`, `suspend`, `logout`
- `cmi.suspend_data`: String data for resuming progress (max 4096 chars)
- `cmi.launch_data`: Initialization parameters passed by LMS
- `cmi.student_preference.audio`: Audio level (0-100)
- `cmi.interactions.n.id`: Interaction identifier
- `cmi.interactions.n.type`: `true-false`, `choice`, `fill-in`, `matching`, `performance`
- `cmi.interactions.n.result`: `correct`, `wrong`, `unanticipated`, `neutral`
- `cmi.interactions.n.latency`: Time to complete interaction (HH:MM:SS)

**SCORM 2004 Data Model** (extended):
- `cmi.completion_status`: `completed`, `incomplete`, `not attempted`, `unknown`
- `cmi.success_status`: `passed`, `failed`, `unknown`
- `cmi.score.scaled`: Normalized score (0.0 to 1.0)
- `cmi.progress_measure`: Completion percentage (0.0 to 1.0)
- `cmi.session_time`: ISO 8601 duration (PT1H30M5S)
- `cmi.objectives.n.id`: Learning objective identifier
- `cmi.objectives.n.score.scaled`: Objective-specific score
- `cmi.objectives.n.success_status`: Objective completion status
- `cmi.interactions.n.timestamp`: ISO 8601 timestamp
- `cmi.interactions.n.description`: Interaction description
- `adl.nav.request`: Navigation request (`continue`, `previous`, `exit`, `suspendAll`)

### MLC-Specific Suspend Data Format

MLC games store progress state in `cmi.suspend_data` to enable session resumption.

**JSON Structure** (serialized as string):
```json
{
  "mlc_version": "5.0",
  "game_id": "G-01542",
  "stage_type": "quiz",
  "attempt_number": 2,
  "questions_attempted": 8,
  "questions_correct": 6,
  "current_question_index": 8,
  "mistakes_by_concept": {
    "PITCH_READING_TREBLE": 2,
    "NOTE_DURATION": 0
  },
  "mastery_progress": 0.75,
  "last_checkpoint": "2025-10-12T15:30:00Z"
}
```

**Size Constraint**: SCORM 1.2 limits suspend_data to 4096 characters. SCORM 2004 increases limit to 64,000 characters. MLC minimizes suspend data to essential progress markers only.

## Behavior and rules

### SCORM Package Generation Workflow

**1. User Initiates Package Generation**:
- Teacher or Admin navigates to game, sequence, or assignment detail page
- Clicks "Export as SCORM Package" action button
- SCORM package configuration modal appears

**2. Configuration Options**:
- **SCORM Version**: Radio button selection: SCORM 1.2, SCORM 2004 (3rd Edition), SCORM 2004 (4th Edition)
- **Package Scope**: 
  - Single Game (one stage or all stages)
  - Full Sequence (all assignments in sequence)
  - Single Assignment (one assignment from sequence)
  - Custom Collection (select multiple games manually)
- **Stages to Include**: Checkboxes for Learn, Play, Quiz, Challenge, Review
- **Scoring Mode**: 
  - Quiz Only (score based solely on quiz stage)
  - Average All Stages (average scores across all included stages)
  - Cumulative (sum of all stage scores)
- **Include Instructional Content**: Toggle for including learn stages, videos, text instructions
- **Package Name**: Text input (auto-populated, editable)
- **Package Description**: Textarea for metadata description

**3. Validation**:
- Verify all selected content is accessible and published (not draft or archived)
- Check that all game assets are available and up-to-date
- Confirm user has permission to export selected content
- Validate package size won't exceed reasonable limits (recommend <500MB)

**4. Package Generation Job**:
- Create SCORM package entity with status `queued`
- Queue background job for package generation (see [Background Jobs](../18-architecture/background-jobs.md))
- Display confirmation: "SCORM package generation started. You'll be notified when ready."

**5. Package Assembly Process**:
- **Generate Manifest**: Create imsmanifest.xml with metadata, organization structure, resources
- **Wrap Games**: Generate HTML5 wrappers with SCORM API communication for each game
- **Bundle Assets**: Copy game files, audio, graphics, shared libraries to package directory structure
- **Inject SCORM API**: Include scorm_api_wrapper.js for LMS communication
- **Add Metadata Files**: Include README, license, technical requirements documentation
- **Create ZIP Archive**: Compress entire package structure into single ZIP file
- **Validate Package**: Run automated SCORM conformance tests
- **Store and Generate Download URL**: Upload to temporary storage with pre-signed URL

**6. Completion and Delivery**:
- Update package entity status to `completed`
- Send in-app and email notification with download link
- Download link expires after 24 hours
- User downloads ZIP file, imports into their LMS

### SCORM API Communication Flow

**1. Package Launch in LMS**:
- Student clicks SCORM package link in LMS course
- LMS creates new SCO (Sharable Content Object) session
- LMS loads game HTML file in iframe or new window
- LMS provides SCORM API object for communication

**2. API Initialization**:
- Game JavaScript loads and executes
- MLC SCORM API wrapper searches for LMS API object:
  - SCORM 1.2: `API` object in window hierarchy
  - SCORM 2004: `API_1484_11` object in window hierarchy
- Wrapper calls `Initialize()` (SCORM 1.2) or `Initialize("")` (SCORM 2004)
- Retrieve launch data and suspend data from LMS
- Restore game state if suspend data exists (resuming session)

**3. During Gameplay**:
- Track interactions and progress locally in game state
- Periodically commit data to LMS (every 30 seconds or on significant events):
  - Update `cmi.core.session_time` or `cmi.session_time`
  - Log interactions to `cmi.interactions` array
  - Update `cmi.progress_measure` if SCORM 2004
- Handle errors gracefully if LMS API calls fail

**4. Score Calculation and Reporting**:
- When student completes quiz or assessment stage:
  - Calculate final score (0-100 raw, 0.0-1.0 scaled)
  - Determine success status (passed if score >= pass_threshold)
  - Set `cmi.core.score.raw` and `cmi.core.lesson_status` (SCORM 1.2)
  - Set `cmi.score.scaled`, `cmi.completion_status`, `cmi.success_status` (SCORM 2004)
  - Commit data to LMS immediately

**5. Session Suspend or Exit**:
- If student pauses or leaves before completion:
  - Serialize current game state to JSON
  - Store in `cmi.suspend_data`
  - Set `cmi.core.exit` to `suspend` (SCORM 1.2) or `cmi.exit` to `suspend` (SCORM 2004)
  - Call `Commit()` to persist data
- If student completes game:
  - Set `cmi.core.exit` to `logout` or leave blank
  - Set `cmi.core.lesson_status` to `completed` or `passed`
  - Call `Commit()` then `Terminate()`

**6. API Error Handling**:
- If `Initialize()` fails: Log error, proceed in standalone mode (no LMS communication)
- If `SetValue()` fails: Log error, retry once, continue gameplay if persistent failure
- If `Commit()` fails: Log error, retry on next attempt, warn user if critical data loss risk
- If `Terminate()` fails: Log error, attempt multiple retries before giving up

### Sequencing and Navigation Rules

**SCORM 2004 Simple Sequencing** enables prerequisite enforcement and adaptive navigation.

**Common Sequencing Rules for MLC Packages**:

**Prerequisite Completion**:
- Quiz stages require completion of Learn stage
- Review stages require completion of Quiz stage with passing score
- Next assignment unlocks only after previous assignment is completed

**Example Rule (XML)**:
```xml
<imsss:sequencingRules>
  <imsss:preConditionRule>
    <imsss:ruleConditions conditionCombination="all">
      <imsss:ruleCondition condition="satisfied" referencedObjective="obj_learn_stage" />
    </imsss:ruleConditions>
    <imsss:ruleAction action="disabled" />
  </imsss:preConditionRule>
</imsss:sequencingRules>
```

**Rollup Rules**:
- Assignment is complete when all child items (stages) are complete
- Sequence is complete when all assignments are complete
- Score rolled up from individual stage scores based on configured scoring mode

**Adaptive Navigation**:
- If student fails quiz (score < pass_threshold), option to retry quiz or return to Learn stage
- If student achieves mastery (score >= 90%), option to skip review stage
- "Continue" button enabled only when prerequisites are met

**SCORM 1.2 Limitations**: SCORM 1.2 does not support sequencing rules. MLC packages targeting SCORM 1.2 must implement prerequisite logic within game JavaScript (less robust, LMS cannot enforce).

### Package Validation and Testing

**Automated Validation**:
Before package download is enabled, automated tests verify:
- **Manifest Validity**: imsmanifest.xml parses correctly and conforms to SCORM schema
- **Resource References**: All files referenced in manifest exist in package
- **File Integrity**: All files have valid checksums, no corruption
- **API Compatibility**: SCORM API wrapper included and correctly referenced
- **Metadata Completeness**: Required LOM elements are present and valid
- **Package Size**: Total size is reasonable (<500MB recommended)

**Testing Recommendations**:
- Test package in SCORM conformance testing tool (e.g., ADL SCORM Test Suite, SCORM Cloud)
- Import package into target LMS platforms (Canvas, Moodle, Blackboard) and verify launch
- Complete gameplay session and verify score reporting to LMS gradebook
- Test suspend/resume functionality
- Verify package works in LMS iframe and new window launch modes

**Common Validation Errors**:
- Missing or malformed imsmanifest.xml
- Broken file paths (incorrect relative paths, missing files)
- SCORM API calls with incorrect syntax
- Suspend data exceeds size limits
- Metadata schema violations

## UX requirements (if applicable)

### SCORM Package Generation Interface

**Location**: Export options available in:
- Game detail page (single game export)
- Sequence detail page (sequence or assignment export)
- Assignment detail page (single assignment export)
- Admin content management interface (bulk exports)

**Export Button**:
- Label: "Export as SCORM Package"
- Icon: Package/download icon
- Placement: In actions menu or toolbar alongside other export options

**Configuration Modal**:
- **Header**: "Generate SCORM Package: {Content Name}"
- **SCORM Version Selection**: Radio buttons with help text explaining differences
  - ○ SCORM 1.2 (maximum compatibility with older LMS platforms)
  - ○ SCORM 2004 3rd Edition (recommended, supports sequencing and navigation)
  - ○ SCORM 2004 4th Edition (latest, best for modern LMS platforms)
- **Package Scope**: Dropdown or radio buttons
  - Single Game (all stages)
  - Single Game (select stages)
  - Full Sequence
  - Single Assignment
  - Custom Collection (opens multi-select game picker)
- **Stages to Include**: Checkboxes (if single game or custom collection)
  - ☑ Learn
  - ☑ Play
  - ☑ Quiz
  - ☑ Challenge
  - ☑ Review
- **Scoring Mode**: Dropdown
  - Quiz Only (score based on quiz stage only)
  - Average All Stages (average of all stage scores)
  - Cumulative (sum of all stage scores)
- **Include Instructional Content**: Toggle switch
  - When ON: Includes learn stages, instructional videos, text content
  - When OFF: Quiz and assessment stages only
- **Package Name**: Text input, auto-populated with content name, editable
- **Package Description**: Textarea for metadata description (max 500 characters)
- **Advanced Options** (collapsible section):
  - Include accessibility metadata
  - Include alternative audio descriptions
  - Set typical learning time (duration estimate)
  - Set difficulty level (beginner, intermediate, advanced)
- **Action Buttons**:
  - "Cancel" (secondary button, closes modal)
  - "Generate Package" (primary button, submits request)

**Progress and Notification**:
- After submitting: Toast notification appears: "SCORM package generation started. Estimated time: 2-5 minutes. You'll be notified when ready."
- In-app notification center shows progress: "Generating SCORM package... (processing)"
- When complete: "Your SCORM package is ready. Download {filename}" with clickable download link
- If failed: "SCORM package generation failed: {error_message}. Please try again or contact support."

### Package Download Interface

**Download Page**:
- **Package Details**:
  - Package Name
  - Content Type (Game, Sequence, Assignment, Collection)
  - SCORM Version
  - File Size
  - Generated Date
  - Expiration Date (24 hours from generation)
- **Download Button**: Large, prominent "Download SCORM Package (ZIP)" button
- **Installation Instructions**: Expandable section with step-by-step guide for importing into common LMS platforms (Canvas, Moodle, Blackboard)
- **Technical Requirements**: List of browser and LMS requirements
- **Support Links**: Link to SCORM troubleshooting guide and support contact

**Package History** (optional):
- Table listing past SCORM package exports
- Columns: Content Name, SCORM Version, Generated Date, Status, Download Link
- Filter by content type, status, date range
- "Download" button for completed packages (if not expired)
- "Re-generate" button for expired packages

### Accessibility

- **WCAG 2.1 AA Compliance**: All SCORM export interfaces meet accessibility standards
- **Keyboard Navigation**: Modal and forms fully keyboard navigable
- **Screen Reader Support**: Form labels, help text, and error messages announced
- **High Contrast Mode**: Status indicators and buttons have sufficient contrast
- **Accessible Generated Packages**: SCORM packages include accessibility metadata, alternative text for images, and keyboard navigation in games

## Telemetry

### SCORM Package Events

**`scorm_package_requested`**
- **When**: User initiates SCORM package generation
- **Properties**:
  - `package_id`: UUID
  - `content_type`: game | sequence | assignment | collection
  - `content_id`: Source content identifier
  - `scorm_version`: scorm_1_2 | scorm_2004_3rd_edition | scorm_2004_4th_edition
  - `package_scope`: single_game | sequence_full | assignment | custom_collection
  - `stages_included`: Array of stage types [learn, play, quiz, review]
  - `scoring_mode`: quiz_only | average_all_stages | cumulative
  - `include_instructional_content`: boolean
  - `user_id`: Requesting user ID
  - `organization_id`: Organization context

**`scorm_package_generated`**
- **When**: SCORM package generation completes successfully
- **Properties**:
  - `package_id`: UUID
  - `content_type`: game | sequence | assignment | collection
  - `scorm_version`: scorm_1_2 | scorm_2004_3rd_edition | scorm_2004_4th_edition
  - `file_size_bytes`: Size of generated ZIP
  - `game_count`: Number of games included in package
  - `asset_count`: Number of media assets bundled
  - `generation_duration_ms`: Time from request to completion
  - `validation_passed`: boolean

**`scorm_package_generation_failed`**
- **When**: SCORM package generation fails
- **Properties**:
  - `package_id`: UUID
  - `content_type`: game | sequence | assignment | collection
  - `scorm_version`: scorm_1_2 | scorm_2004_3rd_edition | scorm_2004_4th_edition
  - `error_type`: validation_failed | asset_missing | permission_denied | system_error
  - `error_message`: Descriptive error text
  - `generation_duration_ms`: Time from request to failure

**`scorm_package_downloaded`**
- **When**: User downloads SCORM package ZIP file
- **Properties**:
  - `package_id`: UUID
  - `content_type`: game | sequence | assignment | collection
  - `scorm_version`: scorm_1_2 | scorm_2004_3rd_edition | scorm_2004_4th_edition
  - `file_size_bytes`: Size of downloaded file
  - `time_since_generation_hours`: Hours between generation and download
  - `download_method`: in_app_notification | package_history | direct_link

**`scorm_package_link_expired`**
- **When**: User attempts to access an expired SCORM package download link
- **Properties**:
  - `package_id`: UUID
  - `content_type`: game | sequence | assignment | collection
  - `time_since_generation_hours`: Hours since package was generated

### Sampling Rules
- **Never sampled**: `scorm_package_requested`, `scorm_package_generation_failed` (critical audit events)
- **10% sampling**: `scorm_package_downloaded`, `scorm_package_link_expired` (volume reduction)
- **Never sampled for System Admins**: All events captured at 100% for audit trail

## Permissions

Access control for SCORM package generation follows the platform's [Roles Matrix](../02-roles-and-permissions/roles-matrix.md).

### SCORM Package Generation Permissions

**Generate SCORM Package - Own Custom Content**: Teacher, Teacher-Admin, Subscriber-Admin, Content Editor, System Admin
**Generate SCORM Package - Organization Content**: Teacher-Admin (own org), Subscriber-Admin (own org), Content Editor, System Admin
**Generate SCORM Package - First-Party Sequences (LIFE, SOLF, EVAL)**: Content Editor, System Admin only (proprietary content)
**Generate SCORM Package - Method-Aligned Sequences**: Requires active entitlement for specific method/publisher; Content Editor, System Admin
**Configure Advanced SCORM Options**: Content Editor, System Admin only
**Bulk SCORM Package Generation**: System Admin only
**View SCORM Package History**: User (own exports), Subscriber-Admin (own organization), System Admin (all)

### Content Restrictions

**Custom Sequences**: Creator and organization members with appropriate roles can generate SCORM packages.
**First-Party Content**: MLC proprietary sequences (LIFE, SOLF, EVAL) can only be packaged by Content Editors and System Admins to protect pedagogical IP.
**Method-Aligned Content**: Organizations must have active subscription/entitlement for specific method books to generate SCORM packages.
**Archived Content**: Only Content Editors and System Admins can package archived games or sequences.

## Dependencies

### Internal Dependencies

**Game Registry** ([Games Registry Overview](../08-games-registry-and-authoring/games-registry-overview.md)):
- Game metadata for SCORM package manifest
- Game assets for bundling (HTML5 games, audio, graphics)
- Stage definitions for package structure

**Sequence Model** ([Sequence Model](../09-sequences-and-curriculum-tools/sequence-model.md)):
- Sequence hierarchy for organization structure in manifest
- Gating rules for SCORM 2004 sequencing
- Target scores and pass thresholds for success criteria

**Assignment Model** ([Assignment Model](../07-content-architecture/assignment-model.md)):
- Assignment structure for packaging
- Completion criteria for SCORM tracking

**Export Infrastructure** ([Export Specs](../20-imports-and-automation/export-specs.md)):
- Background job queue for package generation
- File storage and download delivery
- Export job tracking and history

**Event Model** ([Event Model](../15-analytics-and-reporting/event-model.md)):
- Telemetry for SCORM package generation and downloads
- Audit trail for compliance

**Background Jobs** ([Background Jobs](../18-architecture/background-jobs.md)):
- Async processing for package generation
- Retry logic for failed package assembly
- Queue management and prioritization

**API Design** ([API Design Principles](../18-architecture/api-design-principles.md)):
- REST endpoints for SCORM package requests
- Download URL generation and expiration

**Security and Privacy** ([Security Privacy](../18-architecture/security-privacy.md)):
- Pre-signed URLs for package downloads
- Encryption of packages containing student-specific data
- License and rights management for packaged content

### External Dependencies

**SCORM Specifications**:
- **SCORM 1.2** (ADL, 2001): Run-Time Environment, Content Aggregation Model
- **SCORM 2004 3rd Edition** (ADL, 2006): Run-Time Environment, Sequencing and Navigation, Content Aggregation Model
- **SCORM 2004 4th Edition** (ADL, 2009): Updated specifications with errata corrections

**IEEE LOM** (Learning Object Metadata): Metadata schema for educational content description.

**Dublin Core**: Metadata element set for resource description.

**ADL SCORM Test Suite**: Conformance testing tools for validating SCORM packages.

**SCORM Cloud** (Rustici Software): Optional third-party service for package testing and validation.

### Related Documentation

- [LMS LTI Integration](./lms-ltispec-future.md) - Alternative approach for live LMS integration without content packaging
- [xAPI Implementation](./xapi-implementation.md) - Learning analytics data exchange, complements SCORM packages
- [Sequence Exports](../09-sequences-and-curriculum-tools/sequence-exports.md) - CSV/JSON export formats for sequences
- [Games Importer](../08-games-registry-and-authoring/games-importer.md) - Game content import and registry management
- [Export Specs](../20-imports-and-automation/export-specs.md) - General export infrastructure and workflows
- [API Design Principles](../18-architecture/api-design-principles.md) - REST API design for SCORM package endpoints
- [Background Jobs](../18-architecture/background-jobs.md) - Async job processing for package generation

## Supporting Documents Referenced

This SCORM compliance specification draws from the following source documents:

- [Course import.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Course%20import.xlsx%20-%20Sheet1.csv) — Course structure and sequencing informing SCORM organization hierarchy and content packaging patterns
- [MLC SeqCreate 2020.xlsx - specs.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20specs.csv) — Sequence specifications informing SCORM sequencing rules and prerequisite logic
- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Game metadata and asset structure informing SCORM resource bundling and manifest generation
- [System Admin Re-design.docx.txt](../00-ORIG-CONTEXT/System%20Admin%20Re-design.docx.txt) — Admin tools and export workflows informing SCORM package generation interfaces

## Open questions

### Technical Implementation

**HTML5 Game Wrappers**: Should MLC games be wrapped in HTML5 containers for SCORM, or should we convert games to pure HTML5 (no server dependencies)?
- **Wrapper Approach**: Easier implementation, maintains existing game codebase
- **Pure HTML5**: Better compatibility, truly offline-capable, but requires significant game refactoring
- **Recommendation**: Start with wrapper approach; evaluate pure HTML5 conversion for high-demand games

**Asset Bundling Strategy**: Should SCORM packages bundle all assets locally, or reference CDN-hosted assets when online?
- **Local Bundling**: Larger packages, but fully offline-capable
- **CDN References**: Smaller packages, but require internet connectivity
- **Recommendation**: Default to local bundling; provide option for CDN references to reduce package size if customer has reliable connectivity

**SCORM Version Priority**: Which SCORM version should be the default recommendation?
- **SCORM 1.2**: Broadest LMS compatibility, but lacks sequencing
- **SCORM 2004 3rd Edition**: Good balance of features and compatibility
- **SCORM 2004 4th Edition**: Latest standard, best features, may have limited LMS support
- **Recommendation**: Default to SCORM 2004 3rd Edition; offer 1.2 for legacy LMS compatibility

**Suspend Data Compression**: Should suspend data be compressed to fit within SCORM 1.2's 4096-character limit?
- **Pros**: Enables richer progress state storage
- **Cons**: Adds complexity, potential compatibility issues
- **Recommendation**: Use JSON minification (no whitespace) for SCORM 1.2; avoid compression unless customer requests it

### Content Packaging Scope

**Multi-Game Collections**: Should teachers be able to create custom multi-game SCORM packages?
- **Use Case**: Package multiple related games into single SCORM import
- **Challenge**: Complex manifest, larger package size
- **Recommendation**: Support custom collections in Phase 2 based on customer demand

**Localization in Packages**: Should SCORM packages include multiple language versions of game content?
- **Pros**: Single package works for multilingual classrooms
- **Cons**: Larger package size, increased complexity
- **Recommendation**: Single language per package initially; support multi-language packages if demand warrants

**Dynamic vs. Static Content**: Should SCORM packages include dynamic features (randomized questions, adaptive difficulty)?
- **Dynamic**: Better learning experience, but complicates SCORM implementation
- **Static**: Simpler, more reliable, but less engaging
- **Recommendation**: Start with static content; explore dynamic features using JavaScript-based randomization (no server calls)

### LMS Compatibility

**Platform-Specific Testing**: Which LMS platforms should be prioritized for SCORM compatibility testing?
- **Canvas**: Large market share in higher ed and K-12
- **Moodle**: Open-source, widely used globally
- **Blackboard**: Significant market share in higher ed
- **D2L Brightspace**: Growing market share
- **Schoology**: K-12 focus
- **Recommendation**: Test and certify compatibility with Canvas, Moodle, and Blackboard for initial launch; expand based on customer LMS distribution

**SCORM Cloud Integration**: Should MLC integrate with SCORM Cloud for testing and validation?
- **Pros**: Industry-standard testing, customer confidence
- **Cons**: Additional cost, dependency on third-party service
- **Recommendation**: Use SCORM Cloud for internal testing and validation; provide customers with SCORM Cloud testing links for pre-import verification

**LMS-Specific Workarounds**: How should MLC handle known LMS quirks and non-standard behaviors?
- **Document quirks**: Maintain LMS compatibility matrix
- **Implement workarounds**: Add platform detection and workarounds in SCORM API wrapper
- **Recommendation**: Document known issues; implement critical workarounds only; encourage customers to contact their LMS vendor for standard compliance

### Business and Licensing

**Licensing Terms for SCORM Packages**: How should MLC license SCORM packages to prevent unauthorized redistribution?
- **DRM**: Technical protection (complex, may break in some LMS platforms)
- **License Agreement**: Legal terms only (easier, relies on customer compliance)
- **Watermarking**: Embed organization identifier in packages for traceability
- **Recommendation**: Include license agreement in package README; embed organization ID in manifest metadata for traceability; avoid DRM due to LMS compatibility concerns

**Pricing Model**: Should SCORM package generation be:
- **Included in subscription**: No additional cost, drives adoption
- **Add-on fee**: Per-package or per-seat fee for SCORM exports
- **Premium tier only**: Available only to Ensemble tier and above
- **Recommendation**: Include SCORM generation in Ensemble tier and above; Solo/Prelude users excluded or pay per-package fee

**Package Updates**: If MLC updates game content, should existing SCORM packages be invalidated or updated?
- **Invalidate**: Customers must re-generate packages to get updates
- **Auto-update**: Packages automatically reference latest content (requires CDN, not fully offline)
- **Notification**: Notify customers when content updates are available, prompt re-download
- **Recommendation**: Packages are static snapshots; notify customers of significant content updates and recommend re-generation

### Future Enhancements (Out of Scope for Initial Release)

**CMI5 Profile**: Implement CMI5 (xAPI profile for launched content) as the modern successor to SCORM. CMI5 combines benefits of SCORM (packaging) and xAPI (rich analytics).

**SCORM Package Marketplace**: Create a marketplace where third-party publishers can distribute MLC-compatible SCORM packages.

**Interactive Package Builder**: Drag-and-drop interface for creating custom SCORM packages from game library.

**Adaptive SCORM Packages**: Generate packages with adaptive difficulty based on embedded learner model (requires JavaScript-based adaptation, no server calls).

**SCORM Analytics Dashboard**: Track SCORM package usage, import rates, completion rates across customer LMS platforms.

**Bulk Package Generation**: System Admin tool to generate SCORM packages for all sequences in a library at once.

---

## Implementation Notes

**Phase 1 (MVP)**:
- SCORM 1.2 and SCORM 2004 3rd Edition support
- Single game packaging (all stages or selected stages)
- Single assignment packaging from sequences
- Basic SCORM API wrapper for communication with LMS
- Manual package generation via UI
- Pre-signed download URLs with 24-hour expiration

**Phase 2 (Enhanced)**:
- SCORM 2004 4th Edition support
- Full sequence packaging with sequencing rules
- Custom multi-game collections
- Advanced scoring modes (average, cumulative)
- Bulk package generation for System Admins
- SCORM Cloud integration for testing

**Phase 3 (Advanced)**:
- CMI5 profile support (xAPI-based successor to SCORM)
- Dynamic package features (randomized questions, adaptive difficulty)
- Multi-language packages
- SCORM package analytics dashboard
- Scheduled automated package generation
- Interactive package builder UI

**Testing Requirements**:
- ADL SCORM Test Suite for conformance validation
- Manual testing in Canvas, Moodle, and Blackboard LMS
- Automated validation of imsmanifest.xml against SCORM schemas
- Load testing for package generation (100 concurrent package requests)
- End-to-end testing: package generation → LMS import → gameplay → score reporting

**Documentation Deliverables**:
- SCORM Package User Guide (for teachers and admins)
- LMS Import Instructions (Canvas, Moodle, Blackboard)
- SCORM Technical Reference (for developers and LMS admins)
- Troubleshooting Guide for common SCORM issues
- SCORM Compatibility Matrix (tested LMS platforms and versions)