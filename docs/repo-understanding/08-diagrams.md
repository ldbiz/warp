# Architecture diagrams

These diagrams intentionally stop at subsystem boundaries. Many internal models and views are omitted so that startup and behavior remain legible.

## 1. Runtime flow

```mermaid
flowchart TD
    Launcher[script/run, script/run-tui, or channel binary] --> Channel[ChannelState: identity, endpoints, feature cohorts]
    Channel --> Args[warp_cli argument and environment parsing]
    Args --> Role{LaunchMode or worker}
    Role -->|GUI, TUI, CLI, daemon, test| Boot[Shared run_internal bootstrap]
    Role -->|terminal/plugin/proxy/search worker| Worker[Dedicated worker entrypoint]
    Boot --> Foundation[Logging, tracing, paths, settings, secure storage]
    Foundation --> Core[Auth, server clients, persistence, terminal, cloud, AI, MCP]
    Core --> Mode{Selected surface}
    Mode -->|App or test| GUI[Native WarpUI windows and workspaces]
    Mode -->|TUI| TUI[RootTuiView and terminal draw/input driver]
    Mode -->|CLI or daemon| Headless[Command execution or remote service loop]
    GUI --> Loop[Actions, events, async tasks, rendering and persistence]
    TUI --> Loop
    Headless --> Loop
```

## 2. Module and file interaction

```mermaid
flowchart LR
    Bins[app/src/bin and warp_tui/src/bin] --> App[app/src/lib.rs]
    CLI[crates/warp_cli] --> App
    Channel[warp_core/channel and paths] --> App
    Settings[crates/settings and app/src/settings] --> App

    App --> Workspace[workspace, tabs, panes, root_view]
    App --> Terminal[app/src/terminal]
    App --> AI[app/src/ai and crates/ai]
    App --> Drive[app/src/drive and cloud_object crates]
    App --> Server[app/src/server and warp_server_client]
    App --> Persist[app/src/persistence and crates/persistence]

    Server --> GraphQL[crates/graphql]
    Server --> External[Warp API, RTC, Oz, auth services]
    Drive --> GraphQL
    AI --> Server
    AI --> MCP[crates/mcp]
    AI --> Terminal

    Workspace --> GUI[warpui and warpui_core GUI elements]
    Terminal --> Emulator[crates/warp_terminal]
    App --> TUI[crates/warp_tui]
    TUI --> TuiCore[warpui_core TUI elements/runtime]
    TUI --> Terminal
```

## 3. Agent conversation with a tool call

```mermaid
sequenceDiagram
    actor User
    participant Surface as GUI, TUI, or CLI
    participant Agent as Agent conversation model
    participant Context as Project context and index
    participant Service as Authenticated AI service
    participant Tool as Local or MCP tool
    participant Store as Conversation and persistence models

    User->>Surface: Submit prompt and attachments
    Surface->>Agent: Start or continue conversation
    Agent->>Context: Assemble repository, rules, skills, and open-file context
    Agent->>Service: Send authenticated conversation request
    Service-->>Agent: Stream message and structured tool request
    Agent-->>Surface: Publish transcript events
    Agent->>Surface: Request permission when required
    User-->>Surface: Approve, deny, or modify scope
    Surface->>Agent: Permission decision
    Agent->>Tool: Execute approved command, file, or MCP operation
    Tool-->>Agent: Structured result
    Agent->>Service: Continue with tool result
    Service-->>Agent: Stream final response and usage
    Agent->>Store: Update local/server-backed conversation state
    Agent-->>Surface: Publish completed blocks and effects
```

## 4. Terminal input and rendering

```mermaid
sequenceDiagram
    actor User
    participant View as GUI or TUI terminal view
    participant Input as Input/editor model
    participant PTY as Local TTY and shell process
    participant Model as Terminal emulator/model
    participant Session as Workspace/session persistence

    User->>View: Type or paste input
    View->>Input: Dispatch key/action
    Input->>PTY: Write accepted terminal bytes
    PTY-->>Model: Shell output and integration markers
    Model-->>View: Screen, cursor, and semantic state changed
    Model->>Session: Update blocks, history, and restorable metadata
    View-->>User: Render updated pixels or terminal cells
```
