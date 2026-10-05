# Core concepts

## Agent session

**Meaning in this repo:** The in-memory owner of the canonical conversation state and the live agent configuration needed to continue it.

**Where it appears:** Agent-core and loop code; session persistence mirrors enough state to resume a conversation.

**Related files:** `src/agent_core.*`, `src/agent_loop.*`, `src/session.*`.

## Item log

**Meaning in this repo:** The provider-independent canonical conversation. It is a flat sequence of `struct item` values rather than a provider-native message tree.

Items can represent user and assistant messages, tool calls/results, reasoning, model-turn boundaries, and per-turn usage. Origins distinguish ordinary content from synthetic continuation, compaction, interruption, and task-note records.

**Related files:** `src/provider.h`, `src/turn.*`, `src/session.*`, `src/history.c`, `src/transcript.c`.

## Turn

**Meaning in this repo:** One provider `stream()` round trip producing assistant output and optionally tool calls.

A **user turn** begins with one user prompt and can contain multiple provider turns while tools are requested and results are fed back.

**Related files:** `src/turn.*`, `src/agent_loop.*`, `src/provider.h`.

## Context

**Meaning in this repo:** The provider-independent model input assembled from the current system prompt, a model-visible slice of conversation items, tool declarations, effort setting, image capability, and session identity.

Compaction can move the context floor forward without deleting older canonical items.

**Related files:** `src/provider.h`, `src/agent_core.*`, `src/compact.c`.

## Stream event

**Meaning in this repo:** The common event vocabulary emitted by provider adapters. Events cover text deltas, tool-call construction, reasoning, retries, progress, successful completion, and errors.

This is a major isolation boundary: the agent loop consumes common events rather than OpenAI-, Anthropic-, or other provider-specific response structures.

**Related files:** `src/provider.h`, `src/turn.*`, `src/providers/*events*.c`.

## Provider definition and provider instance

**Meaning in this repo:** A provider definition describes how a selectable provider is constructed and what provider-specific capabilities it has. A live provider instance exposes streaming, model discovery, effort support, metadata probing, and optional usage reporting.

Shipped definitions live in the registry; configuration can overlay them or add data-driven compatible providers.

**Related files:** `src/providers/registry.c`, `src/providers/provider_config.*`, `src/provider.*`.

## Wire dialect

**Meaning in this repo:** A protocol-specific serializer/parser pair beneath a provider. It maps Hax's common context into a remote API request and maps SSE or response payloads back into common events.

**Related files:** `src/providers/wire.*`, `src/providers/chat_*.*`, `src/providers/responses_*.*`, `src/providers/anthropic_*.*`.

## Model metadata

**Meaning in this repo:** The resolved capabilities and limits of the selected model, combining provider-reported data, cached catalog data, and user overrides.

It influences context limits, image/tool capability, effort choices, compaction, and cost estimation.

**Related files:** `src/model_meta.*`, `src/catalog.*`, `src/provider.h`.

## Tool

**Meaning in this repo:** A model-callable capability declared through the common tool contract. Built-in tools include shell execution, reading, editing, writing, and task management.

Compiled-in tools are registered centrally rather than discovered through a plugin runtime.

**Related files:** `src/tool.h`, `src/agent_core.c`, `src/agent_tool.c`, `src/tools/`.

## Background task

**Meaning in this repo:** Work that outlives the immediate shell-tool wait window and is tracked for later completion rather than blocking the entire foreground agent turn indefinitely.

**Related files:** `src/tools/task_registry.c`, `src/tools/task_wait.c`, `src/tools/bash_process.c`.

## Session file

**Meaning in this repo:** An append-oriented JSONL representation of a resumable conversation, stored under an XDG state directory grouped by working directory.

It contains a header/selection state and serialized conversation items. Session identity is also passed to providers that can use it for affinity or prompt-cache keys.

**Related files:** `src/session.*`, `docs/sessions.md`, `src/provider.h`.

## Preset

**Meaning in this repo:** A named provider/model/effort stance, optionally with system-prompt additions, tint, and a description.

Described presets can also become delegation targets; simple named favorites need not.

**Related files:** `src/config.*`, `docs/configuration.md`, selection/slash-command code.

## Transcript versus history

**Meaning in this repo:** Two deliberately different projections of the same underlying conversation.

- **Transcript** exposes the model-facing representation for inspection/debugging.
- **History** reconstructs the user-facing conversation display.

Neither is the persistence format itself.

**Related files:** `src/transcript.c`, `src/history.c`, `src/session.c`.
