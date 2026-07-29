# Configuration and environment

## Configuration layers

| Configuration | Location | Purpose and runtime effect |
|---|---|---|
| Cargo workspace/build features | `Cargo.toml`, `app/Cargo.toml`, crate manifests | Select platform integrations and capabilities such as GUI, TUI, local filesystem/TTY, crash reporting, preview channel, and integration-test helpers. |
| Channel configuration | `app/src/bin/`, `crates/warp_core/src/channel/`, generated internal configuration via `crates/warp_channel_config/` | Fixes app identity, service endpoints, feature cohorts, log names, telemetry, updates, crash reporting, and static MCP OAuth credentials. |
| Typed user settings | `crates/settings/src/`, `app/src/settings/` | Controls product behavior including appearance, input, terminal sessions, AI, privacy, SSH, panes, and synchronization. |
| Public settings file | Surface/channel-specific config directory, conventionally `settings.toml` | User-editable settings when the settings-file feature is active; watched and reloaded by the settings system. |
| Private preferences | Native/private settings backend | Holds settings explicitly marked private rather than exposing them in public TOML. |
| Secure storage | Platform keychain/credential store or protected fallback | Stores login and model-provider secrets. The TUI uses a separate service name; integration tests use a no-op implementation. |
| SQLite persistence | `crates/persistence/migrations/` and launch-mode state directories | Stores durable, noncritical state used for session restoration and cached cloud objects. App, TUI, and remote-daemon scopes are isolated. |
| Repository configuration | `.warp/`, `.mcp.json`, project rules/skills | Supplies repository workflows, agent instructions/skills, and MCP servers discovered while working in a project. |

## Important environment variables

This is the operational subset most relevant to understanding startup. The codebase defines additional internal, CI, shell-bootstrap, telemetry, and test variables.

| Variable | Required? / default | Effect |
|---|---|---|
| `WARP_API_KEY` | Optional; no default | Supplies non-interactive authentication to shared CLI modes and the TUI. It is a secret and should not be committed or logged. |
| `WARP_SERVER_ROOT_URL` | Optional; channel default | Overrides the main HTTP service only on channels that allow server redirection. The development script's `SERVER_ROOT_URL` input is mapped to this form. |
| `WARP_WS_SERVER_URL` | Optional; channel default | Overrides the real-time GraphQL/WebSocket endpoint on permitted development channels. `script/run` maps `WS_SERVER_URL` into it. |
| `WARP_SESSION_SHARING_SERVER_URL` | Optional; channel default | Overrides the session-sharing service on permitted development channels. |
| `WITH_LOCAL_SERVER` | Optional; unset | Development-script switch that selects localhost-oriented server defaults. It does not alter release binaries' override policy. |
| `SERVER_ROOT_URL` | Optional; `http://localhost:8080` when using the documented local-server flow | Input consumed by `script/run` for the local server's HTTP endpoint. |
| `WS_SERVER_URL` | Optional; `ws://localhost:8080/graphql/v2` in the documented default flow | Input consumed by `script/run` for local real-time GraphQL. |
| `WARP_DATA_PROFILE` | Optional; ignored in release builds | Adds a debug-only namespace suffix so parallel development instances isolate settings, state, and credentials. |
| `WARP_AGENT_CONFIG_FILE` | Optional; CLI-defined default behavior | Selects an agent CLI configuration file. |
| `WARP_OUTPUT_FORMAT` | Optional; CLI default | Selects structured versus human-facing output for supported commands. |
| `WARP_CLI_MODE` | Optional; unset | Makes the shared binary behave as a standalone CLI and print CLI help rather than opening the GUI when no command is supplied. |
| `OZ_RUN_ID` | Optional; unset | Associates supported agent/runner activity with an Oz ambient-agent task. Invalid identifiers are treated as absent. |
| `WARP_CLOUD_MODE_DEFAULT_HOST` | Optional; Warp-hosted/default selection | Development/user override for the default execution host used by cloud/agent orchestration flows. |
| `WITH_SANDBOX_TELEMETRY` | Optional; unset | Local-channel switch enabling sandbox telemetry through an additional feature flag. |
| `GITHUB_ACTIONS` | CI-provided | Selects CI behavior and identifies command-line agents started in GitHub Actions. |

## Channel defaults and consequences

The OSS binary constructs a public configuration in source:

- main service: `https://app.warp.dev`;
- real-time service: `wss://rtc.app.warp.dev/graphql/v2`;
- session sharing: `wss://sessions.app.warp.dev`;
- Oz: `https://oz.warp.dev`;
- telemetry, crash reporting, and autoupdate: disabled because their optional configurations are absent.

Stable, Preview, Dev, and Local wrappers load channel data through `warp_channel_config`. The generated values are not all present in this checkout, so those exact endpoints and credentials should be treated as build inputs rather than duplicated in documentation.

## Settings and path isolation

- Stable/Preview, Dev, Local, Integration, and OSS use distinct application IDs or directory suffixes.
- Debug-only `WARP_DATA_PROFILE` adds another suffix for isolated development instances.
- Portable Warp-authored configuration lives under a channel-specific `.warp*` home directory; platform-native state/config locations are used where appropriate.
- The TUI uses a `.warp_cli*` directory on macOS or a `cli` subdirectory on other platforms, plus its own MCP file, secure-storage suffix, and database.
- TUI settings remain local and do not participate in GUI cloud-settings synchronization.

## Local, test, and production examples

```bash
# Normal OSS/local development selection
./script/run

# Local server with documented defaults
WITH_LOCAL_SERVER=1 ./script/run

# Local server with explicit HTTP and real-time endpoints
WITH_LOCAL_SERVER=1 \
  SERVER_ROOT_URL=http://localhost:8082 \
  WS_SERVER_URL=ws://localhost:8082/graphql/v2 \
  ./script/run

# Isolate a debug instance's local data
WARP_DATA_PROFILE=repo-understanding ./script/run

# Non-interactive TUI authentication; provide the value through secret management
WARP_API_KEY='<secret>' ./script/run-tui

# Main workspace test suite
cargo nextest run --no-fail-fast --workspace --exclude command-signatures-v2
```

## Credentials and secrets

Expected secret-bearing configuration includes Warp API keys, login tokens, provider API keys, MCP OAuth client secrets, Firebase/channel configuration, and optional telemetry/crash-reporting credentials.

Runtime code routes user tokens through secure storage or explicit process input. Internal channel configuration is generated separately; do not copy credentials from generated artifacts into settings, fixtures, logs, or documentation.
