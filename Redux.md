# Redux 

## Install dependencies

To start, we need to add Redux and React Redux as dependencies of our project so we can use them.
```
npm install redux
```

## Motivation

* Our code must manage more state than ever before
* When a system is opaque and non-deterministic, it's hard to reproduce bugs or add new features.
* We're mixing two concepts that are very hard for the human mind to reason about:mutation and asynchronicity
* Redux attempts to make state mutations predictable

## Three principles

* Single source of truth
The state of your whole application is stored in an object tree within a single store.
* State is read-only
The only way to change the state is to emit an action, an object describing what happened.
* Changes are made with pure functions
To specify how the state tree is transformed by actions, you write pure reducers.

## Basics

### ACTIONS

Actions are payloads of information that send data from your application to your store

* Actions are plain JavaScript objects
* Actions must have a type property
* You send them to the store using store.dispatch().

```
const ADD_TODO = 'ADD_TODO'

{
  type: ADD_TODO,
  text: 'Build my first Redux app'
}
```

#### Action Creators

* functions that create actions

```
function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}
```

```
dispatch(addTodo(text))
dispatch(completeTodo(index))
```

#### Bound action creators

```
const boundAddTodo = text => dispatch(addTodo(text))
const boundCompleteTodo = index => dispatch(completeTodo(index))

boundAddTodo(text)
boundCompleteTodo(index)
```

### REDUCERS

* Reducers are "pure functions". They should not have any side-effects nor mutate the state – they must return a modified copy.

'''(Note: A pure function doesn't depend on and doesn't modify the states of variables out of its scope. Concretely, that means a pure function always returns the same result given same parameters. Its execution doesn't depend on the state of the system. Pure functions are a pillar of functional programming)'''

#### Designing the State Shape

```
{
  visibilityFilter: 'SHOW_ALL',
  todos: [
    {
      text: 'Consider using Redux',
      completed: true,
    },
    {
      text: 'Keep all state in a single tree',
      completed: false
    }
  ]
}
```

#### Handling Actions

* Each reducer takes 2 parameters: the previous state (state) and an action object. 

```
(previousState, action) => newState
```

* We use a switch statement to determine when an action.type matches

```
function todoAppReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return  { 
        ...state,
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      };
    case SET_VISIBILITY_FILTER:
      return { 
        ...state,
        visibilityFilter: action.filter
      };
    default:
      return state
  }
}
```

#### Should NEVER DO
Things you should never do inside a reducer:
* Mutate its arguments;
* Perform side effects like API calls and routing transitions;
* Call non-pure functions, e.g. Date.now() or Math.random().

#### Splitting Reducers
```
export default function todosReducer(state = initialState, action) {
  switch (action.type) {
    case ADD_TODO:
      return  { 
        ...state,
        todos: [
          ...state.todos,
          {
            text: action.text,
            completed: false
          }
        ]
      };
    default:
      return state
  }
}
```

```
export default function visibilityFilterReducer(state = initialState, action) {
  switch (action.type) {
    case SET_VISIBILITY_FILTER:
      return { 
        ...state,
        visibilityFilter: action.filter
      };
    default:
      return state
  }
}
```

* Individual reducers are combined into a single rootReducer to create the discrete properties of the state.
With the individual reducers created, we need to combine them into a rootReducer to create a single object

```
function todoApp(state = {}, action) {
  return {
    visibilityFilterReducer: visibilityFilterReducer(state.visibilityFilterReducer, action),
    todosReducer: todosReducer(state.todosReducer, action)
  }
}
```

'''Note that each of these reducers is managing its own part of the global state. The state parameter is different for every reducer, and corresponds to the part of the state it manages.'''

* Redux provides a utility called combineReducers() that does the same boilerplate logic

```
import { combineReducers } from 'redux';
import todosReducer from './todosReducer';
import visibilityFilterReducer from './visibilityFilterReducer';

export default combineReducers({
    todosReducer,
    visibilityFilterReducer
});
```

### STORE

The Store is the object that brings actions and reducers together.

* Holds application state
* Allows access to state via getState();
* Allows state to be updated via dispatch(action);
* Registers listeners via subscribe(listener);
* Handles unregistering of listeners via the function returned by subscribe(listener).

```
import { createStore } from 'redux'
import todoApp from './reducers'

let initialState = {};

let store = createStore(todoApp, initialState)
```

#### Dispatching Actions

```
// Dispatch some actions
store.dispatch(addTodo('Learn about actions'))
store.dispatch(addTodo('Learn about reducers'))
store.dispatch(addTodo('Learn about store'))
```

### DATA FLOW

1. call `store.dispatch()``
2. The Redux Store calls the reducer functions you gave it
```
(previousState, action) => newState
```
3. The root reducer may combine the output of multiple reducers into a single state tree
4. The Redux store saves the complete state tree returned by the root reducer.
 If you use bindings like React Redux, this is the point at which component.setState(newState) is called.

