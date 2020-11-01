# optimization

- [server端优化](#server)
- [动画优化](#animation)

# server

- [node](#node)

## node

- [compression压缩响应](#compression)
- [compression对指定的消息进行压缩](#compression-filter)
- [compression压缩水平](#compression-level)

### compression

压缩服务器发送出去的数据。可以用 `zlib`，也可以使用 `compression` 中间件模块

`compression` 会检查消息头的 `Accept-Encoding`，判断库夫段能接收哪种编码。如果消息头没有这个域，则不会对响应的消息做任何处理。如果有 `gzip` 或 `deflate`，则进行响应进行压缩。
```js
const connect = require('connect');
const compression = require('compression');
connect()
  .use(compression({ threshold: 0 }))
  .use((req, res, next) => {
    res.setHeader('Content-Type', 'text/plain');
    res.end('This response is compressed');
  })
  .listen(3000);
```

发送一个 `Accept-Encoding` 为 `gzip` 的请求
```
curl http://localhost:3000 -i -H 'Accept-Encoding: gzip'
```

`-i` 是让 `curl` 显示消息头，这样，可以看到 `ContentEncoding` 是 `gzip`。消息主体被压缩过，所以是乱码字符

去掉 `-i` ，将响应消息通过管道转给 `gunzip`，可以看到解压后的内容

```
curl http://localhost:3000 -H 'Accept-Encoding: gzip' | gunzip
```

[compression-demo案例](https://github.com/sileny/node-demo/tree/main/compression-demo)


但不是所有的消息都需要压缩

### compression-filter

`compression` 在默认的 `filter` 函数里包含了 `MIME` 类型的 `text/*`、`*/json`、`*/javascript`，因此，只会压缩这些响应数据

如果要改变这种默认的行为，需要传入定制的 `filter`

```js
const connect = require('connect');
const compression = require('compression');

function shouldCompress(req, res) {
  if (req.headers['x-no-compression']) {
    // don't compress responses with this request header
    return false
  }

  // fallback to standard filter function
  return compression.filter(req, res)
}

connect()
  .use(compression({ threshold: 0, filter: shouldCompress }))
  .use((req, res, next) => {
    res.setHeader('Content-Type', 'text/plain');
    res.end('This response is compressed');
  })
  .listen(3000);

```

这样就可只对普通文本进行压缩

### compression-level

```js
connect()
  .use(compression({ level: 3, memLevel: 8 }));
```

- `level` 为 `3`，表示压缩率低，但压缩速度快
- `memLevel` 为 `8`，表示使用更多的内存进行压缩

# animation

动画优化

尽量不要使用 `width`、`height`、`margin`、`padding` 来做动画。

`transform` 实现的动画与合成器线程相关，不需要等待主线程的样式计算或通过js动态计算，同时，它可以开启 `gpu` 加速，不会导致布局调整


