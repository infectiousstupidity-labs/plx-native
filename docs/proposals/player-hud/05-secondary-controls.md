# [Proposal][HUD-05] Keep Subtitles, Audio and More in the right secondary-control cluster

**Priority:** P1. **Depends on:** HUD-01.

## Current behavior
The existing right-anchored three-disc row in `appkit/player_hud.rs` is **already** ordered 0=Subtitles, 1=Audio, 2=More. `ControlSlot` conditionally replaces it with Skip or Up Next, so it cannot just be copied into the new player layout. Its click and D-pad activation use `PlayerScreen::group_of/place/record_stops` and a deferred `ArmControlRow` request. The Tracks menu is the same overlay for Subtitles and Audio tabs; the More popover and Timing capsule have separate lifecycle rules.

## Proposed change
Keep **Subtitles** on the RIGHT with Audio and More/Settings, **never in the centered transport buttons**. Preserve all three as functional secondary controls; keep the established safe right edge, suitable spacing and discoverable selected/hover states. Recompute title maximum width from actual control bounds rather than letting lengthy show names or changes in the center row cause overlap. Maintain normal captions/language controls and the single shared track popover; do not implement a second subtitle picker.

Document and preserve behavior when a Skip pill or Up Next control temporarily occupies the right slot. Decide visually how secondary buttons remain discoverable when the slot is substituted, rather than silently breaking intro/credits behavior. Remote focus must be able to move from the centered transport to this right group; pointer may activate only the controls currently painted.

## Acceptance
- Subtitles, Audio and More appear aligned **on the right**, in that order when ordinary playback chrome is displayed; none in centered transport group.
- Each opens the correct current overlay/tab; the submenu can be dismissed via Back and leaves focus on its opener.
- Exact matches for rendered rect, D-pad region and pointer stop, including the control-slot Skip/Up Next stand-in cases.
- Long English/localized titles truncate/reveal within measured free width and never overlap icons.
- Existing subtitle selection, subtitle timing/styles, audio switching, More quality controls, skip-marker and Up Next regressions remain green.
