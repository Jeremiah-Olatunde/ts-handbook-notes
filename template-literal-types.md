---
source: https://www.typescriptlang.org/docs/handbook/2/template-literal-types.html
tags:
  - backend
  - frontend
  - draft
  - tshb
  - typescript
---
template literals build on string literal types and can expand string literals into a union of multiple string literals

```typescript
type Foo = "foo";
type Bar = "bar";
type FooBar = `${Foo} ${Bar}`; // "foo bar"
```


```typescript
type T = "x" | "y";
type A = `${T} ${T}`;
// "x x" | "x y" | "y x" | "y y"
// cross multiplication
```

template literals can be combined with generic type parameter inferencing to do some powerful things


```typescript
type Watched<T> = T & { 
  // U
  on: <U extends (keyof T & string)>(
    // using template literals
    // firstNameChanged, ageChanged e.t.c
    changed: `${U}Changed`, 
    f: (newValue: T[U]) => void,
  ) => void;
};

type WatchedPerson = Watched<{
  firstName: string,
  age: number,
}>;

const person: WatchedPerson = (() => {throw ""})();

/**
 * type "ageChanged" is passed to changed paramter which has type `${U}Changed`
 * the typechecker uses type inference to extract `age` as U
 * it then verifies that "age" does indeed extend (keyof T & string)
 * then "age" is used to index T to get the type of the property
 * which is then used as newValue parameter type
 * shit is actually insane for real
*/

person.on("ageChanged", (newValue) => {
  // newvalue inferred to be number
});
```

***intrinsic***
`Uppercase<String>`
`Lowercase<String>`
`Capitalize<String>`
`Uncapitalize<String>`
[see](https://github.com/microsoft/TypeScript/pull/40580)

note

```typescript
type First<S extends string> = S extends `${infer U}${infer V}${infer W}` ? U : never;
type Second<S extends string> = S extends `${infer U}${infer V}${infer W}` ? V : never;
type Rest<S extends string> = S extends `${infer U}${infer V}${infer W}` ? W : never;

type H = First<"hello">; // "h"
type E = Second<"hello">; // "e"
type LLO = Rest<"hello">; // "llo"

```

> [!question]
>  *"fairly easily"* lol
> > [!quote]
> > Note that the `Capitalize<S>` and `Uncapitalize<S>` intrinsic types could fairly easily be implemented in pure TypeScript using conditional types and template literal type inference  
> > *~ [see](https://github.com/microsoft/TypeScript/pull/40580)*

for once they weren't lying.

```typescript
type Alphabet = {
  a: 'A',
  b: 'B',
  c: 'C',
  d: 'D',
  e: 'E',
  f: 'F',
  g: 'G',
  h: 'H',
  i: 'I',
  j: 'J',
  k: 'K',
  l: 'L',
  m: 'M',
  n: 'N',
  o: 'O',
  p: 'P',
  q: 'Q',
  r: 'R',
  s: 'S',
  t: 'T',
  u: 'U',
  v: 'V',
  w: 'W',
  x: 'X',
  y: 'Y',
  z: 'Z',
  A: 'a',
  B: 'b',
  C: 'c',
  D: 'd',
  E: 'e',
  F: 'f',
  G: 'g',
  H: 'h',
  I: 'i',
  J: 'j',
  K: 'k',
  L: 'l',
  M: 'm',
  N: 'n',
  O: 'o',
  P: 'p',
  Q: 'q',
  R: 'r',
  S: 's',
  T: 't',
  U: 'u',
  V: 'v',
  W: 'w',
  X: 'x',
  Y: 'y',
  Z: 'z'
}

type Head<S> = S extends `${infer U}${infer W}` ? U : never;
type Tail<S> = S extends `${infer U}${infer W}` ? W : never;

type InvertCase<T> = 
  Head<T> extends keyof Alphabet 
    ? `${Alphabet[Head<T>]}${Tail<T>}`
    : never;

type A = InvertCase<"hello">
```


generating the `Alphabet` type using JS is sort of cheating. same thing as using `instrinsic` for compiler provided implementation. 

find a better way

```typescript

// generate type
 const alpha = Array(26).fill(0).map((x, i) => {
   return [String.fromCharCode(i + 97), String.fromCharCode(i + 65)];
 });

const alphabetObjectType = Object.fromEntries(alpha);
```