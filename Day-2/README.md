# Day 02 / 30 — Liquid Blob Background

> **30 Days of Insane Web Development** — Building one wild UI experiment every day.

![Day 02](https://img.shields.io/badge/Day-02%20%2F%2030-a21caf?style=for-the-badge)
![SVG](https://img.shields.io/badge/SVG-Filters-ec4899?style=for-the-badge&logo=svg)
![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-f7df1e?style=for-the-badge&logo=javascript)
![CSS](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)

---

## What I Built?

A **liquid blob background** that morphs, floats, and reacts to your cursor in real time — built entirely with SVG filters and Vanilla JS. Zero canvas libraries. Zero WebGL. Just pure SVG filter magic.

- 5 blobs floating organically using sine wave math
- Nearest blob gets attracted toward your cursor/touch
- Continuous color cycling — hues shift slowly across all blobs
- Turbulence frequency spikes when you move fast — blobs go wild
- Styled as a phone frame (390×844px) — built for Instagram Reels

---

## Live Demo

> Open `index.html` in any modern browser and move your cursor across the blobs!

---

## How It Works

### The Gooey SVG Filter — The Whole Secret

Everything you see is powered by 3 chained SVG filters inside a single `<filter>` tag:

```html
<filter id="goo">

  <!-- Step 1: Generate organic noise -->
  <feTurbulence type="fractalNoise"
    baseFrequency="0.012 0.01"
    numOctaves="3" />

  <!-- Step 2: Displace the blobs using that noise -->
  <feDisplacementMap scale="55" />

  <!-- Step 3: Threshold the alpha to create crisp gooey edges -->
  <feColorMatrix type="matrix" values="1 0 0 0 0
                                        0 1 0 0 0
                                        0 0 1 0 0
                                        0 0 0 22 -10" />
</filter>
```

The `feColorMatrix` last row `0 0 0 22 -10` is the real magic — it cranks up the alpha channel contrast so blobs have sharp, gooey edges instead of a soft blur.

---

### Blob Floating — Sine Wave Math

Each blob drifts organically using independent sine/cosine waves on X and Y axes:

```js
let cx = b.ox + Math.sin(t * b.fx + b.phase) * b.ax;
let cy = b.oy + Math.cos(t * b.fy + b.phase) * b.ay;
```

| Parameter | What it Controls |
|-----------|-----------------|
| `ox / oy` | Origin (rest position) of the blob |
| `ax / ay` | Amplitude — how far it drifts from origin |
| `fx / fy` | Frequency — how fast it oscillates |
| `phase`   | Phase offset — so blobs don't all move in sync |

Each blob has different values so they all feel unique and never repeat together.

---

### Cursor Reaction

When you move your cursor, two things happen simultaneously:

```js
function reactToPointer(px, py) {
  // 1. Warp the turbulence based on distance from center
  const dist = Math.hypot(dx, dy);
  targetBfx = 0.012 + dist * 0.022;
  targetBfy = 0.010 + dist * 0.018;

  // 2. Attract the nearest blob toward the pointer
  const force = 1 - minD / 200;
  closest._px = px;
  closest._pf = force;
}
```

The further you are from center, the more the turbulence distorts. The closest blob within `200px` gets pulled toward your cursor with a force that decays over time.

---

### Turbulence Evolution

The noise seed slowly shifts over time so the blob shapes never repeat:

```js
const seedShift = Math.sin(t * 0.00018) * 0.003;
turb.setAttribute('baseFrequency',
  `${(bfx + seedShift).toFixed(5)} ${(bfy + seedShift * 0.8).toFixed(5)}`
);
```

The displacement scale also pulses gently:

```js
const scaleBase = 55;
const scalePulse = Math.sin(t * 0.00035) * 18;
disp.setAttribute('scale', (scaleBase + scalePulse).toFixed(1));
```

---

### Color Cycling

All 5 blob gradients slowly rotate through hues using `hsl()`:

```js
const shift = Math.sin(hueT * 0.012 + g.h1 * 0.01) * 25;
stops[0].setAttribute('stop-color', `hsl(${g.h1 + shift}, 90%, 58%)`);
stops[1].setAttribute('stop-color', `hsl(${g.h2 + shift}, 80%, 30%)`);
```

Each gradient shifts at a slightly different rate so the color relationships stay interesting.

---

## Project Structure

```
day-02-liquid-blob/
│
└── index.html    ← Everything in one file (HTML + CSS + JS)
```

---

## Tech Stack

| Tech        | Usage                                                    |
|-------------|----------------------------------------------------------|
| SVG Filters | `feTurbulence`, `feDisplacementMap`, `feColorMatrix`     |
| Vanilla JS  | Animation loop, pointer tracking, color cycling          |
| CSS3        | Layout, badge, typography, overlay gradients             |

---

## Key Concepts

- **`feTurbulence`** — generates Perlin/fractal noise used to warp shapes
- **`feDisplacementMap`** — displaces pixel positions using the noise map
- **`feColorMatrix`** — manipulates RGBA channels; here used to threshold alpha for gooey edges
- **`Math.sin / Math.cos`** — independent oscillation on X and Y for organic float
- **`requestAnimationFrame`** — smooth 60fps animation loop
- **Lerp** — `bfx += (targetBfx - bfx) * 0.025` smoothly transitions turbulence on mouse move

---

## Design Decisions

- Dark `#08060f` background makes the glowing blobs look like light sources
- 5 blobs in purple, pink, blue, magenta, and cyan — complementary but not clashing
- Overlay gradient adds subtle top-lighting so it doesn't look flat
- The bottom code line `baseFrequency="0.012" · scale="55"` surfaces the key parameters visually
- Color swatches strip shows live blob colors — educational and aesthetic at the same time

---

## Try It Yourself

```bash
git clone https://github.com/yourusername/30-days-insane-webdev.git
cd 30-days-insane-webdev/day-02-liquid-blob
open index.html
```

---

## Want to Experiment?

Try tweaking these values and refresh:

```js
// More dramatic cursor reaction
targetBfx = 0.012 + dist * 0.08;   // was 0.022

// Bigger displacement = more warp
const scaleBase = 100;   // was 55

// Slower, dreamier blob movement
{ ax: 30, ay: 30, fx: 0.0003, fy: 0.0003 }

// Faster color cycling
hueT += 0.5;   // was 0.12
```

---

## The Series

| Day | Project | Status |
|-----|---------|--------|
| 01 | 🧲 Magnetic Cursor | ✅ Done |
| 02 | 🫧 Liquid Blob Background | ✅ Done |
| 03 | 🃏 3D Card Flip Gallery | 🔜 Tomorrow |
| ... | ... | ... |
| 30 | 🌗 Dark/Light Mode Flip Toggle | 🔜 Coming |

---

## 📲 Follow Along

> Built this in a day as part of my **#30DaysOfInsaneWebDev** challenge.  
> Follow on Instagram for the daily Reels 👉 **@vladlen.codes**

---

## 📄 License

MIT — use it, remix it, build on it. Star ⭐ the repo if this helped you!
