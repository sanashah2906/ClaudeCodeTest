# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Workflow

After completing any meaningful unit of work — a new feature, a bug fix, a new file, or a significant edit — commit and push to GitHub immediately. Never leave work uncommitted at the end of a session.

Commit message format: short imperative summary on the first line (e.g. `Add enemy collision logic`, `Fix player health reset on new game`). Push after every commit:

```bash
git add <specific files>
git commit -m "your message"
git push
```

Always add files by name, not `git add .`, to avoid accidentally staging unintended files.

## Running the Games

Both games are static HTML/JS — open directly in a browser, no server needed:

```bash
open tictactoe.html
open shooter/index.html
```

No build step, no package manager, no dependencies.

## Repository Structure

```
tictactoe.html          # Self-contained 2-player Tic Tac Toe (HTML + CSS + JS in one file)
CLAUDE.md               # This file
shooter/
  index.html            # Entry point — loads all JS in dependency order via <script> tags
  js/
    utils.js            # Math helpers and circle-collision detection (load first)
    input.js            # INPUT object + initInput(canvas) — keyboard & mouse polling state
    animation.js        # ANIM_CONFIG + updateAnimation(entity, dt) — frame-based animation
    renderer.js         # All Canvas drawing: characters, pixel font, particles, buttons
    entities.js         # Player, Enemy (Grunt/Charger/Tank), Bullet classes
    levels.js           # LEVELS array, wave spawner, procedural level generation (level 4+)
    hud.js              # HUD overlay, menu/game-over/level-complete screens
    main.js             # GAME object, requestAnimationFrame loop, setState() state machine
```

## Tic Tac Toe Architecture

Single self-contained file (`tictactoe.html`) — HTML, CSS, and JS all inline. No dependencies.

**State**: Three module-level variables drive the entire game — `board` (9-element array of `''|'X'|'O'`), `current` (active player), `over` (boolean lock). The `scores` object (`{X, O, Draw}`) persists across rounds and is never reset until page reload.

**Win detection** (`checkWin`): Iterates the 8 hardcoded winning triplets (`WINS`) and returns the matching index triple, or `null`. Called after every move; the returned indices are used to add the `.win` CSS class for the highlight glow.

**Cell rendering**: Cells are plain `<div>` elements identified by `data-i` attributes (0–8). On click, `board[i]` is set, `textContent` is written directly, and CSS classes (`.x`/`.o`/`.taken`/`.win`) drive all visual state — no canvas, no JS drawing.

**`init()`**: Resets `board`, `current`, `over`, strips all classes from cells, and clears `textContent`. Does **not** reset `scores`.

**Git history**:
- `0da3fb4` — Initial commit: "Add Tic Tac Toe web game" (Apr 25 2026) — 2-player browser game with score tracking, win detection, and retro dark theme.
- Repo: https://github.com/sanashah2906/ClaudeCodeTest

## Shooter Game Architecture

**No modules, no bundler.** All globals. Scripts must be loaded in dependency order in `index.html`.

**State machine** (`main.js`): `MENU → PLAYING → LEVEL_COMPLETE → PLAYING → GAME_OVER`. Only `setState(newState)` may change `GAME.state` — it handles cleanup and setup synchronously.

**Entity lifecycle**: Entities set `alive = false` to die; they are filtered out at the top of each update tick with `.filter(e => e.alive)`. Never splice arrays during iteration.

**Rendering**: All art is procedural Canvas 2D — `fillRect`/`arc` only, no images. Every draw function does `ctx.save()` → `ctx.translate(entity.x, entity.y)` → `ctx.rotate(angle)` → draw → `ctx.restore()`. `drawPixelText()` uses a 5×7 bitmap font table in `renderer.js` (`FONT_DATA`).

**Input**: Poll `INPUT.keys`, `INPUT.mouseX/Y`, `INPUT.mouseDown` each frame. Call `clearFrameInput()` at the end of each tick to reset one-shot flags like `INPUT.mouseClicked`.

**Animation**: Each entity has `entityType`, `animState`, `animFrame`, `animTimer`. Call `updateAnimation(entity, dt)` each frame; draw functions branch on `animFrame` to vary visuals.

**dt cap**: Always cap delta time to 100ms (`Math.min(dt, 100)`) to prevent position jumps on tab re-focus.
