# Interpolation Curve Specification and Implementation Summary (Current)

Updated: 2026-02-23
Target:
- `src/ui-controller.ts`
- `src/mmd-manager.ts`
- `src/types.ts`
- `index.html`
- `src/index.css`

## 1. Specification Policy (MMD-aligned)

- Interpolation is cubic Bezier. Start point `(0,0)` / end point `(127,127)` fixed.
- Control points are 4 values of `(x1,y1),(x2,y2)`, each value is `0..127` integer.
- Interpolation for interval `A -> B` uses "interpolation value held by arrival side key B".
- Bones are 4ch independent: `X / Y / Z / rotation`
- Camera is 6ch independent: `X / Y / Z / rotation / distance / FoV`

## 2. Interpolation Display Rules in Timeline

- If key exists at selected frame, display that key's interpolation.
- Intermediate frames (`A < f < B`) display interpolation of back key `B` effective for interval.
- Do not display after last key (strictly MMD-aligned).
- Camera is 1 row display, but internally holds and draws 6ch simultaneously.
- For bone rotation-only tracks, Pos series displayed as non-editable channels.

## 3. Interpolation Panel UI (Current)

- Interpolation panel is "graph priority" layout.
- Type is dropdown:
  - `All` (default): All channels superimposed display
  - Single channel display: Can display only 1ch as edit preparation
- Color coding:
  - X: Red series
  - Y: Green series
  - Z: Blue series
  - Rot: Amber series
  - Dist: Cyan series
  - FoV: Pink series
- Dotted line display means "displayed but not edit target (available=false)".

## 4. Edit Implementation (Drag)

- Drag interpolation points to update control points.
- Values are clamped + rounded to `0..127` each time.
- Reflection to edit target array is direct write via `interpolationChannelBindings`.
- Regenerate runtime animation at drag end and seek to current frame to synchronize appearance.

## 5. Save Implementation on Key Registration (This Reflection)

- With `addKeyframeAtCurrentFrame()`, take snapshot of "currently displayed interpolation curve" before registration.
- After `mmdManager.addTimelineKeyframe()` success, insert the following to new key position:
  - Frame number array
  - Value array (position/rotation/distance/FoV)
  - Interpolation array (bone 4ch / camera 6ch)
- Bone values use current posture of `getBoneTransform()`, rotation converted to Quaternion block and saved.
- Camera values save actual values of `getCameraPosition()/Rotation()/Distance()/Fov()`.
- With this processing, "interpolation at key registration" remains in source animation side that is project save target.

## 6. Relationship with Project Save

- Project save serializes source animation with `serializeModelAnimation` / `serializeCameraTrack`.
- Therefore, interpolation edit/key registration save is reflected in project JSON.
- Arrays are saved reversibly in packed format (`u8-b64`, `f32-b64`, `u32-delta-varint-b64`).

## 7. Unimplemented/Future

- Property (display/IK) step edit UI
- VMD export (including interpolation reconstruction)
- Automated difference verification of rotation interpolation with MMD actual machine
