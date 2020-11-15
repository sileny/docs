# optimization

- [server端优化](#server)
- [动画优化](#animation)
- [雅虎军规](https://github.com/sileny/docs/blob/master/yahoo/rules.md)
- [大量操作dom](#dom)

# server


虽然代码压缩成 `gz` 格式的了，但也需要在服务端做配置，下面介绍几种配置方式

- [node](#node)
- [node express](#express)
- [spring boot](#spring)
- [nginx](#nginx)


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



## express

1、安装 `compression`
```
npm install compression
```

2、配置
```js
const express = require('express');
const compression = require('compression');

const app = express();
//尽量在其他中间件前使用compression
app.use(compression());
```

如果，只是对某些请求使用此功能，可以这样
```js
app.use(compression({filter: shouldCompress}))
function shouldCompress (req, res) {
  if (req.headers['x-no-compression']) {
    // 这里就过滤掉了请求头包含'x-no-compression'
    return false
  }
  return compression.filter(req, res)
}
```



## nginx

1、打开 `nginx` 配置文件 `nginx.conf`

2、找对应内容，并配置如下
```
gzip on;
gzip_min_length 1k;
gzip_buffers 4 16k;
gzip_http_version 1.0;
gzip_comp_level 5;
gzip_types text/plain text/css application/x-javascript application/xml application/javascript image/jpeg image/gif image/png;
gzip_proxied off;
gzip_vary off;
gzip_disable "MSIE [1-6]\.";
gzip_static off;
```

3、说明相关配置

**gzip on;**

　　开启Gzip

**gzip_min_length 1k;**

　　大于1k才会压缩

**gzip_buffers 4 16k;**

　　默认值: `gzip_buffers 4 4k/8k`
  
　　设置系统获取几个单位的缓存用于存储 `gzip` 的压缩结果数据流。 例如 `4 4k` 代表以 `4k` 为单位，按照原始数据大小以 `4k` 为单位的 `4` 倍申请内存。`4 8k` 代表以 `8k` 为单位，按照原始数据大小以 `8k` 为单位的 `4` 倍申请内存。如果没有设置，默认值是申请跟原始数据相同大小的内存空间去存储 `gzip` 压缩结果。

**gzip_http_version 1.0;**

　　默认值: `gzip_http_version 1.1`

　　识别 `http` 的协议版本。由于早期的一些浏览器或者 `http` 客户端，可能不支持 `gzip` 自解压，用户就会看到乱码，所以做一些判断还是有必要的。 注：21世纪都来了，现在除了类似于百度的蜘蛛之类的东西不支持自解压，99.99%的浏览器基本上都支持 `gzip` 解压了，所以可以不用设这个值,保持系统默认即可。

**gzip_comp_level 2;**

　　默认值: `gzip_comp_level 1`

　　`gzip`压缩比，`1` 压缩比最小处理速度最快，`9` 压缩比最大但处理最慢（传输快但比较消耗 `cpu`）

**gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;**

　　默认值: `gzip_types text/html`

　　匹配 `MIME` 类型进行压缩，（无论是否指定）`text/html`类型总是会被压缩的。

**gzip_proxied off;**

- `Nginx` 作为反向代理的时候启用，开启或者关闭后端服务器返回的结果，匹配的前提是后端服务器必须要返回包含 `Via` 的 `header` 头。
- `off` - 关闭所有的代理结果数据的压缩
- `expired` - 启用压缩，如果header头中包含 `Expires` 头信息
- `no-cache` - 启用压缩，如果header头中包含 `Cache-Control:no-cache` 头信息
- `no-store` - 启用压缩，如果header头中包含 `Cache-Control:no-store` 头信息
- `private` - 启用压缩，如果header头中包含 `Cache-Control:private` 头信息
- `no_last_modified` - 启用压缩,如果header头中不包含 `Last-Modified` 头信息
- `no_etag` - 启用压缩 ,如果header头中不包含 `ETag` 头信息
- `auth` - 启用压缩 , 如果header头中包含 `Authorization` 头信息
- `any` - 无条件启用压缩

**gzip_vary off;**

　　默认值：`gzip_vary off;`

　　主要提供给代理服务器使用，根据 `Vary` 头做不同的处理。例如，对于支持 `gzip` 的请求反向代理缓存服务器将返回 `gzip` 内容，不支持 `gzip` 的客户端返回原始内容。   

**gzip_disable "MSIE [1-6]\.";**

　　ie6及以下不支持压缩，所以关闭压缩

**gzip_static off;**

　　启动预压缩功能，对所有类型的文件都有效
　　编译 `nginx` 的时候需要把 `--with-http_gzip_static_module` 参数加上。
　　压缩都是动态的，`gzip_static on;` 直接读取已经压缩好的文件不是动态压缩,对于不支持 `gzip` 的请求则读取原文件

_注意：_
- `gzip_static` 配置优先级高于 `gzip`
- 开启 `nginx_static` 后，对于任何文件都会先查找是否有对应的 `gz` 文件
- `gzip_types` 设置对 `gzip_static` 无效

4、保存文件，重启 `nginx`

重启命令：`nginx` 安装目录 `./sbin/nginx -s reload`

5、查看是否开启压缩功能

执行命令：`curl curl -I -H "Accept-Encoding: gzip, deflate" http://192.168.20.78:8888/admin`

输出内容如下，
```
curl: (6) Could not resolve host: curl
HTTP/1.1 200 OK
Server: nginx/1.15.5
Date: Wed, 22 Jul 2020 11:08:34 GMT
Content-Type: text/html
Last-Modified: Wed, 22 Jul 2020 10:53:11 GMT
Connection: keep-alive
ETag: W/"5f181a97-2ced"
Content-Encoding: gzip
```


## spring

`spring` 默认不开启 `gzip` 压缩，需要手动开启，配置如下，
```yml
server:
  compression:
    min-response-size: 1024 # 但是需要注意并不是说所有的接口都会使用 gzip 压缩，默认情况下，仅会压缩 2048 字节以上的内容，这里改为压缩 1024 字节以上内容
    enabled: true
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json,application/xml
```






# animation

动画优化

尽量不要使用 `width`、`height`、`margin`、`padding` 来做动画。

`transform` 实现的动画与合成器线程相关，不需要等待主线程的样式计算或通过js动态计算，同时，它可以开启 `gpu` 加速，不会导致布局调整

# dom

尽量少地操作dom元素，重新渲染代码昂贵，尤其在首屏

如果确实需要大量的 `dom` 操作，可以使用 [fastdom](https://github.com/wilsonpage/fastdom)




