# Architecture

## What this template is

This is a template repository for creating new Unity-MCP extension packages.
It provides boilerplate that gets customized via `commands/init.ps1`. After
init, the repo becomes a real Unity package with custom MCP tools that plug
into Unity-MCP (`com.ivanmurzak.unity.mcp`).

## Top-level layout

```
Unity-AI-Tools-Template/
├── commands/                 # PowerShell helpers (init, bump-version, open-projects)
├── docs/                     # User-facing docs (Deploy guides, manual rename, images)
│   └── claude/               # On-demand notes for Claude Code (this folder)
├── Installer/                # Standalone Unity project that builds .unitypackage installer
├── Unity-Package/            # The package source — only `Assets/root/` ships
│   └── Assets/
│       └── root/             # Files inside this folder become the package
│           ├── package.json
│           ├── Editor/       # Editor-only scripts (Editor API access)
│           ├── Runtime/      # Runtime scripts (no Editor API, ships in builds)
│           ├── Tests/
│           │   ├── Editor/   # EditMode tests
│           │   └── Runtime/  # PlayMode tests
│           ├── Samples~/
│           └── Documentation~/
├── Unity-Tests/              # Multi-version Unity test rigs (2022.3.62f3, 2023.2.22f1, 6000.3.1f1)
├── .github/workflows/        # CI: bump_version, test_unity_plugin, release-sample, test-pr-sample
├── README.md
└── LICENSE
```

> Anything outside `Unity-Package/Assets/root/` is NOT shipped to consumers.
> It exists only for testing, CI, and showcasing.

## Tech stack

- **Language:** C# 9.0, `netstandard2.1`
- **Unity:** 2022.3+ (minimum). Tested against 2022.3.62f3, 2023.2.22f1, 6000.3.1f1.
- **Dependency:** `com.ivanmurzak.unity.mcp` v0.68.0 (declared in `Unity-Package/Assets/root/package.json`)
- **Assemblies (placeholders set by init.ps1):**
  - `YOUR_PACKAGE_ID.Editor`
  - `YOUR_PACKAGE_ID.Runtime`
  - `YOUR_PACKAGE_ID.Editor.Tests`
  - `YOUR_PACKAGE_ID.Tests`
- **Test framework:** NUnit + Unity Test Framework (EditMode + PlayMode)
- **Scripts:** PowerShell (`init.ps1`, `bump-version.ps1`, `get-version.ps1`,
  `open-all-projects-windows.ps1`, `open-all-projects-unix.sh`,
  `update-ai-game-developer.ps1`)

## Tool decision: Editor vs Runtime

When adding a new MCP tool:

- **Editor tool** → put under `Unity-Package/Assets/root/Editor`
  - Works in Edit Mode and Play Mode in the Editor.
  - Has access to Editor APIs.
  - NOT shipped in game builds.
- **Runtime tool** → put under `Unity-Package/Assets/root/Runtime`
  - Works in Edit Mode and Play Mode in the Editor.
  - NO access to Editor APIs.
  - Ships in game builds.

## MCP tool pattern

Tools are static classes attributed with `[McpPluginToolType]`, with one
operation per method attributed with `[McpPluginTool(...)]`. Follow the
Unity-MCP-Plugin convention of one operation per file via partial classes.

```csharp
[McpPluginToolType]
public static class MyCustomTool
{
    [McpPluginTool("my-custom-feature", Title = "Do my custom feature")]
    [Description("Put here the tool description for LLM.")]
    public static Task<bool> DoTurn(
        [Description("Help LLM understand this property.")] int figureId,
        [Description("Help LLM understand this property.")] Vector2Int position)
    {
        // background-thread work here
        return MainThread.Instance.RunAsync(() =>
        {
            // main-thread work here
            return true;
        });
    }
}
```

## Init flow (placeholders)

`commands/init.ps1 -PackageId "com.company.package" -PackageName "My Package"`
renames directories, files, and replaces tokens like `YOUR_PACKAGE_ID`,
`YOUR_PACKAGE_ID_LOWERCASE`, `YOUR_PACKAGE_NAME`, `YOUR_PACKAGE_NAME_INSTALLER`,
`YOUR_PACKAGE_NAME_INSTALLER_FILE`. Keep these placeholder tokens consistent
in any new files you add — `init.ps1` is the single source of truth.

After init, open both `Installer/` and `Unity-Package/` in Unity Editor (via
`commands/open-all-projects-*.ps1/.sh`) so Unity generates the `.meta` files.

## Coding rules

- `#nullable enable` at the top of every C# file.
- Copyright box comment header in every C# file.
- Follow MCP tool patterns from Unity-MCP-Plugin (partial classes, one op per file).
- High cohesion, low coupling — many small files preferred.

## Release artifact

The release workflow exports a `.unitypackage` from `Installer/` named
`YOUR_PACKAGE_NAME_INSTALLER_FILE.unitypackage` via the build method
`YOUR_PACKAGE_ID.Installer.PackageExporter.ExportPackage`, then attaches it
to the GitHub Release for the version tag.
