# Manual Test Checklist

Updated: 2026-02-23
Target scope:
- Timeline
- Bone editing
- VMD/VPD loading
- Playback/seek/physics

## 1. Prerequisites
- Able to load 2 PMX/PMD models
- Prepare arbitrary VMD, VPD, camera VMD
- Able to switch physics ON/OFF

## 2. Model/Timeline Basics
- [ ] Bone/morph rows appear in timeline after model load
- [ ] Switch to camera 1 row (`Camera`) when selecting `0: Camera`
- [ ] Return to bone row display when reselecting model
- [ ] Track content switches when changing active model

## 3. Selection Synchronization
- [ ] Bone panel dropdown synchronizes when selecting timeline row
- [ ] Timeline row synchronizes when selecting in bone panel
- [ ] Both bone panel and timeline synchronize when clicking 3D bone
- [ ] Only selected bone becomes emphasized color

## 4. Bone Operations
- [ ] Move slider not shown for rotation-only bones
- [ ] Can operate both Pos and Rot for movable bones
- [ ] Overlay follows after slider operation
- [ ] Model does not break with gizmo move/rotate

## 5. Key Editing
- [ ] Key added to current frame with `Register`
- [ ] When re-registering on same frame with existing key, latest value/interpolation is prioritized (later wins)
- [ ] Selected key or current frame key disappears with `Delete`
- [ ] Selected key moves 1f with `Alt+←/→`
- [ ] Can move normal frames with `←/→` when no selected key
- [ ] Actual value snapshot of position/rotation/distance/FoV is saved when registering camera key

## 5-2. Interpolation Editing
- [ ] Can drag edit bone interpolation (X/Y/Z/rotation)
- [ ] Can drag edit camera interpolation (X/Y/Z/rotation/distance/FOV)
- [ ] Can switch between `All` and single channel display with type dropdown
- [ ] Interpolation of back key (B key) displayed at intermediate frames
- [ ] No interpolation display after last key

## 6. VMD/VPD Loading
- [ ] Keys displayed in timeline after VMD load
- [ ] Pose key registered to current frame with VPD load
- [ ] Merged with existing keys with continuous VPD load
- [ ] Keys displayed in camera 1 row with camera VMD load

## 7. Playback/Stop
- [ ] Play/pause/stop works with audio
- [ ] Can play even without audio
- [ ] Auto stop when reaching end frame
- [ ] After auto stop, maintain end frame without returning to 0f

## 8. Seek/Physics
- [ ] Can move to start/end with Home/End
- [ ] No physics runaway (blow away) after Home/End
- [ ] Bone overlay hidden during playback
- [ ] Does not break with gizmo drag even when physics ON

## 9. Regression Notes (Optional Record)
- Test date:
- Used model:
- Used motion:
- Issues:
- Reproduction steps:

## 10. Project Save/Load
- [ ] Can save project with keyframes over 300f
- [ ] `keyframes` block at end in saved JSON
- [ ] Arrays output in packed format (`u8-b64` / `f32-b64` / `u32-delta-varint-b64`) in saved JSON
- [ ] Keys after 300f displayed in timeline after project load
- [ ] Plays normally after 300f on playback after project load
- [ ] Can load and play old format (number array) project

## 11. WebGPU Smoke
- [ ] Badge becomes `WebGPU` at startup (available environment)
- [ ] Automatically fallback to `WebGL2` in WebGPU unavailable environment
- [ ] PMX load/VMD playback/camera operation works on WebGPU
- [ ] No extreme darkening/whiteout when adjusting color temperature/gamma
- [ ] Appearance (brightness) does not break after project save/reload
