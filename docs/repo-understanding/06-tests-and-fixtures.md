# Tests and fixtures

## Test structure

| Layer | Framework / location | What it verifies |
|---|---|---|
| Rust unit tests | Built-in Rust test harness; adjacent `*_tests.rs` or `mod_test.rs` files | Pure models, parsers, state transitions, settings, protocol conversion, rendering calculations, and platform-independent subsystem contracts. |
| TUI render/model tests | Rust tests under `crates/warp_tui/src/*_tests.rs` with shared `test_fixtures.rs` | Exact cell-grid output, layout, menus, keyboard interaction, sessions, agent/tool blocks, permissions, diffs, and transcript behavior. |
| Desktop integration tests | Custom Builder/TestStep framework in `crates/integration/src/`; scenarios in `crates/integration/tests/` | Real-window GUI, terminal, shell, restoration, settings, workspace, notebook, code-review, and Agent Mode behavior. |
| Schema/code-generation tests | `crates/warp_graphql_schema/`, settings schema tests, generated-settings binary tests | Source schemas remain compatible with generated clients and public settings shape. |
| Documentation tests | Rustdoc examples across workspace crates | Public/internal documented APIs compile and behave as asserted. |

The repository convention keeps unit tests in separate files and includes them at the end of the corresponding module with `#[cfg(test)]` and `#[path = "..."]`. This preserves readable implementation modules while allowing tests access to private details.

## Canonical commands

```bash
# Full local presubmit: formatting, linting, native/shader checks, tests and docs
./script/presubmit

# Main workspace tests
cargo nextest run --no-fail-fast --workspace --exclude command-signatures-v2

# Completer's alternate v2 feature set
cargo nextest run -p warp_completer --features v2

# Rust documentation tests
cargo test --doc

# Focused package while developing
cargo nextest run -p <package-name>
```

Nextest marks tests over 30 seconds as slow and terminates them after twice that interval. GUI integration tests consume two scheduler threads; CI retries them twice to reduce known flake impact and emits JUnit output.

## Important executable specifications

### Application and terminal

- `crates/integration/src/test/bootstrapping.rs`, `input.rs`, `history.rs`, `subshell.rs`, and `keyboard_protocol.rs` document shell/PTY integration.
- `session_restoration.rs`, `pane_restoration.rs`, `workspace.rs`, and `notebooks.rs` document durable UI/session reconstruction.
- `settings_file_hot_reload.rs`, `settings_file_migration.rs`, `settings_file_errors.rs`, and `settings_private.rs` document the split settings backends.
- `app/src/terminal/**/*_tests.rs` covers lower-level terminal construction, models, input, and platform behavior.

### Agents and code

- `crates/integration/src/test/agent_mode.rs` and `ai_assistant.rs` exercise desktop agent flows.
- `app/src/ai/**/*_tests.rs` documents conversation persistence, provider routing, context, credentials, codebase indexing, and orchestration submodels.
- `crates/warp_tui/src/agent_block_tests.rs`, `tui_permission_prompt_tests.rs`, `tui_file_edits_view_tests.rs`, and related files document terminal-native agent presentation.
- `crates/integration/src/test/code_review.rs`, `file_tree.rs`, and `goto_line.rs` cover coding surfaces.

### Cloud and configuration

- `crates/graphql/src/api/*_tests.rs` checks GraphQL operation models and conversions.
- Tests in cloud-object crates cover domain merging, client behavior, and persistence.
- `app/src/drive/*_tests.rs` checks Drive indexes, panels, and export behavior.
- `crates/warp_core/src/channel/state_tests.rs` and `paths_tests.rs` check endpoint/channel state and namespace paths.

## Fixtures and sample data

`crates/integration/tests/data/` contains curated SQLite databases and small external files. The database fixtures represent states that are difficult or slow to construct through UI actions:

- restored command blocks, background blocks, tabs, code panes, notebooks, workflows, and settings;
- deleted working directories, duplicate shareable IDs, JSON cloud objects, and small-window layouts;
- launch configurations, themes, workflows, Markdown/Rust files, and shell scripts for rendering/execution tests.

`crates/integration/assets/` contains helper programs that report terminal keyboard sequences and event modes. These let integration tests distinguish terminal protocol behavior from visual assertions.

`crates/editor/test_fixtures/` supplies editor-specific documents. Many other unit tests construct temporary directories, in-memory models, mocked clients, or inline snapshots rather than depending on global fixture directories.

## Practical test selection

- Change a pure crate/model: run its package tests first.
- Change GUI terminal/workspace behavior: run focused unit tests and the relevant custom integration scenario, then the workspace suite.
- Change TUI presentation/input: run the corresponding `warp_tui` tests; these are intentionally not GUI integration tests.
- Change GraphQL/settings schema: run generator/schema tests and inspect generated-file diffs.
- Change platform-native behavior: local tests cover only the host platform; use the repository's cross-platform verification workflow when the change can differ by OS.

## Visible coverage gaps and limits

- The complete desktop GUI requires a real display and native services; unit tests cannot validate GPU output or all OS integrations.
- Platform-specific shell, keychain, update, windowing, and packaging behavior cannot be proven from one host.
- Cloud end-to-end behavior depends on services outside this repository; many tests instead use mocks or persisted fixtures.
- Runtime feature-flag combinations are too numerous for every combination to receive integration coverage.
- TUI tests strongly cover deterministic rendering, but real terminal capabilities, escape-sequence compatibility, and host theme probing still need runtime verification.
