# OOP

Object-oriented programming (OOP) is a computer programming model that organizes software design around data, or objects, rather than functions and logic.

# primitive vs non-primitive

primitive
The predefined data types provided by javascript language are known as primitive data types. They are also called built in data types.
eg. boolean, number, string, symbol, undefined, null

The data types that are derived from primitive data types of the JavaScript language are known as non-primitive data types. It is also known as derived data types or reference data types.
eg. Object

# Hoisting

Hoisting is JavaScript's default behavior of moving variable declarations to the top.
variables declared using `var` and function declarations are hoisted by default. That means they can be accessed before declaration.

# Closures

A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time.

# Event loop

### Stack -

This is where all your javascript code gets pushed and executed one by one as the interpreter reads your program, and gets popped out once the execution is done. if your statement is asynchronous then that code gets forwarded to Event table, this table is responsible for moving your asynchronous code to callback/event queue after relevant time.

### Heap -

This is where all the memory allocation happens for your variables, that you have defined in your program.

### Callback Queue -

This is where your asynchronous code gets pushed to, and waits for the execution.

### Event Loop -

Then comes the Event Loop, which keeps running continuously and checks the Main stack, if it has any frames to execute, if not then it checks Callback queue, if Callback queue has codes to execute then it pops the message from it to the Main Stack for the execution.

### Job Queue -

Apart from Callback Queue, browsers have introduced one more queue which is “Job Queue”, reserved only for new Promise() functionality. So when you use promises in your code, you add .then() method, which is a callback method. These `thenable` methods are added to Job Queue once the promise has returned/resolved, and then gets executed.
All `thenable` callbacks of the promise are called first, then the `setTimeout` callback is called.Job Queue has high priority in executing callbacks, if event loop tick comes to Job Queue, it will execute all the jobs in job queue first until it gets empty, then will move to callback queue.

🎆Your asynchronous code will run after “Main Stack” is done with all the task execution.

🎆That is the good part: Your current statements/functions in the stack will run to completion. Async code can not interrupt them. Once your async code is ready to execute, it will wait for main stack to be empty.

🎆That also means that it is not guaranteed that your `setTimeout` or any other async code will run exactly after the time that you have specified. That time is the minimum time after which your code will executed, it can be delayed if Main stack is busy executing existing code.

🎆If you use `0ms` time in your `setTimeout`, it won’t run immediately (If main stack is busy).

```js
setTimeout(function () {
  console.log('Message 1')
}, 0)

console.log('Message 2')
```

In above example, the first output will be “Message 2”, then “Message 1”, even though the `setTimeout` is set to run after 0 millisecond. Once the browser encounters the `setTimeout` it pops it from main Stack to Callback Queue, where it waits for main stack to finish the second `console.log`, then `setTimeout` gets back to main Stack, and runs the first `console.log`.

If you are doing too much heavy computation, then it will make the browser unresponsive, because your main thread is blocked and can not process any other task. So user will be unable to do any click on your webpage. That’s when Browser throws “Script is taking too much time to execute” error, and gives you option to “kill the script” or “wait” for it.

# Async js

By default javascript runs code in a non-blocking way. This means that code which takes some time to finish (like accessing API or reading content from the local file system etc.) is being executed in the background in parallel code execution is continued. this behavior is known as asynchronous programming.
Because js is executed in non-blocking way you have to take additional measures(Callbacks, Promises, Async fns) to deal with that code if you rely on the result before continue next tasks.
Synchronous means execution happens step by step. Some tasks can block main thread if they take some time to finish that's why they are made asynchronous(non-blocking).

# Promises

A Promise is an object representing the eventual completion or failure of an asynchronous operation
Promises are used to handle asynchronous operation in js.
Promises are thenable objects used to handle async operations in our code.

```javascript
Promise.then(
  res => {},
  error => {}
) // error callback executes only if server returns error

Promise.then(res => {}).catch(err => {}) // error callback executes in case of server returns error as well as something goes wrong inside promise res callback body
```

# Strict mode

`'use strict'`

The `'use strict'` directive was new in ECMAScript version 5.
It is not a statement, but a literal expression, ignored by earlier versions of JavaScript.
The purpose of `'use strict'` is to indicate that the code should be executed in strict mode.
default is sloppy mode.

Rules:
Using a variable, without declaring it, is not allowed
Using an object, without declaring it, is not allowed
Deleting a variable (or object) or function is not allowed
Duplicating a parameter name is not allowed
Writing to a read-only property is not allowed
Writing to a get-only property is not allowed
Deleting an un-deletable property is not allowed

# JSON (JavaScript Object Notation)

JSON is a format for storing and transporting data.
JSON is text, and text can be transported anywhere, and read by any programming language.

In JSON, values must be one of the following data types:

a string
a number
an object (containing valid JSON values)
an array
a boolean
null

JSON values cannot be one of the following data types:

a function
a date
undefined

# JS expression & statements

At its simplest terms, expressions are evaluated to produce a value. On the other hand, statements are executed to make something happen.

3+2 //expression
let a = 3; //statement

Fixed values are called Literals.
Variable values are called Variables.

# defer,async

If the defer attribute is set, it specifies that the script is downloaded in parallel to parsing the page, and executed after the page has finished parsing.
If async is present: The script is downloaded in parallel to parsing the page, and executed as soon as it is available (before parsing completes)
If neither async or defer is present: The script is downloaded and executed immediately, blocking parsing until the script is completed

# let vs var

var

function scope
cannot be re-declared
must declared before use (hoisted)

let

block scope
can be re-declared
can be declared after use

# Geolocation Api

if (navigator.geolocation) {
navigator.geolocation.getCurrentPosition((position)=>{
console.log(position.coords.latitude)
console.log(position.coords.longitude)
})
} else {
x.innerHTML = "Geolocation is not supported by this browser.";
}

# JS Date

```javascript
new Date()
new Date(year, month, day, hours, minutes, seconds, milliseconds)
new Date(milliseconds) // milliseconds since 1 Jan 1970
new Date(date string)
```

Note: JavaScript counts months from 0 to 11:
January = 0.
December = 11.

`getFullYear, getDate, getDay, getMilliseconds, getSeconds, getHours, getMinutes, getMonth`

The `getTime` method returns the number of milliseconds, since the 1 Jan 1970

The `Date.parse` method parses a string representation of a date, and returns the number of milliseconds since January 1, 1970

```js
Date.parse(dateString)
```

The static `Date.now` method returns the number of milliseconds elapsed since January 1, 1970

# Call Stack

The mechanism the JS interpreter uses to keep track of its place in a script that calls multiple functions. How JS "knows" what function is currently being run and what functions are called from within that function etc. Browsers come with Web APIs that are able to handle certain tasks in the background (like making requests or setTimeout)The JS call stack recognizes these Web API functions and passes them off to the browser to take care of Once the browser finishes those tasks, they return and are pushed onto the stack as a callback.

# Authentication, Authorization -

Authentication - Process of verifying who a particular user is
Authorization - Verifying what a specific user has access to.

# functions -

Functions allow us to write reusable, modular code We define a "chunk" of code that we can then execute at a laterpoint.

# method -

method is function which is stored as js object property.

# Higher Order Functions -

functions that accepts function as argument or return other functions.

# Compiler vs Interpreter -

A compiler translates complete high-level programming code into machine code at once. As the source code is already converted into machine code, the code execution time becomes short. It stores the converted machine code from your source code program on the disk. A compiler takes an enormous time to analyze source code. However, overall compiled programming code runs faster as compared to an interpreter. The compiler generates an output of a program (in the form of an .exe file) that can run separately from the source code program. A compiled program is generated into an intermediate object code, and it further required linking. So there is a requirement for more memory.

An interpreter translates one statement of programming code at a time into machine code. As the source code is interpreted line-by-line, error detection and correction become easy. It never stores the machine code at all on the disk. An interpreter takes less time to analyze source code as compared to a compiler. However, overall interpreted programming code runs slower as compared to the compiler. The interpreter doesn't generate a separate machine code as an output program. So it checks the source code every time during the execution. An interpreted program does not generate an intermediate code. So there is no requirement for extra memory.

# 20. Call, Apply and bind

In simple words, bind creates new function, call and apply executes the function whereas apply expects the parameters in array

```js
function add() {
  console.log(`addition is: ${this.n + this.m}`)
}

const obj = { n: 1, m: 1 }

// We are passing obj as this
add.call(obj, 1, 2)
add.apply(obj, [1, 2])
add.bind(obj, 1, 2)() //* Bind returns new function that you need to execute for result

// output -
// addition is: 2
// addition is: 2
// addition is: 2

// use bind if you want to have a function with required `this` at later point
```

# Pure and Impure functions

The pure functions are the functions whose returned value depends solely on the values of their arguments.
Pure functions do not have any observable side effects, such as network or database calls. The pure functions just calculate the new value. You can be confident that if you call
the pure function with the same set of arguments, you're going to get the same returned value. They are predictable.
Pure functions do not modify the values passed to them.

Impure functions may call the database or the network, they may have side effects, they may operate on the DOM, and they may override the values that you pass to them.
