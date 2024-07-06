---
title: Docker 学习笔记
date: 2024-06-06 12:28:00 -0400
categories: [学习笔记, Docker]
tags: [docker]
description: 本文记录一些关于 Docker 的笔记
---

## Docker 与虚拟机的区别

操作系统粗略来讲可以分两层，一层是内核层，负责与电脑硬件交互，一层是应用层，与用户交互。内核层是硬件和应用之间的桥梁。

虚拟机虚拟的是内核层和应用层。所以它很大，但好处是虚拟系统跟主机系统不需要有相关性。比如在 Windows 主机上可以虚拟 Linux 系统，反过来也行。因为虚拟机把内核层也虚拟了，就不关心主机的内核是什么。

Docker 只虚拟了应用层，而直接使用主机的内核层。好处是体积相对小，运行也快，但是这要求主机内核和虚拟机内核保持一致，否则 Docker 里的应用就没法运行了，就像 Windows 上的软件无法运行在 Linux 系统上一样。

Docker 是针对 Linux 平台开发的，因此要求主机是 Linux 内核。但是为了解决在其他平台使用 Docker 的问题，Docker Desktop 出现了。

简单来说，Docker Desktop 自带一个 Linux 内核，因此可以运行在其他系统上。（这也是为什么 Docker Desktop 在 Windows 上需要使用 WSL。）

## 基本使用

列举所有的镜像：

```shell
docker images # 等同于
docker image list
```
{: .nolineno}

查看所有正在运行的容器：

```shell
docker ps
```
{: .nolineno}

拉取某个特定版本的镜像（默认从 Docker Hub 上拉取）：

```shell
docker pull nginx:1.24
```
{: .nolineno}

拉取之后可以运行该镜像（当然不拉取直接运行，如果镜像不存在本地，也会自动拉取）：

```shell
docker run nginx:1.24
```
{: .nolineno}

运行后会持续占用当前终端，如果要查看当前正在运行的容器，需要再开一个终端查看。

`Ctrl + C` 结束当前终端运行的容器。

如果不想让运行的容器占用当前终端，可以加一个参数 `-d`，即 detached：

```shell
docker run -d nginx:1.24
# 4baf2b0e2f83c80f27cfb58f9a48fc59864c8a9022c98b021a687f4469ebe2fe
```
{: .nolineno}

该命令只会返回容器 ID，不会占用终端。

此时如果想看到容器运行时的日志，可以运行：

```shell
docker logs 4baf2b0e2f83
```
{: .nolineno}

目前为止，对于运行的容器，我们尚不能进入到容器内。其中一种办法是端口绑定。即把容器内的某个端口绑定到主机的某个端口，这样通过访问主机的这个端口，就访问到了容器内的那个端口。

先停掉正在运行的容器：

```shell
docker stop 4baf2b0e2f83
```
{: .nolineno}

使用 `-p` 参数指定端口绑定：

```shell
docker run -d -p 8181:80 nginx:1.24
# 6dcc28f46871270e840719e1975aaa14312ae5764391b5e5cdb893ed4632ec8f
```
{: .nolineno}

此时在主机访问 [http://localhost:8181/](http://localhost:8181/) 就会看到 Nginx 的默认界面了。

端口绑定的顺序是 `主机端口:容器端口` 。

由于每次运行 `docker run` 都会创建一个新的容器，因此目前为止已经存在很多容器了。

要查看所有容器，而不仅仅是正在运行的容器，可以使用 `-a` 参数：

```shell
docker ps -a
```
{: .nolineno}

要重新运行某个已经关闭的容器，可以运行：

```shell
docker start 4baf2b0e2f83
```
{: .nolineno}

目前为止都使用容器 ID 来开启或关闭容器，ID 不太好记，因此也可以使用容器名称（可以指定多个名称）：

```shell
docker stop gracious_merkle exciting_shockley
```
{: .nolineno}

要在创建容器时指定容器名称，使用 `--name` 参数：

```shell
docker run --name hello_nginx -d -p 8282:80 nginx:1.24
```
{: .nolineno}

## Dockerfile 基本写法

假设有一个项目，目录结构如下：

```plaintext
simpleserver/
  src/
    server.js
  package.json
```

要基于这个项目创建一个 Docker 镜像，需要写一个 Dockerfile 文件，并将该文件放在项目根目录下。

写 Dockerfile 的基本思路就是，先找一个要运行该项目的容器。因为这是一个 JavaScript 项目，因此需要 node 来运行，所以选择 node 容器：

```dockerfile
FROM node:20.11.1
```

然后把项目文件都拷贝到容器内（目标路径最后的斜杠不能少；详见 [Dockerfile COPY](#dockerfile-copy) ）：

```dockerfile
COPY package.json /app/
COPY src /app/
```

然后指定项目的工作目录，这是之后所执行的所有命令的工作目录：

```dockerfile
WORKDIR /app
```

在项目运行前，一般都需要一些准备工作，比如一个 JavaScript 项目可能需要先运行 `npm install` 以安装项目所需要的依赖（`RUN` 后面直接跟命令）：

```dockerfile
RUN npm install
```

最后就是当一个容器开始运行时，它要运行什么？对于当前这个容器来说，当然就是运行这个 JavaScript 项目了。在本地我们会使用 `node src/server.js` 运行，对于容器来说亦如此（命令的区别见下一节解释）：

```dockerfile
CMD ["node", "server.js"]
```

`CMD` 和 `RUN` 的区别是，前者在整个 Dockerfile 中只能存在一次，且存在于最后，即容器运行后所要运行的那个东西。

因此目前为止，整个 Dockerfile 如下：

```dockerfile
FROM node:20.11.1

COPY package.json /app/
COPY src /app/

WORKDIR /app

RUN npm install

CMD ["node", "server.js"]
```

写完 Dockerfile 之后，就可以制作镜像了。命令如下：

```shell
docker build -t simpleserver:1.0 .
```
{: .nolineno}

`-t` 指定的是这个镜像的名字以及标签（版本），最后的参数是 Dockerfile 的所在目录。

如果该命令是在 Dockerfile 的同级目录下执行的，则可以使用 `.` 表示当前目录。

制作完成后运行 `docker images` 就可以看到我们自己的镜像了，使用的方法与其他镜像无异。

## Dockerfile COPY

[官方文档](https://docs.docker.com/reference/dockerfile/#copy)：

- 源文件必须在构建环境下，即所有源文件必须且只能来自 Dockerfile 所在的目录内，不能来自上层目录等。
- 如果源是一个目录，则会拷贝目录下的所有内容，但不会拷贝这个目录本身。
- 如果源是一个文件，目标路径以 `/` 结尾，那么认为目标路径是一个目录，源会被创建为 `<dest>/<src>` 。
- 如果一下子指定了好几个源文件，则目标路径必须为目录，且必须以 `/` 结尾。
- 如果源是一个文件，目标路径没有以 `/` 结尾，则认为目标路径是一个文件，`<src>` 会被重命名为 `<dest>` 。
- 如果目标路径不存在，会自动创建，其所有不存在的父路径也会自动创建，类似于 `mkdir -p` 。
- 目标路径可以是绝对路径，也可以是相对路径，如果是相对路径，则其父目录是由 `WORKDIR` 指定的目录。所以 `WORKDIR` 的作用其实就像 `cd` 。随时切换之后命令的工作目录。

看完这些，就理解了，前面的 Dockerfile 文件中的 `COPY src /app/` 其实只拷贝了 `server.js` 文件，而没有 `src` 目录。所以之后的 `CMD` 命令中使用的是 `"server.js"` 而不是 `"src/server.js"`，也就是说，这个镜像的部分目录结构是这样的：

```text
/app/
  server.js
  package.json
```

如果一定要实现原目录结构，则可以这样编写：

```dockerfile
FROM node:20.11.1

COPY package.json /app/
COPY src /app/src/

WORKDIR /app

RUN npm install

CMD ["node", "src/server.js"]
```

## 没有 compose 的情况

`compose` 是用来帮助多个容器协同工作的。

因为每个容器都是一个单独的环境，所以它们之间并不互通。要想建立连接，需要使用 `docker network` 。

查看现有的 Docker 网络（默认有三个）：

```shell
docker network ls
# NETWORK ID     NAME      DRIVER    SCOPE
# e5201791bc03   bridge    bridge    local
# 72b747f4ccb7   host      host      local
# e6aeca684788   none      null      local
```
{: .nolineno}

创建新的网络（最后一个参数是自定义的网络名称）：

```shell
docker network create mongo-network
```
{: .nolineno}

使用 mongo 镜像创建容器，绑定端口，指定一些参数，并声明其所属的网络：

```shell
docker run -d \
-p 27017:27017 \
-e MONGO_INITDB_ROOT_USERNAME=admin \
-e MONGO_INITDB_ROOT_PASSWORD=123456 \
--network mongo-network \
--name mongodb \
mongo:latest
```
{: .nolineno}

然后使用 mongo-express 镜像创建容器（mongo-express 是一个 UI 界面）：

```shell
docker run -d \
-p 8081:8081 \
-e ME_CONFIG_MONGODB_ADMINUSERNAME=admin \
-e ME_CONFIG_MONGODB_ADMINPASSWORD=123456 \
-e ME_CONFIG_MONGODB_SERVER=mongodb \
--network mongo-network \
--name mongo-express \
mongo-express:latest
```
{: .nolineno}

两个容器都启动之后，就可以访问 [http://localhost:8081/](http://localhost:8081/) 查看 Mongo Express 了，默认账密是 admin:pass。

## 使用 Compose

`compose` 其实就是把上面的两个命令保存在一个 YAML 文件中，这样该配置文件可以随着项目一下发布，用户可以直接使用该文件构建这个容器网络，而不需要手动敲命令了。这个对于需要很多容器协同工作的情况很有利。

下面是配置文件内容：

```yaml
services:
    mongodb:
        image: mongo
        ports:
            - 27017:27017
        environment:
            MONGO_INITDB_ROOT_USERNAME: admin
            MONGO_INITDB_ROOT_PASSWORD: 123456
    mongo-express:
        image: mongo-express
        ports:
            - 8081:8081
        environment:
            ME_CONFIG_MONGODB_ADMINUSERNAME: admin
            ME_CONFIG_MONGODB_ADMINPASSWORD: 123456
            ME_CONFIG_MONGODB_SERVER: mongodb
        depends_on:
            - "mongodb"
```

关于首行的 `version` ，见 [Version and name top-level elements](https://docs.docker.com/compose/compose-file/04-version-and-name/) 、[https://stackoverflow.com/a/76157215](https://stackoverflow.com/a/76157215) 。

一个 `service` 就是一个容器，名称任意。每个服务下面指定镜像、端口映射、环境变量等，跟前面手动敲的命令别无二致。不过这里少了指定网络，因为当使用 `docker compose` 的时候，默认会给所有的服务（容器）创建一个网络。

运行命令：

```shell
docker compose -f .\mongo-services.yaml up -d
```
{: .nolineno}

`-f` 指定 YAML 文件；

`up` 类似于 `run` ，会创建容器，当然在此之前会先创建一个网络；

`-d` 是 detached 模式，在启动容器时不占用当前终端。

默认来说，执行该配置文件时，每个服务都是同时启动的。但是有时候有些服务会依赖另一个或几个服务。比如上面的场景中，`mongo-express` 依赖于 `mongo` ，因为只有数据库启动了，UI 才能连接到它，所以就有一行 `depends_on` ，指定了该服务依赖于哪些服务。只有当所有其依赖的服务都启动了，该服务才会启动。

在参数 `up` 的位置，还有其他几个：

- `start` ：启动已创建的容器，类似 `docker start` 。
- `stop` ：停止正在运行的容器，类似 `docker stop` 。
- `down` ：停止容器，删除容器，删除网络。把整个都清理了。

## 特别说明

前面说 `up` 跟 `run` 类似，但有一点重要的区别是，`run` 每次都会创建一个新的容器，但是 `up` 并不会，它会根据特定规则的名字构建容器，如果发现该名字的容器已经存在，就直接使用，而不会重新创建。

因此如果我们运行

```shell
docker compose -f .\mongo-services.yaml up
docker compose -f .\mongo-services.yaml stop
docker compose -f .\mongo-services.yaml up -d
```
{: .nolineno}

两次 `up` 启动的是同一堆容器，而没有新的被创建。

名字的构建规则就是 `工作目录名-服务名-序号` 。

`--project-name` 或者 `-p` 可以指定工作目录名。形如：

```shell
docker compose -p projects -f services.yaml up -d
```
{: .nolineno}

如果有的服务是我们自己构建的容器，则使用 `build` ：

```yaml
services:
    myapp:
        build: .
        ports:
            - 3000:3000
        environment:
            USERNAME: admin
            PASSWORD: 123456
    ...
```

`build` 指定 Dockerfile 文件所在的目录，一般来说跟该配置文件同级，因此使用 `.` 指定当前目录。构建完成后会自动创建容器。其他的参数就基本相同了。

## 安全问题

因为上面的 YAML 配置文件一般会随着项目一起上传到 Github 等托管网站，把账号密码直接写在文件里不是什么好事，我们可以用环境变量代替。

比如把上面的 `admin` 和 `123456` 替换为 `${APP_USERNAME}` 和 `${APP_PASSWORD}` 。（当然，该文件的其他用到该账密的地方也一并替换）

这样在实际运行时，只需要在终端中指定这两个变量的值，就可以了。
