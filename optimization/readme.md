# optimization

- [server端优化](#server)
- [dns](#dns)
- [动画优化](#animation)
- [雅虎军规](https://github.com/sileny/docs/blob/master/yahoo/rules.md)
- [大量操作dom](#dom)
- [缓存](#cache)

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



# dns

DNS 实现域名到IP的映射。通过域名访问站点，每次请求都要做DNS解析。目前每次DNS解析，通常在200ms以下。一般采用DNS Prefetch 一种DNS 预解析技术，当你浏览网页时，浏览器会在加载网页时对网页中的域名进行解析缓存，这样在你单击当前网页中的连接时就无需进行DNS的解析，减少用户等待时间，提高用户体验

```
<link rel="dns-prefetch" href="www.baidu.com" />
```




# animation

动画优化

尽量不要使用 `width`、`height`、`margin`、`padding` 来做动画。

`transform` 实现的动画与合成器线程相关，不需要等待主线程的样式计算或通过js动态计算，同时，它可以开启 `gpu` 加速，不会导致布局调整

# dom

尽量少地操作dom元素，重新渲染代码昂贵，尤其在首屏

如果确实需要大量的 `dom` 操作，可以使用 [fastdom](https://github.com/wilsonpage/fastdom)




# cache


- [nginx静态资源缓存](#静态资源缓存)
- [nginx缓存指定目录下的资源](#nginx缓存指定目录下的资源)
- [nginx location](#location)
- [http2](#http2)
- [proxy_cache_path](#proxy_cache_path)
- [缓存location配置](#location)
- [localForage](./localForage.md)


## 静态资源缓存

nginx 提供了 `expires`、`ETag`、`if-modifyied-since` 指令来实现浏览器缓存控制

- `expires`

```
location /img {
    expires 90d; # 缓存90天
}
```

对于静态资源，nginx 会自动添加 `ETag` 响应头信息。

- `if-modified-since`

此指令用来指定 `nginx` 如何拿到服务端的 `Last-Modified` 和浏览器端的 `if-modified-since` 时间进行比较。

- 默认地，`if-modified-since` 执行的是 `exact` 匹配；
- 也可以使用 `if-modified-since _before`，表示只要文件最后修改的时间早于或者等于浏览器的 `if-modified-since` 时间，返回 304

```
location = /cache {
　　proxy_pass http://backend_tomcat/cache$is_args$args;
　　expires 5s;
}
```

1．浏览器发起请求，首先到Nginx，Nginx根据URL在Nginx本地查找是否有代理层本地缓存。

2．Nginx没有找到本地缓存，则访问后端获取最新的文档，并放入到Nginx本地缓存中，返回200状态码和最新的文档给浏览器。

3．Nginx找到本地缓存，首先验证文档是否过期（`Cache-Control:max-age=5`），如果过期，则访问后端获取最新的文档，并放入Nginx本地缓存中，返回200状态码和最新的文档给浏览器；如果文档没有过期，即if-modified-since与缓存文档的last-modified匹配，则返回304状态码给浏览器。




## nginx缓存指定目录下的资源

```
http {
    # ...
    # 静态资源缓存：第一步
    proxy_cache_path tmp-test levels=1:2 keys_zone=tmp-test:100m inactive=7d max_size=1000g;
    # ...

    server {
        # ...
        # 静态资源
        location ~* ^.+\.(css|js|ico|gif|jpg|jpeg|png|webp)$ {
            # 静态资源缓存：第二步
            proxy_cache tmp-test; # 和 `proxy_cache_path tmp-test levels`的保持一致
            proxy_cache_valid  200 206 304 301 302 10d;
            proxy_cache_key $uri;
            proxy_set_header   Host             $host:$server_port;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
            expires 90d;
            proxy_pass http://localhost:8080; # 指向了开发环境的地址
        }
        # ...
    }
}
```
介绍下 `proxy_cache_path` 配置

- `proxy_cache_path tmp-test`：缓存的路径为与 `nginx` 启动程序所在的目录同级的 `tmp-test`
- `levels`：表示创建两级目录结构，比如 `/export/cache/proxy_cache//*/`，将所有文件放在一级目录结构中如果文件量很大会导致访问文件效率低。
- `keys_zone`：设置存储所有缓存 `key` 和相关信息的共享内存区，1M大约能存储8000个key。
- `inactive`：inactive指定被缓存的内容多久不被访问将从缓存中移除，以保证内容的新鲜;默认10分钟。
- `max_size`：最大缓存阀值，`cachemanager`进程会监控最大缓存大小，当缓存达到该阀值，该进程将从缓存中移除最近最少使用的内容。
- `use_temp_path`：如果为 `on`，则内容首先被写入临时文件(`proxy_temp_path`)，然后重命名到 `proxy_cache_path` 指定的目录;如果设置为 `off`，则内容直接被写入到 `proxy_cache_path` 指定的目录。
- `proxy_cache` 启用proxy cache，并指定key_zone。另外，如果 `proxy_cache off` 表示关闭掉缓存。

## location

```
location = /content {
    proxy_cache cache;
    proxy_cache_key $scheme$proxy_host$request_uri;
    proxy_cache_valid 200 5s;
    proxy_pass http://localhost/content$is_args$args;
    add_header cache-status $upstream_cache_status;
}
```

`proxy_cache`：指定使用哪个共享内存区域存储缓存键和相关信息;

`proxy_cache_key`：设置缓存使用的key，默认为访问的完整URL，根据实际情况设置缓存key;

`proxy_cache_valid`：为不同的响应状态码设置缓存时间;如果是 `proxy_cache_valid 5s` 则 `200`、`301`、`302`响应将被缓存;

`proxy_cache_lock`：当多个客户端同时请求同一份内容时，如果开启 `proxy_cache_lock` (默认`off`)则只有一个请求被发送至后端;其他请求将等待该内容返回;当第一个请求返回时，其他请求将从缓存中获取内容返回;当第一个请求超过了 `proxy_cache_lock_timeout` 超时时间(默认 `5s`)，则其他请求将同时请求到后端来获取响应，且响应不会被缓存;启用 `proxy_cache_lock` 可以应对雪崩效应。






## http2

网站升级到 `http2`，会相对节省流量

1.`nginx`

添加 `nginx` 配置，原本 `https` 的 `listen` 为 `listen 443 ssl;`，修改为 `listen 443 ssl http2;`，然后，重启 `nginx`

2.`HTTP/2的优势`

- 多路复用
可以用同一个连接传输多个资源。这可以使得 `http1` 的某些优化变得不必要。比如，
  - 使用雪碧图技术，把多张小图合成一张大图，减少请求数
  - 合并JS和CSS，减少请求数

在 `http 1.1` 时代，由于需要建立多个 `TCP` 连接，服务器需要更多的线程来处理请求，同样地，浏览器也需要，所以浏览器会限制同一个域的同时请求数。Chrome是限制6个，总连接数是17个，其它浏览器的个数有所浮动，但差不多。而 `http2` 几乎没有请求并发限制


- server push

html没有加载，其他资源是不会加载的，而 `http2` 可以让服务把客户端很有可能请求的资源先 `push` 到客户端，不用等到请求的时候才发送。这样可以提高整体的渲染速度。

`nginx` 官网目前暂不支持 `Server Push`，下面是一个 `http2` 的 `node.js` 的 demo
```js
response.push('/img/banner.png');
```


3.报文头压缩和二进制编码

`http2` 修改了报文的传输格式。由 `http1.1` 变为 `http2` 的 `HEADERS frame` 和 `DATA frame` 等的形式

如果浏览器不支持 `http/2` 会怎么样呢？也是能够正常打开的，为什么呢？因为建立 `https` 连接的时候需要先握手，浏览器或者客户端会发送一个Client Hello的包，这个包里面会说明它是否支持 `http2`，`nginx` 就能够根据握手信息决定是否使用 `http/2`，如果客户端不支持就使用 `http/1.1`



