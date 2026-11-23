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
status: working
agents:
- 'Claude Code (Claude Opus 5.5)'
humans: []
date: '2026-11-23'
links: []
tags: [passthrough, mashup, ue4ss, reshade, depth-compositing, fabric]
---
# Exit 8 x Minecraft passthrough (UE4SS Lua + ReShade add-on + Fabric)

> You walk The Exit 8's corridor as usual while a real Minecraft 26.3 client follows Exit 8's camera; its blocks, hand
> and hotbar are composited into Exit 8's picture with depth. Covering, sealing or mining an anomaly with blocks makes
> that corridor count as normal. Verified in both real games with a no-mouse driver (Lua teleport/look + Minecraft input
> over WebSocket): door seal, four cover anomalies, one mine anomaly, judge -> `MoveFloor(1)`. Lens distortion and the
> slide while turning were measured and fixed with real mouse input (layers stay within 0.5 px while turning).

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
  the depth buffer is undistorted. Measured mapping: the final picture at uv shows the scene at
  `0.5 + (uv - 0.5)(k0 + k1 r^2 + k2 r^4)` with r in plain uv (no aspect scaling), k0 0.91586, k1 0.32376, k2 -0.42115
  (centre magnified about 9%, 0.55 px residual at 1280x720). The blendable's `Weight` is writable from Lua at runtime.
- The first-person camera hangs on a spring arm `SmoothCameraRotation` with rotation lag (speed 15, a game setting
  applied by `Apply_CameraLagSpeed`; substepping on, max step 1/60 s) that chases the control rotation.
- Tick order: the pawn's `ReceiveTick` runs after the PlayerController (input already in the control rotation) but before
  the camera manager update, so `PlayerCameraManager:GetCameraRotation()` read there is last frame's camera.

## Build steps
1. `python scripts/preflight.py && python scripts/gen_sheet.py` (design sheets -> constants for Lua, Java, C++).
2. `cd mc && ./gradlew build`; `host/addon/fetch_deps.sh && host/addon/build.bat`.
3. `python scripts/e8drive.py play` (Minecraft dev client, install UE4SS mod + ReShade into `Binaries/Win64`, launch).

## Verification
Driver scenarios read UE4SS.log and Minecraft's latest.log: o0 pipe, o1 pose, o2 depth/marker, o3 seal/cover/mine
(judge fires, `Change true -> false`, `MoveFloor(1)`), plus screenshots that were looked at.
- Lens: still A/B shots with the material weight 1/0/1 and time dilation 0.0001, block matching (1122 inlier pairs), then
  a glass-shell overlay: the corridor's end wall lines up with the fit and sits ~10 px off without it.
- Turning: a debug view writes Exit 8's and Minecraft's log depth into the red and green channels of one presented
  frame, so a screenshot gives the horizontal offset between the layers directly. Swing +-40 deg at 3 deg/tick:
  70.7 px uncorrected -> 0.2 px; real mouse (SendInput) 0.3-0.5 px.
- Sound: Minecraft's audio session (java.exe) peaks at 0.41 while placing/breaking (pycaw meter). Nobody listened yet.
- Not verified: position prediction while walking (not done; 3-7 cm lag at walking speed).

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
9. **Barrel distortion fit from indirect cues failed.** Depth-edge chamfer and tile-grout vs block-grid overlap gave
   "scale 0.92 + k1", rejected because the centre looked right. Cause: the model lacked a centre scale and stretched r
   by the aspect ratio. **Fix:** toggle the material's blendable weight and match the two pictures; fit k0 + k1 r^2 +
   k2 r^4 in plain uv. Freeze idle camera sway first (global time dilation 0.0001; ticks keep running).
10. **"Reprojection does nothing."** Measured as a share of depth-mismatched pixels during 360 deg spins. Causes: the pose
   was one frame stale (read before the camera update), and spins put walls in view where the offset is ambiguous.
   **Fix:** measure the offset in one frame (depth pair debug view) while swinging down the corridor; pose lag 0/1/2
   gave 36/71/103 px, one tick per step, which exposed the stale pose.
11. **Predicting with the control-rotation delta only.** Right for a steady scripted turn, ~15 px off with a real mouse,
   because the spring arm's lag only matches the delta at constant speed. **Fix:** replay the arm's QInterpTo step in
   Lua; its socket rotation already turned with the pawn this frame, so subtract the pawn's turn to get the previous
   arm rotation. Error 1.46 deg -> 0.03 deg (yaw), 1.12 -> 0.01 (pitch).
12. **Restarting the hidden Minecraft client.** A window-title search misses a hidden window; the old client keeps the
   WebSocket port. **Fix:** find the dev client by command line (`fabric.dli.config`, `KnotClient`).
13. **Scripted runs after a run.** Exit 8 moves its two corridor actors forward as you advance; fixed teleport targets
   then point at empty space and the fall ends in `GameOver`. **Fix:** restart the game before each scripted run.

## Assets
None: Minecraft's own blocks (`stone`, `dirt`, `oak_planks`, `barrier` as invisible markers).

## Cost and time
Several sessions, 2026-10-15 to 2026-11-23.

## Open questions
- Position prediction while walking (pawn velocity x dt) or full 6-DoF reprojection with Minecraft's depth.
- Exit 8 isn't pinned; a game update may rename the reflected Blueprint names.
