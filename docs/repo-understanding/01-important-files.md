# Important files and directories

This is intentionally selective. It lists the files that best explain Hax as a working system.

## Entrypoints

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `src/main.c` | Process entrypoint | Connects configuration, CLI selection, provider/session setup, and the selected front end. |
| `src/agent.c` | Interactive front end | Owns REPL-facing behavior and presentation hooks while delegating shared execution to the core. |
| `src/oneshot.c` | One-shot front end | Powers non-interactive prompt execution and clean stdout-oriented use from scripts. |
| `src/cli.c`, `src/cli.h` | CLI parsing | Defines the invocation surface that selects one-shot, resume, provider/model, and other process options. |

## Core application logic

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `src/agent_core.c`, `src/agent_core.h` | Shared agent session setup | Centralizes behavior that must be identical across interactive and one-shot use, including the built-in tool set. |
| `src/agent_loop.c`, `src/agent_loop.h` | Agent continuation loop | Coordinates model requests, tool calls, subsequent turns, cancellation, and turn limits. |
| `src/agent_dispatch.c` | Turn dispatch | Connects completed model turns to the next agent action. |
| `src/agent_tool.c` | Tool-call execution support | Bridges provider-independent tool calls to registered Hax tools and their results. |
| `src/turn.c`, `src/turn.h` | Turn state machine | Converts streamed provider events into owned, provider-independent conversation items. |
| `src/provider.h` | Core data contracts | Defines items, context, stream events, model metadata, and the provider interface. |
| `src/tool.h` | Tool contract | Stable seam for compiled-in tools. |
| `src/compact.c` | Context compaction | Summarizes older history without deleting the canonical item log. |

## Providers and transport

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `src/providers/registry.c` | Provider registry | Holds shipped provider definitions and their selection priority. |
| `src/providers/provider_config.c` | Provider configuration | Applies provider-specific configuration and supports configurable compatible/custom endpoints. |
| `src/providers/wire.c`, `src/providers/wire.h` | Wire dialect abstraction | Separates provider request/stream formats from the common agent event model. |
| `src/providers/http_provider.c` | Shared HTTP-provider behavior | Reuses authentication, endpoint, and HTTP mechanics across provider families. |
| `src/transport/` | HTTP, SSE, retry, OAuth, CA support | Owns network mechanics independently of provider and agent behavior. |
| `src/model_meta.c`, `src/catalog.c` | Model capability and catalog data | Resolve context limits, image/tool support, pricing, and metadata used by live agent decisions. |

## Tools and execution

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `src/tools/bash.c` and `src/tools/bash_*.c` | Shell tool | Implements command execution, shell selection, environment, output handling, process lifecycle, timeout/background behavior, and command classification helpers. |
| `src/tools/read.c`, `edit.c`, `write.c` | File tools | Supply the basic repository-reading and editing capabilities exposed to the model. |
| `src/tools/task_registry.c`, `task_wait.c` | Background tasks | Track detached work and make completion available to later turns. |

## Persistence and presentation

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `src/session.c`, `src/session.h` | Session persistence | Stores resumable JSONL conversations grouped by working directory. |
| `src/history.c` | User-facing conversation reconstruction | Rebuilds display history from canonical conversation items. |
| `src/transcript.c` | Model-facing transcript | Produces an inspectable view of what the model sees. |
| `src/render/` | Conversation rendering | Handles Markdown, tool display, differential output, progress, and display state. |
| `src/terminal/` | Terminal interaction | Owns input, pickers, themes, width handling, notifications, clipboard, and VT details. |

## Configuration

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `src/config.c`, `src/config.h` | Config registry and resolution | Implements typed settings and their precedence across runtime overrides, environment, state, config, and defaults. |
| `docs/configuration.md` | User-facing configuration reference | Documents XDG locations, precedence, presets, tool settings, provider settings, and environment variables. |
| `docs/providers.md` | Provider reference | Documents supported providers, compatible endpoints, authentication, and provider-specific behavior. |

## Tests and fixtures

| File / directory | Role | Why it matters |
| --- | --- | --- |
| `tests/meson.build` | Test inventory | Mirrors production modules into unit tests and registers end-to-end scenarios. |
| `tests/harness.c`, `tests/harness.h` | C test harness | Common assertion and temporary-directory support. |
| `tests/loopback.c`, `tests/loopback.h` | Network fixture | Allows transport/provider behavior to be tested without real remote services. |
| `tests/tools/bash_fixtures.c`, `.h` | Shell fixture support | Shared deterministic test infrastructure for bash-tool behavior. |
| `tests/e2e/` | Binary-level scenarios | Exercises CLI, one-shot, interrupt, and interactive behavior through the built program. |
