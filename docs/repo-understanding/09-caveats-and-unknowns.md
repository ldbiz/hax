# Caveats and unknowns

## Scope of this analysis

This pack describes the repository at the `repo-analysis` branch point. It is based on the source, repository guidance, tests, and user-facing documentation present at that revision.

It intentionally does not evaluate whether every remote provider currently honors the protocol assumptions encoded by its adapter.

## Long-lived external control surface

The inspected documentation clearly exposes:

- an interactive REPL;
- one-shot `-p` / piped invocation;
- persisted session continuation.

No first-class long-lived RPC protocol comparable to an IDE/host control channel was identified in the inspected Hax documentation and architecture files.

For an embedding or always-running copilot design, do not assume a hidden stable daemon API exists. Verify the current CLI/source surface before designing around process reuse. One-shot invocation and session persistence are the explicit non-interactive surfaces confirmed here.

## Process and working-directory semantics

Hax is strongly working-directory aware:

- sessions are grouped by encoded current working directory;
- environment/context discovery depends on the running process;
- the bash tool executes as a child-process capability of the Hax process.

A host that wants Hax to behave as if it were invoked from an existing shell must therefore verify which pieces of live shell state are inherited at process launch and which state changes occur only inside Hax/tool child processes.

In particular, a child process cannot mutate its parent's shell state. Any integration that needs persistent parent-shell changes must define an explicit synchronization mechanism rather than assuming normal subprocess behavior can provide it.

## Security and permission boundary

Hax executes model-requested tools with the operating-system permissions of the Hax process. The repository philosophy emphasizes a small Unix-style tool rather than per-command permission prompts or a plugin sandbox.

Any use in an automatically triggered shell path should separately decide what trust and execution policy is required. That policy is not established by the provider abstraction itself.

## Session compatibility

The session format is designed for resume and includes provider-independent items plus selection metadata. The exact backward-compatibility promise for arbitrary future format changes is not established by the architecture files inspected here.

Before changing serialized item kinds, origins, headers, or selection fields, inspect the current loading/migration behavior and existing session-format documentation in full.

## Provider drift

Provider APIs, model IDs, context limits, pricing, and supported effort/tool/image features can change without a Hax release.

The repository mitigates this with live provider probes, configurable compatible providers, and catalog metadata, but tests against serializers and mock endpoints cannot guarantee remote behavior indefinitely.

## Platform coverage

The repository explicitly supports Linux, macOS, FreeBSD, and OpenBSD, with Windows use directed through WSL.

Some process, terminal, certificate, and library behavior is platform-specific. A source-level architecture review cannot replace execution on the target operating systems.

## Performance assumptions

The codebase is designed around low startup/runtime overhead and a native single binary. Recent repository work also contains explicit performance-oriented changes.

This pack does not benchmark startup latency, memory, provider round-trip overhead, or long-session behavior. Treat performance numbers as a separate measurement task.

## Questions worth confirming before major integration work

1. Is one-shot process invocation sufficient, or does the intended host need a persistent bidirectional control protocol?
2. Which shell state must be reflected continuously: cwd only, exported environment, shell options/functions/aliases, command history, or more?
3. Should an external host allow the full default tool set, restrict tools, or isolate the Hax process?
4. Should integration rely on Hax session persistence, or should the host own conversation identity/state and use Hax as a stateless executor?
5. Are provider/model selection and credentials host-managed or left to Hax's existing config/state resolution?

These are integration decisions, not uncertainties about the basic repository architecture.
