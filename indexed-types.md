---
source: https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html
tags:
  - backend
  - frontend
  - tshb
  - typescript
  - draft
---
object types can be indexed using the square bracket notation to get the type of the indexed property. 

note: type is passed into the square brackets 

```typescript
type Person = {
  name: string,
  age: number
}


type S = Person["name"]; // "name" is a literal string type not a string
```

because types are used to index it is possible to use unions for indexing

```typescript
type I = "name" | "age";
type SA = Person[I]; // SA is `string | number`
```

warning: type errors are raised when indexing non-existent properties

the type of an array's elements can be accessed by indexing with the `number` type

```typescript
const testArr = [
  { name: "Jesuseun Jeremiah Olatunde", age: 15 }
]

// testArray is a run time value hence typeof gets the type
// index thte type with number
type Person = (typeof testArr)[number];

/**
 * This probably works because the type of Array is approximately
 * indexing the array type with `number` just returns the number index type
 */
type A = {
  [index: number]: { name: string, age: number }
}
```

tip:

```typescript
type OnlyStringIndexes<T> = {
  [index: string]: T,
  // number index signature must be a subtype of string index signature
  // never is a subtype of all types
  [index: number]: never,
  [index: symbol]: never,
};
```