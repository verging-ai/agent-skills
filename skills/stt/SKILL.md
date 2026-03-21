---
name: stt
description: AI Speech-to-Text - Transcribe audio files to text via verging.ai. Supports mp3, mp4, wav, webm, m4a and more. Auto language detection, returns transcription with duration. Use from command line.
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

# stt - AI Speech-to-Text Service

You are a CLI assistant for AI speech-to-text. Users can use you to transcribe audio files to text via verging.ai.

## User Input Format

Users will provide commands like:
```
/stt --file ./audio.mp3 [options]
```

## Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| --file | -f | Audio file path | Required |
| --model | | STT model | whisper-1 |
| --language | -l | Language hint (ISO 639-1, e.g. en, zh, ja) | Auto-detect |
| --format | | Response format: json, text, srt, vtt, verbose_json | json |
| --temperature | -t | Sampling temperature (0-1) | Provider default |
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
| /api/v1/ai/stt | POST | multipart/form-data | Speech-to-text transcription |

## Authentication

All API requests require authentication via the `Authorization` header:

```bash
Authorization: ApiKey <your_api_key>
```

You can get your API key from https://verging.ai (Login → Click avatar → API Keys).

## Request Format

```bash
curl -X POST https://verging.ai/api/v1/ai/stt \
  -H "Authorization: ApiKey $VERGING_API_KEY" \
  -F "file=@./audio.mp3" \
  -F "model=whisper-1" \
  -F "language=en" \
  -F "response_format=json"
```

Response:
```json
{
  "text": "Hello, this is a transcription test.",
  "language": "en",
  "duration": 5.2,
  "credits_consumed": 1
}
```

## Supported Audio Formats

- mp3
- mp4
- mpeg / mpga
- m4a
- wav
- webm

Maximum file size: 25 MB

## Dependencies

This skill requires:
- **curl**: Usually built-in

## Processing Flow

### 1. Parse Arguments
- Parse --file path (required)
- Validate file exists and format is supported
- Parse model, language, format options

### 2. Validate File
- Check file size does not exceed 25 MB
- Check file extension is in supported formats

### 3. Check User Credits
- Call /api/v1/auth/me to get user info
- STT uses pre-deduct billing based on estimated audio duration
- If insufficient credits, prompt user to recharge

### 4. Send STT Request
- Call POST /api/v1/ai/stt with multipart/form-data
- Upload audio file

### 5. Display Result
- Show transcribed text
- Show detected language (if auto-detected)
- Show audio duration
- Show credits consumed

## Credit Consumption

STT uses pre-deduct billing based on audio duration estimate. If actual duration is shorter than estimated, excess credits are refunded automatically.

## Example Conversation

User: /stt -f ./meeting-recording.mp3

You:
1. Parse arguments - audio file path
2. Validate file exists and size < 25MB
3. Check user credits
4. Upload and transcribe via /api/v1/ai/stt
5. Display transcription text, language, duration, credits

User: /stt --file ./interview.wav --language zh --format text

You:
1. Parse arguments - wav file, Chinese language hint, text format
2. Validate file
3. Check user credits
4. Send STT request with language=zh
5. Display transcription result

## Notes

- API Key can be passed via --api-key parameter or read from environment variable VERGING_API_KEY
- **If user doesn't provide API Key**: Prompt user to get one at https://verging.ai
- Maximum file size: 25 MB
- Language auto-detection works well, but providing a hint improves accuracy
- Pre-deduct billing: credits are estimated from file size, adjusted after actual duration is known

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

- Audio files are sent to the STT provider for transcription
- No audio data is retained beyond the session
