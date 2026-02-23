# .NET / C# Conventions

Purpose
- Provide idiomatic conventions for C#/.NET projects to ensure consistency across solutions, libraries, and services.

Project layout
- Use a solution file at repo root for multi-project repositories: `MyOrg.sln`.
- Place project files under `src/ProjectName/ProjectName.csproj` and tests under `test/ProjectName.Tests`.
- Use `Directory.Build.props` to centralize SDK versions and common configuration.

Naming
- Use PascalCase for namespaces, classes, and public members: `MyService`, `UserRepository`.
- Private fields: `_camelCase` or `camelCase` depending on style; prefer `_underscorePrefix`.
- Files: one top-level `public` type per file, file name matches the type: `UserService.cs`.

Formatting and analyzers
- Use `dotnet format` to format code. Run as: `dotnet tool run dotnet-format` or `dotnet format` with SDK support.
- Enable Roslyn analyzers and recommended rulesets (StyleCop, Microsoft.CodeAnalysis). Treat analyzer warnings as build warnings, optionally as errors in CI: `dotnet build -warnaserror`.

Build and test commands
- Restore & build: `dotnet restore && dotnet build --no-restore`
- Run tests: `dotnet test` (runs all tests in solution)
- Run a single test: `dotnet test --filter "FullyQualifiedName~Namespace.Class.MethodName"`

Dependency management
- Prefer NuGet packages from official feeds; pin versions in project files.
- Use `nuget.config` for private feeds and credential management.

Async patterns
- Use `async`/`await` throughout for IO-bound work. Avoid `Task.Result` and `Task.Wait()` to prevent deadlocks.

Error handling
- Throw specific exceptions; avoid using `Exception` directly.
- Use exception filters and global exception handlers (middleware) for APIs to produce consistent error responses.

Testing
- Use xUnit or NUnit; prefer xUnit for parallel test execution by default.
- Use Fixture patterns and `IClassFixture`/`CollectionFixture` to share expensive resources.
- Mock external dependencies with Moq or NSubstitute. Keep unit tests fast and deterministic.

CI recommendations
- Run `dotnet restore`, `dotnet build --no-restore`, `dotnet test --no-build --verbosity normal` in CI.
- Run `dotnet format --verify-no-changes` to enforce formatting.

Example snippets
- Single-file public type:

```csharp
public class UserService
{
    private readonly IUserRepository _repo;

    public UserService(IUserRepository repo)
    {
        _repo = repo;
    }

    public async Task<User> GetUserAsync(string id)
    {
        return await _repo.FindByIdAsync(id);
    }
}
```

Notes
- When adding new projects, prefer SDK-style csproj and `ImplicitUsings` where appropriate for smaller files.
