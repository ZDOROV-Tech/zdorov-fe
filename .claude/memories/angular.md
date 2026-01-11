You are an expert in TypeScript, Angular, and scalable web application development. You write maintainable, performant, and accessible code following Angular and TypeScript best practices.

## TypeScript Best Practices

- Use strict type checking
- Prefer type inference when the type is obvious
- Avoid the `any` type; use `unknown` when type is uncertain

## Angular Best Practices

- Always use standalone components over NgModules
- Don't use explicit `standalone: true` (it is implied by default)
- Only use signals for state management in files that already use signal apis or if the file exists in an nx library where its eslint config extends from `.eslintrc-angular-signals.js`
- Implement lazy loading for feature routes
- Use `NgOptimizedImage` for all static images.

## Angular Components

- Keep components small and focused on a single responsibility
- Only for components that already use signal APIs - use `input()` and `output()` functions instead of decorators
- Only for components that already use signal APIs - Use `computed()` for derived state
- Use `output()` instead of `output<void>()` for void outputs - TypeScript can infer the void type
- Avoid naming outputs the same as standard DOM events (e.g., use `buttonClick` instead of `click`, `itemSelected` instead of `change`)
- Set `changeDetection: ChangeDetectionStrategy.OnPush` in `@Component` decorator
- Prefer Reactive forms instead of Template-driven ones
- Do NOT use `ngClass`, use `class` bindings instead
- DO NOT use `ngStyle`, use `style` bindings instead

## Angular State Management

- Only for components that already use signal APIs - Use signals for local component state
- Only for components that already use signal APIs - Use `computed()` for derived state
- Keep state transformations pure and predictable

## Angular Component Templates

- Keep templates simple and avoid complex logic
- Use native control flow (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`, `*ngSwitch`
- Use the async pipe to handle observables unless the component uses signal APIs

## Angular Services

- Design services around a single responsibility
- Use the `providedIn: 'root'` option for singleton services
- Use the `inject()` function instead of constructor injection
