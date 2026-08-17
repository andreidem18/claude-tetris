# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

**Running the game:** This is a vanilla JavaScript project with no build process. Start any HTTP server and open `index.html`:

```bash
# Python 3
python3 -m http.server 8000

# Node.js
npx serve .

# Then open http://localhost:8000 in your browser
```

Direct file opening (`open index.html`) works but may have CORS issues with some browsers.

## Project Overview

A complete Tetris implementation using:
- **HTML5 Canvas** for rendering
- **Vanilla JavaScript (ES6+)** for game logic
- **CSS3** for styling (dark theme, flexbox layout)
- **No dependencies** — ~305 lines of game.js, no build tools needed

## Architecture & Key Concepts

### Board Model
- `board`: 2D array (20×10) where each cell holds a color index (0 = empty, 1–7 = tetromino colors)
- Board coordinates: top-left is (0,0); Y increases downward

### Piece System
- `current`: actively falling piece (properties: `type`, `shape`, `x`, `y`)
- `next`: queued piece shown in preview
- `shape`: 4×4 matrix with color indices; `0` marks empty cells within the bounding box
- Pieces defined statically in `PIECES[]` array (I, O, T, S, Z, J, L)

### Game Loop (`loop` function)
```
requestAnimationFrame(loop)
  → accumulate elapsed time (dt)
  → if dt ≥ dropInterval: drop piece or lock it
  → draw() to canvas
  → reschedule next frame
```

Driven by `dropInterval` (milliseconds), which decreases as level increases: `max(100, 1000 - (level - 1) × 90)`.

### Collision Detection (`collide` function)
Checks if a piece's shape at position (x, y) overlaps the board boundary or existing blocks. Used for:
- Movement validation (left/right)
- Rotation validation
- Hard drop calculation
- Locking piece when it hits bottom

### Wall Kicks (`tryRotate` function)
When rotation collides, automatically nudges the piece ±1, ±2 columns horizontally before giving up. Enables realistic rotation near walls.

### Ghost Piece (`ghostY` function)
Projects where the current piece would land by repeatedly testing collisions downward. Rendered at 20% opacity for player guidance.

### Scoring
- **Line clears**: `LINE_SCORES[linesCleared] × currentLevel`
  - 1 line: 100 pts | 2 lines: 300 pts | 3 lines: 500 pts | 4 lines (Tetris): 800 pts
- **Hard drop**: 2 points per cell fallen
- **Soft drop**: 1 point per row descended

### Level Progression
- Level increments every 10 cleared lines
- Drop speed increases (interval gets shorter)
- Displayed in HUD; affects score multiplier

## File Structure

| File | Purpose |
|------|---------|
| `index.html` | DOM structure, canvas elements, overlay for pause/game-over |
| `style.css` | Dark theme styling, flexbox layout, overlays with blur effect |
| `game.js` | All game logic: board, pieces, collision, rendering, input handling |
| `README.md` | User-facing documentation in Spanish |

## Common Customization Points

These constants at the top of `game.js` control core mechanics:

| Constant | Default | Effect |
|----------|---------|--------|
| `COLS` | `10` | Board width in cells |
| `ROWS` | `20` | Board height in cells |
| `BLOCK` | `30` | Pixel size of each cell |
| `COLORS` | 7 hex codes | Piece colors (index 1–7) |
| `LINE_SCORES` | `[0,100,300,500,800]` | Points per line clears |
| `dropInterval` | `1000` | Initial drop speed (ms) |

⚠️ **Important**: If you change `COLS`, `ROWS`, or `BLOCK`, update `<canvas width>` and `height` in `index.html` to match:
- `width = COLS × BLOCK`
- `height = ROWS × BLOCK`

## Key Functions at a Glance

| Function | Purpose |
|----------|---------|
| `createBoard()` | Initialize empty board matrix |
| `randomPiece()` | Generate a new random tetromino |
| `collide(shape, x, y)` | Test if shape overlaps board/blocks |
| `rotateCW(shape)` | Rotate shape 90° clockwise (transpose + reverse) |
| `tryRotate()` | Rotate with wall kick fallback |
| `merge()` | Bake current piece into board |
| `clearLines()` | Remove full rows, update score/level |
| `ghostY()` | Calculate landing row for ghost piece |
| `softDrop() / hardDrop()` | Fast descent mechanics |
| `lockPiece()` | Merge, clear lines, spawn next |
| `spawn()` | Move `next` → `current`, generate new `next` |
| `draw()` | Render grid, board blocks, ghost, current piece |
| `drawNext()` | Render preview canvas |
| `loop(ts)` | Main game loop via requestAnimationFrame |
| `init()` | Reset game state and start |

## Input Handling

Events are bound via `document.addEventListener('keydown', ...)`:
- **Arrow Left/Right**: Move piece (`current.x ±= 1`)
- **Arrow Down**: Soft drop (descend 1 row, +1 point)
- **Arrow Up / X**: Rotate clockwise with wall kicks
- **Space**: Hard drop (instant descent to bottom, +2 pts/row)
- **P**: Toggle pause (shows overlay, cancels animation frame)

Pause/game-over states prevent input except P key.

## Rendering Pipeline

1. **Grid** (`drawGrid`): Light lines at 0.5px separating cells
2. **Board** (`draw` main loop): Iterate cells, call `drawBlock` for non-empty blocks
3. **Ghost piece** (`draw`): Draw shape at `ghostY()` with `globalAlpha = 0.2`
4. **Current piece** (`draw`): Draw shape at current (x, y) at full opacity
5. **Preview** (`drawNext`): Render `next.shape` on separate canvas, centered in 4×4 grid

Each block gets a subtle white highlight at its top edge (1px, 12% opacity) for depth.

## State Management

Global variables hold game state:
- `board`: the 20×10 matrix
- `current`, `next`: piece objects
- `score`, `lines`, `level`: HUD values
- `paused`, `gameOver`: state flags
- `dropAccum`, `dropInterval`, `lastTime`: loop timing
- `animId`: requestAnimationFrame ID (used to cancel on pause/end)

Game over triggers when a new piece cannot spawn (collides at origin), not on board overflow.

## Rendering Optimization Notes

- Canvas is cleared and fully redrawn each frame (no dirty rectangles)
- No off-screen canvas or layer separation
- Ghost piece alpha blending happens per-block, not per-frame
- Rendering cost scales with `O(ROWS × COLS)` per frame

Performance is solid at 60 FPS on modern browsers even without optimization.

## Testing & Debugging

To test locally:
1. Start HTTP server: `python3 -m http.server 8000`
2. Open `http://localhost:8000`
3. Interact: test piece movement, rotation near walls, line clears, pause, game over

To debug in browser DevTools:
- Set breakpoints in `game.js`
- Inspect `board` array to see board state
- Check `current`, `next`, `score`, `level` in console
- Call `init()` to reset mid-game

## Design Notes

- **Arcade aesthetic**: Dark background (#0f0f17), cyan/yellow/purple piece colors, monospace score font
- **Responsive layout**: Flexbox centers game container; scales well on desktop but not mobile-optimized
- **Locale formatting**: `score.toLocaleString()` adds thousands separators
- **ES6 idioms**: Arrow functions, destructuring, `Array.from()`, template literals throughout
