---
title: GO 控制结构if-else
date: 2021-08-16 10:26:38
tags: GO入门
---
> 简单例子:

```
if condition1 {
	// do something	
} else if condition2 {
	// do something else	
} else {
	// catch-all or default
}
```

**注意:**

1.即使当代码块之间只有一条语句时,大括号也不可被省略。

2.关键字 if 和 else 之后的**左**大括号 `{` 必须和关键字在同一行

```
if x{
}
else {	// 无效的
}
```
&nbsp;
> 使用方式

当 if 结构内有 break,continue,goto 或者 return 语句时,Go 代码的常见写法是省略 else 部分.无论满足哪个条件都会返回 x 或者 y 时,一般使用以下写法：
```
if condition {
	return x
}
return y
```

**注意事项** 不要同时在 if-else 结构的两个分支里都使用 return 语句,这将导致编译报错 `function ends without a return statement`（您可以认为这是一个编译器的 Bug 或者特性）.（ **译者注：该问题已经在 Go 1.1 中被修复或者说改进** ）

>一些有用的例子

1.判断一个字符串是否为空：
- `if str == "" { ... }`
- `if len(str) == 0 {...}` (个人习惯)

2.判断运行 Go 程序的操作系统类型,这可以通过常量 `runtime.GOOS` 来判断.
```
if runtime.GOOS == "windows"	 {
    .	..
} else { // Unix-like
    .	..
}
```

这段代码一般被放在 init() 函数中执行.这儿还有一段示例来演示如何根据操作系统来决定输入结束的提示：

```
var prompt = "Enter a digit, e.g. 3 "+ "or %s to quit."

func init() {
    if runtime.GOOS == "windows" {
        prompt = fmt.Sprintf(prompt, "Ctrl+Z, Enter")		
    } else { //Unix-like
        prompt = fmt.Sprintf(prompt, "Ctrl+D")
    }
}
```

3.函数 `Abs()` 用于返回一个整型数字的绝对值:

```
func Abs(x int) int {
    if x < 0 {
        return -x
    }
    return x	
}
```

4.`isGreater` 用于比较两个整型数字的大小:

```
func isGreater(x, y int) bool {
    if x > y {
        return true	
    }
    return false
}
```

> if-else包含一个初始化语句

例如:

```
val := 10
if val > max {
	// do something
}
```

也可以这样写

```
if val := 10; val > max {
	// do something
}
```

注意:**使用简短方式 `:=` 声明的变量的作用域只存在于 if 结构中**（在 if 结构的**大括号之间**,如果使用 if-else 结构则在 else 代码块中变量也会存在)。 如果**变量在 if 结构之前就已经存在**,那么在 if 结构中,该**变量原来的值会被覆盖**.最简单的解决方案就是**不要在初始化语句中声明变量**.

```
package main

import "fmt"

func main() {
	var first int = 10
	var cond int

  //基础款，声明变量赋予好值，再进行if判断
	if first <= 0 {
		fmt.Printf("first is less than or equal to 0\n")
	} else if first > 0 && first < 5 {
		fmt.Printf("first is between 0 and 5\n")
	} else {
		fmt.Printf("first is 5 or greater\n")
	}
	
	//声明变量，在if判断中，先赋值再判断。这个cond赋予的值会覆盖上面的声明变量。
	if cond = 5; cond > 10 {
		fmt.Printf("cond is greater than 10\n")
	} else {
		fmt.Printf("cond is not greater than 10\n")
	}
	
	fmt.Println(cond) //等于5
	
	//在if判断中，声明变量赋值再判断。
	if cond2 := 5; cond2 > 10 {
		fmt.Printf("cond2 is greater than 10\n")
	} else {
		fmt.Printf("cond2 is not greater than 10\n")
	}
	
	fmt.Println(cond2) //报错，未声明cond2。因为if中声明的变量的作用域只存在于if结构中。
}
```

输出：

```
first is 5 or greater
cond is not greater than 10
```

