# state-hook

先看一个例子

```js
import React, { useState } from 'react';

function Example() {
  // 声明一个叫 `count` 的 state 变量，初始值为0
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

这里的 `useState` 就是一个 `Hook`。

通过调用 `useState` 给组件添加一些内部 `state`。在 `React` 重新渲染时保留这个 `state`。

`useState` 返回一个键值对：`当前的状态` 和 `更改该状态的函数`(类似于 `this.setState`)，但，不会把新的 `state` 和旧的 `state` 合并。

上面的案例里，计数器是从 `0` 开始的，即 `state` 就是 `0`。当然，这个 `state` 也可以定义成一个对象。`setCount` 就是修改状态的方法。

因为这个特性，所以，可以在函数组件里记录 `state` 和 引入生命周期等特性函数。

>Hook不能在 `class` 组件里使用


当然，也可以多次使用 `state hook`
```js
function ExampleWithManyStates() {
  // 声明多个 state 变量！
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```
