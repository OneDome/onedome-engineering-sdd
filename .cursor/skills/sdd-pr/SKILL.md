---
name: sdd-pr
description: >-
  Opens a GitHub pull request in onedome-engineering-sdd so Jira picks up
  the spec/plan. Use when the user runs /sdd-pr, asks to attach SDD artifacts
  to a Jira ticket via a PR, or says to publish the spec/plan to GitHub.
---

# SDD PR (link to Jira)

Open one PR in **this** repository (`onedome-engineering-sdd`) only. Jira GitHub integration shows the PR on the issue when the **Jira key is in the PR title and body**.

Do not open PRs in application repos. Do not change Jira status. Do not implement application code.

## When to run

- Human asked `/sdd-pr` or to create the SDD PR.
- Spec is approved in `specs/<JIRA-KEY>/task-state.yaml` (`specification.status` and `specification.approval.status` are `approved`).
- If `plan.md` exists, it must also be approved.

## Steps

Work in the SDD repo root. Follow the user's git/PR safety rules (no force push, no `--no-verify`, no `git config`).

1. Read `specs/<JIRA-KEY>/task-state.yaml`. Refuse if spec (and plan, if present) is not approved.
2. Jira key = `task-state.yaml` `jira.key` (or the key the user named). Title = `jira.title`.
3. Branch: `sdd/<JIRA-KEY>` from current `main` (create if needed). Pattern is `config/workflow.yaml` `git.sdd_branch_pattern`.
4. Stage only:
   - `specs/<JIRA-KEY>/spec.md`
   - `specs/<JIRA-KEY>/plan.md` if it exists
   - `specs/<JIRA-KEY>/task-state.yaml`
   - `specs/<JIRA-KEY>/knowledge/sources.md` if it exists
   - Do not stage `.idea/`, secrets, or unrelated paths unless the user asked to include SDD process files.
5. Commit with a message that starts with the Jira key. Example:

```
DEAL-2097 Add approved sourcing spec and plan
```

6. `git push -u origin HEAD` (needs network).
7. `gh pr create` with:
   - **Title:** `<JIRA-KEY> <jira.title>` (key first so Jira matches)
   - **Body:** must contain the Jira key on its own line and a short summary. Use:

```
<JIRA-KEY>

Approved SDD spec (WHAT/WHY) and plan (HOW) for this ticket.

- Spec: specs/<JIRA-KEY>/spec.md
- Plan: specs/<JIRA-KEY>/plan.md
```

8. Write the PR URL into `task-state.yaml` `specification.pr` (and `planning.pr` if the plan is in this PR): `repository: onedome-engineering-sdd`, `number`, `url`. Keep `sdd.branch` as the branch name.
9. Return the PR URL. Tell the user it should appear on the Jira issue if GitHub for Jira is connected.

## Refuse

- Spec not approved.
- User asked to PR `hub-service` / `integration-service` from this skill.
- Empty commit (no spec files).
