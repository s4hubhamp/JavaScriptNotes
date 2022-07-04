# Understanding Asynchronous JavaScript

## Synchronous programming

Each line of code is executed step by step, whereas in a asynchronous programming control can be moved to next step without completion or rejection of the current step.

```js
function makeGreeting(name) {
  return `Hello, my name is ${name}!`
}

const name = 'Miriam'
const greeting = makeGreeting(name)
console.log(greeting)
// "Hello, my name is Miriam!"
```

Here, makeGreeting() is a synchronous function because the caller has to wait for the function to finish its work and return a value before the caller can continue.

In above example we've seen the simple Synchronous code. But let's say if we want to set a timeout for 10 seconds synchronously then this will block our main thread completely(since all synchronous code execution happens on the one main thread for one tab). This will result in very bad user experience as user will be not able to perform basic scroll, copy or even click actions for 10 seconds.

For this very purpose web browsers provide WEB-API's to handle these blocking tasks on a different parallel threads. Blocking operations like `setTimeout` and `AJAXRequests` are handed over to WEB-API's

# Asynchronous JavaScript

In a programming world asynchronous means non-blocking.

Think about _async_ as **Don't block the event loop**

## The browser event loop

Each browser tab runs in a single process: the **event loop**. This loop executes browser-related things (so-called tasks) that it is fed via a task queue.

Examples of tasks are:

1. Parsing HTML
2. Executing JavaScript code in script elements
3. Reacting to user input (mouse clicks, key presses, etc.)
4. Processing the result of an asynchronous network request

Items 2–4 are tasks that run JavaScript code, via the engine built into the browser. They terminate when the code terminates. Then the next task from the queue can be executed. The following diagram gives an overview of how all these mechanisms are connected.

[Event Loop Diagram](https://exploringjs.com/es6/images/async----event_loop.jpg)

Event loop got its name because of how it's usually implemented

```js
while (queue.waitForMessage()) {
  queue.processNextMessage()
}
// queue.waitForMessage() waits synchronously for a message to
// arrive (if one is not already available and waiting to be handled).
```

Here is another

```js
while (browser.alive) {
  if (stack.empty && !queue.empty) {
    stack.push(queue.next)
  }
}
```

Event loop can be represented in three basic parts -

1. Stack
2. Heap
3. Queue

### The JavaScript call stack

When a function f calls a function g, g needs to know where to return to (inside f) after it is done. This information is usually managed with a stack, the call stack. Let’s look at an example.

```js
function h(z) {
  // Print stack trace
  console.log(new Error().stack) // (A)
}
function g(y) {
  h(y + 1) // (B)
}
function f(x) {
  g(x + 1) // (C)
}
f(3) // (D)
return // (E)
```

Initially, when the program above is started, the call stack is empty. After the function call f(3) in line D, the stack has one entry:

Location in global scope
After the function call g(x + 1) in line C, the stack has two entries:

Location in f
Location in global scope
After the function call h(y + 1) in line B, the stack has three entries:

Location in g
Location in f
Location in global scope
The stack trace printed in line A shows you what the call stack looks like:

> Error
> at h (stack_trace.js:2:17)
> at g (stack_trace.js:6:5)
> at f (stack_trace.js:9:5)
> at `<global>` (stack_trace.js:11:1)
> Next, each of the functions terminates and each time the top entry is removed from the stack.
> After function f is done, we are back in global scope and the call stack is empty.
> In line E we return and the stack is empty, which means that the program terminates.

- **Each function call creates a new frame(execution context) on stack**
- **Nested functions have access to the parent variables and functions through the closures**

### Heap

Objects are allocated in a heap which is just a name to denote a large (mostly unstructured) region of memory.

### Queue

A JavaScript runtime uses a message queue, which is a list of messages to be processed. Each message has an associated function that gets called to handle the message.

At some point during the event loop, the runtime starts handling the messages on the queue, starting with the oldest one. To do so, the message is removed from the queue and its corresponding function is called with the message as an input parameter. As always, calling a function creates a new stack frame for that function's use.

The processing of functions continues until the stack is once again empty. Then, the event loop will process the next message in the queue (if there is one).

**Queue is a to-do list for the browser sorted by when they entered the queue.**

### Timers

```js
setTimeout(callback, ms)
```

After ms milliseconds, callback is added to the task queue. It is important to note that ms only specifies when the callback is added, not when it actually executed. That may happen much later, especially if the event loop is blocked (as demonstrated later in this chapter).

setTimeout() with ms set to zero is a commonly used work-around to add something to the task queue right away. However, some browsers do not allow ms to be below a minimum (4 ms in Firefox); they set it to that minimum if it is.

### Displaying DOM changes

For most DOM changes (especially those involving a re-layout), the display isn’t updated right away. “Layout happens off a refresh tick every 16ms” and **must be given a chance to run via the event loop**.

### Run-to-completion semantics

JavaScript has so-called run-to-completion semantics: The current task is always finished before the next task is executed. That means that each task has complete control over all current state and doesn’t have to worry about concurrent modification.

Let’s look at an example:

```js
setTimeout(function () {
  // (A)
  console.log('Second')
}, 0)
console.log('First') // (B)
```

#### steps -

- function starting in line A is added to the task queue immediately. But since all code in current call stack isn't completed yet so that callback has to wait until line B executes.
- function in line B executed
- callback is pushed to call stack and executed

That's how above program will always print like -

```css
 First
 Second
```

### Examples for Blocking the loop -

By now we are surely know that, **Any block of Javascript code that enters the main thread will run until completion of last line.**

Consider following two examples -

#### Example 1

```js
console.log('Hi MoM')
let infinite = true
setTimeout(function () {
  infinite = false
}, 1000)
while (infinite) {
  console.log('still running', Date.now())
}
alert('Hey')
```

You might expect to see the alert after 1 second. Right?? But that won't happen.
Let's see the steps that happen after engine sees the above code.
(note - Engine will see this block if code as a `script` tag will direct engine towards the file that contain the code.)

1. Engine sees the code (via the scripts tag)
2. A block of code is moved onto the call stack
3. sees console.log and executes the code after creating it's own execution context on top of the current context(in above case the Global Execution Context)
4. Engine sees the setTimeout and quickly hands it over to Web-API's, then web API's will add this function to the task queue after 1 second.
5. start the while loop
6. infinite=false from the callback waits on the queue
7. infinite still true on the stack
8. Heap sends the function to queue
9. infinite still true on the stack
10. Event Loop puts that function(callback) on hold, as the stack is busy

and so on...

It turns out, the stack is so busy running the while loop, our short function to turn off infinite never gets a chance to get into the stack. That’s a deadlock.

#### Example 2 -

```html
<a id="block" href="">Block for 5 seconds</a>
<p>
  <button>This is a button</button>
</p>

<div id="statusMessage"></div>
<script>
  document.getElementById('block').addEventListener('click', onClick)

  function onClick(event) {
    event.preventDefault()

    setStatusMessage('Blocking...')

    // Call setTimeout(), so that browser has time to display
    // status message
    setTimeout(function () {
      sleep(5000)
      setStatusMessage('Done')
    }, 0)
  }
  function setStatusMessage(msg) {
    document.getElementById('statusMessage').textContent = msg
  }
  function sleep(milliseconds) {
    var start = Date.now()
    while (Date.now() - start < milliseconds);
  }
</script>
```

##### steps

1. Anchor tag get's clicked, and event related to it fired and added to the tasks queue
2. `setStatusMessage` get's called and set's the message to "Blocking..."
3. The callback related to `setTimeout` get's pushed into task queue immediately
4. Somewhere between step 2 and 3 browser get's a chance to repaint display.(Because call stack is empty). And we can see "Blocking..." in the `statusMessage`.
5. After repaint happens callback of `setTimeout` that was waiting in the task queue get's pushed on call stack for execution
6. The `sleep` function get's called and performs while loop for 5 seconds. During this time browser tab becomes unresponsive(user interface doesn’t work) as main thread is busy performing iteration.
7. finally message is set to "Done" and call stack again get's empty.
8. browser repaint display.

<details>
<summary>What will happen if I don't wrap `sleep` inside the `setTimeout` ?</summary>

If we won't wrap `sleep` inside the `setTimeout` function callback, then it will get executed immediately after step 2 and we will only see the repaint after all remaining script gets executed. This way we will directly see the "done" status message since browser will repaint with latest state!

</details>

#### There are multiple ways to handle Asynchronous results -

##### 1. Handling via use of events -

In this pattern for asynchronously receiving results, you create an object for each request and register event handlers with it: one for a successful computation, another one for handling errors. The following code shows how that works with the `XMLHttpRequest` API:

```js
var req = new XMLHttpRequest()

// Initializes the request. Creates the request object
//Third argument is for sending request asynchronously. Default is also true.
req.open('GET', url, true)

req.onload = function () {
  if (req.status == 200) {
    processData(req.response)
  } else {
    console.log('ERROR', req.statusText)
  }
}

req.onerror = function () {
  console.log('Network Error')
}

req.send() // Add request to task queue(Sends the request)
```

Note that the last line doesn’t actually perform the request, it adds it to the task queue. Therefore, you could also call that method right after open(), before setting up onload and onerror. Things would work the same, due to JavaScript’s run-to-completion semantics.

##### 2. Handling via callbacks

If you handle asynchronous results via callbacks, you pass callback functions as trailing parameters to asynchronous function or method calls.

The following is an example in Node.js. We read the contents of a text file via an asynchronous call to fs.readFile():

```js
// Node.js
fs.readFile('myfile.txt', { encoding: 'utf8' }, function (error, text) {
  // (A)
  if (error) {
    // ...
  }
  console.log(text)
})
```

If readFile() is successful, the callback in line A receives a result via the parameter text. If it isn’t, the callback gets an error (often an instance of Error or a sub-constructor) via its first parameter.

The same code in classic functional programming style would look like this:

```js
// Functional
readFileFunctional(
  'myfile.txt',
  { encoding: 'utf8' },
  function (text) {
    // success
    console.log(text)
  },
  function (error) {
    // failure
    // ...
  }
)
```

**Any JavaScript code that get executed will be executed on the single thread for that particular tab, AlongSide user interactions(copy,click).**

## Async Function

Async function is a function declared with an `async` keyword, and the await keyword permitted within it. The async functions enables asynchronous, promise based behavior to be written in a cleaner style, avoiding need to explicitly configure promise chains.

```js
// function expression
async function name() {
  await somePromise()
}

// function declaration
let foo = async function () {}
let foo = async () => {}
```

Async functions always return a promise. If the return value of an async function is not explicitly a promise, it will be implicitly wrapped in a promise.

The body of an async function can be thought of as being split by zero or more await expressions. Top-level code, up to and including the first await expression (if there is one), is run synchronously. In this way, an async function without an await expression will run synchronously. If there is an await expression inside the function body, however, the async function will always complete asynchronously.

```js
async function foo() {
  const result1 = await new Promise(resolve => setTimeout(() => resolve('1')))
  const result2 = await new Promise(resolve => setTimeout(() => resolve('2')))
}
foo()
```

- The first line of the body of function foo is executed synchronously, with the await expression configured with the pending promise. Progress through foo is then suspended and control is yielded back to the function that called foo.

- Some time later, when the first promise has either been fulfilled or rejected, and call stack is empty control moves back into foo. The result of the first promise fulfillment (if it was not rejected) is returned from the await expression. Here 1 is assigned to result1. Progress continues, and the second await expression is evaluated. Again, progress through foo is suspended and control is yielded.

- Some time later, when the second promise has either been fulfilled or rejected, control re-enters foo. The result of the second promise resolution is returned from the second await expression. Here 2 is assigned to result2. Control moves to the return expression (if any). The default return value of undefined is returned as the resolution value of the current promise.

[MDN Example on async_functions_and_execution_order](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function#async_functions_and_execution_order)

## Promises

A Promise represents eventual completion or rejection of an asynchronous operation.
Promises are alternatives to callbacks for delivering the results of an asynchronous operation.

```js
asyncFunc1() // (A)
  .then(result1 => {
    // Use result1
    return asyncFunction2() // (B)
  })
  .then(result2 => {
    // (C)
    // Use result2
  })
  .catch(error => {
    // Handle errors of asyncFunc1() and asyncFunc2()
  })
```

First `.then` gets the result of asyncFunc1 upon settlement or if it gets rejected then `.catch` gets called with error.

Note that even if we decided to return nothing anything from previous `.then` at (B) the `then` callback in chain still gets executed with `result2` as undefined.

If you chain asynchronous function calls via then(), they are executed sequentially, one at a time:

```js
asyncFunc1().then(() => asyncFunc2())
```

If you don't do that and call all of them immediately, they are basically executed in parallel.

```js
asyncFunc1()
asyncFunc2()
```

`Promise.all()` enables you to be notified once all results are in (a join in Unix process terminology). Its input is an Array of Promises, its output a single Promise that is fulfilled with an Array of the results.

```js
Promise.all([asyncFunc1(), asyncFunc2()])
  .then(([result1, result2]) => {
    //
  })
  .catch(err => {
    // Receives first rejection among the Promises
  })
```

A promise is always in one of the following states:

1. pending
2. fulfilled
3. rejected
4. settled (A promise is settled exactly once and remains unchanged.)

Compared to callbacks as continuations, Promises have the following advantages:

1. No inversion of control: similarly to synchronous code, Promise-based functions return results, they don’t (directly) continue – and control – execution via callbacks. That is, the caller stays in control.
2. Chaining is simpler: If the callback of then() returns a Promise (e.g. the result of calling another Promise-based function) then then() returns that Promise (how this really works is more complicated and explained later). As a consequence, you can chain then() method calls:

```js
asyncFunction1(a, b)
  .then(result1 => {
    console.log(result1)
    return asyncFunction2(x, y)
  })
  .then(result2 => {
    console.log(result2)
  })
```

3. Composing asynchronous calls (loops, mapping, etc.): is a little easier, because you have data (Promise objects) you can work with.
4. Error handling: As we shall see later, error handling is simpler with Promises, because, once again, there isn’t an inversion of control. Furthermore, both exceptions and asynchronous errors are managed the same way.
5. Cleaner signatures: With callbacks, the parameters of a function are mixed; some are input for the function, others are responsible for delivering its output. With Promises, function signatures become cleaner; all parameters are input.
6. Standardized: Prior to Promises, there were several incompatible ways of handling asynchronous results (Node.js callbacks, XMLHttpRequest, IndexedDB, etc.). With Promises, there is a clearly defined standard: ECMAScript 6. ES6 follows the standard Promises/A+ [1]. Since ES6, an increasing number of APIs is based on Promises.

With Node.js-style callbacks, reading a file asynchronously looks like this:

```js
fs.readFile('config.json', function (error, text) {
  if (error) {
    console.error('Error while reading config file')
  } else {
    try {
      const obj = JSON.parse(text)
      console.log(JSON.stringify(obj, null, 4))
    } catch (e) {
      console.error('Invalid JSON in file')
    }
  }
})
```

with Promises, the same functionality is used like this:

```js
readFilePromisified('config.json')
  .then(function (text) {
    // (A)
    const obj = JSON.parse(text)
    console.log(JSON.stringify(obj, null, 4))
  })
  .catch(function (error) {
    // (B)
    // File read error or JSON SyntaxError
    console.error('An error occurred', error)
  })
```

There are still callbacks, but they are provided via methods that are invoked on the result (then() and catch()). The error callback in line B is convenient in two ways: First, it’s a single style of handling errors (versus if (error) and try-catch in the previous example). Second, you can handle the errors of both readFilePromisified() and the callback in line A from a single location.

`catch()` is simply more convenient(and recommended) alternative to calling `then()`. That is following two invocations are equal.

```js
promise.then(result, error => {
  /* rejection */
  // (A)
})

promise.catch(error => {
  /* rejection */
})
```

Note that if we reject return `Promise.reject` from line A the error will not get caught if we don't have `.catch`

**A Promise is both a container for a value and an event emitter.**

Normal event emitters specialize in delivering multiple events, starting as soon as you register.

In contrast, Promises specialize in delivering exactly one value and come with built-in protection against registering too late: the result of a Promise is cached and passed to event listeners that are registered after the Promise was settled.

### Timing out the promise

```js
function timeout(ms, promise) {
  return new Promise(function (resolve, reject) {
    promise.then(resolve)
    setTimeout(function () {
      reject(new Error('Timeout after ' + ms + ' ms')) // (A)
    }, ms)
  })
}
```

### `Promise.resolve()`

`Promise.resolve` is used to convert any value (Promise, thenable or other) to a promise. In fact, it is used by `promise.all()` and `promise.race()` to convert arrays of arbitrary values to array of Promises.

```js
const val = Promise.resolve('Hello')
val instanceof Promise // true
val.then(res => console.log(res)) // logs -> 'Hello'

const val2 = Promise.resolve(() => {})
val2.then(res => console.log(res)) // logs -> () => {}
```

### `Promise.reject()`

Promise.reject(err) returns a Promise that is rejected with err:

```js
const myError = new Error('Problem!')
Promise.reject(myError).catch(err => console.log(err === myError)) // true
```

### Handling exceptions in Promise-based functions

```js
function asyncFunc() {
  doSomethingSync() // We are doing something sync (A)
  return doSomethingAsync().then(result => {})
}
```

If an exception is thrown in line A then the whole function throws an exception. There are two solutions to this problem.

You can catch exceptions and return them as rejected Promises:

```js
function asyncFunc() {
  try {
    doSomethingSync()
    return doSomethingAsync().then(result => {})
  } catch (err) {
    return Promise.reject(err)
  }
}
```

executing the sync code inside a callback

```js
function asyncFunc() {
  return Promise.resolve() // Simply creating promise for chain .then
    .then(() => {
      doSomethingSync()
      return doSomethingAsync()
    })
    .then(result => {})
}
```

```js
function asyncFunc() {
  return new Promise((resolve, reject) => {
    doSomethingSync()
    resolve(doSomethingAsync())
  }).then(result => {})
}
```

### `Promise.all()`

`Promise.all(iterable)` takes an iterable over Promises (thenables and other values are converted to Promises via Promise.resolve()). Once all of them are fulfilled, it fulfills with an Array of their values. If iterable is empty, the Promise returned by all() is fulfilled immediately.

```js
Promise.all([asyncFunc1(), asyncFunc2()])
  .then(([result1, result2]) => {
    // ···
  })
  .catch(err => {
    // Receives first rejection among the Promises
    // ···
  })
```

### `Promise.race()`

Promise.race(iterable) takes an iterable over Promises (thenables and other values are converted to Promises via Promise.resolve()) and returns a Promise P. The first of the input Promises that is settled passes its settlement on to the output Promise. If iterable is empty then the Promise returned by race() is never settled.

```js
Promise.race([
    httpGet('http://example.com/file.txt'),
    delay(5000).then(function () {
        throw new Error('Timed out')
    });
])
.then(function (text) { ··· })
.catch(function (reason) { ··· });
```

## Optimization and priority decisions

Event loop gets input from several parts of the browser, such as JavaScript callbacks, network requests, calculating styles, layout and rendering/painting.

These browser tasks are optimized by browsers differently. They sometimes wait in the background, giving space to microtasks and JavaScript tasks before getting attention from the Event loop.

```js
console.log('script start') // (A)

// (B)
setTimeout(function () {
  console.log('setTimeout')
}, 0)

Promise.resolve()
  // (C)
  .then(function () {
    console.log('promise1')
  })
  // (D)
  .then(function () {
    console.log('promise2')
  })

console.log('script end') // (E)

// Output ->
// script start
// script end
// promise1
// promise2
// setTimeout
```

steps ->

1. _A_ logs "script start"
2. _B_ gets handed over to different web API thread, callback gets added to the **tasks** queue immediately
3. Promise resolved immediately and callback _C_ gets added in a **microtasks** queue
4. `then()` at _C_ also returns promise which gets resolved immediately and callback _D_ gets added in a **microtasks** queue
   > Note that WebAPI's add callbacks related to steps 2,3 and 4 in the background
5. _A_ logs "script end"
6. First all callbacks for microtasks gets executed from **microtasks queue** and later tasks from **tasks queue**

NOTE -> Between steps 5 and 6 browser may decide to paint the screen.

Refer above example in more detail at - [jakearchibald.com](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)

Another example from Jake - [here](https://www.youtube.com/watch?v=cCOL7MC4Pl0)

### requestAnimationFrame -

The `window.requestAnimationFrame()` method tells the browser that you wish to perform an animation and requests that the browser calls a specified function to update an animation before the next repaint. The method takes a callback as an argument to be invoked before the repaint.

We want to animate box on button click, going to 1000px right when we click and then coming left by 500px

1. Without using requestAnimationFrame -

```js
button.addEventListener('click', () => {
  box.style.transform = 'translateX(1000px)'
  box.style.transition = 'transform 1s ease-in-out'
  box.style.transform = 'translateX(500px)'
})
```

since all Javascript on stack must gets executed before rendering happens the value that gets picked by browser while rendering is the final value `'translateX(500px)'`

Note that we are not using `'translateX(-500px)'` because we are just setting final position we want box to be which is 500px right.

2. Using requestAnimationFrame -

```js
button.addEventListener('click', () => {
  box.style.transform = 'translateX(1000px)'
  box.style.transition = 'transform 1s ease-in-out'

  requestAnimationFrame(() => {
    box.style.transform = 'translateX(500px)'
  })
})
```

In above example callback of `requestAnimationFrame()` gets called before rendering happens. Since we have exactly one `requestAnimationFrame()` call we are registering our callback to the immediate render after stack get's empty. So our box will have `translateX(500px)` in the transform at the time of rendering(painting) the element.

To solve above problem we need to nest `requestAnimationFrame()` inside the `requestAnimationFrame()` so it gets called after first paint.

```js
button.addEventListener('click', () => {
  box.style.transform = 'translateX(1000px)' // (A)
  box.style.transition = 'transform 1s ease-in-out'

  requestAnimationFrame(() => {
    // Might include line (A) here
    requestAnimationFrame() => {
      box.style.transform = 'translateX(500px)'
    }
  })
})
```

**Tasks queues - Tasks happen one at a time.**

**Animation Callbacks queue(i.e requestAnimationFrame) - All current registered callbacks happen until completion except one that is queued while processing current animation callbacks, those ones will be executed before next render**

**Microtasks queue - Microtasks are processed to completion including any additional queued items. i.e - If you are adding items to the queue as quickly as you are processing them you are processing microtasks forever. The event loop cannot continue until all tasks are completed.**
