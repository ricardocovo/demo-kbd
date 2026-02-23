# Logging & Metrics Conventions

This document specifies the logging formats, trace ID propagation, metrics naming, Prometheus scraping endpoints, examples for local log aggregation, and guidance on redacting PII.

## Logging formats

- Use structured logs (JSON) for all production services.
  - Top-level fields (recommended): timestamp, level, service, env, trace_id, span_id, message, error, attributes
  - timestamp: RFC3339 (UTC), e.g. 2024-01-02T15:04:05Z
  - level: one of DEBUG, INFO, WARN, ERROR
  - service: short service name (kebab-case)
  - env: deployment environment (production, staging, development)
  - trace_id / span_id: propagated tracing identifiers (hex or UUID as specified below)
  - attributes: object/map containing key-value pairs for contextual data

Example JSON log:

{
  "timestamp": "2024-01-02T15:04:05Z",
  "level": "INFO",
  "service": "user-service",
  "env": "production",
  "trace_id": "4bf92f3577b34da6a3ce929d0e0e4736",
  "span_id": "00f067aa0ba902b7",
  "message": "User lookup succeeded",
  "attributes": {"user_id": "u-1234", "region": "us-west-2"}
}

- For human debugging (local development), pretty-printed logs are acceptable but production must remain structured.
- Avoid embedding multi-line stack traces in a single string field; instead provide an `error` object with `type`, `message`, and optional `stack` (stack may be omitted in production logs depending on privacy/security requirements).

## Trace ID propagation

- Use W3C Trace Context (traceparent) as the primary propagation format where possible.
  - traceparent: `00-<trace-id>-<span-id>-<flags>` (trace-id: 16-byte hex; span-id: 8-byte hex)
- In logs include `trace_id` and `span_id` fields extracted from the active trace context.
- When instrumenting HTTP and RPC frameworks, ensure incoming `traceparent`/`tracestate` headers are respected and forwarded on outgoing requests.
- If the environment uses another tracing system (e.g., X-Cloud-Trace), map its identifiers to `trace_id` in logs and include the vendor-specific header as an attribute (e.g., `x_cloud_trace_context`).

## Metrics naming conventions

Follow Prometheus best practices with a few organization-specific conventions:

- Use snake_case for metric names.
- Prefix metrics with a service or namespace when appropriate: `<service>_<subsystem>_<metric>` or `app_<service>_<metric>`.
- Use metric types appropriately: counters (total counts) end with `_total` for counters that are monotonically increasing; gauges for instantaneous values; histograms and summaries for distributions.
- Examples:
  - user_service_requests_total (counter)
  - user_service_request_duration_seconds (histogram)
  - user_service_in_flight_requests (gauge)

- Labels should be used sparingly and have low cardinality. Common labels: method, route, status_code, region, instance
- Avoid using user identifiers or other high-cardinality values as labels. Put them in logs instead (subject to PII rules below).

## Prometheus scraping endpoints

- Expose a /metrics endpoint on each service instance, defaulting to port 9100 for sidecars or the service’s configured port.
- Example endpoints:
  - http://localhost:9100/metrics
  - http://<service-host>:<port>/metrics

- Recommended settings for Prometheus job config:

  - job_name: 'user-service'
    static_configs:
      - targets: ['user-service-1:9100','user-service-2:9100']

- Secure /metrics endpoints behind service mesh or network policies as needed; do not expose metrics endpoints to the public internet.

## Local log aggregation examples

- To run a simple local aggregation using jq + tail (for JSON logs written to a file):

  # stream logs and pretty-print selected fields
  tail -F /var/log/user-service.log | jq -c '. | {timestamp, level, service, trace_id, message, attributes}'

- Using Promtail + Loki (example with Docker Compose):
  1. Start Loki + Promtail via a local docker-compose.yml (project should include or reference an example compose file).
  2. Configure Promtail to tail your service logs and push to Loki.
  3. Query logs in Grafana connected to Loki.

- Quick docker example (assumes docker-compose.yml is present and configured):

  docker-compose up -d loki promtail grafana
  # view logs in Grafana at http://localhost:3000

- Example: aggregate logs across multiple services locally with journalctl (systemd):

  journalctl -u user-service -f | jq -c '. | {timestamp, level, trace_id, message}'

## Redacting PII from logs

- Principle: Logs should not contain plaintext PII (Personally Identifiable Information). Treat the following as PII unless explicitly allowed and approved: full names, email addresses, national IDs, credit card numbers, social security numbers, birthdates, and raw authentication tokens.

- Strategies:
  - Never log raw authentication tokens, API keys, or passwords.
  - Hash, redact, or tokenize PII before logging. For example, log `user_id_hash` or `user_id_trunc` instead of the raw identifier.
  - Use a central logging helper that applies redaction rules consistently before emitting logs.
  - For structured attributes, store a boolean `contains_pii: true` if the event required capturing PII for debugging; do not include the raw values.
  - For error traces that include request/response bodies, ensure any PII fields are removed or masked (`"email": "[REDACTED]"`).

- Example mask function (pseudocode):

  redacted_email = email.replace(/(.)(.*)(@.*)/, "$1***$3")

- Audit logs periodically for accidental PII leaks and include logging redaction checks in deployment gates for production releases.

## Additional notes

- Keep log sizes reasonable. Avoid logging large payloads by default.
- Correlate logs and metrics using trace_id where possible to make debugging easier.
- Document any deviations from these conventions in this file and get approvals from the platform team.

## Azure-specific Observability Standards

Purpose
- Provide guidance for instrumenting, collecting, and operating logs, metrics, and traces on Microsoft Azure using Azure Monitor, Application Insights, and Log Analytics.

Key services and mapping
- Application Insights: primary SDK-level ingestion for application traces, requests, exceptions, and custom telemetry.
- Log Analytics workspace (Azure Monitor Logs): centralized store for diagnostics logs, activity logs, and custom telemetry queries (Kusto Query Language).
- Azure Monitor Metrics: high-cardinality numeric metrics exposed via the Azure metrics platform.
- Diagnostic Settings: configure resource diagnostics to route platform logs and metrics to Log Analytics, Event Hubs, and/or Storage accounts.

Instrumentation
- Use OpenTelemetry where possible and configure the Azure Monitor OpenTelemetry exporter (or native Application Insights SDK for language-specific features).
- Ensure traces include W3C trace context and map `trace_id` to Application Insights `operation_Id` and `span_id` to `id` for correlation.
- Add application-specific customDimensions for additional context (service, env, region, deployment_id) rather than high-cardinality user data.
- Configure adaptive sampling in Application Insights to control ingestion costs while preserving representative traces; set lower sampling in staging/diagnostic environments.

Resource configuration
- Create a dedicated Log Analytics workspace per environment or per subscription/region according to org policy.
- Enable Diagnostic Settings on platform resources (App Service, Functions, AKS, Storage, SQL) to forward platform logs to the workspace.
- Use Azure Policy to enforce diagnostic settings and to ensure retention and export rules are applied consistently.

Alerts and Action Groups
- Define metric and log alerts in Azure Monitor with clear thresholds and runbooks attached. Use severity levels and distinct action groups per alert type.
- Action Groups should integrate with Teams/Slack, PagerDuty, or other on-call systems and have recovery playbook links.
- Prefer log alerts (KQL) for complex conditions and metric alerts for straightforward numeric thresholds.

Dashboards and Workbooks
- Provide shared Workbooks for common operational views (traffic, errors, latency, resource utilization). Store Workbooks in source control where possible (ARM templates or bicep).
- Create role-specific dashboards: SRE, Product, Security. Keep Workbooks lightweight and focused on actionable signals.

Retention and Cost Management
- Set retention policies on Log Analytics workspace based on compliance (e.g., 90–365 days). Use archive-to-storage for long-term retention to reduce cost.
- Monitor ingestion and query costs; configure daily caps or alerts on ingestion volume to avoid runaway bills.

Security and Access
- Control access via Azure RBAC: grant least privilege to workspaces, diagnostics, and dashboards.
- Restrict who can create or modify action groups and alert rules.
- Enable resource locks or policy guards for production monitoring resources where appropriate.

Kusto queries and saved queries
- Use named saved queries for common investigations and store canonical KQL snippets in `docs/observability/` for reuse.
- Examples:
  - Recent errors:

    ```kql
    exceptions
    | where timestamp > ago(24h)
    | summarize count() by operation_Name, type
    | order by count_ desc
    ```

  - Request duration percentiles:

    ```kql
    requests
    | where timestamp > ago(1h)
    | summarize p50=percentile(duration,50), p95=percentile(duration,95) by operation_Name
    ```

Automation and IaC
- Provision workspaces, diagnostic settings, alert rules, and workbooks via IaC (ARM, Bicep, Terraform) to ensure reproducibility.
- Include deployment of monitoring resources in the same pipeline that deploys the application, or a dependent pipeline that runs immediately after.

Operational guidance
- Use container insights for AKS to gather node, pod, and container metrics; enable container telemetry and integrate with Log Analytics.
- For App Service and Functions, enable Application Insights and ensure `APPLICATIONINSIGHTS_CONNECTION_STRING` is set via Key Vault or secure config.
- For VMs and virtualized resources, use Azure Monitor Agent (AMA) or Log Analytics Agent as appropriate.

CLI examples
- Create a Log Analytics workspace:

  az monitor log-analytics workspace create -g <rg> -n <workspace-name> --location <loc>

- Configure diagnostic settings to send a resource's diagnostics to a workspace:

  az monitor diagnostic-settings create --resource <resource-id> --workspace <workspace-id> --name "send-to-workspace" --logs '[{"category":"AuditEvent","enabled":true}]' --metrics '[{"category":"AllMetrics","enabled":true}]'

- Query logs via az cli:

  az monitor log-analytics query -w <workspace-id> --query "requests | where timestamp > ago(1h) | summarize count() by resultCode"

Validation and testing
- Validate that traces and logs correlate end-to-end by exercising a request and verifying `operation_Id` appears in both Application Insights and Log Analytics entries.
- Include observability checks in CI smoke tests (e.g., verify metrics endpoint responds, validate diagnostic settings were applied post-deploy).

Cost & data hygiene
- Avoid logging verbose request/response bodies to Application Insights by default; use sampling or only send error payloads in sanitized form.
- Periodically review high-cardinality customDimensions and reduce or hash values to control cardinality and storage costs.

References
- Azure Monitor docs: https://docs.microsoft.com/azure/azure-monitor
- Application Insights: https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview

