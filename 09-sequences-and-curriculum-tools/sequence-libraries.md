# Sequence Libraries

## Purpose
Catalog of first-party and method-aligned Sequences available for assignment and AI generation.

## Library buckets
- First-party core
  - LIFE - Lifetime Musician
  - SOLF - Solfege
  - EVAL - Evaluation
- Method-aligned
  - Publisher and method sequences mapped to skills, units, pages, and explicit Game stages
- State or district curricula
- Custom teacher sequences

## Hal Leonard Student Piano Library - exemplars
These PDFs show the exact correlation pattern we will mirror in the AI importer and authoring UI: Unit, page range, Category, Game, Skills/Concepts, and stage targets.

- Book A - Pages 4 to 79. Early skills include high vs low, steady beat, finger numbers, groups of 2 and 3 black keys, and quarter vs half notes. Lists per-row Learn, Play, Quiz target values.
- Book B - Pages 4 to 79. Introduces first staff work, guide notes, middle C pentascale, and simple aural patterns with per-stage targets.
- Book 1 - Intro plus Units 1 to 5. Adds rhythm identification, staff birds playback, guide notes, and beginner pentascales with explicit targets for each stage.
- Book 2 - Units 1 to 5. Expands to bass and treble pentascales, rhythm writers, interval work, and pattern reading with MIDI variants.
- Book 3 - Units 1 to 5. Broader interval sets, key signatures, more complex rhythm tasks, and ledger line reading.
- Book 4 - Units 1 to 5. Adds triplets, 3-8 and 6-8 meters, more ledger lines, and triads with inversions, plus timed note-reading games.
- Book 5 - Units 1 to 6. Key signatures through multiple sharps and flats, chord quality identification, relative minors, dictation, and advanced rhythm writers.

## Metadata per sequence
- sequence_id, name, version, visibility, instrument
- source_reference: book, edition, year, page mapping
- concept coverage and estimated hours
- who_can_assign: Teacher, Teacher-Admin, Subscriber-Admin

## Import and export
- JSON and CSV import-export for portability and backup. See AI sequence generator for schema.

## QA gates
- Coverage checks ensure a baseline set of skills per level before publish
- Health checks detect missing or deprecated Game references
