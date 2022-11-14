---
title: 扩展管理工具composer
date: 2021-10-09 10:47:51
tags: 知识点
---

### 前言

作为最热门的php的扩展工具，`composer`在各个php框架基本都会用到，本文介绍一下实用常规操作。

### composer的作用

Composer是 PHP 用来管理依赖（dependency）关系的工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会帮你安装这些依赖的库文件。

### 初始化扩展包

先创建一个空目录，再在空目录里执行 Composer 自带的 init 命令，自动生成 Composer 的配置文件(composer.json)。

```
$ mkdir composer_test
$ cd composer_test
$ composer init
```

**请注意：** Minimum Stability(官方说明) 要输入 dev，表明我们的扩展包最小稳定版的开发版。否则 Composer 的默认 Minimum Stability 是 stable，扩展包需要打上版本号才能被 Composer 认为是稳定版(stable)，前期开发调试时简单点处理，设置为 dev，这样引入开发中的 Composer 扩展包时不需要去管什么版本号，等开发调试过了再说。

type 字段的[官方说明](https://getcomposer.org/doc/04-schema.md#type)。
version 字段的[官方说明](https://getcomposer.org/doc/04-schema.md#version)。

```
    Welcome to the Composer config generator

This command will guide you through creating your composer.json config.

Package name (<vendor>/<name>) [waq/composer_test]:
Description []:
Author [晴x <24xxxxxx50@qq.com>, n to skip]:
Minimum Stability []: dev
Package Type (e.g. library, project, metapackage, composer-plugin) []:
License []:

Define your dependencies.

Would you like to define your dependencies (require) interactively [yes]? yes
Search for a package:
Would you like to define your dev dependencies (require-dev) interactively [yes]? yes
Search for a package:
Add PSR-4 autoload mapping? Maps namespace "Waq\ComposerTest" to the entered relative path. [src/, n to skip]:

{
    "name": "waq/composer_test",
    "autoload": {
        "psr-4": {
            "Huangyanyu\\ComposerTest\\": "src/"
        }
    },
    "authors": [
        {
            "name": "晴x",
            "email": "24xxxxxx50@qq.com"
        }
    ],
    "minimum-stability": "dev",
    "require": {}
}

Do you confirm generation [yes]? y
Generating autoload files
Generated autoload files
PSR-4 autoloading configured. Use "namespace Waq\ComposerTest;" in src/
Include the Composer autoloader with: require 'vendor/autoload.php';
```


### 包的环境要求和自动加载规范

环境要求通过 composer.json 里的 require 字段来限制。

自动加载规范通过 autoload 字段来定义。

现在假定需要 PHP 的版本大于等于 7.0.0，并且当前包的 src 目录下的文件以 Waq\ComposerTest 命令空间来加载，在 composer.json 中增加以下代码：

```
"require": {
  "php": ">=7.0"
},
"autoload": {
  "psr-4": {
    "Waq\\ComposerTest\\": "src"
  }
}
```

如果需要引入其他的非 PSR-4 规范的源码文件，可能会需要用到 autoload 下的 file(官方说明) 字段，每一次请求 PHP 时都是包含这些文件。

例如要包含钉钉 SDK 的代码，可以在 composer.json 中增加以下代码：

```
"autoload": {
  "files": [
    "lib/taobao-sdk-PHP/TopSdk.php"
  ]
}
```

### 扩展包里创建类文件

src 目录下新建 Calc.php 文件，内容如下：

```
<?php

namespace Waq\ComposerTest;

class Calc
{
    // 计算平方值
    public function square($val)
    {
        return $val * $val;
    }
}
```

### 在项目中引入本地的扩展包

项目要引入本地目录的扩展包，先要在 composer.json 文件中加入以下内容：

```
"repositories": [
    {
        "type": "path",
        "url": "/Users/waq/project/composer_test"
    }
]
```

然后在项目目录下执行 Composer 命令：

```
composer require waq/composer_test:@dev
```

或者

```
composer require waq/composer_test:dev-master
```

**注意：** 上面命令中的 `waq/composer_test` 就是 Composer 扩展包里 composer.json 的 name 字段值。@dev 或 dev-master 表示引入的扩展包是主分支的最新版，当还未将扩展包提交到 Github 仓库或者发布到 Packagist 上时，一般这样引入扩展包。

### 测试项目中引入的本地扩展包

新建测试 PHP 文件：

```
<?php
$calc = new \Waq\ComposerTest\Calc();
echo $calc->square('12');
```

### 基本操作

`composer install` 安装依赖包，当你下载一个项目后，这或许是你改做的第一件事。

`composer require/update` 更新依赖包，我更喜欢用composer require。

`composer remove` 卸载依赖包

### 指定版本

update则无法在命令行传入新的版本号，需要先手动在`composer.json`中指定**新的版本号**，然后执行更新命令。但require可以直接在命令行传入版本号，例如:

```
composer require hashids/hashids:2.0.0
```

升级

```
composer require hashids/hashids:3.0.0
```

降级

```
composer require hashids/hashids:2.0.4
```

### 符号

版本号范围大于/大于等于：`>1.2.3` ` >=1.2.3`

小于/小于等于：`<1.2.3` ` <=1.2.3`

确切的版本号：`1.2.3`

~1.2.3: `1.2.3 <= version < 1.3`

^1.2.3: `1.2.3 <= version < 2.0`

### 指定分支



### 更新

### 全局参数(常用)

- -v 表示正常输出。
- -vv 表示更详细的输出。
- -vvv 则是为了 debug。
- **--help (-h):** 显示帮助信息。

### 常见的坑

> **卡着不动**

因为某些特殊情况，老外的源特别慢，我们就得把源改为国内的。阿里某个云。

```
composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
```

> **添加-vvv选项**

这样就知道compose这个家伙到底啥情况了。

```
composer install -vvv
```



> 提示不符合composer.json需要的版本，例如>=php7，但实际<php7也能正常运行，所以忽略

这样就可以正常运行了

```
composer install --ignore-platform-reqs
```



> Warning from https://mirrors.aliyun.com/composer: Support for Composer 1 is deprecated and some packages will not be available. You should upgrade to Composer 2.

解决方法：
更新一下 composer 工具版本：

```
composer self-update
```

出现错误:

```
The "https://getcomposer.org/versions" file could not be downloaded: SSL operation failed with code 1.

解决文章：
https://stackoom.com/question/3t3EF
```


具体解决流程：

```
问题：
composer update "xxx/xxx"

Warning from https://mirrors.aliyun.com/composer: You are using an outdated version of Composer. Composer 2.0 is now available and you should upgrade. See https://getcomposer.org/2

原因：从报错信息可以看出是composer版本太旧的。

解决办法：升级本地的composer版本, composer self-update

执行composer self-update

macOS: Installing composer fails with error “The ”https://getcomposer.org/versions“ file could not be downloaded: SSL operation failed with code 1”

解决：
php -i | grep php.ini

cat php.ini | grep "openssl.cafile"
cat php.ini | grep "curl.cainfo"

brew install openssl

mkdir -p /usr/local/etc/openssl/ curl 'http://curl.haxx.se/ca/cacert.pem' -o '/usr/local/openssl/cert.pem'

php.ini
openssl.cafile="/usr/local/etc/openssl/cert.pem"
curl.cainfo="/usr/local/etc/openssl/cert.pem"

另外，如果还不行，可以再看看
allow_url_fopen = On  
user_agent="PHP"  #这个我是；注释的，但不影响
```

获取php.ini
```
php -i | grep php.ini  或 php --ini

Configuration File (php.ini) Path => /usr/local/etc/php/5.6
Loaded Configuration File => /usr/local/etc/php/5.6/php.ini
```

获取cert地址
```
php -r "print_r(openssl_get_cert_locations());"

Array
(
    [default_cert_file] => /usr/local/opt/openssl/ssl/cert.pem
    [default_cert_file_env] => SSL_CERT_FILE
    [default_cert_dir] => /usr/local/opt/openssl/ssl/certs
    [default_cert_dir_env] => SSL_CERT_DIR
    [default_private_dir] => /usr/local/opt/openssl/ssl/private
    [default_default_cert_area] => /usr/local/opt/openssl/ssl
    [ini_cafile] => 
    [ini_capath] => 
)
```

### 参考文章

[composer指定php,composer 更新指定的依赖包](https://blog.csdn.net/weixin_29525745/article/details/116037185)

[PHP扩展管理工具COMPOSER的坑、常见问题和解决方法](https://www.freesion.com/article/22651405122/)

[Composer设置忽略版本匹配的方法](http://www.thinkphp.cn/code/2430.html)

[Warning from https://mirrors.aliyun.com/composer: Support for Composer 1 is deprecated and some packages will not be available. You should upgrade to Composer 2.](https://www.cnblogs.com/aze999/p/15062222.html)

[创建 PHP Composer 包并使用的操作指南](https://www.cnblogs.com/imzhi/p/create-php-composer.html)

## 后续

如果后面有空的话，也可能继续写一篇创建自己的composer包。
