---
name: google-tts
description: |
  Convert documents and text to audio using Google Cloud Text-to-Speech.
  Use this skill when the user wants to: narrate a document, read aloud text,
  generate audio from a file, convert text to speech, create a recording
  of documentation or analysis, create a podcast from a document, or use
  Google TTS/text-to-speech. Trigger phrases: "read this aloud", "narrate this",
  "create a recording", "text to speech", "TTS", "convert to audio",
  "audio from document", "listen to this", "generate audio", "google tts",
  "create a podcast".
---

# Google Cloud Text-to-Speech

Converts text and documents into audio using Google Cloud TTS API. Supports Neural2, WaveNet, Studio, and Standard voices across 40+ languages.

## Setup

API key via `GOOGLE_TTS_API_KEY` env var or `skills/google-tts/config.json` with `{"api_key": "..."}`.
Requires `ffmpeg` for multi-chunk documents. Optional: `pip install PyPDF2 python-docx` for PDF/DOCX.

## Commands

### List Voices

```bash
python skills/google-tts/scripts/google_tts.py voices --language en-US --type Neural2
python skills/google-tts/scripts/google_tts.py voices --json
```

### Text-to-Speech

```bash
# From text or document (PDF, DOCX, MD, TXT)
python skills/google-tts/scripts/google_tts.py tts --text "Hello world" --output ~/Downloads/hello.mp3
python skills/google-tts/scripts/google_tts.py tts --file /path/to/doc.pdf --output ~/Downloads/narration.mp3

# With voice, rate, pitch, encoding options
python skills/google-tts/scripts/google_tts.py tts --file doc.md --voice en-US-Neural2-F --rate 0.9 --encoding MP3 --output ~/Downloads/out.mp3
```

### Podcast Generation

Takes a JSON script with alternating speakers, synthesizes each with a different voice.

```json
[
  {"speaker": "host1", "text": "Welcome to our podcast!"},
  {"speaker": "host2", "text": "Thanks for having me..."}
]
```

```bash
python skills/google-tts/scripts/google_tts.py podcast --script /tmp/script.json --output ~/Downloads/podcast.mp3
python skills/google-tts/scripts/google_tts.py podcast --script /tmp/script.json --voice1 en-US-Neural2-J --voice2 en-US-Neural2-H --rate 0.9 --output ~/Downloads/podcast.mp3
```

## Workflow

### Single-Voice Narration

1. If user provides a file path, use `--file`. For generated content, write clean prose to `/tmp/tts_input.md` first.
2. Default voice: `en-US-Neural2-D` (male) or `en-US-Neural2-F` (female). Use Neural2 for best quality/cost balance.
3. Generate: `python skills/google-tts/scripts/google_tts.py tts --file /tmp/tts_input.md --output ~/Downloads/recording.mp3`
4. Report file location and size. Default output to `~/Downloads/`.

### Podcast from Document

1. Extract text: `python skills/google-tts/scripts/extract.py /path/to/document.pdf`
2. Generate a two-host conversation script as JSON:
   - Natural discussion, not verbatim reading. Host 1 leads, Host 2 reacts/analyzes.
   - Include intro and outro. Vary turn lengths. Keep turns under 4000 chars.
3. Write script to `/tmp/podcast_script.json`
4. Generate: `python skills/google-tts/scripts/google_tts.py podcast --script /tmp/podcast_script.json --output ~/Downloads/podcast.mp3`
5. Clean up temp files.

## Reference

- **Recommended voice type**: Neural2 (~$4/1M chars, high quality)
- **Speaking rate**: 0.25-4.0 (0.85-0.95 good for technical content)
- **Pitch**: -20.0 to 20.0 semitones
- **Encodings**: MP3 (default), LINEAR16 (.wav), OGG_OPUS (.ogg)
- API limit: 5000 bytes/request. Script auto-chunks at sentence boundaries.
