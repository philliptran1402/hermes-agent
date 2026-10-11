# 01 — Repository map

This is a selective map of important areas, not a complete directory tree.

| Area | Responsibility and evidence |
|---|---|
| Agent core | `run_agent.py` exposes the `AIAgent` facade; `agent/` contains turn phases, prompts, providers, memory, and compression. Example: `agent/conversation_loop.py::run_conversation`. |
| Tool layer | `model_tools.py` coordinates schemas and dispatch; `toolsets.py` defines groups; `tools/registry.py` stores `ToolEntry` metadata (schema, handler, checks, environment, async flag) at [L181-L197](../../tools/registry.py#L181-L197); `tools/*` contains implementations and execution environments. |
| CLI | `hermes_cli/main.py::main` is the `hermes` executable declared in [pyproject.toml L557-L560](../../pyproject.toml#L557-L560); `cli.py::HermesCLI` fronts the REPL and `hermes_cli/` contains commands, config, and setup. |
| Integrations | `gateway/platforms/` contains messaging adapters; `plugins/platforms/` contains plugin adapters; `plugins/model-providers/`, `plugins/web/`, and `plugins/context_engine/` extend runtime capabilities. `plugins/plugin_loader.py::load_plugin_module` dynamically loads plugin modules [L144-L170](../../plugins/plugin_loader.py#L144-L170). |
| UI | `ui-tui/` is the TypeScript TUI; `tui_gateway/` is its Python JSON-RPC backend; `apps/desktop/` is Electron; `web/` is the dashboard. |
| Infrastructure | `hermes_state*.py` implements SQLite persistence; `cron/` handles jobs and delivery; `hermes_platform/` resolves host facts and resources; `pm/` manages dependencies/runtime; `docker/` and `scripts/` contain deployment and developer tooling. |
| Tests | `tests/` contains the pytest suite arranged around source areas; `pyproject.toml` sets `testpaths = ["tests"]` [L839-L853](../../pyproject.toml#L839-L853). JavaScript workspaces have separate tooling. |
| Content/tooling | `skills/` contains loadable skills; `optional-skills/` contains skills that are shipped but not active by default; `plugin-catalog/` contains catalog metadata; `website/` contains Docusaurus docs. A skill is Markdown instruction content, not an executable Python tool. |

## Verified entry points

| Command/process | Entry | Connects to |
|---|---|---|
| `hermes` | `hermes_cli.main:main` | CLI dispatcher/REPL → configuration and `AIAgent` |
| `hermes-agent` (legacy) | `agent.legacy_cli:main` | Legacy CLI; not traced end to end |
| `hermes-acp` | `acp_adapter.entry:main` | ACP adapter; flow not traced |
| Gateway | `gateway.run:main` → `start_gateway` | Runner startup, platform adapters, `MessageEvent` handling |
| Python API | `run_agent.py::AIAgent` | `agent.agent_init.init_agent`, then `agent.conversation_loop` |
| TUI backend | `tui_gateway.entry:main` | JSON-RPC server; TUI/Desktop clients |
| Cron | scheduler/job subsystem | Job store → `run_job` → agent → delivery; standalone CLI invocation was not traced end to end |

`pyproject.toml` verifies three console scripts. It does not establish how every service is installed or launched on every operating system. UI and gateway setup commands also depend on workspace manifests, host, and configuration.
