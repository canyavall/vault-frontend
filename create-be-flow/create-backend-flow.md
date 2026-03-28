---
title: Create Backend Flow
category: create-be-flow
tags:
  - backend-flow
  - flow-structure
  - orchestration
  - step-definition
  - flow-manifest
  - action-steps
  - error-handling
  - sse-events
description: >-
  Create a new backend flow in @nori/core with steps/ folder, step JSONs,
  actions/ folder, orchestrator, and CLAUDE.md
when: >-
  When creating a new backend flow in packages/core/src/features, setting up
  flow architecture, defining step sequences, or implementing flow orchestration
  patterns.
rules:
  - packages/core/src/features/**/flow.json
  - packages/core/src/features/**/steps/*.json
  - packages/core/src/features/**/actions/*.ts
  - packages/core/src/features/**/*.ts
  - packages/core/src/features/**/CLAUDE.md
required_knowledge: []
created: '2026-03-28T13:32:27.668Z'
updated: '2026-03-28T13:32:27.668Z'
---
# Create Backend Flow

When asked to create a new backend flow, follow this exact structure.

## Location

`packages/core/src/features/{domain}/{flow-name}/`

## Required files

```
{flow-name}/
  flow.json               (flow manifest — describes the flow as a whole)
  steps/
    01-{step-name}.json
    02-{step-name}.json
    ...
  actions/
    {step-name}.ts        (one per action step)
  {flow-name}.ts          (orchestrator)
  CLAUDE.md               (flow documentation)
```

## flow.json manifest (backend)

Every backend flow MUST have a `flow.json` at its root with these fields:

```json
{
  "id": "{flow-name}",
  "name": "{Flow Name}",
  "domain": "{domain}",
  "package": "core",
  "description": "Purpose-driven description explaining WHY the flow exists",
  "trigger": {
    "type": "api_call",
    "endpoint": "POST /api/{resource}"
  },
  "input": "{ field: type, ... }",
  "output": "{ field: type, ... }",
  "steps": [
    { "step": "01-{step-name}", "type": "action", "next": "02-{step-name}" },
    { "step": "02-{step-name}", "type": "action", "next": null }
  ],
  "error_strategy": {
    "default": "abort",
    "retryable_steps": ["01-{step-name}"]
  },
  "sse_events": [
    "{domain}:{flow}:started",
    "{domain}:{flow}:{step-name}",
    "{domain}:{flow}:completed"
  ]
}
```

- `steps[].next` can be: a step ID (linear), an object with condition keys (branching), `"abort"`, or `null` (end)
- `input`/`output` should reference actual TypeScript type shapes
- `trigger.endpoint` is the HTTP route that triggers this flow
- `sse_events` use colon-separated naming: `{domain}:{flow}:{step}`
- Add `depends_on` array if this flow calls other flows

## Step JSON format (backend)

Every step JSON MUST have ALL these fields:

```json
{
  "type": "action|flow_call",
  "what": "Human-readable description of what this step does",
  "why": "Business reason this step exists",
  "where": {
    "entry_point": "features/{domain}/{flow-name}/{flow-name}.ts",
    "implementation": "features/{domain}/{flow-name}/actions/{step-name}.ts"
  },
  "output": {
    "console": false,
    "file": false
  },
  "success_handling": {
    "criteria": "What constitutes success",
    "event": {
      "action": "{domain}_{flow}_{step}",
      "status": "success",
      "message": "Human-readable success message",
      "data_fields": ["relevant", "data", "fields"]
    }
  },
  "error_handling": [
    {
      "scenario": "What can go wrong",
      "criteria": "How to detect this error",
      "action": "What to do about it",
      "severity": "fatal|error|warning|info",
      "recoverable": true,
      "event": {
        "action": "{domain}_{flow}_{step}",
        "status": "error",
        "message": "Error: {details}",
        "data_fields": ["error"]
      }
    }
  ],
  "decisions": [
    {
      "date": "YYYY-MM-DD",
      "reason": "Short reason",
      "rationale": "Detailed explanation of why this decision was made"
    }
  ],
  "calls": [
    "features/{domain}/{flow-name}/actions/{step-name}"
  ]
}
```

For `flow_call` type steps, add `flow_id` field and point `where.implementation` to the target flow's orchestrator.

## Orchestrator pattern

```typescript
import type { FlowEmitter } from '../../shared/utils/flow-emitter'

export async function run{FlowName}(input: {Input}, emitter?: FlowEmitter) {
  emitter?.emit('flow:started', { flow: '{flow-name}' })

  // Step 01
  emitter?.emit('step:started', { step: '01-{step-name}' })
  const result01 = await {stepAction}(input)
  emitter?.emit('step:completed', { step: '01-{step-name}', result: result01 })

  // ... more steps

  emitter?.emit('flow:completed', { flow: '{flow-name}' })
  return result
}
```

## CLAUDE.md format

```markdown
# {Flow Name} Flow

{One-line description}

## Steps

1. **{Step name}** — {Description} -> [steps/01-{step-name}.json](steps/01-{step-name}.json)
2. **{Step name}** — {Description} -> [steps/02-{step-name}.json](steps/02-{step-name}.json)
```

## Event naming conventions

Two naming conventions are used — one for step action IDs, one for SSE wire events:

| Context | Format | Example |
|---------|--------|---------|
| Step JSON `action` field | `{domain}_{flow}_{step}` (underscores) | `vault_registration_test_git_access` |
| SSE events (contract/wire) | `{domain}:{flow}:{step}` (colons) | `vault:registration:testing-access` |

The **server layer** maps step action IDs to SSE event names. Backend flows emit using the underscore format via FlowEmitter; the server translates to colon format before sending to the client.

## Skill workflow order

Use skills in this order when building a new feature:

1. **create-contract** — Define types, schemas, SSE events in `@nori/shared` first
2. **create-be-flow** — Build backend flow in `@nori/core` (references contract types)
3. **create-fe-flow** — Build frontend flow in `@nori/app` (references contract for api_call steps)
4. **create-step** — Add individual steps to existing flows as needed

## Checklist

- [ ] steps/ folder with numbered JSON files
- [ ] actions/ folder with one .ts per action step
- [ ] Orchestrator file at flow root
- [ ] CLAUDE.md with step listing and links
- [ ] flow.json with id, name, domain, package, description, trigger, input, output, steps, error_strategy, sse_events
- [ ] All step JSONs have ALL required fields (type, what, why, where, output, success_handling, error_handling, decisions, calls)
- [ ] Feature domain CLAUDE.md updated to list the new flow
