---
source: https://www.typescriptlang.org/docs/handbook/2/everyday-types.html
tags:
  - backend
  - frontend
  - javascript
  - typescript
  - draft
---
JavaScript has 7 primitives types as mentioned [[types|here]]. Most of these have identically names corresponding types in TypeScript

```typescript
let a: number = Math.random();
let b: string = "hello world";
let c: boolean = Math.random() < 0.5;
let d: bigint = 10000000000n;
let e: symbol = Symbol("id");
let f: undefined = undefined;
let g: null = null;
```

arrays are specified using the `<type>[]` syntax
```typescript
const testArr: number[] = [0, 1, 3];
```

`any` is a catch-all type that can be assigned to any value without throwing type errors. Any properties accessed of an `any` type is in turn also an `any` type

***type annotations***
type annotations can be added to variable declarations as has been show in previous examples. However in most cases TypeScript can automatically infer the types during variable declarations.

***Functions***
The parameters and return values of functions can be type annotated as well. Like with variable declarations the return value type can usually be automatically inferred.

```typescript
function randomPause(min: number, max: number): Promise<number> {
  const pauseTime: number = Math.random() * (max - min) + min;
  return new Promise(resolve => setTimeout(resolve, pauseTime, pauseTime))
}

// note the type of result is correctly inferred to be `number`
randomPause(1000, 2000).then(result => {
  console.log("paused for", result, "ms");
})
```

***contextual typing***
refers to when the type-checker automatically determines the types of the parameters and return value of an anonymous function be examining the context in which the function is defined.

```typescript
const letters = "hello world".split("");

// letter, idx, and letterArr parameters are all corretly typed
letters.map((letter, idx, letterArr) => letter.toUpperCase())
```

***Object Types***
The syntax for specifying object types is very similar to object literals, only the types of properties are placed where they values would be instead.
note that `;` or `,` can be used to separate properties

optional properties are specified using `?:` as opposed to `:` between the key and the type.
they have the type of `undefined | type` where type whatever type is specified 

```typescript
function printCoord(pt: { x: number, y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}

printCoord({ x: 3, y: 7 });
```

***Union Types***
union types are types formed by combining two or more types. Each type is called a union member. Values ascribed to this type can be of any of the member types of the union.

```typescript
const stringOrNumber: string | number = Math.random() < 0.5 ? 42 : "meaningOfLife";
```

when working with union types the type-checker only allows operations which are valid for all members of the union. 

> [!question]
> Investigate how this compares to conventions set theory. The set `A U B` is a combination of all the elements of sets `A` and `B`. The set `A ∩ B` contains only elements in both `A` ***and*** `B`.   
> 
> Through this lens the type-checker only allows operations `∈` set of intersection of operations supported by its members 
> 

***type narrowing*** allows for the use of operations only supported by specific type members.
Type narrowing occurs in situations where the type-checker can deduce more specific types typical in `if` statements with type checks

```typescript
function doSomething(x: string | number[]){
  // slice and lenght are supported operations on both string and number[]
  console.log(x.slice(0, x.length / 2));
  if(typeof x === "string"){
    // type of x narrowed to just string
    // hence can use toUpperCase operation
    console.log(x.toUpperCase())
  } else {
    // type of x narrows to number[]
    // can use reduce operation of array
    // note that p and v are correctly types using contextual typing
    console.log(x.reduce((p, v) => p + v), 0);
  }
}
```


***type aliases***
type aliases are different names for specified types Any type at all can be given a type alias. The type-checker substitutes occurrences of the type alias with the original type. Consequentially type aliases do not allow for algebraic types like haskell or rust. 

```typescript
type StringOrNumber = string | number;
type Person = { name: string, age: number, canHokeyPokey?: boolean };
```


***structural typing***
TypeScript is only concerned with the structure of the types; that they support the operations expected of them, and not the type itself. this makes typescript a structurally type type system

> [!quote]
> A **structural type system** (or **property-based type system**) is a major class of [type systems](https://en.wikipedia.org/wiki/Type_systems "Type systems") in which type compatibility and equivalence are determined by the type's actual structure or definition and not by other characteristics such as its name or place of declaration.  
> 
> *~ [wikipedia](https://en.wikipedia.org/wiki/Structural_type_system)*

***interfaces***
allow for the naming of object types.

```typescript
interface Person { 
  name: string, 
  age: number, 
  canHokeyPokey?: boolean 
};
```

type aliases and interfaces are similar in use and operation. the major difference is that type aliases are not extensible after declaration while interfaces can be extended with the `extends` syntax

```typescript
// adding new fields to Person
interface Person {
  isMarried: boolean
}

// extending Person
interface Athlete extends Person {
  sport: string, 
  team: string,
  awards: string[]
}

const person: Person = {
  name: "foo bar",
  age: 22, 
  canHokeyPokey: true,
  isMarried: true
}

const athlete: Athlete = {
  ...person, // note how typing works with spread operators...wild
  sport: "football",
  team: "arsenal",
  awards: []
}
```

the intersection operator `&` can be used to join two type aliases together.

```typescript
type Athelete = { sport: string, team: string, awards: string[] }

type PersonAthelete = Person & Athelete; 
```

more differences
- Prior to TypeScript version 4.2, type alias names [_may_ appear in error messages](https://www.typescriptlang.org/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWZWhfYAjABMAMwALA+gbsVjoADqgjKESytQPxCHghAByXigYgBfr8LAsYj8aQMUASbDQcRSExCeCwFiIQh+AKfAYyBiQFgOPyIaikSGLQo0Zj-aazaY+dSaXjLDgAGXgAC9CKhDqAALxJaw2Ib2RzOISuDycLw+ImBYKQflCkWRRD2LXCw6JCxS1JCdJZHJ5RAFIbFJU8ADKC3WzEcnVZaGYE1ABpFnFOmsFhsil2uoHuzwArO9SmAAEIsSFrZB-GgAjjA5gtVN8VCEc1o1C4Q4AGlR2AwO1EsBQoAAbvB-gJ4HhPgB5aDwem-Ph1TCV3AEEirTp4ELtRbTPD4vwKjOfAuioSQHuDXBcnmgACC+eCONFEs73YAPGGZVT5cRyyhiHh7AAON7lsG3vBggB8XGV3l8-nVISOgghxoLq9i7io-AHsayRWGaFrlFauq2rg9qaIGQHwCBqChtKdgRo8TxRjeyB3o+7xAA), sometimes in place of the equivalent anonymous type (which may or may not be desirable). Interfaces will always be named in error messages.
- Type aliases may not participate [in declaration merging, but interfaces can](https://www.typescriptlang.org/play?#code/PTAEEEDtQS0gXApgJwGYEMDGjSfdAIx2UQFoB7AB0UkQBMAoEUfO0Wgd1ADd0AbAK6IAzizp16ALgYM4SNFhwBZdAFtV-UAG8GoPaADmNAcMmhh8ZHAMMAvjLkoM2UCvWad+0ARL0A-GYWVpA29gyY5JAWLJAwGnxmbvGgALzauvpGkCZmAEQAjABMAMwALLkANBl6zABi6DB8okR4Jjg+iPSgABboovDk3jjo5pbW1d6+dGb5djLwAJ7UoABKiJTwjThpnpnGpqPBoTLMAJrkArj4kOTwYmycPOhW6AR8IrDQ8N04wmo4HHQCwYi2Waw2W1S6S8HX8gTGITsQA).
- Interfaces may only be used to [declare the shapes of objects, not rename primitives](https://www.typescriptlang.org/play?#code/PTAEAkFMCdIcgM6gC4HcD2pIA8CGBbABwBtIl0AzUAKBFAFcEBLAOwHMUBPQs0XFgCahWyGBVwBjMrTDJMAshOhMARpD4tQ6FQCtIE5DWoixk9QEEWAeV37kARlABvaqDegAbrmL1IALlAEZGV2agBfampkbgtrWwMAJlAAXmdXdy8ff0Dg1jZwyLoAVWZ2Lh5QVHUJflAlSFxROsY5fFAWAmk6CnRoLGwmILzQQmV8JmQmDzI-SOiKgGV+CaYAL0gBBdyy1KCQ-Pn1AFFplgA5enw1PtSWS+vCsAAVAAtB4QQWOEMKBuYVUiVCYvYQsUTQcRSBDGMGmKSgAAa-VEgiQe2GLgKQA).
- Interface names will [_always_ appear in their original form](https://www.typescriptlang.org/play?#code/PTAEGEHsFsAcEsA2BTATqNrLusgzngIYDm+oA7koqIYuYQJ56gCueyoAUCKAC4AWHAHaFcoSADMaQ0PCG80EwgGNkALk6c5C1EtWgAsqOi1QAb06groEbjWg8vVHOKcAvpokshy3vEgyyMr8kEbQJogAFND2YREAlOaW1soBeJAoAHSIkMTRmbbI8e6aPMiZxJmgACqCGKhY6ABGyDnkFFQ0dIzMbBwCwqIccabcYLyQoKjIEmh8kwN8DLAc5PzwwbLMyAAeK77IACYaQSEjUWY2Q-YAjABMAMwALA+gbsVjNXW8yxySoAADaAA0CCaZbPh1XYqXgOIY0ZgmcK0AA0nyaLFhhGY8F4AHJmEJILCWsgZId4NNfIgGFdcIcUTVfgBlZTOWC8T7kAJ42G4eT+GS42QyRaYbCgXAEEguTzeXyCjDBSAAQSE8Ai0Xsl0K9kcziExDeiQs1lAqSE6SyOTy0AKQ2KHk4p1V6s1OuuoHuzwArMagA) in error messages, but _only_ when they are used by name.

***Type Assertions***

type assertions allow for the manual specification of a type for situations where the compiler can't figure it out

```typescript
const div = document.getElementById("div") as HTMLDivElement;
// or
const div: HTMLDivElement = document.getElementById("div");
// or
const div = <HTMLDivElement>document.getElementById("div");
```

there is no runtime validation of type assertions hence exception as a result of operations on incorrect types may still occur. For example if `document.getElementById` returns `null`.
Best to manually input these checks of assert to `HTMLDivElement | null`

Only type assertions which convert to a _more specific_ or _less specific_ version of a type are allow. However multiple chained assertions with the first being to a type like `any` can skirt this

```typescript
const x = "hello" as any as number;
```


***literal types***
specific values of a particular type of values can be used as types.

```typescript
let hello: "hello" = "hello";
// hello can not be reassigned to any other string as "hello" is the only
// value type

// the type checked infers literal types when using const declarations
// as the values will never change
const foo = "foo";
```

being types themselves, type literals can be combined to form larger more useful types (such as with unions)

```typescript

type PrimaryColor = "red" | "green" | "blue";
function assignPrimaryColor(color: PrimaryColor): void {
  const div = document.getElementById("div") as (HTMLDivElement | null);
  if(div === null) throw new Error("div not found");

  div.style.backgroundColor = color;
}

type BinaryDigit = 0 | 1; // why didn't I just call it Bit??

function toBase10(x: BinaryDigit[]): number {
  // note how s has to be explicity annotated 
  // the type BinaryDigit was inferred which in this case, was wrong.
  return x.reduce((s: number, bit, i) => s + bit * Math.pow(2, x.length - 1 - i), 0);
}

type Bool = true | false; // wait that's just regular bool
```

***literal inference***
when the type of an object is inference the types of its properties are not inferred to be literal types. it is assumed they will be changed to another value of the same type.
appending `as const` to a property value indicates that the value is a literal type

the `!` operator removes `null` and `undefined` from a type explicitly. This is done when the fact that a value can not be `null`  or `undefined` can not be inferred by the type system