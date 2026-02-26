# Day 07 / 30 — Glitch Hover Effect

> **30 Days of Insane Web Development** — Building one wild UI experiment every day.

![Day 07](https://img.shields.io/badge/Day-07%20%2F%2030-ff003c?style=for-the-badge)
![Pure CSS](https://img.shields.io/badge/Pure-CSS-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![Zero JS](https://img.shields.io/badge/Core%20Effect-Zero%20JS-success?style=for-the-badge)
![No Libraries](https://img.shields.io/badge/Libraries-Zero-success?style=for-the-badge)

---

## What I Built?

A **CRT-style RGB glitch effect** — red and cyan channels split, scanlines tear horizontally, cards shake and distort — all in pure CSS. Zero JavaScript for the core effect. Click anywhere to trigger an instant glitch burst on demand.

- RGB channel split using `::before` and `::after` pseudo-elements
- `clip-path` horizontal band slicing — mimics real CRT scanline tears
- `steps(1)` timing — makes every frame an instant digital jump, not a smooth transition
- Red stripe sweep across glitch cards on independent timers
- CRT scanline overlay — `repeating-linear-gradient` on the phone frame
- Noise flash layer with `mix-blend-mode: screen` fires during glitch bursts
- Click anywhere — JS cranks animation speed to `0.12s` for an instant burst
- Phone frame (390×844px) — built for Instagram Reels

---

## Live Demo

> Open `index.html` — hover the GLITCH text or click anywhere to trigger a burst!

---

## How It Works

### The RGB Split — 1 Element, 3 Layers, Zero Images

The entire effect lives on a single `<div>`. The text is duplicated using CSS `content: attr(data-text)` on `::before` and `::after`, then each copy gets a different color and `clip-path` slice:

```html
<!-- data-text mirrors the visible text so CSS can copy it -->
<div class="glitch" data-text="GLITCH">GLITCH</div>
```

```css
/* Red channel — left-shifted, clips a horizontal band */
.glitch::before {
  content: attr(data-text);  /* auto-copies text from data attribute */
  position: absolute;
  inset: 0;
  color: #ff003c;
  clip-path: polygon(0 15%, 100% 15%, 100% 38%, 0 38%);
  animation: glitch-r 3s steps(1) infinite;
}

/* Cyan channel — right-shifted, different band */
.glitch::after {
  content: attr(data-text);
  position: absolute;
  inset: 0;
  color: #00f0ff;
  clip-path: polygon(0 55%, 100% 55%, 100% 72%, 0 72%);
  animation: glitch-c 3s steps(1) infinite;
}
```

3 layers. 1 element. No images. No canvas.

---

### `steps(1)` — The Secret That Makes It Feel Digital

This one timing function is everything:

```css
/* smooth — looks organic, wrong vibe */
animation: glitch-r 3s ease infinite;

/* steps(1) — instant jumps only, no in-between frames */
animation: glitch-r 3s steps(1) infinite;
```

`steps(1)` tells the browser to jump directly between keyframe states with zero interpolation — exactly how a broken CRT flickers. Remove it and the effect immediately looks like a design trend instead of a broken screen.

---

### Clip-Path Slicing — How the Tear Works

Each keyframe frame shows a different horizontal band of the text, at a different offset:

```css
@keyframes glitch-r {
  0%, 89%, 100% {
    transform: none;
    opacity: 0;            /* hidden most of the time */
  }

  90% {
    transform: translate(-4px, 0);   /* shift left */
    opacity: 1;
    clip-path: polygon(0 15%, 100% 15%, 100% 38%, 0 38%); /* top band */
  }

  91% {
    transform: translate(4px, 0);    /* shift right */
    clip-path: polygon(0 62%, 100% 62%, 100% 75%, 0 75%); /* lower band */
  }

  92% {
    transform: translate(-5px, 2px);
    clip-path: polygon(0 5%, 100% 5%, 100% 25%, 0 25%);  /* different band */
  }

  93% {
    transform: translate(3px, -1px);
    clip-path: polygon(0 40%, 100% 40%, 100% 55%, 0 55%);
  }

  95% {
    opacity: 0;           /* snap off */
  }
}
```

The red and cyan channels use different `clip-path` bands and opposite translate directions — creating the authentic RGB separation you'd see on a broken monitor.

---

### Body Skew — The Shaking Text

The main text element also animates independently with translate and skewX:

```css
@keyframes glitch-body {
  0%, 90%, 100% {
    transform: none;
    filter: none;
  }

  91% { transform: translate(-3px, 1px) skewX(-2deg); filter: brightness(1.1); }
  92% { transform: translate(3px, -1px); }
  93% { transform: translate(-3px, 2px) skewX(3deg); }
  94% { transform: none; }
  95% { transform: translate(4px, -2px) skewX(-2deg); }
  96% { transform: translate(-3px, 1px); }
  97% { transform: none; filter: none; }
}
```

`skewX` is the detail that sells it — horizontal distortion exactly like a degaussed CRT.

---

### Hover Amplification

Hovering the text cranks the animation speed from `3s` down to `0.3s`:

```css
.glitch:hover::before,
.glitch:hover::after {
  animation-duration: 0.3s;  /* 10× faster than normal */
  opacity: 1;
}

.glitch:hover {
  animation-duration: 0.25s;
}
```

No JS needed. Pure CSS state change on `:hover`.

---

### CRT Scanlines — CSS Pseudo-Element

The scanline grid is a `repeating-linear-gradient` on the phone's `::after`:

```css
#phone::after {
  content: '';
  position: absolute;
  inset: 0;
  pointer-events: none;
  background: repeating-linear-gradient(
    to bottom,
    transparent 0px,
    transparent 3px,
    rgba(0, 0, 0, 0.12) 3px,
    rgba(0, 0, 0, 0.12) 4px   /* 1px dark band every 4px */
  );
}
```

Completely free — no extra DOM elements, no images.

---

### Noise Flash Overlay

A full-screen vertical stripe pattern flickers during glitch moments:

```css
.noise-flash {
  background: repeating-linear-gradient(
    90deg,
    transparent 0px,
    rgba(255, 0, 60, 0.03) 1px,
    transparent 2px,
    rgba(0, 240, 255, 0.03) 3px,
    transparent 4px
  );
  mix-blend-mode: screen;  /* adds to underlying colors, never darkens */
  animation: noise-flicker 3s steps(1) infinite;
}
```

`mix-blend-mode: screen` is critical — it adds light to the image rather than covering it, so the background always shows through.

---

### Click-to-Burst — JS

The only JS in the project temporarily overrides animation speed:

```js
phone.addEventListener('click', () => {
  // Speed up from 3s → 0.12s (25× faster)
  glitch.style.animationDuration = '0.12s';
  noiseFlash.style.animationDuration = '0.12s';

  // Reset after 400ms
  setTimeout(() => {
    glitch.style.animationDuration = '';
    noiseFlash.style.animationDuration = '';
  }, 400);
});
```

Inline `style` overrides the CSS animation — remove the style and the CSS value takes back over automatically.

---

### Random Burst Timer

A self-scheduling `setTimeout` fires random intensity bursts every 1.8–4s:

```js
function randomGlitchBurst() {
  const intensity = Math.random();
  const dur = 80 + Math.floor(Math.random() * 220);

  glitch.style.animationDuration = `${0.18 + intensity * 0.2}s`;

  setTimeout(() => {
    glitch.style.animationDuration = '';
  }, dur);

  // Schedule next burst at random interval
  setTimeout(randomGlitchBurst, 1800 + Math.random() * 2200);
}
```

---

## Project Structure

```
day-07-glitch-effect/
│
└── index.html    ← Everything in one file (HTML + CSS + JS)
```

---

## Tech Stack

| Tech       | Usage                                                   |
|------------|---------------------------------------------------------|
| CSS3       | RGB split, clip-path, keyframes, scanlines, noise flash |
| Vanilla JS | Click burst, random intensity timer                     |
| CSS Custom Properties | `--r`, `--c`, `--g` color variables         |

---

## Key Concepts

- **`content: attr(data-text)`** — copies element text into a pseudo-element automatically
- **`clip-path: polygon()`** — masks a horizontal band of the pseudo-element
- **`steps(1)`** — instant frame jumps with zero interpolation — the key to digital feel
- **`skewX()`** — horizontal distortion that sells the CRT monitor broken-ness
- **`mix-blend-mode: screen`** — adds light without covering — perfect for overlays
- **`repeating-linear-gradient`** — CRT scanlines with zero extra DOM elements
- **Inline style override** — setting `el.style.animationDuration` temporarily overrides CSS

---

## Design Decisions

- Red `#ff003c` and cyan `#00f0ff` are the classic RGB split colors — exactly what you'd see on a real CRT
- Green `#39ff14` used only for the `[ACTIVE]` tags — neon terminal green, third channel reference
- `Share Tech Mono` font for labels and badge — monospace reinforces the terminal/broken-screen aesthetic
- Glitch fires at 90–97% of the keyframe timeline — text is stable 90% of the time, which makes the burst more surprising
- Cards have independent delay offsets (`0.0s`, `1.3s`, `0.7s`) so they never all glitch simultaneously
- `steps(1)` on every animation — consistency makes the whole scene feel like one broken system

---

## Try It Yourself

```bash
git clone https://github.com/yourusername/30-days-insane-webdev.git
cd 30-days-insane-webdev/day-07-glitch-effect
open index.html
```

---

## 💡 Want to Experiment?

```css
/* More frequent glitch — fires at 70% instead of 90% */
0%, 69%, 100% { opacity: 0; }
70% { opacity: 1; ... }

/* Wider RGB separation */
.glitch::before { transform: translate(-12px, 0); }
.glitch::after  { transform: translate(12px, 0);  }

/* Green third channel */
.glitch::before { color: #39ff14; }

/* Glitch every element, not just text */
.gcard {
  animation: card-glitch 1s steps(1) infinite;
}
```

```js
// Continuous glitch — never stops
glitch.style.animationDuration = '0.15s';

// Slower random bursts
setTimeout(randomGlitchBurst, 5000 + Math.random() * 5000);
```

---

## The Series

| Day | Project | Status |
|-----|---------|--------|
| 01 | Magnetic Cursor | ✅ Done |
| 02 | Liquid Blob Background | ✅ Done |
| 03 | 3D Card Flip Gallery | ✅ Done |
| 04 | Particle Text Explosion | ✅ Done |
| 05 | Infinite Warping Marquee | ✅ Done |
| 06 | Animated Gradient Mesh | ✅ Done |
| 07 | Glitch Hover Effect | ✅ Done |
| 08 | Cursor Paint Trail | 🔜 Tomorrow |
| ... | ... | ... |
| 30 | Dark/Light Mode Flip Toggle | 🔜 Coming |

---

## Follow Along

> Built this in a day as part of my **#30DaysOfInsaneWebDev** challenge.
> Follow on Instagram for the daily Reels 👉 **@vladlen.codes**

---

## 📄 License

MIT — use it, remix it, build on it. Star ⭐ the repo if this helped you!
