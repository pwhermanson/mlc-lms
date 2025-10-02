# AI Sequence Generator

## Goal
Use Gemini to auto-create Sequences from method book metadata or free-text teacher prompts, then keep the teacher in the loop to approve and edit.

## Inputs
- Method book metadata - title, edition, level, unit, page, lesson, and skill statements
- Optional teacher prompt - free text like "introduce dotted quarter in 3-4 and reinforce G pentascale"
- Local game graph - each Game stage tagged with concepts, difficulty, instrument, visual_or_aural, keyboard vs non-keyboard

## Process outline
1) Normalize book data into units, pages, and skill tags.
2) Concept extraction - model identifies target concepts from the book lines.
3) Game mapping - rank Games by concept match, difficulty fit, and prior student data.
4) Sequence proposal - construct Groups and Assignments with Learn, Play, Quiz, optional Challenge, and scheduled Review steps.
5) Teacher review - show rationale and allow swap, reorder, or retarget thresholds.
6) Save - persist as sequence_version with source_reference to the method.

## Output
- sequence JSON with steps, stage targets, and review offsets
- audit trail with confidence scores and mappings used

## JSON schema - method book ingestion (draft)
```json
{
  "book": {
    "title": "Hal Leonard Student Piano Library",
    "level": "Book 2",
    "edition_year": 2015
  },
  "units": [
    {
      "name": "Unit 1",
      "pages": "p.4-11",
      "items": [
        {
          "skill_text": "Identify Treble G, Middle C, Bass F on the staff",
          "concept_tags": ["guide_notes", "treble_staff", "bass_staff", "reading"],
          "suggested_games": ["grand-staff-guide-notes-1"],
          "stages": ["learn", "play", "quiz"]
        }
      ]
    }
  ]
}
```

## Safety rails
- Only propose Games available for the student instrument and device profile
- Enforce stage presence rules and minimum thresholds
- Keep teacher approval mandatory for publication

## Telemetry
- Store acceptance rate, edits applied, and learning outcomes to improve ranking
