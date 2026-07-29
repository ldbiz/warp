# Core concepts

## Agentic development environment

**Meaning in this repository:** Warp combines an interactive terminal and code surfaces with first-party and bring-your-own coding agents. Agents can receive project context, call tools, edit files, execute commands, and preserve conversations.

**Where it appears:** Agent orchestration and presentation live primarily under `app/src/ai/`; reusable AI models live in `crates/ai/`; CLI agent commands are defined by `crates/warp_cli/`; TUI transcript presentation lives in `crates/warp_tui/`.

**Related files:** `app/src/ai/agent/`, `app/src/ai/orchestration/`, `app/src/ai/agent_conversations_model.rs`, `crates/ai/src/`, `crates/warp_tui/src/agent_block.rs`.

## Terminal session

**Meaning in this repository:** A live shell/PTY together with an emulated terminal model, input state, working directory, shell integration, and presentation. Sessions can occupy panes/tabs, be restored from persistence, or host agent-driven commands.

**Where it appears:** The app-level lifecycle is in `app/src/terminal/`; emulation primitives are in `crates/warp_terminal/`; the TUI adapts sessions in `terminal_session_view.rs`.

**Related files:** `app/src/terminal/local_tty/`, `app/src/terminal/terminal_manager.rs`, `app/src/terminal/view.rs`, `crates/warp_terminal/`, `crates/warp_tui/src/terminal_session_view.rs`.

## Block

**Meaning in this repository:** A semantic unit of terminal or agent transcript content. Terminal commands and their output can be treated as structured units rather than an undifferentiated character stream; agent messages and tool calls have corresponding transcript blocks.

**Where it appears:** Terminal block state and filtering live under `app/src/terminal/`; Agent Mode block presentation lives under `app/src/ai/blocklist/`; the TUI has terminal and agent block views.

**Related files:** `app/src/terminal/model/block.rs`, `app/src/terminal/model/blocks.rs`, `app/src/ai/blocklist/`, `crates/warp_tui/src/terminal_block.rs`, `crates/warp_tui/src/agent_block.rs`.

## Workspace, tab, and pane

**Meaning in this repository:** These form the desktop composition hierarchy. A workspace owns a collection of tabs and pane groups; panes host terminal sessions and other surfaces such as notebooks, code, Drive, and settings.

**Where it appears:** Workspace behavior spans `app/src/workspace/`, `app/src/workspaces/`, `app/src/tab/`, and `app/src/pane_group/`.

**Related files:** `app/src/workspace/mod.rs`, `app/src/workspaces/`, `app/src/tab/mod.rs`, `app/src/pane_group/`, `app/src/root_view.rs`.

## Entity and handle

**Meaning in this repository:** Long-lived models and views are owned by the global WarpUI application. Other components keep typed handles rather than directly owning those values, and use an `AppContext`/model/view context for reads, updates, subscriptions, tasks, and notifications.

**Where it appears:** This pattern is implemented in WarpUI Core and used throughout `app/` and `warp_tui` startup and rendering.

**Related files:** `crates/warpui_core/src/core/app.rs`, `crates/warpui_core/src/core/entity.rs`, `crates/warpui_core/src/core/window.rs`, `app/src/lib.rs`.

## Action and event

**Meaning in this repository:** Actions represent intent such as commands and keybindings. Entity events notify subscribers of model/view state changes, allowing presentation and services to remain connected without direct ownership.

**Where it appears:** Core dispatch is in WarpUI Core; product action definitions and subscriptions are distributed across views and models.

**Related files:** `crates/warpui_core/src/actions.rs`, `crates/warpui_core/src/core/action.rs`, `crates/warpui_core/src/event.rs`, `app/src/command_palette.rs`.

## Launch mode

**Meaning in this repository:** The process role selected after parsing arguments: desktop app, one-shot/headless CLI, test, remote proxy, remote daemon, or TUI. It controls UI presence, settings namespace, persistence scope, logging destination, indexing, HTTP server startup, and initialization details.

**Where it appears:** Defined and consumed by shared startup in `app/src/lib.rs`.

**Related files:** `app/src/lib.rs`, `crates/warp_cli/src/lib.rs`, `crates/warp_tui/src/session.rs`.

## Channel and channel state

**Meaning in this repository:** Stable, Preview, Dev, Local, Integration, and OSS builds share code but differ in identity, enabled feature cohorts, service endpoints, telemetry/crash/update capabilities, and persistent namespaces.

**Where it appears:** Channel binaries initialize a process-global `ChannelState`; channel configuration is compiled into internal builds or explicitly constructed for OSS/integration.

**Related files:** `app/src/bin/`, `crates/warp_core/src/channel/`, `crates/warp_channel_config/`.

## Feature flag cohort

**Meaning in this repository:** Runtime checks expose product capabilities to Debug, Dogfood, Preview, Release, or Local populations. Channel binaries add the appropriate cohort, and feature checks gate behavior and visible surfaces.

**Where it appears:** Flag definitions live in `crates/warp_features/`; the app re-exports/initializes them and channel binaries add cohort slices.

**Related files:** `crates/warp_features/src/lib.rs`, `crates/warp_core/src/features.rs`, `app/src/features.rs`, `app/src/bin/local.rs`.

## Warp Drive and cloud object

**Meaning in this repository:** Drive is the cloud-backed domain for synchronized objects such as workflows, notebooks, folders, and shared content. A cloud object has identity, permissions, versioned state, persistence, and real-time updates.

**Where it appears:** Domain/client/persistence crates implement synchronization; `app/src/drive/` and feature modules present and edit objects.

**Related files:** `crates/cloud_objects/`, `crates/cloud_object_models/`, `crates/cloud_object_client/`, `crates/cloud_object_persistence/`, `app/src/drive/`.

## Codebase index and project context

**Meaning in this repository:** Project metadata, rules, open files, repository state, and indexed source provide grounded context to Agent Mode. Indexing support depends on launch mode and is intentionally disabled in the TUI until concurrent persistence is safe.

**Where it appears:** Initialization is composed in `app/src/lib.rs`; reusable indexing and project-context models live in `crates/ai/` and repository metadata support in `crates/repo_metadata/`.

**Related files:** `crates/ai/src/index/`, `crates/ai/src/project_context/`, `app/src/ai/codebase_auto_indexing.rs`, `crates/repo_metadata/`.

## MCP server

**Meaning in this repository:** A Model Context Protocol server contributes tools or resources that agents can use. Warp supports file-based MCP configuration, OAuth credentials for providers, connection management, and separate GUI/TUI configuration paths.

**Where it appears:** Protocol/client behavior is in `crates/mcp/`; app-side configuration and lifecycle are in `app/src/ai/mcp/`.

**Related files:** `crates/mcp/`, `app/src/ai/mcp/`, `crates/warp_core/src/channel/config.rs`, `crates/warp_core/src/paths.rs`.

## Persistence scope

**Meaning in this repository:** Durable SQLite state is partitioned between the desktop app, TUI, and remote daemon. This avoids incompatible migrations or concurrent writers across binaries with different lifecycles/versions.

**Where it appears:** Startup selects the scope; the app persistence module opens/migrates the database; Diesel schema/migrations live in the persistence crate.

**Related files:** `app/src/lib.rs`, `app/src/persistence/`, `crates/persistence/src/schema.rs`, `crates/persistence/migrations/`.
