# ref

- [新写法](#新写法)
- [函数式写法](#函数式写法)


## 新写法

`ref` 的用法

- 先创建ref
- 将创建的ref绑定到dom元素上

```jsx
class App extends React.Component {
  constructor() {
    super();
    this.state = {
      inputValue: ''
    };
    // 创建ref(也可以通过调用从react导入createRef方法创建)
    this.inputDom = React.createRef();
  }

  handleAdd = () => {
    this.setState({
      inputValue: ''
    }, () => {
      // 清理完表单后，将焦点放在该元素上
      this.inputDom.focus();
    });
  };

  render() {
    return (
      <div class="app">
        {/* 绑定ref实例到dom元素上 */}
        <input type="text" placeholder="" ref={this.inputDom}>
      </div>
    )
  }
}
```

## 函数式写法

不推荐的写法，看[这里](https://reactjs.org/docs/refs-and-the-dom.html#caveats-with-callback-refs)
```jsx
class App extends React.Component {
  constructor() {
    super();
    this.inputDom = null;
  }

  render() {
    return (
      <div class="app">
        {/* 绑定ref实例到dom元素上 */}
        <input type="text" placeholder="" ref={dom => {
          console.log(`执行了多次`);
          this.inputDom = dom;
        }>
      </div>
    )
  }
}
```

推荐使用下面的写法，即，`You can avoid this by defining the ref callback as a bound method on the class, but note that it shouldn’t matter in most cases`
```jsx
class App extends React.Component {
  constructor() {
    super();
    this.inputDom = null;
  }

  saveInput(dom) {
    console.log(`只会执行一次`);
    this.inputDom = dom;
  }

  render() {
    return (
      <div class="app">
        {/* 绑定ref实例到dom元素上 */}
        <input type="text" placeholder="" ref={this.saveInput}>
      </div>
    )
  }
}
```
