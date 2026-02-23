# Dependency & Supply Chain Management Policy

This document describes policies and recommended practices for dependency management, SBOM generation, vulnerability scanning, automated dependency updates, and advisory response SLAs.

1. Lockfiles and Pinning

- Always commit dependency lockfiles to the repository (e.g., package-lock.json, yarn.lock, Pipfile.lock, poetry.lock, go.sum). Lockfiles ensure reproducible builds and make vulnerability scanning more accurate.
- For ecosystems without lockfiles, prefer pinned/minimal ranges for production dependencies and use tooling that can produce a deterministic dependency graph.
- Update lockfiles via automated PRs (Dependabot / Renovate) and review dependency upgrades before merging.

2. SBOM (Software Bill of Materials)

Goals:
- Produce an SBOM for each build artifact (container image, package, or release) and store it as part of the CI artifact set.
- Support machine-readable formats (CycloneDX, SPDX) for downstream tooling and audits.

Recommended generation commands (examples):

- Using Syft (CycloneDX JSON):
  - syft . -o cyclonedx-json > sbom.cdx.json
  - syft . -o spdx-json > sbom.spdx.json

- Using cyclonedx-bom (if installed):
  - cyclonedx-bom -o sbom.cdx.xml --input-type folder .

CI integration notes:
- Generate an SBOM in the build pipeline for every CI build that produces an artifact.
- Attach SBOMs to release artifacts and store in the artifact repository.

3. Vulnerability Scanning

Policy:
- Run automated vulnerability scans on every merge to main and on a schedule (weekly) against source, built artifacts, and generated SBOMs.
- Use multiple scanners if possible to reduce false negatives (e.g., Trivy/Grype/Snyk).

Example scanning commands:

- Grype (scan filesystem or SBOM):
  - grype dir:./
  - grype sbom:sbom.cdx.json

- Trivy (filesystem scan):
  - trivy fs --severity HIGH,CRITICAL .

- Snyk (if using Snyk):
  - snyk test --all-projects

- CI should fail or create an advisory ticket according to the severity policy below.

4. Automated Dependency Updates (Dependabot / Renovate)

Dependabot example (.github/dependabot.yml):

- version: 2
  updates:
    - package-ecosystem: "npm"
      directory: "/"
      schedule:
        interval: "weekly"
      open-pull-requests-limit: 10
    - package-ecosystem: "github-actions"
      directory: "/"
      schedule:
        interval: "weekly"

Renovate notes:
- Renovate can be used instead as an alternative with more advanced grouping and scheduling.
- Configure grouping and automerging cautiously; require human review for major version upgrades.

5. Advisory Triage & Response SLAs

Triage process:
- New advisories from scanners, Dependabot/ Renovate PRs, or external feeds are triaged within the stated SLA window.
- Triage should include: reproduce/verify, determine exploitability in our environment, identify mitigation or upgrade path, and create a remediation plan (PR, workaround, or risk acceptance).

Response SLAs (time to triage and either remediate or create a documented mitigation plan):
- Critical (remote code execution, active exploit affecting production): Triage within 24 hours, mitigation/patch within 72 hours or escalate to on-call.
- High (privilege escalation, high-impact data exposure): Triage within 48 hours, mitigation/patch within 7 days.
- Medium (denial-of-service, lower-impact issues): Triage within 7 days, mitigation/patch within 30 days.
- Low (informational, low risk): Triage within 30 days.

Escalation:
- If a fix cannot be implemented within the SLA, escalate to the security lead and product owner. Document compensating controls and temporary mitigations.

6. Reporting & Metrics

- Maintain an inventory of SBOMs and scan results associated with releases.
- Track Mean Time To Triage (MTTT) and Mean Time To Remediate (MTTR) by severity.
- Weekly vulnerability digest to stakeholders and monthly executive summary for significant issues.

7. Continuous Improvement

- Periodically review scanner coverage and false-positive rates. Add additional scanners or tune rules as needed.
- Reconcile SBOM contents with runtime inspection (e.g., container image runtime) to ensure SBOM accuracy.

Appendix: Quick Example Commands

- Generate SBOM (CycloneDX JSON) with Syft:
  - syft . -o cyclonedx-json > sbom.cdx.json

- Scan SBOM with Grype:
  - grype sbom:sbom.cdx.json

- Scan filesystem with Trivy for high/critical:
  - trivy fs --severity HIGH,CRITICAL .

- Run Grype against a container image:
  - grype docker:myorg/myimage:latest

- Create Dependabot config: .github/dependabot.yml (see examples above)


Framework-specific considerations

Node / JavaScript / TypeScript

- Lockfiles: commit `package-lock.json` or `yarn.lock` (for Yarn) and use `npm ci` or `yarn --frozen-lockfile` in CI for reproducible installs.
- Vulnerability tooling: run `npm audit` for quick checks, and integrate `snyk test` or `npm audit` + `grype`/`trivy` against built artifacts.
- Native modules: audit native bindings (node-gyp) and pin underlying native library versions; prefer prebuilt binaries from trusted sources.
- SBOM: `syft . -o cyclonedx-json > sbom.cdx.json` and `grype sbom:sbom.cdx.json` to scan.

Python

- Lockfiles: prefer `poetry.lock` or `pip-tools` generated `requirements.txt` files; use `pip install -r requirements.txt` or `poetry install` in CI.
- Vulnerability tooling: use `pip-audit`, `safety`, or `bandit` (SAST) alongside SBOM-based scanners (Grype/Trivy).
- Native wheels & C-extensions: pin platform-specific wheels where possible; verify source distributions (sdist) builds in CI to detect compilation-time issues.
- SBOM: generate with Syft and scan with Grype: `syft . -o spdx-json > sbom.spdx.json` then `grype sbom:sbom.spdx.json`.

.NET / NuGet

- Lockfiles: `packages.lock.json` (enable RestoreLockedMode in CI) or commit `obj/project.assets.json` per org policy for reproducibility.
- Vulnerability tooling: use `dotnet list package --vulnerable` and `dotnet restore` combined with third-party scanners (e.g., Snyk, WhiteSource/Dependabot checks for NuGet).
- Native dependencies: Windows/native runtime dependencies should be tracked and pinned; validate runtime images for containerized apps.
- SBOM: Syft works with .NET projects as well (run against the built output or project folder) and `cyclonedx` tooling can be used.

Go

- Lockfiles: commit `go.sum` and use `go mod download` or `go env -w GOPROXY` and `GOSUMDB` for verification.
- Vulnerability tooling: use `govulncheck` (official) or `gosec` for static checks; use `grype`/`trivy` for image scanning.
- Native cgo dependencies: pin and vendor C libs where feasible or document required system packages in CI images.
- SBOM: `syft` at module root or built binary: `syft ./ -o spdx-json > sbom.spdx.json`.

Rust

- Lockfiles: `Cargo.lock` should be committed for applications; library crates may omit it per Rust guidelines but prefer committing for reproducible builds.
- Vulnerability tooling: `cargo audit` is the standard; add `cargo deny` for policy enforcement and `cargo-audit` in CI.
- Native libs and linking: document system dependencies and link-time flags; prefer static linking for consistent runtime behavior when appropriate.
- SBOM: `cargo` crates are supported by Syft; generate SBOMs for release artifacts.

DoneJS / Other JS frameworks

- Treat DoneJS projects like Node.js: commit lockfiles, inspect `package.json` to find test/build runners, and scan with the same Node toolchain scanners.

Frontend / HTML / CSS / Web assets

- Package managers: `npm`/`yarn`/`pnpm` lockfiles apply for frontends as well; ensure host toolchain versions (node) are pinned via `.nvmrc` or engines field.
- Third-party assets: audit CDN-hosted or externally loaded scripts and prefer bundling vetted dependencies into release artifacts.
- Content Security Policy (CSP): maintain a CSP policy and validate third-party script requirements during dependency upgrades.

Containers and images

- Build-time SBOM: generate SBOMs for container images after build (Syft supports images: `syft docker:myorg/image:tag -o spdx-json > image-sbom.spdx.json`).
- Image scanning: run `trivy image --severity HIGH,CRITICAL myorg/image:tag` or `grype docker:myorg/image:tag` as part of CI gating.
- Base image pinning: avoid `:latest` and pin to specific digest or immutable tags; prefer minimal base images (distroless) where appropriate.

Native & OS packages

- For systems that rely on OS packages (apt/yum), capture the package versions used in image builds and include them in SBOMs when possible.
- Use package signature verification and pinned repositories for production images.

General guidance

- Native/compiled deps require special attention: include build reproducibility checks, CI artifact verification, and SBOMs for both source and binary artifacts.
- For language ecosystems that support multiple registries, restrict installs to approved registries via config (npmrc, pip index-url, nuget.config).
- CI should validate lockfile integrity (e.g., run `npm ci`, `pip install -r`, `go mod download`) and fail early on mismatches.

Package manager details

Python (pip / pip-tools / poetry)

- Recommended tools:
  - pip for installs, pip-tools (`pip-compile`) for deterministic `requirements.txt` generation, or Poetry for modern dependency management and lockfiles.
- Typical commands:
  - Create venv and install: `python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`
  - Using pip-tools: `pip-compile pyproject.toml` or `pip-compile requirements.in` then `pip-sync` in CI agents if desired.
  - Using Poetry: `poetry install --no-interaction --no-ansi` in CI; `poetry lock` to update lockfile locally.
- Private indexes and credentials:
  - Configure private indices via `pip.conf` or `--index-url`; use tokenized credentials stored in CI secrets (do not commit credentials).
- Lockfile strategy:
  - Commit `requirements.txt` (from pip-compile) or `poetry.lock` for reproducible installs; prefer `pip-compile` for complex dependency resolution transparency.

.NET / NuGet

- Recommended tools:
  - dotnet CLI (`dotnet restore`, `dotnet build`, `dotnet test`) and nuget configuration files (`nuget.config`) for feeds.
- Typical commands:
  - Restore & build: `dotnet restore && dotnet build --no-restore`
  - Restore in locked mode: enable `RestoreLockedMode` or use `dotnet restore --use-lock-file` when using lockfiles.
- Lockfiles & reproducibility:
  - Use `packages.lock.json` to lock package versions for apps. Use `dotnet restore --locked-mode` in CI to ensure lockfile consistency.
- Private feeds:
  - Configure `nuget.config` with feed URLs and use CI secrets or service principals for authentication. Avoid committing credentials.
- Auditing packages:
  - Use `dotnet list package --vulnerable` or third-party scanners (Snyk, WhiteSource) and integrate into CI.

DoneJS / npm (DoneJS is Node-based)

- Recommended tools:
  - npm (or yarn/pnpm depending on project) as the package manager; DoneJS projects typically use `npm` commands.
- Typical commands:
  - Install CI deps: `npm ci` (uses package-lock.json) for reproducible installs.
  - Local install: `npm install` for development.
- Lockfiles & engines:
  - Commit `package-lock.json` (or `yarn.lock`/`pnpm-lock.yaml`) and pin Node versions via `.nvmrc` or the `engines` field in `package.json`.
- Private registries:
  - Configure `.npmrc` to point to private registries and use CI secrets for auth tokens; avoid committing tokens.
- Native modules / build tooling:
  - For native addons, ensure CI runners include platform build tools or use prebuilt binaries where feasible.

Notes
- These are recommended conventions — adapt schedules and SLAs to match organizational risk appetite.
- Ensure that credentials and secret materials are never included in SBOMs or published artifacts.
