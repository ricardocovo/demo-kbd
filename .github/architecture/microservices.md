# Microservices Patterns

Purpose
- Guidance and checklists for implementing microservices in this repository.

Service lifecycle
- Create a small, focused API and minimal datastore for each service.
- Define CI/CD pipelines per service and keep deployable artifacts isolated.

Data and storage
- Each service owns its database schema; use async events to propagate changes to interested parties.
- Backup and restore plans must exist for each service's critical data.

Service communication
- Synchronous APIs: use gRPC or HTTP/JSON with versioned APIs.
- Asynchronous messaging: use topics/queues for decoupled events; include schema registry for event contracts.

Operational considerations
- Health checks, readiness/liveness, and graceful shutdown implementation required for each service.
- Implement resource limits and requests in container orchestration manifests.

Security
- Enforce per-service RBAC and encrypt data in transit and at rest.
