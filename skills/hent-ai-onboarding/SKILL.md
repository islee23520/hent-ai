---
name: hent-ai-onboarding
description: Hent-ai emotion image onboarding — guide users through creating a custom character and 6 emotion images for the emotion-image plugin. Triggers on "온보딩", "onboarding", "캐릭터 이미지 만들어줘", "감정 이미지 생성", "setup character", "이미지 온보딩".
---

# Hent-ai Onboarding

Conversational onboarding for creating emotion images. No coded state machine — the agent drives the entire flow.

## Asset Directory

Resolve the target `imageDir` in order:
1. Plugin config `imageDir` if explicitly set
2. Profile-scoped: `~/.openclaw/workspace/.hent-ai/emotion-image-assets/`
3. Fallback: `~/.clawhip/github/IYENTeam/Hent-ai/assets/`

All generated images go into this directory.

## Flow

### 1. Ask for character description

Ask what character they want (e.g. "cute orange cat", "pixel art robot"). The user may also attach a reference image.

### 2. Handle attached images

If the user attaches an image, ask:
- Use it **directly** as the base character, or
- Use it as a **style reference** for generation

### 3. Generate base character

```
image_generate(prompt="[user description], clean illustration style, square format, simple background, high quality PNG")
```

Show the result. Ask for approval.
- Approved → proceed to step 4
- Feedback → regenerate with feedback
- Cancel (`취소`/`cancel`/`종료`/`그만`) → abort

Save approved image as `base.png` in the asset directory.

### 4. Generate 6 emotion variants

For each emotion, one at a time, use the base image as reference:

| Emotion | Filename | Cues |
|---------|----------|------|
| happy | `happy.png` | smiling, celebrating, thumbs up |
| neutral | `neutral.png` | calm, relaxed, default expression |
| loyalty | `loyalty.png` | saluting, nodding, attentive |
| sorry | `sorry.png` | apologetic, bowing, sheepish |
| confused | `confused.png` | head tilt, question mark, puzzled |
| focused | `focused.png` | concentrating, working, determined |

Prompt template:
```
Same character as the reference image, expressing [emotion]. [cues]. Simple background, consistent art style.
```

For each:
- Show result, ask approval
- Accept feedback for regeneration
- User may attach their own image instead
- On approval, save to asset directory

### 5. Complete

Confirm all 7 images saved (base + 6 emotions). Report the asset directory path.

## Rules

- One emotion at a time. Do not batch-generate.
- Never generate text/speech bubbles in images.
- Keep the same character identity across all variants.
- User can abort at any step with: `취소`, `cancel`, `종료`, `그만`
- Respond in the user's language.

## Labeled Image Pools (optional, post-onboarding)

After basic onboarding, users can add multiple images per emotion with labels:

```jsonc
// in plugin config emotionMap
{
  "happy": [
    { "file": "happy-stage.png", "label": "stage", "weight": 2 },
    { "file": "happy-date-night.png" }
  ]
}
```

Hent-ai auto-infers labels from filenames (e.g. `happy-date-night.png` → `date night`) and prefers matching labels in bot response context.
