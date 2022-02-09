# 网络基础配置

大量的互联网应用服务包括多个服务组件，这往往需要多个容器之间通过网络通信进行相互配合。

Docker 目前提供了映射容器端口到宿主主机和容器互联机制来为容器提供网络服务。

### 端口映射

在启动容器的时候，如果不指定参数，容器外部是无法访问容器内的网络应用和服务的。

当容器中运行一些网络应用，要让外部能访问到这些应用时，可以通过 `-P` 或 `-p` 参数来指定端口映射。当使用 `-P` 标记时，Docker 会随机映射一个 49000\~49900 的端口至容器内部开放的网络端口。

```shell
docker run -d -P --name my_postgres postgres:latest
```

当使用`-p`标记则可以指定要映射的端口，并且，一个指定端口只能绑定一个容器。支持的格式如下：

1、ip:host\__port/container\_port_

_2、ip::container\_port_

_3、host\_port/container\_port_

`postgres`的默认端口是 5432，我想把这个端口映射到宿主主机的 5000 端口上，命令如下：

```shell
docker run -d -p 5000:5432 my_postgres postgres:latest
```

{% hint style="info" %}
通过`docker inspect container_id`可以获取容器的网络配置信息
{% endhint %}

### 容器通信

容器的连接系统是除了容器端口映射之外的另一种可以与容器中应用交互的方式。它会在源与接受容器之间建立一个隧道，接受容器可以看到源容器指定的信息。

使用 --link 参数可以使容器之间安全的进行交互。参数格式：container\_name:alias\_name

下面先创建一个数据库容器

```
docker run -d --name db postgres
```

然后创建一个 web 容器，并连接 db 容器

```
docker rund -d --link db:db --name web carldu/blog-server
```

此时，web 容器和 db 容器建立了联系，我们可以在 web 容器里直接访问到 db 容器。

