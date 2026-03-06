---
name: video-prompting
description: Draft and refine prompts for video generation models (text-to-video and image-to-video). Use when a user asks for a "video prompt" or a model-specific prompt such as Ovi, Sora, Veo 3, Wan 2.2, or LTX-2, including requests like "text-to-video prompt", "image-to-video prompt", or "write a prompt for [model]".
---

# Video Prompting

## Overview

Turn a user’s intent into a strong, model-compliant video prompt by routing to the correct model guide and applying its formatting/tokens.

Model-specific guidance lives in `references/models/`. This file is the entry point: pick the model, ask the minimum clarifying questions, then draft the prompt in that model’s expected format.

## Model Index

- Ovi: `references/models/ovi/prompting.md`
- Sora (Sora 2): `references/models/sora/prompting.md`
- Veo 3 / 3.1: `references/models/veo3/prompting.md`
- Wan 2.2: `references/models/wan22/prompting.md`
- LTX-2: `references/models/ltx2/prompting.md`

To add a new model later: create `references/models/<model>/prompting.md`, then add it to this index.

## Workflow

### Step 1 — Identify the model and input mode

If the user did not name a model, ask which model they are using (or offer supported options from the Model Index).

Then confirm the input mode:

- Text-to-video (t2v), or
- Image-to-video (i2v)

If i2v: ask the user to share the image (optional, but it will help you generate a better prompt). Use the image as an anchor according to the chosen model’s guidance (e.g., keep identity/wardrobe/composition stable; focus your text on motion/camera/what changes).

If the chosen model has versions, duration constraints, or required parameters, ask the minimum questions needed to select the right format (see the model guide).

### Step 2 — Load the model reference and follow its format

Open the model’s `prompting.md` from the Model Index and follow its rules strictly (tokens, audio formatting, parameter constraints, recommended structure).

### Step 3 — Draft the prompt as a coherent clip

Default structure (adapt to the model’s style and required sections):

1. Subject(s): who/what, distinctive details
2. Setting: where/when, lighting, mood
3. Action progression: what changes over time (start → beat → beat → end)
4. Camera: framing/movement only if it matters
5. Dialogue/audio: only if the model supports it, using the model’s exact format

Avoid keyword soup. Prefer a single, well-described shot unless the user explicitly wants multiple cuts/shots.

### Step 4 — Output

Default: output only the final prompt text.

If the user asks for options: provide 2–3 distinct prompt variants, each fully self-contained and compliant with the model’s formatting.

If the model uses required API parameters (e.g., duration/size), include a short “Recommended parameters” line only when the user has specified them or explicitly asks for them.
