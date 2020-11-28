# 入门


## 开始


**在CentOS 7上安装Docker**

使用 yum 从软件仓库安装 Docker
```
yum install docker
```

首先启动 Docker 的守护进程
```
service docker start
```

如果想要 Docker 在系统启动时运行，执行
```
chkconfig docker on
```

>Docker 在 CentOS 上好像和防火墙有冲突，应用防火墙规则后可能导致 Docker 无法联网，重启 Docker 可以解决


## 运行一个容器


使用 Docker 最关键的一步就是从镜像创建容器。有两种方式可以创建一个容器：使用 `docker create` 命令创建容器，或者使用 `docker run` 命令运行一个新容器。两个命令并没有太大差别，只是前者创建后并不会立即启动容器

以 Ubuntu 为例，我们启动一个新容器，并将 Ubuntu 的 Shell 作为入口

```
docker run -it ubuntu:latest sh -c '/bin/bash'
```

成功创建了一个 Ubuntu 的容器，并将当前终端连接为这个 Ubuntu 的 `bash shell`。这时候就可以愉快地使用 Ubuntu 的相关命令了

- `-i` 表示这是一个交互容器，会把当前标准输入重定向到容器的标准输入中，而不是终止程序运行
- `-t` 指为这个容器分配一个终端


按 `Ctrl+D`  可以退出这个容器


在容器运行期间，可以通过 `docker ps` 命令看到所有当前正在运行的容器。添加 `-a` 参数可以看到所有创建的容器

```
[root@localhost ~]# docker ps -a
CONTAINER ID        IMAGE                        COMMAND                CREATED             STATUS                      PORTS               NAMES
cb2b06c83a50        ubuntu:latest                "sh -c /bin/bash"      7 minutes ago       Exited (0) 7 seconds ago                        trusting_morse 
```


每个容器都有一个唯一的 `CONTAINER ID` 标识，通过 ID 可以对这个容器进行管理和操作

在创建容器时，可以通过 `--name`  参数指定一个容器名称，如果没有指定系统将会分配一个，就像这里的`「trusting_morse」`

按 `Ctrl+D`  退出容器时，命令执行完了，所以容器也就退出了。要重新启动这个容器，可以使用 `docker start`  命令


```
docker start -i trusting_morse
```

同样，`-i` 参数表示需要交互式支持


>每次执行 `docker run` 命令都会创建新的容器，建议一次创建后，使用 `docker start/stop` 来启动和停用容器


## 存储

在 Docker 容器运行期间，对文件系统的所有修改都会以增量的方式反映在容器使用的联合文件系统中，并不是真正的对只读层数据信息修改。每次运行容器对它的修改，都可以理解成对夹心饼干又添加了一层奶油。这层奶油仅供当前容器使用。当删除 Docker 容器，或通过该镜像重新启动时，之前的更改将会丢失。这样做并不便于我们持久化和共享数据

>容器内数据持久化的问题，参考[在容器中保存数据](./在容器中保存数据.md)

在创建容器时，通过 -v 参数可以指定将容器内的某个目录作为数据卷加载


```
docker run -it -v /home/www ubuntu:latest sh -c '/bin/bash'
```

在容器中会多一个 `/home/www` 挂载点，在这个挂载点存储数据会绕过联合文件系统。可以通过下面的命令来找到这个数据卷在主机上真正的存储位置

```
docker inspect -f {{.Volumes}} trusting_morse
```

看到输出了一个指向到 `/var/lib/docker/vfs/dir/...` 的目录。CD 进入后可以发现在容器中对 `/home/www`  的读写创建，都会反映到这儿。事实上，`/home/www` 就是挂载自这个位置


有时候，我们需要将本地的文件目录挂载到容器内的位置，同样是使用数据卷这一个特性，`-v` 参数格式为

```
docker run -it -v [host_dir]:[container_dir]
```


`host_dir` 是主机的目录，`container_dir` 是容器的目录



## 网络

Docker 容器内的系统工作起来就像是一个虚拟机，容器内开放的端口并不会真正开放在主机上。可以使我们的容器更加安全，而且不会产生容器间端口的争用。想要将 Docker 容器的端口开放到主机上，可以使用类似端口映射的方式

在 `Docker` 容器创建时，通过指定 `-p` 参数可以暴露容器的端口在主机上

```
docker run -it -p 22 ubuntu sh -c '/bin/bash'
```

现在我们就将容器的 `22` 端口开放在了主机上，注意主机上对应端口是自动分配的。如果想要指定某个端口，可以通过 `-p [主机端口]:[容器端口]` 参数指定



容器和容器之间想要网络通讯，可以直接使用 `--link`  参数将两个容器连接起来。例如 `WordPress` 容器对 `some-mysql` 的连接：

```
docker run --name some-wordpress --link some-mysql:mysql -p 8080:80 -d wordpress
```



## 环境变量

通过 Docker 打包的应用，对外就像是一个密闭的 exe 可执行文件。有时候希望 Docker 能够使用一些外部的参数来使用容器，这时候参数可以通过环境变量传递进去

比如 MySQL 数据库连接这种的东西。环境变量通过 -e 参数向容器传递：

```
docker run --name some-wordpress -e WORDPRESS_DB_HOST=10.1.2.3:3306 \
    -e WORDPRESS_DB_USER=... -e WORDPRESS_DB_PASSWORD=... -d wordpress
```


# 创建镜像

Docker 强大的威力在于可以把自己开发的应用随同各种依赖环境一起打包、分发、运行。要创建一个新的 Docker 镜像，通常基于一个已有的 Docker 镜像来创建。Docker 提供了两种方式来创建镜像：把容器创建为一个新的镜像、使用 `Dockerfile` 创建镜像


## 将容器创建为镜像

为了创建一个新的镜像，先创建一个新的容器作为基底
```
docker run -it ubuntu:latest sh -c '/bin/bash'
```


现在我们可以对这个容器进行修改了，例如我们可以配置 PHP 环境、将我们的项目代码部署在里面等

```
apt-get install php
```

当执行完操作之后，我们按 Ctrl+D  退出容器，接下来使用 `docker ps -a` 来查找我们刚刚创建的容器 ID

```
docker ps -a
```


可以看到我们最后操作的那个 Ubuntu 容器。这时候只需要使用 `docker commit`  即可把这个容器变为一个镜像了

```
docker commit 8d93082a9ce1 ubuntu:myubuntu
```

这时候 docker 容器会被创建为一个新的 Ubuntu 镜像，版本名称为 `myubuntu` 。以后我们可以随时使用这个镜像来创建容器了，新的容器将自动包含上面对容器的操作

如果我们要在另外一台机器上使用这个镜像，可以将一个镜像导出
```
docker save -o myubuntu.tar.gz ubuntu:myubuntu
```

现在我们可以把刚才创建的镜像打包为一个文件分发和迁移了。要在一台机器上导入镜像，只需要：
```
docker import myubuntu.tar.gz
```

这样在新机器上就拥有了这个镜像

>通过导入导出的方式分发镜像并不是 Docker 的最佳实践，因为可以使用 Docker Hub。Docker Hub 提供了类似 GitHub 的镜像存管服务。一个镜像发布到 Docker Hub 不仅可以供更多人使用，而且便于镜像的版本管理。在一个企业内部可以通过自建 Docker Registry 的方式来统一管理和发布镜像。将 Docker Registry 集成到版本管理和上线发布的工作流之中



## 使用 Dockerfile 创建镜像


使用命令行的方式创建 Docker 镜像通常难以自动化操作，可以使用 `Dockerfile` 来创建 `Docker` 镜像


`Dockerfile` 是一个纯文本文件，它记载了从一个镜像创建另一个新镜像的步骤

写好 `Dockerfile` 文件之后，就可以轻而易举的使用 `docker build` 命令来创建镜像

以下命令在 `Dockerfile` 中常被使用

命令 | 参数 | 说明
---|---|---
# |	- |	注释说明
ADD |	<src> <dest> |	将本地的<src>添加到镜像容器中的<dest>位置
CMD |	["executable","param1","param2"] |	容器启动时默认执行的命令。注意一个Dockerfile中只有最后一个CMD生效。
ENTRYPOINT |	["executable", "param1", "param2"] |	容器的进入点
ENV |	<key> <value> 或者 <key> = <value> |	指定环境变量值
EXPOSE |	<port> [<port>...] |	将指定的端口暴露给主机
FROM |	<image>[:<tag>] |	从一个已有镜像创建，例如ubuntu:latest
MAINTAINER |	Author <some-one@example.com> |	镜像作者名字，如Max Liu <some-one@example.com>
RUN |	<cmd>或者['cmd1', 'cmd2'…] |	在镜像创建用的临时容器里执行单行命令
VOLUME |	<path>或者['/var', 'home'] |	将指定的路径挂载为数据卷


下面是一个 `Dockerfile` 的例子
```
# This is a comment
FROM ubuntu:14.04
MAINTAINER username <username@example.com>
RUN apt-get update && apt-get install -y ruby ruby-dev
RUN gem install sinatra 
```

- `CMD` 命令可用指定 Docker 容器启动时默认的命令，例如上面例子提到的 `docker run -it ubuntu:latest sh -c '/bin/bash'`。其中 `sh -c '/bin/bash'`  就是通过手工指定传入的 `CMD`。如果不加这个参数，那么容器将会默认使用 CMD 指定的命令启动
- `ENTRYPOINT` 用来指定特定的可执行文件、Shell 脚本，并把启动参数或 `CMD` 指定的默认值，当作附加参数传递给 `ENTRYPOINT`

```
# 只能对mysql进行操作
ENTRYPOINT ['/usr/bin/mysql']
# 默认连接到192.168.100.128 这个主机
CMD ['-h 192.168.100.128', '-p']
```


假设这个镜像内已经准备好了 `mysql-client`，那么通过这个镜像，不加任何额外参数启动容器，将会给我们一个 `mysql` 的控制台，默认连接到 `192.168.100.128` 这个主机。然而我们也可以通过指定参数，来连接别的主机。但是不管无论如何，我们都无法启动一个除了 `mysql` 客户端以外的程序。因为这个容器的 `ENTRYPOINT` 就限定了我们只能在 `mysql` 这个客户端内做事情。这下是不是明白了~


在准备好 Dockerfile 之后，我们就可以创建镜像
```
docker build -t starlight36/simpleoa .
```


