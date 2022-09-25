# SPA

A single-page application is an application that loads a single HTML page and all the necessary assets (such as JavaScript and CSS) required for the application to run. Any interactions with the page or subsequent pages do not require a round trip to the server which means the page is not reloaded.

# React is Declarative

with react you will not tell react that a certain HTML Element should be created and inserted in specific place on UI as vanilla JS, instead with react you will always define the desired end state depending on different conditions and after that it's React job which element should be added and not added.

# JSX

JSX stands for JavaScript XML(Extensible Markup Language).
JSX allows us to write HTML in React.
JSX makes it easier to write and add HTML in React.

JSX allows us to write HTML elements in JavaScript and place them in the DOM without any `createElement` and/or `appendChild` methods.

JSX converts HTML tags into react elements.

```jsx
const el = <h1>I Love JSX!</h1> //JSX
const el = React.createElement(
  'div',
  {},
  React.createElement('h1', {}, 'I do not use JSX!'),
  '...'
) // without JSX
ReactDOM.render(el, document.getElementById('root'))
```

# Component

React Component is just a function which returns JSX

# State

State of a component is an object that holds some information that may change over the lifetime of the component.

`useState` hook registers some state(data) for a component in which it is called, it registers for specific component instance, And whenever state changes react reloads the component with updated state.

# How Component Functions Are Executed

First react goes to index.js and sees app component function then it executes that function(creates component).
when react sees component function in jsx it executes that function. after executing all the component functions and then it evaluates the overall
result and translates it to DOM instructions.

however React don't repeats that that's why we need state.

# Hooks

Hooks are a new addition in React 16.8. They let you use state and other React features without writing a class.
Hooks let you use more of React’s features without classes.

### `useEffect`

Every time a component is rendered or re-rendered then useEffect function is called.
A side effect is any execution that affects something outside the scope of the function being executed. For example, if a function modifies a global variable, then that function is introducing a side effect — the global variable doesn’t belong to the scope of the current function.

Some examples of side effects in React components are:

Making asynchronous API calls for data
Setting a subscription to an observable
Manually updating the DOM element
Updating global variables from inside a function

```js
useEffect(() => {}) // useEffect will be called after each re-rendering of component.
useEffect(() => {}, []) // useEffect will be called only once at initial rendering of component.
useEffect(() => {}, [d]) // useEffect will be called after d-changes.
```

### `useReducer`

```jsx
const [state, dispatchFn] = useReducer(reducerFn, initialState, initFn)
```

# Two way data binding

```jsx
<input type='text' value={enteredTitle} onChange={titleChangeHandler} />
```

# 9. Keys

Key is the way in which react separate each item apart. so if we do not associate key and add the item as the first element to the list then react does not know each element uniquely and therefore it has to rely on lists length and then update each element in the DOM. However if we associate unique key to each list item react will know previous unchanged elements in DOM and it will only add one element at the start of the list.

We can use keys when we wan't to re-render a specific component on some state change.

```jsx
<ErrorBoundary key={state.name}></ErrorBoundary>
```

# Adding dynamic inline styles

```jsx
<div style={{ height: barHeight, backgroundColor: 'red' }}></div>
```

# Adding dynamic classes

```jsx
<div className={`form-control ${!isValid ? 'invalid' : ''}`}></div>
```

# Difference when assigning default state to Redux store vs inside React context

When we create a store in redux then our reducer function has to have a default state but when we use useReducer then we must pass default state
as a second parameter to useReducer function. React will not run reducer of useReducer at first time automatically whereas Redux does.

```js
Redux.createStore((state = {count: 0}, action) => {
switch (action.type) {
case "..."
break;
default: {
return state
}
}
})

useReducer(reducer, {count: 0})
```

# Testing

Types of tests -

1. Unit tests: Test the individual building blocks in isolation.
2. Integration tests: Test the combination of multiple building blocks.
3. End2end tests(e2e): Tests complete scenarios in your app as the user would experience them.
