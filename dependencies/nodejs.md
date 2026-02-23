# Node.js Dependency Management (npm / yarn / pnpm)

Purpose
- Guidance for managing Node.js dependencies, lockfiles, private registries, and CI reproducibility.

Lockfiles and installs
- Commit lockfiles: `package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml` depending on the package manager in use.
- Use `npm ci` in CI for reproducible installs (requires package-lock.json). For Yarn: `yarn install --frozen-lockfile`; for pnpm: `pnpm install --frozen-lockfile`.

Node engine and runtime
- Pin Node.js versions via `.nvmrc`, `engines` in `package.json`, or use toolchains in CI images to ensure consistent environments.

Private registries and auth
- Use `.npmrc` to configure private registries and tokens. Store tokens in CI secrets and avoid committing `.npmrc` with auth tokens.
- For scoped packages, configure registry per-scope to separate public and private dependencies.

Vulnerability scanning and SBOM
- Use `npm audit` for quick local checks; integrate `snyk`, `npm audit CI`, or `grype/trivy` (against artifacts/SBOMs) in CI.
- Generate SBOMs with Syft and scan: `syft . -o cyclonedx-json > sbom.cdx.json` followed by `grype sbom:sbom.cdx.json`.

Native modules and build tools
- Ensure CI runners include required build tools for native modules (node-gyp, python, make, build-essential). Use prebuilt binaries where possible.

Monorepo considerations
- For monorepos, consider pnpm or Yarn workspaces; centralize lockfile strategy and CI caching per workspace layout.

Automated updates
- Configure Dependabot or Renovate for npm and group updates sensibly. Do not automerge major version bumps without review.

CI tips
- Cache node_modules or package manager caches (npm cache, pnpm store) for faster builds.
- Run `npm ci` then `npm test` in CI; fail on audit or configure policy to create issues for vulnerabilities.

Best practices summary
- Commit lockfiles and pin Node runtime.
- Use SBOM + scanner workflows in CI.
- Protect private registry credentials and use scoped registries for internal packages.
