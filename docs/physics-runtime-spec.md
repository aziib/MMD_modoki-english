# Physics Implementation Specification (Current)

Updated: 2026-03-12

## Purpose

- Document current physics implementation in `MMD_modoki`.
- Align Bullet priority / Ammo fallback behavior with UI display.

## Target Scope

- Implementation files: `src/mmd-manager.ts`, `src/ui-controller.ts`, `index.html`, `src/main.ts`
- Priority runtime: babylon-mmd `MultiPhysicsRuntime` + `MmdBulletPhysics`
- Fallback runtime: babylon-mmd `MmdAmmoJSPlugin` + `MmdAmmoPhysics`

## Backend Selection Specification

1. Start `initializePhysics()` asynchronously when `MmdManager` starts.
2. First initialize Bullet backend.
3. Fallback to Ammo.js backend only when Bullet initialization fails.
4. If Ammo also fails, continue startup with physics disabled.

Internal state:

- `physicsBackend = "bullet" | "ammo" | "none"`
- Build model physics only when `physicsAvailable = true`
- Even if `physicsEnabled = false`, PMX / VMD / playback can continue

## Bullet Backend Initialization Specification

1. Load `spr/index_bg.wasm` with `?url` resolution.
2. Convert `spr` wasm bindgen module to plain object and supplement `memory` and `createTypedArray`.
3. Generate `MultiPhysicsRuntime` and register to scene.
4. Generate `MmdBulletPhysics` and connect as physics implementation on MMD runtime side.

Current settings:

- fixed time step: `1 / 120`
- max sub steps: `120`
- gravity: `Vector3(0, -98, 0)`

## Ammo Fallback Initialization Specification

1. Resolve `ammo.wasm.wasm` with `?url` and `fetch`.
2. Explicitly inject wasm binary with `Ammo({ wasmBinary })`.
3. Create `MmdAmmoJSPlugin` and execute `scene.enablePhysics(...)`.
4. Generate `MmdAmmoPhysics` and connect to MMD runtime side.

Current settings:

- gravity: `Vector3(0, -98, 0)`
- max steps: `120`
- fixed time step: `1 / 120`

## Model Load Specification

- `loadPMX` proceeds after `physicsInitializationPromise` completes.
- When physics is available, `createMmdModel(..., { buildPhysics: { disableOffsetForConstraintFrame: true } })`
- When physics is unavailable, `createMmdModel(..., { buildPhysics: false })`
- Maintain `disableOffsetForConstraintFrame: true` for both Bullet / Ammo backends

## Gravity Application Specification

- When using Bullet, reflect to `MultiPhysicsRuntime.setGravity(...)`.
- When using Ammo, reflect to Babylon physics engine (`scene.getPhysicsEngine()?.setGravity(...)`).
- Handle gravity changes on UI with same API without being aware of backend.

## ON / OFF Specification

- Physics ON / OFF switches all rigid bodies `1 / 0` with `model.rigidBodyStates`.
- Public API:
  - `isPhysicsAvailable()`
  - `getPhysicsEnabled()`
  - `setPhysicsEnabled(enabled)`
  - `togglePhysicsEnabled()`
  - `getPhysicsBackendLabel()`
- State notification:
  - `onPhysicsStateChanged(enabled, available)`

## UI Specification

Physics toggle:

- Button ID: `btn-toggle-physics`
- Label ID: `physics-toggle-text`
- Display:
  - `Physics ON`
  - `Physics OFF`
  - `Physics unavailable`

Top panel backend badge:

- ID: `physics-type-badge`
- Display:
  - `Bullet`
  - `Ammo`
  - `Off`
- Color scheme:
  - `Bullet`: Green
  - `Ammo`: Amber
  - `Off`: Gray

## Disposal Specification

- When using Bullet, execute `bulletPhysicsRuntime.unregister()` and `dispose()`.
- When using Ammo, follow Babylon physics engine / plugin disposal path.
- On completion, return to `physicsBackend = "none"`.

## Error Handling Specification

- When Bullet initialization fails:
  - Output warning to console
  - Fallback to Ammo backend
- When Ammo also fails:
  - `physicsAvailable = false`
  - `physicsEnabled = false`
  - `physicsBackend = "none"`
  - Notify `onPhysicsStateChanged(false, false)`
  - Notify `onError("Physics init warning: ...")`

## Known Constraints

- Physics parameters are currently hardcoded.
- Backend manual selection UI is not implemented, only automatic selection.
- User switching equivalent to `disableBidirectionalTransformation` is not implemented.
- Detailed debug display (rigid body / constraint visualization) is not implemented.
- Electron side V8 old-space is expanded to `4096MB`, but does not guarantee memory limit for extremely heavy multiple models.
