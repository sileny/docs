# life-cycle

生命周期

## mounting

当组件实例被创建并插入 DOM 中时，其生命周期调用顺序如下

- `constructor()`
- `static getDerivedStateFromProps()`
- `render()`
- `componentDidMount()`

## update

当组件的 props 或 state 发生变化时会触发更新。组件更新的生命周期调用顺序如下：

- [`static getDerivedStateFromProps()`](#getderivedstatefromprops)
- [`shouldComponentUpdate()`](#shouldcomponentupdate)
- [`render()`](#render)
- [`getSnapshotBeforeUpdate()`](#getsnapshotbeforeupdate)
- [`componentDidUpdate()`](#componentdidupdate)

### getDerivedStateFromProps

语法
```
static getDerivedStateFromProps(props, state)
```

`getDerivedStateFromProps` 会在调用 `render` 方法之前调用，并且在初始挂载及后续更新时都会被调用。它应返回一个对象来更新 `state`，但是后续对 `state` 进行设置将不会触发更新。如果返回 `null` 则不更新任何内容


### shouldComponentUpdate

```js
shouldComponentUpdate(nextProps, nextState)
```

根据 `shouldComponentUpdate()` 的返回值，判断 React 组件的输出是否受当前 `state` 或 `props` 更改的影响。默认行为是 `state` 每次发生变化组件都会重新渲染。大部分情况下，你应该遵循默认行为

该方法可以用来进行性能优化。

可以将 `this.props` 与 `nextProps` 以及 `this.state` 与 `nextState` 进行比较，并返回 `false` 以告知 `React` 可以跳过更新。请注意，返回 `false` 并不会阻止子组件在 `state` 更改时重新渲染


### render

`render()` 方法是 `class` 组件中唯一必须实现的方法

### getsnapshotbeforeupdate

语法
```js
getSnapshotBeforeUpdate(prevProps, prevState)
```

`getSnapshotBeforeUpdate()` 在最近一次渲染输出（提交到 DOM 节点）之前调用

该生命周期的任何返回值将作为参数传递给 `componentDidUpdate()`

>此用法并不常用

### componentDidUpdate

语法
```
componentDidUpdate(prevProps, prevState, snapshot)
```

`componentDidUpdate()` 会在更新后会被立即调用。首次渲染不会执行此方法

在这里，可以根据新的值做些什么。比如，更新数据，更新ui

>可以在这里直接调用 `setState`，但是必须被包裹在一个条件语句里


## unmount

当组件从 DOM 中移除时会调用如下方法：

- `componentWillUnmount()`

## error-handling

当渲染过程，生命周期，或子组件的构造函数中抛出错误时，会调用如下方法：

- [`static getDerivedStateFromError()`](#getderivedstatefromerror)
- [`componentDidCatch()`](#componentdidcatch)

### getDerivedStateFromError

语法
```js
static getDerivedStateFromError(error)
```

该方法将抛出的错误作为参数。可以利用错误返回值，返回一个值以更新 `state`

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染可以显降级 UI
    return { hasError: true };
  }

  render() {
    if (this.state.hasError) {
      // 可以渲染任何自定义的降级  UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```


### componentDidCatch

语法
```js
componentDidCatch(error, info)
```

此生命周期在后代组件抛出错误后被调用

```js
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    // 更新 state 使下一次渲染可以显示降级 UI
    return { hasError: true };
  }

  componentDidCatch(error, info) {
    // "组件堆栈" 例子:
    //   in ComponentThatThrows (created by App)
    //   in ErrorBoundary (created by App)
    //   in div (created by App)
    //   in App
    logComponentStackToMyService(info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      // 你可以渲染任何自定义的降级 UI
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```
