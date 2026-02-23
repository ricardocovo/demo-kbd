# .NET Dependency Management (NuGet)

Purpose
- Guidance for NuGet package management, lockfiles, private feeds, and CI reproducibility.

Lockfiles and locked restores
- Use `packages.lock.json` to pin package versions for applications. Enable lockfile generation via `dotnet restore` settings or `nuget` tooling.
- In CI, restore with locked mode where supported: `dotnet restore --use-lock-file` or configure `RestoreLockedMode` to ensure reproducible restores.

Restores and builds (CI)
- Typical CI steps:
  - `dotnet restore --verbosity minimal`
  - `dotnet build --no-restore --configuration Release`
  - `dotnet test --no-build --configuration Release`

Private feeds and authentication
- Configure `nuget.config` with feed URLs and credentials; prefer using service principals or CI-managed credentials injected at runtime.
- Avoid committing credentials; use Azure Artifacts or authenticated feeds with tokens stored in CI secrets.

Vulnerability scanning and SBOM
- Use `dotnet list package --vulnerable` for quick checks; integrate Snyk or similar scanners for richer advisories.
- Generate SBOMs for built artifacts via Syft or CycloneDX tooling: run against build outputs or the project folder.

Transitive/native dependencies
- Track and review transitive dependencies; for native runtime dependencies, include OS package manifests in SBOMs where applicable.

Versioning strategy
- Prefer explicit minor/patch pinning for production apps and use automated PRs for dependency updates. Require human review for major upgrades.

CI tips
- Cache NuGet package folders (`~/.nuget/packages`) between CI runs to speed restores.
- Enforce `dotnet format --verify-no-changes` and analyzer rules in CI to keep code quality consistent.

Best practices summary
- Commit and enforce lockfiles for apps.
- Use private feeds with proper auth in CI.
- Integrate vulnerability scanning and SBOM generation in pipelines.
