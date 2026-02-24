# Day 05 / 30 — Infinite Warping Marquee

> **30 Days of Insane Web Development** — Building one wild UI experiment every day.

![Day 05](https://img.shields.io/badge/Day-05%20%2F%2030-7c3aed?style=for-the-badge)
![CSS Only](https://img.shields.io/badge/CSS-3D%20Perspective-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-f7df1e?style=for-the-badge&logo=javascript)
![No Libraries](https://img.shields.io/badge/Libraries-Zero-success?style=for-the-badge)

---

## What I Built?

**8 rows of text scrolling infinitely** — each at a different speed, different size, and different direction — all wrapped in a CSS 3D perspective warp that makes it look like a tunnel. Zero WebGL. Zero libraries. Just `perspective` + `rotateX` + a sine wave.

- 8 rows scrolling at different speeds — creates a parallax depth illusion
- Alternating scroll directions — odd rows go left, even rows go right
- CSS 3D perspective warp — `rotateX(22deg)` tilts the whole stage
- Size + opacity hierarchy — center rows are bigger, brighter, faster
- Accent colored words in center rows for visual punch
- Sine wave speed pulse — animation breathes, never feels robotic
- Vertical vignette — rows fade in/out from darkness at top and bottom
- Phone frame (390×844px) — built for Instagram Reels

---

## Live Demo

> Open `index.html` in any modern browser — no interaction needed, just watch it run!

---

## How It Works

### The 3D Warp — Just 2 CSS Properties

No WebGL. No Canvas. The entire 3D tunnel effect is:

```css
/* Parent — defines the depth of the 3D space */
.stage {
  perspective: 600px;
  perspective-origin: 50% 50%;
}

/* Child — tilts the whole row stack in 3D space */
.warp-container {
  transform: rotateX(22deg);
  transform-style: preserve-3d;
}
```

`perspective: 600px` sets how far the "camera" is. Lower = more dramatic warp. Higher = flatter. `rotateX(22deg)` tilts the container like a table top viewed from above.

---

### Infinite Seamless Scroll — Pure CSS

The scroll animation is a CSS keyframe that translates `-50%`:

```css
.track {
  animation: scroll-left var(--dur) linear infinite;
}

@keyframes scroll-left {
  from { transform: translateX(0);    }
  to   { transform: translateX(-50%); }
}

@keyframes scroll-right {
  from { transform: translateX(-50%); }
  to   { transform: translateX(0);    }
}
```

The seamless trick — content is duplicated so `-50%` lands exactly back at the start:

```js
// Duplicate words so the loop is invisible
const allWords = [...data.words, ...data.words];

// Then double again for extra safety at high speeds
track.innerHTML = html + html;
```

When `translateX` reaches `-50%`, it snaps back to `0` — and because the second half is identical to the first, the jump is invisible.

---

### Speed Hierarchy — Faking Depth with Duration

Each row has a different `--dur` CSS custom property. Slower = feels further away:

```css
.row-0 { --dur: 32s; }   /* top    — slowest  = far    */
.row-1 { --dur: 26s; }
.row-2 { --dur: 20s; }
.row-3 { --dur: 16s; }   /* center — fastest  = close  */
.row-4 { --dur: 16s; }   /* center — fastest  = close  */
.row-5 { --dur: 20s; }
.row-6 { --dur: 26s; }
.row-7 { --dur: 32s; }   /* bottom — slowest  = far    */
```

Combined with size and opacity scaling, this creates a convincing parallax tunnel without any 3D math.

---

### Size + Opacity Hierarchy

Text scales up and brightens toward the center rows:

```css
/* Far rows — small, dim */
.row-0 .item, .row-7 .item {
  font-size: 0.72rem;
  color: rgba(167, 139, 250, 0.28);
}

/* Center rows — large, bright */
.row-3 .item, .row-4 .item {
  font-size: 1.35rem;
  color: rgba(240, 220, 255, 0.92);
}
```

The eye reads smaller + dimmer as further away — reinforcing the depth illusion from the speed hierarchy.

---

### Speed Pulse — The Sine Wave Trick

A `requestAnimationFrame` loop gently oscillates all animation speeds by ±18%:

```js
let speedT = 0;

function pulseSpeed() {
  speedT += 0.008;
  const factor = 1 + 0.18 * Math.sin(speedT);  // ±18% wave

  tracks.forEach(t => {
    const base = parseFloat(
      getComputedStyle(t.closest('.row')).getPropertyValue('--dur')
    ) || 20;
    t.style.animationDuration = (base / factor) + 's';
  });

  requestAnimationFrame(pulseSpeed);
}
```

Without this, the marquee feels mechanical. With it, it breathes. One sine wave transforms "CSS animation" into "living thing".

---

### Edge Fade — Mask Image

Each row fades out at left and right edges using a CSS mask:

```css
.row {
  -webkit-mask-image: linear-gradient(
    90deg,
    transparent 0%,
    black 12%,
    black 88%,
    transparent 100%
  );
  mask-image: linear-gradient(
    90deg,
    transparent 0%,
    black 12%,
    black 88%,
    transparent 100%
  );
}
```

The vertical fade (top + bottom of phone) uses the same technique but on the `.vignette` overlay layer.

---

## Project Structure

```
day-05-warping-marquee/
│
└── index.html    ← Everything in one file (HTML + CSS + JS)
```

---

## Tech Stack

| Tech       | Usage                                              |
|------------|----------------------------------------------------|
| CSS3       | 3D perspective, keyframe animation, mask-image     |
| Vanilla JS | Row building, speed pulse, direction alternation   |
| CSS Custom Properties | Per-row duration via `--dur` variable   |

---

## Key Concepts

- **`perspective`** — defines the 3D depth of the viewing space (like camera distance)
- **`rotateX(22deg)`** — tilts the entire container in 3D space
- **`translateX(-50%)`** — the seamless loop trick; works because content is doubled
- **`linear infinite`** — constant speed, never eases in or out
- **`--dur` CSS custom property** — per-row speed control without JS
- **`mask-image`** — fades row edges without extra DOM elements
- **`Math.sin(speedT)`** — sine wave oscillation on animation duration for organic breathing

---

## Design Decisions

- `rotateX(22deg)` was tuned by feel — 15° is too flat, 30° is too dramatic
- `perspective: 600px` for the phone width (`390px`) gives a strong but not distorted warp
- Alternating directions (odd = left, even = right) doubles visual complexity with zero extra code
- Accent words (`WARP`, `EXPLODE`) are only in center rows — keeps the highlight meaningful
- The frosted glass pill behind the headline prevents text collision with scrolling rows
- Speed pulse `± 18%` is subtle enough to feel natural but noticeable enough to matter

---

## Try It Yourself

```bash
git clone https://github.com/yourusername/30-days-insane-webdev.git
cd 30-days-insane-webdev/day-05-warping-marquee
open index.html
```

---

## Want to Experiment?

```css
/* More dramatic warp */
.stage { perspective: 300px; }
.warp-container { transform: rotateX(35deg); }

/* Flat (no warp) — compare the difference */
.warp-container { transform: rotateX(0deg); }

/* All rows same speed — kills the depth illusion */
.row-0, .row-1, .row-2, .row-3,
.row-4, .row-5, .row-6, .row-7 { --dur: 20s; }
```

```js
// Stronger speed pulse — ±40%
const factor = 1 + 0.40 * Math.sin(speedT);

// Faster pulse rhythm
speedT += 0.025;   // was 0.008
```

---

## 📅 The Series

| Day | Project | Status |
|-----|---------|--------|
| 01 | Magnetic Cursor | ✅ Done |
| 02 | Liquid Blob Background | ✅ Done |
| 03 | 3D Card Flip Gallery | ✅ Done |
| 04 | Particle Text Explosion | ✅ Done |
| 05 | Infinite Warping Marquee | ✅ Done |
| 06 | Animated Gradient Mesh | 🔜 Tomorrow |
| ... | ... | ... |
| 30 | Dark/Light Mode Flip Toggle | 🔜 Coming |

---

## 📲 Follow Along

> Built this in a day as part of my **#30DaysOfInsaneWebDev** challenge.
> Follow on Instagram for the daily Reels 👉 **@vladlen-codes**

---

## 📄 License

MIT — use it, remix it, build on it. Star ⭐ the repo if this helped you!
