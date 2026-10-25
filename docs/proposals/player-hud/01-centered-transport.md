# [Proposal][HUD-01] Add an actually centered, functional playback control group

**Priority:** P0. **Depends on:** none.

## Current behavior
`appkit/player_hud.rs::draw_hud` draws the seek bar, title, a right-anchored `ControlSlot::Discs` row (Subtitles, Audio, More) and bottom Info/Chapters tabs. It does **not** draw or register a centered playback group. `PlayerScreen::key_ok` toggles when the seek bar has focus; it is not a focused center-button press. `screens/player/input.rs::HudNav` has no transport-button cursor.

## Proposed change
Add a separate **center-anchored transport group beneath the progress bar**, with at least **rewind 10 / Play-Pause / forward 30**. The center of Play/Pause's hit rect must be screen center (960 at 1920). Use measured geometry, shared `HStack`, `TransportButton`/related shared widgets and `Focusable` stops. Do not position by eyeballing or conflate this group with the right-hand secondary controls. The design reference also showed Previous/Next; those are tracked separately as HUD-09 and must **not** be displayed until backed by correct episode actions.

Implement each centered action with real seek/toggle semantics, applying range clamping and respecting paused/playing behavior. Decide whether the skip interval follows existing settings or fixed 10/30 only after checking current transport settings and their UI contract. Preserve existing D-pad left/right scrub when the **seek bar** is focused; on the **transport row** those same keys move focus among controls rather than secretly seeking.

## Source anchors
`rust-modules/appkit/src/player_hud.rs`: `SB_Y`, `BTN_Y`, `ELEM_ROW_BASE`, `draw_hud`, `ctrl_row_hit_rect`, `ControlSlot`. `rust-modules/screens/src/player/mod.rs`: `groups`, `place`, `record_stops`, `handle_click`, `key_ok`, `key_scrub_fresh`. `rust-modules/screens/src/player/input.rs`: `HudNav`, `HudState`. `rust-modules/screens/src/registry.rs`: `PlayerReq`.

## Acceptance
- Center of Play/Pause equals center of 1920 canvas, irrespective of localized title length and the contents of the right cluster.
- Play/Pause visibly switches icon/accessible state; OK, LG remote transport keys and Magic Remote click produce one correct action, not double-toggle.
- Rewind/forward work while playing and paused and never pass a seek beyond `[0, duration]`; error/loading/no-seek sessions do not show operable phantom controls.
- All buttons have correct independent hit rectangles and engine focus identity. Button-row Left/Right navigates; scrub-focused Left/Right still previews seeking.
- Test focus ordering, hit-test/draw equality, seeking and pause transitions; do not regress `tests/player_pointer.py` or the pending-scrub tests.
