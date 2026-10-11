# 06 — Technology stack and development

## Declared stack

| Component | Evidence and use |
|---|---|
| Python | Package `hermes-agent` requires `>=3.11,<3.15` in `pyproject.toml` [L5-L18](../../pyproject.toml#L5-L18); agent, CLI, gateway, and tools are Python. |
| OpenAI-compatible SDK | `openai` is a runtime dependency under a Python version marker [pyproject.toml L40-L45](../../pyproject.toml#L40-L45); this does not mean OpenAI is the only supported provider. |
| Optional Python extras | Provider/backend dependencies such as Anthropic, web search, image generation, and TTS are declared as extras [L218-L254](../../pyproject.toml#L218-L254). Availability depends on extra/config. |
| SQLite | Session tables are declared in `hermes_state_common.py`; WAL/locking behavior depends on runtime and host. |
| TypeScript/JavaScript | `ui-tui`, `apps/desktop`, and `web`; Node is not the agent core runtime based on the inspected Python entry points. Workspace scripts were not exhaustively collected. |
| Electron | Desktop shell in `apps/desktop`; packaged builds were not verified on a host. |
| Docker/s6 | Container lifecycle files exist under `docker/`; this does not mean every deployment uses Docker. |

## Repository commands verified (not executed)

- Executables `hermes`, `hermes-agent`, `hermes-acp` are declared in [`pyproject.toml`](../../pyproject.toml#L557-L560).
- Dev/test extras are declared in [`pyproject.toml`](../../pyproject.toml#L530-L547).
- `scripts/run_tests.sh` is the repository-prescribed test runner, documented in `tests/AGENTS.md`; it supports a full suite or a selected test path. Not run.
- `python scripts/check` is mentioned in root `AGENTS.md` as the repository check command. Not run.
- PEP 517 build backend `setuptools.build_meta` is declared at [`pyproject.toml`](../../pyproject.toml#L549-L555). The complete build command/effects were not verified.
- JavaScript workspace dev/test/build commands were not collected from every workspace manifest.

## Test strategy and CI

Pytest is configured for `tests/` and excludes `integration` and `live` markers by default [pyproject.toml L839-L853](../../pyproject.toml#L839-L853). Root instructions require per-file isolation and a temporary `HERMES_HOME`; JavaScript suites run separately. These are configuration/repository rules, not test results. Coverage, CI matrix, all E2E paths, and test counts by flow were not examined.
