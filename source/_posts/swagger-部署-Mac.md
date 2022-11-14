---
title: swagger 部署(Mac)
date: 2022-06-10 15:29:26
tags: 工具推荐
---

## 编辑(swagger-edit)

### 安装

一：安装node.js

二：安装http-server

```
npm install -g http-server
```

三：下载 [swagger-edit](https://github.com/swagger-api/swagger-editor)

```
git clone https://github.com/swagger-api/swagger-editor.git
```

四：运行

```
cd path/to/swagger-edit #就是进入到刚刚克隆的swagger-edit文件夹下
http-servers swagger-editor
```

五：查看

```
默认运行在8080 port
在浏览器打开窗口 http://localhost:8080
```



## 显示部分(swagger-ui)

一： 安装：下载 [swagger-ui](https://github.com/swagger-api/swagger-ui)

二：创建一个文件夹用于**swagger项目**，然后初始化到节点

```
cd path/to/swagger/project
npm init
```

三：

然后你可以检查这里有一个`package.json`；*(npm init后出现`package.json`很正常)*

复制dist文件(swagger-ui里面的)到**swagger项目**；*(就是上面的project)*

安装express模块：

```
npm install express
```

四：

有一个node_modules文件夹和一个`package-lock.json`

创建一个index.js在**swagger**项目下：

```
var express = require('express'); 

var app = express();  
app.use('/swagger', express.static('dist'));
app.get('/', function (req, res) {  
    res.send('Hello World!'); 
});  

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');  
});
```

五：测试

```
node index.js

http://127.0.0.1:3000/  #Hello World
http://127.0.0.1:3000/swagger/ #页面
```



自定义样式：

修改dist中的`index.html(新版在swagger-initializer.js里面了)`的url,可以指定到自定义的json
 默认的url: "http://petstore.swagger.io/v2/swagger.json"
 修改成 ./swagger.json 或`http://localhost:8092/v2/doc`



**重点：**

你可以下载json例子从swagger editor.

假如主机不是基于swagger url，则需要指定主机,修改`swagger.json`里面的host

```
host:****:**   //swagger.json
```

### 参考链接

[swagger 部署(Mac )](https://www.cnblogs.com/jackey2015/p/11130193.html)

[Linux系列之安装Swagger UI教程](https://dandelioncloud.cn/article/details/1436927564110729218)

