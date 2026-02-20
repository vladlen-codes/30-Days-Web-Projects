# Day 01 / 30 — Magnetic Cursor

> **30 Days of Insane Web Development** — Building one wild UI experiment every day.

![Day 01 - Magnetic Cursor](https://img.shields.io/badge/Day-01%20%2F%2030-8b5cf6?style=for-the-badge)
![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-f7df1e?style=for-the-badge&logo=javascript)
![HTML](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)

---

## What I Built?

A **magnetic cursor effect** where UI elements get physically attracted to your cursor as it moves nearby — built from scratch with zero libraries. Just pure math and `requestAnimationFrame`.

-  Buttons magnetically pull toward the cursor within a set radius
-  Custom cursor with a lagging ring for a juicy, premium feel
-  Particle trail that spawns glowing purple/magenta particles as you move
-  Idle demo loop — auto-animates when the user hasn't moved yet
-  Styled as a phone frame (390×844px) — perfect for Instagram Reels

---

##  Live Demo

> Open `index.html` in your browser and move your cursor near the buttons!

---

##  How It Works

### The Magnetic Effect

The core magic is a `Magnetic` class that calculates the distance between your cursor and a target element. If you're within the `radius`, it pushes the element toward you using linear interpolation (lerp).

```js
move(e) {
  const dx = px - x, dy = py - y;
  const dist = Math.hypot(dx, dy);

  if (dist < this.R) {
    const f = (this.R - dist) / this.R;   // falloff factor
    this.tx = dx * f * this.S;            // target X offset
    this.ty = dy * f * this.S;            // target Y offset
  }
}
```

### The Lerp Smoothing

One line makes it feel buttery smooth:

```js
this.cx += (this.tx - this.cx) * this.L;
```

`lerp(current, target, 0.12)` — the lower the last value, the more floaty and laggy the motion feels.

### Spawning a Magnetic Element

```js
new Magnetic(element, { radius: 120, strength: 0.6, lerp: 0.14 });
```

| Parameter  | Description                                              |
|------------|----------------------------------------------------------|
| `radius`   | Distance in px before magnet activates                   |
| `strength` | How far the element moves (0–1)                          |
| `lerp`     | Smoothing factor — lower = floatier, higher = snappier   |

---

##  Project Structure

```
day-01-magnetic-cursor/
│
└── index.html       ← Everything in one file (HTML + CSS + JS)
```

---

##  Tech Stack

| Tech         | Usage                                  |
|--------------|----------------------------------------|
| HTML5        | Structure & canvas element             |
| CSS3         | Custom cursor, animations, glass UI    |
| Vanilla JS   | Magnetic physics, particle system, rAF |
| Canvas API   | Particle trail rendering               |

---

##  Key Concepts

- **`Math.hypot(dx, dy)`** — calculates distance between two points
- **`requestAnimationFrame`** — smooth 60fps animation loop
- **Linear Interpolation (Lerp)** — `a + (b - a) * t` for smooth easing
- **`getBoundingClientRect()`** — gets element position relative to viewport
- **`mix-blend-mode: difference`** — makes the cursor dot invert colors underneath

---

##  Design Decisions

- Dark `#07060f` background to make the purple glow pop
- Animated ambient orbs in the background add depth without distraction
- Grid overlay with a radial mask creates a subtle techy feel
- The cursor ring expands and changes color on hover — tiny detail, big impact
- Bottom bar shows `lerp(current, target, 0.12)` as a code snippet — educational and aesthetic

---

##  Try It Yourself

1. Clone this repo
2. Open `index.html` in any modern browser
3. Move your cursor near the buttons — feel the pull 🧲

```bash
git clone https://github.com/yourusername/30-days-insane-webdev.git
cd 30-days-insane-webdev/day-01-magnetic-cursor
open index.html
```

---

##  Want to Experiment?

Try tweaking these values in the JS:

```js
// Super dramatic — big radius, strong pull, floaty
new Magnetic(btn, { radius: 200, strength: 0.9, lerp: 0.05 });

// Subtle and snappy
new Magnetic(btn, { radius: 60, strength: 0.3, lerp: 0.25 });

// Dreamy and slow
new Magnetic(btn, { radius: 150, strength: 0.5, lerp: 0.04 });
```

---

##  The Series

| Day | Project | Status |
|-----|---------|--------|
| 01  | Magnetic Cursor | ✅ Done |
| 02  | Liquid Blob Background | 🔜 Tomorrow |
| 03  | 3D Card Flip Gallery | 🔜 Coming |
| ... | ... | ... |
| 30  | Dark/Light Mode Flip Toggle | 🔜 Coming |

---

##  Follow Along

> Built this in a day as part of my **#30DaysOfInsaneWebDev** challenge.  
> Follow on Instagram for the daily Reels 👉 **@vladlen.codes**

---

##  License

MIT — use it, remix it, build on it. Just don't forget to star ⭐ the repo!
