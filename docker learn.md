# Preface

网址：

- https://www.docker.com/
- https://docs.docker.com/

Docker 各项组件：

（1）Docker Desktop：Docker 的桌面端应用程序，可以用来很方便的操作 Docker，当然也可以选择直接使用 docker-cli；

（2）Docker Engine：开源容器技术，可以将应用程序构建并容器化；

（3）Docker Build：是 Docker Engine 中用的最广泛的特性之一，可以使用 Docker Build 构建镜像；

（4）Docker Compose：Compose 是一种工具，适用于多容器的应用，比如 SpringBoot 微服务项目，整个项目由多个服务模块组成，每个服务模块都可以容器化，此时可以使用 Docker Compose；

（5）Docker Hub：Docker 官方提供的一个用来存储镜像的服务，相当于容器的仓库；



# CentOS Install Docker-ce

根据官网教程即可：https://docs.docker.com/engine/install/centos/



# Docker-Engine Cammand

主要学习如何使用 docker-cli：https://docs.docker.com/reference/

docker cli 的命令有很多，按需查找，下面介绍一些常用的。



## docker search

从 Docker-Hub 中查找 images；

```bash
docker search [OPTIONS] TERM
```





## docker pull

从 Registry 中下载 image。

```bash
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```





## docker images

列出所有本机 pull 下来的 images；

```bash
docker images [OPTIONS] [REPOSITORY[:TAG]]
```



## docker run

根据指定的 image 创建一个新的 container；

useage：

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```



## docker exec

在运行的容器中执行命令；

```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

容器在运行过程中是和宿主机隔绝的，如果要访问容器中运行的某个服务，则需要借助 docker exec 命令访问目标容器，然后执行相关命令。

比如进入容器内部的操作系统：

```bash
docker exec -it CONTAINER bash
```



## docker ps

列出所有正在运行的容器的信息；



## docker container

https://docs.docker.com/engine/reference/commandline/container/

通过 docker run 运行容器后，可以通过 docker container 系列的命令管理运行中的容器，常用的命令有：

（1）docker container ls：和 docker ps 一样；

（2）docker container rm：和 docker rm 一样；

（3）**docker container prune**：移除所有处于 stopped 状态的容器；

。。。。。。

## docker volume

https://docs.docker.com/engine/reference/commandline/volume_create/

容器中所有状态不会影响到宿主机，一旦结束容器运行，其中所有的数据也随之消失，如果想要将某些数据持久化到宿主机磁盘，可以通过创建 volume 的形式将物理机的某个目录和容器内的某个目录关联，容器中该目录下的数据变化会在物理机中发生。需要注意的时一旦结束了容器，创建的 volume 不会受到影响，如果某个 volume 没有容器引用，记得清除它们。通过 `docker volume rm` 或 `docker volume prune`。









# Use Docker Registry

构建私有 Docker 镜像仓库：

- https://docs.docker.com/registry/
- https://github.com/distribution/distribution

