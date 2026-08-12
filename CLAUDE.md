# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Vanilla-JS Tetris on HTML5 Canvas. No build, no bundler, no `package.json`, no dependencies, no test suite. Three files: `index.html`, `style.css`, `game.js` (~300 lines, all logic).

## Running

```bash
xdg-open index.html          # direct open works
python3 -m http.server 8000  # or any static server
```

There is nothing to build, lint, or test. Verification is manual in a browser.

User-facing strings (overlay text, README, HTML `lang="es"`) are Spanish; keep new UI text Spanish. Code identifiers and comments are English.

## Architecture (`game.js`)

Single global scope, `'use strict'`, no modules. Loaded via plain `<script>` at end of `<body>`, so DOM lookups at top level are safe.

- **State**: module-level `let board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId`. `init()` resets all of it and is also the restart handler — any new state variable must be reset there or it leaks across games.
- **Board**: `ROWS × COLS` array of ints; `0` = empty, `1..7` = piece type, which doubles as the index into `COLORS` and `PIECES` (both arrays start with a `null` placeholder so indices line up). Keep that alignment when adding pieces.
- **Pieces**: square matrices; rotation is a fresh transposed array from `rotateCW`, not in-place. `tryRotate` applies kick offsets `[0,-1,1,-2,2]` on x only (not SRS).
- **Collision**: `collide(shape, ox, oy)` is the single gate for every movement — left/right, gravity, rotation, ghost projection, and the spawn game-over check. Row `ny < 0` is allowed (piece above the board) but out-of-bounds x and `ny >= ROWS` are not.
- **Loop**: `requestAnimationFrame` accumulator in `loop()`; when `dropAccum >= dropInterval` the piece drops one row or `lockPiece()` runs (merge → clearLines → spawn). Pause cancels the frame and restarts the loop with a reset `lastTime`; note `dropAccum` is set to `0` rather than decremented by `dropInterval`, so the drop cadence is frame-quantized.
- **Rendering**: full clear + redraw each frame — grid, board, ghost (`globalAlpha 0.2`), current piece. `drawBlock` is shared between the board and the NEXT preview canvas via a `size` argument.
- **Difficulty**: `level = floor(lines/10) + 1`, `dropInterval = max(100, 1000 - (level-1)*90)`, both recomputed only inside `clearLines`.
- **Input**: one `keydown` listener switching on `e.code`; `KeyP` is handled before the paused/gameOver guard. `updateHUD()` is called at the end of the handler.

## Geometry coupling

`COLS`, `ROWS`, `BLOCK` in `game.js` must match the `<canvas id="board">` `width`/`height` in `index.html` (`COLS*BLOCK` × `ROWS*BLOCK`, currently 300×600). The preview uses its own `NB = 30` inside `drawNext` against the 120×120 `#next-canvas` (a 4×4 cell area). Changing any of these requires editing both files.

## DOM contract

`game.js` grabs these IDs at load and does not null-check them: `board`, `next-canvas`, `score`, `lines`, `level`, `overlay`, `overlay-title`, `overlay-score`, `restart-btn`. Renaming an ID in `index.html` breaks the game at startup.
