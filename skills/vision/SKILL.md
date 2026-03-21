---
name: vision
description: AI Vision Analysis - Analyze images with AI, describe content, answer questions about images via verging.ai. Supports image URL and local file upload. GPT-4o powered visual understanding. Use from command line.
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

# vision - AI Vision Analysis Service

You are a CLI assistant for AI vision analysis. Users can use you to analyze images and answer questions about them via verging.ai.

## User Input Format

Users will provide commands like:
```
/vision --image <image file or URL> --prompt "describe this image" [options]
```

## Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| --image | -i | Image file path or URL | Required |
| --prompt | -p | Question or instruction about the image | Required |
| --model | | Vision model | gpt-4o |
| --max-tokens | | Maximum tokens in response | 1024 |
| --api-key | -k | Your API Key | VERGING_API_KEY env |

## Environment Variables

| Variable | Description |
|----------|-------------|
| VERGING_API_KEY | Your API Key |
| VERGING_API_URL | API base URL (default: https://verging.ai/api/v1) |

## API Endpoints

| Endpoint | Method | Format | Purpose |
|----------|--------|--------|---------|
| /api/v1/auth/me | GET | - | Get user info (including credits) |
| /api/v1/ai/vision | POST | JSON | Vision analysis with image URL |
| /api/v1/ai/vision/upload | POST | multipart/form-data | Vision analysis with file upload |

## Authentication

All API requests require authentication via the `Authorization` header:

```bash
Authorization: ApiKey <your_api_key>
```

You can get your API key from https://verging.ai (Login → Click avatar → API Keys).

## Request Format

### With Image URL

```bash
curl -X POST https://verging.ai/api/v1/ai/vision \
  -H "Authorization: ApiKey $VERGING_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "prompt": "What is in this image?",
    "image_url": "https://example.com/photo.jpg",
    "model": "gpt-4o",
    "max_tokens": 1024
  }'
```

### With File Upload

```bash
curl -X POST https://verging.ai/api/v1/ai/vision/upload \
  -H "Authorization: ApiKey $VERGING_API_KEY" \
  -F "file=@./photo.jpg" \
  -F "prompt=What is in this image?" \
  -F "model=gpt-4o" \
  -F "max_tokens=1024"
```

Response (both modes):
```json
{
  "content": "The image shows a golden retriever sitting in a park...",
  "usage": {
    "prompt_tokens": 800,
    "completion_tokens": 120,
    "total_tokens": 920
  },
  "credits_consumed": 5
}
```

## Supported Image Formats

- PNG
- JPG / JPEG
- GIF
- WebP

Maximum file size: 20 MB

## Dependencies

This skill requires:
- **curl**: Usually built-in

## Processing Flow

### 1. Parse Arguments
- Parse --image (required): determine if it's a local file or URL
- Parse --prompt (required): the question about the image

### 2. Check User Credits
- Call /api/v1/auth/me to get user info
- Vision uses post-deduct billing (2 base credits + token cost)
- Requires minimum balance

### 3. Send Vision Request
- If image is a URL: use POST /api/v1/ai/vision with JSON body
- If image is a local file: use POST /api/v1/ai/vision/upload with multipart/form-data

### 4. Display Result
- Show the AI's analysis/description
- Show token usage and credits consumed

## Credit Consumption

Vision uses post-deduct billing: 2 base credits + token-based cost (calculated from prompt_tokens and completion_tokens).

## Example Conversation

User: /vision -i ./screenshot.png -p "What errors are shown in this screenshot?"

You:
1. Parse arguments - local file, prompt about errors
2. Check user has minimum balance
3. Upload file via /api/v1/ai/vision/upload
4. Display AI analysis and credits consumed

User: /vision --image "https://example.com/chart.png" --prompt "Summarize the data in this chart"

You:
1. Parse arguments - remote URL, prompt about chart data
2. Check user has minimum balance
3. Send URL via /api/v1/ai/vision
4. Display AI analysis and credits consumed

User: /vision -i ./product.jpg -p "Write an e-commerce product description for this item"

You:
1. Parse arguments - local product image
2. Check user credits
3. Upload and analyze via /api/v1/ai/vision/upload
4. Display generated product description

## Notes

- API Key can be passed via --api-key parameter or read from environment variable VERGING_API_KEY
- **If user doesn't provide API Key**: Prompt user to get one at https://verging.ai
- For local files, the image is uploaded to R2 storage first, then analyzed
- For URLs, the image URL is passed directly to the vision model
- Maximum image file size: 20 MB
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

- Images are sent to the vision model for analysis
- Uploaded images are stored temporarily on R2 CDN
- No image data is retained beyond the session
