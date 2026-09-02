# Awesome Nano Banana

A practical guide to getting images out of Google's Gemini image models (Nano Banana) that look like someone with taste made them. Verified against Google's docs and independent testing as of September 2026. Every non-obvious claim links to its source.

## Which model

| Nickname | Model ID | Use it for |
|---|---|---|
| Nano Banana Pro | `gemini-3-pro-image` | The frame you ship. Best text rendering, 3 style refs, 5 character refs, 4K. Slowest and priciest. |
| Nano Banana 2 | `gemini-3.1-flash-image` | Daily driver. 512 to 4K, 10 object refs, video input, half the price of Pro. |
| Nano Banana 2 Lite | `gemini-3.1-flash-lite-image` | Bulk and low latency. 1K only, no search grounding. |
| Nano Banana (original) | `gemini-2.5-flash-image` | Legacy. Max 3 input images. Only if you are already on it. |

Anything with a `-preview` suffix was retired in July 2026 and returns an error.

Iterate on Nano Banana 2 at 1K. Switch to Pro at 2K or 4K for the final render. Per-image prices are in [docs/api.md](docs/api.md).

## The five rules

1. **Describe the scene in sentences.** Gemini reads prose. `dog, park, 4k, masterpiece` is noise. "A golden retriever shaking water off after a swim, late afternoon sun behind it, 85mm at f/2" is a brief.
2. **Use the vocabulary of the craft.** Lens, aperture, lighting setup, film stock, composition. Each term moves the output in a known direction. See [docs/vocabulary.md](docs/vocabulary.md).
3. **There is no negative prompt.** Say what is there. "An empty street at dawn, no cars" works because "empty" and "dawn" carry the meaning, not because "no" does.
4. **Edit, don't re-roll.** One change per turn. Name what must stay the same before you name what changes. After three or four edits, restart from the original.
5. **Text goes in quotes, short, with a named font.** Under a headline's worth. Generate the copy first in text, then ask for the image with that copy.

## Guides

| File | What it covers |
|---|---|
| [docs/prompting-guide.md](docs/prompting-guide.md) | How to structure a prompt. Google's templates. Photorealism, illustration, products, text, negative space, comics. |
| [docs/editing-guide.md](docs/editing-guide.md) | Editing existing images. Identity lock, composition lock, multi-image fusion, character consistency, aspect ratio tricks, transparent backgrounds. |
| [docs/vocabulary.md](docs/vocabulary.md) | Glossary. Lenses, film stocks, lighting, color grades, composition, illustration and 3D styles, with what each term does to the image. |
| [docs/prompt-library.md](docs/prompt-library.md) | Copy-paste prompts for the viral use cases: figurines, isometric buildings, try-on, restoration, infographics, headshots, storyboards. |
| [docs/troubleshooting.md](docs/troubleshooting.md) | Failure modes and the workaround for each: face drift, text garbling, color shift, refusals, returned-unchanged, quality decay over turns. |
| [docs/api.md](docs/api.md) | Calling it from code. Model IDs, prices, aspect ratio and size fields, reference caps, limits, Vertex differences. |

## Quick start in the Gemini app or AI Studio

Paste this, swap the brackets:

```
A photorealistic [shot type] of [subject with two or three concrete details],
[doing what], in [setting]. Lit by [light source and quality], creating a
[mood] atmosphere. Shot on [lens] at [aperture], emphasizing [texture or
detail]. [Aspect ratio].
```

Then edit conversationally: "Keep everything the same, but make the light warmer." Not a new prompt.

## Claude Code slash command

The repo ships a `/nano-banana` skill for [Claude Code](https://claude.com/claude-code). Give it a rough idea, it asks only the questions that change the prompt, writes one prompt by these guides, copies it to your clipboard, and tells you which model and settings to use.

Inside this repo it works as is:

```
claude
/nano-banana a hero image for a coffee brand landing page, warm, with room for a headline
```

To use it from any directory, copy the skill into your user skills:

```bash
cp -r .claude/skills/nano-banana ~/.claude/skills/
```

Clipboard support: `pbcopy` on macOS, `wl-copy` or `xclip` on Linux, `clip.exe` on WSL. Source: [.claude/skills/nano-banana/SKILL.md](.claude/skills/nano-banana/SKILL.md).

## Sources this guide leans on

- Google's image generation docs: https://ai.google.dev/gemini-api/docs/image-generation
- Google's prompting guide for Nano Banana: https://cloud.google.com/blog/products/ai-machine-learning/ultimate-prompting-guide-for-nano-banana
- Google developers blog on prompting 2.5 Flash Image: https://developers.googleblog.com/en/how-to-prompt-gemini-2-5-flash-image-generation-for-the-best-results/
- Max Woolf's hands-on test of Nano Banana Pro: https://minimaxir.com/2025/12/nano-banana-pro/
- Chase Jarvis's JSON prompting A/B test: https://chasejarvis.com/blog/does-json-prompting-actually-work-tested-with-nano-banana/
- Banana100, the paper on quality decay over sequential edits: https://arxiv.org/abs/2604.03400
- Prompt archives: [PicoTrex/Awesome-Nano-Banana-images](https://github.com/PicoTrex/Awesome-Nano-Banana-images), [ZeroLu/awesome-nanobanana-pro](https://github.com/ZeroLu/awesome-nanobanana-pro), [JimmyLv/awesome-nano-banana](https://github.com/JimmyLv/awesome-nano-banana)

## Contributing

Open a PR with a prompt and the image it produced, or a fix with a source. Prompts without an attribution or a result image get closed.
