---
source: https://www.typescriptlang.org/docs/handbook/2/types-from-types.html
tags:
  - chartjs
  - frontend
  - typescript
  - tshb
  - draft
---
> [!quote]
> Generic programming is a style of computer programming in which algorithms are written in terms of data types (which are specified later) that are then instantiated when needed for specific types provided as parameters. 
> 
> This approach permits writing common functions or types that differ only in the set of types on which they operate when used, thus reducing duplicate code. 

The identify function returns its argument. A type variable `T` is a declared after the function name using the angle bracket syntax. It is a stand in to the concrete type that will be provided at the time of invocation. 

```typescript
function identity<T>(x: T): T {
  return x;
}
```

The concrete type can be specifies manually using the angle bracket syntax to pass in the concrete type or it maybe inferred by the type-checker. Once specified the type-checker replaces all occurrences of the type variable with the concrete type.

```typescript
identity<number>(10);
/**
 * function identity(x: number): number {
 *   return x;
 * }
**/

identity<stirng>("hello world");
identity(true); // T is boolean

identity<number>()
```

recall the type-checker only allows operations which are supported by a type. type variables are standing for all types and as such the type-checker blocks most operations. 

```typescript
/**
 * T is a stand in for any type at all (number, Array<string>, { name: string} )
 * hence the length property access operation raises a type error
**/
function safeHead<T>(xs: T): T | null {
  if(xs.length === 0) return null;
  return xs[0];
}


/**
 * to fix this the type variable can be passed as an argument to the Array 
 * type constructor (note the shorthand syntax can be used)
 * now the type-checker knows xs is an Array and we use generics to specify that 
 * it is an array of any type
**/

function safeHead<T>(xs: Array<T>): T | null {
  if(xs.length === 0) return null;
  return xs[0];
}


```

***generic types***
generic types are essentially functions for types. The parameters of these *"functions"* are the type variables. The arguments are the concrete types. They return a new type which is simple the generic type with all instances of the type variable replaces with the concrete type.

function types, interfaces, type aliases, object types can all be made generic.

```typescript
function identity<T>(x: T): T {
  return x;
}

/**
 * F is a generic type alias that takes a concrete type and returns a new type
 * notice this new type is not itself generic
 * note how the type argument has to be specified to identity as well
 * F isn't a generic function type.
 * hence the generic function, identity, is not assignable to it
 * F is a generic type alias that describes concrete function type
 * the genric function typem identity, must be turned into a concrete function type
 * this is done by specifying a concrete type
**/
type F<T> = (x: T) => T;
const f: F<number> = identity<number>; 

/**
 * G on the other hand is a type alias for a generic function type
 * identity is a generic function type
 * bada bing bada boom
**/

type G = <T>(x: T) => T;
const g: G = identity; 

// the same thing can be done with call signatures

// H is a generic type interface with a type parameter T
// passing a concrete type argument into H return a non-generic interface
// with a non-generic call signature
interface H<T> {
  (x: T): T
}

// same shift with the F, specify number
const h: H<number> = identity<number>

// blah blah blah H is a non-generic interface with a generic call signature
// blah blah that's what identity is blah blah
interface H {
  <T>(x: T): T
}

const h: H = identity;  
```

note that placing the generic type parameter on the interface or type alias allows other members of the type to use that same type.

***generic classes***

> [!question]
the example in the handbook is interesting in that it describes a generic class that describes the types of a zero value and and add function. and then implements instances of these class for say `number`, `string` e.t.c. that sounds a lot like Haskell's type classes

```typescript
class Addable<T> {
  zeroValue: T;
  add: (augend: T, addend: T) => T;
}
```

***generic constraints***
restriction can be placed on what concrete types type parameters can accept. These restrictions are called constraints. The `extends` keyword is used to specify type constrains. 

```typescript
function getLength<T extends { length: number }>(x: T): number {
  return x.length;
}
```

A type parameter constrained with extends can only accept concrete types which have the properties of the extended type. note the differences between this and *excess property checks*. The concrete type may have multiple other properties, all that matters is that it possesses the properties described on the extended type (a la the structural typing).

```typescript
type Lengthable = { length: number };

function f(x: Lengthable): number {
  return x.length;
} 

f({
  length: 25,
  // type error excess property checks enforce 
  // only declared properties be defined on the type
  age: 22 
})

function g<T extends Lengthable>(x: T): number {
  return x.length;
}

g({
  length: 25,
  // not type error
  // hail structural typing
  age: 22 
})

```

note:
this makes sense. the behavior of  excess property checks was chosen to avoid the accidental mistyping of properties and other developer errors. type constraints on the other hand makes the intention it very clear and deliberate.

***type parameters to constrain type parameters***

```typescript
function getProperty<Type, Key extends keyof Type>(obj: Type, key: Key) {
  return obj[key];
}
 
let x = { a: 1, b: 2, c: 3, d: 4 };
```

***class generics***
revisit

***default type parameters***

- A type parameter is deemed optional if it has a default.
- Required type parameters must not follow optional type parameters.
- Default types for a type parameter must satisfy the constraint for the type parameter, if it exists.
- When specifying type arguments, you are only required to specify type arguments for the required type parameters. Unspecified type parameters will resolve to their default types.
- If a default type is specified and inference cannot choose a candidate, the default type is inferred.
- A class or interface declaration that merges with an existing class or interface declaration may introduce a default for an existing type parameter.
- A class or interface declaration that merges with an existing class or interface declaration may introduce a new type parameter as long as it specifies a default.