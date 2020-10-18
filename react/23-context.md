# context

在一个典型的 React 应用中，数据是通过 `props` 属性自上而下（由父及子）进行传递的，在复杂的项目里很繁杂，有可能只是需要实现局部的功能，却需要将数据通过 `props` 进行层层传递。

`Context` 的设计是为了实现全局的数据状态共享

- [class-to-context](#class-to-context)
- [React.createContext](#ReactcreateContext)
- [Context.Provider](#Contextprovider)
- [Class.contextType](#ClasscontextType)
- [Context.Consumer](#ContextConsumer)

## class-to-context

class 的写法
```js
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  // Toolbar 组件接受一个额外的“theme”属性，然后传递给 ThemedButton 组件。
  // 如果应用中每一个单独的按钮都需要知道 theme 的值，这会是件很麻烦的事，
  // 因为必须将这个值层层传递所有组件。
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
```

使用 `context`，可以避免中间元素传递 `props`

```js
// Context 可以让我们无须明确地传遍每一个组件，就能将值深入传递进组件树。
// 为当前的 theme 创建一个 context（“light”为默认值）。
const ThemeContext = React.createContext('light');

class App extends React.Component {
  render() {
    // 使用一个 Provider 来将当前的 theme 传递给以下的组件树。
    // 无论多深，任何组件都能读取这个值。
    // 在这个例子中，我们将 “dark” 作为当前的值传递下去。
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

// 中间的组件再也不必指明往下传递 theme 了。
function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 指定 contextType 读取当前的 theme context。
  // React 会往上找到最近的 theme Provider，然后使用它的值。
  // 在这个例子中，当前的 theme 值为 “dark”。
  static contextType = ThemeContext;
  // contextType 被重赋值为一个由 `React.createContext()` 创建的 `Context` 对象。
  // 这样，可以使用 `this.context` 来消费最近 `Context` 上的那个值

  render() {
    return <Button theme={this.context} />;
  }
}
```

## React.createContext

内部遵循观察者模式，返回一个 `Context` 对象。

语法
```js
const MyContext = React.createContext(defaultValue);
```

当 `React` 渲染一个订阅了这个 `Context` 的组件，这个组件会从组件树中匹配最近的 `Provider` 中读取 `context` 的值。

**只有**当组件所处的组件树里没有匹配到 `Provider` 时，`defaultValue` 才会生效

## Context.Provider

数据的提供方。也就是可以被订阅的数据源。

`Provider` 接收一个 `value` 属性，传递给消费者组件

- 一个 `Provider` 可以对应多个 `consumer`
- 当 `Provider` 的 `value` 值发生变化时，它内部的所有消费组件都会被重新渲染
- `Provider` 和内部的 `consumer` 组件都不受限制于 `shouldCompoentUpdate` 函数

## Class.contextType

挂载到 class 上的静态 `contextType` 属性会被重新赋值为一个由 `React.createContext` 创建的 `Context` 对象。这样，可以使用 `this.context` 来消费最近的 `Context` 上的那个值。可以在声明周期中访问到它，也可以在 `render` 函数里访问到

```js
class MyClass extends React.Component {
  componentDidMount() {
    let value = this.context;
    /* 在组件挂载完成后，使用 MyContext 组件的值来执行一些有副作用的操作 */
  }
  componentDidUpdate() {
    let value = this.context;
    /* ... */
  }
  componentWillUnmount() {
    let value = this.context;
    /* ... */
  }
  render() {
    let value = this.context;
    /* 基于 MyContext 组件的值进行渲染 */
  }
}
MyClass.contextType = MyContext;
```

## Context.Consumer

```js
<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

这里，需要函数作为子元素（`function as a child`）这种做法，函数接收当前的 `context` 值，返回一个 `React` 节点。传递给函数的 `value` 值等同于往上组件树离这个 `context` 最近的 `Provider` 提供的 `value` 值。如果没有对应的 `Provider`，`value` 参数等同于传递给 `createContext()` 的 `defaultValue`
