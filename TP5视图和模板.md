![image](http://www.thinkphp.cn/Public/favicon.ico)
####Thinkphp5.0 视图和版本的学习记录总结，详细内容参看官方提供的完全开发手册（虽然文档写的很烂，看一遍还不定能明白是干嘛的，这也是我总结记录的一个原因）。[ThinkPHP5.0完全开发手册http://www.kancloud.cn/manual/thinkphp5/118003](http://www.kancloud.cn/manual/thinkphp5/118003).

> ###文档中经常出现的`视图`、`模板`、`模板引擎`这三个概念究竟如何理解？

* 视图：即是MVC中的V，也就是在模块下面的`view`目录下的html文件,承载着页面内容显示和用户交互相关。

* 模板：在这里我理解为视图就是模板，在`fetch`,`display`等方法中传入的模板参数就是视图文件的路径。

* 模板引擎：就是生成、解析模块的一个机制或者一个封装的操作。解析模板中的一些规则，最终转化为PHP代码。以模板传递变量为例：

```
// 模板变量赋值
$this->assign('name','ThinkPHP');
```
在模板中使用变量：

```
模板变量 ： {$name}
```

经过模板引擎解析后该代码转化为：

```
模板变量 ： <?php echo $name; ?>
```

其中`{`,`}`是在配置文件中模板的标签标记，模板引擎解析定义好的标记，在按照约定的操作来解析模板中的代码为PHP代码，最后转为php文件输出。这下理解了吧，模板引擎就是干这些事情的。

tp5中模板引擎包含PHP原生模板和Think模板引擎，默认的Think,这些在实际中一般用不到，全部都按默认的即可。此外TP5还支持比较有名的`Smarty模板`，需要一些设置操作。

关于模板其实其原理都是差不多，都是为了方便前后端分离操作，有人说php语言本身就可以充当模板和其他模板一样直接嵌入在在html中，所以其他模板没有存在的必要性，这个不同的人各执一词没法讨论。

> ### 视图中`fetch`、`display`方法如何区别及使用

继承了`\think\Controller`类的控制器中可以直接调用`$this->fetch('hello',['name'=>'thinkphp']);`这里的fetch方法是controller的方法，display方法也是一样。fetch方法源代码如下：

```
    /**
     * 加载模板输出
     * @access protected
     * @param string $template 模板文件名
     * @param array  $vars     模板输出变量
     * @param array  $replace  模板替换
     * @param array  $config   模板参数
     * @return mixed
     */
    protected function fetch($template = '', $vars = [], $replace = [], $config = [])
    {
        return $this->view->fetch($template, $vars, $replace, $config);
    }
```
其实调用的也是view中的fetch方法。关于`fetch`、`display`方法作用文档解释如下：


| 方法   	|  说明 		|
|--------	|------	------	|	
| fetch   	| 渲染模板输出	|
| display	| 渲染内容输出	|
| assign	|模板变量赋值	|
|engine		|初始化模板引擎	|

根据以上解释你能分出方法`fetch/display`的区别和作用吗？我的理解如下：

* fetch方法： 用来获取模板并输出显示，默认不带任何参数 自动定位当前操作的模板文件。如果传入参数，参数是具体的一个模板，这个方法较为常用。
* display方法：不使用模板文件，直接传入的参数是具体的内容（可以是字符串或其他内容文档），然后直接输出，传入参数如果为空可能会什么都不显示。这个方法貌似用的不多。


> ###模板输出替换

模板的输出替换就是在模板中替换一些特定的字符串，这个有点类似于宏定义在代码编译时期的直接替换。

替换的字符变量必须在应用的`config.php文件中` `view_replace_str`指定。

```
    // 视图输出字符串内容替换
    'view_replace_str'       => [
        '__PHP__' => 'Hypertext Preprocessor',
    ],
```
这样可以全局在模板中可以直接使用'\__PHP\__' , 然后就行输出内容'Hypertext Preprocessor'。定义的内容必须在`view_replace_str`中以数组的形式存储，否则可能会不起作用。


> ###模板变量输出

模板中可以输出变量，当然是由于模板引擎的作用。变量也可以原样输出即不被引擎解析


可以使用`literal标签`来防止模板标签被解析，例如：

```
{literal}
    Hello,{$name}！
{/literal}
```
上面的{$name}标签被literal标签包含，因此并不会被模板引擎解析，而是保持原样输出。

模板输出中可以是赋值的变量，也可以是系统变量、系统配置参数、系统常量等数据。此外关于变量还可以进行一些运算操作和函数的使用。

#### 感悟
官方完全开发手册反反复复看了几遍，有时去看看源码，有时依然懵逼。现在意识到即使完全掌握了TP，不会HTML，不会CSS还是做不出像样的东西来，听说`bootstrap`适合我这样不懂前端的菜鸟，而且还有基于bootstrap的可视化在线布局工具，可以导出代码，然后在此基础上加以修改。
找了两个可视化布局地址：

* [http://www.layoutit.com/build?](http://www.layoutit.com/build?)

* [http://www.bootcss.com/p/layoutit/?](http://www.bootcss.com/p/layoutit/?)

基于此我还在慢慢的学习。肯定有理解不到位的地方或者其他更好的学习途径和方法，若你能看到，谢谢指教！


