# Structure

## Applicability

Use this guide for code changes, project scaffolding, and architectural work. Follow the target project's framework, language, package boundaries, and existing layout first. These are complementary policies, not instructions to create layers or switch agents. Project selection and approval boundaries are in [AGENTS.md](AGENTS.md).

## Rules

- Make the smallest structure change needed for the task. Do not reorganize working code, generate empty folders, or introduce a monorepo merely to match an example.
- Keep feature-specific UI, state, API calls, business rules, and tests near their owners where the framework permits. Extract shared code only when there is a real common responsibility, not a speculative future use.
- Keep dependency direction clear. Avoid circular imports and feature-to-feature access through private internals; use established public interfaces or narrowly shared contracts.
- Follow framework rules for routing, static assets, generated files, migrations, and test discovery. Use the project's naming and file-extension conventions.
- Keep configuration separate from secrets. Client-visible configuration and shared contracts must not expose server credentials, persistence internals, or privileged operations; see [SECURITY.md](SECURITY.md).

Use the following preferred layouts where they fit the project. Bracketed names stand for real feature/module names, and `...` denotes other necessary files/folders rather than a literal directory. Keep only the parts needed; framework-required paths and existing layouts still take precedence.

## Server Example

This feature-first layout applies to any backend language. Folder names describe responsibilities, not requirements for classes, inheritance, dependency-injection modules, or additional layers.

```text
server/
|-- app/
|   |-- [feature_name]/
|   |   |-- controller/
|   |   |-- services/
|   |   |-- dto/
|   |   \-- middleware/
|   \-- common/
|       |-- services/
|       |-- controller/
|       \-- modules/
\-- ...                         # Other necessary folders and files
```

Controllers handle transport and response mapping; services own business rules and authorization; DTOs describe request/response data contracts; feature middleware handles feature-specific request behavior. DTOs and static types do not replace runtime validation. These responsibilities may remain in fewer files in a small project; do not create unused folders merely to complete the tree.

Put genuinely shared server services, controllers, and modules in `app/common/`; keep feature-specific code with its feature instead of turning `common/` into a miscellaneous dumping ground. Configuration, persistence/ORM files, migrations, tests, and entry/build files remain in the locations required by the project. Validate configuration at the established startup boundary and do not force repository abstractions around a working ORM. Versioned migrations and isolated database tests follow [DATABASE.md](DATABASE.md).

### Endpoint Ownership

These are responsibilities within the existing architecture, not mandatory new directories or middleware libraries.

- The HTTP boundary/gateway or appropriate shared middleware owns CORS and applicable rate limiting; feature middleware handles feature-specific rules. Choose ordering deliberately: coarse IP/request limits can precede authentication, while identity quotas require caller identification. Keep actual request permissions intact during preflight handling; follow [SECURITY.md](SECURITY.md).
- Controllers parse and validate pagination, sort, and filter parameters against DTO/contracts using existing runtime validation. Existing configuration owns origin allowlists, quotas, and server-enforced page bounds rather than scattering them across handlers.
- Services enforce resource ownership, tenant scope, and domain-specific query permissions. Client filters must not replace or override that trusted scope.
- Existing persistence/query/ORM code applies the authorized scope and validated filters, then stable sorting and pagination to the query itself. Avoid fetching all records or sorting just one fetched page; do not move query execution into DTOs or client containers. Detailed query and response rules are in [DATABASE.md](DATABASE.md).

## Client Example

Apply this layout separately to web and React Native clients. The `.tsx` filename illustrates a React/TypeScript feature container; preserve the target project's language, extensions, and component conventions rather than requiring a migration.

```text
client/
\-- src/
    |-- app/                    # Screens/pages, layouts, routing/navigation
    |-- modules/
    |   \-- [module_name]/
    |       |-- components/
    |       |-- service/
    |       \-- features/
    |           \-- feature_name.feature.tsx
    |-- components/
    |   |-- ui/
    |   \-- common/
    |-- libs/
    \-- ...                     # Other necessary source folders and files
```

- `src/app/` owns all screens/pages, layout setup, routing/navigation setup, and header/tab-navigation composition. Screens/pages render the appropriate module feature container rather than duplicating the module's UI assembly.
- `src/modules/[module_name]/components/` owns module-specific UI building blocks; `service/` owns that module's API/service access using existing request and response contracts.
- `src/modules/[module_name]/features/feature_name.feature.tsx` is a container component that composes the components needed for that feature. It can use module services, shared UI, and existing state patterns when needed; `features/` is not merely a hooks/state folder or a compulsory nested module layer.
- `src/components/ui/` holds reusable UI primitives, while `src/components/common/` holds genuinely cross-module components. Keep module-specific components in their module instead of promoting everything into shared UI.
- `src/libs/` holds shared infrastructure, SDK/transport helpers, and utilities, not a second home for feature-specific business logic. Preserve public module boundaries and avoid shared code importing feature-private internals or creating circular dependencies.

Keep routing, entry points, assets, tests, and native platform files in the framework/bundler's required locations, including files outside `src/`. Do not install a router or force web-only UI onto React Native merely to match the example. Preserve server/client boundaries, response contracts and runtime safeguards from [CODING_STYLE.md](CODING_STYLE.md), and platform-appropriate accessibility/responsiveness checks from [QA.md](QA.md).

## Full-Stack Example

Use this layout only if separate applications/packages fit the project. A single deployable application with framework-native backend routes can remain a single application.

```text
project/
|-- server/                     # Server layout above
|-- client/                     # Web or React Native client layout above
|-- packages/
|   \-- contracts/             # Only genuinely shared API contracts
\-- tests/
    \-- e2e/                   # Cross-application critical journeys
```

Share only stable, necessary contracts such as request/response schemas or types. Do not share server secrets, ORM models as public APIs, or authorization decisions with the client. Keep package ownership and build dependencies explicit. Reuse existing workspace and delivery tooling instead of adding it automatically; see [PIPELINE.md](PIPELINE.md).

When both web and native clients exist, apply the client pattern within each existing application root. This example does not require combining them into a new workspace or sharing platform-specific UI.

## Checklist

- Identify the requested server, web, React Native, or full-stack scope from manifests, routes, imports, and existing code.
- Read applicable project and nested instructions before selecting file locations.
- Map server code to feature/controller/service/DTO/middleware responsibilities and genuinely shared `common/` code; map client code to app composition, modules, shared components, and libraries.
- Check that screens/pages render the named feature container, which composes the needed components without duplicating UI assembly or inventing unnecessary layers.
- For endpoint changes, verify middleware ordering, validated query contracts, and ownership of authorization-scoped filtering, sorting, and pagination.
- Add only necessary files; check imports, public interfaces, asset handling, server/client boundaries, and framework-required web/native locations.
- Place tests where existing tooling discovers them; follow [QA.md](QA.md) and load [PERFORMANCE.md](PERFORMANCE.md) when costs or performance are relevant.
- Review the diff for unrelated moves, unused layers, generated artifacts, and exposed secrets; report any intentional structural change.
