# react-router-dom

- [基本demo](#project-construct)
- [render动态组件](#render)

## project-construct

项目结构
```
|-src/
|---Article/              // 组件--start--
|-----detail.js
|-----index.js
|---decorators/           // 装饰器
|-----withCopyright.js    // 页面底部copyright
|---Home/
|-----index.js
|---Login/
|-----index.js
|---Another.js            // 装饰器的写法
|---App.js                // 组件--end--
|---index.js              // 启动程序
|---router.js             // 路由配置
```

- `Article`

`Article/detai.js`
```js
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

`Article/index.js` 文章列表页组件
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

- `Home`

`Home/index.js`
```js
import React from 'react';

export default class Home extends React.Component {
  render() {
    return <div>home</div>;
  }
}

- `Login`

`Login/index.js`
```js
import React from 'react';

export default class Login extends React.Component {
  render() {
    console.log(this.props);
    return <div>login</div>;
  }
}

- `Another.js`

`Another.js` 装饰器组件
```js
import React from 'react';
import PropTypes from 'prop-types';
import withCopyright from './decorators/withCopyright';

@withCopyright
class Another extends React.Component {
  render() {
    return <div>{this.props.name}</div>;
  }
}

Another.propTypes = {
  name: PropTypes.string
};

export default Another;

```

- `App.js`

直接调用方法，与装饰器稍有差异
```js
import React from 'react';

import withCopyright from './decorators/withCopyright';

function App() {
  return <div className="App">App</div>;
}

export default withCopyright(App);

```

- `index.js`

```js
import React from 'react';
import ReactDOM from 'react-dom';
import RouterConfig from './router';

ReactDOM.render(
  <React.StrictMode>
    <RouterConfig/>
  </React.StrictMode>,
  document.getElementById('root')
);

```

- `router.js`

路由配置
```js
import React from 'react';
import { BrowserRouter as Router, Switch, Route } from 'react-router-dom';
import Home from './Home';
import Article from './Article';
import Detail from './Article/detail';
import App from './App';
import Login from './Login';
import Another from './Another';

export default class RouterConfig extends React.Component {
  render() {
    return <Router>
      <Switch>
        <Route path={'/login'} component={Login}/>
        <Route path={'/app'} component={App}/>
        <Route path={'/another'} component={Another} name={'another-name'}/>
        <Route path={'/home'} component={Home}/>
        <Route path={'/article'} component={Article} exact/>
        <Route path={'/article/:id'} component={Detail}/>
      </Switch>
    </Router>;
  }
}

```

- `withCopyright.js`

```js
import React from 'react';

const withCopyright = Cmp => {
  return class WithCopyright extends React.Component {
    render() {
      return (
        <div className={'withCopyright'}>
          <Cmp {...this.props} />
          <div>WithCopyright</div>
        </div>
      );
    }
  };
};

export default withCopyright;

```

## render

动态渲染组件


