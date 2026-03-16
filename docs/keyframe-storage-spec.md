# Keyframe Storage Specification (Current)

Updated: 2026-02-23
Target:
- `src/mmd-manager.ts`
- `src/types.ts`

## 1. Data Structure

### 1-1. Track Unit
- Type: `KeyframeTrack`
- `name`: Track name
- `category`: `root | camera | semi-standard | bone | morph`
- `frames`: `Uint32Array` (ascending, no duplicates)

Reference: `src/types.ts:45`

### 1-2. Actual Storage
- Model: `WeakMap<MmdModel, Map<string, Uint32Array>>`
- Camera: `cameraKeyframeFrames: Uint32Array`
- `Map` key is `createTrackKey(category,name)` (separator is `\u001f`)

Reference:
- `src/mmd-manager.ts:133`
- `src/mmd-manager.ts:210`

## 2. Invariant Conditions
- `frames` is always ascending
- No duplicates in `frames`
- Add/delete/move generates new array in immutable style and replaces

Reference:
- `src/mmd-manager.ts:83`
- `src/mmd-manager.ts:105`
- `src/mmd-manager.ts:126`

## 3. Operation Specification

### 3-1. has
- `hasTimelineKeyframe(track, frame)`
- frame is `Math.floor`, normalized to lower limit 0
- camera category references `cameraKeyframeFrames`

Reference: `src/mmd-manager.ts:1165`

### 3-2. add
- `addTimelineKeyframe(track, frame)`
- If existing frame, no-op (`false`)
- On change, `emitMergedKeyframeTracks()`

Reference: `src/mmd-manager.ts:1179`

### 3-3. remove
- `removeTimelineKeyframe(track, frame)`
- If non-existent, no-op (`false`)
- On change, `emitMergedKeyframeTracks()`

Reference: `src/mmd-manager.ts:1201`

### 3-4. move
- `moveTimelineKeyframe(track, from, to)`
- Implementation is `remove + add`
- Even if existing key at move destination, ultimately converges to no-duplicate array

Reference: `src/mmd-manager.ts:1223`

## 4. Injection from Load
- On VMD/VPD load, rebuild with `buildModelTrackFrameMapFromAnimation()`
- Can offset to load frame by giving `frameOffset` (used in VPD)

Reference:
- `src/mmd-manager.ts:3729`
- `src/mmd-manager.ts:2102`

## 5. Track Regeneration
- Output tracks are generated every time with `getActiveModelTimelineTracks()` / `getCameraTimelineTracks()`.
- Since filtered to display target bones only, cases where non-displayed even if remaining in storage exist.

Reference:
- `src/mmd-manager.ts:3765`
- `src/mmd-manager.ts:3858`

## 6. Current Constraints
- Timeline management (`MmdManager`) is centered on frame position, but on key registration, `UIController` inserts value/interpolation snapshot to source animation to synchronize.
- Camera is 1 row display, but edit data holds 6ch of `X/Y/Z/rotation/distance/FoV`.
- Property tracks (display/IK) are not supported.
