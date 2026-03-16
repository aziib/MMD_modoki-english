# WebM Output Current Specification / Implementation
Updated: 2026-03-13

## 1. Overview
- Save `.webm` from `Output > WebM video`
- Output fps is `24 / 30 / 60`
- When `With audio` is ON, mux loaded audio
- Codec selection UI is normally not shown
- Internal default codec is `VP9`
- During output, lock main UI and show simplified progress in busy overlay

## 2. UI
Target files:

- `index.html`
- `src/ui-controller.ts`
- `src/index.css`

Current items in output panel:

- Aspect ratio
- Long side
- Width / Height
- FPS
- `With audio`
- `PNG image`
- `WebM video`

Notes:

- `PNG Seq` is removed from UI
- Codec selection UI is also removed
- New defaults use `VP9` internally

## 3. Timeline Basis
MMD timeline is handled on 30fps basis.

- `timelineFrameCount = endFrame - startFrame + 1`
- `totalOutputFrames = round((timelineFrameCount / 30) * outputFps)`

This means:

- For 30fps output, timeline 1 frame = video 1 frame
- For 60fps output, frame count increases but playback time is maintained
- For audio-enabled output, align video/audio length

## 4. Composition

### Main UI renderer
Target:

- `src/ui-controller.ts`

Role:

- Save output UI input to `ProjectOutputState`
- Build `WebmExportRequest` and send to main process
- Manage background export lock and busy overlay

### Main process
Target:

- `src/main.ts`
- `src/preload.ts`
- `src/types.ts`

Role:

- Start/accept WebM export job
- Generate hidden exporter window
- Relay progress/state to owner window
- Streamed save
- Close exporter window and release UI lock on completion

### Exporter renderer
Target:

- `src/renderer.ts`
- `src/webm-exporter.ts`

Role:

- Create fresh `MmdManager` on hidden window
- Import project state to isolated scene
- Execute frame capture / encode / save
- Return `finishWebmExportJob(jobId)` to main on completion

## 5. Output Procedure
1. `MmdManager.create(canvas)` in hidden exporter window
2. `importProjectState(project, { forExport: true })`
3. `setTimelineTarget("camera")`
4. `pause()`, `setAutoRenderEnabled(false)`
5. `seekTo(startFrame)`
6. Determine codec / bitrate
7. Decode / slice audio if necessary
8. Generate `Output + WebMOutputFormat + StreamTarget`
9. Render / capture / encode per frame
10. `close -> finalize -> finishWebmExportJob`

## 6. Capture Path
Currently uses the following for stability priority.

- reusable `RenderTargetTexture`
- `readPixels()`
- `VideoSample(RGBA)`

Not adopted:

- `canvas -> VideoSample`
- `ImageBitmap -> 2D canvas -> VideoSample`

These had environments where black frames appeared, so currently not used.

## 7. Audio Track
Target:

- `src/webm-exporter.ts`

Specification:

- Only mux when `With audio` is ON and audio is loaded
- Do not play sound on exporter scene side
- Decode original audio file separately and use

Audio codec:

- Priority: `opus`
- Fallback: `vorbis`

Audio bitrate:

- mono: `128 kbps`
- stereo and above: `192 kbps`

## 8. codec / bitrate
Target:

- `src/webm-exporter.ts`

Video codec:

- Internal default: `VP9`
- At runtime, prioritize `prefer-hardware`
- Fallback to `no-preference` when not supported

Default bitrate:

- 1080p30: `8 Mbps`
- 1080p60: `12 Mbps`
- 1440p30: `16 Mbps`
- 1440p60: `24 Mbps`
- 4K30: `35 Mbps`
- 4K60: `53 Mbps`

Notes:

- `keyFrameInterval` is currently `5`
- This value is still being adjusted

## 9. Save Method
Target:

- `src/webm-exporter.ts`
- `src/main.ts`

Save uses streamed save.

1. Exporter calls `beginWebmStreamSave(filePath)`
2. Chunks come out from `StreamTarget`
3. Pass to main process with `writeWebmStreamChunk(saveId, bytes, position)`
4. After close, `finishWebmStreamSave(saveId)`
5. On error, `cancelWebmStreamSave(saveId)`

Since completed WebM is not transferred in bulk IPC at the end, stall at completion is reduced.

## 10. Progress Display
Target:

- `src/renderer.ts`
- `src/ui-controller.ts`

phase:

- `initializing`
- `loading-project`
- `checking-codec`
- `opening-output`
- `encoding`
- `closing-track`
- `finalizing`
- `finishing-job`
- `completed`
- `failed`

UI only shows simplified display.

- phase
- `encoded / total`
- current frame

Detailed measurement values and internal logs are normally not output to UI.

## 11. Startup Optimization
- Use `importProjectState(..., { forExport: true })`
- Reduce `waitForAnimationFrames(3)` to `1`
- Reduce unnecessary UI synchronization from active model switching in export import

## 12. Known Constraints
- Capture depends on `readPixels()`, so there is still room for speed improvement
- Not HDR output
- Codec UI is hidden, but internal selection assumes `VP9`
