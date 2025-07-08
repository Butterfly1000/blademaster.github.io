---
title: GO time篇
date: 2021-04-26 20:24:38
tags:
- Go
- time
categories: 技术
---

### go Time篇

当前日期 time.Now().Format("2006-01-02")

当前时间 time.Now().Format("2006-01-02 15:04:05")

年月日 year, mon, day := time.Now().UTC().Date()      mon格式为month需要转换

也可以单独获取

加一天 time.Parse("2006-01-02 15:04:05", date).AddDate(0, 0, 1).Format("2006-01-02 15:04:05")

减一天 time.Parse("2006-01-02 15:04:05", date).AddDate(0, 0, -1).Format("2006-01-02 15:04:05")

0 0 1 前面两个零分别代表年，月    date初始日期



[golang包time用法详解](https://blog.csdn.net/wschq/article/details/80114036)

[golang map 获取某个值](https://blog.csdn.net/qq_41004440/article/details/89252653)

https://blog.csdn.net/weixin_39524842/article/details/111890157