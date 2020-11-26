# 离线存储

`localForage` 是一个 js 库，通过简单类似 `localStorage` API 的异步存储来改进离线存储

特性：
- 能存储多种类型的数据，不仅仅是字符串
- 优雅降级的策略，如果浏览器不支持 `indexedDB` 或 `WebSQL`，则使用 `localStorage`。

# 安装

- npm
```
npm install localforage
```

- bower
```
bower install localforage
```

- 浏览器端
```js
<script src="localforage.js"></script>
<script>console.log('localforage is: ', localforage);</script>
```

# API

- [getItem](#getitem)
- [setItem](#setitem)
- [removeItem](#removeitem)
- [clear](#clear)
- [length](#length)
- [key](#key)
- [keys](#keys)
- [iterate](#iterate)
- [setDriver](#setdriver)
- [config](#config)
- [customDriver](#customdriver)
- [driver](#driver)
- [ready](#ready)
- [supports](#supports)
- [createInstance](#createinstance)
- [dropInstance](#dropinstance)

## getItem

```js
getItem(key, successCallback);
```

获取 `key` 对应的值。如果 `key` 不存在，返回 `null`

>当存储 `undefined` 时，`getItem()` 的结果也会返回 `null`。由于 `localStorage` 限制和处于兼容性原型，`localStorage` 无法存储 `undefined`

```js
localforage.getItem('somekey').then(function(value) {
  // 当离线仓库中的值被载入时，此处代码运行
  console.log(value);
}).catch(function(err) {
  // 当出错时，此处代码运行
  console.log(err);
});
```

也可以写为回调版本
```js
// 回调版本：
localforage.getItem('somekey', function(err, value) {
  // 当离线仓库中的值被载入时，此处代码运行
  console.log(value);
});
```


## setItem

```js
setItem(key, vlaue, successCallback);
```

存储数据

可以存储的数据类型：
- Array
- ArrayBuffer
- Blob
- Float32Array
- Float64Array
- Int8Array
- Int16Array
- Int32Array
- Number
- Object
- Uint8Array
- Uint8ClampedArray
- Uint16Array
- Uint32Array
- String

>当使用 `localStorage` 或 `WebSQL` 时，二进制数据会被序列化，导致空间占用增大。

```js
localforage.setItem('somekey', 'some value').then(function (value) {
  // 当值被存储后，可执行其他操作
  console.log(value);
}).catch(function(err) {
  // 当出错时，此处代码运行
  console.log(err);
});

// 不同于 localStorage，还可以存储非字符串类型
localforage.setItem('my array', [1, 2, 'three']).then(function(value) {
  // 如下输出 `1`
  console.log(value[0]);
}).catch(function(err) {
  // 当出错时，此处代码运行
  console.log(err);
});

// 可以存储 AJAX 响应返回的二进制数据
req = new XMLHttpRequest();
req.open('GET', '/photo.jpg', true);
req.responseType = 'arraybuffer';

req.addEventListener('readystatechange', function() {
  if (req.readyState === 4) { // readyState 完成
    localforage.setItem('photo', req.response).then(function(image) {
      // 如下为一个合法的 <img> 标签的 blob URI
      var blob = new Blob([image]);
      var imageURI = window.URL.createObjectURL(blob);
    }).catch(function(err) {
      // 当出错时，此处代码运行
      console.log(err);
    });
  }
});
```


## removeItem

```js
removeItem(key, successCallback);
```

删除 `key` 对应的值。

```js
localforage.removeItem('key').then(function() {
  console.log('key对应的值被成功移除掉');
}).catch(error => {
  // 错误信息输出
  console.log(error);
})
```


## clear

```
clear(successCallback)
```

删除数据库中的所有的 `key`，重置数据库。

```js
localforage.clear().then(function() {
  // 成功删除全部的数据
  console.log('database is now empty');
}).catche(error => {
  // 错误信息输出
  console.log(error);
});
```


## length

获取 `key` 的个数。

```js
localforage.length().then(function(len) {
  // 数据库key的个数
  console.log(len);
}).catch(error => {
  // 错误
  console.log(error);
});
```


## key

根据 `key` 的索引获取值

```js
localforage.key(2).then(function(keyName) {
  // key 名
  console.log(keyName);
}).catch(function(err) {
  // 当出错时，此处代码运行
  console.log(err);
});
```


## keys

返回数据仓库中的所有的 `key`

```js
localforage.keys().then(function(keys) {
  // 包含所有 key 名的数组
  console.log(keys);
}).catch(function(err) {
  // 当出错时，此处代码运行
  console.log(err);
});
```


## iterate

```js
iterate(iteratorCallback, successCallback)
```

迭代数据仓库中的所有的键值对。

`iteratorCallback` 在每个键值对上都对调用一次。格式如下，
- value 第一参数
- key 第二参数
- iteratorNumber 第三参数

>通过在 `iteratorCallback` 回调函数中返回一个非 `undefined` 的值，能提前退出 `iterate`。`iteratorCallback` 返回的值就是正地迭代返回的结果，将被传入 `successCallback`

```js
// 同样的代码，但使用 ES6 Promises
localforage.iterate(function(value, key, iterationNumber) {
  // 此回调函数将对所有 key/value 键值对运行
  console.log([key, value]);
}).then(function() {
  console.log('Iteration has completed');
}).catch(function(err) {
  // 当出错时，此处代码运行
  console.log(err);
});
```

提前退出迭代
```js
// 提前退出迭代：
localforage.iterate(function(value, key, iterationNumber) {
  if (iterationNumber < 3) {
      console.log([key, value]);
  } else {
      return [key, value];
  }
}).then(function(result) {
  console.log('Iteration has completed, last iterated pair:');
  console.log(result);
}).catch(function(err) {
  // 当出错时，此处代码运行
  console.log(err);
});
```


## setDriver

```js
// 使用driverName
setDriver(driverName)

// 设置使用驱动的顺序
setDriver([driverName, nextDriverName])
```

设置驱动的顺序

默认情况下，`localForage` 按照以下顺序选择数据仓库的后端驱动：

1. IndexdDB
2. WebSQL
3. localStorage

如果想使用指定的的驱动，可以使用 `setDriver()`，参数可以是以下的一个或者是多个
- localforage.INDEXEDDB
- localforage.WEBSQL
- localforage.LOCALSTORAGE

>如果你尝试加载的后端驱动在浏览器上不可用，`localForage` 将使用以前使用的后端驱动中的一个。这意味着如果你试图强制 `Gecko` 浏览器使用 `WebSQL`，则会失败并继续使用 `IndexedDB`

```js
// 强制设置 localStorage 为后端的驱动
localforage.setDriver(localforage.LOCALSTORAGE);

// 列出可选的驱动，以优先级排序
localforage.setDriver([localforage.WEBSQL, localforage.INDEXEDDB]);
```


## config

设置 `localForage` 选项。在调用 `localForage` 前必先调用它，但可以在 `localForage` 被加载后调用。使用此方法设置的任何配置值都将保留，即使在驱动更改后，所以你也可以先调用 `config()` 再次调用 `setDriver()`

可选配置：
- driver

要使用的首选驱动。与上面的 `setDriver` 的值格式相同。

默认值：`[localforage.INDEXEDDB, localforage.WEBSQL, localforage.LOCALSTORAGE]`

- name

数据库的名称。一般对应的是程序的名称。默认值是 `localforage`

- size

数据库的大小（以字节为单位）。默认值为 `4980736`

- storeName

数据仓库的名称。在 `IndexedDB` 中为 `dataStore`；在 `WebSQL` 中为 `key/value` 的名称。默认值为 `keyvalpairs`

- version

数据库的版本。可用于升级

- description

数据库的一些描述信息。

```js
// 将数据库从 “localforage” 重命名为 “Hipster PDA App”
localforage.config({
  name: 'Hipster PDA App'
});

// 将强制使用 localStorage 作为存储驱动，即使其他驱动可用。
// 可用配置代替 `setDriver()`。
localforage.config({
  driver: localforage.LOCALSTORAGE,
  name: 'I-heart-localStorage'
});

// 配置不同的驱动优先级
localforage.config({
  driver: [localforage.WEBSQL,
    localforage.INDEXEDDB,
    localforage.LOCALSTORAGE],
  name: 'WebSQL-Rox'
});
```


## customdriver

自定义实现包含一个 `_support` 属性，类型为 `boolean`，或者返回一个 `Promise`，该 `Promise` 的结果返回布尔值。如果省略了 `_support`，则默认值是是 `true`。可以用它来标识当前的浏览器支持自定义的驱动。
```js
// 此处为驱动的实现
var myCustomDriver = {
    _driver: 'customDriverUniqueName',
    _initStorage: function(options) {
        // 在此处自定义实现...
    },
    clear: function(callback) {
        // 在此处自定义实现...
    },
    getItem: function(key, callback) {
        // 在此处自定义实现...
    },
    key: function(n, callback) {
        // 在此处自定义实现...
    },
    keys: function(callback) {
        // 在此处自定义实现...
    },
    length: function(callback) {
        // 在此处自定义实现...
    },
    removeItem: function(key, callback) {
        // 在此处自定义实现...
    },
    setItem: function(key, value, callback) {
        // 在此处自定义实现...
    }
}

// 为 localForage 添加驱动。
localforage.defineDriver(myCustomDriver);
```


## driver

`localforage.driver()` 返回当前正在使用的驱动的名称，在异步的程序中返回 `null`，若初始化未能找到可用的驱动也为 `null`

```js
localforage.driver();
```


## ready

`localForage` 会对所有的数据 api 方法调用进行缓冲排序，`ready()` 方法可以确定异步程序初始化是否完成。在此处，可以得知当前使用的是哪一个驱动。

```js
localforage.ready().then(function() {
  // 当 localforage 将指定驱动初始化完成时，此处代码运行
  console.log(localforage.driver()); // locaStorage
}).catch(function() {
  // 当没有可用的驱动时，`ready()` 将会失败
  console.log(e);
});
```


## supports
```
supports(driverName)
```

返回一个 `boolean` 类型的值，表示是否支持 `driverName`

```js
localforage.supports(localforage.INDEXEDDB); // true
```


## createInstance
创建并返回一个 `localForage` 实例。每个实例都有独立的数据库，彼此之间互不影响。

```js
const store = localforage.createInstance({
  name: 'store'
});

const otherStore = localforage.createInstance({
  name: 'otherStore'
});

// 设置某个数据仓库 key 的值不会影响到另一个数据仓库
store.setItem('key', 'value');
otherStore.setItem('key', 'value');
```


## dropInstance

- 调用时，若不传参，将删除**当前实例**的 `数据仓库`。
```js
localforage.dropInstance().then(function() {
  console.log(`dropped the store of the current instance`);
});
```

- 调用时，若参数指定了 `name` 和 `storeName` 属性的对象，会删除指定的 `数据仓库`
```js
localforage.dropInstance({
  name: 'otherName',
  storeName: 'otherStore'
}).then(function() {
  console.log(`dropped the otherStore`);
});
```

- 调用时，若仅指定了 `name` 属性对象，则删除指定的 `数据库`（包含所有的数据仓库）
```js
localforage.dropInstance({
  name: 'otherName'
}).then(function() {
  console.log(`dropped the otherName database`);
});
```
