# docker

操作容器
```bash
# 重启容器
$ docker restart containerId

# 停止容器
$ docker stop containerId

# 删除容器
$ docker rm containerId

# 启动镜像(启动一个后台运行的名称为web端口映射为80的nginx镜像)
$ docker run -d -p 80:80 --name web nginx

# 再启动一个(启动一个后台运行的名称为web8080端口映射为8080的nginx镜像)
$ docker run -d -p 8080:80 --name web8080 nginx
```

操作镜像
```bash
# 删除镜像
$ docker rmi imageId
```


# Dockerfile

```Dockerfile
# 1.1、拉取最新版镜像
FROM nginx

# 1.2、拉取指定版本的镜像
#FROM nginx:1.16.1

# 1.3、了解镜像结构
# docker run -it --rm nginx bash(以交互的方式进入容器，这里我们可以找到html目录。--rm:用完就删掉)

# 1.4、往index.html里写入内容
#RUN echo "hello Docker" > /usr/local/nginx/html/index.html

# 1.5、往docker镜像里写入文件
COPY nginx.conf /etc/nginx/

# 1.6、往镜像里写入文件夹
COPY dist /usr/local/nginx/html
```

# build

构建镜像
```bash
$ docker build -t web .
```

# run

运行镜像(后台运行，映射到80端口)
```bash
# -p 宿主机端口:容器端口
$ docker run -d -p 80:80 web
```

至此，就实现了一次成功的docker部署。


# build-delete

在构建成功后删除文件
```Dockerfile
FROM nginx

# docker容器里的工作目录
WORKDIR cd /usr/local/nginx/html
RUN rm -rf dist

COPY nginx.conf /etc/nginx
COPY dist /usr/local/nginx/html
```

# prune

删除虚悬镜像
```bash
$ docker stop containerId
$ docker rm containerId
$ docker prune
```

`docker prune` 命令
```
#删除 所有未被 tag 标记和未被容器使用的镜像：
docker image prune

#删除 所有未被容器使用的镜像：
docker image prune -a

#删除 所有停止运行的容器：
docker container prune

#删除 所有未被挂载的卷：
docker volume prune

#删除 所有网络：
docker network prune

#删除 docker 所有资源：
docker system prune
```

处理虚悬镜像
```
#查询显示虚悬镜像
docker images -f dangling=true

#删除虚悬镜像
docker rmi $(docker images -q -f dangling=true)
```


# cmd

其实所有的指令都在客户端端实现的，以RESTful的方式请求server端


# deploy

直接将上传的 `zip` 解压部署到 `docker`

```Dockerfile
FROM nginx

WORKDIR /usr/local/nginx/html

RUN rm -rf dist
COPY nginx.conf /etc/nginx

# 拷贝到当前目录(/usr/local/nginx/html)
COPY dist.zip .

RUN unzip dist.zip

RUN rm -rf dist.zip

# 将工作目录切换到本身的目录
WORKDIR /usr/local/nginx
```

上面这样做的原因是分层存储，内存不共享


# tar-deploy

直接使用 `tar` 包部署
```bash
# 将当前目录打包
$ tar -zcvf dist.tar.gz .
```

```Dockerfile
FROM nginx

WORKDIR /user/local/nginx/html

RUN rm -rf dist

# 配置文件默认在 `/etc/` 目录下
COPY nginx.conf /etc/nginx

# 自动解压缩tar包，然后，会自动删除tar.gz文件
ADD dist.tar.gz .
#等价于下面3行命令
#COPY dist.zip .
#RUN unzip dist.zip
#RUN rm -rf dist.zip

WORKDIR /usr/local/nginx
```


# volume

在docker容器里创建一个文件，其实是在宿主机上创建一个文件，然后，别的docker容器也可以读取到该文件，可以实现文件的共享。

如果删除了docker容器，那么，文件还存在。

之前的部署将文件拷贝到镜像里，现在可以不将文件copy到镜像里，直接映射宿主机上的目录到干净的nginx镜像里，同样，可以实现部署。

```bash
# -v 宿主机的目录:容器的目录
$ docker run -d -p 80:80 --name web nginx -v /usr/local/nginx/html:/usr/local/nginx/html
```

可以通过 `docker ps -a` 查看是否启动成功

以交互的方式进入容器，看下对应的 `/usr/loca/nginx/html` 的变化。

```bash
$ docker exec -it web bash
```

此时，看到的docker镜像里的目录是没有变化的。

同样，可以再次启动一个占用8081端口的容器

```bash
$ docker run -d -p 8081:80 --name web8081 nginx -v /usrs/local/nginx/html:/usrs/local/nginx/html
```

上面俩容器共享的是宿主机上 `/usr/local/nginx/html` 目录下的文件


# 修改容器

docker 进入容器的4种方式
```
docker exec -it CONTAINER_ID /bin/bash
docker exec -it CONTAINER_ID /bin/sh
docker exec -it CONTAINER_ID bash
docker exec -it CONTAINER_ID sh
```

# mysql

docker部署 `mysql` 数据库

```bash
$ docker run -p 3306:3306 --name mysql \
-v /usr/local/docker/mysql/conf:/etc/mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql
```

- `var` 目录存储数据
- `e` 环境变量(官方仓库提供的环境变量)

然后，可以使用 `navcat` 连接容器化的数据库


# import-sql

导入数据库

导入 `sql` 文件太大可能会报错，需要停止之前的 `mysql` 镜像修改默认的配置。

可以将配置的部分做成数据库卷，即使，容器消亡，配置还在。

`exit` 退出容器。`docker stop containerId` 停止镜像。`docker rm containerId` 删除镜像

> 默认的sql初始配置大小为16M

```bash
$ docker run -p 3306:3306 --name mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql
```

```bash
$ docker exec -it mysql bash
$ cd /etc/mysql/mysql.conf.d/
$ cat mysqld.cnf
$ cd ..
$ ls -al
$ cd conf.d/
$ ls -al
$ cat mysqldump.cnf
$ cd ..
$ cd mysql.conf.d/
$ ls -al
$ echo "max_allowed_packet=128M" >> mysqld.cnf
$ docker restart containerId
```

可以将配置的部分做成数据库卷，即使，容器消亡，配置还在。

将容器内的文件复制到宿主机。

先进入容器看下配置。

```
$ docker exec -it mysql bash
$ cd /etc/mysql
$ ls -al
```

在宿主机里进行copy。

```bash
# 将mysql容器里的etc目目录下的mysql文件夹拷贝到当前目录
root@unbuntu:/usr/local/docker/mysql/conf# docker cp mysql:/etc/mysql .
root@unbuntu:/usr/local/docker/mysql/conf# ls
root@unbuntu:/usr/local/docker/mysql/conf# cd mysql
root@unbuntu:/usr/local/docker/mysql/conf/mysql# mv *.* ..
root@unbuntu:/usr/local/docker/mysql/conf# rm -rf mysql
root@unbuntu:/usr/local/docker/mysql/conf# cd ..
root@unbuntu:/usr/local/docker/mysql# docker stop mysqlContainerId
root@unbuntu:/usr/local/docker/mysql# docker rm mysqlContainerId
root@unbuntu:/usr/local/docker/mysql# docker rmi mysqlImageId
root@unbuntu:/usr/local/docker/mysql# docker run -p 3306:3306 --name mysql \
-v /usr/local/docker/mysql/conf:/etc/mysql \
-v /usr/local/docker/mysql/logs:/var/log/mysql \
-v /usr/local/docker/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
-d mysql
```


# container-deploy

项目容器化部署

此时的项目目录：
```
|-- Dockerfile
|-- dist/
```

```bash
# -v 宿主机目录:docker镜像目录
$ docker run -d -p 80:80 web -v /usr/local/nginx/html:/usr/local/nginx/html
# 查看是否启动成功
$ docker ps -a
# 查看日志
$ docker logs web
# 一直监听日志的变化
$ docker logs -f web
```

# docker-deploy-java

docker 部署 java

1、Dockerfile编写

```Dockerfile
FROM openjdk:8-jre

RUN mkdir /app

COPY app.jar /app

CMD java -jar /app/app.jar --spring.profiles.active=prod

EXPOSE 8761
```


2、构建运行

```
docker build -t app .
docker run -d -p 8761:8761 app
```


# docker-eureka-cluster

集群

```docker-compose.yml

version: '3'
services:
  eureka-1:
    restart: always
    image: app
    container_name: eureka-1
    ports:
     - 8761:8761
  eureka-2:
    restart: always
    image: app
    container_name: eureka-2
    ports:
     - 8762:8761
  eureka-3:
    restart: always
    image: app
    container_name: eureka-3
    ports:
     - 8763:8761
```

后台运行
```
docker-compose up -d
```

