# Inference and API integration

## Confirmed desktop architecture

`ApiKeyManager` stores an `ApiKeys` JSON blob under secure-storage key `AiApiKeys`. It supports Google, Anthropic, OpenAI, OpenRouter and multiple `CustomEndpoint`s. Custom endpoints have URL, key, model name/alias/stable UUID `config_key`, and one of OpenAI Chat Completions, OpenAI Responses, or Anthropic Messages schemas. The desktop does **not** issue the provider inference HTTP call: `RequestParams` serializes credentials/config into the Warp multi-agent request. Workspace entitlements gate whether `api_keys_for_request` and `custom_model_providers_for_request` populate it.

Model selection uses server-fetched `LLMInfo` plus locally synthesized custom endpoint/router entries. A selected `LLMId` becomes `ModelConfig.base` (and profile-specific CLI/computer-use IDs). “Auto” and custom routers are IDs/config sent to the service; final task classification/routing is absent.

## Comparison

| Integration | Desktop configuration | Credential storage | Desktop request path | Confirmed backend dependency | Confirmed credit behaviour | Important limitations | Evidence |
|---|---|---|---|---|---|---|---|
| Warp-managed inference | Server model catalog, `LLMPreferences`/execution profile model ID. | Warp authentication storage; no provider key. | Model ID in `Request.settings.model_config` to `ServerApi`. | Yes: generation is at Warp endpoint. | Client sends usage/fallback intent; charging not confirmable. | Provider and routing hidden. | `app/src/ai/llms.rs`; `agent/api{,/impl}.rs` |
| Auto routing | Auto/server model entry or local custom router; selected as an `LLMId`. | None beyond account/settings. | ID and optional router rules sent to Warp. | Yes for final routing. | Not confirmable from this repository. | Complexity/prompt categories do not prove production router algorithm. | `custom_model_routers.rs`; `api.rs` |
| OpenAI BYOK | Provider key UI/API manager; OpenAI models from catalog. | OS secure storage `AiApiKeys`. | Key included in request `ApiKeys` when entitlement enabled. | Yes in shown path. | `allow_use_of_warp_credits` exists; enforcement not confirmable. | No consumer ChatGPT subscription integration is implied. | `crates/ai/src/api_keys.rs`; `api.rs` |
| Anthropic BYOK | Provider key UI/API manager. | OS secure storage. | Included in Warp request. | Yes in shown path. | Not confirmable from this repository. | Claude consumer subscription is distinct; managed secrets for local CLI harness are a separate feature. | Same as above |
| Google BYOK | Provider key UI/API manager. | OS secure storage. | Included in Warp request. | Yes in shown path. | Not confirmable from this repository. | GEAP credentials are an additional, separate enterprise path. | Same as above |
| Custom OpenAI-compatible endpoint | Name, URL, key, models, schema; Chat Completions or Responses. | OS secure storage. | `CustomModelProviders` (including endpoint/key/model config) sent to Warp. | Yes for interactive Warp Agent. | Not confirmable from this repository. | URL may be localhost, but Warp service cannot necessarily reach desktop localhost; no direct local client is shown. | `api_keys.rs:CustomEndpoint`; `agent/api.rs` |
| OpenRouter | Dedicated provider key; catalog/provider enum includes OpenRouter. | OS secure storage. | Key in `ApiKeys` to Warp. | Yes in shown path. | Not confirmable from this repository. | UI naming does not prove request transformation. | `ApiKeys.open_router`; `llms.rs` |
| LiteLLM | No dedicated protocol type; configured as custom endpoint using a supported schema. | OS secure storage. | Custom provider definition to Warp. | Yes in shown path. | Not confirmable from this repository. | Compatibility depends on endpoint and absent backend. | custom endpoint types; Free-AI modal wording is documentation/UI evidence only. |
| z.ai | No dedicated confirmed `ApiKeys` field found; may be a catalog model or custom endpoint. | Custom endpoint key if user configures it. | Custom provider to Warp, if configured. | Yes for that path. | Not confirmable from this repository. | Name alone does not prove first-class support. | `ApiKeys`; custom endpoint schema |
| Local model/local endpoint | A localhost URL can be entered as a custom endpoint. | OS secure storage. | Still sent to Warp service in interactive Agent path. | Yes. | Not confirmable from this repository. | Repository does not prove Warp backend can reach machine-local URLs; no desktop provider request was found. | `CustomEndpoint`; `generate_multi_agent_output` |
| Grok/SuperGrok | OAuth flow and connected-token state, separate from pasted keys. | OS secure storage key `GrokOAuthTokens`. | Access token injected into Warp request; request can wait for client-managed refresh. | Yes. | Not confirmable from this repository. | Consumer subscription authorization is provider/server-controlled. | `crates/ai/src/api_keys.rs`; response stream model |

“Backend dependency” describes the current interactive graphical Warp Agent path, not separate local third-party CLI harness features in `agent_sdk`.

## Authentication, streaming, fallback

Warp server authentication is applied by `ServerApi`; provider credentials are request settings. The response is a Warp event stream, not raw OpenAI/Anthropic SSE in the desktop. The client supports request retries/resume and a user setting `can_use_warp_credits_for_fallback`, encoded with API keys. That field confirms intent only. Provider errors, fallback ordering, token accounting, credit eligibility and charging require the missing service.

## Feature/entitlement gates

`UserWorkspaces::is_byo_api_key_enabled` and `is_custom_inference_enabled` decide whether configured secrets enter a request. Feature flags gate custom routers and model/UI capabilities. Available models and account capabilities are server-fed and can differ from source defaults. A stored key therefore does not prove it will be used.
