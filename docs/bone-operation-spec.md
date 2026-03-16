# Bone Operation Specification and Implementation Notes

Updated: 2026-02-22
Target implementation:
- `src/mmd-manager.ts`
- `src/ui-controller.ts`
- `src/bottom-panel.ts`
- `src/timeline.ts`
- `src/types.ts`
- `index.html`

## 1. Purpose
- As MMD editing basics, visualize bones of selected model and enable selection/operation from UI/3D.
- Synchronize selection state of timeline, bone panel, and 3D overlay.
- Handle only "bones that should be displayed as edit targets" according to PMX metadata.

## 2. User Specification (Current)

### 2-1. Bone Display
- Bones are drawn as 2D overlay on 3D view (separate layer from model drawing).
- Automatically hide bone display during playback.
- Display only when targeting model, hide when targeting camera.
- Overlay transparency is 50%.

### 2-2. Bone Selection
- No matter where selected from, selection is mutually synchronized.
- Timeline bone track selection
- Bottom bone panel dropdown selection
- Click selection on 3D overlay
- Selected bone is emphasized by changing color.
- When overlapping, draw selected bone last and bring to front.

### 2-3. Bone Operation
- From bottom bone panel, can edit move/rotate of selected bone with sliders.
- Can move/rotate with 3D handles (Babylon.js Gizmo).
- Only display/enable channels that can be operated according to bone type.
- Immovable bones: Rotation only
- Non-rotatable bones: Move only
- Both immovable: No operation UI

### 2-4. Timeline Cooperation
- Bone rows are arranged based on model bone order.
- `root` category (e.g., All Parents) is always displayed as head group.
- Keyframe editing provides `register/delete/←→(1 frame move)`.

## 3. Data Specification

### 3-1. ModelInfo / BoneControlInfo
- `ModelInfo.boneNames` is list of display target bones.
- `ModelInfo.boneControlInfos` holds operation attributes per bone.
- `BoneControlInfo.movable` / `BoneControlInfo.rotatable` is operability.
- `BoneControlInfo.isIk` / `BoneControlInfo.isIkAffected` used for display style judgment.

Definition: `src/types.ts`

### 3-2. Display Target Bone Extraction Rules
Extract bones from `mmdMetadata` at model load.
- Adopt only bones with PMX visible flag set
- Exclude bones assigned to physics rigid bodies (except follow mode)
- Exclude duplicate names
- Give separate flags to IK bones/IK affected bones

Implementation: `src/mmd-manager.ts:1785`

## 4. Implementation Composition

### 4-1. MmdManager (Core)

#### Selection State/Target Switch
- `timelineTarget` manages `model | camera`.
- Update selected bone with `setBoneVisualizerSelectedBone()`.
- Reevaluate gizmo replacement on selection update.

Implementation: `src/mmd-manager.ts:432`, `src/mmd-manager.ts:443`

#### Bone Overlay Generation
- Build parent-child pairs prioritizing runtime bones.
- Fallback from Babylon `Skeleton` if not available.
- Apply mesh world matrix as needed to absorb model differences and correct.

Implementation: `src/mmd-manager.ts:619`

#### Bone Drawing
- Reproject and draw every frame with `updateBoneVisualizer()`.
- Lines drawn as "needle" shape.
- Markers use `circle/square` separately.
- Normal colors use blue series (normal bones) and orange series (IK / IK affected) separately.
- Selected color is red series.
- Draw in non-selected -> selected order to front selected.

Implementation: `src/mmd-manager.ts:759`, `src/mmd-manager.ts:992`, `src/mmd-manager.ts:1022`, `src/mmd-manager.ts:1049`

#### Click Picking
- Receive on `render-canvas` pointer events.
- Treat as click only if movement amount between press/release positions is small.
- Select nearest bone within 14px radius from projected point group.

Implementation: `src/mmd-manager.ts:299`, `src/mmd-manager.ts:960`, `src/mmd-manager.ts:1383`

#### Gizmo Operation
- Create one `GizmoManager` and attach `proxy node` to selected bone.
- Turn position/rotation gizmo ON/OFF according to bone attributes.
- Temporarily turn physics OFF during gizmo drag, return to original state on completion.
- Inverse convert proxy world conversion to bone local and reflect.

Implementation: `src/mmd-manager.ts:452`, `src/mmd-manager.ts:508`, `src/mmd-manager.ts:543`, `src/mmd-manager.ts:1528`

#### Slider Operation API
- Get current offset value with `getBoneTransform()` (returns rotation in deg).
- `setBoneTranslation()` reflects with rest position + offset.
- `setBoneRotation()` reflects with Euler(deg) -> Quaternion.
- After reflection, force matrix update with `invalidateBoneVisualizerPose()`.

Implementation: `src/mmd-manager.ts:3253`, `src/mmd-manager.ts:3278`, `src/mmd-manager.ts:3291`, `src/mmd-manager.ts:3304`

### 4-2. BottomPanel (Bone Panel)
- Build dropdown when receiving model info.
- Dynamically generate sliders according to `movable/rotatable` per bone.
- Slider input immediately reflects to `setBoneTranslation/Rotation`.
- Notify selection change externally with `onBoneSelectionChanged`.

Implementation: `src/bottom-panel.ts:41`, `src/bottom-panel.ts:146`, `src/bottom-panel.ts:229`

### 4-3. UIController (Synchronization Control)
- When timeline selection changes: Update bone panel selection and bone visualizer selection.
- When bone panel selection changes: Update timeline corresponding row selection and bone visualizer selection.
- When 3D pick: Reflect to bone panel and update timeline selection.

Implementation: `src/ui-controller.ts:336`, `src/ui-controller.ts:797`, `src/ui-controller.ts:1201`, `src/ui-controller.ts:1214`, `src/ui-controller.ts:1228`

### 4-4. Timeline (Bone Track Selection)
- Tracks identified by `name + category`.
- Can select from outside with `selectTrackByNameAndCategory()`.
- Row click selects track, key near click selects frame.

Implementation: `src/timeline.ts:236`, `src/timeline.ts:528`, `src/timeline.ts:558`

## 5. Appearance Specification (MMD-aligned)
- Separate color systems for IK/normal (orange/blue).
- For rotation only/IK affected, use circle as basic shape; for movable/IK body, use square.
- Give inner fill to marker center to ensure visibility.
- Expand line width/marker size when selected.

Implementation: `src/mmd-manager.ts:992`, `src/mmd-manager.ts:1057`

## 6. Current Constraints
- Keyframe "register" is currently centered on frame number management, channel value full edit UI is not implemented.
- Interpolation curve (X/Y/Z/rotation) editing is not implemented.
- Bone name category judgment (root/semi-standard) is currently hardcoded-oriented.

Related: `docs/mmd-basic-task-checklist.md`

## 7. Change Checklist
- If bone extraction conditions changed, confirm display consistency of bone panel/timeline/overlay.
- If gizmo reflection changed, confirm rotation/movement does not break with parent-child hierarchy bones.
- If click judgment changed, confirm misselection rate in dense areas (face/fingers).
- If playback control changed, confirm overlay/gizmo not displayed during playback.
