---
title: GO 包package和import
date: 2021-08-16 10:17:15
tags: GO入门
---
> 什么是package

package用于对我们的程序进行***分类\***,以便易于维护. 每个go文件都属于某个包. 每个Go应用程序必须具有`main`包。包名称应以小写字母书写. 如果更改并重新编译package,则必须重新编译使用此package的所有代码程序！

> Import 别名导入

 `import mongo "mywebapp/libs/mongodb/db"`

> Import 省略package名导入

这里的点.符号表示,对包 lib 的调用直接省略包名,您我以后就是一家人,不分彼此,您的东西就像我就的一样,随便用.

```
package main
import . "github.com/libragen/felix/lib"
func main() {
	SayHello() //如果没有.忽略包名,那么需要lib.SayHello()。不过这样可能要注意函数名或变量名是否会冲突，个人不推荐
}
```

> Import  执行初始化工作导入

`import _ "github.com/libragen/felix/lib"`

形象解释:这里说的是我还不准备现在使用您们家的东西,但得提前告诉一声.您先做好准备,先准备好饭菜,等我来就行,也有可能我压根就不来.

具体解释:特殊符号“_” **仅仅**会导致 lib 执行初始化工作,如初始化全局变量,**调用 init 函数**。而正常导入一个包的时候，该包的init和其他函数都会被导入。

>  package和文件的关系

**一个文件夹下只能有一个package.**

import后面的其实是`GOPATH`开始的相对目录路径,包括最后一段.

- 但由于一个目录下只能有一个`package`,所以`import`一个路径就等于是`import`了这个路径下的包.
- 注意,这里指的是“直接包含”的go文件. 如果有子目录,那么子目录的父目录是完全两个包.
- 比如您实现了一个计算器`package`,名叫calc,位于calc目录下; 但又想给别人一个使用范例,于是在`calc`下可以建个`example`子目录（`calc/example/`）, 这个子目录里有个`example.go（calc/example/example.go）`. 此时,`example.go`可以是main包,里面还可以有个main函数.
&nbsp;
&nbsp;

**一个`package`的文件不能在多个文件夹下.**
- 如果多个文件夹下有重名的`package`,它们其实是彼此无关的package.
- 如果一个`go`文件需要同时使用不同目录下的同名`package`,需要在`import`这些目录时为每个目录指定一个package的**别名**.