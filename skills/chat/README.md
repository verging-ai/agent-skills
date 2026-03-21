# chat - AI Chat Completion

LLM chat completion proxy via verging.ai. Supports GPT-4o and other models, streaming and non-streaming modes.

## Install

```bash
npx skills add verging-ai/agent-skills --skill chat
```

## Get API Key
1. Visit https://verging.ai
2. Login → Click user avatar (top right) → Select "API Keys" from dropdown
3. Create key → Set: export VERGING_API_KEY="your_key"

## Options (optional)
- --model: LLM model (default: gpt-4o)
- --temperature: Sampling temperature (0-2)
- --max-tokens: Max response tokens
- --stream: Enable streaming output

## Credits
- Post-deduct billing based on token usage
