# Prerequirement

- Docker 环境；

参考网址：

Redis 官网关于容器化的 get started：https://redis.io/docs/stack/get-started/install/docker/

注意，这里使用的是 Redis-stack；

/data/redis

/opt/redis-stack/etc/redis-stack.conf

```bash
docker run --name redis -d -p 6379:6379 -p 8001:8001 -v /usr/local/docker_data/redis/local_data/:/data/redis -v /usr/local/docker_data/redis/local-redis-stack.conf:/opt/redis-stack/etc/redis-stack.conf redis/redis-stack:latest redis-server /opt/redis-stack/etc/redis-stack.conf
```

Redis 各个版本的配置文件 example：https://redis.io/docs/management/config/

修改配置文件：

```bash
daemonize no; # 无需修改, 因为 docker 启动容器时已经附加了 -d
# bind 127.0.0.1 -::1 # 注释掉, 允许远程连接
# protected-mode no # 改为 no, 允许远程连接
requirepass 123456 # 设置密码访问 redis-server
dir /data/redis # 注意 redis-stack 镜像中 Redis 数据存放的位置是 /data/redis
appendonly yes # 允许数据持久化
```



启动 redis-stack：

```bash
docker run -d -p 6379:6379 -p 8001:8001 -v /usr/local/docker_data/redis/local_data/:/data -v /usr/local/docker_data/redis/redis-stack.conf:/redis-stack.conf --name redis redis/redis-stack:latest
```



注意：使用 RedisDesktopManager 工具连接容器中的 docker 时，本机不要使用代理工具，会一致提示连接错误。