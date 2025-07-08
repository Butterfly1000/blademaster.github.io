---
title: Go 控制结构for循环loop
date: 2021-08-16 10:39:28
tags:
- Go
- "for"
- 编程语言
categories: 技术
---
for循环是编程语言中一种循环语句，**for 是 Go 语言唯一的循环语句.Go 语言中并没有其他语言比如 C 语言中的 while 和 do while 循环.**

> for循环

```
for initialisation; condition; post {  
}
```

这是典型的 for 循环三个表达式,第一个为初始化表达式或赋值语句;第二个为循环条件判定表达式;第三个为循环变量修正表达式,即此处的 post。

示例:
```
package main  
import "fmt"  
func main() {  
   for a := 0; a < 12; a++ {  
      fmt.Print(a,"\n")  
   }  
}  
```
&nbsp;

> for嵌套

Go 语言允许用户在循环内使用循环.接下来我们将为大家介绍嵌套循环的使用.

以下实例使用循环嵌套来输出 2 到 100 间的素数：
```
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var i, j int

   for i=2; i < 100; i++ {
      for j=2; j <= (i/j); j++ {
         if(i%j==0) {
            break; // 如果发现因子,则不是素数
         }
      }
      if(j > (i/j)) {
         fmt.Printf("%d  是素数\n", i);
      }
   }  
}
```

> break

break 语句用于在完成正常执行之前突然终止 for 循环,之后程序将会在 for 循环下一行代码开始执行。简单说就是整个for循环都结束了。

> continue

continue 语句用来跳出 for 循环中当前循环.在 continue 语句后的所有的 for 循环语句都不会在本次循环中执行.循环体会在一下次循环中继续执行。简单说就是for循环当前对应值的循环体内容终止执行，但for循环会继续匹配下一个值。

> goto

Go 语言的 goto 语句可以**无条件地转移到过程中指定的行**.

goto 语句通常与条件语句配合使用.可用来实现条件转移, 构成循环,跳出循环体等功能.

**但是,在结构化程序设计中一般不主张使用 goto 语句, 以免造成程序流程的混乱,使理解和调试程序都产生困难.**

**直接看示例：**

在变量 a 等于 15 的时候跳过本次循环并回到循环的开始语句 LOOP 处：

```
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 10

   /* 循环 */
   LOOP: for a < 20 {
      if a == 15 {
         /* 跳过迭代 */
         a = a + 1
         goto LOOP
      }
      fmt.Printf("a的值为 : %d\n", a)
      a++     
   }  
}
```

解析:

```
goto 和 LOOP配合使用在for循环中是比较经典的案例。上面的例子，a=a+1后a=15,跳转到LOOP处,for循环条件判断位置进行判断。
结果:
a的值为 : 10
a的值为 : 11
a的值为 : 12
a的值为 : 13
a的值为 : 14
a的值为 : 16
a的值为 : 17
a的值为 : 18
a的值为 : 19
```

但是，语法:

```
goto label;
..
.
label: statement;
```

说明，LOOP你可以随意变换其他值(比如A、LO...)。下面也是一种使用例子:

```
package main

import "fmt"

func main() {
   /* 定义局部变量 */
   var a int = 10

   /* 循环 */
   for a < 20 {
      if a == 15 {
         /* 跳过迭代 */
         a = a + 1
         goto A
      }
      fmt.Printf("a的值为 : %d\n", a)
      a++
   }

   A : fmt.Println("对，就是你")
}
```

结果:

```
a的值为 : 10
a的值为 : 11
a的值为 : 12
a的值为 : 13
a的值为 : 14
对，就是你

//没错，跳到A执行后，for循环终止了。
```
