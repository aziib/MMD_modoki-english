# Timeline Data Flow

Updated: 2026-02-22
Target:
- `src/ui-controller.ts`
- `src/mmd-manager.ts`
- `src/timeline.ts`

## 1. Purpose
- Fix responsibility boundaries and event directions around timeline.

## 2. Component Responsibilities
- `Timeline`:
- Canvas drawing, row/key selection, seek input firing
- `UIController`:
- UI event bundling, selection synchronization, edit command dispatch
- `MmdManager`:
- Actual data holding (frame arrays/playback state) and execution, notification firing

## 3. Main Flow

### 3-1. Track Update Flow
1. `MmdManager.emitMergedKeyframeTracks()` generates track array  
2. Notify `onKeyframesLoaded(tracks)`  
3. `UIController` executes `timeline.setKeyframeTracks(tracks)`  
4. `UIController` reapplies selection synchronization (bone panel/visualizer)

Reference:
- `src/mmd-manager.ts:3907`
- `src/ui-controller.ts:774`

### 3-2. Seek Flow
1. `Timeline.onSeek(frame)` fires  
2. `UIController -> mmdManager.seekTo(frame)`  
3. `MmdManager.onFrameUpdate(frame,total)` notification  
4. `UIController -> timeline.setCurrentFrame(frame)` + frame display update

Reference:
- `src/timeline.ts:84`
- `src/ui-controller.ts:333`
- `src/ui-controller.ts:706`

### 3-3. Selection Synchronization Flow (Bone)
1. Timeline selection change  
2. `UIController.syncBottomBoneSelectionFromTimeline()`  
3. `UIController.syncBoneVisualizerSelection()`  
4. Reverse direction (bone panel/3D pick -> timeline) is similar

Reference:
- `src/ui-controller.ts:336`
- `src/ui-controller.ts:1201`
- `src/ui-controller.ts:1214`
- `src/ui-controller.ts:797`

### 3-4. Key Edit Flow
1. Edit request from UI (button/shortcut)  
2. `UIController` calls `add/remove/moveTimelineKeyframe`  
3. `MmdManager` updates frame array  
4. Reflect to redraw with `emitMergedKeyframeTracks()`

Reference:
- `src/ui-controller.ts:1268`
- `src/mmd-manager.ts:1179`

## 4. Implementation Rules
- `Timeline` holds state, but source-of-truth is `MmdManager` side.
- Always reflect via `emitMergedKeyframeTracks()` after edit.
- Suppress recursive loops with `syncingBoneSelection` for selection synchronization.

## 5. Change Notes
- Do not update model directly from `Timeline` (always `UIController -> MmdManager`).
- When adding new track category:
- `TrackCategory` definition
- `MmdManager` track generation
- `Timeline` coloring/drawing
- `UIController` selection synchronization conditions
