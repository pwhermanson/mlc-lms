# CSV Spec Games

This document defines the CSV file format specification for importing game data into the MLC-LMS platform. It specifies the exact structure, field requirements, validation rules, and formatting guidelines for CSV files used to create or update game records in bulk.

> **Related Documents**: For bulk import workflow and UI, see [Games Importer](../08-games-registry-and-authoring/games-importer.md). For game data model and field definitions, see [Game Model](../08-games-registry-and-authoring/game-model.md). For metadata field details, see [Games Metadata](../08-games-registry-and-authoring/games-metadata.md).

## Purpose

This specification enables:

- **Efficient Game Migration**: Content administrators can migrate hundreds of games from MLC 4.0 to MLC 5.0 rapidly using CSV files generated from legacy system exports
- **Standardized Data Format**: Ensures consistent CSV structure for game imports, reducing errors and support overhead during content migration
- **Automated Metadata Population**: Enables automatic generation of slugs, stage configurations, and default metadata when not explicitly provided
- **Data Validation**: Provides clear validation rules to catch errors before import, ensuring data quality and referential integrity
- **Legacy System Compatibility**: Maintains compatibility with MLC 4.0 "Games import master" format (GAM-XXXX numbering) while supporting MLC 5.0 enhancements
- **Bulk Content Operations**: Allows content editors to create, update, or deprecate multiple games in a single validated transaction

This specification is narrowly focused on the CSV file format itself. For the bulk operations workflow, import UI, preview functionality, and batch processing details, see [Games Importer](../08-games-registry-and-authoring/games-importer.md).

## Scope

**Included:**
- CSV file structure and formatting requirements
- Required and optional column specifications
- Data type definitions and constraints for all game fields
- Field-level validation rules and error codes
- Legacy ID mapping from MLC 4.0 to MLC 5.0 format
- Stage configuration patterns and auto-generation rules
- Asset URL formatting and validation requirements
- Concept tag and taxonomy code specifications
- Character encoding and special character handling
- Example CSV templates and sample data rows
- Validation error codes and messages

**Excluded:**
- Bulk operations UI and workflow (see [Games Importer](../08-games-registry-and-authoring/games-importer.md))
- Import batch processing and job management (see [Bulk Ops Jobs](./bulk-ops-jobs.md))
- Game data model beyond CSV fields (see [Game Model](../08-games-registry-and-authoring/game-model.md))
- Asset upload pipeline and CDN management (see [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md))
- Sequence and assignment CSV specs (see [CSV Spec Courses Assignments](./csv-spec-courses-assignments.md))
- API endpoints for import operations (see [REST Endpoints](../18-architecture/rest-endpoints.md))

## Data model (if applicable)

### CSV Column Definitions

The game import CSV supports the following columns. Column order is flexible, but the first row **must** contain column headers that exactly match these names (case-insensitive).

#### Required Columns

| Column Name | Data Type | Max Length | Description | Example |
|-------------|-----------|------------|-------------|---------|
| `name` | String | 200 | Game title as displayed to students and teachers | `Grand Staff Guide Notes 1` |
| `slug` | String | 200 | URL-safe identifier (lowercase, hyphens only) | `grand-staff-guide-notes-1` or leave blank for auto-generation |
| `instrument` | String | 50 | Primary instrument focus | `piano`, `general_music`, `guitar`, `voice`, `band`, `orchestra` |
| `category` | String | 100 | Primary pedagogical category | `Staff`, `Rhythm`, `Intervals`, `Chords`, `Scales`, `Symbols`, `Keyboard Elements` |
| `level` | String | 50 | Difficulty level | `beginner`, `elementary`, `intermediate`, `advanced`, `mastery` |
| `difficulty` | Integer | - | Granular difficulty rating within level (1-10 scale) | `2` |

**Notes:**
- If `slug` is blank, the system will auto-generate from `name` using kebab-case normalization
- Auto-generated slugs have duplicates resolved by appending numeric suffixes: `grand-staff-1`, `grand-staff-2`
- `category` must match a valid category from the game taxonomy (see [Game Taxonomy](../07-content-architecture/game-taxonomy.md))

#### Optional Core Metadata Columns

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `legacy_id` | String | 50 | Original MLC 4.0 game identifier for migration tracking | `GAM-0950` | `null` |
| `description` | String | 2000 | Full text description of game mechanics and learning objectives | `Students identify notes on the grand staff using guide notes as anchors` | Empty string |
| `short_description` | String | 200 | Brief summary for catalog cards and tooltips | `Learn to identify notes using guide notes` | Empty string |
| `status` | String | 20 | Lifecycle status | `draft`, `live`, `deprecated` | `draft` |
| `mode` | String | 50 | Sensory learning mode | `visual`, `aural`, `kinesthetic`, `mixed` | `visual` |
| `device_profile` | String | 50 | Device compatibility profile | `any`, `desktop_preferred`, `touch_friendly`, `midi_required` | `any` |
| `keyboard_mode` | String | 50 | Keyboard interaction type | `non_keyboard`, `virtual_keyboard`, `midi_required`, `midi_optional` | `non_keyboard` |
| `concept_tags` | String | 1000 | Comma-separated list of concept slugs | `guide_notes,treble_staff,bass_staff` | Empty string |
| `standards` | String | 1000 | Comma-separated list of standard codes | `custom:mlc:L1.staff.guide_notes,NAFME:MU:Cr1.1.1` | Empty string |
| `pedagogical_notes` | String | 2000 | Free-text notes for teachers about how to use the game | `Best used after introducing treble and bass clefs separately` | Empty string |

#### Asset URL Columns

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `html5_bundle_url` | URL | 500 | CDN URL to game HTML5 bundle | `https://cdn.mlc/games/G-01542/v3/index.html` | `null` |
| `thumbnail_url` | URL | 500 | CDN URL to game thumbnail image (recommended: 400x300px PNG or JPG) | `https://cdn.mlc/thumbs/G-01542.png` | `null` |
| `audio_sprite_url` | URL | 500 | CDN URL to audio sprite file for game sounds | `https://cdn.mlc/audio/G-01542-sprite.mp3` | `null` |
| `game_path` | String | 500 | Legacy game path from MLC 4.0 (for migration mapping) | `/game_uploads/GAM-0950-2E/index.html` | `null` |

**Notes:**
- Asset URLs are validated via HTTP HEAD request during import validation
- Missing or inaccessible assets generate warnings but do not block import
- Games with missing critical assets are set to `status: draft` until assets are uploaded

#### Game Feature Flags

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `active` | Boolean | - | Is game enabled and accessible (legacy MLC 4.0 field) | `true`, `false`, `1`, `0`, `Y`, `N` | `true` |
| `keyboard` | Boolean | - | Requires on-screen keyboard interaction (legacy field) | `true` | `false` |
| `timed` | Boolean | - | Game has time limit constraints | `true` | `false` |
| `midi_capable` | Boolean | - | Supports MIDI keyboard input | `true` | `false` |
| `mic_supported` | Boolean | - | Supports microphone audio input | `false` | `false` |
| `solfege_text` | Boolean | - | Uses solfege syllables instead of letter names | `true` | `false` |
| `free_trial_eligible` | Boolean | - | Available in free trial mode | `true` | `false` |

#### Scoring and Target Columns

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `target_score` | Integer | - | Default target score for completion (0-100 scale) | `85` | `80` |
| `pass_threshold` | Integer | - | Minimum score to pass and achieve mastery (0-100 scale) | `70` | `60` |
| `time_limit_seconds` | Integer | - | Default time limit in seconds (null for untimed) | `180` | `null` |

**Notes:**
- These values serve as defaults for all stages unless stage-specific overrides are provided
- Individual stages can override these values (see Stage Configuration Columns below)

#### Stage Configuration Columns

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `stages` | JSON String | 500 | JSON array defining which stage types to create | `["learn","play","quiz","review"]` | Auto-generated based on level |
| `learn_enabled` | Boolean | - | Create Learn stage | `true` | `true` |
| `play_enabled` | Boolean | - | Create Play stage | `true` | `true` |
| `quiz_enabled` | Boolean | - | Create Quiz stage | `true` | `true` |
| `challenge_enabled` | Boolean | - | Create Challenge stage (optional) | `false` | `false` |
| `review_enabled` | Boolean | - | Create Review stage | `true` | `true` |
| `learn_target_score` | Integer | - | Override target score for Learn stage | `null` | `null` |
| `play_target_score` | Integer | - | Override target score for Play stage | `70` | Game default |
| `quiz_target_score` | Integer | - | Override target score for Quiz stage | `85` | Game default |
| `challenge_target_score` | Integer | - | Override target score for Challenge stage | `null` | `null` |
| `review_target_score` | Integer | - | Override target score for Review stage | `80` | Game default |

**Notes:**
- If `stages` column is provided (JSON format), it takes precedence over individual `*_enabled` columns
- If both `stages` and `*_enabled` columns are omitted, system auto-generates stages based on `level`:
  - **beginner/elementary**: Creates `learn`, `play`, `quiz`, `review`
  - **intermediate/advanced**: Creates `play`, `quiz`, `challenge`, `review`
  - **mastery**: Creates `play`, `challenge` only

#### Legacy MLC 4.0 Compatibility Columns

| Column Name | Data Type | Max Length | Description | Example | Default |
|-------------|-----------|------------|-------------|---------|---------|
| `game_number` | String | 50 | Legacy game number (same as `legacy_id`) | `GAM-1680-2E` | `null` |
| `stage_id` | String | 50 | Legacy stage identifier (1=Learn, 2=Play, 3=Quiz, 4=Challenge, 5=Review) | `GAM-0950-2` | Derived from `legacy_id` |
| `skill_level` | String | 10 | Legacy skill level code (mapped to `level`) | `P` (Primer), `1`, `2A`, `3B` | Mapped to `level` |
| `skill_area_id` | String | 100 | Legacy skill area identifier | `Piano Key Names` | Mapped to `concept_tags` |
| `skill_type` | String | 10 | Legacy V=Visual, A=Aural | `V`, `A` | Mapped to `mode` |
| `language` | String | 10 | Language code | `E` (English), `S` (Spanish) | `en` |

**Notes:**
- Legacy columns are provided for migration from MLC 4.0 "Games import master" format
- These columns are automatically mapped to MLC 5.0 equivalents during import
- If both legacy and new columns are provided, new columns take precedence

### CSV File Format Requirements

**File Encoding:**
- UTF-8 encoding required (with or without BOM)
- Line endings: Unix (LF), Windows (CRLF), or Mac (CR) all supported

**CSV Structure:**
- First row must contain column headers (exact name match, case-insensitive)
- Subsequent rows contain game data (one game per row)
- Column order is flexible
- Empty rows are skipped
- Rows with all empty required fields are skipped with warning

**Field Formatting:**
- Text fields: Enclosed in double quotes if containing commas, quotes, or line breaks
- Boolean fields: Accept `true`/`false`, `1`/`0`, `Y`/`N`, `yes`/`no` (case-insensitive)
- Integer fields: Numeric digits only, no commas or decimal points
- JSON fields: Valid JSON string, enclosed in double quotes and escaped
- URL fields: Must start with `http://` or `https://`
- Date fields: ISO 8601 format `YYYY-MM-DD`

**Special Characters:**
- Quotes within text fields must be escaped with double quotes: `"He said ""hello"""`
- Newlines within text fields are preserved when field is quoted
- Accented and Unicode characters supported in UTF-8 encoding

### Slug Auto-Generation Rules

When `slug` is blank or omitted, the system automatically generates slugs using the following algorithm:

**Basic Pattern:**
```
{name-in-kebab-case}
```

**Example:**
- Game Name: `Grand Staff Guide Notes 1`
- Generated Slug: `grand-staff-guide-notes-1`

**Normalization Steps:**
1. Convert entire name to lowercase
2. Replace spaces with hyphens
3. Remove all special characters except hyphens and alphanumeric
4. Collapse multiple consecutive hyphens into single hyphen
5. Trim leading and trailing hyphens
6. Check for duplicate slugs in the system

**Duplicate Resolution:**
If the generated slug already exists, the system appends a numeric suffix and retries:

- First occurrence: `grand-staff-guide-notes-1`
- Second occurrence: `grand-staff-guide-notes-1-2`
- Third occurrence: `grand-staff-guide-notes-1-3`

**Manual Slug Best Practices:**
- Use lowercase letters only
- Use hyphens to separate words (not underscores or spaces)
- Keep slugs concise but descriptive (target: 3-6 words)
- Include key learning concept in slug when possible
- Avoid redundant words like "game" (implied by context)

### Legacy ID Mapping

**MLC 4.0 Game ID Format:**
```
GAM-XXXX-YZ
```
Where:
- `GAM` = Game prefix
- `XXXX` = 4-digit game number (e.g., `0950`, `1680`)
- `Y` = Stage number: 1=Learn, 2=Play, 3=Quiz, 4=Challenge, 5=Review
- `Z` = Language code: E=English, S=Spanish, etc.

**MLC 5.0 Game ID Format:**
```
G-XXXXX
```
Where:
- `G` = Game prefix
- `XXXXX` = Sequential 5-digit identifier assigned by system

**Migration Mapping:**
During CSV import, if `legacy_id` is provided:
1. System checks if game with that `legacy_id` already exists
2. If exists, update operation performed (upsert mode)
3. If not exists, new game created with new `game_id` assigned
4. Legacy ID stored in `legacy_id` field for reference and audit

**Stage ID Mapping:**
Legacy stage identifiers are converted to MLC 5.0 format:
- MLC 4.0: `GAM-0950-2E` (Play stage in English)
- MLC 5.0: `G-01542-play` (stage type as suffix)

### Example CSV Templates

#### Minimal Required Columns (New Game Creation)

```csv
name,slug,instrument,category,level,difficulty
"Grand Staff Guide Notes 1","grand-staff-guide-notes-1","piano","Staff","beginner",2
"Interval Rockets 1","interval-rockets-1","piano","Intervals","intermediate",4
"Rhythm Pop 1","rhythm-pop-1","general_music","Rhythm","elementary",2
```

#### Full Metadata Import (Comprehensive)

```csv
name,slug,instrument,category,level,difficulty,legacy_id,description,concept_tags,html5_bundle_url,thumbnail_url,target_score,stages
"Grand Staff Guide Notes 1","grand-staff-guide-notes-1","piano","Staff","beginner",2,"GAM-0950","Students identify notes on the grand staff using guide notes as anchors for spatial relationships","guide_notes,treble_staff,bass_staff,grand_staff","https://cdn.mlc/games/G-01542/v3/index.html","https://cdn.mlc/thumbs/G-01542.png",85,"[""learn"",""play"",""quiz"",""review""]"
"Interval Rockets 1","interval-rockets-1","piano","Intervals","intermediate",4,"GAM-1380","Identify melodic intervals visually on staff with rocket animation for correct answers","intervals,melodic_intervals,staff_reading","https://cdn.mlc/games/G-01543/v2/index.html","https://cdn.mlc/thumbs/G-01543.png",80,"[""play"",""quiz"",""challenge"",""review""]"
```

#### Legacy MLC 4.0 Migration Format

```csv
legacy_id,game_number,title,description,thumbnail_img,Category,skill_level,skill_type,active,keyboard,timed,solfege_text,midi_capable,language,game_path,target_score
"GAM-1680","GAM-1680-2E","LetterFly 3 CDEFGAB - Play","White Keys--Identify and play on the onscreen keyboard.","/images/LetterFly.jpg","Keyboard elements","P","V",1,1,0,0,0,"E","/game_uploads/GAM-1680-2E/index.html",300
```

**Note:** This legacy format is automatically translated to MLC 5.0 format during import. The `title` field maps to `name`, `thumbnail_img` to `thumbnail_url` (with CDN prefix added), etc.

## Behavior and rules

### Validation Rules

The import system enforces validation at multiple levels to ensure data quality and system integrity.

#### Field-Level Validation

**Required Field Validation:**
- Error Code: `ERR_MISSING_REQUIRED_FIELD`
- Message: `Row {row_num}, field '{field_name}': Required field is missing or empty`
- All required columns must be present and non-empty
- Whitespace-only values treated as empty

**String Length Validation:**
- Error Code: `ERR_FIELD_TOO_LONG`
- Message: `Row {row_num}, field '{field_name}': Value exceeds maximum length of {max_length} characters`
- All string fields must not exceed specified maximum length

**Data Type Validation:**
- Error Code: `ERR_INVALID_DATA_TYPE`
- Message: `Row {row_num}, field '{field_name}': Expected {expected_type}, got {actual_type}`
- Integer fields must contain only numeric digits
- Boolean fields must be valid boolean representation
- URL fields must be well-formed with `http://` or `https://` protocol
- JSON fields must be valid JSON syntax

**Enumerated Value Validation:**
- Error Code: `ERR_INVALID_ENUM_VALUE`
- Message: `Row {row_num}, field '{field_name}': Invalid value '{value}'. Allowed values: {allowed_values}`
- Fields with fixed value sets must match allowed values exactly:
  - `instrument`: piano, general_music, guitar, voice, band, orchestra, choir, keyboard, brass, woodwind, percussion, strings
  - `category`: Staff, Rhythm, Intervals, Chords, Scales, Symbols, Keyboard Elements, Melody, Harmony, Musical Terms, Key Signatures, Meter, Pitch, Tonal Memory
  - `level`: beginner, elementary, intermediate, advanced, mastery
  - `mode`: visual, aural, kinesthetic, mixed
  - `status`: draft, live, deprecated
  - `device_profile`: any, desktop_preferred, touch_friendly, midi_required, mic_required
  - `keyboard_mode`: non_keyboard, virtual_keyboard, midi_required, midi_optional

**URL Accessibility Validation:**
- Error Code: `WARN_ASSET_NOT_FOUND`
- Severity: Warning (allows import to proceed)
- Message: `Row {row_num}, field '{field_name}': Asset URL returned {http_status}. Game may not be playable until asset is uploaded.`
- System performs HTTP HEAD request to verify asset accessibility
- Timeout after 5 seconds per URL
- 404 or 5xx responses generate warnings but do not block import

#### Cross-Record Validation

**Duplicate Slug Detection:**
- Error Code: `ERR_DUPLICATE_SLUG`
- Message: `Row {row_num}, field 'slug': Duplicate slug '{slug}' found in row {other_row_num}`
- Slugs must be unique within CSV file
- If updating existing games (upsert mode), slug collision with different game ID is error

**Duplicate Name Detection:**
- Error Code: `WARN_DUPLICATE_NAME`
- Severity: Warning (allows import to proceed)
- Message: `Row {row_num}, field 'name': Duplicate name '{name}' found in row {other_row_num}. Consider making names unique for better discoverability.`
- Names should be unique but duplicates are allowed with warning

**Duplicate Legacy ID Detection:**
- Error Code: `ERR_DUPLICATE_LEGACY_ID`
- Message: `Row {row_num}, field 'legacy_id': Duplicate legacy ID '{legacy_id}' found in row {other_row_num}. Legacy IDs must be unique for migration integrity.`
- Legacy IDs must be unique across all games (ensures 1:1 mapping from MLC 4.0)

#### Referential Integrity Validation

**Concept Tag Validation:**
- Error Code: `WARN_INVALID_CONCEPT_TAG`
- Severity: Warning (allows import to proceed)
- Message: `Row {row_num}, field 'concept_tags': Concept tag '{tag}' not found in taxonomy. Game will be created but may not appear in filtered searches.`
- Concept tags should reference existing taxonomy entries (see [Game Taxonomy](../07-content-architecture/game-taxonomy.md))
- Invalid tags generate warnings but do not block import
- System may suggest similar valid tags based on fuzzy matching

**Standards Code Validation:**
- Error Code: `WARN_INVALID_STANDARD`
- Severity: Warning
- Message: `Row {row_num}, field 'standards': Standard code '{code}' not found in standards registry.`
- Standards codes should reference existing curriculum standards
- Invalid codes generate warnings but do not block import

**Category Validation:**
- Error Code: `ERR_INVALID_CATEGORY`
- Message: `Row {row_num}, field 'category': Category '{category}' not found in game taxonomy. Use one of: {valid_categories}`
- Category must exactly match a category from the game taxonomy
- This is a hard error that blocks import (category is required for classification)

#### Business Rule Validation

**MIDI Requirements Consistency:**
- Error Code: `ERR_MIDI_INCONSISTENCY`
- Message: `Row {row_num}: Game has keyboard_mode='midi_required' but midi_capable=false. Set midi_capable=true for MIDI-required games.`
- Games with `keyboard_mode: midi_required` must have `midi_capable: true`

**Solfege Mode Consistency:**
- Error Code: `WARN_SOLFEGE_MISSING_TAGS`
- Severity: Warning
- Message: `Row {row_num}: Game has solfege_text=true but concept_tags does not include 'solfege' or related tags. Consider adding appropriate solfege tags.`
- Games with `solfege_text: true` should include solfege-related concept tags

**Score Range Validation:**
- Error Code: `ERR_INVALID_SCORE_RANGE`
- Message: `Row {row_num}: target_score ({target}) must be greater than or equal to pass_threshold ({threshold}). Adjust target_score or pass_threshold.`
- `target_score` must be >= `pass_threshold`
- Both values must be in range 0-100

**Stage Configuration Validation:**
- Error Code: `ERR_MISSING_CORE_STAGES`
- Message: `Row {row_num}: Game must include at least 'play' and 'quiz' stages. Current stages: {current_stages}`
- At minimum, games must have Play and Quiz stages enabled
- Learn is highly recommended but not strictly required
- Challenge is optional
- Review is recommended for spaced repetition

### Import Mode Behavior

**Create Mode:**
- All games in CSV must be new (no matching `game_id`, `slug`, or `legacy_id`)
- Any existing game match results in error and import is blocked
- Use case: Initial migration, onboarding new game collection

**Update Mode:**
- All games in CSV must exist (matched by `game_id`, `legacy_id`, or `slug`)
- Any game not found results in error and import is blocked
- Only fields present in CSV are updated, others remain unchanged
- Use case: Bulk metadata updates, re-tagging, fixing typos

**Upsert Mode:**
- Creates new games if not found, updates existing if found
- Match priority: 1) `game_id`, 2) `legacy_id`, 3) `slug`
- Most flexible but requires careful validation to avoid unintended updates
- Use case: Incremental imports, synchronizing external game database

### Transaction and Rollback

**Atomicity:**
- Default behavior: All-or-nothing transaction
- If any row fails validation or processing, entire import rolls back
- No partial imports allowed in default mode

**Progressive Mode:**
- Optional: Process in batches of 50 rows
- Each batch commits independently
- Failed batches reported but don't affect successful batches
- Allows partial imports with detailed error reporting

**Rollback Window:**
- Imports can be rolled back within 30 days of completion
- Rollback deletes newly created games (if no external references)
- Rollback reverts updated games to pre-import state
- Rollback not available if games have been manually edited since import

## UX requirements (if applicable)

See [Games Importer](../08-games-registry-and-authoring/games-importer.md) for complete UX specification including:
- Upload interface layout and controls
- Validation results panel and error reporting
- Preview panel with diff visualization
- Processing progress indicators
- Completion summary and rollback options

## Telemetry

CSV import operations emit telemetry events tracked by the Games Importer service (see [Games Importer](../08-games-registry-and-authoring/games-importer.md#telemetry)).

**CSV-Specific Properties:**
- `csv_row_count`: Total number of data rows in uploaded CSV
- `csv_column_count`: Number of columns in CSV
- `csv_file_size_bytes`: Size of uploaded CSV file
- `csv_encoding`: Detected character encoding (UTF-8, etc.)
- `legacy_id_count`: Number of rows with legacy IDs (migration tracking)
- `auto_generated_slug_count`: Number of auto-generated slugs
- `validation_error_count`: Total validation errors
- `validation_warning_count`: Total validation warnings

## Permissions

**Read CSV Spec Documentation:**
- Available to all authenticated users

**Upload and Execute CSV Import:**
- System Administrator: Full access to all import modes
- Content Editor: Dry-run mode only, requires System Administrator approval for live imports

**Download CSV Templates:**
- System Administrator and Content Editor

**View Import History:**
- System Administrator: All imports
- Content Editor: Own imports only

See [Games Importer](../08-games-registry-and-authoring/games-importer.md#permissions) for detailed permission matrix.

## Supporting Documents Referenced

This CSV specification draws from the following source documents:

- [Games import master.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20import%20master.xlsx%20-%20Sheet1.csv) — Legacy MLC 4.0 game import master format informing column structure, legacy ID mapping (GAM-XXXX), and field naming conventions
- [Games Loaded.xlsx - Sheet1.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Sheet1.csv) — Legacy game inventory list informing game metadata structure, stage configuration patterns, and asset path mappings
- [Games Loaded.xlsx - Free Trial Games - New Graphics.csv](../00-ORIG-CONTEXT/Games%20Loaded.xlsx%20-%20Free%20Trial%20Games%20-%20New%20Graphics.csv) — Free trial game list informing free trial eligibility flags and graphics asset specifications
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Game implementation and structure specifications informing game model, stage types (LEARN/PLAY/QUIZ/CHALLENGE/REVIEW), and scoring configuration

---

## Dependencies

**Internal Dependencies:**
- [Games Importer](../08-games-registry-and-authoring/games-importer.md): Implements CSV parsing, validation, and import workflow
- [Game Model](../08-games-registry-and-authoring/game-model.md): Defines target game entity structure and validation rules
- [Games Metadata](../08-games-registry-and-authoring/games-metadata.md): Provides field definitions and metadata constraints
- [Game Taxonomy](../07-content-architecture/game-taxonomy.md): Provides valid categories, concept tags, and classification codes
- [Games Assets Pipeline](../08-games-registry-and-authoring/games-assets-pipeline.md): Handles asset upload and CDN URL generation
- [Bulk Ops Jobs](./bulk-ops-jobs.md): Generic background job infrastructure for batch processing

**External Dependencies:**
- CSV parsing library (e.g., Papa Parse, csv-parser, Python csv module)
- URL validation and HTTP client for asset accessibility checks
- UTF-8 encoding detection and conversion library
- JSON parsing and validation library

**Legacy System Dependencies:**
- MLC 4.0 Games Loaded spreadsheet (`Games Loaded.xlsx - Sheet1.csv`)
- MLC 4.0 Games import master (`Games import master.xlsx - Sheet1.csv`)
- Legacy asset path mapping for `/game_uploads/` to CDN URLs
- Category code translation table from MLC 4.0 to MLC 5.0 taxonomy

## Open questions

1. **Auto-Generated Slug Conflicts**  
   **Question:** Should system auto-append incremental suffixes to duplicate slugs or prompt user to manually resolve?  
   **Options:**
   - A) Auto-append numeric suffix (e.g., `game-name-2`) and proceed
   - B) Raise validation error and require manual slug editing  
   **Recommendation:** Option A for efficiency during bulk migration, with warning in import report

2. **Stage Auto-Generation Defaults**  
   **Question:** Should all games default to 5-stage structure (Learn/Play/Quiz/Challenge/Review) or minimal 3-stage (Play/Quiz/Review)?  
   **Options:**
   - A) Always create Learn, Play, Quiz, Review (4 stages) for consistency
   - B) Only create Play, Quiz, Review (3 stages) unless explicitly requested  
   **Recommendation:** Option A with Challenge optional, as Learn stage is pedagogically valuable

3. **Asset URL Validation Timeout**  
   **Question:** What timeout should be used for HTTP HEAD requests to validate asset URLs?  
   **Options:**
   - A) 5 seconds (fast feedback, may miss slow-responding CDNs)
   - B) 15 seconds (more reliable, slower validation)  
   **Recommendation:** Option A (5 seconds) with retry option in preview if warnings appear

4. **Legacy ID Uniqueness Enforcement**  
   **Question:** Should duplicate `legacy_id` values be a hard error or a warning?  
   **Options:**
   - A) Hard error - blocks import until resolved (ensures migration integrity)
   - B) Warning - allows import but flags potential issue  
   **Recommendation:** Option A for migration phase, then relax to warning after MLC 4.0 migration complete

5. **Concept Tag Auto-Suggestion**  
   **Question:** Should system suggest concept tags based on game name and category?  
   **Example:** Game named "Treble Staff Lines" auto-tagged with `treble_staff`, `staff_reading`  
   **Options:**
   - A) No auto-suggestion, require explicit tags in CSV
   - B) Auto-suggest during preview, admin can accept/reject
   - C) Auto-apply tags, admin can edit post-import  
   **Recommendation:** Option B - show suggestions in preview panel with one-click accept for each game

6. **Empty Optional Column Handling**  
   **Question:** Should completely empty optional columns be treated as "keep existing value" or "clear to null" in update mode?  
   **Options:**
   - A) Empty = clear to null (explicit clearing)
   - B) Empty = no change (preserve existing value)  
   **Recommendation:** Option B with special sentinel value `__CLEAR__` to explicitly clear fields

7. **Category Taxonomy Version**  
   **Question:** Should CSV import validate against current taxonomy version or allow legacy category names with auto-mapping?  
   **Options:**
   - A) Strict validation against current taxonomy only
   - B) Accept legacy category names and auto-map to new taxonomy  
   **Recommendation:** Option B with warning message showing mapping applied
