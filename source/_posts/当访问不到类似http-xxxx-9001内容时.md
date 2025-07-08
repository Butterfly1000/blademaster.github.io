---
title: 当访问不到类似http-xxxx-9001内容时
date: 2021-04-25 15:47:35
tags:
- 网络
- 问题排查
categories: 技术
---
1. 查看是否使用阿里云，确认安全组是否开启端口
2. 查看防火墙端口是否开启
`https://www.cnblogs.com/zxg-blog/p/9835263.html`
3. 查看netstat -lnpt | grep "端口" 查看端口是否允许
4. 查看是否是0.0.0.0，而非127.0.0.1
5. 通过telnet 0.0.0.0 9001
6. 通过curl 0.0.0.0 9001
7. 通过ping 0.0.0.0 9001
8. 端口是否被占用