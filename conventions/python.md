# Python Conventions

Purpose
- Conventions to keep Python code consistent across the repository.

Project layout
- Prefer `src/` layout for packages: `src/my_package/...` and tests in `tests/`.
- Use virtual environments (`venv` or `virtualenv`) and pin dependencies via `requirements.txt` or `pyproject.toml` + `poetry`.

Naming and typing
- Follow PEP8 for naming: snake_case for functions and variables, PascalCase for classes.
- Use type hints consistently; run `mypy` in CI for critical modules.

Formatting and linting
- Use `black` for formatting: `black .` and `black --check .` in CI.
- Use `ruff` or `flake8` for linting: `ruff check .`.
- Use `isort` for import sorting: `isort .` and `isort --check-only .` in CI.

Testing
- Use pytest; keep unit tests fast and deterministic.
- Test file naming: `tests/test_module.py` and test functions `def test_...()`.
- Run a single test: `pytest tests/test_module.py::test_function`.
- Use fixtures for shared setup and parametrize for table-driven tests.
- For integration tests that require services, use test containers (pytest-docker or testcontainers-python) and mark them as `integration` to exclude by default: `pytest -m "integration"`.

Mocking
- Use `unittest.mock` or `pytest-mock` for mocking. Prefer dependency injection for easier testing.

Type checking
- Run `mypy --strict` for modules that require strong guarantees; adopt gradually.

Security
- Avoid storing secrets in source; load from environment or secret managers.
- Run Bandit or Semgrep for SAST checks focusing on Python patterns.

Example snippet

```py
class ConfigError(Exception):
    pass


def load_config(path: str) -> dict:
    try:
        with open(path) as f:
            return json.load(f)
    except FileNotFoundError as e:
        raise ConfigError(f"config not found: {path}") from e
```
