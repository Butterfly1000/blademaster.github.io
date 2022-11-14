---
title: 'Go-解决go的fatal error: concurrent map writes map非并发安全'
date: 2021-07-06 11:13:18
tags: GO
---

>普及概念：

#### 普及1

[Golang sync.WaitGroup的用法](https://studygolang.com/articles/12972?fr=sidebar)

```
package main

import (
    "fmt"
    "time"
)

func main(){
    for i := 0; i < 100 ; i++{
        go fmt.Println(i) //并行
    }
    time.Sleep(time.Second)
}

```
上述方法，其实就是通过sleep等待go并行线程都完成再往下走，但是这个也是有问题的因为sleep的时间无法精确。

* 可以考虑使用管道来完成上述操作：

```
func main() {
    c := make(chan bool, 100)
    for i := 0; i < 100; i++ {
        go func(i int) {
            fmt.Println(i)
            c <- true
        }(i)
    }

    for i := 0; i < 100; i++ {
        <-c
    }
}
```
首先可以肯定的是使用管道是能达到我们的目的的，而且不但能达到目的，还能十分完美的达到目的。



* 但go语言中有一个其他的工具`sync.WaitGroup` 能更加方便的帮助我们达到这个目的。

`WaitGroup` 对象内部有一个计数器，最初从0开始，它有三个方法：`Add(), Done(), Wait()` 用来控制计数器的数量。`Add(n)` 把计数器设置为`n` ，`Done()` 每次把计数器`-1` ，`wait()` 会阻塞代码的运行，直到计数器地值减为0。

```
func main() {
    wg := sync.WaitGroup{}
    wg.Add(100) //计算器设置为100
    for i := 0; i < 100; i++ {
        go func(i int) {
            fmt.Println(i)
            wg.Done() //减1
        }(i)
    }
    wg.Wait() //等待100减到0，取消阻塞，往下执行
}
```

但是要注意：

1.我们不能使用`Add()` 给`wg` 设置一个负值

2.WaitGroup对象不是一个引用类型

WaitGroup对象不是一个引用类型，在通过函数传值的时候需要使用地址：

```
func main() {
    wg := sync.WaitGroup{}
    wg.Add(100)
    for i := 0; i < 100; i++ {
        go f(i, &wg)
    }
    wg.Wait()
}

// 一定要通过指针传值，不然进程会进入死锁状态
func f(i int, wg *sync.WaitGroup) { 
    fmt.Println(i)
    wg.Done()
}
```

#### 普及2

[Go语言defer (延迟执行语句)](http://c.biancheng.net/view/61.html)

Go语言的 defer 语句会将其后面跟随的语句进行延迟处理，在 defer 归属的函数即将返回时，将延迟处理的语句按 defer 的**逆序**进行执行，也就是说，**先**被 defer 的语句**最后**被执行，**最后**被 defer 的语句，**最先**被执行。

```
好处及用处就不用说了。
这边讲关于sync.WaitGroup的实际应用。
defer wg.Done()  //可以将这条语句放在函数中，不用担心句末遗忘，或者后续添加内容遗忘移动位置
```

#### 普及3

[golang中sync.RWMutex和sync.Mutex区别](https://www.cnblogs.com/setevn/p/8977922.html)

golang中sync包实现了两种锁**Mutex （互斥锁）**和**RWMutex（读写锁）**，其中RWMutex是基于Mutex实现的，只读锁的实现使用类似引用计数器的功能．

其中**Mutex**为互斥锁，Lock()加锁，Unlock()解锁，使用Lock()加锁后，便不能再次对其进行加锁，直到利用Unlock()解锁对其解锁后，才能再次加锁．适用于**读写不确定**场景，即读写次数没有明显的区别，并且只允许只有一个读或者写的场景，所以该锁叶叫做**全局锁**。

**RWMutex**是一个读写锁，该锁可以加多个读锁或者一个写锁，其经常用于读次数远远多于写次数的场景．

**[RWMutex的使用主要事项](https://blog.csdn.net/u010230794/article/details/78554370)**

- **1、读锁的时候无需等待读锁的结束**
- **2、读锁的时候要等待写锁的结束**
- **3、写锁的时候要等待读锁的结束**
- **4、写锁的时候要等待写锁的结束**


> 正题

#### 问题代码
```
		var wg sync.WaitGroup
		sortUrl := make(map[int]string) //map
		for index, url := range matchStr {
			wg.Add(1)
			go func(url string, i int) {
				defer wg.Done()
				screenChangeUrl, err := ChangeImageToOurService(url, accountId, "Claw-GooglePlay-AppInfo-Screenshot-", i, AMS_STORE_PREFIX)
				if err == nil {
					sortUrl[i] = screenChangeUrl //问题
				}
			}(url, index)
		}
		wg.Wait()
```

#### 为什么会报"fatal error: concurrent map writes map"？

因为map不是并发安全的 , 当有多个并发的groutine读写同一个map时，会出现panic错误。

#### 解决方案

在继续使用Map类型的情况下，常规解决方案一般分为两种。

* 并发的groutine定义多个map，进行读写。这样就不存在同时读写一个map。这种的试用场景就是，

```
a := make(map[int]string)
b := make(map[int]string)
go func(){
   a[1] = 5
}

go func(){
    b[1] =5
}
```



* 但更多是通过锁(适合上面代码)

```
    var wg sync.WaitGroup
    locker := new(sync.Mutex) //互斥锁，这边也可以根据场景考虑用写锁
    sortUrl := make(map[int]string)
    for index, url := range matchStr {
        wg.Add(1)
        go func(url string, i int) {
            locker.Lock() //锁定
            defer wg.Done()
            defer locker.Unlock() //解锁
            screenChangeUrl, err := ChangeImageToOurService(url, accountId, "Claw-GooglePlay-AppInfo-Screenshot-", i, AMS_STORE_PREFIX)
            if err == nil {
                sortUrl[i] = screenChangeUrl
            }
        }(url, index)
    }
```
这样就可以有效解决报错问题。但是会无形造成串行，因为都在等待解锁。

#### 使用并行的解决方案

* 所以，我们想了两种方案，一种弃用map类型(这个该文章不描述)，另一种使用sync.Map(版本支持)。

```
因为map本就是不适合并行的，所以go官方推出了sync.Map。
通过测试，
		var wg sync.WaitGroup
		var t2 = time.Now()
		var sortUrl sync.Map //新增
		start := time.Now()
		for index, url := range matchStr {
			wg.Add(1)
			go func(url string, i int) {
				defer wg.Done()
				urlString := "url" + "," + fmt.Sprintf("%d",i)
				if i == 2 {
					time.Sleep(3 * time.Second)
				}
				if i == 3 {
					time.Sleep(8 * time.Second)
				}
				sortUrl.Store(i, urlString) //新增
			}(url, index)
		}
		wg.Wait()
		elapsed := time.Since(start)
log.Debug(elapsed)
```
结果:elapsed为8.005294657s，证明可以并行。

* 但是sync.Map和Map的使用却是有很大区别，下面介绍sync.Map的使用。
```
var sortUrl sync.Map //定义
sortUrl.Store(i, urlString) //使用i为key,urlString为value

// 遍历所有sync.Map中的键值对
sortUrl.Range(func(k, v interface{}) bool {
  key := k.(int) //注意转换成int类型
  keys = append(keys, key)
  return true
})

for k := range keys {
  key,_ := sortUrl.Load(k) //获取值，返回value interface{}, ok bool
  keyValue := key.(string) //转成string类型
  uploadSuccessUrl = append(uploadSuccessUrl, keyValue)
}
```

更多操作可参考文档:[Go语言sync.Map (在并发环境中使用的map)](http://c.biancheng.net/view/34.html)

#### 附录介绍一些需要的类型转换文档:
[Golang 中整数转字符串的方法](https://www.jb51.net/article/142447.htm)
[golang-interface转string](https://blog.csdn.net/xujiamin0022016/article/details/109226605)
[golang学习之interface与其它类型转换](https://studygolang.com/articles/7323)