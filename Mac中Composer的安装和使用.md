
![Composer](http://image.phpcomposer.com/d/1e/768684492400b1470aa7882b29d5c.png)

Composer 是 PHP5.3以上 的一个依赖管理工具。你可以在自己的项目中声明所依赖的外部工具库（libraries），Composer 会安装这些依赖的库文件。它仅仅是一个依赖关系的管理，如同在`iOS开发`中Swift 和 Objective-C工程中使用的`CocoaPods`一样。


### 安装composer

安装前需确保系统PHP版本在5.3以上,在终端中执行以下命令下载Composer可执行文件：

> curl -sS https://getcomposer.org/installer | php

此操作会下载最新版本到当前的工作目录中。然后在当前路径下就可以操作了，如查看Composer版本：

```
php composer.phar --version //Composer version 1.4.2 2017-05-17 08:17:52

```
如果下载失败也不用纠结了，直接去手动下载合适的版本[https://getcomposer.org/download/](https://getcomposer.org/download/)，结果的一样的。

这应该算是局部安装了，当跳出当前目录还是无法正常使用，这肯定不是我们所期望的。如果要想全局生效需把`composer.phar`移到系统`/usr/local/bin/`目录下：

> mv composer.phar /usr/local/bin/composer

然后在全部就可以使用`composer`,再也不用每次都输入长长的`php composer.phar`了。至此算是安装完毕。



```
//版本更新，如果有则更新到最新版本
composer selfupdate
```
更新完后会提示
`Use composer self-update --rollback to return to version 1.4.1`可以回退到上一版本。


###使用composer


在我们的项目目录下创建文件`composer.json`添加所需要的依赖库的信息，例如需要"monolog/monolog"，"phpmailer/phpmailer"这两个库，json格式如下：

```
{
    "require": {
        "monolog/monolog": "1.0.*",
        "phpmailer/phpmailer": "~5.2"
    }
}

```

然后在终端执行：
>composer install

composer根据json配置开始下载所依赖的库文件，安装完毕后（若无）会生成一个`composer.lock`文件，如果你熟悉`cocoapods`的话应该知道也有个文件`Podfile.lock`。

`composer.lock`作用锁定当前的配置文件，如果已存在，在下次执行`install`操作时会自动读取composer.lock中的信息，即使你已经修该了`composer.json`文件此时也不会生效。

> composer update

此操作会直接从`composer.json`文件读取信息，下载库文件，然后同步更新`composer.lock`。此时这个操作可以看作先删除composer.lock文件，然后在执行install命令操作。

> composer update  monolog/monolog

指定某一个库的更新，其他的没有影响。

以上为compose的简单使用，有了它再使用第三方库操作起来是不是感觉很简单、很方便。


### 关于composer.json文件

以上我们使用的.json文件就一个`require`属性，其实`composer`还支持其他很多属性供我们添加一些其他配置信息。部分属性如下：

```	
name
description
version
type
keywords
homepage
time
license
authors
...

```
具体属性代表的意义及支持的全部属性参看[https://getcomposer.org/doc/04-schema.md](https://getcomposer.org/doc/04-schema.md)


一般情况下我们的项目工程中一个`require`属性就可以了，这里这个composer.json文件为了便于区分暂且称之为A.json。当我们下载了其他第三方库时可发现其目录下也有个composer.json(称之为B.json)或composer.lock。
B.json以`monolog/monolog`为例composer.json文件配置如下：

```
{
  "name": "monolog/monolog",
  "description": "Logging for PHP 5.3",
  "keywords": ["log","logging"],
  "homepage": "http://github.com/Seldaek/monolog",
  "type": "library",
  "license": "MIT",
  "authors": [
    {
      "name": "Jordi Boggiano",
      "email": "j.boggiano@seld.be",
      "homepage": "http://seld.be"
    }
  ],
  "require": {
    "php": ">=5.3.0"
  },
  "autoload": {
    "psr-0": {"Monolog": "src/"}
  }
}

```

这两个json文件肯定是不同的，在我们执行操作的时候都是用的A.json。B.json 属于第三方库本身的配置文件，和项目的配置依赖没有关系，B.json在我们要制作自己的库文件然后发布供别人下载使用时是必须的，通过它别人才能找到我们发布的库，这里暂且不谈。只需要记住只有根目录下的composer.json才是真正的项目依赖配置文件。



###关于镜像

Composer在安装或更新的时候可能会出现失败或无法访问的情况，这是由于访问的外部网络可能被墙了。所以为Composer配置了一个国内提供的镜像，终端中执行:

> composer config -g repo.packagist composer https://packagist.phpcomposer.com


以后每次的下载或更新都是访问的国内服务器了，具体Packagist / Composer 镜像参看[phpcomposer](http://www.phpcomposer.com)



###更多了解

 * ###[https://getcomposer.org](https://getcomposer.org)
 * ###[http://www.phpcomposer.com](http://www.phpcomposer.com)






