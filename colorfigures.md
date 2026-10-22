---
name: colorfigures
description: Generate a single, self-contained interactive HTML file featuring a hand-inked, flat-coloured isometric everyday object or machine with working buttons, 360-degree turntable rotation, see-through internal mechanism view, and numbered part labels. Use whenever the user describes an object, machine, device, or process and wants an interactive figure or Colorfigure.
---

# Colorfigures — Single-File Interactive Isometric Figure Generator

You build **Colorfigures**: hand-inked, flat-coloured interactive isometric machines and everyday objects delivered as a **single, self-contained HTML file** with zero external dependencies.

> **IMPORTANT STANDALONE RULE:** Do NOT attempt to read external `references/`, `assets/`, or `scripts/` files. Everything you need—the mathematical 3D isometric projection kernel, painter's-order depth sorter, SVG hand-drawn wobble filter, see-through cutaway system, and HTML/CSS/JS shell—is provided inline in this file. Always output one complete, runnable `.html` file with no placeholders or omitted code.

---

## 1. Core Experience & Non-Negotiable Features

Every figure you generate MUST include all six of these capabilities:

1. **Hand-Inked Isometric Aesthetic + Wobble Toggle**
   - Warm editorial paper background (`#f6f2e8`), crisp dark ink outlines (`#1c1917`, `stroke-width: 1.8`, `stroke-linejoin: round`, `stroke-linecap: round`), and flat cel-shaded/risograph fills.
   - An SVG `<filter id="wobbleFilter">` using `feTurbulence` + `feDisplacementMap` applied to the drawing group, controlled by a **`Hand-drawn wobble`** toggle pill.
   - A **`Colour`** toggle/switcher that cycles between curated flat-colour palettes (Warm Editorial, Risograph Coral/Teal, Technical Blueprint, Ink Monochrome).

2. **Full 360° Turntable Rotation + Zoom**
   - Every object is built in true 3D coordinates `(x, y, z)` and projected to 2D SVG using an isometric turntable projection.
   - Dragging anywhere on the canvas smoothly rotates the yaw angle (`yaw`) a full 360° around the vertical axis (`Z` up).
   - Provide quick camera buttons: `⟲`, `front`, `⟳`, plus zoom controls `−`, `100%`, `+`.

3. **See-Through Mode (X-Ray / Cutaway)**
   - Every 3D primitive or face is tagged with `layer: 'shell'` (outer housing/casing/doors) or `layer: 'inner'` (internal workings: gears, heating coils, springs, valves, pistons, wiring, fluids, bimetallic discs).
   - When **`See-through`** is toggled ON, `'shell'` geometry drops to `fill-opacity: 0.14` and dashed/lightened strokes (`stroke-opacity: 0.35`), clearly revealing the internal mechanism operating inside.

4. **Numbered Part Labels That Light Up When Active**
   - Define 4 to 7 key functional components with 3D anchor coordinates `(x, y, z)`, a short uppercase label (e.g., `1 HEATER COIL`, `2 BIMETALLIC DISC`), and an `isActive(state)` predicate.
   - When **`Labels`** is toggled ON, render numbered circular callout badges projected onto the SVG AND a numbered pill strip above/below the canvas.
   - Whenever a part is actively working in the current simulation state, its badge and pill light up with the live accent colour (`--accent`) and pulse gently.

5. **Buttons That Actually Work (State Machine + Physics Loop)**
   - Decompose the object's real-world operation into a clear, causal state machine (e.g., for a Kettle: `empty` -> `filled` -> `heating` (water temp rises 20°C -> 100°C, bubbles rise, steam vents, bimetallic disc flexes) -> `clicked-off` -> `pouring`).
   - Provide **interactive action buttons** in the top bar (with keyboard shortcuts in parentheses, e.g., `Fill water (F)`, `Switch on (Space)`, `Pour (P)`) AND make the actual SVG parts clickable (`cursor: pointer`).
   - Include a live status line explaining the current state and prompting the next action (e.g., *"empty — fill with water (F) to start"*).

6. **"The big idea" Educational Callout**
   - Include a concise, plain-language **"The big idea"** card explaining the core mechanical or physical principle behind how the object works (e.g., *"Spinning fast flings the water out through the little holes in the drum, while the clothes get pressed against the wall."* or *"Steam travels down a hidden tube to heat a bimetallic disc at the base, which snaps concave at 100°C and trips the switch."*).

---

## 2. How to Construct Any Object in 3D Primitives

Because the user can rotate the figure 360°, **never hardcode static 2D isometric polygons**. Instead, build the object procedurally each frame inside `buildScene(state, t)` using the provided 3D helper primitives:

- `addBox(scene, { x, y, z, w, d, h, color, layer, id, onClick, active })`
  - Adds a 3D rectangular prism centered at `(x, y, z)` with width `w` (X), depth `d` (Y), height `h` (Z).
  - Supports rotation around its own center or hinge point via `rx, ry, rz` or `hinge` (for opening doors, flipping switches, tilting levers, lifting tonearms).
- `addCylinder(scene, { x, y, z, r, h, axis, segments, color, layer, id, onClick, active, openTop })`
  - Adds a 3D cylinder or disc along `'z'`, `'x'`, or `'y'` axis (great for drums, kettles, platters, pipes, buttons, dials, coins, wheels).
- `addSphere(scene, { x, y, z, r, color, layer, active })`
  - Adds a 2.5D shaded ink sphere/circle at projected `(x, y, z)` (great for gumballs, water droplets, steam puffs, indicator LEDs, bearings).
- `addLine3D(scene, { p1: [x1,y1,z1], p2: [x2,y2,z2], color, width, dashed, layer, active })`
  - Adds 3D structural rods, springs, wires, water streams, or airflow arrows.
- `addFaceDetail(scene, { points3D, color, stroke, fillOpacity, layer, onClick, active })`
  - Adds custom 3D polygons (cutouts, windows, display screens, gauge needles, slots, perforations) that sort and rotate seamlessly with the rest of the model.

### Design Checklist for Richness & Legibility
- **Scale & Proportions:** Work in a coordinate space roughly `-120` to `+120` on X/Y/Z. Keep proportions chunky, charming, and unmistakable.
- **Always Build the Inside:** Even when `See-through` starts OFF, always model the internal components (`layer: 'inner'`)—heating elements, gears, chutes, valves, circuits, fluids—so turning on `See-through` (or opening a door/hatch) is a magical reveal.
- **Secondary Motion:** Add small continuous animations driven by `t` (elapsed time): rising steam particles, bubbling water, rotating drums/records, vibrating motors, glowing filaments, or swaying pendulums.

---

## 3. Master Self-Contained HTML Template

Use the following complete HTML/CSS/JS architecture as your foundation. Replace the `OBJECT_CONFIG` and `buildScene(scene, state, t)` / `ACTIONS` sections with the rich, specific geometry and state machine for the user's requested object.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Fig. 01 — Interactive Colorfigure</title>
<style>
  :root {
    --bg: #f6f2e8;
    --panel: #efeadd;
    --ink: #1c1917;
    --muted: #6b6358;
    --accent: #e05a3a;
    --accent-soft: #fde6dd;
    --highlight: #f5b73b;
    --border: #d8d0c0;
    --font-sans: 'Inter', -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    --font-mono: 'JetBrains Mono', 'SF Mono', ui-monospace, Menlo, monospace;
  }

  [data-palette="risograph"] {
    --bg: #f4f1ea;
    --panel: #e9e4d8;
    --ink: #182232;
    --accent: #ff4e50;
    --accent-soft: #ffe0e0;
    --highlight: #2ab7ca;
  }

  [data-palette="blueprint"] {
    --bg: #e8eff7;
    --panel: #d8e4f2;
    --ink: #0f2547;
    --muted: #4a6388;
    --accent: #d9480f;
    --accent-soft: #ffd8c4;
    --border: #b8cce4;
  }

  [data-palette="mono"] {
    --bg: #f5f5f3;
    --panel: #e7e7e3;
    --ink: #141413;
    --muted: #666660;
    --accent: #141413;
    --accent-soft: #d8d8d2;
    --border: #ccccc4;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: var(--bg);
    color: var(--ink);
    font-family: var(--font-sans);
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px 16px 32px;
    user-select: none;
    -webkit-font-smoothing: antialiased;
  }

  .plate {
    width: 100%;
    max-width: 880px;
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  /* Header */
  .plate-header {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    flex-wrap: wrap;
    gap: 8px;
    border-bottom: 1.5px solid var(--ink);
    padding-bottom: 10px;
  }
  .fig-title {
    font-size: 1.25rem;
    font-weight: 700;
    letter-spacing: -0.02em;
  }
  .fig-num {
    font-family: var(--font-mono);
    font-size: 0.85rem;
    font-weight: 600;
    color: var(--muted);
    margin-right: 8px;
  }
  .fig-subtitle {
    font-size: 0.88rem;
    color: var(--muted);
  }

  /* Primary Action Buttons */
  .action-bar {
    display: flex;
    flex-wrap: wrap;
    gap: 8px;
    align-items: center;
    justify-content: space-between;
  }
  .action-group {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
  }
  button.btn {
    appearance: none;
    background: #fffdf8;
    color: var(--ink);
    border: 1.5px solid var(--ink);
    border-radius: 999px;
    padding: 6px 13px;
    font-family: var(--font-sans);
    font-size: 0.82rem;
    font-weight: 600;
    cursor: pointer;
    box-shadow: 0 2px 0 var(--ink);
    transition: transform 0.08s ease, box-shadow 0.08s ease, background 0.15s ease;
    display: inline-flex;
    align-items: center;
    gap: 6px;
  }
  button.btn:hover:not(:disabled) {
    background: var(--accent-soft);
  }
  button.btn:active:not(:disabled) {
    transform: translateY(2px);
    box-shadow: 0 0 0 var(--ink);
  }
  button.btn:disabled {
    opacity: 0.4;
    cursor: not-allowed;
    box-shadow: none;
  }
  button.btn.primary, button.btn.active {
    background: var(--accent);
    color: #fff;
  }
  .kbd {
    font-family: var(--font-mono);
    font-size: 0.7rem;
    opacity: 0.75;
    padding: 1px 4px;
    border-radius: 4px;
    background: rgba(0,0,0,0.08);
  }

  /* Canvas Stage */
  .stage-wrap {
    position: relative;
    background: var(--panel);
    border: 1.5px solid var(--ink);
    border-radius: 14px;
    height: 460px;
    overflow: hidden;
    cursor: grab;
    box-shadow: inset 0 2px 6px rgba(0,0,0,0.03);
  }
  .stage-wrap:active { cursor: grabbing; }

  svg#stage {
    width: 100%;
    height: 100%;
    display: block;
  }

  /* Floating Stage Overlays */
  .hud-top-left {
    position: absolute;
    top: 12px;
    left: 12px;
    display: flex;
    gap: 6px;
    flex-wrap: wrap;
    pointer-events: none;
  }
  .readout-badge {
    font-family: var(--font-mono);
    font-size: 0.76rem;
    font-weight: 600;
    padding: 4px 9px;
    border-radius: 6px;
    background: rgba(255, 253, 248, 0.9);
    border: 1.2px solid var(--ink);
    color: var(--ink);
  }
  .readout-badge.live {
    background: var(--accent);
    color: #fff;
  }

  .hud-bottom {
    position: absolute;
    bottom: 12px;
    left: 12px;
    right: 12px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 8px;
    pointer-events: none;
  }
  .hud-cluster {
    display: flex;
    gap: 4px;
    background: rgba(255, 253, 248, 0.92);
    border: 1.2px solid var(--ink);
    border-radius: 999px;
    padding: 3px;
    pointer-events: auto;
  }
  .hud-btn {
    border: none;
    background: transparent;
    color: var(--ink);
    font-family: var(--font-mono);
    font-size: 0.74rem;
    font-weight: 600;
    padding: 4px 9px;
    border-radius: 999px;
    cursor: pointer;
  }
  .hud-btn:hover { background: rgba(0,0,0,0.06); }
  .hud-btn.on {
    background: var(--ink);
    color: var(--bg);
  }

  /* Labels Strip */
  .labels-strip {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    padding: 4px 0;
  }
  .label-pill {
    font-family: var(--font-mono);
    font-size: 0.73rem;
    font-weight: 600;
    padding: 4px 10px;
    border-radius: 6px;
    border: 1.2px solid var(--ink);
    background: #fffdf8;
    color: var(--ink);
    display: inline-flex;
    align-items: center;
    gap: 6px;
    transition: all 0.15s ease;
  }
  .label-pill .num {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    width: 17px;
    height: 17px;
    border-radius: 50%;
    background: var(--ink);
    color: #fff;
    font-size: 0.68rem;
  }
  .label-pill.active {
    background: var(--accent-soft);
    border-color: var(--accent);
  }
  .label-pill.active .num {
    background: var(--accent);
  }

  /* Status & Big Idea Footer */
  .status-line {
    font-family: var(--font-mono);
    font-size: 0.8rem;
    color: var(--muted);
    display: flex;
    justify-content: space-between;
    align-items: center;
    flex-wrap: wrap;
    gap: 8px;
  }
  .big-idea {
    background: #fffdf8;
    border: 1.5px solid var(--ink);
    border-radius: 10px;
    padding: 12px 16px;
    font-size: 0.88rem;
    line-height: 1.45;
    display: flex;
    gap: 12px;
    align-items: baseline;
  }
  .big-idea-tag {
    font-family: var(--font-mono);
    font-size: 0.72rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.04em;
    color: var(--accent);
    white-space: nowrap;
  }
  .clickable-part { cursor: pointer; }
  .clickable-part:hover { filter: brightness(1.06); }
</style>
</head>
<body>

<div class="plate">
  <header class="plate-header">
    <div>
      <span class="fig-num" id="figNum">Fig. 01</span>
      <span class="fig-title" id="figTitle">Object Name</span>
    </div>
    <div class="fig-subtitle" id="figSubtitle">Interactive isometric mechanism — drag to turn 360°</div>
  </header>

  <div class="action-bar">
    <div class="action-group" id="actionButtons"></div>
    <div class="action-group">
      <button class="btn active" id="btnWobble">Hand-drawn wobble</button>
      <button class="btn" id="btnPalette">Colour</button>
    </div>
  </div>

  <div class="stage-wrap" id="stageWrap">
    <div class="hud-top-left" id="readoutHud"></div>

    <svg id="stage" viewBox="-400 -230 800 460">
      <defs>
        <!-- Signature Hand-Inked Wobble Filter -->
        <filter id="wobbleFilter" x="-10%" y="-10%" width="120%" height="120%">
          <feTurbulence id="turb" type="fractalNoise" baseFrequency="0.022" numOctaves="3" seed="3" result="noise" />
          <feDisplacementMap in="SourceGraphic" in2="noise" scale="2.3" xChannelSelector="R" yChannelSelector="G" />
        </filter>
      </defs>
      <g id="viewport">
        <g id="sceneGroup" filter="url(#wobbleFilter)"></g>
        <g id="labelsGroup"></g>
      </g>
    </svg>

    <div class="hud-bottom">
      <div class="hud-cluster">
        <button class="hud-btn" id="rotLeft" title="Rotate Left">⟲</button>
        <button class="hud-btn" id="rotFront" title="Reset Front View">front</button>
        <button class="hud-btn" id="rotRight" title="Rotate Right">⟳</button>
      </div>
      <div class="hud-cluster">
        <button class="hud-btn" id="zoomOut">−</button>
        <button class="hud-btn" id="zoomReset">100%</button>
        <button class="hud-btn" id="zoomIn">+</button>
      </div>
      <div class="hud-cluster">
        <button class="hud-btn" id="btnSeeThrough">See-through</button>
        <button class="hud-btn on" id="btnLabels">Labels</button>
      </div>
    </div>
  </div>

  <div class="labels-strip" id="labelsStrip"></div>

  <div class="status-line">
    <span id="statusText">Ready — press a button or click parts on the figure.</span>
    <span>Drag figure to orbit · Scroll or +/− to zoom</span>
  </div>

  <div class="big-idea">
    <span class="big-idea-tag">The big idea</span>
    <span id="bigIdeaText"></span>
  </div>
</div>

<script>
/* ============================================================================
   1. 3D ISOMETRIC PROJECTION & PAINTER'S-ORDER KERNEL
   ============================================================================ */
const cam = {
  yaw: -Math.PI / 4,     // 45 deg classic isometric yaw
  pitch: Math.PI / 6,    // 30 deg isometric elevation
  zoom: 1.0,
  targetYaw: -Math.PI / 4,
  wobble: true,
  seeThrough: false,
  showLabels: true,
  paletteIdx: 0
};
const PALETTES = ['default', 'risograph', 'blueprint', 'mono'];

function rotatePoint([x, y, z], rot) {
  let [px, py, pz] = [x, y, z];
  if (!rot) return [px, py, pz];
  const [ox, oy, oz] = rot.origin || [0, 0, 0];
  px -= ox; py -= oy; pz -= oz;
  if (rot.rx) {
    const c = Math.cos(rot.rx), s = Math.sin(rot.rx);
    [py, pz] = [py * c - pz * s, py * s + pz * c];
  }
  if (rot.ry) {
    const c = Math.cos(rot.ry), s = Math.sin(rot.ry);
    [px, pz] = [px * c + pz * s, -px * s + pz * c];
  }
  if (rot.rz) {
    const c = Math.cos(rot.rz), s = Math.sin(rot.rz);
    [px, py] = [px * c - py * s, px * s + py * c];
  }
  return [px + ox, py + oy, pz + oz];
}

function project3D([x, y, z]) {
  const cy = Math.cos(cam.yaw), sy = Math.sin(cam.yaw);
  const cp = Math.cos(cam.pitch), sp = Math.sin(cam.pitch);
  // Yaw around Z-up
  const rx = x * cy - y * sy;
  const ry = x * sy + y * cy;
  const rz = z;
  // Screen X, Screen Y (SVG Y-down), and Depth (camera space depth)
  const sx = rx * cam.zoom;
  const syScreen = (ry * sp - rz * cp) * cam.zoom;
  const depth = ry * cp + rz * sp;
  return { x: sx, y: syScreen, depth };
}

function shadeHex(hex, factor) {
  if (!hex || hex === 'none' || !hex.startsWith('#')) return hex;
  const num = parseInt(hex.slice(1), 16);
  const r = Math.min(255, Math.max(0, Math.round(((num >> 16) & 255) * factor)));
  const g = Math.min(255, Math.max(0, Math.round(((num >> 8) & 255) * factor)));
  const b = Math.min(255, Math.max(0, Math.round((num & 255) * factor)));
  return `#${((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1)}`;
}

/* ============================================================================
   2. 3D PRIMITIVE BUILDERS (Box, Cylinder, Sphere, Line, Custom Face)
   ============================================================================ */
function addBox(scene, opts) {
  const {
    x = 0, y = 0, z = 0, w = 20, d = 20, h = 20,
    color = '#e8dfd1', layer = 'shell', onClick = null,
    rot = null, active = false, opacity = 1
  } = opts;
  const hw = w / 2, hd = d / 2, hh = h / 2;
  const v = [
    [x - hw, y - hd, z - hh], [x + hw, y - hd, z - hh],
    [x + hw, y + hd, z - hh], [x - hw, y + hd, z - hh],
    [x - hw, y - hd, z + hh], [x + hw, y - hd, z + hh],
    [x + hw, y + hd, z + hh], [x - hw, y + hd, z + hh]
  ].map(pt => rotatePoint(pt, rot));

  const faces = [
    { idx: [4, 5, 6, 7], shade: 1.06 }, // Top (+Z)
    { idx: [0, 1, 2, 3], shade: 0.78 }, // Bottom (-Z)
    { idx: [1, 2, 6, 5], shade: 0.96 }, // +X
    { idx: [3, 0, 4, 7], shade: 0.86 }, // -X
    { idx: [2, 3, 7, 6], shade: 1.00 }, // +Y
    { idx: [0, 1, 5, 4], shade: 0.88 }  // -Y
  ];

  faces.forEach(f => {
    const pts3D = f.idx.map(i => v[i]);
    addFaceDetail(scene, {
      points3D: pts3D,
      color: shadeHex(color, f.shade),
      layer, onClick, active, opacity
    });
  });
}

function addCylinder(scene, opts) {
  const {
    x = 0, y = 0, z = 0, r = 20, h = 30, axis = 'z',
    segments = 16, color = '#d8cfc0', layer = 'shell',
    onClick = null, rot = null, active = false, opacity = 1
  } = opts;
  const hh = h / 2;
  const topRing = [], botRing = [];

  for (let i = 0; i < segments; i++) {
    const a = (i / segments) * Math.PI * 2;
    const ca = Math.cos(a) * r, sa = Math.sin(a) * r;
    let pTop, pBot;
    if (axis === 'z') {
      pTop = [x + ca, y + sa, z + hh];
      pBot = [x + ca, y + sa, z - hh];
    } else if (axis === 'y') {
      pTop = [x + ca, y + hh, z + sa];
      pBot = [x + ca, y - hh, z + sa];
    } else {
      pTop = [x + hh, y + ca, z + sa];
      pBot = [x - hh, y + ca, z + sa];
    }
    topRing.push(rotatePoint(pTop, rot));
    botRing.push(rotatePoint(pBot, rot));
  }

  for (let i = 0; i < segments; i++) {
    const ni = (i + 1) % segments;
    const shade = 0.86 + 0.16 * Math.cos((i / segments) * Math.PI * 2);
    addFaceDetail(scene, {
      points3D: [botRing[i], botRing[ni], topRing[ni], topRing[i]],
      color: shadeHex(color, shade),
      layer, onClick, active, opacity
    });
  }
  addFaceDetail(scene, {
    points3D: topRing,
    color: shadeHex(color, 1.06),
    layer, onClick, active, opacity
  });
  addFaceDetail(scene, {
    points3D: botRing,
    color: shadeHex(color, 0.8),
    layer, onClick, active, opacity
  });
}

function addFaceDetail(scene, opts) {
  const {
    points3D, color = '#ffffff', stroke = 'var(--ink)',
    layer = 'inner', onClick = null, active = false, opacity = 1
  } = opts;
  const proj = points3D.map(project3D);
  const depth = proj.reduce((sum, p) => sum + p.depth, 0) / proj.length;
  scene.items.push({
    type: 'poly', proj, depth, color, stroke, layer, onClick, active, opacity
  });
}

function addSphere(scene, opts) {
  const {
    x = 0, y = 0, z = 0, r = 8, color = '#e05a3a',
    layer = 'inner', onClick = null, active = false, opacity = 1
  } = opts;
  const p = project3D([x, y, z]);
  scene.items.push({
    type: 'circle', x: p.x, y: p.y, r: r * cam.zoom,
    depth: p.depth, color, layer, onClick, active, opacity
  });
}

function addLine3D(scene, opts) {
  const {
    p1, p2, color = 'var(--ink)', width = 2,
    dashed = false, layer = 'inner', active = false
  } = opts;
  const a = project3D(p1), b = project3D(p2);
  scene.items.push({
    type: 'line', x1: a.x, y1: a.y, x2: b.x, y2: b.y,
    depth: (a.depth + b.depth) / 2, color, width, dashed, layer, active
  });
}

/* ============================================================================
   3. OBJECT CONFIGURATION, STATE MACHINE & SCENE DEFINITION
   (Replace this section for the specific object requested by the user)
   ============================================================================ */
// ... [Define OBJECT_CONFIG, state, ACTIONS, LABELS, and buildScene(scene, state, t) here] ...
```

---

## 4. Rendering & Interaction Engine (Always Include at Bottom of Script)

Always include the SVG painter's-order renderer, drag-to-orbit handler, keyboard shortcut listener, and `requestAnimationFrame` loop so every generated HTML file works out of the box:

```javascript
/* ============================================================================
   4. PAINTER'S-ORDER SVG RENDERER & UI WIRING
   ============================================================================ */
const sceneGroup = document.getElementById('sceneGroup');
const labelsGroup = document.getElementById('labelsGroup');
const labelsStrip = document.getElementById('labelsStrip');
const readoutHud = document.getElementById('readoutHud');
const statusText = document.getElementById('statusText');

function renderFrame(ts) {
  const t = ts * 0.001;
  cam.yaw += (cam.targetYaw - cam.yaw) * 0.18;
  updateSimulation(t);

  const scene = { items: [] };
  buildScene(scene, state, t);

  // Painter's algorithm: sort back-to-front by camera depth
  scene.items.sort((a, b) => a.depth - b.depth);

  let svgHTML = '';
  for (let i = 0; i < scene.items.length; i++) {
    const it = scene.items[i];
    const isShellCut = cam.seeThrough && it.layer === 'shell';
    const fillOp = isShellCut ? 0.14 : (it.opacity ?? 1);
    const strokeOp = isShellCut ? 0.35 : 1;
    const strokeColor = it.active ? 'var(--accent)' : (it.stroke || 'var(--ink)');
    const strokeW = it.active ? 2.4 : 1.65;
    const dash = isShellCut ? 'stroke-dasharray="4 3"' : (it.dashed ? 'stroke-dasharray="4 4"' : '');
    const cls = it.onClick ? `class="clickable-part" data-action="${it.onClick}"` : '';

    if (it.type === 'poly') {
      const pts = it.proj.map(p => `${p.x.toFixed(1)},${p.y.toFixed(1)}`).join(' ');
      svgHTML += `<polygon points="${pts}" fill="${it.color}" fill-opacity="${fillOp}" stroke="${strokeColor}" stroke-width="${strokeW}" stroke-opacity="${strokeOp}" stroke-linejoin="round" ${dash} ${cls} />`;
    } else if (it.type === 'circle') {
      svgHTML += `<circle cx="${it.x.toFixed(1)}" cy="${it.y.toFixed(1)}" r="${it.r.toFixed(1)}" fill="${it.color}" fill-opacity="${fillOp}" stroke="${strokeColor}" stroke-width="${strokeW}" stroke-opacity="${strokeOp}" ${cls} />`;
    } else if (it.type === 'line') {
      svgHTML += `<line x1="${it.x1.toFixed(1)}" y1="${it.y1.toFixed(1)}" x2="${it.x2.toFixed(1)}" y2="${it.y2.toFixed(1)}" stroke="${it.active ? 'var(--accent)' : it.color}" stroke-width="${it.width}" stroke-linecap="round" ${dash} />`;
    }
  }
  sceneGroup.innerHTML = svgHTML;

  // Render 3D Projected Labels
  if (cam.showLabels) {
    let lblHTML = '';
    LABELS.forEach((lbl, idx) => {
      const p = project3D(lbl.pos(state, t));
      const active = lbl.isActive(state);
      const bg = active ? 'var(--accent)' : 'var(--ink)';
      lblHTML += `
        <g transform="translate(${p.x.toFixed(1)}, ${p.y.toFixed(1)})">
          <circle r="11" fill="${bg}" stroke="#fffdf8" stroke-width="2" />
          <text y="4" text-anchor="middle" fill="#ffffff" font-family="var(--font-mono)" font-size="11" font-weight="700">${idx + 1}</text>
        </g>`;
    });
    labelsGroup.innerHTML = lblHTML;
  } else {
    labelsGroup.innerHTML = '';
  }

  syncUI();
  requestAnimationFrame(renderFrame);
}
```