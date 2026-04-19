# Rendering Guide (HyperFrames)

> Source: https://hyperframes.heygen.com/guides/rendering
> Render compositions to MP4, MOV, or WebM locally or in Docker.

**Read this when** you're ready to export a finished composition, need transparent video, or want deterministic CI-friendly output.

## Quick Path

```bash
npx hyperframes doctor                     # verify Node 22, FFmpeg 7, Chrome, Docker
npx hyperframes preview                    # watch in browser
npx hyperframes render --output out.mp4    # render
```

## Rendering Modes

### Local (default)
- Puppeteer + bundled Chromium + system FFmpeg
- Fast startup, GPU acceleration available
- **Use for**: development, quick previews, benchmarking
- **Avoid for**: CI/CD (platform-dependent output)

### Docker (deterministic)
```bash
npx hyperframes render --docker --output out.mp4
```
- Pinned Chrome + fonts + FFmpeg — identical output across platforms
- **Use for**: CI/CD, team sharing, AI-agent rendering, production
- **Tradeoff**: slower startup, no GPU

---

## CLI Flags

| Flag                        | Values                 | Default                | Purpose                                  |
| --------------------------- | ---------------------- | ---------------------- | ---------------------------------------- |
| `--output`                  | path                   | `renders/<name>.mp4`   | Output file location                     |
| `--format`                  | mp4 \| mov \| webm     | mp4                    | Output codec format                      |
| `--fps`                     | 24 \| 30 \| 60         | 30                     | Frames per second                        |
| `--quality`                 | draft \| standard \| high | standard           | Encoding preset                          |
| `--crf`                     | 0–51                   | —                      | Quality override (lower = higher quality)|
| `--video-bitrate`           | e.g. `10M`, `5000k`    | —                      | Target bitrate                           |
| `--workers`                 | 1–8 or `auto`          | auto                   | Parallel capture processes               |
| `--max-concurrent-renders`  | 1–10                   | 2                      | Simultaneous renders allowed             |
| `--gpu`                     | —                      | off                    | Enable hardware acceleration             |
| `--docker`                  | —                      | off                    | Deterministic Docker rendering           |
| `--quiet`                   | —                      | off                    | Suppress verbose output                  |

## Quality Presets

| Preset     | CRF | x264 speed  | Use case                               |
| ---------- | --- | ----------- | -------------------------------------- |
| `draft`    | 28  | ultrafast   | Quick iteration                        |
| `standard` | 18  | medium      | General use, visually lossless @ 1080p |
| `high`     | 15  | slow        | Final delivery, near-lossless          |

Override examples:
```bash
npx hyperframes render --crf 15 --output pristine.mp4
npx hyperframes render --video-bitrate 10M --output controlled.mp4
```

## Workers

Default = half your CPU cores, capped at 4.

| Machine        | Cores | Default workers |
| -------------- | ----- | --------------- |
| M1 MacBook Air | 8     | 4               |
| M3 MacBook Pro | 12    | 4 (capped)      |
| 4-core laptop  | 4     | 2               |
| 2-core VM      | 2     | 1               |

```bash
npx hyperframes render --workers 1 --output out.mp4       # force single
npx hyperframes render --workers auto --output out.mp4    # let it decide
npx hyperframes render --workers 8 --output out.mp4       # max parallel
```

Short compositions (<60 frames): use `--workers 1`. Long compositions (30s+) on 8+ cores, 16GB+ RAM: bump workers.

## Concurrent Renders (Producer server)

```bash
npx hyperframes render --max-concurrent-renders 2 --output out.mp4
```
Or env var: `PRODUCER_MAX_CONCURRENT_RENDERS=2`
Queue status: `GET /render/queue`
Streaming events: `POST /render/stream`

---

## Transparent Video

### MOV (recommended)
```bash
npx hyperframes render --format mov --output overlay.mov
```
ProRes 4444 codec. Industry standard — import into CapCut / Final Cut / Premiere / DaVinci / After Effects.

### Format Comparison

| Format | Codec       | Transparency | Editors                                      | Browsers         | Size  |
| ------ | ----------- | ------------ | -------------------------------------------- | ---------------- | ----- |
| MOV    | ProRes 4444 | ✅           | CapCut, Final Cut, Premiere, DaVinci, AE    | ❌               | Large |
| WebM   | VP9         | ✅           | ❌ (renders black)                           | Chrome, Firefox  | Small |
| MP4    | H.264       | ❌           | All                                          | All              | Small |

### Implementation
- Captures frames as PNG with alpha channel instead of JPEG
- Sets Chrome background transparency via `Emulation.setDefaultBackgroundColorOverride`
- **Your HTML must NOT set background on `html` / `body`**

### Verify
- Open WebM on a checkerboard background in a browser
- Import MOV in a video editor, overlay on footage
- Or use rotato.app/tools/transparent-video

---

## Mode Selection Matrix

| Scenario                 | Recommended |
| ------------------------ | ----------- |
| Development              | Local       |
| CI/CD pipeline           | Docker      |
| Team sharing             | Docker      |
| Quick preview            | Local       |
| AI agent rendering       | Docker      |
| Performance benchmarking | Local       |

## Tips

- Use `draft` during dev → `standard` or `high` for final
- `npx hyperframes benchmark` to tune settings for your machine
- `--gpu` speeds up long compositions in local mode
