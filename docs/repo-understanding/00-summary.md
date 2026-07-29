# Repository summary

## Purpose

Warp is an open-source agentic development environment built around a terminal. The repository contains the desktop terminal application, a headless terminal UI, coding-agent experiences, cloud-backed collaboration and synchronization, and the libraries that support those surfaces.

## Technology stack

- **Primary implementation:** Rust 2024 in a large Cargo workspace.
- **Desktop UI:** WarpUI, Warp's entity/view framework, with native windowing and GPU rendering.
- **Headless UI:** `warp_tui`, rendered as a terminal cell grid through WarpUI Core.
- **Terminal execution:** local shells and PTYs, shell integration, a terminal model, and worker processes.
- **Concurrency and networking:** asynchronous Rust, HTTP, WebSocket/GraphQL, and server-sent event clients.
- **Persistence:** SQLite through Diesel, plus typed settings, platform preference stores, files, and secure credential storage.
- **Platform targets:** macOS, Linux/FreeBSD, Windows, WebAssembly, and remote/headless processes, with capability-specific conditional compilation.

## Main runtime model

A small channel-specific binary establishes the build's identity, service endpoints, and feature set, then enters `warp::run`. That shared bootstrap parses GUI, CLI, and worker commands and selects a `LaunchMode`.

All substantial modes initialize a WarpUI `App` and register shared singleton models in an `AppContext`. Models hold authentication, settings, persistence, server clients, terminal state, AI state, and cloud objects; views refer to models and other views through handles and react to actions/events.

The GUI adds native windows and GPU-rendered views. The TUI uses the same shared application/model initialization but mounts `RootTuiView` and a terminal draw/input driver instead of a desktop window.

## Main external services and dependencies

- Warp's HTTP application API and real-time GraphQL endpoint.
- Oz agent-management services and remote agent workers.
- Firebase Authentication; OAuth and platform secure storage for credentials.
- Warp Drive cloud-object synchronization and session-sharing services.
- Optional telemetry, crash reporting, release/update, and staging IAP services, depending on channel configuration.
- Local developer tools and services such as shells, PTYs, Git, language servers, MCP servers, and imported terminal configuration.

## Main entry points

| Entry point | Purpose |
|---|---|
| `./script/run` / `cargo run` | Selects the local or OSS channel and starts the desktop GUI. |
| `app/src/bin/{oss,local,stable,preview,dev}.rs` | Creates channel state and delegates to the shared app runtime. |
| `app/src/lib.rs` | Parses commands, selects launch mode, and initializes the shared system. |
| `./script/run-tui` | Selects/builds a channel-specific headless TUI executable. |
| `crates/warp_tui/src/session.rs` | Parses TUI options, mounts the TUI, and creates a terminal session after login. |
| `crates/warp_cli/src/lib.rs` | Defines the shared GUI/CLI/worker command-line contract. |

## Architecture at a glance

- `app/` is the composition root and contains most product behavior: terminal, Agent Mode, code editing/review, Drive, authentication, settings, workspaces, and desktop views.
- `crates/warp_core`, `crates/warpui_core`, and `crates/warpui` provide cross-cutting application, entity/view, rendering, platform, action, and feature-flag infrastructure.
- Focused crates own reusable capabilities such as terminal emulation, editing, GraphQL, cloud objects, persistence, MCP, and HTTP.
- `crates/warp_tui` is a separate presentation layer over much of the same initialized app state. It deliberately isolates TUI settings, secure storage, and persisted state from the GUI.

## Read these first

1. `README.md` — product scope, licensing, and contributor orientation.
2. `AGENTS.md` — current architecture, development workflow, and front-end distinctions.
3. `Cargo.toml` — workspace boundaries and shared dependencies.
4. `app/src/lib.rs` — launch modes and shared dependency composition.
5. `app/src/terminal/` — shell lifecycle, terminal model, input, and rendering behavior.
6. `app/src/ai/` — Agent Mode, conversations, tools, indexing, and orchestration.
7. `crates/warpui_core/src/core/` — entity handles, application context, windows, and actions.
8. `crates/warp_tui/src/session.rs` and `crates/warp_tui/src/root_view.rs` — headless front-end lifecycle.
9. `crates/warp_core/src/channel/` and `crates/warp_core/src/paths.rs` — build-channel configuration and storage isolation.
10. `crates/integration/` — executable behavioral specifications for the desktop app.
