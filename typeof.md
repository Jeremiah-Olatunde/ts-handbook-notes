---
source: https://www.typescriptlang.org/docs/handbook/2/typeof-types.html
tags:
  - backend
  - frontend
  - tshb
  - typescript
  - draft
---
`typeof` when used in a *type* context returns the type of its operand

```typescript
let x = "hello";
const foo: typeof x = "foo";

const y = "hello"; // literal string type
const bar: typeof x = "foo"; // type error

function square(x: number): number {
  return x ** 2;
}

type N = ReturnType<typeof square>
```

note:
`typeof` can only be used on identifiers (variable names) or their properties. this limitation exists to avoid confusion with the `typeof` operator in JavaScript