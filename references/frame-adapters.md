# Frame Adapters (HyperFrames)

> Source: https://hyperframes.heygen.com/concepts/frame-adapters
> Bring your own animation runtime to HyperFrames.

**Read this when** you want to use a non-GSAP runtime (Lottie, Three.js, CSS/WAAPI, Anime) or you're building a custom adapter.

**API status:** v0 (experimental). The deterministic frame-seek contract is stable; method signatures may change before v1.

## The core question

> "What should the screen look like at frame N?"

If your runtime can answer that deterministically, it integrates into HyperFrames. The adapter **never** manages timing — it only responds to seek commands.

## Host ↔ Adapter flow

1. Host initializes adapter with context
2. Host queries `getDurationFrames()`
3. For each frame (0 → duration):
   - Host normalizes frame (clamp + floor)
   - Host calls `seekFrame(frame)`
   - Adapter updates DOM / canvas state
   - Host captures pixel buffer
4. Host calls `destroy()`

## API Reference (v0)

### Context
```ts
type FrameAdapterContext = {
  compositionId: string;
  fps: number;
  width: number;
  height: number;
  rootElement?: HTMLElement;
};
```

### Adapter interface
```ts
type FrameAdapter = {
  id: string;
  init?: (ctx: FrameAdapterContext) => Promise<void> | void;
  getDurationFrames: () => number;
  seekFrame: (frame: number) => Promise<void> | void;
  destroy?: () => Promise<void> | void;
};
```

- `id` — unique identifier
- `init()` — optional setup, receives context
- `getDurationFrames()` — total frames, finite non-negative integer
- `seekFrame(frame)` — position animation; must be **idempotent**
- `destroy()` — optional cleanup

## Required semantics

1. **Duration** — finite integer ≥ 0
2. **Arbitrary seeking** — forward, backward, random order
3. **Idempotency** — identical frame request → identical output
4. **Clamping** — respect adapter range bounds internally
5. **Pause-driven** — never clock-driven; always seek-responsive

## Host normalization

```ts
normalizedFrame = clamp(Math.floor(frame), 0, durationFrames);
```

Typical render loop:
```ts
await adapter.init?.({ compositionId, fps, width, height, rootElement });
const durationFrames = adapter.getDurationFrames();

for (let frame = 0; frame <= durationFrames; frame += 1) {
  await adapter.seekFrame(frame);
  // capture pixel buffer
}

await adapter.destroy?.();
```

## Determinism contract (non-negotiable)

- **Canonical time:** `t = frame / fps`
- No `Date.now()`, no drift logic
- No unseeded randomness
- No runtime network (fetches) during capture
- `fps`, `width`, `height` immutable across session
- Finite duration only — no infinite animations
- Deterministic quantization

## Supported runtimes

| Runtime         | Status   | Mechanism                                     |
| --------------- | -------- | --------------------------------------------- |
| GSAP            | ✅ ship  | `timeline.seek(frame / fps)`                  |
| CSS / WAAPI     | planned  | `animation.currentTime`                       |
| Lottie          | planned  | `goToAndStop(frame)`                          |
| Three.js / WebGL| planned  | compute deterministic scene state             |
| SVG / Anime     | planned  | implement seek + duration contract            |

Community adapters welcome for any frame-seekable runtime.

## Building a custom adapter

```ts
const MyAdapter: FrameAdapter = {
  id: 'my-runtime',

  async init(ctx) {
    // initialize runtime, mount DOM, setup state
  },

  getDurationFrames() {
    return 300; // deterministic, finite
  },

  async seekFrame(frame) {
    const time = frame / this.fps;
    this.runtime.setTime(time);
  },

  async destroy() {
    // clean up timers, listeners, DOM
  }
};
```

## Conformance tests

Every adapter should pass:

1. **Repeatability** — seek same frame twice → identical output
2. **Random access** — seek `[90, 10, 50, 10]` → deterministic results
3. **Bounds safety** — negative / overflow values don't crash
4. **Duration validity** — returns finite integer
5. **Resource cleanup** — no leaked timers or listeners after `destroy()`
