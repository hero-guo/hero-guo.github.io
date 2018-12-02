---
title: 在 React 项目中使用 Redux 和 Redux-Saga
date: 2018-12-02 21:52:32
tags:
  - JavaScript
  - React
  - Redux
  - Redux-Saga
categories: JavaScript
photos:
---
关于 React 的数据管理库现在社区有很多解决方案，[Mobx](https://mobx.js.org/)，[Redux](https://redux.js.org/)，[RxJs](https://rxjs-dev.firebaseapp.com/)等。关于他们的优劣与适合的场景网上有很多讨论。本文主要介绍 Redux 与 Redux MiddleWare 。
<!-- more -->
##### 何时需要使用 Redux

1. 多层级组件之间需要共享数据
2. 应用状态嵌套层级很深
3. 期望使用单一数据流



##### Redux 核心概念

使用普通对象描述应用程序的状态，强制将状态的更改描述为一个操作，每次操作由一个动作 `action` 触发。通过 `reducer` 函数将动作和状态联系起来。

Redux的实现有三大基本原则：

1. 单一数据源
2. State 是只读的
3. 使用纯函数来执行修改

#### Redux 使用

Redux 的源码很少，使用起来也很简单，比较难理解的是它的中间件思想。

1.安装

   ```shell
   npm install --save redux react-redux redux-saga
   ```

2.使用 `create-react-app` 创建项目之后，创建 store 文件夹

   ```
   // store 文件夹
   ├── index.js
   ├── middleware
   │   ├── index.js
   ├── reducers
   │   ├── index.js
   └── sagas
       ├── index.js
   ```

3.创建 store

   ```javascript
   //store/index.js
   /**
    * @导出全局 store 基于 redux-sage
    * @format
    */
   // redux-devtools-extension 可以在浏览器里面查看 store 的状态
   import { applyMiddleware, createStore } from 'redux'
   import { composeWithDevTools } from 'redux-devtools-extension/logOnlyInProduction'
   import { localStore } from 'helpers' // localStorage 的封装
   import { sageMiddleware } from './middleware'
   import rootSaga from './sagas'
   import rootReducer from './reducers'
   
   const middlewares = [sageMiddleware]
   const composeEnhancers = composeWithDevTools({
     // redux-devtools-extension 的一些配置。一般情况不需要配置
   })
   // 如果需要从本地获取数据同步到 store 里面
   const xxx = localStore.get('xxx') || {} // 获取本地数据
   const store = createStore(
     rootReducer, // redux 数据处理函数
     xxx, // 初始化数据，可以不填
     composeEnhancers( // redux 中间件 增强 redux 的数据处理能力
       applyMiddleware(...middlewares)
     )
   )
   store.runSage = sageMiddleware.run
   store.runSage(rootSaga) // 动态地运行 saga
   function handleChange() { // redux 被触发的监听函数
     // 存储 store 数据到本地缓存
     const state = store.getState() // 获取当前 store 里面的全部数据
     localStore.set('xxx', state) // 存储到 localStorage
   }
   store.subscribe(handleChange)
   // Enable Webpack hot module replacement for reducers
   if (module.hot) { // webpack 热更新时候同步 redux
     module.hot.accept('./reducers', () => {
       store.replaceReducer(require('./reducers/index').default)
     })
   }
   export default store
   
   ```

4.创建 reducer

   ```javascript
   // store/reducers/index.js
   // combineReducers 是一个高阶函数。主要作用是合并多个 reducer
   import { combineReducers } from 'redux'
   import todo from './todo'
   export default combineReducers({ todo })
   ```

   ```javascript
   // store/reducers/todo.js
   // 定义 action
   export const ADD_TODO_PENDDING = 'todo/ADD_TODO_PENDDING'
   export const ADD_TODO_SUCCEEDED = 'todo/ADD_TODO_SUCCEEDED'
   export const ADD_TODO_FAILURED = 'todo/ADD_TODO_FAILURED'
   
   const initialState = { // 传入初始化 state
     todos: [],
     loading: true
   }
   // 根据 action.type 处理 state，不能修改 state，可以使用 Object.assing() 或者 展开运算符 {...state} 生成一个副本
   // 只要传入参数相同，返回计算得到的下一个 state 就一定相同。没有特殊情况、没有副作用，没有 API 请求、没有变量修改，单纯执行计算。
   export default function reducer(state = initialState, action = {}) {
     const { payload } = action
     switch (action.type) {
       case ADD_TODO_PENDDING:
         return { ...state }
       case ADD_TODO_SUCCEEDED:
         return { ...state, todos: payload, loading: false }
       case ADD_TODO_FAILURE:
         return { ...state, loading: false }
       default:
         return state
     }
   }
   ```

5.创建 sagas

   ```javascript
   // store/sagas/index.js
   import { all, fork } from 'redux-saga/effects'
   import todo from './todo'
   
   export default function* root() {
     yield all([fork(todo)])
   }
   ```

   ```javascript
   // store/sagas/todo.js
   import * as actionTypes from '../reducers/todo'
   import { $request } from 'helpers' // 基于 axios 封装的请求方法
   import { call, put } from 'redux-saga/effects'
   import { takeLatest } from 'redux-saga'
   
   function* addTodo(action) {
     try {
       const { data } = yield call(() => $request.post('/addTodo', action))
       yield put({ type: actionTypes.ADD_TODO_SUCCEEDED, payload: data })
     } catch (e) {
       yield put({ type: actionTypes.ADD_TODO_FAILURED})
     }
   }
   
   export default function* saga() {
     yield* takeLatest(actionTypes.ADD_TODO_PENDDING, addTodo)
   }
   ```

6.连接 React 和 Redux

   ```javascript
   // src/routes
   import React from 'react'
   import { Provider } from 'react-redux'
   import { BrowserRouter, Route, Switch } from 'react-router-dom'
   import Loadable from 'react-loadable'
   import store from './store'
   import { NoMatch, Loading } from 'components'
   const AddTodo = Loadable({
     loader: () => import('./container/AddTodo'),
     loading: Loading
   })
   export default function Routes() {
     return (
       <Provider store={store}> // 传递 store
         <BrowserRouter>
           <Switch>
   		  <Route path="/" component={App} />
             <Route path="/add" component={AddTodo} />
             <Route component={NoMatch} />
           </Switch>
         </BrowserRouter>
       </Provider>
     )
   }
   
   ```

   ```javascript
   // src/index.js
   import React from 'react'
   import ReactDOM from 'react-dom'
   import './style/antd.less'
   import Routes from './routes'
   ReactDOM.render(<Routes />, document.getElementById('root'))
   ```

7.触发 action

   ```javascript
   // container/AddTodo.js
   import React, { Component } from 'react'
   import { connect } from 'react-redux'
   import * as actionTypes from '../store/reducers/todo'
   @connect({ state => { data: state.todo } })
   class AddTodo extends Component {
       handleAdd = data => {
           // 触发 redux 的 action
           this.props.dispatch({type: actionTypes.ADD_TODO_PENDDING, data })
       }
       render() {
           // 省略布局实现
       }
   }
   ```

#### Redux 的中间件

Redux 里的 `createStore(reducer, [preloadedState], enhancer)` 第三个参数就是传入的中间件，第二个参数是可选的，如果不传，第二个参数就是中间件。主要的作用是提供 action 被发起之后，到达 reducer 之前的扩展点。可以记录每次 action 被触发的日志`redux-logger`，可以添加浏览器扩展`redux-devtools-extension` 。我们经常是用的功能是增强 action 的能力，因为 action 被定义为一个对象，原始情况下只能处理同步操作。通过添加中间件，我们可以传递一个发送网络请求的 action。

```javascript
// redux thunk
const thunk = store => next => action =>
  typeof action === 'function' ?
    action(store.dispatch, store.getState) :
    next(action)
```

#### Redux-Saga

Redux 的增强 action 使其具有网络请求能力的中间件有很多种 `redux-thunk` `redux-promise` `redux-saga` `redux-observable` 等。各有优劣，这里主要讲一下比较流行的 `redux-saga`。

`redux-saga`是一个旨在处理应用程序的副作用（网络请求，用户事件，浏览器缓存等），使其更容易管理，更容易测试的库。上面的代码里面已经展现了它的基本使用方法，这里我们主要讲一下它的基本原理。

`redux-saga` 是基于 ES6的新特性 `Generators` 实现的。它内部实现了很多辅助函数，这些辅助函数用于处理 action。 对于一个网络请求我们默认会有三种状态，类似于 Promise，`PENDDING`，`FULFILLED`，`REJECTED`。也就是说一次完整的请求我们需要设置三种类型的 action。

常用的一些辅助方法有

- `call(fn, ...args)` 发送一个请求，第一个参数是一个函数，期望返回一个 `promise`，同步调用，等待调用的返回
- `fork(fn, ...args)` 同 `call` 使用方法相似，只是它**非阻塞调用** 的形式执行 `fn`
- `put(action)` 类是与 redux 的 dispatch 函数，主要作用是触发一个 action
- `take(pattern)` 是最基本的一个方法，很多高级的方法都是基于他实现的。主要作用是在action 触发之前 `Generators` 函数是暂停的。
- `select(selector, ...args)` 选择当前 store 里面的 state
- `takeEvery(pattern, saga, ...args)`  `take` 的高级应用，监听 action 的触发，每次触发都会执行一次`pattern`
- `takeLatest(pattern, saga, ...args)` 与 `takeEvery`相似， 每次触发都会执行 `pattern`，但是他会取消上一次执行中的 `saga`

