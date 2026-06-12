# Reflex — Reaction Time Lab

A precision reaction-time game built as a single static HTML file, with a live **Three.js** 3D scene as the visual stimulus. Click *start*, wait for the chamber to flash green, and react as fast as you can — your time is measured to the millisecond, rated against human benchmarks, and tracked across the session.

Built as a portfolio piece to demonstrate vanilla JavaScript, careful timing, 3D graphics programming, and UI/UX polish — with no framework and no build step.

> **Live demo:** _add your deployed URL here_
> **Screenshot:** _add a screenshot or GIF here (`docs/preview.gif`)_

---

## Highlights

- **Honest millisecond timing.** The measurement clock is fully decoupled from the render loop, so frame rate and GPU load can't skew the result (see [How timing works](#how-timing-works)).
- **Three.js stimulus chamber.** A faceted icosahedron floats in a lit chamber with an ambient particle field, snapping color and punching scale the instant the stimulus fires.
- **Performance ratings.** Every attempt is scored against reaction-time bands, from *Amazing* to *Below average*.
- **Session statistics.** Best, average, attempt count, and a recent-attempts strip (taller = faster).
- **False-start detection.** Reacting before the green flash is caught and flagged.
- **Graceful degradation.** If WebGL is unavailable, the game falls back to a color-driven panel and still plays.
- **Accessible & responsive.** Keyboard support, ARIA labeling, reduced-motion support, and a mobile-first layout.
- **Zero dependencies to install.** One HTML file; Three.js loads from a CDN.

---

## Getting started

This is a static site — no build, no package manager.

**Quickest:** open `reflex-reaction-lab.html` directly in a modern browser.

**Recommended (so relative paths like the favicon resolve):** serve the folder over a local HTTP server.

```bash
# Python 3
python -m http.server 8000

# or Node
npx serve .
```

Then visit `http://localhost:8000/reflex-reaction-lab.html`.

> Three.js is loaded from a CDN, so an internet connection is required on first load.

---

## How to play

1. Press **Start** (or the **Spacebar**, or tap the chamber) to arm it.
2. **Hold steady** — after a random 1–5 second delay, the chamber turns green.
3. **React** the instant it does, by tapping the chamber or pressing Space.

Reacting before the green flash is a **false start** and the round resets. Your reaction time, rating, and updated session stats appear after each successful attempt.

### Scoring

| Rating         | Reaction time     |
| -------------- | ----------------- |
| Amazing        | under 180 ms      |
| Very good      | 180 – 219 ms      |
| Good           | 220 – 259 ms      |
| Average        | 260 – 329 ms      |
| Below average  | 330 ms and above  |

Bands are calibrated around the typical human visual reaction time of roughly 250 ms. They're defined in a single `rate()` function and are easy to retune.

---

## How timing works

The core engineering goal was an accurate measurement that holds up *while* a 3D scene renders continuously. The approach:

- All timing uses `performance.now()`, a high-resolution monotonic clock.
- The stimulus onset time is captured the moment the game transitions to the *go* state.
- The reaction time is simply `performance.now()` at the user's input minus that onset time.

The `requestAnimationFrame` render loop only *reflects* game state — it animates the object and lerps colors — but it never participates in the measurement. As a result, render frame rate has no effect on the recorded number. The one unavoidable constant is the sub-frame latency (≤ ~16 ms) between recording the onset and the pixels actually being painted, which is inherent to any browser-based reaction test.

No browser storage APIs are used; all session state (times, stats, current game state) lives in plain in-memory JavaScript variables and resets on reload.

---

## Tech stack

- **HTML5 / CSS3** — single-file layout, CSS custom properties for theming, `backdrop-filter` panels.
- **Vanilla JavaScript** — a small explicit state machine (`idle → waiting → go → result`, plus a `falsestart` branch), DOM and event handling, no framework.
- **Three.js (r128)** — `WebGLRenderer`, perspective camera, ambient + point + directional lights, `MeshStandardMaterial` with flat shading, wireframe edges, and a buffer-geometry particle field.
- **Google Fonts** — Poppins for headings, Inter (with tabular figures) for data and UI.

---

## Project structure

```
.
├── reflex-reaction-lab.html   # the entire app: markup, styles, and logic
├── favicon.png                # site icon (referenced from the project root)
└── README.md
```

Everything — structure, styling, 3D scene, and game logic — lives in the single HTML file for easy hosting and review.

---

## Design system

| Token            | Hex       | Use                          |
| ---------------- | --------- | ---------------------------- |
| Background       | `#0F172A` | Page background              |
| Surface          | `#1E293B` | Cards and chamber            |
| Primary text     | `#F1F5F9` | Headings and values          |
| Secondary text   | `#94A3B8` | Labels and hints             |
| Ready / go       | `#10B981` | Start affordance, GO state   |
| Waiting          | `#F59E0B` | Armed / holding state        |
| Result           | `#3B82F6` | Measurement display          |
| Error            | `#EF4444` | False start                  |

The active state color drives the 3D object, the chamber glow, and the status indicator together, so the instrument visibly "arms," "holds," and "fires."

---

## Accessibility

- The chamber is a focusable control (`role="button"`, keyboard-operable via Enter/Space).
- ARIA labels announce the current state and the result, including the rating.
- Visible `:focus-visible` outlines and high-contrast text on backdrop panels.
- `prefers-reduced-motion` is respected: decorative motion (bobbing, particle drift, spin) is toned down while the essential state cues remain.

---

## Browser support

Works in current versions of Chrome, Firefox, Safari, and Edge. A WebGL-capable browser gets the full 3D experience; if WebGL initialization fails, the app detects it and switches to a color-driven fallback panel so the game remains fully playable.

---

## Performance & memory

- A single 3D mesh is reused across all states — no per-round allocation.
- Pixel ratio is capped to keep rendering light on high-DPI displays.
- Resize handling is throttled with `requestAnimationFrame`.
- GPU resources (geometries, materials, renderer) are disposed on `pagehide`, and the animation loop is canceled on teardown.

---

## Customization

- **Rating bands** — edit the thresholds in the `rate()` function.
- **Stimulus delay** — change the `1000 + Math.random() * 4000` range in `arm()`.
- **Colors** — update the CSS custom properties in `:root` and the matching `HEX` map in the script.
- **3D object** — swap the `IcosahedronGeometry` for any other Three.js geometry.

---

## Possible enhancements

- Median and a small distribution view (reaction times are right-skewed, so median is often more representative than the mean).
- Persisting best scores (would require adding a storage layer).
- Audio cue option and a configurable number of rounds per session.

---

## License

Released under the MIT License — feel free to adapt and reuse. _Add a `LICENSE` file if you publish this._

---

## Credits

3D rendering powered by [Three.js](https://threejs.org/). Typography by [Inter](https://rsms.me/inter/) and [Poppins](https://fonts.google.com/specimen/Poppins).
