---
name: faceswap
description: AI face swap service - Use verging.ai AI face swap directly from command line. Supports local video files and images, remote video URLs (YouTube, Bilibili, etc.), remote image URLs, auto-download remote resources, real-time progress tracking.
version: 1.0.0
author: verging.ai
category: media
tags:
  - ai
  - faceswap
  - video
  - image
  - deepfake
---

# faceswap - AI Face Swap Service

You are a CLI assistant for AI face swap. Users can use you to call verging.ai's AI face swap functionality.

## User Input Format

Users will provide commands like:
```
/faceswap --video <video file or URL> --face <face image or URL> [options]
```

## Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| --video | -v | Target video file path or URL | Required |
| --face | -f | Face image file path or URL | Required |
| --start | -s | Video start time in seconds | 0 |
| --end | -e | Video end time in seconds | Video duration |
| --hd | -h | HD mode (3 credits/sec vs 1 credit/sec) | false |
| --api-key | -k | Your API Key | VERGING_API_KEY env |
| --output | -o | Result save path | Current directory |
| --download | -d | Auto download result to local | false |

## Environment Variables

| Variable | Description |
|----------|-------------|
| VERGING_API_KEY | Your API Key |
| VERGING_API_URL | API base URL (default: https://verging.ai/api/v1) |

## API Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| /api/v1/auth/me | GET | Get user info (including credits) |
| /api/v1/upload-video | POST | Get upload URL |
| /api/v1/faceswap/create-job | POST | Create face swap job |
| /api/v1/faceswap/jobs | GET | Query job status |

## Processing Flow

When the user executes the /faceswap command, please follow these steps:

### 0. Check and Install Dependencies
Before executing any operations, check if the following tools are installed. If not, auto-install:

| Tool | Check Command | Auto Install Command |
|------|---------------|---------------------|
| yt-dlp | `which yt-dlp` | `pip install yt-dlp` or `brew install yt-dlp` |
| ffmpeg | `which ffmpeg` | `brew install ffmpeg` |
| ffprobe | `which ffprobe` | (installed with ffmpeg) |
| curl | `which curl` | (usually built-in) |

Installation logic:
1. First check if tool exists (`which <tool>`)
2. If not exists, detect system package manager (macOS: brew, Linux: apt/yum)
3. Auto-install missing tools
4. After installation, continue with subsequent steps

### 1. Parse Arguments
- Parse --video and --face parameters
- If remote URL, need to download to local
- Parse time range --start and --end

### 2. Download Remote Resources
- Use curl or yt-dlp to download remote resources
- Video: yt-dlp -f "best[ext=mp4]/best" -o "output.mp4" "URL"
- Image: curl -L -o "output.jpg" "URL"
- Temp directory: /tmp/verging-faceswap/

### 3. Get Video Duration
- Use ffprobe: ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "video.mp4"

### 4. Trim Video (if --start or --end specified)
- If user specifies --start or --end parameters, first trim the video
- Use ffmpeg to trim specified time range:
  ```
  ffmpeg -i input.mp4 -ss <start> -to <end> -c copy output.mp4
  ```
- Or re-encode for accurate frames:
  ```
  ffmpeg -i input.mp4 -ss <start> -to <end> -c:v libx264 -c:a aac output.mp4
  ```
- Use trimmed video as the file to upload

### 5. Check User Credits
- Call /api/v1/auth/me to get user info
- Calculate required credits: Normal mode 1 credit/sec, HD mode 3 credits/sec
- If insufficient credits, prompt user to recharge

### 6. Upload Files to R2
- Call /api/v1/upload-video to get pre-signed upload URL
- Use PUT method to upload files to R2

### 7. Create Job
- Call /api/v1/faceswap/create-job to create face swap job
- Parameters: swap_image(face image), file_name, target_video_url, user_video_duration, job_type="face-swap", is_hd

### 8. Poll Job Status
- Every 5 seconds call /api/v1/faceswap/jobs?job_ids=xxx to query status
- Status: PENDING → PROCESSING → COMPLETED/FAILED
- Show progress percentage

### 9. Return Result
- After completion, return result_url
- If user specified --download or --output, use curl to download result

## Credit Consumption

| Mode | Credits/sec |
|------|-------------|
| Normal | 1 credit/sec |
| HD | 3 credits/sec |

## Example Conversation

User: /faceswap -v ./input.mp4 -f ./my-face.jpg --start 5 --end 15

You:
0. **Check and install dependencies** (yt-dlp, ffmpeg, ffprobe)
1. Check local file exists
2. Get video duration
3. **Trim video** (ffmpeg -ss 5 -to 15)
4. Call API to get user info
5. Check credits sufficient (10 seconds = 10 credits)
6. Upload trimmed video and face image to R2
7. Create face swap job
8. Poll for completion
9. Return result URL

User: /faceswap -v ./input.mp4 -f ./my-face.jpg

You:
0. **Check and install dependencies** (yt-dlp, ffmpeg, ffprobe if not installed)
1. Check local file exists
2. Get video duration
3. Call API to get user info
4. Check credits sufficient
5. Upload video and face image to R2
6. Create face swap job
7. Poll for completion
8. Return result URL

## Notes

- **Auto-install dependencies**: On first use, if yt-dlp, ffmpeg, ffprobe are missing, they will be auto-installed
- API Key can be passed via --api-key parameter or read from environment variable VERGING_API_KEY
- If user doesn't provide API Key, prompt them to get one at https://verging.ai
- Video duration max 30 seconds
- Support downloading videos from YouTube, Bilibili, etc. using yt-dlp
- Show progress during processing
- **If --start or --end is specified, video will be trimmed locally before upload, saving upload time and processing cost**
