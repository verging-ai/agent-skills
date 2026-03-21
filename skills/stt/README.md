# stt - AI Speech-to-Text

Transcribe audio files to text via verging.ai. Supports mp3, wav, webm, m4a and more.

## Install

```bash
npx skills add verging-ai/agent-skills --skill stt
```

## Get API Key
1. Visit https://verging.ai
2. Login → Click user avatar (top right) → Select "API Keys" from dropdown
3. Create key → Set: export VERGING_API_KEY="your_key"

## Options (optional)
- --model: STT model (default: whisper-1)
- --language: Language hint (ISO 639-1)
- --format: Response format (json, text, srt, vtt)

## Supported Formats
mp3, mp4, mpeg, mpga, m4a, wav, webm (max 25MB)

## Credits
- Pre-deduct billing based on audio duration
