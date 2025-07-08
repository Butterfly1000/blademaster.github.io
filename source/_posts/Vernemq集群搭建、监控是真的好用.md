---
title: Vernemq集群搭建、监控是真的好用
date: 2022-03-24 15:55:29
tags:
- VerneMQ
- MQTT
- 消息队列
- 集群
categories: 技术
---
## 安装

**下载**

```
wget https://github.com/vernemq/vernemq/releases/download/1.12.3/vernemq-1.12.3.centos7.x86_64.rpm
```

**yum安装**

```
$ yum -y install ./vernemq-1.12.3.centos7.x86_64.rpm
```

安装完成后会自动生成配置文件**/etc/vernemq/vernemq.conf**，修改配置文件，仅修改而不是全部替换

```
# 是否开启匿名访问，开启匿名访问后将不验证用户名及密码
allow_anonymous = off
# 若配置集群需要开发内部通讯
allow_register_during_netsplit = on
allow_publish_during_netsplit = on
allow_subscribe_during_netsplit = on
allow_unsubscribe_during_netsplit = on
# 配置MQTT可连接地址以及端口0.0.0.0及代表任意IP可连接
listener.tcp.default = 0.0.0.0:1883
#服务器内网ip，集群通讯端口
#这里值得注意的是我测试发现如果配置外网的IP集群是无法正常通信的
listener.vmq.clustering = 172.xx.xxx.219:44053
#VerneMQweb监控页面以及端口
listener.http.default= 0.0.0.0:8085
#集群中节点名称，避免重复
nodename = vmqNode1@172.xx.xxx.219
```

如果未开启匿名访问，我们就需要为**Vernemq**添加相关的账号密码

```
vmq-passwd -c /etc/vernemq/vmq.passwd admin #回车键 输入密码并验证 19491001
```

接下来就是配置topic的读写权限，默认状态下允许所有用户对所有的topic可读写。

但安全和规范起见，建议大家规定各任务之间不同的topic并对权限加以控制。修改配置文件：/etc/vernemq/vmq.acl

```
#添加如下内容
topic read $SYS/## ACL for user 'admin'user admintopic test/#

这里的#代表统配，例如test/#代表test及以下所有topic
```

启动vernemq

```
$ systemctl start vernemq
```

若为集群则使用如下命令加入任意集群节点:

```
$ vmq-admin cluster join discovery-node=vmqNode1@172.xx.xxx.219   #节点名称
```

查看节点状态：`vmq-admin cluster show`，或者通过web监控页面查看集群以及节点状态，访问如下地址：http(s)://ip:8085/status，这里的端口为上方配置文件中配置的listener.http.default信息中的端口。

## 运行

**启动(除了上面的之外):**

```
vernemq start|stop|restart(成功) 或者 /etc/init.d/vernemq start|stop|restart(应该还要配置，参考)
```

**版本升级**

1. 下载升级版本的二进制文件
2. 执行 rpm -Uvh vernemq-[version].centos7.x86_64.rpm
3. 重启vernemq 服务
4. 注意：要看下旧的进程是否存在，存在的话，需要kill 进程，重新启动

## 实践

> MQTTBox请求(订阅/发布)

通过下面连接添加谷歌商店拓展应用**MQTTBox**

`https://www.hivemq.com/blog/mqtt-toolbox-mqttbox/`

**MQTTBox**使用参考文章：

[Mac 下MQTT免费测试工具MQTTBox](https://blog.csdn.net/qq_20042935/article/details/101195038)

> node.js请求(订阅/发布)

```
报错：npm WARN saveError ENOENT: no such file or directory, open 'package.json'
处理方式：npm install -y

报错：Error: Cannot find module 'mqtt'
处理方式：npm install mqtt

完整操作
# mkdir -p node
# cd node
# npm install -y
# npm install mqtt
# vi app.js
# vi publish.js
# node app.js
# node publish.js

结果
start
connected
hello
good man
```

**app.js**

```
var mqtt = require('mqtt');
var client  = mqtt.connect('mqtt://47.xx.xxx.104:1883',{username:'admin',password:'19491001'});

client.on('connect', function () {
    console.log("connected");
    client.subscribe('hello');
    setTimeout(function(){
        client.publish('presence', 'Hello mqtt')
    },10)
});

client.on('message', function (topic, message) {
    // message is Buffer
    console.log(topic);
    console.log(message.toString());
    client.end()
});

console.log("start");
```

**publish.js**

```
var mqtt = require('mqtt');
var client  = mqtt.connect('mqtt://47.xx.xxx.104:1883',{username:'admin',password:'19491001'});

setInterval(function (){
     client.publish('hello','good man',{qos:1,retain:true})
},2000);
```

**注意：** 如果`allow_anonymous = on`，那么只需要`mqtt.connect('mqtt://47.xx.xxx.104:1883')`

## 参考文章

[mosquitto查看订阅记录_MQTT还在使用mosquitto？Vernemq集群搭建、监控是真的好用](https://blog.csdn.net/weixin_39767645/article/details/111671642)

[webrtc 的 signal 服务器 VerneMQ 的权限校验](https://kebingzao.com/2018/06/03/vernemq-verify/)