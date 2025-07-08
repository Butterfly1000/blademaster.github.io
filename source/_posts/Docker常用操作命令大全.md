---
title: Docker常用操作命令大全
date: 2021-08-17 14:30:37
tags:
- Docker
- 命令行
categories: 技术
---
## 环境

Linux  Centos7

## docker进程相关命令

- `systemctl start docker` #启动docker服务
- `systemctl stop docker` #停止docker服务
- `systemctl restart docker`#重启docker服务
- `systemctl status docker` #查看docker服务状态
- `systemctl enable docker`#设置开机启动服务



## docker容器管理命令

`docker run -d --name {your_name} {image_name}` # 运行容器

> 参数说明：
>
> -i：以交互模式运行容器，通常与 -t 同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。
>
> -t：为容器重新分配一个伪输入终端，通常与 -i 同时使用。
>
> -d：以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec 进入容器。退出后，容器不会关闭。
>
> -it 创建的容器一般称为交互式容器；
>
> -id 创建的容器一般称为守护式容器。
>
> --name：为创建的容器命名。
>
> -p：指定容器的端口，格式为：主机(宿主)端口:容器端口 
>
> -P(大写): 随机端口映射，容器内部端口**随机**映射到主机的端口
>
> --volume , -v: 绑定一个卷,格式:宿主机目录/文件:容器内目录/文件
>
> ​                       目录必须是绝对路径,如果目录不存在，会自动创建,可以挂载多个数据卷。

```
例子：
docker run -p 80:80 -v /data:/data -d nginx:latest

绑定容器的 8080 端口，并将其映射到本地主机 127.0.0.1 的 80 端口上。
$ docker run -p 127.0.0.1:80:8080/tcp ubuntu bash
```

- `docker ps` # 查看正在运行的容器
- `docker ps -s -a` #查看当前所有容器
- `docker stop {容器ID}` #停止容器
- `docker restart {容器ID}` #重启容器
- `docker kill {容器ID}` #杀死容器
- `docker rm -f {容器ID}` #删除已经停止的容器(这个会彻底释放内存)

**如果容器是运行状态则删除失败，需要停止容器才能删除。**



## docker镜像管理命令

- `docker images` #查看当前机器的所有镜像

  >参数：
  >
  >-a/--all 列出所有镜像
  >
  >-q/--quiet 只显示镜像的id

- `docker images –q` # 查看所用镜像的id

- `docker search {镜像名称}` #搜索镜像，网络中查找需要的镜像

  `$docker search redis`

  **查找stars大于3000的镜像**：`docker search mysql --filter=stars=3000`

- `docker pull {镜像名称}[:tag]` #从Docker仓库拉取镜像，名称:版本号

​     `$docker pull mysql` 或`$docker pull mysql:5.7` 

- `docker push {镜像名称}` #推送镜像

- `docker rmi [-f] {镜像名称/镜像id}` #删除本地机器的镜像

- `docker rmi [-f]  {镜像id} {镜像id} {镜像id}` #删除多个本地机器的镜像

- `docker rmi [-f] docker images -q` #删除所有本地镜像

- `docker tag 镜像名称:tag 镜像名称:tag` #为一个镜像打tag

  > 将镜像ubuntu:15.10标记为 runoob/ubuntu:v3 镜像

  `docker tag ubuntu:15.10 runoob/ubuntu:v3`

  **查看:** `docker images runoob/ubuntu:v3`

- `docker save {image_name} > {new_image_name}.tar` #镜像打包成一个tar包

- `docker load < {image_name}.tar` #解压一个镜像tar包



## 查看日志信息

- `docker logs -f {容器ID}`  #查看容器日志
- `docker info` #查看docker服务的信息
- `docker inspect {容器ID}` # 获取镜像的元信息，详细信息



## 查看进程

`$ docker top f42ae22e4b72`



## 与容器交互的命令

### 进入正在运行的容器

```
docker exec -it 容器ID或者容器名 /bin/bash
```

> exec的意思是在容器中运行⼀个命令。/bin/bash是固有写法，作用是因为docker后台必须运行一个进程，否则容器就会退出，在这里表示启动容器后启动 bash。
>
> -d :分离模式: 在后台运行
>
> -i :即使没有附加也保持STDIN 打开
>
> -t :分配一个伪终端



> 退出容器

`exit` #退出也关闭容器;

`Ctrl+P+Q` #退出不关闭容器



**拷贝文件**

`docker cp 主机文件路径 容器ID或容器名:容器路径` #宿主机文件拷贝到容器中

`docker cp 容器ID或容器名:容器路径 主机文件路径` #容器文件拷贝到宿主机中



**原文链接：**

[docker命令记不住？docker常用操作命令大全](https://juejin.cn/post/6993582707582173198)

[Docker常用命令](https://juejin.cn/post/6996126578048499743)

[Docker 命令大全](https://www.runoob.com/docker/docker-command-manual.html)
