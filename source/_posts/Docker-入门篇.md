---
title: Docker入门教程
date: 2021-10-02 21:39:41
tags:
- Docker
- 入门
categories: 技术
---

## 安装Docker

### 安装环境

此处在Centos7进行安装，可以使用以下命令查看CentOS版本

```
lsb_release -a
```

在 CentOS 7安装docker要求系统为64位、系统内核版本为 3.10 以上，可以使用以下命令查看

```
uname -r
```

### 用yum源安装

**查看是否已安装docker列表**

```
yum list installed | grep docker
```

**安装docker**

```
yum -y install docker
```

[linux 安装docker](https://www.cnblogs.com/kingsonfu/p/11576797.html)


### 查找docker安装路径

Linux系统下查找安装包所在目录的方法,rpm查找安装包路径

```
Linux rpm 命令用于管理套件
参数说明：
-a 　查询所有套件。
-l 　显示套件的文件列表。
-q 　使用询问模式，当遇到任何问题时，rpm指令会先询问用户

rpm -qa | grep docker

rpm -ql docker-ce-19.03.13-3.el7.x86_64

which
使用指令"which"查看"nginx"的绝对路径，输入如下命令
which nginx

whereis
使用 whereis 命令，能够方便的定位到文件系统中可执行文件的位置
whereis nginx
```

### 查看是否安装成功

```
# docker version 或者# docker info
```

### Docker 需要用户具有 sudo 权限，为了避免每次命令都输入`sudo`，可以把用户加入 Docker 用户组

```
$ sudo usermod -aG docker $USER
```

### Docker 运行命令

```
# systemctl 命令的用法
$ sudo systemctl start docker 
```

## image文件

**Docker 把应用程序及其依赖，打包在 image 文件里面。**

只有通过这个文件，才能生成 Docker 容器。**image 文件可以看作是容器的模板**。Docker 根据 image 文件**生成**容器的实例。同一个 image 文件，可以生成多个同时运行的容器实例。

image 是二进制文件。实际开发中，一个 image 文件往往通过继承另一个 image 文件，加上一些个性化设置而生成。举例来说，你可以在 Ubuntu 的 image 基础上，往里面加入 Apache 服务器，形成你的 image。

```
# 列出本机的所有 image 文件。
$ docker image ls# 删除 image 文件$ docker image rm [imageName]
```

image 文件是通用的，一台机器的 image 文件拷贝到另一台机器，照样可以使用。一般来说，为了节省时间，我们应该尽量使用别人制作好的 image 文件，而不是自己制作。即使要定制，也应该基于别人的 image 文件进行加工，而不是从零开始制作。

为了方便共享，image 文件制作完成后，可以上传到网上的仓库。Docker 的官方仓库 [Docker Hub](https://hub.docker.com/) 是最重要、最常用的 image 仓库。此外，出售自己制作的 image 文件也是可以的。



> 将 image 文件从仓库抓取到本地

```
$ docker image pull library/hello-world
```

上面代码中，`docker image pull`是抓取 image 文件的命令。`library/hello-world`是 image 文件在仓库里面的位置，其中`library`是 image 文件所在的组，`hello-world`是 image 文件的名字。

由于 **Docker 官方提供** 的 image 文件，都放在[`library`](https://hub.docker.com/r/library/)组里面，所以它的是默认组，**可以省略**。

```
$ docker image pull hello-world
```

> 查看 image 文件

```
$ docker image ls
```

> 运行这个 image 文件生成容器实例并运行

```
$ docker container run hello-world
```

注意，`docker container run`命令具有自动抓取 image 文件的功能。如果发现本地没有指定的 image 文件，就会从仓库自动抓取。因此，前面的`docker image pull`命令并不是必需的步骤。

> 手动终止容器运行

```
$ docker container kill [containID]
```

## 容器文件

**image 文件生成的容器实例，本身也是一个文件，称为容器文件。**

```
# 列出本机正在运行的容器
$ docker container ls
或
$ docker ps

# 列出本机所有容器，包括终止运行的容器
$ docker container ls --all
```

终止运行的容器文件，依然会占据硬盘空间，可以使用[`docker container rm`](https://docs.docker.com/engine/reference/commandline/container_rm/)命令删除。

```bash
$ docker container rm [containerID]
```

## Dockerfile 文件

学会使用 image 文件以后，接下来的问题就是，如何可以生成 image 文件？如果你要推广自己的软件，势必要自己制作 image 文件。

这就需要用到 Dockerfile 文件。它是一个文本文件，用来配置 image。Docker 根据 该文件生成二进制的 image 文件。

下面通过一个实例，演示如何编写 Dockerfile 文件。


## 实例：制作自己的 Docker 容器

这部分内容就不做抄录了，附上参考和转载文章地址 [Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html) 和 [Docker 微服务教程](https://www.ruanyifeng.com/blog/2018/02/docker-wordpress-tutorial.html),


## image 仓库的镜像网址
本教程需要从仓库下载 image 文件，但是国内访问 Docker 的官方仓库很慢，还经常断线，所以要把仓库网址改成国内的镜像站。这里推荐使用官方镜像 registry.docker-cn.com 。下面是我的 Debian 系统的默认仓库修改方法，其他系统的修改方法参考官方文档。

打开/etc/default/docker文件（需要sudo权限），在文件的底部加上一行。

DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
然后，重启 Docker 服务。

```
$ sudo service docker restart
```
现在就会自动从镜像仓库下载 image 文件了。


## github镜像

```
https://hub.fastgit.org/
```

**原文链接：**

[Docker 入门教程](http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)