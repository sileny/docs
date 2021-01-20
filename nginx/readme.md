# nginx

- [灰度发布](#灰度发布)

# 根据ip实现灰度发布

1.原理

同时把两个不同版本的代码拉成两个项目，根据ip来判断用户可以去哪个项目，灰度发布的项目目录指向高版本的项目，其他ip的所有用户仍然访问相对的低版本的项目。

2.nginx配置
```
server {
    listen 80;

    server_name  domain.com;

    gzip on;
    charset utf-8;

    set $current_dir  /var/www/a/dist;

    if ($remote_addr = 1.2.3.4) {
        set $current_dir  /var/www/b/build;
     }

    location / {
        root $current_dir;
        index  index.html;
        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```
