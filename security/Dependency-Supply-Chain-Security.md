# Dependency & Supply Chain Security

Purpose
- Controls and practices to manage risk from dependencies, package registries, CI inputs, and build systems.

Key controls
- Use automated dependency update tools (Dependabot, Renovate) with review policies for major upgrades.
- Lockfiles: commit lockfiles for deterministic builds (`package-lock.json`, `poetry.lock`, `go.sum`, `Cargo.lock`).
- Verify sources: prefer official package registries and verify checksums or signatures when available.
- CI isolation: run dependency installs and builds in ephemeral, minimal-privilege runners.

Reproducible builds
- Aim for reproducible builds where artifacts can be recreated and verified from source.
- Record build provenance with SBOMs (Software Bill of Materials) and artifact metadata.

Third-party code review
- Require a security review for new direct dependencies that are critical or have native extensions.
- Track transitive dependency risks for high-sensitivity parts of the codebase.

SBOM and vulnerability scanning
- Generate SBOMs as part of CI (CycloneDX or SPDX formats) and store them with releases.
- Integrate vulnerability scanners (OS package scanners, language-specific scanners, and vulnerability databases) into CI.

Responding to advisories
- Define an SLA for responding to critical vulnerabilities (e.g., triage within 24 hours, mitigation plan within 72 hours).
- Maintain a rollback and mitigation plan for critical dependency issues.

Checklist
- [ ] Lockfiles committed and validated in CI
- [ ] SBOM generated for releases
- [ ] Vulnerability scanners integrated into CI

