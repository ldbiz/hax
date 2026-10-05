# Architecture diagrams

## 1. Runtime flow

```mermaid
flowchart TD
    Start["hax process"] --> Resolve["CLI + config + saved state"]
    Resolve --> Provider["Construct provider and model metadata"]
    Resolve --> Session["Create / resume session"]
    Session --> Frontend{"Frontend"}
    Frontend -->|interactive| Agent["agent.c"]
    Frontend -->|one-shot| OneShot["oneshot.c"]
    Agent --> Loop["agent_core + agent_loop"]
    OneShot --> Loop
    Loop --> Context["Provider-independent context"]
    Context --> Adapter["Provider adapter / wire dialect"]
    Adapter --> Net["HTTP / SSE transport"]
    Net --> Adapter
    Adapter --> Event["stream_event"]
    Event --> Turn["turn.c"]
    Turn --> Calls{"Tool calls?"}
    Calls -->|yes| Tool["agent_tool + tools/*"]
    Tool --> Loop
    Calls -->|no| Result["Final frontend output"]
    Turn --> Log["Canonical item log"]
    Log --> Persist["session JSONL"]
```

## 2. Module interaction map

```mermaid
flowchart LR
    subgraph Frontends
      A["agent.c"]
      O["oneshot.c"]
    end

    subgraph Core
      AC["agent_core"]
      AL["agent_loop"]
      AD["agent_dispatch"]
      AT["agent_tool"]
      T["turn"]
      C["compact"]
    end

    subgraph ProviderLayer["Provider layer"]
      R["providers/registry"]
      W["providers/wire + protocol adapters"]
      P["provider.h contracts"]
      M["model_meta / catalog"]
    end

    subgraph IO["External execution"]
      TR["transport/*"]
      Tools["tools/*"]
    end

    subgraph StateAndUI["State and presentation"]
      S["session"]
      H["history"]
      X["transcript"]
      Render["render/* + terminal/*"]
    end

    A --> AC
    O --> AC
    AC --> AL
    AL --> AD
    AD --> AT
    AL --> P
    P --> W
    R --> W
    W --> TR
    W --> T
    T --> AL
    AT --> Tools
    AC --> C
    M --> AC
    AL --> S
    A --> H
    A --> X
    H --> Render
```

## 3. Model turn and tool-call sequence

```mermaid
sequenceDiagram
    participant U as User / caller
    participant F as Frontend
    participant L as Agent loop
    participant P as Provider adapter
    participant T as Turn state machine
    participant X as Tool
    participant S as Session log

    U->>F: Submit prompt
    F->>L: Start user turn
    L->>P: Stream common context
    P-->>T: Text/tool/reasoning events
    T-->>L: Completed common turn items
    L->>S: Append conversation state

    alt Model requested a tool
        L->>X: Execute tool call
        X-->>L: Tool result
        L->>S: Append tool result
        L->>P: Stream updated context
        P-->>T: Next model-turn events
        T-->>L: Completed turn
    end

    L-->>F: Final response
    F-->>U: Render / print result
```

## 4. Conversation representations

```mermaid
flowchart TD
    Items["Canonical struct item log"] --> Context["Model-visible context"]
    Items --> Session["JSONL session persistence"]
    Items --> History["User-facing history"]
    Items --> Transcript["Model-facing transcript"]
    Context --> Provider["Provider-native request serialization"]
    Session --> Resume["Resume / continue"]
    Resume --> Items
```

The important distinction is that provider payloads, terminal display, and persisted session data are projections around the common item model rather than competing sources of truth.
