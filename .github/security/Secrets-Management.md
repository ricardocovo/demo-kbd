# Secrets Management

Purpose
- Best practices for handling credentials, API keys, tokens, and other sensitive configuration items.

Storage and access
- Use a managed secrets store (AWS Secrets Manager, HashiCorp Vault, Azure Key Vault, or GitHub Secrets for CI) rather than code or plaintext files.
- Control access via IAM policies and role-based access; enable audit logging on secrets access.

Rotation and lifecycle
- Rotate secrets regularly and on key events (compromise, role change, departure).
- Automate rotation where possible and use short-lived credentials for service accounts.

CI and local developer experience
- Do not hardcode secrets in workflows; use repository or organization secrets and environments.
- Provide developer guidance to use local secret injection (direnv, .env via local vault agent), and never commit .env files.

Detection and prevention
- Add secret-scanning pre-commit hooks and CI checks; monitor and auto-revoke exposed secrets when detected.

Recovery
- Have automated scripts or documented steps to rotate/revoke compromised secrets quickly; publish a runbook with required approvals and steps.

Checklist
- [ ] Secrets stored in managed secret store
- [ ] Access audited and least-privilege applied
- [ ] Automated rotation where possible

