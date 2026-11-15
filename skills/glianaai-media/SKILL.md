---
name: glianaai-media
description: "GlianaAI media: generating or editing images, video, speech, music or transcripts through GlianaAI (api.glianalabs.com). Picking a model by quality and price, the fields that drive the price (resolution, duration, quality, text length), reference images and source video, and cheaper multi-step video recipes. Use with the glianaai skill."
license: MIT
---

# GlianaAI media

Every media model runs through `POST /v1/infer` with `{"model": "<id>", ...inputs}` and returns
`{"output": {"url": ..., "contentType": ...}, "costMicroUsd": n}` (some models return a list of
URLs). Follow the loop in the `glianaai` skill: find, read the schema, quote, ask, run, return.

## Picking a model

1. `GET /v1/models` and keep the category you need:
   `text-to-image`, `image-to-image`, `text-to-video`, `image-to-video`, `video-to-video`,
   `tts`, `stt`, `music`.
2. Sort by `unitPriceMicroUsd` for the cheap end; `releasedAt` (newest first) for the latest.
3. Default to the cheapest model that fits. Pick a pricier one only when the user asks for quality,
   names a model, or needs something the cheap one lacks (readable text in the image, a specific
   style, audio in the video, a voice that speaks Indonesian well). The schema shows what each
   model accepts.

## Fields that change the price

Always set them, and quote with the same values (`GET /v1/price?model=<id>&field=value`):

| Kind | Field | If left unset |
|---|---|---|
| Video | `duration` (seconds) | billed at the longest |
| Video | `resolution` (e.g. `480p`, `720p`, `1080p`) | billed at the highest tier |
| Images priced by quality (gpt-image family) | `quality` (`low` … `max`) | billed at the top tier |
| Images | `n` / `num_images` | each extra image is billed |
| Speech (tts) | `text` | priced by its length |
| Music | `music_length_ms` | billed at the longest |
| Upscale | `target` (megapixels) | smallest bracket that covers it |

A 5-second video can cost several times more at 1080p than at 480p. For a draft or a test, use
the lowest resolution and shortest duration the user will accept.

## Inputs that are files

Reference images, a starting frame, a source video or audio for transcription are URLs. Local
file: `POST /v1/media` with the raw bytes and its `content-type`, then pass the returned `url`.
Check the schema for the field name (`image_input`, `images`, `image`, `video_uri`, `audio_url`
…): it differs per model, and an unknown field is dropped, not sent.

## Cheaper video: recipes

Text-to-video models are the expensive end. A recipe makes an image first and animates it, often
for much less, as one call at one price:

- `POST /v1/recipes/image-to-video` — `{"prompt": "...", "motionPrompt": "..."}`
- `POST /v1/recipes/image-to-video-hd`, `/v1/recipes/image-to-video-audio` — higher quality,
  with sound
- Every recipe with its current price and inputs: `GET /v1/recipes`

## Speech, music, transcripts

- **tts:** `{"model": "<tts id>", "text": "..."}` plus the model's voice field from its schema.
  Long text costs more: confirm before reading a whole document aloud.
- **stt:** pass the audio URL in the model's audio field; the output is the transcript (some
  models add timestamps).
- **music:** a `prompt` for style and mood; lyrics or instrumental per the model's schema.

## Returning results

Give the URL and what it cost. Write the file into the project only when the user asked for a
file (download the URL). Do not describe an image in words instead of showing the link.
