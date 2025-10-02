# Design Language

## Purpose

This document defines the visual design language and style guidelines for the MLC-LMS platform. It ensures consistency across all user interfaces while supporting the platform's educational mission and accessibility requirements.

## Design Philosophy

### Educational Focus
- Clean, uncluttered interfaces that don't distract from learning
- Clear visual hierarchy that guides attention to important content
- Consistent patterns that reduce cognitive load for students
- Engaging but not overwhelming visual design

### Accessibility First
- High contrast ratios for readability
- Clear typography and spacing
- Predictable interaction patterns
- Support for assistive technologies

### Scalable and Consistent
- Design system that works across all user roles
- Responsive design that adapts to different screen sizes
- Consistent visual language across all features
- Easy to maintain and extend

## Visual Identity

### Brand Personality
- **Approachable**: Friendly and welcoming to learners of all ages
- **Professional**: Reliable and trustworthy for educators
- **Engaging**: Visually interesting without being distracting
- **Clear**: Easy to understand and navigate

### Color Philosophy
- Use color purposefully to guide attention and indicate status
- Ensure sufficient contrast for accessibility
- Support both light and dark themes
- Use color consistently across all interfaces

## Typography

### Font Hierarchy
- **Primary Font**: System fonts for optimal performance and accessibility
- **Fallback Stack**: Arial, Helvetica, sans-serif
- **Monospace**: For code, technical content, and data display

### Text Sizes
- **Heading 1**: 2.5rem (40px) - Page titles and major sections
- **Heading 2**: 2rem (32px) - Section headers
- **Heading 3**: 1.5rem (24px) - Subsection headers
- **Heading 4**: 1.25rem (20px) - Component headers
- **Body Large**: 1.125rem (18px) - Important body text
- **Body Regular**: 1rem (16px) - Standard body text
- **Body Small**: 0.875rem (14px) - Secondary information
- **Caption**: 0.75rem (12px) - Labels and fine print

### Line Height
- **Headings**: 1.2 - Tight spacing for visual impact
- **Body Text**: 1.5 - Comfortable reading experience
- **Dense Content**: 1.4 - For data tables and lists

## Color System

### Primary Colors
- **Primary Blue**: #2563EB - Main brand color, CTAs, links
- **Primary Blue Dark**: #1D4ED8 - Hover states, active elements
- **Primary Blue Light**: #3B82F6 - Secondary actions, highlights

### Secondary Colors
- **Success Green**: #10B981 - Success states, completed tasks
- **Warning Orange**: #F59E0B - Warnings, attention needed
- **Error Red**: #EF4444 - Errors, critical actions
- **Info Blue**: #3B82F6 - Information, neutral actions

### Neutral Colors
- **Gray 900**: #111827 - Primary text, headings
- **Gray 700**: #374151 - Secondary text, labels
- **Gray 500**: #6B7280 - Tertiary text, placeholders
- **Gray 300**: #D1D5DB - Borders, dividers
- **Gray 100**: #F3F4F6 - Backgrounds, cards
- **Gray 50**: #F9FAFB - Page backgrounds
- **White**: #FFFFFF - Pure white backgrounds

### Semantic Colors
- **Mastery Gold**: #F59E0B - Achievement, mastery indicators
- **Progress Blue**: #3B82F6 - Progress bars, completion
- **Challenge Purple**: #8B5CF6 - Challenge stages, advanced content
- **Review Green**: #10B981 - Review stages, retention

## Spacing System

### Base Unit
- **4px**: Base spacing unit for consistent rhythm
- **8px**: Small spacing (2 units)
- **16px**: Medium spacing (4 units)
- **24px**: Large spacing (6 units)
- **32px**: Extra large spacing (8 units)
- **48px**: Section spacing (12 units)
- **64px**: Page spacing (16 units)

### Component Spacing
- **Padding**: 16px standard, 24px for cards
- **Margins**: 16px between elements, 32px between sections
- **Gaps**: 8px for small gaps, 16px for medium gaps
- **Borders**: 1px standard, 2px for emphasis

## Layout Principles

### Grid System
- **12-column grid** for flexible layouts
- **Breakpoints**: Mobile (320px), Tablet (768px), Desktop (1024px)
- **Container max-width**: 1200px for optimal reading
- **Gutters**: 16px on mobile, 24px on desktop

### Component Layout
- **Cards**: Rounded corners (8px), subtle shadows
- **Buttons**: Consistent padding and border radius
- **Forms**: Clear labels and input spacing
- **Navigation**: Consistent spacing and alignment

## Interactive Elements

### Buttons
- **Primary**: Solid background, white text
- **Secondary**: Outline style, colored text
- **Tertiary**: Text-only, colored text
- **Size variants**: Small (32px), Medium (40px), Large (48px)
- **States**: Default, Hover, Active, Disabled, Loading

### Form Controls
- **Input fields**: Clear borders, focus states
- **Selects**: Dropdown with clear options
- **Checkboxes**: Clear checked/unchecked states
- **Radio buttons**: Clear selection states
- **Labels**: Always associated with controls

### Navigation
- **Primary nav**: Horizontal, clear hierarchy
- **Secondary nav**: Breadcrumbs, pagination
- **Mobile nav**: Collapsible, touch-friendly
- **Active states**: Clear indication of current page

## Visual Feedback

### Loading States
- **Skeleton screens**: For content loading
- **Spinners**: For actions in progress
- **Progress bars**: For multi-step processes
- **Placeholders**: For empty states

### Success States
- **Checkmarks**: For completed actions
- **Toast notifications**: For success messages
- **Color changes**: Green for success
- **Animation**: Subtle success animations

### Error States
- **Error messages**: Clear, actionable text
- **Field highlighting**: Red borders for errors
- **Toast notifications**: For error messages
- **Help text**: Guidance for fixing errors

## Animation and Motion

### Principles
- **Purposeful**: Animations should serve a function
- **Subtle**: Not distracting from content
- **Consistent**: Same animations for similar actions
- **Accessible**: Respect reduced motion preferences

### Common Animations
- **Fade in/out**: For content appearing/disappearing
- **Slide**: For navigation and panels
- **Scale**: For button interactions
- **Progress**: For loading and completion

### Duration
- **Fast**: 150ms for micro-interactions
- **Medium**: 300ms for page transitions
- **Slow**: 500ms for complex animations

## Accessibility Guidelines

### Color Contrast
- **AA Standard**: 4.5:1 for normal text
- **AAA Standard**: 7:1 for large text
- **Color independence**: Information not conveyed by color alone

### Focus Management
- **Visible focus**: Clear focus indicators
- **Logical order**: Tab order follows visual hierarchy
- **Skip links**: For keyboard navigation
- **Focus trapping**: In modals and overlays

### Text and Content
- **Readable fonts**: Clear, legible typefaces
- **Adequate spacing**: Comfortable line height and spacing
- **Clear labels**: Descriptive text for all elements
- **Alternative text**: For images and icons

## Responsive Design

### Mobile First
- **320px**: Minimum supported width
- **Touch targets**: Minimum 44px for touch
- **Readable text**: Minimum 16px font size
- **Thumb-friendly**: Important actions within thumb reach

### Tablet Design
- **768px+**: Tablet-optimized layouts
- **Sidebar navigation**: Persistent navigation
- **Split views**: Content and navigation side-by-side
- **Touch and mouse**: Support both input methods

### Desktop Design
- **1024px+**: Full desktop experience
- **Hover states**: Rich hover interactions
- **Keyboard shortcuts**: Power user features
- **Multi-column layouts**: Efficient use of space

## Implementation Guidelines

### CSS Architecture
- **Component-based**: Modular CSS components
- **Utility classes**: For common patterns
- **CSS custom properties**: For theming and consistency
- **Mobile-first**: Responsive design approach

### Design Tokens
- **Colors**: Semantic color tokens
- **Spacing**: Consistent spacing scale
- **Typography**: Font and text tokens
- **Shadows**: Consistent shadow system

### Quality Assurance
- **Cross-browser testing**: Ensure consistency
- **Accessibility testing**: WCAG compliance
- **Performance**: Optimize for speed
- **User testing**: Validate with real users

