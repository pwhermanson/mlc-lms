# AI Sequence Generator
## STATUS ⚠️ Backburner Feature

This feature is on the backburner pending advances in AI music comprehension. As of October 2025, current LLMs (including GPT-4, Gemini, Claude) struggle to accurately read and understand music notation, making them unreliable for automated curriculum generation. However, this specification documents the intended design for when AI capabilities mature, allowing the system to be implemented rapidly once the underlying technology is ready.

## Purpose

Define an AI-assisted workflow that enables Teachers and Content Editors to rapidly generate pedagogically sound Sequences from method book content or natural language prompts, with comprehensive human oversight and approval. This tool will dramatically reduce the time required to create curriculum-aligned sequences while maintaining the high educational quality standards established in [Pedagogy Principles](../00-foundations/pedagogy-principles.md).

For the underlying sequence data structure that the AI will generate, see [Sequence Model](sequence-model.md). For manual authoring workflows that this feature augments (but does not replace), see [Sequence Authoring](sequence-authoring.md).

## Vision and Future Possibilities

### Current AI Limitations (October 2025)
Testing with multiple leading AI models has revealed significant limitations:
- **Music notation reading**: Inconsistent recognition of staff notation, clefs, accidentals, and rhythmic values
- **Pedagogical sequencing**: Difficulty understanding optimal concept progression and prerequisite relationships
- **Context understanding**: Limited ability to parse method book layout, unit structure, and implied skill progressions
- **Spatial reasoning**: Poor performance with keyboard diagrams, finger patterns, and hand position illustrations

### Expected Future Capabilities
When AI music comprehension matures (estimated 2026-2028), the system should be able to:
- **Parse method books**: Extract structured curriculum data from PDF scans of popular method series
- **Understand pedagogy**: Recognize concept dependencies and optimal learning progressions
- **Map to games**: Intelligently match pedagogical goals to the 400+ games in the MLC library
- **Generate sequences**: Propose complete curriculum pathways with appropriate gating and review scheduling
- **Learn from outcomes**: Improve recommendations based on student performance data and teacher feedback

### Strategic Value
- **Publisher partnerships**: Rapidly create MLC-aligned sequences for new method book editions
- **Teacher empowerment**: Enable teachers to generate custom sequences from any curriculum source
- **Personalization**: Generate student-specific remediation sequences based on performance patterns
- **Scale**: Support hundreds of method books without manual curriculum mapping
- **Competitive advantage**: First-to-market with AI-assisted music curriculum generation

## Scope

### Included
- AI model integration architecture and prompt engineering
- Method book content ingestion and normalization
- Natural language prompt processing for custom sequences
- Game matching algorithms using concept tags and metadata
- Sequence assembly with pedagogically sound structure
- Teacher review and approval interface
- Iterative refinement workflow with AI-human collaboration
- Confidence scoring and explainability for AI recommendations
- Feedback loop for continuous model improvement
- Safety guardrails and validation rules
- Telemetry for generation quality and acceptance rates
- Integration with existing sequence authoring and library systems

### Excluded
- General AI infrastructure and hosting (handled by platform team)
- Method book licensing and copyright clearance (legal/business function)
- Manual sequence authoring tools (see [Sequence Authoring](sequence-authoring.md))
- Assignment delivery and student progression (see [Assignment Generation](../10-assignments-engine/assignment-generation.md))
- Student performance analytics (see [Teacher Reports](../04-teacher-experience/teacher-reports.md))

## Technical Architecture (Future Implementation)

### AI Model Requirements

**Primary capabilities needed:**
- **Vision models**: OCR and layout understanding for method book PDFs
- **Music notation parsing**: Specialized models for staff notation recognition (e.g., Audiveris, OMR systems)
- **Natural language understanding**: Process teacher prompts and skill descriptions
- **Reasoning and planning**: Construct logical learning progressions
- **Retrieval augmented generation (RAG)**: Query game library with semantic similarity

**Candidate model stack (as of October 2025, will evolve):**
- **Vision**: GPT-4 Vision, Claude 3 Opus, or Google Gemini Pro Vision for layout parsing
- **Music OCR**: Dedicated OMR (Optical Music Recognition) models like Audiveris or Oemer
- **Orchestration**: LangChain or LlamaIndex for RAG pipeline and prompt chaining
- **Embeddings**: OpenAI text-embedding-3 or sentence-transformers for semantic game search
- **Reasoning**: Latest frontier models (GPT-5, Claude 4, Gemini 2.0 when available)

### System Components

```
┌─────────────────────────────────────────────────────────┐
│  AI Sequence Generator Service                         │
├─────────────────────────────────────────────────────────┤
│  ┌──────────────────┐  ┌──────────────────┐            │
│  │  Input Processor │  │  Game Retriever  │            │
│  │  - Book Parser   │  │  - Vector Search │            │
│  │  - Prompt NLU    │  │  - Concept Graph │            │
│  └──────────────────┘  └──────────────────┘            │
│                                                         │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Sequence Assembly Engine                        │  │
│  │  - Progression Logic                             │  │
│  │  - Gating Rules                                  │  │
│  │  - Review Scheduling                             │  │
│  └──────────────────────────────────────────────────┘  │
│                                                         │
│  ┌──────────────────┐  ┌──────────────────┐            │
│  │  Validator       │  │  Explainer       │            │
│  │  - Safety Rules  │  │  - Confidence    │            │
│  │  - Completeness  │  │  - Rationale     │            │
│  └──────────────────┘  └──────────────────┘            │
└─────────────────────────────────────────────────────────┘
         │                          │
         ▼                          ▼
┌─────────────────┐        ┌─────────────────┐
│  Games Registry │        │  Sequence Store │
│  (Read Only)    │        │  (Write)        │
└─────────────────┘        └─────────────────┘
```

## Data Model

### Generation Request

```json
{
  "request_id": "gen-req-uuid-v4",
  "created_by_user_id": "U-12345",
  "org_id": "ORG-789",
  "request_type": "method_book", // or "natural_language" or "student_remediation"
  "input": {
    "method_book": {
      "publisher": "Faber Piano Adventures",
      "series_name": "Piano Adventures",
      "book_level": "Level 2B",
      "edition_year": 2018,
      "content_source": "pdf_upload_id_abc123", // or extracted text
      "pages_to_process": [4, 5, 6, 7, 8, 9, 10, 11], // specific page range
      "focus_concepts": ["cross_hand_arpeggios", "major_scales_C_G_F"] // optional filter
    },
    "natural_language_prompt": null,
    "student_context": null
  },
  "generation_config": {
    "target_instrument": "piano",
    "target_age_range": "8-12",
    "estimated_duration_weeks": 8,
    "difficulty_range": [2, 5], // 1-10 scale
    "include_midi_games": true,
    "include_aural_games": true,
    "review_scheduling": "standard_7_day"
  },
  "status": "processing", // draft | processing | ready_for_review | approved | rejected
  "created_at": "2025-10-12T14:30:00Z",
  "completed_at": null
}
```

### Generated Sequence Proposal

```json
{
  "proposal_id": "prop-uuid-v4",
  "request_id": "gen-req-uuid-v4",
  "generated_at": "2025-10-12T14:32:15Z",
  "model_version": "gpt-5-music-2026-03",
  "confidence_score": 0.87, // 0.0 - 1.0
  "sequence": {
    "name": "Piano Adventures 2B - Units 1-3",
    "visibility": "method_aligned",
    "instrument": "piano",
    "metadata": {
      "publisher": "Faber Piano Adventures",
      "book_level": "Level 2B",
      "edition_year": 2018,
      "target_age": "8-12",
      "estimated_hours": 24
    },
    "groups": [
      {
        "group_id": "G-01",
        "name": "Unit 1: Cross-Hand Arpeggios",
        "source_pages": "4-11",
        "assignments": [
          {
            "assignment_id": "A-01",
            "name": "Introduction to Broken Chords",
            "source_lesson": "Page 4-5: Listen and Play",
            "ai_rationale": "Introduces basic cross-hand movement before adding rhythmic complexity",
            "confidence": 0.92,
            "steps": [
              {
                "step_id": "S-01",
                "element_type": "game",
                "element_id": "G-01542",
                "game_name": "Grand Staff Guide Notes A",
                "stage": "learn",
                "ai_rationale": "Reinforces staff reading foundation needed for arpeggio patterns",
                "confidence": 0.88,
                "gating": { "min_attempts": 1 },
                "targets": null
              },
              {
                "step_id": "S-02",
                "element_type": "game",
                "element_id": "G-01542",
                "stage": "play",
                "confidence": 0.91,
                "gating": { "min_attempts": 1 },
                "targets": { "target_score": 70, "pass_threshold": 60 }
              },
              {
                "step_id": "S-03",
                "element_type": "game",
                "element_id": "G-01542",
                "stage": "quiz",
                "confidence": 0.94,
                "gating": { "require_previous_steps": true },
                "targets": { "target_score": 85, "pass_threshold": 80 }
              }
            ],
            "alternative_games": [
              {
                "game_id": "G-01020",
                "game_name": "Pattern Reading 1",
                "rationale": "Alternative if student needs more stepwise reading practice",
                "confidence": 0.76
              }
            ]
          }
        ]
      }
    ]
  },
  "validation_results": {
    "all_games_exist": true,
    "all_stages_enabled": true,
    "progression_valid": true,
    "warnings": [
      "Game G-01542 requires MIDI keyboard; ensure student device compatibility"
    ],
    "suggestions": [
      "Consider adding review step after Assignment A-03 for long-term retention"
    ]
  },
  "teacher_feedback": null, // populated during review
  "approval_status": "pending_review",
  "approved_by_user_id": null,
  "approved_at": null
}
```

### Method Book Content Schema

Structured representation extracted from method books:

```json
{
  "book": {
    "publisher": "Hal Leonard",
    "series_name": "Student Piano Library",
    "book_level": "Book 2",
    "edition_year": 2015,
    "isbn": "978-1-4950-1234-5",
    "target_age": "8-12",
    "instrument": "piano"
  },
  "units": [
    {
      "unit_id": "unit-1",
      "name": "Review and Preview",
      "pages": "4-11",
      "estimated_weeks": 2,
      "prerequisite_concepts": ["guide_notes", "basic_rhythm"],
      "items": [
        {
          "item_id": "item-1-1",
          "lesson_title": "Treble G, Middle C, Bass F",
          "page_numbers": [4, 5],
          "skill_text": "Identify Treble G, Middle C, Bass F on the staff",
          "concept_tags": ["guide_notes", "treble_staff", "bass_staff", "note_reading"],
          "visual_or_aural": "visual",
          "keyboard_required": false,
          "difficulty_estimate": 2, // 1-10 scale
          "pedagogical_notes": "Foundation for all staff reading; critical mastery concept",
          "extracted_by": "ai_vision_model",
          "confidence": 0.89
        },
        {
          "item_id": "item-1-2",
          "lesson_title": "Steps and Skips",
          "page_numbers": [6, 7],
          "skill_text": "Identify melodic motion as steps up, steps down, or skips",
          "concept_tags": ["melodic_contour", "intervals", "aural_skills"],
          "visual_or_aural": "aural",
          "keyboard_required": false,
          "difficulty_estimate": 3,
          "pedagogical_notes": "Prepares for interval recognition and sightreading",
          "extracted_by": "ai_vision_model",
          "confidence": 0.85
        }
      ]
    }
  ]
}
```

### Teacher Feedback Schema

```json
{
  "feedback_id": "fb-uuid-v4",
  "proposal_id": "prop-uuid-v4",
  "user_id": "U-12345",
  "submitted_at": "2025-10-12T15:45:00Z",
  "overall_rating": 4, // 1-5 stars
  "overall_comment": "Good progression but needs more rhythm work in Unit 2",
  "step_edits": [
    {
      "assignment_id": "A-02",
      "action": "insert_step",
      "position": 2,
      "new_step": {
        "element_type": "game",
        "element_id": "G-03120",
        "stage": "play",
        "reason": "Students need more eighth-note practice before syncopation"
      }
    },
    {
      "step_id": "S-07",
      "action": "replace_game",
      "new_element_id": "G-02840",
      "reason": "Pick the Pattern 2 better aligns with book examples"
    },
    {
      "assignment_id": "A-05",
      "action": "reorder",
      "new_order": [3, 1, 2, 4],
      "reason": "Quiz should come after both practice games"
    }
  ],
  "accepted": true,
  "published_sequence_id": "SEQ-FABR-2B-001"
}
```

## Workflow and Process

### Method Book Ingestion Workflow

**Step 1: Content Upload and Extraction**
1. Teacher or Content Editor uploads method book PDF or provides structured text
2. System extracts:
   - Table of contents and unit structure
   - Page images with music notation
   - Skill statements and lesson descriptions
   - Supplementary text (instructions, performance notes)
3. OCR and vision models parse visual content
4. NLP models extract pedagogical intent and concept tags
5. Human reviewer validates extraction quality and corrects errors

**Step 2: Concept Mapping**
1. AI model analyzes each lesson's skill statements
2. Maps to MLC concept taxonomy (guide_notes, intervals, rhythm, scales, etc.)
3. Estimates difficulty level (1-10) based on:
   - Concept complexity
   - Prerequisite requirements
   - Typical student age/level
   - Comparative analysis with existing sequences
4. Identifies visual vs. aural learning modalities
5. Flags MIDI keyboard requirements

**Step 3: Game Retrieval and Ranking**
1. For each lesson item, query game library using:
   - Semantic similarity (embedding-based search)
   - Exact concept tag matching
   - Difficulty range filtering
   - Instrument/device compatibility
2. Rank candidate games by:
   - Concept alignment score (0.0 - 1.0)
   - Historical effectiveness data (average student scores, time-to-mastery)
   - Teacher ratings and adoption metrics
   - Pedagogical fit based on sequence context
3. Return top 3-5 candidates with confidence scores

**Step 4: Sequence Assembly**
1. Group lessons into logical Units (Groups)
2. For each Unit, create Assignments following MLC pedagogy:
   - Introduction assignment (Learn stages)
   - Practice assignments (Play stages)
   - Assessment assignments (Quiz stages)
   - Optional challenge assignments
3. Apply staging rules:
   - Learn before Play before Quiz
   - Review scheduled 7 days after Quiz (configurable)
   - Challenge as optional final step
4. Set targets and gating:
   - Use game metadata defaults
   - Adjust based on book difficulty progression
   - Apply class policy overrides if applicable
5. Insert non-game elements:
   - Instructional videos for complex concepts
   - Text elements for practice tips
   - Reward elements at milestone completions

**Step 5: Validation and Safety Checks**
1. Verify all referenced games exist and are in `live` status
2. Check stage availability (some games lack Challenge)
3. Validate progression logic (no circular dependencies)
4. Ensure device compatibility (MIDI, microphone requirements)
5. Flag warnings:
   - Missing prerequisite concepts
   - Difficulty spikes
   - Unusually long/short assignments
6. Generate validation report with actionable suggestions

**Step 6: Teacher Review Interface**
1. Present sequence proposal in split-pane view:
   - Left: Method book content (extracted text + images)
   - Right: Generated sequence structure
2. Show confidence scores and AI rationale for each mapping
3. Provide inline editing tools:
   - Swap game (show alternatives)
   - Adjust targets and gating
   - Reorder steps
   - Insert/delete steps
   - Add custom instructions
4. Highlight low-confidence mappings for manual review
5. Show validation warnings and suggestions prominently

**Step 7: Iterative Refinement**
1. Teacher makes edits and provides feedback
2. System learns from edits:
   - Track which games are swapped (preference signals)
   - Record rationale text for future prompts
   - Update game ranking models
3. Teacher can request re-generation of specific assignments
4. AI proposes alternatives based on feedback

**Step 8: Approval and Publishing**
1. Teacher reviews final sequence
2. Submits for approval (if org requires pedagogical review)
3. System creates published sequence version
4. Sequence appears in library with proper visibility scope
5. Teachers can assign to students immediately
6. Feedback loop: monitor student performance and teacher satisfaction

### Natural Language Prompt Workflow

For custom teacher prompts like "Create a 4-week sequence for 10-year-olds focusing on dotted quarter notes in 3/4 time and reinforcing G major pentascale":

**Step 1: Prompt Parsing**
1. Extract key entities:
   - Duration: 4 weeks
   - Age: 10 years
   - Focus concepts: dotted_quarter_notes, 3/4_time, G_major_pentascale
   - Proficiency level: "reinforcing" (implies intermediate review)
2. Infer additional context:
   - Estimated practice frequency (3x/week → ~12 sessions)
   - Device profile (likely piano, possibly MIDI)
   - Visual vs. aural mix (both for well-rounded skill building)

**Step 2: Concept Expansion**
1. Identify prerequisite concepts:
   - quarter_notes (should already be mastered)
   - 3/4_time_signature (basic understanding needed)
   - G_pentascale (primary focus)
2. Identify related concepts to include:
   - rhythm_reading_in_3/4
   - staff_reading_in_G_position
   - treble_clef_G_major_notes
3. Plan progression arc:
   - Week 1: Review prerequisites
   - Week 2: Introduce dotted quarter
   - Week 3: Practice and apply in context
   - Week 4: Assessment and synthesis

**Step 3: Game Selection**
1. Query game library for concept matches
2. Prioritize games tagged with target concepts
3. Balance visual and aural modalities
4. Include variety of game mechanics for engagement
5. Ensure prerequisite games are easier than focus games

**Step 4: Sequence Structure Generation**
1. Create 4 Groups (one per week)
2. Populate assignments with standard Learn-Play-Quiz pattern
3. Schedule reviews at appropriate intervals
4. Add optional challenges for advanced students
5. Insert text/video elements for context

**Step 5: Teacher Review and Refinement**
(Same as method book workflow steps 6-8)

### Student-Specific Remediation Workflow

For generating personalized remediation sequences based on performance data:

**Step 1: Performance Analysis**
1. Identify concepts with below-target performance (score < 70%)
2. Analyze error patterns (timing issues, note reading errors, etc.)
3. Determine likely root causes (missing prerequisites, need for spaced repetition)

**Step 2: Remediation Plan**
1. Select 2-3 foundational concepts to address
2. Find games at easier difficulty levels
3. Create short focused sequence (5-10 assignments)
4. Schedule intensive review pattern (3-day spacing)

**Step 3: Auto-Assignment**
1. Generate sequence automatically when triggers are met
2. Optionally notify teacher for approval
3. Insert into student's assignment queue
4. Monitor progress and adjust if needed

## Game Matching Algorithm

### Semantic Search Pipeline

```python
# Pseudocode for game matching

def match_games_to_lesson(lesson_item, config):
    # Extract concept embeddings
    concept_embedding = embed_concepts(lesson_item.concept_tags)
    skill_embedding = embed_text(lesson_item.skill_text)
    
    # Query vector database
    semantic_candidates = vector_db.query(
        query_vector=concept_embedding,
        filters={
            "instrument": config.target_instrument,
            "status": "live",
            "difficulty": range(lesson_item.difficulty_estimate - 1, 
                                lesson_item.difficulty_estimate + 2)
        },
        limit=20
    )
    
    # Re-rank with concept graph
    concept_scores = []
    for game in semantic_candidates:
        score = calculate_concept_overlap(
            lesson_concepts=lesson_item.concept_tags,
            game_concepts=game.concept_tags,
            concept_graph=load_concept_taxonomy()
        )
        concept_scores.append((game, score))
    
    # Filter by device requirements
    if not config.include_midi_games:
        concept_scores = [
            (g, s) for g, s in concept_scores 
            if g.device_profile != "midi_required"
        ]
    
    # Incorporate historical performance data
    final_scores = []
    for game, concept_score in concept_scores:
        historical_score = get_game_effectiveness(
            game_id=game.game_id,
            concept=lesson_item.concept_tags[0],  # primary concept
            student_age=config.target_age_range
        )
        combined_score = (0.6 * concept_score) + (0.4 * historical_score)
        final_scores.append((game, combined_score, concept_score, historical_score))
    
    # Sort and return top candidates
    final_scores.sort(key=lambda x: x[1], reverse=True)
    return [
        {
            "game": game,
            "confidence": combined,
            "concept_match": concept,
            "effectiveness": historical,
            "rationale": generate_rationale(game, lesson_item)
        }
        for game, combined, concept, historical in final_scores[:5]
    ]
```

### Concept Taxonomy and Graph

The system maintains a hierarchical concept graph:

```
Music Concepts
├── Pitch
│   ├── Note Reading
│   │   ├── Guide Notes
│   │   ├── Staff Lines and Spaces
│   │   └── Ledger Lines
│   ├── Intervals
│   │   ├── 2nds
│   │   ├── 3rds
│   │   └── [...]
│   └── Scales
│       ├── Major Pentascales
│       ├── Major Scales
│       └── Minor Scales
├── Rhythm
│   ├── Note Values
│   │   ├── Whole Note
│   │   ├── Half Note
│   │   ├── Quarter Note
│   │   └── [...]
│   ├── Time Signatures
│   │   ├── 2/4
│   │   ├── 3/4
│   │   └── 4/4
│   └── [...]
├── Harmony
│   ├── Intervals (harmonic)
│   ├── Triads
│   └── Chord Progressions
└── [...]
```

Concept relationships include:
- **prerequisite**: Concept A must be learned before Concept B
- **reinforces**: Concept A supports retention of Concept B
- **related**: Concepts often taught together
- **alternative**: Different approaches to same learning goal

This graph enables intelligent game selection based on:
1. Direct concept matches
2. Prerequisite coverage
3. Reinforcement opportunities
4. Balanced concept distribution

## UX Requirements

### Teacher Review Interface

**Layout:**
- Split-pane editor with method book content on left, generated sequence on right
- Sticky header showing overall progress, confidence score, and validation status
- Inline editing with drag-and-drop reordering
- Quick access to game library for swapping elements
- Comment threads for collaborative review

**Key Features:**
1. **Confidence Indicators**: Color-coded badges on each step (green > 0.8, yellow 0.6-0.8, red < 0.6)
2. **AI Rationale**: Hover or click to see why each game was selected
3. **Alternative Games**: Right-click step to see alternative game suggestions
4. **Bulk Operations**: Select multiple steps and apply changes (adjust targets, change difficulty)
5. **Validation Panel**: Prominent warnings and suggestions with one-click fixes
6. **Preview Mode**: Simulate student view to understand assignment flow
7. **Comparison View**: Toggle between AI proposal and manual sequence (if editing existing)

**Workflow Actions:**
- Accept All (for high-confidence generations)
- Review Low-Confidence Items (filter to < 0.7 confidence)
- Publish Draft (save for later editing)
- Submit for Review (send to pedagogical reviewer)
- Publish Sequence (make live and assignable)
- Request Regeneration (specific assignments or entire sequence)

### Accessibility
- Full keyboard navigation for review interface
- Screen reader announcements for confidence scores and validation warnings
- High contrast mode for confidence indicators
- Keyboard shortcuts for common editing operations (swap game, adjust targets, reorder)

## Validation and Safety Rules

### Content Validation
- **Game existence**: All referenced `game_id` values must exist in Games Registry
- **Stage availability**: Verify each stage is enabled for the specified game
- **Asset health**: Check that game bundles and media are accessible
- **Device compatibility**: Warn if MIDI/microphone games exceed expected device availability

### Pedagogical Validation
- **Prerequisite coverage**: Ensure foundational concepts appear before advanced concepts
- **Difficulty progression**: Flag spikes (> 2 point jump) in difficulty scores
- **Stage sequence**: Enforce Learn → Play → Quiz → Challenge → Review order
- **Target reasonableness**: Warn if targets deviate significantly from game metadata defaults

### Structural Validation
- **Assignment length**: Warn if assignments have < 3 or > 15 steps
- **Group balance**: Flag groups with significantly more/fewer assignments than others
- **Review scheduling**: Ensure reviews are scheduled only after Quiz is passed
- **Optional steps**: Verify optional steps don't gate required progression

### Safety Guardrails
- **No auto-publishing**: Require explicit teacher approval for all generated sequences
- **Change tracking**: Log all edits for audit and learning purposes
- **Rollback capability**: Allow teachers to revert to AI proposal if manual edits introduce issues
- **Confidence thresholds**: Sequences with average confidence < 0.6 require pedagogical review before publishing

## Telemetry and Feedback Loop

### Generation Metrics

**Request telemetry:**
- `ai_generation_requested`
  - `request_type`: method_book | natural_language | student_remediation
  - `user_id`, `org_id`
  - `input_characteristics`: book publisher, page count, prompt length, etc.
  - `config_options`: duration, difficulty range, included modalities

**Processing telemetry:**
- `ai_generation_completed`
  - `request_id`, `proposal_id`
  - `model_version`: which AI model was used
  - `processing_time_ms`: total generation duration
  - `total_games_proposed`: count of unique games
  - `average_confidence`: mean confidence score across all steps
  - `validation_warnings`: count and types of warnings generated

**Review and approval telemetry:**
- `ai_proposal_reviewed`
  - `proposal_id`, `user_id`
  - `time_to_first_review_minutes`: how long after generation
  - `review_duration_minutes`: time spent reviewing
  - `approval_decision`: accepted | rejected | needs_revision
  - `overall_rating`: 1-5 stars
  - `feedback_provided`: boolean

**Edit telemetry:**
- `ai_proposal_edited`
  - `proposal_id`, `user_id`
  - `edit_type`: swap_game | adjust_targets | reorder | insert | delete
  - `assignment_id`, `step_id` (if applicable)
  - `old_game_id`, `new_game_id` (for swaps)
  - `edit_rationale`: free text reason provided by teacher

**Outcome telemetry:**
- `ai_generated_sequence_published`
  - `sequence_id`, `proposal_id`
  - `total_edits_applied`: count of teacher modifications
  - `edit_rate`: edits / total_steps
  - `acceptance_rate`: steps unchanged / total_steps
  - `low_confidence_edit_rate`: edits to steps with confidence < 0.7

**Usage telemetry:**
- `ai_generated_sequence_assigned`
  - `sequence_id`, `student_count`
  - `assigned_by_user_id`, `org_id`
  - Time tracking: days_since_generation

**Performance telemetry:**
- `ai_generated_sequence_student_outcome`
  - `sequence_id`, `student_id`
  - `completion_rate`: assignments completed / total assignments
  - `average_score`: mean quiz scores
  - `time_to_mastery`: days from first assignment to sequence completion
  - `compared_to_baseline`: performance vs. manually authored sequences

### Feedback Loop and Model Improvement

**Continuous learning mechanisms:**

1. **Game Preference Learning**
   - Track which games are frequently swapped out (negative signals)
   - Track which games are kept or added manually (positive signals)
   - Update game ranking weights based on context (concept, age, method book)

2. **Confidence Calibration**
   - Compare predicted confidence scores to actual acceptance rates
   - Recalibrate confidence thresholds quarterly
   - Flag overconfident or underconfident predictions for model retraining

3. **Concept Mapping Refinement**
   - When teachers provide rationale for game swaps, extract concept insights
   - Update concept taxonomy based on emerging patterns
   - Identify missing concepts or granularity issues

4. **Performance-Based Optimization**
   - Correlate student outcomes with specific game selections
   - Identify games that consistently lead to higher mastery rates
   - Adjust ranking algorithms to favor high-performing games

5. **Prompt Engineering**
   - A/B test different prompt structures for generation quality
   - Track which prompt variations lead to higher acceptance rates
   - Maintain prompt library with versioning and performance metrics

6. **Human-in-the-Loop Refinement**
   - Monthly review sessions with pedagogical experts
   - Analyze low-rated proposals to identify failure patterns
   - Create adversarial test cases for edge scenarios

## Integration with Existing Systems

### Dependencies

**Required integrations:**
- **[Games Registry](../08-games-registry-and-authoring/games-registry-overview.md)**: Query game metadata, concept tags, difficulty ratings
- **[Sequence Model](sequence-model.md)**: Generate compliant sequence JSON
- **[Sequence Authoring](sequence-authoring.md)**: Integrate into authoring UI for teacher review and editing
- **[Sequence Libraries](sequence-libraries.md)**: Publish generated sequences with proper visibility scopes
- **[Assignment Generation](../10-assignments-engine/assignment-generation.md)**: Generated sequences work seamlessly with assignment engine
- **[Search Indexing](../11-search-and-discovery/search-indexing.md)**: Index generated sequences for discovery
- **[Event Model](../15-analytics-and-reporting/event-model.md)**: Emit telemetry in canonical format

**Data flow:**
1. Teacher initiates generation via Sequence Authoring UI
2. AI Generation Service queries Games Registry for candidate games
3. Generation Service creates proposal and stores in Sequence Store (draft state)
4. Teacher reviews in Authoring UI, applies edits, provides feedback
5. On approval, sequence published to Libraries with method_aligned or custom visibility
6. Teachers discover and assign sequence via normal assignment workflows
7. Assignment Generation Engine treats AI-generated sequences identically to manual sequences
8. Student outcomes feed back to AI service for continuous improvement

### API Endpoints (Future Implementation)

```
POST /api/v1/ai-sequence/generate
  Body: GenerationRequest (see data model)
  Response: { proposal_id, status, estimated_completion_seconds }

GET /api/v1/ai-sequence/proposals/{proposal_id}
  Response: GeneratedSequenceProposal (see data model)

POST /api/v1/ai-sequence/proposals/{proposal_id}/feedback
  Body: TeacherFeedback (see data model)
  Response: { feedback_id, proposal_updated }

POST /api/v1/ai-sequence/proposals/{proposal_id}/regenerate
  Body: { assignment_ids: [], config_overrides: {} }
  Response: { proposal_id, regenerated_count }

POST /api/v1/ai-sequence/proposals/{proposal_id}/publish
  Body: { sequence_name, visibility, org_id? }
  Response: { sequence_id, sequence_version }

GET /api/v1/ai-sequence/analytics/acceptance-rates
  Query: ?start_date=...&end_date=...&user_id=...
  Response: { acceptance_rate, avg_confidence, edit_rate, [...] }
```

## Permissions

**Generate Sequences:**
- **Teachers**: Can generate custom sequences scoped to their `org_id`
- **Content Editors**: Can generate first-party and method-aligned sequences
- **System Admins**: Full access to all generation capabilities

**Review and Approve:**
- **Teachers**: Can approve their own generated sequences for their org
- **Content Editors**: Can approve method-aligned sequences for platform-wide use
- **Pedagogical Reviewers**: Can approve or request revisions on flagged proposals

**Access Analytics:**
- **Teachers**: Can view analytics for their own generated sequences
- **Admins**: Can view org-wide analytics
- **System Admins**: Full access to platform-wide analytics and model performance metrics

**Provide Feedback:**
- **All users with generation permissions**: Can rate and comment on proposals

## Open Questions and Future Enhancements

### Current Open Questions
1. **Model hosting**: Self-hosted open-source models vs. API-based commercial models (cost vs. control tradeoff)
2. **Confidence thresholds**: What minimum confidence score should trigger mandatory pedagogical review?
3. **Regeneration limits**: Should there be limits on how many times a teacher can request re-generation to prevent API cost abuse?
4. **Publisher licensing**: How to handle copyright when ingesting method book PDFs? (Legal/business decision)
5. **Multi-language support**: Priority order for supporting method books in languages other than English?
6. **Student-specific generation**: How much student performance data should influence sequence generation vs. maintaining consistent curriculum standards?

### Future Enhancement Ideas
1. **Real-time collaboration**: Multiple teachers co-reviewing and editing same generated sequence
2. **Version comparison**: Diff view showing changes between generated proposal versions
3. **Template library**: Save generation configs as templates for reuse across similar method books
4. **Batch generation**: Generate sequences for entire method series (Books 1-5) in one request
5. **Visual method book overlay**: Render generated sequence steps directly on method book page images
6. **Student preference learning**: Incorporate individual student game preferences into remediation generation
7. **Adaptive sequences**: Sequences that automatically adjust based on real-time student performance during progression
8. **Multi-modal input**: Accept audio recordings of teachers explaining their curriculum goals
9. **Collaborative filtering**: "Teachers who used this method book also liked these games..."
10. **Explainable AI dashboard**: Deep dive into model decision-making for pedagogical researchers

## Phasing and Rollout Strategy

### Phase 0: Foundation (Q4 2025 - Q1 2026)
- Monitor AI music comprehension capabilities (model releases, research papers)
- Build and test method book content extraction pipeline on small samples
- Develop concept taxonomy and game embedding system
- Create evaluation framework and test datasets

### Phase 1: Alpha (Q2 2026) - **Assumes AI capabilities mature**
- Limited release to 5-10 expert Content Editors for first-party sequence generation
- Focus on natural language prompts (simpler than method book ingestion)
- Manual validation of all generated sequences before any student use
- Gather qualitative feedback on UX and AI quality

### Phase 2: Beta (Q3 2026)
- Expand to 50 pilot teachers for custom sequence generation
- Introduce method book ingestion for 3-5 popular series (Faber, Hal Leonard, Alfred)
- Implement teacher review interface with confidence scores and editing
- Begin collecting quantitative metrics (acceptance rates, edit rates, time savings)

### Phase 3: General Availability (Q4 2026)
- Launch to all teachers with appropriate permissions
- Support 20+ method book series
- Enable student-specific remediation generation
- Continuous model improvement based on feedback loop

### Phase 4: Advanced Features (2027+)
- Real-time collaboration on sequence review
- Adaptive sequences with dynamic adjustment
- Multi-language method book support
- Integration with publisher content partnerships

## Success Metrics

### Primary KPIs
- **Acceptance Rate**: % of generated sequences published without major revisions (target: > 70%)
- **Time Savings**: Average time to create sequence: AI-assisted vs. manual (target: 80% reduction)
- **Student Outcomes**: Comparison of mastery rates: AI-generated vs. manually authored sequences (target: parity or better)
- **Teacher Satisfaction**: Net Promoter Score for AI generation feature (target: > 50)

### Secondary KPIs
- **Generation Volume**: Number of sequences generated per month
- **Edit Rate**: Average edits per sequence (lower is better, target: < 15%)
- **Confidence Accuracy**: Correlation between AI confidence scores and teacher acceptance
- **Coverage**: % of popular method books with AI-generated sequence options (target: > 90% by end of Phase 3)

### Quality Metrics
- **Pedagogical Validity**: Expert review ratings of generated sequences (target: > 4.0/5.0)
- **Content Health**: % of generated sequences passing all validation rules (target: > 95%)
- **Student Engagement**: Completion rates for AI-generated vs. manual sequences (target: parity)

## Conclusion

The AI Sequence Generator represents a strategic investment in future capabilities that will dramatically scale MLC's ability to support diverse curricula and personalized learning paths. While current AI limitations prevent immediate implementation, this specification provides a comprehensive roadmap for rapid deployment once the underlying technology matures. By maintaining teacher oversight, implementing rigorous validation, and continuously learning from feedback, the system will augment human expertise rather than replace it, embodying the best of AI-assisted educational technology.

