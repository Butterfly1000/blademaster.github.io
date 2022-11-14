---
title: Hexo博客中插入音乐
date: 2021-08-03 15:07:05
tags: 随笔
---

### 成品

具体成功已体现在`网络掩码`文章底部。

### 操作

一款html5音乐播放器：Aplayer。把Aplayer加入hexo需要用到hexo-tag-aplayer插件。

切换到hexo目录，运行：
```
npm install --save hexo-tag-aplayer
```

安装完成后，在需要添加音乐的地方加上：
```
{% aplayer "歌曲名称" "作者" "音乐_url" "封面图片_url" "autoplay" %}
```

[文章链接](https://www.jianshu.com/p/6e41e3191963)

### 网易云外链的处理

网易云提供的外链连接无法成功播放，所以找到一篇解决的文章。

例如：杨钰莹的心雨，网址是：`http://music.163.com/#/song?id=317151`

很明显，ID是317151

那么，这首歌的真实地址就是：`http://music.163.com/song/media/outer/url?id=317151.mp3`

[文章链接](https://www.cnblogs.com/MirageFox/p/7995929.html)

### 问题

mac端(`win端没有测试过`)控制不了音量大小，只能空过电脑或耳机条件。

```
{% aplayer "U" "许杨玉琢" "https://music.163.com/song/media/outer/url?id=1475649128.mp3" "/yyzka.jpeg" %}
```

当`_config.yml asset_inject: false`时，上面是无效的，需要用下面。

<div class="aplayer no-destroy" data-id="1846489859" data-server="netease" data-type="song"  data-autoplay="false" data-lrcType="-1"> </div>



