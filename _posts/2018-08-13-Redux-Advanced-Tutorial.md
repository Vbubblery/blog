# Redux Advanced Tutorial

[![Foo](https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67)](https://github.com/reduxjs/redux)

---

## I. Middleware

To understand the **middleware**, let's think about the problem from the perspective of the framework author: If you want to add a **function**, where do you add it?

- **Reducer**: pure function, only work on the **State**, cannot woroking on another operations
- **View**: one-to-one correspondence with **State**, can be seen as the visual layer of **State** and is not suitable for other functions.
- **Action**: stores the data, the carrier of the message.

This **function** can be added only by sending this step of the **Action**, the **store.dispatch()** method.

```jsx
let next = store.dispatch;
store.dispatch = function dispatchAndLog(action) {
  console.log('dispatching', action);
  next(action);
  console.log('next state', store.getState());
}
```

In the above code, the **store.dispatch** is redefined, and the **print** function is added before and after the **Action** is sent.

The middleware is a function that modifies the **store.dispatch** method and adds additional functionality between the two steps of the **Action** and executing the **Reducer**.

## II. Middleware usage

```jsx
import { applyMiddleware, createStore } from 'redux';
import createLogger from 'redux-logger';
const logger = createLogger();

const initialState = {
  count:0,
};

const store = createStore(
  reducer,
  initialState,
  applyMiddleware(logger)
);
```

### Attention

Middleware is in order

```jsx
const store = createStore(
  reducer,
  applyMiddleware(thunk, promise, logger)
);
```

## III. Aasynchronous

The synchronization operation only emits an Action, but 3 actions for asynchronous.

- Action initiated
- Action successful
- Action fails

The 3 actions could be:

```jsx
{ type: 'FETCH_POSTS' }
{ type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
{ type: 'FETCH_POSTS', status: 'success', response: { ... } }
```

At the same time, the **State** of the asynchronous operation is also modified.

```jsx
let state = {
  // ... 
  isFetching: true,
  didInvalidate: true,
  lastUpdated: 'xxxxxxx'
};
```

In the above code, 

- **isFetching** means whether the data is being fetched.
- **didInvalidate** means whether the data is out of date
- **lastUpdated** means the last update time.

Now, the idea of asynchronous operations is very clear.

- At the beginning of the operation, an Action is sent, the State is triggered to update to the "In Operation" state, and the View is re-rendered.
- After the operation is finished, an Action is sent, triggering the State update to the "End of Operation" state, and the View is re-rendered again.

## IV. redux-thunk Middleware

Asynchronous operations must send at least two Actions: 

1. The user triggers the first Action, which is the same as the synchronous operation, no problem
2. How can the system automatically send the second Action at the end of the operation?

The answer is in the **Action Creator**.

```jsx
class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    dispatch(fetchPosts(selectedPost))
  }

// ...
```

The above code is an example of an **asynchronous** component. After the load is successful (**componentDidMount** method), it sends out (**dispatch** method) an **Action**, requesting data **fetchPosts(selectedSubreddit)** from the server. The **fetchPosts** here are **Action Creator**.

## V.redux-promise Middleware

Another solution for asynchronous, **Action Creator** return a **Promise** object.

We need **redux-promise** now.

```jsx
import { createStore, applyMiddleware } from 'redux';
import promiseMiddleware from 'redux-promise';
import reducer from './reducers';

const store = createStore(
  reducer,
  applyMiddleware(promiseMiddleware)
); 
```

This middleware makes the **store.dispatch** method accept a **Promise** object as a parameter.

To implement this, **Action Creator** has two methods. For first way, the return value is a Promise object.

```jsx
import { createAction } from 'redux-actions';

class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    // send Synchronize Action
    dispatch(requestPosts(selectedPost));
    // send Asynchronize Action
    dispatch(createAction(
      'FETCH_POSTS', 
      fetch(`/some/API/${postTitle}.json`)
        .then(response => response.json())
    ));
  }
```

For the second way:

The payload property of the Action object is a Promise object. And it need import **createAction** method from redux-actions.

```jsx
import { createAction } from 'redux-actions';

class AsyncApp extends Component {
  componentDidMount() {
    const { dispatch, selectedPost } = this.props
    // send Synchronize Action
    dispatch(requestPosts(selectedPost));
    // send Asynchronize Action
    dispatch(createAction(
      'FETCH_POSTS', 
      fetch(`/some/API/${postTitle}.json`)
        .then(response => response.json())
    ));
  }
```

