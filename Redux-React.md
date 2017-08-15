
## Usage with React

Redux works especially well with libraries like React and Deku because they let you describe UI as a function of state, and Redux emits state updates in response to actions.

```
npm install --save react-redux
```

## Components

* Presentational Components
* Container Components

## Implementing Container Components

* A container component is just a React component that uses store.subscribe() to read a part of the Redux state tree and supply props to a presentational component it renders.
* Use of React Redux library's connect() function
  * Map state to props
  * Map dispatch to props

### Map state to props

Tells how to transform the current Redux store state into the props

```
const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = state => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}
```

Using spread operator

```
const mapStateToProps = (state) => {
    return {
        ...state.todosReducer
    };
};
```

Reduced syntax

```
const mapStateToProps = (state) => ({
  ...state.todosReducer
});
```

### Map dispatch to props

Returns callback props that you want to inject into the presentational component
We need another function to be able to dispatch our onTodoClick() action creator with a prop.

```
const mapDispatchToProps = dispatch => {
  return {
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  }
}
```

### Reducing boilerplate

```
const mapDispatchToProps = dispatch => ({
    onTodoClick: id => {
      dispatch(toggleTodo(id))
    }
  });
```

#### Using bound action creator

```
export function onTodoClick(id) {
  return dispatch => {
    dispatch(toggleTodo(id));
  }
}
```

```
import { onTodoClick } from './actions';

const mapDispatchToProps = dispatch => ({
    onTodoClick
  });
```

#### Using bindActionCreators

Importing dependencies

```
import { bindActionCreators } from 'redux';
import * as actionCreators from './actions';
```

bind actions and map to props

```
const mapDispatchToPropsBind = dispatch => bindActionCreators(actionCreators, dispatch);
```

## Export connected to store component

```
import { connect } from 'react-redux'

export connect(mapStateToProps, mapDispatchToProps)(TodoList)
```


## Passing the Store

* For using Redux in React all container components need access to the Redux store so they can subscribe to it
* Use the special React Redux component called <Provider> to magically make the store available to all container components in the application without passing it explicitly


```
import React from 'react'
import { render } from 'react-dom'
import { Provider } from 'react-redux'
import { createStore } from 'redux'
import todoApp from './reducers'
import App from './components/App'

let store = createStore(todoApp)

render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById('root')
)
```