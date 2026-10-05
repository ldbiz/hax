# Tests and fixtures

## Test structure

Hax has two main test layers:

1. **C unit/component tests** linked against the same production static library as the executable.
2. **Python end-to-end scenarios** that exercise behavior visible only through the built binary.

The normal repository command is:

```sh
make tests
```

Focused tests can be run through:

```sh
scripts/check.sh test <name>...
```

The repository also defines sanitizer build directories for Address/UndefinedBehavior Sanitizer and ThreadSanitizer runs.

## Unit tests

`tests/meson.build` mirrors the production source layout. A production module normally has a corresponding `test_*.c` file at the same conceptual path.

Examples:

| Production area | Representative tests |
| --- | --- |
| Agent loop/state | `test_agent.c`, `test_agent_core.c`, `test_agent_loop.c`, `test_agent_dispatch.c` |
| Conversation assembly | `test_turn.c`, `test_history.c`, `test_transcript.c` |
| Persistence | `test_session.c`, `test_session_prune.c` |
| Configuration/model selection | `test_config.c`, `test_select_config.c`, `test_select_model.c`, `test_model_meta.c` |
| Provider translation | protocol-specific tests under `tests/providers/` |
| Rendering/terminal | tests under `tests/render/` and `tests/terminal/` |
| Shell and file tools | `tests/tools/test_bash*.c`, `test_read.c`, `test_edit.c`, `test_write.c` |
| Network transport | `tests/transport/test_http.c`, `test_sse.c`, `test_retry.c`, `test_oauth.c` |

The unit harness is intentionally lightweight. `tests/harness.h` supplies assertion macros and scratch-directory helpers.

## Shared fixtures

A `test_support` static library contains reusable fixtures instead of duplicating test setup across translation units.

Important shared support includes:

- `tests/harness.c/.h` — assertions and test lifecycle support;
- `tests/loopback.c/.h` — local network peer/server behavior;
- `tests/tools/bash_fixtures.c/.h` — deterministic shell-tool fixtures.

The repository guidance explicitly prefers reusing or extracting shared fixtures over creating parallel copies.

## Provider testing

Most provider and transport behavior is designed to be testable without a live LLM API.

Provider body serializers, event translators, model metadata, retry behavior, and wire abstractions have separate tests. A mock provider can drive complete agent behavior without network credentials.

For behaviors that genuinely depend on HTTP interaction, loopback/fake endpoints are preferred over paid provider calls.

## End-to-end tests

The registered Python scenarios are:

- `tests/e2e/test_oneshot.py`
- `tests/e2e/test_repl_interrupt.py`
- `tests/e2e/test_repl_smoke.py`

These run the built Hax binary with a controlled mock provider.

Interactive tests use tmux because the REPL and pickers require an actual terminal. The harness sends keys and captures pane output, which makes terminal-visible behavior testable without hand-written pseudo-terminal code.

## What the tests document well

The current layout provides strong structural coverage of:

- provider-independent agent and turn state;
- provider serialization and stream parsing;
- tool execution;
- session persistence;
- configuration resolution;
- rendering and terminal primitives;
- one-shot and core REPL behavior;
- network retry and transport behavior.

The test organization itself documents subsystem ownership: new behavior is expected to be tested at the lowest layer that can observe it, with end-to-end tests reserved for executable-level effects.

## Coverage gaps or limits visible from the repository

These are not necessarily defects:

- Real provider interoperability can change independently of the repository; deterministic provider tests cannot guarantee every remote API remains compatible.
- The end-to-end suite intentionally covers a small number of high-value terminal scenarios rather than every REPL command or provider combination.
- Platform-specific behavior is distributed across Linux/macOS/BSD code paths; full confidence still depends on CI or manual runs on the relevant platforms.
- Performance characteristics such as startup latency and long-session memory use are architectural goals but are not fully represented by functional tests alone.

## Validation guidance for documentation-only changes

This repository-understanding branch changes documentation only. The important validation is:

- ensure the branch diff contains only `docs/repo-understanding/`;
- preview Mermaid diagrams;
- verify referenced source paths still exist.

A full build/test run is not necessary to prove Markdown-only changes, though `make tests` remains the standard behavioral suite after application changes.
