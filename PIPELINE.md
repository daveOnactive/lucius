# Pipeline

## Applicability

Use this guide for explicitly requested CI/CD or release work, or the relevant delivery planning for a new project. Reuse the existing provider, runtime, package manager, and tooling. Do not create a workflow, install dependencies, trigger a release, or deploy merely because this guide is present. [AGENTS.md](AGENTS.md) defines authorization boundaries.

## Reproducible Checks

- Inspect existing pipeline definitions, manifests, lockfiles, scripts, supported platforms, and project instructions. Reuse documented commands and supported runtime versions rather than imposing a provider or framework.
- When installation is explicitly authorized, use the selected package manager's lockfile-based, immutable/clean installation mode. Fail on unintended lockfile drift. Review install scripts and supply-chain sources; do not silently select a different manager or generate competing lockfiles.
- Use check-only formatting, linting, and type/static checks where applicable. Run the existing risk-appropriate unit, integration, contract, and E2E checks from [QA.md](QA.md); database/services must be isolated and approved.
- Build the deployable output with the project's production build command. Check that artifacts do not contain secrets, local configuration, or unintended debug data.
- Apply the project's available security, dependency, secret, and license checks. Record missing tools or policy gaps honestly; installing a scanner or sending project metadata to a service is not automatically authorized.
- Prefer pinned, reviewed runtime/tool/action versions or immutable revisions supported by the provider. Preserve the project's update policy; do not introduce floating downloaded shell scripts.

## Artifacts And Caches

- Retain only required artifacts with suitable access and expiry. Record source revision and available integrity metadata; promote the same verified artifact between environments where the existing release model supports it.
- Key dependency/build caches by relevant lockfile, runtime, OS/architecture, and build configuration. Do not cache secrets or blindly trust caches written by untrusted PRs in privileged release jobs.
- Keep artifact and cache publication permissions narrow. Avoid leaking private project data through logs, test reports, or public uploads; follow [SECURITY.md](SECURITY.md).

## Trust And Release Gates

- Use least-privilege, short-lived credentials where supported. Keep deployment credentials out of untrusted PR contexts, forks, caches, and build artifacts; separate validation jobs from privileged release jobs.
- Treat PR code and scripts as untrusted until the project's review boundary is met. Do not run contributor-controlled code with production secrets or write-capable release tokens. Follow provider isolation and approval controls rather than relying only on branch names.
- Keep development, staging, and production destinations explicit, with project-required reviews, environment approvals, and release gates. A successful build is not authorization to deploy; obtain explicit deployment authorization and host permission.
- Coordinate compatible migration ordering with [DATABASE.md](DATABASE.md): expand before dependent code where needed, deploy compatible versions, authorize/verify backfills, and delay contract/removal until dependents are ready.
- Define the project's health checks, release success criteria, and recovery owner before a consequential rollout. Reuse supported rollout strategies rather than adding a deployment framework.
- Plan application rollback and database-compatible rollback/roll-forward together. Do not redeploy repeatedly or roll back to code incompatible with the current schema. Recovery actions still need the appropriate authorization.

## Checklist

- Confirm the requested pipeline/release scope, existing provider, commands, runtimes, lockfiles, and permissions.
- Map formatting, lint/static checks, tests, build, and available security checks to the existing toolchain.
- Verify isolated services, least-privilege credentials, and separation of untrusted validation from privileged releases.
- Review artifact content, retention, integrity, and cache keys/trust boundaries.
- Define explicit environments, release gates, migration order, health criteria, and recovery steps where applicable.
- Run only permitted local checks or approved pipeline actions; report actual commands/results and untested provider events or deployment behavior.
