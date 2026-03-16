# Playback/Seek/Physics Policy

Updated: 2026-02-22
Target:
- `src/mmd-manager.ts`
- `src/ui-controller.ts`

## 1. Playback Mode
- `play()` has 2 systems.
- With audio: Runtime normal playback
- Without audio: Manual progression with `manualPlaybackWithoutAudio = true`

Reference: `src/mmd-manager.ts:2263`

## 2. Playback Without Audio
- Progress by converting `deltaMs` to 30fps every frame.
- Advance `manualPlaybackFrameCursor` and reflect with `seekAnimation`.
- Clamp with `totalFrames` as upper limit.

Reference: `src/mmd-manager.ts:1571`

## 3. Seek

### 3-1. Normal Seek
- `seekTo(frame)`:
- Integerize `frame` and lower limit 0
- If `frame > totalFrames`, expand `totalFrames`
- Immediately reflect to runtime and notify `onFrameUpdate`

Reference: `src/mmd-manager.ts:2298`

### 3-2. Head/End Jump
- `seekToBoundary(frame)`:
- If playing, `pause()` once
- `seekTo(frame)`
- `stabilizePhysicsAfterHardSeek()`
- If originally playing, restart `play()`

Reference: `src/mmd-manager.ts:2311`

## 4. Physics Stabilization
- Execute `stabilizePhysicsAfterHardSeek()` to prevent inertia runaway after large jumps.
- Implementation:
- Reinitialize rigid bodies with `applyPhysicsStateToAllModels()`
- Reapply `seekAnimation` to current frame

Reference: `src/mmd-manager.ts:2325`

## 5. End Stop Behavior
- Detect `frame >= total` on `onFrameUpdate` side and `stopAtPlaybackEnd()`.
- Actually `pause()` + `seekTo(totalFrames)`, maintaining end frame.
- Does not return to 0 frame like `stop()`.

Reference:
- `src/ui-controller.ts:706`
- `src/ui-controller.ts:1356`

## 6. Consistency with Bone Editing
- During playback, hide/disable bone overlay/gizmo.
- During gizmo drag, temporarily turn physics OFF, return on completion.

Reference:
- `src/mmd-manager.ts:457`
- `src/mmd-manager.ts:763`
- `src/mmd-manager.ts:1528`

## 7. Operation Rules (Recommended)
- Always confirm regression when changing playback control:
- Can play both with and without audio
- Maintain end frame after end stop
- No physics runaway with Home/End rapid fire
- No blow away with gizmo operation during physics ON
