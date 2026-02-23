# Cloud Architecture Guidance

Purpose
- Provide cloud-specific recommendations for deploying services, choosing managed offerings, and organizing infrastructure in cloud providers.

Principles
- Infrastructure as code: define resources via Terraform, Bicep, or ARM and store in `infra/`.
- Immutable infrastructure: build immutable artifacts and deploy artifacts rather than modifying running servers.
- Environment parity: keep staging environments as close to production as feasible.

Cloud service choices (examples)
- Compute: prefer managed containers (AKS/EKS/GKE) or serverless (Functions/Lambda) where operational overhead should be minimized.
- Databases: prefer managed DBs (Azure SQL, RDS, Cloud Spanner) for production; use read replicas and backups.
- Storage: use object storage (Azure Blob, S3) for static assets and backups.
- Messaging: managed message brokers (Azure Service Bus, Kafka as a managed service) for event-driven work.

Network & security
- Use private networks and service endpoints; avoid exposing internal services publicly.
- Use identity-based access (managed identities, IAM roles) for services to access platform resources.
- Apply network policies (NSG, Security Groups) and service meshes for fine-grained traffic control.

Multi-region & DR
- Define failover strategies and data replication policies for multi-region deployments if required by RTO/RPO.

Cost & tagging
- Tag resources by team, environment, and cost-center; use cloud provider budget and alerting features.

Operational tooling
- Use provider telemetry (Azure Monitor, CloudWatch) alongside application-level telemetry.
- Automate security scans (IaC scanning, container scanning) in CI/CD.

Repository structure recommendations
- infra/terraform or infra/bicep for IaC
- deployments/ for K8s manifests, helm charts, or compose files
- scripts/ to automate environment setup and bootstrap

Examples and templates
- Reference templates for AKS deployments, Helm charts, and Azure managed resource configurations when available.
