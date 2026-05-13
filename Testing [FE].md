# Testing

### Overview

The page highlights "behavior-driven" testing that simulates genuine user interactions instead of focusing on implementation details. The key principles are:

1. Reflect actual user usage of the application.
2. Detect real bugs rather than superficial changes.
3. Ensure tests are straightforward and maintainable.
4. Check for accessibility compliance.

### Testing Philosophy

#### Use rules for test coverage

1. Aim for 80%+ coverage for business logic.
2. 100% coverage for critical paths (auth, payments, data mutations).
3. Don't chase 100% coverage at the expense of test quality.

#### Do / Do NOT test

Do NOT test

1. Third-party library internals (React Router, React Query).
2. CSS/styling (unless it affects behavior).
3. Exact HTML structure (implementation detail).
4. Private/internal component states.

```typescript
// Example: BAD
test('should call useState with initial value', () => {
    const spy = jest.spyOn(React, 'useState');
    render(<Counter />);
    expect(spy).toHaveBeenCalledWith(0);
});
```

Do test

1. User-visible behavior.
2. Integration between your code.
3. Accessibility.
4. Error/Loading states.
5. Edge cases.

```typescript
// Example: GOOD
test('should display initial count of 0', () => {
    render(<Counter />);
    expect(screen.getByText('Count: 0')).toBeInTheDocument();
});
```

#### Do NOT use snapshot testing

Snapshot tests compare rendered output against stored snapshots. While convenient, they create more problems than they solve. We don't use snapshot testing because:

1. **Brittle Tests and False Positives:** Snapshot tests often fail due to minor, unrelated changes (like formatting or small UI tweaks), leading to frequent, unnecessary updates of snapshots rather than catching real bugs. Also, Developers may get used to simply updating snapshots when tests fail, potentially missing actual regressions or unintended changes in the application.
2. **Difficult to Review/Poor Documentation:** Snapshots, especially large ones, are hard to review and understand. It's challenging to spot meaningful differences in a large block of serialized output. Also, Snapshot tests don't clearly describe what behavior or output is expected, making it harder for others to understand the intent of the test.
3. **Better Alternatives Exist:** More targeted tests (such as assertions on specific output, user interactions, or accessibility) provide clearer, more maintainable, and more meaningful coverage.

Incorrect

```jsx
// Fails when CSS class names change, formatting updates, or any minor UI tweak
// Doesn't tell you WHAT broke or WHY it matters
test("should render user profile", () => {
  const { container } = render(<UserProfile user={mockUser} />);
  expect(container).toMatchSnapshot();
});
```

Correct

```jsx
// Tests actual behavior - will only fail if meaningful content changes
test("should display user name and email", () => {
  render(<UserProfile user={mockUser} />);
  expect(screen.getByRole("heading", { name: "John Doe" })).toBeInTheDocument();
  expect(screen.getByText("john.doe@example.com")).toBeInTheDocument();
});
```

### Test Types

#### Unit Tests

Unit tests verify isolated pieces of code (functions, utilities, single components) in complete isolation from external dependencies.

**Characteristics**

1. Fast execution (no network, no heavy rendering).
2. No external dependencies.
3. Test one thing at a time.
4. Easy to debug when they fail.

```typescript
describe("formatCurrency", () => {
  test("should format USD correctly", () => {
    expect(formatCurrency(1234.56, "USD")).toBe("$1,234.56");
  });
});
```

#### Integration Tests

Integration tests verify that multiple units work together correctly. They test the flow between components, API calls, state management, and user interactions.

**Characteristics**

1. Test realistic user scenarios.
2. Include multiple components.
3. May include real API calls (with MSW).
4. Test the integration points between units.

```typescript
describe('UserProfile integration', () => {
    test('should load and display user data', async () => {
        // Don't mock the data layer, test the full flow
        render(<UserProfile userId="123" />);
        expect(await screen.findByText('John Doe')).toBeInTheDocument();
    });
});
```

#### E2E Tests

Typically, we do not write E2E tests on the frontend ourselves, since we usually have a dedicated ATA engineer per team responsible for creating E2E tests and covering the most critical scenarios. **It's up to the team to decide** if E2E tests should be applied by FE developers.

#### Accessibility Tests

Accessibility tests verify that your application is usable by people with disabilities. We can write effective a11y tests using `Testing Library's` built-in queries and DOM assertions.

We don't use `jest-axe` for accessibility verifications, and strictly rely on the existing API provided by Testing Library.

**What to Test:**

1. **Semantic HTML** - Using proper roles and landmarks
2. **ARIA attributes** - aria-label, aria-describedby, aria-invalid, aria-live
3. **Keyboard navigation** - Tab order, arrow keys, Enter/Escape
4. **Focus management** - Focus trapping, focus restoration
5. **Screen reader announcements** - Live regions, status updates
6. **Form accessibility** - Labels, error associations, required fields

```typescript
describe('LoginForm accessibility', () => {
    // Case 1. Keyboard navigation
    test('should be keyboard navigable', async () => {
        const user = userEvent.setup();
        render(<LoginForm />);

        await user.tab();
        expect(screen.getByLabelText(/username/i)).toHaveFocus();

        await user.tab();
        expect(screen.getByLabelText(/password/i)).toHaveFocus();
    });

    // Case 2. Screen reader announcements
    test('should announce form errors to screen readers', async () => {
        render(<LoginForm />);
        const submitBtn = screen.getByRole('button', { name: /submit/i });

        await userEvent.click(submitBtn);

        expect(screen.getByRole('alert')).toHaveTextContent(/username is required/i);
    });
});
```

### Conventions

#### Use Arrange-Act-Assert (AAA) Pattern

Structure tests clearly:

- `Arrange`: Setup
- `Act`: User interaction
- `Assert`: Expectations

Incorrect

```jsx
// Mixed arrange/act/assert - hard to follow
test("should filter products", async () => {
  render(<ProductList />);
  await userEvent.click(screen.getByRole("button", { name: /filter/i }));
  const checkbox = screen.getByLabelText(/in stock only/i);
  await userEvent.click(checkbox);
  expect(screen.getAllByRole("listitem")).toHaveLength(5);
  await userEvent.selectOptions(
    screen.getByLabelText(/category/i),
    "electronics",
  );
  expect(screen.getAllByRole("listitem")).toHaveLength(2);
});
```

Correct

```jsx
// Clear structure - easy to understand test flow
test("should filter products by category and stock status", async () => {
  // Arrange
  render(<ProductList />);

  // Act
  await userEvent.click(screen.getByRole("button", { name: /filter/i }));
  await userEvent.click(screen.getByLabelText(/in stock only/i));
  await userEvent.selectOptions(
    screen.getByLabelText(/category/i),
    "electronics",
  );

  // Assert
  expect(screen.getAllByRole("listitem")).toHaveLength(2);
  expect(screen.queryByText("Out of Stock Product")).not.toBeInTheDocument();
});
```

#### Use describe () blocks for test grouping

Use `describe` blocks to organize tests by rendering, interactions, edge cases, or accessibility. This makes your test files easier to navigate and understand. Keep nesting to 2ΓÇô3 levels to maintain readability and avoid complexity.

```typescript
describe("ComponentName", () => {
  describe("Rendering", () => {
    test("...");
  });

  describe("User Interactions", () => {
    test("...");
  });

  describe("Edge Cases", () => {
    test("...");
  });

  describe("Accessibility", () => {
    test("...");
  });

  describe("Error Handling", () => {
    test("...");
  });
});
```

#### Use `.spec` prefix for test file

Common convention across JavaScript/TypeScript ecosystems.

Incorrect

```
/Button
  Button.test.tsx
```

Correct

```
/Button
  Button.spec.tsx
```

#### Do NOT use `__tests__` folder

Place test files next to the code they test. This improves discoverability and makes refactoring easier.

We don't use **tests** because:

1. **Extra Boilerplate:** Adds additional directories, which can feel unnecessary for small projects or simple components.
2. **Inline/Adjacent Tests:** Placing test files next to the code they test (e.g., Button.tsx and Button.test.tsx in the same folder) can improve discoverability and make refactoring easier.
3. **Flexibility:** Modern test runners can find tests anywhere, so you're not required to use a **tests** folder.

Incorrect

```tsx
/src
  /components
    /Button
      /__tests__
        Button.tsx
        Button.spec.tsx
  /utils
      /__tests__
      formatDate.spec.ts
    formatDate.ts
```

Correct

```tsx
/src
  /components
    /Button
        Button.tsx
        Button.spec.tsx
  /utils
    formatDate.ts
    formatDate.spec.ts
```

#### Use `test('...')`

Incorrect

```tsx
describe("LoginForm", () => {
  it("should display error on invalid credentials", async () => {
    // test code
  });
});
```

Correct

```tsx
describe("LoginForm", () => {
  test("should display error on invalid credentials", async () => {
    // test code
  });
});
```

#### Use 'should' (lowercased) as the starting word for the test

Pattern: `test('should [expected behavior] when [condition]', ...)`

Incorrect

```tsx
test("displays error when form is invalid", ...)
```

Correct

```tsx
test("should display error when form is invalid", ...)
```

#### Use parametrized tests with test.each

Use `test.each<T>` for testing multiple scenarios with the same logic

Incorrect

```typescript
function isButtonEnabled(username, password) {
  return username.length > 0 && password.length >= 8;
}

test('isButtonEnabled("user", "password123") should return true', () => {
  expect(isButtonEnabled("user", "password123")).toBe(true);
});

test('isButtonEnabled("", "password123") should return false', () => {
  expect(isButtonEnabled("", "password123")).toBe(false);
});
```

Correct

Explicitly pass the type as generic for the `.each<T>` block for better type-checking.

```typescript
const isButtonEnabled = (username, password) => {
  return username.length > 0 && password.length >= 8;
};

test.each<[string, string, boolean]>([
  ["user", "password123", true],
  ["", "password123", false],
])(
  'isButtonEnabled("%s", "%s") should return %s',
  (username, password, expected) => {
    expect(isButtonEnabled(username, password)).toBe(expected);
  },
);
```

#### Use the correct setup order

Follow the correct order for `import` and `jest.mock`, as an example:

Correct

```typescript
import { screen, waitFor } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

import { renderWithProviders } from "~/utils/testUtils";

// Import component under test
import { ComponentName } from "./ComponentName";

// Step 1. Mock third-party libraries when needed (absolute path)
jest.mock("react-router-dom", () => ({
  ...jest.requireActual("react-router-dom"),
  useNavigate: jest.fn(),
}));

// Step 2. Mock analytics/tracking utilities (~ path)
jest.mock("~/utils", () => ({
  ...jest.requireActual("~/utils"),
  trackHeapEvent: jest.fn(),
}));
jest.mock("~/components/Session/Session.context", () => ({
  useSession: jest.fn(),
}));

// Step 3. Mock '..' relative path if required.

describe("ComponentName", () => {
  // Test suites go here
});
```

### Mocking

#### Use mocks/Do NOT mock

Do NOT mock

1. Internal utilities (unless truly expensive).
2. React hooks (unless third-party).
3. Simple helper functions.
4. Constants.

Do mock

1. External API calls (fetch, axios)
2. Third-party services (analytics, feature flags)
3. Browser APIs (localStorage, window.location)
4. Expensive computations
5. Time-dependent code (Date.now(), setTimeout)

#### Use explicit type definitions for mocks

This allows catching errors with missing/incorrect field types.

Incorrect

```typescript
export const personJobProfile = {
  positionDetails: {
    title: "Senior Product Manager",
    employmentCategory: null,
  },
  managerId: 111111,
  managerPath: "/111111/222222",
  hireDate: 1047340800000,
  refNo: "100",
  isManager: true,
  isRetired: false,
};
```

Correct

```typescript
export const personJobProfile: ClientServiceV3Types["JobProfile"] = {
  positionDetails: {
    title: "Senior Product Manager",
    employmentCategory: null,
  },
  managerId: 111111,
  managerPath: "/111111/222222",
  hireDate: 1047340800000,
  refNo: "100",
  isManager: true,
  isRetired: false,
};
```

#### Use MSW for mocking API

Using MSW is a preferable approach for mocking API responses.

```typescript
// tests/mocks/handlers.ts
import { rest } from 'msw';

export const handlers = [
    rest.get('/api/users/:id', (req, res, ctx) => {
        return res(
            ctx.status(200),
            ctx.json({ id: req.params.id, name: 'John Doe' })
        );
    }),

    rest.post('/api/users', async (req, res, ctx) => {
        const body = await req.json();
        return res(ctx.status(201), ctx.json({ id: '123', ...body }));
    })
];

// tests/mocks/server.ts
import { setupServer } from 'msw/node';
import { handlers } from './handlers';

export const server = setupServer(...handlers);

// jest.setup.js
beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

// In tests
test('should fetch user data', async () => {
    render(<UserProfile userId="123" />);
    expect(await screen.findByText('John Doe')).toBeInTheDocument();
});
```

### Testing Library Best Practices

#### Use tests for testing behavior, not implementation

Test what the user sees and does, not internal state or methods.

Incorrect

```jsx
expect(componentInstance.state.isLoggedIn).toBe(true);
```

Correct

```jsx
expect(screen.getByText("Welcome!")).toBeInTheDocument();
```

#### Use accessible queries

Prefer queries like `getByRole`, `getByLabelText`, and `getByText` over `getByTestId`.

Follow the officially recommended Testing Library **query priority list** https://testing-library.com/docs/queries/about/#priority, if you need to choose the getBy\* query

Incorrect

```typescript
const button = screen.getByTestId("submit-btn");
```

Correct

```typescript
const button = screen.getByRole("button", { name: /submit/i });
```

#### Use async utilities for async code

Use `findBy*(built-in waitFor)` queries and await for elements that appear asynchronously.

Incorrect

```jsx
test("should load user data", () => {
  render(<UserDashboard userId="123" />);

  // Race condition - test might pass/fail randomly
  expect(screen.getByText("Welcome, John")).toBeInTheDocument();
});
```

Correct

```jsx
test("should load user data", async () => {
  render(<UserDashboard userId="123" />);

  // Wait for async data to load
  expect(await screen.findByText("Welcome, John")).toBeInTheDocument();
  expect(screen.queryByText(/loading/i)).not.toBeInTheDocument();
});
```

#### Use @testing-library/user-event over fireEvent

While `fireEvent` triggers DOM events directly, `user-event` mimics how users actually interact with your application (e.g., typing, clicking, tabbing), including event order and timing. This leads to tests that better reflect real-world usage and help catch issues that might not appear with direct event firing

Incorrect

```jsx
const { getByLabelText } = render(<MyComponent />);
const input = getByLabelText(/username/i);

fireEvent.change(input, { target: { value: "John" } });

expect(input.value).toBe("John");
```

Correct

```jsx
// 1. Clicking disabled elements (userEvent will throw, fireEvent won't)
test("should prevent clicking disabled button", async () => {
  render(<SubmitButton disabled />);
  const button = screen.getByRole("button");

  // userEvent will throw an error (good - catches bugs)
  await expect(userEvent.click(button)).rejects.toThrow();

  // fireEvent would silently "succeed" (bad - hides bugs)
  // fireEvent.click(button); // This doesn't fail!
});

// 2. Keyboard interactions
test("should navigate with keyboard", async () => {
  render(<Dropdown options={["A", "B", "C"]} />);

  await userEvent.tab(); // Focus first element
  await userEvent.keyboard("{ArrowDown}"); // Navigate

  // fireEvent can't do this naturally
});

// 3. Complex interactions
test("should handle file upload", async () => {
  render(<FileUploader />);
  const file = new File(["content"], "test.pdf", { type: "application/pdf" });
  const input = screen.getByLabelText(/upload/i);

  await userEvent.upload(input, file);

  expect(screen.getByText("test.pdf")).toBeInTheDocument();
});
```

fireEvent is acceptable in a few cases:

1. **Non-user events**: Events that don't come from user interaction

   ```jsx
   // Window resize, scroll events
   fireEvent.resize(window, { innerWidth: 500 });
   ```

2. Testing event handlers directly: When testing low-level component behavior

   ```jsx
   // Testing that onMouseEnter is wired correctly
   fireEvent.mouseEnter(element);
   ```

3. Performance: For very large test suites where userEvent async overhead matters.

#### Use setup() for multiple interactions

When performing multiple user actions in sequence, create a user instance:

Slower

```
await userEvent.click(button1);
await userEvent.click(button2);
await userEvent.type(input, 'text');
```

Faster

```jsx
const user = userEvent.setup();
await user.click(button1);
await user.click(button2);
await user.type(input, "text");
```

### Useful Links

#### Testing Libraries

- React Testing Library: https://testing-library.com/react
- user-event: https://testing-library.com/docs/user-event/intro
- MSW v2: https://mswjs.io/
- Jest: https://jestjs.io/

#### Best Practices

- Common Testing Mistakes: https://kentcdodds.com/blog/common-mistakes-with-react-testing-library
- Testing Implementation Details: https://kentcdodds.com/blog/testing-implementation-details
- Write Tests Like a User: https://kentcdodds.com/blog/write-tests

#### Accessibility Testing

- WCAG 2.2: https://www.w3.org/TR/WCAG22/
- ARIA in HTML: https://www.w3.org/TR/html-aria/
- ARIA Authoring Practices Guide: https://www.w3.org/WAI/ARIA/apg/
- Testing Library Accessibility: https://testing-library.com/docs/queries/about/#priority
