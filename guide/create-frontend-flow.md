---
title: Create Frontend Flow
category: guide
tags:
  - frontend-flow
  - flow-structure
  - step-json
  - flow-manifest
  - solidjs
  - wizards
  - architecture
description: >-
  Create a new frontend flow in @nori/app with steps/ folder, step JSONs,
  components, and CLAUDE.md
when: >-
  When creating a new frontend flow, setting up flow.json, defining steps, or
  building wizard/dialog components in the app package.
rules:
  - packages/app/src/features/**/*/flow.json
  - packages/app/src/features/**/*/steps/*.json
  - packages/app/src/features/**/*Wizard.tsx
  - packages/app/src/features/**/validate-*.ts
  - packages/app/src/features/**/CLAUDE.md
required_knowledge: []
created: '2026-03-28T13:32:27.668Z'
updated: '2026-03-28T13:32:27.668Z'
---
# Create Frontend Flow

When asked to create a new frontend flow, follow this exact structure.

## Location

`packages/app/src/features/{domain}/{flow-name}/`

## Required files

```
{flow-name}/
  flow.json                  (flow manifest — describes the flow as a whole)
  steps/
    01-{step-name}.json
    02-{step-name}.json
    ...
  {FlowName}Wizard.tsx      (orchestrator component, or Panel/Dialog)
  {StepComponent}.tsx        (one per ui_action step)
  validate-{name}.ts         (one per validation step)
  CLAUDE.md                  (flow documentation)
```

## flow.json manifest (frontend)

Every frontend flow MUST have a `flow.json` at its root with these fields:

```json
{
  "id": "{flow-name}",
  "name": "{Flow Name}",
  "domain": "{domain}",
  "package": "app",
  "description": "Purpose-driven description explaining WHY the flow exists",
  "trigger": { "type": "user_action" },
  "contract": {
    "endpoint": "POST /api/{resource}",
    "request": "@nori/shared:{RequestType}",
    "response": "@nori/shared:{ResponseType}",
    "sse_events": ["{domain}:{flow}:started", "{domain}:{flow}:completed"]
  },
  "steps": [
    { "step": "01-{step-name}", "type": "ui_action", "next": { "on_save": "02-{step-name}", "on_cancel": "close_dialog" } },
    { "step": "02-{step-name}", "type": "api_call", "next": { "on_success": "03-{step-name}", "on_error": "01-{step-name}" } }
  ],
  "error_strategy": { "default": "abort" }
}
```

- Use `contract` (singular) when the flow uses one API endpoint
- Use `contracts` (array) when the flow uses multiple endpoints
- Contract references use `@nori/shared:{TypeName}` format — NEVER reference backend internal paths
- `steps[].next` values must be step IDs, `"navigate:{path}"`, `"close_dialog"`, `"navigate_back"`, or `null`
- For inline UI states that don't map to steps, use `"inline_state:{description}"`

## Step JSON types

### ui_action — Renders a SolidJS component

```json
{
  "type": "ui_action",
  "what": "Show vault registration form",
  "why": "Collect git URL, vault name, and branch from user",
  "where": {
    "component": "features/{domain}/{flow-name}/{FlowName}Wizard.tsx",
    "step_component": "features/{domain}/{flow-name}/{StepComponent}.tsx"
  },
  "ui": {
    "renders": "form|display|editor|form+editor|dialog",
    "fields": ["field1", "field2"],
    "validation_schema": "@nori/shared:{schemaName}"
  },
  "transitions": {
    "on_valid": "02-next-step",
    "on_cancel": "close_dialog"
  },
  "error_handling": [],
  "decisions": []
}
```

### validation — Runs a client-side validation function

```json
{
  "type": "validation",
  "what": "Validate git URL format",
  "why": "Reject invalid URLs before calling backend",
  "where": {
    "implementation": "features/{domain}/{flow-name}/validate-{name}.ts"
  },
  "transitions": {
    "on_valid": "03-next-step",
    "on_invalid": "01-previous-step"
  },
  "error_handling": [],
  "decisions": []
}
```

### api_call — Bridge to backend flow (THE CONNECTION POINT)

```json
{
  "type": "api_call",
  "what": "Submit vault registration to backend",
  "why": "Server validates, clones, persists, and indexes",
  "where": {
    "component": "features/{domain}/{flow-name}/{FlowName}Wizard.tsx"
  },
  "contract": {
    "endpoint": "POST /api/{resource}",
    "request_type": "@nori/shared:{RequestType}",
    "response_type": "@nori/shared:{ResponseType}",
    "error_type": "@nori/shared:ApiError",
    "sse_events": [
      "{domain}:{flow}:started",
      "{domain}:{flow}:step-name",
      "{domain}:{flow}:completed",
      "{domain}:{flow}:error"
    ]
  },
  "backend_flow": "core/features/{domain}/{backend-flow-name}",
  "transitions": {
    "on_success": "04-show-result",
    "on_error": "01-show-form"
  },
  "error_handling": [
    {
      "scenario": "Network error",
      "action": "Show error toast, stay on current step",
      "severity": "error"
    }
  ],
  "decisions": []
}
```

## CLAUDE.md format

```markdown
# {Flow Name} Wizard|Panel|Dialog

{One-line description}

**Backend flow**: `core/features/{domain}/{backend-flow-name}`
**Contract**: `@nori/shared/contracts/{domain}.contract.ts`

## Steps

1. **{Step name}** — {Description} -> [steps/01-{step-name}.json](steps/01-{step-name}.json)
```

## Transition target types

Transition values can be:

| Pattern | Example | Meaning |
|---------|---------|---------|
| Step reference | `"02-show-form"` | Go to another step in the same flow |
| Navigation | `"navigate:/knowledge/{entry_id}"` | Client-side route navigation with param interpolation |
| Navigate back | `"navigate_back"` | Go to previous page in browser history |
| Close dialog | `"close_dialog"` | Close the current dialog/wizard |

Always use a **step reference** or **navigate_back** for `on_error` transitions — never use undefined targets.

## Skill workflow order

Use skills in this order when building a new feature:

1. **create-contract** — Define types, schemas, SSE events in `@nori/shared` first
2. **create-be-flow** — Build backend flow in `@nori/core`
3. **create-fe-flow** — Build frontend flow in `@nori/app`
4. **create-step** — Add individual steps to existing flows as needed

## Feature names MUST match across packages

Frontend: `app/features/vault/vault-registration/`
Backend:  `core/features/vault/vault-registration/`
Contract: `shared/contracts/vault.contract.ts`

## Checklist

- [ ] steps/ folder with numbered JSON files
- [ ] Orchestrator component (Wizard/Panel/Dialog)
- [ ] One .tsx per ui_action step
- [ ] One validate-*.ts per validation step
- [ ] CLAUDE.md with backend flow reference and contract reference
- [ ] flow.json with id, name, domain, package, description, trigger, contract(s), steps, error_strategy
- [ ] api_call step references the contract from @nori/shared
- [ ] Feature domain CLAUDE.md updated to list the new flow
