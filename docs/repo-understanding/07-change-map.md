# Change map

This map points to the smallest useful starting area for common future changes. Verify the current source before editing; the paths below are architectural entry points rather than exhaustive file lists.

## If I need to change startup or invocation behavior

Start with:

- `src/main.c`
- `src/cli.c`, `src/cli.h`
- `src/agent.c`
- `src/oneshot.c`

Why: `main` and CLI parsing decide process mode and initial selections. Interactive and one-shot code should stay thin; behavior common to both belongs below them.

Caveat: avoid implementing the same continuation behavior twice. Shared agent behavior normally belongs in `agent_core` or `agent_loop`.

## If I need to change the model/tool continuation loop

Start with:

- `src/agent_loop.c`
- `src/agent_dispatch.c`
- `src/agent_tool.c`
- `src/turn.c`
- `src/provider.h`

Why: these files define how streamed model output becomes a turn, how tool calls are executed, and when another provider turn is made.

Caveat: keep provider-native JSON and terminal rendering outside this layer.

## If I need to change provider support

Start with:

- `src/providers/registry.c`
- `src/providers/provider_config.c`
- `src/providers/http_provider.c`
- `src/providers/wire.c`
- the relevant protocol body/event files
- `src/provider.h`

Why: a provider is preferably data plus shared protocol/transport behavior. The registry is the shipped selection surface, while wire/protocol modules own serialization and parsing.

Caveat: do not add provider-specific behavior to the generic agent loop merely because one API has a unique response shape.

## If I need to add or change a model-callable tool

Start with:

- `src/tool.h`
- `src/agent_core.c`
- the relevant file under `src/tools/`
- `meson.build`
- corresponding `tests/tools/` coverage

Why: compiled-in tools use a common contract and central registration.

Caveat: tool process/filesystem behavior can have user-visible side effects. Preserve the existing separation between tool execution, model conversation state, and terminal presentation.

## If I need to change shell execution

Start with:

- `src/tools/bash.c`
- `src/tools/bash_env.*`
- `src/tools/bash_shell.*`
- `src/tools/bash_process.*`
- `src/tools/bash_output.*`
- `src/tools/bash_cd_strip.*`
- `src/tools/bash_classify.*`
- `tests/tools/test_bash*.c`

Why: the shell tool is intentionally decomposed into command interpretation, process management, environment, output, and shell concerns.

Caveat: background-task behavior also crosses into `task_registry` and `task_wait`.

## If I need to change sessions, resume, or conversation persistence

Start with:

- `src/session.c`, `src/session.h`
- `src/session_picker.*`
- `src/session_prune.*`
- `src/provider.h`
- `tests/test_session*.c`

Why: the persisted format mirrors provider-independent conversation items and selection metadata.

Caveat: changes here can affect backward compatibility with existing user session files. Distinguish canonical persisted state from the history/transcript display projections.

## If I need to change context or compaction

Start with:

- `src/compact.c`
- `src/agent_core.*`
- `src/provider.h`
- `src/model_meta.*`
- `tests/test_compact.c`

Why: context is derived from the canonical item log and selected model capabilities.

Caveat: compaction currently preserves old canonical records and changes the model-visible floor. Do not accidentally turn summarization into destructive history rewriting.

## If I need to change configuration

Start with:

- `src/config.c`, `src/config.h`
- `docs/configuration.md`
- `tests/test_config.c`
- selection tests if provider/model/preset resolution changes

Why: user-facing settings belong in the config registry and should be consumed by canonical key.

Caveat: direct environment reads are reserved for bootstrap/process concerns or intentionally environment-only secrets. Preserve the documented precedence model.

## If I need to change terminal presentation

Start with:

- `src/render/`
- `src/terminal/`
- `src/history.c`
- `src/transcript.c`

Why: presentation is deliberately separate from agent, provider, and persistence state.

Caveat: interactive terminal changes may require a tmux-driven end-to-end test; pure formatting/state changes should be tested below that level when possible.

## If I need to change provider/model metadata or usage accounting

Start with:

- `src/model_meta.*`
- `src/catalog.*`
- `src/agent_usage.*`
- `src/agent_stats.*`
- provider metadata hooks where necessary

Why: the provider protocol should report facts; common code resolves capabilities and computes cross-provider usage/cost views.

Caveat: provider-reported exact cost and locally estimated cost are distinct concepts in the item/usage model.
