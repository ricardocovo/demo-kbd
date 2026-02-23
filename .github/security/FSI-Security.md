# Financial Services Industry (FSI) Security

Purpose
- Provide domain-specific security controls and compliance considerations for code and systems that operate in financial or similarly regulated contexts.

Scope
- Systems processing payments, personal financial data, customer account management, ledgers, reconciliation, settlement, and reporting.

Regulatory posture and compliance
- Identify applicable regulations early (PCI-DSS, SOC 2, GLBA, PSD2, MiFID II depending on jurisdiction and product scope).
- Maintain documentation and evidence for controls (change logs, access reviews, audit trails) to support compliance audits.

Data protection
- Classify data by sensitivity (PII, financial data, transaction records) and apply stronger controls for high-sensitivity data.
- Encryption: enforce encryption at rest (disk-level and/or field-level) and in transit (TLS). Use strong, managed encryption keys.
- Tokenization: use tokenization for PANs and other highly sensitive identifiers instead of raw storage where feasible.

Transaction integrity
- Ensure idempotency for financial APIs and maintain transactional atomicity where required.
- Use cryptographic signatures for non-repudiation on critical financial messages or batch files.

Operational controls
- Segregation of duties: separate development, test, and production environments with strict access controls.
- Change management: require approvals and rollbacks for production changes; log deployments and maintain immutable artifact provenance.

Monitoring, detection, and alerting
- Monitor for anomalous transactions, sudden volume spikes, or unusual settlement patterns.
- Maintain fraud detection hooks and integrate with external fraud providers when needed.

Third-party and vendor risk
- Conduct security assessments for vendors handling financial data; require SOC2-Type II or equivalent evidence where applicable.
- Contractually require breach disclosure timelines and secure handling of data.

Incident response and reporting
- Define escalation and regulatory notification timelines (e.g., PCI requires specific timelines). Prepare templates and contacts for regulators and affected customers.

Testing and assurance
- Run penetration testing and red-team exercises focusing on financial flows and reconciliation paths.
- Include test environments with realistic but sanitized data; never use production PII in tests.

Checklist
- [ ] Applicable regulations identified and documented
- [ ] Data classification and encryption in place
- [ ] Transaction integrity and idempotency enforced
- [ ] Vendor security assessments completed

