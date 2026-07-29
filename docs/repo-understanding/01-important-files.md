# Important files and directories

This list is intentionally selective. It identifies composition points and subsystem roots rather than every workspace crate.

## Entrypoints

| File / directory | Role | Why it matters |
|---|---|---|
| `script/run` | Cross-platform GUI development launcher. | Chooses internal local versus OSS builds, installs channel configuration when available, and maps development options into the Cargo invocation. |
| `app/src/bin/` | Channel-specific desktop binaries. | Each binary fixes the channel identity, endpoints, integrations, and default feature cohorts before entering shared startup. |
| `app/src/lib.rs` | Shared composition root. | Defines launch modes, dispatches CLI/workers, constructs the WarpUI app, and registers nearly every long-lived service/model. |
| `script/run-tui` | Headless TUI launcher. | Selects the channel binary, builds bundled resources, and runs the terminal-native front-end. |
| `crates/warp_tui/src/session.rs` | TUI process and session bootstrap. | Handles login/API-key operation, mounts the TUI driver, and creates/restores the first terminal conversation. |

## Configuration

| File / directory | Role | Why it matters |
|---|---|---|
| `Cargo.toml` and `app/Cargo.toml` | Workspace, binary, feature, and dependency definitions. | They reveal which capabilities are shared, optional, platform-specific, or tied to a release channel. |
| `crates/warp_core/src/channel/` | Channel-specific runtime configuration and global channel state. | This is the authoritative source for server, Oz, telemetry, update, crash-reporting, app-ID, and MCP OAuth configuration. |
| `crates/warp_core/src/paths.rs` | Persistent path policy. | It prevents channels, profiles, GUI, TUI, and remote daemons from accidentally sharing incompatible state. |
| `crates/settings/src/` and `app/src/settings/` | Typed settings infrastructure and product settings groups. | Settings affect startup, UI, terminal behavior, privacy, sync, themes, input, AI, and front-end-specific behavior. |
| `crates/warp_features/` | Feature-flag definitions and rollout cohorts. | Product behavior is commonly gated at runtime and varies by channel/cohort. |

## Core application logic

| File / directory | Role | Why it matters |
|---|---|---|
| `app/src/terminal/` | Terminal sessions, PTYs, shell integration, input, blocks, and terminal views. | This is the foundation inherited from Warp's terminal product and the execution substrate used by agents. |
| `app/src/ai/` and `app/src/ai_assistant/` | Agent Mode, conversations, tools, context, orchestration, and AI presentation. | These modules implement the repository's agentic-development workflows. |
| `app/src/workspace/`, `app/src/workspaces/`, and `app/src/root_view.rs` | Window/workspace/session composition. | They connect tabs, panes, terminals, notebooks, code, Drive, settings, and global UI into a running desktop app. |
| `crates/warpui_core/src/core/` | Application context, entities, handles, actions, and windows. | This is the shared state/lifecycle model on which both front-ends depend. |
| `crates/warp_terminal/` | Terminal emulation and terminal-domain primitives. | It interprets PTY output and maintains the screen/state consumed by GUI and TUI presentation. |
| `crates/warp_tui/src/` | Headless transcript and terminal UI. | It maps shared models and terminal/agent events into a cell-grid UI with terminal-native input. |

## External integrations

| File / directory | Role | Why it matters |
|---|---|---|
| `app/src/server/` and `crates/warp_server_client/` | Authenticated service-client composition and API access. | Most cloud, account, AI, and synchronization behavior crosses this boundary. |
| `crates/graphql/` and `crates/warp_graphql_schema/` | Typed GraphQL operations and source schema. | They define the client/server contract for users, workflows, notebooks, AI, permissions, and cloud objects. |
| `crates/cloud_objects/`, `crates/cloud_object_client/`, and `app/src/drive/` | Cloud-object domain, synchronization client, and Drive UI. | These implement shared/synchronized workflows, notebooks, folders, and related permissions/actions. |
| `app/src/auth/` and `crates/firebase/` | Authentication lifecycle and Firebase integration. | Login state gates cloud-backed and TUI behaviors and supplies credentials to server clients. |
| `crates/mcp/` and `app/src/ai/mcp/` | Model Context Protocol clients and configuration. | MCP extends agent capabilities with user-configured external tools and resources. |

## UI and assets

| File / directory | Role | Why it matters |
|---|---|---|
| `crates/warpui/` and `crates/warpui_core/src/elements*` | GUI renderer/elements and shared GUI/TUI element abstractions. | Product views rely on these layout, paint, input, and windowing contracts. |
| `app/assets/`, `app/resources/`, and `crates/warp_assets/` | Embedded and bundled fonts, themes, icons, shaders, shell assets, and other resources. | Missing or incorrectly packaged assets can alter startup and visible behavior even when Rust compiles. |

## Tests and fixtures

| File / directory | Role | Why it matters |
|---|---|---|
| `crates/integration/src/` and `crates/integration/tests/` | GUI integration framework, scenarios, and fixture data. | These tests exercise real app behavior such as shell bootstrapping, restoration, input, workspaces, settings, and Agent Mode. |
| `crates/warp_tui/src/*_tests.rs` | Render-to-lines and model tests for the TUI. | They are the principal executable specification for headless layout, interaction, sessions, and agent transcript rendering. |
| Tests adjacent to `app/src/**` | Product subsystem unit tests in separate files. | They document module-level contracts without exposing internal app modules publicly. |

## Build and support scripts

| File / directory | Role | Why it matters |
|---|---|---|
| `script/bootstrap` | Platform setup and shared-skill installation. | It establishes native build prerequisites and the expected local development environment. |
| `script/presubmit` and `script/format` | Canonical verification and formatting entrypoints. | Their exact command set is the local approximation of required CI checks. |
