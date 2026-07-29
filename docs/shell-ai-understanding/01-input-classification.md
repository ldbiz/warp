# Input classification

## State model and entry point

The graphical terminal editor is the first relevant application boundary. Editor-change handling obtains a `ParsedTokensSnapshot` from the completer and calls `BlocklistAIInputModel::detect_and_set_input_type`; submission consults the same model. The model is terminal-surface-scoped rather than global.

`InputConfig` is the actual state pair:

* `input_type: InputType::{Shell, AI}` selects the route;
* `is_locked` says whether automatic detection may change it.

Thus “automatic mode” is not a third input type: it is Shell or AI with `is_locked == false`. `InputMode::new(protocol_input_type, is_locked)` mirrors it for session sharing. The default is Shell and locked when the applicable auto-detection setting is off. `InputModePolicy` separates shared state machinery from graphical context decisions; `GuiInputModePolicy` allows locked AI only in appropriate agent/rich-input views.

## Per-keystroke and submission timing

Classification runs while editing, not only after Enter. Each change can start `detect_and_set_input_type`. It aborts the prior handle, applies immediate guards/denylist handling synchronously, then `ctx.spawn`s history matching, alias expansion, and classifier invocation asynchronously. Yield points allow newer keystrokes or an explicit lock to cancel stale work. The ONNX calculation itself is CPU-local and synchronous inside that async task. Submission uses the state already selected; `handle_input_buffer_submitted` resets/unlocks policy for the next buffer. Empty input preserves the current type.

There is no network request in this pipeline. “Asynchronous” here prevents blocking/stale UI work; it does not imply cloud classification.

## Decision pipeline (in order)

1. **Guards:** `AgentMode` and contextual auto-detection settings must be enabled; the mode must be unlocked; active/pending agent terminal use and pending attachments disable detection. A short interval after explicit changes suppresses it.
2. **Parse:** the caller supplies `warp_completer::ParsedTokensSnapshot` (raw buffer plus parsed shell tokens). No first parsed token means no decision.
3. **Configured command denylist:** an exact first-token match in `AISettings.autodetection_command_denylist` forces Shell before history parsing.
4. **History:** successful/not-command-not-found session commands are fuzzy matched newest-first. With `NldPromptHistoryMatch`, agent prompts are also matched; the newer/better applicable match selects Shell or AI.
5. **Conversation continuity:** a small natural-language follow-up allowlist after an AI block selects AI. A one-word natural-language allowlist can retain AI.
6. **Aliases:** `warp_completer::util::expand_aliases` uses the live `CompletionContext` before generic classification.
7. **Classifier:** `InputClassifier::detect_input_type(snapshot, Context { current_input_type, is_agent_follow_up })` applies shell/NL heuristics, then ONNX or heuristic scoring.
8. **Commit:** only the still-current, still-unlocked task updates `InputConfig`; a telemetry event records a changed type and only includes text when AI UGC telemetry policy permits.

Command resolution enters `is_likely_shell_command` through completer metadata. Operators, assignments, paths/known executable shapes, parsed command structure, the shell keyword allowlist, and the proportion of natural-language tokens can settle Shell before ML. The classifier does not execute the command to test it. Failed `command not found` history entries are deliberately excluded.

## Classifier construction and fallback

`InputClassifierModel::new` is a singleton. Feature priority is V1, then V2, then V3 in source; a normal production build enables one. Windows bundling enables V3, causing `OnnxClassifier::new(BertTinyV3)`. `RustEmbed` includes the shared tokenizer and only the model selected by `cfg_attr`. `input_classifier`'s `nld_classifier_v3` feature selects `onnx_candle`.

`OnnxClassifier::detect_input_type` first applies one-word NL and shell-keyword checks and `is_likely_shell_command`; otherwise it tokenizes and runs embedded ONNX inference. `p_shell > p_ai` selects Shell; ties select AI. A construction failure causes the app-level `HeuristicClassifier`. A caught ONNX panic marks that instance and all subsequent classifications use heuristic scoring. An ordinary inference error preserves `Context.current_input_type`, which is the uncertainty/failure behavior—not a network retry.

## Explicit overrides and locking

Mode-switch actions/shortcuts call `set_input_config`, temporarily suppress auto-detection, and can lock the selected route. Agent view/rich input, attachments, history selection, and agent control also impose policy-specific locks. Prefix processing and actions are covered by `input_model_tests.rs`; explicit forcing is authoritative, so it bypasses ML rather than adding a classifier feature. The exact Windows accelerator shown to users comes from the active keymap (see [Windows specifics](05-windows-specifics.md)), and must be runtime-verified because keymaps/config can override defaults.

Before submission, Shell versus AI is visible through the input-mode affordance/branding and agent input presentation driven by `InputTypeChanged`/`LockChanged`; lock state controls its interaction. This is route preview, not a server acknowledgment.

## Representative evidence-backed inputs

| Input | Signals/code path | Expected route/boundary |
|---|---|---|
| `cat myfile` | Parsed first command, `cat` shell keyword/resolution, `is_likely_shell_command`; aliases are expanded first. | Shell locally; exact result can be strengthened by history. |
| `what is in myfile` | Multiple natural-language words, no resolved shell command; reaches V3 if no earlier history/follow-up decision. | AI locally, then remote only after submission. |
| A configured denylisted first token | Exact `autodetection_command_denylist` match. | Shell without history/ONNX. |
| A recently successful command | `most_recent_close_match` against session history. | Shell. |
| A recent agent prompt (flag enabled) | Prompt-history match, reconciled against command match. | AI. |
| A recognized short follow-up after an AI block | `is_agent_follow_up_input`. | AI. |
| Empty buffer | No first parsed token. | Preserve current route. |
| ONNX inference error | `InputClassifierFallbackCurrentInput`. | Preserve current route. |
| ONNX panic | One-time panic marker, then heuristic classifier. | Heuristic result, locally. |

These are pipeline expectations, not a promise that arbitrary similarly worded strings receive the same model score.
