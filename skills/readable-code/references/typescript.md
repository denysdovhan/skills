# TypeScript / JavaScript Rules

Language-specific rules for TS, JS, JSX, and Node. Apply them on top of the
general rules in [SKILL.md](../SKILL.md). Rules numbered from 15 to continue the
general list.

## 15. Prefer `const`, Then `let`, Never `var`

- Default to `const`. A reader can then assume the binding never changes.
- Use `let` only when the variable is reassigned. If you need `let` just to set
  a value in an `if/else`, extract a function that returns it instead.
- `var` is function-scoped and hoisted, which surprises readers. Don't use it.

## 16. Keep Array Chains Short and Named

- Use `map`, `filter`, `find`, `some`, `every` for **simple transforms and
  filters**.
- If a chain needs more than two or three steps, or a callback spans multiple
  lines, extract named helpers or use a `for...of` loop.
- Avoid `reduce` unless it is a true fold (sum, count, merge). Building objects
  or arrays with `reduce` is usually clearer as a loop, `Object.fromEntries`,
  or `Map.groupBy` / `Object.groupBy` where available.
- Use `for...of` over arrays, never `for...in` (it iterates keys, including
  inherited ones).

```ts
// Good: simple and readable
const names = users.filter((user) => user.isActive).map((user) => user.name);

// Bad: too much going on
const result = groups
  .flatMap((g) => g.items)
  .filter((i) => i.valid && i.type === "A")
  .reduce((acc, i) => ({ ...acc, [i.id]: transform(i) }), {});

// Good: break it up
const validItems = getValidItems(groups, "A");
const entries = validItems.map((item) => [item.id, transform(item)]);
const result = Object.fromEntries(entries);
```

## 17. Use Destructuring — Shallowly

- Destructure objects and tuples to make the data shape explicit:
  `const { id, email } = user` instead of repeating `user.id`, `user.email`.
- Destructure options objects in the parameter list so the accepted options
  are visible at the signature.
- Don't destructure more than one level deep in one statement. Nested patterns
  with defaults and renames are hard to read.

## 18. Use `?.` and `??` for Missing Values

- Use optional chaining (`user?.address?.city`) instead of `&&` chains.
- Use `??` for defaults instead of `||`. `||` also replaces `0`, `""`, and
  `false`, which is usually a bug.
- If you need `?.` many times on the same value, narrow it once with a guard
  clause instead.

## 19. Prefer `async`/`await` Over `.then` Chains

- `await` reads top to bottom like synchronous code. Long `.then` chains hide
  the order of operations.
- Run independent work in parallel with `Promise.all`, and name what each
  promise produces.
- Handle errors where you can do something useful. Don't wrap every `await` in
  its own `try/catch`.

```ts
// Bad: sequential when it doesn't need to be, nested callbacks
fetchUser(id).then((user) => fetchOrders(id).then((orders) => render(user, orders)));

// Good
const [user, orders] = await Promise.all([fetchUser(id), fetchOrders(id)]);
render(user, orders);
```

## 20. Let Types Document Intent

- Name object shapes with `type` or `interface` instead of repeating large
  inline object types.
- Use **discriminated unions** instead of a bag of optional fields or booleans.
  The type then tells the reader which fields exist in each state.
- Prefer string literal unions or `as const` objects over loose `string` for
  fixed sets of values.
- Avoid `any`. Use `unknown` and narrow it. `any` silently turns off the
  checks a reader relies on.
- Let inference handle local variables. Annotate function parameters and
  exported/public return types, where the type is documentation.
- Avoid non-null assertions (`!`) and type casts (`as`) unless you explain why
  the compiler can't see what you know.

```ts
// Bad: which fields exist when?
type Request = {
  isLoading: boolean;
  error?: Error;
  data?: User;
};

// Good: each state is explicit
type Request =
  { status: "loading" } | { status: "error"; error: Error } | { status: "success"; data: User };
```

## 21. Use Options Objects Instead of Positional Flags

- Replace boolean or optional positional arguments with a named options object
  so the call site reads on its own.

```ts
// Bad: what do true and false mean?
createUser(name, email, true, false);

// Good
createUser({ name, email, sendWelcomeEmail: true, isAdmin: false });
```

## 22. Keep JSX Declarative

- Compute values and conditions **above** the `return`, not inline in JSX.
- Use early returns for loading, empty, and error states instead of wrapping
  the whole tree in conditionals.
- Use `cond && <X />` only when `cond` is a boolean. `0 && <X />` renders `0`.
- No nested ternaries in JSX. Extract a component or a variable.
- If a component's JSX doesn't fit on one screen, extract child components.

```tsx
// Bad
return (
  <div>
    {isLoading ? (
      <Spinner />
    ) : error ? (
      <ErrorView error={error} />
    ) : items.length ? (
      <List items={items} />
    ) : (
      <Empty />
    )}
  </div>
);

// Good
if (isLoading) return <Spinner />;
if (error) return <ErrorView error={error} />;
if (items.length === 0) return <Empty />;

return <List items={items} />;
```

## 23. Small Habits That Remove Surprise

- Use `===` and `!==`. Loose equality coerces types in ways few readers
  remember.
- Use template literals over string concatenation.
- Use `Map` and `Set` for dynamic keys and membership checks instead of plain
  objects used as dictionaries.
- Prefer named exports unless the codebase or framework requires default
  exports. Names then stay consistent across imports.
