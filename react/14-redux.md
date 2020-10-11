# redux

主要有以下的概念
- `state`: 数据对象
- `store`: 存储数据(`state`)的仓库
- `dispatch`: 将触发方法
- `action`: 是一个动作，即，触发数据改变的方法
- `reducer`: 是一个函数，通过动作获取，改变数据，生成一个新的 `state`，返回给store，从而改变页面展示

## install

安装
```
npm install redux
```

## examples

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';

// 函数：用来改变数据，并返回state给store
function reducer(state = { num: 0 }, action/* 动作类型 */) {
  switch (action.type) {
    case 'increment':
      state.num++;
      break;
    case 'decrement':
      state.num--;
      break;
  }
  // 返回一个新的对象
  return { ...state };
}

// 创建仓库，存储reducer返回的数据
const store = createStore(reducer);

function increment() {
  // 通过仓库的`dispatch`调用方法修改数据
  store.dispatch('increment');
}

function decrement() {
  // 通过仓库的`dispatch`调用方法修改数据
  store.dispatch('decrement');
}

const Counter = props => {
  const state = store.getState();
  const { num } = state;
  return (
    <div>
      <h1>计数器数量: { num }</h1>
      <button onClick={increment}>++</button>
      <button onClick={decrement}>--</button>
    </div>
  )
};

ReactDOM.render(<Counter></Counter>, document.getElementById('root'));

// 订阅变化 - 点击按钮之后重新渲染
store.subscribe(() => {
  ReactDOM.render(<Counter></Counter>, document.getElementById('root'));
});
```
