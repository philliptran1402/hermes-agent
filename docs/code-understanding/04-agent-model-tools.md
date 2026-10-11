# 04 — Agent, models, and tools

## Agent loop and stopping conditions

`AIAgent` is the facade/mixin in `run_agent.py`; its constructor calls `agent.agent_init.init_agent` ([L214-L285](../../run_agent.py#L214-L285)). `run_conversation` in [`agent/conversation_loop.py`](../../agent/conversation_loop.py#L1709-L1745) accepts the user message, system message/history, callbacks, and metadata. Turn phases split preflight, request, error handling, tools, compaction, and finalization. A model response with content can complete a turn; tool calls produce results that are appended to history and the loop continues. Stop gates include iteration/budget limits, interrupts, errors/empty responses, context overflow, and runtime-specific exits; not all exits represent success.

Python synchronous code calling an HTTP/model SDK can occupy its thread. Unlike Node promises, a synchronous call does not yield to an event loop. Gateway async routing does not imply that every model call is nonblocking.

## Provider/model abstraction and API modes

Provider resolution builds the client and request parameters; `api_mode` selects a compatible protocol. Modes present in source include `chat_completions`, `codex_responses`, `codex_app_server`, `anthropic_messages`, and `bedrock_converse` (branches in [`run_agent.py`](../../run_agent.py#L488-L508), [`agent/conversation_loop.py`](../../agent/conversation_loop.py#L1638-L1645), [`agent/turn_request_assembly.py`](../../agent/turn_request_assembly.py#L182-L247), and [`agent/client_lifecycle.py`](../../agent/client_lifecycle.py#L67-L78)). This may not be a complete list. Wire behavior was not verified against live APIs.

## Tool pipeline

`tools/registry.py` discovers built-in modules (`discover_builtin_tools`). Its `ToolEntry` contains schema, handler, availability check, environment requirements, and async metadata [L181-L197](../../tools/registry.py#L181-L197). `toolsets.py` maps toolset names; `model_tools.py` constructs schema lists and `handle_function_call` normalizes arguments/aliases, handles bridges/connectors, runs guards/middleware, and dispatches. An OpenAI-style function schema tells a model how to request a tool; it does not execute the tool.

### Trace: `terminal`

`model_tools.py` calls `discover_builtin_tools()` during module initialization [L149](../../model_tools.py#L149). `tools/registry.py::discover_builtin_tools` scans Python modules that self-register [L96-L115](../../tools/registry.py#L96-L115). `tools/terminal_tool.py` registers `name="terminal"`, the `terminal` toolset, `TERMINAL_SCHEMA`, `_handle_terminal`, and `check_terminal_requirements` [L1639-L1647](../../tools/terminal_tool.py#L1639-L1647). After the model requests the tool, `handle_function_call` normalizes and routes through bridge/guard/middleware/dispatch; `_handle_terminal` validates arguments and calls `terminal_tool(...)` [L1582-L1635](../../tools/terminal_tool.py#L1582-L1635). Backend selection depends on configuration. Approval and sandbox behavior depend on backend and config; `terminal` is not necessarily sandboxed.

### Adding a tool or provider

For a tool, follow `tools/AGENTS.md`: add a self-registering implementation, schema/handler/check function, toolset selection, and behavior tests; account for async handlers and resource lifecycle. For a provider, implement the extension in `plugins/model-providers/<name>/` and wire resolver/config/setup behavior with provider tests. These paths are grounded in the repository's extension guidance; plugin kinds have different contracts.

## MCP and plugins

MCP (Model Context Protocol) connects external tool servers; `tools/mcp_tool.py` and lifecycle modules manage server discovery and schemas. The plugin loader dynamically imports modules while coordinating parents/siblings and module locks [plugin_loader.py L144-L170](../../plugins/plugin_loader.py#L144-L170). Do not assume plugin isolation means process sandboxing; whether user plugin imports are refused depends on isolation configuration.

## Context, compression, caching

Context includes the system prompt, conversation history, runtime metadata/context files, and memory recall. Compression reduces history when thresholds are reached. The repository's invariant is to keep the system prefix stable for prompt caching; intentional compaction is the exception. Memory providers can `prefetch` cached recall before a turn and `sync_turn` after it. For thresholds and provider-specific behavior, inspect `agent/conversation_compression.py`, `agent/prompt_builder.py`, and `agent/memory_manager.py`; this guide does not generalize their numeric settings.
