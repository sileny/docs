# react-redux


## install

```
npm i react-redux
```

## example

创建一个 `Counter` 组件，渲染父组件传递过来的数据
```js
class Counter extends React.Component {
  render() {
    const value = this.props.value;
    const increment = this.props.increment; // 指向了`mapDispatchToProps`关联的方法
    const decrement = this.props.decrement;
    return (
      <div>
        <h1>计数器数量: { value }</h1>
        <button onClick={increment}>++</button>
        <button onClick={decrement}>--</button>
      </div>
    )
  }
}
```

定义 `state`
```js
function mapStateToProps(state) {
  return {
    value: state.num // 将num映射为value
  };
}
```

定义通过何种操作(`action`)修改数据状态(`state`)
```js
const incrementAction = { type: 'increment' };
const decrementAction = { type: 'decrement' };
```

定义通过指定操作(`action`)修改数据状态的方法
```js
function mapDispatchToProps(dispatch) {
  return {
    increment: () => { dispatch(incrementAction) },
    decrement: () => { dispatch(decrementAction) }
  };
}
```

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

当点击点击 `++` 按钮，将会触发 `increment` 类型的操作，之后，将会通过 `dispatch` 调用 `incrementAction` 的方法，该方法会执行 `reducer` 的 `state.num++` 并返回新的 `state`

点击 `--` 按钮，也是如此

下面是完整的代码
```js
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import { Provider, connect } from 'react-redux';

const incrementAction = { type: 'increment' };
const decrementAction = { type: 'decrement' };

// 函数：用来改变数据，并返回state给store
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

// 创建仓库，存储reducer返回的数据
const store = createStore(reducer);

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
    increment: () => { dispatch(incrementAction) },
    decrement: () => { dispatch(decrementAction) }
  };
}

class Counter extends React.Component {
  render() {
    const value = this.props.value;
    const increment = this.props.increment; // 指向了`mapDispatchToProps`关联的方法
    const decrement = this.props.decrement;
    return (
      <div>
        <h1>计数器数量: { value }</h1>
        <button onClick={increment}>++</button>
        <button onClick={decrement}>--</button>
      </div>
    )
  }
}

// 将数据状态(state)和修改数据的方法(dispatch)通过connect添加到组件上，形成一个新的组件
const App = connect(mapStateToProps, mapDispatch)(Counter);
// 如果不需要传入state，只需要传入方法，那么，第一参数为null即可，即，connect(null, mapDispatch)(Counter)

// Provider将store与App根组件进行关联
ReactDOM.render(<Provider store={ store }>
  <App />
</Provider>, document.getElementById('root'));
```

该案例原理：
- 所有的数据状态丢到 `Provider` 全局组件上，其内部的任意一个组件都可以共享 `store` 里的数据
- ui操作通过 `mapDispatchToProps` 内部的 `dispatch` 来触发 `action` 指定类型的变更
- `reducer` 在接收到 `action` 类型变更指令后。更新数据 `state`
