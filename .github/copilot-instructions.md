# Copilot instructions for demo-kbd

This file is organized as a set of "folders" (top-level sections) representing major standards categories for the repository. Each folder describes the expected artifacts, how Copilot sessions should discover and use them, and what to update when project content changes.

Security/
- Purpose: store security-related guidance and enforcement points that Copilot should respect when suggesting code or changes.
- What to include:
  - Secrets and credential rules: where secrets are allowed and where they must never appear (e.g., no secrets in code, use environment variables and secret managers).
  - Dependency vulnerability policy: how to run scanners (e.g., `npm audit`, `snyk test`, `cargo audit`) and how to act on findings.
  - Security testing commands and CI gates (static analysis, SAST, secret scanning).
- How Copilot should use this folder:
  - Prefer secure defaults in suggestions (parameterized queries, safe deserialization, input validation).
  - If no security files exist, prompt the user before adding secrets or automatic fixes that would change security posture.

Testing/
- Purpose: canonical testing commands, single-test invocation examples, and testing standards for the repository.
- What to include:
  - Exact commands to run the full test suite and a single test per supported language/framework (Node, Python, Go, Rust, .NET, DoneJS, UI/E2E).
  - Guidance on test types (unit, integration, UI/e2e), test fixtures, handling flakiness, coverage collection, and parallelization.
  - CI best-practices and example job snippets to run tests reliably.
- How Copilot should use this folder:
  - Consult Testing/ first for authoritative test run commands, single-test examples, and CI practices before proposing run instructions or CI changes.
  - Prefer existing test scripts (npm script, tox, Makefile, task) and respect project-specific concurrency and environment setup.
  - When suggesting test commands, include exact single-test invocations and any required environment variables or fixtures.

Architecture/
- Purpose: high-level overview describing core components, runtime layout, and data flow.
- What to include:
  - Short narrative (1–3 paragraphs) describing the system, key services/modules, and runtime shape (monolith, services, libs).
  - Important file-layout expectations (src/, cmd/, pkg/, services/, web/, api/).
- How Copilot should use this folder:
  - Use it to contextualize cross-file changes and to decide where new files belong.

Conventions/
- Purpose: project-specific patterns that are not obvious from single files.
- What to include:
  - Naming conventions, error-handling patterns, preferred testing idioms, branches and git commit message format.
  - Example code snippets demonstrating idiomatic patterns.
- How Copilot should use this folder:
  - Follow these patterns when proposing changes or generating new files; when in doubt, ask the user which convention to follow.

Deployment/
- Purpose: deployment, release, and infrastructure automation rules and commands.
- What to include:
  - List of deployment pipelines and workflow files (paths under `.github/workflows/`), environment promotion rules, and rollback procedures; commands the workflows run.
  - Release, tagging, and deployment conventions, required checks for merges and deployment gates.
- How Copilot should use this folder:
  - Reference these when proposing workflow or deployment edits and ensure suggested changes maintain required gates and do not bypass deployment approvals.

Dependencies/
- Purpose: policies and commands for dependency management and upgrades.
- What to include:
  - How to add/update dependencies, preferred tools (dependabot, Renovate), and commands to validate upgrades.
  - Guidelines for pinning versions vs. ranges.
- How Copilot should use this folder:
  - When suggesting dependency changes, include upgrade commands and test run steps; avoid blind upgrades without tests.

Documentation/
- Purpose: where to put developer-facing docs (README, API docs, design notes).
- What to include:
  - Canonical README overview, API docs generator commands, and location for design ADRs (docs/ or adr/).
- How Copilot should use this folder:
  - Update docs alongside code changes and propose documentation edits when adding public APIs.

Observability/
- Purpose: tracing, logging, metrics, and how to validate observability artifacts.
- What to include:
  - Commands to run logs locally, expected log formats, metrics endpoints, and tracing setup.
- How Copilot should use this folder:
  - Ensure instrumentation is added to new services and suggest sensible defaults for logging levels.

Performance/
- Purpose: performance testing commands and benchmarks.
- What to include:
  - Benchmark scripts, load-test commands (e.g., `wrk`, `k6`), and perf budgets.
- How Copilot should use this folder:
  - When recommending algorithmic or data-structure changes, consider perf test coverage and propose benchmark updates.

Existing content and discovery
- The previous single-file advice (build/test detection, AI config searches) has been folded into the Build-Test-Lint and Other-AI-Config checks below.
- Copilot sessions should still begin by checking for language root files (package.json, pyproject.toml, go.mod, Cargo.toml) and CI definitions under `.github/workflows/`.
- Also check for other AI assistant config files and incorporate them: CLAUDE.md, .cursorrules, AGENTS.md, .windsurfrules, CONVENTIONS.md, .clinerules, .github/copilot-instructions.md.

Quick start for future Copilot runs
- Step 1: Read Security/, Build-Test-Lint/, and Architecture/ folders to ground the session.
- Step 2: If making changes, follow Conventions/ for style and CI-CD/ for workflow edits.
- Step 3: Run commands from Build-Test-Lint/ and Dependencies/ to validate changes.

MCP servers
- If you want MCP servers configured (e.g., Playwright for web projects, or a test runner agent), say which server(s) to configure and basic target (browser/e2e) and they will be added.

Maintainer notes
- Keep folder sections minimal but precise: prefer a single authoritative command for each common action and a one-sentence summary of intent.
- Update this file when adding toolchains or CI; Copilot uses these sections as the primary source of truth for automation suggestions.

(Generated from repository state at 2026-02-23)
