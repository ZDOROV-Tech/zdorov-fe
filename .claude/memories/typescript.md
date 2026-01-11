# TypeScript Best Practices

## Language Service - CRITICAL REQUIREMENT

**ALWAYS use the lsp\_\* tools from the typescript-language-server MCP server when working with TypeScript files.** This is MANDATORY for all TypeScript code analysis, refactoring, and error checking.

### Required LSP Tool Usage:

**Before any code changes:**

- **ALWAYS** use `lsp_find_references` to find all usages before modifying methods/types/variables
- **ALWAYS** use `lsp_get_definitions` to understand symbol definitions
- **ALWAYS** use `lsp_get_hover` to get type information

**During refactoring:**/

- Use `lsp_get_code_actions` to discover available quick fixes and refactorings

**For validation:**

- **ALWAYS** run `lsp_get_diagnostics` after making changes to verify no new errors
- Use `lsp_get_completion` for accurate auto-completion suggestions

### Common Workflows:

- **Method changes**: `lsp_find_references` → modify → `lsp_get_diagnostics`
- **Type changes**: `lsp_find_references` → `lsp_get_hover` → modify → `lsp_get_diagnostics`
- **Refactoring**: `lsp_get_definitions` → `lsp_find_references` → modify → `lsp_get_diagnostics`

Don't ever use these MCP tools from the typescript-language-server MCP server:

- lsp_rename_symbol
- get_project_overview
- search_symbols
- get_symbol_details
- replace_range
- replace_regex
- list_memories
- read_memory
- write_memory
- delete_memory
- list_dir
- get_symbols_overview
- index_external_libraries
- get_typescript_dependencies
- search_external_library_symbols
- resolve_symbol
- get_available_external_symbols
- parse_imports
- index_onboarding
- get_symbol_search_guidance
- get_compression_guidance

## Type System

- Prefer interfaces over types for object definitions
- Use type for unions, intersections, and mapped types
- Avoid using `any`, prefer `unknown` for unknown types
- Use strict TypeScript configuration
- Leverage TypeScript's built-in utility types
- Use generics for reusable type patterns

## Naming Conventions

- Use PascalCase for type names and interfaces
- Use camelCase for variables and functions
- Use UPPER_CASE for constants
- Use descriptive names with auxiliary verbs (e.g., isLoading, hasError)

### Framework-Specific Naming

**React Components:**

- Prefix interfaces for React props with 'Props' (e.g., ButtonProps)
- Use PascalCase for React component names

**Angular Components:**

- Use descriptive interface names without generic suffixes (e.g., ButtonConfig, UserData)
- Angular component classes should end with 'Component' (e.g., UserProfileComponent)
- Angular service classes should end with 'Service' (e.g., UserDataService)
- Angular directive classes should end with 'Directive' (e.g., HighlightDirective)
- Angular pipe classes should end with 'Pipe' (e.g., CurrencyFormatPipe)
- Always use the DestroyRef + takeUntilDestroyed pattern instead of using onDestroy with a destroy subject.

## Code Organization

- Keep type definitions close to where they're used
- Export types and interfaces from dedicated type files when shared
- Place shared types in a `types` directory
- Co-locate component props with their components

## Functions

- Use explicit return types
- Implement proper error handling with custom error types
- Use function overloads for complex type scenarios
- Prefer async/await over Promises (except when working with RxJS observables)

## Best Practices

- Use readonly for immutable properties
- Leverage discriminated unions for type safety
- Use type guards for runtime type checking
- Implement proper null checking
- Avoid type assertions unless necessary

## Error Handling

- Create custom error types for domain-specific errors
- Use Result types for operations that can fail
- Implement proper error boundaries
- Use try-catch blocks with typed catch clauses
- Handle Promise rejections properly

## Patterns

- Use the Builder pattern for complex object creation
- Implement the Repository pattern for data access
- Use the Factory pattern for object creation
- Leverage dependency injection
- Use the Module pattern for encapsulation

## Performance

- Use in operator instead of Object.keys(obj).includes
