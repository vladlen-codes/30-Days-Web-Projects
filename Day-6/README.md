# Day 06 / 30 — Animated Gradient Mesh

> **30 Days of Insane Web Development** — Building one wild UI experiment every day.

![Day 06](https://img.shields.io/badge/Day-06%20%2F%2030-db2777?style=for-the-badge)
![WebGL](https://img.shields.io/badge/WebGL-GLSL-a855f7?style=for-the-badge)
![Fragment Shader](https://img.shields.io/badge/Fragment-Shader-0891b2?style=for-the-badge)
![No Libraries](https://img.shields.io/badge/Libraries-Zero-success?style=for-the-badge)

---

## What I Built?

An **Apple-style animated gradient mesh** powered by a WebGL fragment shader — 6 color nodes drifting on independent sine paths, blending together using Inverse Distance Weighting (IDW). Every color is generated from a cosine formula. No hex codes. No images. Just a GPU program running on every pixel, 60 times a second.

- 6 animated color nodes drifting on independent sine/cosine paths
- Colors generated entirely from cosine math — zero hardcoded hex values
- IDW blending — every pixel blends all 6 nodes weighted by distance
- Film grain overlay — makes it feel tactile and photographic
- Vignette — darkens edges for a cinematic focused composition
- Runs entirely on the GPU — zero CPU per-pixel work
- Phone frame (390×844px) — built for Instagram Reels

---

## Live Demo

> Open `index.html` in any modern browser — just watch the colors breathe!

---

## 🧠 How It Works

### What Is a Fragment Shader?

A fragment shader is a tiny program that runs **once per pixel, on the GPU, every frame**. For a 390×844 canvas at 60fps that's `390 × 844 × 60 = ~19.7 million executions per second` — all in parallel.

The shader receives the pixel's position (`gl_FragCoord`) and outputs a color (`gl_FragColor`). Everything between those two is the effect.

---

### Colors From Cosine — No Hex Codes

Every color in the mesh is generated from a single cosine formula:

```glsl
// The cosine palette formula
// Outputs smooth RGB values in 0.0–1.0 range
vec3 c0 = 0.5 + 0.5 * cos(vec3(
  0.0 + t * 0.06,   // red channel — phase + drift speed
  1.0 + t * 0.05,   // green channel — phase + drift speed
  2.6               // blue channel — static phase
));
```

`0.5 + 0.5 * cos(x)` always outputs values between 0 and 1 — perfect for colors. By using different phase offsets per channel (R, G, B) and multiplying `t` by different speeds, each color node slowly evolves through the spectrum independently.

| Node | Base Hue | Drifts Toward |
|------|----------|---------------|
| c0   | Purple   | Indigo        |
| c1   | Pink     | Rose          |
| c2   | Cyan     | Teal          |
| c3   | Blue     | Violet        |
| c4   | Yellow   | Orange        |
| c5   | Magenta  | Fuchsia       |

---

### Animated Node Positions

Each node drifts on its own orbit using independent sine/cosine paths:

```glsl
// Node 0 — orbits upper-left area
vec2 p0 = vec2(
  0.25 + 0.22 * sin(t * 0.40),         // x oscillates ±0.22
  0.18 + 0.14 * cos(t * 0.33)          // y oscillates ±0.14
);

// Node 1 — orbits upper-right area
vec2 p1 = vec2(
  0.80 + 0.14 * cos(t * 0.37 + 1.2),  // +1.2 phase offset = never syncs with p0
  0.28 + 0.20 * sin(t * 0.29)
);
```

The phase offsets (`+ 1.2`, `+ 2.4` etc.) ensure no two nodes ever move in sync — if they did, the mesh would pulse uniformly instead of flowing organically.

---

### IDW Blending — How Every Pixel Gets Its Color

Inverse Distance Weighting means: the closer a node is to a pixel, the more it influences that pixel's color.

```glsl
float EXP = 4.0;  // power — higher = tighter, more distinct color regions

// Weight = 1 / distance^4
// Nearby nodes dominate, far nodes barely contribute
float w0 = 1.0 / pow(max(distance(fuv, fp0), 0.0001), EXP);
float w1 = 1.0 / pow(max(distance(fuv, fp1), 0.0001), EXP);
// ... w2 through w5

float wSum = w0+w1+w2+w3+w4+w5;

// Weighted average of all 6 node colors
vec3 col = (c0*w0 + c1*w1 + c2*w2 + c3*w3 + c4*w4 + c5*w5) / wSum;
```

With `EXP = 4.0` transitions are crisp but smooth. `EXP = 2.0` gives a washed blurry mix. `EXP = 8.0` gives sharp Voronoi-like regions.

---

### Tone Mapping + Saturation

Raw cosine colors can look muddy. Two post-processing steps fix this:

```glsl
// 1. Gamma brighten — pow < 1 lifts shadows
col = pow(col, vec3(0.88));

// 2. Saturation boost
float lum = dot(col, vec3(0.299, 0.587, 0.114)); // luminance
col = mix(vec3(lum), col, 1.35);                  // 1.35× saturation
```

---

### Film Grain

A hash function generates pseudo-random noise per pixel per frame:

```glsl
float hash(vec2 p) {
  return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453);
}

float grain(vec2 uv, float t) {
  return hash(uv + fract(t * 0.07)) * 0.10 - 0.05; // ±5% brightness noise
}

col += grain(uv, t);
```

`fract(t * 0.07)` makes the grain animate slowly — it flickers subtly rather than being a static texture.

---

### Vignette

Darkens edges using a radial smooth falloff:

```glsl
float vignette(vec2 uv) {
  vec2 d = uv - 0.5;
  d.y *= u_res.y / u_res.x;  // correct for aspect ratio
  return 1.0 - smoothstep(0.35, 0.85, length(d) * 1.5);
}

col *= mix(0.60, 1.0, vignette(uv)); // edges at 60% brightness
```

---

### WebGL Bootstrap

The minimal setup to get a shader running in the browser:

```js
const gl = canvas.getContext('webgl');

// Compile + link shaders
const prog = gl.createProgram();
gl.attachShader(prog, compileShader(VS, gl.VERTEX_SHADER));
gl.attachShader(prog, compileShader(FS, gl.FRAGMENT_SHADER));
gl.linkProgram(prog);
gl.useProgram(prog);

// Full-screen quad — 2 triangles covering clip space (-1 to 1)
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array([
  -1,-1,  1,-1,  -1, 1,
  -1, 1,  1,-1,   1, 1,
]), gl.STATIC_DRAW);

// Render loop — pass elapsed time to shader every frame
function render() {
  gl.uniform1f(uTime, (performance.now() - t0) / 1000);
  gl.drawArrays(gl.TRIANGLES, 0, 6);
  requestAnimationFrame(render);
}
```

---

## Project Structure

```
day-06-gradient-mesh/
│
└── index.html    ← Everything in one file (HTML + CSS + JS + GLSL)
```

---

## Tech Stack

| Tech       | Usage                                          |
|------------|------------------------------------------------|
| WebGL      | GPU-accelerated canvas rendering               |
| GLSL       | Fragment shader — per-pixel color computation  |
| Vanilla JS | WebGL bootstrap, render loop, uniform passing  |
| CSS3       | UI overlay, badge, glassmorphism pills         |

---

## Key Concepts

- **Fragment shader** — GPU program that runs once per pixel per frame
- **`gl_FragCoord`** — built-in GLSL: current pixel's screen position
- **`gl_FragColor`** — built-in GLSL output: the pixel's final color
- **Cosine palette** — `0.5 + 0.5 * cos(phase + t * speed)` generates smooth cycling colors
- **IDW** — blend colors by `1/distance^n` for smooth spatial interpolation
- **Uniform** — a value passed from JS → shader each frame (`u_time`, `u_res`)
- **`smoothstep`** — GLSL's smooth cubic interpolation between two edge values
- **Film grain via hash** — deterministic pseudo-random noise from pixel coordinates

---

## Design Decisions

- `EXP = 4.0` for IDW — crisp color regions without looking like Voronoi cells
- 6 nodes covers a 390×844 canvas well — fewer looks sparse, more is overkill
- Phase offsets on paths ensure zero synchronization — organic, never robotic
- Grain at `±5%` is the sweet spot — visible up close, invisible in motion
- Vignette floor at `0.60` keeps edges dark but not black — preserves atmosphere
- `pow(col, 0.88)` gamma lift tuned by eye — prevents dark muddy patches

---

## Try It Yourself

```bash
git clone https://github.com/yourusername/30-days-insane-webdev.git
cd 30-days-insane-webdev/day-06-gradient-mesh
open index.html
```

---

## Want to Experiment?

```glsl
/* Sharper color regions */
float EXP = 8.0;   // was 4.0

/* Softer, more washed mix */
float EXP = 2.0;

/* Faster color evolution */
vec3 c0 = 0.5 + 0.5*cos(vec3(0.0 + t*0.3, 1.0 + t*0.25, 2.6));

/* Add a 7th node */
vec2 p6 = vec2(0.50 + 0.20*sin(t*0.28 + 1.0), 0.50 + 0.20*cos(t*0.36));
vec3 c6 = 0.5 + 0.5*cos(vec3(6.0 + t*0.06, 3.5 + t*0.05, 2.0));
float w6 = 1.0 / pow(max(distance(fuv, p6), 0.0001), EXP);
// Add w6 to wSum and col calculation

/* Remove grain */
// col += grain(uv, t);

/* Remove vignette */
// col *= mix(0.60, 1.0, vignette(uv));
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
| 07 | Glitch Hover Effect | 🔜 Tomorrow |
| ... | ... | ... |
| 30 | Dark/Light Mode Flip Toggle | 🔜 Coming |

---

## Follow Along

> Built this in a day as part of my **#30DaysOfInsaneWebDev** challenge.
> Follow on Instagram for the daily Reels 👉 **@vladlen.codes**

---

## 📄 License

MIT — use it, remix it, build on it. Star ⭐ the repo if this helped you!
