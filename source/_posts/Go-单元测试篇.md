---
title: Go-单元测试篇(简单)
date: 2021-07-06 10:44:59
tags:
- Go
- 单元测试
categories: 技术
---

>参考文章

[golang test测试使用](https://studygolang.com/articles/2491)

[Go Test 单元测试简明教程](https://geektutu.com/post/quick-go-test.html)

[Golang 单元测试执行 _test.go 中的某个 func 方法](https://learnku.com/articles/33446)

>正文

为什么需要单元测试，因为你可能修改的只是一个方法，但如果整个接口都放进去测试，明显效率不高。

* 创建测试go文件，测试文件以 _test.go 结尾;引入包 import "testing";测试用的参数有且只有一个，在这里是 t *testing.T。

```
package main

import "testing"

func TestAdd(t *testing.T) {
	if ans := Add(1, 2); ans != 3 {
		t.Errorf("1 + 2 expected be 3, but %d got", ans)
	}

	if ans := Add(-10, -20); ans != -30 {
		t.Errorf("-10 + -20 expected be -30, but %d got", ans)
	}
}
```

基准测试(benchmark)的参数是 *testing.B，TestMain 的参数是 *testing.M 类型。

* 运行 go test，该 package 下所有的测试用例都会被执行。

```
$ go test
ok      example 0.009s
```

* go test -v，-v 参数会显示每个用例的测试结果，另外 -cover 参数可以查看覆盖率。

```
$ go test -v
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
=== RUN   TestMul
--- PASS: TestMul (0.00s)
PASS
ok      example 0.007s
```

* 但我们更常用的是运行其中一个，例如要运行上面的TestAdd方法，用`-run`参数指定，该参数支持通配符 *，和部分正则表达式，例如 ^、$。
```
$ go test -run TestAdd -v
=== RUN   TestAdd
--- PASS: TestAdd (0.00s)
PASS
ok      example 0.007s
```
当然，go单元测试还有其他的使用内容，不过这边仅给最简单的。具体可以看上面的链接文档。

> 真实场景注意点

* 报错：flag provided but not defined: -test.timeout

处理方法：注释掉flag.Parse()

理由：从flag.Parse()源码发现，他把go test的test当作一个参数处理，而flag没有对这个参数做处理导致解析失败。[链接](https://my.oschina.net/u/3223370/blog/4272500)

* 最好在测试文件添加初始化，完成数据库之类的默认配置
```
func init()  {
	initConfig()
	initRedis()
}
```