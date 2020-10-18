# react

- [支持装饰器写法](#decorators)
- [添加git提交时的代码校验git-husky](#git-husky)

## decorators

首先，`cra` 支持装饰器的写法，需要将配置进行修改

```
yarn eject
```

然后，在 `package.json` 的 `babel` 添加 `plugins` 配置
```json
  "babel": {
    "plugins": [
      ["@babel/plugin-proposal-decorators", { "legacy": true }]
    ]
  }
```

不支持装饰器的写法
```js
import React from 'react';
import PropTypes from 'prop-types';
import withCopyright from './withCopyright';

class Another extends React.Component {
  render() {
    return <div>{this.props.name}</div>;
  }
}

Another.propTypes = {
  name: PropTypes.string
};

export default withCopyright(Another);

```

改成支持装饰器的写法如下，
```js
import React from 'react';
import PropTypes from 'prop-types';
import withCopyright from './withCopyright';

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

高阶组件代码如下，
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

然后，运行 `yarn start`

## git-husky

详细的说明参考[这里](https://github.com/sileny/docs/tree/master/git#githooks)

初始化仓库
```
git init
```

安装依赖
```
yarn add husky lint-staged -D
```

`husky` 在安装过程中会在 `.git/hooks` 文件夹中生成一系列的 git hook 脚本

在 `package.json` 添加如下配置

```json
"husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/**/*.{js,jsx,json}": [
      "prettier --write",
      "eslint --fix"
    ],
    "src/**/*.{css,scss,less,json,html,md,markdown}": [
      "prettier --write"
    ]
  },
```
