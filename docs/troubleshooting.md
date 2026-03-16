# Troubleshooting

## `Cannot find module 'supports-color'` error

### Symptoms

`supports-color` is not found via `chalk` when running `npm start`.

### Cause

There are dependencies in `package-lock.json`, but `node_modules` is in an incomplete state.

This is likely to occur when using `node_modules` from the moved folder as is, or when the dependency tree is broken midway.

### Solution

```bash
npm ci
npm start
```

Rebuild dependencies according to the lockfile with `npm ci`.

## Linux version zip won't start

### Symptoms

- Aborts immediately after startup on Ubuntu 24.04 etc.
- Console shows `chrome-sandbox` or `setuid sandbox` related errors

### Cause

In Linux zip distribution, it is difficult to maintain the owner or `4755` attribute of `chrome-sandbox` as is, and Electron/Chromium sandbox requirements may not be met.

### Solution

- The current packaged build includes a temporary measure to start with `--no-sandbox` and `--disable-setuid-sandbox` for Linux only.
- If it still doesn't start, start from the terminal and check additional error logs.
- As a permanent solution, we are considering reviewing Linux distribution formats such as AppImage / Flatpak.

## App starts but cannot load models

Check points:

- Is `webSecurity: false` remaining in `src/main.ts`?
- Is the relative placement of PMX and textures broken?
- If the load path has many Japanese/special characters, try with a short path for now

## X accessories become black and white/checkered

### Symptoms

- `.x` is displayed but textures are not applied
- Console shows `Texture not found` or `ERR_FILE_NOT_FOUND`

### Cause

- `TextureFilename` in `.x` does not match the actual file placement
- Referenced extension (`.bmp` etc.) is different from the actual file

### Solution

1. Check the relative path placement of `.x` and textures
2. Check the uppercase/lowercase and extension of texture file names
3. If possible, convert to `.png` and place with the same base name

## Many Lint warnings

Current rules allow warnings.

```bash
npm run lint
```

Development can continue if `error` is 0.

## Top panel remains `Physics unavailable`

### Symptoms

- Physics button remains `Physics unavailable`
- Console shows wasm loading errors such as `expected magic word 00 61 73 6d`

### Typical causes

- Both Bullet's `spr/index_bg.wasm` and Ammo's `ammo.wasm.wasm` initialization failed
- Dev server cache is referencing old bundles
- wasm URL resolution returns HTML instead of wasm

### Solution

1. Restart the development server (restart `electron-forge start`)
2. If it still doesn't work, delete `node_modules/.vite` and restart
3. Check if console shows `Bullet physics initialization failed` or `Physics initialization failed`

Current implementation initializes Bullet backend first, and falls back to Ammo backend only on failure.

## Top panel shows `Ammo`

### Symptoms

- App works but top panel backend badge shows `Ammo`
- Doesn't become the expected `Bullet`

### Meaning

Bullet backend initialization failed and started with Ammo fallback.

### Solution

1. Check if console shows `Bullet physics initialization failed`
2. Completely close and restart the Electron app
3. If it still doesn't work, delete `node_modules/.vite` and restart
4. Check Bullet initialization exception contents such as `spr/index_bg.wasm` resolution failure or `object is not extensible`
