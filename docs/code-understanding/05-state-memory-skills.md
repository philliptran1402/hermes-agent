# 05 — State, memory, and skills

## Session storage

`hermes_state.py::SessionDB` is a facade; schema responsibilities are split among sibling modules. `hermes_state_common.py` declares `schema_version`, `system_prompts`, `sessions`, `messages`, usage/meta, and gateway state tables [L373-L449](../../hermes_state_common.py#L373-L449). `hermes_state_messages.py` owns message serialization/append; `hermes_state_search.py` provides message/session search; `hermes_state_sessions.py` handles session metadata and lifecycle. Search may use SQLite FTS/index projections depending on runtime; query plans and performance were not measured.

The schema establishes SQLite as a local relational store. Cron uses a separate serialized file store with atomic replacement rather than the session DB transaction. Cron writes use locks, staging, verification/merge, and atomic replacement in [`cron/jobs.py`](../../cron/jobs.py#L1554-L1608).

## Memory, profiles, skills, and context

| Concept | Role | Storage/lifecycle |
|---|---|---|
| Conversation/session | Transcript and conversation metadata | SQLite through `SessionDB`; session history and search |
| Memory provider | Recall and cross-turn/session memory; may expose tools | Plugin implements `MemoryProvider`; `initialize`, `prefetch`, `sync_turn`, `shutdown` [L84-L150](../../agent/memory_provider.py#L84-L150) |
| User profile | Profile-specific runtime home and configuration | Profile-aware `HERMES_HOME`; exact file layout depends on loader |
| Skill | Task instructions in `SKILL.md`, which can be loaded/installed/archived | Built-ins in `skills/`; optional skills installed separately; user-created skills under the home directory |
| Context | Content sent to the model for a turn: prompt, history, recall, files, tool results | Built per turn/session; not all context is durable |

“Memory” is not the entire transcript; a “skill” is not fine-tuning. Context is the representation sent to the model, while storage is the durable source.

## Concurrency, isolation, and recovery

Evidence includes a lock around cron file writes and session-turn leases/compression locks/generation tables in the SQLite schema. The gateway claims an active session before awaiting and uses durable markers, but the complete locking/isolation semantics were not inspected, so serializability is not claimed. Profile scope must be bound in background callbacks/threads; `os.environ` and globals do not become per-profile automatically. Migrations/recovery logic exists in `hermes_state_schema.py`; migrations and crash recovery were not executed or validated.

## What “self-improving” means in the inspected code

Verified mechanisms include creating/editing skills, tracking skill usage, and curator logic that reviews, marks stale, or archives skills. Memory providers also expose recall/sync hooks. These mechanisms persist and curate external knowledge artifacts. The inspected flows provide no evidence that model weights are updated or that the model is fine-tuned; “self-improving” here describes improvements to context/skills, not model training.
