# [Proposal][HUD-03] Turn the existing Chapters overlay into the first inline expansion

**Priority:** P1. **Depends on:** HUD-02.

## Current behavior
`appkit/chapters_panel.rs` already supplies 288×162 16:9 still cards, chapter title/time, left-right focus, active-chapter selection using playhead, horizontal scroll, pointer stops, and `on_ok` seek. It reads `metadata.playing_chapters()` from the playing leaf and therefore does not confuse a show detail page with the playing episode. However its `CH_TOP=684` and modal appear semantics assume the current one-off overlay, and `overlay.rs` owns its single focus group.

## Proposed change
Reuse `ChaptersState` / `ChaptersPart` as a **persistent in-player chapter rail** exposed on the first Down (not a separate takeover card). Introduce a layout input / shared measured slot to replace the fixed overlay-only vertical coordinate. The timeline, centered transport and right cluster remain visible above; row focus and horizontal scrolling live below. Ensure artwork uses the same image cache and playing-item server scope. Render graceful placeholders for missing artwork; keep chapter labels and timestamps legible at 10 ft.

The entry chapter should be the one at or preceding the current position, **not always chapter 0**; subsequent Up/Down should preserve selection if the item is unchanged. OK seeks once to the selected start; the expected post-seek expansion/collapse policy should be tested. Do not fetch chapters twice or read `metadata.current()` for the list.

## Acceptance
- Down selects the correct current chapter with focus ring, not a glowing halo; Left/Right scroll with clamped bounds and no offscreen focus.
- On-screen rectangles and hit/hover targets agree during and after the vertical transition, including LG pointer movement/clicks.
- Empty/unavailable chapters never leave a blank row; a movie with no chapters continues Down directly to Info.
- Long chapter names elide/wrap predictably; missing artwork, very many chapters and very short/long durations remain usable.
- Actual chapter OK performs one seek using the chapter's time and correct playback state; do not activate a chapter when the rail is not visible.
- Add host tests for geometry/focus/scroll/selection and replay/simulator screenshots for chapter rail state.
