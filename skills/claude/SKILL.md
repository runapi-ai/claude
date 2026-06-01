---
name: claude
description: Call the Claude API (claude-opus, claude-sonnet, claude-haiku) through RunAPI using the official Anthropic SDK or any Anthropic-compatible client. Use when the user asks for Claude / Anthropic chat, streaming messages, multimodal vision input, tool use, extended thinking, or token counting, or when they want to point an existing Anthropic SDK setup at RunAPI as the base URL.
documentation: https://runapi.ai/models/claude.md
provider_page: https://runapi.ai/providers/anthropic.md
catalog: https://runapi.ai/models.md
metadata:
  openclaw:
    homepage: https://runapi.ai/models/claude
    primaryEnv: ANTHROPIC_API_KEY
    requires:
      env:
      - ANTHROPIC_API_KEY
      - ANTHROPIC_BASE_URL
    envVars:
    - name: ANTHROPIC_API_KEY
      required: true
      description: RunAPI API key used by Anthropic-compatible SDKs.
    - name: ANTHROPIC_BASE_URL
      required: true
      description: Set to https://runapi.ai for Claude on RunAPI.
---

# Claude on RunAPI

Use the official **Anthropic SDK** (Python, TypeScript, Ruby) — or any
Anthropic-compatible HTTP client — and switch the base URL to
`https://runapi.ai`. The endpoint speaks the Anthropic Messages protocol
(`POST /v1/messages`), so no client code changes beyond `base_url` and
`api_key`.

## Setup

```dotenv
ANTHROPIC_API_KEY=YOUR_RUNAPI_TOKEN
ANTHROPIC_BASE_URL=https://runapi.ai
```

Get a RunAPI API Key at <https://runapi.ai/api_keys>.

| Language | Init |
|---|---|
| Python | `anthropic.Anthropic(api_key=..., base_url="https://runapi.ai")` |
| TypeScript | `new Anthropic({ apiKey: ..., baseURL: "https://runapi.ai" })` |
| Ruby | `Anthropic::Client.new(api_key: ..., base_url: "https://runapi.ai")` |
| curl | `POST https://runapi.ai/v1/messages` with `x-api-key:` header |

`x-api-key` and `Authorization: Bearer ...` both work; the SDK uses
`x-api-key` by default.

## Core recipe — non-streaming message

```python
import anthropic

client = anthropic.Anthropic(
    api_key="YOUR_RUNAPI_TOKEN",
    base_url="https://runapi.ai",
)

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system="You are a helpful assistant.",
    messages=[{"role": "user", "content": "Explain quantum computing simply."}],
)
print(message.content[0].text)
print(message.usage)  # input_tokens / output_tokens
```

```typescript
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({
  apiKey: "YOUR_RUNAPI_TOKEN",
  baseURL: "https://runapi.ai",
});

const message = await client.messages.create({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  system: "You are a helpful assistant.",
  messages: [{ role: "user", content: "Explain quantum computing simply." }],
});
```

`max_tokens` is required by the Anthropic API.

## Streaming

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "Write a haiku about coding."}],
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

```typescript
const stream = await client.messages.stream({
  model: "claude-sonnet-4-6",
  max_tokens: 1024,
  messages: [{ role: "user", content: "Write a haiku about coding." }],
});

for await (const event of stream) {
  if (event.type === "content_block_delta") {
    process.stdout.write(event.delta.text);
  }
}
```

Streaming runs through a regional edge proxy so the request does not hold a
Rails/Puma thread. Long generations (extended thinking, large `max_tokens`)
should always stream.

## Vision / multimodal

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1024,
  "messages": [
    {
      "role": "user",
      "content": [
        { "type": "text", "text": "What is in this image?" },
        {
          "type": "image_url",
          "image_url": { "url": "https://example.com/image.png" }
        }
      ]
    }
  ]
}
```

Image input uses the standard Anthropic `image_url` block. URLs must be
publicly fetchable.

## Tool use / web search / reasoning

```json
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 1024,
  "reasoning_effort": "high",
  "include_thoughts": true,
  "tools": [
    { "type": "function", "function": { "name": "googleSearch" } }
  ],
  "messages": [
    { "role": "user", "content": "What's the latest on Claude 4.7?" }
  ]
}
```

- `reasoning_effort`: `"low"` or `"high"`. Supported on every model below.
- `include_thoughts`: returns reasoning content. Only `claude-sonnet-4-5-20250929`
  and `claude-sonnet-4-6` support this.
- Web access uses a `googleSearch` function tool.
- Set header `anthropic-beta: interleaved-thinking-2025-05-14` to interleave
  thinking blocks with output.

## Token counting

```bash
curl -X POST "https://runapi.ai/v1/messages/count_tokens" \
  -H "x-api-key: YOUR_RUNAPI_TOKEN" \
  -H "anthropic-version: 2023-06-01" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "messages": [{"role": "user", "content": "How many tokens?"}]
  }'
```

Returns `{"input_tokens": <n>}`. Image blocks use a 512-token heuristic; for
exact response usage read `usage` from the actual `POST /v1/messages` response.

## List models

```bash
curl https://runapi.ai/v1/models -H "x-api-key: YOUR_RUNAPI_TOKEN"
```

Returns Anthropic-compatible model objects.

## Supported models

| Model ID | Use when |
|---|---|
| `claude-opus-4-8` | Strongest general model — agents, complex reasoning |
| `claude-opus-4-7` | Previous Opus generation for stable workloads |
| `claude-opus-4-6` | High-end reasoning workloads |
| `claude-sonnet-4-6` | Balanced default for production chat |
| `claude-opus-4-5-20251101` | Pin Opus 4.5 snapshot |
| `claude-sonnet-4-5-20250929` | Pin Sonnet 4.5 snapshot (supports `include_thoughts`) |
| `claude-haiku-4-5-20251001` | Highest throughput lightweight model |

Aliases auto-resolve to dated snapshots: `claude-opus-4-5`,
`claude-sonnet-4-5`, `claude-haiku-4-5`.

## Connect Claude Code itself

```bash
ANTHROPIC_BASE_URL=https://runapi.ai \
ANTHROPIC_API_KEY=YOUR_RUNAPI_TOKEN \
claude
```

## Agent rules

- Always pass `max_tokens` — the Anthropic API rejects requests without it.
- Use streaming for any response longer than a few hundred tokens. Do not hold
  the agent on a long blocking request.
- `include_thoughts` only works on the two Sonnet models listed above; do not
  send it on Opus or Haiku.
- Pricing, rate limits, quotas — link to <https://runapi.ai/models/claude.md>,
  not this skill file.
- For exact token usage read `usage` from the `POST /v1/messages`
  response, not from `/v1/messages/count_tokens`.

## Routing

- Model page: <https://runapi.ai/models/claude.md>
- Provider page: <https://runapi.ai/providers/anthropic.md>
- Catalog: <https://runapi.ai/models.md>
