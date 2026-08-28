---
name: ngxs-best-practices
description: >
  Global NGXS best practices: typed actions, no action barrels, selector memoization, and state performance.
  Trigger: When creating Redux/NGXS actions, state handlers, writing selectors, or optimizing state performance.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

# NGXS Global Best Practices

This skill consolidates global rules and patterns for NGXS (State Management).

## 1. Actions & Explicit Imports

- **Never Create Action Barrels**: Do not export all actions as an object (`export const Actions = { ... }`). This breaks tree-shaking and hides dependencies.
- **Explicit Imports**: Import only the specific actions you need directly from the `.actions.ts` file.
- **Typed Actions**: Every action must have strongly typed parameters in its constructor (no `any`) and a `static readonly type` in `[Feature] Verb` format.

**Example**:
```typescript
// Actions
export class ViaticosLiquidacionGetAction {
  static readonly type = '[ViaticosLiquidacion] Get item';
  constructor() {}
}

export class ViaticosUpdateStatusAction {
  static readonly type = '[ViaticosLiquidacion] Change status';
  constructor(public data: IChangeStatusRequest) {} 
}

// State Handler
@Action(ViaticosUpdateStatusAction)
changeStatus(ctx: StateContext<Model>, { data }: ViaticosUpdateStatusAction) {
  return this.service.changeStatus(data);
}
```

## 2. Selector Performance & Memoization

Avoid O(n) operations on large arrays in every selector access. Use NGXS memoization features.

- **Use `createSelector`**: Chain selectors by passing them as dependencies to `createSelector` rather than accessing state directly to filter/map arrays.
- **Avoid New References**: Only use `patchState` when data actually changes. Do not pass new array references continuously if data is the same.
- **Normalize Large Datasets**: For large arrays, normalize the state to an object indexed by ID for O(1) lookups instead of filtering large arrays.
- **API Pagination**: Avoid loading 20k items into the store. Paginate at the API level and store only the current page.

**Example (Memoization)**:
```typescript
@Selector()
static items(state: MyStateModel) {
  return state.items;
}

// ✅ Memoized selector depends on another selector
static byStatus(status: EEstadosViaticos) {
  return createSelector(
    [MyState.items], 
    (items) => items.filter(l => l.estado === status)
  );
}
```

## 3. Strict State Typing and Defaults

- **Strict Selector Typings**: Do not use implicit `any` in state selectors. When using `store.select()`, strongly type the state parameter so that property accesses are validated by TypeScript. Avoid `state => state.my_feature` (inferred as any if AppState is not global). Use `store.select((state: { my_feature: FeatureStateModel }) => state.my_feature.prop)`.
- **Safe Defaults**: Initialize array properties in NGXS `@State` defaults with empty arrays `[]`, never `null`, to prevent `null` pointer exceptions when components subscribe.

## 4. No Duplicate Persistent State (Derive Subsets via Selectors)

**Do NOT store filtered subsets or views of the same entity (e.g. `list` and `activos`) as separate properties in the `@State` model.**

- **Problem**: When items are created, updated, or removed, maintaining parallel collections (`list` and `activos`) causes synchronization bugs and stale data for downstream consumers.
- **Rule**: Store only the single source of truth (`list: Entity[]`) in the `@State` model. Create derived/memoized selectors to filter subsets.

**Example**:
```typescript
// ❌ BAD (Duplicate state collections)
export interface GenerosStateModel {
  list: Genero[];
  activos: Genero[];
}

// ✅ GOOD (Single source of truth + derived selector)
export interface GenerosStateModel {
  list: Genero[];
}

@State<GenerosStateModel>({
  name: 'generos',
  defaults: { list: [] }
})
@Injectable()
export class GenerosState {
  @Selector()
  static getList(state: GenerosStateModel): Genero[] {
    return state.list || [];
  }

  @Selector()
  static getActivos(state: GenerosStateModel): Genero[] {
    return (state.list || []).filter(item => item && item.activo);
  }
}
```

