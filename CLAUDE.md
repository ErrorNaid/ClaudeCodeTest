# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the games

No build step — open any HTML file directly in a browser:

```bash
xdg-open tictactoe.html   # Linux
xdg-open shooter.html
```

## Project structure

Each game is a single self-contained HTML file with inline CSS and JS. No dependencies, no bundler, no package manager.

## Git workflow

**Claude Code must commit and push after every meaningful unit of work** — feature additions, bug fixes, and significant edits each get their own commit. Never leave completed work uncommitted. This ensures the project history is always traceable and any change can be reverted cleanly.

```bash
git add <file>
git commit -m "short imperative summary of what and why"
git push
```

Rules:
- One logical change per commit — do not batch unrelated edits
- Commit message focuses on the *why*, not just the *what*
- Always push immediately after committing so GitHub is never behind local

## Bee Shooter architecture (`shooter.html`)

The entire game runs in one `requestAnimationFrame` loop calling `update()` then `draw()` each frame.

**State machine** — `gs` variable drives everything:
- `menu` → `playing` → `transition` → `playing` (level 2) → `victory`
- Any state → `gameover` when `lives <= 0`

**Core data structures:**
- `P` — player object (position, bullets array, invincibility timer, shoot cooldown)
- `enemies[]` / `eBullets[]` / `particles[]` — world object arrays, filtered each frame to remove dead entries
- `waveQueue[]` — slice of `WAVES[level]`; `waveIdx` advances when `enemies.length === 0` after an 85-frame delay

**Enemy types** (`type` field on each enemy object):
| type | movement | shoots | hp |
|------|----------|--------|----|
| `basic` | straight down | no | 1 |
| `zigzag` | sine-wave horizontal | yes | 1–2 |
| `fast` | straight down, fast | yes | 1 |
| `boss` | descends then cosine oscillation | 3-way spread | 25 |

**Adding a new enemy type:** add a draw branch in `drawEnemy()`, a movement/shoot branch in the enemy loop inside `update()`, and reference it in a `WAVES` entry.

**Adding a new level:** extend the `WAVES` object with a new key, then update the level-complete branch in `update()` (`if (level === 1) { level = 2 ...`) to chain to the new level instead of going to `victory`.

**Collision detection** — all via `hitTest(ax,ay,aw,ah, bx,by,bw,bh)` (AABB). All positions are center-based; subtract half-width/height when passing to `hitTest`.

## Tic Tac Toe architecture (`tictactoe.html`)

DOM-based (no canvas). `board[]` is a flat 9-element array; `WINS` is the static list of winning triples. Score state (`scores.X/O/D`) persists across rounds; `init()` resets only the board.
