---
source: https://www.typescriptlang.org/docs/handbook/2/functions.html
tags:
  - backend
  - frontend
  - javascript
  - typescript
  - tshb
---
***function type expression***  
these are syntactically similar to arrow functions

```typescript
/*
type F = (arg0: type0, ..., argN: typeN) => rtnType;
*/
```

```typescript
type F = (x: number, i: number, xs: number[]) => string;

// note how only one parameter is specified
const f: F = x => x.toString();
const stringArr: string[] = [0, 1, 2, 3, 4, 5].map(f);
```


***call signatures***  
function type expressions describe only the call signature of a function but give no information about the properties the function might have. call signatures use to declare the types of callable objects (i.e. functions) with properties. 

```typescript
/* 
type F = {
  prop0: type0,
  ...
  propN: typeN,
  (arg0: type0, ..., argN: typeN): returnType
}
*/ 
```

```typescript
type Counter = {
  (): number,
  count: number
}

function makeCounter(): Counter {
  const counter: Counter = () => counter.count++; 
  counter.count = 0;
  return counter;
}

const counter = makeCounter();
console.log(counter());
```

> [!question]
> what happens if multiple call signatures are combined on the same type


***construct signatures***
prepend the new keyword in front of a call signature

```typescript
/* 
type Constructor = {
  prop0: type0,
  ...
  propN: typeN,
  new (arg0: type0, ..., argN: typeN): returnType
}
*/ 
```

some constructors can be called with or without new (e.g. `Date`). call and construct signatures can be combines on the same type


***generic functions***
generics are used to describe a correspondence between two values in a function declarations. such as relating the types of the parameters, return values and or variables declared within the function body together.

generics types are declared using type parameters. the generic types are then used to annotate the types of the parameters, return value or declared variables.

```typescript
// return type inference works for generics as well
// T | null need not be specefied
function head<T>(array: T[]): T | null {
  return array[0] ?? null;
}

head([0, 1, 2, 3]); // (1)
head("hello world".split(" ")); // (2)
```

the type-checker can infer the concrete types for generic type parameters. in the example above the concrete type of `number` is inferred for the generic type parameter `T`  on line `(1)`. while `string` is inferred on line `(2)`. 

in situations where the type-checker is unable to infer the type (or for the sake of explicitness)
the concrete types can be specified manually using the angle bracket syntax

```typescript
head<boolean>([]); // the type of an empty array can't be inferred 
```

***constraints***
constraints can be used to limit the kind of concrete types that generic type parameters can assume.

```typescript
type Lengthable = { length: number };

function longest<T extends Lengthable>(a: T, b: T): T {
  return a.length < b.length ? b : a;
}

// concrete type of `number[]` infered
// arrays have length property
const longerArray = longest([1, 2], [1, 2, 3]);

// concrete type literal of 'alice' | 'bob'
const longerString = longest("alice", "bob");

// Error! Numbers don't have a 'length' property
const notOK = longest(10, 100);
```


note pitfall
```typescript
type Lengthable = { length: number };

function minLen<T extends Lengthable>(obj: T, min: number): T {
  if (obj.length >= min) return obj;

  // when obj is passed in the concrete type of obj is infered for T
  // if said concrete type satifies the constraint
  // therefore the functions return type is the concrete type of obj
  // { length: min } satifies the constraint it is NOT the concrete type for T
  return { length: min }; // !!TYPE ERROR!!
}
```

***generic guidelines***  
- When possible, use the type parameter itself rather than constraining it
- Use a few type parameters as possible
- Type parameters should always relate the types of at least two values

***optional parameters***  
optional type parameters can be specified using syntax similar to optional properties on object types `param?: Type` . like optional properties `param` gets the type `Type | undefined` as unpassed optional parameters default to undefined. 

default parameters may be specified instead of an options parameter like so `param: number = 10`. note that type inference works with default parameters so the type can be omitted `param = "hello"` or `param = 10` 

note that `string` and `number` types are inferred as opposed to the literal types. the type-checkers assumes the value could be changed.

***optional parameters in callbacks***
in JavaScript a if a function is called with more arguments than parameters specified the surplus arguments are ignored. TypeScript follows this behavior as well, as a result function with fewer parameters can always be assigned types of functions with more parameters. 

*warning rough thoughts ahead*
the mistake is often made of using optional parameters in callbacks. 
but this is false
the callback is *always* invoked with all the argument.

imagine making a number to return the result of doubling its argument. 
and then making the argument (which the function will always be invoked with) optional.
stupid right???

if the callback doesn't intend to use some of the arguments passed in it can just not specify the parameters. then when the caller invokes the callback with the excess arguments they are just ignored

```typescript
type F = (x: string, i: number, xs: string[]) => string;

// toUpper only accepts on argument
// F has 3 params specified
const toUpper: F = x => x.toUpperCase();

// the extra arguments passed in to toUpper by map are ignored
"hello word".split("").map(toUpper); 
```

***function overloads***  
functions that can be called with a variety of different signatures are said to be overloaded. these functions can be given overload signatures in TypeScript to represent this behavior. 

overload signatures are function declarations without the function body. A function can have two or more overload signatures. the overload signatures are places above the actual function implementation. the signature of the function implementation is called the implementation signature.

each overload signature specifies a specific manner in which the overloaded function can be called. each overload signature must be *compatible* with the implementation signature. That is be a subtype of the implementation signature. for example the type of a parameter in an an overload signature must be assignable to the type of the corresponding parameter in the overload signature. like how  `string` is assignable to `string | number`

note that the implementation signature is *invisible* to callers of the function. invoking a function with argument that match the implementation signature but none of the overload signatures is an error

```typescript
// bad overload example I think
function boolToString(x: false): "false";
function boolToString(x: true): "true";
function boolToString(x: boolean): string {
  return x.toString();
}

boolToString(true); // the return type is  `"false"`
boolToString(false); // the return type is  `"true"`

// the tyoe of Math.random() < 0.5 is `boolean` i.e `true | false`
// while that is compatible with the implementation signature
// it is not with any of the overloads
boolToString(Math.random() < 0.5);

// note how writing boolToString in this manner allows for literal return types
// whereas using the non-overloaded version forces the return type to be `string`
// because toString returns `string`
// in reality the function will always return `"true"` or `"false"`

function boolToString(x: boolean): "true" | "false" {
  // this is a type error
  // toString has a return type of `string`
  // from toString's perspective it could be any `string`
  // but from the context it is obvious that it will either be "true" or "false"
  // to resovle the type error we would have to result to type assertions.
  // or the overloaded version above
  return x.toString();
}

```


***guildelines***
- prefer union types
  - but considering the example above what would be best practice
  - surely the more specific return type of `true | false` is better than `string`
  - surely using the parameter type to provide a specific return type is better than a union 
  - but would the situation where that would be possible even come up in real life

***declaring*** `this`
typically the values of `this` is inferred using code flow analysis. however this is not always sufficient. the ECMAScript specification states that `this` is an invalid parameter name. TypeScript uses this syntax space to allow for the declaration of `this` type in the function body


***other types***
`void` return type of function which do not return a value. note that `void` and `undefined` are not equivalent 

`object` represents any non-primitive type. note that `Object` represents instances of the `Object` constructor; never use it. 

`unknown` represent any value. it differs from `any` in that all operations on values of `unknown` are illegal.

`never` returned from function which never return a value i.e. always throw

`Function` represents instances of the `Function` constructor i.e. they have `call`, `bind` and `apply` methods and can be invoke. note: prefer `() => void` to type functions that are not intended to be called.

***rest arguments***
use `const` tuple types

***parameter destructuring***
use object types

```typescript
// Same as prior example
type ABC = { a: number; b: number; c: number };
function sum({ a, b, c }: ABC) {
  console.log(a + b + c);
}
```

`void` return type
a return type of `void` does not force a function to not return something in the way that a return type of `number` forces a function to return `number`. functions with `void` return types may return any value at all. that value regardless of what it is will always be given a type of `void`. the type system prevents any operations on values of type `void` making any returned values useless as far as the type system is concerned. 

essentially any value can be assigned to type `void`. so functions with type specifying a return type like `number` or `string` can be assigned to, or used in places where a function returning `void` is expected.

This behavior exists so that the following code is valid even though `Array.prototype.push` returns a number and the `Array.prototype.forEach` method expects a function with a return type of `void`.