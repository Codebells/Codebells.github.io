---
title: docker常用指令基础
date: 2022-10-28 20:14:59
categories: tech
tags: tools
category_bar: true
excerpt: 常用docker命令记录
---

---

![docker架构](docker/576507-docker1.png)

| 概念                   | 说明                                                         |
| :--------------------- | :----------------------------------------------------------- |
| Docker 镜像(Images)    | Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。  |
| Docker 容器(Container) | 容器是独立运行的一个或一组应用，是镜像运行时的实体。         |
| Docker 客户端(Client)  | Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。 |
| Docker 主机(Host)      | 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。       |
| Docker Registry        | Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub([https://hub.docker.com](https://hub.docker.com/)) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 **<仓库名>:<标签>** 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 **latest** 作为默认标签。 |
| Docker Machine         | Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。 |

# docker安装

直接官方安装脚本自动安装吧，省的麻烦

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
# 启动docker服务
systemctl start docker
# 开机自启动
systemctl enable docker.service
```

# docker-compose安装

```shell
sudo curl -L "[https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-](https://github.com/docker/compose/releases/download/v2.2.2/docker-compose-)$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose version
```

# docker基础

```shell
#搜索ubuntu系统镜像
docker search ubuntu
#拉取最新的ubuntu镜像
docker pull ubuntu
#列出镜像
docker images
#列出当时运行的容器
docker ps
docker stop <ID/Name>
docker start
#docker run是新建容器并启动，docker start 是启动停止的容器
docker run
docker restart
docker rm
docker cp
#例子以命令行模式进入ubuntu容器
docker run -it ubuntu /bin/bash
```

- -d 后台运行
- -P 选项（大写）：随机端口映射
- -p 选项（小写）：指定端口映射,**前面是宿主机端口后面是容器端口**，如`docker run nginx -p 8080:80`，将容器的 80 端口映射到宿主机的 8080 端口，然后使用`localhost:8080`就可以查看容器中 nginx 的欢迎页了
- -t: 在新容器内指定一个伪终端或终端。
- -i: 允许你对容器内的标准输入 (STDIN) 进行交互。
- -v 选项：挂载宿主机目录，**前面是宿主机目录，后面是容器目录**,如`docker run -d -p 80:80 -v /dockerData/nginx/conf/nginx.conf:/etc/nginx/nginx.conf nginx` 挂载宿主机的`/dockerData/nginx/conf/nginx.conf`的文件，这样就可以在宿主机对`nginx`进行参数配置了,注意目录需要用绝对路径，不要使用相对路径，如果宿主机目录不存在则会自动创建。
- --rm : 停止容器后会直接删除容器，这个参数在测试是很有用，如`docker run -d -p 80:80 --rm nginx`
- --name : 给容器起个名字，否则会出现一长串的自定义名称如 `docker run -name niginx -d -p 80:80 - nginx`

### docker ps的各个输出的含义

CONTAINER ID: 容器 ID。

IMAGE: 使用的镜像。

COMMAND: 启动容器时运行的命令。

CREATED: 容器的创建时间。

STATUS: 容器状态。

状态有7种：

created（已创建）
restarting（重启中）
running 或 Up（运行中）
removing（迁移中）
paused（暂停）
exited（停止）
dead（死亡）
PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。

NAMES: 自动分配的容器名称。

## docker -d后如何进入

- **docker attach**：进入容器后exit，容器也会停止
- **docker exec**：推荐大家使用 docker exec 命令，因为此命令在exit时只会退出容器终端，但不会导致容器的停止

## docker 导入导出

```
#导出容器 1e560fca3906 快照到本地文件 ubuntu.tar
docker export 1e560fca3906 > ubuntu.tar
#将快照文件 ubuntu.tar 导入到镜像 test/ubuntu镜像tag为v1
cat docker/ubuntu.tar | docker import - test/ubuntu:v1
```
