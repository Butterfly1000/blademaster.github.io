---
title: 简单使会用React
date: 2022-06-10 15:44:12
tags: 知识点
---
## 环境

Linux Centos7

## 一、安装node.js(npm)

```
### yum安装npm(完成这个即可)

curl --silent --location https://rpm.nodesource.com/setup_10.x | bash -
yum install -y nodejs
npm install -g cnpm --registry=https://registry.npm.taobao.org
npm config set registry https://registry.npm.taobao.org

### 常用操作
npm init
npm install
npm run build
npm -v


### node.js升级
//安装npm升级工具
npm install -g n
//安装最近的稳定版本
n stable
---------------------------

  installing : node-v16.15.0
       mkdir : /usr/local/n/versions/node/16.15.0
       fetch : https://nodejs.org/dist/v16.15.0/node-v16.15.0-linux-x64.tar.xz
     copying : node/16.15.0
   installed : v16.15.0 to /usr/local/bin/node
      active : v10.24.1 at /bin/node
---------------------------
记得更换/bin/node下面的node

### 升级npm
升级到最新版本： npm i -g npm to update
升级到指定版本： npm -g i npm@XXX 

```



## 二、基础操作

```
## package.json
mkdir -p sa-frontend
cd sa-frontend
npm init

然后就有了package.json了

## 生成html
touch index.html
```



## 三、安装React

```
$ cnpm install -g create-react-app 
$ create-react-app sa-frontend //如果提示node.js版本太低，就按照上面的升级
-------------------
Success! Created sa-frontend at /sa-frontend
Inside that directory, you can run several commands:

  npm start
    Starts the development server.

  npm run build
    Bundles the app into static files for production.

  npm test
    Starts the test runner.

  npm run eject
    Removes this tool and copies build dependencies, configuration files
    and scripts into the app directory. If you do this, you can’t go back!

We suggest that you begin by typing:

  cd sa-frontend
  npm start

Happy hacking!
-------------------
$ cd sa-frontend/
$ npm start
```



## 四、html文件中引入react

```
<html>
  <head>
  </head>
  <body>
    <div id="root"></div>
    <!-- 1 引入js文件 -->
    <script src="./node_modules/react/umd/react.development.js"></script>
    <script src="./node_modules/react-dom/umd/react-dom.development.js"></script>
    <!-- 2 创建react元素 -->
     <script>
      //   param1 元素名称
      //   param2 元素属性
      //   param3 第三个及以后的参数 元素的子节点
  
      const title =  React.createElement('h1',null,"hello React")

       // 3. 渲染react元素
       //  param1 要渲染的react元素
       //  param2 挂载点 
       ReactDOM.render(title,document.getElementById('root'))
     </script>
  </body>
</html>
```



## 运行

> 运行npm start

访问：`localhost:3000`

> 如果不想用默认端口3000

```
cd node_modules/react-scripts/scripts
vi start.js

找到
const DEFAULT_PORT = parseInt(process.env.PORT, 10) || 3000;

将这边3000修改成你想要的端口9033
```



这样就成功了，如果不成功可以查看端口是否开放。



## 项目

> npm run build

这会在项目树中生成一个名为**build**的文件夹。该文件夹包含了我们的ReactJS应用程序所需的所有静态文件。



> **用Nginx提供静态文件访问**

将sa-frontend/build文件夹的内容移到**[nginx安装目录]/html**中。

这样，生成的index.html文件可以在[nginx安装目录]/html/index.html（**这是Nginx的默认访问文件**）中访问到。

默认情况下，Nginx服务器会监听80端口。可通过修改[nginx安装目录]/conf/nginx.conf文件中的server.listen参数来指定其他端口9034。

使用浏览器打开localhost:9034，ReactJS应用程序将会出现。



**nginx.conf**

```
server {
     listen 9034;
     server_name  localhost;

     charset 'utf-8';

     root   /data/wwwroot/kubernetes/sa-frontend-item/html;
     index  index.html index.htm;

     error_log   /data/logs/sa/error.log;
     access_log  /data/logs/sa/access.log main;
     location / {
        try_files $uri $uri/ /index.html;
      }

      location ^~ /assets/ {
         gzip_static on;
         expires max;
         add_header Cache-Control public;
      }

      error_page 500 502 503 504 /500.html;
      client_max_body_size 20M;
      keepalive_timeout 10;
}
```

## 参考文章

[React入门一：React简介及基本使用](https://blog.csdn.net/qq_39008205/article/details/118551913?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522165234334316782395318349%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=165234334316782395318349&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-2-118551913-null-null.142^v9^pc_search_result_cache,157^v4^control&utm_term=react&spm=1018.2226.3001.4187)

[怎么用npm安装react？](https://www.html.cn/qa/react/14370.html)

[linux服务器部署react项目步骤详解](https://blog.csdn.net/addccc/article/details/123956910)

