# Day 04 / 30 — Particle Text Explosion

> **30 Days of Insane Web Development** — Building one wild UI experiment every day.

![Day 04](https://img.shields.io/badge/Day-04%20%2F%2030-f472b6?style=for-the-badge)
![Canvas API](https://img.shields.io/badge/Canvas-API-818cf8?style=for-the-badge)
![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-f7df1e?style=for-the-badge&logo=javascript)
![No Libraries](https://img.shields.io/badge/Libraries-Zero-success?style=for-the-badge)

---

## What I Built?

Particles **assemble from random scatter into text**, drift gently, then **explode with real gravity** — all on HTML Canvas with zero libraries. Click to explode early or watch the auto-cycle run through 5 words with unique color palettes.

- Particles sample actual text pixels to know where to form
- Lerp-based formation — each particle smoothly flies to its target
- Real gravity + air friction on explosion
- 5 words × 5 unique color palettes — cycles automatically
- Click/tap anywhere to trigger explosion early
- Motion trail effect — semi-transparent fill instead of clear
- Phone frame (390×844px) — built for Instagram Reels

---

## Live Demo

> Open `index.html` in any modern browser. Watch the particles form, drift, then explode — or tap to trigger it early!

---

## How It Works

### Step 1 — Sample Text Pixels (The Core Trick)

Text is rendered to a hidden off-screen canvas, then every pixel is read. If a pixel's alpha channel is `> 128`, it's part of a letter — and becomes a particle target:

```js
function sampleText(word, palette) {
  // Render text to a hidden canvas
  const tmp = document.createElement('canvas');
  const tctx = tmp.getContext('2d');
  tctx.font = `900 130px Inter`;
  tctx.fillText(word, W / 2, H / 2);

  // Read every pixel
  const imageData = tctx.getImageData(0, 0, W, H).data;

  for (let y = 0; y < H; y += STEP) {
    for (let x = 0; x < W; x += STEP) {
      const idx = (y * W + x) * 4;

      if (imageData[idx + 3] > 128) {   // alpha > 128 = part of letter
        pts.push({ x, y, color });
      }
    }
  }
}
```

`STEP = 5` means we sample every 5th pixel — controls how dense the particle text looks. Lower = more particles, higher = sparse.

---

### Step 2 — Particles Lerp to Their Targets

Each particle starts at a random position and lerps toward its sampled text coordinate:

```js
formStep() {
  // Accelerate toward target
  this.vx += (this.tx - this.x) * this.ease;
  this.vy += (this.ty - this.y) * this.ease;

  // Damping — prevents overshooting
  this.vx *= 0.8;
  this.vy *= 0.8;

  this.x += this.vx;
  this.y += this.vy;
}
```

`ease` is randomized per particle (`0.06 – 0.12`) so they arrive at slightly different times — giving the formation an organic, breathing feel rather than snapping all at once.

---

### Step 3 — Explosion with Real Physics

On explode, each particle gets a random burst velocity. Then gravity and friction take over every frame:

```js
explodeStep() {
  if (!this.exploded) {
    // Random burst in any direction
    const angle = Math.random() * Math.PI * 2;
    const speed = Math.random() * 14 + 5;
    this.vx = Math.cos(angle) * speed;
    this.vy = Math.sin(angle) * speed - 4;  // slight upward bias
    this.exploded = true;
  }

  this.vy += 0.35;    // gravity pulls down every frame
  this.vx *= 0.96;    // air friction slows horizontal movement
  this.vy *= 0.96;    // air friction slows vertical movement
  this.alpha -= 0.016; // fade out
  this.size  *= 0.992; // shrink slightly
}
```

`vy += 0.35` is the entire gravity system. One line.

---

### The Motion Trail Effect

Instead of `ctx.clearRect()` each frame, a semi-transparent fill creates a fading trail:

```js
// Each frame — don't clear, paint a dark semi-transparent rect
ctx.fillStyle = 'rgba(6, 4, 15, 0.22)';
ctx.fillRect(0, 0, W, H);
```

`0.22` opacity means old frames fade slowly — particles leave glowing streaks during explosion. Increase it for sharper/cleaner, decrease for longer trails.

---

### Color Gradient Across Letters

Colors aren't random — they're interpolated across the word from left to right using lerp:

```js
const t = x / W;   // 0 at left edge, 1 at right edge
const ci = Math.floor(t * (palette.length - 1));
const color = lerpColor(palette[ci], palette[ci2], cf);
```

Each word has its own palette:

| Word | Colors |
|------|--------|
| BOOM | Pink → Purple → Indigo |
| CODE | Sky → Indigo → Purple |
| WEB  | Teal → Cyan → Indigo |
| FIRE | Orange → Pink → Purple |
| WILD | Yellow → Orange → Pink |

---

## Project Structure

```
day-04-particle-explosion/
│
└── index.html    ← Everything in one file (HTML + CSS + JS)
```

---

## Tech Stack

| Tech        | Usage                                           |
|-------------|-------------------------------------------------|
| Canvas API  | Particle rendering, off-screen text sampling    |
| Vanilla JS  | Particle physics, animation loop, state machine |
| CSS3        | Phone frame, badge, UI overlay                  |

---

## Key Concepts

- **`ctx.getImageData()`** — reads raw pixel data from a canvas as a flat array
- **`imageData[idx + 3]`** — the alpha channel of each pixel (0 = transparent, 255 = solid)
- **Lerp physics** — `vx += (target - x) * ease` pulls particles toward a point
- **Damping** — `vx *= 0.8` prevents oscillation and overshooting
- **Gravity** — `vy += 0.35` added every frame simulates downward acceleration
- **Motion trail** — semi-transparent fill instead of clear creates cinematic streaks
- **State machine** — `forming → formed → exploding` controls which logic runs each frame

---

## Design Decisions

- `STEP = 5` balances particle density vs performance — goes heavy without dropping frames
- Random `ease` per particle (`0.06–0.12`) makes formation feel organic, not mechanical
- Upward bias on explosion `vy - 4` makes the burst feel energetic, not just outward
- 60 ambient background dots pulse at different speeds — adds depth to the dark background
- Auto-explode after ~2.4s (`formTimer > 145`) keeps the demo alive without user input
- `document.fonts.ready.then()` ensures Inter is loaded before pixel sampling — prevents fallback font shapes

---

## Try It Yourself

```bash
git clone https://github.com/yourusername/30-days-insane-webdev.git
cd 30-days-insane-webdev/day-04-particle-explosion
open index.html
```

---

## Want to Experiment?

```js
// Denser particles (more detail, heavier)
const STEP = 3;   // was 5

// Longer explosion trails
ctx.fillStyle = 'rgba(6, 4, 15, 0.10)';  // was 0.22

// Stronger gravity — particles fall faster
this.vy += 0.7;   // was 0.35

// Slower, floatier formation
this.ease = 0.02 + Math.random() * 0.02;  // was 0.06–0.12

// Add your own words and palettes
const WORDS    = ['YOUR', 'NAME', 'HERE'];
const PALETTES = [
  ['#ff0080', '#7928ca'],
  ['#0070f3', '#00dfd8'],
  ['#ff4d4d', '#f9cb28'],
];
```

---

## 📅 The Series

| Day | Project | Status |
|-----|---------|--------|
| 01 | Magnetic Cursor | ✅ Done |
| 02 | Liquid Blob Background | ✅ Done |
| 03 | 3D Card Flip Gallery | ✅ Done |
| 04 | Particle Text Explosion | ✅ Done |
| 05 | Infinite Warping Marquee | 🔜 Tomorrow |
| ... | ... | ... |
| 30 | Dark/Light Mode Flip Toggle | 🔜 Coming |

---

## Follow Along

> Built this in a day as part of my **#30DaysOfInsaneWebDev** challenge.
> Follow on Instagram for the daily Reels 👉 **@vladlen.codes**

---

## 📄 License

MIT — use it, remix it, build on it. Star ⭐ the repo if this helped you!
