---
description: Docker 数据卷
---

# Docker Volume

### 什么是 Volume

为了实现数据持久化，使容器之间可以共享数据。可以将容器内的目录，挂载到宿主机上或其他容器内，实现同步和共享的操作。即使将容器删除，挂载到本地的数据卷也不会丢失

### 使用 Volume

使用命令：

```bash
dokcer run -it -v 主机内目录:容器内目录 镜像名/id
```

将容器内目录挂载到主机内目录上，通过**docker inspect**命令查看该容器即可以看到挂载信息:

![](.gitbook/assets/docker\_inspect.png)

建立挂载关系后，容器和主机的文件将同步和共享

### 匿名挂载

```shell
docker run -d  -v 容器内目录  镜像名/id  # 匿名挂载
```

匿名挂载后，使用 **docker volume ls **命令查看所有挂载的卷：

![](.gitbook/assets/docker\_volume\_ls.png)

每一个VOLUME NAME对应一个挂载的卷，由于挂载时未指定主机目录，因此无法直接找到目录。

### 具名挂载

```bash
docker run -d -v 卷名：容器内目录 镜像名/id # 具名挂载
```

\
