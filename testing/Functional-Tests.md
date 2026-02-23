# Functional Tests

Purpose
- Validate interactions between multiple components to ensure a feature works end-to-end within a service boundary (e.g., API + DB + cache).

Characteristics
- Slower than unit tests but faster and more focused than full integration or E2E tests.
- May use in-memory stores, lightweight containers, or test doubles for certain external systems.

Execution patterns
- Run against ephemeral resources provisioned in CI (test containers, docker-compose) or lightweight in-memory substitutes.
- Tag or place functional tests in a separate folder (e.g., `tests/functional/`) and expose CI scripts like `test:functional`.

Single-test invocation examples
- Python (pytest marker): `pytest tests/functional/test_feature.py::test_scenario`
- .NET: `dotnet test --filter "Category=Functional" -- TestCaseFilter` (use traits/categories)
- Node: `npx jest tests/functional/my.feature.test.js -t "pattern"`

Framework guidance
- Python: use pytest fixtures scoped to module or session for shared resources and pytest-docker or testcontainers for dependent services.
- .NET: use TestServer (Microsoft.AspNetCore.TestHost) for in-memory host where appropriate; for external dependencies use Docker images in CI.
- Node/DoneJS: use testcontainers-node or docker-compose to provision DBs and queues for functional validation.

State management
- Ensure functional tests clean up state between runs; use transactional rollbacks or tearDown hooks to restore cleanliness.
- Use idempotent seeding scripts for predictable starting data.

Flakiness and reliability
- Prefer deterministic resource provisioning and avoid network flakiness by using local containerized dependencies in CI.
- Record and upload logs/artifacts for failed functional tests to aid debugging.

CI recommendations
- Run functional tests on merge-to-main or nightly, or in a dedicated job that runs after unit tests pass.
- Parallelize functional tests where possible by sharding test files across runners.
