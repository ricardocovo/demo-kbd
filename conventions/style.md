# Project Style & Conventions (DEPRECATED)

This file has been split into framework-specific convention files under `.github/conventions/`:

- `.github/conventions/dotnet.md` — .NET/C# conventions
- `.github/conventions/python.md` — Python conventions
- `.github/conventions/nodejs.md` — Node.js / TypeScript / JavaScript conventions
- `.github/conventions/html-css.md` — HTML & CSS conventions

Please update links in READMEs and other docs to point to the new files. The original content was moved; keep this DEPRECATED placeholder for discovery.

  - Use kebab-case for filenames and directories: `user-service.ts`, `data-access/`.
  - Test files: append `.test` or `.spec` before extension: `user.test.js`, `auth.spec.ts`.

- Variables, functions, and methods
  - JavaScript/TypeScript: camelCase for variables and functions, PascalCase for classes and React components.
    - Good: `getUserById`, `UserRepository`
    - Bad: `get_user_by_id`, `userRepository` (when used as a class)
  - Python: snake_case for functions and variables, PascalCase for classes.
  - Go: mixedCase (golang convention) for exported identifiers start with uppercase.

- Constants
  - Use UPPER_SNAKE_CASE for global constants that are truly constant across the app (e.g., `DEFAULT_TIMEOUT_MS`).
  - Use `const` in JS/TS where appropriate.

- Tests and fixtures
  - Keep test names descriptive. Use the `should` or `when` style where helpful: `shouldReturnUserWhenIdExists` or `whenUserNotFound_thenReturns404`.

## 2. Error-handling Patterns

- General principles
  - Do not swallow errors. Always handle or propagate with context.
  - Distinguish between internal errors (log with debug info) and user-facing errors (return safe messages).
  - Centralize error logging and uniform error shapes for APIs.

- Language-specific notes
  - Go: prefer wrapping errors with context using `%w` and `fmt.Errorf`:

```go
if err != nil {
    return fmt.Errorf("fetching user %s: %w", userID, err)
}
```

  - JavaScript/TypeScript: use Error subclasses for typed errors, attach metadata in a field (avoid overloading message):

```ts
class NotFoundError extends Error {
  status = 404
  constructor(message: string) {
    super(message)
    this.name = 'NotFoundError'
  }
}

throw new NotFoundError('user not found')
```

  - Python: raise specific exceptions and include context. Wrap only when adding meaningful context.

- API error responses
  - Return structured error objects: `{ code: 'USER_NOT_FOUND', message: 'User not found' }`.
  - Log stack traces server-side, but return minimal details to clients.

## 3. Testing Idioms

- Fast, deterministic unit tests
  - Prefer unit tests that run fast and avoid external network or DB calls. Use mocks/fakes for dependencies.

- Table-driven tests (recommended where appropriate)
  - Example (Go):

```go
func TestAdd(t *testing.T) {
    cases := []struct{ a, b, want int }{
        {1, 2, 3},
        {2, 2, 4},
    }
    for _, c := range cases {
        t.Run(fmt.Sprintf("%d+%d", c.a, c.b), func(t *testing.T) {
            if got := Add(c.a, c.b); got != c.want {
                t.Fatalf("want %d, got %d", c.want, got)
            }
        })
    }
}
```

- Test organization
  - Group tests next to the code (same directory) using `.test` or `_test` naming conventions.
  - Integration tests should be separate and runnable via a different script (e.g., `npm run test:integration`).

- Determinism
  - Seed randomness in tests or avoid randomness. Use fixed timestamps or clock injection when testing time-dependent logic.

## 4. Git Commit Message Format

- Commit summary (first line)
  - Format: `<type>(<scope>): <short summary>`
  - Keep the first line <= 72 characters.
  - Types: feat, fix, docs, style, refactor, perf, test, chore

- Body
  - Leave a blank line after the summary and include a more detailed explanation if needed. Wrap lines at ~72 characters.

- Co-authorship requirement
  - If more than one person substantially contributed to a commit (pair programming, joint debugging, PR merges that combine work), you MUST include a `Co-authored-by` trailer for each additional author in the commit message. Example:

```
fix(auth): handle token expiry when refreshing

Ensure refresh token is validated before attempting exchange.

Co-authored-by: Jane Doe <jane@example.com>
Co-authored-by: John Smith <john@example.com>
```

- Example full commit message:

```
feat(api): add user profile endpoint

Add GET /users/:id to return public profile information.
Include caching headers and minimal fields to reduce payload size.

Co-authored-by: Pair Programmer <pair@example.com>
```

## 5. Code Snippet Examples

- JavaScript: naming & error handling

```js
// good: camelCase, Error subclass
class ValidationError extends Error {
  constructor(field, message) {
    super(message)
    this.field = field
    this.name = 'ValidationError'
  }
}

function validateUser(user) {
  if (!user.email) throw new ValidationError('email', 'email is required')
}
```

- Python: clear exceptions and naming

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

- Go: table-driven tests and error wrapping

```go
// repo.go
func (r *Repo) Get(id string) (*User, error) {
    u, err := r.db.Find(id)
    if err != nil {
        return nil, fmt.Errorf("repo get %s: %w", id, err)
    }
    return u, nil
}
```

## Notes

- These conventions are guidelines to keep the codebase consistent. When in doubt, prefer clarity and consistency with surrounding code.

