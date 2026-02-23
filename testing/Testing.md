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

Unit testing ("unittest" style) — per supported stack

- Python (unittest)
  - Structure: tests in `tests/` package, module-level `TestCase` classes that inherit from `unittest.TestCase`.
  - Run discovery: `python -m unittest discover -s tests -p "test_*.py"` or run a single test: `python -m unittest tests.test_module.TestClass.test_method`
  - Use `setUp`/`tearDown` or `setUpClass`/`tearDownClass` for fixtures; prefer pytest for richer features but `unittest` is appropriate for stdlib-only projects.
  - Assertions: use `self.assertEqual`, `self.assertRaises`, `self.assertTrue`, etc.

- .NET (xUnit / MSTest / NUnit — "unittest" concepts)
  - xUnit example: decorate methods with `[Fact]` for unit tests and `[Theory]` + `[InlineData]` for parameterized cases. MSTest uses `[TestMethod]`, NUnit uses `[Test]`.
  - Run a single test via `dotnet test --filter "FullyQualifiedName~Namespace.Class.MethodName"` or filter by trait/category: `dotnet test --filter "Category=Unit"`.
  - Use test fixtures (IClassFixture / CollectionFixture in xUnit) for shared expensive setup.

- Node / DoneJS (Jest, Mocha, Jasmine — "unit test")
  - Jest: test files `*.test.js` or `*.spec.js`; run a single test by name: `npx jest -t "pattern"` or run a file: `npx jest path/to/file.test.js`.
  - Mocha: `npx mocha path/to/test.js --grep "pattern"`.
  - Prefer small, fast tests without network/DB dependencies; use test doubles (sinon, testdouble) or dependency injection.

Functional & Integration tests — per supported stack

- Python
  - Mark integration tests with pytest markers: `@pytest.mark.integration` and run via `pytest -m integration`.
  - Use pytest fixtures scoped to module/session to manage resources like DB containers (pytest-docker/testcontainers).

- .NET
  - Use `[Trait("Category","Integration")]` or custom `Category` attributes and run with `dotnet test --filter "Category=Integration"`.
  - Use TestServer (Microsoft.AspNetCore.TestHost) or Docker-based dependencies for realistic integration tests.

- Node / DoneJS
  - Place integration tests under `test/integration/` and expose npm scripts: `"test:integration": "mocha test/integration --reporter spec"` or similar for Jest.
  - Use testcontainers-node or docker-compose to provision dependent services in CI.

UI / E2E and cross-stack functional flows

- Drive end-to-end flows from any stack using Playwright or Cypress; keep adapters thin so backend teams can exercise flows independently.
- For cross-stack contract testing, use consumer-driven contract frameworks (Pact) where appropriate.

- CI orchestration
  - Run unit tests on push/PR. Run integration/functional suites on merge-to-main or protected-branch jobs, or on scheduled/nightly runs depending on runtime cost.

- Examples: single-test invocations (recap)
  - Python unittest single: `python -m unittest tests.test_module.TestClass.test_method`
  - pytest single (if used): `python -m pytest tests/test_module.py::test_function`
  - dotnet single: `dotnet test --filter "FullyQualifiedName~Namespace.Class.TestName"`
  - Jest single: `npx jest -t "pattern"`

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
