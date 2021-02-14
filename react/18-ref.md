# ref

- [新写法](#新写法)
- [函数式写法--不推荐](#函数式写法)


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
        <input type="text" placeholder="" ref={dom => this.inputDom = dom}>
      </div>
    )
  }
}
```
