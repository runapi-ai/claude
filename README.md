# Claude API Skill for RunAPI

[![skills.sh](https://skills.sh/b/runapi-ai/claude)](https://skills.sh/runapi-ai/claude)

Call the Claude API through RunAPI with the official Anthropic SDK — point any
Anthropic client at `https://runapi.ai`, keep your `claude-opus-*`,
`claude-sonnet-*`, `claude-haiku-*` model strings, and pay through one RunAPI
balance. This skill teaches Claude Code, Codex, Gemini CLI, Cursor, and 50+
agents how to wire the Anthropic SDK up to the Claude API on RunAPI.

The canonical agent file is `skills/claude/SKILL.md`.

## Install the skill

```bash
npx skills add runapi-ai/claude -g
```

Or paste this prompt to your AI agent:

```text
Install the claude skill for me:

1. Clone https://github.com/runapi-ai/claude
2. Copy the skills/claude/ directory into your
   user-level skills directory (e.g. ~/.claude/skills/
   for Claude Code, ~/.codex/skills/ for Codex).
3. Verify that SKILL.md is present.
4. Confirm the install path when done.
```

## Use the Claude API on RunAPI

The Claude API on RunAPI speaks the standard Anthropic Messages protocol, so
the official Anthropic SDK works unchanged once `base_url` points at
`https://runapi.ai` and `api_key` is set to a RunAPI API Key.

```python
import anthropic

client = anthropic.Anthropic(
    api_key="YOUR_RUNAPI_TOKEN",
    base_url="https://runapi.ai",
)

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Hello, Claude!"}],
)
print(message.content[0].text)
```

```javascript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: "YOUR_RUNAPI_TOKEN",
  baseURL: "https://runapi.ai",
});

const message = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Hello, Claude!" }],
});
console.log(message.content[0].text);
```

```bash
curl -X POST "https://runapi.ai/v1/messages" \
  -H "x-api-key: YOUR_RUNAPI_TOKEN" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello, Claude!"}]
  }'
```

Get a RunAPI API Key at <https://runapi.ai/api_keys>.

## Connect Claude Code itself

```bash
ANTHROPIC_BASE_URL=https://runapi.ai \
ANTHROPIC_API_KEY=YOUR_RUNAPI_TOKEN \
claude
```

## Supported Claude models

| Model ID | Notes |
|---|---|
| `claude-opus-4-7` | Strongest general model |
| `claude-opus-4-6` | High-end reasoning |
| `claude-sonnet-4-6` | Balanced reasoning + speed |
| `claude-opus-4-5-20251101` | Dated Opus 4.5 snapshot |
| `claude-sonnet-4-5-20250929` | Dated Sonnet 4.5 snapshot |
| `claude-haiku-4-5-20251001` | Fastest, lowest cost |

Aliases that resolve to the dated snapshots: `claude-opus-4-5`,
`claude-sonnet-4-5`, `claude-haiku-4-5`.

## Routing

- Claude API on RunAPI: <https://runapi.ai/models/claude>
- Claude API docs: <https://runapi.ai/docs#claude>
- Provider page: <https://runapi.ai/providers/anthropic>
- Browse the full RunAPI catalog: <https://runapi.ai/models>
- Skill repository: <https://github.com/runapi-ai/claude>

## Agent rules

- Keep API keys in `ANTHROPIC_API_KEY` (or your secret manager); never inline
  them in commits or shell history.
- Stream long responses (`stream: true`) so the agent can release the
  terminal/IO loop early.
- For pricing, rate-limit, and commercial-usage answers, link to
  <https://runapi.ai/models/claude> rather than this README.

## License

Licensed under the Apache License, Version 2.0.
