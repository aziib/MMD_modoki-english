# Camera Implementation/Specification Notes (Current)

Target:

- `src/mmd-manager.ts`
- `src/ui-controller.ts`
- `index.html`

Updated: 2026-02-21

## Camera Body

- Type: `ArcRotateCamera`
- Initial values: `alpha=-PI/2`, `beta=PI/2.2`, `radius=30`, `target=(0,10,0)`
- `lowerRadiusLimit=2`, `upperRadiusLimit=100`
- `wheelDeltaPercentage=0.01`

## Public API (Main)

- `getCameraFov()` / `setCameraFov(degrees)`
- `getCameraDistance()` / `setCameraDistance(distance)`
- `setCameraView("left" | "front" | "right")`
- `setCameraTarget(x, y, z)`
- `getCameraRotation()` / `setCameraRotation(xDeg, yDeg, zDeg)`

## Camera UI (Shown)

Currently shown camera panel operators:

- Viewpoint buttons: Left / Front / Right
- FOV: `10..120`
- DoF ON/OFF
- Front correction: `0..20000` mm
- DoF lens: `1..4096`

## Camera UI (Hidden)

Items hidden with `dof-row-hidden`:

- Distance (`cam-distance`)
- DoF quality
- DoF focus
- DoF F-stop
- Front suppression
- Focal length inversion
- DoF focal length

Notes:

- Internal logic is effective even if UI hidden, and initial values are maintained

## DoF Internal Specification

- Main DoF is `DefaultRenderingPipeline.depthOfField`
- `dofBlurLevel` default: Medium
- `dofFStop` default: 2.8
- `dofLensSize` default: 30
- Auto focus enabled (follows gaze point)
- Focus band radius: 6000mm (6m)
- Front correction default: 10000mm (10m)
- Front suppression default: 400% (internal value 4.0)

## FoV Linkage

### DoF Focal Length

- Auto convert from FoV to focal length (sensor width 36mm)
- Inversion flag implemented (currently UI hidden)

### Aberration

- FoV linkage is enabled
- FoV=30 is neutral (0%)
- FoV on wide-angle side goes +100% direction, telephoto side goes -100% direction
- Actual application is value multiplied by "aberration influence (0..100%)"

## Processing Order (End)

The order at the end is fixed:

1. Final aberration post process
2. FXAA

Maintain `aberration -> AA` with `enforceFinalPostProcessOrder()`.
