###@property关键字

property属于xcode编译器的关键字主要有三个作用：

* 声明成员变量访问的set/get方法
* 实现set/get方法操作
* 生成带下划线的实例变量

声明语法：`@property(attr1,attr2,attr3，...)NSString* name;`

可使用的修饰关键字根据功能可分为三类：

 1. assign、retain、copy 关于set方法中属性引用计数相关 ，默认值assgin
 2. nonatomic、atomic 关于是否原子访问，默认是atomic
 3. readonly、readwrite 关于访问控制操作,默认值readwrite
 
 所以声明属性 `@property NSString* name;` 等价于 `@property(assgin,atomic,readwrite)NSString * name;`
 
 此外ARC之后还增加了weak、strong属性,其中weak功能与assgin相似，在修饰的属性要释放时，weak会自动其值置为nil,即使在对它发送消息时也不会crash，因为oc中可以给一个空的对象发送消息。而assign不会，直接造成野指针，若在发送消息直接导致crash。strong与retain相似，在很多场景基本通用。
 
 
###属性修饰符
* `assign` 直接简单的赋值，可用来修饰NSInteger,double等C类型的数据类型，也可用来修饰OC对象类型，但对变量的引用计数没有作用，如果所指向的对象释放的话就会造成野指针。其set方法如下：

```
-(void)setName:(NSString *)name
{
    _name = name;
}
```
* `retain` 修饰常用的NSArray，NSDictionary等OC对象，使新值的引用计数加1，赋值时先释放旧值，在retain新值赋值其set方法如下：

```
-(void)setName:(NSString *)name
{
    if (_name != name) {
        [_name release];
    }
    _name = [name retain];
}

``` 
 
* `copy` 复制一个新的对象，不影响原对象的引用计数。使用copy修饰的属性必须遵循NSCopying协议 其set方法如下：
 
 ```
 -(void)setName:(NSString *)name
{
    if (_name != name) {
        [_name release];
    }
    _name = [name copy];
}

 ```
 * `readonly/readwrite` 默认readwrite生成set、get方法，readonly:只生成get方法
 * `nonatomic/atomic` 默认atomic 原子访问，当访问对象时线程要加锁处理的，所以会有一些性能上的开销，nonatomic 非原子操作，多线程访问读写操作估计会出现异常。
 
 
###重写set/get方法、修改方法名 对property的影响
* 重写set方法后，编译器只生成get方法和下划线的成员变量
* 重写get方法后，只生成set方法和下划线的成员变量
* 重写set/get方法后，啥也没了，连个下划线的成员变量也没了

在property中修改set/get方法名,eg

`@property(getter=getMyName,setter=setMyName:)NSString * name;`

此时生成的set方法和get方法如下
> -(void)setMyName:(NSString *)name

> -(NSString *)getMyName
> 


###@synthesize关键字

在xcode4.5以前和`property`一起使用，作用：生成set/get方法的实现，添加一个带下划线的成员变量。

在@implementation中：`@synthesize name = _name;`  生成属性name的set、get方法来操作_name实例变量。

在xcode4.5及以后由于`property`实现了其功能，所以基本也不在用了。


### 属性 ！= &nbsp; 变量

综上可知：

`@property(assgin,atomic,readwrite)NSString * name;`

`name`在这里是一个属性，或者说是属性变量，一个属性除了有个变量之外还有相关的set或get访问方法，生成的`_name`才是真正的成员变量。

所以属性变量和成员变量完全是两个不同的东西。

