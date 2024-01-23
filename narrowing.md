---
source: https://www.typescriptlang.org/docs/handbook/2/narrowing.html
tags:
  - backend
  - frontend
  - draft
  - typescript
---
the type-checker analyzed control flow constructs and overlays type analysis over them. Conditional expressions that allow the type-checker to analyze the most specific type possible for a given branch of executions are called type guards (e.g. `if(typeof x === "number")`). The process of refining types to more specific types than declared is called _narrowing_.


`typeof` type guards  
the type-checker can use `typeof` checks in order to narrow down the types of values. It is aware of the behaviors of the operate, event he esoteric ones such as `typeof null === "object"`

```typescript
function foo(x: string | string[] | null){
  if(typeof x === "object"){
    // x here is narrowed to string[] | null
    // as they both pass the conditional checl
  }
}
```

***truthiness narrowing***
truthy values are coerced into `true` in a Boolean context, falsy values are coerced to `false`.
`Boolean` function is used to coerce values into their Boolean equivalents. The double negation `!!` also has a similar purpose. When using `!!` to coerce values the type-checker assumes a literal Boolean type `true` or `false` for the result. While for `Boolean` function it assumed the general type `boolean`

```typescript
const foo = Boolean("hello world"); // typeof foo boolean
const bar = !!0; // typeof bar is false
```

truthiness narrows can be used to guard against `null` or `undefined` types 

```typescript
function foo(x: string | string[] | null){
  if(x && typeof x === "object"){
    // x here is narrowed to string[]
    // if x === null it would be coerced to false
    // if x === string the typeof would fail
    // hence in this execution branch x can only be string[]
  }
}
```


> [!note]
> Truthiness checking on primitives can be error prone due to coercion caveats. In the example above the empty string `""` is coerced to `false` and not handled by the body of the `if` statement. This also has the effect of making the type of `x` be `string | null` as opposed to just `null` if an `else` statement was attached.

> [!tip]
> - A linter can be used to catch possible errors such as these
> - or just specifically check for `null` with `if(x !== null)`


***equality narrowing***
when comparing a value of a union type to a value of another type the type-checker narrows the types of the values to one common between the two values. If comparisons are done with literal values then the type is narrowed to a literal type

```typescript
function foo(x: string | number, y:  string | boolean){
  if(x === y){
    // x and y must both be strings
  } else if (x === "") {
    // x type is a ""
  }
}
```

the type-checker also handler the caveats of `==` checks. Specifically the sweet couple of `null` and `undefined`

```typescript
function foo(bar: null | undefined | string){
  if(bar != null){
    // bar type is string
    // both null and undefined fail the comparison
  }
}
```

`in` ***operator narrowing***

> [!note]
>  `key in obj` operator checks for a property not just on `obj` but on objects in it's prototype chain as well

> [!tip]
> `key in obj`  returns `true` even if the value of `key` is `undefined`. This can be used to differentiate between properties that are on an object but have their value set to `undefined` and properties that do not exist at all on an object

type Fish = { swim: () => void };
type Bird = { fly: () => void };

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };

function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    // animal narrowed to Fish
    return animal.swim();
  }

  // animal narrowed to bird
  return animal.fly();
}
```

```typescript
type Fish = { swim?: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    // passes if swim is defined on animal but set to undefined
    // of is swin is set to function
    // either of those cases animal must be a Fish
    // ergo animal narrowed to Fish
    // and the type of swim is (undefined | () => void)
    // attempting to call swim is an error
  }

  // animal not narrowed at all here
  // it is possible that animal is of type Fish
  // but the swim property is not defined at all on animal
  // hence it would fail the conditional above
  // and an animal of type Fish could reach here
  // hence animal is still Fish | Bird
  return animal.fly();
}
```

`instanceof` ***narrowing***
`x instanceof X` checks to see if `X.prototype` is on the prototype chain of `x`. the type-checker can used `instanceof` checks to narrow types


***assignments***
narrowing also occurs during assignments 

```typescript
let x: number | string = Math.random() < 0.5 ? 0 : "0";

// type of x number | string

x = 10; // 10 is assignable to type number | string

console.log(x); // type of x is number

// narrowed type is number
// declared type is still number | string
// "10" is assignable to number | string
x = "10"; 

// narrowed type is now string
```

the type of x has been narrowed to `number` (it other words it has a *narrowed type* of . The type checker knows that unless `x` is reassigned it will always be a `number`. note that the literal number type `10` is not chosen by the type checker due to is being declared with `let` meaning it maybe changed to another `number` at any time.

note that  `x` is reassigned to `"10"` even though its *narrowed type* is currently `number`. Assignability to always checked against the type used in the declaration of a value (i.e. the *declared type*). The narrowed type becomes `string`. (again not the string literal type `"10"`)


note that type narrowing can occur at declaration as well

```typescript
let x: string | number = 10;

// narrowed type of x here is number
// however the assigned type is still string | number
console.log(x); 
```


***control flow analysis***
control flow analysis is the analysis of code based on the reachability. the type-checker uses control flow analysis  in conjunction with the narrowing techniques discussed earlier.

***type predicates***

> [!tldr]
> custom type guards are really just run time checks to see if a specific operation is supported on a value before using the operation on that value. static type-checker in the mud.

a predicate is a function that returns a `boolean` value. TypeScript allows user defined type guard via functions that return *type predicates*. A user defined type guard is simply a function that takes a value of a union type and returns a `boolean` value indicating whether or not the value is of a specific member of the union type. However the return type of the type guard is not `boolean` but instead a *type predicate* of the form `parameterName is TargetType`.

the function itself takes a value and may specify any custom logic to determine whether or not a value of a desired type  `true` or `false`. 

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

note how type assertion is used to force the type of `pet` to`Fish` so that the `swim` property can be examined to see if it is `undefined` (i.e. it doesn't exist). This would fail without the type assertion as only operations supported by all members of a unions are valid and `swim` is only supported by `Fish`.

note that this type guard may not produce accurate results if `swim`  was an optional property on `Fish`. In that case `swim` may be undefined (either by not existing on the object or being assigned to `undefined`) but still be of type `Fish`.

> [!warning]
> Mistakes in type guards can not be cause by the type-checker and can leak out into surrounding code

```typescript
interface Person { name: string };
interface Athelete extends Person { sport: string };

function isAthelete(person: Person | Athelete): person is Athelete {
  return true; // improper validation simulation
}


function bar(x: Person | Athelete){
  if(isAthelete(x)){
    // always runs
    // accessing undefined property no errors
    console.log(x.sport);
  } else {
    console.log(x.name)
  }
}

bar({name: "jjo"}) // calling bar with an objec that is not of type Athelete
```

***discriminated (tagged) unions***

> [!quote]
> a discriminated union is a [data structure](https://en.wikipedia.org/wiki/Data_structure "Data structure") used to hold a value that could take on several different, but fixed, types. Only one of the types can be in use at any one time, and a **tag** field (present on all types) explicitly indicates which one is in use.
> 
> ***~ [Wikipedia](https://en.wikipedia.org/wiki/Tagged_union)***

> [!question]
> Primagen pointed out his dislike of the structural typing system of TypeScript. The lack of algebraic data types. However the Wikipedia article pointed out above points out that the algebraic data types of haskell are essentially discriminated unions and that discriminated unions can be implemented in any language. Investigate using discriminated unions in TypeScript in a manner similar to Haskell 

```typescript
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

`Shape` type is intended to represent a value that can be either a circle or a square. The idea being that circle's store their radius and squares store their length. However encoding this in this manner posses a multitude of problems. One both the `radius` and the `sideLength` fields could be omitted. checking the type of `kind` gives no information about the presence of either `raduis` or `sideLength`

```typescript
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    // radius type undefined | number
    return Math.PI * shape.radius ** 2;
  }

  // sideLength type undefined | number
  // the kind type guard is essentially useless
}
```

A better approach would be to use discriminated unions

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}
 
type Shape = Circle | Square;
```

Notice now how all values of type `Shape`  have the `kind` property. But the type members of `Shape` (i.e. `Circle` and `Square`) each have their own distinct values for the `kind` property. In that way the value of `kind` *discriminates* (see what I did there) which member of the *union* the a value of type `Shape` is. (say it with me now, *discriminated union*)

***never type***
when all the possible types of a value have been narrowed away the type-checker ascribes the value the `never` type. `never` is assignable to every type, however no type is assignable to `never`

> [!quote]
> Tagged unions are most important in [functional programming](https://en.wikipedia.org/wiki/Functional_programming "Functional programming") languages such as Haskell where they are called _datatypes_ and the compiler is able to verify that all cases of a tagged union are always handled, avoiding many types of errors.
> 
> ***~ [Wikipedia](https://en.wikipedia.org/wiki/Tagged_union)***

never in conjunction with discriminated unions can be used to implement a similar exhaustiveness checking as seen in the haskell compiler

```typescript
type Shape = Circle | Square;
 
 
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      // kind === "circle" hence shape of type `Circle`
      return Math.PI * shape.radius ** 2;
    case "square":
      // kind === "square" hence shape of type `Square
      return shape.sideLength ** 2;
    default:
      // shape is of type never
      // as kind can only be "circle" or "square"
      // ensuring shape is always of type never in default case
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

in the example above if a new member `Triangle` is added to the union the type of `shape` becomes `Triangle` in the default case. This throws a type error as `Triangle` can not be assigned to `never`. In this manner never and discriminated unions force the handling of all types.