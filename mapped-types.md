---
source: https://www.typescriptlang.org/docs/handbook/2/mapped-types.html
tags:
  - backend
  - frontend
  - tshb
  - typescript
  - draft
---
recall index signatures which allow for the specification of the types of properties before they are declared. mapped types build on this. 

mapped types take a union type and return an object type. the keys of the returned object key are members of the union type. the type of the values of these keys can then be specified in a manner similar to index signatures

```typescript
type Test = {
  [key in "hello" | "world"]: boolean;
}

/**
 * Test = {
 *  hello: boolean;
 *  world: boolean;
 * }
 */
```

mapped types are typically used in conjunction with the `keyof` operator and generic type parameters. Assuming we are given an object with certain optional properties each with different types. we define a function that takes this object and returns a new object with the same properties but with `boolean` values indicating whether or not the property existed on the original object. 

```typescript
type ToMap = {
  name?: string,
  age?: number,
  isMarried?: boolean,
}

type Mapping<T> = {
  [key in keyof T]: boolean
}

type Mapped = Mapping<ToMap>;

/**
 * Mapped = {
 *  name: boolean;
 *  age: boolean;
 *  isMarried: boolean;
 * }
 */

function f(x: ToMap): Mapped {
  return {
    name: !!x.name,
    age: !!x.age,
    isMarried: !!x.age,
  }
}
```

***mapping modifiers***
`readonly` and `?` can be applied to properties during mapping.
`(+|-)` is prepended to the modifier to add or remove it .

```typescript
type Freeze<T> = {
  +readonly [K in keyof T]: T[K]
}

type Mandatory<T> = {
  [K in keyof T]-?: T[K]
}

type Person = {
  name?: string,
  age?: number,
  isMarried?: boolean,
}

type FrozenPerson = Freeze<Person>;
type CompletePerson = Mandatory<Person>;
type FrozenCompletePerson = Freeze<Mandatory<Person>>;
```

***remapping***
the `as` key word can be used to remap the keys in mapped types. `never` can be used to exclude certain keys.

```typescript

type Person = {
  name: string,
  age: number
}

type ToGetter<T> = T extends string ? `get${Capitalize<T>}` : never;

// filters out all non string keys
type GoGetter<T> = {
  [K in keyof T as ToGetter<K>]: () => T[K]
}

type HighAchievingPerson = GoGetter<Person>

/**
 * HighAchievingPerson = {
 *  getName: () => string,
 *  getAge: () => number,
 * }
 */
```

note
the use of `<string & K>`  as opposed to a conditional type to select only the keys that are strings. see previous note for how `&` works on union types

```typescript
type Getters<Type> = {
  [K in keyof Type as `get${Capitalize<string & K>}`]: () => Type[K]
};
```

```typescript
type A = string & number; // never
type B = string & (string | number); // string
type C = string & "x"; // x
```

note:
The type definition for `Capitilze` uses a keyword `intrinsic`. 

> [!quote]
> The new intrinsic keyword is used to indicate that the type alias references a compiler provided implementation.  
> *~  [see](https://github.com/microsoft/TypeScript/pull/40580#issue-702383594)*

tip:
unions of any kind can be used.

```typescript
type EventConfig<Events extends { kind: string }> = {
    // E -> SquareEvent, CircleEvent
    // E["kind"] -> "square", "circle"
    [E in Events as E["kind"]]: (event: E) => void;
}
 
type SquareEvent = { kind: "square", x: number, y: number };
type CircleEvent = { kind: "circle", radius: number };
 
type Config = EventConfig<SquareEvent | CircleEvent>
```