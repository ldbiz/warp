# Agent request path

## From AI route to conversation

On AI submission, the terminal controller does not write the prompt as shell bytes. It activates the graphical agent surface and uses the active conversation when appropriate or creates/forks one through the conversation/history models. Conversation selection is explicit state (`AIConversationId` plus optional `ServerConversationToken`); tests cover selected/open-elsewhere/available entries. Follow-up input remains associated with the current conversation rather than being reinterpreted as a new shell block.

The client constructs `RequestInput`/`RequestParams`. Inputs can include the typed prompt and explicit attachments/chips. `SessionContext` provides terminal/session facts such as current working directory, shell/session type and relevant block context. MCP tool/resource descriptions, memory/Drive toggles, model IDs, execution profile, permissions and supported-tool capabilities are added automatically from settings. Files/images and selected blocks are explicit attachments; a tool can later request file reads. Git/file/project context may be derived locally by context models or requested via tools, but the absent service may also augment context, so it is unsafe to claim every context field is automatic.

## Desktop-to-Warp boundary

`agent::api::generate_multi_agent_output` converts the local inputs, optionally redacts detected secrets, and builds `warp_multi_agent_api::Request`. Its settings contain base/CLI/computer-use model IDs, context/tool capabilities, optional provider keys/custom provider definitions, and `allow_use_of_warp_credits`. It then calls `ServerApi::generate_multi_agent_output`. That is the confirmed remote boundary. There is no provider HTTP client in this interactive request path.

The remote implementation—model selection behind Auto, provider request translation, model call, server context, billing and orchestration—is not in this repository.

## Streaming back

The server API returns `ResponseStream<Item = ResponseEvent>`. `ResponseStreamModel` owns cancellation/retry identity and forwards events. Controller code hydrates streamed conversation messages, suggestions, task state and client actions into local blocks. A conversation token links later turns. Error/retry code can resume a stream, but provider-level retry/failover policy is not visible.

## Tool/action loop

1. The remote stream emits an agent action with an action ID/type and arguments.
2. The desktop action model preprocesses it and determines whether it is automatically executable, requires a permission UI, or is denied.
3. Local executors implement shell command/output, file reads/search/glob/edits, MCP calls/resources, questions, computer use (when supported), and related actions. Tool availability is advertised in each request.
4. Proposed shell commands are rendered in the agent block. `BlocklistAIPermissions` combines execution profile, autonomy, read-only/risky metadata and per-terminal/workspace decisions. “Always allow”, ask, and deny states are separate from classification.
5. When accepted, `ShellCommandExecutor` emits `ExecuteCommand`; terminal wiring inserts it into the active block/session and submits it to the configured shell. It observes shell-integration completion, captures formatted output/exit/timestamps, and forms `AIAgentActionResultType`.
6. That result is sent in the continuing agent conversation/stream protocol. Long-running commands support reads, PTY writes and explicit `TransferShellCommandControlToUser`; handback uses a oneshot signal.

Thus an AI suggestion does not itself execute. Control is explicit: user approval or policy -> local executor -> shell; the agent regains structured results, while transfer actions can leave the user controlling the live process.

## Automatic versus explicit context

| Context | Desktop behavior |
|---|---|
| Prompt/model/conversation token | Automatic from submission/current selection. |
| CWD, shell and session type | Automatic `SessionContext` where available. |
| Applicable terminal/block context | Gathered by context/request models; exact selection depends on entry point. |
| Attached files/images/blocks | Explicit user attachment; bytes/content may become request input. Attachments lock AI route. |
| File contents | Explicit attachment or local read tool result; not all workspace files are sent automatically by the shown request constructor. |
| Git/project information | Context models/tools can collect it; exact server augmentation is unavailable. |
| MCP catalog | Active server tool/resource descriptors are automatically advertised; invocation is a later action. |
| Memory/Drive | Controlled by settings and represented as request flags/context. |
| Terminal command results | Local executor returns them after an agent action. |
