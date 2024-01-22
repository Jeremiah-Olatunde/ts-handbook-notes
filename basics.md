---
source: https://www.typescriptlang.org/docs/handbook/2/basic-types.html
tags:
  - javascript
  - typescript
  - backend
  - frontend
  - draft
---

questionable statement made by the handbook

> [!quote]
> We can _observe_ by reading the code that this function will only work if given an object with a callable `flip` property, but JavaScript doesn’t surface this information in a way that we can check while the code is running.
> 
> ```javascript
>function fn(x) {
>  return x.flip();
>}
>```
>


***static type-checking***
_Static types systems_ describe the shapes and behaviors of what our values will be when we run our programs


***non-exception failure***
the refers to things that would not lead to exceptions in JavaScript but would be flagged as errors by the `tsc`. such as
- accessing a property that does not exist on an object
- logic errors in if statements leading to unreachable code
- unreachable code below a return statement

***tooling***
refers to the use of the core type-checker of the typescript to provide features that give real-time aid in writing code

***the compiler***
`tsc input.ts` compiles `input.ts` and produces `input.js`. It alerts of possible errors found by TypeScript by logging them to the console.

***emitting with errors***
`--noEmitOnError` option tells the compiler whether or not to still produce the JavaScript version of a file if an error was detected. 

***type annotations***

```typescript
function fibonacci(n: number): number {
  if(n == 0 || n == 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const input: number = 10;
const output: number = fibonacci(input);
```

note:
most times the compiler can automatically infer the types of variables.

***downleveling***
by default the TypeScript compiler produces ES3 compatible JavaScript code. The specific ECMAScript version targeted by the compiler can be specified using the `--target` option

***strictness***
The aggressiveness of the type-checker in regards to validation and type-checking can be finetuned using options provided by the compiler. There are a multitude of these such options. The `--strict` flag (`"strict": true` config file) toggles all these options on simultaneously. However each of these options can be set or unset individually 

`noImplicitAny`   
prevents the type-checker form falling back to the `any` type when a type can not be inferred.

`strictNullChecks`
by default `null` and `undefined` can be assigned to values of any type. Setting `"strictNullChecks": true` prevents this behavior and forces `null` and `undefined` values to be explicitly handled.
