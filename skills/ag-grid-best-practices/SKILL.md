---
name: ag-grid-best-practices
description: >
  Global AG-Grid best practices: Ivy compatibility, responsive layout (flexbox), theming, and state persistence.
  Trigger: When configuring ag-grid-angular, setting up columns/renderers, fixing layout issues, or persisting grid state.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

# AG-Grid Global Best Practices

This skill consolidates global rules and patterns for AG-Grid development.

## 1. Angular 13+ (Ivy) Compatibility

**Never use `AgGridModule.withComponents([])` in Angular 13+. Use `AgGridModule` directly.**

- Import `AgGridModule` normally in your module.
- Declare custom cell renderers in your module.
- Configure renderers via `frameworkComponents` in `GridOptions`.

**Example**:
```typescript
// Component
export class MyComponent {
  gridOptions: GridOptions = {
    frameworkComponents: {
      btnEdit: BtnNominaTableCellRendererComponent
    },
    columnDefs: [
      { cellRenderer: 'btnEdit' }
    ]
  };
}
```

## 2. Responsive Layout & Theming

**Always use `ag-theme-balham` class.**

- **Theme**: ALWAYS use `class="ag-theme-balham"`, never `ag-theme-alpine`. Confirmed against the codebase: 206 usages of `ag-theme-balham`, 0 of `ag-theme-alpine`.
- **Grid Sizing**: use `calc(100vh - Xpx)` in the component SCSS with `display: block` on `<ag-grid-angular>` — this is this codebase's actual, established convention (163 SCSS files use this pattern, including files edited the same day this was last verified), not the flexbox + `min-height: 0` approach a prior version of this rule recommended. Full detail and the rest of the AG-Grid v18-specific rules (column resize, async auto-sizing, `id` column hiding, `cellRenderer` sanitization) live in the `ag-grid-style` skill — read that one for HTML/SCSS/TS implementation detail; this section only covers the corrected sizing/theming fact.

⚠️ **Correction (2026-08-28):** a prior version of this rule said *"Never use fixed pixel heights or fragile `calc(100vh - Xpx)`. Use flexbox + `min-height: 0`"* — this was aspirational and never matched actual practice in this codebase. Verified via grep: `calc(100vh` appears in 163 `.scss` files (vs. 9 for `min-height:\s*0`, most of those likely unrelated to grid sizing), and the most recently modified files at verification time (same-day edits) still used the `calc(100vh - Xpx)` pattern — confirming it as the live convention, not legacy debt being phased out. `ag-grid-style` (this project's own authored skill, not a generic import) already documented `calc(100vh - Xpx)` correctly; this skill was the one out of sync with reality.

## 3. Grid State Persistence

Best practices for persisting and restoring AG Grid state (filters, columns) avoiding race conditions and typing errors.

- **Avoid Race Conditions**: When restoring state, wrap `setFilterModel` and `setColumnState` in a `setTimeout(() => {}, 300)`.
- **Typing**: Type `ColumnState` dynamically as `ReturnType<ColumnApi['getColumnState']>` to avoid `any` in older versions.
- **Global Helper**: Use a global `restoreGridState` helper.

**Example**:
```typescript
export function restoreGridState(
  gridApi: GridApi | undefined,
  gridColumnApi: ColumnApi | undefined,
  savedFilterModel: { [key: string]: unknown } | undefined,
  savedColumnState: ReturnType<ColumnApi['getColumnState']> | undefined
) {
  setTimeout(() => {
    if (gridApi && savedFilterModel) {
      gridApi.setFilterModel(savedFilterModel);
      gridApi.onFilterChanged();
    }
    if (gridColumnApi && savedColumnState) {
      gridColumnApi.setColumnState(savedColumnState);
    }
  }, 300);
}
```
