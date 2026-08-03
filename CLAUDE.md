# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A classic Tetris implementation in vanilla JavaScript, HTML5 Canvas, and CSS. No dependencies, no build step, no `package.json`. The entire game logic lives in a single file, `game.js` (~300 lines).

## Running the game

There is no build/test/lint tooling. To run:

```bash
start index.html       # Windows: open directly in the browser
# or serve statically, e.g.
npx serve .
python3 -m http.server 8000
```

Then open the served URL, or just open `index.html` directly — both work since there's no bundler or module system (`game.js` is loaded as a plain script).

## Architecture

Three files cooperate directly, no framework:

- **`index.html`** — DOM structure: the main `<canvas id="board">` (300×600, i.e. `COLS × BLOCK` by `ROWS × BLOCK`), a side panel (`#score`, `#lines`, `#level`), a `<canvas id="next-canvas">` for the next-piece preview, and an `#overlay` used for both PAUSE and GAME OVER states.
- **`style.css`** — dark/retro arcade styling only; no layout logic depends on it.
- **`game.js`** — all game logic, structured as top-level state variables + functions (no classes, no modules):
  - **Board model**: `board` is a `ROWS × COLS` matrix; each cell is `0` (empty) or a color index `1–7` identifying which piece locked there.
  - **Pieces**: `PIECES` are defined as square matrices; `rotateCW` rotates via transpose + row reversal.
  - **Collision**: `collide(shape, ox, oy)` checks board bounds and overlap with locked cells.
  - **Wall kicks**: `tryRotate()` attempts the rotated shape at offsets `[0, -1, 1, -2, 2]`, keeping the first that doesn't collide.
  - **Game loop**: `loop(ts)`, driven by `requestAnimationFrame`, accumulates elapsed time in `dropAccum` and advances the piece one row once `dropInterval` is exceeded.
  - **Line clearing**: `clearLines()` scans bottom-to-top, splicing out full rows and unshifting empty ones at the top.
  - **Scoring**: `LINE_SCORES = [0, 100, 300, 500, 800]` multiplied by `level`; hard drop adds 2 pts/row dropped, soft drop adds 1 pt/row.
  - **Leveling**: level increases every 10 lines; `dropInterval = max(100, 1000 - (level - 1) * 90)` ms.
  - **Ghost piece**: `ghostY()` projects the current piece straight down to its landing row; drawn at `globalAlpha = 0.2`.

### Control flow

```
init() → createBoard() → next = randomPiece() → spawn() → requestAnimationFrame(loop)

loop(ts): accumulate dt → drop piece or lockPiece() when dropInterval exceeded → draw() → re-schedule

keydown: move / rotate / soft-drop / hard-drop / pause (P)
```

`spawn()` triggers `endGame()` (Game Over overlay) if the newly spawned piece immediately collides.

### Tunable constants (top of `game.js`)

`COLS`, `ROWS`, `BLOCK`, `COLORS`, `LINE_SCORES`, `dropInterval`. If `COLS`/`ROWS`/`BLOCK` change, update the `width`/`height` of `<canvas id="board">` in `index.html` to match (`COLS × BLOCK` by `ROWS × BLOCK`).

## Notes

- README.md is in Spanish and documents the same architecture in more detail — check it for user-facing behavior descriptions if needed.
- No test suite, linter, or CI exists in this repo.
