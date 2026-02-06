# Food Ordering System - Style Guide

## Overview
This style guide defines the visual design system, component patterns, and coding conventions for the food ordering system.

## Design System

### Color Palette

#### Primary Colors
```css
--primary-50: #FFF7ED;   /* Light orange tint */
--primary-100: #FFEDD5;
--primary-200: #FED7AA;
--primary-300: #FDBA74;
--primary-400: #FB923C;
--primary-500: #F97316;  /* Main brand color - Orange */
--primary-600: #EA580C;
--primary-700: #C2410C;
--primary-800: #9A3412;
--primary-900: #7C2D12;
```

#### Neutral Colors
```css
--neutral-50: #FAFAFA;
--neutral-100: #F5F5F5;
--neutral-200: #E5E5E5;
--neutral-300: #D4D4D4;
--neutral-400: #A3A3A3;
--neutral-500: #737373;
--neutral-600: #525252;
--neutral-700: #404040;
--neutral-800: #262626;
--neutral-900: #171717;
```

#### Semantic Colors
```css
--success: #10B981;   /* Green for confirmed orders */
--warning: #F59E0B;   /* Amber for pending states */
--error: #EF4444;     /* Red for errors/cancellations */
--info: #3B82F6;      /* Blue for informational messages */
```

### Typography

#### Font Family
```css
--font-primary: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
--font-heading: 'Poppins', 'Inter', sans-serif;
```

#### Font Sizes
```css
--text-xs: 0.75rem;    /* 12px */
--text-sm: 0.875rem;   /* 14px */
--text-base: 1rem;     /* 16px */
--text-lg: 1.125rem;   /* 18px */
--text-xl: 1.25rem;    /* 20px */
--text-2xl: 1.5rem;    /* 24px */
--text-3xl: 1.875rem;  /* 30px */
--text-4xl: 2.25rem;   /* 36px */
```

#### Font Weights
```css
--font-normal: 400;
--font-medium: 500;
--font-semibold: 600;
--font-bold: 700;
```

### Spacing
```css
--space-1: 0.25rem;   /* 4px */
--space-2: 0.5rem;    /* 8px */
--space-3: 0.75rem;   /* 12px */
--space-4: 1rem;      /* 16px */
--space-6: 1.5rem;    /* 24px */
--space-8: 2rem;      /* 32px */
--space-12: 3rem;     /* 48px */
--space-16: 4rem;     /* 64px */
```

### Border Radius
```css
--radius-sm: 0.25rem;   /* 4px */
--radius-md: 0.5rem;    /* 8px */
--radius-lg: 0.75rem;   /* 12px */
--radius-xl: 1rem;      /* 16px */
--radius-full: 9999px;  /* Fully rounded */
```

### Shadows
```css
--shadow-sm: 0 1px 2px 0 rgb(0 0 0 / 0.05);
--shadow-md: 0 4px 6px -1px rgb(0 0 0 / 0.1);
--shadow-lg: 0 10px 15px -3px rgb(0 0 0 / 0.1);
--shadow-xl: 0 20px 25px -5px rgb(0 0 0 / 0.1);
```

## Component Patterns

### Button Styles

#### Primary Button
```tsx
<button className="btn-primary">
  Place Order
</button>
```

```css
.btn-primary {
  background-color: var(--primary-500);
  color: white;
  padding: var(--space-3) var(--space-6);
  border-radius: var(--radius-md);
  font-weight: var(--font-semibold);
  transition: background-color 0.2s;
}

.btn-primary:hover {
  background-color: var(--primary-600);
}
```

#### Secondary Button
```css
.btn-secondary {
  background-color: white;
  color: var(--primary-500);
  border: 2px solid var(--primary-500);
  padding: var(--space-3) var(--space-6);
  border-radius: var(--radius-md);
  font-weight: var(--font-semibold);
}
```

### Card Component
```tsx
<div className="card">
  <img src="/food.jpg" alt="Food item" className="card-image" />
  <div className="card-content">
    <h3 className="card-title">Margherita Pizza</h3>
    <p className="card-description">Fresh mozzarella, tomatoes, basil</p>
    <div className="card-footer">
      <span className="card-price">$12.99</span>
      <button className="btn-primary">Add to Cart</button>
    </div>
  </div>
</div>
```

```css
.card {
  background: white;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
  overflow: hidden;
  transition: transform 0.2s, box-shadow 0.2s;
}

.card:hover {
  transform: translateY(-4px);
  box-shadow: var(--shadow-lg);
}

.card-image {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.card-content {
  padding: var(--space-4);
}

.card-title {
  font-size: var(--text-lg);
  font-weight: var(--font-semibold);
  margin-bottom: var(--space-2);
}

.card-price {
  font-size: var(--text-xl);
  font-weight: var(--font-bold);
  color: var(--primary-500);
}
```

### Badge Component
```tsx
<span className="badge badge-success">Delivered</span>
<span className="badge badge-warning">Preparing</span>
<span className="badge badge-error">Cancelled</span>
```

```css
.badge {
  display: inline-block;
  padding: var(--space-1) var(--space-3);
  border-radius: var(--radius-full);
  font-size: var(--text-sm);
  font-weight: var(--font-medium);
}

.badge-success {
  background-color: #D1FAE5;
  color: #065F46;
}

.badge-warning {
  background-color: #FEF3C7;
  color: #92400E;
}

.badge-error {
  background-color: #FEE2E2;
  color: #991B1B;
}
```

## Coding Conventions

### File Naming
- **Components**: PascalCase (e.g., `MenuItem.tsx`, `CartSummary.tsx`)
- **Utilities**: camelCase (e.g., `formatPrice.ts`, `validateOrder.ts`)
- **Pages**: kebab-case or Next.js conventions (e.g., `menu/[id].tsx`)

### Component Structure
```tsx
// 1. Imports
import { useState } from 'react';
import { MenuItem } from '@/types/menu';

// 2. Type definitions
interface MenuItemCardProps {
  item: MenuItem;
  onAddToCart: (item: MenuItem) => void;
}

// 3. Component
export function MenuItemCard({ item, onAddToCart }: MenuItemCardProps) {
  // 3a. Hooks
  const [quantity, setQuantity] = useState(1);
  
  // 3b. Event handlers
  const handleAddToCart = () => {
    onAddToCart({ ...item, quantity });
  };
  
  // 3c. Render
  return (
    <div className="card">
      {/* Component JSX */}
    </div>
  );
}
```

### TypeScript Conventions
- Always define prop types using interfaces
- Use type inference where possible
- Avoid `any` type; use `unknown` if type is truly unknown
- Export types from dedicated type files

### CSS Conventions
- Use CSS modules or Tailwind CSS
- Follow BEM naming if using plain CSS
- Keep specificity low
- Use CSS variables for theming

### State Management
- Use React Context for global state (cart, user)
- Use local state for component-specific data
- Consider Zustand for complex state management

## Accessibility

### ARIA Labels
```tsx
<button aria-label="Add Margherita Pizza to cart">
  Add to Cart
</button>
```

### Keyboard Navigation
- All interactive elements must be keyboard accessible
- Use proper focus indicators
- Implement logical tab order

### Color Contrast
- Maintain WCAG AA compliance (4.5:1 for normal text)
- Don't rely solely on color to convey information

## Responsive Design

### Breakpoints
```css
--screen-sm: 640px;   /* Mobile */
--screen-md: 768px;   /* Tablet */
--screen-lg: 1024px;  /* Desktop */
--screen-xl: 1280px;  /* Large desktop */
```

### Mobile-First Approach
```css
/* Mobile styles (default) */
.menu-grid {
  grid-template-columns: 1fr;
}

/* Tablet and up */
@media (min-width: 768px) {
  .menu-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* Desktop and up */
@media (min-width: 1024px) {
  .menu-grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

## Performance Guidelines

1. **Image Optimization**: Use Next.js Image component with proper sizing
2. **Code Splitting**: Lazy load components not needed on initial render
3. **Memoization**: Use `React.memo`, `useMemo`, `useCallback` appropriately
4. **Bundle Size**: Keep bundle size under 200KB (gzipped)

## Testing Conventions

- Write unit tests for utilities and helpers
- Write integration tests for server actions
- Write E2E tests for critical user flows (ordering process)
- Maintain >80% code coverage