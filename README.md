# AgentRouter CLI Configurations 🚀

This repository contains the configuration files and workarounds needed to perfectly integrate **AgentRouter** with popular AI coding assistants (Qwen, Claude Code, OpenCode, and Codex). 

At its core, this setup correctly maps AgentRouter's Base URLs, Models, and API Keys to each tool's specific format. It also includes fixes for known tool-specific quirks (like downgrading Codex and bypassing stream crashes).

## 📥 Installation

First, clone this repository to your server:
```bash
git clone https://github.com/clickdot/agentrouter-cli-configs.git
cd agentrouter-cli-configs
```

---

## 🔧 1. The Stream Proxy (Recommended)
**Why you need this:** When streaming responses, AgentRouter occasionally injects a `data: null` block and a `billing_summary` event. This will completely crash strict JSON parsers like **Qwen** and **OpenCode**. It also strictly enforces `User-Agent` headers.

This repository includes `proxy.py`, a lightweight local proxy that filters the stream on the fly and spoofs the User-Agent.

**To run the proxy in the background:**
```bash
nohup python3 proxy.py >/tmp/ar-proxy.log 2>&1 &
```
*(Tip: Add the line above to your `~/.bashrc` using the absolute path to `proxy.py` so it starts automatically).*

---

## 2. Qwen Configuration
Qwen requires the settings to explicitly list the model and base URL, otherwise it ignores the `-m` flag.

**File:** `~/.qwen/settings.json`
```json
{
  "tools": { 
    "approvalMode": "yolo" 
  },
  "env": {
    "OPENAI_API_KEY": "YOUR_AGENT_ROUTER_TOKEN", 
    "OPENAI_BASE_URL": "http://127.0.0.1:8787/v1",
    "OPENAI_MODEL": "claude-opus-4-8"
  },
  "modelProviders": {
    "openai": [
      {
        "id": "claude-opus-4-8",
        "name": "[AgentRouter] claude-opus-4-8",
        "baseUrl": "http://127.0.0.1:8787/v1"
      }
    ]
  },
  "model": {
    "name": "claude-opus-4-8",
    "baseUrl": "http://127.0.0.1:8787/v1"
  }
}
```
> **Note:** If you chose not to run `proxy.py`, change `http://127.0.0.1:8787/v1` to `https://agentrouter.org/v1`.

---

## 3. Claude Code Configuration
Claude Code uses Anthropic standards, so the Base URL specifically **omits** `/v1`. Additionally, if you run as `root` on your server, you must use `"acceptEdits"` instead of `"bypassPermissions"`.

**1. Add to `~/.bashrc`:**
```bash
export ANTHROPIC_AUTH_TOKEN="YOUR_AGENT_ROUTER_TOKEN"
export ANTHROPIC_BASE_URL="https://agentrouter.org"
export ANTHROPIC_MODEL="claude-opus-4-8"
```

**2. File:** `~/.claude/settings.json`
```json
{
  "permissions": { "defaultMode": "acceptEdits" },
  "theme": "dark"
}
```

---

## 4. OpenCode Configuration
OpenCode fails to find the authentication cookie if relying on environment variables, so the key must be hardcoded. 

**1. File:** `~/.local/share/opencode/auth.json`
```json
{
  "agentrouter": {
    "apiKey": "YOUR_AGENT_ROUTER_TOKEN"
  }
}
```

**2. File:** `~/.config/opencode/config.json` *(or `opencode.json` in your project)*
```json
{
  "provider": {
    "agentrouter": {
      "npm": "@ai-sdk/openai-compatible",
      "options": {
        "baseURL": "http://127.0.0.1:8787/v1",
        "apiKey": "YOUR_AGENT_ROUTER_TOKEN"
      }
    }
  },
  "model": "agentrouter/gpt-5.5"
}
```
> **Note:** If you chose not to run `proxy.py`, change `http://127.0.0.1:8787/v1` to `https://agentrouter.org/v1`.

---

## 5. Codex Configuration
AgentRouter only supports `chat` completions, but new versions of Codex completely dropped `chat` support. You must downgrade Codex first.

**1. Downgrade Codex:**
```bash
npm install -g @openai/codex@0.80.0
```

**2. File:** `~/.codex/config.toml`
```toml
model = "gpt-5.5"
model_provider = "openai-chat-completions"
preferred_auth_method = "apikey"

[model_providers.openai-chat-completions]
base_url = "https://agentrouter.org/v1"
env_key = "OPENAI_API_KEY"
wire_api = "chat"
```
*(Make sure `export OPENAI_API_KEY="YOUR_TOKEN"` is set in your `~/.bashrc`).*
