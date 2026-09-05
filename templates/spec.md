---
schema_version: 1
jira_key: null
title: null
status: draft                     # draft | in_review | approved | changes_requested | rejected
---

# Specification: `<JIRA-KEY>`

WHAT and WHY only. Do not describe HOW. Fill from evidence. Do not invent behaviour.

## Summary

<!-- One short paragraph. -->

## Background

<!-- Why this change is needed, using Jira/Confluence evidence. -->

## Current behaviour

<!-- What the system does today. Source of truth: source code. -->

### Evidence

| Claim | Kind | Reference |
| --- | --- | --- |
| | jira \| confluence \| code | <!-- issue key, page URL/id, or repo-relative path --> |

## Desired behaviour

<!-- Observable outcomes. If unknown, add an Open Question instead. -->

## Out of scope

<!-- Explicit non-goals from evidence, or Open Questions if scope is unclear. -->

## Impact

### Affected repositories

<!-- Use ids from config/repositories.yaml. If the registry is empty, list unknown and add Open Questions. Do not invent repository names. -->

| Repository id | Impact | Notes |
| --- | --- | --- |
| | none \| read \| change | |

### Cross-repository concerns

<!-- Contracts, shared data, ordering, or unknown. -->

## Acceptance criteria

<!-- Given / When / Then. Each criterion must be testable without implementation detail. -->

1. **Given** …, **When** …, **Then** …
2. **Given** …, **When** …, **Then** …

## Open Questions

<!-- Required when evidence is missing or conflicting. Blocking questions prevent approval. -->

```yaml
- id: OQ-1
  question: ""
  blocking: true
  evidence_tried: []              # jira | confluence | code | knowledge
  resolution: null
```

## Knowledge notes

<!-- Optional. Existing knowledge/ paths consulted and whether they were verified against code. -->
