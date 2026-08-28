---
name: angular-component-structure
description: "Trigger: When creating, refactoring, or modifying Angular components, modules, or templates. Standardized Angular architecture and component structure guidelines."
license: MIT
metadata:
  author: frontend_rh
  version: "2.0"
---

# Skill: angular-component-structure

## Activation Contract

Activate this skill whenever designing, coding, editing, or auditing any Angular Component (TS controllers, HTML templates, SCSS styles, or Angular Modules).

## Hard Rules

- **State Isolation**: NEVER inject business services (e.g., `EmployeeService`, `PayrollService`). ALL business logic, HTTP calls, and data transformations MUST live in the NGXS State. Components must ONLY dispatch actions and select state.
- **Dependency Injection**: Only inject UI-related services (`Store`, `MatDialog`, `Router`, `ChangeDetectorRef`, `SweetAlertHelper`).
- **No Inline Styles**: NEVER use inline `style="..."` attributes in HTML. Put all styles inside the component SCSS file.
- **No Complex HTML Logic**: HTML templates MUST NOT contain mathematical, complex conditional, or expensive function logic. Use component **Getters** instead.
- **Strict Typing**: ALWAYS declare explicit variable types (native AG-Grid types, custom interfaces for error casting, exact API response signatures). All methods MUST declare an explicit return type. `any` is prohibited — canonical rule in `typescript-best-practices` skill §1.
- **Form Initialization**: ALWAYS initialize `FormControl` values to `null`, never `''`/`0`/`false` — full pattern and rationale in `typescript-best-practices` skill §2.
- **Select Controls**: ALWAYS use `<app-quick-search>` (`QuickSearchComponent`) for select inputs — RAW `<mat-select>`/native `<select>` without search are prohibited. Full implementation pattern in `angular-best-practices` skill §5.
- **Subscription Error Handling**: ALWAYS implement error callbacks in every `.subscribe()` block to prevent UI freeze.
- **Alerts**: NEVER inject `ToastrService` or use raw external notifications. ALWAYS inject `SweetAlertHelper` in the constructor — full implementation pattern in `angular-best-practices` skill §6.
- **API Endpoint Definition**: NEVER hardcode API URLs in services; load from `app.constants.ts` via `EndPoint()`, never interpolate query params (use `HttpParams`) — full detail in `api-design-best-practices` skill §2-3.
- **UX in valueChanges**: NEVER force values inside `valueChanges` synchronously without `{ emitEvent: false }`.
- **CSS Performance**: NEVER use heavy rendering properties (`box-shadow`, `filter`, `backdrop-filter`) on scrollable grids, lists, or fixed layouts to avoid lag.

## Decision Gates

| Aspect | Action | Reason |
| --- | --- | --- |
| State access | Use `@Select(state => ...)` or `store.select()` | Ensures single source of truth |
| Class Member Visibility | Use `readonly` for constants, `private` for local properties, `public` for template-accessed properties | Keeps API surface minimal and clean |
| Constructor logic | Only inject dependencies; keep body EMPTY | Lifecycle hooks (`ngOnInit`) handle initialization |
| No matching NGXS action exists for a needed fetch | Do NOT force-fit an unrelated existing action just to satisfy "no business service" — check what side effects that action's `@Action` handler has first (e.g. does it overwrite shared/global state used by another feature?). If reusing it would leak an unrelated side effect, keep the direct service injection and add a short comment stating why, rather than silently corrupting shared state | A generic-looking action (e.g. `FunnelLoadApplication`) can carry feature-specific side effects (mutating a kanban's `selectedApplication`); reusing it from an unrelated component is a correctness bug disguised as a compliance fix. Confirmed on `EvaluationViewerComponent` (chain/recruitment-06-evaluation-viewer): kept `EvaluationService` injection, documented inline, rather than misusing `FunnelLoadApplication` |
| `@Input() set x(...)` accessors | Goes in bucket 8 (Getters/Setters) with the other getters, NOT with the plain `@Input()` fields in bucket 3 | Bucket 3 is for plain typed fields; a `set` accessor is structurally a setter regardless of the `@Input` decorator |

## Execution Steps

1. Order class declarations strictly:
   - 1. Observables, Selectors (`@Select`), and `destroy$: Subject<boolean>` (exact type — `next(true)` requires the boolean generic, not `Subject<void>`).
   - 2. Grid options, Column definitions, Table datasources, and `@ViewChild`.
   - 3. Strictly-typed Class variables (No `any`) — plain `@Input()` fields included; `@Input() set` accessors do NOT belong here (see bucket 8).
   - 4. Constructor (Injection only).
   - 5. Lifecycle hooks (`ngOnInit` first, `ngOnDestroy` ALWAYS last).
   - 6. Initialization helper methods (`createForm()`, etc.).
   - 7. Presentation & Event handlers (`onSave()`, etc.).
   - 8. Component Getters and Setters, including `@Input() set` accessors.
2. In `ngOnDestroy`, ALWAYS invoke `this.destroy$.next(true)` and `this.destroy$.complete()` to free RxJS subscriptions.

## Output Contract

Ensure the following outputs are produced:
- Valid TS Controller following the specified member declaration ordering.
- Strictly-typed class properties and explicit return types.
- Clean HTML template and performance-optimized SCSS files.
