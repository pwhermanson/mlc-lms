# Product Vision

## Purpose

This document sets the north star for the MusicLearningCommunity (MLC) LMS rebuild. It explains what we are building, who it serves, why it matters, and how we will measure success. It is the single source of truth that keeps design, engineering, content, and operations aligned.

## Historical Foundation

MLC was founded in 2006 by Christine Hermanson, a nationally-recognized pioneer in music education technology who was described by The New York Times as "a giant of multimedia music who began her work on education computer programs 18 years ago." The platform has served over 60,000 students in more than 40 countries, with teachers reporting that students advance 3-5 months ahead of their peers after one year of using the system.

The original vision was to create "a playground for Lifetime Musicians in Training" - a mastery-first approach to music literacy that combines the best elements of technology with proven pedagogical methods. This rebuild preserves that core mission while modernizing the platform for scale, accessibility, and enhanced learning outcomes.

---

## One-sentence vision

A mastery-first, AI-enabled music learning platform that maps any teaching method to the exact practice a student needs, scales from single studios to large schools, and proves learning through quantitative outcomes.

---

## What MLC is

MLC teaches music literacy with short, concept-focused learning elements built around staged games: Learn (tutor), Play (practice), Quiz (quantitative mastery assessment), optional Challenge (worldwide competition), and a distinct Review stage (retention testing) that records its own score. Not every game includes Challenge. Scores persist to student history and drive progression.

Each game teaches a very specific musical concept through simple, non-threatening, and fun interactions. Games cover aural and visual Pitch, Rhythm, Melody, Chords, Scales, Memory Playing, Intervals, Staff, Symbols, Keyboard Elements, Pattern Playing, and Musical Terms.

Students advance through named Sequences such as Lifetime Musician (default), Solfege (fixed do), and Evaluation; teachers can also use method-aligned sequences that mirror popular books and curricula like Alfred, Faber, and state/national standards.

Parents and students can access MLC anytime to reinforce lesson objectives from home or school, with full MIDI keyboard support for kinesthetic learning.

---

## Why MLC exists

* **Accelerate literacy** with measurable mastery rather than time-spent. Teachers report students advance 3-5 months ahead of peers after one year of use.
* **Reduce teacher workload** via reusable sequences, clear reporting, and bulk tools for schools. The platform pays for itself when teachers charge students $1-2/month.
* **Scale reliably** across roles and institutions with structured hierarchies and admin visibility.
* **Support special needs learners** - the simple, engaging game presentation has proven particularly effective for children with autism and other special needs.
* **Enable anywhere learning** - cloud-based access means no software installation, continuous updates, and 24/7 availability from any internet-enabled device.

---

## Who we serve

* **Students** (ages 3-87) seeking clear, bite-sized practice that builds confidence and tracks progress, including those with special needs who benefit from the simple, non-threatening game design.
* **Teachers** (private studio, public school, homeschool) who want assignments aligned to their preferred method and instant insight into mastery.
* **Teacher-Admins and Subscriber-Admins** who manage teachers, classes, seats, and billing in studios and schools.
* **System Admin** who supports at scale with safe visibility and auditability.
* **Parents** who want to see measurable progress in their children's music education and support home practice.
* **Schools and Districts** using the platform for General Music, Band, Choir, and after-school programs with bulk management tools.

---

## Core value propositions

1. **Mastery you can see**: Target scores by stage with retained history and scheduled Review for retention. Quantitative assessment provides objective measures of concept mastery.
2. **Method-aware automation**: Sequences align to method books and standards; assignments generate in one click. Supports Alfred, Faber, RCM, MTNA state evaluations, and state/national standards.
3. **School-ready operations**: Bulk CSV tools, mass assignment, school POs, statements, and role hierarchies. Scales from solo teachers to large districts.
4. **Accessible, anywhere practice**: Designed for home and classroom reinforcement with full MIDI keyboard support and responsive design.
5. **Proven effectiveness**: 17+ years of success with over 60,000 students worldwide, with teachers reporting 3-5 month advancement over peers.
6. **Special needs inclusive**: Simple, engaging design particularly effective for children with autism and other special needs.

---

## Product goals

* Increase concept mastery rates and shorten time-to-mastery across cohorts, maintaining the proven 3-5 month advancement over peers.
* Cut teacher prep time for assignments and reduce administrative overhead for schools through automation and bulk tools.
* Support thousands of students per customer with fast, filterable admin surfaces and scalable architecture.
* Grow adoption from trial to paid across Solo and Ensemble tiers with clear seat scaling and flexible pricing.
* Maintain 50% trial-to-paid conversion rate while expanding global reach beyond the current 40+ countries.
* Preserve the simple, engaging game design that has proven effective for special needs learners while modernizing the technical foundation. 

---

## Guiding principles

* **Mastery first**: Every feature should make it easier to teach, practice, assess, and retain.
* **Sequence before search**: Default to guided Sequences; keep All Games for exploration and targeted assignments.
* **Scale by design**: Roles, bulk ops, billing, and statements are core.
* **Clarity and safety**: Admin visibility without password sharing, with full audit logs.
* **Data-driven iteration**: Use events, scores, and usage to evolve content and UX.
* **Teacher-in-the-loop AI**: AI proposes; teachers approve, edit, and assign.

---

## Scope at a glance

* **Student**: Dashboard, Assignment Play, All Games, badges, leaderboards, messaging.
* **Teacher**: Assignment Builder, progress reports, announcements, class settings.
* **Admin**: Member management, CSV imports, mass assignment, billing, PO and statements, promo and sponsor codes.
* **SysAdmin**: Global user index, member structure view, billing tools, audit logs, feature flags.

---

## AI enablement vision

* Preload publisher method metadata by level, unit, page, and stated objectives.
* Use concept tags on each game to map book objectives to the best Learn/Play/Quiz/Review steps.
* Generate a proposed sequence that teachers can accept or edit, then save as a versioned library item for reuse.
* Support natural-language prompts such as "introduce dotted quarter in 3-4 and reinforce G pentascale" to fetch and assign the best games.

---

## Success metrics

* **Learning**: Quiz mastery rate, Review retention rate, time-to-mastery by concept, student advancement rate (target: maintain 3-5 month advantage over peers).
* **Teacher efficiency**: Time to create assignments; number of bulk operations completed; teacher satisfaction scores.
* **Scale and reliability**: Median load times and filter responsiveness for large user lists; support ticket volume; system uptime (target: 99.9%).
* **Commercial**: Trial to paid conversion (target: maintain 50%+); seat growth by tier and volume bands; customer retention rate.
* **Accessibility**: Special needs learner engagement and progress rates; MIDI keyboard usage adoption.
* **Global reach**: Student growth in new markets; international teacher adoption rates. 

---

## Out of scope for the current phase

* Major graphics re-skins of the games; the HTML5 redesign is functional and enhancements are future work.
* Deep third-party SIS or LTI integrations beyond import/export and email delivery.
* Mobile native apps; focus is responsive web with PWA considerations.
* Advanced video/audio content creation tools; focus on game-based learning elements.
* Real-time collaborative features; maintain individual mastery-focused approach.
* Advanced analytics beyond core learning metrics and teacher reporting.

---

## Risks and mitigations

* **Content mapping gaps**: Build a robust games registry with concept tags and a teacher feedback loop.
* **School scale complexity**: Ship proven CSV tools, mass assignment, and a performant system admin index.
* **Billing edge cases**: Document seat rules, pause and reactivation, POs, and statements; instrument webhooks for accuracy.  

---

## Supporting Documents Referenced

This product vision draws from the following source documents:

- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Core mission and value propositions
- [MLC History Draft 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20History%20Draft%202020.docx.txt) — Founder background, historical success data, and educational effectiveness
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Platform capabilities and parent/teacher messaging
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Learn/Play/Quiz/Challenge/Review model
- [MLC SeqCreate 2020.xlsx - LIFE SEQ.csv](../00-ORIG-CONTEXT/MLC%20SeqCreate%202020.xlsx%20-%20LIFE%20SEQ.csv) — Sequence structure and curriculum pathways
- [MLC Tiered Pricing 2020.xlsx - Full Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Full%20Calc.csv) — Pricing tiers and seat scaling model
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role hierarchies and capabilities
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Seat management and scaling rules
- [pianoAdventures.doc.txt](../00-ORIG-CONTEXT/pianoAdventures.doc.txt) — Method book alignment approach
- [AlphaMasterList2020.xlsx - Master.csv](../00-ORIG-CONTEXT/AlphaMasterList2020.xlsx%20-%20Master.csv) — Game library scope (450+ games)

---

## Source anchors

* **Game structure, stages, scoring, and sequences** - Learn/Play/Quiz/Challenge/Review model with quantitative assessment.
* **Parent access and reinforcement message** - Home practice support and progress visibility.
* **Spec library contents and breadth of the system** - 450+ games covering all music literacy concepts.
* **Phase 2 school features, CSV tools, statements, POs** - Bulk operations for institutional scale.
* **Roles and hierarchies for Solo and Ensemble** - Flexible permission model for different organizational needs.
* **System admin scaling and visibility needs** - Support for thousands of students with audit trails.
* **Tiered pricing and seat scaling** - Solo (5-19 seats), Ensemble (20+ seats) with volume discounts.
* **Historical success data** - 60,000+ students served, 40+ countries, 3-5 month advancement rates.
* **Special needs effectiveness** - Proven benefits for children with autism and other special needs.
* **Method alignment** - Support for Alfred, Faber, RCM, MTNA evaluations, and state/national standards. 

---
