# Authentication and Authorization

Purpose
- Prescribe secure authentication and authorization mechanisms for human users and services.

Scope
- All systems that authenticate users or authorize actions, including web apps, APIs, background jobs, and admin tooling.

Authentication
- Prefer delegated authentication (OIDC/OAuth2) with a trusted identity provider where possible.
- Passwords: if storing credentials, use a slow adaptive hash (bcrypt/argon2id/scrypt) with well-chosen parameters and rotate as needed.
- MFA: require multi-factor authentication for all interactive user accounts with elevated privileges and for administrative access.
- Token lifetimes: issue short-lived access tokens and use refresh tokens with bound client checks and revocation paths.
- Session management: set secure, HttpOnly cookies, use SameSite attributes, and bind sessions to user agents or IPs if needed.

Service-to-service
- Use mutual TLS, short-lived mTLS certificates, or signed JWTs with audience and scope claims for service authentication.
- Do not rely on static long-lived secrets for service identity unless rotated and scoped.

Authorization
- Always perform authorization checks on the server side; never rely on client-side enforcement.
- Use RBAC or ABAC depending on complexity: prefer RBAC for simplicity and ABAC for dynamic attribute-based policies.
- Principle of least privilege: assign the smallest role or permission set necessary.
- Implement resource-level checks (e.g., owner-based access control) and check both action and resource scope.

Token handling
- Store tokens securely (HttpOnly cookies or secure token stores). Avoid logging tokens or including them in URLs.
- Validate token signatures, expiration, issuer, audience, and scopes on every request.

Auditing
- Record authentication events, privilege escalations, and critical authorization denials to an tamper-evident log.

Common pitfalls and mitigations
- Do not roll custom auth protocols; use proven frameworks.
- Avoid storing tokens in localStorage for web apps; prefer secure cookies.

Checklist
- [ ] MFA enabled for admins
- [ ] Token validation implemented and tested
- [ ] Short token lifetimes with refresh/revocation
- [ ] Service identities use mTLS or signed short-lived credentials

