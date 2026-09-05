---
name: sdd-development
description: >-
  Developer flow for an approved SDD task: /start-development proposes DEV
  subtasks, creates app branches after I approve the development breakdown,
  implements one subtask at a time on continue, /finish-development when all
  DEV items are accepted. Use when the user runs those commands, says continue,
  I approve the development breakdown, or I accept this subtask.
---

# SDD development

Execution of an approved spec + plan. Backend only. No `development.md`. No extra slash commands.

Developer-facing: `/start-development`, `continue`, `/finish-development`.  
Internal chat gates: `I approve the development breakdown`, `I accept this subtask`.

Do not infer those gates from `ok`, `ок`, `lgtm`, `fine`, `добре`.

## Config

- Checkouts: `config/repositories.yaml` (`workspace.root` + `path`)
- Branch name: `config/workflow.yaml` `git.implementation_branch_pattern` (`feature/<JIRA-KEY>`, e.g. `feature/DEAL-2097`)
- Base branch: `git.specification_pr_target_branch` (`main`)
- State: `specs/<JIRA-KEY>/task-state.yaml`

Older `development:` blocks may lack `status`, `decomposition`, `subtasks`. Treat missing `status` as `not_started`, missing `decomposition.status` as `pending`, missing `subtasks` as empty. Fill the template shape when writing.

Never force push, never `git reset --hard`, never delete remote branches, never skip hooks. Do not open application PRs. Do not push unless asked. Do not change Jira status. Do not write Confluence or `knowledge/` during development (candidates only in task-state).

Commit application code **only after** `I accept this subtask` (one commit per listed repo that has changes for that DEV). Do not commit on `continue`. No empty commits. No `--no-verify`.

## `/start-development <JIRA_KEY>`

### Validate (stop and explain if any fail)

1. `specs/<KEY>/` exists.
2. `spec.md` exists; `specification.status` and `specification.approval.status` are `approved`.
3. `plan.md` exists; `planning.status` and `planning.approval.status` are `approved`.
4. Spec/plan Open Questions are empty (`None` or no items).
5. Lifecycle is `plan_approved` or `in_development` (resume).
6. `repositories[]` has at least one `impact: change` whose `id` is in `config/repositories.yaml`.
7. That checkout exists on disk.

### Resume

If `development.decomposition.status` is `approved` and `subtasks` is non-empty:

- Do **not** regenerate the breakdown or recreate branches.
- Report: lifecycle, development.status, each DEV id/title/status, `current_subtask`, repo branches.
- If `development.status` is `in_progress`, tell them to type `continue` (or `/finish-development` if all DEV are `completed`).
- Stop unless they also said `continue` in the same message.

### Propose breakdown (first run only)

Read approved `plan.md` and `spec.md`. Propose **2–7** DEV items by technical cohesion (contract, mapping, integration, validation, API, tests). Not one-per-file, class, method, repo, or AC.

Show in chat (no new file):

```text
DEV-01 — <title>
Repositories: <ids>
Covers: AC-…
Depends on: —
Tests: <one line>
```

Then recommended order. Stop. Do not write subtasks as approved. Do not create branches. Do not edit application code.

Optional: stash a **pending** proposal in `development.subtasks` with `development.status: awaiting_breakdown_approval` and `decomposition.status: pending` so resume can re-show it. If you do, still wait for the approval phrase.

### `I approve the development breakdown`

1. Persist `development.decomposition` (`status: approved`, `approved_by`, `approved_at`, `method: chat`) and the agreed `subtasks` (`status: pending`).
2. Create/reuse branches (below).
3. Set `lifecycle.status: in_development`, `development.status: in_progress`, `development.started_at`, `development.developer` (name they give, else OS username), `current_subtask` = first DEV with empty/completed `depends_on`.
4. Stop. Tell them to type `continue` to implement `current_subtask`.

If they merge/split/reorder in chat, apply that list, then persist.

### Branches (after breakdown approval only)

For each `repositories[]` with `impact: change`:

1. Resolve path: `workspace.root` / `path`.
2. `git status` / `git fetch`.
3. **Unsafe → stop:** missing repo; detached unexpected HEAD; uncommitted changes that are **not** already on the task branch.
4. Branch = `feature/<JIRA-KEY>`. If it exists locally or on origin, checkout/track it. Else `git checkout -b feature/<KEY>` from updated `origin/main` (or `main` / `master` if that is the default).
5. Push `-u` if origin has no such branch. No empty commit.
6. Write `repositories[].branch`.

Idempotent: existing task branch → reuse.

## Implement one subtask (`continue`)

Requires `decomposition.status: approved` and `development.status: in_progress`.

1. Load spec, plan, task-state, `current_subtask`. If null, pick first pending DEV whose `depends_on` are all `completed`.
2. Set that item `status: in_progress` if it was `pending`.
3. Inspect **current** code in listed repos on the task branch. Do not trust stale plan paths blindly.
4. Implement **only** this DEV. No later DEV, no drive-by refactors. If a mapping, field name, or code path is not obvious from spec/plan/workbook/current code: implement the evidenced subset and leave `// TODO(dev): …` for the developer. Do not invent Dataverse names or product behaviour. List TODOs in the summary.
5. Add/update tests for this change. Run the narrowest relevant tests.
6. Show:

```text
## Subtask
DEV-XX — title

## Changes Made
## Files Changed
## Tests
## Test Results
## Acceptance Criteria
## Deviations
## TODO for developer
## Risks / Notes
## Next
```

7. Do **not** mark `completed`. Do **not** commit. Wait for `I accept this subtask`.

### `I accept this subtask`

1. Mark current DEV `completed`.
2. **Commit** that subtask in each `repositories[]` listed on the DEV (`impact: change` checkouts on `feature/<JIRA-KEY>`):
   - `git status` / `git diff` / `git log` (message style).
   - Stage only this DEV’s files. Skip a repo with no changes (no empty commit).
   - Do not include the next DEV’s WIP. Do not commit secrets. Do not skip hooks.
   - Message via HEREDOC, why-focused: `<JIRA-KEY> | <DEV-id>: <short why>` (example: `DEAL-2097 | DEV-02: expand Hub Client prefill from Fact Find`).
   - Report the hash(es). Do not push unless asked. Do not open an application PR.
3. Set `current_subtask` to the next pending DEV whose dependencies are completed, or `null` if none.
4. If more remain, they may type `continue`. If none, tell them `/finish-development`.

### Plan deviation

**Minor** (helper name, file location, equivalent wiring; same behaviour/API/architecture/scope): do it; note under Deviations.

**Not obvious locally** (workbook has a display name but no Dataverse logical name; picklists do not line up; nested SOAP type unclear): **not** a PLAN DEVIATION. Leave `// TODO(dev): …` and continue the rest of the subtask.

**Material** (missing API, wrong service, new repo, compat break, ambiguous WHAT): stop with:

```text
PLAN DEVIATION
Subtask:
Plan expected:
Actual code:
Evidence:
Impact:
Recommended options:
```

Do not silently redesign. Do not decide product behaviour. If WHAT is wrong, say spec may need re-approval.

### Knowledge candidates

Append to `development.knowledge_candidates` only (`type`, `note` or `title`/`url`, `discovered_during`). Do not write `knowledge/`.

## `/finish-development <JIRA_KEY>`

Validate: spec/plan still approved; every DEV `completed`; no unanswered PLAN DEVIATION in this chat that was left open; do not invent “all tests in the monorepo passed” — only that accepted subtasks had their tests run.

Then: `development.status: implementation_complete`, `lifecycle.status: in_review`. Concise implementation summary (DEV list, repos/branches, ACs, candidates).

Do not: Jira Done, merge, application PRs, Confluence, `knowledge/` curation.
