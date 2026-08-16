# Prime Agent v0.7.2 — Privacy/Provider Lockdown Audit

Date: 2026-08-12

## External HTTP Call Sites

| File | Call | Purpose |
|------|------|---------|
| `packages/coding-agent/src/utils/version-check.ts:121` | `fetch()` | Calls PrimeIntellect release API |
| `packages/coding-agent/src/utils/tools-manager.ts:134` | `fetch()` | Calls GitHub API for tool versions |
| `packages/coding-agent/src/core/model-registry.ts:991` | `fetch()` | Codex models URL |
| `packages/coding-agent/src/cli-main.ts:32` | `import("undici")` | Undici polyfill import |
| `packages/coding-agent/src/core/telemetry.ts` | `fetch()` | POST to `api.primeintellect.ai/agent-analytics/events` |
| `packages/coding-agent/src/core/agent-traces.ts:514,803` | `fetch()` | POST to `api.primeintellect.ai/agent-traces/...` |
| `packages/coding-agent/src/core/prime-inference-models.ts:49` | `fetch()` | GET from `api.pinference.ai/models` |
| `packages/ai/src/mcp/oauth.ts:49,219` | `fetch()` | OAuth token exchange (external IdP) |
| `packages/ai/src/providers/openai-codex-responses.ts:240` | `fetch()` | Codex URL resolution (configurable) |
| `packages/agent/src/proxy.ts:152` | `fetch()` | Proxy stream |

## Telemetry / Analytics Hardcoded Domains

| Domain | File | Constant |
|--------|------|----------|
| `api.primeintellect.ai` | `core/telemetry.ts` | `DEFAULT_TELEMETRY_ENDPOINT` |
| `api.primeintellect.ai` | `core/agent-traces.ts` | inline URL template |
| `api.pinference.ai` | `core/prime-inference-models.ts` | `PRIME_INFERENCE_BASE_URL` |

No Sentry, Mixpanel, PostHog, Datadog, Firebase, or other analytics SDKs found.

## Providers to Remove

| Provider file | Reason |
|---------------|--------|
| `packages/ai/src/providers/anthropic.ts` | Non-OpenAI |
| `packages/ai/src/providers/google.ts` | Non-OpenAI |
| `packages/ai/src/providers/google-vertex.ts` | Non-OpenAI |
| `packages/ai/src/providers/mistral.ts` | Non-OpenAI |
| `packages/ai/src/providers/bedrock-provider.ts` | Non-OpenAI/AWS |

Package.json export entries to remove: `./anthropic`, `./google`, `./google-vertex`, `./mistral`, `./bedrock-provider`.

## Prime-Intelligence Specific Files to Remove

| File | Reason |
|------|--------|
| `packages/coding-agent/src/core/telemetry.ts` | Telemetry client + analytics sink |
| `packages/coding-agent/src/core/agent-traces.ts` | Trace upload to PrimeIntellect |
| `packages/coding-agent/src/core/prime-inference-auth.ts` | PrimeInference OAuth provider |
| `packages/coding-agent/src/core/prime-inference-models.ts` | PrimeInference model list |
| `packages/coding-agent/src/core/prime-inference-model-selection.ts` | PrimeInference model selection |
| `packages/coding-agent/src/utils/version-check.ts` | Version check against PrimeIntellect |
| `packages/ai/src/mcp/oauth.ts` | Generic OAuth (external IdPs) |
| `prime-agent-runtime/` | IPython kernel + home-phone |

## Remaining External Calls After Strip

| File | Reason retained |
|------|-----------------|
| `packages/coding-agent/src/utils/tools-manager.ts:134` | GitHub tool-install version check (can be disabled via config) |
| `packages/agent/src/proxy.ts:152` | User-configured proxy endpoint |
| `packages/ai/src/providers/openai-codex-responses.ts:240` | User-configured codex URL |
| `packages/ai/src/mcp/oauth.ts` | REMOVED |

All provider HTTP calls go to user-configured base URLs — that is by design.
