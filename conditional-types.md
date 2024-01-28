---
source: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
tags:
  - backend
  - frontend
  - tshb
  - typescript
  - draft
---
conditional types allow for the selection of types based on on certain criteria.

```typescript
type T = A extends B ? C : D;
```


`NameOrId` has a generic parameter `T`. A type constraint is used on the parameter to enforce that is be a subtype of the `number | string` union type. if `T` extends the `number` type then an `IdLabel` type is returned. Otherwise (i.e. `T` is a  `string` or `string | number`) returns `NameLabel`  type

```typescript
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```