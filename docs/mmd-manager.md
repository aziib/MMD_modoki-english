# MmdManager Implementation Notes

Target: `src/mmd-manager.ts`

## Role

- Babylon.js initialization (Engine/Scene/Camera/Light)
- MMD runtime management (PMX/PMD, VMD, camera VMD, audio)
- Physics (Ammo + babylon-mmd) initialization and enable/disable switching
- Post effect management (DoF, lens series, gamma, AA)
- Value application from UI layer and frame synchronization notification

## Current Rendering Pipeline

This project separates DoF and lens series.

1. Execute main DoF with `DefaultRenderingPipeline`
2. `LensRenderingPipeline` is used for highlight/edge blur purposes
3. Aberration is applied near the final stage with a custom `PostProcess` (`finalLensDistortionPostProcess`)
4. AA is applied last with `FxaaPostProcess`

Notes:

- The order of aberration and AA is fixed by `enforceFinalPostProcessOrder()`
- Re-attaching to always become `aberration -> AA`

## DoF / Lens Key Points

- Main DoF: `DefaultRenderingPipeline.depthOfField`
- `dofBlurLevelValue` default: `Medium`
- `dofFStopValue` default: `2.8`
- `dofLensSizeValue` default: `30`
- `dofAutoFocusToCameraTarget` is `true`
- `dofAutoFocusInFocusRadiusMm` is `6000` (about 6m)
- `dofNearSuppressionScaleValue` is `4.0`
- `dofAutoFocusNearOffsetMmValue` is `10000` (10m)

### Aberration

- FoV linkage is enabled (`dofLensDistortionFollowsCameraFov = true`)
- Neutral FoV: `30`
- Telephoto end FoV: `10` -> `-100%` side
- Wide-angle end FoV: `120` -> `+100%` side
- Influence: `dofLensDistortionInfluenceValue` (`0..1`, default `0`)
- LensRenderingPipeline side `distortion` is fixed at `0`
- Actual aberration application is performed in a custom final pass

## Post Correction

- `postEffectContrastValue` default: `1`
- `postEffectGammaValue` default: `2`
- Gamma is corrected with a dedicated PostProcess
- AA is switched with `antialiasEnabledValue` (default `true`)

## UI Reflection Status (2026-02-21)

Multiple items are hidden on the HTML side using `dof-row-hidden`.

- Hidden: Camera distance, DoF quality, DoF focus, DoF F-stop, front suppression, focal length inversion, DoF focal length
- Hidden: Contrast, aberration
- Shown: Gamma, aberration influence, lens blur, edge blur, contour line, etc.

For detailed UI items, refer to `docs/camera-implementation-spec.md`.

## UI/Input/Output Notes (2026-02-24)

- Toolbar loading is integrated into the `Load File` button
  - Supported: `pmx/pmd/vmd/vpd/mp3/wav/ogg`
  - VMD falls back in the order of `camera VMD -> motion` or `motion -> camera VMD` depending on situation and filename
- Drag & drop loading is handled by `UIController.setupFileDrop()`
  - Load `pmx/pmd` first, then `vmd/vpd`, then audio in sequence
  - Loading is blocked during PNG sequence output
- As Electron environment difference handling, drop file paths are resolved by `webUtils.getPathForFile(file)` in `preload.ts`
  - Traditional `File.path` is treated as fallback
- Shader panel has been removed from current UI (right panel hidden operation)
- When starting UI hidden mode, show a toast that says "press ESC to return"

## Notes

- `src/mmd-manager.ts` is CP932 system encoding, so be careful about character corruption when editing
- Since aberration is applied at the final stage, touching Lens side distortion parameters does not reflect in appearance (intentional specification)
