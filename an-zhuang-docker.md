# 安装 Docker

### 安装准备

{% hint style="info" %}
环境准备
{% endhint %}

1、需要一点 Linux 基础

2、Ubuntu(Bionic Beaver) 18.04 / CentOS 7

{% hint style="info" %}
环境查看
{% endhint %}

```shell
# 系统内核是 3.1 以上
master@server:~$ uname -r
4.15.0-162-generic
# 查看系统信息
master@server:~$ cat /etc/os-release
NAME="Ubuntu"
VERSION="18.04.6 LTS (Bionic Beaver)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 18.04.6 LTS"
VERSION_ID="18.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=bionic
UBUNTU_CODENAME=bionic
```

### 官方脚本自动安装（推荐）

使用命令如下：

```shell
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```

也可以使用国内 daocloud 一键安装命令：

```shell
curl -sSL https://get.daocloud.io/docker | sh
```

### 手动安装

详细安装步骤请查看[文档](https://docs.docker.com/engine/install/ubuntu/)

#### 卸载旧版本

Docker 的旧版本被称为 docker，docker.io 或 docker-engine 。如果已安装，请卸载它们：

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

#### 设置存储库

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker 。

```shell
# 1. 更新 apt 并允许 apt 通过 https 使用存储库
$ sudo apt-get update
$ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
    
# 2. 添加Docker官方的GPG密钥
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg

# 3. 设置稳定存储库
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### 安装 Docker Engine

更新索引库，然后安装 docker

{% hint style="info" %}
docker-ce：docker 社区版（推荐）

docker-ee：docker 企业版
{% endhint %}

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

#### 验证安装成功

```
$ docker --version
Docker version 20.10.11, build dea9396
```

### 使用 Shell 脚本安装

Docker 在 [get.docker.com ](https://get.docker.com)和 [test.docker.com](https://test.docker.com) 上提供了方便脚本，用于将快速安装 Docker Engine-Community 的边缘版本和测试版本。脚本的源代码在 docker-install 仓库中。&#x20;

{% hint style="danger" %}
不建议在生产环境中使用这些脚本，在使用它们之前，您应该了解潜在的风险：
{% endhint %}

* 脚本需要运行 root 或具有 sudo 特权。因此，在运行脚本之前，应仔细检查和审核脚本。
* 这些脚本尝试检测 Linux 发行版和版本，并为您配置软件包管理系统。此外，脚本不允许您自定义任何安装参数。从 Docker 的角度或您自己组织的准则和标准的角度来看，这可能导致不支持的配置。
* 这些脚本将安装软件包管理器的所有依赖项和建议，而无需进行确认。这可能会安装大量软件包，具体取决于主机的当前配置。
* 该脚本未提供用于指定要安装哪个版本的 Docker 的选项，而是安装了在 edge 通道中发布的最新版本。
* 如果已使用其他机制将 Docker 安装在主机上，请不要使用便捷脚本。

本示例使用 [get.docker.com ](https://get.docker.com)上的脚本在 Linux 上安装最新版本的Docker Engine-Community。要安装最新的测试版本，请改用 test.docker.com。在下面的每个命令，取代每次出现 get 用 test。

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

如果要使用 Docker 作为非 root 用户，则应考虑使用类似以下方式将用户添加到 docker 组：

```bash
$ sudo usermod -aG docker your-user
```

### 卸载 Docker

先删除安装包，然后删除镜像、容器、配置文件等内容

```shell
$ sudo apt-get purge docker-ce docker-ce-cli containerd.io
$ sudo rm -rf /var/lib/docker
$ sudo rm -rf /var/lib/containerd
```

###

