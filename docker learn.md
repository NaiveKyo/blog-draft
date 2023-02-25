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



## Docker run

根据指定的 image 创建一个新的 container；

useage：

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```









# Use Docker Registry

构建私有 Docker 镜像仓库：

- https://docs.docker.com/registry/
- https://github.com/distribution/distribution

