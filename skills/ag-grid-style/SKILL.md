---
name: ag-grid-style
description: "Trigger: When implementing or modifying an AG-Grid component. Standardized AG-Grid implementation matching the liquidaciones module convention."
license: MIT
metadata:
  author: frontend_rh
  version: "2.0"
---

# Skill: ag-grid-style

## Activation Contract

Activate this skill whenever writing, editing, or refactoring AG-Grid instances, column definitions, or grid-related layout configurations.

## Hard Rules

- **Theme**: ALWAYS use the `ag-theme-balham` class on `<ag-grid-angular>`. Do NOT use `ag-theme-alpine`.
- **HTML Layout**: Wrap the grid in standard Bootstrap container row/columns. Pass only `[rowData]` and `[gridOptions]` in HTML template. Column definitions MUST be defined programmatically inside `gridOptions`.
- **SCSS Height**: ALWAYS use `calc(100vh - Xpx)` in the SCSS file for responsiveness. Set `display: block` on `<ag-grid-angular>`. Do NOT use fixed inline height style attributes.
- **AG-Grid v18 Column Resize**: ALWAYS set `enableColResize: true` directly in `gridOptions` to support resizing. Do NOT use `resizable: true` in `defaultColDef` (v18 incompatible).
- **Sensitive Fields**: NEVER show `id` columns to the user; hide them programmatically or omit from column definitions.
- **Asynchronous Auto-sizing**: ALWAYS use `onRowDataChanged` inside `gridOptions` to auto-size columns asynchronously using `autoSizeColumns()` for all column IDs. NEVER use `sizeColumnsToFit()` to force grid-stretching.
- **Security**: ALWAYS sanitize input values inside custom string templates in `cellRenderer`.

## Decision Gates

| Use Case | Action | Rule |
| --- | --- | --- |
| Grid height sizing | Set `calc(100vh - 300px)` in SCSS | Dynamic fits prevents scrollbar issues |
| Column resizing | Set `enableColResize: true` in `gridOptions` | Standard resize capability |
| Pagination integration | Use standard `<mat-paginator>` below the grid | Unified pagination UI |

## Execution Steps

1. Structure component HTML using standard Bootstrap row/column layout.
2. Formulate programmatical columns and configure `enableColResize: true` inside TS controller.
3. Hook `onRowDataChanged` in `gridOptions` to iterate and auto-size columns asynchronously.
4. Establish responsive dynamic height in component SCSS file.

## Output Contract

Ensure the following artifacts or outputs are produced:
- Valid component template using `ag-theme-balham`.
- TS controller containing compliant `GridOptions`.
- Responsive SCSS file with block display and dynamic height.
