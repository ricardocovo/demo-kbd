# Repository Architecture Overview

This document gives a high-level view of the repository's architecture and guidance on layout and placement of new code.

## Expected runtime shapes

- Monolith
  - Single deployable/service binary or container that contains most application features.
  - Simpler operational model (one runtime, one deployment unit).
  - Suitable for small teams, projects in early stages, or tightly-coupled features that benefit from a single process.

- Modular Monolith
  - Single deployable that is internally modular (clear package boundaries, well-defined interfaces).
  - Encourages separation of concerns while retaining simple deployments.

- Multiple Services (microservices)
  - Multiple independent services each owning a bounded context and data.
  - Services communicate over RPC/HTTP/event buses.
  - Suitable when teams or scale demand independent deployability and ownership.

Guidance: Start as a modular monolith and split into services when operational, scaling, or organizational needs justify it.

## Suggested directory layouts

Option A — Simple / Monorepo for small projects (recommended to start):

- src/                # Primary application source, frameworks, and routes
  - cmd/              # App entry points (e.g., cmd/web, cmd/worker)
  - internal/         # Non-exported packages (language-specific use)
  - pkg/              # Exportable libraries intended for reuse
  - configs/          # Configuration files and templates
  - migrations/       # DB migrations
  - scripts/          # Dev/build scripts

Option B — Go-style / Multi-binary layout:

- cmd/<service>/      # each service or binary entry point
- pkg/                # public libraries shared across services
- internal/           # packages private to this repository
- api/                # API definitions (OpenAPI, protobuf)
- deployments/        # k8s/terraform/compose files

Option C — Multi-service repository (microservices):

- services/
  - service-a/
    - cmd/
    - internal/ or pkg/
    - Dockerfile
    - README.md
  - service-b/
- libs/               # shared libraries used by multiple services
- infra/              # deployment and infra-as-code

Choose the layout that best matches team size, coupling, and deployment model. Keep shared code small and stable in libs/ or pkg/ to minimize coupling.

## Data flow patterns

- Synchronous request/response
  - HTTP/GRPC between clients and services. Use for user-facing APIs and low-latency interactions.

- Asynchronous/event-driven
  - Message queues or event bus (Kafka, RabbitMQ) for decoupling, scaling background work, and eventual consistency.

- Batch/data pipeline
  - Periodic jobs and ETL pipelines that read from sources, transform, and write results.

- CQRS (Command Query Responsibility Segregation)
  - Separate write model (commands) from read model (queries) when read/write concerns diverge.

When designing data flow, prefer clear ownership of data and single-writer patterns. Document events and schemas in api/ or a dedicated events/ directory.

## Guidance: where to place new services or libraries

- New service (small/experimental):
  - services/<service-name>/ (microservices layout)
  - or cmd/<service-name>/ if following a monorepo/multi-binary layout and service will live inside the same process boundary.

- New library intended for reuse across services:
  - libs/ or pkg/
  - Keep APIs stable; add clear tests and small surface area.

- Internal utility or helper used only by one service:
  - internal/ under that service or repository root internal/ for language-specific private packages.

- API definitions and contracts:
  - api/ (OpenAPI, protobuf) or services/<service>/api/

- Deployment, infra, and config:
  - infra/ or deployments/ for k8s/terraform/docker-compose and CI workflows.

## Decision table: choosing a project layout

| Team size / Goal | Recommended layout | Rationale |
|------------------|--------------------|-----------|
| Solo / small team, early-stage | Monolith (src/, cmd/) | Fast iteration, simple CI/CD, low operational overhead |
| Growing team, multiple domains | Modular monolith (cmd/, pkg/, internal/) | Encourages modularity while keeping deployments simple |
| Multiple teams, independent lifecycle | Multi-service (services/, libs/, infra/) | Independent deployability, ownership, scalability |
| Shared libraries across services | Use pkg/ or libs/ with versioning | Minimizes duplication; enforce stable APIs |

(Prefer starting simple and refactoring layout as needs evolve.)

## Conventions & best practices

- Aim for clear ownership: each service or package should have a README that explains purpose, runtime, and interfaces.
- Keep shared libraries small and well-documented; prefer composition over large shared monoliths.
- Document event schemas and API contracts alongside code (api/ or docs/).
- Use cmd/ for entrypoints and keep business logic in pkg/ or internal/ to allow reuse and testing.

## Where to add this file

This file lives at .github/architecture/overview.md and should be referenced from CONTRIBUTING.md or README.md if available.

Related architecture documents
- Software patterns: `.github/architecture/software-architecture.md`
- Cloud guidance: `.github/architecture/cloud-architecture.md`
- Microservices checklist: `.github/architecture/microservices.md`
- WebSockets guidance: `.github/architecture/websockets.md`
- Databases & Storage: `.github/architecture/databases.md`
- Caching & CDN: `.github/architecture/caching.md`
- Mainframe integration patterns: `.github/architecture/mainframe-integration.md`

-- End of architecture overview
