# VMD/VPD Import Behavior Notes

Updated: 2026-02-23
Target:
- `src/mmd-manager.ts`
- `src/ui-controller.ts`

## 1. File Type
- `loadVMD(filePath)` branches by extension
- `.vmd`: Model motion
- `.vpd`: Pose (delegates to `loadVPD`)
- Camera VMD is separate path with `loadCameraVMD(filePath)`

Reference:
- `src/mmd-manager.ts:2016`
- `src/mmd-manager.ts:2148`

## 2. Model VMD
- Current model required. Error if not loaded.
- Hold `currentFrame` at load time and `seekTo` to that frame after application.
- Update `modelSourceAnimationsByModel` with new animation.
- Regenerate timeline frame column with `buildModelTrackFrameMapFromAnimation`.

Reference:
- `src/mmd-manager.ts:2021`
- `src/mmd-manager.ts:2026`
- `src/mmd-manager.ts:2055`

## 3. VPD
- Current model required. Error if not loaded.
- Apply `currentFrame` at load time as `frameOffset`.
- If existing animation, integrate with `mergeModelAnimations(base, overlay)`.
- On same frame conflict, prioritize overlay (new load) side.
- Maintain edit position with `seekTo(loadFrame)` after load.

Reference:
- `src/mmd-manager.ts:2084`
- `src/mmd-manager.ts:2102`
- `src/mmd-manager.ts:2105`
- `src/mmd-manager.ts:3523`

## 4. Camera VMD
- Verify cameraTrack, error if empty.
- Replace camera runtime animation.
- Update `cameraKeyframeFrames` and reflect to timeline (`Camera` 1 row).
- Initialize to `currentFrame = 0` after load.

Reference:
- `src/mmd-manager.ts:2166`
- `src/mmd-manager.ts:2178`
- `src/mmd-manager.ts:2181`

## 5. totalFrames Update Rule
- Basically `refreshTotalFramesFromContent()` recalculates in `emitMergedKeyframeTracks()`.
- If keys exist without audio, ensure length based on `maxFrame`.
- `seekTo(frame)` expands upper limit if `frame > totalFrames`.

Reference:
- `src/mmd-manager.ts:3887`
- `src/mmd-manager.ts:2298`

## 6. UI Reflection
- Notify `onMotionLoaded` / `onCameraMotionLoaded` on load completion.
- UI side executes `timeline.setTotalFrames(frameCount)` and toast display.
- Actual key rows are redrawn separately with `onKeyframesLoaded`.

Reference:
- `src/ui-controller.ts:759`
- `src/ui-controller.ts:766`
- `src/ui-controller.ts:774`

## 7. Current Constraints
- Additional load is "composition", but strict management of edit values is not prepared (centered on frame position).
- Camera value snapshot is saved on key registration, but range editing is not supported.
- VMD export is not implemented.
