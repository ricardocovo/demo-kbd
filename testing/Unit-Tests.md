# Unit Tests

Purpose
- Fast, deterministic tests that exercise a single unit of code (function, class, or small module) in isolation.

Guiding principles
- No network, DB, or filesystem dependencies. Use mocks, fakes, or dependency injection for external interactions.
- Keep tests small and fast (aim for <100ms for individual tests where feasible).
- Name tests clearly: `test_should_do_x_when_y` or `shouldDoXWhenY` depending on language conventions.

Single-test commands (examples)
- Node (Jest): `npx jest -t "pattern"` or `npm test -- -t "pattern"`
- Python (unittest discovery): `python -m unittest tests.test_module.TestClass.test_method`
- Python (pytest): `python -m pytest tests/test_module.py::test_function`
- Go: `go test ./pkg -run ^TestName$`
- Rust: `cargo test test_name -- --exact`
- .NET: `dotnet test --filter "FullyQualifiedName~Namespace.Class.TestName"`

Framework-specific notes
- Python (unittest): use `setUp`/`tearDown` for per-test fixtures and `setUpClass`/`tearDownClass` for class-level setup. Prefer pytest for richer features but `unittest` is available in stdlib.
- .NET (xUnit/NUnit/MSTest): use fixture constructs (IClassFixture/CollectionFixture) for shared expensive resources; mark unit tests with `[Fact]` or equivalent and categorize with traits where supported.
- Node (Jest/Mocha): prefer Jest for new projects; use jest `--runInBand` for single-process debugging.

Best practices
- Avoid global state; reset singletons between tests or design for injection.
- Seed or freeze time in tests (clock injection or time-mocking) for deterministic behavior.
- Use test doubles for external dependencies and keep assertions focused on behavior.

Coverage
- Collect coverage for unit tests and report via CI; require coverage thresholds for critical modules.

Flakiness mitigation
- Avoid sleep-based waits; use condition polling with timeouts.
- Keep tests hermetic and deterministic.
