# Introduction (HyperFrames)

> Source: https://hyperframes.heygen.com/introduction
> Write HTML. Render video. Built for agents.

**Read this when** onboarding a new agent to the HyperFrames paradigm, or when someone asks "why HTML for video?"

## Core philosophy

HyperFrames turns HTML into deterministic, frame-by-frame video.

> **Same input, identical output, every time.**

The pipeline is **seek-driven**:
- Frame calculation: `frame = floor(time * fps)`
- Each frame captured independently via Chrome's `beginFrame` API
- Encoded with FFmpeg

No clocks. No wall-time. No drift.

## HTML as the source of truth

Videos are HTML documents with data attributes:
- `data-start`, `data-duration` — timing
- `data-track-index` — temporal layering (not visual — use `z-index`)
- `data-composition-id`, `data-width`, `data-height` — composition frame

No timeline editor. No proprietary component system. No mandatory React.

Animation runtime is pluggable via **Frame Adapters** — GSAP ships; Lottie / CSS / Three.js / SVG are all possible.

## Why built for agents

Most video tools rely on complex APIs or GUIs that don't suit AI automation. HyperFrames addresses this directly:

- **HTML generation** — LLMs already excel at producing HTML
- **CLI design** — non-interactive by default; all inputs via flags; plain text output; fail-fast on errors
- **Deterministic output** — reliable batch processing + CI integration
- Optional `--human-friendly` flag for interactive terminal workflows

## Who this is for

- **Developers** already fluent in web tech — no proprietary learning curve
- **AI systems** — operates natively in HTML/CLI, no specialized training needed
- **Automated pipelines** — headless Chrome + Docker = reproducible batch renders

## Workflow (mirrors web dev)

1. Write HTML composition with timing attributes
2. Preview in the browser
3. Render to MP4 via CLI

## Ecosystem (5 packages)

- `@hyperframes/core` — types, HTML generation, runtime, linter foundation
- `@hyperframes/engine` — seekable page-to-video capture (Chrome BeginFrame)
- `@hyperframes/producer` — full HTML→video pipeline (encoding, audio mixing, Docker)
- `@hyperframes/studio` — visual editor + live preview + timeline + hot reload
- `@hyperframes/cli` — command-line entry point
