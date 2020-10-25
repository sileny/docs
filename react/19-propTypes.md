# propTypes

- [basic](#basic)
- [shape](#shape)

## basic
添加类型校验
```jsx
import React from 'react';
import PropTypes from 'prop-types';

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

## shape
复杂对象类型

父路由组件
```js
import React from 'react';
import { NavLink as Link } from 'react-router-dom';

export default class Article extends React.Component {
  render() {
    return <ul>
      <li><Link to={'/article/1'}>article 1</Link></li>
      <li><Link to={'/article/2'}>article 2</Link></li>
    </ul>;
  }
}

```
接收父路由组件传递的id
``jsx
import React from 'react';
import PropTypes from 'prop-types';

export default class Detail extends React.Component {
  static propTypes = {
    match: PropTypes.shape({
      params: PropTypes.shape({
        id: PropTypes.oneOfType([PropTypes.string, PropTypes.number]) // id可以实数字类型，也可以是字符串类型
      })
    })
  };

  render() {
    const { match } = this.props;
    return <div>detail -- {match.params.id}</div>;
  }
}

```
