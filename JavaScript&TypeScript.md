# JavaScript/TypeScript

### Overview

However, not all coding standards and conventions can be enforced automatically. This highlights the need for the frontend community to have a single, authoritative source for coding guidelines.

This page aims to outline the rules and best practices that ESLint cannot enforce, which the community should consistently follow in their daily work.

### Naming Conventions

#### Use `camelCase` for functions and their params

Function names must start with a lowercase letter and use `camelCase` where the first letter of each subsequent word is capitalized.

All function parameters must use the `camelCase` naming convention. This applies to regular parameters, optional parameters, default parameters, rest parameters, and destructured object properties.

Incorrect

```typescript
const TrackHeapEvent = (EventName): void => { ... }
```

Correct

```typescript
const trackHeapEvent = (eventName): void => { ... }
```

#### Use `UPPER_SNAKE_CASE` for a constant

By constant, we mean like a config value. Use for any constant like `primitive`, `object,` or `arrays`

Incorrect

```typescript
export const apiUrl = "...";

export const cacheStaleTime = {
  shortTerm: 10 * 1000,
  midTerm: 25 * 1000,
  longTerm: 50 * 1000,
  infinity: Infinity,
};
```

Correct

```typescript
export const API_URL = "...";

export const CACHE_STALE_TIME = {
  shortTerm: 10 * 1000,
  midTerm: 25 * 1000,
  longTerm: 50 * 1000,
  infinity: Infinity,
};
```

#### Use `camelCase` for a constant's property

Incorrect

```typescript
export const CACHE_STALE_TIME = {
  ShortTerm: 10 * 1000,
  MidTerm: 25 * 1000,
  longterm: 50 * 1000,
  infinity: Infinity,
};
```

Correct

```typescript
export const CACHE_STALE_TIME = {
  shortTerm: 10 * 60 * 1000,
  midTerm: 30 * 60 * 1000,
  longTerm: 60 * 60 * 1000,
  infinity: Infinity,
};
```

#### Use prefix with `is`, `has`, `can`, or `should` for booleans

Guideline for naming booleans:

- `is` - state/property (`isActive, isVisible`)
- `has` - possession (`hasPermission, hasChildren`)
- `can` - capability (`canEdit, canDelete`)
- `should` - conditional behavior (`shouldRender, shouldValidate`)

Incorrect

```typescript
let active = true;
let doesUserhasPermission = false;
```

Correct

```typescript
let isActive = true;
let hasPermission = false;
```

#### Do NOT add 'I'

Avoid the `I` prefix for types and interfaces in TypeScript to keep your code clean, modern, and consistent with community standards and official recommendations.

Incorrect

```typescript
type IUser = {
  id: number;
  name: string;
};
```

Correct

```typescript
type User = {
  id: number;
  name: string;
};
```

### Modern Javascript Syntax

#### Use nullish coalescing over ||

Use nullish coalescing for default values (NOT || operator)

Incorrect

```typescript
const count = data.count || 0; // fails for count = 0
```

Correct

```typescript
const count = data.count ?? 0;
```

#### Use string templates

Always use template literals over concatenation.

Incorrect

```typescript
const url = API_URL + "/users/" + userId;
```

Correct

```typescript
const url = `${API_URL}/users/${userId}`;
```

#### Use the arrow function as preferable

For regular functions, prefer arrow functions for consistency.

**Exception**: when you need 'this' binding or hoisting

Incorrect

```typescript
// Incorrect (mixed styles)
function formatUser(user) { ... }
const validateEmail = (email) => { ... };
```

Correct

```typescript
// Correct (consistent arrow functions)
const formatUser = (user: User): FormattedUser => { ... };
const validateEmail = (email: string): boolean => { ... };
```

#### Do NOT use nested ternary operators

Nested operators can be replaced with `if-else` statements or lookup structures for better readability.

Simple single ternaries are acceptable and idiomatic

Incorrect

```typescript
// Nested ternary - hard to read and maintain
const getStatusColor = (status) =>
  status === "active"
    ? "green"
    : status === "pending"
      ? "yellow"
      : status === "error"
        ? "red"
        : "gray";
```

Correct

```typescript
// Option 1: use if-else statements with return
const getStatusColor = (status) => {
  if (status === "active") return "green";
  if (status === "pending") return "yellow";
  if (status === "error") return "red";
  return "gray";
};

// Options 2: Object lookup
const STATUS_COLORS = {
  active: "green",
  pending: "yellow",
  error: "red",
} as const;

const getStatusColor = (status: string) => STATUS_COLORS[status] ?? "gray";
```

### Asynchronous code

#### Use async/await over regular .then

We should utilize `async/await` over the regular `.then`

Incorrect

```typescript
fetchData()
  .then((data) => processData(data))
  .then((result) => updateUI(result));
```

Correct

```typescript
const data = await fetchData();
const result = await processData(data);
updateUI(result);
```

#### Use try/catch for async/await if applicable

Consider handling errors in `async/await` functions with `try/catch` blocks with meaningful messages if the error shouldn't be propagated.

Incorrect

```typescript
const data = await fetchData();
return data;
```

Correct

```typescript
try {
  const data = await fetchData();
  return data;
} catch (error) {
  console.error("Failed to fetch data:", error);
  throw error; // or handle appropriately
}
```

### Code Quality & Documentation

#### [OPTIONAL] Use JSDoc for documenting functions with complex logic

It's accepted to avoid the usage of JSDoc for simple/self-documented code, and we have TS in place.

But it is beneficial when we deal with public APIs, complex business logic, and non-obvious behaviors.

`@param` and `@returns` must be specified.

Incorrect

```javascript
const calculateTotal = (price, tax) => { ... }
```

Correct

```javascript
/**
 * Calculates the total price with tax.
 * @param price - The base price
 * @param tax - The tax rate
 * @returns The total price including tax
 */
const calculateTotal = (price, tax) => { ... }
```

#### Use comments only if required

Use comments only if in several cases, but avoid overcommenting as it brings more complexity

1. **Clarifying Complex Logic:** When the code implements complex algorithms or logic that may not be immediately clear to other developers, comments help explain the reasoning and steps involved.
2. **Documenting Workarounds or Hacks:** If the code includes temporary solutions, workarounds, or hacks due to limitations or bugs, comments should explain why these choices were made.
3. **Explaining Non-Obvious Decisions:** When design decisions or code choices are not obvious, comments provide context about why a particular approach was taken.
4. **Indicating TODOs or Fixes Needed:** Comments can mark areas where improvements, bug fixes, or additional features are required. ΓÇô `// TODO: [TICKET-ID] Description`

### TypeScript Type System

#### Do NOT use `any`

You should use `any` only in exceptional cases (see them below).

Why NOT `any`

1. **Bypasses Type Safety:** `any` disables all type checking for that variable, defeating the purpose of TypeScript.
2. **Hides Errors:** Mistakes (like typos or wrong property access) won't be caught at compile time.
3. **Reduces Code Quality:** Overuse of any leads to code that is hard to maintain and refactor.
4. **No IntelliSense:** TypeScript's editor support and autocompletion are lost with any.
5. **Increases Runtime Errors:** Since anything is allowed, you're more likely to encounter bugs at runtime

Why `unknown` is preferred

1. **Type Safety Enforcement:** `unknown` requires you to perform type checks or type assertions before using the value, preventing accidental misuse and runtime errors.
2. **Prevents Unsafe Operations:** You cannot access properties or call methods on `unknown` types without narrowing their type.
3. **Encourages Explicit Type Handling:** using `unknown` forces you handle all possible types, leading to more robust code.

Incorrect

```typescript
const data: any = JSON.parse(jsonString);
data.user.profile.email; // Could crash if structure is wrong
```

Correct

```typescript
const isApiResponse = (data: unknown): data is ApiResponse => {
  return (
    typeof data === "object" &&
    data !== null &&
    "user" in data &&
    typeof (data as any).user === "object"
  );
};

const data: unknown = JSON.parse(jsonString);

if (isApiResponse(data)) {
  console.log(data.user.profile.email); // Safe!
}
```

Correct ΓÇô **any** is acceptable when:

1. **Gradual Migration** ΓÇô When migrating a JavaScript codebase to TypeScript, any can be used temporarily to allow incremental adoption of types.

```typescript
// @ts-expect-error - TODO: Type this properly (OR-X)
const legacyFunction = (data: any) => {
  // Temporarily using any during migration
  return data.process();
};
```

2. **Prototyping or Quick Proof-of-Concept** ΓÇô During rapid prototyping, you might use `any` to move quickly, with the intention to add proper types later.

```typescript
// Quick prototyping (with clear TODO)
// TODO: Replace any with proper type once API contract is finalized in (OR-X])
const prototypeHandler = (payload: any) => {
  console.log(payload);
};
```

3. **Legacy Code** ΓÇô In legacy codebases where adding strict types everywhere is not feasible, `any` can be used as a stopgap.

```typescript
// 3. Third-party library without types (rare nowadays)
import oldLibrary from "untyped-legacy-library";
const result: any = oldLibrary.someMethod();
```

#### Use `type` alias as the base

You should use `type` alias as a base; `interface` is used only in edge cases.

1. **Union and Intersection Types:** `type` can express unions (A | B) and intersections (A & B), but interfaces cannot.
2. **Primitive, Tuple, and Function Types:** `type` can alias primitives, tuples, and functions, not just object shapes.
3. **Mapped Types and Conditional Types:** `type` supports advanced features like mapped and conditional types.

Correct ΓÇô usage **interface** is accepted when:

1. **Declaration Merging** ΓÇô multiple `interface` declarations with the same name are automatically merged, which is useful for extending types from libraries or in large codebases.

```typescript
interface Window {
  customProp: string;
}
interface Window {
  anotherProp: number;
}
// -> Window now has both properties
```

2. **Extending and Implementing in OOP** ΓÇô `interface` is designed for object-oriented patterns: classes can implement interfaces and interfaces can extend each other.

```typescript
interface Animal {
  speak(): void;
}
class Dog implements Animal {
  speak() {
    /* ... */
  }
}
```

#### Use `T[]`

Prefer `T[]` for its readability, brevity, and alignment with JavaScript conventions.

Incorrect

```typescript
function sum(numbers: Array<number>): number { ... }
```

Correct

```typescript
function sum(numbers: number[]): number { ... }
```

#### Use annotations for `parameters` and `return` types

Always annotating function return types and parameters in TypeScript improves code safety, readability, maintainability, and helps prevent subtle bugs. It's especially important in shared, public, or large-scale codebases.

Incorect

```typescript
// Missing return type - TypeScript infers it but intention is unclear
const generateAPIUrl = (url: string) => { ... }

// Wrong return type annotation
const fetchUser = async (id: number): User => {
    return await fetch(`/api/users/${id}`).then(r => r.json());
    // Returns Promise<User> but annotated as User
}
```

Correct

```typescript
const generateAPIUrl = (url: string): string => {
  return `${API_BASE_URL}${url}`;
};

const fetchUser = async (id: number): Promise<User> => {
  return await fetch(`/api/users/${id}`).then((r) => r.json());
};

// Use void when nothing is returned
const logMessage = (message: string): void => {
  console.log(message);
};
```

#### Use `'as'` **only for specific reasons**

`'as'` disables TypeScript's type checking and can introduce runtime errors if used incorrectly. Always prefer type guards or runtime checks before using `'as'`

Incorrect

```typescript
// Unsafe assertion without validation
const data = await fetchData();
const user = data as User; // Could be wrong!
user.email.toLowerCase(); // Runtime error if email is undefined
```

Correct

```typescript
// Use type guard instead
const data = await fetchData();

if (isUser(data)) {
  data.email.toLowerCase(); // Safe!
}
```

#### Use shortened syntax for generic type parameters

We follow `T, K, V, etc.` naming conventions.

Incorrect

```typescript
// Overly verbose generic names
type ResponseData<ResponseDataType, ErrorType, MetadataType> = {
    data: ResponseDataType;
    error: ErrorType;
    meta: MetadataType;
}

// Unclear single letters
function process<X, Y, Z>(input: X, config: Y): Z { ... }
```

Correct

```typescript
// Conventional names: T (Type), K (Key), V (Value), E (Element)
function mapObject<K extends string, V, R>(
    obj: Record<K, V>,
    mapper: (value: V) => R
): Record<K, R> { ... }
```

#### Use type guards for unknown data

Create type guards for runtime checks if data is unknown.

Correct

```typescript
const isUser = (value: unknown): value is User => {
  return typeof value === "object" && value !== null && "id" in value;
};

if (isUser(user)) {
  console.log(user.id); // Safe!
}
```
