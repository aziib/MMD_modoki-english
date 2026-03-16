# Timeline Specification and Implementation Notes

Updated: 2026-02-23
Target:
- `src/timeline.ts`
- `src/ui-controller.ts`
- `src/mmd-manager.ts`
- `src/types.ts`
- `index.html`

## 1. Purpose
- Visualize motion on a frame-by-frame basis and perform seek/select/key editing.
- Handle model editing and camera editing in the same UI.
- Synchronize bone panel/3D bone selection with track selection.

## 2. UI Specification

### 2-1. Structure
The timeline area in `index.html` consists of the following.
- Edit toolbar: `#btn-kf-add`, `#btn-kf-delete`, `#btn-kf-nudge-left`, `#btn-kf-nudge-right`
- Selection display: `#timeline-selection-label`
- Drawing area:
- Left label: `#timeline-label-canvas`
- Top ruler/playhead: `#timeline-overlay-canvas`
- Main track: `#timeline-canvas`

Reference: `index.html:122`

### 2-2. Layer Design
`Timeline` uses 3 layers of Canvas.
- Static: Row background + key points (`#timeline-canvas`)
- Overlay: Ruler + playhead (`#timeline-overlay-canvas`)
- Label: Left label (`#timeline-label-canvas`)

Redrawing is separated to the minimum necessary.
- `setCurrentFrame`: overlay + static
- `setKeyframeTracks`: static + label (+ resize)
- Scroll: static

Reference: `src/timeline.ts:13`, `src/timeline.ts:294`

## 3. Data Model

### 3-1. Track Type
- `KeyframeTrack`:
- `name`: Bone/morph/camera channel name
- `category`: `root | camera | semi-standard | bone | morph`
- `frames`: Ascending `Uint32Array`

Reference: `src/types.ts:45`

### 3-2. Internal Retention
The following are held on the `MmdManager` side.
- Per-model tracks: `WeakMap<MmdModel, Map<string, Uint32Array>>`
- Camera track: `cameraKeyframeFrames` (common frame column)
- Track keys: `category + separator + name`

Reference: `src/mmd-manager.ts:133`, `src/mmd-manager.ts:210`, `src/mmd-manager.ts:3442`

### 3-3. Frame Array Operations
Frame columns are edited with binary search.
- Add: `addFrameNumber`
- Delete: `removeFrameNumber`
- Move: `moveFrameNumber`
- Duplicate removal merge: `mergeFrameNumbers`

Reference: `src/mmd-manager.ts:51`

## 4. Track Generation Specification

### 4-1. Model Target
When targeting model, use `getActiveModelTimelineTracks()`.
- Pass only visible bones (`activeModelInfo.boneNames`)
- Fill tracks based on PMX-order bones
- Place `root` category at the head group
- Add remaining bones and morphs sequentially
- Add unconsumed tracks on existing Map at the end

Reference: `src/mmd-manager.ts:3765`

### 4-2. Camera Target
When targeting camera, fix display to `Camera` 1 row.
- `Camera`

Camera row shares `cameraKeyframeFrames` and interpolation display is handled with 6ch of `X/Y/Z/rotation/distance/FoV`.

Reference: `src/mmd-manager.ts:3844`

### 4-3. Firing Timing
Track update events are fired with `emitMergedKeyframeTracks()`.
- Add/delete/move
- Target switch (model/camera)
- VMD/VPD/camera VMD loading
- Active model switch/delete

Reference: `src/mmd-manager.ts:3907`, `src/mmd-manager.ts:432`

## 5. Operation Specification

### 5-1. Seek
- Click:
- static click: Track selection + nearby key selection + seek
- overlay click: Seek only
- Drag:
- Move frame horizontally with left button drag
- Frame is clamped with `max(0, frame)`
- API:
- `timeline.onSeek(frame)` -> `mmdManager.seekTo(frame)`

Reference: `src/timeline.ts:115`, `src/timeline.ts:173`, `src/ui-controller.ts:333`

### 5-2. Selection
- Label click: Row selection only
- static click: Row selection + nearby key selection (within 8px)
- Selection state:
- `selectedTrackIndex`
- `selectedFrame` (null if not hit)
- Selection label explicitly indicates key type (`[Bone]` / `[Morph]` / `[Camera]`)

Reference: `src/timeline.ts:528`, `src/timeline.ts:544`, `src/timeline.ts:558`

### 5-3. Key Editing
- Register:
- Register at current frame
- For existing frames, prioritize latest registration and overwrite (later wins)
- Bones/camera insert actual value snapshot and interpolation value to source animation side at registration time
- Delete:
- If selected key exists, that frame; if not, current frame is deletion target
- Move:
- If selected key exists: Move key `±1` frame
- If no selected key: Frame seek

Reference: `src/ui-controller.ts:1268`, `src/ui-controller.ts:1287`, `src/ui-controller.ts:1308`

## 6. Shortcut Specification
- `+`, `NumpadAdd`, `K`, `I`: Key register
- `Delete`: Key delete
- `Alt + ←/→`: Key move (nudge)
- `←/→`: Frame move (`Shift` for 10f)
- `Home/End`: Jump to start/end
- `Space`: Play/pause

Reference: `src/ui-controller.ts:805`

## 7. Cooperation with Playback

### 7-1. Frame Update
- Receiving `mmdManager.onFrameUpdate(frame, total)`:
- Update current/total frame display
- `timeline.setCurrentFrame(frame)`
- Update edit button state

Reference: `src/ui-controller.ts:706`

### 7-2. Stop at End
- Execute `stopAtPlaybackEnd()` when `isPlaying && frame >= total`
- Implementation is `pause()` + `seekTo(totalFrames)`, so end frame is maintained even after stop

Reference: `src/ui-controller.ts:728`, `src/ui-controller.ts:1356`

### 7-3. Playback Without Audio
- When no audio, manual progression at 30fps conversion with `manualPlaybackWithoutAudio`
- When audio exists, adopt runtime's `currentFrameTime`

Reference: `src/mmd-manager.ts:1571`, `src/mmd-manager.ts:2263`

### 7-4. Seek Upper Limit
- `seekTo(frame)` expands `totalFrames` if `frame > totalFrames`.
- Therefore, can progress effectively without upper limit with arrow keys.

Reference: `src/mmd-manager.ts:2298`

## 8. Timeline Reflection on Loading
- VMD:
- Hold current frame at load time, return to that frame after application
- VPD:
- Use current frame as `frameOffset` and insert pose offset
- Merge to existing animation
- Camera VMD:
- Update `cameraKeyframeFrames` and reflect to camera row

Reference: `src/mmd-manager.ts:2026`, `src/mmd-manager.ts:2077`, `src/mmd-manager.ts:2148`

## 9. Bone Selection Synchronization
- Synchronize timeline row selection <-> bone panel selection <-> 3D bone selection.
- Avoid recursive updates with `syncingBoneSelection` flag.
- Target is only `root/semi-standard/bone` categories.

Reference: `src/ui-controller.ts:336`, `src/ui-controller.ts:1201`, `src/ui-controller.ts:1214`, `src/ui-controller.ts:1228`

## 10. Current Constraints
- Range operations for key editing (multiple selection/copy/paste/scale) are not implemented.
- Interpolation is editable, but Property (display/IK) step edit UI is not implemented.
- Property (display/IK) track editing is not implemented.
- VMD export is not implemented.
- MMD actual machine comparison testing for rotation interpolation is not prepared.

Related:
- `docs/mmd-basic-task-checklist.md`
- `docs/mmd-keyframe-bone-interpolation-research.md`
