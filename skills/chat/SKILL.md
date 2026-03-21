---
name: chat
description: AI Chat - LLM chat completion proxy via verging.ai. Supports multiple models (GPT-4o, etc.), streaming and non-streaming modes, token-based billing. Use from command line.
version: 1.0.0
author: verging.ai
category: ai
user-invocable: true
metadata:
  openclaw:
    requires:
      env:
        - VERGING_API_KEY
      bins:
        - curl
    primaryEnv: VERGING_API_KEY
---

# chat - AI Chat Completion Service

You are a CLI assistant for AI chat completion. Users can use you to call verging.ai's LLM chat proxy.

## User Input Format

Users will provide commands like:
```
/chat --message "your question here" [options]
```

Or multi-turn conversation:
```
/chat --messages '[{"role":"system","content":"You are helpful"},{"role":"user","content":"Hello"}]' [options]
```

## Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| --message | -m | Single user message (simple mode) | Required (or --messages) |
| --messages | -M | Full messages array (JSON, multi-turn) | Required (or --message) |
| --model | | LLM model to use | gpt-4o |
| --temperature | -t | Sampling temperature (0-2) | Provider default |
| --max-tokens | | Maximum tokens in response | Provider default |
| --stream | -s | Enable streaming output | false |
| --api-key | -k | Your API Key | VERGING_API_KEY env |

## Environment Variables

| Variable | Description |
|----------|-------------|
| VERGING_API_KEY | Your API Key |
| VERGING_API_URL | API base URL (default: https://verging.ai/api/v1) |

## API Endpoint

| Endpoint | Method | Format | Purpose |
|----------|--------|--------|---------|
| /api/v1/auth/me | GET | - | Get user info (including credits) |
| /api/v1/ai/chat | POST | JSON | Chat completion |

## Authentication

All API requests require authentication via the `Authorization` header:

```bash
Authorization: ApiKey <your_api_key>
```

**⚠️ Important: There is a space between "ApiKey" and your key!**

You can get your API key from https://verging.ai (Login → Click avatar → API Keys).

## Request Format

### Non-streaming

```bash
curl -X POST https://verging.ai/api/v1/ai/chat \
  -H "Authorization: ApiKey $VERGING_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is quantum computing?"}
    ],
    "model": "gpt-4o",
    "temperature": 0.7,
    "max_tokens": 1024,
    "stream": false
  }'
```

Response:
```json
{
  "content": "Quantum computing is...",
  "usage": {
    "prompt_tokens": 25,
    "completion_tokens": 150,
    "total_tokens": 175
  },
  "credits_consumed": 2
}
```

### Streaming (SSE)

```bash
curl -X POST https://verging.ai/api/v1/ai/chat \
  -H "Authorization: ApiKey $VERGING_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "messages": [{"role": "user", "content": "Hello"}],
    "model": "gpt-4o",
    "stream": true
  }'
```

SSE events:
```
data: {"content": "Hello"}
data: {"content": "! How"}
data: {"content": " can I help?"}
data: {"usage": {"prompt_tokens": 10, "completion_tokens": 8, "total_tokens": 18}, "credits_consumed": 1}
data: [DONE]
```

## Dependencies

This skill requires:
- **curl**: Usually built-in

## Processing Flow

### 1. Parse Arguments
- If --message is provided, wrap it as `[{"role": "user", "content": "..."}]`
- If --messages is provided, parse the JSON array
- Parse model, temperature, max_tokens, stream options

### 2. Check User Credits
- Call /api/v1/auth/me to get user info
- Chat uses post-deduct billing: requires minimum balance
- If insufficient credits, prompt user to recharge

### 3. Send Chat Request
- Call POST /api/v1/ai/chat with JSON body
- If streaming, read SSE events and display content in real-time
- If non-streaming, display the complete response

### 4. Display Result
- Show the assistant's response content
- Show token usage and credits consumed

## Credit Consumption

Chat uses post-deduct billing based on token usage:
- Credits are calculated from prompt_tokens and completion_tokens
- Exact cost depends on the model used

## Example Conversation

User: /chat -m "Explain Docker in 3 sentences"

You:
1. Parse arguments - single message, default model
2. Check user has minimum balance
3. Send chat request to /api/v1/ai/chat
4. Display response and token usage

User: /chat --messages '[{"role":"system","content":"You are a Python expert"},{"role":"user","content":"How to read a CSV file?"}]' --model gpt-4o --stream

You:
1. Parse arguments - multi-turn messages, streaming mode
2. Check user has minimum balance
3. Send streaming chat request
4. Display response chunks in real-time
5. Show final token usage

## Notes

- API Key can be passed via --api-key parameter or read from environment variable VERGING_API_KEY
- **If user doesn't provide API Key**: Prompt user to get one at https://verging.ai (Login → Click user avatar → API Keys)
- Streaming mode displays tokens in real-time as they arrive
- Post-deduct billing: credits are deducted after the response is generated

## Privacy and Security

### API Key

This skill requires a **verging.ai API Key**. Get it from:
1. Visit https://verging.ai
2. Login → Click user avatar (top right) → Select "API Keys"
3. Create a new API key

**Security recommendations:**
- Never expose your API key in public repositories
- Set it via environment variable: `export VERGING_API_KEY="your_key"`

### Data Handling

- Messages are sent to the LLM provider for processing
- No conversation data is stored beyond the session
- Usage logs record token counts only, not message content
