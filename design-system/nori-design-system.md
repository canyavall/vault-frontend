---
title: Nori Design System
category: design-system
tags:
  - design-tokens
  - typography
  - color-palette
  - semantic-tokens
  - component-specs
  - theme
  - css-variables
  - primitives
  - dark-mode
  - branding
description: >-
  Nori design tokens (primitives + semantic), typography (Inter + Lato),
  component specs (buttons, cards), and theme rules. Reference when creating or
  editing CSS, components, or theme-related code.
rules: []
required_knowledge: []
created: '2026-03-18T07:57:27.149Z'
updated: '2026-03-18T07:57:27.149Z'
---
# Nori Design System

Complete design token system with primitives, semantic tokens, typography, and component specs.

---

## Tier 1: Primitive Tokens

Raw color values. These are the source of truth — never use directly in components, always reference via semantic tokens.

### Colors

| Token | Light | Dark |
|-------|-------|------|
| `--primitive-blue-500` | `#007FFF` | `#3399FF` |
| `--primitive-blue-200` | `#87CEEB` | `#2D4F5C` |

### Neutrals

| Token | Light | Dark |
|-------|-------|------|
| `--primitive-gray-900` | `#1A1A1A` | `#F5F5F5` |
| `--primitive-gray-700` | `#4A4A4A` | `#B0B0B0` |
| `--primitive-gray-50` | `#F9FAFB` | `#121212` |

### Radius

| Token | Value |
|-------|-------|
| `--radius-md` | `8px` |
| `--radius-sm` | `6px` |

### Shadow

| Token | Light | Dark |
|-------|-------|------|
| `--shadow-soft` | `0 4px 12px rgba(0,0,0,0.05)` | `0 4px 12px rgba(0,0,0,0.3)` |
| `--shadow-btn` | `0 2px 4px rgba(0,127,255,0.2)` | `0 2px 4px rgba(0,127,255,0.2)` |

---

## Tier 2: Semantic Tokens (Purpose-Driven)

These change values based on theme but keep the same name. **Always use semantic tokens in components.**

### Surface & Background

| Token | Light | Dark |
|-------|-------|------|
| `--color-surface-primary` | `#FFFFFF` | `#1A1A1A` |
| `--color-surface-secondary` | `#F3F4F6` | `#262626` |
| `--color-border-subtle` | `rgba(0,0,0,0.1)` | `rgba(255,255,255,0.1)` |

### Typography Colors

| Token | Source |
|-------|--------|
| `--color-text-heading` | `var(--primitive-gray-900)` |
| `--color-text-body` | `var(--primitive-gray-700)` |
| `--color-text-brand` | `var(--primitive-blue-500)` |

### Status Colors (unchanged from current system)

| Token | Light | Dark |
|-------|-------|------|
| `--color-success` | `#16a34a` | `#22c55e` |
| `--color-error` | `#d4183d` | `#f87171` |
| `--color-warning` | `#d97706` | `#fbbf24` |

---

## Typography

### Font Families

| Role | Font | Fallback |
|------|------|----------|
| Headings, UI labels, buttons | **Inter** | `-apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif` |
| Body text, descriptions | **Lato** | `"Segoe UI", Roboto, sans-serif` |
| Code | `ui-monospace` | `monospace` |

### Font Weights

| Usage | Weight |
|-------|--------|
| Headings (`text-heading`) | `700` (Bold) |
| Body (`text-body`) | `400` (Regular) |
| Buttons, labels | `600` (Semi-bold) |

### CSS Variables

```css
--font-heading: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
--font-body: 'Lato', "Segoe UI", Roboto, sans-serif;
--font-mono: ui-monospace, monospace;
```

---

## Component Specs

### Primary Button ("Nori Wrapper" Button)

The primary action button with a stable, professional look.

```
Background:    var(--primitive-blue-500)
Border:        1px solid rgba(255,255,255,0.2)    /* inset look */
Shadow:        0 2px 4px rgba(0, 127, 255, 0.2)
Radius:        6px
Font:          Inter, 600 weight
```

**Interactions:**
- Hover: background shifts to `blue.600` with subtle outer glow
- Active: slightly darker, shadow reduced
- Disabled: opacity 0.4

**Style hook pattern:**
```typescript
// Button.style.ts
const PRIMARY_CLASS = [
  'bg-[var(--primitive-blue-500)]',
  'text-white',
  'border border-white/20',
  'shadow-[0_2px_4px_rgba(0,127,255,0.2)]',
  'rounded-[6px]',
  'font-semibold',
  'hover:bg-[var(--primitive-blue-600)]',
  'hover:shadow-[0_4px_8px_rgba(0,127,255,0.3)]',
  'disabled:opacity-40',
].join(' ');
```

### Info Card ("The Scaffolding")

Content card using a mesh border with left accent.

```
Background:    var(--color-surface-secondary)
Border:        1px solid var(--color-border-subtle)
Left accent:   2px solid var(--primitive-blue-500)   /* "Active Wrapper" indicator */
Header:        Inter Bold, 16px
Content:       Lato Regular, 14px
Radius:        var(--radius-md) (8px)
Shadow:        var(--shadow-soft)
```

**Style hook pattern:**
```typescript
// InfoCard.style.ts
const CARD_CLASS = [
  'bg-[var(--color-surface-secondary)]',
  'border border-[var(--color-border-subtle)]',
  'border-l-2 border-l-[var(--primitive-blue-500)]',
  'rounded-[var(--radius-md)]',
  'shadow-[var(--shadow-soft)]',
].join(' ');

const HEADER_CLASS = 'font-[var(--font-heading)] font-bold text-base text-[var(--color-text-heading)]';
const CONTENT_CLASS = 'font-[var(--font-body)] font-normal text-sm text-[var(--color-text-body)]';
```

---

## Rules

1. **Never use primitive tokens directly in components** — always map through semantic tokens
2. **Font pairing is mandatory**: Inter for headings/UI, Lato for body text
3. **All interactive elements** must use `--radius-sm` (6px) for buttons, `--radius-md` (8px) for cards/containers
4. **Shadows are theme-aware**: lighter in light mode, stronger in dark mode
5. **Brand color** (`--primitive-blue-500`) is reserved for: primary buttons, active indicators, links, and brand accents
6. **Left-accent pattern**: Use 2px left border in `blue.500` to indicate active/selected state on cards
7. **Button hierarchy**: Primary (blue.500 bg), Secondary (surface-secondary bg + border), Ghost (transparent)
8. **Border treatment**: Always use `--color-border-subtle` — never hardcode rgba values in components

---

## Token Mapping Reference

Quick lookup for migrating from old tokens to new:

| Old Token | New Token |
|-----------|-----------|
| `--color-bg` | `--color-surface-primary` |
| `--color-bg-secondary` | `--color-surface-secondary` |
| `--color-bg-tertiary` | `--color-surface-secondary` (consolidate) |
| `--color-text` | `--color-text-heading` |
| `--color-text-muted` | `--color-text-body` |
| `--color-border` | `--color-border-subtle` |
| `--color-accent` | `--color-text-brand` / `--primitive-blue-500` |
| `--color-accent-hover` | `--primitive-blue-200` |
| `--color-btn-add` | `--primitive-blue-500` |
