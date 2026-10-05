# Behaviour walkthroughs

## 1. Run a one-shot agent request

### Trigger

A caller invokes Hax non-interactively, for example:

`hax -p "list TODOs"`

or pipes a prompt into one-shot mode.

### Main files

- `src/main.c`
- `src/cli.*`
- `src/oneshot.c`
- `src/agent_core.*`
- `src/agent_loop.*`
- `src/provider.h`

### Bird's-eye flow

Startup resolves configuration, session behavior, and the selected provider/model. The one-shot front end supplies the prompt to the same agent loop used by the REPL.

The model may answer directly or request one or more tools. Tool results re-enter the conversation and the loop continues until a final response or failure. The one-shot front end then emits clean caller-oriented output rather than entering the interactive input loop.

### Inputs and outputs

**Inputs:** prompt text, current working directory, CLI/environment/config selections, optional existing session.

**Outputs:** final assistant output on the process output streams, optional updated session state, tool side effects when the model invoked tools.

### Tests

`tests/test_oneshot.c` covers one-shot internals; `tests/e2e/test_oneshot.py` covers behavior visible at the executable boundary.

---

## 2. Execute a model-requested shell command

### Trigger

A provider response includes a tool call for the built-in bash tool.

### Main files

- `src/agent_tool.c`
- `src/tools/bash.c`
- `src/tools/bash_env.c`
- `src/tools/bash_shell.c`
- `src/tools/bash_process.c`
- `src/tools/bash_output.c`
- `src/tools/task_registry.c`

### Bird's-eye flow

The turn assembler records the tool call in the common conversation state. Agent dispatch resolves the requested tool and invokes the bash implementation.

The shell-tool subsystem prepares the shell and environment, starts the process, captures output, applies configured timeout/output rules, and returns a provider-independent tool result. Commands that deliberately or automatically outlive the foreground wait can become managed background tasks when tasks are enabled.

The result becomes part of the next model context, so the model can interpret command output and continue.

### Inputs and outputs

**Inputs:** tool-call arguments from the model, current working directory, inherited environment, Hax bash/task settings.

**Outputs:** tool-result text/images as applicable, process side effects, and possibly a managed task record.

### Tests

The `tests/tools/test_bash*.c` family and shared `bash_fixtures` exercise shell, output, process, and classification behavior without relying on a live provider.

---

## 3. Continue or resume a conversation

### Trigger

The user continues the latest session, selects a saved session, or resumes a specific session for interactive or one-shot use.

### Main files

- `src/session.c`, `src/session.h`
- `src/session_picker.*`
- `src/agent_core.*`
- `src/provider.h`

### Bird's-eye flow

Sessions are grouped by working directory and stored as JSONL. Resume loads the header/selection metadata and provider-independent conversation items.

Provider, model, effort, and preset recorded with the conversation participate in configuration resolution so continuation normally returns to the same backend. Explicit CLI selections can override those restored choices.

The reconstructed agent session then enters the normal interactive or one-shot path.

### Inputs and outputs

**Inputs:** session path/ID/selection, current invocation overrides.

**Outputs:** reconstructed conversation state; subsequent turns appended to the resumed session where recording remains available.

### Tests

`tests/test_session.c`, session-pruning tests, selection tests, and REPL/one-shot scenarios collectively cover persistence and visible continuation behavior.

---

## 4. Translate a provider stream into common agent state

### Trigger

The agent loop calls the selected provider for a model response.

### Main files

- `src/provider.h`
- `src/providers/registry.c`
- `src/providers/wire.*`
- protocol-specific body/event files under `src/providers/`
- `src/turn.*`
- `src/transport/`

### Bird's-eye flow

The provider serializes Hax's common context into its native request shape. Shared transport obtains the streamed response.

Protocol-specific parsing translates response fragments into Hax stream events such as text deltas, tool-call fragments, reasoning, retries, completion, or error. The pure turn state machine accumulates those borrowed events into owned canonical items.

This boundary prevents the core agent loop from depending on a provider's native JSON shape.

### Inputs and outputs

**Inputs:** common context, selected model, provider configuration and credentials.

**Outputs:** provider-independent turn items plus usage/provenance data; network diagnostics when enabled.

### Tests

Provider body/event tests, wire tests, HTTP/SSE/retry tests, and `test_turn.c` cover the stages independently.

---

## 5. Compact a long conversation

### Trigger

Automatic context thresholds are reached or compaction is requested through the relevant interactive behavior.

### Main files

- `src/compact.c`
- `src/agent_core.*`
- `src/provider.h`
- `src/session.*`

### Bird's-eye flow

Hax summarizes an older prefix into a synthetic compaction seed. It does not erase the historical canonical items.

Future model contexts start at the newest compaction seed while the session retains older records for history, inspection, and persistence. Compaction therefore changes the model-visible window without redefining the canonical conversation.

### Inputs and outputs

**Inputs:** current conversation items, model context metadata, compaction settings.

**Outputs:** an appended compact-summary item and a smaller subsequent model context.

### Tests

`tests/test_compact.c` and agent/core tests cover compaction boundaries and integration with the conversation model.
