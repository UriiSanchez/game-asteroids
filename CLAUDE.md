# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Clon del clásico arcade **Asteroids** en HTML5 Canvas puro (vanilla JS, sin frameworks ni bundler). Todo el juego vive en un único archivo `game.js`, cargado directamente por `index.html`.

## Running

No build step. Open `index.html` directly in a browser, or serve it locally:

```bash
npx serve .
```

There is no test suite, linter, or package.json in this repo.

## Architecture

Single-file game (`game.js`) organized top to bottom as: input handling → math utils → entity classes → game state → update loop → draw loop → main loop. There is no module system — everything is in one global scope, loaded as a plain `<script>` tag.

- **Input**: `keys`/`justPressed` maps populated by `keydown`/`keyup` listeners; `pressed(code)` consumes a one-shot press (used for shooting and restart), while `keys['ArrowLeft']` etc. are read directly for continuous input (rotation, thrust).
- **Entities**: `Bullet`, `Asteroid`, `Ship`, `Particle` classes, each with `update(dt)` and `draw()`. All positions wrap toroidally via `wrap(v, max)` — the play field has no edges.
- **Asteroids**: size is `1|2|3` (small/medium/large), indexing parallel arrays `RADII`, `SPEEDS`, `POINTS`. `split()` produces two asteroids of `size - 1`; size `1` splits into nothing.
- **Game state** lives in module-level `let` variables (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`, `state`) rather than a state object/class. `state` is one of `'playing' | 'dead' | 'gameover'` and gates what `update()` does each frame.
- **Loop**: `requestAnimationFrame(loop)` computes `dt` (clamped to 0.05s) and calls `update(dt)` then `draw()` each frame. Collision detection is brute-force O(n·m) (bullets × asteroids, ship × asteroids) — fine at this entity count, so don't over-engineer with spatial partitioning unless entity counts grow substantially.
- **Canvas size** is fixed at `W=800, H=600` (matches the `<canvas>` element's width/height in `index.html`), not responsive.

When adding new entity types or power-ups, follow the existing pattern: a class with `update(dt)`/`draw()`, pushed into a module-level array, filtered by a `dead` flag each frame.
