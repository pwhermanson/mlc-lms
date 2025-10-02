# MLC LMS Technology Stack

## Overview

The MLC LMS platform is built using a modern, open-source technology stack designed for full licensing control while maintaining cost-effectiveness through managed cloud services. The architecture prioritizes developer experience, scalability, and the ability to maintain complete ownership of code and data.

## Core Philosophy

- **Full Licensing Control**: All application code is open-source and owned by MLC
- **Cost-Effective Start**: Begin with free tiers, scale as needed
- **No Server Management**: Leverage managed services to focus on product development
- **Future Flexibility**: Easy migration path to self-hosted solutions when needed

## Technology Stack

### Frontend
- **Next.js** - React framework with server-side rendering and static site generation
- **React** - Component-based UI library
- **TypeScript** - Type-safe JavaScript for better code quality
- **Tailwind CSS** - Utility-first CSS framework for styling
- **shadcn/ui** - High-quality, accessible UI component library built on Radix UI and Tailwind CSS

### Backend
- **Next.js API Routes** - Serverless functions for backend logic
- **NextAuth.js** - Authentication and session management
- **Neon PostgreSQL** - Serverless database for data storage
- **Cloudflare R2** - Object storage for game assets and files

### Infrastructure
- **Vercel** - Application hosting and deployment platform
- **Cloudflare** - CDN and security services
- **GitHub** - Version control and deployment triggers

## Architecture Overview

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   User Browser  │    │     Vercel      │    │     Neon DB     │
│                 │◄──►│   (Next.js App) │◄──►│  (PostgreSQL)   │
│  - React UI     │    │                 │    │                 │
│  - Phaser Games │    │  - API Routes   │    │  - User Data    │
└─────────────────┘    │  - Serverless   │    │  - Game Data    │
                       │  - CDN          │    │  - Progress     │
                       └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │  Cloudflare R2  │
                       │  (File Storage) │
                       │  - Game Assets  │
                       │  - User Uploads │
                       └─────────────────┘
```

## Key Benefits

### Cost-Effective
- **$0/month** to start with free tiers
- Scales with usage - pay only for what you use
- No server management or maintenance costs

### Developer Experience
- Single codebase with TypeScript throughout
- Hot reloading and instant feedback
- Automatic deployments from GitHub
- Comprehensive debugging tools

### Performance
- Global CDN for fast content delivery
- Serverless functions scale automatically
- Database connection pooling
- Optimized builds and caching

### Maintainability
- Open-source foundation
- Type safety reduces bugs
- Clear separation of concerns
- Easy to test and debug

## Database Schema

### Core Tables
- **users** - User accounts and authentication
- **games** - Game definitions and configurations
- **user_progress** - Student progress and scores
- **assignments** - Teacher-created assignments
- **courses** - Course structures and curriculum

### Data Storage
- **Structured Data** - PostgreSQL for relational data
- **File Assets** - Cloudflare R2 for media and game files
- **Session Data** - JWT tokens for stateless authentication

## Authentication & Security

### Authentication Flow
- NextAuth.js handles user authentication
- JWT tokens for stateless sessions
- Password hashing with bcrypt
- Session management and security

### Security Features
- HTTPS enforced everywhere
- CSRF protection
- SQL injection prevention
- XSS protection
- Secure cookie handling

## Development Workflow

### Local Development
- Docker Compose for local database
- Hot reloading with Next.js dev server
- TypeScript compilation and checking
- Environment variable management

### Deployment Process
1. Push code to GitHub
2. Vercel automatically detects Next.js
3. Builds and deploys to global CDN
4. Database migrations run automatically
5. Application is live worldwide

## File Structure

```
mlc-lms/
├── components/          # Reusable React components
├── pages/              # Next.js pages and API routes
│   ├── api/           # Serverless API endpoints
│   └── [pages]/       # Frontend pages
├── lib/               # Utility functions and configurations
├── public/            # Static assets
├── styles/            # CSS and styling
├── types/             # TypeScript type definitions
└── docs/              # Documentation
```

## Environment Configuration

### Required Environment Variables
- `DATABASE_URL` - Neon PostgreSQL connection string
- `NEXTAUTH_URL` - Application URL for authentication
- `NEXTAUTH_SECRET` - Secret key for JWT signing
- `R2_ACCOUNT_ID` - Cloudflare R2 account identifier
- `R2_ACCESS_KEY_ID` - R2 access key
- `R2_SECRET_ACCESS_KEY` - R2 secret key
- `R2_BUCKET_NAME` - R2 bucket name for assets

## Migration Path

### Phase 1: Start with Managed Services
- Use Vercel, Neon, and Cloudflare R2
- Focus on building the application
- Validate the product-market fit

### Phase 2: Scale as Needed
- Upgrade to paid tiers as usage grows
- Add additional services (Redis, monitoring)
- Optimize performance and costs

### Phase 3: Consider Self-Hosting
- When revenue justifies the complexity
- Migrate to self-hosted solutions
- Maintain full control over infrastructure

## Technology Choices Rationale

### Why Next.js?
- Built on React (familiar and powerful)
- Server-side rendering for better SEO
- API routes eliminate need for separate backend
- Excellent developer experience
- Strong ecosystem and community

### Why Neon?
- Serverless PostgreSQL (no server management)
- Automatic scaling and backups
- Built-in connection pooling
- Cost-effective for small to medium usage
- Easy migration to self-hosted PostgreSQL

### Why Cloudflare R2?
- S3-compatible API (easy migration)
- No egress fees (unlike AWS S3)
- Global CDN included
- Cost-effective for file storage
- Easy integration with Next.js

### Why Vercel?
- Optimized for Next.js applications
- Automatic deployments from GitHub
- Global CDN and edge functions
- Excellent developer experience
- Easy scaling and monitoring

### Why shadcn/ui?
- Copy-paste components (not a dependency) - full control over code
- Built on Radix UI primitives for accessibility
- Styled with Tailwind CSS for consistency
- TypeScript-first with excellent type safety
- Highly customizable and themeable
- Perfect fit for educational interfaces
- No runtime dependencies - components are part of your codebase

## Future Considerations

### Potential Additions
- **Redis** - For caching and session storage
- **Elasticsearch** - For advanced search capabilities
- **WebSocket support** - For real-time features
- **Mobile app** - Using React Native
- **Analytics** - For user behavior tracking

## Backburner Items

### Phaser.js Game Presentation Layer (Future Enhancement)

**Context**: We have an existing library of individual HTML5 games that need unified presentation and management within the LMS.

**Proposed Use**: Phaser would be used **selectively** as a game presentation and management system, not for creating new games:

- **Game Launcher/Container**: Create a unified interface that loads and presents existing HTML5 games
- **Consistent UI/UX**: Provide consistent menus, transitions, and navigation between games
- **Progress Tracking Interface**: Implement visual progress tracking and completion indicators
- **Game State Management**: Track which games are locked/unlocked, completed, etc.
- **Unified Scoring System**: Standardize how scores are collected and displayed across games

**Why Backburner**: 
- Our existing HTML5 games already work and don't need to be rebuilt
- Phaser doesn't provide built-in gamification features (leaderboards, achievements, etc.)
- Initial focus should be on core LMS functionality and basic game integration
- Gamification can be implemented in the main application frontend rather than within Phaser

**Implementation Approach**:
- Use Phaser as a **wrapper/launcher** for existing games rather than recreating them
- Build gamification features in the main React frontend
- Consider Phaser only after core LMS functionality is validated and working

**Note**: Phaser is MIT-licensed and aligns with our open-source requirements, but it's not essential for the initial launch since we already have working HTML5 games.

### Scaling Strategies
- Horizontal scaling with serverless functions
- Database read replicas for heavy read workloads
- CDN optimization for global performance
- Caching strategies for frequently accessed data

## Proposed Technology Stack

Based on research into open-source components for the MLC LMS platform (see `00-TEMP-CONSTRUCTION-DOCS/Open-source github repos for consideration.md`), the following updated technology stack is proposed:

### **Frontend**: Next.js + React + TypeScript + Tailwind CSS + shadcn/ui
- **State Management**: XState + TanStack Query
- **Forms**: React Hook Form
- **Animations**: Framer Motion
- **Music**: VexFlow + Tone.js + WebMIDI.js + Tonal + Scribbletune
- **Data**: PapaParse + React Table + React Window
- **UI**: React Hot Toast + React Dropzone

### **Key Additions from Open-Source Research**:
- **XState**: Essential for managing the staged game system (Learn→Play→Quiz→Challenge→Review)
- **VexFlow + Tone.js + WebMIDI.js**: Critical music education trio for notation, audio, and MIDI support
- **Tonal + Scribbletune**: Music theory calculations and algorithmic music generation
- **React Hook Form**: Robust form handling for admin interfaces
- **TanStack Query**: Efficient data fetching and caching
- **Framer Motion**: Engaging animations and transitions
- **PapaParse**: CSV processing for school-scale requirements
- **React Table + React Window**: Data tables and performance optimization for large datasets
- **React Hot Toast + React Dropzone**: User feedback and file upload functionality

*Reference: See detailed analysis and implementation recommendations in `00-TEMP-CONSTRUCTION-DOCS/Open-source github repos for consideration.md`*

This technology stack provides a solid foundation for building a modern, scalable, and maintainable LMS platform while maintaining full control over the codebase and data.
