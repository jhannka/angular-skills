---
name: ag-grid-best-practices
description: "Trigger: ag-grid-angular, grid columns, cell renderer, frameworkComponents, grid state, filter model, Ivy. AG-Grid 18 on Angular 13: renderers, theme, height and restoring grid state."
license: Apache-2.0
metadata:
  author: rh-team
  version: "2.1"
  scope: project
  stack: "Angular 13.2, TypeScript 4.5, RxJS 6.5, NGXS 3.7, ag-grid 18"
---

## Activation Contract

Load before configuring `ag-grid-angular`, custom cell renderers, grid layout or grid state persistence. The standard implementation (columns, resize, auto-size) is skill `ag-grid-style`.

## Hard Rules

- Use `AgGridModule` directly, never `AgGridModule.withComponents([])`. Declare custom renderers in the module and register them in `frameworkComponents` of `GridOptions`.
- Theme is `ag-theme-balham`; `ag-theme-alpine` is not used.
- Grid height is `calc(100vh - Xpx)` in the component SCSS with `display: block` on `<ag-grid-angular>`: this is the established convention, do not replace it with flexbox `min-height: 0`.
- Restore grid state through the shared `restoreGridState` helper (`src/app/utils/helper.ts`): `setFilterModel` and `setColumnState` run inside `setTimeout(..., 300)` to avoid a race.
- Type the column state as `ReturnType<ColumnApi['getColumnState']>` instead of `any`.
- A renderer's `refresh(params)` either reapplies the new params (call `agInit(params)`) and returns `true`, or returns `false` so ag-grid recreates it. Returning `true` without reading the params leaves stale data (PR 1864, `phone-row-card.component.ts`).

## Decision Gates

| Need | Read |
|------|------|
| Renderer example, sizing history, state helper | `references/ag-grid-rules.md` |
| Column definitions, resize and auto-size | skill `ag-grid-style` |
| Colors and typography | skill `ui-design-system` |

## Execution Steps

1. Read the matching section of the reference.
2. Follow `ag-grid-style` for the grid itself; use this skill for renderers, layout and state.
3. Run `npm run lint` and the component spec (`ng test --include <spec>`).

## Output Contract

Report changed files, the lint and spec results, and anything not run.
