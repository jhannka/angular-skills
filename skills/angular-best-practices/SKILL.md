---
name: angular-best-practices
description: "Trigger: Angular component, module, template, ngOnInit, subscription, lazy module, permissions guard, ngFor trackBy, select, SweetAlert in frontend_rh. Lifecycle, subscriptions, routes and template rules for Angular 13."
license: Apache-2.0
metadata:
  author: rh-team
  version: "2.1"
  scope: project
  stack: "Angular 13.2, TypeScript 4.5, RxJS 6.5, NGXS 3.7, ag-grid 18"
---

## Activation Contract

Load before creating or changing an Angular component, module, route, template or SCSS under `src/app`. Class member order and state isolation: skill `angular-component-structure`.

## Hard Rules

- The constructor only injects. Form creation, loading and subscriptions go in `ngOnInit` through `createForm()`, `initSubscriptions()` and `loadData()`.
- Subscribe once, in `ngOnInit`/`initSubscriptions`, with `takeUntil(this.destroy$)`. Reload methods only dispatch actions, never subscribe.
- `ngOnDestroy` is the last member: `destroy$.next(true)` then `destroy$.complete()`; never `unsubscribe()` on the Subject.
- Lazy feature modules use `loadChildren` with a dynamic import; the route is protected with `NgxPermissionsGuard` and `redirectTo: '/admin/403'`. Permission codes match across menu, routing and guards.
- Every `*ngFor` has `trackBy` returning a stable id; never iterate an inline array literal.
- Every select control is `<app-quick-search>`; raw `<mat-select>` or native `<select>` without search is not allowed.
- Alerts go through `SweetAlertHelper`; no `import Swal from 'sweetalert2'` in new or refactored code.
- An event binding calls the method: `(click)="validate()"`. `(click)="validate"` only evaluates the reference and the button does nothing (PR 1864, "Validar datos" in `empleado_info_general`).
- Template and SCSS details (heading semantics, accented Spanish text, `rem` units, blob downloads, `nowrap` tables, shared wizard config, numeric `@Input`, template getters, DOM event guards, `ng-deep`, large tables) are in `references/angular-ui-rules.md`.

## Decision Gates

| Need | Read |
|------|------|
| Lifecycle, subscriptions, lazy routes, `trackBy`, selector and alert patterns with examples | `references/angular-rules.md` |
| Template, SCSS, tables, accessibility, downloads | `references/angular-ui-rules.md` |
| Class member order, state in NGXS | skill `angular-component-structure` |
| Typing and forms | skill `typescript-best-practices` |

## Execution Steps

1. Read the matching section of the reference.
2. Match the surrounding module: same permission codes, same alert and selector components.
3. Run `npm run lint` and the spec of the touched component (`ng test --include <spec>`).

## Output Contract

Report changed files, the lint and spec results, and anything not run.
