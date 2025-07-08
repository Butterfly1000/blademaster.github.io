---
title: ICE协议下NAT穿越的实现（STUN&TURN）
date: 2022-03-20 10:55:20
tags:
- 网络
- NAT
- P2P
categories: 技术
---
这一篇感觉写得真好。

NAT是什么？看上一篇《P2P通信原理与实现》

> NAT的副作用以及解决方案

国内移动无线网络运营商在链路上一段时间内没有数据通讯后, 会淘汰NAT表中的对应项, 造成链路中断。

##### 这是NAT带来的第一个副作用：NAT超时:

而国内的运营商一般NAT超时的时间为5分钟，所以通常我们TCP长连接的心跳设置的时间间隔为3-5分钟。

##### 而第二个副作用就是：我们这边文章要提到的NAT墙。

NAT会有一个机制，所有外界对内网的请求，到达NAT的时候，都会被NAT所丢弃，这样如果我们处于一个NAT设备后面，我们将无法得到任何外界的数据。

但是这种机制有一个解决方案：就是如果我们A主动往B发送一条信息，这样A就在自己的NAT上打了一个B的洞。这样A的这条消息到达B的NAT的时候，虽然被丢掉了，但是如果B这个时候在给A发信息，到达A的NAT的时候，就可以从A之前打的那个洞中，发送给到A手上了。

简单来讲，就是**如果A和B要进行通信，那么得事先A发一条信息给B，B发一条信息给A。这样提前在各自的NAT上打了对方的洞，这样下一次A和B之间就可以进行通信了。**

>  四种NAT类型，看上一篇《P2P通信原理与实现》后就完全能理解下面这段话了。

RFC3489 中将 NAT 的实现分为四大类：

1. Full Cone NAT                          （完全锥形 NAT）
2. Restricted Cone NAT                 （限制锥形 NAT ，可以理解为 IP 限制，Port不限制）
3. Port Restricted Cone NAT          （端口限制锥形 NAT，IP+Port 限制）
4. Symmetric NAT                        （对称 NAT）

其中完全最上层的完全锥形NAT的穿透性最好，而最下层的对称形NAT的安全性最高。

简单来讲讲这4种类型的NAT代表什么：

- 如果一个NAT是Full Cone NAT，那么无论什么IP地址访问，都不会被NAT墙掉（这种基本很少）。
- Restricted Cone NAT，仅仅是经过打洞的IP能穿越NAT，但是不限于Port。
- Port Restricted Cone NAT，仅仅是经过打洞的IP+端口号能穿越NAT。
- Symmetric NAT 这种也是仅仅是经过打洞的IP+端口号能穿越NAT，但是它有一个最大的和Cone类型的NAT的区别，它对外的公网Port是不停的变化的：
  比如A是一个对称NAT，那么A给B发信息，经过NAT映射到一个Port:10000，A给C发信息，经过NAT映射到一个Port:10001，这样会导致一个问题，我们服务器根本无法协调进行NAT打洞。

至于为什么无法协调打洞，下面我们会从STUN和TURN的工作原理来讲。



> #### STUN和TURN的实现

##### 1.STUN Server主要做了两件事：

- 接受客户端的请求，并且把客户端的公网IP、Port封装到ICE Candidate中。

* 通过一个复杂的机制，**得到客户端的NAT类型**。

完成了这些STUN Server就会这些基本信息发送回客户端，然后根据NAT类型，来判断是否需要TURN服务器协调进行下一步工作。

我们来讲讲这两步具体做了什么吧：
第一件事就不用说了，其实就是得到客户端的请求，把源IP和Port拿到，添加到ICE Candidate中。

###### 来讲讲第二件事，STUN是如何判断NAT的类型的：

假设B是客户端，C是STUN服务器，C有两个IP分别为IP1和IP2（至于为什么要两个IP，接着往下看）：

###### STEP1.判断客户端是否在NAT后：

B向C的**IP1的port1端口**发送一个UDP 包。C收到这个包后，会把它收到包的**源**IP和port写到UDP包中，然后把此包通过IP1和port1发还给B。

这边还给B包中的IP和port也就是B的NAT的外网 IP和port，也就是说你在STEP1中就得到了B的NAT的外网IP。

熟悉NAT工作原理的朋友可以知道，C返回给B的这个UDP包B一定收到**(因为由B主动先发送给C，B的NAT已经记录下了C的IP和端口)**。如果在你的应用中，向一个STUN服务器发送数据包后，你没有收到STUN的任何回应包，那只有两种可能：1、STUN服务器不存在，或者你弄错了port。2、你的NAT拒绝一切UDP包从外部向内部通过。

当B收到此UDP后，把此UDP中的IP和自己的IP做比较，如果是一样的，就说明自己是在**公网**，下步NAT将去探测防火墙类型，这边不做介绍。如果不一样，说明有NAT的存在，系统进行STEP2的操作。

###### STEP2.判断是否处于Full Cone Nat下：

B向C的IP1发送一个UDP包，请求C通过另外一个IP2和PORT（不同与SETP1的IP1）向B返回一个UDP数据包（现在知道为什么C要有两个IP了吧，虽然还不理解为什么，呵呵）。

我们来分析一下，如果B收到了这个数据包，那说明什么？说明NAT来着不拒，不对数据包进行任何过滤，这也就是STUN标准中的full cone NAT。遗憾的是，Full Cone Nat太少了，这也意味着你能收到这个数据包的可能性不大。如果没收到，那么系统进行STEP3的操作。

###### STEP3.判断是否处于对称NAT下：

B向C的IP2的port2发送一个数据包，C收到数据包后，把它收到包的源IP和port写到UDP包中，然后通过自己的IP2和port2把此包发还给B。

和step1一样，B肯定能收到这个回应UDP包。此包中的port是我们最关心的数据，下面我们来分析：
如果这个port和step1中的port一样，那么可以肯定这个NAT是个CONE NAT，否则是对称NAT。道理很简单：根据对称NAT的规则，当目的地址的IP和port有任何一个改变，那么NAT都会重新分配一个port使用，而在step3中，和step1对应，我们改变了IP和port。因此，如果是对称NAT,那这两个port肯定是不同的。

如果在你的应用中，到此步的时候PORT是不同的，那么这个它就是处在一个对称NAT下了。如果相同，那么只剩下了restrict cone 和port restrict cone。系统用step4探测是是那一种。

###### STEP4.判断是处于Restrict Cone NAT还是Port Restrict NAT之下：

B向C的IP2的一个端口PD发送一个数据请求包，要求C用IP2和不同于PD的port返回一个数据包给B。

我们来分析结果：如果B收到了，那也就意味着只要IP相同，即使port不同，NAT也允许UDP包通过。显然这是Restrict Cone NAT。如果没收到，没别的好说，Port Restrict NAT.

###### 到这里STUN Server一共通过这4步，判断出客户端处于什么类型的NAT下，

###### 然后去做后续的处理：这4步都会返回给客户端它的公网IP、Port和NAT类型，除此之外：

* 如果B处于公网或者Full Cone Nat下，STUN不做其他的了，因为其他客户端可以直接和A进行通信。
* 如果B处于Restrict Cone或者Port Restrict NAT下，STUN还会协调TURN进行NAT打洞。

```
B请求STUN服务器，
STUN服务器返回ICE NAT类型给B;
STUN服务器协调打洞TURN服务器;
TURN服务器命令A给B打洞，B给A打洞；
B和A通信
```

* 如果A处于对称NAT下，那么点对点连接下，NAT是无法进行打洞的。所以为了通信，只能采取最后的手段了，就是转成C/S架构了，STUN会协调TURN进行消息转发（中继）。



##### 2.TURN Server也主要做了两件事：

- ###### 为NAT打洞：

如果A和B要互相通信，那么TURN Server，会命令A和B互相发一条信息，这样各自的NAT就留下了对方的洞，下次他们就可以之间进行通信了。

- ###### 为对称NAT提供消息转发：

当A或者B其中一方是对称NAT时，那么给这一方发信息，就只能通过TURN Server来转发了。

#### 最后补充一下，为什么对称NAT无法打洞：

假如A、B进行通信，而B处于对称NAT之下，那么A与B通信，STUN拿到A，B的公网地址和端口号都为10000，然后去协调TURN打洞，那么TURN去命令A发信息给B，则A就在NAT打了个B的洞，但是这个B的洞是端口号为10000的洞，但是下次B如果给A发信息，因为B是对称NAT，它给每个新的IP发送信息时，都重新对应一个公网端口，所以给A发送请求可能是公网10001端口，但是A只有B的10000端口被打洞过，所以B的请求就被丢弃了。
显然Server是无法协调客户端打洞的，因为协调客户端打得洞仅仅是上次对端为Server发送端口的洞，并不适用于另一个请求。

##### 最后的最后再补充一点，就是NAT打的洞也是具有时效性的，如果NAT超时了，那么还是需要重新打洞的。



**文章地址：**

作者：涂耀辉
链接：https://www.jianshu.com/p/84e8c78ca61d