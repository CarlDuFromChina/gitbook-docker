# Dockerfile

`Dockerfile`是用来构建`docker`镜像的文本文件，文本内容包含了一条条构建镜像所需的指令和说明。

### 定制镜像

这里讲解如何运行 Dockerfile 文件来定制一个镜像，具体的 Dockerfile 文件内的指令详解

#### 1. 构建一个nginx镜像

我们创建一个 Dockerfile 文件，文件内容如下：

```docker
FROM nginx
RUN echo '这是一个本地构建的nginx镜像' > /usr/share/nginx/html/index.html
```

{% hint style="warning" %}
**注意**：Dockerfile 每个保留关键字（指令）都必须是 **大写字母**
{% endhint %}

#### 2. FROM 和 RUN 指令的作用

**FROM**：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。

**RUN**：用于执行后面跟着的命令行命令。有以下俩种格式：

shell 格式：

```shell
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。
```

exec 格式：

```shell
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

{% hint style="warning" %}
**注意**：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。
{% endhint %}

例如：

```docker
FROM node:14-alpine as node
WORKDIR /app
COPY ./ /app/
RUN npm config set registry https://registry.npm.taobao.org
RUN npm install
RUN npm run build
```

以上执行会创建三层镜像，我们可以这样优化：

```docker
FROM node:14-alpine as node
WORKDIR /app
COPY ./ /app/
RUN npm config set registry https://registry.npm.taobao.org \
    && npm install \
    && npm run buildDo
```

如上，以 && 符号连接命令，这样执行后，只会创建 1 层镜像。

#### 3. 开始构建镜像

在 Dockerfile 文件目录下，执行构建操作：

{% hint style="info" %}
. 代表本次执行的上下文路径
{% endhint %}

```
docker build -t [标签] -f [文件路径] .  # 文件名为Dockerfile时可省略，最后的.不要忽略
```

### 命令详解

| 命令             | 效果                                            |
| -------------- | --------------------------------------------- |
| FROM           | 基础镜像：Centos/Ubuntu                            |
| MAINTAINER     | 镜像作者+邮箱                                       |
| RUN            | 镜像构建的时候需要运行的命令                                |
| ADD            | 为镜像添加内容（压缩包）                                  |
| WORKDIR        | 镜像工作目录（进入容器时的目录）                              |
| VOLUME         | 挂载的目录                                         |
| EXPOSE         | 暴露端口配置                                        |
| CMD/ENTRYPOINT | 指定这个容器启动时要运行的命令（CMD替代先前命令，ENTRYPOINT在先前命令后追加） |
| COPY           | 类似于ADD，将文件拷贝到镜像中                              |
| ENV            | 构建时设置环境变量                                     |

#### COPY

复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

格式：

```docker
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
```

\[--chown=\<user>:\<group>]：可选参数，用户改变复制到容器内文件的拥有者和属组。

<源路径>：源文件或者源目录，这里可以是通配符表达式，其通配符规则要满足 Go 的 filepath.Match 规则。例如：

```docker
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

<目标路径>：容器内的指定路径，该路径不用事先建好，路径不存在的话，会自动创建。

#### ADD 

ADD 指令和 COPY 的使用格类似（同样需求下，官方推荐使用 COPY）。功能也类似，不同之处如下：

* ADD 的优点：在执行 <源文件> 为 tar 压缩文件的话，压缩格式为 gzip, bzip2 以及 xz 的情况下，会自动复制并解压到 <目标路径>。
* ADD 的缺点：在不解压的前提下，无法复制 tar 压缩文件。会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。具体是否使用，可以根据是否需要自动解压来决定。

#### CMD

类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:

* CMD 在docker run 时运行。
* RUN 是在 docker build。

**作用**：为启动的容器指定默认要运行的程序，程序运行结束，容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

{% hint style="warning" %}
**注意**：如果 Dockerfile 中如果存在多个 CMD 指令，仅最后一个生效。
{% endhint %}

格式：

```
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] 
CMD ["<param1>","<param2>",...]  # 该写法是为 ENTRYPOINT 指令指定的程序提供默认参数
```

推荐使用第二种格式，执行过程比较明确。第一种格式实际上在运行的过程中也会自动转换成第二种格式运行，并且默认可执行文件是 sh。

#### ENTRYPOINT

类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

但是, 如果运行 docker run 时使用了 --entrypoint 选项，将覆盖 CMD 指令指定的程序。

**优点**：在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

{% hint style="warning" %}
**注意**：如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令，仅最后一个生效。
{% endhint %}

格式：

```docker
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

可以搭配 CMD 命令使用：一般是变参才会使用 CMD ，这里的 CMD 等于是在给 ENTRYPOINT 传参，以下示例会提到。

示例：

假设已通过 Dockerfile 构建了 nginx:test 镜像：

```docker
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

1、不传参运行

```shell
$ docker run nginx:test
```

容器内会默认运行以下命令，启动主进程。

```shell
nginx -c /etc/nginx/nginx.conf
```

2、传参运行

```shell
$ docker shrun nginx:test -c /etc/nginx/new.conf
```

容器内会默认运行以下命令，启动主进程(/etc/nginx/new.conf:假设容器内已有此文件)

```shell
nginx -c /etc/nginx/new.conf
```

#### ENV

设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

格式：

```docker
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

以下示例设置 NODE\_VERSION = 7.2.0 ， 在后续的指令中可以通过 $NODE\_VERSION 引用：

```docker
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

#### ARG

构建参数，与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

构建命令 docker build 中可以用 --build-arg <参数名>=<值> 来覆盖。

格式：

```docker
ARG <参数名>[=<默认值>]
```

#### VOLUME

定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。

作用：

* 避免重要的数据，因容器重启而丢失，这是非常致命的。
* 避免容器不断变大。

格式：

```docker
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

在启动容器 docker run 的时候，我们可以通过 -v 参数修改挂载点。

#### EXPOSE

仅仅只是声明端口。

作用：

* 帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
* 在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

格式：

```docker
EXPOSE <端口1> [<端口2>...]
```

#### WORKDIR

指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

docker build 构建镜像过程中的，每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

格式：

```docker
WORKDIR <工作目录路径>
```

#### USER

用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

格式：

```docker
USER <用户名>[:<用户组>]
```

#### HEALTHCHECK

用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式：

```docker
HEALTHCHECK [选项] CMD <命令>：设置检查容器健康状况的命令
HEALTHCHECK NONE：如果基础镜像有健康检查指令，使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令> : 这边 CMD 后面跟随的命令使用，可以参考 CMD 的用法。
```

#### ONBUILD

用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。当有新的 Dockerfile 使用了之前构建的镜像 FROM test-build ，这时执行新镜像的 Dockerfile 构建时候，会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式：

```docker
ONBUILD <其它指令>
```

#### LABEL

LABEL 指令用来给镜像添加一些元数据（metadata），以键值对的形式，语法格式如下：

```docker
LABEL <key>=<value> <key>=<value> <key>=<value> ...
```

比如我们可以添加镜像的作者：

```docker
LABEL org.opencontainers.image.authors="runoob"
```
