# serviceWorker

**1、 什么是 Service Worker**

Service Worker 是 Chrome 团队提出和力推的一个 WEB API，用于给 web 应用提供高级的可持续的后台处理能力。该 WEB API 标准起草于 2013 年，于 2014 年纳入 W3C WEB 标准草案，当前还在草案阶段。

Service Worker 最主要的特点是：在页面中注册并安装成功后，运行于浏览器后台，不受页面刷新的影响，可以监听和截拦作用域范围内所有页面的 HTTP 请求。

基于 `Service Worker API` 的特性，结合 `Fetch API`、`Cache API`、`Push API`、`postMessage API` 和 `Notification API`，可以在基于浏览器的 web 应用中实现如离线缓存、消息推送、静默更新等 native 应用常见的功能，以给 web 应用提供更好更丰富的使用体验。

1.1 Service Worker 特点

- 网站必须使用 HTTPS。除了使用本地开发环境调试时(如域名使用 localhost)
- 运行于浏览器后台，可以控制打开的作用域范围下所有的页面请求
- 单独的作用域范围，单独的运行环境和执行线程
- 不能操作页面 DOM。但可以通过事件机制来处理

1.2 Service Worker 浏览器支持情况

除了 Safari 和 IE/Edge，大部分现代浏览器都已经得到了支持。

请参考：http://caniuse.com/#feat=serviceworkers

1.3 什么是 PWA

谷歌给以 Service Worker API 为核心实现的 web 应用取了个高大上的名字：Progressive Web Apps（PWA，渐进式增强 WEB 应用），并且在其主要产品上进行了深入的实践。那么，符合 PWA 的应用特点是什么？以下为来自谷歌工程师的解答。

Progressive Web Apps 是:

- 渐进增强 – 能够让每一位用户使用，无论用户使用什么浏览器，因为它是始终以渐进增强为原则。
- 响应式用户界面 – 适应任何环境：桌面电脑，智能手机，笔记本电脑，或者其他设备。
- 不依赖网络连接 – 通过 Service Workers 可以在离线或者网速极差的环境下工作。
- 类原生应用 – 有像原生应用般的交互和导航给用户原生应用般的体验，因为它是建立在 app shell model 上的。
- 持续更新 – 受益于 Service Worker 的更新进程，应用能够始终保持更新。
- 安全 – 通过 HTTPS 来提供服务来防止网络窥探，保证内容不被篡改。
- 可发现 – 得益于 W3C manifests 元数据和 Service Worker 的登记，让搜索引擎能够找到 web 应用。
- 再次访问 – 通过消息推送等特性让用户再次访问变得容易。
- 可安装 – 允许用户保留对他们有用的应用在主屏幕上，不需要通过应用商店。
- 可连接性 – 通过 URL 可以轻松分享应用，不用复杂的安装即可运行。

如有兴趣，可进一步研究一下 Google plus 相关的表现。

**2、Service Worker 的生命周期**

参考 Service Worker 的生命周期，使用 Service Worker 大概需要如下几个过程。

```
install -> installed -> actvating -> Active -> Activated -> Redundant
```

特别说明，进入 Redundant (废弃)状态的原因可能为这几种：

- 安装(install)失败
- 激活(activating)失败
- 新版本的 Service Worker 替换了它并成为激活状态

**3、使用 Service Worker**

首先要注意如下两点：

A. 使用 HTTPS 访问，并且 SSL 证书要正确。关于 https 和 SSL 证书的内容，可以参考：

```
https://lzw.me/a/lets-encrypt-ssl.html
https://lzw.me/a/http2-nginx.html
```
不过域名使用 `localhost` 这种在本机开发的情况下，也是可以进行测试的。

B. 你应该对如下基础知识也有所了解：

- Promise: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise
Promise 是 ES6 新增的规范，主要用于 javasCript 的异步处理。但是这个概念在很早以前就在很多开源库中使用了。

- Fetch API: https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API

Fetch API 用于 HTTP 请求处理，可以替代 XMLHttpRequest 实现异步请求(ajax)，但功能上更为完善。

- Cache API: https://developer.mozilla.org/zh-CN/docs/Web/API/Cache

`Cache API` 用于对 HTTP 请求进行缓存管理，是在 `ServiceWorker` 的规范中定义的，一般也跟 `ServiceWorker` 一起使用，是实现离线应用的关键。但是 `Cache API` 又不依赖于 `Service Worker`，可以单独使用。


**3.1 注册**

在网站页面上注册实现 `Service Worker` 功能逻辑的脚本。例如注册 `/sw/sw.js` 文件，参考代码：

```js
if ('serviceWorker' in navigator) {
    navigator.serviceWorker
        .register('/sw/sw.js', {scope: '/'})
        .then(registration => console.log('ServiceWorker 注册成功！作用域为: ', registration.scope))
        .catch(err => console.log('ServiceWorker 注册失败: ', err));
}
```

其实关键代码只有一行：

```js
navigator.serviceWorker.register('/sw/sw.js')
```

chrome 浏览器下，注册成功后，可以打开 `chrome://serviceworker-internals/` 查看浏览器的 `Service Worker` 信息。

**注意：**

- `Service Worker` 的路径决定了其 `scope` 的范围。

示例中 `sw.js` 是在 `/sw/` 路径下，这使得该 `Service Worker` 默认只会收到 `/sw/` 路径下的 `fetch` 事件。如果存放在网站的根路径下，则将会收到该网站的所有 `fetch` 事件。

- `Service Worker` 没有页面作用域的概念

作用域范围内的所有页面请求都会被**当前激活的 Service Worker 所监控**

**3.2 安装**

前一步在页面中仅注册了 `sw.js` 脚本，具体的逻辑行为则在 `sw.js` 内实现。那么这里面要做什么呢？参考示例：

```js
// 用于标注创建的缓存，也可以根据它来建立版本规范
const CACHE_NAME = "lzwme_cache_v1.0.0";
// 列举要默认缓存的静态资源，一般用于离线使用
const urlsToCache = [
    '/offline.html',
    '/offline.png'
];
 
// self 为当前 scope 内的上下文
self.addEventListener('install', event => {
    // event.waitUtil 用于在安装成功之前执行一些预装逻辑
    // 但是建议只做一些轻量级和非常重要资源的缓存，减少安装失败的概率
    // 安装成功后 ServiceWorker 状态会从 installing 变为 installed
    event.waitUntil(
        // 使用 cache API 打开指定的 cache 文件
        caches.open(CACHE_NAME).then(cache => {
            console.log(cache);
            // 添加要缓存的资源列表
            return cache.addAll(urlsToCache);
        })
    );
});
```

可以看到，示例中在文件 `sw.js` 内监听了 `install` 事件。当 `sw.js` 被安装时会触发 `install` 事件，那么，可以在监听该事件的时候执行安装时需要做的事情。本案例中是用于缓存静态资源，也是最常见的行为。

>只有 `urlsToCache` 中的文件全部安装成功，`service worker` 才会被认为是完全安装成功。否则，会认为安装失败，安装失败进入 `redundant` （废弃）状态。所以，这里应当尽量少缓存资源（一般缓存离线需要，联网时不需要的资源），以便提高成功率。

安装成功后，即进入 `waiting` 或 `active` 状态，在激活状态可通过监听各种事件，实现比较复杂的需求。具体参见后文 `事件处理` 部分。


**3.3 Service Worker 的更新**

如果 `sw.js` 文件的内容有变动，当访问网站页面时浏览器获取了新的资源文件，会认为有更新，于是，会安装新的文件，并触发 `install` 事件。但此时，已经处于激活状态的 service worker 还在运行，新的 service worker 在安装成功后，进入 waiting 状态。直到所有的已经打开的页面全部关闭，旧的 service worker 自动停止，新的 service worker 会在接下来打开的页面里生效。

如果，希望在有的新的版本变化时，所有的页面都得到及时的更新，怎么办？

可以在 `install` 事件里执行 `skipWaiting` 方法跳过 `waiting` 状态，然后会直接进入 `activate` 阶段。接着在 `activate` 事件发生时，通过执行 `clients.claim` 声明方法，更新所有客户端上的 service worker。示例如下：

```js
// 安装阶段跳过等待，直接进入 active
self.addEventListener('install', function(event) {
    event.waitUntil(self.skipWaiting());
});
 
self.addEventListener('activate', event => event.waitUntil(
    Promise.all([
        // 更新客户端
        clients.claim(),
        // 清理旧版本
        caches.keys().then(cacheList => Promise.all(
            cacheList.map(cacheName => {
                if (cacheName !== CACHE_NAME) {
                    caches.delete(cacheName);
                }
            })
        ))
    ])
));
```

另外，注意一点，`sw.js` 文件可能会因为浏览器缓存问题，当文件有了变化后，浏览器还是有旧的文件，这会导致更新得不到响应，如遇到该问题，可尝试这么做：在 service worker 上添加对该文件的过滤规则，不缓存或设置较短的有效期。注意，不要想着不同版本使用不同的文件名称，这会带来混乱的问题，带来维护难度。

>关于 `clients.claim`，可以参考 https://developer.mozilla.org/zh-CN/docs/Web/API/Clients/claim

**手动更新**

其实在页面中，也可以手动来管理更新。参考如下示例：
```js
const version = '1.0.1';
 
navigator.serviceWorker.register('/sw.js').then(reg => {
    if (localStorage.getItem('sw_version') !== version) {
        reg.update().then(() => localStorage.setItem('sw_version', version));
    }
});
```

**3.4 Service Worker 相关事件处理**

在安装过程中实现了资源缓存，安装完成后进入了空闲阶段，此时，可以通过监听各个事件实现各种逻辑。下面对常见的相关事件做简单的介绍。

**3.4.1 install 事件**

当脚本被安装时，会出现 `install` 事件，具体参考前面的 `安装` 部分的示例。

**3.4.2 fetch 事件**

**当浏览器发起请求时，会触发 `fetch` 事件**

service worker 安装成功后进入激活安装状态后就运行于浏览器后台，可以通过 `fetch` 事件拦截到当前作用域内的 `http/https` 请求，并给出自己的响应。结合 `fetch api`，可以简单地处理请求响应，**对网络请求的控制**

这个功能十分强大。

参考下面的示例，这里实现了一个 `缓存优先、降级处理` 的策略逻辑：

监控所有的 `http` 请求，当请求资源已经在缓存里，直接返回缓存里的内容，否则，使用 `fetch api` 继续请求，如果是图片或css、js资源，请求成功后，将它们加入到缓存里；如果是离线状态或请求出错，则降级返回预先缓存的离线内容。

```js
// 联网状态下执行
function onlineRequest(fetchRequest) {
    // 使用 fecth API 获取资源，以实现对资源请求控制
    return fetch(fetchRequest).then(response => {
        // 在资源请求成功后，将 image、js、css 资源加入缓存列表
        if (
            !response
            || response.status !== 200
            || !response.headers.get('Content-type').match(/image|javascript|test\/css/i)
        ) {
            return response;
        }
 
        const responseToCache = response.clone();
        caches.open(CACHE_NAME)
            .then(function (cache) {
                cache.put(event.request, responseToCache);
            });
 
        return response;
    }).catch(() => {
        // 获取失败，离线资源降级替换
        offlineRequest(fetchRequest);
    });
}
// 离线状态下执行，降级替换
function offlineRequest(request) {
    // 使用离线图片
    if (request.url.match(/\.(png|gif|jpg)/i)) {
        return caches.match('/images/offline.png');
    }
 
    // 使用离线页面
    if (request.url.match(/\.html$/)) {
        return caches.match('/test/offline.html');
    }
}
 
self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request)
        .then(hit => {
            // 返回缓存中命中的文件
            if (hit) {
                return hit;
            }
 
            const fetchRequest = event.request.clone();
 
            if (navigator.online) {
                // 如果为联网状态
                return onlineRequest(fetchRequest);
            } else {
                // 如果为离线状态
                return offlineRequest(fetchRequest);
            }
        })
    );
});
```

另外需要注意的是，`fetch` 请求默认是不附带 `cookie` 信息的，但请求静态资源上这没有问题，而且节省了网络请求大小。对于动态页面，可能会因为缺少 `cookie` 而存在问题。

此时，可以给 `fetch` 设置第二个参数，示例：

```js
fetch(fetchRequest, { credentials: 'includes' });
```

**3.4.3 activate 事件**

当安装完成后，进入激活状态，会触发 `activate` 事件，通过监听 `activate` 事件可以做一些 **预处理**，如，对旧版本的更新、对于无用缓存的清理等。

在下面的示例里，实现对旧版本的缓存资源进行清理：

```js
this.addEventListener('activate', event => {
    const cacheWhitelist = ['lzwme_cache_v1.6.0'];
 
    event.waitUntil(
        // 遍历当前的缓存，删除除 `lzwme_cache_v1.6.0` 之外的所有缓存
        caches.keys().then(keyList => {
            return Promise.all(keyList.map(key => {
                if (cacheWhitelist.indexOf(key) === -1) {
                    return caches.delete(key);
                }
            }));
        })
    );
});
```

传递给 `waitUtil` 的 Promise 会阻塞其他的事件，直到他完成。这可以确保清理操作会在第一次 `fetch` 事件之前完成。

在激活状态时也可以执行 `clients.claim` 方法，更新所有客户端上的 service worker。具体参见前文的 `Service Worker 更新` 部分。

**3.4.4 push 事件**

push 事件是为推送准备的，不过需要先了解以下 `Notification API` 和 `PUSH API`

通过 PUSH API，当订阅了推送服务后，可使用推送方式唤醒 service worker 来响应来自系统消息传递服务的消息，即使用户已经关闭了页面。

推送的实现有两步：

不同浏览器需要用不同的消息推送器。以 chrome 上使用 Google Cloud Messaging<GCM> 作为推送服务为例，第一步是注册 application ServerKey（通过GCM注册获取），并在页面上进行订阅或发布订阅。每个会话会有一个独立的端点（endpoint），订阅对象的属性（PushSubscription.endpoint）即为端点值。将端点发送给服务器后，服务器用这个值来发送消息给会话的激活的 service worker（通过GCM与浏览器客户端沟通）

在页面上，使用 `PushManager.subscribe()` 来订阅推送服务。示例：

```js
// 向用户申请通知权限，用户可以选择允许或禁止
// Notification.requestPermission 只有在页面上才可执行，Service Worker 内部不可申请权限
Notification.requestPermission().then(grant => {
    console.log(grant); // 如果获得权限，会得到 granted
    if (Notification.permission === 'denied') {
        // 用户拒绝了通知权限
        console.log('Permission for Notifications was denied');
    }
});
 
let reg;
const applicationServerKey = 'xxx'; // 应用服务器的公钥（base64 网址安全编码）
navigator.serviceWorker.ready.then(_reg => {
    reg = _reg;
    // 获取当前订阅的推送
    return reg.pushManager.getSubscription();
})
.then(subscription => {
    // 获取的结果没有任何订阅，发起一个订阅
    if (!subscription) {
        return reg.pushManager.subscribe({
            userVisibleOnly: true,
            applicationServerKey: applicationServerKey
        });
    } else {
        // 每一个会话会有一个独立的端点(endpoint)，用于推送时后端识别
        return console.log("已订阅 endpoint:", subscription.endpoint);
    }
})
.then(subscription => {
    if (!subscription) {
        return;
    }
 
    // 订阅成功
    console.log('订阅成功！', subscription.endpoint);
 
    // 做更多的事情，如将订阅信息发送给后端，用于后端推送识别
    // const key = subscription.getKey('p256dh');
    // updateStatus(subscription.endpoint, key, 'subscribe');
})
.catch(function (e) {
    // 订阅失败
    console.log('Unable to subscribe to push.', e);
});
```

第二步，在 service worker 中通过监听 `push` 事件对推送的消息做处理，示例：

```js
self.addEventListener('push', function(event) {
    // 读取 event.data 获取传递过来的数据，根据该数据做进一步的逻辑处理
    const obj = event.data.json();
 
    // 逻辑处理示例
    if(Notification.permission === 'granted' && obj.action === 'subscribe') {
        self.registration.showNotification("Hi：", {
            body: '订阅成功 ~',
            icon: '//lzw.me/images/avatar/lzwme-80x80.png',
            tag: 'push'
        });
    }
});
```

这里有一个关于推送(PUSH API)的完整示例可作参考：`https://github.com/chrisdavidmills/push-api-demo`

>有个比较大的问题是，**每个浏览器有它们自己的服务器用来处理消息推送**。在国内 Google 大部分服务不可用的情况下，在 Chrome 中这个推送功能就没用了。另外还涉及后端的处理逻辑，这里不作细说，有兴趣可进一步参阅如下参考。


**关于推送和通知的更多参考：**

- GCM https://developers.google.com/cloud-messaging/gcm
- Notification API https://developer.mozilla.org/zh-CN/docs/Web/API/notification
- PUSH API https://developer.mozilla.org/zh-CN/docs/Web/API/Push_API
- Using_the_Push_API https://developer.mozilla.org/zh-CN/docs/Web/API/Push_API/Using_the_Push_API
- PushSubscription https://developer.mozilla.org/zh-CN/docs/Web/API/PushSubscription
- 向网络应用添加推送通知 https://developers.google.com/web/fundamentals/getting-started/codelabs/push-notifications/
- https://developers.google.com/web/fundamentals/engage-and-retain/push-notifications/
- 【Service Worker】消息推送功能“全军覆没” https://75team.com/post/push-knock-the-door.html
- https://developer.apple.com/notifications/safari-push-notifications/

**3.4.5 sync 事件**

sync 事件由 `ackground sync`(后台同步)发出。background sync 是 Goole 配合 SW 退出的API，用于为SW提供一个可以实现注册和监听通过处理的方法。但不在w3c标准中。在 Chrome 中这也只是一个实验性功能，需要访问 `chrome://flags/#enable-experimental-web-platform-features`，开启该功能，然后重启生效。

后台同步功能允许你一次性或按间隔请求后台数据同步，即使用户没有打开网站，仅唤醒了 `ServiceWorker`，也会如此。

当从页面请求执行此操作的权限，用户将收到提示。后台同步适用于：非常急的更新，特别是哪些需要定期更新的，每次更新一个都发送推送会显得太频繁。如在某个时间推送一篇特色文章或一条消息通知，这在 native 应用中非常常见。

参考下面的示例。

A. 在页面注册 sync：
```js
navigator.serviceWorker.ready.then(function(swRegistration) {
    return swRegistration.sync.register('myFirstSync');
});
```

B. 在 SW 中监听 sync 事件：
```js
self.addEventListener('sync', function(event) {
    if (event.tag === 'myFirstSync') {
        event.waitUntil(doSomething());
    }
});
```

相关参考：

- background-sync https://developers.google.com/web/updates/2015/12/background-sync
- https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/#on-background-sync
- https://developer.mozilla.org/en-US/docs/Web/API/SyncManager


**3.4.6 message 事件(postMessage)**

ServiceWorker 运行于独立的沙盒中，无法直接访问当前页面的 DOM 等信息，但是通过 `postMessage` API，可以实现他们之间的消息传递。

跨文档的 postMessage 消息传递，需要获取接收方的文档句柄。但只有 service worker 注册成功后该句柄才会存在。

```js
function sendMsg(msg) {
    const controller = navigator.serviceWorker.controller;
 
    if (!controller) {
        return;
    }
 
    controller.postMessage(msg, []);
}
 
// 在 serviceWorker 注册成功后，页面上即可通过 navigator.serviceWorker.controller 发送消息给它
navigator.serviceWorker
    .register('/test/sw.js', {scope: '/test/'})
    .then(registration => console.log('ServiceWorker 注册成功！作用域为: ', registration.scope))
    .then(() => sendMsg('hello sw!'))
    .catch(err => console.log('ServiceWorker 注册失败: ', err));
```

在 ServiceWorker 内部，可以通过监听 `message` 事件即可获得消息

```js
self.addEventListener('message', function(ev) {
    console.log(ev.data);
});
```

B. ServiceWorker 发消息给页面

ServiceWorker 内部需要获取页面句柄，这个句柄要从 self.clients 上得到。

```js
self.clients.matchAll().then(clientList => {
    clientList.forEach(client => {
        client.postMessage('Hi, I am send from Service worker！');
    })
});
```

关于 postMessage 的更多介绍，可参考：https://lzw.me/a/html5-postmessage-post-cross-domain.html

**3.4.7 online/offline 事件**

当网络状态发生变化时，会触发 `online` 或 `offline` 事件。结合这两个事件，可以与 Service Worker 结合实现更好的离线使用体验，例如当网络发生改变时，替换/隐藏需要在线状态才能使用的链接导航等。

下面是一个监听 offline 的示例：

```js
self.addEventListener('offline', function() {
    Notification.requestPermission().then(grant => {
        if (grant !== 'granted') {
            return;
        }
 
        const notification = new Notification("Hi，网络不给力哟", {
            body: '您的网络貌似离线了，不过在志文工作室里访问过的页面还可以继续打开~',
            icon: '//lzw.me/images/avatar/lzwme-80x80.png'
        });
 
        notification.onclick = function() {
            notification.close();
        };
    });
});
```

查看当前的网络状态：`navigator.onLine`。`true` 表示联网。


**3.4.8 error 和 unhandledrejection 事件**

当 JS 执行发生错误，会触发 `error` 事件；当 `Promise` 类型的回调发生 `reject` 却没有 `catch` 处理，会触发 `unhandledrejection` 事件。

对于这类事件，前端应当作埋点上报，以便于统计监控和及时发现处理。

一般情况下上报的信息应从 `error` 中读取，主要包括错误堆栈相关信息以便定位。参考如下示例：

```js
self.onerror = function(errorMessage, scriptURI, lineNumber, columnNumber, error) {
    if (error) {
        reportError(error);
    } else {
        reportError({
            message: errorMessage,
            script: scriptURI,
            line: lineNumber，
            column: columnNumber
        });
    }
}
```

监听 `unhandledrejection` 事件：

```js
self.addEventListener('unhandledrejection', function(event) {
    reportError({
        message: event.reason
    })
});
```


**3.4.9 beforeinstallprompt 事件**

当发生 Add to Homescreen (A2HS, 添加到主屏幕)行为的请求时，会触发该事件。它发生于页面中，与 Service Worker 并没有直接关系。

如果你的站点符合 A2HS 的条件(具体参见后文介绍)，浏览器(chrome) 会根据默认的行为算法，来决定何时主动的向用户展示添加到首屏提示。另外，用户也可以通过 chrome 菜单中的 `添加到主屏幕` 选项主动添加。

可以在页面中通过监听 `beforeinstallprompt` 事件，决定是否屏蔽/延迟该行为，或者统计用户选择了允许还是拒绝。示例：

```js
let deferredPrompt; // 用于缓存 beforeinstallprompt 的事件对象
 
window.addEventListener('beforeinstallprompt', function(event) {
    // 阻止该行为，只需要返回 false
    // event.preventDefault();
    // deferredPrompt = event;
    // return false;
 
    // 统计用户的选择
    event.userChoice.then(function(choiceResult) {
        console.log(choiceResult.outcome); // 为 dismissed 或 accepted
        if(choiceResult.outcome === 'dismissed') {
            console.log('User cancelled home screen install');
        } else {
            console.log('User added to home screen');
        }
    });
});
 
// 在 beforeinstallprompt 事件中屏蔽了浏览器的默认行为，在页面中通过按钮让用户主动选择
document.getElementById('addToHomeScreen').addEventListener('click', function() {
    if(deferredPrompt) {
        deferredPrompt.prompt();
    }
})
```

**4、 Service Worker 调试**

使用 Chrome 浏览器，可以通过 `Application` -> `Service Workers` 面板查看和调试。如下图所示：

在顶部有四个选项：

- Offline： 选中则当前站点即断开网络状态
- Update on reload： 选中则当刷新页面时强制更新
- Bypass for network： 选中则资源加载和更新绕过 Service Worker，都要从网络获取
- Update：(更新)按钮执行指定的Service workers 一次性更新。

接着会列举当前站点注册的 Service Worker 的基本信息。每个注册的站点下面，会有这些选项：

- Push： (推送)按钮会模拟推送一个没有 payload 的通知(会触发 push 事件)，也称为 tickle。
- Synch： (同步)按钮模拟一个后台同步事件(会触发 sync 事件)。
- Unregister：(注销)按钮注销指定的Service Worker。查看Clear Storage以注销Service Workers，并通过单击按钮清除所有缓存和存储。
- inspect： 点击 inspect，则会弹出一个开发者窗口。在弹出的调试窗口内，在 source 面板可以调试注册的脚本代码，在 console 面板可以看到 Service Worker 打印的日志，并且可以执行基于当前 Service Worker 上下文的代码。
- Source： 当前正在运行的 Service Worker 安装的时间。
- Clients： Service Worker 作用域的来源
- Status： Service Worker 的状态。旁边有 start、stop 按钮，可以手动来启动、停止 Service Worker。

通过 chrome://serviceworker-internals 也可以打开 serviceWorker 的配置面板，查看所有注册的 serviceworker 情况。注意一点，如无必要，不要选中顶部的 Open DevTools window and pause JavaScript execution on Service Worker startup for debugging 复选框，否则每当刷新页面调试时都会弹出一个开发者窗口来。

参考[更多](https://developers.google.com/web/tools/chrome-devtools/progressive-web-apps)

**5、 Service Worker 安全问题**

可以利用 Appcache 和 ServiceWorker 进行持久型 session hijacking 和 XSS 中提到了服务器权限被拿下，或通过 Wifi 中间人劫持后，对访问者可以制造持久性攻击的问题。

其中 Appcache 主要利用的是 cache-manifest 文件也被缓存时，从站点方面无法进行更新，那么网站方面即使知道了这种问题，也是无能为力。这种 bug 不知道浏览器厂商方面后续是否有针对性处理。如果你还不了解 appcache，可以参考这篇文章：https://lzw.me/a/html5-applicationcache-manifest.html

对于 ServiceWorker 的利用，则是通过构造 ServiceWorker 的脚本并向用户注册。
想象一下，如果站点方面还没有 ServiceWorker 的意识，却被中间人提前利用了，用户被攻击了一次，却导致后续的访问都会被该 ServiceWorker 劫持。

**6、 APP Manifest 与添加到主屏幕**

通过添加到主屏幕功能可以在用户桌面(主屏幕)上创建一个站点的快捷图标，实现和 Native APP 近似的使用体验。通过该图标打开的页面和浏览器中直接访问的体验有些不同，它在体验上更像 Native 应用：

- 和 APP 一样在用户桌面(主屏幕)上存在图标
- 有 splash screen 启动屏
- 可以控制控制屏幕方向
- 可以设置全屏显示，没有浏览器的搜索框、导航栏

要实现这一点的前提条件是：站点注册成功了 Service Worker；定义了一个 `manifest` 文件，并在页面中申明。

好像 Vue 文档就是利用了这个进行了缓存，断网也可访问

**6.1 定义 manifest.json 配置添加到主屏幕功能**

- 创建 manifest.json 文件，并将它放到你的站点目录中。
- 在所有页面引入该文件。
- 可以在 Service Worker 中监听 `beforeinstallprompt` 事件做一些应用内的行为处理。具体参见前文事件处理部分。

在页面 head 区域添加如下内容：
```html
<link rel="manifest" href="/manifest.json">
```

anifest.json 内容示例(相关注释为了解释说明，正式使用应当删除注释)：

```
{
  "scope": "/",
  "name": "志文工作室",
  "short_name": "志文博客",
  "start_url": "/?from=homescreen",
  "display": "standalone", // chrome 58 beta 添加了取值 fullscreen 的支持，更像一个全屏应用
  "description": "每天知道多一点.",
  "dir": "rtl",
  "lang": "cn",
  "orientation": "portrait",
  "theme_color": "#3f51b5",
  "background_color": "#fff",
  "icons": [{
    "src": "images/logo/48x48.webp",
    "sizes": "48x48",
    "type": "image/webp"
  }, {
    "src": "images/logo/72x72.svg",
    "sizes": "72x72",
    "type": "image/svg"
  }, {
    "src": "images/logo/96x96.png",
    "sizes": "96x96",
    "type": "image/png"
  }, {
    "src": "images/logo/144x144.png",
    "sizes": "144x144",
    "type": "image/png"
  }, {
    "src": "images/logo/hd.png",
    "sizes": "168x168 192x192 256x256",
    "type": "image/png"
  }],
  "prefer_related_applications": false,
  "related_applications": [{
    "platform": "web"
  }, {
    "platform": "play",
    "url": "https://play.google.com/store/apps/details?id=com.lzwme.demo"
  }, {
    "platform": "itunes",
    "url": "https://itunes.apple.com/app/com-lzw-me-demo/id500000000"
  }]
}
```

以上为一个 demo 示例，各参数和取值具体含义可参见 MDN(见下面的参考链接)。可以打开志文工作室博客的配置对比参考：https://lzw.me/manifest.json。

最佳实践参考：

- 所有页面都应添加引入代码
- 不要偷懒简化 icon 的配置。应为不同分辨率的屏幕提供不同的 icon。Chrome 会尝试使用最接近 48dp 的图标，比如在 2x 屏上使用 96px 的，在 3x屏上使用 144px 的。
- short_name 会用于主屏幕图标名称，name 和 background_color 会用于 splash 启动屏幕上。
- 为 start_url 配置特殊的参数，可以用于统计从主屏幕打开的用户访问。

**6.2 设置 IOS Safari 上的添加至主屏幕元素**

IOS 上 Safari 有自己的类似标准。方法很简单，在页面的 head 区域添加如下配置即可：

```html
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-title" content="志文工作室">
<link rel="apple-touch-icon" href="images/logo/152x152.png">
```

**6.3 设置 window10 贴片图标**

在页面的 head 区域添加如下内容：

```html
<meta name="msapplication-TileImage" content="images/logo/144x144.png">
<meta name="msapplication-TileColor" content="#2F3BA2">
```
更多参考：

https://developers.google.com/web/fundamentals/engage-and-retain/app-install-banners
https://developers.google.com/web/fundamentals/engage-and-retain/web-app-manifest/
https://developer.mozilla.org/zh-CN/docs/Web/Manifest

**7 博客网站 PWA 快速改造参考**

对于个人博客类站点，基于 PWA 的使用体验基本是相似的。可参考如下步骤为你的博客快速地引入 PWA：

- 参考 https://lzw.me/sw.js，下载它并放到你的站点根目录
- 设计简单的 offline.html 用于离线降级显示。简单的示例参考： https://lzw.me/offline.html
- 参考 https://lzw.me/manifest.json，下载并修改适合你的博客，放到站点根目录
- 在页面的 js 中加入注册 sw.js 的代码。参考前文 Serveice Worker 注册 部分示例
- 在站点页面模板的 head 区域，引入 manilfest.json 文件
- 在 PC 和手机上分别使用 Chrome 浏览器进行体验

在页面中注册 ServiceWorker 代码参考：

```js
// 注册 ServiceWorker
function regSW() {
    if ('serviceWorker' in navigator) {
        // 注册
        navigator.serviceWorker
            .register('/sw.js', {scope: '/'})
            .then( function(registration) {
                console.log('ServiceWorker 注册成功！作用域为: ', registration.scope);
            })
            .catch(function(err) {
                console.log('ServiceWorker 注册失败: ', err);
            });
 
        // SW 消息处理
        navigator.serviceWorker.ready.then(function(reg) {
            if (!window.Notification || !window.MessageChannel) {
                return;
            }
 
            // 建立一个消息管道，用于当前页面与 SW 之间的消息传递，也便于 SW 知道该消息的来源
            var channel = new window.MessageChannel();
 
            channel.port1.onmessage = function(e) {
                console.log('get Message: ', e.data);
                if (!e.data) {
                    return;
                }
 
                // 要求申请通知权限
                if (e.data.type === 'applyNotify') {
                    window.Notification.requestPermission().then(function(grant) {
                        if (grant !== 'granted') {
                            console.log('申请通知权限被拒绝了！')
                            return;
                        }
 
                        reg.active.postMessage({type: 'notify', info: e.data.info}, [channel.port2]);
                    });
                }
            }
 
            reg.active.postMessage('hello', [channel.port2]);
        });
 
        // 掉线通知示例
        $(window).on('offline', function() {
            Notification.requestPermission().then(function (grant) {
                if (grant !== 'granted') {
                    return;
                }
 
                var notification = new Notification("Hi，网络不给力哟", {
                    body: '您的网络貌似离线了，不过在志文工作室里访问过的页面还可以继续打开~',
                    icon: '//lzw.me/images/avatar/lzwme-80x80.png'
                });
 
                notification.onclick = function() {
                    notification.close();
                };
            });
        })
    }
}
```

**8 小结**

通读了近百篇相关文章，形成了上述内容，并在本博客上进行了实践尝试。希望以上内容可以给你提供有帮助的参考。

这里有一个涉及到上述提到的大部分内容的 sw.js 实现，如有兴趣可进一步对比查阅：`https://lzw.me/sw.js`

**9、 其他相关参考**

- https://w3c.github.io/ServiceWorker/
- https://www.w3.org/TR/service-workers/
- https://github.com/w3c/ServiceWorker
- Using Service Workers https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API/Using_Service_Workers
- Service_Worker_API https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API
- Progressive Web Apps https://developers.google.com/web/progressive-web-apps/
- https://developers.google.com/web/fundamentals/getting-started/primers/service-workers
- https://developers.google.com/web/fundamentals/getting-started/codelabs/your-first-pwapp/?hl=zh-cn
- SW 离线指南 https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/
- Service Workers’ API 信息图 https://github.com/delapuente/service-workers-101
- Service Workers 与离线缓存 https://segmentfault.com/a/1190000008491458
- 【Service Worker】生命周期那些事儿 https://segmentfault.com/a/1190000007487049
- Service Worker 生命周期 https://segmentfault.com/a/1190000006061528
- https://www.pangjian.me/2017/02/08/service-worker-offlinemode/
- https://www.slideshare.net/patrickmeenan/service-workers-for-performance
- PWA 在饿了么的实践经验 https://zhuanlan.zhihu.com/p/25800461

