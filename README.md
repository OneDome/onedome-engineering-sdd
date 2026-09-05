# onedome-engineering-sdd

Central repository for SDD artifacts, Cursor agents, governance, and incrementally grown engineering knowledge.

Application repositories stay separate. Open this repository together with them in one Cursor workspace. **Jira** is the source of task requirements. **Confluence** is supporting documentation. **Source code** is the source of truth for current behaviour.

AI drafts specifications. A human Team Lead approves them before planning or development.

## Layout

```text
.cursor/
  agents/
    spec-agent.md          # drafts spec.md (WHAT / WHY)
    plan-agent.md          # drafts plan.md (HOW), only after spec approval
  rules/
    sdd-governance.mdc     # project policy; always applied
specs/                     # one directory per Jira key (created when a task starts)
knowledge/                 # optional reusable notes; grown after tasks, never invented
  architecture/
  domains/
  integrations/
  services/
config/
  repositories.yaml        # workspace + application repo registry (TODOs until populated)
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
2. Team Lead reviews `spec.md` and records approval in `task-state.yaml`.
3. Invoke **plan-agent** only after `specification.status` is `approved`.
4. Implement in application repositories only after the spec (and, when you require it, the plan) is approved.
5. After the task, add only verified facts to `knowledge/`.

## Spec Kit

`specs/` and the spec/plan templates are shaped so GitHub Spec Kit can be adopted later. Spec Kit is **not** initialized. Do not run `specify init` in Phase 1. See `config/workflow.yaml` `spec_kit`.

## Out of scope for this repository

- Application source code
- Domain-specific agents
- Invented architecture documentation
- Jira, Confluence, or GitHub automation
- Credentials or secrets
