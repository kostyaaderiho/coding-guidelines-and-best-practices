# ReactJS

### Overview

However, not all coding standards and conventions can be enforced automatically. This highlights the need for the frontend community to have a single, authoritative source for coding guidelines.

This page aims to outline the rules and best practices that ESLint cannot enforce, which the community should consistently follow in their daily work.

### Component Structure and Organization

#### Use `PascalCase` for the component's name

Incorrect

```typescript
const lifeEventCard = () => { ... }
```

Correct

```jsx
const LifeEventCard = () => { ... }
```

#### Use PascalCase/camelCase for file names

Use `PascalCase` for `Component` files, and `camelCase` for other files. Do not use hyphens (`-`) for word separation.

Incorrect

```
/assets
    /icons
        Error.svg
/components
    /LifeEventCard
        lifeEventCard.tsx
        lifeEventCard.styled.tsx
        Utils.ts
        Types.ts
        index.ts // Re-exports for cleaner imports
/utils
    /heapEvents
        track-heap-event.ts
```

Correct

```typescript
/assets
    /icons
        error.svg
/components
    /LifeEventCard
        LifeEventCard.tsx
        LifeEventCard.styled.tsx
        utils.ts
        types.ts
        index.ts // Re-exports for cleaner imports
/utils
    /heapEvents
        trackHeapEvent.ts
```

#### Do not use React.FC

1. Children prop confusion: React.FC automatically adds the children prop, which may not always be desired.
2. Generics limitations: It can make typing generics and default props more complicated.
3. **No longer recommended**: The React team and TypeScript community generally recommend typing props directly, e.g., function MyComponent(props: MyProps), instead of using React.FC.

Incorrect

```jsx
export type MyComponentProps = {}

const MyComponent: React.FC<MyComponentProps> = (props): JSX.Element => { ... }
```

Correct

```jsx
export type MyComponentProps = {}

const MyComponent = (props: MyComponentProps): JSX.Element => { ... }
```

#### Use the correct Component's structure

Incorrect

```typescript
export const UserProfile = ({ userId, onUpdate }: UserProfileProps): JSX.Element => {
    // hooks first
    const [data, setData] = useState<User | null>(null);

    // effects after state
    useEffect(() => { ... }, [userId]);

    // handlers before render
    const handleSubmit = () => { ... };

    return <div>...</div>;
};
```

Correct

```typescript
export const UserProfile = ({ userId, onUpdate }: UserProfileProps): JSX.Element => {
    // hooks first
    const [data, setData] = useState<User | null>(null);

    // handlers before render
    const handleSubmit = () => { ... };

    // effects after state
    useEffect(() => { ... }, [userId]);

    return <div>...</div>;
};
```

#### Use `self-closing` tags when possible

Incorrect

```typescript
const Parent = (): JSX.Element => {
  return (<div>
      ...
      <Child></Child>
  </div>)
}
```

Correct

```typescript
const Parent = (): JSX.Element => {
  return (<div>
      ...
      <Child />
  </div>)
}
```

#### **Use the explicit returned type in Component**

By specifying `JSX.Element`, you ensure the function always returns a valid JSX element. If you accidentally return something else (like a string or number), TypeScript will show an error.

Incorrect

```jsx
const MyComponent = () => {
  return <div>Hello</div>;
};
```

Correct

```typescript
const MyComponent = (): JSX.Element => {
  return <div>Hello</div>;
};
```

### Props Definition

#### Use PascalCase + `Props` prefix as prop naming

Usage of `"export type"` is optional, depending on the component's usage.

Incorrect

```typescript
export type Props = { ... }

export const LifeEventCard = (props: Props): JSX.Element => { ... }
```

Correct

```typescript
export type LifeEventCardProps = { ... }

export const LifeEventCard = (props: LifeEventCardProps): JSX.Element => { ... }
```

#### Use destructuring in props

Incorrect

```
const Component = (props: ComponentProps) => { ... usage props['something'] }
```

Correct

```typescript
const Component = ({ prop1, prop2 }: ComponentProps) => { ... }
```

#### Use ES 'default props' over .defaultProps

Use destructuring with defaults in props.

Incorrect

```jsx
Component.defaultProps = { ... }
```

Correct

```jsx
const Component = ({
  title = 'Default Title',
  isActive = false
}: ComponentProps) => { ... }
```

### JSX and Rendering Patterns

#### Use shortened truthy props

Use short syntax for passing truthy values.

Incorrect

```jsx
<Button isActive={true} />
```

Correct

```jsx
<Button isActive />
```

#### Use passing a string as is

Pass string as is, `'{}'` is not required

Incorrect

```typescript
<Component id={'my-test-id'} />
```

Correct

```typescript
<Component id='my-test-id' />
```

#### Use the ternary operator instead of '&&'

`&&` can render unwanted falsy values (like 0 or '') in the DOM

Incorrect

```jsx
<>{flag && <div>text</div>}</>
```

Correct

```jsx
<>{flag ? <div>text</div> : null}</>
```

#### Use `null` for explicit empty return

Incorrect

```typescript
const Greeting = (): JSX.Element | null => {
  const [isEnabled, setIsEnabled] = useState(true);

  if (!isEnabled) return undefined;

  return <h1>Welcome back!</h1>;
}
```

Correct

```typescript
const Greeting = (): JSX.Element | null => {
  const [isEnabled, setIsEnabled] = useState(true);

  if (!isEnabled) return null;

  return <h1>Welcome back!</h1>;
}
```

#### Use stable key props in Lists

Use a stable, unique identifier

Incorrect

```jsx
{
  items.map((item, index) => <Card key={index} {...item} />);
}
```

Correct

```jsx
// Γ£à Using stable unique identifier
{
  items.map((item) => <Card key={item.id} {...item} />);
}

// Γ£à When no ID exists, use the combination
{
  items.map((item) => <Card key={`${item.name}-${item.date}`} {...item} />);
}
```

#### Use Composition over props drilling

Incorrect

```jsx
const Card = ({ headerTitle, headerDescription, children }: CardProps) => (
    <div>
        <header>
            <h1>{headerTitle}</h1>
            <p>{headerDescription}</p>
        </header>
        <div>{children}</div>
    </div>
);
```

Correct

```jsx
const Card = ({ header, children }: CardProps) => (
  <div>
    {header}
    <div>{children}</div>
  </div>
);
```

### Hooks and state management

#### Use function form for prev state updates

Rely on the previous state when the useState callback instead of taking the value from useState

Incorrect

```jsx
setCount(count + 1);
```

Correct

```jsx
// Γ£à Good - uses previous state
setCount((prevCount) => prevCount + 1);

// Γ£à Good for objects - spread previous state
setUser((prevUser) => ({ ...prevUser, name: "John" }));
```

#### Use `on` for prefix event handlers

Incorrect

```typescript
const Component = (): JSX.Element => {
    const handleChange  = (e: ChangeEvent<HTMLInputElement>) => { ... }
    return <Input onChange={handleChange} />
}
```

Correct

```typescript
const Component = (): JSX.Element => {
    const onChange  = (e: ChangeEvent<HTMLInputElement>) => { ... }
    return <Input onChange={onChange} />
}
```

#### Use 'memoization' only when necessary

Utilize 'memoization' only in expensively-rendered components or expensive calculations

Incorrect

Don't memoize everything ΓÇô it has overhead

Correct

```jsx
export const ExpensiveComponent = React.memo(({ data }: Props) => {
  // Expensive rendering logic
  return <div>{/* ... */}</div>;
});
```

```jsx
// Use useMemo for expensive calculations
const sortedList = useMemo(() => {
  return items.sort((a, b) => a.value - b.value);
}, [items]);
```
