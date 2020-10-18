# hoc

高阶组件：利用闭包原理暴露出组件

```js
const fn = x => {
  return y => {
    return x + y;
  };
};

// fn(1) 返回了一个函数，fn就是高阶函数
fn(1)(2); // 3
```

高阶组件实现：用一个函数包裹组件

```js
const withCopyright = Cmp => {
  return class WithCopyright extends React.Component {
    render() {
      return <div className={'withCopyright'}>
        <Cmp/>
        <div>WithCopyright</div>
      </div>;
    }
  };
};

function App() {
  return (
    <div className="App">
      App
    </div>
  );
}

export default withCopyright(App);
```
