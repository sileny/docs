# slot

用 `this.props.children` 实现插槽

## render-children
`ParentCmp.js`
```jsx
export default class ParentCmp extends React.Component {
  render() {
    return (
      <div><h1>插槽</h1></div>
    )
  }
}
```

`App.jsx`
```jsx
export default class App extends React.Component {
  render() {
    return (
      <ParentCmp>
        <h3>标题1</h3>
        <h3>标题2</h3>
        <h3>标题3</h3>
      </ParentCmp>
    )
  }
}
```

此时，`ParentCmp` 里面的几个 `h3` 内容不会显示出来，它们被放在 `ParentCmp` 组件的 `this.props.children` 里，将 `ParentCmp.js` 改为如下，
```jsx
export default class ParentCmp extends React.Component {
  render() {
    return (
      <div>
        <h1>插槽</h1>
        {this.props.children}
      </div>
    )
  }
}
```
里面的组件将会展示出来。

## render-named-components

`App.jsx` 里为子组件分别起名为 `header`、`body`、`footer`
```jsx
export default class App extends React.Component {
  render() {
    return (
      <ParentCmp>
        <h3 data-name="header">标题1</h3>
        <h3 data-name="body">标题2</h3>
        <h3 data-name="footer">标题3</h3>
      </ParentCmp>
    )
  }
}
```

需要在 `ParentCmp.jsx` 里进行过滤

```jsx
export default class ParentCmp extends React.Component {
  render() {
    const { children } = this.props;
    const header = children.find(value => value.props['data-name'] === 'header');
    const body = children.find(value => value.props['data-name'] === 'body');
    const footer = children.find(value => value.props['data-name'] === 'footer');
    return (
      <div>
        <h1>插槽</h1>
        <div className="header">{ header }</div>
        <div className="body">{ body }</div>
        <div className="footer">{ footer }</div>
      </div>
    )
  }
}
```
