# Reusable engineering knowledge

Add notes here **incrementally** as SDD tasks complete. Coverage is not a prerequisite for using SDD.

These files are supporting evidence, not the source of truth. **Source code is.** Existing notes may be stale; verify material claims against code before using them in a spec or plan.

Do not invent architecture, domains, services, or integrations in order to look complete.

| Directory | Holds |
| --- | --- |
| `architecture/` | Cross-cutting technical facts that survived verification |
| `domains/` | Business-domain notes learned from completed work |
| `integrations/` | External-system integration notes |
| `services/` | Per-service notes, keyed to `id` values in `config/repositories.yaml` |

When a task produces a durable fact, record the path in that task's `task-state.yaml` `knowledge_updates` list.
