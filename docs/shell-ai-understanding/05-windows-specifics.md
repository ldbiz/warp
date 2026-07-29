# Windows-specific implementation

## Production target and bundle

The Windows release is the graphical `warp` binary from the `app` package. `script/windows/bundle.ps1` drives the release build and adds `nld_classifier_v3,nld_heuristic_v2` to Cargo features (along with the bundle's other features). This is direct packaging evidence that Windows production uses the GUI classifier path, not `./script/run-tui`.

The feature chain is:

`app/nld_classifier_v3 -> input_classifier/nld_classifier_v3 -> onnx_candle -> onnx + Candle`.

`RustEmbed` includes `bert_tiny_tokenizer.json` and, under this feature, `bert_tiny_v3.onnx`. V1/V2 assets exist in the source tree but their `include` attributes are not active in this production feature selection. If V3 initialization fails, `InputClassifierModel` uses `HeuristicClassifier`; `nld_heuristic_v2` also controls current heuristic data/behavior.

## Process and shell launch

Windows local terminal creation uses the Windows PTY implementation around ConPTY and launches the resolved shell/startup program as its child. The repository models materially different launch environments:

* native PowerShell/PowerShell Core, with `pwsh.ps1`/`pwsh_init_shell.ps1` integration;
* Command Prompt/native executable launching (with less shell-hook richness);
* WSL, including distribution identity and Windows/WSL path conversion;
* MSYS2/Git Bash, including MSYS root/path conversion.

All remain shell processes behind the pseudoterminal. Classifier parsing is not substituted for their syntax parsers. Agent file tools use `ai::paths` to convert paths according to `ShellLaunchData`; agent shell commands use PowerShell-specific quoting/pager decoration where necessary.

## Mode switching on Windows

The GUI dispatches typed actions for toggling/setting Shell or AI and renders their active bindings. Windows normally uses Control-oriented bindings where macOS uses Command, but checked-in keymaps, user remapping, remote flags and view context can change the actual chord. Prefix/shortcut tests establish bypass semantics; a production runtime check should establish the displayed Windows chords rather than hard-coding one here.

## Material platform differences

| Area | Windows consequence |
|---|---|
| Classifier | Same local pipeline as GUI macOS/Linux; Windows bundle explicitly selects V3+Candle. |
| PTY | ConPTY rather than Unix PTY. |
| Shell syntax | PowerShell/cmd syntax differs; WSL and Git Bash reintroduce POSIX/MSYS semantics behind ConPTY. |
| Integration metadata | PowerShell bootstrap is provided; cmd/unsupported shells may provide less complete block metadata. |
| Paths/tools | Native, WSL and MSYS path translation is required for file tools and attachments. |
| Agent command decoration | PowerShell uses `Out-Host` and PowerShell escaping; bash/zsh/fish use their own pager forms. |
| TUI | `warp_tui` may share API/key/classifier abstractions, but is not the Windows graphical release path described here. |

## Evidence locations

* `script/windows/bundle.ps1` (`$FEATURES`, cargo build/bundle invocation).
* `app/Cargo.toml` and `crates/input_classifier/Cargo.toml` (feature chain).
* `crates/input_classifier/src/onnx/mod.rs` (`RustEmbed`, `Model`).
* `app/assets/bundled/bootstrap/pwsh*.ps1` (PowerShell integration).
* Windows PTY modules under terminal/process crates; search `Conpty`, `PseudoConsole`, and `ShellLaunchData`.
* `crates/ai/src/paths.rs` (WSL/MSYS/native conversion).
