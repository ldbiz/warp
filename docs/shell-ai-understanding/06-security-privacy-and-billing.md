# Security, privacy, network, and billing boundaries

## Network boundaries

Local classification—including parsed tokens, history matching, alias expansion, heuristics and embedded ONNX—is local-only in executable code. An AI submission crosses to Warp through `ServerApi`. The desktop request may contain provider keys/custom endpoint keys; therefore BYOK credentials **pass through Warp's request infrastructure** in this implementation. No direct interactive desktop-to-OpenAI/Anthropic/Google/custom endpoint call was found. What Warp's service does next is unavailable.

OS “secure storage” is the confirmed at-rest abstraction (`warpui_extras::secure_storage`); the Windows backing implementation should be audited/runtime-tested for the exact Credential Manager/DPAPI location. The repository intentionally avoids plaintext settings storage for `AiApiKeys` and `GrokOAuthTokens`, but secrets are materialized in memory and serialized into requests.

## Data-flow table

| Data | Local only | Potentially transmitted to Warp | Potentially transmitted to provider | Setting/control | Repository limit |
|---|---|---|---|---|---|
| Typed Shell input | Yes for classification and direct PTY execution. | Telemetry may include classification text only under AI UGC telemetry policy; shell/cloud features outside this path not assessed. | Not by shell submission code. | Telemetry/privacy settings. | Provider/network behavior of invoked shell programs is outside Warp. |
| AI prompt | No. | Yes, request input. | Potentially, via absent inference service. | User chooses AI; secret-redaction/privacy controls. | Provider payload/retention unknown. |
| Command history | Local classification reads it. | Selected context/history/action results may be sent; not proven that full history is always sent. | Potentially if included by service. | Context/privacy features. | Server context augmentation unknown. |
| Terminal output | Locally rendered/captured. | Explicit block attachment and agent tool command results can be sent. | Potentially through model request. | Approval/context selection. | Full automatic selection unknown. |
| Current directory | Local session state. | `SessionContext` can include it. | Potentially. | AI submission/context. | Server transformations unknown. |
| Git context | Locally collectable by context/tools. | Potentially when gathered. | Potentially. | Context/tool request and permission. | Exact automatic set unknown. |
| File contents | Local until attached/read. | Yes when attachment or file-read/tool result is included. | Potentially. | Explicit attachment and tool permission/profile. | Server retention unknown. |
| Environment variables | Local shell owns them. | Tool/process context may expose selected values; secrets can be redacted. No evidence all env is sent by default. | Potentially if included. | Secret obfuscation, tool approval. | Exact service context unknown. |
| Provider API keys | Stored locally in secure storage. | Yes when entitlement enabled and used in request settings. | Presumably needed by provider, but service implementation is absent. | BYOK/custom inference entitlement and user setup. | Handling/retention after Warp receipt unknown. |
| Tool results | Produced locally for local tools. | Yes, returned to conversation. | Potentially in subsequent model context. | Tool permissions/autonomy. | Server orchestration unknown. |
| Telemetry | Created locally. | Yes when telemetry is enabled; classification text requires UGC telemetry predicate. | No provider telemetry path confirmed. | `PrivacySettings` and UGC policy. | Backend retention/aggregation unknown. |

## Telemetry relevant to this flow

Executable call sites include `AgentModeChangedInputType` (manual/autodetected, route, buffer length, optional input), agent request/error identifiers and entrypoint/model-related fields, `AutoexecutedAgentModeRequestedCommand`, CLI subagent action execution, and permission/CTA events. Tool-specific executors add action telemetry. Event existence proves emission attempts and fields, not backend retention or dashboards. `should_collect_ai_ugc_telemetry` plus telemetry/privacy settings gates raw classification input; length/type can still be recorded.

## Permission boundary

Direct user Shell commands are intentional terminal input and do not go through agent command approval. Agent-requested shell/file/MCP/computer-use actions do. Supervised/autonomous mode, execution profiles, read-only/risky classification, write-to-PTY policy and per-action decisions determine autoexecute/ask/deny. These are local enforcement points even though action proposals arrive remotely.

## Billing and credits

Locally confirmed:

* the request distinguishes optional BYOK settings from Warp model IDs;
* the client sends `allow_use_of_warp_credits` (derived from `can_use_warp_credits_for_fallback`);
* GraphQL/config types expose Warp-versus-BYOK token-usage categories;
* workspace/account state gates configuration use and UI.

Not locally confirmed: prices, Free allowances, whether a failed BYOK call consumes credits, fallback order, account entitlement evaluation, provider charges, credit accounting, or enforcement. Labels such as “credits”, “Free”, and “fallback” are not billing implementation.
