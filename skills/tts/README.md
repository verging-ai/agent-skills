# tts - AI Text-to-Speech

Convert text to natural speech audio via verging.ai. Multiple voices and output formats.

## Install

```bash
npx skills add verging-ai/agent-skills --skill tts
```

## Get API Key
1. Visit https://verging.ai
2. Login → Click user avatar (top right) → Select "API Keys" from dropdown
3. Create key → Set: export VERGING_API_KEY="your_key"

## Options (optional)
- --voice: alloy, echo, fable, onyx, nova, shimmer (default: alloy)
- --model: tts-1, tts-1-hd (default: tts-1-hd)
- --format: mp3, opus, aac, flac (default: mp3)
- --speed: 0.25 to 4.0 (default: 1.0)
- --output: Save audio file to path

## Credits
- Pre-deduct billing based on character count
