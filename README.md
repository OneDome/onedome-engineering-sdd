# onedome-engineering-sdd

Central repository for SDD artifacts, Cursor agents, governance, and incrementally grown engineering knowledge.

**SDD is backend-only.** Specs, plans, and later implementation cover server-side services. Frontend/UI is out of scope.

Application repositories stay separate. Open this repository together with them in one Cursor workspace. **Jira** is the source of task requirements. **Confluence** is supporting documentation. **Source code** is the source of truth for current behaviour.

AI generates specifications. A human reviewer with sufficient product/domain knowledge explicitly approves them before planning or development. Approval does not depend on organisational role.

## Layout

```text
.cursor/
  agents/
    spec-agent.md          # drafts spec.md (WHAT / WHY)
    plan-agent.md          # drafts plan.md (HOW), only after spec approval
  commands/
    sdd-pr.md              # /sdd-pr — GitHub PR so Jira shows the spec
    start-development.md   # /start-development — propose DEV items; branches after approval
    finish-development.md  # /finish-development — close implementation when all DEV accepted
  skills/
    sdd-pr/                # same workflow when asked in chat
    sdd-development/       # start, continue, accept subtask, finish
  rules/
    sdd-governance.mdc     # project policy; always applied
specs/                     # one directory per Jira key (created when a task starts)
knowledge/                 # optional reusable notes; grown after tasks, never invented
  architecture/
  domains/
  integrations/
  services/
config/
  repositories.yaml        # workspace + application repo registry
  workflow.yaml            # environment mapping and lifecycle config (not policy)
templates/
  spec.md
  plan.md
  task-state.yaml          # canonical machine-readable task record
```

## What each file is for

| Path | Responsibility |
| --- | --- |
| `.cursor/agents/spec-agent.md` | Cursor agent: gather Jira, Confluence, and code evidence; write `spec.md`; never plan or implement. |
| `.cursor/agents/plan-agent.md` | Cursor agent: write `plan.md` only when the spec is approved; never implement. |
| `.cursor/commands/sdd-pr.md` | Slash command: open an SDD PR; Jira key in title/body. |
| `.cursor/commands/start-development.md` | Slash command: propose DEV breakdown; after approval, create app branches. |
| `.cursor/commands/finish-development.md` | Slash command: mark implementation complete when every DEV is accepted. |
| `.cursor/skills/sdd-pr/SKILL.md` | How to open that PR (gh). |
| `.cursor/skills/sdd-development/SKILL.md` | Developer loop: breakdown, `continue`, subtask accept, finish. |
| `.cursor/rules/sdd-governance.mdc` | Binding SDD policy for every session. |
| `config/repositories.yaml` | Known repositories. Empty until real ids/paths are filled in. |
| `config/workflow.yaml` | Jira/Confluence/git mapping, lifecycle enum, Spec Kit status, automation flags. |
| `templates/spec.md` | Human-readable WHAT/WHY template with YAML frontmatter. |
| `templates/plan.md` | Human-readable HOW template. |
| `templates/task-state.yaml` | Schema for per-task lifecycle state consumed by future automation. |
| `specs/<JIRA-KEY>/` | Working copy of spec, plan, and task state for one ticket. |
| `knowledge/` | Verified leftovers from completed work. Not required before starting SDD. |

Policy lives in `.cursor/rules/`. Environment values live in `config/`. Specs do not contain implementation. Plans do not change approved behaviour.

## Task folder convention

```text
specs/<JIRA-KEY>/
  spec.md
  plan.md              # after specification approval
  task-state.yaml
```

`<JIRA-KEY>` is the real Jira issue key. Do not create sample folders.

## How to use (Phase 1, manual)

1. Invoke **spec-agent** with a real Jira key.
2. A **human reviewer** reviews `spec.md` and approves **in chat** (`затверджую` / `I approve this spec`). The agent records that in `task-state.yaml`. Do not treat "looks good", "ок", or similar as approval. The reviewer's job title is irrelevant.
3. Invoke **plan-agent** only after `specification.status` is `approved` and `specification.approval.status` is `approved`.
4. Approve the plan in chat (`I approve this plan`).
5. Invoke **`/sdd-pr`** (or ask to open the SDD PR). That creates a GitHub PR in this repo whose title/body contain the Jira key, so the ticket can show the PR. Do not change Jira status.
6. Invoke **`/start-development <JIRA-KEY>`**. The agent proposes DEV-01…DEV-0n and stops. Approve with `I approve the development breakdown` (creates/reuses application branches named `feature/<JIRA-KEY>`). Then type `continue` to implement the current subtask. Accept with `I accept this subtask`. Repeat until all DEV items are done. Gaps that are not obvious from spec/plan/code become `// TODO(dev): …` for the developer, not invented values.
7. Invoke **`/finish-development <JIRA-KEY>`**. Lifecycle becomes `in_review`. Do not merge, open application PRs, or change Jira status.
8. After the task, add only verified facts to `knowledge/`.

## Spec Kit

`specs/` and the spec/plan templates are shaped so GitHub Spec Kit can be adopted later. Spec Kit is **not** initialized. Do not run `specify init` in Phase 1. See `config/workflow.yaml` `spec_kit`.

## Out of scope for this repository

- Application source (lives in application checkouts; `/start-development` implements there after gates)
- Domain-specific agents
- Invented architecture documentation
- Application PRs, Jira status changes, Confluence writes
- Credentials or secrets
