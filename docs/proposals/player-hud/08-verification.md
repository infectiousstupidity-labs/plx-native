# [Proposal][HUD-08] Make player-HUD visual states and remote interaction a regression gate

**Priority:** P0. **Depends on:** HUD-01–07. This is a delivery gate, not permission to postpone tests until the end.

## Current verification surfaces
`tests/player_shots.sh` already captures HUD, Tracks, Info, Chapters and More using a synthetic playback fixture and remote FIFO, but its commands depend on the **old** row order (Up to the right discs, Down twice to Info/Chapters tabs). `.agents/skills/ui-sim/SKILL.md` provides macOS simulator and Windows 11/WSLg instructions and requires an actual 1920×1080 drawable to judge pixels; Windows simulation lacks full video playback. `docs/testing-player-pointer.md` has concrete Magic Remote/pointer and paused-seek tests. `AGENTS.md` requires committed-tree gates and real-device checks for GPU/video-plane behavior.

## Proposed change
Update the screenshot fixture/driver to automatically capture:
1. **Base HUD:** Play/Pause centered and focused; right side Subtitles/Audio/More; no glow.
2. **One Down:** chapter thumbnail rail visible and focused, with transport still above.
3. **Two Downs:** chapter rail and Info visible, selected/reading focus clear.
4. **Up / Back:** progressive collapse states and correct returned focus.
5. No-chapters movie/episode fallback, long localized title/summary, absent art, selected chapter with long rail, bright/dark image.
6. Subtitles and More open and closed from their right cluster, and conditional Skip/Up Next.

Use existing synthetic mock PMS and screenshot tooling; capture stable final frames **and intermediate animated focus/hit stops** where reproducible. Test actual `Focusable::place`/registered `Stop` equality rather than screenshot-only pixel coordinates. Every UI PR should attach relevant before/after 1920×1080 simulator captures and run the appropriate host tests, `make check`, and `make check-nodef` on its **committed tree**. Do not claim TV correctness from WSLg.

## LG C1 exit criteria
- Test on original Magic Remote: D-pad Down/Up, Back, OK, pointer hover and click, repeat keys, seek while paused/playing, track menu, audio, no chapters, long text.
- Verify overlays with actual video plane, all subtitle types that exist in fixture coverage, glyph/rim contrast, overscan and real device frame-time/redraw impact. Use `tv-lock`/`tv-session`, a synthetic non-private test fixture, real release-profile build for FPS checks.
- Attach photos/screenshots, redacted logs and exact build SHA. Do not export Plex tokens, personal media or household server paths.
