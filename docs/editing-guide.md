# Editing guide

Editing is where Nano Banana beats Midjourney and Flux. It takes a photo, a sketch, a floor plan or a product shot and changes one thing while keeping the rest. This guide is about making "keeps the rest" actually hold.

## How an edit turn works

The model does not nudge pixels. It rebuilds the frame each turn using your image as a reference. That is why identity drifts, colors shift and small details get simplified over several turns. Everything below is a way of fighting that rebuild.

## Lock, then change

The most repeated advice across Google, fal.ai and the best community prompts: name what must stay unchanged before you name what changes.

```
Lock: the person's face, pose, expression, the background, and the lighting.
Change: replace the red coat with an emerald green wool coat.
Constraints: match the original light direction and shadow softness.
```

Google's template for the same thing:

```
Using the provided image, change only the [specific element] to [new
element/description]. Keep everything else in the image exactly the same,
preserving the original style, lighting, and composition.
```

Restate the lock every turn. It does not persist on its own.

## One change per turn

Two edits in one message produce a compromise on both and you cannot tell which instruction caused the problem. Google's phrase for this is "Edit, don't re-roll." Ask for the coat color, look at the result, then ask for the rain.

```
Turn 1: Keep her pose, expression and the street exactly the same. Change
only the coat from red to emerald green.

Turn 2: Keep the coat, pose and lighting as they are now. Change only the
weather to light rain with wet reflections on the pavement.

Turn 3: Everything else identical. Change only the time of day to golden
hour, adjusting light direction and color temperature to match.
```

## Restart after three or four edits

Quality decay over sequential edits is measured, not anecdotal. The Banana100 paper edited 13 seed images through 100 steps each and found artifacts compound into visible noise and instruction-following breaks down, while none of 21 image-quality metrics reliably caught it. ([source](https://arxiv.org/abs/2604.03400))

The working cap is two to four edits. Then start a fresh conversation, attach the original, and describe the full target state in one prompt. If the model has drifted a character's face, Google's own advice is to restart with a detailed description rather than keep correcting.

## Identity lock for faces

The phrasing that recurs across independent sources:

```
Keep the facial features of the person in the uploaded image exactly
consistent: same eye shape and spacing, nose width, mouth shape, jawline,
skin tone, skin texture, hair texture and apparent age. Do not beautify
or replace the face.
```

Reference image quality matters more than the wording. Use 1024px or larger, even frontal light, no glasses or hat, neutral expression. Two references (front and 45 degrees) beat one. References that disagree on hairstyle, age or lighting get averaged into nobody.

Two failure-specific clauses people add:

- Against the "puffy face" drift: "natural face proportions, no puffy cheeks, no plastic or waxy skin, natural catchlights in the eyes."
- Against beautification: "render natural skin texture with visible pores, not an airbrushed look."

Google's high-fidelity template for the same problem:

```
Using the provided images, place [element from image 2] onto [element from
image 1]. Ensure that the features of [element from image 1] remain
completely unchanged. The added element should [description of how the
element should integrate].
```

## Character consistency across a series

A character bible: a fixed identity paragraph you paste verbatim into every prompt, plus a multi-angle character sheet used as a reference image. Vary exactly one axis per generation (scene, or emotion, or wardrobe).

```
CHARACTER: Mira, late twenties, Nigerian-British, heart-shaped face, warm
dark-brown skin with a matte finish, almond-shaped dark brown eyes, 4C coily
hair in a high puff, a small scar above the left eyebrow, athletic-slim
build. Olive-green flight jacket over a cream turtleneck, dark denim, worn
tan combat boots.

Using the CHARACTER above and the attached character sheet exactly, show
Mira laughing mid-conversation in a crowded night market, neon signage
reflected in her eyes, 35mm handheld framing at f/2. Keep her face, hair and
wardrobe identical. 3:2.
```

To make the sheet in the first place:

```
Generate the front, rear, left, right, top and bottom views of this
character on a plain white background, evenly spaced, consistent subject,
same lighting on every view.
```

Give the character a name once and use the name in later turns instead of re-describing the face. Include previously approved outputs as references in subsequent prompts; Google recommends this explicitly.

Reference caps per model are in [api.md](api.md). The caps are ceilings. Two to four clean references beat fourteen mixed ones.

## Multi-image fusion

Give every image a role in the text. Do not upload five images and hope.

```
Using Image 1 (the garment) and Image 2 (the model), create a full-body
fashion photo where the model wears the garment. The garment must drape
naturally on the model's body with realistic folds. Preserve the fabric
texture, color and any logos from Image 1 exactly. Match Image 2's ambient
lighting, color temperature and shadow direction.
```

Google's own formula: `[Reference images] + [Relationship instruction] + [New scenario]`.

```
Using the attached napkin sketch as the structure and the attached fabric
sample as the texture, transform this into a high-fidelity 3D armchair
render. Place it in a sun-drenched, minimalist living room.
```

Style refs (Pro only, up to 3) work the same way: "Take the color palette and brushwork from Image 3."

Pre-resize references to 2K or smaller. Oversized uploads get compressed server-side and one report measured a saturation drop from it.

## Adding and removing

```
Using the provided image of [subject], please [add/remove/modify] [element]
to/from the scene. Ensure the change is [description of how the change
should integrate].
```

"Remove the man from the photo" works without a mask. The model reads the semantic region. For a specific object among several, describe it by position and appearance ("the blue mug on the left edge of the table"), or in the Gemini app draw on the image: since December 2025 you can circle, arrow or scribble directly on the photo instead of describing the region.

## Style transfer and sketch to render

```
Transform the provided photograph of [subject] into [art style]. Preserve
the original composition but render it with [description of stylistic
elements].
```

```
Turn this rough pencil sketch of a [subject] into a [style description]
photo. Keep the [specific features] from the sketch but add [new
details/materials].
```

Describe the style by attributes rather than by artist name. "Thick swirling impasto with visible directional ridges, cobalt blue against golden yellow" gives you Van Gogh's brush on your subject. "In the style of Van Gogh" collapses toward Starry Night.

## Aspect ratio

Through the API, set `aspect_ratio` in config. The model otherwise matches the input image's dimensions, or defaults to 1:1.

In the Gemini app there is no picker. The community workaround is a blank canvas: upload a transparent or white PNG at the target ratio as an extra image and write "use the uploaded blank image as the canvas, fill it entirely." The two-step variant: upload your photo as Image 1 and a blank 16:9 as Image 2, ask it to redraw Image 1's content onto Image 2.

On multi-reference edits the output sometimes takes the ratio of the first reference and sometimes the last. Set it explicitly.

## Transparent backgrounds and cut-outs

Simple: "extract the [subject] and put it on a transparent background." The output is a PNG with a checkerboard or a plain background depending on surface, so verify.

Rigorous, from a Medium write-up by jidefr: generate the subject on pure white, feed that back and ask for the identical render on pure black, then compute the alpha from the luminance difference between the two. Clean edges on hair and glass.

Outpainting: upload an image with transparent margins and say "repair the checkerboard (transparent) parts of the image and restore a complete, coherent photo."

## Upscaling and detail passes

```
Upscale this image to 4K. Enhance skin with realistic pores and texture that
matches the person's age. Keep the enhancement subtle. Do not over-smooth or
add artificial shine. Maintain natural imperfections.
```

To pull a higher-resolution render of an earlier result without altering it, set the size to 4K and send "Don't change anything." One detail pass, not two. Stacking Nano Banana's detail pass with an external upscaler at full strength gives crunchy pores and halos around text.

## Camera moves on an existing image

```
Convert the photo to a top-down view and mark where the photographer was
standing.
```

```
Show the building from a bird's eye view, revealing the ocean behind it.
```

Watch the frame edges. An arXiv evaluation found the model tends to invent content along the periphery instead of respecting the original crop. If the crop matters, say "keep the exact original framing and boundaries."

## Multi-turn in the API

Pass `previous_interaction_id` (Interactions API) or keep the chat history with thought signatures (legacy generateContent) so the model carries context instead of re-reading a re-upload. Keep the whole request under 50MB. Details in [api.md](api.md).
