# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Classic Tetris implemented in vanilla JavaScript (ES6+), HTML5 Canvas, and CSS. No dependencies, no build step, no `package.json`. README (`README.md`) is in Spanish and is the source of truth for gameplay/feature docs — keep it in sync when changing behavior described there.

## Running / testing

There is no build, lint, or test tooling in this repo — just static files served directly.

```bash
# Open directly
xdg-open index.html   # Linux

# Or serve locally (recommended, avoids canvas/file:// quirks)
python3 -m http.server 8000
npx serve .
```

Manual verification only: open in a browser and play. No automated tests exist.

## Architecture

Three cooperating files, all logic lives in `game.js`:

- `index.html` — DOM shell: `<canvas id="board">` (300×600) for the play field, `<canvas id="next-canvas">` for the next-piece preview, HUD spans (`score`/`lines`/`level`), and a hidden `#overlay` reused for both PAUSE and GAME OVER states.
- `style.css` — dark/retro arcade styling only.
- `game.js` — entire game state and logic, structured around a handful of concepts:
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked it there.
  - **Pieces**: `PIECES` are square matrices (color-index-filled). Rotation (`rotateCW`) is a transpose + row-reverse, not per-piece special-cased.
  - **Collision** (`collide`): checks board bounds and existing locked cells for a shape at a given offset. Used for movement, rotation, gravity, and ghost-piece projection alike.
  - **Wall kicks** (`tryRotate`): after rotating, tries offsets `[0, -1, 1, -2, 2]` columns until a non-colliding position is found, else the rotation is discarded.
  - **Game loop** (`loop`): driven by `requestAnimationFrame`, accumulates elapsed time (`dropAccum`) and advances the piece one row once `dropInterval` is exceeded; otherwise just redraws (`draw`).
  - **Locking/clearing**: `lockPiece` → `merge` (bakes current piece into `board`) → `clearLines` (bottom-up scan, splices full rows, unshifts empty ones, recalculates score/level/`dropInterval`) → `spawn` (promotes `next` to `current`, generates a new `next`; if the new piece immediately collides, calls `endGame`).
  - **Ghost piece**: `ghostY` projects `current` straight down via repeated `collide` checks; drawn at `globalAlpha = 0.2`.
  - **Scoring/leveling**: line clears score via `LINE_SCORES = [0,100,300,500,800]` × `level`; hard drop adds 2 pts/row dropped, soft drop 1 pt/row. Level = `floor(lines/10) + 1`; `dropInterval = max(100, 1000 - (level-1)*90)` ms.
  - All mutable game state (`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, timing vars) is module-level, reset in `init()` — there's no encapsulation/class, so be careful about implicit dependencies between functions that read/write these globals directly.

Tunable constants at the top of `game.js`: `COLS`, `ROWS`, `BLOCK` (must stay consistent with the `#board` canvas `width`/`height` in `index.html`, i.e. `COLS×BLOCK` and `ROWS×BLOCK`), `COLORS`, `LINE_SCORES`, initial `dropInterval`.
