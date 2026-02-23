# Performance Testing Guidance

This document provides guidance for running performance benchmarks for this project.

## Tools

- wrk: A simple and fast HTTP benchmarking tool written in C. Good for load testing and latency measurements. (https://github.com/wg/wrk)
- k6: A modern load-testing tool using JavaScript for scripting, rich metrics, thresholds, and CI-friendly output. (https://k6.io/)

## Example Scripts

### wrk example

Run a basic wrk test against an endpoint:

wrk -t12 -c400 -d30s http://localhost:8080/endpoint

Options:
- -t: threads
- -c: concurrent connections
- -d: duration

Capture output to a file:

wrk -t12 -c400 -d30s http://localhost:8080/endpoint > perf/wrk-$(date +%s).log

### k6 example (script)

Save as tests/perf/loadtest.js:

export let options = {
  vus: 100,
  duration: '30s',
  thresholds: {
    'http_req_duration': ['p(95) < 500'] // 95th percentile must be < 500ms
  }
};

import http from 'k6/http';

export default function () {
  http.get('http://localhost:8080/endpoint');
}

Run locally:

k6 run --summary-export=perf/k6-summary-$(date +%s).json tests/perf/loadtest.js

This will produce a JSON summary that can be used in CI or for further processing.

## How to run benchmarks

1. Start a representative environment (local server, docker-compose, or a staging environment with production-like data and configuration).
2. Warm up the system under test before capturing results (e.g., run a short low-load test to bring caches and JITs up).
3. Run multiple iterations and vary loads to understand behavior under different stress levels.
4. Capture raw outputs and structured summaries (e.g., k6 JSON summaries, wrk logs).

Example local flow:

- Start service: docker-compose up -d
- Warmup: k6 run --vus 10 --duration 15s tests/perf/loadtest.js
- Run baseline: k6 run --summary-export=perf/k6-baseline.json tests/perf/loadtest.js
- Run experiment: k6 run --summary-export=perf/k6-experiment.json tests/perf/loadtest.js

## Setting Performance Budgets

A performance budget defines acceptable limits for key metrics. Common metrics:

- p50, p90, p95, p99 latency (http_req_duration)
- requests/sec (throughput)
- error rate (http_req_failed)
- resource usage (CPU, memory)

Examples of budgets:
- p95 latency < 500ms
- error rate < 0.1%
- throughput >= 1000 req/s for the service under test

With k6, embed thresholds in the script (see example). k6 will return a non-zero exit code if thresholds fail, which makes it convenient for CI gating.

For wrk, parse its textual output in a small script to check for thresholds, or wrap wrk in a harness that evaluates results and exits non-zero on failure.

## Where to store results

- Store raw logs and summaries in a dedicated directory in the repository or in an external artifact store.
  - Suggested path inside CI artifacts: perf/artifacts/
  - Suggested path in repo for stable baseline snapshots: perf/baselines/

- In CI, upload perf artifacts (JSON summaries, raw logs) so they are retained alongside the run.
- Keep a short README or index (perf/README.md) pointing to the most recent baselines and how to interpret them.

## Sample GitHub Actions workflow (CI)

.github/workflows/perf.yml

```yaml
name: Performance Tests
on:
  workflow_dispatch:
  push:
    branches: [ main ]

jobs:
  perf-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Start services
        run: |
          # Start any required test environment; replace with your startup commands
          docker-compose up -d --build
      - name: Install k6
        run: |
          sudo apt-get update && sudo apt-get install -y gnupg2
          curl -s https://dl.k6.io/key.gpg | sudo apt-key add -
          echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update && sudo apt-get install -y k6
      - name: Run k6 load test
        run: |
          mkdir -p perf/artifacts
          k6 run --summary-export=perf/artifacts/k6-summary.json tests/perf/loadtest.js || true
      - name: Evaluate thresholds and fail if needed
        run: |
          # k6 exits non-zero if thresholds fail, but if you used || true above to capture output, evaluate here.
          if jq -e '.metrics.http_req_duration.thresholds_failed > 0' perf/artifacts/k6-summary.json >/dev/null 2>&1; then
            echo "Performance thresholds violated"
            cat perf/artifacts/k6-summary.json
            exit 1
          fi
      - name: Upload perf artifacts
        uses: actions/upload-artifact@v4
        with:
          name: perf-artifacts
          path: perf/artifacts/**
```

Notes:
- The example uses k6 thresholds. k6 will exit non-zero when thresholds fail, but capturing summaries and explicitly checking them gives more flexibility.
- For wrk, store logs and add a small parser step to evaluate p95 and throughput against budgets.

## Threshold gating

- Define thresholds near expected production behavior but with some buffer to account for variability.
- Use multiple levels: hard failure thresholds (CI must fail) and warning thresholds (post results but do not fail CI).
- When a threshold is exceeded in CI, fail the job and upload artifacts for analysis.
- Consider running perf tests on dedicated hardware or stable runners to reduce noise.

## Tips and best practices

- Run perf tests outside of peak load windows on shared infrastructure.
- Run multiple iterations and report median or the best stable run.
- Track baselines in source control or an artifact store and compare new runs against the baseline.
- Automate result ingestion (e.g., generate graphs or dashboards) for long-term tracking.

---

Place this file under .github/performance/benchmarks.md. Update thresholds and scripts to match your service endpoints and realistic load profiles.
