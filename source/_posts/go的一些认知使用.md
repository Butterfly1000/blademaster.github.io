---
title: go的一些认知使用
date: 2021-02-01 19:40:48
tags:
- Go
- 编程语言
categories: 技术
---

## 本篇观点不一定正确，目前仅供个人使用

> 数组和切片区别

数组是固定的，但切片有扩展性`append`

> 切片和map

其实map可以当做一个类型，可以存储字符串

> map嵌套

```
a := map[string]map[string]string{}
b,ok := a["name"]
if !ok {
    b := make(map[string]string)
    a["name"] = b
    b["xiaoming"] = "go"
    b["xiaodong"] = "why"
}
```

> struct 继承和重组

结构体，对象。可以通过指针关联方法

```
https://blog.csdn.net/z226688/article/details/108963110
```

