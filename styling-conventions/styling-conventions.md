---
title: Styling Conventions
category: styling-conventions
tags:
  - style-hooks
  - theme-tokens
  - tailwind-css
  - css-custom-properties
  - dynamic-styles
  - component-styling
  - design-tokens
  - style-patterns
description: >-
  Style hook pattern, theme token usage, dynamic styles, and styling
  anti-patterns. Reference when creating or editing .style.ts files.
when: >-
  When creating or editing component styling files (.style.ts), applying theme
  tokens, building dynamic styles based on props, or working with Tailwind CSS
  patterns in components.
rules:
  - '**/*.style.ts'
  - '**/style.ts'
  - src/components/**/*.style.ts
required_knowledge: []
created: '2026-03-28T13:32:27.668Z'
updated: '2026-03-28T13:32:27.668Z'
---
# Styling Conventions

Style hook pattern, theme token usage, and dynamic styling patterns.

---

## MANDATORY: Style Hook Pattern

Every component with non-trivial styles should have a `.style.ts` file:

```
ComponentName/
├── ComponentName.tsx
├── ComponentName.style.ts    # Export useComponentNameStyle hook
├── ComponentName.type.ts
```

```typescript
// ComponentName.style.ts
export const useComponentNameStyle = () => {
  const wrapperStyle = {
    padding: '1rem',
    backgroundColor: 'var(--color-surface)',
  };

  const titleStyle = {
    fontSize: '1.5rem',
    color: 'var(--color-text-primary)',
    marginBottom: '0.5rem',
  };

  return { wrapperStyle, titleStyle };
};
```

---

## Dynamic Styles Based on Props

Pass style-relevant props to the style hook:

```typescript
export const useCardStyle = ({ isActive, variant }: StyleProps) => {
  const cardClass = [
    'card',
    isActive ? 'card--active' : '',
    variant === 'outlined' ? 'card--outlined' : '',
  ].filter(Boolean).join(' ');

  return { cardClass };
};
```

---

## Tailwind CSS Patterns (Nori uses Tailwind v4)

```typescript
// ✅ Use Tailwind utility classes
export const useButtonStyle = ({ isActive }: { isActive: boolean }) => {
  const buttonClass = isActive
    ? 'bg-primary-500 text-white'
    : 'bg-surface text-text-primary';

  return { buttonClass };
};

// ✅ Use CSS custom properties for theme tokens
const wrapperStyle = { padding: 'var(--spacing-4)' };
```

---

## Token Categories

Use design tokens instead of hardcoded values. Nori uses a **two-tier token system** (primitives → semantic). Always use semantic tokens in components; never reference primitives directly.

| Token Type | Example | Notes |
|------------|---------|-------|
| Surface | `var(--color-surface-primary)`, `var(--color-surface-secondary)` | Background colors |
| Text | `var(--color-text-heading)`, `var(--color-text-body)`, `var(--color-text-brand)` | Typography colors |
| Border | `var(--color-border-subtle)` | All borders |
| Status | `var(--color-success)`, `var(--color-error)`, `var(--color-warning)` | Status indicators |
| Radius | `var(--radius-sm)` (6px buttons), `var(--radius-md)` (8px cards) | Border radius |
| Shadow | `var(--shadow-soft)`, `var(--shadow-btn)` | Box shadows |
| Font | `var(--font-heading)` (Inter), `var(--font-body)` (Lato), `var(--font-mono)` | Font families |
| Spacing | Tailwind utilities: `p-4`, `gap-2`, etc. | Use Tailwind for spacing |

---

## Style in Component vs Style Hook

**In component file** — Only for trivial, static, single-value styles:
```tsx
<div class="flex items-center gap-2">
```

**In style hook** — For dynamic styles, theme-dependent values, reused styles:
```tsx
// ComponentName.style.ts
export const useCardStyle = ({ variant }) => ({
  containerClass: variant === 'primary' ? 'bg-primary-500' : 'bg-surface',
});
```

---

## Common Pitfalls

- ❌ Hardcoded colors/spacing — use theme tokens
- ❌ Complex style logic in JSX — move to style hook
- ❌ Duplicate style logic across files — extract to style hook
- ❌ Pixel values without token reference — use `transformPxToRem()` or tokens
- ❌ Inline style objects for dynamic styles — use class names or CSS variables
