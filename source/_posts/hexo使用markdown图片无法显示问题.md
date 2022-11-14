---
title: hexo使用markdown图片无法显示问题(引用,亲测可用)
date: 2021-07-13 10:56:15
tags: 随笔
---

hexo默认无法自动处理文章插入本地图片，需要通过扩展插件支持。

> 如何处理图片路径问题

* 配置_config.yml里面的post_asset_folder:false这个选项设置为true。

* 安装hexo-asset-image，运行hexo n "xxxx"来生成md博文时，/source/_posts文件夹内除了xxxx.md文件还有一个同名的文件夹，把图片放入该文件夹。

```
npm install hexo-asset-image --save 
```

* 使用`![图片类型或空](存放图片文件夹名/图片名.png)`直接插入图片即可。

> 关于插件问题

由于hexo3版本后对很多插件支持有问题，hexo-asset-image插件在处理data.permalink链接时出现路径错误，把年月去掉了，导致最后生成的路径为%d/xxx/xxx需要对其做兼容处理。通过判断当前版本是否等于3的版本做不同的路径分割。

* 在代码中加入：
```
var version = String(hexo.version).split('.');
```
&emsp;&emsp;

* 修改`date.permalink`处理：
```
var link = data.permalink;  
if(version.length > 0 && Number(version[0]) == 3) 
    var beginPos = getPosition(link, '/', 1) + 1; 
else 
    var beginPos = getPosition(link, '/', 3) + 1;
```
重新生成静态文件即可正确显示。

* 可直接安装已经修改过得插件`npm install https://github.com/7ym0n/hexo-asset-image --save`。

**如果不行，考虑用这个**安装插件

`npm install https://github.com/CodeFalling/hexo-asset-image --save`

-------------------------------
作者：菜鸡_快递到了
链接：https://www.jianshu.com/p/3db6a61d3782
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
--------------------------------