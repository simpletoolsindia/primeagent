# Prime Agent — Enterprise Edition

> A self-improving AI coding agent, **sanitized for internal/enterprise use**.
> Stripped of all telemetry, trace upload, version checks, and outbound beacon calls.

---

## What This Is

Prime Agent is an open-source coding agent built around two core ideas:

- **Recursive Language Model (RLM)** — persistent Python (IPython) kernel as the model's tool; the agent writes code to control itself, spawn subagents, run shell, and edit files.
- **Continual Harness** — durable per-session state (skills, subagent specs, memories) that the agent can refine over time.

It is fully usable as a single binary or as an SDK embedded in your own tooling.

---

## Security Posture (Hardened)

This build removes **all** outbound tracking and silent network behavior of the upstream release.

| Component | Status |
|---|---|
| Telemetry (analytics POST) | Removed |
| Trace upload (session transcripts) | Removed |
| Version check (release manifest) | Removed |
| PrimeIntellect auth/login flow | Removed |
| GitHub tool auto-download | Stubbed (no network fetch) |
| OpenAI Codex model discovery | Removed |
| PrimeIntellect-specific provider | Stubbed |
| Demo extensions (Anthropic OAuth, GitLab Duo, Doom) | Removed |
| Example extensions folder | Removed |

**Result:** the agent makes **zero outbound network calls** at startup or during operation, except to the LLM provider endpoint you configure. No phone-home, no install ID, no installation tracking.

This is verified in source: no URLs to `primeintellect.ai`, `r2.dev`, `api.github.com/repos`, or `releases/download` remain in `packages/*/src/`.

---

## Requirements

- **Node.js 20+** (recommended: 22 LTS)
- **Python 3.10+** (for the IPython kernel the agent uses as its primary tool)
- An **OpenAI-compatible LLM endpoint** (see below)

Tested on:
- Linux x86_64 / ARM64
- macOS x86_64 / Apple Silicon

---

## Quick Start

### 1. Install dependencies

```bash
# Node.js (use your distro's package manager, or nvm)
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
nvm install 22

# Python 3 + ipykernel
sudo apt install -y python3 python3-pip python3-venv   # Debian/Ubuntu
# brew install python                                  # macOS

python3 -m venv .venv
source .venv/bin/activate
pip install ipykernel
```

### 2. Build the agent

```bash
cd prime-agent
npm install
npm run build --workspace=@earendil-works/pi-coding-agent
npm run build --workspace=@earendil-works/pi-ai
```

### 3. Create your models config

Create `~/.prime/agent/models.json` (global) or `.prime/agent/models.json` (project):

#### Option A — Local model (Ollama, vLLM, LM Studio, SGLang)

```json
{
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434/v1",
      "api": "openai-completions",
      "apiKey": "ollama",
      "models": [
        { "id": "llama3.1:8b" },
        { "id": "-coder:7b" }
      ]
    }
  }
}
```

For reasoning-capable models on servers that don't understand `developer` role:

```json
{
  "providers": {
    "ollama": {
      "baseUrl": "http://localhost:11434/v1",
      "api": "openai-completions",
      "apiKey": "ollama",
      "compat": {
        "supportsDeveloperRole": false,
        "supportsReasoningEffort": false
      },
      "models": [
        { "id": "gpt-oss:20b", "reasoning": true }
      ]
    }
  }
}
```

#### Option B — OpenAI API

```json
{
  "providers": {
    "openai": {
      "baseUrl": "https://api.openai.com/v1",
      "api": "openai-completions",
      "apiKey": "sk-...",
      "models": [
        { "id": "gpt-4o" },
        { "id": "gpt-4o-mini" }
      ]
    }
  }
}
```

#### Option C — Internal LLM proxy (vLLM, LiteLLM, etc.)

```json
{
  "providers": {
    "internal": {
      "baseUrl": "https://llm.internal.example.com/v1",
      "api": "openai-completions",
      "apiKey": "sk-internal-...",
      "headers": {
        "X-Internal-Team": "platform"
      },
      "models": [
        { "id": ".5-coder-32b-instruct" },
        { "id": "deepseek-coder-v2" }
      ]
    }
  }
}
```

> **Tip:** set the `apiKey` via environment variable instead of hardcoding:
> ```json
> "apiKey": "OPENAI_API_KEY"
> ```
> Then `export OPENAI_API_KEY=sk-...` before launching.

### 4. Launch

```bash
./packages/coding-agent/dist/index.js
```

Or install globally and use the `prime-agent` command:

```bash
npm link --workspace=@earendil-works/pi-coding-agent
prime-agent
```

---

## Provider Configuration Reference

| Field | Required | Description |
|---|---|---|
| `baseUrl` | Yes | OpenAI-compatible endpoint (`/v1` for most servers) |
| `api` | Yes | Set to `"openai-completions"` |
| `apiKey` | Yes | Auth token (or env-var name — see above) |
| `compat.supportsDeveloperRole` | No | Set `false` if server rejects `developer` role |
| `compat.supportsReasoningEffort` | No | Set `false` if server ignores `reasoning_effort` |
| `compat.maxTokensField` | No | `"max_tokens"` or `"max_completion_tokens"` |
| `headers` | No | Extra request headers (e.g., routing/team tags) |
| `models[].id` | Yes | Model identifier (e.g., `gpt-4o`, `llama3.1:8b`) |
| `models[].reasoning` | No | `true` for reasoning-capable models |
| `models[].contextWindow` | No | Override context window size |
| `models[].maxTokens` | No | Override max output tokens |

### Supported OpenAI-compatible servers

Any server exposing `/v1/chat/completions` with the OpenAI schema:

- **Ollama** — `http://localhost:11434/v1`
- **vLLM** — `http://localhost:8000/v1`
- **LM Studio** — `http://localhost:1234/v1`
- **SGLang** — `http://localhost:30000/v1`
- **LiteLLM** — `https://your-proxy/v1`
- **OpenAI** — `https://api.openai.com/v1`
- **** — ` **Any custom internal LLM gateway** speaking the OpenAI Chat Completions schema

---

## Configuration Files

| Location | Scope |
|---|---|
| `~/.prime/agent/settings.json` | Global (all projects) |
| `~/.prime/agent/models.json` | Global provider config |
| `.prime/agent/settings.json` | Project (overrides global) |
| `.prime/agent/models.json` | Project provider overrides |

Edit directly or use `/settings` inside the agent for common options.

---

## Optional: Project Setup

For best results, run inside a disposable clone or worktree:

```bash
git worktree add /tmp/agent-wt main
cd /tmp/agent-wt
prime-agent
```

> **Warning:** Prime Agent executes model-generated Python and project commands with your user permissions. Use a disposable worktree, container, or restricted user account.

---

## Embedding / RPC Mode

Run headless over stdio JSONL for IDE integration or automation:

```bash
prime-agent --mode rpc --provider openai --model gpt-4o
```

Send prompts as JSON lines:

```json
{"id": "req-1", "type": "prompt", "message": "List the files in the current directory"}
```

See `packages/coding-agent/docs/rpc.md` for the full protocol.

---

## Air-gapped / Locked-Down Networks

This build is **network-quiet by default**. The agent will only contact:

1. The `baseUrl` you configured in `models.json`
2. Nothing else.

If you want to be defensive at the network layer, firewall:

- **Allow:** your `baseUrl` host(s) only
- **Deny:** all other outbound traffic from the agent's runtime user

---

## Verification (Post-Install)

Confirm no residual external calls in the source:

```bash
grep -rn "primeintellect.ai\|r2.dev\|api.github.com/repos\|releases/download" \
  packages/*/src/ 2>/dev/null
```

Expected output: empty.

---

## What's NOT Included

This enterprise build does **not** include:

- The PrimeIntellect-hosted inference catalog (`api.pinference.ai`)
- The PrimeIntellect auth/team model
- OAuth flows to PrimeIntellect account
- Auto-update mechanism (you manage versions yourself)
- Demo/showcase extensions (Doom, GitLab Duo, etc.)

If you need any of these, restore them from upstream — but understand they're the same surface that was sanitized out.

---

## License

Same as upstream: see `LICENSE` in this repo.

## Upstream

Originally forked from [PrimeIntellect-ai/prime-agent](https://github.com/PrimeIntellect-ai/prime-agent). This enterprise build has diverged from upstream — do not blindly merge upstream changes without re-running the security audit.