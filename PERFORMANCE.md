# Performance

## Applicability

Use this guide when a task concerns performance or changes rendering, payload size, I/O, concurrency, caching, or database access. For new projects, apply the relevant parts without inventing optimization work. Follow [AGENTS.md](AGENTS.md) for scope and [SECURITY.md](SECURITY.md) for safe measurement.

## Measurement First

- Identify the user-visible problem and use project-defined budgets and service objectives. Do not invent universal latency, bundle-size, or throughput targets.
- Record a baseline with the project's available profiler, traces, query metrics, or test harness. State environment, representative workload/data, metric, and limitations; compare before and after under equivalent conditions.
- If evidence or tooling is missing, report that limitation and propose a bounded measurement approach. Do not describe a speculative improvement as measured.
- Use approved isolated or non-production environments. Production profiling or load tests require explicit authorization and agreed load, duration, monitoring, and stop conditions. Do not generate production traffic because this guide exists.
- Preserve correctness, accessibility, and security. Do not bypass authorization, validation, consistency, or error handling for a faster benchmark.

## Frontend Costs

- Use available measurements to distinguish rendering, scripting, network, and layout problems. Keep state close to its owner and avoid unnecessary work; add memoization, virtualization, or splitting only when evidence and the framework justify it.
- Inspect the changed route's scripts, styles, fonts, and images. Reuse the framework's asset pipeline, size images appropriately, reserve layout space, and defer noncritical resources without breaking critical journeys or accessibility.
- Review request counts, payload sizes, waterfalls, cancellation, and cache behavior. Bound pagination and avoid duplicate fetches; do not change API contracts or load all records merely to simplify the client.
- Check representative devices and viewport sizes using available approved tooling. Report unavailable browser/device checks instead of claiming cross-device results.

## Backend Costs

- Use latency distributions, throughput, error rate, I/O, and memory measurements appropriate to the service. Examine query/service spans without logging sensitive payloads.
- Bound concurrency, queues, payloads, and buffers. Use existing connection pools, timeouts, cancellation, and backpressure patterns; retries must be bounded and respect idempotency.
- Reduce repeated I/O or unnecessary allocations where measured. Consider streaming or batching when compatible with the existing interface, not as an automatic redesign.
- Define cache keys, ownership/tenant scope, expiry, invalidation, and failure behavior before adding caching. Prevent cross-user data leaks and stale authorization decisions; document consistency tradeoffs.
- Review limiter counter latency, expiry, key cardinality, and backend-outage behavior. Verify the intended quota scope across workers/replicas using existing approved facilities; do not assume local counters enforce global limits. Follow [SECURITY.md](SECURITY.md) for identity and denial policy.
- Rate limits do not cap simultaneous work. Preserve bounded concurrency, queues, timeouts, and query/payload budgets even when requests meet their quota; do not turn limiter verification into an unapproved production load test.

## Database Costs

- Inspect query counts, execution plans, and cardinality with the actual engine/ORM. Use an approved environment and representative synthetic or otherwise permitted data; follow [DATABASE.md](DATABASE.md).
- Check N+1 access patterns and use supported batching/eager loading where needed. Select required fields and use bounded pagination with stable ordering; follow [DATABASE.md](DATABASE.md) for endpoint query contracts and consider keyset pagination only where the access pattern warrants it.
- Measure supported filter/sort combinations, deep offsets, and optional total-count queries against representative cardinalities. Review how authorization scope and ordering affect query plans and index use; do not add unbounded filters or exact-count work merely to simplify the client.
- Justify indexes against measured queries, selectivity, write overhead, storage, and migration/locking cost. Do not create speculative indexes or execute schema changes without the applicable approval.
- Understand profiling commands before running them. Execution variants such as `EXPLAIN ANALYZE` can run the query and take locks; they are not automatically safe read-only inspection, especially with writes.

## Checklist

- Identify the affected user journey or service path and its existing budget or objective.
- Establish an approved, reproducible baseline and inspect the relevant rendering, network, I/O, or query evidence.
- For affected endpoints, review limiter state costs and scoped filter/sort/pagination query budgets without weakening security or changing contracts gratuitously.
- Choose the smallest supported improvement and record correctness, caching, or concurrency tradeoffs.
- Run relevant regression checks from [QA.md](QA.md), including security and consistency cases where affected.
- Compare the same metrics/workload after the change; distinguish measured results from hypotheses.
- Report measurement conditions, regressions, unavailable checks, and any unapproved production testing that was not performed.
