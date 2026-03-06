# verging.ai Agent Skills

AI-powered media processing skills for Claude Code.

## Features

### Face Swap
AI换脸服务 - 直接在命令行中使用 verging.ai 的 AI 换脸功能。

- 支持本地视频文件和图片
- 支持远程视频 URL (YouTube，Bilibili 等)
- 支持远程图片 URL
- 自动下载远程资源
- 实时进度跟踪
- 视频裁剪功能（指定起止时间）

## Installation

### 1. Add Marketplace

```bash
/plugin marketplace add verging-ai/agent-skills
```

### 2. Install Skills

```bash
/plugin install faceswap
```

## Usage

### Face Swap

```bash
# 基本用法
/faceswap --video ./input.mp4 --face ./my-face.jpg

# 指定时间范围
/faceswap -v ./video.mp4 -f ./face.jpg --start 5 --end 30

# 使用远程视频
/faceswap -v "https://youtube.com/watch?v=xxx" -f ./face.jpg --hd

# 自动下载结果
/faceswap -v ./video.mp4 -f ./face.jpg --download
```

### Options

| Option | Short | Description | Default |
|--------|-------|-------------|---------|
| --video | -v | Target video file or URL | Required |
| --face | -f | Face image file or URL | Required |
| --start | -s | Start time in seconds | 0 |
| --end | -e | End time in seconds | Video duration |
| --hd | -h | HD mode (3 credits/sec vs 1 credit/sec) | false |
| --api-key | -k | Your API Key | VERGING_API_KEY env |
| --output | -o | Output directory | Current dir |
| --download | -d | Auto download result | false |

### Environment Variables

```bash
# Set your API Key
export VERGING_API_KEY="your_api_key_here"

# Optional: Set API URL (default: https://verging.ai/api/v1)
export VERGING_API_URL="https://verging.ai/api/v1"
```

## Get API Key

1. Visit [https://verging.ai](https://verging.ai)
2. Register/Login
3. Go to API Keys section
4. Create a new API Key

## Credits

- Normal mode: 1 credit/second
- HD mode: 3 credits/second

## More Skills Coming Soon

- Video Enhancement
- Background Removal
- Image to Video
- And more...

## License

MIT
