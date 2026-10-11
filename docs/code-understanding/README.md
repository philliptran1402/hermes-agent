# Understanding Hermes Agent

## Goal and scope

This guide helps a backend engineer familiar with Node.js navigate Hermes's Python code and multi-surface runtime. It analyzes commit `cdce56faf239eaeebbccf312ed4fa13ec72be256` (reviewed 2026-10-11); the working tree was clean before these notes were created. Source code is the primary evidence; `AGENTS.md` helps route the review and distinguish project intent from implementation. This is a targeted review of key paths, not a file-by-file audit of the repository. No secrets or `.env` values were read, and no agent, API, or external service was started.

## What Hermes does

Hermes is a Python agent runtime that calls models and tools through surfaces such as a CLI and messaging gateway. TUI, Desktop, and Web clients connect through their own backends. The repository also contains scheduling, session storage, plugins, skills, and development tooling. Console scripts are declared in [`pyproject.toml`](../../pyproject.toml#L557-L560); `AIAgent` is the facade in [`run_agent.py`](../../run_agent.py#L214-L219), and the turn entry is [`agent/conversation_loop.py::run_conversation`](../../agent/conversation_loop.py#L1709-L1725).

## Contents

- [01 — Repository map](01-repository-map.md)
- [02 — Architecture](02-architecture.md)
- [03 — Runtime flows](03-runtime-flows.md)
- [04 — Agent, models, and tools](04-agent-model-tools.md)
- [05 — State, memory, and skills](05-state-memory-skills.md)
- [06 — Technology stack and development](06-tech-stack-and-development.md)
- [07 — Code review](07-code-review.md)
- [08 — Refactoring roadmap](08-refactoring-roadmap.md)

## Reading paths

| Time | Read in order |
|---|---|
| 30 minutes | [`hermes_cli/main.py`](../../hermes_cli/main.py#L1-L48) `main` → [`cli.py`](../../cli.py#L894-L910) `HermesCLI` → [`run_agent.py`](../../run_agent.py#L214-L219) `AIAgent` → [`agent/conversation_loop.py`](../../agent/conversation_loop.py#L1709-L1745) `run_conversation` → [`model_tools.py`](../../model_tools.py#L872-L930) `handle_function_call` |
| 2 hours | Add [`tools/registry.py`](../../tools/registry.py#L181-L197), [`toolsets.py`](../../toolsets.py#L11-L75), [`hermes_state_common.py`](../../hermes_state_common.py#L373-L449), [`gateway/run_inbound.py`](../../gateway/run_inbound.py#L1285-L1345), [`gateway/run.py`](../../gateway/run.py#L5750-L5800), and `cron/scheduler.py::run_job` |
| One day | Read [02](02-architecture.md) and [03](03-runtime-flows.md), then [04](04-agent-model-tools.md), [05](05-state-memory-skills.md), [06](06-tech-stack-and-development.md), and [07](07-code-review.md); trace one real turn from request to tool backend |

## Limits

Line references are tied to the SHA above and should be rechecked after updating the analyzed commit. Large gateway, cron, and agent modules were inspected by relevant symbols rather than read sequentially. Tests, lint, builds, and runtime behavior were not executed. Specific integrations/platforms, all plugins, provider API behavior, and shutdown behavior on every host remain unverified. Vietnamese notes, if maintained locally, are kept under the ignored `vi/` directory.
