# Practical change map

Use this as a starting point, then trace types and tests from the named composition point. Several directories are large; do not assume similarly named GUI and TUI views share rendering code.

## If I need to change startup or runtime behavior

**Look first at:**

- `app/src/lib.rs` for argument dispatch, `LaunchMode`, dependency registration, and mode-specific behavior.
- `app/src/bin/` or `crates/warp_tui/src/bin/` for channel-only behavior.
- `crates/warp_tui/src/session.rs` for TUI parsing, mounting, login gating, and resume behavior.
- `app/src/root_view.rs` and `app/src/workspace/` for desktop window/workspace creation.

**Caveats:** Preserve initialization ordering: settings and secure storage precede auth; auth precedes server clients; foundational models must exist before subscribers. Headless modes must not start fixed-port GUI services or write into GUI/TUI-incompatible persistence scopes.

## If I need to change configuration

**Look first at:**

- `crates/settings/src/` for storage, schema, surface, privacy, and synchronization rules.
- `app/src/settings/` for product-level typed settings and initialization.
- `crates/warp_core/src/channel/` for build/channel endpoints and integration availability.
- `crates/warp_core/src/paths.rs` for channel, data-profile, and surface path isolation.
- `crates/warp_cli/src/lib.rs` for CLI/environment inputs.

**Caveats:** Decide explicitly whether a setting is public/private, GUI/TUI/both, and cloud-synced/local. New toggleable settings also need Command Palette discoverability and context flags. Release channels intentionally reject service URL overrides.

## If I need to change terminal or shell behavior

**Look first at:**

- `app/src/terminal/input.rs` and `app/src/terminal/view.rs` for app-level input/presentation behavior.
- `app/src/terminal/local_tty/` for shell spawning, worker processes, PTY I/O, and exported shell environment.
- `app/src/terminal/terminal_manager.rs` for session/model orchestration.
- `crates/warp_terminal/` for emulation and terminal-domain state.
- `crates/warp_tui/src/terminal_session_view.rs` for headless adaptation.

**Caveats:** Terminal model locks are non-reentrant and nested acquisition can freeze the app. Shell bootstrap behavior is platform- and shell-specific. A shared model change may require separate GUI and TUI verification.

## If I need to change agent behavior

**Look first at:**

- `app/src/ai/agent/` for agent lifecycle, messages, tools, permissions, and file edits.
- `app/src/ai/orchestration/` for multi-agent/worker selection and orchestration.
- `app/src/ai/agent_conversations_model.rs` for conversation collection/lifecycle.
- `crates/ai/` for reusable API, indexing, and project-context logic.
- `app/src/server/` and `crates/warp_server_client/` for authenticated transport.
- `crates/warp_tui/src/agent_block.rs` and related `tui_*` views for headless rendering.

**Caveats:** Separate model/provider selection, remote protocol, tool permission, filesystem mutation, and presentation concerns. Agent events may be consumed by desktop, CLI, and TUI surfaces, so a protocol/model change can have multiple renderers.

## If I need to change cloud synchronization or external services

**Look first at:**

- `app/src/server/` and `crates/warp_server_client/` for client construction and request policy.
- `crates/graphql/` and `crates/warp_graphql_schema/api/schema.graphql` for GraphQL contracts.
- `crates/cloud_objects/`, `crates/cloud_object_client/`, and `crates/cloud_object_persistence/` for synchronized object behavior.
- `app/src/drive/` for Drive-specific user behavior.
- `app/src/auth/` and `crates/firebase/` for authentication.
- `crates/mcp/` and `app/src/ai/mcp/` for MCP integrations.

**Caveats:** Schema edits may require code generation. Preserve auth, privacy, offline/cache, permission, and version-conflict behavior. Exact backend implementation and deployment are not in this client repository.

## If I need to change desktop UI, TUI, or assets

**Desktop GUI — look first at:** `app/src/root_view.rs`, the relevant feature view, `crates/ui_components/`, `crates/warpui/`, and `crates/warpui_core/src/elements.rs`.

**Headless TUI — look first at:** the relevant file in `crates/warp_tui/src/` and TUI elements under `crates/warpui_core/src/elements/tui/`.

**Assets — look first at:** `app/assets/`, `app/resources/`, `crates/warp_assets/`, and the relevant build/resource preparation scripts.

**Caveats:** The front-ends share state but not layout/rendering. GUI work needs real-window visual verification; TUI work needs render-to-lines tests and a real terminal run. Persistent mouse state handles must be owned rather than constructed during rendering.

## If I need to change tests or fixtures

**Look first at:**

- The adjacent `${module}_tests.rs` or `mod_test.rs` for unit-test conventions.
- `crates/integration/src/test/` for the GUI scenario implementation.
- `crates/integration/tests/integration/` for runner registration.
- `crates/integration/tests/data/` for persisted/session fixtures.
- `crates/warp_tui/src/test_fixtures.rs` and matching `*_tests.rs` for TUI rendering.

**Caveats:** Integration is GUI-only. Do not use it for TUI presentation. SQLite fixtures encode historical schema/state and should be changed deliberately rather than regenerated casually.
