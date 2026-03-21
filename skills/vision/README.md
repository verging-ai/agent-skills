# vision - AI Vision Analysis

Analyze images with AI, describe content, answer questions about images via verging.ai.

## Install

```bash
npx skills add verging-ai/agent-skills --skill vision
```

## Get API Key
1. Visit https://verging.ai
2. Login → Click user avatar (top right) → Select "API Keys" from dropdown
3. Create key → Set: export VERGING_API_KEY="your_key"

## Options (optional)
- --model: Vision model (default: gpt-4o)
- --max-tokens: Max response tokens (default: 1024)

## Supported Image Formats
PNG, JPG, JPEG, GIF, WebP (max 20MB)

## Credits
- Post-deduct billing: 2 base credits + token cost
