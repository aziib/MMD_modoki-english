# Sequential PNG Output Specification/Implementation Notes

Updated: 2026-02-24

## Purpose

- Save MMD_modoki's current scene as sequential PNG on a frame-by-frame basis.
- Separate editing UI and export processing to prevent misoperation during output.

## Current Specification (User Perspective)

1. Start with `PNG Seq` button in playback panel.
2. When output destination folder is selected, sequential output starts immediately (no additional confirmation).
3. Output target frames:
   - Start: Current frame
   - End: Current totalFrames
   - Step: 1
4. Output resolution is fixed `1920x1080` (16:9).
5. Output FPS parameter is sent as `30` (as described below, currently not used for time progression in actual implementation).
6. Automatically create sequential subfolder directly under selected folder.
   - Example: `mmd_seq_20260224_153000_0-6543_s1`
7. Filename:
   - `mmd_seq_0000.png` format (minimum 4 digits depending on end frame).

## UI Behavior

- During output, main window gets `ui-export-lock` and edit UI becomes inoperable.
- Display `saved/total/frame` in progress overlay.
- Suppress the following during output:
  - Keyboard operations
  - Drag & drop loading
  - Window close (warn and do not allow closing)

## Implementation Architecture

### 1. Main UI (renderer)

- Create job request with `UIController.exportPNGSequence()`.
- Send result of `mmdManager.exportProjectState()`.
- Output request values:
  - `startFrame`, `endFrame`, `step`, `prefix`
  - `fps`, `precision`
  - `outputWidth`, `outputHeight`
- Receive output status via IPC event and update `ui-export-lock` and progress display.

### 2. Main process

- Receive job with `export:startPngSequenceWindow`.
- After sanitizing input values, issue `jobId` and hold job in Map.
- Start export-only renderer with `mode=exporter&jobId=...`.
- Hold activeCount per owner window and notify status/progress.
- PNG save is done with `file:savePngRgbaToPath`.
  - Rearrange RGBA -> BGRA and save with `nativeImage.toPNG()`.

### 3. Exporter renderer

- Detect `mode=exporter` at startup and skip normal UI initialization.
- Get job once with `takePngSequenceJob(jobId)`.
- Execute `runPngSequenceExportJob()`.
  - Create new `MmdManager`
  - Import project state
  - `seekTo(frame)` per frame -> get screenshot -> put in save queue
- Report progress to main UI at regular intervals.

## Save Processing (Speed Priority)

- Queue method with capture producer + save consumer.
- Current fixed values:
  - `maxQueueLength = 24`
  - `ioWorkerCount = 4`
- Save is executed in parallel with `savePngRgbaFileToPath()` to hide IO wait.

## Data Types

- `PngSequenceExportRequest`
  - `project`, `outputDirectoryPath`, `startFrame`, `endFrame`, `step`
  - `prefix`, `fps`, `precision`, `outputWidth`, `outputHeight`
- `PngSequenceExportState`
  - `active`, `activeCount`
- `PngSequenceExportProgress`
  - `jobId`, `saved`, `captured`, `total`, `frame`

## Current Limitations

1. `fps` / `precision` are in request but not used for time progression control.
2. Export window is configured for internal execution (`show: false`), basically hidden operation.
3. In high-load scenes, GPU capture side becomes bottleneck, and speed may not increase even if IO/GPU usage appears low.

## Future Improvement Candidates

1. Reflect `fps` to actual time progression/physics update step.
2. Clarify meaning of `precision` parameter and enable it.
3. Make output presets (1080p/1440p/4K, start/end range) selectable from UI.
4. Add profile measurement (ms display by capture/save) to visualize bottlenecks.
