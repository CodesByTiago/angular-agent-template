You are an expert in TypeScript, Angular, and scalable web application development.
You write functional, maintainable, performant, and accessible code following Angular and TypeScript best practices.

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Project Architecture & Directory Structure

Enforce a clear separation of concerns using a modular feature-based folder structure (`core`, `shared`, `features`):

- Keep feature-specific services, models, and components encapsulated inside their respective `features/` directory.
- Place domain-agnostic UI building blocks in `shared/`.
- Keep application-wide singletons (auth, logging, global HTTP handlers) inside `core/`.

```
src/app/
│
├── core/                       # Global singletons (Guards, Interceptors, Global Services)
│   ├── guards/                 # Route guards (e.g., auth.guard.ts)
│   ├── interceptors/           # HTTP Interceptors (e.g., jwt.interceptor.ts)
│   └── services/               # Global services (e.g., auth.service.ts)
│
├── shared/                     # Reusable UI elements across multiple features
│   ├── components/             # Generic UI components (buttons, modals, cards)
│   ├── directives/             # Custom directives
│   └── pipes/                  # Formatting pipes (e.g., phone.pipe.ts)
│
├── features/                   # Application domain pages & business logic
│   ├── auth/                   # Auth feature module
│   │   ├── login/              # Login view component
│   │   └── cadastro/           # Register view component
│   │
│   └── produtos/               # Products feature module
│       ├── lista-produtos/     # List view component
│       ├── detalhe-produto/    # Detail view component
│       ├── produto.service.ts  # Feature-specific service
│       └── produto.model.ts    # Feature-specific interfaces/types
│
├── app.component.ts            # Root component
└── app.config.ts               # Global Angular configurations
```

## Angular Best Practices

- NEVER include `standalone: true` in `@Component`, `@Directive`, or `@Pipe` decorators. Standalone is the default in Angular v20+.
  - ❌ BAD: `@Component({ selector: 'app-list', standalone: true, imports: [...] })`
  - ✅ GOOD: `@Component({ selector: 'app-list', imports: [...] })`
- Do NOT set `changeDetection: ChangeDetectionStrategy.OnPush` explicitly. `OnPush` is the default in Angular v22+.
- Use signals for state management
- Implement lazy loading for feature routes
- Do NOT use the `@HostBinding` and `@HostListener` decorators. Put host bindings inside the `host` object of the `@Component` or `@Directive` decorator instead
- Use `NgOptimizedImage` for all static images.
  - `NgOptimizedImage` does not work for inline base64 images.

## Routing

- For each feature that represents a page with routes, create a `[feature].routes.ts` file in the feature's directory
- Export routes using a constant with the naming convention `[FEATURE_NAME]_ROUTES` (e.g., `HOME_ROUTES`, `PRODUCTS_ROUTES`, `USER_ROUTES`)
- Use `loadChildren` in `app.routes.ts` to lazy-load feature routes and keep the main routes file clean
- Keep feature-specific route guards and logic inside the feature's route file
- Example structure:
  - `features/home/home.routes.ts` — exports `HOME_ROUTES`
  - `features/products/products.routes.ts` — exports `PRODUCTS_ROUTES`
  - `features/user/user.routes.ts` — exports `USER_ROUTES` (with auth guard applied)
  - `app.routes.ts` — uses `loadChildren` to dynamically import feature route constants

  Example app.routes.ts:

  ```typescript
  export const routes: Routes = [
    {
      path: 'products',
      loadChildren: () =>
        import('./features/products/products.routes').then((m) => m.PRODUCTS_ROUTES),
    },
  ];
  ```

## Accessibility Requirements

- It MUST pass all AXE checks.
- It MUST follow all WCAG AA minimums, including focus management, color contrast, and ARIA attributes.

### Components

- Keep components small and focused on a single responsibility
- Use `input()` and `output()` functions instead of decorators
- Use `computed()` for derived state
- Do NOT use inline `styles` or `template` strings for non-trivial components. Always extract styles to dedicated `.scss` files (`styleUrl: './component.component.scss'`) and templates to `.html` files (`templateUrl: './component.component.html'`).
- Prefer Signal Forms (`@angular/forms/signals`) for new forms. They are stable in Angular v22+ and provide signal-based state, type-safe field access, and schema-based validation
- When not using Signal Forms, prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead
- Do NOT use `ngStyle`, use `style` bindings instead
- When using external templates/styles, use paths relative to the component TS file.

## State Management

- Use signals for local component state
- Use `computed()` for derived state
- Keep state transformations pure and predictable
- Do NOT use `mutate` on signals, use `update` or `set` instead

## Templates

- Keep templates simple and avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the async pipe to handle observables
- Do not assume globals like (`new Date()`) are available.

## Models & Types

- Feature models: place in `features/[feature]/models/[feature].model.ts`
- Shared interfaces and types: place in `shared/models/`
- Use barrel exports (`index.ts`) in model directories to simplify imports
- Example:
  ```typescript
  // features/products/models/index.ts
  export * from './product.model';
  export * from './product-filter.model';
  ```

## Environment & Configuration

- Use `app.config.ts` for global Angular configurations (providers, interceptors at app level)
- Use feature-specific configs only when necessary
- Consider creating `src/app/core/config/` for environment-specific settings and constants
- Export environment configs as constants for type-safe access

## Services

- Design services around a single responsibility
- **Global Services**: Place in `core/services/` using `@Service` (or `@Injectable({ providedIn: 'root' })`)
- **Feature Services**: Place in `features/[feature]/services/[feature].service.ts`. Provide them in the feature's route definition (`[feature].routes.ts`) for proper route-level scoping and lifecycle cleanup
- Use the `inject()` function instead of constructor injection

Example feature routes with scoped service:

```typescript
// features/products/products.routes.ts
export const PRODUCTS_ROUTES: Routes = [
  {
    path: '',
    providers: [ProductService], // Scoped to this feature route subtree
    loadComponent: () =>
      import('./product-list/product-list.component').then((m) => m.ProductListComponent),
  },
];
```

## Testing Strategy

- **Unit tests**: Place `.spec.ts` files next to the files they test (`component.spec.ts`, `service.spec.ts`)
- **Feature testing**: Organize as `features/[feature]/[component].spec.ts`
- **Isolated Testing**: Test components and services in isolation using mock providers where needed
- **Coverage target**: Aim for high coverage on business logic and services
- Example test structure using `TestBed.runInInjectionContext`:

```typescript
describe('ProductService', () => {
  let service: ProductService;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [ProductService, provideHttpClientTesting()],
    });
    service = TestBed.inject(ProductService);
  });

  it('should load products', () => {
    expect(service).toBeTruthy();
  });
});
```
