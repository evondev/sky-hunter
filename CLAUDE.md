# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

**Sky Hunter AI** — a single-file, co-op hand-tracking arcade shmup. The entire game (HTML, CSS, JS) lives in one file: [index.html](index.html). There is no build step, no package manager, no dependencies installed locally — MediaPipe is loaded from a CDN at runtime.

> Note: the global `~/.claude/CLAUDE.md` describes Next.js / TypeScript feature-folder conventions. **Those do NOT apply here.** This project is intentionally a single vanilla-JS file with no framework, no modules, and no `src/` structure. Do not introduce a build system, split into files, or convert to TypeScript unless explicitly asked.

## Running

Because the game uses `getUserMedia` (webcam), it must be served over `http://localhost` or `https` — opening the file directly (`file://`) will block camera access. Serve the directory with any static server, e.g.:

```bash
python3 -m http.server 8000   # then open http://localhost:8000
```

There are no tests, no lint config, and no build. To "deploy", host `index.html` on any static host.

## Architecture

Everything is in [index.html](index.html), ordered top-to-bottom: `<style>` (theme via CSS custom props `--neon`, `--neon2`, `--gold`, `--green`), HTML overlays/HUD, two CDN `<script>` tags for MediaPipe, then one `<script>` with the whole game.

Key pieces of the game script:

- **Render target** — a single full-screen `<canvas id="game">` drawn with the 2D context. `resize()` keeps it sized to the window and applies `DPR` scaling via `ctx.setTransform`. All gameplay is hand-rolled canvas drawing (`drawShip`, `drawEnemy`, `drawBoss`, `drawDrone`); the webcam frame is drawn underneath the action in `render()` when `useCamera` is true.
- **`Sound`** — an IIFE wrapping a WebAudio `AudioContext`; synthesizes all SFX procedurally (`blip`/`noise`). Must be unlocked by a user gesture — `Sound.init()` is called from the button click handlers.
- **State machine** — `STATE = { MENU, PLAYING, OVER }` with a single `state` variable; `game` object holds all mutable world state (score, lives, level, and the entity arrays `enemies`, `pBullets`, `eBullets`, `orbs`, `particles`, `drones`, plus `boss`).
- **Players** — exactly two persistent slots (`players[0]`, `players[1]`). A slot is `active` only while a controller is mapping to it. Auto-firing is continuous (`firePlayer`); the controller only moves the ship toward a target `(tx, ty)`.
- **Control input** — two interchangeable sources both feed the same `(tx, ty)` target:
  - **Hand tracking**: MediaPipe `Hands` (max 2 hands) → `onHandResults` → `assignHand`. Palm position uses landmark **9** (middle-finger MCP), mirrored horizontally (`1 - p.x`). `handMap` assigns each detected hand label ("Left"/"Right") to the first free player slot; `pl.seen` is a freshness timer so a slot deactivates when its hand disappears.
  - **Mouse fallback**: `mouseCtl = { x, y }` drives player 0 when the camera path isn't used.
- **Main loop** — `loop(now)` via `requestAnimationFrame` computes `dt`, then calls `update(dt)` (spawning, movement, collisions, boss logic, level progression) and `render()`. Levels escalate through `startLevel` → waves → `spawnBoss`/`updateBoss`.

## Conventions in this file

- Plain ES (`"use strict"`), no modules — everything shares one global scope. New helpers go inline in the same `<script>`.
- Short math helpers already exist: `rand`, `clamp`, `dist2`, `addParticles`, `toast`. Reuse them.
- UI text and game copy are in **Vietnamese** — keep new player-facing strings in Vietnamese to match.
- Collisions use squared distance (`dist2`) to avoid `sqrt`; follow that pattern in the hot loop.

## Git

Do not commit or push without explicit user confirmation.
