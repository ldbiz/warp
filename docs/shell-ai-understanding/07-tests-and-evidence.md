# Tests and evidence

## Implementation tests

| Test/source | What it establishes |
|---|---|
| `crates/input_classifier/src/heuristic_classifier/mod_tests.rs` | Shell keywords, natural-language words and heuristic fallback sources/outputs. |
| `crates/input_classifier/src/onnx/mod_tests.rs` | ONNX failure preserves current input and reports fallback source. |
| `crates/input_classifier/src/parser_tests.rs`, `util_tests.rs` | Token preprocessing and allowlist/shell-likelihood utilities. |
| `app/src/ai/blocklist/input_model_tests.rs` | history/prompt matching, recency resolution, config transitions and detection sources. |
| `app/src/ai/blocklist/agent_view/gui_input_mode_policy.rs` tests/callers | GUI lock/unlock policy and agent-view restrictions. |
| GUI integration tests under `crates/integration/src/test/agent_mode.rs`, `input.rs`, `history.rs` | End-to-end input/mode/history intent, with documented PowerShell skips where applicable. |
| `crates/ai/src/api_keys_tests.rs` | secure-storage serialization/reload, provider keys/custom endpoint request conversion. |
| `app/src/settings/ai_tests.rs` | AI settings state and API-key/model controls where covered. |
| `app/src/ai/llms_tests.rs` | custom endpoint entries, aliases/config keys, eligibility and picker merging. |
| `app/src/ai/agent/conversation_tests.rs` | request/usage handling for custom endpoints and conversations. |
| `app/src/ai/blocklist/permissions_tests.rs` | autoexecution/deny/write-to-PTY behavior, including PowerShell continuation cases. |
| `action_model/execute/shell_command_tests.rs` and `execute_tests.rs` | local agent command execution/results and shell-specific quoting. |
| `agent_view/conversation_selection_tests.rs` | current/open/available conversation selection states. |
| `controller/response_stream_tests.rs` | streamed event processing/retry behavior. |

Tests are evidence of intended behavior; only production feature/build files prove which classifier is packaged.

## Build-script evidence

* `script/windows/bundle.ps1:119` appends `nld_classifier_v3,nld_heuristic_v2`.
* `app/Cargo.toml` forwards those features to `input_classifier`.
* `crates/input_classifier/Cargo.toml` maps V3 to `onnx_candle`.
* `crates/input_classifier/src/onnx/mod.rs` conditionally embeds tokenizer and V3 model.
* `app/src/input_classifier.rs` selects V3 and falls back to heuristics.

## Documentation/schema evidence (lower tier)

* GraphQL schema fields `byokTokenUsage`, `warpTokenUsage`, workspace managed BYOK policy and managed-secret types describe contracts, not their server implementation.
* UI text naming OpenRouter/LiteLLM or fallback is product/documentation evidence only.
* Bootstrap scripts establish intended shell integration; real behavior depends on successful injection and shell version.

## Checks performed for this analysis

No expensive test suite or network/account test was run: this patch changes Markdown only, and the task explicitly requests evidence identification rather than modifying behavior. Static checks should verify scope, links/Mermaid fences, and production flags.

## Focused Windows Free-account manual matrix (do not automate billing)

| Scenario | Observation |
|---|---|
| Fresh PowerShell tab; type `cat myfile` slowly | Mode on each prefix, final Shell route, no network before Enter. (`cat` may be an alias in PowerShell; record actual resolution.) |
| Type `what is in myfile` slowly | Prefix transitions, final AI route; disconnect network before typing to prove local classification, then submit to observe remote failure. |
| Toggle/lock Shell and AI using displayed Windows shortcuts | Mode icon/label, lock persistence, post-submit unlock behavior. |
| Configure denylisted token and repeat | Immediate Shell classification and telemetry (without collecting secret text). |
| Re-enter successful command and failed command | History match includes success and excludes command-not-found. |
| AI follow-up after an AI block | Conversation reuse and follow-up allowlist. |
| Repeat in cmd, WSL and Git Bash | ConPTY launch, syntax owner, path conversion, block metadata quality. |
| Configure OpenAI/Anthropic/Google/OpenRouter key one at a time | Secure-store persistence, available model list, outbound host/body redacted trace. |
| Configure LiteLLM/OpenAI-compatible localhost endpoint | Determine whether request is direct or Warp-mediated and whether localhost is reachable; never use a real production secret in capture. |
| Disable Warp credit fallback and force provider error | Record behavior only; confirm billing with official account records, not UI inference. |
| Agent proposes safe/risky/long-running commands | Ask/allow/deny UI, local ConPTY execution, transfer and result return. |

Capture network destinations and certificate peers with redaction. Do not publish keys, prompts containing secrets, or non-public service internals.
