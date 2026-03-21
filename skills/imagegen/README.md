# imagegen - AI Image Generation

Generate images from text prompts via verging.ai. Supports gpt-image-1, dall-e-3 and more.

## Install

```bash
npx skills add verging-ai/agent-skills --skill imagegen
```

## Get API Key
1. Visit https://verging.ai
2. Login → Click user avatar (top right) → Select "API Keys" from dropdown
3. Create key → Set: export VERGING_API_KEY="your_key"

## Options (optional)
- --model: gpt-image-1, dall-e-3 (default: gpt-image-1)
- --size: 1024x1024, 1024x1536, 1536x1024 (default: 1024x1024)
- --quality: auto, low, medium, high (default: auto)
- --n: Number of images 1-4 (default: 1)
- --output: Save images to directory

## Credits
- Pre-deduct billing based on model, size, quality, and count
