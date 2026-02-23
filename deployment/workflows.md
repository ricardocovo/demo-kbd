CI Workflow Guidelines

This document describes recommended CI workflow guidelines for pull requests (PRs), branch protection, release/tagging process, and required status checks. Reference existing workflows in the repository under the .github/workflows/ path and add checks there as needed.

1) CI checks to run on every PR

- Linting
  - Run code linters (e.g., ESLint, ruff, golangci-lint) to catch style and static issues.
  - Fail the PR if lint errors are present.

- Tests
  - Run unit and integration tests (e.g., npm test, pytest, go test) and report coverage where applicable.
  - Include fast, deterministic test suites for PR runs; schedule longer integration tests on branch or nightly jobs.

- Dependency scan
  - Run dependency vulnerability scanning (e.g., GitHub Dependabot, npm audit, pip-audit, Snyk) to detect vulnerable packages.
  - Block merges for critical/high vulnerabilities until resolved.

- Secret scan
  - Run secret scanning (e.g., GitHub Secret Scanning, truffleHog) to detect committed secrets.
  - Immediately block PRs that introduce exposed secrets and require remediation.

- Optional additional checks
  - Static analysis (e.g., type checks, linters with deeper rules)
  - Build or compilation checks for compiled languages
  - Security linters (e.g., bandit for Python)

2) Branch protection settings

- Protect main (or default) branch and any release branches.
- Require pull request reviews before merging:
  - Require at least 1-2 approving reviews depending on repository sensitivity.
  - Optionally require review from code owners for changes in protected paths (CODEOWNERS).
- Require status checks to pass before merging (see section below).
- Require branch be up to date with the base branch before merging (or require successful merge commit check).
- Restrict who can push to the protected branch (e.g., maintainers only) where appropriate.
- Enable enforcement for administrators if desired to prevent direct pushes.

3) Release and tagging process

- Use a documented release process (e.g., GitHub Releases, semantic-release, manual tagging):
  - Create release branches (e.g., release/x.y) as needed.
  - Tag releases consistently (e.g., vMAJOR.MINOR.PATCH) and create GitHub Releases with release notes.
  - Automate changelog generation where possible (e.g., conventional commits + semantic-release).
  - Run full CI (tests, build, integration tests) on release branches and on tags.

- If using automation (CI/CD to publish artifacts):
  - Protect credentials used for publishing in repository secrets.
  - Ensure only tagged commits trigger publish workflows.

4) Required status checks (suggested)

- Lint
- Unit tests
- Build/compile (if applicable)
- Dependency scan
- Secret scan
- (Optional) Code coverage report

These status checks should be configured in the repository settings and referenced by name in branch protection rules. The exact job names will match the workflow job names defined under .github/workflows/*.yml.

5) Referencing and adding checks

- Existing workflow files are located under .github/workflows/. Review existing jobs and add or update jobs to implement the checks above.
- When adding new checks, ensure job names are stable (do not change names frequently) since branch protection rules reference job names.
- Use required status check names that match the job "name:" fields in workflow YAMLs. For matrix jobs, reference the aggregate job name (or individual matrix job names if desired).
- Keep PR checks fast and deterministic; offload longer or flaky workloads to scheduled or branch workflows.

6) Enforcement and Exceptions

- For security fixes or high-priority releases, maintainers may have an exception process documented in CONTRIBUTING.md.
- Any exceptions should be logged and reviewed.

Notes

- Review .github/workflows/ to align job names and to add missing checks.
- When adding new required checks, coordinate with the team so developers can update local workflows and CI expectations.
