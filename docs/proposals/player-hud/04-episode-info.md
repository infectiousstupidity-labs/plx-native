# [Proposal][HUD-04] Reveal readable episode information below the chapter rail

**Priority:** P1. **Depends on:** HUD-02, HUD-03.

## Current behavior
`appkit/info_panel.rs::InfoPanelState` draws a **standalone near-opaque horizontal card** with 16:9 artwork, synopsis, meta/capability chips and `From Beginning` / `Go to Detail` actions. `overlay.rs` mounts it as one modal overlay; its focus group is a column of actions. It uses `metadata.now_playing()` for the leaf and `metadata.current()` only where appropriate as a fallback. The current bottom Info tab opens that card rather than scrolling below chapters.

## Proposed change
Convert the Info presentation to a **second, downward-revealed section**, below the still-visible chapter row. Read-only episode title, description and meaningful tags should lead; actions should be secondary and reached deliberately by focus. Use the playing leaf's episode title, show name, season/episode, summary, duration and existing trustworthy metadata. Retain `InfoAction::FromBeginning` and `GoToDetail` where appropriate without putting a destructive action under the landing focus. Do not duplicate the header or crowd the video with a large opaque floating rectangle.

Design for long summaries: use measured `TextView`/shared `prose_scroll` or a capped+expandable layout rather than tiny text, unexpected clipping or gigantic permanent sheets. A second Down must expose a meaningful first line; any scrolling should be bounded and return cleanly via Up/Back. If there are no chapter thumbnails, the first expansion is the info section. Keep film, metadata and subtitles legible while the section scrolls into place.

## Acceptance
- Info appears BELOW Chapters at level 2; both remain readable and focusable. No screenshots with an overlay replacing all prior rows.
- Correct episode, show, movie and trailer metadata; blank summary, large summary, long names, localized facts, variable chip count and absent stills handled.
- No accidental `From Beginning` or detail navigation from landing focus or Back; actions continue to work with pointer and D-pad.
- Geometry and clipping never intersect title, transport bar, chapter captions or screen safe insets.
- Reuse existing information extraction and shared text/layout components. Add fit/layout/action regressions and 1920×1080 screenshots.
