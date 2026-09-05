---
schema_version: 1
jira_key: null
title: null
status: draft                     # draft | in_review | approved | changes_requested | rejected
spec_status_required: approved
---

# Plan: `<JIRA-KEY>`

HOW only. Must implement the approved specification. Do not change WHAT/WHY here.

## Gate

- Specification path: `specs/<JIRA-KEY>/spec.md`
- Specification must be approved in `task-state.yaml` before this plan is used for implementation.

## Approach

<!-- Technical strategy. No new business behaviour. -->

## Per-repository changes

<!-- One subsection per affected repository. Repository `id` must match config/repositories.yaml when that file is populated. -->

### `<repository-id>`

- **Path:** <!-- from config/repositories.yaml, or TODO -->
- **Change:**
- **Code evidence inspected:** <!-- repo-relative paths -->
- **Branch:** null                # until config/workflow.yaml defines a convention

## Sequencing

<!-- Order of work across repositories, including contract/publish/consume order if relevant. -->

## Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| | | |

## Test plan

<!-- How we will know the acceptance criteria are met. -->

## Knowledge updates

<!-- Paths under knowledge/ to add or verify after implementation. Leave empty if none. -->

```yaml
- path: null
  action: null                    # add | update | verify
  notes: null
```

## Open Questions

<!-- Implementation unknowns. Behaviour unknowns belong in spec.md, not here. -->

```yaml
- id: PQ-1
  question: ""
  blocking: true
  evidence_tried: []
  resolution: null
```
