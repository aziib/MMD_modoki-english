# UI and Operation Flow

## Screen Layout

- Top: File load, project save/load, ground/sky/AA/physics toggles, status display, FPS display
- Left: Playback panel + timeline panel
- Center: 3D viewport
- Right: Effect panel
- Bottom: Info / Interpolation / Bone / Morph / Camera / Lighting / Accessory / Output

Notes on left/right panels:

- Left panel width is adjustable
- Right effect panel width is also adjustable
- Model selection in `Info > Target` and `Effect panel > Target` are mutually synchronized

Layout body is managed in `index.html`, appearance in `src/index.css`.

## UI Control Class

`src/ui-controller.ts` handles the following.

- Button click and shortcut binding
- Receiving callbacks from `MmdManager`
- Reflecting current frame to timeline
- Updating selectors for multiple models and switching active
- Status display and toast display
- Dynamic UI text updates using dictionary keys from `src/i18n.ts` (internal implementation is `i18next`)

`src/i18n.ts` provides the following.

- Locale management (`ja` / `en`)
- Translation resolution and fallback via `i18next`
- String resolution via `t(key, params)`
- DOM reflection to `data-i18n` / `data-i18n-title` attributes
- Runtime switching via `window.mmdI18n.setLocale("ja" | "en")` (for development)

## Bottom Panel Notes

- `Info`:
  - Model statistics display (vertices/bones/morphs)
  - Switch active model with `Target` selector
- `Interpolation`:
  - Interpolation curve editing for selected track
- `Bone`:
  - Position/rotation adjustment for selected bone
- `Morph`:
  - Morph slider adjustment per display frame
- `Lighting`:
  - Azimuth / elevation
  - Light intensity / ambient light
  - Shadow range / boundary width
  - Shadows are always ON in UI operation
  - Shadow density default is `0.0`, usually hidden in UI
- `Camera`:
  - Viewpoint (left/front/right)
  - FOV / distance
  - Startup distance default is `50m`
  - DoF-related items are not shown (operations are consolidated in effect panel)
- `Accessory`:
  - Select `.x` accessory with `Target` selector
  - `Show/Hide`, `Delete`
  - `Parent` (World or model), `Bone` (model center or bone name)
  - Position (X/Y/Z), rotation (Rx/Ry/Rz), scale (Si)
  - When loading `.x`, initial scale is taken as `10x`
  - Texture references are resolved considering both UTF-8 and Shift-JIS
  - MMD-compatible composite specifications like `baseTexture*sphere(.sph/.spa)` are decomposed and read
- `Output`:
  - Aspect ratio, resolution preset, width/height, quality
  - FPS dropdown (`24 / 30 / 60`)
  - `With audio` checkbox
  - `PNG image` / `WebM video`
  - WebM outputs `currentFrame -> totalFrames`
  - If `With audio` is ON and audio is loaded, WebM muxes audio
  - Codec UI is hidden. Internal default is `VP9`
  - During WebM output, background export lock is engaged, and phase and `encoded / total` are briefly displayed in busy overlay

## Effect Panel Notes

- When model is selected in `Info > Target`:
  - Preset assignment per material
- When `Camera` is selected in `Info > Target`:
  - Post effects
    - ImageProcessing series: Gamma / Vignette
    - DefaultRenderingPipeline series: Bloom / Chroma / Grain / Sharpen / DoF
    - Color correction series: LUT (3dl presets + intensity)
    - Scene series: Fog (ON/OFF, density, transparency, color R/G/B)
    - Others: Distortion / EdgeBlur / Edge
  - `ToneMap` is placed at the bottom of the effect panel
  - As a provisional operation prioritizing stability, `Contrast / Exposure / Dither / Curves / Glow / Motion Blur / SSR / VLight` are UI-hidden
  - SSR is always OFF via UI (intensity 0 / enabled false)
  - Bloom is a composite item as `ON/OFF + Weight + Threshold + Kernel`
  - BloomTh UI operation is inverted so that moving right expands the emission range
  - DoF-related controls including `front/back correction` are consolidated in the effect panel
  - Fog is fixed at `Exp2`, with start/end internally fixed at `100 / 300`

## Viewpoint Control Priority

- During playback:
  - Prioritize camera VMD and reflect to viewport
- During stop:
  - Only when `Info > Target = Camera`, reflect camera VMD to viewport
  - When `Info > Target != Camera`, prioritize mouse operations and do not overwrite viewpoint with camera VMD
- Mouse panning:
  - `Middle button drag` and `Shift + right drag` have the same pan behavior

## Timeline Rendering Mechanism

`src/timeline.ts` has a 3 Canvas structure.

- static: Track background + keyframe points
- overlay: Ruler + playback head
- label: Left label column

In terms of performance, the following are adopted.

- Separate drawing with `requestAnimationFrame`
- Draw only visible frame range
- Scan only relevant keyframes with binary search
- When loading camera VMD, display `Camera` lane in a separate row (simultaneous display with model tracks)

## Shortcuts

- `Space`: Play/pause
- `Home`: Jump to start
- `End`: Jump to end
- `← / →`: Move 1 frame
- `Shift + ← / →`: Move 10 frames
- `Ctrl + O`: Load PMX/PMD
- `Ctrl + M`: Load VMD
- `Ctrl + Shift + M`: Load camera VMD
- `Ctrl + Shift + A`: Load audio
- `Ctrl + Shift + S`: PNG output
