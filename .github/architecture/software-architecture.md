# Software Architecture Patterns

Purpose
- Describe common software architecture patterns and guidance for choosing the right pattern for services in this repository.

Patterns covered
- Monolith & Modular Monolith
- Microservices
- Event-driven/Message-driven architectures
- CQRS and Event Sourcing
- Layered vs Hexagonal (Ports & Adapters)

Guidance
- Prefer modular monolith for early-stage systems: keep clear package boundaries and extract services when ownership and scaling needs emerge.
- For microservices, define clear service boundaries (bounded contexts), own the data, and prefer asynchronous communication for high-churn interactions.
- Use hexagonal architecture for complex business logic: separate domain logic from external adapters (DB, queue, HTTP) to improve testability.

Microservices specifics
- Service responsibilities: single responsibility per service, own its schema and datastore, publish clear API contracts (OpenAPI/protobuf), and version APIs.
- Service discovery: use DNS-based discovery (K8s service) or a service registry when dynamic discovery is needed.
- Communication: prefer gRPC for low-latency service-to-service calls and HTTP/JSON for public-facing APIs. Use events for eventual consistency.

State and data ownership
- One service = one source of truth for its data. Avoid direct DB access by other services—use APIs or events for data sharing.
- Handle schema evolution carefully; migrate read models and consumers gradually.

Observability & tracing
- Instrument all services with OpenTelemetry compatible libraries; emit traces, metrics, and structured logs with consistent fields.

Security considerations
- Authenticate and authorize service-to-service calls (mTLS/JWT) and validate inputs at service boundary.

When to split
- Split when teams need independent deployability, scalability, or when coupling impedes velocity.

Examples and references
- Link to example service skeletons under `services/` (if present) and to `api/` for OpenAPI/protobuf definitions.
