# jsx

```jsx
class App extends React.Component {
  render() {
    return (
      <div class="app">
        <h1 class="title">title</h1>
        <p>content</p>
      </div>
    )
  }
}
```

可以用 `React.createElement` 的方式写

```jsx
class App extends React.Component {
  render() {
    return (
      React.createElement(
        'div',
        { className: 'app' },
        React.createElement(
          'h1',
          { className: 'title' },
          'title'
        ),
        React.createElement(
          'p',
          null,
          'content'
        )
      )
    )
  }
}
```
