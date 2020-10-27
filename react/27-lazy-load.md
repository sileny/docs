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


