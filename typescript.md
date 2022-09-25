## Type inference

Typescript can detect the type at runtime

## Type Aliases

Types created using `type` keyword

```ts
type WishList = {
  by: string
  date: Date
}
```

## interfaces

types declared using `interface` keyword.

```ts
interface WishList {
  by: string
  date: Date
}
```

## Union Types

TypeScript’s type system allows you to build new types out of existing ones using a large variety of operators.

```ts
function printId(id: number | string) {
  if (typeof id === 'string') {
    // In this branch, id is of type 'string'
    console.log(id.toUpperCase())
  } else {
    // Here, id is of type 'number'
    console.log(id)
  }
}
```

## Type Assertions

used to inform typescript about elements type.

```ts
const myCanvas = document.getElementById('main_canvas') as HTMLCanvasElement

const myCanvas = <HTMLCanvasElement>document.getElementById('main_canvas')
```

TypeScript only allows type assertions which convert to a more specific or less specific version of a type. This rule prevents “impossible” coercions like:

```ts
const x = 'hello' as number
```

Sometimes this rule can be too conservative and will disallow more complex coercions that might be valid. If this happens, you can use two assertions, first to any (or unknown), then to the desired type:

```ts
const a = expr as any as T
```

## Literal types

```ts
let x: 'hello' = 'hello'
```

By themselves, literal types aren’t very valuable, but by combining literals into unions, you can express a much more useful concept -

```ts
function printText(s: string, alignment: 'left' | 'right' | 'center') {
  // ...
}
printText('Hello, world', 'left')
printText("G'day, mate", 'between') // throws error

function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1
}
```

We can combine literal types with non-literal types -

```ts
interface Options {
  width: number
}
function configure(x: Options | 'auto') {
  // ...
}
configure({ width: 100 })
configure('auto')
configure('automatic') // throws error
```

### Literal Inference

When you initialize a variable with an object, TypeScript assumes that the properties of that object might change values later. For example, if you wrote code like this:

```ts
const obj = { counter: 0 }
if (someCondition) {
  obj.counter = 1
}
```

Now if we have `handleRequest` function like -

```ts
function handleRequest(url: string, method: 'GET' | 'POST') {}
```

If we tried to call it like -

```ts
const req = { url: 'https://example.com', method: 'GET' }
handleRequest(req.url, req.method) // (A) throws error
```

function call at _A_ throws error because inferred type of req.method is string

solution -

1. You can change the inference by adding a type assertion in either location:

```ts
const req = { url: 'https://example.com', method: 'GET' as 'GET' }
handleRequest(req.url, req.method as 'GET')
// Don't use this because this will not error even if req.method is 'GUESS'
```

2. You can use as const to convert the entire object to be type literals:

```ts
const req = { url: 'https://example.com', method: 'GET' as const }
const req = { url: 'https://example.com', method: 'GET' } as const
handleRequest(req.url, req.method)
```

## Non-null Assertion Operator (Postfix!)

TypeScript also has a special syntax for removing null and undefined from a type without doing any explicit checking. Writing ! after any expression is effectively a type assertion that the value isn’t null or undefined:

```ts
function liveDangerously(x?: number | null) {
  console.log(x!.toFixed()) // No error
}
```

## BigInt

`BigInt` is a primitive wrapper object used to represent to represent and manipulate primitive `bigint` values which are too large to be represented by the `number` primitive.

creating big integers:

Big integer can be created with BigInt constructor or using `n` at the end.

```js
let foo: bigint = BigInt(100) // the BigInt function
let bar: bigint = 100n // a BigInt literal

console.log(3.141592 * 10000n) // error
console.log(3145 * 10n) // error
console.log(BigInt(3145) * 10n) // okay!
```

see also -
`Number.MAX_SAFE_INTEGER`
`Number.isSafeInteger()`

## typeof `null` and `NaN`

```ts
typeof null // object
typeof NaN // number
typeof 0n // bigint

function printAll(strs: string | string[] | null) {
  if (typeof strs === 'object') {
    // throws error because null is also object
    for (const s of strs) {
      console.log(s)
    }
  } else if (typeof strs === 'string') {
    console.log(strs)
  }
}

function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === 'object') {
    // also Array.isArray() can be used here
    for (const s of strs) {
      console.log(s)
    }
  } else if (typeof strs === 'string') {
    console.log(strs)
  }
}
```

## Type Predicates

Type predicates are a special return type that signals to the Typescript compiler what type a particular value is. Type predicates are always attached to a function that takes a single argument and returns a boolean. Type predicates are expressed as argumentName is Type.

```ts
interface Cat {
  numberOfLives: number
}
interface Dog {
  isAGoodBoy: boolean
}

function isCat(animal: Cat | Dog): animal is Cat {
  return typeof animal.numberOfLives === 'number'
}
```

For sample function, isCat, is executed at run time just like all other type guards. Since this function returns a boolean and includes the type predicate animal is Cat, the Typescript compiler will correctly cast the animal as Cat if isCat evaluates as true. It will also cast animal as Dog if isCat evaluates as false.

```ts
let animal: Cat | Dog = ...

if (isCat(animal)) {
  // animal successfully cast as a Cat
} else {
  // animal successfully cast as a Dog
}
```

## Discriminated unions

```ts
interface Shape {
  kind: 'circle' | 'square'
  radius?: number
  sideLength?: number
}
```

With declared type as above if we have to define function called `getArea()` which will return area if shape is `circle`, We need to use non-null assertions(`!`) operator to convince typescript that shape will always contain the radius property.

```ts
function getArea(shape: Shape) {
  if (shape.kind === 'circle') {
    return Math.PI * shape.radius! ** 2
  }
}
```

But above code is error-prone if we start to move code around(changing `shape.radius`)

Instead we can narrow down shape and create two types like below -

```ts
interface Circle {
  kind: 'circle'
  radius: number
}

interface Square {
  kind: 'square'
  sideLength: number
}

type Shape = Circle | Square
```

Now we don't need to use non-null assertions.

## Functions

