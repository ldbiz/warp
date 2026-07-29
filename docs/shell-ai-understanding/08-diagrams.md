# Architecture diagrams

## 1. Input interpretation and routing

```mermaid
flowchart LR
  U[User types in GUI editor] --> P[ParsedTokensSnapshot]
  P --> G{Locked or forced?}
  G -- yes --> M[Current Shell or AI InputConfig]
  G -- no --> C[Local async detection]
  C --> H[denylist, history, follow-up, aliases, shell heuristics]
  H --> O[Embedded BERT tiny V3 ONNX / heuristic fallback]
  O --> M
  M -->|Shell + submit| S[Terminal session]
  M -->|AI + submit| A[Agent RequestParams]
  A --> W[Warp backend]
  W --> MP[Model provider]
```

## 2. Windows shell execution

```mermaid
flowchart LR
  subgraph D[Windows graphical desktop client]
    E[Input editor] --> TV[Terminal view / ActiveSession]
    R[TerminalModel grid and blocks]
    HI[History persistence]
  end
  TV -->|bytes| CP[Windows ConPTY]
  CP --> SH[Configured shell process: PowerShell, cmd, WSL shell, Git Bash]
  SH -->|syntax execution| OS[Local programs and filesystem]
  SH -->|PTY bytes + shell integration metadata| CP
  CP --> R
  R --> HI
```

## 3. AI request and tool execution

```mermaid
sequenceDiagram
  actor User
  participant Desktop as Windows desktop client
  participant Warp as Warp backend / agent service
  participant Provider as Model provider
  participant Shell as Local shell via ConPTY
  participant FS as Local filesystem

  User->>Desktop: Submit input routed as AI
  Desktop->>Desktop: Select/create conversation; gather local context
  Desktop->>Warp: multi-agent Request (model, context, tools, optional BYOK)
  Warp->>Provider: Provider request (implementation unavailable)
  Provider-->>Warp: Model stream/tool proposal
  Warp-->>Desktop: ResponseEvent stream / client action
  Desktop-->>User: Render proposal and request permission if needed
  User->>Desktop: Allow / deny / take control
  alt shell tool allowed
    Desktop->>Shell: Submit agent command
    Shell->>FS: Read/write/execute as command requires
    Shell-->>Desktop: Output, exit status, block metadata
    Desktop->>Warp: Structured action result
    Warp->>Provider: Continue conversation (unavailable implementation)
  else denied
    Desktop->>Warp: Denied/cancelled action result
  end
  Warp-->>Desktop: Continued streamed response
```

## 4. BYOK/custom endpoint trust boundary

```mermaid
flowchart TB
  subgraph Machine[User's Windows machine]
    UI[AI settings]
    SS[OS secure storage: AiApiKeys]
    Client[Warp graphical client]
    LF[Local files and terminal context]
    LocalEP[Possible localhost custom endpoint]
    UI --> SS
    SS --> Client
    LF -->|only when gathered/attached/tool result| Client
  end
  subgraph WarpInfra[Warp infrastructure]
    API[ServerApi endpoint]
    Agent[Agent/routing service - source absent]
    API --> Agent
  end
  subgraph ProviderInfra[Provider/custom infrastructure]
    Provider[OpenAI / Anthropic / Google / OpenRouter / custom endpoint]
  end
  Client -->|prompt + context + optional key, URL, schema, model config| API
  Agent -->|not confirmable how transformed/authenticated| Provider
  Agent -->|event stream| API
  API --> Client
  Client -. no direct interactive provider call found .-> Provider
  Agent -. localhost reachability not established .-> LocalEP
```

Dashed edges explicitly mean “not established by this repository,” not guaranteed network paths.
