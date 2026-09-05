---
schema_version: 1
jira_key: null
jira_title: null
status: draft
created_at: null
updated_at: null
approval:
  status: pending
  approved_by: null
  approved_at: null
---

# Specification: `<JIRA-KEY>`

DRAFT **backend** business specification. WHAT and WHY only. Fill from evidence. Do not invent behaviour. Do not describe HOW. Do not specify frontend/UI.

The Specification Agent must leave `status: draft` and `approval.status: pending` while drafting. Human Approval is an explicit phrase in chat (`затверджую` / `I approve this spec`); the agent then records it. Job title is irrelevant.

## Distinctions

| Kind | Where it belongs |
| --- | --- |
| Explicitly requested future behaviour | Expected Behaviour, Business Rules, Acceptance Criteria |
| Verified current behaviour | Current Behaviour |
| Assumptions | Assumptions (never copied into requirements) |
| Unknowns | Open Questions |

## Business Context

<!-- Why the change is needed. Cite Jira/Confluence. -->

## Problem

<!-- The concrete problem being solved. -->

## Current Behaviour

<!-- Only behaviour verified from code, Jira, Confluence, or previous approved specs. If a previous spec is used, verify material claims against code. If docs conflict with code, state both; code is implementation truth. -->

## Expected Behaviour

<!-- Requested future **backend** behaviour (APIs, events, persistence, jobs, server-side rules). No screens, CSS, or client app work. No implementation decisions unless they are explicit requirements in evidence. -->

## User / System Flows

<!-- Backend-observable scenarios (who/what calls the service, what is stored, what is returned or emitted). Omit UI walkthroughs. Omit rather than speculate. -->

### Flow 1 — `<short name>`

1. **Given** …
2. **When** …
3. **Then** …

## Business Rules

<!-- Number stably. Prefer fewer high-quality rules. -->

- **BR-001:** …
- **BR-002:** …

## Acceptance Criteria

<!-- Objectively verifiable. No hidden implementation. -->

- **AC-001:** Given …, When …, Then …
- **AC-002:** Given …, When …, Then …

## Edge Cases

<!-- Only from evidence, or point to an Open Question. -->

- …

## Assumptions

<!-- Optional. Must not be treated as requirements. -->

- …

## Dependencies

<!-- Other work, systems, or decisions this change depends on. Unknown → Open Question. -->

- …

## Potentially Affected Systems

<!-- Preliminary. Backend checkouts only as delivery targets. Frontend may be listed as a consumer, not as work this spec will implement. Use ids from config/repositories.yaml when known. Do not invent names. -->

| System / repository id | Why it might be affected | Confidence |
| --- | --- | --- |
| | | low \| medium \| high |

## Out of Scope

<!-- Always include frontend/UI unless the user explicitly overrides this process (they should not). Add other non-goals from evidence. -->

- Frontend / UI (screens, client apps, styling). SDD delivers backend only.
- …

## Open Questions

<!-- Unresolved decisions only. Do not silently resolve. After Human Review answers an item: fold the answer into Expected Behaviour / Business Rules / Acceptance Criteria, then delete that OQ. Do not leave Closed stubs. Do not keep question-and-answer pairs in this section. -->

### OQ-001

- **Question:**
- **Why it matters:**
- **Evidence checked:**
- **What cannot proceed safely without the answer:**

### OQ-002

- **Question:**
- **Why it matters:**
- **Evidence checked:**
- **What cannot proceed safely without the answer:**

## Evidence / References

### Jira

- <!-- key, field, or comment id -->

### Confluence

- <!-- space key, page id, title, URL -->

### Source code

- <!-- repository id + path -->

### Previous specs

- <!-- specs/<JIRA-KEY>/spec.md — note whether verified against code -->

### Engineering knowledge

- <!-- knowledge/ path — note whether verified against code -->
