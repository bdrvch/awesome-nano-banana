# Prompting guide

How to write a prompt that Nano Banana turns into something worth keeping. Everything here applies to `gemini-3-pro-image`, `gemini-3.1-flash-image` and `gemini-3.1-flash-lite-image`. The older `gemini-2.5-flash-image` follows the same rules with weaker text rendering.

## Why prose beats keywords

Stable Diffusion was trained on comma-separated tags through a short CLIP encoder, so keyword stacking worked there. Gemini's image models sit on the same language model that writes text. They read your prompt as a description, not as a tag cloud. Google says it directly in its prompting guide: "A simple list of keywords won't cut it; you need to describe the scene narratively." ([source](https://cloud.google.com/blog/products/ai-machine-learning/ultimate-prompting-guide-for-nano-banana))

That kills the old boilerplate. `masterpiece, 8k, ultra detailed, trending on artstation` carries no visual information and crowds out the words that do. Drop it.

The one place prose loses: when a prompt has many hard constraints at once (five subjects, each with a fixed position, color and pose), Max Woolf found structured, line-broken prompts with capitalised imperatives like `All kittens MUST follow these descriptions EXACTLY` got better adherence than a flowing paragraph. ([source](https://minimaxir.com/2025/12/nano-banana-pro/))

So the working rule is: prose for description, structure for constraints. One prompt can carry both.

## The structure

Google's own formula, from its prompting guide:

```
[Subject] + [Action] + [Location or context] + [Composition] + [Style]
```

The expanded version that covers most real prompts:

| Slot | What goes there | Example |
|---|---|---|
| Subject | Who or what, with two or three concrete details | "an elderly fisherman with a silver beard and a weathered orange raincoat" |
| Action | What they are doing, in the present tense | "coiling a rope on the deck" |
| Setting | Where, when, weather | "a small wooden boat in a Norwegian fjord at golden hour" |
| Lighting | Source, direction, quality | "low sun from camera-left, warm rim light on the beard" |
| Camera | Shot type, lens, aperture | "medium close-up, 85mm at f/1.8, shallow depth of field" |
| Style and grade | Film stock, era, render style, palette | "Kodak Portra 400, fine grain, muted teal shadows" |
| Text | Exact string in quotes, font, position | "the word 'NORD' in a heavy condensed sans-serif, lower third" |
| Format | Aspect ratio | "4:5" |

Google's canonical worked example, verbatim:

```
A striking fashion model wearing a tailored brown dress, sleek boots, and
holding a structured handbag. Posing with a confident, statuesque stance,
slightly turned. A seamless, deep cherry red studio backdrop. Medium-full
shot, center-framed. Fashion magazine style editorial, shot on medium-format
analog film, pronounced grain, high saturation, cinematic lighting effect.
```

## Google's six best practices

Verbatim from the [image generation docs](https://ai.google.dev/gemini-api/docs/image-generation):

- **Be hyper-specific.** Instead of "fantasy armor," describe it: "ornate elven plate armor, etched with silver leaf patterns, with a high collar and pauldrons shaped like falcon wings."
- **Provide context and intent.** "Create a logo for a high-end, minimalist skincare brand" yields better results than "Create a logo."
- **Iterate and refine.** Follow up with "That's great, but can you make the lighting a bit warmer?" or "Keep everything the same, but change the character's expression to be more serious."
- **Use step-by-step instructions.** "First, create a background of a serene, misty forest at dawn. Then, in the foreground, add a moss-covered ancient stone altar. Finally, place a single, glowing sword on top of the altar."
- **Use semantic negative prompts.** Instead of "no cars," describe the scene positively: "an empty, deserted street with no signs of traffic."
- **Control the camera.** Use photographic and cinematic language: `wide-angle shot`, `macro shot`, `low-angle perspective`.

Two more from the Vertex docs: start with "create an image of" or the model may answer in text, and pass thought signatures back in multi-turn editing so reasoning context carries across turns.

## Templates by use case

These are Google's published templates. They are skeletons, not incantations. Fill every bracket with something concrete.

### Photorealistic scene

```
A photorealistic [type of shot] of a [subject description] in a [setting
description]. [Description of the light]. Shot from a [camera angle]
with a [lens type].
```

Google's example: "A photorealistic wide-angle shot of a vibrant coral reef teeming with tropical fish. Crystal-clear turquoise water with sunbeams filtering down from the surface, illuminating a sea turtle gliding gracefully over the coral. Shot from a low perspective with a wide-angle lens. Aspect ratio 16:9."

What makes realism read as real: a named lens and aperture, one light source with a direction, a film stock or a camera body as style shorthand, and one imperfection (grain, a stray hair, condensation, a fingerprint on glass). Perfectly clean output looks like a render.

### Stylized illustration and stickers

```
A [style] of a [subject, with details about accessories or actions]
doing [activity]. The design features [visual qualities, e.g., bold outlines,
cel-shading, etc.] and [color/background preference].
```

Google's example: "A kawaii-style sticker of a happy red panda wearing a tiny bamboo hat. It's munching on a green bamboo leaf. The design features bold, clean outlines, simple cel-shading, and a vibrant color palette. The background must be white."

Name the medium (gouache, risograph, ligne claire) and the texture (paper tooth, halftone). The [vocabulary](vocabulary.md) has a full list. Describe the style by its attributes rather than by an artist's name; it transfers better and survives policy changes.

### Text in images

```
Create a [image type] for [brand/concept] with the text "[text to render]"
in a [font style]. The design should be [style description], with a
[color scheme].
```

Rules that hold up in testing:

- Put the exact string in quotes.
- Name the font and describe it: "heavy blocky Impact font" beats either half alone. The model knows widely used faces (Helvetica, Times, Roboto, Fira Code, Comic Sans were verified by Woolf).
- Keep it under a headline's worth. Single-line text is where Google claims sub-10% error rates; paragraphs still garble.
- For several text elements, give each its own line, position and style.
- Google's text-first trick: generate the copy in a text turn first, then ask for the image containing that copy.

Google's multi-text example:

```
A high-end, glossy commercial beauty shot of a sleek, minimalist nude-colored
face moisturizer jar resting on a warm studio background. The lighting is soft
and radiant. Next to the product, render three lines of text with the
following exact styling: For the top line, the word 'GLOW' in a flowing,
elegant Brush Script font. For the middle line, the text '10% OFF' in a heavy,
blocky Impact font. For the bottom line, the text 'Your First Order' in a
thin, minimalist Century Gothic font.
```

### Product mockups

```
A high-resolution, studio-lit product photograph of a [product description]
on a [background surface/description]. The lighting is a [lighting setup,
e.g., three-point softbox setup] to [lighting purpose]. The camera angle is
a [angle type] to showcase [specific feature]. Ultra-realistic, with sharp
focus on [key detail]. [Aspect ratio].
```

Describe the material, not the object: "matte-black stainless steel with a brushed cap" rather than "bottle". Add a contact shadow or the product floats. If the product is invented, add "use original trade dress, avoid real logos and certification marks" so it does not borrow a real brand.

### Minimalist and negative space

Backgrounds for landing pages and slides where copy goes on top.

```
A minimalist composition featuring a single [subject] positioned in the
[bottom-right/top-left/etc.] of the frame. The background is a vast, empty
[color] canvas, creating significant negative space. Soft, subtle lighting.
[Aspect ratio].
```

Say where the empty space is and what it is for: "the right 40% is an unbroken expanse of backdrop reserved for headline copy."

### Sequential art

```
Make a 3 panel comic in a [style]. Put the character in a [type of scene].
```

Google recommends Pro or 3.1 Flash Image for this. For anything longer than three panels, generate one panel per turn with the character sheet attached, not a grid. Grids past 4x4 lose quality.

### Grounded in search

Pro and 3.1 Flash Image can run Google Search before rendering. Google's example: "Make a simple but stylish graphic of last night's Arsenal game in the Champion's League." Useful for infographics on recent events. It renders numbers as pixels and does not validate them, so supply the data yourself when accuracy matters.

## Structured and JSON prompts

A wave of content claims JSON prompts give "100% accuracy" or stop concept bleeding. No Google source says this. The only controlled test found is Chase Jarvis's: he converted an image to JSON, converted that back to prose, ran both twice in Pro, and got images that "all look essentially the same, with the expected random variation." ([source](https://chasejarvis.com/blog/does-json-prompting-actually-work-tested-with-nano-banana/))

JSON is a fine templating format for a pipeline that generates thousands of images and needs version control. Its values should still be descriptive phrases. It is not a comprehension aid.

## Aspect ratio and size

Set them in the API, not in the prompt. Field names and casing are in [api.md](api.md). Mentioning "16:9" inside the prompt text helps the composition but does not reliably change the output dimensions. In the Gemini app, where there is no picker, upload a blank PNG at the target ratio and say "use this canvas". Details in the [editing guide](editing-guide.md).

## Checklist before you hit enter

- Is there a subject with at least two concrete physical details?
- Is there one light source with a direction?
- Is there a lens or shot type?
- Is every "no X" rewritten as a positive description?
- Is any text in quotes, short, with a font?
- Is the aspect ratio set in config?
- Is it under about 150 words unless the constraints need more?
