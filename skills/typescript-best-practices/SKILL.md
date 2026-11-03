---
name: typescript-best-practices
description: "Trigger: TypeScript, strict typing, any, FormControl, reactive form initialization, module structure, unhandled promise in frontend_rh. Typing, form and module rules for TypeScript 4.5."
license: Apache-2.0
metadata:
  author: rh-team
  version: "2.2"
  scope: project
  stack: "Angular 13.2, TypeScript 4.5, RxJS 6.5, NGXS 3.7, ag-grid 18"
---

## Activation Contract

Load before writing TypeScript, a reactive form, an interface or a feature module under `src/app`.

## Hard Rules

- Never `any`: use interfaces, generics or `unknown`. HTTP calls return `Observable<IResponse<T>>`; a call with no body returns `void`.
- No tsconfig enables `strict`, `strictNullChecks` or `noImplicitAny`; write code as if they were on, and do not add casts to silence them.
- Initialize form controls to `null` (booleans explicitly `false`/`true`) and load data later with `patchValue()` in `ngOnInit`.
- A feature module has one folder per component (`.ts`, `.html`, `.scss`); the `.module.ts`, `-routing.module.ts` and `.interface.ts` stay at the feature root; names are kebab-case.
- Check the runtime type (`typeof`, `instanceof`) before using an external payload; do not trust `as` casts.
- No `async` callback inside `.subscribe()` without a surrounding `try/catch`; prefer `from()` with `catchError` in the pipe.
- No inline `import(...)` types; use static imports at the top of the file.
- Domain interfaces live in `src/app/core/interfaces/*.interface.ts` (`@core/interfaces`), never inside a `*.service.ts`.
- A function that follows `rejection`, `cause` or `parent` links tracks the objects it visited (`Set`) and stops on a cycle; a self-referencing error must not overflow the stack (`src/app/utils/chunk-load-error.ts`, PR 1888).
- Validate an object before rest/spread destructuring. Check numeric ids with `id == null` or `Number.isFinite(id) && id > 0`, never `!id` (0 is falsy).

## Decision Gates

| Need | Read |
|------|------|
| Examples and rationale for each rule | `references/typescript-rules.md` |
| Component class order and injection rules | skill `angular-component-structure` |
| Selectors and state typing | skill `ngxs-best-practices` |

## Execution Steps

1. Read the matching section of the reference.
2. Type first (interface in `@core/interfaces`), then the code that uses it.
3. Run `npm run lint` and the touched spec (`ng test --include <spec>`).

## Output Contract

Report changed files, the lint and spec results, and anything not run.
