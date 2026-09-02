# API reference

Verified against https://ai.google.dev/gemini-api/docs/image-generation and the pricing page on 2026-09-02. Google changes these pages often. If something here disagrees with the live doc, the live doc wins.

## Models

| | Nano Banana Pro | Nano Banana 2 | Nano Banana 2 Lite | Nano Banana (original) |
|---|---|---|---|---|
| Model ID | `gemini-3-pro-image` | `gemini-3.1-flash-image` | `gemini-3.1-flash-lite-image` | `gemini-2.5-flash-image` |
| Status | Stable, GA 2026-05-28 | Stable, GA 2026-05-28 | Stable, GA 2026-06-30 | Stable, legacy |
| Sizes | 1K, 2K, 4K | 512, 1K, 2K, 4K | 1K | ~1024px |
| Aspect ratios | 10 | 14 | 10 | 10 |
| Object refs | 6 | 10 | 14 | up to 3 total |
| Character refs | 5 | 4 | none | |
| Style refs | 3 | none | none | |
| Max input images | 14 | 14 | 14 | 3 |
| Thinking | always on, cannot disable | `minimal` (default) or `high` | `minimal` or `high` | none |
| Search grounding | web | web and image search | none | none |
| Video input | no | yes | unclear | no |
| Context / max output tokens | 65,536 / 32,768 | 131,072 / 32,768 | 65,536 / 4,096 | 65,536 / 32,768 |

Every `-preview` ID was retired on 2026-07-17 and errors out. There is no "Nano Banana 3". Imagen 4 was shut down 2026-08-17.

Aspect ratios:

- All models: `1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9`
- 3.1 Flash Image adds `1:4, 4:1, 1:8, 8:1`
- Vertex lists `9:21` as well for Pro and 3.1 Flash. Unverified on the Developer API, test before relying on it.

Reference caps above come from the API doc. Google's marketing pages say "up to 14 objects" for Pro. The API doc says 6. Trust the API doc.

## Pricing per image

Developer API, paid tier. Batch API and Flex are half price. Source: https://ai.google.dev/gemini-api/docs/pricing

| Model | 512 | 1K | 2K | 4K |
|---|---|---|---|---|
| `gemini-3-pro-image` | | $0.134 | $0.134 | $0.24 |
| `gemini-3.1-flash-image` | $0.045 | $0.067 | $0.101 | $0.151 |
| `gemini-3.1-flash-lite-image` | | $0.0336 | | |
| `gemini-2.5-flash-image` | | $0.039 | | |

Input images cost tokens too: 1,120 per image on 3.1 Flash and Lite, 560 on Pro. Search grounding: 5,000 free requests a month shared across Gemini 3.x, then $14 per 1,000.

Practical plan: iterate on 3.1 Flash at 1K ($0.067), ship on Pro at 2K ($0.134) or 4K ($0.24). A 4K Pro render is 3.6x the cost of a 1K Flash draft.

## Which API

Two APIs work today.

- **Interactions API** (`client.interactions.create`). GA since June 2026 and recommended for new projects. Snake_case config under `response_format`.
- **generateContent** (legacy). Still fully supported. Config under `generation_config` / `imageConfig`.

On Vertex (now "Gemini Enterprise Agent Platform"), `imageConfig` is marked deprecated in favor of `responseFormat.image`, which uses enums (`ASPECT_RATIO_SIXTEEN_BY_NINE`, `IMAGE_SIZE_TWO_K`). See the Vertex section.

## Text to image

Python:

```python
from google import genai
import base64

client = genai.Client()  # reads GEMINI_API_KEY

interaction = client.interactions.create(
    model="gemini-3.1-flash-image",
    input="Create an image of a nano banana dish in a fancy restaurant with a Gemini theme",
    response_format={
        "type": "image",
        "aspect_ratio": "16:9",
        "image_size": "2K",
    },
)

with open("out.png", "wb") as f:
    f.write(base64.b64decode(interaction.output_image.data))
```

JavaScript:

```javascript
import { GoogleGenAI } from "@google/genai";
import * as fs from "node:fs";

const ai = new GoogleGenAI({});
const interaction = await ai.interactions.create({
  model: "gemini-3.1-flash-image",
  input: "Create an image of a nano banana dish in a fancy restaurant with a Gemini theme",
  response_format: { type: "image", aspect_ratio: "16:9", image_size: "2K" },
});
fs.writeFileSync("out.png", Buffer.from(interaction.output_image.data, "base64"));
```

REST:

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/interactions" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-3.1-flash-image",
    "input": [{"type": "text", "text": "Create an image of a nano banana dish in a fancy restaurant"}],
    "response_format": {"type": "image", "aspect_ratio": "16:9", "image_size": "2K"}
  }'
```

## Image editing

Python, one input image:

```python
from google import genai
import base64

client = genai.Client()

with open("cat.png", "rb") as f:
    image_b64 = base64.b64encode(f.read()).decode("utf-8")

interaction = client.interactions.create(
    model="gemini-3.1-flash-image",
    input=[
        {"type": "text", "text": "Add a small knitted wizard hat on its head. Keep everything else exactly the same."},
        {"type": "image", "data": image_b64, "mime_type": "image/png"},
    ],
)

with open("out.png", "wb") as f:
    f.write(base64.b64decode(interaction.output_image.data))
```

REST, several images with a role each:

```bash
curl -s -X POST "https://generativelanguage.googleapis.com/v1beta/interactions" \
  -H "x-goog-api-key: $GEMINI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gemini-3-pro-image",
    "input": [
      {"type": "text", "text": "Using Image 1 (the garment) and Image 2 (the model), a full-body fashion photo of the model wearing the garment."},
      {"type": "image", "mime_type": "image/png", "data": "<BASE64_IMG_1>"},
      {"type": "image", "mime_type": "image/png", "data": "<BASE64_IMG_2>"}
    ],
    "response_format": {"type": "image", "aspect_ratio": "4:5", "image_size": "2K"}
  }'
```

## Config fields

Interactions API, all snake_case:

| Field | Values | Notes |
|---|---|---|
| `response_format.type` | `"image"`, or a list like `[{"type":"text"},{"type":"image"}]` | Default returns both text and image |
| `response_format.aspect_ratio` | `"16:9"` etc. | String with a colon |
| `response_format.image_size` | `"512"`, `"1K"`, `"2K"`, `"4K"` | Uppercase K. Lowercase `1k` is rejected |
| `response_format.mime_type` | `"image/png"`, `"image/jpeg"` | Optional |
| `generation_config.thinking_level` | `"minimal"`, `"high"` | 3.1 Flash and Lite only. Pro is always on |
| `tools` | `[{"type": "google_search", "search_types": ["web_search", "image_search"]}]` | `image_search` is 3.1 Flash only |
| `previous_interaction_id` | an interaction id | Multi-turn editing. Carries thought signatures |
| `store` | `false` | Stateless call |

Default size behavior: the model matches the input image's dimensions, otherwise generates 1:1.

There is no seed. There is no negative prompt field. Safety settings exist but the second-layer `IMAGE_SAFETY` filter cannot be disabled.

Legacy `generateContent` in Python uses `response_modalities=['Image']` and `response_format={"image": {"aspect_ratio": ..., "image_size": ...}}` inside `GenerateContentConfig`. REST uses camelCase `imageConfig: {aspectRatio, imageSize}` as a sibling of `responseModalities` under `generationConfig`. One official Vertex edit-images page nests `responseModalities` inside `imageConfig`; that is a doc bug, do not copy it.

## Multi-turn editing

```python
first = client.interactions.create(model="gemini-3.1-flash-image", input=[...])

second = client.interactions.create(
    model="gemini-3.1-flash-image",
    input="Keep everything the same, but make the lighting warmer.",
    previous_interaction_id=first.id,
)
```

Keep the whole request under 50MB. Restart the chain after three or four edits (see [troubleshooting](troubleshooting.md)).

## Rate limits

Per-tier requests-per-minute and images-per-minute are not published. They show in AI Studio for your key. The only published per-tier numbers are Batch enqueued tokens:

| Model | Tier 1 | Tier 2 | Tier 3 |
|---|---|---|---|
| 3.1 Flash Image | 1M | 250M | 750M |
| 3.1 Flash Lite Image | 2M | 270M | 1B |
| 3 Pro Image | 2M | 270M | 1B |
| 2.5 Flash Image | 3M | 400M | 1B |

Batch: 100 concurrent jobs, 2GB input file, 20GB storage, up to 24 hours turnaround, half price.

## Vertex differences

Docs moved to `docs.cloud.google.com/gemini-enterprise-agent-platform/`. The old `/vertex-ai/generative-ai/docs/` paths still resolve but serve stale preview-era content.

- Env: `GOOGLE_GENAI_USE_ENTERPRISE=True`, `GOOGLE_CLOUD_PROJECT`, `GOOGLE_CLOUD_LOCATION=global`. Replaces `GOOGLE_GENAI_USE_VERTEXAI`.
- Endpoint: `https://aiplatform.googleapis.com/v1/projects/PROJECT/locations/global/publishers/google/models/gemini-3.1-flash-image:generateContent`
- `responseFormat.image` uses enums. 14 aspect ratios including `1:8` and `8:1`, four sizes (`IMAGE_SIZE_FIVE_TWELVE` through `IMAGE_SIZE_FOUR_K`), JPEG only, delivery `INLINE` or `URI`.
- The legacy `ImageConfig.personGeneration` (`ALLOW_ALL` / `ALLOW_ADULT` / `ALLOW_NONE`) has no documented equivalent in the new format.
- 2.5 Flash Image on Vertex: 32,768 context, hard cap of 3 input images, 10 output images.
- Provisioned Throughput buys a quota window, not speed. 1 GSU of 3.1 Flash Image = one image per 435 seconds. 1 GSU of Pro = one per 1,230 seconds, roughly 70 a day. Details: https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/provisioned-throughput/gemini-3-nano-banana-models
- Video input (3.1 Flash Image) samples 1 fps at 70 tokens per frame, audio ignored. GA 2026-08-31.

## Third-party endpoints

| Host | Model | Price |
|---|---|---|
| fal.ai | `fal-ai/gemini-3-pro-image-preview` and `/edit` | $0.15 at 1K/2K, $0.30 at 4K |
| Replicate | `google/nano-banana-pro` | $0.134 / $0.24 |
| OpenRouter | `google/gemini-3-pro-image-preview` | token-priced, $120 per M image tokens |

ComfyUI nodes: [darkamenosa/comfy_nanobanana](https://github.com/darkamenosa/comfy_nanobanana), [ShmuelRonen/ComfyUI-NanoBanano](https://github.com/ShmuelRonen/ComfyUI-NanoBanano). Adobe Photoshop and Firefly ship Pro inside Generative Fill. Figma has a first-party Gemini 3 Pro Image integration. Third-party hosts may lag Google's model IDs; check theirs before pinning.

## Google's stated limitations

From the image generation doc, verbatim where it matters:

- Best-performing languages: EN, ar-EG, de-DE, es-MX, fr-FR, hi-IN, id-ID, it-IT, ja-JP, ko-KR, pt-BR, ru-RU, ua-UA, vi-VN, zh-CN.
- No audio input. Video input only on 3.1 Flash Image.
- "The model won't always follow the exact number of image outputs that the user explicitly asks for."
- "When generating text for an image, Gemini works best if you first generate the text and then ask for an image with the text."
- 3.1 Flash Image search grounding does not return real-world images of people.
- "All generated images include a SynthID watermark."
- From the model cards: poor at small text and long paragraphs, occasional left/right confusion, limited 3D reasoning and factuality, character consistency not always preserved.

## Terms

Google does not claim ownership of generated output and commercial use is allowed under the Gemini API terms. You likely hold no enforceable copyright on raw AI output in the US, so nobody can be stopped from generating something similar. Free consumer Gemini conversations may be used for training; API traffic is stated not to be. Read the live terms before a client contract: https://ai.google.dev/gemini-api/terms
