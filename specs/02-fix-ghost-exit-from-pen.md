# SPEC 02 — Fix ghost exit from pen

> **Status:** Implemented
> **Depends on:** SPEC 01
> **Date:** 2026-10-13
> **Objective:** Fix ghosts getting stuck in the pen by forcing them upward through the door until clear.

## Scope

**In:**

- Add exit logic to `moveGhost` in `src/js/game.js` that detects when a released ghost is still inside the pen region.
- Force the ghost upward (direction `up`) until it reaches the door row (row 13 or higher).
- Once clear of the pen, return control to the normal behavior pattern (`decideGhost`).
- Apply this logic uniformly to all four ghost kinds.

**Out of scope:**

- Changes to the release threshold logic or release state tracking.
- Changes to ghost behavior patterns (chaser, ahead, mirror, fleer).
- Changes to collision detection or pathfinding.
- Rendering or visual feedback for the exit state.

## Data model

No new data structures. The existing `game.ghost` objects already track `released`, `x`, and `y`. We use the pen region bounds already known from `GHOST_STARTS` in `maze.js` (cells within rows 14–15, columns 13–14).

## Implementation plan

1. In `src/js/game.js` `moveGhost`, after checking if the ghost is released, add an exit check: if the ghost is inside the pen region (rows 14–15, columns 13–14), force `dir = up` and move the ghost.
2. Once the ghost's `y` coordinate is 13 or less (above the pen), stop forcing the exit direction and let `decideGhost` choose the direction normally.
3. Test manually: open `src/index.html`, watch each ghost exit the pen cleanly on its release threshold, and verify it does not get stuck.

## Acceptance criteria

- [ ] On release, each ghost moves upward through the pen without looping or getting stuck.
- [ ] Each ghost exits the pen and reaches the maze area (row 13 or above).
- [ ] Once outside the pen, the ghost follows its normal behavior pattern.
- [ ] All four ghosts (chaser, ahead, mirror, fleer) exit cleanly.
- [ ] On life loss, ghosts re-enter the pen and exit again cleanly on the next frame.
- [ ] No console errors during normal play.

## Decisions

- **Yes:** force upward direction while in the pen. Simple, deterministic, and prevents circular logic inside the cage.
- **Yes:** pen region defined by rows 14–15 and columns 13–14. Matches the existing `GHOST_STARTS` positions and is already a known constant boundary.
- **Yes:** apply the same logic to all four ghost kinds. Consistency and simplicity; no behavior-specific exit logic needed.

## Risks

None identified. The exit logic is purely a movement constraint that applies before behavior selection, so it cannot break existing pathfinding or collision logic.
