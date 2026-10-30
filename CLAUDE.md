# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Asteroids clone in plain HTML5 Canvas + vanilla JS. No dependencies, no bundler, no package.json, no tests, no linter. The README and in-game text are in Spanish; keep new user-facing strings and code comments in Spanish to match.

## Running

Open `index.html` directly in a browser, or serve statically with `npx serve .` (http://localhost:3000). There is no build step.

## Architecture

Everything lives in `game.js`, loaded as a classic script (not a module) by `index.html`, which supplies the 800x600 `<canvas id="canvas">`. `W`/`H` are hardcoded in `game.js` and must stay in sync with the canvas attributes.

- **Loop**: `requestAnimationFrame` → `update(dt)` then `draw()`. `dt` is in seconds and clamped to 0.05 so a background tab doesn't cause huge jumps. All speeds, cooldowns and timers are in per-second units.
- **State machine**: the global `state` is `'playing' | 'dead' | 'gameover'`, handled at the top of `update()`. `'dead'` is a 2s respawn pause (`deadTimer`) in which asteroids and particles keep moving but the ship, bullets and collisions are frozen. `'gameover'` waits for Space to call `initGame()`. Game state is module-level `let` globals (`ship, bullets, asteroids, particles, score, lives, level`), reset in `initGame()`.
- **Entities**: `Ship`, `Asteroid`, `Bullet`, `Particle` classes each with `update(dt)` and `draw()`. Removal is via a `dead` flag; `update()` rebuilds the arrays with `.filter(x => !x.dead)` each frame. The screen is toroidal: positions use the `wrap()` helper (except `Particle`, which does not wrap).
- **Asteroid sizes** are 1/2/3, and `RADII`, `SPEEDS` and `POINTS` are arrays indexed by size (index 0 is unused padding). `split()` returns two children of `size - 1`. Note that smaller asteroids score more.
- **Collision**: circle-based using `dist()` and `radius`. Ship vs asteroid uses `a.radius * 0.82` as a forgiving hitbox. The ship is immune while `ship.invincible > 0`, which also drives the blink in `Ship.draw()`.
- **Input**: `keys[code]` holds the held state. `pressed(code)` is an edge-triggered read of `justPressed[code]` that consumes the flag. Space fires only on a fresh press (with a cooldown), and holding it does not autofire. Rotation and thrust read `keys` directly.
- **Level progression**: when `asteroids.length === 0`, `nextLevel()` clears bullets and particles, resets the ship, and spawns `3 + level` large asteroids outside a `SAFE_DIST` radius of the center.

## README vs. code

The README mentions power-ups and a "shooting star" asteroid type, but neither exists in `game.js`. Treat the code as the source of truth.
