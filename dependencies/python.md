# Python Dependency Management (pip / Poetry / pip-tools)

Purpose
- Provide guidance for managing Python dependencies, lockfiles, private indexes, and CI practices.

Lockfile strategies
- Poetry: prefer `pyproject.toml` + `poetry.lock` for modern projects. Use `poetry lock` to update and `poetry install --no-interaction --no-ansi` in CI.
- pip-tools: source lists (`requirements.in`) compiled to `requirements.txt` with `pip-compile`. Commit the compiled `requirements.txt` for reproducible installs and run `pip install -r requirements.txt` in CI.
- Plain pip: if using `requirements.txt`, pin exact versions (`package==1.2.3`) for production dependencies and use `pip-compile` where possible.

Installation commands (CI)
- poetry: `poetry install --no-interaction --no-ansi`
- pip (from requirements): `python -m pip install -r requirements.txt`
- Use wheels in CI to avoid expensive builds: build wheels in a preparatory job or rely on manylinux wheels from PyPI.

Private registries and credentials
- Configure private indexes via `pip.conf` or `POETRY_HTTP_BASIC_<name>` environment variables for Poetry.
- Store credentials in CI secrets; never commit credentials to source.

Vulnerability scanning and SBOM
- Use `pip-audit` or `safety` for Python package vulnerabilities.
- Generate SBOMs with Syft: `syft . -o spdx-json > sbom.spdx.json` and scan with Grype: `grype sbom:sbom.spdx.json`.

Native extensions and platform-specific wheels
- Pin platform-specific wheels where reproducibility is important; run sdist and wheel builds in CI to validate builds.
- Use manylinux wheels or prebuilt binaries for commonly used native libraries to avoid runtime compilation.

Dev workflows and local dev
- Use a venv for local development: `python -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt`.
- Recommend `pip-compile` + `pip-sync` or Poetry for consistent dev environments.

Testing & CI
- Cache pip wheel cache in CI and reuse between runs to speed builds.
- Run `pip-audit` or `safety` in CI; treat critical findings as actionable and follow the advisory SLAs in management.md.

Best practices summary
- Commit lockfiles (`poetry.lock` or compiled `requirements.txt`).
- Use SBOMs and vulnerability scanners in CI.
- Keep private registry credentials in secrets and use ephemeral tokens where supported.
