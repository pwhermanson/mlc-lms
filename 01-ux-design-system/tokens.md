# Design Tokens

## Purpose

This document defines the design tokens for the MLC-LMS platform. Design tokens are the visual design atoms of the design system — specifically named entities that store visual design attributes. They ensure consistency across all components and interfaces while making it easy to maintain and update the design system.

## Color Tokens

### Primary Colors
```css
--color-primary-50: #EFF6FF;
--color-primary-100: #DBEAFE;
--color-primary-200: #BFDBFE;
--color-primary-300: #93C5FD;
--color-primary-400: #60A5FA;
--color-primary-500: #3B82F6;
--color-primary-600: #2563EB;
--color-primary-700: #1D4ED8;
--color-primary-800: #1E40AF;
--color-primary-900: #1E3A8A;
```

### Secondary Colors
```css
--color-success-50: #ECFDF5;
--color-success-100: #D1FAE5;
--color-success-200: #A7F3D0;
--color-success-300: #6EE7B7;
--color-success-400: #34D399;
--color-success-500: #10B981;
--color-success-600: #059669;
--color-success-700: #047857;
--color-success-800: #065F46;
--color-success-900: #064E3B;

--color-warning-50: #FFFBEB;
--color-warning-100: #FEF3C7;
--color-warning-200: #FDE68A;
--color-warning-300: #FCD34D;
--color-warning-400: #FBBF24;
--color-warning-500: #F59E0B;
--color-warning-600: #D97706;
--color-warning-700: #B45309;
--color-warning-800: #92400E;
--color-warning-900: #78350F;

--color-error-50: #FEF2F2;
--color-error-100: #FEE2E2;
--color-error-200: #FECACA;
--color-error-300: #FCA5A5;
--color-error-400: #F87171;
--color-error-500: #EF4444;
--color-error-600: #DC2626;
--color-error-700: #B91C1C;
--color-error-800: #991B1B;
--color-error-900: #7F1D1D;
```

### Neutral Colors
```css
--color-gray-50: #F9FAFB;
--color-gray-100: #F3F4F6;
--color-gray-200: #E5E7EB;
--color-gray-300: #D1D5DB;
--color-gray-400: #9CA3AF;
--color-gray-500: #6B7280;
--color-gray-600: #4B5563;
--color-gray-700: #374151;
--color-gray-800: #1F2937;
--color-gray-900: #111827;
```

### Semantic Colors
```css
--color-mastery-gold: #F59E0B;
--color-progress-blue: #3B82F6;
--color-challenge-purple: #8B5CF6;
--color-review-green: #10B981;
--color-text-primary: #111827;
--color-text-secondary: #374151;
--color-text-tertiary: #6B7280;
--color-background-primary: #FFFFFF;
--color-background-secondary: #F9FAFB;
--color-border-primary: #D1D5DB;
--color-border-secondary: #E5E7EB;
```

## Typography Tokens

### Font Families
```css
--font-family-primary: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
--font-family-mono: 'SF Mono', Monaco, 'Cascadia Code', 'Roboto Mono', Consolas, 'Courier New', monospace;
```

### Font Sizes
```css
--font-size-xs: 0.75rem;    /* 12px */
--font-size-sm: 0.875rem;   /* 14px */
--font-size-base: 1rem;     /* 16px */
--font-size-lg: 1.125rem;   /* 18px */
--font-size-xl: 1.25rem;    /* 20px */
--font-size-2xl: 1.5rem;    /* 24px */
--font-size-3xl: 1.875rem;  /* 30px */
--font-size-4xl: 2.25rem;   /* 36px */
--font-size-5xl: 3rem;      /* 48px */
--font-size-6xl: 3.75rem;   /* 60px */
```

### Font Weights
```css
--font-weight-thin: 100;
--font-weight-light: 300;
--font-weight-normal: 400;
--font-weight-medium: 500;
--font-weight-semibold: 600;
--font-weight-bold: 700;
--font-weight-extrabold: 800;
--font-weight-black: 900;
```

### Line Heights
```css
--line-height-none: 1;
--line-height-tight: 1.25;
--line-height-snug: 1.375;
--line-height-normal: 1.5;
--line-height-relaxed: 1.625;
--line-height-loose: 2;
```

### Letter Spacing
```css
--letter-spacing-tighter: -0.05em;
--letter-spacing-tight: -0.025em;
--letter-spacing-normal: 0em;
--letter-spacing-wide: 0.025em;
--letter-spacing-wider: 0.05em;
--letter-spacing-widest: 0.1em;
```

## Spacing Tokens

### Base Spacing Scale
```css
--space-0: 0;
--space-px: 1px;
--space-0-5: 0.125rem;  /* 2px */
--space-1: 0.25rem;     /* 4px */
--space-1-5: 0.375rem;  /* 6px */
--space-2: 0.5rem;      /* 8px */
--space-2-5: 0.625rem;  /* 10px */
--space-3: 0.75rem;     /* 12px */
--space-3-5: 0.875rem;  /* 14px */
--space-4: 1rem;        /* 16px */
--space-5: 1.25rem;     /* 20px */
--space-6: 1.5rem;      /* 24px */
--space-7: 1.75rem;     /* 28px */
--space-8: 2rem;        /* 32px */
--space-9: 2.25rem;     /* 36px */
--space-10: 2.5rem;     /* 40px */
--space-11: 2.75rem;    /* 44px */
--space-12: 3rem;       /* 48px */
--space-14: 3.5rem;     /* 56px */
--space-16: 4rem;       /* 64px */
--space-20: 5rem;       /* 80px */
--space-24: 6rem;       /* 96px */
--space-28: 7rem;       /* 112px */
--space-32: 8rem;       /* 128px */
```

### Component Spacing
```css
--space-component-xs: var(--space-2);   /* 8px */
--space-component-sm: var(--space-4);   /* 16px */
--space-component-md: var(--space-6);   /* 24px */
--space-component-lg: var(--space-8);   /* 32px */
--space-component-xl: var(--space-12);  /* 48px */
```

## Border Radius Tokens

### Base Radius Values
```css
--radius-none: 0;
--radius-sm: 0.125rem;   /* 2px */
--radius-base: 0.25rem;  /* 4px */
--radius-md: 0.375rem;   /* 6px */
--radius-lg: 0.5rem;     /* 8px */
--radius-xl: 0.75rem;    /* 12px */
--radius-2xl: 1rem;      /* 16px */
--radius-3xl: 1.5rem;    /* 24px */
--radius-full: 9999px;
```

### Component Radius
```css
--radius-button: var(--radius-md);
--radius-card: var(--radius-lg);
--radius-input: var(--radius-base);
--radius-modal: var(--radius-xl);
--radius-avatar: var(--radius-full);
```

## Shadow Tokens

### Shadow Definitions
```css
--shadow-xs: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
--shadow-sm: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
--shadow-base: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
--shadow-md: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
--shadow-lg: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
--shadow-xl: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
--shadow-2xl: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
--shadow-inner: inset 0 2px 4px 0 rgba(0, 0, 0, 0.06);
```

### Component Shadows
```css
--shadow-button: var(--shadow-sm);
--shadow-card: var(--shadow-base);
--shadow-modal: var(--shadow-xl);
--shadow-dropdown: var(--shadow-lg);
--shadow-focus: 0 0 0 3px rgba(59, 130, 246, 0.5);
```

## Z-Index Tokens

### Layer Definitions
```css
--z-index-dropdown: 1000;
--z-index-sticky: 1020;
--z-index-fixed: 1030;
--z-index-modal-backdrop: 1040;
--z-index-modal: 1050;
--z-index-popover: 1060;
--z-index-tooltip: 1070;
--z-index-toast: 1080;
```

## Breakpoint Tokens

### Responsive Breakpoints
```css
--breakpoint-sm: 640px;
--breakpoint-md: 768px;
--breakpoint-lg: 1024px;
--breakpoint-xl: 1280px;
--breakpoint-2xl: 1536px;
```

### Container Max Widths
```css
--container-sm: 640px;
--container-md: 768px;
--container-lg: 1024px;
--container-xl: 1280px;
--container-2xl: 1536px;
```

## Animation Tokens

### Duration
```css
--duration-75: 75ms;
--duration-100: 100ms;
--duration-150: 150ms;
--duration-200: 200ms;
--duration-300: 300ms;
--duration-500: 500ms;
--duration-700: 700ms;
--duration-1000: 1000ms;
```

### Easing Functions
```css
--ease-linear: linear;
--ease-in: cubic-bezier(0.4, 0, 1, 1);
--ease-out: cubic-bezier(0, 0, 0.2, 1);
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
--ease-bounce: cubic-bezier(0.68, -0.55, 0.265, 1.55);
```

### Common Animations
```css
--animation-fade-in: fadeIn var(--duration-200) var(--ease-out);
--animation-fade-out: fadeOut var(--duration-150) var(--ease-in);
--animation-slide-up: slideUp var(--duration-300) var(--ease-out);
--animation-slide-down: slideDown var(--duration-300) var(--ease-out);
--animation-scale-in: scaleIn var(--duration-200) var(--ease-out);
--animation-scale-out: scaleOut var(--duration-150) var(--ease-in);
```

## Accessibility Tokens

### Focus Indicators
```css
--focus-ring-width: 2px;
--focus-ring-offset: 2px;
--focus-ring-color: var(--color-primary-500);
--focus-ring-opacity: 0.5;
```

### High Contrast Colors
```css
--color-high-contrast-text: #000000;
--color-high-contrast-background: #FFFFFF;
--color-high-contrast-border: #000000;
--color-high-contrast-focus: #0000FF;
```

## Usage Guidelines

### CSS Custom Properties
All tokens are defined as CSS custom properties and can be used in stylesheets:
```css
.button {
  background-color: var(--color-primary-600);
  color: var(--color-white);
  padding: var(--space-3) var(--space-6);
  border-radius: var(--radius-button);
  box-shadow: var(--shadow-button);
  font-size: var(--font-size-base);
  font-weight: var(--font-weight-medium);
}
```

### JavaScript Access
Tokens can be accessed in JavaScript for dynamic styling:
```javascript
const primaryColor = getComputedStyle(document.documentElement)
  .getPropertyValue('--color-primary-600');
```

### Design System Integration
Tokens should be used consistently across all components and interfaces to ensure visual consistency and maintainability.

## Token Maintenance

### Adding New Tokens
1. Define the token in the appropriate category
2. Use semantic naming conventions
3. Document the token's purpose and usage
4. Update component implementations
5. Test across all breakpoints and themes

### Updating Existing Tokens
1. Consider backward compatibility
2. Update all references to the token
3. Test visual regression
4. Document changes in changelog
5. Communicate changes to the team

### Token Naming Conventions
- Use kebab-case for all token names
- Include category prefix (color-, font-, space-, etc.)
- Use semantic names when possible
- Include scale indicators for numeric values
- Be descriptive but concise

