# 🃏 Day 03 / 30 — 3D Card Flip Gallery

> **30 Days of Insane Web Development** — Building one wild UI experiment every day.

![Day 03](https://img.shields.io/badge/Day-03%20%2F%2030-6d28d9?style=for-the-badge)
![CSS 3D](https://img.shields.io/badge/CSS-3D%20Transforms-1572B6?style=for-the-badge&logo=css3&logoColor=white)
![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-f7df1e?style=for-the-badge&logo=javascript)
![No Libraries](https://img.shields.io/badge/Libraries-Zero-success?style=for-the-badge)

---

## What I Built

A **3D card flip gallery** with 4 cards that flip to reveal content on the back — built with pure CSS 3D transforms. No libraries, no GSAP, no WebGL. Just `transform-style: preserve-3d` and math.

- 4 cards with unique gradient themes — flip to reveal facts on the back
- Auto-cycle loop — cards flip sequentially, hold, then unflip in reverse
- Hover tilt — subtle 3D depth effect when you mouse over each card
- Springy cubic-bezier — cards slightly overshoot 180° for a bouncy premium feel
- Progress dots — update live as each card flips
- Phone frame (390×844px) — built for Instagram Reels

---

## Live Demo

> Open `index.html` in any modern browser and click the cards — or just watch the auto-cycle!

---

## How It Works

### The Entire Flip — Just 2 CSS Properties

This is the whole effect. No JS animation, no keyframes — just toggling a class:

```css
.card {
  transform-style: preserve-3d;
  transition: transform 0.72s cubic-bezier(0.34, 1.3, 0.64, 1);
}

.card.flipped {
  transform: rotateY(180deg);
}
```

`transform-style: preserve-3d` tells the browser to render child elements in 3D space. `rotateY(180deg)` spins the card around the Y axis. That's literally it.

---

### The Springy Bounce

The `cubic-bezier(0.34, 1.3, 0.64, 1)` is what makes it feel premium instead of robotic:

```
cubic-bezier(0.34, 1.3, 0.64, 1)
                    ^^^
              Overshoots 1.0 — causes the slight bounce past 180°
```

Swap it with `ease` and the flip feels flat. This one line is the difference between "ok" and "wow".

---

### The Back Face Trick

Both `.front` and `.back` are stacked absolutely on top of each other. `backface-visibility: hidden` makes each face invisible when it's rotated away from you:

```css
.face {
  position: absolute;
  inset: 0;
  backface-visibility: hidden;
}

/* Back face starts pre-rotated so it's hidden by default */
.back {
  transform: rotateY(180deg);
}
```

When the card is at 0°: front is visible, back is hidden.
When the card is at 180°: back is visible, front is hidden.

---

### Click to Flip — Pure JS Class Toggle

```js
wraps.forEach((wrap, i) => {
  wrap.addEventListener('click', () => {
    const card = wrap.querySelector('.card');
    card.classList.toggle('flipped');
  });
});
```

One line. Toggle `.flipped` → CSS does the rest.

---

### Auto-Cycle Loop

Cards flip one by one with staggered delays, hold, then unflip in reverse:

```js
async function autoLoop() {
  while (true) {
    // Flip cards 0 → 1 → 2 → 3
    for (let i = 0; i < 4; i++) {
      setCardFlip(i, true);
      await sleep(900);       // 900ms between each flip
    }

    await sleep(1800);        // Hold all flipped for 1.8s

    // Unflip in reverse: 3 → 2 → 1 → 0
    for (let i = 3; i >= 0; i--) {
      setCardFlip(i, false);
      await sleep(600);
    }

    await sleep(1200);        // Pause before next cycle
  }
}
```

The `async/await + sleep()` pattern keeps the timing clean without nested `setTimeout` hell.

---

### Hover Tilt — Subtle 3D Depth

When you hover over an unflipped card, it tilts toward your cursor:

```js
wrap.addEventListener('mousemove', e => {
  const r = wrap.getBoundingClientRect();

  // Normalize mouse position to -1 → 1 range
  const nx = ((e.clientX - r.left) / r.width - 0.5) * 2;
  const ny = ((e.clientY - r.top) / r.height - 0.5) * 2;

  const tiltX = ny * -8;   // max 8° tilt on X axis
  const tiltY = nx * 8;    // max 8° tilt on Y axis

  card.style.transform = `rotateX(${tiltX}deg) rotateY(${tiltY}deg)`;
});
```

Normalizing to `-1 → 1` makes the tilt work for any card size — no hardcoded pixel values.

---

## Project Structure

```
day-03-3d-card-flip/
│
└── index.html    ← Everything in one file (HTML + CSS + JS)
```

---

## Tech Stack

| Tech        | Usage                                          |
|-------------|------------------------------------------------|
| CSS3        | 3D transforms, `preserve-3d`, `backface-visibility`, transitions |
| Vanilla JS  | Click handler, auto-cycle loop, hover tilt     |
| SVG         | Inline icons on card fronts and backs          |

---

## Key Concepts

- **`transform-style: preserve-3d`** — enables 3D space for child elements
- **`backface-visibility: hidden`** — hides a face when rotated away from viewer
- **`perspective: 900px`** — set on the parent to define the 3D depth feeling
- **`rotateY(180deg)`** — flips the card around the vertical axis
- **`cubic-bezier(0.34, 1.3, 0.64, 1)`** — overshoot easing for a springy bounce
- **`async/await + sleep()`** — clean sequential animation timing without callback hell

---

## Design Decisions

- 4 unique gradient themes — purple, pink, blue, green — each card feels like a world
- Back face uses darker background with a colored icon, divider line, and a CSS tech tag
- Progress dots at the bottom track which cards are flipped — small detail, big polish
- The bottom code bar surfaces `rotateY(180deg)` and the cubic-bezier visually
- Auto-cycle starts 800ms after load so the initial render doesn't feel rushed

---

## Try It Yourself

```bash
git clone https://github.com/yourusername/30-days-insane-webdev.git
cd 30-days-insane-webdev/day-03-3d-card-flip
open index.html
```

---

## Want to Experiment?

```css
/* Flip on X axis instead of Y — top-to-bottom flip */
.card.flipped {
  transform: rotateX(180deg);
}

/* Diagonal flip — feels wild */
.card.flipped {
  transform: rotateY(180deg) rotateZ(10deg);
}

/* Slower, more dramatic */
transition: transform 1.4s cubic-bezier(0.34, 1.3, 0.64, 1);

/* More aggressive tilt */
const tiltX = ny * -20;
const tiltY = nx * 20;
```

---

## The Series

| Day | Project | Status |
|-----|---------|--------|
| 01 | Magnetic Cursor | ✅ Done |
| 02 | Liquid Blob Background | ✅ Done |
| 03 | 3D Card Flip Gallery | ✅ Done |
| 04 | Particle Text Explosion | 🔜 Tomorrow |
| ... | ... | ... |
| 30 | Dark/Light Mode Flip Toggle | 🔜 Coming |

---

## 📲 Follow Along

> Built this in a day as part of my **#30DaysOfInsaneWebDev** challenge.  
> Follow on Instagram for the daily Reels 👉 **@vladlen-codes**

---

## 📄 License

MIT — use it, remix it, build on it. Star ⭐ the repo if this helped you!
