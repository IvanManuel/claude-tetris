# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A classic Tetris implementation in vanilla JavaScript using HTML5 Canvas — no frameworks, no dependencies, no build step. Three files: `index.html` (DOM/canvas structure), `style.css` (dark/retro styling), `game.js` (all game logic, ~300 lines).

## Running

There is no build or install step. Open `index.html` directly, or serve it locally:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

There are no tests, linter, or package.json in this repo.

## Architecture (`game.js`)

Everything lives in one file with module-level mutable state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `dropInterval`, etc.) set up in `init()`.

- **Board model**: `ROWS × COLS` matrix; each cell is `0` (empty) or a color index 1–7 identifying the locked piece.
- **Pieces**: defined as square matrices in `PIECES` (index 0 unused, 1–7 = I/O/T/S/Z/J/L). Rotation is `rotateCW` — transpose + row reversal, no piece-specific rotation tables.
- **Collision** (`collide`): checks a shape at an offset against board bounds and existing locked cells.
- **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` and keeps the first that doesn't collide.
- **Game loop** (`loop`): driven by `requestAnimationFrame`; accumulates delta time in `dropAccum` and advances the piece one row once it exceeds `dropInterval`.
- **Locking** (`lockPiece` → `merge` + `clearLines` + `spawn`): merges the current piece into `board`, clears completed rows (shifting from the bottom up, reinserting empty rows at the top), then spawns the next piece.
- **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/cell dropped, soft drop adds 1 pt/row.
- **Leveling/speed**: level = `floor(lines / 10) + 1`; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
- **Ghost piece** (`ghostY`): projects the current piece straight down to its landing row, drawn at `globalAlpha = 0.2`.
- **Game over**: triggered in `spawn()` when a freshly spawned piece immediately collides.

All keyboard input is handled by a single `keydown` listener (arrows to move/rotate/soft-drop, Space for hard drop, P to pause); the restart button re-invokes `init()`.

## Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK` (cell size in px), `COLORS`, `LINE_SCORES`, initial `dropInterval`. If `COLS`, `ROWS`, or `BLOCK` change, update the `<canvas id="board">` `width`/`height` in `index.html` to match (`COLS × BLOCK` and `ROWS × BLOCK`).
