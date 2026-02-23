# Copilot instructions for demo-kbd

This file is organized as a set of top-level sections mirroring the repository's folder structure. Each section describes the purpose of its folder, the files it contains, and how Copilot should use that content.

## Repository Folder Structure

```
demo-kbd/
├── .github/
│   └── copilot-instructions.md       ← this file
├── README.md
├── architecture/
│   ├── overview.md                   ← system overview and runtime shape
│   ├── software-architecture.md
│   ├── cloud-architecture.md
│   ├── microservices.md
│   ├── databases.md
│   ├── caching.md
│   ├── mainframe-integration.md
│   └── websockets.md
├── conventions/
│   ├── style.md                      ← general style and commit format
│   ├── dotnet.md
│   ├── nodejs.md
│   ├── python.md
│   └── html-css.md
├── dependencies/
│   └── management.md                 ← dependency policies and upgrade commands
├── deployment/
│   └── workflows.md                  ← pipeline definitions and deployment gates
├── documentation/
│   ├── docs-guidelines.md
│   ├── business-docs.md
│   ├── technical-docs.md
│   └── implementation-plan.md
├── observability/
│   └── logging-metrics.md            ← logging format, metrics endpoints, tracing
├── performance/
│   └── benchmarks.md                 ← benchmark scripts and perf budgets
├── security/
│   ├── General-Security-Measures.md
│   ├── API-Security.md
│   ├── Authentication-Authorization.md
│   ├── Secrets-Management.md
│   ├── Dependency-Supply-Chain-Security.md
│   ├── FSI-Security.md
│   └── Incident-Response-and-Monitoring.md
└── testing/
    ├── Unit-Tests.md
    ├── Integration-Tests.md
    └── Functional-Tests.md
```

---

## security/
- **Files:** `General-Security-Measures.md`, `API-Security.md`, `Authentication-Authorization.md`, `Secrets-Management.md`, `Dependency-Supply-Chain-Security.md`, `FSI-Security.md`, `Incident-Response-and-Monitoring.md`
- Purpose: store security-related guidance and enforcement points that Copilot should respect when suggesting code or changes.
- What to include:
  - Secrets and credential rules: secrets must never appear in code — use environment variables and secret managers (see `Secrets-Management.md`).
  - Dependency vulnerability policy: how to run scanners (e.g., `npm audit`, `snyk test`, `cargo audit`) and how to act on findings (see `Dependency-Supply-Chain-Security.md`).
  - Security testing commands and CI gates: static analysis, SAST, secret scanning (see `Incident-Response-and-Monitoring.md`).
- How Copilot should use this folder:
  - Prefer secure defaults in suggestions (parameterized queries, safe deserialization, input validation).
  - If no security files exist, prompt the user before adding secrets or automatic fixes that would change security posture.

## testing/
- **Files:** `Unit-Tests.md`, `Integration-Tests.md`, `Functional-Tests.md`
- Purpose: canonical testing commands, single-test invocation examples, and testing standards.
- What to include:
  - Exact commands to run the full test suite and a single test per supported language/framework (Node, Python, Go, Rust, .NET, UI/E2E).
  - Guidance on test types (unit, integration, functional/e2e), test fixtures, handling flakiness, coverage collection, and parallelization.
  - CI best-practices and example job snippets to run tests reliably.
- How Copilot should use this folder:
  - Consult `testing/` first for authoritative test run commands, single-test examples, and CI practices before proposing run instructions or CI changes.
  - Prefer existing test scripts (npm script, tox, Makefile, task) and respect project-specific concurrency and environment setup.
  - When suggesting test commands, include exact single-test invocations and any required environment variables or fixtures.

## architecture/
- **Files:** `overview.md`, `software-architecture.md`, `cloud-architecture.md`, `microservices.md`, `databases.md`, `caching.md`, `mainframe-integration.md`, `websockets.md`
- Purpose: high-level overview describing core components, runtime layout, and data flow.
- What to include:
  - Short narrative describing the system, key services/modules, and runtime shape (see `overview.md`).
  - Cloud and microservices topology, database and caching strategies, mainframe integration patterns, and WebSocket usage.
- How Copilot should use this folder:
  - Use it to contextualize cross-file changes and to decide where new files or services belong.
  - Consult `databases.md` and `caching.md` when suggesting data access patterns or storage choices.

## conventions/
- **Files:** `style.md`, `dotnet.md`, `nodejs.md`, `python.md`, `html-css.md`
- Purpose: project-specific patterns that are not obvious from single files.
- What to include:
  - Naming conventions, error-handling patterns, preferred testing idioms, branch naming (`feature/`, `fix/`, `chore/`), and git commit message format (Conventional Commits) — see `style.md`.
  - Language-specific idioms in `dotnet.md`, `nodejs.md`, `python.md`, `html-css.md`.
- How Copilot should use this folder:
  - Follow these patterns when proposing changes or generating new files; when in doubt, ask the user which convention to follow.

## deployment/
- **Files:** `workflows.md`
- Purpose: deployment, release, and infrastructure automation rules and commands.
- What to include:
  - List of deployment pipelines and workflow files (paths under `.github/workflows/`), environment promotion rules, and rollback procedures.
  - Release, tagging, and deployment conventions, required checks for merges and deployment gates.
- How Copilot should use this folder:
  - Reference `workflows.md` when proposing workflow or deployment edits and ensure suggested changes maintain required gates and do not bypass deployment approvals.

## dependencies/
- **Files:** `management.md`
- Purpose: policies and commands for dependency management and upgrades.
- What to include:
  - How to add/update dependencies, preferred tools (Dependabot, Renovate), and commands to validate upgrades.
  - Guidelines for pinning versions vs. ranges.
- How Copilot should use this folder:
  - When suggesting dependency changes, include upgrade commands and test run steps from `management.md`; avoid blind upgrades without tests.

## documentation/
- **Files:** `docs-guidelines.md`, `business-docs.md`, `technical-docs.md`, `implementation-plan.md`
- Purpose: developer-facing and business-facing docs, API docs, and design notes.
- What to include:
  - Canonical guidelines for writing docs (see `docs-guidelines.md`), technical specs, business requirements, and the implementation plan.
- How Copilot should use this folder:
  - Update docs alongside code changes and propose documentation edits when adding public APIs.

## observability/
- **Files:** `logging-metrics.md`
- Purpose: tracing, logging, metrics, and how to validate observability artifacts.
- What to include:
  - Expected log formats, metrics endpoints, and tracing setup (see `logging-metrics.md`).
- How Copilot should use this folder:
  - Ensure instrumentation is added to new services and suggest sensible defaults for logging levels.

## performance/
- **Files:** `benchmarks.md`
- Purpose: performance testing commands and benchmarks.
- What to include:
  - Benchmark scripts, load-test commands (e.g., `wrk`, `k6`), and perf budgets (see `benchmarks.md`).
- How Copilot should use this folder:
  - When recommending algorithmic or data-structure changes, consider perf test coverage and propose benchmark updates.

---

## Existing content and discovery
- Begin by checking for language root files (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`) and CI definitions under `.github/workflows/`.
- Also check for other AI assistant config files and incorporate them: `CLAUDE.md`, `.cursorrules`, `AGENTS.md`, `.windsurfrules`, `CONVENTIONS.md`, `.clinerules`, `.github/copilot-instructions.md`.

## Quick start for future Copilot runs
- **Step 1:** Read `security/`, `architecture/overview.md`, and the folder structure above to ground the session.
- **Step 2:** If making changes, follow `conventions/style.md` for style and `deployment/workflows.md` for workflow edits.
- **Step 3:** Run commands from `testing/` and `dependencies/management.md` to validate changes.

## MCP servers
- If you want MCP servers configured (e.g., Playwright for web projects, or a test runner agent), say which server(s) to configure and basic target (browser/e2e) and they will be added.

## Maintainer notes
- Keep folder sections minimal but precise: prefer a single authoritative command for each common action and a one-sentence summary of intent.
- Update this file and the folder structure diagram when adding new folders, files, or toolchains; Copilot uses this file as the primary source of truth for navigation and automation suggestions.

(Generated from repository state at 2026-02-23)
