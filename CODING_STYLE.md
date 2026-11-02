# Coding Style

## Applicability

Read this guide for code changes, including frontend, backend, and full-stack work. Follow the target project's instructions, language, tooling, and established conventions first. [AGENTS.md](AGENTS.md) defines scope and permission boundaries; this guide does not authorize unrelated refactors, installations, or external operations.

## Comments And Naming

- Avoid unnecessary comments that repeat what the code plainly shows. Prefer clear code and meaningful names over comments explaining obvious steps.
- Keep concise comments for intent, constraints, invariants, and non-obvious tradeoffs. Keep them accurate; retain required documentation, license notices, and necessary tooling directives.
- Use meaningful, domain-specific variable, function, and component names. Name values for what they represent and functions for their action or result; avoid vague names and unexplained abbreviations. Follow the project's naming conventions.

## Reuse And DRY

- Before adding or extracting a function, hook, component, utility, service, or contract, search the validated active project's existing implementations, exports, callers, and relevant tests. Reuse a suitable implementation directly instead of copying, wrapping, or recreating equivalent behavior. Lucius examples and unrelated projects are not the active project's existing code.
- Read candidates before reusing them. Preserve public interfaces and server/client/package boundaries; extend existing code minimally only for a compatible task requirement. Introduce new code only for a demonstrated gap and explain why existing candidates do not fit.
- Follow DRY (Don't Repeat Yourself) for genuinely repeated behavior and business rules. Extract shared code only after project-local discovery shows a real common responsibility or explicit shared-use requirement. Keep interfaces focused and feature-specific code near its owner; follow [STRUCTURE.md](STRUCTURE.md).
- Do not combine unrelated responsibilities merely because their code looks similar. Avoid speculative abstractions, unnecessary layers, and one-use wrappers that add no value. Reusability should reduce duplication without making code harder to understand or change.

## Frontend Response Handling

- Define or reuse a type or contract for the expected API response using the project's existing language and tooling. Match the actual nested payload, including optional and nullable fields. Reuse existing generated types or schemas rather than duplicating contracts; do not migrate the frontend language solely for this rule.
- Consume the response in its existing shape when it already fits the UI. Prefer dot notation for known properties, such as `response.data.order.customer.name`. Use appropriate array access or bracket notation for array items and dynamic keys.
- Avoid unnecessary `map`, `flatMap`, `forEach`, `filter`, copying, renaming, flattening, and data normalization just to access response values. Do not replace these operations with equivalent unnecessary loops.
- Use transformations when the UI or an established consumer actually requires them, including collection rendering, required filtering, deliberate view-model conversion, or normalization for an existing cache or state model. Keep them purposeful and do not change the API contract merely to avoid client transformations; see [PERFORMANCE.md](PERFORMANCE.md) for relevant costs.
- Static types do not validate runtime JSON. Preserve required parsing and validation, optional-field checks, and loading, empty, error, and success handling. Follow [SECURITY.md](SECURITY.md) and [QA.md](QA.md); do not bypass safeguards or install a validator merely to satisfy this style guide.

Illustrative TypeScript with synthetic data. A real response comes from the project's existing request/parsing layer:

```ts
type OrderResponse = {
  data: {
    order: {
      id: string;
      customer: { name: string };
    };
  };
};

const response: OrderResponse = {
  data: {
    order: {
      id: "example-order",
      customer: { name: "Example Customer" },
    },
  },
};

const customerName = response.data.order.customer.name;
```

## File Size

- Keep new or substantially changed authored source files below 500 lines of code. Split growing files by cohesive responsibility into appropriate functions, components, or modules before reaching the limit.
- Generated and vendor artifacts are excluded. Leave unrelated existing oversized files untouched; this rule does not authorize repository-wide cleanup.
- Do not compress formatting or fragment code arbitrarily to game the limit. If mandatory project constraints prevent a sensible split, report the conflict instead of silently disregarding the rule.
- This is an authoring rule, not automatic enforcement. Do not add file-length tooling or configuration solely to enforce it.

## Checklist

- Remove redundant comments while preserving useful explanations and required notices or directives.
- Verify that names communicate domain meaning and follow project conventions.
- Search the active project's implementations and callers first, reuse compatible code, and justify new helpers or abstractions with a demonstrated gap.
- Match frontend response contracts to actual payloads and access known properties directly; justify transformations and preserve runtime safeguards and UI states.
- Keep affected authored source files below 500 lines of code through cohesive separation, without unrelated refactors or artificial fragmentation.
