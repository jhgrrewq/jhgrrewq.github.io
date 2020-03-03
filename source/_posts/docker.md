## 环境配置难度

虚拟机是带环境安装的一种解决方案，可以在一种操作系统里运行另一种才做系统，但仍有一些缺点：

- 资源占用多
- 冗余步骤多
- 启动慢

## linux 容器

由于虚拟机的这些缺点，linux 发展了另一种虚拟化技术：linux 容器

**linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离**。或者说对容器内进程来说，他接触的资源都是虚拟的，从而实现和底层系统的隔离

- 启动快

容器内的应用直接就是底层系统的一个进程，而不是虚拟机内部的进程。启动容器相当于启动本机的一个进程，而不是启动一个操作系统

- 资源占用少

容器只占用需要的资源，多个容器可共享资源；虚拟机因为是完整的操作系统，不可避免需要占用所有资源并独享资源

- 体积小

容器只要包含用的组件即可；虚拟接则是整个操作系统的打包

- 一致运行环境
- 持续交付和部署
- 更轻松的迁移

## docker

**docker 属于 linux 容器的一种封装，提供简单易用的容器接口，是目前最流行的 linux 容器解决方案**

 docker 将应用程序和该程序的依赖打包在一个文件。运行该文件会生成一个虚拟容器。程序在该虚拟容器里运行，无需担心环境问题

docker 接口相当简单，可以方便创建和使用容器。容器还可以进行版本管理、复制、分享、修改等

## docker 使用流程

- 通常会写一个或多个 Dockerfile 文件，一个文件对应一个容器。根据 Dockerfile 的文件配置可以烧录一个镜像（可理解为类似制作一个 USB启动盘）。当制作完成镜像，可以运行这个镜像，运行成功就变成了容器
- 对于启动和关联多个容器，可编写 docker-compose.yml
- 对容器进行编排做更多配置，可使用 Kubernetes 在不间断服务情况下更新容器

## docker 安装

docker 有两个版本： 社区版（Community Editon， 缩写CE）和企业版（Enterprise Edition，缩写EE), 一般使用社区版

### 安装

```js
# 升级 apt
sudo apt-get update
# 添加相关软件包
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
# 下载软件包的合法性，需要添加软件源的 GPG 密钥
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# source.list 中添加 Docker 软件源
sudo add-apt-repository "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu  $(lsb_release -cs)  stable"
# 安装 docker CE
sudo apt-get update
sudo apt-get install docker-ce
# 启动 docker CE
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker // 更新用户组
# 测试
docker run hello-world
# 使用 -ti 标志可以接管 docker 启动的 shell
docker run -ti ubuntu bash
```

### 镜像加速

```js
 # /etc/docker/daemon.json
{
  "registry-mirrors": [
    "https://dockerhub.azk8s.cn",
    "https://reg-mirror.qiniu.com"
] }

sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 手动构建镜像

- `docker image` 查看本地已有镜像

```js
$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              775349758637        6 weeks ago         64.2MB
hello-world         latest              fce289e99eb9        11 months ago       1.84kB
```

- `docker pull`  下载镜像 (所有镜像可在 hub.docker.com 中找到)

```js
$ docker pull centos
Using default tag: latest
latest: Pulling from library/centos
729ec3a6ada3: Pull complete 
Digest: sha256:f94c1d992c193b3dc09e297ffd54d8a4f1dc946c37cbeceb26d35ce1647f88d9
Status: Downloaded newer image for centos:latest
docker.io/library/centos:latest
```

- `docker rmi`  删除镜像 (后面可跟名称或者 imageId)

```js
$ docker rmi centos
Untagged: centos:latest
Untagged: centos@sha256:f94c1d992c193b3dc09e297ffd54d8a4f1dc946c37cbeceb26d35ce1647f88d9
Deleted: sha256:0f3e07c0138fbe05abcb7a9cc7d63d9bd4c980c3f61fea5efa32e7c4217ef4da
Deleted: sha256:9e607bb861a7d58bece26dd2c02874beedd6a097c1b6eca5255d5eb0d2236983
```

- `docker run`  运行一个镜像，产生一个容器

- `docker ps -a`  查看所有容器，不加 a 参数会隐藏停止的容器

```js
$ docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
admin@VM-0-9-ubuntu:~$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS                          PORTS               NAMES
0ba76322c986        ubuntu              "/bin/bash"         About a minute ago   Exited (0) About a minute ago                       laughing_shirley
287fe66ab63c        hello-world         "/hello"            19 minutes ago       Exited (0) 19 minutes ago                           affectionate_wu
```

- `docker rm`  删除一个停止容器
- `docker stop`  停止运行一个容器

```js
# 移除一个运行的容器会发生错误 
docker rmi fce289e99eb9
Error response from daemon: conflict: unable to delete fce289e99eb9 (must be forced) - image is being used by stopped container 76650e1cb919
```

### 编写 Dockerfile 文件

Dockerfile 所有指令可以在 Docker Documentation 中找到

#### 通过 Dockerfile 构建镜像

- `docker build` 在当前目录下找到 Dockerfile 文件构建

#### FROM 指令

自定义镜像基本是基于官方的基础镜像进行更改，使用 From 指令表示以何种镜像为基础

```bash
FROM Ubuntu
```

#### RUN 命令

run 命令用来执行一些 shell 命令，如安装依赖等，通常建议把命令尽量写到一行，因为一个指令就是一个镜像层，安装的命令何在一个层是一种最佳实践

```bash
RUN apt-get install -y vim
```

#### VOLUME 指令

将当前目录下所有文件同步到镜像中，默认 docker 容器中数据都是只读，如果要让他可写，并且可以共享给其他容器必须创建 volume

```bash
VOLUME ['/code']
WORKDIR /code
RUN node index.js
docker run -v $PWD/app:/code
```

#### COPY 和 ADD

这两个命令是把文件复制到容器中，相较而言，ADD 读取远程网络文件更加优秀，一般使用 COPY 即可

```bash
COPY . /code
```

#### EXPORT

将本机端口映射到容器内部端口

```bash
# 将本机 80 端口映射到容器内部 8080 端口
EXPORT 80 8080
```

#### WORKDIR

用于切换目录，类似 cd 命令

```bash
WORKDIR /code
```

#### 运行一个 server 实例

- 创建 Dockerfile (命令可使用小写)

```bash
# 从 node 镜像的 slim 版本构建， slim 版本是 node 应用最精简容器
from node:slim
# 将当前目录复制到容器 /code 目录下
copy . /code
# 进入 /code 目录
workdir /code
# 暴露容器 80 端口
expose 80
# 安装依赖
run npm install
# 运行 index.js 文件
cmd node index.js
```

- 创建 index.js

```js
const server = require('server')
const { get } = server.router
server({ port: 8080 }, [
  get('/', ctx => 'hello docker')
])
```

- 安装依赖

```bash
npm i server -D
```

- 忽略 node_modules

```js
# 创建 .dockerignore
node_modules
```

- 构建镜像,   t 参数可指定名字

```js
docker build . -t my_server
```

- 运行镜像，生成容器

```bash
docker run -p 8080:8080 my_server
```

### docker compose

#### 安装

```bash
sudo apt-get install docker-compose
```

#### 命令

- build 

```bash
# dicker-compose.yml 藐视了多个容器的属性，运行 build 命令会将自定义 Dockerfile build 成镜像再运行
# 若需要的镜像不存在，会通过 pull 从镜像仓库拉去
docker-compose build
```

- up

```bash
# up 可启动 docker-compose.yml 中所有服务
# -d 可让服务在后台运行
docker-compose up -d
```

- logs

```bash
# logs 查看后台启动的服务日志
docker-compose logs
```

- down

```bash
# 将所有通过 up 命令创建的容器停止并销毁
docker-compose down
```

- ps

```bash
# 查看通过 docker-compose 启动的容器的详细信息
docker-compose ps
```

- stop

```bash
# 停止通过 docker-compose up 启动容器
docker-compose stop
```

- start

```bash
# 启动被 stop 命令停止的容器
docker-compose start
```

- restart

```bash
# 重启容器
docker-compose restart
```

- exec

```bash
# 在某个容器中执行命令， 默认会接管一个 shell
docker-compose exec web sh
```

