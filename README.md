# Yami Proxy

A tool proxy that enables abliterated LLMs (uncensored models) to work with Clawdbot tool calling.

## What It Does

Clawdbot expects OpenAI-compatible tool calling, but abliterated models running on Ollama don't always handle tool schemas correctly. This proxy:

1. Intercepts requests from Clawdbot
2. Injects a custom system prompt with tool instructions in human-readable format
3. Translates tool calls between Clawdbot and Ollama
4. Handles streaming responses

## Architecture

```
Clawdbot --> Yami Proxy (port 4000) --> Ollama (port 11434)
```

## Setup

### 1. Configure Clawdbot

Point your model provider to the proxy instead of Ollama directly:

```json
{
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://127.0.0.1:4000/v1",
        "api": "openai-completions"
      }
    }
  }
}
```

### 2. Start the Proxy

```bash
cd /home/dev/yami
python3 tool_proxy.py [model] [ollama_url]

# Example:
python3 tool_proxy.py "huihui_ai/qwen3-coder-abliterated:30b" "http://192.168.50.100:11434"
```

### 3. Run as Background Service

```bash
nohup python3 tool_proxy.py "model_name" "http://ollama:11434" > proxy.log 2>&1 &
```

## Configuration

### Identity Injection

The proxy injects a system prompt (`OMNI_VECTOR_IDENTITY`) that defines:
- Agent personality and behavior
- Communication style
- Task execution philosophy

**Edit this section in `tool_proxy.py` to customize your agent's personality.**

### Key Lessons Learned

1. **Always acknowledge the human first** — Don't start executing tasks before responding to greetings/questions
2. **Communication > Execution** — Being helpful means being responsive, not just productive
3. **Avoid roleplay bloat** — Keep the identity prompt functional, not theatrical
4. **Session context persists** — Changing files doesn't affect running sessions; restart the gateway AND clear sessions

## Files

| File | Purpose |
|------|---------|
| `tool_proxy.py` | Main proxy script |
| `SOUL.md` | Agent personality (workspace file, optional) |
| `IDENTITY.md` | Agent identity details |
| `TOOLS.md` | Local tool configurations |

## Troubleshooting

### Agent ignores messages and dumps code
The identity prompt is telling it to "execute without explanation". Fix the `OMNI_VECTOR_IDENTITY` section in `tool_proxy.py` to include communication guidelines.

### Changes to files don't take effect
1. Restart the proxy: `pkill -f tool_proxy && python3 tool_proxy.py ...`
2. Clear sessions: `rm ~/.clawdbot/agents/main/sessions/*.jsonl`
3. Restart gateway: `clawdbot gateway restart`

### Proxy not responding
Check if it's running: `curl http://127.0.0.1:4000/health`

## License

MIT — do whatever you want with it.

---

*Built for the Rudu Army* 🔓
