# Quickstart (HyperFrames)

> Source: https://hyperframes.heygen.com/quickstart
> Create, preview, and render your first HyperFrames video in under two minutes.

**Read this when** setting up a new project or onboarding someone who's never touched HyperFrames.

## Prerequisites

```bash
node --version    # needs 22+
```

FFmpeg:
- macOS: `brew install ffmpeg`
- Ubuntu/Debian: `sudo apt install ffmpeg`
- Windows: `winget install ffmpeg`
- Verify: `ffmpeg -version`

## Scaffold a project

### Option A — AI-agent way (recommended)
Install the skills for your coding agent:
```bash
npx skills add heygen-com/hyperframes
```
Registers `/hyperframes`, `/hyperframes-cli`, `/gsap` slash commands.

### Option B — manual
```bash
npx hyperframes init my-video
cd my-video
```
Skip prompts:
```bash
npx hyperframes init my-video --non-interactive --example blank
```
With source video for auto-transcription:
```bash
npx hyperframes init my-video --example warm-grain --video ./intro.mp4
```

## Minimal composition (`index.html`)

```html
<div id="root" data-composition-id="my-video"
     data-start="0" data-width="1920" data-height="1080">

  <h1 id="title" class="clip"
      data-start="0" data-duration="5" data-track-index="0"
      style="font-size: 72px; color: white; text-align: center;
             position: absolute; top: 50%; left: 50%;
             transform: translate(-50%, -50%);">
    Hello, Hyperframes!
  </h1>

  <script src="https://cdn.jsdelivr.net/npm/gsap@3/dist/gsap.min.js"></script>

  <script>
    const tl = gsap.timeline({ paused: true });
    tl.from("#title", { opacity: 0, y: -50, duration: 1 }, 0);
    window.__timelines = window.__timelines || {};
    window.__timelines["my-video"] = tl;
  </script>
</div>
```

### The three rules this exercises

1. Root has `data-composition-id`, `data-width`, `data-height`
2. Timed elements have `class="clip"` + `data-start` + `data-duration` + `data-track-index`
3. GSAP timeline is `{ paused: true }` and registered on `window.__timelines`

## Preview + Render

```bash
npx hyperframes preview                    # browser, hot reload
npx hyperframes render --output output.mp4 # capture + encode
```

Output shows frame progress + final resolution / duration / fps.

## Full docs
https://hyperframes.mintlify.app/llms.txt
