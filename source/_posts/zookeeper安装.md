---
title: zookeeper安装
date: 2022-03-21 11:26:14
tags: 网络
---
### 安装

[**【ZooKeeper Notes 2】ZooKeeper快速搭建**](https://blog.51cto.com/nileader/795230)

[**zookeeper-3.5.5安装报错：找不到或无法加载主类 org.apache.zookeeper.server.quorum.QuorumPeerMain-新版本zookeeper易犯错误**](https://blog.csdn.net/jiangxiulilinux/article/details/96433560)

#### 单机部署

1. 安装jdk
2. 地址：`https://mirror.bjtu.edu.cn/reverse/apache-archive/zookeeper/zookeeper-3.7.0/`
3. 拉取`wget https://mirror.bjtu.edu.cn/reverse/apache-archive/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz --no-check-certificate`
4. 解压 `tar -zxvf apache-zookeeper-3.7.0-bin.tar.gz`
5. 修改名称`mv apache-zookeeper-3.7.0-bin zookeeper-3.7.0 `
6. 移动到你想的位置 `mv zookeeper-3.7.0/ /usr/local/`
7. 复制 `cp /usr/local/zookeeper-3.7.0/conf/zoo_sample.cfg  /usr/local/zookeeper-3.7.0/conf/zoo.cfg`
8. vi zoo.cfg，修改 `dataDir=/usr/local/zookeeper-3.7.0/data`
9. 创建数据目录：`mkdir /usr/local/zookeeper-3.7.0/data`
10. 启动zookeeper：`/usr/local/zookeeper-3.7.0/bin/zkServer.sh start`
11. 检测是否成功启动:`/usr/local/zookeeper-3.7.0/bin/zkCli.sh` 或`echo stat|nc localhost 2181`
12. 记得要开启2181端口,以及修改zoo.cfg   `admin.serverPort=9051`,因为默认会用8080，可以会占用，导致启动却无进程。
13. 查看进程`ps -ef | grep zookeeper`


## 简单使用

**zoo.cfg** 五大基础参数配置，其中 `tickTime` 就是基础时间。 `initLimit` 就是初始化的最大值， `syncLimit` 就是 异步交互的最大值。 `dataDir` 就是保存数据和快照的目录， `clientPort` 就是端口

通过 `bin/zkCli.sh -server 127.0.0.1:2181` 可以连接这台zk的 cli 服务。

```text
[zk: 127.0.0.1:2181(CONNECTED) 1] ls /
[gopush-cluster-message, gopush-cluster-comet, zookeeper]
```

