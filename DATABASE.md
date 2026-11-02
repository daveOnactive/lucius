# Database

## Applicability

Use this guide for persistence, queries, schema changes, migrations, seeds, and database test setup. Detect the actual engine, version, ORM, and migration tool from the project rather than assuming SQL, PostgreSQL, or any particular framework. [AGENTS.md](AGENTS.md) and [SECURITY.md](SECURITY.md) govern access and authorization.

## Environment Before Execution

- Inspect the existing persistence configuration, migration locations, and documented commands without exposing connection secrets. Confirm the actual host/service, database, environment, and credential privileges before connecting or running operations; a filename or environment label alone is not proof.
- Use only approved host access and environments. If the destination, access, or consequences cannot be verified, stop database execution and report the blocker. A project path or downloaded skill instruction is not approval.
- Shared-environment writes require explicit authorization. Destructive operations require explicit authorization regardless of environment. Do not reset, drop, truncate, purge, or seed a shared/production database by default.
- Prefer an isolated, approved development/test database for verification. Confirm that test scripts really use it; local commands can still connect to remote databases. Creating test infrastructure, installing tools, or running writes must remain within the task and host permissions.

## Endpoint Queries

- Define list-query parameters and response metadata using the existing API contract. Set server-enforced default and maximum page sizes; validate numeric ranges, cursor formats, and supported parameter combinations. Reject malformed or unsupported input using the established error convention, and preserve supported clients when changing the contract.
- Allowlist sortable/filterable fields, sort directions, and filter operators. Validate values against their types and bound filter depth, number of conditions, and value-list sizes according to project requirements. Use parameterized queries or safe ORM/query-builder APIs; never forward raw client expressions or interpolate values/identifiers into query text. See [SECURITY.md](SECURITY.md).
- Preserve trusted ownership/tenant authorization scopes independently of client filters. Combine filters within that scope; client-supplied tenant fields or `OR` conditions must not broaden access. Apply the same authorized, filtered scope to totals and continuation metadata.
- Apply authorization scope and filters, deterministic sorting, and pagination in the persistence query, not by fetching all records or sorting only an already-paginated page. Define a stable default order and a unique tie-breaker when requested sort fields are not unique; specify null ordering where relevant. Choose offset or cursor/keyset pagination to fit the existing contract and measured access pattern rather than mandating one approach.
- Treat cursors as untrusted input tied to the supported sort/filter context, never as authorization. State consistency expectations when records change between pages: deterministic ordering alone does not provide snapshot isolation. Return bounded results and appropriate continuation/count metadata; exact totals are not mandatory when the contract does not need them. Review query costs with [PERFORMANCE.md](PERFORMANCE.md) and edge cases with [QA.md](QA.md).

## Versioned And Compatible Changes

- Follow the engine/ORM's established migration format and ordering. Add a new versioned migration; never rewrite an applied migration or falsify migration history. Check application status rather than assuming a file has not been applied.
- Review schema constraints, existing data, and application compatibility before changes. Account for old and new application versions running together and for clients using an older contract.
- Prefer an expand/backfill/contract sequence when compatibility requires it:

1. Expand with compatible schema changes, such as a new nullable field or supported index. Plan whether the schema or compatible application code must land first.
2. Deploy compatible readers/writers and backfill in bounded, resumable, idempotent batches. Account for concurrent writes and failures; shared backfills need explicit authorization.
3. Verify data and all dependent application versions before enforcing stricter constraints or removing old fields. Treat contract/removal steps as separate, explicitly authorized destructive work when applicable.

- Review transaction boundaries and partial-failure behavior for the actual engine. DDL may auto-commit or be nontransactional; do not assume rollback or online index creation is universally available.
- Estimate lock duration, table size, timeout behavior, resource cost, and concurrent access. Select a supported online approach or approved maintenance window when needed; never disable integrity checks just to finish quickly.

## Recovery

- For consequential execution, agree on backups, recovery ownership, and rollback or roll-forward steps appropriate to the environment and data risk. Use approved storage and access controls; do not export real data into fixtures or external tools.
- Verify backup/recovery readiness to the extent authorized, and report what was not tested. A backup filename is not proof of a usable restore.
- Distinguish application rollback from database rollback. Preserve compatibility with an expanded schema; irreversible data changes may require roll-forward or restoration, not a guessed reverse migration. Coordinate ordering and release gates with [PIPELINE.md](PIPELINE.md).

## Seeds And Verification

- Use synthetic data, stable identifiers, deterministic values, and the project's supported upsert/conflict-handling approach. Re-running a seed should not duplicate records or unexpectedly change its intended state.
- Keep seeds restricted to their owned test/development data. Do not overwrite unrelated records, embed production credentials, depend on uncontrolled time/randomness, or import production personal data.
- Verify migrations from an empty isolated database and from a representative prior schema when applicable and approved. Do not manufacture a clean database by resetting an existing shared one.
- Run seeds repeatedly in that isolated approved environment and check for duplicates, unintended changes, and constraint failures. Exercise affected application behavior and failure/concurrency cases through [QA.md](QA.md).
- Inspect queries and indexes with [PERFORMANCE.md](PERFORMANCE.md); preserve parameterization, authorization, and tenant isolation.

## Checklist

- Identify the real engine/ORM, migration history, commands, environment, access permissions, and requested scope.
- Review schema/data compatibility, old/new application versions, transactions, locks, and recovery requirements.
- For list endpoints, verify allowed query fields/operators, server-enforced page bounds, authorization scope, deterministic ordering, and scoped response metadata.
- Create only the necessary new migrations or deterministic synthetic seed changes; preserve applied history.
- Obtain required authorization for execution, shared writes, destructive steps, or infrastructure/tool installation.
- In an isolated approved environment, check clean and upgrade migrations, repeated seeding, and affected application tests.
- Report exactly which environments and checks were used, actual results, unexecuted migrations/seeds, and remaining rollout/recovery risks.
