# Build, Test, and Lint — Common Toolchains

This document lists exact commands and examples for running the full build/test/lint suites and single-test invocations for common toolchains. Use these locally and in CI.

---

## Node (npm)

- Install deps:

  npm install

- Run full test suite:

  npm test

  or if your project uses scripts:

  npm run test

- Run a single test (Jest example, match by name/pattern):

  npm test -- -t "pattern"

  or:

  npm run test -- -t "pattern"

- Build (if applicable):

  npm run build

Notes: pass `--` before args that should be forwarded to the test runner.

---

## Python (pytest)

- Install deps (example using venv):

  python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt

- Run full test suite:

  pytest

- Run a single test (file::testname):

  pytest tests/test_module.py::test_function

- Run tests matching a substring:

  pytest -k "substring"

- Run with verbose and capture disabled (useful locally):

  pytest -v -s

---

## Go

- Fetch deps (modules):

  go mod download

- Run full test suite (all packages in module):

  go test ./...

- Run tests for a package:

  go test ./pkg/name

- Run a single test by name (regex):

  go test -run '^TestMyFunction$' ./pkg/name

- Run with race detector:

  go test -race ./...

---

## Rust

- Fetch deps/build toolchain: Rustup + cargo

- Run full test suite:

  cargo test

- Run a single test (name or substring):

  cargo test my_test_name

- Run exact test name match:

  cargo test my_test_name -- --exact

- Run with release optimizations (if needed):

  cargo test --release

---

## Generic Makefile

- Typical full test invocation (depends on project):

  make test

  or if make defines a `check` target:

  make check

- Run a single test via environment or arguments (varies by Makefile):

  make test TEST=TestMyFunction

  or invoke the underlying command directly (recommended) e.g., `go test -run '^TestX$'` or `pytest tests/foo.py::test_bar`.

Note: Make targets vary by repository. Prefer invoking toolchain commands directly for single tests when Makefile does not expose a convenient parameter.

---

## Linters & Formatters — Guidance

These are recommended commands for common linters/formatters and how to run them locally and in CI.

Node (ESLint / Prettier):

- Check linting (no fix):

  npx eslint . --ext .js,.jsx,.ts,.tsx

- Fix issues automatically:

  npx eslint . --ext .js,.jsx,.ts,.tsx --fix

- Check formatting with Prettier:

  npx prettier --check "**/*.{js,jsx,ts,tsx,json,md,css,scss}" 

- Format with Prettier:

  npx prettier --write "**/*.{js,jsx,ts,tsx,json,md,css,scss}"

Python (ruff / flake8 / black):

- Lint with ruff:

  ruff check .

- Auto-fix with ruff (where supported):

  ruff check --fix .

- Format (Black):

  black .

- Check formatting only:

  black --check .

Go (gofmt / golangci-lint / go vet):

- Show unformatted files:

  gofmt -l .

- Format files in place:

  gofmt -w .

- Run vet:

  go vet ./...

- Run golangci-lint (if used):

  golangci-lint run

Rust (rustfmt / clippy):

- Check formatting:

  cargo fmt -- --check

- Format in place:

  cargo fmt

- Lint (Clippy):

  cargo clippy --all-targets --all-features -- -D warnings

---

## Running locally vs CI

- Locally: Use the above commands in your shell/IDE. Prefer running formatters with `--check` in CI and `--write` or equivalent locally to auto-apply fixes.

- In CI: Add explicit steps that:
  - Install dependencies
  - Run the full test suite for the repository's language(s)
  - Run linters with check-only flags (so CI fails on style issues)
  - Run formatters in check mode (e.g., `black --check`, `cargo fmt -- --check`, `prettier --check`)

Example CI snippet (pseudo-YAML):

  - name: Install
    run: npm ci
  - name: Lint
    run: npx eslint . --ext .js,.ts
  - name: Format check
    run: npx prettier --check "**/*"
  - name: Test
    run: npm test

- Parallelization: When possible, run language-specific jobs in parallel (e.g., Node tests in one job, Python tests in another) to reduce total CI time.

- Caching: Cache language-specific dependencies (npm, pip, Go modules, cargo) to speed up CI runs.

---

## Tips

- Prefer invoking the toolchain directly for single-test runs (e.g., `pytest file::test`, `go test -run`, `cargo test name`) rather than through wrappers when debugging.
- Keep CI checks strict: lint/format checks should fail the build to maintain consistent code quality.

---


## .NET (dotnet)

- Restore & build:

  dotnet restore && dotnet build --no-restore

- Run full tests:

  dotnet test

- Run a single test (filter by fully-qualified name or traits):

  dotnet test --filter "FullyQualifiedName~Namespace.Class.MethodName"

  or filter by category/trait (e.g., `Category=Unit`):

  dotnet test --filter "Category=Unit"

- CI tips: `dotnet test --no-build --verbosity normal`; enforce formatting with `dotnet format --verify-no-changes`.

---

## DoneJS

- Install deps:

  npm ci

- Build (CLI or npm script):

  npx donejs build

  or

  npm run build

- Test (CLI wrapper, underlying runner varies by project):

  npx donejs test

- Run a single test: invoke the underlying test runner directly or pass path/pattern to the CLI, e.g.:

  npx donejs test test/path/to/file.test.js

  or

  npm test -- test/path/to/file.test.js

- Notes: DoneJS projects may use Karma/Jasmine or another runner; inspect package.json to learn the underlying test command.

---

# Testing Standards: Unit, Functional (Integration), and UI (E2E)

These standards unify expectations across languages and frameworks and help keep tests reliable and fast.

Unit tests
- Definition: small, fast, deterministic tests that exercise a single function or class in isolation.
- Characteristics: no network or DB, use mocks/fakes for external dependencies, run in-memory, complete in milliseconds.
- Naming: `test_function_name` or `should_do_x_when_y`; keep names descriptive and table-driven tests for multiple cases.
- Speed: aim for <100ms per test on CI where feasible; keep overall unit suite under a few minutes.
- Running single-unit tests examples:
  - Node/Jest: `npm test -- -t "pattern"`
  - Python/pytest: `pytest tests/test_module.py::test_function`
  - .NET: `dotnet test --filter "FullyQualifiedName~Namespace.Class.MethodName"`
- Best practices: avoid global state, isolate time via clock injection or fixed seeds, run with coverage enabled in CI.

Functional / Integration tests
- Definition: exercise interactions between components (DB, caches, external services) to validate end-to-end behavior of a feature.
- Characteristics: slower than unit tests, may require containerized services or test doubles for costly external providers.
- Isolation: use ephemeral test databases, docker-compose/testcontainers, and clear state between tests.
- Tagging: mark integration tests (pytest marker `integration`, xUnit Category, or Jest project) and exclude them from quick local runs.
- Running examples:
  - pytest integration marker: `pytest -m integration`
  - Go: use build tags or separate integration packages (e.g., `go test ./... -tags=integration`)
  - .NET: `dotnet test --filter "Category=Integration"`
- Best practices: seed test data programmatically, ensure idempotency, write clear setup/teardown, and use stable endpoints.

UI / End-to-End tests
- Definition: tests that run in a real browser (or headless) to validate user flows and UI correctness.
- Tools: Playwright, Cypress, Selenium, or Puppeteer. Prefer Playwright or Cypress for modern apps.
- Stability: use robust selectors (data-testid or data-test attributes), avoid CSS-dependent selectors, and wait for stable application states.
- Running examples:
  - Playwright: `npx playwright test` (single test: `npx playwright test tests/example.spec.ts -g "title"`)
  - Cypress: `npx cypress run --spec "cypress/e2e/my_spec.cy.js"`
- CI considerations: run E2E in a dedicated job with isolated test environment; use video/screenshots for failures; retry flaky tests sparingly.

Flakiness and reliability
- Reduce flakiness by:
  - Avoiding timing-based assertions; wait for state/DOM conditions.
  - Using retries only as a last resort and instrumenting flaky test reports.
  - Running tests in the same environment locally and in CI (containers/localstack for AWS, etc.).

Test data and fixtures
- Use factories and fixtures to generate deterministic test data; prefer in-memory stores or ephemeral DBs.
- Do not use production data in tests; sanitize or synthesize realistic fixtures.
- Keep fixtures versioned with the test suite or provision them as code (seed scripts).

Coverage and quality gates
- Require a minimum coverage threshold for unit tests (e.g., 70–90% depending on risk). Enforce via CI tools (coverage.py, nyc, dotnet coverage tools).
- Prefer meaningful coverage metrics (functionality-critical modules) over raw percentage; complement with mutation testing where appropriate.

Parallelization and performance
- Split test suites into parallel CI shards (unit vs integration vs e2e) and run language jobs concurrently.
- Cache dependencies to speed CI and use test runners' parallel options (`pytest -n`, `jest --runInBand` vs default parallel, `dotnet test` options).

Maintenance
- Fail builds on new flaky tests; triage and either fix or quarantine with a clear ticket.
- Document test ownership and where to find related test infra (Docker images, fixtures, mock servers).

Document created by repository guidelines.
