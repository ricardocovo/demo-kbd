# API Security

Purpose
- Guidance for designing, implementing, testing, and operating secure APIs exposed by services in this repository.

Scope
- HTTP/REST/gRPC endpoints, RPC interfaces, WebSocket endpoints, and any network-facing interfaces implemented by repository code.

Design principles
- Authentication and authorization are orthogonal to transport. Always enforce auth before performing privileged actions.
- Validate all client inputs at boundary: use schemas (OpenAPI/JSON Schema/protobuf) and reject unknown fields when possible.
- Use explicit allow-lists for parameters and request payload fields.
- Fail closed on schema validation errors.

Transport and encryption
- Require TLS for all network traffic in transit. Reject plaintext connections in production.
- Enforce strong TLS configurations: TLS 1.2+ with modern ciphers, HSTS where applicable, and certificate pinning for internal critical services if needed.

Rate limiting and abuse resistance
- Implement per-API and per-client rate-limiting and quotas; use sliding window or token-bucket algorithms.
- Detect anomalous traffic patterns and throttle or block suspicious clients.

Input validation and sanitization
- Use schema-driven validation for request bodies and query parameters.
- Normalize and canonicalize inputs before validation.

Response handling
- Avoid leaking sensitive information in error messages (stack traces, internal IDs). Provide generic error codes and safe debug logs.
- Set appropriate caching headers and never cache responses containing personal or sensitive data without explicit controls.

Access control patterns
- Use OAuth2 scopes or API keys with least privilege for service-to-service and third-party access.
- For internal service-to-service APIs, prefer mutual TLS or signed JWTs with short lifetimes.

Logging and observability
- Log requests and responses at appropriate levels; redact or omit sensitive fields (PII, tokens, credentials) before logging.
- Correlate logs with request identifiers and propagate trace IDs.

Testing
- Include automated API contract tests (OpenAPI-based) and fuzzing/negative tests to exercise unexpected inputs.
- Run security-focused API tests: auth bypass, IDOR, injection, rate-limit evasion.

CI checks
- Validate OpenAPI schema formats, run contract tests, static analysis for known insecure patterns in request handling.

Example tools and commands
- OpenAPI lint: `spectral lint openapi.yaml`
- API fuzzing: `ffuf`, `boofuzz`, or `grpc-fuzzer` for gRPC
- Contract tests: `dredd`, `schemathesis`

Checklist
- [ ] TLS enforced
- [ ] Schema validation present
- [ ] Rate limiting configured
- [ ] Sensitive fields redacted in logs

