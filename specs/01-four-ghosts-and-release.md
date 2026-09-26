# SPEC 01 — Four ghosts with distinct behaviors and score-based release

> **Status:** Approved
> **Depends on:** —
> **Date:** 2026-09-25
> **Objective:** Add four ghosts with distinct classic behaviors (chaser, ahead-ambusher, mirror-ambusher, fleer) that are released from the pen progressively as the player scores.

## Scope

**In:**

- Expand the ghost count from 2 to 4 (`src/js/maze.js` `GHOST_STARTS`).
- Four behavior patterns: `chaser`, `ahead`, `mirror`, `fleer`.
- The `chaser` aggressively targets Pac-Man at every intersection.
- Pen confinement: unreleased ghosts stay idle (bobbing) inside the pen until their threshold is met.
- Release thresholds at 2.5%, 5%, 7.5% and 10% of the level's total points. At 10% all four are out.
- On losing a life, ghosts return to the pen but keep their released state (released ghosts exit again immediately).
- Render: idle bobbing for confined ghosts and one color per ghost (red, pink, cyan, orange).

**Out of scope (for future specs):**

- Frightened mode (power pellets, blue/eatable ghosts).
- A level system beyond the single maze.
- Ghost "eyes return to pen" (requires an eat mechanic).
- A* pathfinding; greedy target-selection stays.

## Data model

```js
// maze.js — GHOST_STARTS
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'chaser' }, // red
  { x: 14, y: 14, kind: 'ahead'  }, // pink
  { x: 13, y: 15, kind: 'mirror' }, // cyan
  { x: 14, y: 15, kind: 'fleer'  }, // orange
];
```

```js
// game.js — per-game state additions
const game = {
  // ...existing fields
  totalDots,        // initial dot count (for threshold fractions)
  dotsEaten: 0,
  ghosts: [ {
    x, y, dir, speed, kind,
    released: false, // confined in pen until true
  } ],
};
```

`RELEASE_FRACTIONS = [0.025, 0.05, 0.075, 0.10]` — ghost `i` is released when `dotsEaten >= RELEASE_FRACTIONS[i] * totalDots`. Since each dot is worth 10 points, dots-eaten is equivalent to score but robust if other scoring is added later.

## Implementation plan

1. `src/js/maze.js`: replace `GHOST_STARTS` with the 4 entries above (all inside pen cells 13–14 × rows 14–15, all transitable).
2. `src/js/game.js` `createGame`: add `totalDots`/`dotsEaten`, and `released: false` on each ghost; increment `dotsEaten` in `movePacman` when eating a dot.
3. `src/js/game.js`: release check in `moveGhost` — if `!g.released`, release when its fraction is reached, then return (stays in pen). While confined, the ghost does not move.
4. `src/js/game.js` `decideGhost`: implement the four patterns (chaser = existing `hunter` logic; ahead = target 4 cells ahead of Pac-Man; mirror = target `2 * pacman - chaserPos`; fleer = move away when Pac-Man within 8 cells, else random). Remove the old `hunter`/`random` branches.
5. `src/js/game.js` `resetPositions`: keep `released` flags untouched.
6. `src/js/render.js`: reorder `GHOST_COLORS` to match kinds (red, pink, cyan, orange); add idle bobbing for `!g.released` ghosts.

Manual test after each step: open `src/index.html`, confirm no console errors and ghosts still move.

## Acceptance criteria

- [ ] Four ghosts appear, colored red, pink, cyan, orange.
- [ ] At start all four are inside the pen, bobbing in place, and none leaves it.
- [ ] Eating the 7th dot (70 pts) releases the chaser, which exits the pen.
- [ ] Eating the 14th dot (140 pts) releases the ahead ghost.
- [ ] Eating the 21st dot (210 pts) releases the mirror ghost.
- [ ] Eating the 28th dot (280 pts) releases the fleer; all four roam the maze.
- [ ] The chaser reduces its Manhattan distance to Pac-Man at every intersection (aggressive pursuit).
- [ ] The fleer increases its distance to Pac-Man when within 8 cells.
- [ ] Losing a life returns all ghosts to the pen; already-released ghosts exit again on the next frames.
- [ ] No console errors during normal play.

## Decisions

- **Yes:** classic behavior set (chaser/ahead/mirror/fleer). Faithful to the original and each is observably distinct.
- **Yes:** confine unreleased ghosts in the pen with idle bobbing. Clear progression, classic feel.
- **Yes:** chaser released first (2.5%), then ahead 5%, mirror 7.5%, fleer 10%. Gives the aggressive ghost priority.
- **Yes:** thresholds computed from dots eaten. Equivalent to score today (10 pts/dot), robust to future scoring changes.
- **Yes:** released state persists across life loss. Score persists, so released ghosts re-exit immediately, matching the original.
- **No:** frightened mode. It is a self-contained mechanic that deserves its own spec.

## Risks

| Risk | Mitigation |
| --- | --- |
| A freshly released ghost may choose a direction away from the door and wander inside the pen | While the ghost is still in the pen region, force `up` until it clears the door row |
| Mirror target is odd while the chaser is still confined | Acceptable and consistent with the original; only a transient behavior |

## What is **not** in this spec

- Frightened mode (power pellets, blue/eatable ghosts).
- Level system beyond the single maze.
- Ghost eyes returning to the pen.
- A* pathfinding.

Each one of those, if it lands, goes in its own spec.