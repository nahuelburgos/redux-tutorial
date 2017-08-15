# Async Actions

## Installing dependency
```
npm install redux-thunk --save
```

## Designing State Shape

```
{
  isLoading: false
  posts: [],
  didInvalidate: false,
  error: null
}
```

## Handling Actions

```
import {
  LOAD_FEED_REQUEST,
  LOAD_FEED_RESPONSE,
  LOAD_FEED_ERROR,
  INVALIDATE_FEED
} from './actions';

const initialState = {
  isLoading: false
  posts: [],
  didInvalidate: false,
  error: null
};

function feedReducer(state = initialState, action) {
  switch (action.type) {
    case INVALIDATE_FEED:
      return {
        ...state,
        didInvalidate: true
      })
    case LOAD_FEED_REQUEST:
      return {
        ...state,
        isLoading: true,
      };
    case LOAD_FEED_RESPONSE:
      return {
        ...state,
        feed: action.data,
        isLoading: false,
      };
    case LOAD_FEED_ERROR:
      return {
        ...state,
        error: action.error,
        feed: undefined,
        isLoading: false,
      };
    default:
      return state;
  }
}

export default feedReducer;
```

## Async Action Creators

```
import { getFeed } from '../../api/feed';

export const LOAD_FEED_REQUEST = 'zoopix-app/Feed/LOAD_FEED_REQUEST';

const loadFeedRequest = () => ({
  type: LOAD_FEED_REQUEST,
});

export const LOAD_FEED_RESPONSE = 'zoopix-app/Feed/LOAD_FEED_RESPONSE';

const loadFeedResponse = response => ({
  type: LOAD_FEED_RESPONSE,
  feed: response.data,
});

export const LOAD_FEED_ERROR = 'zoopix-app/Feed/LOAD_FEED_ERROR';

const loadFeedError = error => ({
  type: LOAD_FEED_ERROR,
  error,
});

// Meet our first thunk action creator!
// Though its insides are different, you would use it just like any other action creator:
// store.dispatch(getFeed(petId, pagination))

export function loadFeed(petId) {
  
  // Thunk middleware knows how to handle functions.
  // It passes the dispatch method as an argument to the function,
  // thus making it able to dispatch actions itself.

  return dispatch => {

     // First dispatch: the app state is updated to inform
    // that the API call is starting.

    dispatch(loadFeedRequest());

    // The function called by the thunk middleware can return a value,
    // that is passed on as the return value of the dispatch method.

    // In this case, we return a promise to wait for.
    // This is not required by thunk middleware, but it is convenient for us.

    return getFeed(petId)
     // We can dispatch many times!
        // Here, we update the app state with the results of the API call.
      .then(response => dispatch(loadFeedResponse(response)))

      .catch(error => dispatch(loadFeedError(error)));
  };
}
```

- Note:
Because most browsers don't yet support fetch natively, we suggest that you use isomorphic-fetch library:
```
  // Do this in every file where you use `fetch`
import fetch from 'isomorphic-fetch'
```
Be aware that any fetch polyfill assumes a Promise polyfill is already present. The easiest way to ensure you have a Promise polyfill is to enable Babel's ES6 polyfill in your entry point before any other code runs:

```
// Do this once before any other code in your app
import 'babel-polyfill'
```

## Thunk Middleware

```
import thunkMiddleware from 'redux-thunk'
import { createStore, applyMiddleware } from 'redux'
import rootReducer from './reducers'

const loggerMiddleware = createLogger()

const store = createStore(
  rootReducer,
  applyMiddleware(
    thunkMiddleware, // lets us dispatch() functions
  )
)
```

## Using getState

```
function loadFeed(petId) {
  return dispatch => {
    dispatch(loadFeedRequest());
    return getFeed(petId)
      .then(response => dispatch(loadFeedResponse(response)))
      .catch(error => dispatch(loadFeedError(error)));
  };
}

function shouldFetchPosts(state, petId) {
  const feed = state.feed
  if (!feed) {
    return true
  } else if (posts.isLoading) {
    return false
  } else {
    return state.didInvalidate
  }
}


export function loadFeedIfNeeded(petId) {
  // Note that the function also receives getState()
  // which lets you choose what to dispatch next.

  // This is useful for avoiding a network request if
  // a cached value is already available.

  return (dispatch, getState) => {
    if (shouldLoadFeed(getState(), petId)) {
      // Dispatch a thunk from thunk!
      return dispatch(loadFeed(petId))
    } else {
      // Let the calling code know there's nothing to wait for.
      return Promise.resolve()
    }
  }
}
```





