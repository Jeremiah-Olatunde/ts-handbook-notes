---
source: https://www.typescriptlang.org/docs/handbook/2/objects.html
tags:
  - backend
  - frontend
  - draft
  - typescript
  - tshb
---
super super drafty

```typescript
// object types using type aliases
type Person = {
  name: string,
  age: number
};

// object types using interfaces
interface Person {
  name: string,
  age: number
}

function greet({name: string, age: number}){
  console.log("hello from", name)
}
```

***property modifiers***

***optional properties***

```typescript
interface Person {
  name: string,
  age: number,
  martialStatus?: boolean, // type is `boolean | undefined`
}
// use undefined checks to narrow types of optional properties
```

note type annotation syntax is similar to object destructing syntax for renaming destructured variables. annotate the whole destructing object instead


***readonly properties***
properties can be prepended with the `readonly` type modifier. the type-checker enforces that the property will never be rewritten to. this is not enforced at run-time however. note that `readonly` modifier is similar to `const` for declaring variables in that it does not prevent the modification of property that are objects only reassignment 

```typescript
type Person = {
  birthYear: number,
  readonly mother?: Person,
};

const person: Person = {
  birthYear: 2001,
  mother: { birthYear: 1974 }
} 

// modification of readonly mother prop is valid
person.mother!.birthYear += 10; 

// reassingment invalid
person.mother = { birthYear: 1984 }
```

***index signatures***

specifies the type that will be returning when an object is indexed with a value of a the specific type. the only supported index types are `string`, `number` and `symbol` as these are the only valid values that can be used as object keys in values.

multiple index types can be supported but if both `string` and `number` index types are used then the type return from a `number` index must be a subtype of the type return from `string` indexing. Recall that when an object is indexed with a `number` in JavaScript it is coerced to a `string`. Ergo numeric indexing is actually just string indexing. this caveat does not apply to `symbol`s

```typescript

interface Weird {
  [index: string]: string | number,
  // number index type is a subtype of the string index type
  // i.e. a value of type string is assignable to the type string | number
  [index: number]: string

  // symbols can just do whatever they want
  [index: symbol]: boolean,
}

const x: Weird=  {
  0: "hello world",
  "name": 42,
  [Symbol("foobar")]: Math.random() < 0.5,
}
```

specifying the index type for `string` or `number` index enforces the constraints on all keys of that type. 

```typescript
interface NumberDictionary {
  [index: string]: number;

  // any explicitly specified keys must match the index type
  length: number; // ok
  name: string; // type error
}
```

use union types for the index types to skirt around this if need be.

specifying an index type also has the adverse effect of making any property access using that index type valid. even for properties that aren't defined. the type-checker don't notice turning this values into essentially a live bomb

```typescript

interface CreateBomb {
  [index: string]: number;
}

const dict: CreateBomb = {
  exists: 42,
};

// foo does not exists
// accessing foo should be a type error
// or at least be undefined | number
// but nah, it's type is number accoring to the type-checker
const liveBombBasically = dict.foo;


try {
  // accessing a propert of undefined
  // we all know how that goes
  liveBombBasically.toFixed(2);
} catch(error) {
  // bomb containment unit
  console.log("I'm not mad, I'm just dissappointed", error);
}
```

tip: `readonly` can be used in conjunction with index types to prevent assignment to indexes of a certain type

***excess property checks***
this one is weird.

recall typescript is structurally typed. All that matters is that a value supports the operations intended to be used on it. that is what types in structural type systems do, they describe the operations supported by a value of that type. 

*"In theory"* an object type need only convey that the property access operation on and object is safe ***(i.e. the property exists on the object)***. *"In theory"* an object value should be assignable to a certain object type if that object has the properties specified on the the object type regardless of whether or not the object has other *"excess"* properties.

```typescript
type Person = {
  name: string
}

function getName(person: Person){
  return person.name;
}

getName({
  name: "Jesuseun Jeremiah Olatunde",

  // In theory the structure of the object 
  // satifies the required operations specified by the Person type
  // i.e. accessing the name property
  // based on the behaviour of structural type systems this should be fine
  age: 22,
  isMarried: "he wished",
})

```

yeah but shit doesn't go down like that. it raises and error. the type-checker errs on the side of cautions assumes the extra properties are accidental. this behavior is referred to as excess property checks

this can be resolved using index types to specify that the object may have other types that we don't care about.

```typescript
type Person = {
  // string is a subtype of the index type any
  // so this is valid
  name: string,
  [index: string]: any,
}

function foo(p: Person): string {
  return p.name;
}

foo({
  name: "bar",
  age: 22,
})
```

***extending types***
the `extends` keyword allows a type declared using the `interface` keyword to copy members from another type and add its own types. `extends` can be used with multiple types i.e. `StreetSweepingBicyle` from [javascript.info](https://javascript.info/mixins) .  
`extends` can only extend object types.  
note: if the extending type contains a property found on the extended type then its type on the extending class must be a subtype of the type on the extended class.

***intersecting types***
`&` is mainly used to combine object types. 

if both the operands are object types the intersection operator returns a new object type containing the properties of both its operands.   

if both operands are union types (assuming union types with single members exist e.g. `string` is just a union type with one member `string`) then the returned type is the intersection (in the set theory sense) of the members of the union types.

if both operands are object types and both have the same property type then the `&` of the types are used following the two rules above

***generic object types***

generics are like functions but for types. they take as their argument a type and return a new type typically constructed from the type argument

```typescript
type Box<T> = {
  content: T
}

type NumberBox = Box<number>;
/**
 * type NumberBox = {
 *   content: number
 * }
**/
type StringBox = Box<string>;
type BoolBox = Box<boolean>;
```

generics work for type aliases as well allowing for type generics to be using when constructing union types.

```typescript
type Option<T> = T | null;
type Result<T, E extends Error> = T | E;
```

here type generics are used to simulate the Option and Result enums found in Rust. 

> [!question]
> The differences between the TypeScript and Rust version of the enums is worth further investigation. My Rust skill at the moment are *rusty*. The TypeScript version are really just aliases. This makes sense due to the structurally typed nature of TypeScript. In Rust the enums are like their own concrete types. Like actual containers for their values akin to objects. 
> 
> The `Option` enum in Rust is like a literal box complete with methods to operate on the value that may or may not be in the box (`unwrap`, `filter`, `map`; shout out to Haskell). These methods are the guardians preventing the programmer from accidentally performing an operation on a value that does not exist. 
> 
> The TypeScript version above simply indicates to the type system that the value could be `T` which is replace with a concrete type or `null`. The type-checker itself is the guardian preventing the programmer from performing accidental operations on a `null` value by enforcing that the possible null values be handled
> 
> A TypeScript alternative for the `Option` enum would probably be implemented as a class
> (project idea)


```typescript
class MockOption<T>{
  constructor(private value: T | null = null){}

  map<U>(mapping: (value: T) => U): MockOption<U> {
    if(this.value === null) return new MockOption();
    return new MockOption(mapping(this.value));
  }

  filter(predicate: (value: T) => boolean): MockOption<T> {
    if(this.value === null) return new MockOption();
    return predicate(this.value) ? new MockOption(this.value) : new MockOption();
  }
}

const x = new MockOption(10);
console.log(x);

const xmap = x.map(x => x * 2);
console.log(xmap);


const xfilter = x.filter(x => x > 50);
console.log(xfilter);
```


generic types can be used in generic functions too.

```typescript
function safeHead<T>(xs: T[]): Option<T> {
  return xs[0] || null;
}

```

***built in generics***
the array type is generic i.e. `Array<T>`. `number[]` is an alias for `Array<number>` . `Map<K, V>`, `Set<V>`, `Promise<T>` are other examples.

***read only array***
`ReadonlyArray<T>` specifies an array that can not be modified . `readonly T[]` is the shorthand.

***tuples***
tuples are arrays with a fixed number of indexes with each index having a fixed type. This property allows them to be used as arguments for functions that destructure their parameters. 

Tuples provide index out of bounds type errors.

the `length` property of tuples is a literal number type equal to the length of the tuples

```typescript
const foo: [number, number] = [0, 1];
const len = foo.length; // type is 2
foo[2] // type error
```

tuples may have optional properties. these are specified by placing a question mark after the type. they can only come at the end of the tuple. the optional parameters affects the length of the tuple, making it a union of literal number types of all the possible lengths.

```typescript
const foo: [number, number, number?] = [0, 1];
const len = foo.length; // type is 2 | 3
```

tuples may have rest elements which must be of an array or tuple type. they describe the type of the elements at a section of the tuple.

```typescript
const foo: [number, string, ...boolean[]] = [42, "meaningOfLife", true, true, false]
const foo: [...boolean[], number, string] = [ true, true, false, 42, "meaningOfLife"]
const foo: [number, ...boolean[], string] = [42, true, "meaningOfLife"]

// this is just a number | boolean array with extra steps
const bar: [...(number | boolean)[]] = [10, 20, true, 20, false]
```

and `readonly` tuples are a thing