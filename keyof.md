---
source: https://www.typescriptlang.org/docs/handbook/2/keyof-types.html
tags:
  - backend
  - draft
  - frontend
  - typescript
  - tshb
---
the `keyof` type operator takes an object type and produce a string or numeric literal union of its keys

```typescript
type Person = {
  name: string,
  age: number
}

type NameAge = keyof Person;

const x: NameAge = "name";
const y: NameAge = "age";

```

if the operand has an index signature then the type of that is returned

```typescript
type N = keyof Array<number>;

const x: N = 0;
const y: N = 1;
const z: N = 10;

const a: N = "0"; // type error
```

note that if the index signature is a string `keyof` returns `string | number` as opposed to just `string`. Due to the fact that indexing an object with a `number` is the same as indexing with a `string` as the former is coerced to the latter. 

one would think this should also apply to numeric index signatures. because numeric indexes are also converted to strings even when accessing array elements.

The possible rationale behind is might be that numeric index signatures are used for pure array types. as such indexing with an actual string would probably be a mistake. sure you could access `array.length` with `array["length]"` but no one actually does that. basically there are no properties on an array type that you'd index with a string. (convention). so the type-checker enforces this to prevent error.

however (I guess) when making an object with a string index signature perhaps having properties to be indexed with numbers are more normal. tbh idk

just remember the the type of the number index signature has to be a subtype of the string index signature