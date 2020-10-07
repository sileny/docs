# express

`express` 源码解读

`module.exports = require('./lib/express');` 指定了入口文件为 `express.js`

`express.js` 内部依赖了以下模块

```js
var bodyParser = require('body-parser');
var EventEmitter = require('events').EventEmitter;
var mixin = require('merge-descriptors');
var proto = require('./application');
var Route = require('./router/route');
var Router = require('./router');
var req = require('./request');
var res = require('./response');
```

- `body-parser` 是因为消息解析器的方法太多，单独从 `express` 抽离出来的一个模块，专门用来处理请求主体。
- `events` 是 `Node.js` 构建于惯用的异步事件驱动架构，通过触发器触发命名事件来调用函数。更多详细文档参考[events](http://nodejs.cn/api/events.html)
- `merge-descriptors` 通过 `Object.defineProperty` 用来扩展对象的属性。[例子](https://github.com/sileny/merge)
- `application` 是 `express` 的核心代碼
