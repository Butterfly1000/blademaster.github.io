---
title: Channel 信道
date: 2022-03-22 15:00:09
tags:
- Go
- Channel
- 并发
categories: 技术
---
说到channel，就一定要说一说线程了。任何实际项目，无论大小，并发是必然存在的。并发的存在，就涉及到线程通信。在当下的开发语言中，线程通讯主要有两种，共享内存与消息传递。共享内存一定都很熟悉，通过共同操作同一对象，实现线程间通讯。消息传递即通过类似聊天的方式。golang对并发的处理采用了协程的技术。golang的goroutine就是**协程**的实现。协程的概念很早就有，简单的理解为轻量级线程，goroutine就是为了解决并发任务间的通信而设计的。golang解决通信的理念是：不要通过共享内存来通信,而应该通过**通信**来共享内存。golang解决方案是消息传递机制，消息的传递就是通过channel来实现的。

**Channal是什么？**Channal就是用来通信的，就像Unix下的管道一样，在Go中是这样使用Channel的。

下面的程序演示了一个goroutine和主程序通信的例程。这个程序足够简单了。

```
package main
import "fmt"
func main() {
    //创建一个string类型的channel
    channel := make(chan string)
    //创建一个goroutine向channel里发一个字符串
    go func() { channel <- "hello" }()
    msg := <- channel
    fmt.Println(msg)
}
```

**指定channel的buffer**

指定buffer的大小很简单，看下面的程序：

```
package main
import "fmt"
func main() {
    channel := make(chan string, 2)
    go func() {
        channel <- "hello"
        channel <- "World"
    }()
    msg1 := <-channel
    msg2 := <-channel
    fmt.Println(msg1, msg2)
}
```



**Channel的阻塞**

现在谈一谈对channe阻塞l的理解。　　

　　发送者角度：对于同一个通道，发送操作（协程或者函数中的），在**接收者准备好之前是阻塞的**。如果chan中的数据**无人接收**，就**无法再给通道传入其他数据**。因为新的输入无法在通道非空的情况下传入。所以发送操作会等待 chan 再次变为可用状态：就是通道值被接收时（可以传入变量）。

　　接收者角度：对于同一个通道，**接收操作是阻塞的**（协程或函数中的），直到发送者可用：如果通道中没有数据，接收者就阻塞了。

**案例1:**

```
package main

import (
    "fmt"
)

func f1(in chan int) {
    fmt.Println(<-in)
}

func main() {
    out := make(chan int)
    out <- 2   //13行
    go f1(out)
}
```

运行结果：fatal error: all goroutines are asleep - deadlock!

这是由于第13行之前不存在对out的接收，所以，对于out <- 2来说，永远是阻塞的，即一直会等下去。

```
这很明显，因为out<-2插入后并没有被使用，就一直阻塞在等待被使用，走不到go fl(out)。所以将go fl(out)放在out<-2前面即可。
```



**案例2:**

```
package main

import (
    "fmt"
)

func main() {
    c1 := make(chan int)
    func(){
    	time.Sleep(time.Second*2)
    	c1<-"result 1"
	  }()
		fmt.Println("c1 is",<-c1)
}
```

结果：deadlock，因为push和pull永远不可能同时发生，这就是阻塞channel的不当用法。

解决方法

```
func main() {
    c1 := make(chan int)
    go func(){
    	time.Sleep(time.Second*2)
    	c1<-"result 1"
	  }()
		fmt.Println("c1 is",<-c1)
}
#通过在另一个协程中run push代码，使得channel的生产和消费可以同时对接，正常的阻塞使用方式。
```

另外的解决方法

```
func main() {
    c1 := make(chan int,1)
    func(){
    	time.Sleep(time.Second*2)
    	c1<-"result 1"
	  }()
		fmt.Println("c1 is",<-c1)
}

# 给channel加一个buffer，只要buffer没用尽，大家就不用阻塞。
```



**案例3:**

**注意**，channel默认上是阻塞的，也就是说，如果Channel满了，就阻塞写，如果Channel空了，就阻塞读。于是，我们就可以使用这种特性来同步我们的发送和接收端。

下面这个例程说明了这一点，代码有点乱，不过我觉得不难理解。

```
package main

import "fmt"
import "time"

func main() {
    channel := make(chan string) 

    go func() {
        channel <- "hello" //发送方等待接收
        fmt.Println("write \"hello\" done!")

        channel <- "World" //Reader在Sleep，这里在阻塞
        fmt.Println("write \"World\" done!")

        fmt.Println("Write go sleep...")
        time.Sleep(3*time.Second)
        channel <- "channel"
        fmt.Println("write \"channel\" done!")
    }()

    time.Sleep(2*time.Second)
    fmt.Println("Reader Wake up...") //首先打印

    msg := <-channel
    fmt.Println("Reader: ", msg)

    msg = <-channel
    fmt.Println("Reader: ", msg)

    msg = <-channel //Writer在Sleep，这里在阻塞
    fmt.Println("Reader: ", msg)
}
```

结果：

```
Reader Wake up...
Reader:  hello 
write "hello" done! 
write "World" done! 
Write go sleep...
Reader:  World
write "channel" done! 
Reader:  channel

解释：
Reader Wake up...  //没有问题，不解释

Reader:  hello
因为go func()的代码在前，所以会先执行”channel <- "hello"“，这时候被没有消费，阻塞。
当主程序运行到msg := <-channel，打印了Reader:  hello，然后执行到下面的msg = <-channel，没有数据，被阻塞。

write "hello" done! 
write "World" done! 
Write go sleep...
go func()执行channel<-"hello"阻塞被释放，执行代码fmt.Println("write \"hello\" done!")；
然后执行channel <- "World"的时候，插入数据，因为主程序已有读取请求，所以没有阻塞，并打印write "World" done!和Write go sleep...; 然后，channel <- "channel"因为需要等待3s未马上执行到，所以会比主程序慢。

Reader:  World
主程序里面msg = <-channel因channel <- "World"，有数据了，解除阻塞打印Reader:  World，并再执行下面代码msg = <-channel，这时候因为channel <- "channel"还在等待中，再次被阻塞。

write "channel" done!
直到3s后，channel <- "channel"解除阻塞，打印write "channel" done!

Reader:  channel
主程序解除阻塞，再打印Reader:  channel
```

**详解go语言 make(chan int, 1) 和 make (chan int) 的区别**

**无缓冲区channel**

用make(chan int) 创建的chan, 是无缓冲区的, send 数据到chan 时，在没有协程取出数据的情况下， 会阻塞当前协程的运行。ch <- 后面的代码就不会再运行，直到channel 的数据被接收，当前协程才会继续往下执行。

```
package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan int) // 创建无缓冲channel
    go func() {
      fmt.Println("time sleep 5 second...")
      time.Sleep(5 * time.Second)
      <-ch
      fmt.Println("read...")
    }()
    fmt.Println("即将阻塞...")
    ch <-1  // 协程将会阻塞，等待数据被读取
    fmt.Println("ch 数据被消费，主协程退出")
}

#结果：
即将阻塞...
time sleep 5 second...
send...
ch 数据被消费，主协程退出

##这边可以看到接收方的代码先打印，然后打印发送方下面的代码，这个符合逻辑，接收解锁执行，发送解锁执行。
```



**有缓冲区channel**

channel 的缓冲区为1，向channel 发送第一个数据，主协程不会退出。发送第二个时候，缓冲区已经满了， 此时阻塞主协程。

```
package main

import (
    "fmt"
    "time"
)

func main() {
      ch := make(chan int, 1) // 创建有缓冲channel
      go func() {
         fmt.Println("time sleep 5 second...")
         time.Sleep(5 * time.Second)
         <-ch
         fmt.Println("read...")
      }()
      ch <-1 // 协程不会阻塞，等待数据被读取
      fmt.Println("第二次发送数据到channel， 即将阻塞")
      ch <-1 // 第二次发送数据到channel, 在数据没有被读取之前，因为缓冲区满了， 所以会阻塞主协程。
      fmt.Println("ch 数据被消费，主协程退出")
}

结果：
第二次发送数据到channel， 即将阻塞   //阻塞了，因为buffer为1，等待前面的消费
time sleep 5 second...
read...               //消费了
ch 数据被消费，主协程退出 //插入，执行
```



**多个Channel的select**

```
package main
import "time"
import "fmt"
func main() {
    //创建两个channel - c1 c2
    c1 := make(chan string)
    c2 := make(chan string)
    //创建两个goruntine来分别向这两个channel发送数据
    go func() {
        time.Sleep(time.Second * 1)
        c1 <- "Hello"
    }()
    go func() {
        time.Sleep(time.Second * 1)
        c2 <- "World"
    }()
    //使用select来侦听两个channel
    for i := 0; i < 2; i++ {
        select {
          case msg1 := <-c1:
              fmt.Println("received", msg1)
          case msg2 := <-c2:
              fmt.Println("received", msg2)
        }
    }
}
```

注意：上面的select是阻塞的，所以，才搞出ugly的for i <2这种东西。

**Channel select阻塞的Timeout**

解决上述那个for循环的问题，一般有两种方法：一种是阻塞但有timeout，一种是无阻塞。我们来看看如果给select设置上timeout的。

```
    for {
        timeout_cnt := 0
        select {
        case msg1 := <-c1:
            fmt.Println("msg1 received", msg1)
        case msg2 := <-c2:
            fmt.Println("msg2 received", msg2)
        case  <-time.After(time.Second * 30)：
            fmt.Println("Time Out")
            time_cnt++
        }
        if time_cnt > 1 {
            break
        }
    }
```

上面代码中高亮的代码主要是用来让select返回的，注意 case中的time.After事件。

**Channel的无阻塞**

好，我们再来看看无阻塞的channel，其实也很简单，就是在select中加入default，如下所示：

```
    for {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        default: //default会导致无阻塞
            fmt.Println("nothing received!")
            time.Sleep(time.Second)
        }
    }
```

**Channel的关闭**

关闭Channel可以通知对方内容发送完了，不用再等了。参看下面的例程：

```
package main
import "fmt"
import "time"
import "math/rand"
func main() {
    channel := make(chan string)
    rand.Seed(time.Now().Unix())
    //向channel发送随机个数的message
    go func () {
        cnt := rand.Intn(10)
        fmt.Println("message cnt :", cnt)
        for i:=0; i<cnt; i++{
            channel <- fmt.Sprintf("message-%2d", i)
        }
        close(channel) //关闭Channel
    }()
    var more bool = true
    var msg string
    for more {
        select{
        //channel会返回两个值，一个是内容，一个是还有没有内容
        case msg, more = <- channel:
            if more {
                fmt.Println(msg)
            }else{
                fmt.Println("channel closed!")
            }
        }
    }
}
```



参考文章：

[golang协程——通道channel阻塞](https://www.cnblogs.com/xiaofengshuyu/p/5190824.html)

[golang channel阻塞与非阻塞用法](https://zhuanlan.zhihu.com/p/22620172)

[详解go语言 make(chan int, 1) 和 make (chan int) 的区别](https://blog.csdn.net/qq_31406415/article/details/110521553)

[GO 语言简介（下）— 特性](https://coolshell.cn/articles/8489.html#goroutine)

