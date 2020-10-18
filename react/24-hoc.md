# hoc

- [closure闭包](#closure)
- [basic-hoc](#basic-hoc)
- [advanced-hoc](#advanced-hoc)

高阶组件：利用闭包原理暴露出组件

```js
const fn = x => {
  return y => {
    return x + y;
  };
};

// fn(1) 返回了一个函数，fn就是高阶函数
fn(1)(2); // 3
```

## basic-hoc

高阶组件实现：用一个函数包裹组件

```js
const withCopyright = Cmp => {
  return class WithCopyright extends React.Component {
    render() {
      return <div className={'withCopyright'}>
        <Cmp/>
        <div>WithCopyright</div>
      </div>;
    }
  };
};

function App() {
  return (
    <div className="App">
      App
    </div>
  );
}

export default withCopyright(App);
```

但是当需要父组件传值到内部的某个组件的时候，该做法就实现不了，在高阶组件内部需要将接收到的属性传递给子组件

## advanced-hoc

定义一个高阶组件，将接收到的属性传递给子组件
```js
import React from 'react';

const withCopyright = Cmp => {
  return class WithCopyright extends React.Component {
    render() {
      return <div className={'withCopyright'}>
        <Cmp {...this.props}/>
        <div>WithCopyright</div>
      </div>;
    }
  };
};

export default withCopyright;

```

定义一个简单的 `App` 组件
```
import React from 'react';

import withCopyright from './withCopyright';

function App() {
  return (
    <div className="App">
      App
    </div>
  );
}

export default withCopyright(App);
```

定义 `Another` 组件，该组件需要接收外部的传值

```js
import React from 'react';
import PropTypes from 'prop-types';
import withCopyright from './withCopyright';

class Another extends React.Component {
  render() {
    return <div>
      {this.props.name}
    </div>;
  }
}

Another.propTypes = {
  name: PropTypes.string
};

export default withCopyright(Another);

```

在 `index.js` 里使用这些组件
```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';
import Another from './Another';

ReactDOM.render(
  <React.StrictMode>
    <App />
    <Another name={1}/>
  </React.StrictMode>,
  document.getElementById('root')
);

serviceWorker.unregister();

```
