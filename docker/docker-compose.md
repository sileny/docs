# docker-compose

[docker-compose-install](https://docs.docker.com/compose/install/)

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ sudo chmod +x /usr/local/bin/docker-compose
```

> Note: If the command docker-compose fails after installation, check your path. You can also create a symbolic link to /usr/bin or any other directory in your path.

```bash
$ sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ docker-compose --version
```


# 编排docker-compose.yml

docker-compose.yml内容如下：
```yml
version: '3'
services:
  web:
    build: .
    ports:
      # 宿主机端口:镜像端口
      - "80:80"
```


# 构建并启动程序

```bash
# 启动
$ docker-compose up
# 后台启动
$ docker-compose up -d

# 查看镜像是否运行
$ docker ps -a
```


# 停止

```bash
$ docker-compose down
$ docker-compose stop
```


# docker-compose开发环境部署

docker-compose.yml
```yml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

- `volumes` 挂载宿主机的当前有 `docker-compose.yml` 的目录到容器的 `/code` 目录，这样不必重新构建镜像
- `environment` 开发模式下当文件发送变化后自动重新加载


# 自定义部署

docker-compose.yml
```yml
version: '3'
services:
  web:
    restart: always
    image: 'web:v1.0'
    container_name: web
    build: .
    ports:
      - "80:80"
    volumes:
      - /usr/local/nginx/html:/usr/local/nginx/html
  mysql:
    restart: always
    image: 'mysql:v1.0'
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      TZ: Asis/Shanghai
      MYSQL_EOOR_PASSWORD: 123456
    volumes:
      - .:/var/lib/mysql
    command:
      --max_allowed_packet=128M
```

启动
```bash
$ docker-compose up -d
```


# 编排俩服务
```yml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```
定义了俩服务: `web` and `redis`.
