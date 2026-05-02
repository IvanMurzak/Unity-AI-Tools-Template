# Unity-AI-Tools-Template

This is the **template repo** for creating new Unity-MCP extension packages.
Boilerplate gets customized by `commands/init.ps1` (renames placeholders like
`YOUR_PACKAGE_ID`, `YOUR_PACKAGE_NAME`). Changes here propagate to every
future extension created from it — keep placeholder tokens consistent.

## Build / run

```powershell
# 1. Initialize a new package from the template
./commands/init.ps1 -PackageId "com.company.package" -PackageName "My Package"

# 2. Open both Unity projects so .meta files are generated
./commands/open-all-projects-windows.ps1   # or open-all-projects-unix.sh

# 3. Bump version across all files in lock-step
./commands/bump-version.ps1 -NewVersion "1.0.1"   # add -WhatIf to preview

# 4. Get current version
./commands/get-version.ps1
```

Package source: `Unity-Package/Assets/root/` (only this folder ships).
Multi-version test rigs: `Unity-Tests/{2022.3.62f3,2023.2.22f1,6000.3.1f1}`.

## Find detail in

- `docs/claude/architecture.md` — layout, tech stack, Editor-vs-Runtime decision, MCP tool pattern, init flow, coding rules
- `docs/claude/release.md` — `bump-version.ps1` mechanics, `bump_version.yml`, release pipeline outputs
- `docs/claude/ci.md` — workflow files, required GitHub secrets, test matrix, PR safety guards
- `README.md` — user-facing setup walkthrough
- `docs/Deploy-OpenUPM.md`, `docs/Deploy-GitHub.md`, `docs/Deploy-npmjs.md` — registry deploy guides
