# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the game

No build step — open `index.html` directly in a browser or serve with any static server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

On Windows: `start index.html`

## Architecture

Three files, no dependencies, no bundler:

- `index.html` — DOM structure: a `<canvas id="board">` (300×600 px) for the main board and `<canvas id="next-canvas">` (120×120 px) for the piece preview. An `#overlay` div handles both PAUSE and GAME OVER states via a `hidden` CSS class toggle.
- `style.css` — dark/retro aesthetic using CSS variables, flexbox layout, and `backdrop-filter` on overlays.
- `game.js` — all game logic (~305 lines, `'use strict'`, no modules).

### game.js internals

**State** lives in module-level `let` vars: `board` (ROWS×COLS matrix), `current`/`next` (piece objects `{type, shape, x, y}`), `score`, `lines`, `level`, `paused`, `gameOver`, `dropAccum`, `dropInterval`, `animId`.

**Key functions and their roles:**
- `collide(shape, ox, oy)` — bounds + overlap check; used everywhere before any move/rotate
- `tryRotate()` — applies `rotateCW` then tries wall-kick offsets `[0, -1, 1, -2, 2]`
- `lockPiece()` — merge → clearLines → spawn; triggers `endGame()` if new piece collides on spawn
- `loop(ts)` — `requestAnimationFrame` loop; accumulates `dropAccum` and calls `lockPiece` or moves piece down when `dropAccum >= dropInterval`
- `draw()` — clears canvas, draws grid, locked board blocks, ghost piece (alpha 0.2), then current piece
- `init()` — full reset; called on page load and on restart button click

**Tunable constants** at the top of `game.js`: `COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`. If `COLS`, `ROWS`, or `BLOCK` change, the canvas `width`/`height` attributes in `index.html` must be updated to match (`COLS × BLOCK` and `ROWS × BLOCK`).

**Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` × level; soft drop +1/row, hard drop +2/cell fallen.

**Speed**: `dropInterval = Math.max(100, 1000 − (level − 1) × 90)` ms; level increments every 10 lines.
