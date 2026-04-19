# Performance Guide (HyperFrames)

> Source: https://hyperframes.heygen.com/guides/performance
> Keep preview playback smooth and diagnose expensive compositions.

**Read this when** preview stutters, playback drops frames, or you're about to add blur/shadow/filter effects.

## Preview vs Render

- **Preview** plays in real time. Slow frames = visible stutter.
- **Render** captures frames sequentially. Slow frames only extend render time — they don't cause visible pauses.

**Stutter in preview doesn't mean the system is broken.** It signals frames exceed the 33ms (30fps) budget. The final MP4 will still be smooth.

## Expensive CSS Patterns

### `backdrop-filter: blur()`
Each blur over a large area forces the compositor to read pixels from behind the element, run a blur kernel, and composite the result.

**Fix:**
- Limit stacked blur layers to **2–3 max**
- Avoid large radii (64px, 128px) over expansive areas
- Convert **static** blurs to PNG overlays instead

### `filter: blur()` and `filter: drop-shadow()`
Same cost model as `backdrop-filter`, applied to the element itself rather than its background.

### Many shadowed elements
`box-shadow` / `text-shadow` on numerous **animated** elements force re-rasterization each frame.

### Large gradients + `mask-image` + `backdrop-filter`
Triggers additional compositor passes. Reach for a pre-rendered image.

---

## Image Sizing

Decoded bitmap memory: `bitmap_bytes = width × height × 4`.

**Rule of thumb:** resize sources to max **2× canvas dimensions**. For a 1920×1080 canvas, 3840×2160 is already plenty — 8K is pure waste.

```bash
mogrify -path resized -resize 3840x3840\> *.jpg
```

---

## Measuring Performance

1. Start preview: `npx hyperframes preview`
2. Open Chrome DevTools → Performance tab (Cmd+Option+I / Ctrl+Shift+I)
3. Record during 3–5 seconds of playback
4. Analyze main-thread tasks:
   - **Composite Layers / Paint** — compositor cost
   - **Decode Image** — image decode overhead
   - **Layout / Recalculate Style** — layout thrashing
   - **Script** — JS execution

## When preview is legitimately slow

The render will still be fine. For smoother in-browser review:
```bash
npx hyperframes render --quality draft --output preview.mp4
```
Then scrub the MP4.
