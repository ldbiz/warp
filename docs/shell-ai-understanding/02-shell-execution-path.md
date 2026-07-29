# Shell execution path on Windows

## User-typed command

1. The graphical input editor owns the buffer and `BlocklistAIInputModel` supplies the selected `InputType`.
2. On Shell submission, terminal view/input handling records submitted input and sends its encoded bytes plus carriage return/newline to the active session/PTY controller. This is distinct from the AI request constructor.
3. `ActiveSession` and the writeable-PTY event loop transport bytes. `TerminalModel` consumes bytes read back from the PTY, applies terminal escape sequences to its grid, and correlates shell-integration metadata with blocks.
4. On Windows, the local session is backed by the platform pseudoterminal implementation (ConPTY). ConPTY connects the GUI terminal to the configured child process.
5. **The configured shell process interprets syntax.** Warp parses typed text for completion, classification, blocks and safety-related presentation; it does not replace PowerShell/cmd/bash parsing for ordinary execution.

The important boundary is therefore:

`Warp editor/classifier -> byte stream -> ConPTY -> configured shell -> child programs`.

## Windows shell families

Shell launch data distinguishes native PowerShell, WSL and MSYS2/Git Bash environments; shell discovery/startup code chooses an executable and bootstrap appropriate to that family. PowerShell receives `app/assets/bundled/bootstrap/pwsh.ps1` integration. WSL launches into a distribution and uses POSIX paths inside it; MSYS2/Git Bash needs Windows/MSYS path conversion. Command Prompt has less rich shell integration than the explicitly bootstrapped shells. Other shells may run as terminal processes even when Warp cannot provide full block metadata/completions.

ConPTY is transport, not a shell. For WSL, the ConPTY child ultimately invokes WSL and the Linux shell interprets syntax; for Git Bash/MSYS2, its bash does; for PowerShell, PowerShell does; for cmd, `cmd.exe` does.

## Output, blocks, and history

The PTY reader feeds `TerminalModel`, whose parser updates the rendered grid. Shell bootstrap hooks report prompt/preexec/precmd metadata (command, CWD, start/completion, exit status); the block list turns that into a command block. History persistence uses captured command/exit metadata. Output without integration is still rendered as terminal bytes, but block/history fidelity can be lower. `BlockMetadataReceivedEvent` is also what agent command execution watches for completion.

## Interception and safety

For an ordinary user-typed Shell submission, Warp may:

* classify/force the route before submission;
* expand UI history/completions but not silently substitute classifier text;
* encode bracketed paste/newline and apply terminal input behavior;
* run explicit Warp workflow/command-palette actions or special integration handling;
* capture/redact presentation/history according to settings.

The agent `CommandExecutionPermission` checks are **not** a general confirmation gate for direct user commands. A user intentionally submitting Shell retains control; the configured shell is the execution authority. Product features may warn about pasted/multiline input or intercept explicit Warp commands, but this analysis found no universal risk confirmation replacing direct shell submission.

Agent-issued commands are different: `ShellCommandExecutor::should_autoexecute` checks read-only/risky metadata and permissions, may require confirmation, optionally decorates pager-producing commands, emits `ExecuteCommand`, and returns block output/exit status as an action result. See [agent path](03-agent-request-path.md).
