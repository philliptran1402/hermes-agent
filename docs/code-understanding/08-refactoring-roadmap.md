# 08 — Refactoring roadmap (proposals only)

This roadmap does not change application code. Priorities start with risk reduction before larger structural changes.

| Priority | Change | Goal/scope | Benefit | Risk and verification |
|---|---|---|---|---|
| Quick win 1 | Document and test the tool-execution contract | Cover schema → authorization/approval → handler → backend, starting with `terminal` | Clarifies the security boundary and surface contract | Tests may overfit implementation; verify registry/guards and a temporary backend |
| Quick win 2 | Add a flow-parity matrix | Test shared core behavior through CLI, gateway, TUI/cron where the same behavior is actually used | Catches adapter differences | Adds suite cost; start with authorization/session persistence where harnesses exist |
| Quick win 3 | Make tool selection explicit in execution context | Gradually replace `_last_resolved_tool_names` reads with scoped parameters/context | Reduces coupling across concurrent requests | Legacy callers need compatibility; verify overlapping sessions/subagents across profiles |
| Architecture step 1 | Group `AIAgent` init options | Introduce `ProviderOptions`, `SessionOptions`, `ToolOptions`, and callback groups; normalize legacy kwargs at the facade | Easier config review/testing and less wiring drift | Many callsites change; preserve behavior contracts and migrate one surface at a time |
| Architecture step 2 | Extract gateway inbound stages | Give admission, routing, command handling, and turn execution explicit inputs/outputs | Makes guard order easier to review/test | Reordering can affect security/concurrency; extract only with trace tests preserving invariants |
| Architecture step 3 | Define cross-store recovery contracts | Jobs, transcripts, execution ledger, and delivery queue | Makes crash recovery easier to reason about | Do not imply a cross-store transaction; identify sources of truth and reconciliation first |

## Rollout principles

1. Add behavior tests against current code to capture behavior that must remain.
2. Add the new seam alongside the old one; migrate one caller/surface at a time.
3. Compare local data/telemetry if available, without changing prompt/cache semantics.
4. Remove the old path only after consumers have migrated and process/profile concurrency is covered.

Large changes should be justified by profiling, bug reports, or demonstrated test gaps. This review did not measure runtime bottlenecks.
