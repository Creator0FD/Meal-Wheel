# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-file static web app — a spin wheel for deciding what to eat. The entire application lives in `index.html` with no build step, no package manager, and no external runtime dependencies.

## Running it

Open `index.html` directly in a browser, or serve it with any static HTTP server:

```bash
python3 -m http.server 8080
# then visit http://localhost:8080
```

There are no tests, no linter config, and no CI pipeline.

## Architecture

Everything is in one file: `index.html` contains the HTML structure, a `<style>` block for all CSS, and a `<script>` block for all logic.

**State** (three global JS variables):
- `options[]` — the current list of meal names shown on the wheel
- `currentRotation` — the wheel's current angle in radians (starts at `-Math.PI/2`)
- `spinning` — boolean lock that prevents re-entrant spins

**Rendering** — `drawWheel()` uses the HTML5 Canvas API to draw all slices, divider lines, labels, and the center hub on each frame. It reads `options`, `currentRotation`, and the `colors` palette array.

**Spin animation** — `spinWheel()` uses `requestAnimationFrame` with a quartic ease-out over ~4.8 seconds. After the animation completes, `getWinner()` calculates the winning slice by comparing the pointer's fixed position (`3π/2`) against `currentRotation` modulo the slice arc.

**Persistence** — `localStorage` under the key `mealWheelOptions` stores the options array as JSON. Loaded on init via `loadSavedOptions()`; written on "Update Wheel" via `saveOptionsToDevice()`.

**Color palette** — `colors` is an array of 10 `[light, dark]` pairs used as radial gradient stops. Slices cycle through it via `i % colors.length`.

**External scripts loaded at runtime** (no local copies):
- Google Fonts (`Playfair Display`, `DM Sans`) via `<link>`
- Google Analytics (`gtag.js`) via async `<script>`
- Buy Me a Coffee button via `cdnjs.buymeacoffee.com`

## Key functions

| Function | Purpose |
|---|---|
| `drawWheel()` | Full canvas redraw from current state |
| `spinWheel()` | Kicks off animation, disables button, calls `getWinner()` on finish |
| `getWinner()` | Maps `currentRotation` to the options array index under the pointer |
| `getResultLabel()` | Returns a time-of-day string ("Tonight's dinner", etc.) |
| `updateOptions()` | Reads textarea, validates ≥2 items, saves, redraws |
| `clearSavedOptions()` | Wipes `localStorage`, restores `defaultOptions` |
| `resetWheel()` | Resets rotation to `-Math.PI/2` and redraws without changing options |

## CSS variables

Defined on `:root` — change these to retheme the app:
- `--accent-orange` / `--accent-amber` — primary brand colors
- `--bg-warm`, `--card`, `--ink`, `--ink-soft`, `--muted` — surface and text colors
- `--radius-card`, `--shadow-warm`, `--border` — card chrome
