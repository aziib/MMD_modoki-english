# Release Process Notes

Updated: 2026-03-13

## Purpose

- Releases are performed by pushing tags in `vX.Y.Z` format.
- Building zip distributions and attaching assets to GitHub Release is done by [`.github/workflows/build-zips.yml`](/d:/DevTools/Projects/MMD_modoki/.github/workflows/build-zips.yml).
- When tag is pushed, Windows / macOS / Linux zips are built and automatically attached to prerelease with the same tag name.

## Procedure

1. Update version in `package.json` and `package-lock.json`.
2. If necessary, update public-facing links and descriptions in `README.md` and `docs/README.md`.
3. Perform operation verification.
4. Commit changes and push to `main`.
5. Create tag and push.

```bash
git tag v0.1.3
git push origin v0.1.3
```

6. Confirm that GitHub Actions `Build Zip Packages` succeeds.
7. Confirm the generated prerelease in GitHub Releases.

## Automatically Created Items

- Windows zip
- macOS zip
- Linux zip
- Initial version of prerelease body
- Attachment of zip assets to release

Release name becomes `MMD modoki vX.Y.Z`.

## Verification Points

- Are zips for 3 OSs lined up in release assets?
- Is it treated as prerelease?
- Is zip name the expected version?
- If Linux version notes or known bugs are necessary, are they reflected in release note?

## Linux Version Notes

- Linux version zip is verified by starting with `--no-sandbox`.
- If necessary, also use `--disable-setuid-sandbox`.
- This is a temporary measure to avoid startup failure caused by `chrome-sandbox`, so write the same note in distribution announcement.

Startup example:

```bash
./MMD_modoki --no-sandbox
```

If necessary:

```bash
./MMD_modoki --no-sandbox --disable-setuid-sandbox
```

## Supplementary

- Local `npm run make:zip` is for verification for local OS. Official distributions use GitHub Actions results.
- When workflow fails, zip can be confirmed from Actions artifact, but usually confirm from release assets.
