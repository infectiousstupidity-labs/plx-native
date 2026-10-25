# [Proposal][HUD-09] Decide and implement real Previous/Next episode actions before displaying them

**Priority:** P2. **Depends on:** HUD-01. Deferrable.

## Finding
The latest visual concept includes Previous and Next icons on either side of Play/Pause, but the actual player's `PlayerReq` in `screens/registry.rs` declares Transport, SeekTo, Info and the conditional Up Next action; it does **not** expose a generic Previous Episode action. Existing `appkit/up_next.rs` is an end-of-episode next-offer with different semantics. Drawing transport icons without backing behavior would repeat the current problem of misleading controls.

## Proposed change
First determine and document what `Previous` / `Next` should mean in a TV episode player (adjacent **episode**, not chapter, not ±10/30 seconds), and which Plex playlist/season context may legitimately supply that transition. Do not treat `Next Episode` as universally available merely because the current item is an episode. Reuse existing route/session transition effects if they support the intent; otherwise add one correctly owned request and tests, not a UI-only handler.

Until both actions exist and have valid targets, **do not draw them or reserve dead focus stops**. The initial centered cluster (rewind / Play-Pause / forward) is therefore a valid stage-one deliverable. When enabled, preserve Play/Pause's exact 960px center with symmetric outer controls. Differentiate previous/next from chapter seeking and the time-skip buttons through icons/feedback.

## Acceptance
- Explicit documented behavior when previous/next item is available, absent, failed, or at series/season boundaries.
- Each button has a real, tested activation that preserves current item state/cleanup and does not fight existing Up Next.
- Icon is absent/inert without an actionable item, inaccessible through focus/click and does not shift the centered Play/Pause coordinate.
- No regression to existing Next Episode countdown, Skip Intro/Credits or Back behavior.
