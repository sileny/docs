# react-redux

- [install](#install)
- [example](#example)
- [multiple-reducer](#multiple-reducer)
- [使用decorator实现redux计数器](#decorators)

## install

```
npm i react-redux
```

## example

需求：
- 当点击点击 `++` 按钮，将会触发 `increment` 类型的操作，之后，将会通过 `dispatch` 调用 `incrementAction` 的方法，该方法会执行 `reducer` 的 `state.num++` 并返回新的 `state`
- 点击 `--` 按钮，也是如此


项目主要结构：
```
|--public/
|--src/
|----App.js
|----index.js
```


组件设计实现：

> `App` 组件的实现

创建一个 `App` 组件，渲染父组件传递过来的数据
```js
function App(props) {
  const { value, increment, decrement } = props;
  return (
    <div className="App">
      <div>
        <h1>计数器数量: { value }</h1>
        <button onClick={increment}>++</button>
        <button onClick={decrement}>--</button>
      </div>
    </div>
  );
}
```

定义 `App` 组件需要的数据状态 `state`
```js
function mapStateToProps(state) {
  return {
    value: state.num // 将num映射为value，按照设计模式的角度来看，就是接口适配
  };
}
```

定义希望通过何种操作(`action`)修改数据状态(`state`)
```js
const incrementAction = { type: 'increment' };
const decrementAction = { type: 'decrement' };
```

定义通过指定操作(`action`)修改数据状态的方法，该方法通过 `dispatch` 实现
```js
function mapDispatchToProps(dispatch) {
  return {
    increment: () => { dispatch(incrementAction) },
    decrement: () => { dispatch(decrementAction) }
  };
}
```
>`dispatch`方法是默认注入的方法

将 `App` 组件的需要的数据状态和修改数据状态的方法通过 `connect` 来连接。即，点击 `++` 按钮实现数值新增，点击 `--` 实现数值减少

至此，`App`组件已经完成。下面来实现父组件的数据传值


> 父组件的实现

定义进行数据变更的业务方法(`reducer`)
```js
function reducer(state = { num: 0 }, action) {
  switch (action.type) {
    case 'increment':
      state.num++;
      break;
    case 'decrement':
      state.num--;
      break;
  }
  return { ...state };
}
```

创建仓库，存储 `reducer` 返回的数据
```js
const store = createStore(reducer);
```

将数据通过 `Provider` 组件传递给 `App` 组件
```jsx
<Provider store={ store }>
  <App />
</Provider>
```

>`Provider`组件是一个顶级组件，数据可以被内部的所有的子组件拿到


下面是完整的代码

父组件代码如下，
```js
// index.js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

import { createStore } from 'redux';
import { Provider } from 'react-redux';

// 函数：用来改变数据，并返回state给store
function reducer(state = { num: 0 }, action) {
  switch (action.type) {
    case 'increment':
      state.num++;
      break;
    case 'decrement':
      state.num--;
      break;
    default:
      break;
  }
  return { ...state };
}

// 创建仓库，存储reducer返回的数据
const store = createStore(reducer);

ReactDOM.render(
  <React.StrictMode>
    <Provider store={ store }>
      <App />
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();

```

`App`组件代码
```js
import React from 'react';
import PropTypes from 'prop-types';
import './App.css';

import { connect } from 'react-redux';

const incrementAction = { type: 'increment' };
const decrementAction = { type: 'decrement' };

// 将state映射到props
function mapStateToProps(state) {
  return {
    value: state.num // 将num映射为value
  };
}

// 将修改state的方法映射到props，即，dispatch的类型传递给action
// mapDispatchToProps默认将dispatch作为第一参数
function mapDispatchToProps(dispatch) {
  return {
    increment: () => { dispatch(incrementAction); },
    decrement: () => { dispatch(decrementAction); }
  };
}

function App(props) {
  const { value, increment, decrement } = props;
  return (
    <div className='App'>
      <div>
        <h1>计数器数量: { value }</h1>
        <button onClick={increment}>++</button>
        <button onClick={decrement}>--</button>
      </div>
    </div>
  );
}

App.propTypes = {
  value: PropTypes.number,
  increment: PropTypes.func,
  decrement: PropTypes.func
};

export default connect(mapStateToProps, mapDispatchToProps)(App);

```

该案例原理：
- 所有的数据状态丢到 `Provider` 全局组件上，其内部的任意一个组件都可以共享 `store` 里的数据
- ui操作通过 `mapDispatchToProps` 内部的 `dispatch` 来触发 `action` 指定类型的变更
- `reducer` 在接收到 `action` 类型变更指令后。更新数据 `state`

## multiple-reducer

一个项目里可能有多个 `reducer`，它们可以单独放在一个 `js` 文件里。然后，将它们都放在 `reducers` 目录下

现在，将 `reducer` 方法单独转移到 `counter.js` 里。

多个 `reducer` 使用 `redux` 的 `combineReducers` 方法进行连接

```js
// reducers/index.js
import { combineReducers } from 'redux'
import counter from './counter'
import otherReducer from './otherReducer'

export default combineReducers({
  counter,
  otherReducer
});

```

`src/index.js` 写法如下
```js
import { createStore } from 'redux';
import rootReducer from './reducers'; // 状态：state
const store = createStore(rootReducer);
```

## decorators

从上面的案例可以看出，顶级组件负责维护数据状态，子组件用来渲染数据。

因此，可以将数据放在 `Provider` 组件上，通过 `Counter` 组件渲染实时的数据

项目结构：
```
|src/
|--redux-demo/
|----actions/
|------index.js
|----reducers/
|------counter.js
|------index.js
|----Counter.js
|----store.js
|--index.js
```

根程序
`src/index.js`
```js
import React from 'react';
import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import store from './redux-demo/store';
import Counter from './redux-demo/Counter';

ReactDOM.render(
  <Provider store={store}>
    <Counter />
  </Provider>,
  document.getElementById('root')
);
```

计数器的装饰器方式实现
`src/redux-demo/Counter.js`
```js
import React from 'react';
import { connect } from 'react-redux';
import PropTypes from 'prop-types';
import { decrementAction, incrementAction } from './actions';

@connect(
  state => {
    const { counter } = state;
    return {
      counterValue: counter.num
    };
  },
  dispatch => ({
    decrement: () => dispatch(decrementAction),
    increment: () => dispatch(incrementAction)
  })
)
class Counter extends React.Component {
  static propTypes = {
    decrement: PropTypes.func,
    increment: PropTypes.func,
    counterValue: PropTypes.number
  };
  render() {
    const { counterValue, decrement, increment } = this.props;
    return <div className="counter">
      <button onClick={decrement}>--</button>
      <span>{counterValue}</span>
      <button onClick={increment}>++</button>
    </div>;
  }
}

export default Counter;

```

状态数据
`src/redux-demo/store.js`
```js
import reducers from '../redux-demo/reducers';
import { createStore } from 'redux';

const store = createStore(reducers);

export default store;

```

`src/redux-demo/actions/index.js`
```js
export const incrementAction = { type: 'increment' };
export const decrementAction = { type: 'decrement' };

```

计数器变更 `reducer`
`src/redux-demo/reducers/counter.js`
```js
const counter = (state = { num: 0 }, action) => {
  switch (action.type) {
    case 'increment':
      state.num++;
      break;
    case 'decrement':
      state.num--;
      break;
    default:
      break;
  }
  return { ...state };
};

export default counter;

```

整合多个 `reducer`
`src/redux-demo/reducers/index.js`
```js
import { combineReducers } from 'redux';
import counter from './counter';

export default combineReducers({
  counter
});

```
