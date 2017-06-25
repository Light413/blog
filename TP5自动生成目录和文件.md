业余学习PHP好大一段时间了，初次接触这个框架，很多资料都是3.x的，特别目录文件也是傻瓜式生成。作为新一代的TP5.0之后好像之前的方式不能用了，可以自定义生成的目录和文件，这个听起来貌似很方便也很灵活。所以开始动手以下操作：。

###下载框架
作为新手自然想到去github下载最新版本。直接`download`之后没有发现`thinkphp`目录及内容，在官方文档中发现git方式需要两个步骤：

	1.首先克隆下载应用项目仓库
	git clone https://github.com/top-think/think tp5
	
	2.然后切换到tp5目录下面，再克隆核心框架仓库：
	git clone https://github.com/top-think/framework thinkphp
	两个仓库克隆完成后，就完成了ThinkPHP5.0的Git方式下载

###自动生成目录文件
开启服务后，把tp5放到指定的目录下，按照手册新建`build.php`生成目录配置文件

```
return [
// 生成运行时目录
    '__file__' => ['common.php','test.php'],
  
    // 定义index模块的自动生成
    'index'    => [
        '__file__'   => ['common.php'],
        '__dir__'    => ['behavior', 'controller', 'model', 'view'],
        'controller' => ['Index','Test', 'UserType'],
        'view'       => ['index/index'],
    ],

    'home'    => [
        '__file__'   => ['common.php'],
        '__dir__'    => ['behavior', 'controller', 'model', 'view'],
        'controller' => ['Test', 'UserType'],
    ],


];

```

然后在`index.php`中

```
// 定义应用目录
define('APP_PATH', __DIR__ . '/../application/');
// 加载框架引导文件
require __DIR__ . '/../thinkphp/start.php';

// 读取自动生成定义文件
$build = include 'build.php';

// 运行自动生成
\think\Build::run($build);

```

以上基本准备完毕，打开浏览器`http://localhost/tp/public/`直接报错：

![Snip20170413_2.png](http://upload-images.jianshu.io/upload_images/1859207-cff64ffcee15e7a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

目前我的目录：

![Snip20170413_1.png](http://upload-images.jianshu.io/upload_images/1859207-3ec72c3ab373a3bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

直接按照提示新建index模块、控制器如下:

![Snip20170413_3.png](http://upload-images.jianshu.io/upload_images/1859207-0bfdf5711bc245e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再次运行率，不在报错了，按build中的规则生成了自定义的目录文件，—_-。
即使以后继续运行，原有的目录文件已存在会直接跳过也不会再次生成。

### 总结
本来一个小小的问题，自己没有新建index模块之前怎么都是报错，可能真的要自己新建一个，如果这样的话自动生成好像不那么自动了。查看官方文档也没有相应的描述，目前我就这样了。大不了把不需要的index模块再删掉。关于这个国产的TP5神器我还在进一步的学习了解中。

###20170508更新
>初始的时候为什么会报错找不到`index`模块？

经学习发现，在tp5 中`convention.php文件中`的`惯例配置文件中`关于模块的配置默认已定义了默认加载的模块为index，部分代码如下

```
    // +----------------------------------------------------------------------
    // | 模块设置
    // +----------------------------------------------------------------------

    // 默认模块名
    'default_module'         => 'index',
    // 禁止访问模块
    'deny_module_list'       => ['common'],
    // 默认控制器名
    'default_controller'     => 'Index',
    // 默认操作名
    'default_action'         => 'index',
    // 默认验证器
    'default_validate'       => '',
    // 默认的空控制器名
    'empty_controller'       => 'Error',
    // 操作方法前缀
    'use_action_prefix'      => false,
    // 操作方法后缀
    'action_suffix'          => '',
    // 自动搜索控制器
    'controller_auto_search' => false,

```
所以开始加载的时候必须已经存在index模块和相关控制器。

* 更改默认配置的加载模块

根据配置的优先级`惯例配置->应用配置->扩展配置->场景配置->模块配置->动态配置`,我们可以在模块配置config.php中修改默认加载的模块已覆盖惯例配置中的配置。

`'default_module' => 'home',`//修改为默认加载home模块

* 也可以在初始的inde.php中修改添加如下：

```
// 定义应用目录
define('APP_PATH', __DIR__ . '/../mydemo/');

define('BIND_MODULE','home');

// 加载框架引导文件
require __DIR__ . '/../thinkphp/start.php';
```
绑定默认到home的模块。


以上能很好地解释为什么在自动生成模块的时候必须新建一个index的模块和相关的控制器了。