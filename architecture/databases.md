# Databases & Storage Patterns

Purpose
- Document recommended database patterns, backups, migrations, and storage choices.

Database choices
- OLTP: use relational databases for transactional systems (Postgres, Azure SQL). Use connection pooling for efficiency.
- OLAP: use data warehouses for analytics workloads (Snowflake, BigQuery, Synapse).
- NoSQL: use document or key/value stores (CosmosDB, DynamoDB) for flexible schema or high-scale access patterns.

Migrations
- Keep migrations in source control and apply via CI/CD automation (e.g., Flyway, Liquibase, Alembic, EF Migrations).
- Use a migration staging strategy: apply to staging, run smoke tests, then apply to production during safe windows.

Backups & DR
- Automate backups and test restores periodically. Retain backups according to compliance requirements.
- Consider point-in-time recovery and geo-redundant storage for critical datasets.

Storage
- Use object storage (Azure Blob / S3) for large unstructured data and blobs.
- For large files, prefer signed URLs and CDN for public serving.

Security
- Encrypt data at rest and in transit; use managed keys or KMS for key management.

Scaling
- Use read replicas, partitioning (sharding), or caching layers for scale-out-read heavy workloads.

Operational
- Monitor DB metrics (connections, locks, slow queries) and set alerts for threshold breaches.
