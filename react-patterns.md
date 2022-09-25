# Context Module Functions

Let's take a look at an example of a simple context and a reducer combo:

```js
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({ step = 1, initialCount = 0, ...props }) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return { ...state, count: state.count + change }
        }
        case 'decrement': {
          return { ...state, count: state.count - change }
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    { count: initialCount }
  )

  const value = [state, dispatch]
  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

export { CounterProvider, useCounter }
// src/screens/counter.js
import { useCounter } from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  const increment = () => dispatch({ type: 'increment' })
  const decrement = () => dispatch({ type: 'decrement' })
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
    </div>
  )
}
// src/index.js
import { CounterProvider } from 'context/counter'

function App() {
  return (
    <CounterProvider>
      <Counter />
    </CounterProvider>
  )
}
```

In above example you can see that user of our Counter context needs to handle the increment and decrement functions which call dispatch.

The first thing we can do to improve upon above example is to keep the all possible action functions inside the context itself. You'll notice that we have to put it in React.useCallback so we can list our "helper" functions in dependency lists):

```js
const increment = React.useCallback(
  () => dispatch({ type: 'increment' }),
  [dispatch]
)
const decrement = React.useCallback(
  () => dispatch({ type: 'decrement' }),
  [dispatch]
)
const value = { state, increment, decrement }
return <CounterContext.Provider value={value} {...props} />

// now users can consume it like this:

const { state, increment, decrement } = useCounter()
```

From above example we can see that we need to memoize all the functions in order to use them into dependency arrays.

As Dan Abramov says -
Helper methods are object junk that we need to recreate and compare for no purpose other than superficially nicer looking syntax.

What Dan recommends (and what Facebook does) is pass dispatch as we had originally. And to solve the annoyance we were trying to solve in the first place, they use importable "helpers" that accept dispatch. Let's take a look at how that would look:

```js
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({ step = 1, initialCount = 0, ...props }) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return { ...state, count: state.count + change }
        }
        case 'decrement': {
          return { ...state, count: state.count - change }
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    { count: initialCount }
  )

  const value = [state, dispatch]

  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

const increment = dispatch => dispatch({ type: 'increment' })
const decrement = dispatch => dispatch({ type: 'decrement' })

export { CounterProvider, useCounter, increment, decrement }
// src/screens/counter.js
import { useCounter, increment, decrement } from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={() => decrement(dispatch)}>-</button>
      <button onClick={() => increment(dispatch)}>+</button>
    </div>
  )
}
```

# Compound Components

Compound components are generally two components which work together to accomplish a useful task. Typically one component is the parent, and the other is the child.

```js
const allowedTypes = [ToggleOff, ToggleOn, ToggleButton]

function Toggle({ children }) {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)

  return React.Children.map(children, child => {
    if (allowedTypes.includes(child.type)) {
      // We have a function component which is a part of allowedTypes
      const childCopy = React.cloneElement(child, { toggle, on })
      return childCopy
    } else if (typeof child.type === 'string') {
      return child
    } else {
      // We have a function component which is not a part of allowedTypes
      // We can throw error or simply return the component
      return child
    }
  })
}

const ToggleOn = ({ on, children }) => (on ? children : null)
const ToggleOff = ({ on, children }) => (on ? null : children)
const ToggleButton = props => <Switch on={props.on} onClick={props.toggle} />

function App() {
  return (
    <div>
      <Toggle>
        <ToggleOn>The button is on</ToggleOn>
        <ToggleOff>The button is off</ToggleOff>
        <span>Hi</span>
        <ToggleButton />
      </Toggle>
    </div>
  )
}

export default App
```

As seen in above example `ToggleOn`, `ToggleOff` and `ToggleButton` are the child components that are rendering
some content depending upon the state in the parent component `Toggle`.
`Toggle` consists all the needed state and state updaters. We are passing the state to each child as a prop.
Now user of `Toggle` component will decide how he wants the UI to look like.
i.e - He may wants only button and don't really care about the messages like - the button is on and off...
Then User can only specify only needed children like -

```js
function App() {
  return (
    <div>
      <Toggle>
        <ToggleButton />
      </Toggle>
    </div>
  )
}
```

Drawback -

The Components that rely on state inside parent component should be direct children of parent. You can't have

```jsx
<Toggle>
  <div>
    <ToggleOn>The button is on</ToggleOn>
    <ToggleOff>The button is off</ToggleOff>
  </div>
  <span>Hi</span>
  <ToggleButton />
</Toggle>
```

We can use `context` to overcome above drawback(**Flexible Compound Components**).
https://kentcdodds.com/blog/compound-components-with-react-hooks

# Flexible Compound Components

The Flexible Compound Components Pattern only differs from the previous plain compound components pattern in that it uses React context. You should use this version of the pattern more often.

```js
const ToggleContext = React.createContext()
ToggleContext.displayName = 'ToggleContext'

function Toggle({ children }) {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)

  return (
    <ToggleContext.Provider value={{ on, toggle }}>
      {children}
    </ToggleContext.Provider>
  )
}

function useToggle() {
  const context = React.useContext(ToggleContext)
  if (context === undefined) {
    throw new Error('useToggle must be used within a <Toggle />')
  }
  return context
}

function ToggleOn({ children }) {
  const { on } = useToggle()
  return on ? children : null
}

function ToggleOff({ children }) {
  const { on } = useToggle()
  return on ? null : children
}

function ToggleButton({ ...props }) {
  const { on, toggle } = useToggle()
  return <Switch on={on} onClick={toggle} {...props} />
}

function App() {
  return (
    <div>
      <Toggle>
        <ToggleOn>The button is on</ToggleOn>
        <ToggleOff>The button is off</ToggleOff>
        <div>
          <ToggleButton />
        </div>
      </Toggle>
    </div>
  )
}

export default App
```

# Prop Collections and Getters

The Prop Collections and Getters Pattern allows your hook to support common use cases for UI elements people build with your hook.

In typical UI components, you need to take accessibility into account. For a button functioning as a toggle, it should have the aria-pressed attribute set to true or false if it's toggled on or off. In addition to remembering that, people need to remember to also add the onClick handler to call toggle.

Lots of the reusable/flexible components and hooks that we'll create have some common use-cases and it'd be cool if we could make it easier to use our components and hooks the right way without requiring people to wire things up for common use cases.

```js
const callAll =
  (...fns) =>
  (...args) =>
    fns.forEach(fn => fn?.(...args))

function useToggle() {
  const [on, setOn] = React.useState(false)
  const toggle = () => setOn(!on)

  function getTogglerProps({ onClick, ...props } = {}) {
    return {
      'aria-pressed': on,
      onClick: callAll(onClick, toggle),
      ...props,
    }
  }

  return {
    on,
    toggle,
    getTogglerProps,
  }
}

function App() {
  const { on, getTogglerProps } = useToggle()
  return (
    <div>
      <Switch {...getTogglerProps({ on })} />
      <hr />
      <button
        {...getTogglerProps({
          'aria-label': 'custom-button',
          onClick: () => console.info('onButtonClick'),
          id: 'custom-button-id',
        })}>
        {on ? 'on' : 'off'}
      </button>
    </div>
  )
}

export default App
```

# State Reducer

The State Reducer Pattern inverts control over the state management of your hook and/or component to the developer using it so they can control the state changes that happen when dispatching events.

we want to prevent the toggle from updating the toggle state after it's been clicked 4 times in a row before resetting. We could easily add that logic to our reducer, but instead we're going to apply a computer science pattern called "Inversion of Control" where we effectively say: "Here you go! You have complete control over how this thing works. It's now your responsibility."

```js
// state reducer
// 💯 state reducer action types
// http://localhost:3000/isolated/final/05.extra-2.js

import * as React from 'react'
import { Switch } from '../switch'

const callAll =
  (...fns) =>
  (...args) =>
    fns.forEach(fn => fn?.(...args))

const actionTypes = {
  toggle: 'toggle',
  reset: 'reset',
}

function toggleReducer(state, { type, initialState }) {
  switch (type) {
    case actionTypes.toggle: {
      return { on: !state.on }
    }
    case actionTypes.reset: {
      return initialState
    }
    default: {
      throw new Error(`Unsupported type: ${type}`)
    }
  }
}

function useToggle({ initialOn = false, reducer = toggleReducer } = {}) {
  const { current: initialState } = React.useRef({ on: initialOn })
  const [state, dispatch] = React.useReducer(reducer, initialState)
  const { on } = state

  const toggle = () => dispatch({ type: actionTypes.toggle })
  const reset = () => dispatch({ type: actionTypes.reset, initialState })

  function getTogglerProps({ onClick, ...props } = {}) {
    return {
      'aria-pressed': on,
      onClick: callAll(onClick, toggle),
      ...props,
    }
  }

  function getResetterProps({ onClick, ...props } = {}) {
    return {
      onClick: callAll(onClick, reset),
      ...props,
    }
  }

  return {
    on,
    reset,
    toggle,
    getTogglerProps,
    getResetterProps,
  }
}
// export {useToggle, toggleReducer, actionTypes}

// import {useToggle, toggleReducer, actionTypes} from './use-toggle'

function App() {
  const [timesClicked, setTimesClicked] = React.useState(0)
  const clickedTooMuch = timesClicked >= 4

  function toggleStateReducer(state, action) {
    if (action.type === actionTypes.toggle && clickedTooMuch) {
      return { on: state.on }
    }
    return toggleReducer(state, action)
  }

  const { on, getTogglerProps, getResetterProps } = useToggle({
    reducer: toggleStateReducer,
  })

  return (
    <div>
      <Switch
        {...getTogglerProps({
          disabled: clickedTooMuch,
          on: on,
          onClick: () => setTimesClicked(count => count + 1),
        })}
      />
      {clickedTooMuch ? (
        <div data-testid='notice'>
          Whoa, you clicked too much!
          <br />
        </div>
      ) : timesClicked > 0 ? (
        <div data-testid='click-count'>Click count: {timesClicked}</div>
      ) : null}
      <button {...getResetterProps({ onClick: () => setTimesClicked(0) })}>
        Reset
      </button>
    </div>
  )
}

export default App
```

# Control Props

The Control Props pattern allows users to completely control state values within your component. This differs from the state reducer pattern in the fact that you can not only change the state changes based on actions dispatched but you also can trigger state changes from outside the component or hook as well.

Sometimes, people want to be able to manage the internal state of our component from the outside. The state reducer allows them to manage what state changes are made when a state change happens, but sometimes people may want to make state changes themselves. We can allow them to do this with a feature called "Control Props."

This concept is basically the same as controlled form elements in React that you've probably used many times: 📜 https://reactjs.org/docs/forms.html#controlled-components

```js
function MyCapitalizedInput() {
  const [capitalizedValue, setCapitalizedValue] = React.useState('')

  return (
    <input
      value={capitalizedValue}
      onChange={e => setCapitalizedValue(e.target.value.toUpperCase())}
    />
  )
}
```

In this case, the "component" that's implemented the "control props" pattern is the <input />. Normally it controls state itself (like if you render <input /> by itself with no value prop). But once you add the value prop, suddenly the <input /> takes the back seat and instead makes "suggestions" to you via the onChange prop on the state updates that it would normally make itself.

This flexibility allows us to change how the state is managed (by capitalizing the value), and it also allows us to programmatically change the state whenever we want to, which enables this kind of synchronized input situation:

```js
function MyTwoInputs() {
  const [capitalizedValue, setCapitalizedValue] = React.useState('')
  const [lowerCasedValue, setLowerCasedValue] = React.useState('')

  function handleInputChange(e) {
    setCapitalizedValue(e.target.value.toUpperCase())
    setLowerCasedValue(e.target.value.toLowerCase())
  }

  return (
    <>
      <input value={capitalizedValue} onChange={handleInputChange} />
      <input value={lowerCasedValue} onChange={handleInputChange} />
    </>
  )
}
```

```js
const callAll =
  (...fns) =>
  (...args) =>
    fns.forEach(fn => fn?.(...args))

const actionTypes = {
  toggle: 'toggle',
  reset: 'reset',
}

function toggleReducer(state, { type, initialState }) {
  switch (type) {
    case actionTypes.toggle: {
      return { on: !state.on }
    }
    case actionTypes.reset: {
      return initialState
    }
    default: {
      throw new Error(`Unsupported type: ${type}`)
    }
  }
}

function useControlledSwitchWarning(
  controlPropValue,
  controlPropName,
  componentName
) {
  const isControlled = controlPropValue != null
  const { current: wasControlled } = React.useRef(isControlled)

  React.useEffect(() => {
    warning(
      !(isControlled && !wasControlled),
      `\`${componentName}\` is changing from uncontrolled to be controlled. Components should not switch from uncontrolled to controlled (or vice versa). Decide between using a controlled or uncontrolled \`${componentName}\` for the lifetime of the component. Check the \`${controlPropName}\` prop.`
    )
    warning(
      !(!isControlled && wasControlled),
      `\`${componentName}\` is changing from controlled to be uncontrolled. Components should not switch from controlled to uncontrolled (or vice versa). Decide between using a controlled or uncontrolled \`${componentName}\` for the lifetime of the component. Check the \`${controlPropName}\` prop.`
    )
  }, [componentName, controlPropName, isControlled, wasControlled])
}

function useOnChangeReadOnlyWarning(
  controlPropValue,
  controlPropName,
  componentName,
  hasOnChange,
  readOnly,
  readOnlyProp,
  initialValueProp,
  onChangeProp
) {
  const isControlled = controlPropValue != null
  React.useEffect(() => {
    warning(
      !(!hasOnChange && isControlled && !readOnly),
      `A \`${controlPropName}\` prop was provided to \`${componentName}\` without an \`${onChangeProp}\` handler. This will result in a read-only \`${controlPropName}\` value. If you want it to be mutable, use \`${initialValueProp}\`. Otherwise, set either \`${onChangeProp}\` or \`${readOnlyProp}\`.`
    )
  }, [
    componentName,
    controlPropName,
    isControlled,
    hasOnChange,
    readOnly,
    onChangeProp,
    initialValueProp,
    readOnlyProp,
  ])
}

function useToggle({
  initialOn = false,
  reducer = toggleReducer,
  onChange,
  on: controlledOn,
  readOnly = false,
} = {}) {
  const { current: initialState } = React.useRef({ on: initialOn })
  const [state, dispatch] = React.useReducer(reducer, initialState)

  if (process.env.NODE_ENV !== 'production') {
    // eslint-disable-next-line react-hooks/rules-of-hooks
    useControlledSwitchWarning(controlledOn, 'on', 'useToggle')
    // eslint-disable-next-line react-hooks/rules-of-hooks
    useOnChangeReadOnlyWarning(
      controlledOn,
      'on',
      'useToggle',
      Boolean(onChange),
      readOnly,
      'readOnly',
      'initialOn',
      'onChange'
    )
  }

  const onIsControlled = controlledOn != null
  const on = onIsControlled ? controlledOn : state.on

  function dispatchWithOnChange(action) {
    if (!onIsControlled) {
      dispatch(action)
    }
    onChange?.(reducer({ ...state, on }, action), action)
  }

  const toggle = () => dispatchWithOnChange({ type: actionTypes.toggle })
  const reset = () =>
    dispatchWithOnChange({ type: actionTypes.reset, initialState })

  function getTogglerProps({ onClick, ...props } = {}) {
    return {
      'aria-pressed': on,
      onClick: callAll(onClick, toggle),
      ...props,
    }
  }

  function getResetterProps({ onClick, ...props } = {}) {
    return {
      onClick: callAll(onClick, reset),
      ...props,
    }
  }

  return {
    on,
    reset,
    toggle,
    getTogglerProps,
    getResetterProps,
  }
}

function Toggle({ on: controlledOn, onChange, readOnly }) {
  const { on, getTogglerProps } = useToggle({
    on: controlledOn,
    onChange,
    readOnly,
  })
  const props = getTogglerProps({ on })
  return <Switch {...props} />
}

function App() {
  const [bothOn, setBothOn] = React.useState(false)
  const [timesClicked, setTimesClicked] = React.useState(0)

  function handleToggleChange(state, action) {
    if (action.type === actionTypes.toggle && timesClicked > 4) {
      return
    }
    setBothOn(state.on)
    setTimesClicked(c => c + 1)
  }

  function handleResetClick() {
    setBothOn(false)
    setTimesClicked(0)
  }

  return (
    <div>
      <div>
        <Toggle on={bothOn} onChange={handleToggleChange} />
        <Toggle on={bothOn} onChange={handleToggleChange} />
      </div>
      {timesClicked > 4 ? (
        <div data-testid='notice'>
          Whoa, you clicked too much!
          <br />
        </div>
      ) : (
        <div data-testid='click-count'>Click count: {timesClicked}</div>
      )}
      <button onClick={handleResetClick}>Reset</button>
      <hr />
      <div>
        <div>Uncontrolled Toggle:</div>
        <Toggle
          onChange={(...args) =>
            console.info('Uncontrolled Toggle onChange', ...args)
          }
        />
      </div>
    </div>
  )
}
```
