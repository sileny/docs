# propTypes

添加类型校验
```jsx
class App extends React.Component {
  // 复杂数组类型
  static propTypes = {
    todos: PropTypes.arrayOf(PropTypes.shape({
      id: PropTypes.number.isRequired,
      title: PropTypes.string.isRequired,
      isCompleted: PropTypes.bool.isRequired
    })).isRequired
  };

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
