# Node.js / JavaScript / TypeScript Conventions

Purpose
- Conventions for Node.js services, libraries, and frontend code using JavaScript or TypeScript.

Project layout
- Keep source in `src/` for libraries and `packages/` for monorepos, tests in `__tests__` or `tests/`.
- Use `tsconfig.json` for TypeScript projects and `package.json` scripts for common tasks.

Formatting and linting
- Use Prettier for formatting and ESLint for linting. Run `prettier --check` and `eslint --max-warnings=0` in CI.
- Enforce consistent imports and avoid `console.log` in production code; use a logging library.

TypeScript
- Enable `strict` mode in `tsconfig.json` for new projects. Prefer explicit types for public APIs.

Error handling
- Use `async/await` with try/catch. Where appropriate, centralize error handling middleware for Express/Koa.
- Prefer specific error classes to carry status codes for HTTP handlers.

Testing
- Use Jest for unit tests or Mocha+Chai where preferred. Keep unit tests isolated and fast.
- Run a single test: `npm test -- -t "pattern"` or `npx jest path/to/test -t "name"`.
- Integration tests and E2E should be in separate scripts (e.g., `test:integration`, `test:e2e`).

Dependency management
- Use `npm ci` in CI for reproducible installs. Commit `package-lock.json`.
- Use Renovate/Dependabot for automated updates and review major changes.

Examples
- Test example (Jest):

```ts
import { sum } from '../src/math'

test('adds two numbers', () => {
  expect(sum(1, 2)).toBe(3)
})
```

Notes
- For monorepos, consider tools like Nx or Lerna and define standard scripts at root for building and testing.
