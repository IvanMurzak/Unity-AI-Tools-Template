# Release & Version Management

## Bump version locally

Use the bump script to update versions across all files in lock-step
(`package.json`, `Installer.cs` version constant, etc.):

```powershell
# Apply
./commands/bump-version.ps1 -NewVersion "1.0.1"

# Preview only (no writes)
./commands/bump-version.ps1 -NewVersion "1.0.1" -WhatIf
```

What the script does (`commands/bump-version.ps1`):

- Validates semver format (`major.minor.patch`, optional pre-release/build).
- Reads the current version from `Unity-Package/Assets/root/package.json`.
- Updates these files atomically:
  - `Unity-Package/Assets/root/package.json` — `"version": "X.Y.Z"`
  - `Installer/Assets/YOUR_PACKAGE_NAME_INSTALLER/Installer.cs` — `public const string Version = "X.Y.Z";`
- No-ops cleanly if new version equals current.
- Reminds you to commit the result.

> Add new version-bearing files to the `$VersionFiles` array in
> `bump-version.ps1` so they stay in lock-step.

## Bump version via CI

`.github/workflows/bump_version.yml` is a `workflow_dispatch` job that:

1. Validates the input version (`X.Y.Z`).
2. Creates branch `release/<version>`.
3. Runs `commands/bump-version.ps1 -NewVersion "<version>"`.
4. Commits as `chore: bump version to <version>`.
5. Opens a PR back to the dispatched branch.

Trigger from the GitHub Actions UI with the desired version.

## Release flow

Once the bump PR is merged into `main`, `release.yml` (renamed from
`release.yml-sample` after init — see `docs/claude/ci.md`) runs and:

1. Reads the version from `package.json`.
2. Skips if a tag matching that version already exists.
3. Builds and tests `Installer/` in EditMode.
4. Exports `YOUR_PACKAGE_NAME_INSTALLER_FILE.unitypackage` via
   `YOUR_PACKAGE_ID.Installer.PackageExporter.ExportPackage`.
5. Runs the multi-version Unity test matrix (EditMode + PlayMode + Standalone
   on 2022.3.62f3, 2023.2.22f1, 6000.3.1f1).
6. Creates a Git tag and GitHub Release with auto-generated commit summary.
7. Attaches the `.unitypackage` to the Release.
8. Cleans up the build artifact.

The released package can then be installed via OpenUPM, GitHub, or npmjs —
see `docs/Deploy-OpenUPM.md`, `docs/Deploy-GitHub.md`, `docs/Deploy-npmjs.md`.

## Get current version

```powershell
./commands/get-version.ps1
```
