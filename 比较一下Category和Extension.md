###Category 和 Extension介绍

`Category`

* 可以给当前已知的类添加方法（类方法和实例方法）,这个类可以是自定义的类，也可以是系统自带的类。当添加的方法和类的原有方法重名时，会‘覆盖’类的原有方法.
> ps:原类的方法还是存在的,并不是真的给覆盖了。因为在运行时当给对象发消息，会根据对象的‘isa’指针找到所属的类，然后在类的方法列表中根据方法名寻找，当找到第一个就返回了，而通过category添加的方法又在方法列表的最上面所以没有机会找到类原来方法`。
* 可以通过`property`添加属性，但也只是对属性的set、get方法声明，没有实现，也不能生成带下划线的变量。
* 不能添加成员变量。注意，通过runtime 关联对象添加的不叫属性，也不叫成员变量，它和当前类没有根本上的关系，只是仅仅的连在了一起。

`Extension`

* 可以给一个已知的类添加方法（类方法和实例方法）,这个类只能是自定义的类，不可以是系统自带的类，因为不能得到`.m`文件。由于在已知的类.m &nbsp;中操作所以可以对外界隐藏一些方法或变量，使其成为私有。
* 可以添加属性和成员变量，也可以为属性生成set、get方法和默认的实现。通过extension添加的方法和变量真的成为了当前类的一部分。


###不同之处
eg:分别给自定义的`User`类添加Category,Extension

```
//Category
@interface User (InnterAdd)
-(void)test;
@end
@implementation User (InnterAdd)

-(void)test
{
    NSLog(@":%s",__func__);
}

@end

//Extension
@interface User ()

@property (copy)NSString * addr;

@end

```

表现形式上，Category有名称，有自己的.m文件，extension没有名称看起来想一个匿名的分类，没有.m文件。 Category也可以没有名称，这样在表现上就一样了。

`Category` 是在运行时加载处理的，这个过程中把添加的方法添加到类的方法列表中，由于类的内存结构布局是在编译时期确定的，所以在运行时不能再添加实例变量否则就破坏了内存结构（访问非法内存地址存在不可预知的错误，所以是禁止的）。那为啥能添加方法而不能添加成员变量呢？在runtime层，category用结构体category_t（在objc-runtime-new.h中可以找到此定义）如下：

```
typedef struct category_t {
    const char *name;
    classref_t cls;
    struct method_list_t *instanceMethods;
    struct method_list_t *classMethods;
    struct protocol_list_t *protocols;
    struct property_list_t *instanceProperties;
} category_t;
```
从中可看出可以添加实例方法，类方法，甚至可以实现协议，添加属性。没有表示实例变量的存贮。

`Extension`在编译时处理的，添加的变量和方法和类密切的在一起，类消失它也消失。


#####以上就是Category和Extension基础介绍，若要继续深入就得进一步研究runtime了。还在继续学习中。

//////

```
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
objc_getAssociatedObject(id object, const void *key)
void objc_removeAssociatedObjects(id object)

```



