# Unit Testing Best Practices

Comprehensive guidelines for writing maintainable, reliable, and effective unit tests in Angular and React applications.

## 🎯 Core Principles

### Test Behavior, Not Implementation

- **DO**: Test what the component/service does, not how it does it
- **DON'T**: Test private methods, internal state, or implementation details
- **FOCUS**: User interactions, API contracts, and business logic outcomes

### Keep Tests Simple and Focused

- **One concept per test**: Each test should verify a single behavior
- **Clear test names**: Describe what is being tested and expected outcome
- **Minimal setup**: Only include what's necessary for the specific test

### Write Tests That Enable Refactoring

- **Avoid brittle tests**: Don't couple tests to internal implementation
- **Test public interfaces**: Focus on inputs/outputs and observable behaviors
- **Enable safe refactoring**: Tests should pass after internal code changes

## 🅰️ Angular Testing Patterns

### Use Spectator Over TestBed When Possible

Spectator provides a cleaner, more intuitive API for most testing scenarios.

```typescript
// ✅ PREFERRED - Using Spectator for services
import { createServiceFactory, SpectatorService } from '@ngneat/spectator/jest';
import { MockProvider } from 'ng-mocks';

const createService = createServiceFactory({
  service: MyService,
  providers: [
    MockProvider(DependencyService, {
      someMethod: jest.fn().mockReturnValue('mock-value'),
    }),
  ],
});

let spectator: SpectatorService<MyService>;
let service: MyService;

beforeEach(() => {
  spectator = createService();
  service = spectator.service;
});

// ✅ PREFERRED - Using Spectator for components
import { createComponentFactory, Spectator } from '@ngneat/spectator/jest';

const createComponent = createComponentFactory({
  component: MyComponent,
  imports: [MockModule(SomeModule)],
  providers: [MockProvider(MyService)],
  detectChanges: false,
});

let spectator: Spectator<MyComponent>;

beforeEach(() => {
  spectator = createComponent();
});

// ❌ AVOID - TestBed when Spectator can be used
// Only use TestBed for complex integration scenarios
```

### Mocking Services with MockProvider

Always use `MockProvider` from `ng-mocks` for clean service mocking.

```typescript
// ✅ CORRECT - Using MockProvider
import { MockProvider } from 'ng-mocks';

const createService = createServiceFactory({
  service: MyService,
  providers: [
    MockProvider(HttpClient),
    MockProvider(UserService, {
      getCurrentUser: jest.fn().mockReturnValue({ id: '123' }),
      isAdmin: jest.fn().mockReturnValue(false),
    }),
  ],
});

// ❌ AVOID - Manual service mocking
providers: [
  {
    provide: UserService,
    useValue: {
      getCurrentUser: jest.fn(),
      // Missing methods can cause runtime errors
    },
  },
];
```

### Standalone Components Testing

For standalone components, use direct imports without modules.

```typescript
// ✅ CORRECT - Standalone component testing
const createComponent = createComponentFactory({
  component: MyStandaloneComponent,
  imports: [
    // Import other standalone components directly
    OtherStandaloneComponent,
    // Use MockModule for NgModules
    MockModule(LegacyModule),
  ],
});
```

### Signal-Based Component Testing

When testing components that use signals, focus on the observable behavior.

```typescript
// ✅ CORRECT - Testing signal-based components
it('should update display when signal changes', () => {
  const component = spectator.component;

  // Act - trigger signal change through public interface
  component.updateValue('new-value');
  spectator.detectChanges();

  // Assert - verify observable outcome
  expect(spectator.query('[data-testid="display-value"]')).toHaveText('new-value');
});
```

## ⚛️ React Testing Patterns

### Use Testing Library for Component Testing

Follow Testing Library principles: test like a user would interact.

```typescript
// ✅ CORRECT - Testing Library patterns
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

describe('MyComponent', () => {
  it('should update display when button is clicked', async () => {
    const user = userEvent.setup();
    render(<MyComponent />);

    const button = screen.getByRole('button', { name: /update/i });
    await user.click(button);

    expect(screen.getByText('Updated!')).toBeInTheDocument();
  });
});

// ❌ AVOID - Testing implementation details
// Don't access component instance or internal state
```

### Hook Testing with renderHook

Use `renderHook` from Testing Library for custom hook testing.

```typescript
// ✅ CORRECT - Hook testing
import { renderHook, act } from '@testing-library/react';

it('should toggle value when toggle is called', () => {
  const { result } = renderHook(() => useToggle(false));

  expect(result.current.value).toBe(false);

  act(() => {
    result.current.toggle();
  });

  expect(result.current.value).toBe(true);
});
```

### Async Testing Patterns

Handle promises and observables correctly in tests.

```typescript
// ✅ CORRECT - Promise-based async testing
import { firstValueFrom } from 'rxjs';

it('should emit value from observable', async () => {
  const valuePromise = firstValueFrom(service.getData());
  service.triggerData();

  const result = await valuePromise;
  expect(result).toBe('expected-value');
});

// ❌ AVOID - Using done callbacks
// This can lead to silent test failures
it('should emit value', (done) => {
  service.getData().subscribe((value) => {
    expect(value).toBe('expected');
    done(); // Problematic if assertion fails
  });
});
```

## 🎭 Mocking Best Practices

### Mock External Dependencies, Not Internal Logic

Only mock things your unit doesn't control.

```typescript
// ✅ CORRECT - Mock external dependencies
const mockHttpClient = {
  get: jest.fn().mockResolvedValue({ data: 'mock-data' }),
  post: jest.fn().mockResolvedValue({ success: true }),
};

// ✅ CORRECT - Mock side effects
const mockRouter = {
  navigate: jest.fn(),
  navigateByUrl: jest.fn(),
};

// ❌ AVOID - Mocking internal methods of the unit under test
// This creates brittle tests coupled to implementation
```

### Use Minimal Mock Implementations

Only mock what's needed for the specific test.

```typescript
// ✅ CORRECT - Minimal mocking
MockProvider(ComplexService, {
  // Only mock methods used in this test
  getUser: jest.fn().mockReturnValue({ id: '123' }),
  // Don't mock unused methods
});

// ❌ AVOID - Over-mocking
// Mocking every method creates maintenance overhead
```

### Reset Mocks Between Tests

Ensure test isolation by resetting mocks.

```typescript
// ✅ CORRECT - Mock cleanup
beforeEach(() => {
  jest.clearAllMocks();
  // Or for specific mocks:
  // mockService.someMethod.mockClear();
});
```

## 🚫 Avoid Spies Unless Necessary

Test spies should be used sparingly as they create fragile tests by coupling to implementation details rather than observable behavior.

### The Problem with Spies

**Spies couple tests to internal implementation details**, making tests brittle and prone to breaking when implementation changes even if behavior remains constant. This creates the **"Fragile Test" anti-pattern** where tests fail not because functionality is broken, but because internal code structure changed.

#### What Are Test Spies?

Test spies are functions that monitor calls to existing methods, allowing you to verify that specific methods were called, how many times, and with what arguments:

```typescript
// Example of a test spy
const spy = jest.spyOn(service, 'internalMethod');
service.publicMethod();
expect(spy).toHaveBeenCalledWith('expected-arg');
```

#### Why Spies Are Problematic

- **Implementation Coupling**: Tests become dependent on how code works internally, not what it produces
- **Refactoring Resistance**: Safe refactoring becomes impossible as tests break when moving code around
- **False Security**: Tests may pass while actual behavior is broken if you're only testing method calls
- **Maintenance Overhead**: Every internal change requires updating tests that spy on implementation

### When Spies Are Acceptable (Rare Cases)

#### 1. Integration Testing with Stable External APIs

Only when testing integration with third-party services that have stable, unlikely-to-change APIs:

```typescript
// ✅ ACCEPTABLE - Third-party API integration
it('should send analytics event to external service', () => {
  const analyticsSpy = jest.spyOn(thirdPartyAnalytics, 'track');

  component.performAction();

  expect(analyticsSpy).toHaveBeenCalledWith('user_action', {
    userId: '123',
    timestamp: expect.any(Number),
  });
});

// ✅ ACCEPTABLE - External router navigation
it('should navigate to correct route', () => {
  const navigateSpy = jest.spyOn(router, 'navigate');

  service.redirectToHome();

  expect(navigateSpy).toHaveBeenCalledWith(['/home']);
});
```

#### 2. Temporary Legacy System Testing

As a **temporary measure** when adding tests to large legacy systems before refactoring:

```typescript
// ⚠️ TEMPORARY - Legacy system under test
// Use spies to expose coupling points before refactoring
it('should call legacy helper during data processing', () => {
  const legacySpy = jest.spyOn(legacyService, 'processData');

  service.handleUserInput(testData);

  expect(legacySpy).toHaveBeenCalled();
  // This test will intentionally break during refactoring
  // helping identify what needs to be updated
});
```

### When to Avoid Spies (99% of Cases)

#### Don't Spy on Internal Methods

```typescript
// ❌ AVOID - Spying on internal implementation
it('should call internal helper method', () => {
  const helperSpy = jest.spyOn(component, 'validateInput');

  component.submitForm(formData);

  expect(helperSpy).toHaveBeenCalled(); // Fragile, implementation detail
});

// ✅ BETTER - Test the actual behavior
it('should show validation error for invalid input', () => {
  component.submitForm(invalidFormData);

  expect(component.errorMessage).toBe('Please enter a valid email');
  expect(component.isSubmitting).toBe(false);
});
```

#### Don't Spy Just to Verify Method Calls

```typescript
// ❌ AVOID - Testing method calls without behavior verification
it('should call save method', () => {
  const saveSpy = jest.spyOn(service, 'save');

  service.updateUser(userData);

  expect(saveSpy).toHaveBeenCalled(); // What if save() does nothing?
});

// ✅ BETTER - Test the actual outcome
it('should persist user changes', async () => {
  const updatedUser = await service.updateUser(userData);

  expect(updatedUser.name).toBe(userData.name);
  expect(updatedUser.lastModified).toBeDefined();
});
```

### Better Alternatives to Spies

**In 9 out of 10 cases, there's a less fragile way to express the test:**

1. **Test outputs and side effects** instead of method calls
2. **Mock external dependencies** rather than spying on internal methods
3. **Verify state changes** instead of implementation steps
4. **Test user-observable behavior** rather than internal mechanics

```typescript
// ❌ FRAGILE - Spying on implementation
it('should call validation and then save', () => {
  const validateSpy = jest.spyOn(service, 'validate');
  const saveSpy = jest.spyOn(service, 'save');

  service.processUser(userData);

  expect(validateSpy).toHaveBeenCalledBefore(saveSpy);
});

// ✅ ROBUST - Testing behavior and outcomes
it('should save user when data is valid', async () => {
  const result = await service.processUser(validUserData);

  expect(result.success).toBe(true);
  expect(result.user.id).toBeDefined();
});

it('should reject user when data is invalid', async () => {
  const result = await service.processUser(invalidUserData);

  expect(result.success).toBe(false);
  expect(result.errors).toContain('Invalid email format');
});
```

**Remember**: Good tests should enable refactoring by testing behavior, not implementation. If your test breaks when you rename a private method or rearrange internal logic without changing functionality, it's likely too coupled to implementation details.

## 🧪 Test Organization and Structure

### Follow Arrange-Act-Assert Pattern

Structure tests with clear phases.

```typescript
// ✅ CORRECT - AAA Pattern
it('should update user profile when valid data submitted', () => {
  // Arrange
  const validUser = { name: 'John Doe', email: 'john@example.com' };
  const mockResponse = { success: true };
  mockUserService.updateProfile.mockResolvedValue(mockResponse);

  // Act
  const result = service.updateUserProfile(validUser);

  // Assert
  expect(result).resolves.toEqual(mockResponse);
  expect(mockUserService.updateProfile).toHaveBeenCalledWith(validUser);
});
```

### Write Descriptive Test Names

Test names should clearly describe the scenario and expected outcome.

```typescript
// ✅ CORRECT - Descriptive test names
describe('UserService', () => {
  describe('updateProfile', () => {
    it('should update user profile when valid data provided', () => {});
    it('should throw validation error when email is invalid', () => {});
    it('should retry request once when network error occurs', () => {});
  });
});

// ❌ AVOID - Vague test names
it('should work correctly', () => {});
it('should test updateProfile', () => {});
```

### Group Related Tests with Describe Blocks

Use nested describe blocks to organize tests logically.

```typescript
// ✅ CORRECT - Logical grouping
describe('AuthService', () => {
  describe('login', () => {
    describe('when credentials are valid', () => {
      it('should return user data');
      it('should store auth token');
    });

    describe('when credentials are invalid', () => {
      it('should throw authentication error');
      it('should not store any tokens');
    });
  });
});
```

## 📊 Testing Async Code

### RxJS Observable Testing

Use `firstValueFrom` for converting Observables to Promises in tests.

```typescript
// ✅ CORRECT - Observable testing
import { firstValueFrom } from 'rxjs';

it('should emit filtered data', async () => {
  const data$ = service.getFilteredData();
  service.setFilter('active');

  const result = await firstValueFrom(data$);
  expect(result).toEqual([{ id: 1, status: 'active' }]);
});

// ✅ CORRECT - Testing multiple emissions
it('should emit multiple values', async () => {
  const emissions: any[] = [];
  const subscription = service.data$.subscribe((value) => emissions.push(value));

  service.addData('first');
  service.addData('second');

  subscription.unsubscribe();
  expect(emissions).toEqual(['first', 'second']);
});
```

### Promise-Based Testing

Always use async/await for cleaner promise testing.

```typescript
// ✅ CORRECT - Async/await pattern
it('should fetch user data', async () => {
  mockHttp.get.mockResolvedValue({ data: mockUser });

  const user = await service.getUserData('123');

  expect(user).toEqual(mockUser);
});

// ❌ AVOID - Promise chaining in tests
it('should fetch user data', () => {
  return service.getUserData('123').then((user) => {
    expect(user).toEqual(mockUser);
  });
});
```

## 🎨 Component Testing Specifics

### Test User Interactions, Not Implementation

Focus on how users interact with your components.

```typescript
// ✅ CORRECT - User-focused testing
it('should show error message when form is submitted with empty email', async () => {
  const user = userEvent.setup();
  render(<LoginForm />);

  const submitButton = screen.getByRole('button', { name: /login/i });
  await user.click(submitButton);

  expect(screen.getByText(/email is required/i)).toBeInTheDocument();
});

// ❌ AVOID - Implementation-focused testing
it('should set emailError state when validate is called with empty email', () => {
  const component = new LoginForm();
  component.validate({ email: '' });
  expect(component.emailError).toBe('Email is required');
});
```

### Test Conditional Rendering

Verify that components render correctly based on props/state.

```typescript
// ✅ CORRECT - Testing conditional rendering
it('should show loading spinner when data is loading', () => {
  render(<DataComponent loading={true} />);
  expect(screen.getByTestId('loading-spinner')).toBeInTheDocument();
});

it('should show data when loading is complete', () => {
  render(<DataComponent loading={false} data={mockData} />);
  expect(screen.getByText(mockData.title)).toBeInTheDocument();
  expect(screen.queryByTestId('loading-spinner')).not.toBeInTheDocument();
});
```

## 🚀 Performance and Maintenance

### Keep Tests Fast

- Minimize DOM operations
- Use shallow rendering when possible
- Mock heavy dependencies
- Avoid unnecessary async operations

### Make Tests Maintainable

- Extract common test utilities
- Use page object patterns for complex components
- Keep test data minimal and relevant
- Refactor tests when production code changes

### Test Coverage Goals

- Aim for high coverage of business logic
- Focus on critical paths and edge cases
- Don't chase 100% coverage at the expense of test quality
- Use coverage to identify untested scenarios, not as a goal itself

## 🛑 Common Anti-Patterns to Avoid

### Don't Test Framework Code

```typescript
// ❌ AVOID - Testing Angular/React framework behavior
it('should call ngOnInit', () => {
  expect(component.ngOnInit).toHaveBeenCalled();
});

// ✅ CORRECT - Test your code's behavior in lifecycle hooks
it('should load user data on component initialization', () => {
  expect(mockUserService.getCurrentUser).toHaveBeenCalled();
});
```

### Don't Test Third-Party Libraries

```typescript
// ❌ AVOID - Testing library behavior
it('should format date using moment.js', () => {
  expect(moment('2023-01-01').format('YYYY-MM-DD')).toBe('2023-01-01');
});

// ✅ CORRECT - Test your integration with the library
it('should display formatted date in user timezone', () => {
  const component = createComponent({ date: '2023-01-01' });
  expect(component.getFormattedDate()).toBe('Jan 1, 2023');
});
```

### Avoid Testing Multiple Scenarios in One Test

```typescript
// ❌ AVOID - Multiple scenarios in one test
it('should handle user authentication', () => {
  // Testing login
  service.login(validCredentials);
  expect(service.isAuthenticated()).toBe(true);

  // Testing logout
  service.logout();
  expect(service.isAuthenticated()).toBe(false);

  // Testing invalid credentials
  expect(() => service.login(invalidCredentials)).toThrow();
});

// ✅ CORRECT - Separate tests for each scenario
it('should authenticate user with valid credentials', () => {
  service.login(validCredentials);
  expect(service.isAuthenticated()).toBe(true);
});

it('should log out authenticated user', () => {
  service.login(validCredentials);
  service.logout();
  expect(service.isAuthenticated()).toBe(false);
});

it('should throw error for invalid credentials', () => {
  expect(() => service.login(invalidCredentials)).toThrow();
});
```

## 🔧 Development and Debugging

### Always Run Type Checking and Linting After Generating Tests

After generating or writing tests, always run type checking and linting to ensure all errors are resolved.

```bash
# ✅ REQUIRED - Always run these commands after generating tests
pnpm nx lint <project-name>
pnpm nx type-check-unit-tests
```

### Use Jest APIs, Not Jasmine

This workspace uses Jest, not Jasmine. Always use Jest functions and APIs.

```typescript
// ✅ CORRECT - Use Jest APIs
const mockFn = jest.fn();
const mockFnWithReturn = jest.fn().mockReturnValue('value');
const mockFnWithResolve = jest.fn().mockResolvedValue('async-value');
const spy = jest.spyOn(service, 'method');

// ❌ NEVER USE - Jasmine APIs (ESLint rule: jest/no-jasmine-globals)
const mockFn = jasmine.createSpy();
const spy = jasmine.createSpy('methodName');
jasmine.clock().install();
```

### Avoid Committing Focused Tests

Never commit focused test functions - they prevent other tests from running.

```typescript
// ❌ NEVER COMMIT - These prevent other tests from running
fdescribe('MyComponent', () => {});
fit('should work', () => {});
describe.only('MyService', () => {});
it.only('should pass', () => {});

// ✅ CORRECT - Use these for debugging locally, then remove
describe('MyComponent', () => {});
it('should work', () => {});
```

### Use the Global logHtml Utility for Debugging

This workspace provides a global `logHtml` utility for debugging component HTML.

```typescript
// ✅ CORRECT - Debug component state
it('should render correctly', () => {
  spectator = createComponent();

  // Use for debugging - remove before committing
  logHtml(spectator.element);

  expect(spectator.component).toBeTruthy();
});
```

### Test Isolation and Cleanup

Ensure tests don't affect each other through shared state.

```typescript
// ✅ CORRECT - Proper cleanup
beforeEach(() => {
  jest.clearAllMocks();
  // Reset any global state
  TestBed.resetTestingModule();
});

afterEach(() => {
  // Clean up subscriptions, timers, etc.
  if (component.subscription) {
    component.subscription.unsubscribe();
  }
});
```

## 📚 Project-Specific Testing Utilities

This workspace provides several testing utilities to make tests cleaner and more maintainable:

### RxLet Testing Module

For components using `@rx-angular/template/let`:

```typescript
// ✅ CORRECT - Using RxLetTestingModule
import { RxLetTestingModule } from '@cu/testing-common-mocks';

const createComponent = createComponentFactory({
  component: MyComponent,
  imports: [RxLetTestingModule],
});
```

### Mock Utilities Available

The workspace provides several mock utilities in `libs/testing/`:

- `createMockCuLocalStorage()` - For mocking local storage
- `createMockLocalStorage()` - For browser localStorage
- `provideMockErrorHandler()` - For error handling service
- `provideMockHttpClient()` - For HTTP client mocking

```typescript
// ✅ CORRECT - Using workspace mock utilities
import { createMockCuLocalStorage } from '@cu/testing-common-mocks';

const { cuLocalStorageMock } = createMockCuLocalStorage();
jest.mock('@cu-local-storage/cu-local-storage.const', () => ({
  cuLocalStorage: cuLocalStorageMock,
}));
```

## 💡 Testing Tips for This Workspace

### Handling Zone.js in Tests

The test setup automatically configures Zone.js. Don't manually import or configure it.

```typescript
// ❌ AVOID - Zone.js is already configured
import 'zone.js/testing';

// ✅ CORRECT - Just use async/await or testing utilities
it('should handle async operations', async () => {
  const result = await service.asyncMethod();
  expect(result).toBeDefined();
});
```

### Working with Nx Project Structure

When testing cross-library dependencies, use the proper import paths:

```typescript
// ✅ CORRECT - Use library aliases
import { SomeService } from '@cu/core-services';
import { UtilFunction } from '@cu/common-utils';

// ❌ AVOID - Relative paths to other libraries
import { SomeService } from '../../../core/services/src/lib/some.service';
```

### Testing with NgRx

Use the provided `provideMockStore` for state management testing:

```typescript
// ✅ CORRECT - NgRx testing
import { provideMockStore } from '@ngrx/store/testing';

const createService = createServiceFactory({
  service: MyService,
  providers: [
    provideMockStore({
      initialState: { user: { id: '123' } },
    }),
  ],
});
```

## 🔄 Test Generation Workflow

When generating or writing tests, follow this workflow to ensure quality:

1. **Generate/Write Tests**: Create your test files following the patterns above
2. **Run Tests**: Verify tests pass with `pnpm nx test <project-name>`
3. **Type Check**: Run `pnpm nx type-check-unit-tests` to catch TypeScript errors
4. **Lint**: Run `pnpm nx lint <project-name>` to catch style and best practice violations
5. **Fix Issues**: Resolve any type checking or linting errors before proceeding
6. **Review**: Ensure tests follow the best practices outlined in this guide

```bash
# ✅ COMPLETE TEST GENERATION WORKFLOW
pnpm nx test <project-name>        # Verify tests pass
pnpm nx type-check-unit-tests  # Check TypeScript
pnpm nx lint <project-name>        # Check linting rules
```

**CRITICAL**: Never consider test generation complete until all type checking and linting errors are resolved. These tools catch common mistakes and ensure consistency with the codebase standards.

Remember: Good tests serve as documentation, enable refactoring, catch regressions, and build confidence in your code. Write tests that you and your team will thank you for later!
