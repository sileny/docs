# react-router-dom

- [基本demo](#project-construct)
- [render动态组件，可以根据权限显示对应的组件](#render)
- [js导航](#js-nav)
- [使用withRouter写一个通用的回到首页导航组件](#nav-cmp)

## project-construct

初期项目结构
```
|-src/
|---decorators/           // 装饰器
|-----withCopyright.js    // 页面底部copyright
|---pages/
|-----Another/
|-------Another.js            // 装饰器的写法
|-----Article/              // 组件--start--
|-------detail.js
|-------index.js
|-----Home/
|-------index.js
|-----Login/
|-------index.js
|---router/
|-----index.js            // 路由配置
|-----util.js             // 辅助方法
|---App.js                // 组件--end--
|---index.js              // 启动程序
```

完整的案例在[这里](https://github.com/sileny/react-app-hoc)

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

`to={'/article/1'}` 是显式传参给 `Detail` 组件，在 `Detail` 组件里可以通过 `this.props.match.props` 来获取数据

隐式传参，借助于 `Link` 的 `to` 的 `state` 属性，案例[在此](#send-data-to-component)


- `Home`

`Home/index.js`
```js
import React from 'react';

export default class Home extends React.Component {
  render() {
    return <div>home</div>;
  }
}
```

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
```

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

- `router/index.js`

路由配置
```js
import React from 'react';
import { BrowserRouter as Router, Switch, Route, NavLink as Link } from 'react-router-dom';
import Home from '../pages/Home';
import Article from '../pages/Article';
import Detail from '../pages/Article/detail';
import App from '../App';
import Login from '../pages/Login';
import Another from '../pages/Another';

export default class RouterConfig extends React.Component {
  render() {
    return (
      <Router>
        <ul>
          <li>
            <Link to={'/login'}>login</Link>
          </li>
          <li>
            <Link to={'/app'}>app</Link>
          </li>
          <li>
            <Link to={'/another'}>another</Link>
          </li>
          <li>
            <Link to={'/home'}>home</Link>
          </li>
          <li>
            <Link to={'/article'}>article</Link>
          </li>
        </ul>
        <Switch>
          <Route path={'/login'} component={Login} />
          <Route path={'/app'} component={App} />
          <Route path={'/another'} component={Another} name={'another-name'} />
          <Route path={'/home'} component={Home} />
          <Route path={'/article'} component={Article} exact />
          <Route path={'/article/:id'} component={Detail} />
        </Switch>
      </Router>
    );
  }
}

```

>`<Route path={'/another'} component={Another} name={'another-name'}/>`渲染了 `Another` 组件，但是，`name={'another-name'}`并没有渲染出来，下面会用[两种方式](#render)解决此类问题

`Link` 的 `to` 参数可以是一个对象，案例参考[这里](#send-data-to-component)

`Switch` 表示匹配到第一个之后，不会匹配后续的路由

`<Route path={'/article'} component={Article} exact />` 的 `exact` 如果不加上，那么，`<Route path={'/article/:id'} component={Detail} />` 匹配的组件将不会显示出来，`exact` 表示严格匹配到 `/article` 路由后将只显示 `Article/index.js` 组件内容

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

动态渲染组件，可以根据权限显示对应的组件

- 方式一

```jsx
<Route path={'/dynamic'} render={() => <Another name={'another-dynamic-name'}/>}/>
```

`render` 属性为 `function` 类型


- 方式二

传参的形式

```jsx
<Route path={'/dynamic2'} render={routeProps => dynamicRender(Another, routeProps, { name: 'another-dynamic-name-2' })}/>
```

`router/util.js` 代码如下
```js
import React from 'react';

const dynamicRender = (Cmp, routeProps, options) => {
  return <Cmp {...routeProps} {...options} />;
};

export default dynamicRender;

```

用法如下，
```js
<Route
  path={'/dynamic-render-with-data'}
  render={routeProps => {
    return dynamicRender(Another, routeProps, { name: 'dynamicRender another name' });
  }}
/>
```

访问 `/dynamic-render-with-data` 路由即可看到效果

`dynamicRender` 方法实际上是对组件进行了一次封装


`render` 方法默认注入的 `routeProps` 参数为 `RouteComponentProps` 类型

```typescript
export interface RouteComponentProps<
    Params extends { [K in keyof Params]?: string } = {},
    C extends StaticContext = StaticContext,
    S = H.LocationState
> {
    history: H.History<S>;
    location: H.Location<S>;
    match: match<Params>;
    staticContext?: C;
}
```

`History` 具有以下方法，可以用作导航
```typescript
export interface History<HistoryLocationState = LocationState> {
    length: number;
    action: Action;
    location: Location<HistoryLocationState>;
    push(path: Path, state?: HistoryLocationState): void;
    push(location: LocationDescriptor<HistoryLocationState>): void;
    replace(path: Path, state?: HistoryLocationState): void;
    replace(location: LocationDescriptor<HistoryLocationState>): void;
    go(n: number): void;
    goBack(): void;
    goForward(): void;
    block(prompt?: boolean | string | TransitionPromptHook<HistoryLocationState>): UnregisterCallback;
    listen(listener: LocationListener<HistoryLocationState>): UnregisterCallback;
    createHref(location: LocationDescriptorObject<HistoryLocationState>): Href;
}
```

`Location` 反映了当前的 `location` 信息

```typescript
export interface Location<S = LocationState> {
    pathname: Pathname;
    search: Search;
    state: S;
    hash: Hash;
    key?: LocationKey;
}
```

`match` 匹配到的路由信息

```typescript
export interface match<Params extends { [K in keyof Params]?: string } = {}> {
    params: Params;
    isExact: boolean;
    path: string;
    url: string;
}
```

## send-data-to-component

- `query`
- 动态路由 `params`
- `Link.to.state`

下面介绍 `Link` 这种隐式的传参，由于不可见，所以该方式可以用作 [埋点](https://github.com/sileny/docs/tree/master/bury)

继续修改上面的 `Article/detail.js`，完整代码如下，
```js
import React from 'react';
import PropTypes from 'prop-types';

export default class Detail extends React.Component {
  static propTypes = {
    match: PropTypes.shape({
      params: PropTypes.shape({
        id: PropTypes.oneOfType([PropTypes.string, PropTypes.number]) // id可以实数字类型，也可以是字符串类型
      })
    }),
    location: PropTypes.shape({
      state: PropTypes.shape({
        flag: PropTypes.oneOfType([PropTypes.string, PropTypes.number]) // id可以实数字类型，也可以是字符串类型
      })
    })
  };

  render() {
    const { match, location } = this.props;
    return <div>
      <div>params.id -- {match.params.id}</div>
      <div>state.flag -- {location.state.flag}</div>
    </div>;
  }
}

```

路由添加了如下代码，
```js
<Link to={{
  pathname: '/article/3',
  state: {
    flag: '123'
  }
}}
>Link.to.state</Link>
```

路由完整代码如下，
```js
import React from 'react';
import { BrowserRouter as Router, Switch, Route, NavLink as Link } from 'react-router-dom';
import Home from '../pages/Home';
import Article from '../pages/Article';
import Detail from '../pages/Article/detail';
import App from '../App';
import Login from '../pages/Login';
import Another from '../pages/Another';

export default class RouterConfig extends React.Component {
  render() {
    return (
      <Router>
        <ul>
          <li>
            <Link to={'/login'}>login</Link>
          </li>
          <li>
            <Link to={'/app'}>app</Link>
          </li>
          <li>
            <Link to={'/another'}>another</Link>
          </li>
          <li>
            <Link to={'/home'}>home</Link>
          </li>
          <li>
            <Link to={'/article'}>article</Link>
          </li>
          <li>
            <Link to={{
              pathname: '/article/3',
              state: {
                flag: '123'
              }
            }}
            >Link.to.state</Link>
          </li>
        </ul>
        <Switch>
          <Route path={'/login'} component={Login} />
          <Route path={'/app'} component={App} />
          <Route path={'/another'} component={Another} name={'another-name'} />
          <Route path={'/home'} component={Home} />
          <Route path={'/article'} component={Article} exact />
          <Route path={'/article/:id'} component={Detail} />
        </Switch>
      </Router>
    );
  }
}

```

## js-nav

`router/index.js` 添加如下代码

```
<li>
  <Link to={'/js-nav'}>js-nav</Link>
</li>
```

在 `pages` 目录下添加 `Nav/JsNav.js`，代码如下
```js
import React from 'react';
import PropTypes from 'prop-types';

export default class JsNav extends React.Component {
  static propTypes = {
    history: PropTypes.shape({
      push: PropTypes.func
    })
  };

  render() {
    return (
      <div>
        <button onClick={() => this.props.history.push('/home')}>this.props.history.push(&apos;/home&apos;)</button>
      </div>
    );
  }
}

```

`push` 方法的参数和 `Link` 的 `to` 的属性是一样的

所以，也可以写成这样
```js
<button
  onClick={() => {
    this.props.history.push({
      pathname: '/home',
      state: {
        flag: 'btn-to-home'
      }
    });
  }}
>
  go home
</button>
```

## nav-cmp

目的：在任意一个组件里都可以直接使用该组件返回到 `/home` 页面

```js
import React from 'react';
import PropTypes from 'prop-types';
import { withRouter } from 'react-router-dom';

@withRouter
class BackHome extends React.Component {
  static propTypes = {
    history: PropTypes.shape({
      push: PropTypes.func
    })
  };

  render() {
    return (
      <button
        onClick={() => {
          this.props.history.push({
            pathname: '/home',
            state: {
              flag: 'btn-to-home'
            }
          });
        }}
      >
        withRouter go home
      </button>
    );
  }
}

export default BackHome;

```

可以将 `<BackHome/>` 嵌入到 `JsNav.js` 里，同样可以实现回退到 `/home`

基于以上按钮，可以写一个跳转到任意路由的组件

```js
import React from 'react';
import PropTypes from 'prop-types';
import { withRouter } from 'react-router-dom';

@withRouter
class PageButton extends React.Component {
  static propTypes = {
    history: PropTypes.shape({
      push: PropTypes.func
    }),
    pathname: PropTypes.string,
    state: PropTypes.object
  };

  render() {
    const { history, pathname, state } = this.props;
    return (
      <button
        onClick={() => {
          history.push({
            pathname,
            state
          });
        }}
      >
        withRouter PageButton 可以条件到任意的路由页面
      </button>
    );
  }
}

export default PageButton;

```

当然，也可以使用 `withRouter` 方法来包裹 `PageButton` 组件，写法如下，
```js
export default withRouter(PageButton);
```
