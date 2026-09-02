# Troubleshooting

Symptom, cause, fix. Sources linked where the cause is measured rather than folk knowledge.

## The face changed

**Symptom.** After two or three edits the person looks older, puffier, waxier, or like a cousin.

**Cause.** The model rebuilds the whole frame each turn from your reference. Small deviations compound. It also simplifies: one documented case reduced a character's four eye colors to two by the second step.

**Fix.**
- Restate the identity lock every turn: "Keep the exact facial features, eye spacing, nose width, jawline, skin tone and apparent age from the reference."
- Add "natural face proportions, no puffy cheeks, no plastic or waxy skin."
- Use better references: 1024px or more, even frontal light, no glasses or hat, two angles.
- After three or four edits, start a new conversation with the original image and the full description.
- Name the character once and refer to the name, not a re-description.

## Quality gets worse each turn

**Symptom.** Grain, mush, drifting colors, instructions ignored after many edits.

**Cause.** Measured. The Banana100 paper ran 13 seeds through 100 sequential edits and found artifacts accumulate into severe noise, while none of 21 image-quality metrics reliably caught it. ([arxiv 2604.03400](https://arxiv.org/abs/2604.03400))

**Fix.** Cap at two to four edits. Then extract what worked into a full prompt and regenerate from the original. Filmmakers push edits until the scene is conceptually right, then re-generate clean from a description plus references.

## Text is garbled

**Symptom.** Misspellings, invented glyphs, dropped letters, wrong font weight, ghost fragments.

**Cause.** Text is rendered as texture. Single-line short strings are where Google claims under 10% error on Pro. Paragraphs, small type and dense layouts are still weak. The Flash model cards say "poor in small text (often blurry in 1k model), long paragraphs, page length."

**Fix.**
- Quote the exact string. Name the font and describe it.
- Keep it under a headline. Split several text elements into separate lines with positions.
- Use Pro and 2K or 4K for anything with text.
- Generate the copy in a text turn first, then ask for the image containing it (Google's own advice).
- Anything longer than a headline: set the type in Figma or Illustrator over a generated background with negative space.

## Colors shifted or went oversaturated

**Symptom.** Skin turns reddish and blotchy, saturation climbs, background tint changes, all despite "keep colors unchanged".

**Cause.** Reported on Google's forum for image-to-image with Pro and 3.1 Flash without a fix. One partial mechanism: oversized references get compressed server-side, with a measured saturation drop. The infamous yellow tint of the original 2.5 Flash Image is fixed in Pro.

**Fix.**
- Pre-resize references to 2K or smaller.
- Say the grade explicitly: "muted natural skin tones, no teal-orange grading, match the reference's white balance."
- Color-match in post. It is a two-minute fix in any editor and more reliable than another turn.

## It returned the original unchanged

**Symptom.** Output looks identical to the input.

**Cause.** Ambiguous instruction, or the model answered in text, or the edit region was not identified.

**Fix.** Start with "Edit this image:" or "Generate an image where...". Name the region by position and appearance. In the Gemini app, draw on the image. If it keeps happening, restate the change as a full scene description rather than a delta.

## It changed something I did not ask for

**Symptom.** You asked for a new coat, the background also changed.

**Cause.** No lock was stated. The model does not scope tightly by default.

**Fix.** Lock first, then change. End with "keep everything else exactly the same." One change per turn.

## It refused

**Symptom.** A block, or a polite text reply instead of an image.

**Cause.** Two safety layers. The first is configurable through safety settings. The second (`IMAGE_SAFETY`) cannot be turned off through the API. Categories that commonly trip it: recognizable public figures from text prompts, famous IP characters, minors, watermark removal, financial document edits, face swaps, suggestive content. Text-to-image of real public figures is refused; editing an uploaded photo of a real person is often not, which is a gap Google's use policy still covers under deceptive impersonation.

**Fix.** Describe the person or character by attributes rather than name. If it is a legitimate use hitting a false positive, rephrase the intent ("a documentary photo of a courtroom" rather than anything that sounds like a document forgery) and try again in a fresh conversation. Do not go looking for jailbreaks; they are documented and they are also against the terms.

## Wrong aspect ratio

**Symptom.** Square output, or the ratio of the wrong reference image.

**Cause.** Ratio is a config field, not a prompt word. The model matches the input image's dimensions by default, or 1:1. With multiple references it may pick the first or the last.

**Fix.** Set `aspect_ratio` in the API config (see [api.md](api.md)). In the app, upload a blank canvas at the target ratio and say "fill this canvas entirely."

## Hands, eyes, reflections

**Symptom.** Extra fingers, asymmetric eyes, a mirror showing a different scene.

**Fix.** Ask for a targeted edit: "Fix the left hand to have five natural fingers holding the cup; change nothing else." If it persists, regenerate at 2K. Check every output against the list at the bottom.

## Grids and multi-image requests fall apart

**Symptom.** Asked for a 4x4 or 8x8 grid, got mush. Asked for six images, got four.

**Cause.** Token budget per image is fixed. Google states the model "won't always follow the exact number of image outputs" asked for. Woolf found quality collapses at 8x8 even at 4K.

**Fix.** 2x2 and 2x3 grids are reliable. For more, generate one image per turn with the character sheet attached.

## Stylized prompt came back photoreal

**Symptom.** You asked for an ugly cartoon, got a clean render.

**Cause.** Pro has a realism bias and pulls stylized prompts toward polish.

**Fix.** Name the medium and its imperfections: "risograph with visible misregistration", "claymation with fingerprints", "rough felt-tip marker sketch on lined paper, uneven line weight." Add a texture word. Use 3.1 Flash Image, which is less polished by default.

## "Pro" got worse overnight

**Symptom.** Results changed with no change on your side.

**Cause.** In February 2026 the Gemini app made Nano Banana 2 the default and moved true Pro behind the Regenerate menu. Preview model IDs were retired in July 2026.

**Fix.** In the API, pin `gemini-3-pro-image`. In the app, check which model is selected before comparing.

## Slow or timing out

**Symptom.** 20 to 60 seconds per image on Pro.

**Cause.** Pro runs a thinking pass that cannot be disabled. 4K roughly doubles output tokens.

**Fix.** Iterate on 3.1 Flash Image at 1K. Only send the final prompt to Pro at 2K or 4K. For volume, use the Batch API at half price. On Vertex, Provisioned Throughput buys a quota window, not speed: one GSU of Pro is roughly one image per 20 minutes.

## Watermark

Every image carries an invisible SynthID plus C2PA credentials. There is no setting to remove it. The visible sparkle in the corner became optional in the Gemini app in August 2026 and never appeared on API or AI Studio output.

## Seed and reproducibility

There is no seed parameter. Two calls with the same prompt give different images. The closest substitute: lock a specific prompt, use an approved output as a reference for the next generation, and over-generate then pick. If reproducibility is contractual, this is the wrong model.

## Output QC checklist

Before you ship:

- Hands: finger count, joints, grip
- Eyes: symmetry, alignment, catchlights
- Text: every letter, font weight, no ghost fragments
- Face against the reference, if there is one
- Limb count and bend physics
- Reflections and mirrors match the scene
- Logos and marks not warped or borrowed from a real brand
- Perspective and vanishing points on architecture and products
- Repeated tiles in fabric, brick, foliage
- Frame edges: nothing invented along the border if the crop matters
