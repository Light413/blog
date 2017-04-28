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
[link](http://)