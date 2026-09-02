---
name: nano-banana
description: Write a Nano Banana (Gemini image) prompt from a rough idea, following this repo's guides, and copy it to the clipboard. Asks clarifying questions only when the answer changes the prompt. Use for /nano-banana <what you want>, or any request to generate or edit an image with Gemini.
---

# /nano-banana

Turn `$ARGUMENTS` (a rough idea, in any language) into one production-ready Nano Banana prompt, copy it to the clipboard, and tell the user which model and settings to use.

The guides live in `docs/` at the repo root. If they are present, read the ones you need before writing. If this skill was installed globally and the docs are missing, the rules below are enough.

| Need | Read |
|---|---|
| Any prompt | `docs/prompting-guide.md` |
| The user attached or mentioned an existing image | `docs/editing-guide.md` |
| Photo, film, lighting, illustration or 3D style words | `docs/vocabulary.md` |
| The idea matches a known trend (figurine, isometric, try-on, restoration, infographic, headshot) | `docs/prompt-library.md`, start from that prompt |

## Step 1: classify

Decide from the request:

- **Mode**: generate (no source image) or edit (user has an image to change).
- **Kind**: photo, illustration, 3D, product, text-heavy (poster, logo, infographic), diagram, comic.
- **What is already specified**: subject, purpose, style, ratio, text, model.

## Step 2: ask only what changes the prompt

Use `AskUserQuestion`, one round, at most four questions, and skip every question the request already answers. Never ask about things you can default. Candidates, in priority order:

1. **Purpose and placement** if unknown and it matters: "Where will this go?" (social post, landing page hero, print, thumbnail, personal). Purpose changes composition and negative space.
2. **Aspect ratio** if unknown: offer 1:1, 4:5, 16:9, 9:16 with the likeliest first given the purpose.
3. **Style direction** if the request is ambiguous between photoreal and illustrated, or the illustration medium is open: offer three concrete directions with one-line descriptions, not "realistic / cartoon".
4. **Text on the image** if the kind is text-heavy and the exact string is missing: ask for the exact words.
5. **Edit scope** in edit mode if unclear: what must stay locked (face, background, pose) and what changes.

Defaults when not asked: 4:5 for people and products, 16:9 for scenes and heroes, 1:1 for icons and stickers. Model: `gemini-3.1-flash-image` for drafts, `gemini-3-pro-image` when there is text on the image, a face that must stay consistent, or the user said "final". Size: 1K for drafts, 2K for finals, 4K only on request.

If the request is clear and small, skip the questions entirely.

## Step 3: write the prompt

Always in English. Prose, not keyword lists. No `4k, masterpiece, trending on artstation`. No "no X": rewrite every absence as a positive description. Under about 150 words unless the constraints need more.

Generate mode, cover in this order and drop slots that do not apply:

1. Shot type and subject with two or three concrete physical details (material, color, texture, age).
2. Action, present tense.
3. Setting, time, weather.
4. Light: one source, its direction, its quality (soft, hard, warm, rim).
5. Camera: lens and aperture for photos; medium and texture for illustration; render style for 3D.
6. Style and grade: film stock, palette, era. Describe an artist's style by attributes, never by name.
7. Text: exact string in quotes, font named and described, position. Keep under a headline.
8. Composition: framing, where the empty space is and what it is for.
9. One imperfection for photorealism (grain, condensation, a stray hair).

Edit mode, structure as lock, change, constraints:

```
Using the provided image, keep [face, pose, background, lighting] exactly
the same. Change only [one thing] to [description]. Match the original
light direction, color temperature and shadow softness. Keep everything
else unchanged.
```

One change per prompt. If the user wants several changes, write them as numbered turns and say so. If several reference images are involved, give each a role: "Image 1 (the garment), Image 2 (the model)".

For faces that must persist: include the identity lock sentence from `docs/editing-guide.md` ("Keep the facial features of the person in the uploaded image exactly consistent: same eye shape and spacing, nose width, mouth shape, jawline, skin tone, skin texture, hair texture and apparent age. Do not beautify.").

For products with an invented brand: append "Use original trade dress; avoid real logos and certification marks."

## Step 4: copy to the clipboard

Write the prompt to a temp file first so quoting cannot break it, then copy. Only the prompt goes to the clipboard, not the settings.

```bash
f="$(mktemp)"; cat > "$f" <<'PROMPT'
<prompt text>
PROMPT
if command -v pbcopy >/dev/null; then pbcopy < "$f"
elif command -v wl-copy >/dev/null; then wl-copy < "$f"
elif command -v xclip >/dev/null; then xclip -selection clipboard < "$f"
elif command -v clip.exe >/dev/null; then clip.exe < "$f"
else echo "no clipboard tool"; fi; rm -f "$f"
```

If no clipboard tool exists, say so and show the prompt; do not silently skip.

## Step 5: report

Show, in this order and nothing else:

1. The prompt in a fenced block.
2. One settings line: model, aspect ratio, size, and whether search grounding is on.
3. For multi-turn edits: the follow-up turns, numbered.
4. One line on where to set the ratio (API `response_format.aspect_ratio`, or upload a blank canvas at that ratio in the Gemini app).

Then: "Copied to clipboard." Nothing after that.
