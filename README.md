# MusicLearningCommunity LMS (MLC 5.0)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Next.js](https://img.shields.io/badge/Next.js-14-black)](https://nextjs.org/)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.0-blue)](https://www.typescriptlang.org/)
[![React](https://img.shields.io/badge/React-18-blue)](https://reactjs.org/)

> A mastery-first, AI-enabled music learning platform that maps any teaching method to the exact practice a student needs, scales from single studios to large schools, and proves learning through quantitative outcomes.

## 🎵 What is MLC?

MusicLearningCommunity (MLC) is a learning platform that teaches music literacy with game-based, mastery-focused learning. Each game covers a single concept, delivered as staged experiences: **Learn**, **Play**, **Quiz**, optional **Challenge**, and a distinct **Review** stage that records a separate score.

Students can work through automated Sequences such as Lifetime Musician, Solfege, and Evaluation, or through method-aligned sequences that mirror popular books and curricula. The experience is accessible to students at home or school and is designed to reinforce teacher-led instruction.

### Key Features

- 🎮 **Game-based Learning**: 450+ concept-focused games covering all music literacy areas
- 📊 **Quantitative Assessment**: Target scores by stage with retained history and scheduled Review
- 🎯 **Mastery-First Approach**: Students advance 3-5 months ahead of peers after one year of use
- 🏫 **School-Ready Operations**: Bulk CSV tools, mass assignment, role hierarchies
- ♿ **Accessibility**: Proven effectiveness for students with special needs, including autism
- 🎹 **MIDI Integration**: Full support for MIDI keyboards via USB or Bluetooth
- 🌍 **Global Reach**: Served 60,000+ students across 40+ countries since 2006

## 🚀 Quick Start

### Prerequisites

- Node.js 18+ 
- npm or yarn
- PostgreSQL database (or Neon account)
- Cloudflare R2 account (for file storage)

### Installation

```bash
# Clone the repository
git clone https://github.com/your-org/mlc-lms.git
cd mlc-lms

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with your configuration

# Run database migrations
npm run db:migrate

# Start development server
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000) to see the application.

## 🏗️ Technology Stack

### Core Technologies
- **Frontend**: Next.js 14 + React 18 + TypeScript
- **Styling**: Tailwind CSS + shadcn/ui
- **Database**: Neon PostgreSQL (serverless)
- **Storage**: Cloudflare R2 (S3-compatible)
- **Authentication**: NextAuth.js
- **Deployment**: Vercel

### Music-Specific Libraries
- **VexFlow**: Music notation rendering
- **Tone.js + WebMIDI.js**: Audio and MIDI support
- **Tonal + Scribbletune**: Music theory calculations and generation

### Additional Tools
- **XState**: State machine orchestration for game stages
- **TanStack Query**: Data fetching and caching
- **React Hook Form**: Form handling
- **Framer Motion**: Animations and transitions
- **PapaParse**: CSV processing for bulk operations

See [Technology Stack Documentation](00-foundations/techstack.md) for complete details.

## 📁 Project Structure

```
mlc-lms/
├── 00-foundations/          # Core documentation and specifications
├── 01-ux-design-system/     # Design system and UI components
├── 02-roles-and-permissions/ # User roles and access control
├── 03-student-experience/   # Student-facing features
├── 04-teacher-experience/   # Teacher tools and interfaces
├── 05-admin-subscriber-experience/ # Admin dashboards
├── 06-system-admin/         # System administration
├── 07-content-architecture/ # Content management
├── 08-games-registry-and-authoring/ # Game creation tools
├── 09-sequences-and-curriculum-tools/ # Curriculum management
├── 10-assignments-engine/   # Assignment system
├── 11-search-and-discovery/ # Search and content discovery
├── 12-gamification/         # Badges, leaderboards, achievements
├── 13-messaging-and-notifications/ # Communication system
├── 14-payments-and-subscriptions/ # Billing and subscriptions
├── 15-analytics-and-reporting/ # Analytics and reporting
├── 16-accessibility-and-inclusion/ # Accessibility features
├── 17-integrations/         # Third-party integrations
├── 18-architecture/         # System architecture
├── 19-quality-and-operations/ # Quality assurance
├── 20-imports-and-automation/ # Import/export tools
├── 21-mobile-and-offline/   # Mobile and offline support
├── 22-content-style-guides/ # Content guidelines
├── 23-legal-and-compliance/ # Legal and compliance
├── 24-roadmap-and-phasing/  # Development roadmap
├── 25-research-and-inspiration/ # Research and inspiration
├── 26-repo-operations/      # Repository operations
├── 27-social-learning/      # Social learning features
├── 28-advanced-assessment/  # Advanced assessment tools
├── 29-content-management/   # Content management system
├── 30-ai-and-machine-learning/ # AI and ML features
└── 00-TEMP-CONSTRUCTION-DOCS/ # Temporary construction documents
```

## 🎯 Target Users

### Students (Ages 3-87)
- Individual learners and group class participants
- Special needs learners (proven effective for autism spectrum)
- Both regular students (unique accounts) and generic students (shared accounts)

### Teachers
- Private studio instructors
- Public school music teachers
- Homeschool educators
- Can assign sequences, monitor progress, and communicate with students

### Administrators
- **Teacher-Admins**: Manage multiple teachers and students in Ensemble subscriptions
- **Subscriber-Admins**: Handle billing, seat allocation, and subscription management
- **System Admin**: Global user management and system-wide analytics

## 🎮 Game Structure

Each game follows a structured learning progression:

1. **Learn** - Tutorial stage introducing the concept
2. **Play** - Practice stage with immediate feedback
3. **Quiz** - Mastery assessment with quantitative scoring
4. **Challenge** - Optional worldwide competition (not all games)
5. **Review** - Retention testing scheduled based on mastery

### Game Categories
- **Aural**: Pitch, Rhythm, Melody, Intervals
- **Visual**: Staff, Symbols, Keyboard Elements
- **Theory**: Chords, Scales, Musical Terms
- **Skills**: Memory Playing, Pattern Playing

## 📊 Pricing Tiers

### Prelude (Free Trial)
- 19 student slots
- 30-day free trial
- Full access to all features

### Solo (5-19 seats)
- $7.95/month base
- $0.80 per additional seat
- Perfect for private studios

### Ensemble (20+ seats)
- $19.95/month base
- $0.20 per additional seat (in sets of 5)
- Designed for schools and large institutions

## 🛠️ Development

### Available Scripts

```bash
# Development
npm run dev          # Start development server
npm run build        # Build for production
npm run start        # Start production server
npm run lint         # Run ESLint
npm run type-check   # Run TypeScript checks

# Database
npm run db:migrate   # Run database migrations
npm run db:seed      # Seed database with sample data
npm run db:reset     # Reset database

# Testing
npm run test         # Run tests
npm run test:watch   # Run tests in watch mode
npm run test:coverage # Run tests with coverage
```

### Environment Variables

Create a `.env.local` file with the following variables:

```env
# Database
DATABASE_URL="postgresql://username:password@host:port/database"

# Authentication
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="your-secret-key"

# Cloudflare R2
R2_ACCOUNT_ID="your-account-id"
R2_ACCESS_KEY_ID="your-access-key"
R2_SECRET_ACCESS_KEY="your-secret-key"
R2_BUCKET_NAME="your-bucket-name"
```

## 📚 Documentation

- [Executive Summary](00-foundations/executive-summary.md) - Project overview and goals
- [Product Vision](00-foundations/product-vision.md) - Vision and strategic direction
- [Technology Stack](00-foundations/techstack.md) - Complete technical architecture
- [Open Source Components](00-TEMP-CONSTRUCTION-DOCS/Open-source%20github%20repos%20for%20consideration%20.md) - Research on open-source libraries

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Workflow

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🏆 Success Metrics

- **Learning Outcomes**: Students advance 3-5 months ahead of peers
- **Teacher Efficiency**: 50% reduction in assignment prep time
- **Scale**: Support thousands of students per school
- **Accessibility**: Proven effectiveness for special needs learners
- **Global Reach**: 60,000+ students across 40+ countries

## 📞 Support

- **Documentation**: Check the `/00-foundations/` directory for comprehensive documentation
- **Issues**: Use GitHub Issues for bug reports and feature requests
- **Discussions**: Use GitHub Discussions for questions and community support

## 🙏 Acknowledgments

Founded by Christine D. Morton Hermanson, a University of Michigan Music School graduate with K-12 teaching certification and over 30 years of experience in music education technology innovation. Christine's work includes partnerships with early music education software companies, 17 years directing the MTNA National Symposium on Technology in Music, and creating the original MLC games using Adobe Flash for her Master's thesis.

---

**MusicLearningCommunity** - Building lifetime musicians through technology, one game at a time. 🎵
