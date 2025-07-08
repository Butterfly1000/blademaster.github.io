---
title: Jenkins
date: 2022-03-21 19:03:13
tags:
- Jenkins
- CI/CD
categories: 技术
---
[超级详细！开发集成工具Jenkins使用教程](https://mp.weixin.qq.com/s/mPkhy7YvrYO_HnfL-rho3w)

[docker | jenkins 自动化CI/CD，后端躺着把运维的钱挣了！(下)](https://juejin.cn/post/7064389514470359053)

`docker + jenkins:`使用jenkins监听git仓库的变化，一旦发生变化就**自动拉取git仓库代码，构建docker镜像，然后自动部署，运行容器**。只要push了代码，则新一版的项目就会由jenkins自动部署到指定服务器。

`账号:浅蝶果果 密码:qqx`

> 创建仓库

访问`https://hub.docker.com/`

点击`菜单栏 Repositories`  -> `Create Repository`

> 安装Jenkins

[官方连接](https://www.jenkins.io/zh/doc/tutorials/build-a-python-app-with-pyinstaller/)

```
docker run \
  -d \
  --rm \
  -u root \
  -p 9026:8080 \
  -v jenkins-data:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  jenkinsci/blueocean
```

因为我8080已应用别的内容，就修改成9026。

请求：localhost:8080，会跳转到页面让你填写初始密码

**获取初始密码**

进入`docker exec -it a3c5c0caecfb  bash`

`cat /var/jenkins_home/secrets/initialAdminPassword`  //这个页面会有提示地址的

`b950574b0c2c43b78e03ee0a8a2aecc3`

**选择推荐安装**

**创建第一个管理员用户**

```
king
123456aa
123456aa
王
2496978350@qq.com
```

**保存，然后进入后台咯**

## Docker创建私有仓库(学习)

[docker for mac 创建私有仓库](https://www.cnblogs.com/huangenai/p/10012672.html)

[Mac 环境部署Docker私有仓库](https://blog.csdn.net/weixin_33724659/article/details/93810200)

拉取镜像

```
docker pull registry
```

运行registry

```
docker run -d -p 5000:5000 -v /Users/huangyanyu/WWW/docker/registry:/var/lib/registry registry
```

* **-i**: 交互式操作。
* **-t**: 终端。
* **-v:** 将宿主机目录挂载到容器里, 或者说把镜像路径映射到本机。`-v /宿主机目录:/容器目录`
* **-d** 指定容器的运行模式，后台运行，默认不会进入容器

- **-P (大写):**是容器内部端口**随机**映射到主机的端口。
- **-p(小写) :** 是**容器内部端口**绑定到**指定**的**主机端口**。-p **指定**的**主机端口**  **容器内部端口**
- **docker attach**
- **docker exec**：推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

`fe1a0525e19102c5208ab141cc96dfebe51f5445dc71c2ee181d32fd05995f36`

```
//查看运行容器
docker ps

//进入容器  fe1a0525e191是容器id 在上一步骤中获得 
sudo docker attach fe1a0525e191 （失败）
sudo docker exec -it fe1a0525e191 /bin/bash (失败)
```



本地仓库非安全配置 user/<username>/.docker/daemon.json

```
cat .docker/daemon.json 
{
  "insecure-registries" : [
    "127.0.0.1:5000"
  ],
  "debug" : true,
  "experimental" : true,
  "registry-mirrors" : [
    "https://8q2dp9p9.mirror.aliyuncs.com"
  ]
}
```



查看仓库中的镜像

```
curl -XGET http://127.0.0.1:5000/v2/_catalog
```