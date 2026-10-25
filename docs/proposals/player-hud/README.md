# Player HUD redesign — proposed issue backlog

Status: **proposal only**. Review before implementation. Target: native 1920×1080 LG webOS player; **not** a browser/Web UI recreation. Based on `main` at `db75bbbfd7ecab91043817849abce457826a3bb6` (2026-10-25). No runtime files were changed.

## Desired UI and interaction

1. **Base HUD:** compact identity/title and seek bar above **centered transport controls**; right-hand **Subtitles, Audio and More** cluster. Subtitles **must never** be in the centered playback cluster. Retain the hardware video beneath a dark bottom gradient rather than a large blurred or glass card.
2. With **Play/Pause focused**, press **DOWN**: the layout scrolls/reflows to reveal a focusable horizontal **chapter thumbnail row**; retain the playbar and controls above. Focus lands on the chapter containing the playhead.
3. Press **DOWN** again: reveal **episode/movie details** (title, description, readable metadata and any sensible actions) *below chapters*. UP reverses the journey. When no chapters exist, Down opens Info directly.
4. **BACK** collapses Info first, Chapters second, then follows the normal HUD hide/exit behavior; subordinate track/More menus preserve their own Back policies.
5. Strong, **non-glowing** focused appearance: solid fill or restrained 1–2px keyline, no halo/bloom, no giant white disc. 24px ordinary content floor, overscan margins, restrained motion, stable focus and pointing geometry.
6. Never invent unavailable episode-navigation controls: implement or explicitly suppress Previous/Next buttons until they perform real actions. The core center group is rewind / pause-play / forward.

## Code audit — verified starting point

- [`appkit/src/player_hud.rs`](../../../rust-modules/appkit/src/player_hud.rs): `draw_hud` composes show/episode identity, scrubber, 3 **right-anchored** Subtitles/Audio/More discs, and bottom `Info`/`Chapters` tabs. There is **no centered clickable transport row**. Geometry and associated pointer hit areas are in this file. The currently focused scrubber knob itself paints a glow.
- [`screens/src/player/input.rs`](../../../rust-modules/screens/src/player/input.rs): `HudNav` uses `focus=0` scrubber, `1` right discs, `2` bottom tabs; auto-hide and dismiss state are here.
- [`screens/src/player/mod.rs`](../../../rust-modules/screens/src/player/mod.rs): `key_updown` explicitly walks those 3 existing regions, `key_ok` dispatches a control-row press or opens separate Info/Chapters overlays. `Focusable::groups/place` registers the scrubber, right discs and tabs for pointer/D-pad routing. Preserve the pointer stop contract.
- [`screens/src/player/overlay.rs`](../../../rust-modules/screens/src/player/overlay.rs): `PlayerOverlayScreen` contains **one** of Tracks, Info, Chapters, More, Timing at a time, with its own focus and Back rules. Info/Chapters currently dismiss on Down past their lower edge and return to tabs; they are not a vertically stacked two-level document.
- [`appkit/src/chapters_panel.rs`](../../../rust-modules/appkit/src/chapters_panel.rs): already builds a 16:9 chapter-thumbnail rail, chooses the active chapter by playback position, clamps scroll, seeks by chapter start and reads **`metadata.playing_chapters()`**, not stale detail-page metadata. Fixed `CH_TOP=684` currently assumes old layout.
- [`appkit/src/info_panel.rs`](../../../rust-modules/appkit/src/info_panel.rs): already has synopsis, 16:9 still, metadata and `From Beginning` / `Go to Detail` actions. Currently draws a near-opaque **card** with a fixed button column and assumes it replaces the transport middle.
- [`screens/src/registry.rs`](../../../rust-modules/screens/src/registry.rs): existing `PlayerReq` supports Transport, SeekTo, Info, OpenOverlay and ArmControlRow. It does **not** declare a generic Previous Episode request. `up_next` is a separate conditional end-of-episode control.
- [`ui/src/widgets.rs`](../../../rust-modules/ui/src/widgets.rs): `TransportButton`/control rim styling is shared; `ControlGround::Unkeyed` focused rims currently can produce an inner glow. Add an intentional non-glow shared treatment; don't change other screens accidentally.
- [`tests/player_shots.sh`](../../../tests/player_shots.sh): simulator already captures base HUD, Tracks, Info, Chapters, More — script assumes existing 3-row key path and must be updated to capture the three proposed progressive levels.
- [`AGENTS.md`](../../../AGENTS.md), [`ui/src/CLAUDE.md`](../../../rust-modules/ui/src/CLAUDE.md) and [`ui-sim`](../../../.agents/skills/ui-sim/SKILL.md): native 1920×1080, shared focus/layout/motion, simulated screenshots and real TV checks. Do not add screen-local focus engines, color literals or manual animation systems.

## Proposed tasks — suggested sequence

| ID | Priority | Issue-ready proposal | Depends on |
|---|---|---|---|
| HUD-01 | P0 | [Centered playable transport row](01-centered-transport.md) | — |
| HUD-02 | P0 | [Progressive Down/Up/Back navigation](02-progressive-navigation.md) | HUD-01 |
| HUD-03 | P1 | [Inline expanding chapter rail](03-chapter-rail.md) | HUD-02 |
| HUD-04 | P1 | [Inline episode info/details section](04-episode-info.md) | HUD-02, HUD-03 |
| HUD-05 | P1 | [Right-side secondary controls, including Subtitles](05-secondary-controls.md) | HUD-01 |
| HUD-06 | P1 | [No-glow visual polish and layout motion](06-visual-polish.md) | HUD-01–05 |
| HUD-07 | P0 | [Behavior preservation and edge cases](07-behavior-compatibility.md) | HUD-02–05 |
| HUD-08 | P0 | [Automated UI snapshots and LG hardware verification](08-verification.md) | HUD-01–07 |
| HUD-09 | P2 | [Previous/Next episode capability — no dead buttons](09-episode-navigation.md) | HUD-01 |

Each linked file is a standalone GitHub-issue body with scope, source anchors and acceptance criteria. Split into small, independently reviewed commits/PRs rather than a UI rewrite. The P2 navigation issue may be deferred without blocking the three-state UI if the unavailable buttons are simply not shown.

## Explicit non-goals

No changes to media decoding, Plex server protocol, backend chapter ingestion, subtitle codec support, home screen or unrelated UI. No changes to `main` from this proposal. Preserve existing Up Next/Skip conditional control priorities. Do not publish private Plex servers, addresses, token paths or recordings.
