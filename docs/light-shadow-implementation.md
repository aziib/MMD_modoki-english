# Light/Shadow Implementation Notes (Toon Separation + Flat Light)

This document summarizes the current implementation of "light face/shadow face separation" and "light color/shadow color control".  
Target code is mainly the following.

- `src/mmd-manager.ts`
- `src/ui-controller.ts`
- `index.html`
- `src/index.css`

## 1. Overall Composition

Implementation flow is the following 3 stages.

1. UI fader (`index.html` + `ui-controller.ts`)
2. Runtime value holding (`MmdManager` getter/setter)
3. MMD material toon color + shader patch (`mmd-manager.ts`)

With `MmdManager.patchMmdToonLightSeparationShader()`, replace babylon-mmd fragment code,  
separating "shadow side multiplication" and "light side addition" (both GLSL/WGSL).

## 2. UI Parameters and Meaning

### Items Used in Light Mode

- `light-azimuth`, `light-elevation`
  - Directional light direction
- `light-color-r/g/b` (0..255)
  - Normalized to `0..2` in `ui-controller.ts` (`/127.5`)
  - Pass to `setLightColor(r,g,b)`

### Items Used in Shadow Mode

- `light-intensity` (label is "影の強さ" but actually dirLight intensity)
  - `0..200` -> `0..2`
- `light-shadow`
  - `shadowDarkness` (0..1)
- `light-shadow-color-r/g/b`
  - 0..255 -> 0..1 and pass to `setShadowColor`
- `light-toon-shadow-influence`
  - Toon shadow influence (0..1)

### Items Hidden but Values Effective

- `light-flat-strength` (0..10% / actual value 0..0.1)
- `light-flat-color-influence` (0..1)
- `light-self-shadow-softness`
- `light-occlusion-shadow-softness`

These are always hidden with `light-row--always-hidden`.

## 3. Assignment to Toon Color

Reflected to materials with `applyToonShadowInfluenceToMeshes()`.

- `toonTextureMultiplicativeColor = (lightR, lightG, lightB, lightFlatStrength)`
- `toonTextureAdditiveColor = (shadowR, shadowG, shadowB, toonShadowInfluence)`

Actual clamping:

- Light color scale: `0..2` (`clampLightColorScale`)
- Shadow color/Toon influence: `0..1` (`clampColor01`)
- Light face intensity: `0..0.1` on setter side

## 4. Separation Logic on Shader Side

With `patchMmdToonLightSeparationShader()`, inject the following (conceptual formula).

1. Mask generation
   - `selfMask = smoothstep(..., info.ndl)`
   - `occlusionMask = smoothstep(..., shadow)`
   - `litMask = smoothstep(..., selfMask * occlusionMask)`
   - `shadowMask = 1 - litMask`

2. Shadow face (multiplication)
   - `toonShadowBand = mix(shadowTint, toonRaw, toonInfluence)`
   - `shadowTerm = info.diffuse * mix(1, toonShadowBand, shadowMask)`
   - `diffuseBase += shadowTerm`

3. Light face (addition)
   - `lightBoost = max(lightTint - 1, 0)`
   - `toonFlatLightMask = litMask * f(flatStrength, lightBoostEnergy)`
   - `toonFlatLightColor = lightBoost * g(flatStrength, lightFlatColorInfluence)`
   - `color += toonFlatLightColor * toonFlatLightMask` in `CUSTOM_FRAGMENT_BEFORE_FOG`

Points:

- Shadow is "multiplication to base color".
- Light is "addition to separate layer".
- Therefore, shadow face and light face can be handled independently.

## 5. Lighting Items Saved/Restored

Project save (`serializeProject`) includes the following.

- `azimuth`, `elevation`
- `intensity`, `ambientIntensity`, `temperatureKelvin`
- `lightColor`
- `lightFlatStrength`, `lightFlatColorInfluence`
- `shadowColor`, `toonShadowInfluence`
- `shadowEnabled`, `shadowDarkness`
- `shadowEdgeSoftness` (old compatibility)
- `selfShadowEdgeSoftness`, `occlusionShadowEdgeSoftness`

On restore (`restoreProject`), also supports fallback from old `shadowEdgeSoftness`.

## 6. UI Display Control Notes

`applyLightMode()` in `ui-controller.ts` adds/removes `light-row--hidden` to `.light-row--light` / `.light-row--shadow`.  
Therefore, cases where `hidden` attribute alone causes re-display exist, and for items to always hide,  
use `.light-row--always-hidden { display:none !important; }`.

## 7. Notes on Shader Modification (WGSL)

Errors that occurred frequently in the past:

- `return FragmentOutputs` and function return type mismatch
- Reassignment to `let` variables (WGSL `let` is immutable)
- Redeclaration of same name variables
- Reference to undefined variables (`toonFlatLightMask`, `emissiveColor` etc.)

Safe modification procedure:

1. Add same meaning changes to both GLSL/WGSL
2. Prevent missing variable declaration additions in `CUSTOM_FRAGMENT_MAIN_BEGIN`
3. Always match reference target variables on `CUSTOM_FRAGMENT_BEFORE_FOG` side
4. After changes, confirm by passing model load on WebGPU
