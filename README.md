# Yami Proxy

A tool proxy that enables abliterated LLMs to work with Clawdbot tool calling.

## Recommended Setup

**Model:** `huihui_ai/glm-4.7-flash-abliterated:q8_0`
- Native tool + thinking capabilities
- Better instruction following than qwen3-coder
- ~200K context window

**Why GLM over Qwen3-Coder:**
| Feature | GLM-4.7-flash | Qwen3-Coder |
|---------|---------------|-------------|
| Native tools | ✅ Yes | ❌ No |
| Thinking capability | ✅ Yes | ❌ No |
| Instruction following | Good | Chaotic |
| Context window | 200K | 262K |

## What It Does

Clawdbot expects OpenAI-compatible tool calling, but local models need help. This proxy:

1. Intercepts requests from Clawdbot
2. Injects a custom identity prompt (Maho-style: conversational, honest, methodical)
3. Converts tool schemas to human-readable format
4. Handles streaming responses

## Architecture

```
Clawdbot --> Yami Proxy (port 4000) --> Ollama (port 11434)
```

## Setup

### 1. Configure Clawdbot

Point your model provider to the proxy:

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
python3 tool_proxy.py "huihui_ai/glm-4.7-flash-abliterated:q8_0" "http://192.168.50.100:11434"

# Or as background service:
nohup python3 tool_proxy.py "huihui_ai/glm-4.7-flash-abliterated:q8_0" "http://192.168.50.100:11434" > proxy.log 2>&1 &
```

## Identity Injection (Maho-Style)

The proxy injects behavioral guidelines:

- **Communication First** — Explain what youre doing before/during
- **Execution Discipline** — ONE command at a time, wait for completion
- **Honesty** — Never lie about output or claim false success
- **Tools vs Talk** — Explanation questions get text, execution gets tools

## Configuration

### Context Window
```python
max_context = 500  # Number of messages to keep
```

### Key Files
| File | Purpose |
|------|---------|
| `tool_proxy.py` | Main proxy script |
| `SOUL.md` | Agent personality (Maho-style) |
| `IDENTITY.md` | Agent identity details |

## Troubleshooting

### Model responds in Chinese or goes crazy
Switch to GLM-4.7-flash — qwen3-coder can be unstable.

### Model narrates but doesnt execute
Ensure proxy is running and Clawdbot points to port 4000.

### Tool loops (same tool called repeatedly)  
Clear sessions: `rm ~/.clawdbot/agents/main/sessions/*.jsonl`

### Broken pipe crashes
Restart proxy. Consider adding error handling for production use.

## Lessons Learned

1. **Model matters** — GLM > Qwen3-Coder for tool use
2. **Always acknowledge humans first** — Dont start executing before greeting
3. **One command at a time** — No parallel execution without explicit request
4. **Session context persists** — Clear sessions when changing behavior

## License

MIT

---

*Built for the Rudu Army* 🔓
