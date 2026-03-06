---
name: deapi
description: AI media generation via deAPI. Transcribe YouTube/audio/video, generate images from text, text-to-speech, OCR, remove backgrounds, upscale images, create videos, generate embeddings. 10-20x cheaper than OpenAI/Replicate.
user-invocable: true
disable-model-invocation: false
---

# deAPI Media Generation

AI-powered media tools via decentralized GPU network.

## Available Commands

| Command | Use when user wants to... |
|---------|---------------------------|
| `/transcribe` | Transcribe YouTube, Twitch, Kick, X videos, or audio files |
| `/generate-image` | Generate images from text descriptions |
| `/generate-audio` | Convert text to speech (TTS) |
| `/generate-video` | Create video from text or animate images |
| `/ocr` | Extract text from images (OCR) |
| `/remove-bg` | Remove background from images |
| `/upscale` | Upscale image resolution (2x/4x) |
| `/transform-image` | Apply style transfer to images |
| `/embed` | Generate text embeddings for semantic search |
| `/deapi-setup` | Configure result delivery (webhooks/websockets) for server apps |
| `/deapi-balance` | Check account balance and remaining credits |

## Quick Examples

```
/transcribe https://youtube.com/watch?v=...
/generate-image a sunset over mountains
/generate-audio "Hello world" --voice am_adam
```

## Setup

Requires `DEAPI_API_KEY` environment variable:

```bash
export DEAPI_API_KEY=your_key
```

Get your API key at [deapi.ai](https://deapi.ai) (free $5 credit).

## API Pattern

All deAPI requests are async:
1. Submit job → get `request_id`
2. Poll status every 10s
3. When `done` → fetch from `result_url`

For detailed API parameters, see [docs/api-reference.md](../docs/api-reference.md).
