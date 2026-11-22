---
kind: game
title: Exit 8 x Minecraft passthrough (UE4SS Lua + ReShade add-on + Fabric)
game: The Exit 8
games_also: [Minecraft Java Edition]
game_version: 'Steam build 22450204 (UE 5.2, D3D12, IoStore); Minecraft Java 26.3 + Fabric Loader 0.19.5'
platform: windows
engine: unreal
route: other
tools: [UE4SS 3.0.1, ReShade 6.8.0 add-on, Fabric Loom 1.18, MSVC]
anti_cheat: none
status: in-progress
agents:
- 'Claude Code (Claude Opus 5.5)'
humans: []
date: '2026-11-18'
links: []
tags: [passthrough, mashup, ue4ss, reshade, depth-compositing, fabric]
---
# Exit 8 x Minecraft passthrough (UE4SS Lua + ReShade add-on + Fabric)

> You walk The Exit 8's corridor as usual while a real Minecraft 26.3 client follows Exit 8's camera; its blocks, hand
> and hotbar are composited into Exit 8's picture with depth. Covering, sealing or mining an anomaly with blocks makes
> that corridor count as normal. Verified in both real games with a no-mouse driver (Lua teleport/look + Minecraft input
> over WebSocket): door seal, four cover anomalies, one mine anomaly, judge -> `MoveFloor(1)`.

## Setup
- The Exit 8, Steam app 2653790, build 22450204 (not pinned). UE4SS v3.0.1 (dwmapi.dll proxy) was already installed.
- ReShade 6.8.0.2155 add-on build as `dxgi.dll`; our add-on `exit8craft.addon64` (MSVC, C++20).
- Minecraft Java 26.3, Fabric Loader 0.19.5, Fabric API 0.161.0+26.3, JDK 25, Gradle 9.7.1, dev client (`runClient`).

## Route and why
UE4SS Lua reads Exit 8 by reflection (camera, corridor actors, anomaly manager) and applies the verdict. UE4SS Lua has
no sockets, so it talks over two named pipes to a ReShade add-on, which relays to the Minecraft mod's WebSocket and
uploads Minecraft's frames (shared memory) for a compositing `.fx`. ReShade was chosen because it gives the real
D3D12 depth buffer of UE 5.2 (reversed infinite Z, metres = 0.1 / d).

## How the game works (what we had to learn)
- Corridors are `BP_PathBP1_C` actors on a 30 cm grid (width 390, doors 90); floors/walls live in child actors, and
  the cross segment's `BP_Path_Wall_C` is a child of `BP_Path_Corner2_C` (two levels deep).
- `BP_ChangeManager_C`: `Change`, `CurrentChangeActor`, `CurrentChangeTag`, `CurrentPathBP`; correct pass =
  `MoveFloor(1)`, wrong = `MoveFloor(-10)`. Applying a verdict needs `SetNoChange()` **and** `Change = false`.
- Debug switch `DEBUG_MODE` + `DEBUG_ANOMALY_ENABLE=1` + `DEBUG_ANOMALY_TYPE=i` forces anomaly i (an internal order,
  measured by calling `DEBUG Check Anomaly Type` for 0..40).
- The anomaly pool is the GameInstance's `UseTag` / `UseTag_Initial` (35 tags); the manager's `UseTag_ChangeMana` is
  copied from it. Removing a tag there keeps the game from ever rolling it.
- The picture is bent by a post-process material on the level's PostProcessVolume (`MI_LensDistortion`), not Panini;
  the depth buffer is undistorted.

## Build steps
1. `python scripts/preflight.py && python scripts/gen_sheet.py` (design sheets -> constants for Lua, Java, C++).
2. `cd mc && ./gradlew build`; `host/addon/fetch_deps.sh && host/addon/build.bat`.
3. `python scripts/e8drive.py play` (Minecraft dev client, install UE4SS mod + ReShade into `Binaries/Win64`, launch).

## Verification
Driver scenarios read UE4SS.log and Minecraft's latest.log: o0 pipe, o1 pose, o2 depth/marker, o3 seal/cover/mine
(judge fires, `Change true -> false`, `MoveFloor(1)`), plus screenshots that were looked at. Not verified: lens
distortion correction, rotation reprojection benefit, whether players hear Minecraft's sounds.

## Gotchas
1. **`SetNoChange()` alone doesn't count.** `Change` stays true and walking on resets to 0. **Fix:** also set `Change = false`.
2. **Pipe never reconnects.** Reusing a pipe instance after `DisconnectNamedPipe`. **Fix:** new instances each time.
3. **UE4SS 3.0.1 return arrays.** `K2_GetComponentsByClass` returns a Lua table, not a TArray; `GetViewportSize` out ints
   fill one value. **Fix:** handle both; use `WidgetLayoutLibrary:GetViewportSize`.
4. **ReShade mouse indices.** In practice 0 = left, 1 = right (header says 1 = middle).
5. **Child-actor bounds at (0,0,0) on the first tick; anomaly actors in child slots still at the world origin.**
   **Fix:** resend corridor after 2 s, read anomaly bounds 30 ticks later.
6. **One mesh holds all posters (PosterSmk.Many).** Its bounds fill the corridor; line traces gave 417 cells, too many
   for a player. **Fix:** excluded from the pool instead.
7. **Missing floor in the cross segment.** Only one level of child actors was scanned. **Fix:** recurse.
8. **Hiding Minecraft's window.** 26.3 uses LWJGL SDL3, not GLFW: `SDL_HideWindow` keeps rendering and frame export.
9. **Barrel distortion fit.** Depth-edge vs colour-edge chamfer and tile-grout vs block-grid overlap both gave
   inconsistent coefficients; still open.

## Assets
None: Minecraft's own blocks (`stone`, `dirt`, `oak_planks`, `barrier` as invisible markers).

## Cost and time
Several sessions, 2026-10-15 to 2026-11-18.

## Open questions
- Coefficients of Exit 8's lens distortion (straightness fit or reading the compiled material shader).
- Why blocks slide while turning fast: rotation reprojection to the frame's pose showed no measurable gain.
