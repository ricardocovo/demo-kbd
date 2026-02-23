# Documentation Guidelines

This document describes where to place repository documentation, how to update the README when the public API changes, commands for generating API docs, and the ADR (Architecture Decision Record) process.

## Where to place docs

- README.md (root)
  - Short, high-level overview of the repository: purpose, quick start, supported platforms, and references to more detailed docs.
  - Keep usage examples, installation instructions, and a brief public API summary here.
  - For public libraries, include a short "Public API" section (or a link to API docs) that lists the most important exported types/functions and example usage.

- docs/ (top-level)
  - Use for end-user and developer-facing documentation that is longer than what belongs in the README.
  - Organize by topic, e.g. docs/getting-started.md, docs/usage.md, docs/configuration.md, docs/internals.md.
  - If you publish a docs site (e.g., using MkDocs, Docusaurus, Hugo), the docs/ folder should contain the source for the site.

- docs/api/ or api-docs/
  - Generated API reference output or hand-written API guides live here. If your project generates API docs, commit either the generated output to this folder (for hosted sites) or commit a small README with instructions and keep generated artifacts out of the repo (preferred for most projects).

- adr/
  - Store Architecture Decision Records here. Each ADR is a single Markdown file named with an incremented number and short-title, e.g. `adr/0001-record-utility-library.md`.
  - ADRs capture important architecture decisions and their context, alternatives, and consequences.

## Updating README on public API changes

When the repository's public API (exported functions, types, endpoints, or contract) changes, do the following:

1. Update README.md to reflect the change if it affects the quick-start, examples, or the short Public API summary.
2. Add or update CHANGELOG.md (if present) with a brief entry describing the incompatible or notable change. Mark breaking changes clearly.
3. If you maintain a semantically versioned release process, bump the version and note the API change in the release notes.
4. If the change affects consumers, consider adding migration notes under docs/migration.md with code examples showing before/after usage.
5. Run and commit API documentation generation (see below) or update hand-written API docs so published references remain correct.

Checklist for PRs that change public API:

- [ ] README updated (examples & short API summary) if necessary
- [ ] docs/ updated (examples, migration guides) if necessary
- [ ] CHANGELOG.md updated with a clear description
- [ ] API reference regenerated and published (if automated)
- [ ] Tests updated to cover new/changed behaviour

## API doc generation commands (examples)

Below are common example commands. Adjust for the languages/tools used in this repository.

- TypeScript / JavaScript (TypeDoc)
  - Install: `npm install --save-dev typedoc`
  - Generate: `npx typedoc --out docs/api src`

- Python (Sphinx & autodoc)
  - Init (once): `sphinx-quickstart docs/sphinx` and enable autodoc in conf.py
  - Generate: `sphinx-apidoc -o docs/sphinx/source mypackage` then `sphinx-build -b html docs/sphinx/source docs/sphinx/build`

- Go
  - Generate: `godoc -http=:6060` or use `pkgsite` generation; for static HTML consider `gddo` or `go doc` tooling as appropriate.

- Java (Javadoc)
  - Generate: `javadoc -d docs/api -sourcepath src/main/java -subpackages com.example`

- Generic: Keep the generated output in docs/api/ when you want the docs versioned with the repository, or generate during CI and publish to your docs host (GitHub Pages, Netlify, etc.).

Include a short task in the project's CI describing how to build and publish API docs.

## ADR (Architecture Decision Record) process

Purpose: capture architectural decisions that have significant impact on the project.

Location: adr/ (top-level) or docs/adr/

Naming convention: `adr/NNNN-short-title.md` where NNNN is a zero-padded incrementing number (0001, 0002, ...).

Process:
1. Create an ADR proposal file using the template below and add it to `adr/`.
2. Open a PR that includes the ADR and changes (if any) required to implement it (or a follow-up issue/PR).
3. Discuss in the PR. When consensus is reached, merge the ADR. If the decision is provisional, mark it as `Proposed` or `Draft` in the status field.
4. If the decision is later changed, add a new ADR that records the changed decision and references the earlier ADR.

ADR template (copy to `adr/NNNN-short-title.md`):

---
Title: Short descriptive title
Status: Proposed | Accepted | Deprecated | Rejected
Date: YYYY-MM-DD
Deciders: List of people/roles who decided

Context
-------
What is the issue? Why does it matter?

Decision
--------
What is the chosen option?

Consequences
------------
What becomes easier/harder? Any tradeoffs or follow-ups?

---

Sample README snippet for public API section

```
## Public API

This project exposes the following public API (high-level):

- FooDoer.doThing(options): Does the main thing. See docs/api/FooDoer.md for full reference.
- BarClient.request(endpoint, params): Sends a request to the internal service and returns a typed response.

For full API reference, see docs/api/ or the generated site at <link>.
```

Sample minimal ADR (adr/0001-example.md)

```
Title: Choose HTTP client library
Status: Accepted
Date: 2026-02-23
Deciders: Alice, Bob

Context
-------
We need an HTTP client library that works in both server and CLI environments. Options considered: lib-a, lib-b, lib-c.

Decision
--------
We will use lib-b because it supports timeouts, retries, and has a permissive license.

Consequences
------------
- Integration work required for reconciling error types
- Tests need to be updated to mock the new client
```

Guidance: keep docs short in README and move long-form documentation to docs/. Use ADRs to record important decisions and keep them discoverable.

---

If you need a tailored version of these commands for your project's language/tooling, add notes to the top of this file describing what language the repository uses and the canonical commands to generate API docs.
