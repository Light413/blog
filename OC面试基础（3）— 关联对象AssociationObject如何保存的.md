
目的是主要分析在runtime中关联对象操作是如何实现的，数据对象时如何保存的。

###1、代码举例
给一个已知的User类添加分类，添加一个`name`属性并关联

```
	#import "User.h"
	
	@interface User (Add)
	@property(nonatomic,copy)NSString * name;
	@end

	@implementation User (Add)
	-(void)setName:(NSString *)name
	{
	    objc_setAssociatedObject(self, @selector(name), name, OBJC_ASSOCIATION_COPY_NONATOMIC);
	}
	
	-(NSString *)name
	{
	  return   objc_getAssociatedObject(self, @selector(name));
	}
	@end

```

查看runtime编译后的c++代码 ： 打开相应的文件路终端径运行`clang -rewrite-objc User+Add.m`得到一c++文件，去除一些头文件的引入和很多不知道干嘛用的结构体定义，简易代码如下:

```

	// @interface User (Add)
	// @property(nonatomic,retain)NSString * name;
	/* @end */
	
	typedef struct objc_method *Method;
	typedef struct objc_ivar *Ivar;
	typedef struct objc_category *Category;
	typedef struct objc_property *objc_property_t;
	typedef struct {
	    const char *name;
	    const char *value;
	} objc_property_attribute_t;
	
	
	// @implementation User (Add)
	static void _I_User_Add_setName_(User * self, SEL _cmd, NSString *name) {
	    objc_setAssociatedObject(self, sel_registerName("name"), name, OBJC_ASSOCIATION_COPY_NONATOMIC);
	}
	
	static NSString * _I_User_Add_name(User * self, SEL _cmd) {
	  return objc_getAssociatedObject(self, sel_registerName("name"));
	}
	
	// @end
	

```
以上看出runtime主要通过`objc_setAssociatedObject`进行对象关联处理的。

###2、runtime代码分析
我下载使用的[objc4-680.tar.gz](https://opensource.apple.com/tarballs/objc4/),打开工程编译报错缺少一些文件，不过不影响代码阅读。在`runtime.h`中找到了方法 `objc_setAssociatedObject`依次调用

* `objc_setAssociatedObject_non_gc(object, key, value, policy);`
* `_object_set_associative_reference(object, (void *)key, value, policy);`
在`_object_set_associative_reference`中 实现了关联操作。其代码如下

```
	void _object_set_associative_reference(id object, void *key, id value, uintptr_t policy) {
	    // retain the new value (if any) outside the lock.
	    ObjcAssociation old_association(0, nil);
	    id new_value = value ? acquireValue(value, policy) : nil;
	    {
	        AssociationsManager manager;
	        AssociationsHashMap &associations(manager.associations());
	        disguised_ptr_t disguised_object = DISGUISE(object);
	        if (new_value) {
	            // break any existing association.
	            AssociationsHashMap::iterator i = associations.find(disguised_object);
	            if (i != associations.end()) {
	                // secondary table exists
	                ObjectAssociationMap *refs = i->second;
	                ObjectAssociationMap::iterator j = refs->find(key);
	                if (j != refs->end()) {
	                    old_association = j->second;
	                    j->second = ObjcAssociation(policy, new_value);
	                } else {
	                    (*refs)[key] = ObjcAssociation(policy, new_value);
	                }
	            } else {
	                // create the new association (first time).
	                ObjectAssociationMap *refs = new ObjectAssociationMap;
	                associations[disguised_object] = refs;
	                (*refs)[key] = ObjcAssociation(policy, new_value);
	                object->setHasAssociatedObjects();
	            }
	        } else {
	            // setting the association to nil breaks the association.
	            AssociationsHashMap::iterator i = associations.find(disguised_object);
	            if (i !=  associations.end()) {
	                ObjectAssociationMap *refs = i->second;
	                ObjectAssociationMap::iterator j = refs->find(key);
	                if (j != refs->end()) {
	                    old_association = j->second;
	                    refs->erase(j);
	                }
	            }
	        }
	    }
	    // release the old value (outside of the lock).
	    if (old_association.hasValue()) ReleaseValue()(old_association);
	}

```
我第一次看到以上代码直接蒙圈，都是一些c++的语法，不知道具体都是干啥的。静下来一行一行的仔细看可以推测出其大概处理流程。关联的对象保存在一个hash表中，只是这个hash表有点深，大表套小表，表中还有表一层一层的相关联。可以描述为：一个系统级别的主表1->表2->表3->封装后的属性和要关联的value。
最后通过`object->setHasAssociatedObjects();`标记对象已有关联。


大致流程如下图：
![]()



#####名词解释：

* AssociationsManager 类似于一个单例对象，保存着整个系统的关联对象数据。包含有一个多线程操作的锁和AssociationsHashMap的表。
* AssociationsHashMap 保存的对象的地址（一个类对象）和这个类全部关联的对象的hash table.
* ObjectAssociationMap 一个类全部关联的对象，key为索引。
* ObjcAssociation 保存的最小结构单元数据，要关联的value，和关联策略。
* `ObjectAssociationMap::iterator j = refs->find(key);` c++中的迭代操作，遍历对象中的key-value.
* `old_association = j->second;` 获取j对象的第二个数据，也就是`ObjcAssociation`对象。

#####关联对象的释放：

根据关联对象的存储结构我们可以知道，如果要释放一个对象的关联的对象也需要从hash 表中一层一层的给找出来，依次释放。释放操作是在被关联的对象释放时进行的。所以最终也是在`dealloc`方法中调用`void _object_remove_assocations(id object)`实现的。
依次调用顺序如下：

	1.	- (id)free
	2.id (*_dealloc)(id) = _object_dispose;
	3. id object_dispose(id obj)
	4. void *objc_destructInstance(id obj) 
	5. void _object_remove_assocations(id object)

###总结
以上皆为runtime关联对象如何保存的分析总结，可能有理解的不到位的地方，还在研究中。



