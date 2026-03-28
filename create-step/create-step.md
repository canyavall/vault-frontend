---
title: Create Step
category: create-step
tags:
  - workflow
  - step-creation
  - json-schema
  - backend-steps
  - frontend-steps
  - implementation-patterns
  - step-numbering
  - action-files
  - component-files
  - validation
description: >-
  Create a new step JSON file and its corresponding action/component for an
  existing flow
when: >-
  When you need to add a new step to an existing flow in either the backend
  (packages/core/) or frontend (packages/app/) codebase. Use this when creating
  step JSON files and their corresponding action/component implementations.
rules:
  - packages/core/**/steps/*.json
  - packages/core/**/actions/*.ts
  - packages/app/**/steps/*.json
  - packages/app/**/*.tsx
  - packages/app/**/validate-*.ts
required_knowledge: []
created: '2026-03-28T13:32:27.668Z'
updated: '2026-03-28T13:32:27.668Z'
---
# Create Step

When asked to add a new step to an existing flow, follow these rules.

## 1. Determine step type by location

- If in `packages/core/` → backend step (type: `action` or `flow_call`)
- If in `packages/app/` → frontend step (type: `ui_action`, `validation`, or `api_call`)

## 2. Find the next step number

Look at existing files in the flow's `steps/` folder. Use the next number in sequence. If the last file is `05-something.json`, the new step is `06-new-step.json`.

## 3. Create the step JSON

Place it in `{flow}/steps/{NN}-{step-name}.json` with ALL required fields for its type.

### Backend step required fields
- type, what, why, where (entry_point + implementation), output, success_handling, error_handling, decisions, calls

### Frontend step required fields
- type, what, why, where (component or implementation), transitions, error_handling, decisions
- For ui_action: add `ui` field (renders, fields, validation_schema)
- For api_call: add `contract` field (endpoint, request_type, response_type, sse_events) and `backend_flow`

## 4. Create the implementation file

### Backend action step
Create `{flow}/actions/{step-name}.ts`:

```typescript
/**
 * Step {NN}: {What}
 *
 * {Why}
 */

export async function {stepName}(input: {InputType}): Promise<{OutputType}> {
  // Implementation
}
```

### Frontend ui_action step
Create `{flow}/{StepComponent}.tsx`:

```tsx
import { Component } from 'solid-js'

interface {StepName}Props {
  onNext: (data: any) => void
  onBack?: () => void
}

const {StepName}: Component<{StepName}Props> = (props) => {
  return (
    // Implementation
  )
}

export default {StepName}
```

### Frontend validation step
Create `{flow}/validate-{name}.ts`:

```typescript
import { {schema} } from '@nori/shared'

export function validate{Name}(data: unknown) {
  return {schema}.safeParse(data)
}
```

## 5. Update the flow CLAUDE.md

Add the new step to the numbered list in the flow's CLAUDE.md file.

## 6. flow.json sync

When creating steps via the Nori API (flow system UI), the `flow.json` manifest's `steps` array is automatically synced. If you create steps manually (outside the API), update `flow.json`'s steps array to include the new step with its `step` ID, `type`, and `next` routing.

## Related skills

- **create-contract** — If the new step is an `api_call`, ensure the contract exists first
- **create-be-flow** / **create-fe-flow** — Use these to create entire flows; use `create-step` only to add steps to existing flows

## Checklist

- [ ] Step JSON in steps/ with correct number
- [ ] Implementation file (action .ts or component .tsx)
- [ ] Flow CLAUDE.md updated with new step
- [ ] All required JSON fields present
- [ ] Backend step action field uses `{domain}_{flow}_{step}` (underscores)
- [ ] SSE events in contracts use `{domain}:{flow}:{step}` (colons)
