# PowerShell Copilot Instructions

## Project Architecture
- **Monorepo**: Contains PowerShell engine, modules, SDK, native shims, and test infrastructure.
- **Key directories**:
  - `src/`: Core engine, modules, and host projects. E.g., `System.Management.Automation`, `Microsoft.PowerShell.Commands.*`, `powershell/` (shim entrypoint).
  - `test/`: Pester (PowerShell) and xUnit (C#) tests. Subfolders by test type.
  - `tools/`: Build, CI, and developer scripts.
  - `assets/`: Packaging, manifests, and branding.

## Build & Test Workflows
- **Build**: Use `Start-PSBuild` (from `build.psm1`).
  - Clean build: `Start-PSBuild -Clean -Output <dir>`
  - Output defaults to `debug` subfolder.
- **Bootstrap**: Run `Start-PSBootstrap` to set up dependencies.
- **Test**:
  - Pester: Run `Start-PSPester` from repo root (uses dev-built PowerShell).
  - xUnit: See `test/xUnit/README.md` for .NET test details.
  - To run a subset: `Start-PSPester -Tests <pattern>`
- **Debugging**: Always launch new `pwsh` processes with `-noprofile` and from the dev build (`$PsHome/pwsh`).

## Project Conventions
- **Cross-platform**: All code must work on Windows, Linux, macOS. Use platform guards (`$IsWindows`, `$IsLinux`, `$IsMacOS`).
- **Test Portability**: Use Pester's `-Skip` for platform-specific tests.
- **Pending Tests**: Mark with `-Pending` instead of skipping; file a GitHub issue.
- **SDK meta-project**: `Microsoft.PowerShell.SDK` groups subprojects and runtime dependencies.
- **Entry Point**: `src/powershell/` is a .NET CLI shim, delegating to `Microsoft.PowerShell.ConsoleHost`.

## Patterns & Integration
- **No direct changes to Windows PowerShell 5.1**: This repo is for PowerShell 7+ only.
- **Modules**: `src/Modules/` contains built-in modules, loaded by the engine.
- **Native code**: `src/powershell-native/` and `src/libpsl-native/` for platform interop.
- **CI/CD**: See `.github/workflows/` for GitHub Actions pipelines.

## Examples
- **Platform guard in test**:
  ```powershell
  It "Runs only on Windows" -Skip:($IsLinux -or $IsMacOS) { ... }
  ```
- **Launching dev PowerShell**:
  ```powershell
  $powershell = Join-Path -Path $PsHome -ChildPath "pwsh"
  & $powershell -noprofile -command "..."
  ```

## References
- [Main README](../README.md)
- [Test Guide](../test/README.md)
- [Pester Test Guide](../test/powershell/README.md)
- [Contribution Guide](CONTRIBUTING.md)

## Contribution & Commit Conventions
- **Commit messages**: Use present tense, imperative mood, and summarize in 50 characters or less. Leave a blank line between subject and body. Example:
  > Add support for new feature
- **PR titles**: Describe the change, not just the issue number. Prefix with resource name if relevant. Example: `New-SqlConnection: add parameter 'ConnectionCredential'`.
- **Copyright headers**: New `.cs` files:
  ```csharp
  // Copyright (c) Microsoft Corporation.
  // Licensed under the MIT License.
  
  // ...
  ```
  New `.ps1`/`.psm1` files:
  ```powershell
  # Copyright (c) Microsoft Corporation.
  # Licensed under the MIT License.
  
  # ...
  ```
- **Module manifests**: New `.psd1` files must start with:
  ```powershell
  Author = "PowerShell"
  Company = "Microsoft Corporation"
  Copyright = "Copyright (c) Microsoft Corporation."
  ```
