# Architecture Overview

## Overall Structure

This application has a 3-layer Electron structure.

- Main Process: `src/main.ts`
- Preload: `src/preload.ts`
- Renderer: `src/renderer.ts`

The following components are assembled on the Renderer side.

- `MmdManager`: Main execution body for Babylon.js / babylon-mmd
- `mmd-manager-x-extension`: Extends `MmdManager` with X accessory functionality
- `x-file-loader`: Interprets `.x` (text format) as a Babylon SceneLoader plugin
- `Timeline`: Keyframe rendering and seek UI
- `BottomPanel`: Morph and model information UI
- `UIController`: Connects DOM events with the above components

Notes:

- Models are held multiple times within `MmdManager`, and the active target is switched from the UI
- Accessories (`.x`) are held on the extension side, and parent model/parent bone and display state are changed from the UI
- `.x` loading does not rely on Babylon's direct URL reading, but reads binary on the Renderer side and passes it to `x-file-loader`
- `.x` text is automatically detected by comparing the number of replacement characters between UTF-8 and Shift-JIS
- `.x` `baseTexture*sphere(.sph/.spa)` format is decomposed into diffuse / sphere on the loader side
- `.x` accessories have an initial scale of `10x` at load time to match the MMD stage
- Ground display is toggled ON/OFF via the upper toolbar

## Startup Flow

1. Vite builds Main/Preload/Renderer with `electron-forge start`
2. `main.ts` creates a `BrowserWindow`
3. `preload.ts` exposes `window.electronAPI`
4. `renderer.ts` initializes each class
5. `UIController -> MmdManager` is called in response to user operations

## IPC Roles

Handlers are provided on the Main side.

- `dialog:openFile`: File selection
- `file:readBinary`: Binary reading
- `file:getInfo`: Get file information
- `file:savePng`: PNG save dialog + write

Renderer accesses these only via Preload.

## Build and Distribution

- Configuration: `forge.config.ts`
- Vite entry points
  - Main: `src/main.ts`
  - Preload: `src/preload.ts`
  - Renderer: `index.html` + `src/renderer.ts`

Main commands:

- Development launch: `npm start`
- Lint: `npm run lint`
- Distribution build: `npm run package`, `npm run make`
