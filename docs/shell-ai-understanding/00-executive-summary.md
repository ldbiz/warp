# Windows shell/AI routing: executive summary

## Direct answers

| Question | Repository-supported answer |
|---|---|
| Main Windows application | The release target is the **graphical `app` crate**, not `warp_tui`. `script/windows/bundle.ps1` builds `warp` with GUI/release features. |
| Is classification local? | Yes. `BlocklistAIInputModel::detect_and_set_input_type` invokes the in-process `InputClassifier`; the ONNX runner consumes embedded bytes. No HTTP client occurs in this path. |
| Production Windows classifier | `script/windows/bundle.ps1` enables `nld_classifier_v3,nld_heuristic_v2`; therefore `InputClassifierModel::new` first constructs `OnnxClassifier(BertTinyV3)` using Candle. If construction fails it uses `HeuristicClassifier`; an ONNX panic also permanently falls back to heuristics. |
| Independent from inference? | Yes. Natural-language detection is local and can select AI even if authentication/network inference later fails. |
| Ordinary commands | Warp sends the editor bytes/newline through `ActiveSession`/terminal model to the PTY. On Windows the PTY is ConPTY; the configured PowerShell, cmd/WSL/MSYS2 shell process ultimately interprets syntax. |
| AI requests | The desktop assembles a `warp_multi_agent_api::Request`, including model/context/tool capabilities and optional credentials, and sends it through `ServerApi::generate_multi_agent_output` to Warp's remote agent endpoint. |
| Harness location | The interactive Warp Agent orchestration visible here is split: request/context, streamed-event handling, permissions, and local tools are in the desktop; model routing and generation behind the Warp endpoint are absent. Do not describe it as wholly local or wholly remote. |
| BYOK/custom billing | Not proven. The client sends optional key/custom-provider data and `allow_use_of_warp_credits`; schemas expose usage categories. Pricing, entitlement, charging, and fallback enforcement are server-side unknowns. |

## Architecture in one paragraph

Keystrokes update the graphical input editor and repeatedly schedule an abortable asynchronous **local** classification. Locked Shell/AI state and explicit mode actions bypass that work. Submission reads the selected `InputType`: Shell writes to the live terminal session; AI selects/creates a conversation, forms a remote agent request, and consumes a stream of messages/actions. Agent tool actions are dispatched back into desktop executors; shell actions pass through a separate permission policy before being written to the same terminal. Provider keys and custom endpoint definitions are stored in OS secure storage but serialized into the request to Warp, so this repository does **not** establish direct desktop-to-provider networking.

## Five findings to retain

1. Classification and model inference are separate systems and separate trust boundaries.
2. Windows production embeds `bert_tiny_v3.onnx` plus `bert_tiny_tokenizer.json`; it does not bundle all three model versions into that feature build.
3. `cat myfile` reaches shell heuristics/resolution (and commonly the ONNX model only if earlier signals do not settle it); `what is in myfile` lacks a resolved command signal and is selected as AI by the local classifier.
4. User-typed shell commands and agent-requested shell commands share terminal transport but not policy: agent commands have permission/autoexecution checks.
5. BYOK credentials are included in a Warp agent request when enabled; no code here proves provider-direct transport or billing semantics.

## Read these files first

* `app/src/ai/blocklist/input_model.rs` — state, contextual signals, async classification.
* `app/src/input_classifier.rs` and `crates/input_classifier/src/onnx/mod.rs` — production construction, embedding, heuristics/fallback.
* `app/src/terminal/view/input.rs` and `app/src/terminal/model/session/active_session.rs` — submission boundary.
* `app/src/ai/agent/api.rs` and `app/src/ai/agent/api/impl.rs` — request construction and Warp boundary.
* `crates/ai/src/api_keys.rs` — secure storage and credential/custom-provider wire conversion.
* `app/src/ai/blocklist/action_model/execute/shell_command.rs` and `permissions.rs` — agent-to-local-shell control.
* `script/windows/bundle.ps1` — production classifier feature evidence.

## Claims needing runtime or external confirmation

The exact shipped key bindings after remote configuration, model catalog and Auto choices, endpoint host/certificate behavior, whether the service calls providers directly or proxies/transforms requests, server context augmentation, retention, entitlements, credits and pricing all require a production Windows trace and current official/service documentation. See [caveats](09-caveats-and-unavailable-backends.md).
