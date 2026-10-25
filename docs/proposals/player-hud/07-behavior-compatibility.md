# [Proposal][HUD-07] Preserve player/menu/seek/subtitle behavior across progressive HUD states

**Priority:** P0. **Depends on:** HUD-02–05.

## Current behavior
`PlayerScreen` owns HUD lifetime, pending scrubs and Back exit-arming; `PlayerOverlayScreen` owns Tracks/More/Timing/Info/Chapters as separate surfaces. `HudPolicy` may be Normal, Lifted or Hidden. Existing tests cover repeated Back, failed playback, pending and paused seeks, pointer stops, visible-at-press snapshots, and modal playback keys. `appkit/player_hud.rs` and `screens/player/mod.rs` apply subtitle/image-subtitle lift when the HUD occupies the bottom of the picture.

## Proposed change
Write an explicit compatibility matrix for **base / chapters / info / modal** states before refactoring the view structure. Ensure player transport keys work while reading chapters/info; main D-pad Down/Up walks levels only when the corresponding focus region owns the key. Tracks, More and Timing keep their distinct input/Back behavior. A pending scrub must cancel (not seek) when leaving to chapter/info; seek previews cannot be committed by a stale OK. Back on Info collapses it, then Chapters collapses, then the normal HUD is hidden/exit-armed. The same-frame event ordering hazard must be tested rather than assuming separate input batches.

Treat video subtitles and image subtitles as high-priority content: adjust actual clearance geometry according to the **expanded HUD bounds**, not the old fixed `SUB_CEIL_Y`, avoiding both collisions and moving text unnecessarily when chrome collapses. Do not dim/dismiss chapters just because the film pauses or while an overlay remains open. Preserve marker-triggered Skip, Up Next countdown/Watch Credits, error states, buffering and resume title behavior.

## Acceptance
- Test table covers: paused/playing; absent chapters; modal over each visible level; Back repeatedly; same-batch Down→Back/OK; held/repeated Down; finger/mouse pointer motion and clicks; scrubbing and deferred commits; Skip/Up Next; failures/reload/item switch.
- No hidden element has a hit stop or is activated via deferred commit. Opening a modal at level 2 does not create two independently focused player controls.
- Subtitle/caption and bitmap positioning remains collision-free at every expanded level on LG hardware.
- Existing `screens/player/mod.rs` and `overlay_tests.rs` tests pass unless intentionally updated with equivalent stronger assertions.
- Regression must demonstrate failure on the old code (or a reproducible old-behavior trace) before fixing accidental behavior changes.
