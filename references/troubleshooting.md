# Troubleshooting (HyperFrames)

> Source: https://hyperframes.heygen.com/guides/troubleshooting
> Solutions for common HyperFrames issues — symptom, cause, fix.

**Read this when** something fails: CLI errors, missing composition, preview not updating, render ≠ preview, Docker fails.

## Fast diagnostics

```bash
npx hyperframes doctor    # validate Node / FFmpeg / Docker / Chrome
npx hyperframes info      # system + project details (paste when opening issues)
npx hyperframes lint      # structural problems
npx hyperframes validate  # runtime problems (JS, missing assets, contrast)
```

---

## Issues & Fixes

### "No composition found"
The directory has no valid composition. Either:
- `npx hyperframes init` to scaffold from an example, or
- Add the root element to `index.html`:
```html
<div id="root" data-composition-id="my-video"
     data-start="0" data-width="1920" data-height="1080">
  <!-- elements here -->
</div>
```
Root MUST have `data-composition-id`.

### "FFmpeg not found"
Local rendering needs FFmpeg:
- macOS: `brew install ffmpeg`
- Ubuntu/Debian: `sudo apt install ffmpeg`
- Windows: ffmpeg.org, add `bin/` to PATH
- Verify: `ffmpeg -version`, then re-run `npx hyperframes doctor`
- Or skip install entirely with Docker: `npx hyperframes render --docker --output out.mp4`

### Lint errors

| Error                        | Fix                                                              |
| ---------------------------- | ---------------------------------------------------------------- |
| Missing `data-composition-id`| Add to root element                                              |
| Missing `class="clip"`       | Add to timed visible elements                                    |
| Overlapping timelines        | Same-track clips can't overlap temporally                        |
| Unmuted video                | Add `muted`, or set `data-has-audio="true"`                      |
| Deprecated `data-layer`/`data-end` | Replace per HTML Schema Reference                          |

### Preview not updating
1. Editing the correct `index.html`?
2. Check terminal for preview server errors
3. Stop/restart `npx hyperframes preview`
4. Hard refresh: Cmd+Shift+R (mac) / Ctrl+Shift+R (Win/Linux)
5. Clear browser cache for CSS changes

### Preview stutters / low FPS (but render is smooth)
**Cause:** frames exceed 16–33ms paint budget. See `performance.md`.
Usual culprits:
- Stacked `backdrop-filter: blur()` (especially > 32px radius)
- High-res source images (>4K) in small regions
- `filter: blur()` or `filter: drop-shadow()` on large elements
- Animated `box-shadow` / `text-shadow`

Diagnose with Chrome DevTools → Performance → Record.

Quick workaround: `npx hyperframes render --quality draft --output preview.mp4` and scrub that.

### Render differs from preview
Font availability, Chrome version, or GPU compositing drift between your machine and the preview.
**Fix:** deterministic Docker render:
```bash
npx hyperframes render --docker --output out.mp4
```

### Docker mode fails
```bash
docker info
```
- Docker not running → start Docker Desktop / daemon
- Permission denied → `sudo usermod -aG docker $USER`, restart shell
- Image pull fails → check internet (first render downloads the image)

### Slow rendering
1. `--quality draft` during dev
2. `npx hyperframes benchmark` to find optimal worker count
3. `--gpu` for hardware acceleration (local mode)
4. Drop `--fps 24` if 30 isn't needed
5. Remove unnecessary elements; simplify animation

---

## Still stuck

1. `npx hyperframes info` — capture environment
2. Search https://github.com/heygen-com/hyperframes/issues
3. Open an issue with the `info` output + repro steps
