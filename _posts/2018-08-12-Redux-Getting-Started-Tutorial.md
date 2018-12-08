# Redux Getting Started Tutorial

[![Foo](https://camo.githubusercontent.com/f28b5bc7822f1b7bb28a96d8d09e7d79169248fc/687474703a2f2f692e696d6775722e636f6d2f4a65567164514d2e706e67)](https://github.com/reduxjs/redux)

---

## Before we start it, maybe you don't need it

First, Redux is a very useful framework, but it is not necessary.
> If you don't know whether need Redux, then you don't need it.

> You only need Redux when you get problems that React can't solve.

So, if your UI layer is very simple and doesn't have a lot of interaction, Redux is unnecessary, if you use it, it adds complexity on your project.

## I. Prerequisite knowledge

Read this article, you only need to understand React. If you still know Flux, it's better, it's easier to understand some concepts, but it's not necessary.

Redux has a good [document](https://redux.js.org/). Read it!

## II. Design thinking

Redux's design is simple, just two sentences.

- The web application is a state machine, the view and the state are in one-to-one correspondence.
- All states are stored in an object

## III. Installation

```
yarn add redux react-redux redux-devtools-extension redux-thunk
```

## IV. Basic & APIs
### 4.1 Store

The Store is where you save your data, and you can think of it as a container. There can only be one **Store** for the app.

Redux provides the **createStore** function to generate the Store.


```jsx
import { createStore } from 'redux';
const store = createStore(fn);
```
### 4.2 State

The **Store** contains all the data. If you want to get the data at a certain point in time, you need to take a snapshot of the Store. The data set at this point in time is called **State**. The state of the current moment can be obtained by **store.getState()**.


```jsx
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```
But, a **State** corresponds to a **View**. As long as the **State** is the same, the **View** is the same.

### 4.3 Action

Changes in **State** will cause changes in **View**. Therefore, the change of **State** must be caused by **View**. 
**Action** is the notification from the **View** that the **State** should change.

**Action** is an object. The type attribute is ++required++, indicating the name of the Action, and other properties can be freely set.


```jsx
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```
In the above code, the name of the Action is **ADD_TODO**, and the information it carries is the string ++*Learn Redux*++.

### 4.4 Action Creator

We can also define a function to generate an **Action**, this function is called **Action Creator**.


```jsx
const ADD_TODO = 'ADD_TODO';

function addTodo(text) {
  return {
    type: ADD_TODO,
    text
  }
}

const action = addTodo('Learn Redux');
```
In the above code, the **addTodo** function is an Action Creator.

### 4.5 store.dispatch()

**store.dispatch()** is the only way that **View** sends an **Action**.

```jsx
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```

But also we can call it with **Action Creator** as:

```jsx
store.dispatch(addTodo('Learn Redux'));
```

### 4.6 Reducer

After the **Store** receives the **Action**, it must give a new **State** so that the **View** will change. This **State** process is called **Reducer**.

The **Reducer** is a function that takes an **Action** and the current **State** as arguments and returns a new State.


```jsx
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});

import { createStore } from 'redux';
const store = createStore(reducer);
```
In the above code, **createStore** accepts **Reducer** as a parameter and generates a new **Store**. Later, whenever **store.dispatch** sends a new **Action**, it will automatically call **Reducer** to get the new **State**.

Reducer can be used as an argument of the array's reduce method. Consider the following example, where a series of Action objects are treated as an array.

```jsx
const actions = [
  { type: 'ADD', payload: 0 },
  { type: 'ADD', payload: 1 },
  { type: 'ADD', payload: 2 }
];

const total = actions.reduce(reducer, 0); // 3
```

## 4.7 Pure Function
The most important feature of the **Reducer** function is that it is a **pure function**.

**Pure functions** are concepts of **functional programming** and must adhere to the following constraints.

> Do not overwrite parameters

> Do not call system I/O API

> Do not call impure methods like Date.now() or Math.random() because it will get different results every time.

Since the **Reducer** is a **pure function**, It must guarantee the same **State**, and it must get the same **View**. But also because of this, the **Reducer** function can not change the **State**, it must return a new **object**, please refer to the following writing.

```jsx
// State is a object
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // or
  return { ...state, ...newState };
}

// State is a Array
function reducer(state, action) {
  return [...state, newItem];
}
```

It is best to set the *State* object to read-only. You can't change it. The only way to get a new **State** is to generate a new object.

### 4.8 store.subscribe()

The **Store** allows you to set the **listener** function using the **store.subscribe** method, which is automatically called once the **State** changes.


```jsx
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

The **store.subscribe** method returns a function that call it to unlisten it.

```jsx
let unsubscribe = store.subscribe(() =>
  console.log(store.getState())
);
unsubscribe();
```

## V. Store Implementation

The previous section introduced the basic concepts in Redux, and we can see that the **Store** provides three methods.

- store.getState()
- store.dispatch()
- store.subscribe()

```jsx
import { createStore } from 'redux';
let { subscribe, dispatch, getState } = createStore(reducer);
```

## VI. Reducer Split

The **Reducer** function is responsible for generating the new **State**. Since the application has only one **State** object, which containing all the data, for a large applications, the **State** must be very large, resulting in a huge **Reducer** function.

For example:


```jsx
const chatReducer = (state = defaultState, action = {}) => {
  const { type, payload } = action;
  switch (type) {
    case ADD_CHAT:
      return Object.assign({}, state, {
        chatLog: state.chatLog.concat(payload)
      });
    case CHANGE_STATUS:
      return Object.assign({}, state, {
        statusMessage: payload
      });
    case CHANGE_USERNAME:
      return Object.assign({}, state, {
        userName: payload
      });
    default: return state;
  }
};
```

In the above code, the three **Actions** change the three properties of **State** respectively.

- ADD_CHAT：chatLog
- CHANGE_STATUS：statusMessage
- CHANGE_USERNAME：userName

There is no connection between these **three properties**, which suggests that we can **split** the **Reducer** function. Different functions are responsible for handling different properties and eventually **merging** them into one large **Reducer**


```jsx
const chatReducer = (state = defaultState, action = {}) => {
  return {
    chatLog: chatLog(state.chatLog, action),
    statusMessage: statusMessage(state.statusMessage, action),
    userName: userName(state.userName, action)
  }
};
```

In the above code, the **Reducer** function is split into three small functions, each responsible for generating the corresponding attribute.

In this way, the **Reducer** is easy to read and write more.

Redux provides a **combineReducers** method for splitting the **Reducer**. You only need to define each of the **Sub-Reducer** functions, and then use this method to combine them into a large **Reducer**.

```jsx
import { combineReducers } from 'redux';

const reducer = combineReducers({
  chatLog: chatLog,
  statusMessage: statusMessage,
  userName: userName
})
```

You can put all the sub-Reducers in a file and then unify them.

```jsx
import { combineReducers } from 'redux'
import * as reducers from './reducers'

const reducer = combineReducers(reducers)
```

## VII. Work Process

![foo](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)

## VIII. Example:

```jsx
const Counter = ({ value, onIncrement, onDecrement }) => (
  <div>
  <h1>{value}</h1>
  <button onClick={onIncrement}>+</button>
  <button onClick={onDecrement}>-</button>
  </div>
);

const reducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT': return state + 1;
    case 'DECREMENT': return state - 1;
    default: return state;
  }
};

const store = createStore(reducer);

const render = () => {
  ReactDOM.render(
    <Counter
      value={store.getState()}
      onIncrement={() => store.dispatch({type: 'INCREMENT'})}
      onDecrement={() => store.dispatch({type: 'DECREMENT'})}
    />,
    document.getElementById('root')
  );
};

render();
store.subscribe(render);
```
