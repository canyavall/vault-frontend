---
title: Nori UI Terminology
category: ui-terminology
tags:
  - ui-terminology
  - layout
  - components
  - naming-conventions
  - floating-windows
  - reference
description: >-
  Official UI terminology for the Nori app. Covers layout containers, the
  floating window system, content component types, and aliases the user may use.
  Reference when the user mentions any UI area by name to ensure you're talking
  about the same thing.
when: >-
  When discussing or implementing UI features, referring to specific UI areas by
  name, working with layout components, or integrating floating windows. Load
  before naming or discussing any UI element.
rules:
  - src/pages/**/*.tsx
  - src/features/**/*.tsx
  - src/components/layout/**/*.tsx
  - src/components/floating-window/**/*.tsx
  - src/stores/floating-window.store.ts
required_knowledge: []
created: '2026-03-28T13:32:27.668Z'
updated: '2026-03-28T13:32:27.668Z'
---
# Nori UI Terminology

Use this as the shared vocabulary for all UI work. When the user refers to a UI area by an unofficial name, **ask for clarification before proceeding**.

---

## Layout Containers

| Term | What it is | File pattern |
|---|---|---|
| **Page** | Route-level container, fills main viewport | `src/pages/{Name}Page.tsx` |
| **Section** | Primary content area rendered inside a Page | `features/{domain}/{flow}/{Name}Section.tsx` |
| **MainAppBar** | The top-most bar, always visible. Contains: project selector (left), flow search Cmd+K (center), Vaults · Roles · Import Project · Error Log icons + Avatar (right). Also called **TopNav** in code. | `components/layout/TopNav/TopNav.tsx` |
| **ProjectAppBar** | The secondary bar shown only when a project is active, below the MainAppBar. Contains: project name + badges, and nav links to Tasks · Flows · Sessions/Chats. Also called **ProjectSubHeader** in code. | `components/layout/ProjectSubHeader/` |

---

## Floating Window System

The right-side overlay system. All pieces together form what we call the **floating window system**.

| Term | What it is | File |
|---|---|---|
| **FloatingWindow** | A single resizable/draggable overlay on the right side of the screen | `components/floating-window/FloatingWindowShell/` |
| **FloatingWindowManager** | Container that renders all currently open FloatingWindows | `components/floating-window/FloatingWindowManager/` |
| **FloatingTabBar** | The vertical strip of icon tabs on the far-right edge; clicking a tab opens or restores a FloatingWindow | `components/floating-window/FloatingTabBar/` |
| **DialogKind** | Enum identifier for which FloatingWindow to open (e.g. `settings`, `vault-list`, `role-list`, `role-create-edit`) | `stores/floating-window.store.ts` |
| **DialogContent** | Internal router that picks which content component to render inside a FloatingWindow based on its `DialogKind` | `components/floating-window/DialogContent/` |

### Known DialogKind values

`settings` · `vault-list` · `role-list` · `role-create-edit` · `role-project-link` · `link-project` · `category-audit` · `validate-category` · `knowledge-generate` · `knowledge-audit` · `category-rewrite` · `repo-extract` · `error-log`

---

## Content Component Types

| Term | What it is | Example |
|---|---|---|
| **Dialog** | Feature content rendered *inside* a FloatingWindow | `VaultRegistrationDialog`, `RoleDialog` |
| **Panel** | Embedded side-by-side content, not a window overlay | `VaultSyncPanel`, `FlowChatPanel`, `KnowledgeDetailPanel` |
| **Accordion** | Collapsible section within a page | `VaultAccordion`, `RoleAccordion` |
| **Card** | Single-item card in a list or grid | `SessionCard`, `NewChatCard` |

> **Note:** `Dialog` as a *feature component* (content inside a FloatingWindow) is different from the `Dialog` UI primitive in `components/ui/Dialog/` (a Kobalte full-screen modal). These are two different things with the same name.

---

## UI Primitives (`components/ui/`)

Generic reusable components: `Dialog` · `Button` · `Badge` · `Alert` · `Avatar` · `FormField` · `CodeEditor` · `Spinner` · `MarkdownContent`

---

## Alias Disambiguation

Users often refer to parts of the UI by informal names. **If the intent is ambiguous, ask before assuming.**

| User may say… | They probably mean… | Clarify if… |
|---|---|---|
| "the top bar", "the header", "the nav bar", "the app bar" | MainAppBar | It could mean ProjectAppBar if a project is active — **clarify** |
| "the main bar", "the main header" | MainAppBar | — |
| "the project bar", "the project header", "the sub-header", "the secondary bar" | ProjectAppBar | — |
| "the bar with tasks/flows/sessions" | ProjectAppBar | — |
| "the sidebar", "the right panel", "the side panel" | FloatingWindow or FloatingTabBar | It's unclear if they mean a specific open window or the tab strip |
| "the tab bar", "the icon strip", "the right tabs" | FloatingTabBar | — |
| "the drawer", "the overlay", "the right-side window" | FloatingWindow | — |
| "the settings", "open settings" | FloatingWindow with `DialogKind = settings` | — |
| "the vault panel", "the vaults window" | FloatingWindow with `DialogKind = vault-list` | — |
| "the roles panel", "the roles window" | FloatingWindow with `DialogKind = role-list` | — |
| "the modal", "the popup", "the dialog" | Could be a UI primitive `Dialog` OR a feature Dialog inside a FloatingWindow | **Always clarify** |
| "open a window" | Could mean FloatingWindow OR native OS window | **Always clarify** |

---

## When to Use FloatingWindow vs Modal Dialog

- **FloatingWindow**: persistent, resizable, user-initiated panels (settings, vault list, role management, tools). Opened via the FloatingTabBar or TopNav buttons. Multiple can be open at once.
- **Modal Dialog** (`components/ui/Dialog`): blocking confirmation prompts, short forms, destructive action confirmations. One at a time, dismisses on confirm/cancel.
