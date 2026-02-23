# demo-kbd

NOTE: DEMO / SAMPLE REPOSITORY — This repository is provided for demonstration purposes and is NOT a complete or prescriptive set of production standards. Use these documents as examples and consult your organization's policies and platform teams before adopting them.

This repository is a minimal demo augmented with a comprehensive set of repository-level documentation to guide development, security, testing, deployment, and architecture.

Docs (primary):
- .github/security/ — General security, API security, AuthN/AuthZ, FSI guidance, secrets, incident response, supply chain
- .github/build-test-lint/ — Build, test, and lint commands (including single-test examples) and testing standards
- .github/architecture/ — High-level software and cloud architecture, microservices, websockets, DBs, caching
- .github/conventions/ — Framework-specific coding conventions (.NET, Python, Node.js, HTML/CSS)
- .github/deployment/ — Deployment and release pipeline guidance (was CI-CD)
- .github/dependencies/ — Dependency & supply chain policies, SBOM, scanning, framework notes
- .github/documentation/ — Business docs, technical docs, and implementation plan templates
- .github/observability/ — Logging, metrics, tracing and Azure-specific observability guidance
- .github/performance/ — Benchmarks and performance testing guidance

Quick start:
1. Read `.github/security/`, `.github/build-test-lint/`, and `.github/architecture/` to ground the session.
2. Use the Build-Test-Lint doc for authoritative commands to run the test suite for the project's language.
3. Follow conventions in `.github/conventions/` when creating or modifying code.

Contributing:
- Follow the repository conventions and include the required Co-authored-by trailer for multi-author commits.
- Update corresponding docs under `.github/` when adding new toolchains, CI workflows, or infra.

If you need a diagram, CI snippets, or an infra example for a specific cloud provider, open an issue or request and it will be added.
