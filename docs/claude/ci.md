# CI / CD

GitHub Actions workflows live in `.github/workflows/`.

## Workflow files

| File                          | Purpose                                                                 | Trigger                          |
|-------------------------------|-------------------------------------------------------------------------|----------------------------------|
| `bump_version.yml`            | Bumps version in lock-step, opens PR                                    | `workflow_dispatch`              |
| `test_unity_plugin.yml`       | Reusable workflow — runs Unity tests on a given project / version / mode | `workflow_call`, `workflow_dispatch` |
| `release.yml-sample`          | Build installer, run full test matrix, tag, GitHub Release              | Push to `main` (after rename)    |
| `test_pull_request.yml-sample`| Run Unity test matrix on PRs                                            | Pull request (after rename)      |

> The `-sample` workflows are inert until renamed. After running `init.ps1`,
> rename:
>
> - `release.yml-sample` → `release.yml`
> - `test_pull_request.yml-sample` → `test_pull_request.yml`

## Required GitHub Secrets

`Settings` → `Secrets and variables` → `Actions`:

- `UNITY_EMAIL` — Unity account email
- `UNITY_PASSWORD` — Unity account password
- `UNITY_LICENSE` — full contents of `Unity_lic.ulf`
  - Windows: `C:/ProgramData/Unity/Unity_lic.ulf`
  - macOS: `/Library/Application Support/Unity/Unity_lic.ulf`
  - Linux: `~/.local/share/unity3d/Unity/Unity_lic.ulf`

## Test matrix

Tests run against three Unity versions × three modes via the reusable
`test_unity_plugin.yml`:

| Unity version  | EditMode | PlayMode | Standalone |
|----------------|:--------:|:--------:|:----------:|
| 2022.3.62f3    | yes      | yes      | yes        |
| 2023.2.22f1    | yes      | yes      | yes        |
| 6000.3.1f1     | yes      | yes      | yes        |

Each test job runs on `ubuntu-latest` against the `unityci/editor` image,
across `base` and `windows-mono` platforms (matrix), via
`game-ci/unity-test-runner@v4`. Library + `~/.cache/unity3d` are cached per
(version, mode, platform) combo.

## PR safety guards in `test_unity_plugin.yml`

- Only runs when triggered via `pull_request_target` if the PR carries the
  `ci-ok` label (maintainer-applied).
- Aborts before secrets are exposed if the PR modifies any file under
  `.github/workflows/`.

## Release pipeline (`release.yml`)

`Push to main` → `check-version-tag` → `build-unity-installer` →
9 parallel test jobs (3 Unity versions × 3 modes) → `release-unity-plugin`
(tag + GitHub Release) → `publish-unity-installer` (attach `.unitypackage`) →
`cleanup-artifacts`. The `INSTALLER_UNITY_VERSION` env var pins the Unity
version used to export the installer (default `2022.3.62f3`).

## When updating CI

- Keep the Unity version list in `release.yml-sample`,
  `test_pull_request.yml-sample`, and the `Unity-Tests/<version>/` project
  folders in sync.
- Update `INSTALLER_UNITY_VERSION` in `release.yml-sample` only when you
  intentionally upgrade the installer's build Unity.
- Bump GitHub Action versions (`actions/checkout`, `actions/cache`,
  `actions/upload-artifact`, `game-ci/*`) periodically.
