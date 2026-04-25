# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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

## Shooter Game Architecture

**No modules, no bundler.** All globals. Scripts must be loaded in dependency order in `index.html`.

**State machine** (`main.js`): `MENU → PLAYING → LEVEL_COMPLETE → PLAYING → GAME_OVER`. Only `setState(newState)` may change `GAME.state` — it handles cleanup and setup synchronously.

**Entity lifecycle**: Entities set `alive = false` to die; they are filtered out at the top of each update tick with `.filter(e => e.alive)`. Never splice arrays during iteration.

**Rendering**: All art is procedural Canvas 2D — `fillRect`/`arc` only, no images. Every draw function does `ctx.save()` → `ctx.translate(entity.x, entity.y)` → `ctx.rotate(angle)` → draw → `ctx.restore()`. `drawPixelText()` uses a 5×7 bitmap font table in `renderer.js` (`FONT_DATA`).

**Input**: Poll `INPUT.keys`, `INPUT.mouseX/Y`, `INPUT.mouseDown` each frame. Call `clearFrameInput()` at the end of each tick to reset one-shot flags like `INPUT.mouseClicked`.

**Animation**: Each entity has `entityType`, `animState`, `animFrame`, `animTimer`. Call `updateAnimation(entity, dt)` each frame; draw functions branch on `animFrame` to vary visuals.

**dt cap**: Always cap delta time to 100ms (`Math.min(dt, 100)`) to prevent position jumps on tab re-focus.
