# Behaviour walkthroughs

## 1. Run an interactive shell command

| Aspect | Description |
|---|---|
| Trigger | A user opens a workspace/session and submits terminal input in a GUI pane or TUI terminal session. |
| Modules involved | `app/src/workspace/`, `app/src/terminal/input.rs`, `app/src/terminal/local_tty/`, `app/src/terminal/terminal_manager.rs`, `crates/warp_terminal/`, and the active GUI/TUI view. |
| Inputs | Key events or pasted text, shell/session settings, current working directory, environment, terminal dimensions, and shell integration state. |
| Outputs | PTY input/output, updated emulated screen and command blocks, rendered cells/pixels, command history, and restorable session state. |
| External calls and side effects | Spawns a local shell/terminal-server process; reads/writes the PTY; may read shell history and repository metadata; persists session metadata. |
| Tests | GUI integration scenarios cover bootstrapping, input, history, subshells, keyboard protocols, session restoration, and workspace behavior. Unit tests cover local TTY environment construction and terminal models. TUI tests cover terminal session state, input detection, rendering, and completion behavior. |

**Bird's-eye flow:** A workspace creates a terminal session backed by a PTY and terminal model. User input is interpreted according to the active input/editor mode and sent to the shell, while shell integration enriches the byte stream with command boundaries and working-directory information.

The emulator converts output into screen state. App-level terminal logic groups relevant activity into blocks, records history/restoration data, and notifies the active front-end to paint the new state.

## 2. Run an agent conversation that uses tools

| Aspect | Description |
|---|---|
| Trigger | A user starts Agent Mode from the GUI/TUI or invokes an agent CLI command with a prompt. |
| Modules involved | `app/src/ai/agent/`, `app/src/ai_assistant/`, `crates/ai/`, `app/src/server/`, `crates/warp_server_client/`, project-context/index models, MCP integration, and agent transcript views. |
| Inputs | Prompt, attachments, open/project files, repository rules and skills, model/provider selection, conversation state, permissions, and available tools/MCP servers. |
| Outputs | Streamed assistant messages, tool calls/results, file edits, shell command results, todos/plans, usage state, and a persisted or server-backed conversation. |
| External calls and side effects | Authenticated AI requests; optional model-provider requests; MCP connections; local command execution; filesystem reads/writes after permission checks; telemetry and cloud conversation updates when configured. |
| Tests | `app/src/ai/**_tests.rs` covers conversations, context, model routing, persisted workspaces, and agent subcomponents. GUI integration has Agent Mode/AI Assistant scenarios. `crates/warp_tui` render/model tests cover agent messages, tool calls, plans, permissions, diffs, and orchestration. |

**Bird's-eye flow:** Shared app initialization makes authentication, server AI clients, project context, agent models, and tool registries available. A prompt creates or continues a conversation and the selected agent streams structured events rather than only plain text.

Views translate those events into conversation blocks. Tool requests cross explicit execution and permission boundaries; their results feed back into the conversation, while edit/command effects are reflected by filesystem, terminal, and code models.

## 3. Restore a workspace or conversation

| Aspect | Description |
|---|---|
| Trigger | Application startup, reopening a saved window/session, or launching the TUI with `--resume <conversation-token>`. |
| Modules involved | `app/src/persistence/`, `crates/persistence/`, `app/src/session_management/`, `app/src/workspace/`, `app/src/ai/restored_conversations.rs`, and `crates/warp_tui/src/session.rs`. |
| Inputs | Launch-mode persistence scope, SQLite records, user settings, filesystem validity, channel/data profile, and optional server conversation token. |
| Outputs | Recreated windows/workspaces/tabs/panes, terminal blocks and working directories, or a restored agent transcript. |
| External calls and side effects | Opens and migrates SQLite; may spawn replacement shells; a token restore contacts Warp's service; invalid or unavailable local state can be skipped/degraded. |
| Tests | Integration fixtures include databases for restored blocks, code, notebooks, settings, workflows, multiple tabs, missing working directories, and duplicate IDs. Integration scenarios cover pane/session restoration. TUI session/resume tests cover token and startup behavior. |

**Bird's-eye flow:** Startup selects a database that belongs to the current surface and channel. Persistence models load only the state appropriate to the launch mode, then workspace/session managers reconstruct view state and create live resources where needed.

Saved terminal pixels are not the entire runtime: restoration also needs pane topology, shell/session configuration, working-directory handling, and semantic block data. TUI server-conversation restoration is separate; it first creates a logged-in local session, then asks the conversation view to load the supplied token.

## 4. Synchronize a Warp Drive object

| Aspect | Description |
|---|---|
| Trigger | Login/startup synchronization, a user creating or editing a workflow/notebook/folder, a permission action, or a real-time server update. |
| Modules involved | `crates/cloud_objects/`, `crates/cloud_object_models/`, `crates/cloud_object_client/`, `crates/cloud_object_persistence/`, `crates/graphql/`, `app/src/cloud_object/`, and `app/src/drive/`. |
| Inputs | Authenticated user, local cached object/version, edits, permissions, and GraphQL/real-time update data. |
| Outputs | Updated local model/cache and Drive presentation; uploaded object mutations; conflict, permission, or sync status where applicable. |
| External calls and side effects | GraphQL HTTP operations, real-time WebSocket traffic, SQLite/cache writes, file export for explicit export actions, and user notifications. |
| Tests | GraphQL operation tests exercise typed API conversion. Cloud-object crates contain model/client tests. Drive tests cover indexes, panels, and exports; integration database fixtures cover restored workflows/notebooks/cloud objects. |

**Bird's-eye flow:** The cloud-object layer separates domain state from transport and local persistence. An authenticated client loads cached objects promptly, reconciles them with the server, and applies real-time updates to shared models.

Drive and feature-specific views observe those models. User edits become versioned mutations through the client rather than direct UI-to-network calls, allowing permissions, conflicts, persistence, and notifications to be handled consistently.
