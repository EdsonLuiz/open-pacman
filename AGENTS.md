# AGENTS.md

## Project

Vanilla-JS Pac-Man clone (`src/`). English docs/README. No build step, no package.json,
no lint, no tests, no typecheck — it is a static page.

## Running

Open `src/index.html` directly in a browser. There is no dev server, build, or test
script.

## Architecture (read first — it is unusual)

The four JS files do **not** use ES modules. They communicate through globals set on
`window`. Script load order in `index.html` is therefore critical and must not be
rearranged:

1. `js/maze.js` — maze grid (28x31) + constants. Exposes `window.MAZE`,
   `TUNNEL_ROW`, `PACMAN_START`, `GHOST_STARTS`.
2. `js/game.js` — state + rules (`createGame`, `update`, `DIRS`). Depends on maze
   globals. Exposes `window.createGame`, `window.update`, `window.DIRS`.
3. `js/render.js` — canvas drawing (`draw`). Exposes `window.draw`.
4. `js/main.js` — entry point: game loop, keyboard, overlay. Uses
   `createGame`/`update`/`draw` globals from the files above.

`maze.js` defines the pristine grid; `createGame` copies it into `game.grid` so dots
can be eaten without mutating `MAZE`.

Key constants: `PACMAN_SPEED = 0.125` (8 frames/cell), `GHOST_SPEED = 0.1` (10
frames/cell). Collision uses `aligned()` (0.001 tolerance) and the ghost collision
uses a 0.5-cell proximity check. The tunnel is row 14 and wraps around the side edges.

## Spec-driven development workflow

This repo uses the `/spec` and `/spec-impl` skills (see `.agents/skills/spec/`).

- Specs live in `specs/` (create the folder on first use), numbered sequentially and
  zero-padded to two digits, e.g. `01-introduce-spec.md`, `02-powerups.md`.
- Each spec header: `Status`, `Depends on`, `Date`, `Objective` (one sentence).
- Spec status values (Spanish): `Borrador` · `En revisión` · `Aprobado` ·
  `Implementado` · `Obsoleto`. Pick one set per repo and stay consistent.
- Before writing code, save the spec, then have the user mark it `Approved`.
- After approval, run `/spec-impl NN-slug` to implement (it auto-creates a git branch
  if `specs/.spec-config.yml` sets `AutoCreateBranch: true`).
- If `specs/.spec-config.yml` does not exist, create it with `AutoCreateBranch: true`.
- Specs use front matter to track their `Status`. A rule must be enforced: no model
  (including this one) is permitted to modify the `Status` field in the front matter
  of any file inside the `spec/` folder. Only a human author may change spec statuses.

## Conventions

- Comments and strings are Spanish; code identifiers are English.
- Indent with 2 spaces, spaces inside parens: `foo( bar )`.
- No generated code, no migrations, no codegen.
