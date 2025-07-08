---
title: Godoc 安装与使用
date: 2022-06-10 15:51:14
tags:
- Go
- Godoc
categories: 技术
---
## 简单操作：

首先要确保，go env的环境变量GOBIN配置正确。

使用vim打开.bash_profile文件； 打开以后，如下输入内容：

```
export GOPATH=/Users/douxiaobo/go

export GOBIN=$GOPATH/bin

export PATH=$PATH:$GOBIN
```

然后，到项目根目录下执行`go get -v -u golang.org/x/tools/cmd/godoc`

最后，执行` godoc -http=:6060`,访问浏览器`localhost:6060`,点击**Packages**，然后找到**项目目录**，点击查看。

## 更多

[看这一篇就够了](https://blog.csdn.net/sdujava2011/article/details/118926883)