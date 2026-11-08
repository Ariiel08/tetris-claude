# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Tetris clone in vanilla JavaScript + HTML5 Canvas. No dependencies, no `package.json`, no build step, no linter, no tests.

## Running

Open `index.html` directly in a browser, or serve the folder statically (preferred):

```bash
python -m http.server 8000   # then open http://localhost:8000
```

## Architecture

Three files: `index.html` (DOM + two canvases + overlay), `style.css` (dark theme), and `game.js` (all logic, loaded as a plain non-module script at the end of `<body>`).

Key points in `game.js` that need more than one function read to understand:

- **All game state lives in module-level `let` globals** (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, `animId`, …). `init()` resets all of them and is also the restart button handler, so any new state must be reset there.
- **Piece type = color index = cell value.** `PIECES[i]` shapes store the value `i` in their filled cells, `merge()` writes those values into `board`, and `COLORS[i]` is used to draw them. Index 0 is empty/null in all three arrays. Adding a piece means adding to `PIECES` and `COLORS` at the same index and changing the `* 7` in `randomPiece()`.
- **Collision is the only rule check.** `collide(shape, x, y)` covers walls, floor and locked cells (cells above the board, `y < 0`, are allowed). Movement, rotation wall kicks (`tryRotate`: offsets 0, ±1, ±2), ghost projection (`ghostY`) and game over (a freshly spawned piece colliding in `spawn()`) are all built on it.
- **The loop and pause are linked through `animId`.** `loop()` runs on `requestAnimationFrame` and adds frame `dt` to `dropAccum` until it reaches `dropInterval`. Pause and game over stop the loop with `cancelAnimationFrame(animId)`. Unpausing resets `lastTime` and calls `loop()` again.
- **Locking runs in a fixed order:** `lockPiece()` → `merge()` → `clearLines()` (which also updates score, level and `dropInterval`) → `spawn()`.
- **Board dimensions are duplicated in HTML.** `COLS × BLOCK` and `ROWS × BLOCK` must match the `<canvas id="board">` `width`/`height` in `index.html` (300×600). The next-piece preview assumes a 4×4 grid of 30px cells (120×120 canvas).

## Conventions

- User-facing text (UI labels, overlay messages, README) is in Spanish; code identifiers are in English.
- `README.md` documents the controls, the scoring formula and the tunable constants. Keep it in sync when those change.
