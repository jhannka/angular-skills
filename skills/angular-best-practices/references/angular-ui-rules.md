# Angular template and UI rules (frontend_rh)

Sections 8-18 of skill `angular-best-practices`.

## 8. HTML Heading Semantics (No Block Elements in Headings)

**Never place `<div>`, `<p>`, or other block elements inside `<h1>`-`<h6>` tags.**

- **Problem**: HTML specification prohibits block-level elements inside phrasing content tags (`<h1>`-`<h6>`). It breaks accessibility and DOM validation.
- **Rule**: Keep headings and action buttons as siblings wrapped in a flexbox container.

**Example**:
```html
<!-- ❌ BAD -->
<h2 class="title-module">
  Título
  <div class="ms-3"><button ...>Nuevo</button></div>
</h2>

<!-- ✅ GOOD -->
<div class="d-flex align-items-center">
  <h2 class="title-module mb-0">Título</h2>
  <button mat-stroked-button color="primary" class="ms-3" (click)="onNew()">
    <mat-icon>add</mat-icon> Nuevo
  </button>
</div>
```




## 9. User-Facing Spanish Text Must Carry Accents

Titles, subtitles, alerts and fallback error messages are reviewed for orthography: write "Provisión", "devolvió", "Anulación", never "Provision"/"devolvio". Run a quick search for unaccented words before committing UI copy.

## 10. Relative Units for Base Typography

Use `rem` (e.g. `font-size: 0.875rem` instead of `14px`) for base font sizes so they follow the user's root font size / zoom preferences.

## 11. Blob Downloads: Delay `URL.revokeObjectURL`

Revoking right after `link.click()` can cancel the download before the browser reads the blob. Revoke in a timeout:

```typescript
link.click();
document.body.removeChild(link);
setTimeout(() => URL.revokeObjectURL(url), 1000);
```

In specs, wrap the test in `fakeAsync`, assert `revokeObjectURL` was not called yet, then `tick(1000)` and assert it was.

## 12. `white-space: nowrap` Tables Need Horizontal Scroll

A table wrapper with `overflow: hidden` clips `nowrap` columns on mobile. Use `overflow-x: auto` (and `overflow-y: hidden` if the rounded border must clip) so the table scrolls instead of being cut.

## 13. Shared Wizard/Presentational Components: Configuration Over Hard-Coded Navigation

- Routes, labels and endpoints come from a typed config `@Input()`; the component never hard-codes them.
- Optional keys gate UI: render the history link only when `config.historyRoute` is set.

```ts
export interface WizardConfig {
  title: string;
  uploadEndpoint: string;
  historyRoute?: string; // optional: hides the link when absent
}
```

## 14. Normalize Numeric @Input Values

- Inputs such as `pageSize` may arrive as strings or `0`/`NaN`. Normalize in a setter.

```ts
@Input() set pageSize(v: number | string) {
  this._pageSize = Math.max(1, Math.floor(Number(v)) || 1);
}
```

## 15. Keep Template Logic in Getters

- Size labels and compound state flags live in getters, not in inline ternaries or chained pipes.
- Format numbers with `formatNumber` and `LOCALE_ID` instead of inline pipes/ternaries.

```ts
constructor(@Inject(LOCALE_ID) private locale: string) {}
get sizeLabel(): string {
  return `${formatNumber(this.file.size / 1024, this.locale, '1.0-1')} KB`;
}
```

## 16. Type-Guard DOM Events and Header Parsing

- Never cast `event.target`: check `event.target instanceof HTMLInputElement`.
- Parse `Content-Disposition` filenames with `decodeURIComponent` inside `try/catch` and fall back to the raw value.

```ts
const input = event.target;
if (!(input instanceof HTMLInputElement) || !input.files?.length) { return; }

let name = raw;
try { name = decodeURIComponent(raw); } catch { /* keep raw */ }
```

## 17. `ng-deep` Scope

- Acceptable only under `:host` and a component root class, to style Angular Material internals.
- Add a one-line comment saying why it is needed.

```scss
:host ::ng-deep .wizard-root .mat-step-header { /* Material internal: no input to style it */ }
```

## 18. Large Result Tables

- Use `ChangeDetectionStrategy.OnPush` and page the table with a "Mostrar más" button instead of rendering every row.
- `cell()`-style helper calls in the template are fine under OnPush with pages of about 50 rows.
