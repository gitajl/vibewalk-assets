# Mascot assets

Drop the mascot GIFs in this folder. They're consumed by two surfaces:

1. **The MCP server** reads them at runtime, base64-encodes them, and embeds them in tool responses as `ImageContent` blocks. Hosts that render inline images (Claude desktop, ChatGPT desktop with MCP) show the mascot above the itinerary.
2. **The mobile Custom GPT** (see `docs/MOBILE_GPT.md`) references the same files via public URL — when you also host these on GitHub Pages or Cloudflare Pages, the GPT instructions point at them by URL and the mascot renders inline in ChatGPT mobile.

Use the **same five files in both places** so the desktop MCP build and the mobile GPT are visually consistent.

If a file isn't here, the MCP server quietly falls back to text-only — no errors. You can ship code first and add GIFs later without code changes.

## Files this folder needs

| Filename | When it shows | Notes |
|---|---|---|
| `presenting.gif` | After `plan_vibe_walk` returns a fresh walk | The headline state. Mascot holds up an itinerary / map. |
| `revising.gif` | After `refine_walk` returns a revision | Mascot scribbles or reroutes. |
| `anchor_confirmed.gif` | After `find_anchor` resolves a start | Mascot points at a location pin. |
| `celebrating.gif` | Optional — host can request it for "all done" moments | Tiny celebration / thumbs up. |
| `confused.gif` | Returned on errors | Mascot scratches head. |

## Specs (treat these as the spec, not suggestions)

| Setting | Value | Why |
|---|---|---|
| **Format** | GIF (animated) | Best universal compatibility across ChatGPT mobile, ChatGPT web, Claude desktop, and other MCP hosts. APNG/WebP look prettier but render as static (first frame only) in many chat clients. |
| **Aspect ratio** | 1:1 (square) | Never gets awkwardly cropped in any chat container. Matches how mascot/avatar art is naturally composed. |
| **Dimensions** | **512 × 512 px** | Crisp on retina mobile and desktop. Chat clients downscale anything larger anyway, so 768/1024 just bloats file size for no visible gain. 256 × 256 is acceptable if file size becomes a problem (slightly soft on retina desktop, fine on phones). |
| **File size** | **Under 300 KB each, hard cap 500 KB** | The MCP server base64-encodes them, inflating size ~33% in every tool response. Mobile networks may also lazy-load or skip pre-fetch on large GIFs. A 512 × 512 character GIF with a 32-color palette and 12–15 frames lands well under 200 KB without effort. |
| **Frame rate** | 12–15 fps | Smooth enough to feel alive, low enough to keep file size down. |
| **Loop length** | 1–2 seconds | Long enough to feel intentional, short enough to not be annoying. Loops infinitely. |
| **Color palette** | 32 colors, optimized | Limited palette is what makes GIFs small. Tools like [ezgif.com optimizer](https://ezgif.com/optimize) handle this in one click. |
| **Style** | Charming-but-tasteful, not childish | Per the product spec. Think "thoughtful little character," not Saturday morning cartoon. |
| **Background** | Transparent **with hard edges**, OR mascot on a small "shadow disc" / stage | See the section below — this matters more than people think. |

### Background, in detail

GIF only supports 1-bit transparency (binary on/off, no alpha channel). That makes anti-aliased edges look ugly against any background that isn't the exact color the artist designed against. Two clean options:

- **Transparent + hard pixel edges.** Pixel-art-ish style. Looks intentional. Works on light or dark mode chat. This is the safest call.
- **Mascot on a small "stage" or shadow disc.** The figure sits on a small opaque pad (an ellipse shadow, a tiny platform, a soft gradient circle), transparent everywhere else. Grounds the character without needing to match the chat background.

**Avoid:** a soft pastel rectangular background. It looks great in ChatGPT light mode and bizarre in dark mode. ChatGPT users on mobile especially split between modes — there's no safe default color.

### Character direction (locked)

The mascot is a chill walking person drawn in **hand-drawn editorial illustration style** — bold confident black ink linework, intentionally loose, contemporary spot-illustration aesthetic (think Tom Froese / Brett Ryder adjacent). Not Pixar. Not 3D. Not flat-vector brand mascot.

Strict three-color palette:

- Black ink for linework
- One accent color: soft indigo periwinkle (`~#8B8BDD`)
- Cream paper background inside the character art (`~#F5F0E8`)

Defining accessory: a vintage **Leica rangefinder camera** on a leather strap around the neck. The camera is the character's identity — it ties to the "photo moments" stop role in the product.

Stage container (composited in post-production): a soft cool-gray rounded square (`~#E8E5E0`, ~16% corner radius) sized 512 × 512.

See [`docs/MASCOT_PRODUCTION.md`](../../docs/MASCOT_PRODUCTION.md) for the full Midjourney workflow and prompts.

### Per-state expression notes

Same character, five different beats. The Leica camera is always present (resting at the chest in most states, in-hand in `anchor_confirmed`).

| State | Mood / pose |
|---|---|
| `presenting.gif` | Holds up a small unrolled paper map and gestures toward it. Warm friendly smile, slight excited body language. Camera at chest. The headline state — users see this most. |
| `revising.gif` | Holding a small notebook and pencil, scribbling. Brow slightly furrowed in focused concentration, head tilted. Camera at chest. |
| `anchor_confirmed.gif` | Holding the Leica up to one eye as if just took a perfect photo, the other hand giving a small thumbs-up. Satisfied beat. |
| `celebrating.gif` | Mid-stride little hop with both arms raised triumphantly, eyes closed in quiet joyful smile. A few small indigo sparkles around the figure. Camera bouncing on its strap. Brief warmth, not a parade. |
| `confused.gif` | Scratching the back of the head, slight shrug, small indigo question mark hovering above. Apologetic but warm — friend who lost the map, not failure state. |

## Producing them

If you don't have an animator, options that have worked in the past:

- Hand-draw frames, assemble + optimize with [ezgif.com](https://ezgif.com)
- Generate stills with an image model, animate with [Runway](https://runwayml.com) or [Pika](https://pika.art), export to GIF
- Commission a small set on Fiverr / Upwork — five looping 512 × 512 GIFs is usually a quick gig (~$50–200 depending on style)
- Use [LottieFiles](https://lottiefiles.com/free-animations/mascot) for inspiration; export Lottie → GIF if you find one

### Quick QA checklist before checking files in

For each GIF:

- [ ] Square (1:1)
- [ ] 512 × 512 px (or 256 × 256 if you had to)
- [ ] Under 300 KB on disk (run through [ezgif optimizer](https://ezgif.com/optimize) if not)
- [ ] Loops cleanly — last frame transitions smoothly to first
- [ ] Background is transparent with hard edges, OR has a small stage/shadow under the figure
- [ ] Looks intentional in BOTH ChatGPT light mode and dark mode (open the file in a browser with each, eyeball it)
- [ ] Same character, recognizable across all five states

## What's missing in this version

The original spec also lists `welcome`, `listening`, and `thinking` states. Those need a UI surface that streams updates over time — an MCP tool response is a single moment, not a sequence. They're parked for an Apps SDK / web companion build. See `docs/ARCHITECTURE.md` decision #1.
