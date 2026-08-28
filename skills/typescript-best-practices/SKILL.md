---
name: typescript-best-practices
description: >
  Global TypeScript and Angular Form/Module best practices: strict typing, form initialization, and module structure.
  Trigger: When writing TypeScript code, defining Angular modules, or initializing reactive forms.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

# TypeScript & Angular Structure Global Best Practices

This skill consolidates global rules and patterns for TypeScript typing and Angular component/form structure.

## 1. TypeScript Strict Typing

**Never use `any`. Use specific interfaces, types, or generics.**

- **HTTP Responses**: Type as `Observable<IResponse<IViaticos>>`.
- **Method Returns**: Use `void` when a method (like DELETE) returns nothing.
- **Unknowns**: Use `unknown` if the type is truly unknown.
- **Strict Mode**: Ensure `strictNullChecks` and `strictPropertyInitialization` are enabled.

**Example**:
```typescript
// ❌ BAD
changeStatus(data: any): Observable<any> { ... }

// ✅ GOOD
changeStatus(data: IChangeStatusRequest): Observable<IResponse<IViaticos>> { ... }
```

## 2. Form Initialization Pattern

**Form controls MUST initialize to `null` (not empty strings) to prevent premature validation errors before data loads.**

- Every text/numeric/select field initializes to `null`.
- Boolean toggles initialize to `false` or `true` explicitly.
- Load data later via `patchValue()` in `ngOnInit`.

**Example**:
```typescript
createForm() {
  this.formData = this.fb.group({
    id: [null],
    empleado_id: [null, Validators.required],
    activo: [true], // boolean exception
  });
}
```

## 3. Module & Component Structure

**Feature modules MUST separate components into subdirectories, not mix all files in one folder.**

- **Folder per Component**: Each component gets its own folder (`viaticos/viaticos.component.ts`, `.html`, `.scss`).
- **Module Root**: The `.module.ts`, `-routing.module.ts`, and `.interface.ts` files stay at the feature root.
- **Naming**: Use kebab-case matching the component selector prefix.

**Example Structure**:
```
src/app/admin/pages/nomina/viaticos/
  ├── viaticos/                   (Component folder)
  │   ├── viaticos.component.ts
  │   ├── viaticos.component.html
  │   └── viaticos.component.scss
  ├── viaticos.module.ts          (Module root)
  ├── viaticos-routing.module.ts
  └── viaticos.interface.ts
```

## 4. Runtime Type Checking vs Type Assertions

**Avoid forcefully casting external payloads or any types (`as string | Date`) if you are not sure of their runtime type.**

- When a variable might be a `string` or a `Date`, check its runtime type before manipulating it.
- **Example:** Use `typeof dateVal === 'string'` or `dateVal instanceof Date` instead of just writing `castDate(dateVal as Date)` which will fail silently at runtime if it's not a Date.

## 5. RxJS Unhandled Promise Rejections

**Avoid passing `async` callbacks directly into RxJS `.subscribe()` blocks without proper exception handling.**

- Passing `subscribe(async res => { ... })` often hides unhandled promise rejections (like a sweetalert that the user cancels or throws).
- **Rule:** Use `async/await` inside `subscribe` ONLY if you wrap the entire block in a `try/catch`.
- **Better:** Convert promises to observables using `from()` and handle them in the `.pipe()` chain using `catchError()`.

## 6. No Inline Dynamic Imports for Typings

**Strictly prohibit the use of inline `import(...)` for typing parameters or variables.**

- **Why:** Using `(state: { my_feature: import('...').StateModel })` inside a `.select()` or any other method is considered a hacky workaround. It breaks clean architecture, clutters the component logic with file paths, and ruins readability.
- **Rule:** Always use static imports at the top of the file (`import { StateModel } from '...'`) and type your parameters cleanly.
- **NGXS Specific:** For state selection, prefer using static selectors exposed by the State class itself (e.g., `this.store.select(ConfigNominaState.info)`) rather than manually typing the callback state parameter. For selector memoization and strict state typing beyond this rule, see `ngxs-best-practices` skill §2-3.

## 7. Decouple Domain Entities from HTTP Infrastructure

**Domain interfaces and models must NEVER be defined inside `*.service.ts` files.**

- **Problem**: When domain entities are declared inside HTTP services, other modules (components, NGXS actions, state) are forced to import from infrastructure services, creating tight coupling and circular dependency risks.
- **Rule**: Place shared domain contracts in dedicated interface files under `src/app/core/interfaces/*.interface.ts` (or `@core/interfaces`).

## 8. Defensive Object Validation & Falsy ID Checks

**Always validate objects before rest/spread destructuring, and avoid falsy checks on numeric IDs.**

- **Destructuring Validation**: `if (!data || typeof data !== 'object') throw new Error(...)` before executing `const { id, ...payload } = data;` to avoid runtime `TypeError`.
- **Falsy ID Validation**: Never check numeric IDs with `if (!id)`, as `0` is a valid number but falsy in JavaScript. Always use `if (id == null)` or `if (typeof id !== 'number' || !Number.isFinite(id) || id <= 0)`.

