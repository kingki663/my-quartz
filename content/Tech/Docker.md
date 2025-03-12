---
creation date: 2024-11-13 11:22
modification date: 2025-03-04 21:09
tags:
  - docker
  - container
---

参考：[Docker 10分钟快速入门_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1s54y1n7Ev/?spm_id_from=333.337.search-card.all.click&vd_source=dbefe8621d153f1b1aaef0768a993d25)

[🐳Docker概念，工作流和实践 - 入门必懂_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1MR4y1Q738/?spm_id_from=333.999.0.0&vd_source=dbefe8621d153f1b1aaef0768a993d25)

[前言 - Docker — 从入门到实践 (gitbook.io)](https://yeasy.gitbook.io/docker_practice/)




# 安装
完成安装并注册DockerHub
参考：[Windows10的Docker安装和配置_win10 docker安装部署_Hu Ziyin的博客-CSDN博客](https://blog.csdn.net/huziyin/article/details/124133078)

# Docker 简介
**Docker** 使用 `Google` 公司推出的 [Go 语言](https://golang.google.cn/) 进行开发实现，基于 `Linux` 内核的 [cgroup](https://zh.wikipedia.org/wiki/Cgroups)，[namespace](https://en.wikipedia.org/wiki/Linux_namespaces)，以及 [OverlayFS](https://docs.docker.com/storage/storagedriver/overlayfs-driver/) 类的 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 等技术，对进程进行封装隔离，属于[操作系统层面的虚拟化技术](https://en.wikipedia.org/wiki/Operating-system-level_virtualization)。由于隔离的进程独立于宿主和其它的隔离的进程，因此也称其为容器。最初实现是基于 [LXC](https://linuxcontainers.org/lxc/introduction/)，从 `0.7` 版本以后开始去除 `LXC`，转而使用自行开发的 [libcontainer](https://github.com/docker/libcontainer)，从 `1.11` 版本开始，则进一步演进为使用 [runC](https://github.com/opencontainers/runc) 和 [containerd](https://github.com/containerd/containerd)。


![Docker Architecture](https://yeasy.gitbook.io/~gitbook/image?url=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fvirtualization%2Fwindowscontainers%2Fdeploy-containers%2Fmedia%2Fdocker-on-linux.png&width=768&dpr=1&quality=100&sign=18c2c53c&sv=2)
**Docker** 在容器的基础上，进行了进一步的封装，从文件系统、网络互联到进程隔离等等，极大的简化了容器的创建和维护。使得 `Docker` 技术比虚拟机技术更为轻便、快捷。

下面的图片比较了 **Docker** 和传统虚拟化方式的不同之处。传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程；而容器内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便。

![Docker Vs VM](https://yeasy.gitbook.io/~gitbook/image?url=https%3A%2F%2F1881212762-files.gitbook.io%2F%7E%2Ffiles%2Fv0%2Fb%2Fgitbook-x-prod.appspot.com%2Fo%2Fspaces%252F-M5xTVjmK7ax94c8ZQcm%252Fuploads%252Fgit-blob-6e94771ad01da3cb20e2190b01dfa54e3a69d0b2%252Fvirtualization.png%3Falt%3Dmedia&width=768&dpr=1&quality=100&sign=f449895f&sv=2)

# 基本概念
镜像（image）：只读模版可以用以创建容器（可以理解为面向对象中的类），可以理解为虚拟机的快照（包含了所需的应用程序、依赖库及软件）

容器（container）：镜像的实例（可以理解为面向对象中的类的实例——对象），可以理解为一个虚拟机，但是比虚拟机体积更小

Docker仓库：用以存储镜像的地方，如：Dockerhub

Dockerfile：用来构建镜像的自动化脚本，运行Dockerfile就好比在操作系统中安装OS和软件一样
# 使用
注意需要在源代码所在位置创建Dockerfile然后再执行后面的操作
Docker使用步骤：Dockerfile->image->container
## Dockerfile
一般包含以下步骤：
1. 安装操作系统
2. 安装程序运行时
3. 复制应用程序、依赖包、配置文件
4. 执行启动命令运行程序

示例：
```Dockerfile
FROM golang:1-alpine3.17 AS builder
WORKDIR /go/src/github.com/meloalright/guora
ENV CC=gcc
COPY . .
RUN apk add --no-cache gcc musl-dev \
    && go build ./cmd/guora && mv guora /go/bin
###############
FROM alpine:3.17
COPY --from=builder /go/bin/guora /usr/local/bin
COPY --from=builder /go/src/github.com/meloalright/guora /guora
COPY configuration.example.yaml /etc/guora/configuration.yaml
WORKDIR /guora
CMD "guora" "-init"
```
`FROM`指明基础镜像`AS`设置该镜像的别名
`WORKDIR`指明该命令之后所有命令的工作目录
`ENV` 设置环境变量 `CC` 为 `gcc`，即指定使用 GCC 编译器进行编译和构建
`COPY . .`将当前目录（即 Dockerfile 所在目录）下的所有文件和子目录复制到容器的 `/go/src/github.com/meloalright/guora` 目录中（也即第一个参数为本地路径，第二个参数为目标路径）
- `RUN apk add --no-cache gcc musl-dev \ && go build ./cmd/guora && mv guora /go/bin`：运行指定的命令，执行构建 Guora 应用程序所需的操作：
    -   `apk add --no-cache gcc musl-dev` 安装必要的构建工具和依赖项。
    -   `go build ./cmd/guora` 编译 Guora 应用程序。
    -   `mv guora /go/bin` 将 Guora 应用程序复制到 `/go/bin` 目录下。
`FROM alpine:3.17`：从官方的 Alpine 3.17 镜像开始构建第二个阶段。

`COPY --from=builder /go/bin/guora /usr/local/bin`：从之前构建的 `builder` 阶段复制 Guora 可执行二进制文件到容器的 `/usr/local/bin` 目录中。
`COPY --from=builder /go/src/github.com/meloalright/guora /guora`：从之前构建的 `builder` 阶段复制 Guora 源代码到容器的 `/guora` 目录中。
`COPY configuration.example.yaml /etc/guora/configuration.yaml`：将主机上的 `configuration.example.yaml` 文件复制到容器内部的 `/etc/guora/configuration.yaml` 文件中，该文件用于配置 Guora 应用程序。
`WORKDIR /guora`：将工作目录设置为 `/guora`。
`CMD "guora" "-init"`：设置容器默认启动的命令，即在启动容器时自动运行 Guora 应用程序，并初始化数据库

注意：`RUN`是创建镜像时使用的命令，而`CMD`是运行容器时使用的
## docker-compose.yaml
用于管理多个Docker容器
示例：
```yaml
version: '3'
services:
  redis-db:
    image: redis:alpine
    volumes:
      - ./data:/data
    restart: always
  guora:
    build: .
    depends_on:
      - redis-db
    restart: always
    ports:
      - "8080:8080"

```
1.  `version: '3'`：指定 Docker Compose 文件的格式版本。  
2.  `services:`：开始定义服务列表，每个服务都是一个运行在 Docker 容器中的应用程序。
3.  `redis-db:`：定义了 Redis 数据库服务。   
    -   `image: redis:alpine`：指定 Redis 镜像的名称和标签，这里使用的是官方提供的 Alpine 版本的 Redis 镜像。
    -   `volumes:`：指定此容器需要挂载的数据卷，这里将本地主机当前目录下的 `data` 目录挂载到容器内部的 `/data` 目录中。
    -   `restart: always`：指定容器在任何情况下都应该自动重启。
4.  `guora:`：定义了 Guora 应用程序服务。
    -   `build: .`：使用 Dockerfile 文件构建 Guora 应用程序容器，`.` 表示 Dockerfile 文件位于当前目录中。
    -   `depends_on:`：指定此容器依赖的其他服务，这里依赖 Redis 数据库服务。
    -   `restart: always`：指定容器在任何情况下都应该自动重启。
    -   `ports:`：将容器内部的端口映射到主机上的端口，这里将容器内部的 8080 端口映射到主机上的 8080 端口。

`docker-compose up --build`构建并启动 Docker Compose 文件中定义的服务。`--build` 参数会强制重新构建所有服务的 Docker 镜像，即使它们已经存在

## .dockerignore
- `.dockerignore` 文件的作用类似于 `git` 工程中的 `.gitignore` 。不同的是 `.dockerignore` 应用于 `docker` 镜像的构建，它存在于 `docker` 构建上下文的根目录，用来排除不需要上传到 `docker`服务端的文件或目录。

- `docker` 在构建镜像时首先从构建上下文找有没有 `.dockerignore` 文件，如果有的话则在上传上下文到 `docker` 服务端时忽略掉 `.dockerignore` 里面的文件列表。

优点：
- 构建镜像时能避免不需要的大文件上传到服务端，从而拖慢构建的速度、网络带宽的消耗；
- 可以避免构建镜像时将一些敏感文件及其他不需要的文件打包到镜像中，从而提高镜像的安全性；

## Dockerhub
### 将镜像上传到Dockerhub
1. 创建Dockerhub账号并在本地验证登录`docker login`
2. 将容器commit成镜像：`docker tag <existing-image> <hub-user>/<repo-name>[:<tag>]`
3. `docker push<hub-user>/<repo-name>:<tag>`

## 基本命令


### 生成镜像
`docker build -t <image-name> .`
注意：`-t`为镜像打上一个标签帮助我们记忆，`.`代表在当前目录下寻找Dockerfile
还有其他可选参数
### 管理镜像
查看：`docker images`
移除：`docker rmi -f <image-name>`


### 创建并运行容器
官方例子：`docker run -d -p 80:80 docker/getting-started`或者写成`docker run -dp 80:80 docker/getting-started`
-   `-d` 在分离模式下（在后台）运行容器。
-   `-p 80:80` 将主机的端口 80 映射到容器中的端口 80。
-   `docker/getting-started` 指定要使用的映像（image）
然后浏览器打开`http://localhost/tutorial/`，能看到页面即可
后面可以加参数，具体可用`docker run --help`来查看可选参数，`-h`是run本身的一个参数故只能用`--help`的全称来获取帮助


常用其他参数：
- `--name your-name`:给容器分配名字
- `-v host-directory container-directory`:将主机文件系统的目录`host-directory`映射到容器内部的目录`container-directory`
- `--privileged=true`:设置容器为特权容器，允许容器在主机上执行一些特权操作

### 获取容器ID
`docker ps`(ps表示process status)
后面可以加参数，具体可用`docker ps -h`来查看可选参数

### 运行容器
`docker start <container-id>`

### 停止容器
`docker stop <container-id>`
Ps:若容器很少，可简写容器ID前几位即可
可选参数只有一个：
-t Seconds to wait for stop before killing it (default 10)
### 移除容器
`docker rm <container-id>`
Ps:若容器很少，可简写容器ID前几位即可
后面可以加参数，具体可用`docker rm -h`来查看可选参数
### 与容器进行交互
进入已运行的容器进行交互
`docker exec -it <container-name> bash` 表示以终端（terminal）和容器进行交互（interactive）

### 查看容器日志
`docker logs <container_id>`

## 使用VScode的Docker插件
参考：[开始在 Visual Studio Code 中使用 Docker 应用 | Microsoft Learn](https://learn.microsoft.com/zh-cn/visualstudio/docker/tutorials/docker-tutorial)

# 制作BV2镜像
## .dockerignore
```json
img
Data
bert
.git
emotionnal
```
Data和bert文件夹主要存放模型，主动忽略，使用数据卷进行映射以降低镜像体积

## Dockerfile
```Dockerfile
# 基础CUDA镜像
FROM cnstark/pytorch:2.0.1-py3.10.11-cuda11.8.0-ubuntu22.04

LABEL maintainer="1581690775@qq.com"
LABEL version="dev-20240525"
LABEL description="Docker api image for Bert-Vits2"

# 安装第三方包
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC
# 最后清理包缓存以减小镜像体积
RUN apt-get update && \
    apt-get install -y --no-install-recommends tzdata ffmpeg libsox-dev parallel aria2 && \
    rm -rf /var/lib/apt/lists/* 

# 复制并安装python依赖
WORKDIR /workspace
COPY requirements.txt /workspace/
RUN pip install --no-cache-dir -r requirements.txt
RUN pip install chardet

# 复制剩余的应用
COPY . /workspace

EXPOSE 5000

CMD ["python", "hiyoriUI.py"]

```

## 调试Docker镜像
### Docker配置镜像如何调试？
1. 使用Docker Build的--debug参数
在使用docker build命令构建镜像时,可以添加--debug参数,这将打印出整个构建过程中的详细信息,包括执行的每个构建步骤、环境变量值以及出现的任何问题。这对于诊断构建过程中出现的问题非常有帮助。

2. 使用Docker Run的--rm参数
在运行容器时,可以使用--rm参数,这将在容器退出后自动删除它。这使得你可以迅速地重新创建和测试容器,而无需手动清理。

3. 进入正在运行的容器
如果容器已经在运行,可以使用`docker exec -it <container_id> /bin/bash`命令进入容器内部,然后手动检查配置文件、运行进程、环境变量等,帮助诊断问题所在。

4. 查看容器日志
使用`docker logs <container_id>`命令查看容器的日志输出,可以快速发现问题所在。如果需要持续监控日志,可以使用docker logs -f <container_id>命令。

5. 使用DockerVolUMEs
通过将本地目录挂载为容器卷,可以在本地直接修改配置文件,然后在容器内重新加载配置,而无需重新构建镜像。这可以加快调试周期。
### 调试BV2镜像
1. 构建：`docker build -t bert-vits2-api .`
2. 运行容器（注意需要指定数据卷）：
```
docker run -it --rm --gpus=all --shm-size="16G" --name bert-vits2-container -v D:\SIP\BV2\Data:/workspace/Data -v D:\SIP\BV2\bert:/workspace/bert -v D:\SIP\BV2\config.yml:/workspace/config.yml -p 5000:5000 bert-vits2-api  /bin/bash
```
