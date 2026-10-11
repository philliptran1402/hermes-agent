# 07 — Evidence-based code review

This is an architecture review of inspected slices, not a bug list for the entire repository. Severity describes potential impact; it does not mean a production incident was observed.

## Strengths

- **Clear tool boundary:** `ToolEntry` packages schema, handler, availability check, and async metadata ([`tools/registry.py`](../../tools/registry.py#L181-L197)); `handle_function_call` is a shared dispatch seam ([`model_tools.py`](../../model_tools.py#L872-L930)). This provides a place for guards, middleware, and tests.
- **Authorization before session/agent work:** admission checks route rejection, ignored channels, and user authorization before the session path ([`gateway/run_inbound.py`](../../gateway/run_inbound.py#L185-L261)).
- **Durability intent:** cron writes use staging, atomic replacement, locking, fsync, and disk merge [jobs.py L1554-L1608](../../cron/jobs.py#L1554-L1608); the SQLite schema includes lease/lock tables.
- **Extensibility contracts:** `MemoryProvider` defines lifecycle hooks, and the plugin loader explicitly coordinates dynamic imports.

## Findings

### R1 — `AIAgent.__init__` accepts a broad set of configuration

- **Severity:** Medium · **Confidence:** High for the observation; medium for impact.
- **Evidence:** `AIAgent.__init__` accepts provider/routing/session/callback/budget/context parameters in [`run_agent.py`](../../run_agent.py#L236-L280), then builds `init_kwargs` from `locals()` [L280-L285](../../run_agent.py#L280-L285).
- **Scenario:** A new option may need wiring across surfaces or may be omitted/misnamed at a callsite.
- **Impact:** More coupling and harder-to-understand construction/testing.
- **Recommendation/trade-off:** Gradually group options into subsystem dataclasses while retaining a compatibility adapter. This requires callsite migration and creates temporary duplication.

### R2 — `gateway.run_inbound::_handle_message` coordinates many decisions

- **Severity:** Medium · **Confidence:** Medium.
- **Evidence:** The function handles admission, metrics, emergency stop, pending replies, stale-agent eviction, busy sessions, slash commands, draining, and active-session claim in [`gateway/run_inbound.py`](../../gateway/run_inbound.py#L1285-L1345).
- **Scenario:** A change in guard order can let a rejected sender reach session work or introduce a race before the claim.
- **Impact:** Ordering is an implicit contract; review and testing are sensitive to platform/profile combinations.
- **Recommendation/trade-off:** Extract stages only with behavior tests that preserve ordering. This adds indirection and requires context/instrumentation to flow through.

### R3 — Tool selection includes process-global state

- **Severity:** Medium · **Confidence:** Medium; confirm the current read/write paths before changing it.
- **Evidence:** The `handle_function_call` docstring says `enabled_tools` defaults to process-global `_last_resolved_tool_names` ([`model_tools.py`](../../model_tools.py#L872-L887)); `agent/AGENTS.md` documents save/restore around subagent execution.
- **Scenario:** Overlapping tasks or profiles may read the wrong selection if the scope is not preserved.
- **Impact:** A session could receive a different tool surface than intended.
- **Recommendation/trade-off:** Pass tool selection explicitly through execution context/contextvars. This increases state passing and requires compatibility with existing callers.

### R4 — Tool execution exposes security-sensitive capabilities

- **Severity:** High if guards/backend configuration are wrong · **Confidence:** High that the capability exists; low that a vulnerability exists.
- **Evidence:** `terminal_tool` runs shell commands through local/container/remote backends [L1-L18](../../tools/terminal_tool.py#L1-L18); `handle_function_call` includes middleware/guard dispatch [model_tools.py L872-L930](../../model_tools.py#L872-L930).
- **Scenario:** Untrusted model/tool input reaches a terminal backend.
- **Impact:** Commands may read/write/delete data or execute with the backend's permissions.
- **Recommendation/trade-off:** Review each backend's threat boundary, preserve approval/allowlist contracts, and add integration tests using a temporary workspace. No bypass or vulnerability is confirmed by the inspected code; this is a review priority, not a vulnerability claim.

### R5 — Multiple runtime surfaces create a broad verification matrix

- **Severity:** Low · **Confidence:** High about scope; medium about risk.
- **Evidence:** Python scripts, TypeScript UIs, adapters, and scheduler are separate surfaces; three CLI entry points are declared [pyproject.toml L557-L560](../../pyproject.toml#L557-L560), with a separate gateway flow.
- **Scenario:** A shared-state/protocol change is checked only through one surface.
- **Impact:** Regression risk grows with adapters/clients, though no specific CI gap was established.
- **Recommendation/trade-off:** Add contract tests at `AIAgent`, registry, session RPC, and adapter boundaries. This increases test runtime and harness maintenance.

### R6 — Job and session stores have different durability models

- **Severity:** Low · **Confidence:** High.
- **Evidence:** Session state uses SQLite schema; cron job writes use staged file replacement in [`cron/jobs.py`](../../cron/jobs.py#L1554-L1608).
- **Scenario:** A reader assumes job definition, session transcript, and delivery outcome share one transaction.
- **Impact:** Recovery/consistency must be handled by orchestration rather than a single cross-store transaction.
- **Recommendation/trade-off:** Document invariants and reconciliation without merging stores absent a concrete need. Reconciliation adds operational logic.

Dependency cycles, coverage, credential leaks, authorization bypasses, reproducible races, and concrete bugs were not verified. Module size alone is not treated as a defect.
