# [Proposal][HUD-06] Apply polished, restrained, no-glow playback HUD styling

**Priority:** P1. **Depends on:** HUD-01–05.

## Current behavior
`appkit/player_hud.rs::draw_playbar` explicitly renders a glow behind the focused scrubber knob; shared `ui/widgets.rs` `control_rim_spec` / `face_rimmed` allow glow on focused `ControlGround::Unkeyed` control faces. The mockups previously generated for this discussion also used an exaggerated luminous ring: **do not copy that look**. Existing dimensions were tuned for a separate right-side row and bottom Info/Chapters pills, not a centered transport plus progressive disclosure.

## Proposed change
Use one restrained premium dark visual language: a low, continuous black-to-transparent bottom scrim; legible white/off-white text; modest inactive controls; **solid or 1–2px focused keyline/fill with no outer halo, bloom or hard-to-read neon ring**. Centered Play/Pause is dominant by placement and proportion, *not* by a glowing border. Keep all elements on 1920×1080 safe margins and readable from a couch.

Do this via named `theme` roles and a **shared widget style variant** for video-plane focus; do not globally delete focus treatment from unrelated screens and do not duplicate `TransportButton` drawing in a new screen-local primitive. Use `HStack`/`VStack`, shared springs and existing clip/material systems, measured title fit and line bounds. Allow smooth **vertical reflow** between base, chapter and info states without moving controls horizontally, focus jumping or continuously blurring the video.

## Acceptance
- No focused control or chapter card produces glow/bloom; there is still an obvious high-contrast remote focus state on bright and dark video frames.
- Play/Pause stays *exactly centered* across states; title/seek/timestamps and right controls remain aligned, legible and non-overlapping.
- Typical and extreme localized labels fit at `1920×1080`; ordinary content remains at or above 24px.
- Expand/collapse motion is brief, interruption-safe and reduced-motion compatible; the resting hidden HUD adds no continual unnecessary redraw.
- Screenshot review at all three levels and at least one bright-scene frame; pointer hit geometry is checked during animation.
