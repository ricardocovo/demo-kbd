# Testing standards and single-test commands

This document provides testing-focused guidance and authoritative single-test commands across common languages and frameworks used in this repository. It also covers testing taxonomy (unit, integration, UI/e2e), flakiness mitigation, fixtures, coverage, parallelization, and CI best practices.

1. Quick single-test command reference

- Node (Jest/Mocha/Vitest)
  - Run full test suite: npm test or npm run test
  - Run a single test by name (Jest): npx jest -t "Test name" --runInBand
  - Run a single file: npx jest path/to/file.test.js
  - If tests are exposed via npm scripts: npm run test -- -t "Test name"

- Python (pytest)
  - Run full suite: python -m pytest
  - Run a single test function: python -m pytest tests/module/test_file.py::test_function
  - Run a single test by keyword: python -m pytest -k "test_name"

- Go
  - Run full suite: go test ./...
  - Run a single test: go test ./pkg -run ^TestName$
  - Run a single file: go test ./path/to/package -run ^TestFunc$

- Rust (cargo)
  - Run full suite: cargo test
  - Run a single test: cargo test test_name -- --exact

- .NET (dotnet)
  - Run full suite: dotnet test
  - Run a single test by filter: dotnet test --filter "FullyQualifiedName~Namespace.Class.TestName" or --filter "Name=TestName"

- DoneJS / App-specific test runners
  - Many DoneJS projects expose tests via npm scripts. Preferred invocation: npm test or npm run test:donejs
  - For single-test runs, prefer the upstream test runner flags (e.g., the underlying test framework's -t or --grep style flags).

- UI / E2E
  - Playwright:
    - Run full suite: npx playwright test
    - Run a single test by title: npx playwright test -g "test title"
    - Run a single file: npx playwright test tests/my.spec.ts
  - Cypress:
    - Run full suite (headed): npx cypress open
    - Run headless single spec: npx cypress run --spec "cypress/e2e/spec.cy.js"
    - Use --env to provide necessary environment variables for CI-run tests

2. Testing types and expectations

- Unit tests
  - Fast, isolated, no external network or DB dependencies.
  - Use mocks/fakes for external calls. Keep runtime under ~100ms per test where possible.

- Integration tests
  - Validate interaction of multiple components (DB, queues, services).
  - Prefer lightweight local dependencies (test containers, in-memory DBs) in CI to improve reliability.

- UI / E2E tests
  - Higher-level acceptance tests validating user flows.
  - Run fewer and more targeted E2E tests in CI; reserve exhaustive suites for nightly/scale runs.

3. Fixtures and test data

- Use fixtures to create predictable, minimal test data.
- Keep fixtures small and composable. Prefer factory patterns to large static datasets.
- For stateful services, include setup/teardown or transactional rollbacks so tests leave no residual state.

4. Flakiness mitigation

- Identify flakiness patterns: timeouts, async ordering, external service rate-limits.
- Prefer deterministic waits (polling for condition) over fixed sleeps.
- Increase timeouts conservatively and only when justified; prefer retry for idempotent asserts.
- Isolate flaky tests into a quarantine runner or mark them as flaky in CI to avoid blocking main pipelines.

5. Coverage

- Collect coverage on unit and integration tests where practical.
- Configure thresholds carefully (e.g., require critical modules to have coverage gates but avoid blocking PRs for marginal drops).
- Publish coverage reports (lcov, Cobertura) to CI artifacts and use quality gates where appropriate.

6. Parallelization and performance

- Run tests in parallel where test isolation is guaranteed (Jest, pytest -n, go test -p).
- For resource-sensitive tests (DB, file system), use sharding or pool-managed fixtures to avoid contention.
- Balance test runtime vs. reliability: excessive parallelism can cause flaky resource contention.

7. CI practices

- Run fast unit tests on push/PR events. Run longer integration and E2E tests on protected branches or nightly schedules.
- Preserve reproducible environments using containerized services, docker-compose, or test containers.
- Cache dependencies between CI runs to speed up installs (npm ci cache, pip wheel cache, cargo registry cache, etc.).
- Capture and upload test artifacts (screenshots, videos, logs, coverage reports) for failed UI/E2E tests.
- Use job matrixes to test multiple runtimes and OS targets when relevant.

8. Language/framework specific notes and examples

- Node
  - Add npm scripts for common single-test and watch workflows: "test", "test:watch", "test:ci".
  - Use --runInBand for debug/single-process runs when necessary.

- Python
  - Use tox or nativem virtualenvs to manage interpreters.
  - Use pytest markers to separate unit/integration and to select tests in CI (pytest -m "integration").

- Go
  - Keep small packages to allow fast targeted go test runs.
  - Use -run with anchors (^...$) for precise test selection.

- Rust
  - Use cargo test -- --show-output to see failed test output in CI logs.

- .NET
  - Use test categories and filters to run targeted subsets in CI.

- UI / E2E
  - For Playwright, prefer test.describe.parallel only when tests are fully isolated.
  - Persist recordings only for failures to save storage; upload artifacts for debugging.

9. Troubleshooting and signals

- When a test fails intermittently, gather: flake frequency, environment differences, recent code changes, resource usage.
- Add diagnostic logging or increase verbosity for failing CI jobs and capture artifacts.

10. Adding new test toolchains

- When introducing a new language or framework, add a section to this file with exact single-test commands, CI snippets, and recommended npm/Makefile scripts.

Appendix: Example single-test commands summary

- Jest: npx jest -t "should do thing"
- pytest: python -m pytest tests/some_test.py::test_func
- go: go test ./pkg -run ^TestFunc$
- cargo: cargo test test_name -- --exact
- dotnet: dotnet test --filter "Name=TestName"
- playwright: npx playwright test -g "test title"
- cypress: npx cypress run --spec "cypress/e2e/spec.cy.js"
