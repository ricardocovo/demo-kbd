# Integration Tests

Purpose
- Validate interactions across system boundaries, including external services, message brokers, storage systems, and cross-service contracts.

Characteristics
- Integration tests exercise realistic stacks and often require provisioning of external services (DB, MQ, caches, third-party APIs).
- Slower and more expensive; typically run in gated pipelines, staging environments, or scheduled jobs.

Patterns and tooling
- Use Testcontainers or docker-compose to provision dependencies deterministically in CI.
- For cloud-dependent services, run integration tests against dedicated staging environments with sanitized data.

Single-test invocation examples
- Python (pytest): `pytest tests/integration/test_integration.py::test_flow`
- .NET: `dotnet test --filter "Category=Integration"`
- Node: `npm run test:integration -- tests/integration/some.test.js`

Contract testing
- Use consumer-driven contract tools (Pact) to validate API contracts between services and publish verifications as part of CI.

Data and environment hygiene
- Never use production PII; use anonymized datasets or generated fixtures.
- Maintain seeded fixtures and migration scripts to bring test environments to expected schema versions.

Reliability & observability
- Capture rich logs, traces, and metrics during integration test runs to diagnose failures.
- Implement retry and backoff for known transient errors in external services but avoid masking genuine regressions.

Scheduling strategy
- Run exhaustive integration suites on protected branches, nightly runs, or pre-release gates depending on cost and duration.
- Use incremental or targeted integration tests during pull requests where feasible (smoke tests).

CI integration examples
- Start dependent services via docker-compose in a job step, run tests, then tear down.
- Upload artifacts (logs, traces, coverage) for post-mortem analysis on failures.
