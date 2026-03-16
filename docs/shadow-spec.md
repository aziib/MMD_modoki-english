# Shadow Specification and Implementation

This document summarizes PMX shadow-related specifications and `MMD_modoki` implementation policies.

Related:
- [Light/Shadow Implementation Notes (Toon Separation + Flat Light)](./light-shadow-implementation.md)
- [Shadow Quality Improvement Investigation Notes](./shadow-quality-investigation.md)

## PMX Material Flags (Shadow Related)

PMX material flags have bits related to shadows.

- `0x02`: Ground Shadow (ground shadow)
- `0x04`: Draw Shadow (project to self-shadow shadow map)
- `0x08`: Receive Shadow (receive self-shadow)

Notes:
- PMX does not have dedicated flags to separate "cast shadow only on other models / cast shadow only on self model".
- Therefore, in actual operation, behavior is determined by renderer-side design (how shadow maps are created).

## Implementation Policy

In `src/mmd-manager.ts`'s `loadPMX`, shadow settings are determined in the following flow.

1. Register all model meshes uniformly as `shadow caster`
2. Set all model meshes uniformly to `receiveShadows = true`
3. Also set ground to `receiveShadows = true` to always enable shadows between models/floor

Notes:
- Current directional light shadows are not limited by PMX material flags.
- The purpose is to reliably cast shadows on "other models" and "floor polygons".
- `preserveSerializationData: true` remains on the loader side, but is not used in current shadow determination.

## Implementation Points

- When loading models, for all meshes
  - `shadowGenerator.addShadowCaster(...)`
  - `mesh.receiveShadows = true`
- For ground
  - `ground.receiveShadows = true`

## Shadow Color (Toon Color)

- Model material's `toonTexture` uses the value set by the PMX loader as is.
- Overwriting to a common grayscale ramp as before is not performed.
- After loading, only `toonTexture` sampling is set to `BILINEAR` to reduce boundary jaggedness.
- Materials without `toonTexture` follow babylon-mmd's default behavior (`ignoreDiffuseWhenToonTextureIsNull`).

## Shadow Generation Settings

Directional light + `ShadowGenerator` settings follow the following policy.

- Map resolution: `min(8192, GPU limit)`
- Filter: `PCF` (`usePercentageCloserFiltering = true`)
- Quality: `QUALITY_HIGH`
- Interpolation: `Contact Hardening` (`useContactHardeningShadow = true`)
  - `contactHardeningLightSizeUVRatio = 0.035`
- Ground contact adjustment
  - `bias = 0.00015`
  - `normalBias = 0.0006`
  - `frustumEdgeFalloff = 0.2`
- Transparent material support
  - `transparencyShadow = true`
  - `enableSoftTransparentShadow = true`
  - `useOpacityTextureForTransparentShadow = true`
- Shadow projection range (cover entire ground)
  - `dirLight.shadowFrustumSize = 220`
  - `dirLight.shadowMinZ = 1`
  - `dirLight.shadowMaxZ = max(500, shadowFrustumSize * 6)`
  - `dirLight.autoUpdateExtends = true`
  - `dirLight.autoCalcShadowZBounds = true`
  - Light source position distance: `dist = max(90, shadowFrustumSize * 0.35)` in `setLightDirection`

## Relationship with UI

Shadow settings are controlled in the lighting UI separately from material flags.

- `index.html`
  - `#light-shadow` (shadow darkness, currently hidden)
  - `#light-shadow-frustum-size` (shadow range)
  - `#light-shadow-softness` (boundary width / contact hardening)
- `src/ui-controller.ts`
  - Apply `setShadowEnabled(true)` at startup (always ON in UI)
  - Update `shadowFrustumSize`
  - Update `shadowEdgeSoftness`

Currently, it is always ON in UI operation, and mainly shadow range and boundary width are adjusted.
`shadowDarkness` is held as an internal value, but is hidden from UI with default `0.0`.

Lighting panel initial values:

- Azimuth: `20`
- Elevation: `-50`
- Light intensity: `0.8`
- Ambient light: `0.2`
- Shadow darkness: `0.0` (UI hidden)
- Shadow range: `220`

Lighting panel constraints:

- UI upper limit for `shadowFrustumSize` is `6000`
- As the range is expanded, shadow density decreases, so it is easier for appearance to be stable if not made larger than necessary

## Known Limitations

- Currently, the policy is "all meshes cast/receive shadows".
  Fine-grained ON/OFF by PMX material flags is not used.
- "Cast shadow only on self model" and "Cast shadow only on other models" cannot be expressed with PMX material flags alone.
- Babylon.js shadow caster registration is per mesh.
  Therefore, completely separated caster control per material within the same mesh is not possible.
- However, with babylon-mmd's default `optimizeSubmeshes=true`, meshes are split per material,
  so in practice it behaves close to per-material.
- As the shadow range is expanded, density per pixel decreases even at the same resolution.
  Trade-off adjustment between `shadowFrustumSize` and resolution is necessary as needed.
