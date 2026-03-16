# Camera VMD Support Notes

This document summarizes the implementation of loading VMD for camera use.  
Target code is mainly `src/mmd-manager.ts` and `src/ui-controller.ts`.

## Purpose

- Load VMD containing camera motion (extension `.vmd`) and play
- Synchronize to same timeline as existing model motion playback flow
- Coexist with existing UI (camera fader in bottom panel)

## Implementation Policy

- Generate `MmdCamera` from babylon-mmd internally and register to `MmdRuntime` with `addAnimatable`
- When loading camera VMD, use `cameraTrack` from `VmdLoader.loadAsync` result to set runtime animation to `MmdCamera`
- Every frame, sync `MmdCamera` state to display-use `ArcRotateCamera` and draw

With this composition, can use babylon-mmd standard camera interpolation while UI side continues to handle assuming `ArcRotateCamera`.

## Load Flow

1. File selection with top toolbar `Camera VMD` button or `Ctrl + Shift + M`
2. `UIController.loadCameraVMD()` calls `MmdManager.loadCameraVMD(filePath)`
3. Binary load with `window.electronAPI.readBinaryFile`
4. Blob URL conversion and execute `VmdLoader.loadAsync("cameraMotion", blobUrl)`
5. If `cameraTrack.frameNumbers.length` is 0, treat as error
6. If existing camera animation exists, dispose and replace
7. Move to head with `mmdRuntime.seekAnimation(0, true)`
8. Notify `onCameraMotionLoaded` and update UI display

## Synchronization Mechanism

- `MmdManager` holds 2 types of cameras internally
  - For display: `ArcRotateCamera` (user operation/UI linkage)
  - For evaluation: `MmdCamera` (VMD camera key interpolation)

- Synchronize in 2 directions
  - On UI operation: `ArcRotateCamera -> MmdCamera`
  - On playback: `MmdCamera -> ArcRotateCamera` (`onBeforeRenderObservable`)

With this, screen display always reflects on `ArcRotateCamera` side even during VMD playback.

## Main APIs

- `loadCameraVMD(filePath): Promise<MotionInfo | null>`
- `onCameraMotionLoaded: (info: MotionInfo) => void`
- `setCameraPosition / setCameraRotation / setCameraTarget / setCameraFov`
  - All sync values to `MmdCamera` on UI manual operation

## UI Specification

- Top panel:
  - Add `Camera VMD` button
- Shortcuts:
  - `Ctrl + Shift + M`: Camera VMD load
- On load success:
  - Status display: `Camera motion loaded`
  - Toast display: `Loaded camera motion: <name>`
- Timeline:
  - Display `Camera` keyframes in separate lane
  - Displayed simultaneously with model VMD bone/morph tracks

## Current Constraints

- Only 1 camera VMD held (overwritten on new load)
- Cannot load VMD with empty camera track
- When model VMD and camera VMD lengths differ, overall frames follow runtime duration

## Future Extension Candidates

- Multiple camera VMD holding and switching
- Camera VMD clear button (currently replaced on reload)
- Display camera keyframes to timeline as separate lane
