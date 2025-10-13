# Competitor Analysis

## Overview
Comprehensive analysis of competitive landscape and market positioning.

## Description
Detailed analysis of competing platforms, market trends, and positioning strategies to inform product development and marketing decisions.

---

## Purpose

This document transforms raw competitive intelligence from the [Competitor Scan 2025](./competitor-scan-2025.md) into strategic insights and actionable recommendations. While the scan documents *what* competitors do, this analysis addresses *how MLC should respond* with positioning, differentiation, and market strategy.

### Why This Document Exists

- **Strategic positioning**: Define where MLC competes and where it deliberately diverges
- **Differentiation clarity**: Articulate sustainable competitive advantages that competitors cannot easily replicate
- **Threat assessment**: Identify competitive risks and develop defensive strategies
- **Opportunity identification**: Discover underserved market segments and feature gaps to exploit
- **Investment prioritization**: Guide product roadmap based on competitive pressure vs. differentiation opportunity

### Relationship to Other Documents

- **Inputs**: [Competitor Scan 2025](./competitor-scan-2025.md) provides raw intelligence; [User Feedback](./user-feedback.md) provides validation of what users value
- **Outputs**: Informs [Product Vision](../00-foundations/product-vision.md), [Roadmap Phasing](../24-roadmap-and-phasing/phase-1-scope.md), and [Pricing Strategy](../14-payments-and-subscriptions/plans-and-pricing.md)
- **Related context**: [Pedagogy Principles](../00-foundations/pedagogy-principles.md) and [Non-Goals](../00-foundations/non-goals.md) establish strategic boundaries

---

## Competitive Positioning Framework

### Market Segmentation

MLC operates in the **music literacy education software** market, which intersects multiple segments:

#### Primary Market: Method-Aligned Music Literacy
- **Target users**: Private studio teachers, music schools, homeschool families
- **Key need**: Supplement method book instruction with structured practice
- **MLC positioning**: Method-aware automation with quantitative mastery tracking
- **Primary competitors**: Sight Reading Factory, Piano Marvel, SmartMusic (institutional), MusicTheory.net (free but limited)

#### Secondary Market: School Music Education Technology
- **Target users**: K-12 general music teachers, band/choir directors, school districts
- **Key need**: Scalable curriculum delivery with administrative oversight and bulk operations
- **MLC positioning**: School-ready operations with hierarchy management and institutional billing
- **Primary competitors**: MusicFirst, Noteflight Learn, Soundtrap for Education

#### Adjacent Market: Consumer Music Learning Apps
- **Target users**: Self-directed learners seeking instrument or theory skills
- **Key need**: Engaging, self-paced learning with immediate feedback
- **MLC positioning**: Serious literacy foundation vs. entertainment-focused apps
- **Primary competitors**: Yousician, Simply Piano, flowkey, Playground Sessions

### MLC's Competitive Sweet Spot

MLC uniquely serves the intersection of:
1. **Pedagogical rigor** (not edutainment)
2. **Method alignment** (works with teacher's existing curriculum)
3. **Quantitative assessment** (measurable mastery, not just engagement time)
4. **Institutional scale** (bulk tools, hierarchies, POs)
5. **Retention focus** (distinct Review stages, not just initial learning)

---

## SWOT Analysis

### Strengths

**Proven Educational Effectiveness**
- 17+ years of documented success (60,000+ students, 40+ countries)
- Teachers report 3-5 month advancement over peers after one year
- Particularly effective for special needs learners (documented with autism spectrum success)
- See [User Feedback](./user-feedback.md) for testimonial evidence

**Unique Pedagogical Approach**
- Learn/Play/Quiz/Challenge/Review staged model with distinct retention assessment
- Quantitative target scores provide objective mastery measures (not subjective "completion")
- Concept-focused games (very specific learning objectives per game)
- See [Pedagogy Principles](../00-foundations/pedagogy-principles.md) for educational foundation

**Method Alignment Capability**
- Sequences mapped to popular method books (Alfred, Faber, RCM, MTNA)
- Teachers can use MLC alongside their preferred teaching approach
- Reduces friction in adoption (not forcing curriculum change)
- See [Sequence Alignment](../09-sequences-and-curriculum-tools/sequence-alignment-to-methods.md)

**Comprehensive Game Library**
- 450+ games covering all music literacy domains (pitch, rhythm, melody, chords, scales, intervals, symbols, keyboard, terms)
- Multiple difficulty levels per concept (scaffolded progression)
- MIDI keyboard support for kinesthetic learning
- See [Game Catalog](../07-content-architecture/game-catalog.md)

**School-Ready Operations**
- Role hierarchy model (Student, Teacher, Teacher-Admin, Subscriber-Admin, System Admin)
- CSV bulk import/export for students, teachers, assignments
- Purchase order and invoice workflows (not just credit card)
- Mass assignment tools for large groups
- See [Admin Experience](../05-admin-subscriber-experience/subscriber-dashboard.md) and [Bulk Operations](../05-admin-subscriber-experience/student-bulk-operations.md)

**Flexible Business Model**
- Solo tier (5-19 seats) for private teachers
- Ensemble tier (20+ seats) with volume pricing for schools
- 30-day Prelude trial (19 seats) with high conversion rate (50%)
- See [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md)

### Weaknesses

**Platform Age and Technical Debt**
- Legacy codebase requires modernization (MLC 5.0 rebuild addresses this)
- HTML5 game conversion from Flash still in progress
- UX patterns from 2012 feel dated compared to modern apps
- See [Legacy Systems](../00-foundations/legacy-systems.md)

**Limited Marketing Reach**
- Historically grown through teacher-to-teacher referrals (minimal advertising)
- Less brand recognition than venture-backed competitors (Yousician, Simply Piano)
- Small marketing budget compared to well-funded competitors

**Mobile Experience Gap**
- Responsive web but no native mobile apps
- Competitors offer polished iOS/Android apps with app store presence
- MIDI support on mobile is limited compared to desktop
- See [Mobile Considerations](../21-mobile-and-offline/responsive-behaviors.md)

**Content Production Constraints**
- Game creation requires significant development resources (not user-generated)
- Slower to add new content compared to competitors with content marketplaces
- Video lessons and multimedia content limited compared to platforms like MusicFirst
- See [Non-Goals](../00-foundations/non-goals.md) for deliberate scope decisions

**Integration Limitations**
- No deep LMS/SIS integration (Clever, Classlink, Canvas, Schoology)
- Manual CSV import/export vs. automated rostering
- No xAPI or SCORM compliance for learning record systems
- See [Integrations](../17-integrations/lms-ltispec-future.md) for future roadmap

### Opportunities

**Underserved Method Alignment Market**
- Most competitors focus on proprietary curricula (forcing teachers to change methods)
- Private teachers strongly value curriculum flexibility (per user feedback)
- Opportunity: Position MLC as the "method-agnostic" platform that works with any teaching approach
- AI-assisted sequence generation could accelerate custom method alignments

**Special Needs Education Niche**
- Documented success with autism spectrum and neurodiverse learners
- Competitors don't explicitly position for this market
- Opportunity: Become the go-to platform for inclusive music education
- See [Neurodiverse Support](../16-accessibility-and-inclusion/neurodiverse-support.md)

**Quantitative Assessment Demand**
- Schools increasingly require measurable learning outcomes (data-driven instruction)
- Competitors focus on engagement metrics (time spent) rather than mastery assessment
- Opportunity: Lead with quantitative mastery data and retention testing
- See [Teacher Reports](../15-analytics-and-reporting/teacher-reports-kpis.md)

**Institutional Billing and Operations Gap**
- Many competitors lack purchase order workflows, invoicing, and bulk management tools
- Schools require these capabilities for adoption at scale
- Opportunity: Target district-level sales with enterprise-ready operations
- See [PO and Invoicing](../14-payments-and-subscriptions/po-and-invoicing-for-schools.md)

**International Expansion**
- Already serving 40+ countries organically
- Competitors focus primarily on US market
- Opportunity: Deliberate international marketing in English-speaking countries (UK, Australia, Canada, etc.)
- See [Localization](../01-ux-design-system/localization-internationalization.md)

**AI-Enhanced Curriculum Tools**
- Teachers want time-saving tools (per feedback: "reduce prep time")
- Competitors have not yet implemented AI-assisted sequence generation
- Opportunity: First-mover advantage in teacher-in-the-loop AI for music education
- See [Product Vision - AI Enablement](../00-foundations/product-vision.md#ai-enablement-vision)

### Threats

**Well-Funded Consumer App Competition**
- Yousician, Simply Piano have raised significant venture capital
- Heavy marketing spend and app store presence
- Risk: Parents and students choose consumer apps over teacher-directed platforms
- Mitigation: Emphasize pedagogical rigor and method alignment that consumer apps lack

**Free Competitors**
- MusicTheory.net, Teoria.com offer free theory and ear training
- Risk: Teachers use free tools for supplemental practice rather than paying for MLC
- Mitigation: Highlight retention testing, method alignment, teacher tools, and progress tracking that free sites don't provide

**LMS Platform Bundling**
- MusicFirst aggregates multiple tools in single subscription
- Schools prefer one-vendor solutions to reduce procurement complexity
- Risk: Bundled platforms win school contracts despite individual tool quality
- Mitigation: Focus on Solo tier (private teachers) where bundling is less relevant; pursue partnerships or integrations

**Emerging AI Tools**
- New platforms leveraging AI for personalized learning and content generation
- Risk: MLC appears outdated if competitors ship AI features first
- Mitigation: MLC 5.0 roadmap includes AI-assisted sequence generation and natural language assignment building

**Mobile-First User Expectations**
- Students expect native mobile apps (not responsive web)
- Risk: Platform feels outdated without iOS/Android apps
- Mitigation: PWA features provide app-like experience; focus on desktop strength (MIDI keyboards work best on desktop)

**Privacy Regulation Complexity**
- COPPA, FERPA, GDPR compliance requirements increasing
- Risk: Non-compliance blocks school sales; competitors with compliance win
- Mitigation: Phase 2 compliance work prioritized for institutional sales
- See [Privacy Policy](../23-legal-and-compliance/privacy-policy.md) and [Compliance Notes](../23-legal-and-compliance/coppa-ferpa-notes.md)

---

## Differentiation Strategy

### Sustainable Competitive Advantages

MLC's differentiation must focus on attributes that are:
1. **Valuable** to target users
2. **Rare** among competitors
3. **Difficult to imitate**
4. **Organized to capture** value

#### Primary Differentiator: Method-Aware Mastery Tracking

**Value**: Teachers want curriculum flexibility and quantitative assessment
**Rarity**: Most competitors force proprietary curricula; few offer method alignment
**Inimitability**: Requires years of method mapping and pedagogical expertise (not just software)
**Organization**: Sequences library, assignment builder, progress reports all structured around this

**Competitive messaging**:
> "MLC works with your teaching method, not against it. Choose Alfred, Faber, RCM, or any method—we provide the practice games aligned to your curriculum with quantitative mastery tracking."

#### Secondary Differentiator: Retention-Focused Assessment

**Value**: Long-term retention matters more than initial learning; Review stages test this
**Rarity**: Competitors test initial learning only; MLC has distinct Review stages with separate scoring
**Inimitability**: Requires pedagogical commitment to spaced repetition and retention testing
**Organization**: Learn/Play/Quiz/Review model embedded in game structure and sequence design

**Competitive messaging**:
> "Other platforms test if students learned it. MLC tests if they *retained* it. Distinct Review stages with scheduled retention testing ensure long-term mastery."

#### Tertiary Differentiator: School-Ready Operations

**Value**: Schools require bulk tools, hierarchies, POs, invoicing, and audit trails
**Rarity**: Consumer apps lack these; some school platforms have them but not all
**Inimitability**: Requires operational complexity and billing infrastructure investment
**Organization**: Role model, CSV tools, billing statements, mass assignments all ship in Phase 2

**Competitive messaging**:
> "From solo teachers to large districts: MLC scales. Bulk imports, mass assignments, purchase orders, and role-based permissions designed for institutional needs."

### Feature Parity vs. Differentiation Decisions

Based on competitive scan findings, MLC should make deliberate choices about feature parity:

#### Where to Match Competitors (Table Stakes)

- **Free trial period**: Competitors offer 7-14 days; MLC offers 30-day Prelude (competitive advantage)
- **Responsive design**: All competitors are mobile-friendly; MLC must be also
- **Teacher dashboards**: Standard feature; MLC must match with progress visibility and assignment tools
- **Basic gamification**: Badges, points, leaderboards are expected; MLC includes these (see [Gamification](../12-gamification/badges-system.md))
- **Email notifications**: Standard communication; MLC includes (see [Notifications](../13-messaging-and-notifications/in-app-notifications.md))

#### Where to Diverge (Differentiation)

- **Native mobile apps**: Competitors have iOS/Android apps; MLC focuses on responsive web + PWA (see [Non-Goals](../00-foundations/non-goals.md))
- **Video lessons**: Competitors include video content; MLC focuses on game-based learning (deliberate choice)
- **Social features**: Competitors have chat, friending; MLC limits to class leaderboards (safety and focus)
- **Content marketplace**: Competitors allow user-generated content; MLC curates all games (quality control)
- **Free tier**: Competitors offer freemium models; MLC uses trial-to-paid conversion (sustainable revenue)

#### Where to Lead (Competitive Advantage)

- **Method alignment**: Most competitors lack this; MLC leads with sequence library
- **Retention testing**: Most competitors don't separate Review from Quiz; MLC does
- **MIDI keyboard support**: Most competitors are touch/mouse only; MLC supports hardware (kinesthetic learning)
- **Bulk operations**: Consumer apps lack this; MLC targets institutional scale
- **Quantitative mastery**: Competitors track engagement time; MLC tracks target score achievement

---

## Market Opportunity Analysis

### Total Addressable Market (TAM)

**Music Education Market Segments**:
- **Private piano teachers**: ~65,000 in US (MTNA membership ~20,000; total market larger)
- **K-12 music programs**: ~130,000 music educators in US public schools (NAfME data)
- **Homeschool families**: ~2.5 million homeschool students in US; estimated 10-15% pursue music education (~250,000-375,000)
- **International markets**: MLC already serves 40+ countries; English-speaking markets add significant TAM

**Software Adoption Rates**:
- Not all teachers use digital tools; adoption varies by age, institution, and resources
- COVID-19 accelerated digital adoption significantly (industry shift)
- Younger teachers (< 40 years old) have higher software adoption rates

**Estimated TAM**: Difficult to quantify precisely, but addressable market in the hundreds of thousands of potential teacher subscribers (each with 5-50+ students).

### Serviceable Addressable Market (SAM)

MLC's realistic target market is teachers who:
1. **Value method alignment** (not willing to change curriculum)
2. **Want quantitative assessment** (not just engagement metrics)
3. **Teach music literacy** (not just performance/instrument technique)
4. **Use technology** (comfortable with digital tools)
5. **Can afford subscription** ($7.95-$19.95/month is within budget)

**Estimated SAM**: Likely tens of thousands of teachers (10-20% of TAM based on these filters).

### Serviceable Obtainable Market (SOM)

MLC's realistic market share depends on:
- Marketing reach and brand awareness
- Product quality and user satisfaction
- Competitive pricing and value proposition
- Distribution channels and partnerships

**Historical evidence**: MLC reached 60,000+ students organically with minimal marketing over 15 years. With deliberate marketing and modernized platform, SOM could expand significantly.

**Phase 1 target**: Regain historical user base and grow 20-30% year-over-year
**Phase 2-3 target**: Expand into institutional markets (schools, districts) for accelerated growth

### Market Trends Favoring MLC

**Digital Transformation in Education**
- COVID-19 permanently shifted music education to hybrid/digital models
- Teachers now expect and use digital tools routinely
- Students are comfortable with online learning platforms

**Data-Driven Instruction**
- Schools require measurable learning outcomes for accountability
- Quantitative assessment (not just engagement time) increasingly valued
- MLC's mastery tracking aligns with this trend

**Special Needs Inclusion**
- Growing demand for accessible, inclusive learning tools
- MLC's documented success with neurodiverse learners is timely
- See [Accessibility](../16-accessibility-and-inclusion/neurodiverse-support.md)

**Subscription Fatigue and Bundling Pressure**
- Schools consolidating tools to reduce vendor management
- Opportunity: Position MLC as comprehensive (not needing multiple tools)
- Risk: Schools choose bundled platforms (MusicFirst) over best-of-breed tools

---

## Strategic Recommendations

### Product Roadmap Priorities

Based on competitive analysis, prioritize features that:

1. **Strengthen differentiation** (method alignment, retention testing, bulk operations)
2. **Address table-stakes gaps** (responsive design, modern UX, teacher dashboards)
3. **Defend against threats** (basic LMS integration, COPPA/FERPA compliance for schools)
4. **Exploit opportunities** (AI-assisted sequences, special needs positioning)

**Phase 1 must-haves** (see [Phase 1 Scope](../24-roadmap-and-phasing/phase-1-scope.md)):
- Modern, responsive UX (address "platform age" weakness)
- Teacher assignment builder with method alignment (strengthen differentiation)
- Student dashboard with progress tracking (match competitors)
- Basic gamification (badges, points, streaks) (table stakes)

**Phase 2 priorities** (see [Phase 2 Scope](../24-roadmap-and-phasing/phase-2-scope.md)):
- Bulk CSV import/export (institutional sales enabler)
- Purchase order and invoicing workflows (school operations)
- Mass assignment tools (school scale)
- Role hierarchy management (enterprise feature)

**Phase 3 opportunities** (see [Phase 3 Scope](../24-roadmap-and-phasing/phase-3-scope.md)):
- AI-assisted sequence generation (competitive lead)
- Basic LMS integration (LTI, Clever, Classlink) (threat mitigation)
- Advanced reporting and analytics (data-driven instruction trend)

### Pricing Strategy

**Maintain current pricing model**:
- Solo: $7.95/month (5 seats) + $0.80/seat up to 19 seats
- Ensemble: $19.95/month (20 seats) + volume pricing beyond 20
- 30-day Prelude trial (19 seats, no credit card)

**Rationale**:
- Pricing is **competitive** with private teacher tools (Piano Marvel ~$10-15/month, Sight Reading Factory ~$60/year)
- Pricing is **below** institutional platforms (SmartMusic ~$40/student/year, MusicFirst ~$2000+/school)
- 30-day trial is **longer** than most competitors (7-14 days typical), creating conversion advantage
- Volume pricing scales well for schools (cost-effective at 50+ students)

**Consider**:
- **Educational discounts** for Title I schools or underserved communities
- **District pricing** for 500+ students (custom quotes)
- **Annual prepay discounts** (10-15% off) to improve cash flow and retention

See [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) for detailed pricing analysis.

### Marketing and Positioning

**Target personas** (see [Personas](../00-foundations/personas.md)):
1. **Private piano teachers** (Solo tier) — emphasize method alignment and time savings
2. **Music school owners** (Ensemble tier) — emphasize bulk operations and multi-teacher management
3. **K-12 general music teachers** (Ensemble tier) — emphasize classroom engagement and assessment
4. **Homeschool parents** (Solo tier) — emphasize self-paced learning and measurable progress

**Core messaging themes**:
- "Works with your teaching method, not against it"
- "Measurable mastery, not just engagement"
- "Practice games that stick: retention testing built-in"
- "From solo teachers to large districts: scales with you"

**Channels**:
- **Teacher conferences**: MTNA, NAfME, state associations (demonstrate product, build relationships)
- **Method publisher partnerships**: Co-marketing with Alfred, Faber (credibility and distribution)
- **Facebook groups**: Music teacher communities (organic referrals, testimonials)
- **Google/Facebook ads**: Target "piano teacher resources," "music theory games"
- **Content marketing**: Blog posts, YouTube videos demonstrating method alignment and teacher workflows

### Partnership Opportunities

**Method Book Publishers**:
- Alfred, Faber, Hal Leonard, RCM
- Opportunity: Co-branded sequences, joint marketing, distribution partnerships
- Value: Credibility, reach, reduced customer acquisition cost

**Educational Technology Platforms**:
- Clever, Classlink (rostering/SSO)
- Canvas, Schoology, Google Classroom (LMS integration)
- Value: Easier school adoption, reduced implementation friction

**Special Needs Organizations**:
- Autism advocacy groups, inclusive education nonprofits
- Opportunity: Case studies, testimonials, targeted marketing
- Value: Positioning as leader in accessible music education

**Music Retailers and Service Providers**:
- Piano stores, music lesson studios
- Opportunity: Referral partnerships, bundled offerings
- Value: Distribution channel for Solo tier subscriptions

---

## Dependencies

### Documents This Analysis Informs

- [Product Vision](../00-foundations/product-vision.md) — Core value propositions and differentiation
- [Non-Goals](../00-foundations/non-goals.md) — What to deliberately not build based on competitive landscape
- [Phase 1 Scope](../24-roadmap-and-phasing/phase-1-scope.md) — Feature prioritization based on competitive pressure
- [Phase 2 Scope](../24-roadmap-and-phasing/phase-2-scope.md) — Institutional features to compete for school market
- [Phase 3 Scope](../24-roadmap-and-phasing/phase-3-scope.md) — Advanced features for competitive lead
- [Plans and Pricing](../14-payments-and-subscriptions/plans-and-pricing.md) — Pricing strategy benchmarked against competitors

### Documents This Analysis References

- [Competitor Scan 2025](./competitor-scan-2025.md) — Raw competitive intelligence (who, what, pricing)
- [User Feedback](./user-feedback.md) — Validation of what users value (testimonials, success metrics)
- [Product Vision](../00-foundations/product-vision.md) — MLC's core mission and differentiation strategy
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) — Educational approach that differentiates MLC
- [Game Catalog](../07-content-architecture/game-catalog.md) — Content library scope for competitive comparison
- [Personas](../00-foundations/personas.md) — Target user segments for positioning
- [Sequence Alignment](../09-sequences-and-curriculum-tools/sequence-alignment-to-methods.md) — Key differentiator vs. competitors

---

## Open Questions

### Research Needs

- **Market sizing**: More precise TAM/SAM/SOM estimates (survey teachers, analyze competitor user bases)
- **Win/loss analysis**: Why do prospects choose MLC vs. competitors? Why do they churn to competitors?
- **Feature demand**: Which competitor features do users actually want vs. nice-to-have?
- **Pricing sensitivity**: Willingness-to-pay research for Solo and Ensemble tiers
- **Brand perception**: How is MLC perceived vs. competitors? (professional vs. dated, rigorous vs. boring, etc.)

### Strategic Decisions

- **Native mobile apps**: Reconsider if competitors' app presence significantly impacts conversion (see [Mobile Considerations](../21-mobile-and-offline/pwa-specifications.md))
- **LMS integration depth**: How deep to integrate with Clever, Canvas, etc.? (basic SSO vs. full rostering vs. gradebook sync)
- **Free tier**: Would freemium model accelerate growth despite lower revenue per user?
- **Content marketplace**: Should MLC ever allow third-party content? (See [Non-Goals](../00-foundations/non-goals.md))
- **Geographic expansion**: Which international markets to prioritize? (UK, Australia, Canada first?)

### Competitive Monitoring

- **Quarterly review**: Update competitive scan and reassess strategic positioning
- **Feature watch**: Track competitor product announcements and new feature releases
- **Pricing watch**: Monitor competitor pricing changes (especially volume/school tiers)
- **Partnership watch**: Which competitors are forming partnerships (publishers, LMS platforms, etc.)?
- **Funding watch**: Competitor funding rounds signal investment areas and competitive threats

---

## Supporting Documents Referenced

This competitor analysis specification draws from the following source documents:

### Primary Source Documents

- [MLC Executive Summary 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20Executive%20Summary%202020.docx.txt) — Core business model, subscription levels, value propositions, and historical context for competitive positioning
- [MLC History Draft 2020.docx.txt](../00-ORIG-CONTEXT/MLC%20History%20Draft%202020.docx.txt) — Founder background, 60,000+ student reach, 40+ country adoption, organic growth patterns, and 3-5 month advancement data
- [MLC Site Content.docx.txt](../00-ORIG-CONTEXT/MLC%20Site%20Content.docx.txt) — Marketing messaging, benefits statements, FAQs, and user-facing value propositions that inform positioning strategy
- [MLC Tiered Pricing 2020.xlsx - Full Calc.csv](../00-ORIG-CONTEXT/MLC%20Tiered%20Pricing%202020.xlsx%20-%20Full%20Calc.csv) — Pricing model details (Solo vs. Ensemble tiers, seat scaling, volume discounts) for competitive pricing analysis
- [Seats.docx.txt](../00-ORIG-CONTEXT/Seats.docx.txt) — Seat management model and billing approach informing institutional operations strategy
- [pianoAdventures.doc.txt](../00-ORIG-CONTEXT/pianoAdventures.doc.txt) — Method book alignment approach (key differentiator vs. competitors)
- [USER ROLES detail.docx.txt](../00-ORIG-CONTEXT/USER%20ROLES%20detail.docx.txt) — Role hierarchy model (Student, Teacher, Admin, SysAdmin) for institutional scale positioning
- [Game Structure.docx.txt](../00-ORIG-CONTEXT/Game%20Structure.docx.txt) — Learn/Play/Quiz/Challenge/Review staged model (unique pedagogical approach)
- [AlphaMasterList2020.xlsx - Master.csv](../00-ORIG-CONTEXT/AlphaMasterList2020.xlsx%20-%20Master.csv) — Game library scope (450+ games) for content comparison

### Internal MLC Context Documents

For understanding MLC's own positioning to contrast with competitors, reference:

- [Product Vision](../00-foundations/product-vision.md) — Core mission, value propositions, AI enablement vision, and success metrics
- [Pedagogy Principles](../00-foundations/pedagogy-principles.md) — Educational philosophy and mastery-first approach
- [Non-Goals](../00-foundations/non-goals.md) — Strategic boundaries and deliberate scope limitations
- [Personas](../00-foundations/personas.md) — Target user segments for market positioning
- [User Feedback](./user-feedback.md) — Testimonials, success metrics, and validation of competitive advantages
- [Competitor Scan 2025](./competitor-scan-2025.md) — Raw competitive intelligence that this analysis interprets strategically

---