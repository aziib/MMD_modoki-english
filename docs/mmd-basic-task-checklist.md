# MMD Basic Function Implementation Status Checklist

Updated: 2026-02-24

## Evaluation Target

- `src/mmd-manager.ts`
- `src/ui-controller.ts`
- `src/renderer.ts`
- `src/preload.ts`
- `src/main.ts`
- `src/png-sequence-exporter.ts`
- `src/timeline.ts`
- `src/bottom-panel.ts`
- `src/types.ts`
- `src/index.css`
- `index.html`
- `docs/physics-task-list.md`
- `docs/physics-runtime-spec.md`

## 1. Model/Motion/Playback

- [x] PMX/PMD loading
- [ ] X model (`.x`) loading
- [x] Multiple model simultaneous holding
- [x] Active model switching
- [x] VMD motion loading (apply to active model)
- [x] VPD pose loading (register offset to selected frame)
- [x] Camera VMD loading
- [x] Audio loading (MP3/WAV/OGG)
- [x] Playback without audio (playable with keys only)
- [x] Play / pause / stop
- [x] Auto stop when reaching final frame (maintain end frame when stopped)
- [x] Frame seek
- [ ] Playback speed switching (not provided in current UI)

## 2.Viewport/Appearance

- [x] Ground display ON/OFF
- [x] Skydome display ON/OFF
- [x] Lighting adjustment (direction/intensity/color temperature)
- [x] Shadow adjustment (density/boundary softness)
- [x] AA ON/OFF
- [x] DoF/lens related adjustment (quality/focal length/F-stop etc.)
- [x] Model outline (edge width) adjustment
- [x] PNG output
- [ ] MP4 output (WebCodecs + mux)
- [x] UI hidden mode (in-app, return with ESC)

## 3. Timeline/Edit Operations

### 3-1. Current Implementation

- [x] Keyframe visualization (bones/morphs/camera)
- [x] Seek from timeline
- [x] Morph manual operation (first 30 items)
- [x] Keyframe editing (add/delete/1 frame move)
- [x] Bone direct operation (fader/gizmo) and keyframe registration
- [x] Camera keyframe editing
- [ ] VMD export
- [x] Project save/load (JSON)
- [x] Save/restore keyframe body and interpolation information to project
- [x] Non-destructive compression of project keyframe data (reversible)

### 3-2. UI Cooperation/Bone Visualization (2026-02-22 implementation reflected)

- [x] Display `0: Camera` in info panel, unify target switching for camera/model
- [x] Add model show/hide/delete to info panel
- [x] Add bone panel/morph panel (dropdown + fader)
- [x] Mutual synchronization of timeline selection/bone panel selection/3D bone selection
- [x] Timeline display based on PMX bone order (`All Parents` at head)
- [x] Bone display excluding PMX visible flag/physics-only bones
- [x] Color change only for selected bone (front display)
- [x] Auto hide bone display during playback
- [x] Display operation channels according to bone type (rotation only/move+rotate)

### 3-3. MMD Compatibility Specification Tasks (2026-02-22 investigation reflected)

- [x] Unify time axis with 30fps fixed frame numbers (UI/internal API)
- [x] Explicit display of key types (Bone/Morph/Property/Camera)
- [x] Bone interpolation editing (4 channels independent: X/Y/Z/rotation)
- [x] Camera interpolation editing (6 channels independent: X/Y/Z/rotation/distance/FOV)
- [x] 0..127 range editing and holding of interpolation parameters
- [ ] Edit/preview Property (display/IK) as step interpolation
- [x] Unify rule for same track same frame conflict as "later wins"
- [ ] MMD actual machine comparison test for rotation interpolation (check difference with representative VMD)
- [ ] Output interpolation/Property information without omission on VMD export

### 3-4. UI/Input/Output Modification (2026-02-24 reflected)

- [x] Integrate toolbar loading function to `Load File` button (PMX/PMD/VMD/VPD/audio)
- [x] Drag & drop loading (multiple files, priority-based sequential processing)
- [x] DnD path resolution using Electron `webUtils.getPathForFile` (environment difference countermeasure)
- [x] Always hide shader panel (remove toggle on UI)
- [x] Show ESC return guide toast when starting UI hidden

## 4. Physics

- [x] Ammo wasm initialization and fallback on failure
- [x] Physics ON/OFF switching
- [x] UI adjustment of gravity acceleration/direction
- [x] Model generation with `disableOffsetForConstraintFrame: true`
- [x] Rigid body reinitialization at head/end jump (suppress inertia runaway)
- [ ] Document and implement operation specification for rigid body modes 0/1/2
- [ ] Switching equivalent to `disableBidirectionalTransformation`
- [ ] Rigid body/constraint debug visualization
- [ ] Stability verification on seek/playback speed change
- [ ] Regression test preparation

## 5. Model Format Extension (Babylon.js Editor Cooperation)

- [ ] Investigation and priority determination of Babylon.js Editor supported 3D formats
- [ ] glTF/GLB loading (static mesh)
- [ ] glTF/GLB loading (skin/animation)
- [ ] OBJ loading
- [ ] STL loading
- [ ] `.babylon` loading
- [ ] Point cloud/Gaussian splat loading (`.ply` / `.splat` / `.spz` / `.sog`, low priority)
- [ ] Coordinate system/unit system normalization per format (Y-up/Z-up, scale)
- [ ] Material/texture difference absorption policy definition per format
- [ ] Edit restriction UI for timeline-excluded formats (read-only guard)

## 6. WebGPU / WGSL

- [x] Add WebGPU startup path (fallback to WebGL2 when unavailable)
- [x] Confirm rendering stability on WebGPU (model/shadows/post effects)
- [x] Define WGSL support policy for custom shaders (including GLSL compatible operation)
- [ ] WGSL conversion of necessary parts (shadows/outline/post effects priority)
- [ ] WebGL2 vs WebGPU performance comparison measurement (FPS/VRAM/startup time)
- [x] Organize known limitations of WebGPU path and reflect to settings UI

## 7. Distribution Build (Release Preparation)

- [ ] Distribution target definition (Windows priority, macOS/Linux optional)
- [ ] Review configuration for `electron-forge make` (maker configuration, artifact format)
- [ ] App information preparation (app icon, ProductName, Version, description)
- [ ] Confirm startup dependency bundling (wasm/model loader/preload)
- [ ] Signature/distribution policy organization (certificate, SmartScreen countermeasures)
- [ ] Installation/startup smoke test in clean environment
- [ ] Distribution procedure documentation (build procedure, known limitations, update procedure)

## 8. Extension Cooperation/Usability

- [ ] WebCodex API cooperation foundation (connection/authentication/error handling)
- [ ] Basic operations via WebCodex API (file loading/playback control/seek)
- [ ] MIDI controller input support (Web MIDI API / Electron environment confirmation)
- [ ] MIDI mapping function (playback series/camera series/morph/bone series)
- [ ] Shortcut key customization (action assignment)
- [ ] Shortcut setting persistence (save/load as user settings)
- [ ] UI multilingual support (i18n dictionary/key operation/runtime switching UI)
- [ ] Theme support (light mode / dark mode switching + persistence)

## Recent Tasks (Priority as MMD Basic Functions)

- [x] Camera editing (position/rotation/distance/FOV) and 6ch interpolation editing
- [x] Camera keyframe editing (save actual value snapshot on add)
- [ ] Property (display/IK) step editing and timeline reflection
- [ ] MMD comparison verification for rotation interpolation (measure difference with representative motion)
- [ ] VMD export
- [ ] MP4 output (WebCodecs + mux)
- [x] Project save/load
- [ ] Digest incomplete physics items along `docs/physics-task-list.md`
- [ ] X model (`.x`) loading PoC implementation
- [ ] First Babylon.js Editor format support (candidate: glTF/GLB)
- [ ] OBJ/STL loading support (minimum static placement path)
- [ ] WebCodex API cooperation PoC (connection confirmation + 1 operation)
- [ ] MIDI controller support PoC (1 device/few knobs)
- [ ] Shortcut key custom setting UI
- [ ] UI multilingual support (start from ja/en)
- [ ] Light mode / dark mode switching support
- [x] WebGPU fallback startup path implementation
- [ ] Establish distribution-oriented build procedure (Windows)
