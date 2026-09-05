# knowledge/

Reusable notes from **completed** SDD tasks. Process: see the root [README](../README.md) (collect during development, curate after).

Source code remains the source of truth. Verify material claims against code before reuse.

| Directory | For |
| --- | --- |
| `domains/` | Bounded business context (e.g. `domains/<context>/` when that area has recurred) |
| `integrations/` | External systems / APIs |
| `services/` | Per-service notes, keyed to `id` in `config/repositories.yaml` |
| `architecture/` | Cross-cutting facts that survived verification |

These directories are empty until a real task produces a note. Do not invent coverage.

Include documentation references (Nuclino, Confluence, vendor docs) with what the doc is, which topic it covers, why it helps, and last-verified date when known.

Record curated paths on the task’s `task-state.yaml` under `knowledge_updates`.
