# [Proposal][HUD-02] Implement Down → Chapters → Info progressive player navigation

**Priority:** P0. **Depends on:** HUD-01.

## Current behavior
`screens/player/input.rs::HudNav` encodes three regions (scrubber, right discs, bottom Info/Chapters tabs). `PlayerScreen::key_updown` manually walks them. `key_ok` opens `OverlayKind::Info` or `Chapters` from the bottom tab. `PlayerOverlayScreen` supports one modal panel at a time and Down at the bottom of Info/Chapters currently dismisses the panel and returns to the tabs.

## Proposed change
Make a **player-owned progressive presentation** with explicit levels: **Base → Chapters → Info**. Starting with focused **Play/Pause**, one **fresh Down** animates/repositions the base HUD upward enough to reveal chapter thumbnails and places focus on the current chapter. Another Down from the chapter row reveals the full episode/movie details **below** that row. Repeated Down at the last level does nothing; Up unwinds one level, restoring semantic focus. Back collapses Info then Chapters before normal HUD hide/exit. Do not make this work by opening two stacked modal screens that each swallow the other's navigation.

Use the shared `FocusEngine` / `Focusable` groups, edges and seats for region movement. Retain `HudState` for lifetime/linger and keep animation in the shared layout/motion system. The active level must be canonical/snapshot-friendly and reconstructible after re-entry, pause/resume or changing items. If no chapters exist, first Down goes to Info; if metadata is unavailable, do not open an empty level. Define where Down from a **non-center** HUD control leads rather than catching every Down indiscriminately.

## Acceptance
- Base / first Down / second Down show three **distinct**, progressive layouts at 1920×1080 with appropriate focus.
- Up and Back reverse the path predictably; Back while a Tracks/More/Timing submenu is open retains that submenu's own Back handling first.
- Repeat presses and two key presses ingested in one frame cannot skip two levels or activate invisible elements; the HUD does not auto-hide during reading.
- A missing chapter rail bypasses directly to Info; missing metadata leaves a safe, navigable base state.
- Changes to item identity, end of playback, failure, interrupted seek or menu open cannot leave stale chapter/info focus or a stranded half-expanded panel.
- Cover key order, level transitions, re-entry, no-chapters and collapse with regression tests; replace old `updown_walks_the_huds_three_rows` expectations intentionally.
