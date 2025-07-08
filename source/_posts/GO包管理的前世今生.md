---
title: GO包管理的前世今生
date: 2021-10-08 11:13:59
tags:
- Go
- 包管理
- 编程语言
categories: 技术
---

## 早期的GO包管理(了解)

早期根本没有所谓的包管理，项目依赖一塌糊涂。那么这样会出现什么问题？

> 在没有包管理方案之前，项目依赖于 GOPATH，带来一些不便之处

* 多项目难以管理。试想一下，A 项目和 B 项目 同时依赖于一个 C 项目，但是依赖于 C 项目的不同版本，如何解决，只能通过多 GOPATH 设置。

```
go命令依赖一个重要的环境变量：$GOPATH
GOPATH允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候Windows是分号;，Linux系统是冒号:
[设置Golang的GOPATH](https://blog.csdn.net/chenjh213/article/details/51381024)
```

* 项目的依赖只能够手动 go get 下载到 GOPATH 中，如果换一台服务器开发项目，光是包依赖都得弄半天。

* 如果是本地想搭建对应的项目，还无法知道之前包的版本，只能通过当初项目创建时间进行`git tag`切换对应版本。

```
git tag #展示全部tag
git checkout  release-v1.0.0 #切换到tag分支

无关-记录
git tag release-v1.0.1 #打上tag
git push --tag #推送
```

> 一些解决方案

* godep，glide，go verdor 这些都是曾经出现过的包管理解决方案。

* 将所有依赖包都上传到一个公共的线上文件夹(类似vendor)，然后从上面直接拷贝。

## 现在的GO包管理(go版本 >= 1.11)

Go.mod是 **Golang1.11版本** 新引入的官方包管理工具用于解决之前没有地方记录依赖包具体版本的问题。

*前导*

Go.mod其实就是一个Modules官方定义为：

Modules是相关Go包的集合，是源代码交换和版本控制的单元。go命令直接支持使用Modules，包括记录和解析对其他模块的依赖性。Modules替换旧的基于GOPATH的方法，来指定使用哪些源文件。

Modules和传统的GOPATH不同，不需要包含例如src，bin这样的子目录，一个源代码目录甚至是空目录都可以作为Modules，只要其中包含有go.mod文件。



**go mod的使用：**

一个很关键的地方在于环境变量GO111MODULE的设置,GO111MODULE有三个值：off, on和auto（默认值）

```
1.GO111MODULE=off，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或GOPATH模式来查找。

2.GO111MODULE=on，go命令行会使用modules，而一点也不会去GOPATH目录下查找。

3.GO111MODULE=auto，默认值，go命令行将会根据当前目录来决定是否启用module功能。
这种情况下可以分为两种情形：
当前目录在GOPATH/src之外且该目录包含go.mod文件
当前文件在包含go.mod文件的目录下面。
Go mod 的命令使用help查看
```

如何在项目中使用？

```
#cd xxx //进入项目
#go mod init  //初始化项目
#ps
在当前目录下生成一个go.mod文件，通过这个文件管理包
```

**注意**：除了go.mod之外，go命令还维护一个名为go.sum的文件，其中包含特定模块版本内容的预期加密哈希。go命令使用go.sum文件确保这些模块的未来下载检索与第一次下载相同的位，以确保项目所依赖的模块不会出现意外更改，无论是出于恶意、意外还是其他原因。 **go.mod和go.sum都应检入版本控制** 。go.sum 不需要手工维护，所以可以不用太关注。

```
项目子目录里是不需要init的，所有的子目录里的依赖都会组织在根目录的go.mod文件里
```



**当依赖第三方包时:**

```
- 最初是go get 拉取，而使用go mod，直接go run 或go build 即可！
例如：go get utils/iGong@v1.3.5
不过，有了go mod后，只要项目 运行 或 创建二进制文件 就会自动拉取依赖。

- go 会自动查找代码中的包，下载依赖包，并且把具体的依赖关系和版本写入到go.mod和go.sum文件中。

- 依赖的第三方包被下载到了$GOPATH/pkg/mod路径下。
```



**依赖包的版本控制：**

GOPATH/pkg/mod里可以保存相同包的**不同版本**。
go.mod中可指定下载包的版本：

```
如果在go.mod中没有指定，go命令会自动下载最新版本。
如果在go.mod用require语句指定包和版本 ，go命令会根据指定的路径和版本下载包。
当然，对于某些缺失的包，可以用replace指定代替包。
指定版本时可以用latest，这样它会自动下载指定包的最新版本,项目也可以放在$GOPATH/src下。
```

```
module app #项目名

go 1.13

require (
	gorm.io/driver/mysql v1.1.2 // indirect
	gorm.io/gorm v1.21.15 // indirect
)

replace github.com/qiniu/bytes => gitlab.airdroid.com/zhuoh/bytes v0.0.0-20191012100200-92558a444c07
```

## 参考原文

[原文链接](https://blog.csdn.net/jkwanga/article/details/106288345)
[原文链接](https://www.jianshu.com/p/d04b36fbdcd6#comments)