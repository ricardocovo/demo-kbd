# Incident Response & Monitoring

Purpose
- Procedures to detect, investigate, and respond to security incidents and to maintain situational awareness.

Detection
- Instrument critical apps and services with structured logs, metrics, and traces.
- Define and maintain alerting rules for security-sensitive signals (auth failures, privilege use, large data exports).

Response playbooks
- Maintain playbooks for common incident types (data exfiltration, compromised credentials, application RCE).
- Each playbook should list: detection signals, initial containment steps, evidence collection, communication, and eradication steps.

Forensics and evidence
- Ensure logs are immutable and preserved for a defined retention period for forensic needs.
- Capture volatile evidence when needed (memory, running processes) using approved tooling and procedures.

Communication and escalation
- Establish incident roles (incident commander, communications lead, forensic lead) and contact lists for internal and external stakeholders.
- Pre-draft communication templates for customers and regulators to speed notifications.

Post-incident
- Conduct a post-incident review to identify root causes, corrective actions, and preventive measures.
- Track remediation tasks back to code or process changes and validate fixes via tests and audits.

Exercises
- Run tabletop and live-fire exercises to validate playbooks and team readiness.

Checklist
- [ ] Playbooks authored and accessible
- [ ] Alerting enabled for key signals
- [ ] Log retention policies documented

