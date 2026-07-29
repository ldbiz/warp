# Caveats and unavailable backends

## What the repository cannot resolve

### Missing services

The repository contains client contracts and generated schemas, but not the Warp API implementation behind `ServerApi::generate_multi_agent_output`, nor the complete Oz/agent service that turns requests into provider calls. Consequently it cannot establish:

* provider HTTP URL, headers, request-body transformations or prompt templates;
* whether keys are held only for a request, encrypted again, logged, cached or retained;
* Auto routing algorithm and provider/model selection at runtime;
* provider retry, failover, rate-limit and error-normalization policy;
* server-side context construction, summarization, memory retrieval or prompt augmentation;
* server-side tool planning/orchestration and validation before client actions;
* which provider receives particular context or tool results.

The desktop proves its own request and action handling, not those server behaviors.

### Remote configuration and accounts

Feature flags and workspace fields can be populated remotely. Source defaults do not prove production values. Available `LLMInfo`, model IDs/context limits, BYOK/custom-inference access, Free-account restrictions and managed policies are server supplied. A setting visible or a key stored locally does not prove entitlement or use.

Pricing, credit enforcement, provider billing, fallback charging, token accounting and plan limits are absent. `allow_use_of_warp_credits`, `warpTokenUsage`, `byokTokenUsage`, modal labels and names are contracts/UI evidence only. They must not be treated as billing rules.

### Privacy and retention

Client redaction and telemetry gates are visible, but transport/service logs, subprocess/provider retention, training use, deletion windows, residency and enterprise policy enforcement are not. Official current privacy/security documentation and contractual terms are required.

### Windows runtime variables

The bundle script proves build features, not that embedded model initialization succeeds on every machine. Shell discovery depends on installed versions/PATH/registry/distributions; integration hooks can fail. User keymaps can replace shortcut defaults. Endpoint reachability, proxies, TLS interception and firewall behavior require runtime testing.

## Potentially confusing adjacent implementations

`agent_sdk` can launch third-party CLI harnesses locally and managed secrets can become child-process environment/config. That is not the same path as a prompt classified into the graphical terminal's Warp Agent request. Likewise `warp_tui` shares API-key and agent abstractions but is not the production Windows graphical client. Claims in this analysis are scoped to the latter unless stated.

A URL such as `http://localhost:...` is syntactically accepted as a custom endpoint, but the interactive request sends that definition to Warp. This alone does not establish support for a model server reachable only from the user's loopback interface.

## Conflicts/evidence cautions

* All V1/V2/V3 model files exist in the checkout; conditional embedding and the Windows feature set mean only V3 is production evidence.
* Comments sometimes call functionality “BYOE”, “BYOK”, “custom inference”, or “custom provider”. Use wire types and executable conversions rather than assuming those labels are identical.
* UI copy names LiteLLM and OpenRouter; only OpenRouter has a dedicated stored-key field, while LiteLLM fits the generic endpoint protocol.
* z.ai naming/catalog entries, if returned remotely, do not establish a dedicated desktop provider implementation.
* GraphQL managed secrets primarily cover managed/local harness scenarios and should not be conflated with the graphical interactive `ApiKeys` secure-storage blob.

## Concrete questions for Windows testing/current official documentation

1. On the current Stable Windows bundle, does startup log successful `bert_tiny_v3.onnx` loading, and what latency/route transitions occur per keystroke?
2. What exact key chords and prefixes are displayed for Shell/AI lock and temporary override in the default Windows keymap?
3. Does cmd receive full block/history metadata, and what differs among Windows PowerShell, PowerShell Core, WSL and Git Bash?
4. Which outbound hosts receive a Warp Agent request configured with each provider key and each custom schema? Use redacted network tracing.
5. Are BYOK keys proxied by Warp, relayed ephemerally, or used by another isolated service, and what retention/logging guarantees apply?
6. Can a custom endpoint on `localhost` or a private LAN be reached, or must it be publicly reachable from Warp infrastructure?
7. Which concrete models does Auto select, and can remote configuration change them during a conversation?
8. What happens on invalid key, provider quota, timeout and context-limit errors with Warp credit fallback on versus off?
9. Which Free-account entitlements gate BYOK, custom inference, tool use and model selection today?
10. How are Warp credits and provider tokens charged for retries, failed calls, tool-only turns and fallback?
11. Which prompt/terminal/file fields are retained by Warp and providers, and how do telemetry/UGC/secret-redaction settings alter transport versus analytics?
12. Does the server add command history, repository context, memory or environment data beyond fields visible in the desktop request?

Answers should be sourced from current runtime traces, account records and official documentation—not inferred from labels in this checkout.
