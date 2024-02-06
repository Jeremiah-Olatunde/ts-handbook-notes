---
source: https://www.typescriptlang.org/docs/handbook/2/conditional-types.html
tags:
  - backend
  - frontend
  - tshb
  - typescript
  - draft
---
conditional types allow for the selection of types based on certain criteria.

```typescript
type T = A extends B ? C : D;
```


`NameOrId` has a generic parameter `T`. A type constraint is used on the parameter to enforce that is a subtype of the `number | string` union type. if `T` extends the `number` type then an `IdLabel` type is returned. Otherwise (i.e. `T` is a  `string` or `string | number`) returns `NameLabel`  type

```typescript
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

In a similar manner to the type narrowing that occurs in various conditional constructs, generic types used in conditional types are constrained by the conditional 

```typescript
/**
 * in the true branch the typechecker infers that T has the property message
 * an indexed access type ("message") is used to return the type
**/ 
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

type Content = MessageOf<Email>; // string
```

note the use of the `unknown` type in the example above. 
one would expect that `T["message"]` would always be of type `unknown` or `never`.  
If `unknown` is replaced with `number` that is the case

```typescript
type MessageOf<T> = T extends { message: number } ? T["message"] : never;

interface Email {
  message: string;
}

interface Text {
  message: number;
}

type EmailContent = MessageOf<Email>; // never
type TextContent = MessageOf<Text>; // number
```

`A extends B` constrains `A` to be a subtype of `B`.  

Again the structurally typed nature of TypeScript comes into play.
an object type `A` is a subtype of an object type `B` if it can be used in all the operations  `B` is used; in the case of object types that refers to the property access operation.
`T extends { propKey: propType }` means the concrete type is used for the generic type `T` must have the `propKey` property i.e. the property access operation for `ConcreteValue[propKey]` is supported

however this also applies to `propType`.  
again due to structural typing the type of the value of the `propKey`  property on the concrete type supplied for `T` must be supported in all the operations that `propType` can be used in. (if `propKey` was `string` this could be `.toString()`, `.split()` etc.) 

The key point being the type of the `propKey` value on the concrete type does not have to be of type `propType` exactly but a subtype of `propType`. if `propType` was `string` then the value could be a string literal.

`T extends { message: string }`, `type T = { message: "hello world" }`.

this in conjunction with type inference explains why `unknown` works.

recall that `unknown` is a super type of all types. 

```typescript
const x: unknown = 5;
const y: unknown = "hello world";
const z: unknown = { foo: "bar" };
```

 `T["message"]` can be of any type at all. the type checker then infers the specific type once the concrete type is supplied. 
 
```typescript
// an example with `number | string` instead of unknown
type MessageOf<T> = T extends { message: number | string } ? T["message"] : never;

interface Email {
  message: string;
}

interface Text {
  message: number;
}

type EmailContent = MessageOf<Email>; // never
type TextContent = MessageOf<Text>; // number
```

The principles outlined above can be used to make a flatten type. 

```typescript
/**
 * T could be Array<number>, Array<string>, e.t.c.
 * indexed types are used to get the type of the elements
 * 
 * type Array = {
 *   [index: number]: someType
 * }
 * type T = Array[number]; // someType
*/
type Flatten<T> = T extends Array<unknown> ? T[number] : T;

type T = Flatten<number[]>;
type S = Flatten<(number | string)[]>;
```

***inferring with conditional types***
using conditional types to apply a constraint an then extracting a type by levering the inferencing capabilities of the type checker is a common pattern in TypeScript development. The `infer` allows types to be inferred from compared types in conditional types.

```typescript
// type of array contents inferred from compared type T
type Flatten<T> = T extends Array<infer U> ? U : T;
```

> [!question]
> Does the quote below mean that one can "manually" fetch out types wherever `infer` is used
> >[!quote]
> >we could have inferred the element type in `Flatten` instead of fetching it out “manually” with an indexed access type  
> >_~ handbook_

```typescript
type GetReturnType<F> = F extends (x: never) => infer R ? R : never;

type A = GetReturnType<(x: bigint) => string>; // string
type B = GetReturnType<(x: number) => boolean>; // boolean
```

note the use of the `never` type. `unknown` doesn't work here. this is due to the way structural typing applies to functions and their arguments.

> [!summary]
> a function type `G` is a subtype of `F` if all of the arguments of `F` are a ***subtype*** of `G`.  
> `never`  is a subtype of all types

```typescript
type F = (x: string) => void;
type G = (x: string | number | boolean) => void;
type H = (x: "hello world") => void;

type A = G extends F ? true : false; // true
type B = H extends F ? true : false; // false

// valid
const g = (x: string | number | boolean) => {
  if(typeof x === "string"){
    /**
     * x is narrowed to string type
     * the full set of string operations are allowed here
    **/
    x.split(" "); // only valid on strings
  }
  
  if(typeof x === "number"){
    x.toFixed(2);
  }

  if(typeof x === "boolean"){
    // some only boolean operation idk
  }
  /** 
   * only operations that apply to string, number and boolean types allowed
   * meaning any operation is apply to x must be valid on string
   * however only the subset of string operations also valid on number and boolean
   * are allowed here
  */
  x.toString(); // valid on all types
}; // (*)

const f: F = g;

// type error
const g: F = (x: "hello world") => undefined;
```

a function type `G` is a subtype of `F` if all of the arguments of `G` are a ***supertype*** of `F`.  

from a structural typing standpoint this makes sense. 

`g` accepts the union type `string | number | boolean` meaning within the body of `g` the type checker enforces that all of these possibilities are handled.

assigning `g` to type `F`  forces now `g` to only accept a `string`. 
this isn't a problem because `g` could initially handle `string | number | boolean` that necessarily means it can handle only a `string`. 

calling `g` with a `string` will run the `string` type guard and the body of g only allows operations that apply to all types (including `string`). 

at no point will an operation not supported on string be called on the argument of g. and if one is it must be after the `string` type guard. (the main goal of structural typing).

assuming it was the other way around

```typescript
type F = (x: string) => void;
type G = (x: string | number) => void;

const f: F = (x: string) => {}

// type error
const g: G = f;
```

`f` can only take a `string` type; the body of `f` applies string operations to `x`. 

it doesn't not use type guards or narrows to account for the possibility of `x` being a type other that a `string` because `x` has explicitly been specified to only ever be of a `string` type.

type `G` specifies that a function can take a `string` or a `number` type. if the code above was valid then a `number` type could be passed to `f` and `string` operations will be applied to that `number` type.

note that this does not apply to the return types. `G extends F`  if the return type of `G` is a subtype of `F`

```typescript
type F = (x: string) => string;
type G = (x: string) => "hello";

type A = G extends F ? true: false;
```

`never` is a subtype of all types (i.e. all types are a super type of `never`). 
this in in contrast to `unknown` which is a super type of all types.

`never` can be assigned to all types...

```typescript
function foo(): never {
  throw "unimplemented";
}

const x: number = foo();
const y: string = foo();

// this is fine because execution will never reach here
x.toExponential(); 
x.split();
```

… and no type can be assigned to `never`

```typescript
// all type errors
const x: never = 5; 
const x: never = "hello";

function foo(): never {
  return 20; // attempting to assigned 20 to never via return
}

function bar(): never {
  // implicity returning undefined
  // attempting to assign undefined to never
}
```

finally...

```typescript
type F = (x: never) => void;
type G = (x: string) => void;


type A = G extends F ? true : false;


const g: G = (x: string) => {
  x.split(" ");
}

/**
 * assigning g to type F forces g to only accept never values
 * nothing can be assigned to never
 * to this means g can't accept anything
 * g can't even be invoked as type F requires and argument
 * essentially assigning g to F prevents g from ever being called
 * and if g is never called then technically the string operations applied
 * to x in the body of g never run and are valid, sort of.
 */

const f: F = g;
```

> [!question]
> write the `Flatten` type without using `infer`


***distributive conditional types***

when given a union type conditional types are applied to each member of the union individually

```typescript
// the handbook uses any
// but given what we've learnt about unknown
type Distributive<T> = T extends unknown ? T[] : never;
type NotDistributive<T> = T[];

type DArr = Distributive<number | string>; // number[] | string[]
type NDArr = NotDistributive<number | string>; // (number | string)[]
```