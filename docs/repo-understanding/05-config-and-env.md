# Configuration and environment

## Configuration model

Hax separates durable user configuration from machine-local interactive state.

| Purpose | Default path |
| --- | --- |
| User configuration | `~/.config/hax/config.json` |
| Remembered interactive selections | `~/.local/state/hax/state.json` |
| Sessions and prompt history | `~/.local/state/hax/sessions/<encoded-cwd>/` |
| Model metadata cache | `~/.cache/hax/catalog.json` |

The corresponding `XDG_CONFIG_HOME`, `XDG_STATE_HOME`, and `XDG_CACHE_HOME` variables relocate these roots.

For ordinary settings, the documented precedence is:

`current-process override → resumed conversation → environment → state.json → config.json → default`

The resumed-conversation layer specifically preserves provider, model, effort, and preset selection. Explicit CLI selection still wins.

## Selection and prompt context

| Setting / environment | Purpose |
| --- | --- |
| `provider` / `HAX_PROVIDER` | Select provider; default is automatic selection. |
| `model` / `HAX_MODEL` | Select provider-specific model. |
| `effort` / `HAX_EFFORT` | Select provider-specific reasoning effort. |
| `preset` / `HAX_PRESET` | Select a named provider/model/role stance. |
| `system_prompt` / `HAX_SYSTEM_PROMPT` | Replace the built-in base prompt. |
| `system_prompt_append` / `HAX_SYSTEM_PROMPT_APPEND` | Add instructions while retaining the base prompt. |
| `no_env` / `HAX_NO_ENV` | Omit Hax's environment context. |
| `no_agents_md` / `HAX_NO_AGENTS_MD` | Disable discovered AGENTS.md context. |
| `no_skills` / `HAX_NO_SKILLS` | Disable skill descriptions. |
| `no_subagents` / `HAX_NO_SUBAGENTS` | Disable delegation guidance. |
| `no_tasks` / `HAX_NO_TASKS` | Disable managed background tasks. |

A preset can specify provider, model, effort, prompt replacement/append, display tint, and description. A description makes the preset eligible as a model-visible delegation target.

## Agent behavior and display

Important behavior settings include:

| Setting | Default | Effect |
| --- | --- | --- |
| `compact.auto` | on | Automatically summarize older context near the model limit. |
| `compact.threshold` | 85 | Percentage of context at which automatic compaction is triggered. |
| `max_turns` | 0 | Maximum model round trips in one user turn; zero means unlimited. |
| `context_limit` | auto | Overrides the context-window size used for display and compaction. |
| `markdown` | on | Render Markdown in terminal output. |
| `show_reasoning` | off | Show provider-emitted reasoning when present. |
| `display_width` | auto | Controls rendered width independently of model behavior. |
| `notify` | auto | Controls completion notification behavior. |
| `theme` | auto | Controls terminal theme selection. |
| `keep_awake` | on | Best-effort prevention of idle sleep during an active turn. |

## Sessions and diagnostics

| Setting / environment | Default | Effect |
| --- | --- | --- |
| `no_session` / `HAX_NO_SESSION` | auto | Disable new session/history writes. Auto mode skips normal recording for the mock provider. |
| `session_retention_days` / `HAX_SESSION_RETENTION_DAYS` | 30 | Prune inactive standard session files; zero disables retention pruning. |
| `transcript` / `HAX_TRANSCRIPT` | unset | Mirror the model-facing transcript to a file. |
| `trace` / `HAX_TRACE` | unset | Write HTTP/SSE diagnostics with authentication redacted. |

Session files can contain prompts, responses, file contents, and tool output, so they should be treated as sensitive user state.

## Shell and task behavior

| Setting | Default | Effect |
| --- | --- | --- |
| `bash.timeout` | 2m | Default foreground command timeout before detaching or terminating according to task mode. |
| `bash.timeout_max` | 30m | Maximum timeout the model may request. |
| `bash.timeout_grace` | 2s | Grace period between SIGTERM and SIGKILL. |
| `bash.background_yield` | 5s | Initial output window before explicit backgrounding detaches. |
| `bash.shell` | bash/sh | Shell executable used by the bash tool. |
| `task.wait_timeout` | 10m | Default wait for a tracked background task. |
| `task.max_running` | 32 | Maximum concurrent tracked tasks. |
| `tool_output_cap` | 50k | Maximum tool-output bytes retained for the model. |

When managed tasks are disabled, a bash timeout kills the command rather than turning it into a tracked background task.

## Network and model metadata

`catalog.url` defaults to models.dev and `catalog.refresh` defaults to 24 hours. Catalog refresh is lazy and only occurs for providers that use catalog identity. A stale cache remains usable.

HTTP settings cover retries, retry backoff, and streaming idle timeout. Standard `CURL_CA_BUNDLE`, `SSL_CERT_FILE`, and `SSL_CERT_DIR` variables can alter certificate lookup.

## Provider credentials

First-party credentials are expected through provider-specific environment variables, including:

- `OPENAI_API_KEY`
- `ANTHROPIC_API_KEY`
- `OPENROUTER_API_KEY`
- `OPENCODE_API_KEY`

Codex uses its login flow rather than a normal API-key setting.

First-party endpoints and credential-variable bindings are deliberately pinned. A different endpoint should be modeled as a compatible/custom provider instead of redirecting a first-party credential.

Custom provider definitions support configurable base URL, protocol behavior, headers, and an `api_key_env` indirection so secrets do not need to be stored directly in `config.json`.

## Development-only controls

The repository documents a deterministic mock provider for tests and manual inspection:

- `HAX_PROVIDER=mock`
- `HAX_MOCK_SCRIPT=<path>`

Set `HAX_NO_SESSION=0` when deliberately testing persistence with the mock provider, and use a scratch `XDG_STATE_HOME` to avoid mixing test sessions with user state.
