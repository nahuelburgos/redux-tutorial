# Middlewares

* It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer. 
* The best feature of middleware is that it's composable in a chain

## Creating middleware

```
const logger = store => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', store.getState())
  return result
}

const crashReporter = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Caught an exception!', err)
    Raven.captureException(err, {
      extra: {
        action,
        state: store.getState()
      }
    })
    throw err
  }
}
```

## Configure Middleware

```
import { createStore, combineReducers, applyMiddleware } from 'redux'

let todoApp = combineReducers(reducers)
let store = createStore(
  todoApp,
  // applyMiddleware() tells createStore() how to handle middleware
  applyMiddleware(logger, crashReporter)
)
```

# Example

```
export const authMiddleware = () => next => action => {  
  if (action.type === AUTH_SUCCESS) {
    if (action.authInfo && action.authInfo.token) {
      auth.authenticateUser(action.authInfo.token);
    } else {
      auth.deauthenticateUser();
    }
  } else if (action.type === LOGOUT) {
    auth.deauthenticateUser();
  }
  next(action);
};
```


