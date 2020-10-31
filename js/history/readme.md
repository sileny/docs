# history

[文档翻译-部分](https://html.spec.whatwg.org/multipage/history.html#the-history-interface)

每浏览一个页面，都会有不同的历史 `session` 会话记录。

每条 `session` 记录都包含以下结构：

- `url`
- `document`： Document | null
- `state` 序列化状态，默认值为 `null`
- `title` 字符串或 null
- `scrollRestoration` [滚动恢复模式](#scrollrestoration)，默认值为 `auto`，另外一个值为 `manual`


## interface

window.History接口对象
```
enum ScrollRestoration { "auto", "manual" };

[Exposed=Window]
interface History {
  readonly attribute unsigned long length;
  attribute ScrollRestoration scrollRestoration;
  readonly attribute any state;
  undefined go(optional long delta = 0);
  undefined back();
  undefined forward();
  undefined pushState(any data, DOMString title, optional USVString? url = null);
  undefined replaceState(any data, DOMString title, optional USVString? url = null);
};
```

### length

返回会话历史记录中的条目数

>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`


### scrollRestoration

返回会话历史记录中当前条目的滚动恢复模式。

可以设置，以更改会话历史记录中当前条目的滚动恢复模式

```js
if ('scrollRestoration' in history)
  history.scrollRestoration = 'manual';
```

- `manual` 滚动条会回到顶部
- `auto` 会记录上一次滚动条所在的位置，刷新之后，停留在上一次滚动条所在的位置


```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <script>
    if ('scrollRestoration' in history)
      history.scrollRestoration = 'auto';
  </script>
</head>
<body>
<div style="height: 1800px;"></div>
</body>
</html>

```
修改 `history.scrollRestoration` 的值之后刷新浏览器试试看


>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`

### state

返回当前的序列化状态，反序列化为一个对象

>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`

### go

在浏览器会话历史记录中后退或前进指定的步骤数。

零增量将重新加载当前页面。

如果增量超出范围，则不执行任何操作

>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`

### back

浏览器会话历史记录里后退一步

如果没有上一页，则什么也不做

>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`

### forward

在历史纪录里向前进一页

如果没有下一页，则不执行任何操作

>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`


### pushState

将具有给定标题和给定URL（如果提供且不为null）的给定数据推送到会话历史记录中。

语法
```js
pushState(data: any, title: string, url?: string | null): void;
```

>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`


### replaceState

更新会话历史记录中的当前条目，以具有给定的数据，标题和URL（如果提供，但不为null）。


语法
```js
replaceState(data: any, title: string, url?: string | null): void;
```

>如果相关连的 Document 不完全激活，会抛出一个 `SecurityError DOMException`


### queue

每个 `顶级的浏览器环境`(top-level browsing context) 有一个 `会话历史遍历队列`(session history traversal queue)，初始值为空，可以往这个队列添加历史记录。

每个 `顶级的浏览器环境`(top-level browsing context) 创建时，必须开始运行下面的算法，被称作 `会话历史事件循环`，是 `并行` 的任务。该事件循环机制如下：

1. 会话历史记录遍历队列不为空
2. 提取第一个任务，然后执行它。以此类推
3. 返回第一步


下面是原文
```
Each top-level browsing context has a session history traversal queue, initially empty, to which tasks can be added.

Each top-level browsing context, when created, must begin running the following algorithm, known as the session history event loop for that top-level browsing context, in parallel:

Wait until this top-level browsing context's session history traversal queue is not empty.

Pull the first task from this top-level browsing context's session history traversal queue, and execute it.

Return to the first step of this algorithm.

The session history event loop helps coordinate cross-browsing-context transitions of the joint session history: since each browsing context might, at any particular time, have a different event loop (this can happen if the user navigates from example.com to shop.example), transitions would otherwise have to involve cross-event-loop synchronization.
```


