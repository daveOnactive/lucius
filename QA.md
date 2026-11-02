# Quality Assurance

## Applicability

Use this guide for code changes, fixes, and the applicable verification planning for new projects. Match tests to risk and observable behavior. Reuse existing test tools and coverage policy rather than introducing a framework or arbitrary percentage target. [AGENTS.md](AGENTS.md) governs project selection and permitted execution.

## Test Selection

- Define acceptance behavior and affected failure modes before changing code. For a bug fix, add a focused regression test and establish that it fails without the fix where feasible; do not claim a reproduction that was not observed.
- Use unit tests for focused logic and boundary cases, integration tests for real component/service interactions, contract tests for API/schema compatibility, and E2E tests for critical user journeys. Choose the layers that can detect the actual risk, not every layer automatically.
- Assert public outcomes, meaningful state changes, and documented errors rather than private implementation details. Keep snapshots narrow and useful; avoid tests that fail merely because internal organization changed.
- Keep fixtures deterministic and synthetic. Control time and randomness using supported tools, reset only owned isolated state, and prefer condition-based bounded waits over fixed sleeps or fragile timing assertions.
- Preserve the project's coverage requirements. Add tests for changed behavior and high-risk gaps; do not chase a universal coverage number or remove failing tests to obtain a green result.

## Frontend Checks

- Cover changed critical journeys, including loading, empty, error, and success states. Verify that failures offer appropriate recovery and that duplicate actions are handled when relevant.
- Check semantic controls, accessible names, keyboard operation, focus behavior, and the project's accessibility expectations using available automated/manual tooling. Automated checks alone do not prove full accessibility.
- Verify responsiveness at the project's supported viewport/device sizes, including content overflow, touch interaction, and changed asset/layout behavior. Preserve the established visual system; see [STRUCTURE.md](STRUCTURE.md).
- Use existing browser tooling where available. An optional browser-testing skill is not a browser, runtime, or test result; missing tooling must be reported and installation requires approval.

## Backend And Database Checks

- Test authentication and authorization separately, including denied access, role/resource ownership, and cross-tenant cases. Verify that errors/logs do not expose secrets or sensitive data; see [SECURITY.md](SECURITY.md).
- Cover input boundaries, invalid payloads, expected errors, dependency failures, timeouts/cancellation, and affected concurrency or idempotency behavior. Use deterministic coordination rather than hoping a race happens.
- Exercise changed API contracts and compatibility with affected consumers, including response/error shapes and older supported versions where relevant.
- Use an isolated approved database and service configuration for integration tests. Confirm the actual connection and script effects before execution; no shared resets, destructive cleanup, or shared writes by default. Follow [DATABASE.md](DATABASE.md) for migration and repeated-seed checks.
- Validate correctness alongside any measured performance change; benchmark success does not replace regression tests. See [PERFORMANCE.md](PERFORMANCE.md).

### Endpoint Policies

- Cover allowed/disallowed origins, methods and headers, credentialed requests, valid/invalid preflights, and CORS headers on success/error/`429` responses. Check browser behavior with available approved tooling; CORS rejection is not a substitute for server authorization.
- Use controlled time and isolated approved limiter state to test quota exhaustion, `429`/retry metadata, window expiry/recovery, caller/tenant isolation, spoofed proxy headers, and the documented counter-backend outage policy. Exercise concurrent requests and multi-worker/replica enforcement when applicable without unapproved production traffic. Follow [SECURITY.md](SECURITY.md).
- Verify pagination defaults and maximum sizes, invalid ranges/cursors, empty and final pages, tied sort values and tie-breakers, and supported sort/filter/pagination combinations. Test changes between pages against the declared consistency contract rather than assuming snapshot stability.
- Cover allowlisted and unsupported sort fields, directions, filter operators, value types, and complexity bounds. Treat injection-like values safely as data; attempts to override ownership/tenant scope, including combined/`OR` filters, must not broaden access. Assert globally ordered page results and authorization-scoped totals/continuation metadata; follow [DATABASE.md](DATABASE.md).

## Execution And Reporting

- Inspect test commands and required services before running them. Start with focused checks, then the broader permitted checks justified by risk and project policy. Do not invoke installers or downloading `npx` commands to fill a tooling gap without explicit approval.
- Diagnose failures without discarding unrelated work or weakening assertions. Distinguish observed pre-existing failures from regressions only when evidence supports that distinction.
- Report exact commands and working directories, actual outcomes, and checks skipped or blocked with reasons. Distinguish automated tests, manual inspection, and documentation review; never call an unperformed check passing.
- For CI changes, check that equivalent existing commands are represented appropriately in [PIPELINE.md](PIPELINE.md)'s delivery model. Do not create or trigger CI merely to satisfy this guide.

## Checklist

- Map requested behavior and regression risks to the smallest effective unit/integration/contract/E2E checks.
- Place tests according to project conventions and use deterministic fixtures with isolated dependencies.
- Cover relevant accessibility/responsiveness, authorization/validation, failure, concurrency, and database cases.
- For affected endpoints, verify CORS and rate-limit behavior plus filter/sort/pagination edge cases, error contracts, and authorization-scoped metadata.
- Run focused checks followed by the appropriate broader permitted checks; preserve evidence and report missing prerequisites.
- Review failures and the diff for weakened assertions, brittle timing, data exposure, or unrelated changes.
- Summarize actual validation, unavailable checks, remaining risks, and any authorized follow-up that was not executed.
