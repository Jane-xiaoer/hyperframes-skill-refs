# Prompt Guide (HyperFrames)

> Source: https://hyperframes.heygen.com/guides/prompting
> How to prompt AI agents to author HyperFrames — two prompt shapes, vocabulary tables, iteration, anti-patterns.

**Read this when** you're about to give an agent a brief for a composition, or when output drifts from intent.

## One-time Setup

```bash
npx skills add heygen-com/hyperframes
```

After install, restart the session. Skills register as slash commands:

| Slash command      | Loads                                                                 |
| ------------------ | --------------------------------------------------------------------- |
| `/hyperframes`     | Composition authoring — HTML, timing, captions, TTS, transitions      |
| `/hyperframes-cli` | CLI — `init`, `lint`, `preview`, `render`, `transcribe`, `tts`        |
| `/gsap`            | GSAP API — timelines, easing, ScrollTrigger, plugins                  |

**Always prefix prompts with `/hyperframes`** so the agent applies composition rules from the start.

---

## The Two Prompt Shapes

### Cold Start — describe the video from scratch

> "Using `/hyperframes`, create a 10-second product intro with a fade-in title over dark background and subtle background music."

> "Make a 9:16 TikTok-style hook video about [topic] using `/hyperframes`, with bouncy captions synced to TTS narration."

Specify for best results:
- **Duration** — "10 seconds", "30s", "5 scenes of 3s each"
- **Aspect ratio** — "16:9", "9:16 vertical", "1:1 square"
- **Mood/style** — "minimal Swiss grid", "warm grain analog", "high-energy social"
- **Key elements** — title, lower third, captions, background video, music

### Warm Start — turn context into video

> "Take a look at this GitHub repo [URL] and explain its uses and architecture using `/hyperframes`."
> "Summarize the attached PDF into a 45-second pitch video using `/hyperframes`."
> "Read this changelog and turn the top three changes into a 30-second release announcement video using `/hyperframes`."
> "Turn this CSV into an animated bar chart race using `/hyperframes`."

Warm-start yields grounded video because the agent writes about specific content rather than inventing copy.

---

## Iterating

Treat HyperFrames like a conversation with a video editor. Small targeted edits beat complete re-specifications:

- "Make the title 2x bigger."
- "Swap to dark mode."
- "Add a fade-out at the end and a lower third at 0:03 with my name and title."
- "The captions are too small and overlap the lower third. Move them up and shrink them."
- "Replace the background music with `assets/track.mp3`."

The agent already has context and skills loaded — don't re-brief.

---

## Vocabulary That Changes Output

### Motion & Easing

| Say this  | Agent uses    | Feels like              |
| --------- | ------------- | ----------------------- |
| smooth    | `power2.out`  | Natural deceleration    |
| snappy    | `power4.out`  | Quick and decisive      |
| bouncy    | `back.out`    | Overshoots then settles |
| springy   | `elastic.out` | Oscillates into place   |
| dramatic  | `expo.out`    | Fast start, long glide  |
| dreamy    | `sine.inOut`  | Slow, symmetrical       |

**Timing shorthand:**
- fast (0.2s) = energy
- medium (0.4s) = professional
- slow (0.6s) = luxury
- very slow (1–2s) = cinematic

### Caption Tones

| Tone         | Typography         | Animation    | Size range |
| ------------ | ------------------ | ------------ | ---------- |
| Hype         | Heavy weight fonts | Scale-pop    | 72–96px    |
| Corporate    | Clean sans-serif   | Fade + slide | 56–72px    |
| Tutorial     | Monospace          | Typewriter   | 48–64px    |
| Storytelling | Serif              | Slow fade    | 44–56px    |
| Social       | Rounded, playful   | Bounce       | 56–80px    |

Per-word styling phrases that work: "scale-pop", "slow fades", "karaoke-style", "brand names larger with accent color", "bounce on emotional keywords", "highlight numbers differently".

### Transitions

| Energy | CSS option     | Shader option       |
| ------ | -------------- | ------------------- |
| Calm   | Blur crossfade | Cross-warp morph    |
| Medium | Push slide     | Whip pan            |
| High   | Zoom through   | Glitch, ridged burn |

Mood descriptions that work: "warm for wellness brand", "cold clinical for tech", "playful bouncy", "dramatic zoom for the reveal".

### Audio-Reactive Animation

| Audio band | Maps to   | Visual effect     |
| ---------- | --------- | ----------------- |
| Bass       | `scale`   | Pulse on the beat |
| Treble     | `glow`    | Shimmer intensity |
| Amplitude  | `opacity` | Breathing         |
| Mids       | `shape`   | Morphing          |

Keep text reactivity subtle (3–6% intensity), background reactivity larger (10–30%).

### Marker Highlights

| Mode        | Effect             | Best for     |
| ----------- | ------------------ | ------------ |
| `highlight` | Marker sweep       | Key phrases  |
| `circle`    | Hand-drawn ellipse | Single words |
| `burst`     | Radiating lines    | Hype moments |
| `scribble`  | Chaotic scratch    | Crossing out |
| `sketchout` | Rectangle outline  | Callouts     |

### Text-to-Speech Voices

Kokoro runs locally, no API key required.

| Content type | Recommended voices     |
| ------------ | ---------------------- |
| Product demo | `af_heart`, `af_nova`  |
| Tutorial     | `am_adam`, `bf_emma`   |
| Marketing    | `af_sky`, `am_michael` |

### Rendering Quality

| Quality    | Use for             |
| ---------- | ------------------- |
| `draft`    | Fast iteration      |
| `standard` | Review and feedback |
| `high`     | Final delivery      |

---

## Rules to Know

Technical requirements — breaking them produces incorrect renders:

1. **Register all timelines** on `window.__timelines` — the renderer cannot seek animations it doesn't know about.
2. **Video elements must be `muted`** — audio goes in separate `<audio>` elements.
3. **No `Math.random()`** — breaks determinism. Use seeded PRNG (e.g., mulberry32).
4. **Synchronous timeline construction** — no `async`/`await` or `fetch()` during GSAP setup.
5. **Timed elements need `class="clip"`** — plus `data-start`, `data-duration`, `data-track-index`.

Best practices (skill applies by default):

6. **Entrance animations on every scene** — elements appearing without animation feel broken.
7. **Transitions between scenes** — jump cuts are almost always unintentional.

---

## Anti-Patterns

- ❌ **Don't ask for React/Vue components.** HyperFrames = plain HTML + `data-*` + GSAP timelines.
- ❌ **Don't ask for 4K or 60fps unless necessary.** Defaults (1920×1080, 30fps) render fast and look excellent.
- ❌ **Don't skip the slash command.** Without `/hyperframes` the agent guesses at video conventions.
- ❌ **Don't paste long error logs without context.** Run `npx hyperframes lint` + `npx hyperframes validate` first.
- ❌ **Don't assume agents know your assets.** Mention file paths explicitly (`assets/intro.mp4`, `assets/logo.png`).

---

## Recommended Workflow

1. `npx hyperframes init my-video` — scaffold (skills auto-install)
2. Open in Claude Code / Cursor / Codex
3. Prompt with `/hyperframes` + one of the shapes above
4. `npx hyperframes preview` — watch in browser as agent edits
5. Iterate with small targeted prompts
6. `npx hyperframes render --output final.mp4` when satisfied
