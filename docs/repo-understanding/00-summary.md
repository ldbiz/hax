# Hax repository summary

## Purpose

Hax is a minimalist terminal-native coding agent implemented as a single C binary. It supports an interactive REPL and non-interactive one-shot use while keeping its internal conversation model independent of any one LLM provider.

Its design deliberately favors small dependencies, ordinary Unix process composition, inspectable state, terminal-native output, and a narrow set of stable extension seams rather than a plugin marketplace or large application framework.

## Technology stack

| Area | Implementation |
| --- | --- |
| Language | C11 |
| Build | Meson, normally driven through the repository Makefile |
| HTTP / streaming | libcurl plus shared HTTP/SSE transport modules |
| JSON | jansson |
| Concurrency | POSIX threads plus focused background-job helpers |
| Terminal UI | In-repo terminal, rendering, picker, input, theme, and VT modules |
| Persistence | JSONL session files under XDG state paths |
| Tests | C unit tests plus Python end-to-end scenarios; tmux drives interactive terminal tests |

## Main runtime model

Hax has one core agent loop shared by two front ends:

- the interactive REPL;
- one-shot mode, exposed by commands such as `hax -p "list TODOs"`.

At a high level:

`user input → construct provider-independent context → stream provider events → assemble a turn → execute requested tools → continue until the model stops requesting tools`

Conversation state is stored as a flat provider-independent item log. Provider adapters translate that state to native provider request formats and translate streamed responses back into common events.

## Main external services and dependencies

The binary can use built-in and compatible LLM providers including OpenAI, Anthropic, Codex, OpenRouter, OpenCode, llama.cpp, and custom endpoints.

Core native dependencies are deliberately small: libcurl, jansson, threads, and an optional math library on platforms that require it. Model metadata may be refreshed from models.dev when enabled.

## Main entry points

- `src/main.c` — process startup and top-level CLI path.
- `src/agent.c` — interactive agent front end.
- `src/oneshot.c` — non-interactive / print-style front end.
- `src/agent_core.c` — shared agent state and built-in tool registration.
- `src/agent_loop.c` — shared continuation loop across model turns and tool calls.
- `src/provider.h` — provider-independent conversation, context, event, and provider contracts.
- `src/tool.h` — compiled-in tool extension seam.

## Architecture summary

Hax keeps provider protocols, agent behavior, tools, persistence, and terminal presentation separated.

The canonical conversation is not a provider-specific message list. It is a flat sequence of `struct item` records owned by the agent session. A provider receives a `struct context` view of those records and emits provider-independent `stream_event` values. The turn state machine turns those events into durable conversation items.

Interactive and one-shot modes sit above the same core loop. Tool implementations sit below it, while provider adapters and shared HTTP/SSE transport handle remote model communication. Sessions persist the common item representation rather than a provider-native payload.

## Read these first

1. `AGENTS.md` — compact architecture map, repository rules, test conventions, and extension seams.
2. `src/provider.h` — core conversation and provider contracts.
3. `src/agent_core.c` — shared agent setup and built-in tools.
4. `src/agent_loop.c` — model/tool continuation behavior.
5. `src/agent.c` and `src/oneshot.c` — interactive versus one-shot front ends.
6. `src/turn.c` / `src/turn.h` — streamed-event to conversation-item state machine.
7. `src/session.c` / `src/session.h` — JSONL conversation persistence and resume state.
8. `src/providers/registry.c` and `src/providers/wire.c` — provider registration and protocol translation.
9. `src/tools/bash.c` and sibling `bash_*.c` files — shell execution path.
10. `docs/configuration.md` and `docs/providers.md` — user-visible configuration and provider behavior.
