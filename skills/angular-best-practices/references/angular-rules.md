# Angular rules (frontend_rh)

Sections 1-7 of skill `angular-best-practices`. Sections 8-18 (templates, SCSS, tables) are in `angular-ui-rules.md`.


## 1. Clean Components & Types

- **No `any` Types**: canonical rule and examples live in `typescript-best-practices` skill §1 — also enforced project-wide via `angular-component-structure`'s Hard Rules.
- **`ngOnDestroy` Placement**: The `ngOnDestroy` lifecycle hook must ALWAYS be placed at the very end of the component class (bucket 5 in `angular-component-structure`'s class-member ordering — see that skill for the full 8-bucket ordering this project enforces).

## 2. Component Lifecycle & Constructor Separation

Constructor MUST inject dependencies only. Form initialization, data loading, and subscriptions belong in `ngOnInit`, NOT the constructor.

**Rules**:
1. **Constructor**: ONLY inject services/dependencies via constructor params. No form creation, API calls, subscriptions, or data initialization.
2. **ngOnInit**: Initialize component state (call `createForm()`, `loadData()`, set defaults).
3. **Separate methods**: Create `createForm()`, `loadData()`, etc. as private methods for single responsibility and testability.
4. **ngOnDestroy**: Always implement for components with subscriptions (use `takeUntil(this.destroy$)`, then `next(true)` and `complete()`; never `unsubscribe()` on the Subject).
5. **Interfaces**: Implement `OnInit, OnDestroy` when used.

**Example** (state goes through the Store, never a business service or `HttpClient`; see `angular-component-structure`):
```typescript
export class MyComponent implements OnInit, OnDestroy {
  private readonly destroy$ = new Subject<boolean>();
  formData: FormGroup;

  constructor(private readonly fb: FormBuilder, private readonly store: Store) {}

  ngOnInit(): void {
    this.createForm();
    this.initSubscriptions();
    this.loadData();
  }

  private createForm(): void {
    this.formData = this.fb.group({ /* ... */ });
  }

  private initSubscriptions(): void {
    this.store.select(MyState.data)
      .pipe(takeUntil(this.destroy$))
      .subscribe(
        (data: MyData) => this.formData.patchValue(data),
        (error: unknown) => this.sweetAlertHelper.captureException(error)
      );
  }

  private loadData(): void {
    this.store.dispatch(new LoadMyData());
  }

  // ALWAYS AT THE BOTTOM
  ngOnDestroy(): void {
    this.destroy$.next(true);
    this.destroy$.complete();
  }
}
```

## 3. Lazy Modules & Permissions

When creating new feature modules with lazy loading:

- Use `loadChildren` with dynamic import in parent routing module.
- In the module's routing file, protect the route with `NgxPermissionsGuard`.

**Example**:
```typescript
{
  path: '',
  component: ViaticosComponent,
  data: {
    title: 'Liquidacion Viaticos',
    option: ROUTES_PERMISSION_CODES.viaticos_view,
    permissions: {
      only: ROUTES_PERMISSION_CODES.viaticos_view,
      redirectTo: '/admin/403'
    }
  },
  canActivate: [NgxPermissionsGuard]
}
```
**Important**: Match permission codes across menu, routing, and guards. Set `redirectTo: '/admin/403'` for unauthorized access.

## 4. `*ngFor` with `trackBy`

**Always provide trackBy to *ngFor. Without it, Angular recreates all DOM nodes on list change.**

**Good (trackBy returns stable identity)**:
```typescript
export class MyComponent {
  trackById(_index: number, item: MyItem): number {
    return item.id;
  }
}
```
```html
<div *ngFor="let item of items$ | async; trackBy: trackById">
  {{ item.name }}
</div>
```

**Rules**:
- Always use trackBy on `*ngFor` (no exceptions).
- Return stable identifier (ID field, primary key).
- Use method reference in template (not arrow function call, e.g. `trackBy: trackById`).
- Do not use index as trackBy if order can change.
- **Never iterate an inline array literal defined directly in the template** (e.g. `*ngFor="let field of [ {key: 'a'}, {key: 'b'} ]"`). Angular re-evaluates that literal on every change-detection cycle, producing a new array reference each time — `trackBy` can't help because `*ngFor` still sees "a different array" and diffs/rebuilds accordingly on every keystroke or unrelated value change nearby. Hoist the array to a `readonly` class field (e.g. `readonly initialCallFields = [...]`) and iterate that instead. Found via review of `application-detail-dialog.component.html` (chain/recruitment-10), 2026-08-25.

## 5. Uso Obligatorio de `QuickSearchComponent` (`<app-quick-search>`) en Selectores

**Regla Mandatoria**:
Siempre que se implemente o modifique un selector de opciones en cualquier formulario o vista del proyecto, se MUST utilizar el componente reutilizable `<app-quick-search>` (`QuickSearchComponent`), el cual integra el buscador con `ngx-mat-select-search`. Queda prohibido el uso de `<mat-select>` directo o selectores `<select>` nativos sin buscador.

**Patrón de Implementación**:
```html
<app-quick-search
  [dataProvider]="(opciones$ | async)"
  [placeholder]="'Seleccione...'"
  [displayField]="'nombre'"
  [appearance]="'outline'"
  formControlName="campo_id"
  (selectionChange)="onSelectionChange($event)">
</app-quick-search>
```

**Reglas**:
- Enlazar reactivamente mediante `formControlName` o `[formControl]` (implementa `ControlValueAccessor`).
- Pasar la propiedad `displayField` según el modelo (ejemplo: `'nombre'` para departamentos/ciudades o la propiedad equivalente).
- Utilizar `(selectionChange)` cuando se requiera reaccionar al cambio de selección o ejecutar cascadas dinámicas (ej. Departamento -> Ciudad).

## 6. Uso Obligatorio de `SweetAlertHelper` en lugar de `sweetalert2` directo

**Regla Mandatoria**:
Queda estrictamente prohibido importar `Swal` directamente de `sweetalert2` (`import Swal from 'sweetalert2'`) en componentes, servicios o directivas. Se DEBE inyectar el servicio centralizado `SweetAlertHelper` (`@services/core/sweet-alert-helper.service`) a través del constructor.

**Patrón de Implementación**:
```typescript
import { Component } from '@angular/core';
import { SweetAlertHelper } from '@services/core/sweet-alert-helper.service';

@Component({ ... })
export class MyComponent {
  constructor(
    private readonly sweetAlertHelper: SweetAlertHelper
  ) {}

  save(): void {
    // Éxito / Alerta personalizada
    this.sweetAlertHelper.createCustomAlert({
      title: 'Título',
      text: 'Registro guardado correctamente',
      type: 'success'
    });

    // Manejo de excepciones
    this.sweetAlertHelper.captureException({
      message: 'Error al procesar la solicitud'
    });
  }
}
```

**Reglas**:
- Inyectar `SweetAlertHelper` vía constructor como dependencia (`private readonly sweetAlertHelper: SweetAlertHelper`).
- Usar `this.sweetAlertHelper.createCustomAlert({ ... })` o `this.sweetAlertHelper.captureException(...)`.
- Prohibir `import Swal from 'sweetalert2'` o `import swal from 'sweetalert2'` en el código nuevo o refactorizado.

## 7. Single Subscription in `ngOnInit` / `initSubscriptions`

**Never subscribe to Store selectors or Observables inside reload/refresh methods (`loadData()`, `loadItems()`) that can be called repeatedly.**

- **Problem**: Calling `.subscribe()` inside `loadData()` every time a modal closes or a refresh action occurs creates memory leaks and accumulates duplicate active subscribers executing multiple times per emission.
- **Rule**: Set up all subscriptions ONCE in `ngOnInit()` (or a dedicated `initSubscriptions()` called from `ngOnInit`) with `takeUntil(this.destroy$)`. Reload methods should ONLY dispatch actions (`this.store.dispatch(new Action())`).

**Example**:
```typescript
// ❌ BAD (Subscribes on every reload)
loadData(): void {
  this.store.dispatch(new GetItemsAction());
  this.items$.pipe(takeUntil(this.destroy$)).subscribe(items => { ... });
}

// ✅ GOOD (Subscribe once, reload only dispatches)
ngOnInit(): void {
  this.initSubscriptions();
  this.loadData();
}

private initSubscriptions(): void {
  this.items$.pipe(takeUntil(this.destroy$)).subscribe(items => {
    this.gridData = items || [];
  });
}

private loadData(): void {
  this.store.dispatch(new GetItemsAction());
}
```
