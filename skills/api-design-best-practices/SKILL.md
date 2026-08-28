---
name: api-design-best-practices
description: >
  Global HTTP API best practices: pagination, parameter validation, and URL construction.
  Trigger: When designing HTTP services, API clients, optimizing data fetching, or building URLs with parameters.
license: Apache-2.0
metadata:
  author: gentleman-programming
  version: "1.0"
---

# API Design & HTTP Global Best Practices

This skill consolidates global rules and patterns for API design and HTTP services.

## 1. HTTP Pagination

**Never load entire datasets in one request. Paginate at the API level.**

- Provide sensible defaults (`page=1`, `pageSize=50`).
- Include `total` in the response for UI pagination.
- Track `page`/`pageSize` in your state to allow for reproducibility and easy navigation.

**Example**:
```typescript
get(page: number = 1, pageSize: number = 50): Observable<IResponse<IPaginatedViaticos>> {
  const url = CONSTANT.VIATICOS.URL.BASE;
  const params = new HttpParams()
    .set('page', page.toString())
    .set('pageSize', pageSize.toString());
  return this.http.get<IResponse<IPaginatedViaticos>>(url, { params });
}
```

## 2. Parameter Validation (URL Construction)

**Validate before interpolation. Never trust parameters in URLs.**

- Fail fast: throw sync errors before building the URL and making the HTTP call.
- Validate numeric IDs (type, `isFinite`, positive).
- Validate string codes (type, non-empty).
- Do not trust TypeScript types at runtime (e.g. `number` could be `NaN`).

**Example**:
```typescript
private validateId(id: any): void {
  if (typeof id !== 'number' || !Number.isFinite(id) || id <= 0) {
    throw new Error(`Invalid ID: ${id}. Must be a positive number.`);
  }
}

show(id: number): Observable<IResponse<IViaticos>> {
  this.validateId(id);  // Fail fast
  const url = `${CONSTANT.VIATICOS.URL.BASE}/${id}`;
  return this.http.get<IResponse<IViaticos>>(url);
}
```

## 3. Centralized URLs & Immutable Payloads

**Always centralize endpoint URLs in `CONSTANT` and use immutable destructuring for request payloads.**

- **Centralized URLs**: Use `CONSTANT.MODULE.URL.BASE` instead of constructing inline URL strings or hardcoding paths in services.
- **Immutable Payload Destructuring**: Never mutate input objects using `delete` operators (e.g. `delete data.id`). Instead, use object destructuring to exclude internal IDs or navigation properties: `const { id, navigationProp: _, ...payload } = data;`.
- **Clean Code Without Comments**: Production service code must be self-documenting through clean naming, strong typing, and structure without redundant JSDoc/inline comments.

**Example**:
```typescript
@Injectable({ providedIn: 'root' })
export class CiudadesService {
  private readonly baseUrl = CONSTANT.CIUDADES.URL.BASE;

  constructor(private readonly http: HttpClient) {}

  save(data: ICiudades): Observable<IResponse<ICiudades>> {
    const { id: _id, departamento: _departamento, ...payload } = data;
    return this.http.post<IResponse<ICiudades>>(this.baseUrl, payload);
  }
}
```
