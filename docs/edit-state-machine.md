# Edit State Transition Notes

Updated: 2026-02-22
Target:
- `src/mmd-manager.ts`
- `src/ui-controller.ts`

## 1. Purpose
- Fix state transitions for playback/stop/seek/bone operation to make behavior differences and issues easier to track.

## 2. Main States
- `Idle`: Non-playback. Editable.
- `PlayingAudio`: Playback with audio.
- `PlayingManual`: Playback without audio (manual frame progression at 30fps conversion).
- `HardSeeking`: Temporary state at head/end jump (internal processing).
- `BoneGizmoDragging`: During bone gizmo drag (physics temporarily OFF).

## 3. State Transitions

### 3-1. Playback Series
- `Idle -> PlayingAudio`: `play()` executed and `audioPlayer !== null`
- `Idle -> PlayingManual`: `play()` executed and `audioPlayer === null`
- `PlayingAudio|PlayingManual -> Idle`: `pause()` or `stop()`
- `Playing* -> Idle`: `UIController.stopAtPlaybackEnd()` on end frame arrival

Reference:
- `src/mmd-manager.ts:2263`
- `src/ui-controller.ts:706`
- `src/ui-controller.ts:1356`

### 3-2. Seek Series
- `Idle/Playing* -> HardSeeking`: `seekToBoundary(frame)`
- `HardSeeking -> Idle/Playing*`: `pause -> seekTo -> stabilizePhysicsAfterHardSeek -> play(if needed)`

Reference:
- `src/mmd-manager.ts:2311`

### 3-3. Gizmo Series
- `Idle -> BoneGizmoDragging`: Gizmo drag start
- `BoneGizmoDragging -> Idle`: Drag end
- Side effects:
- At start: Temporarily OFF if physics ON
- At end: Return if originally ON

Reference:
- `src/mmd-manager.ts:1528`

## 4. Constraints by State
- During `Playing*`:
- Bone overlay hidden
- Bone gizmo disabled
- When `timelineTarget === camera`:
- Bone overlay hidden
- Bone gizmo disabled

Reference:
- `src/mmd-manager.ts:457`
- `src/mmd-manager.ts:763`
- `src/mmd-manager.ts:1098`

## 5. End Stop Policy
- End frame arrival judgment is done on `onFrameUpdate` side.
- On stop, `pause()` and maintain end frame with `seekTo(totalFrames)`.
- Does not return to 0 frame like `stop()`.

Reference:
- `src/ui-controller.ts:728`
- `src/ui-controller.ts:1356`

## 6. Change Checklist
- When touching playback control:
- Confirm end stop for both `PlayingAudio` and `PlayingManual`
- When touching `HardSeeking`:
- Confirm physics runaway does not recur on head/end jump
- When touching bone edit:
- Confirm gizmo/overlay not displayed during playback
