# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla JS Tetris (HTML5 Canvas + CSS). No dependencies, no build step, no package manager, no tests, no linter. The README and UI text are in Spanish.

## Running

Open `index.html` directly in a browser, or serve the folder (e.g. `python3 -m http.server`) and visit it. Reload the page to see changes.

## Architecture

Three files, all logic in `game.js` (a single script with global state, loaded by `index.html`; no modules):

- `index.html` defines the DOM that `game.js` looks up by id at load time (`board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`). Renaming an id requires updating the lookups at the top of `game.js`.
- `game.js` state is a set of module-level `let` variables (`board`, `current`, `next`, `score`, ...) reset by `init()`, which is also the restart handler.
  - Board is a `ROWS×COLS` matrix of ints; `0` is empty and `1–7` index into both `COLORS` and `PIECES`. Pieces are `{type, shape, x, y}` with the shape as a square-ish matrix of the same color ints.
  - Game loop: `loop(ts)` via `requestAnimationFrame` accumulates `dropAccum` against `dropInterval` for gravity, then calls `draw()`. Pause/game-over stop it with `cancelAnimationFrame(animId)`; resuming re-calls `loop` and resets `lastTime`.
  - Piece lifecycle: `lockPiece()` → `merge()` → `clearLines()` (updates score/level/`dropInterval`) → `spawn()` (promotes `next`, ends game if the spawn collides).
  - Rotation is clockwise only (`rotateCW`) with horizontal wall kicks `[0,-1,1,-2,2]` in `tryRotate()`; there is no SRS table.
  - Scoring: `LINE_SCORES[cleared] * level`, plus 1 point per soft-drop cell and 2 per hard-drop cell. Level = `lines/10 + 1`; `dropInterval = max(100, 1000 - (level-1)*90)`.
  - Rendering uses a shared `drawBlock(context, x, y, colorIndex, size, alpha)` for the board, ghost piece (alpha 0.2), current piece and the next-piece preview canvas.
- `style.css` handles layout and the `.hidden` class toggled on the overlay.

Note: the `keydown` handler calls `updateHUD()` after every key; `KeyX` is an extra rotate binding not shown in the on-screen controls.
