# MMD modoki

A local MMD-style editing tool based on Babylon.js Editor and `babylon-mmd`.  
You can load PMX/PMD models, accessories, VMD, camera VMD, and audio, perform timeline editing and preview, and export PNG output.

## Download

- Release list: https://github.com/togechiyo/MMD_modoki/releases

The distribution is provided as OS-specific zip files.

- `mmd-modoki-windows-x64-zip.zip`
- `mmd-modoki-macos-x64-zip.zip`
- `mmd-modoki-linux-x64-zip.zip`

## How to Start

1. Download the zip file for your OS from `Releases`.
2. Extract the zip file.
3. Launch the application from the extracted folder.

Windows:
- `MMD modoki.exe`

macOS:
- `MMD modoki.app`

Linux:
- The Linux version may require launching with `--no-sandbox` depending on your environment. This is a temporary measure to avoid startup failures caused by `chrome-sandbox`.
- Launch the executable directly from the extracted location.

## First Launch Notes

- The macOS version is unsigned, so you may see a Gatekeeper warning.
- The Linux version may require additional libraries depending on your environment.
- As this is an initial release, the save format and UI may be adjusted in the future.

## Features

- Load PMX/PMD models
- Load `.x` accessories
- Load VMD motions and camera VMD
- Load MP3/WAV audio
- Timeline editing
- Adjust bones, morphs, camera, and lighting
- Save PNG, PNG sequence
- Adjust post effects such as DoF, Bloom, LUT

Notes:
- SSAO is always OFF in the current build to reduce load.
- Anti-aliasing uses `MSAA x4 + FXAA`.

## Supported File Types

Supported for normal loading or drag & drop:

- Models: `.pmx` `.pmd`
- Accessories: `.x`
- Motions/poses: `.vmd` `.vpd`
- Camera motions: `.vmd`
- Audio: `.mp3` `.wav`

Load from dedicated UI:

- Projects: `.json` (default filename: `*.modoki.json`)

Notes:

- `.vmd` files are loaded as model motions or camera motions depending on their content.
- `.x` files are expected to be text-format DirectX X files.

## Basic Operations

- `Ctrl + O`: Open PMX/PMD
- `Ctrl + M`: Open VMD
- `Ctrl + Shift + M`: Open camera VMD
- `Ctrl + Shift + A`: Open audio
- `Ctrl + S`: Save project / Overwrite save
- `Ctrl + Alt + S`: Save as
- `Ctrl + Shift + S`: Save PNG
- `Space` or `P`: Play / Stop
- `Delete`: Delete selected keyframe

Mouse:
- Middle button drag: Move view
- Right button drag: Rotate
- Wheel: Zoom

## Development

Required environment:
- Node.js 18 or higher
- npm

Setup:

```bash
npm install
```

Development launch:

```bash
npm start
```

Lint:

```bash
npm run lint
```

Distribution build:

```bash
npm run package
npm run make
```

Create zip distribution:

```bash
npm run make:zip
```

## Documentation

- Documentation index: [docs/README.md](./docs/README.md)
- Architecture: [docs/architecture.md](./docs/architecture.md)
- MmdManager explanation: [docs/mmd-manager.md](./docs/mmd-manager.md)
- UI flow: [docs/ui-flow.md](./docs/ui-flow.md)
- Troubleshooting: [docs/troubleshooting.md](./docs/troubleshooting.md)

## License

- This project: [MIT](./LICENSE)
- Third-party notices: [THIRD_PARTY_NOTICES.md](./THIRD_PARTY_NOTICES.md)
