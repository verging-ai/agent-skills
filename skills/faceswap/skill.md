---
name: faceswap
description: AI换脸服务 - 直接在命令行中使用 verging.ai 的 AI 换脸功能。支持本地视频文件和图片、远程视频URL (YouTube、Bilibili等)、远程图片URL，自动下载远程资源，实时进度跟踪。
version: 1.0.0
author: verging.ai
category: media
tags:
  - ai
  - faceswap
  - video
  - image
  - 换脸
  - deepfake
---

# faceswap - AI换脸服务

你是一个 AI 换脸服务的 CLI 助手。用户可以通过你调用 verging.ai 的 AI 换脸功能。

## 用户输入格式

用户会提供类似以下的命令：
```
/faceswap --video <视频文件或URL> --face <人脸图片或URL> [选项]
```

## 选项解析

| 选项 | 缩写 | 描述 | 默认值 |
|------|------|------|--------|
| --video | -v | 目标视频文件路径或URL | 必需 |
| --face | -f | 人脸图片文件路径或URL | 必需 |
| --start | -s | 视频开始时间(秒) | 0 |
| --end | -e | 视频结束时间(秒) | 视频时长 |
| --hd | -h | 高清模式 (3积分/秒 vs 1积分/秒) | false |
| --api-key | -k | 你的 API Key | 环境变量 VERGING_API_KEY |
| --output | -o | 结果保存路径 | 当前目录 |
| --download | -d | 处理完成后自动下载到本地 | false |

## 环境变量

| 变量 | 描述 |
|------|------|
| VERGING_API_KEY | 你的 API Key |
| VERGING_API_URL | API 基础URL (默认: https://verging.ai/api/v1) |

## API 端点

| 接口 | 方法 | 用途 |
|------|------|------|
| /api/v1/auth/me | GET | 获取用户信息(含credits) |
| /api/v1/upload-video | POST | 获取上传URL |
| /api/v1/faceswap/create-job | POST | 创建换脸任务 |
| /api/v1/faceswap/jobs | GET | 查询任务状态 |

## 处理流程

当用户执行 /faceswap 命令时，请按以下步骤执行：

### 1. 参数解析
- 解析 --video 和 --face 参数
- 如果是远程 URL，需要下载到本地
- 解析时间范围 --start 和 --end

### 2. 远程资源下载
- 使用 curl 或 yt-dlp 下载远程资源
- 视频: yt-dlp -f "best[ext=mp4]/best" -o "output.mp4" "URL"
- 图片: curl -L -o "output.jpg" "URL"
- 临时目录: /tmp/verging-faceswap/

### 3. 获取视频时长
- 使用 ffprobe: ffprobe -v error -show_entries format=duration -of default=noprint_wrappers=1:nokey=1 "video.mp4"

### 4. 裁剪视频 (如果指定了 --start 或 --end)
- 如果用户指定了 --start 或 --end 参数，需要先裁剪视频
- 使用 ffmpeg 裁剪指定时间范围:
  ```
  ffmpeg -i input.mp4 -ss <start> -to <end> -c copy output.mp4
  ```
- 或者重新编码确保帧准确:
  ```
  ffmpeg -i input.mp4 -ss <start> -to <end> -c:v libx264 -c:a aac output.mp4
  ```
- 裁剪后的视频作为待上传文件

### 5. 检查用户积分
- 调用 /api/v1/auth/me 获取用户信息
- 计算所需积分: 普通模式 1积分/秒，HD模式 3积分/秒
- 如果积分不足，提示用户充值

### 6. 上传文件到 R2
- 调用 /api/v1/upload-video 获取预签名上传URL
- 使用 PUT 方法上传文件到 R2

### 7. 创建任务
- 调用 /api/v1/faceswap/create-job 创建换脸任务
- 参数: swap_image(人脸图片), file_name, target_video_url, user_video_duration, job_type="face-swap", is_hd

### 8. 轮询任务状态
- 每5秒调用 /api/v1/faceswap/jobs?job_ids=xxx 查询状态
- 状态: PENDING → PROCESSING → COMPLETED/FAILED
- 显示进度百分比

### 9. 返回结果
- 完成后返回 result_url
- 如果用户指定了 --download 或 --output，使用 curl 下载结果

## 积分消耗

| 模式 | 积分/秒 |
|------|----------|
| 普通 | 1 积分/秒 |
| HD | 3 积分/秒 |

## 示例对话

用户: /faceswap -v ./input.mp4 -f ./my-face.jpg --start 5 --end 15

你:
1. 检查本地文件存在
2. 获取视频时长
3. **裁剪视频** (ffmpeg -ss 5 -to 15)
4. 调用 API 获取用户信息
5. 检查积分充足 (10秒 = 10积分)
6. 上传裁剪后的视频和人脸图片到 R2
7. 创建换脸任务
8. 轮询等待完成
9. 返回结果 URL

用户: /faceswap -v ./input.mp4 -f ./my-face.jpg

你:
1. 检查本地文件存在
2. 获取视频时长
3. 调用 API 获取用户信息
4. 检查积分充足
5. 上传视频和人脸图片到 R2
6. 创建换脸任务
7. 轮询等待完成
8. 返回结果 URL

## 注意事项

- API Key 可以通过 --api-key 参数传入，也可以从环境变量 VERGING_API_KEY 读取
- 如果用户没有提供 API Key，提示他们去 https://verging.ai 获取
- 视频时长最大支持 30 秒
- 支持使用 yt-dlp 下载 YouTube，Bilibili 等视频
- 处理过程中显示进度
- **如果指定了 --start 或 --end，会先在本地裁剪视频再上传，节省上传时间和处理成本**
