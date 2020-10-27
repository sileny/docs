# lazy

懒加载，可以使用 `react-loadable` 组件

```
yarn add react-loadable
```

或

```
npm i react-loadable
```

## example

`LoadableComponent` 组件
```js
import Loadable from 'react-loadable';
import Loading from './Loading';
 
const LoadableComponent = Loadable({
  loader: () => import('./App'),
  loading: Loading,
});
 
export default class App extends React.Component {
  render() {
    return <LoadableComponent/>;
  }
}
```

`App.js`
```js
function App() {
  return <div>app</div>;
}
```

`Loading.js` 组件
```js
function Loading(props) {
  if (props.error) {
    return <div>Error! <button onClick={ props.retry }>Retry</button></div>;
  } else if (props.pastDelay) {
    return <div>Loading...</div>;
  } else {
    return null;
  }
}
```

具体用法参考 [npm文档](https://www.npmjs.com/package/react-loadable) 或 [github文档](https://github.com/thejameskyle/react-loadable)


# 自定义一个懒加载组件

```js
import React, { Component } from 'react';

const Loadable = ({ loader, loading: Loading }) => {
  return class LoadableComponent extends Component {
    state = {
      LoadedComponent: null
    };

    componentDidMount() {
      // import('./Dashboard')
      loader()
        .then(resp => {
          this.setState({
            LoadedComponent: resp.default
          });
        });
    }

    render() {
      const { LoadedComponent } = this.state;
      return LoadedComponent ? <LoadedComponent/> : <Loading/>;
    }
  };
};
```

`loader()` 实际就是一个函数式组件

在 `LoadableComponent` 组件完成渲染时，加载 `loader` 组件，然后，`render` 懒 `load` 到的 `LoadedComponent` 组件
