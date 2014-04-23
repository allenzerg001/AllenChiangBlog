---
layout: post
title: objective-c的class和metaclass
---
#### Class & id 的定义

首先看看在objective-c中对于class和object的定义，这个在/usr/include/objc/objc.h 和 runtime.h中可以找到：

	typedef struct objc_class *Class;
	typedef struct objc_object {
     	Class isa;
	} *id;
	
class是一个objc_class的指针，id（也就是对象）是一个指向objc_object结构体的指针，所以每个对象都有一个纸箱class（objc_class）的isa指针；

#### objc_class 的定义

那么objc_class又是怎样的呢？

	struct objc_class
	{
    	struct objc_class* isa;
	    struct objc_class* super_class;
	    const char* name;
	    long version;
	    long info;
    	long instance_size;
	    struct objc_ivar_list* ivars;
    	struct objc_method_list** methodLists;
	    struct objc_cache* cache;
	    struct objc_protocol_list* protocols;
	};
	
可以看到class又有一个指向class的isa指针，这个class就是我们俗称的metaclass，也就是类对象的isa指向的就是metaclass；

metaclass跟class的区别在于metaclass存储类static开头的方法和static开头的成员（+开头的方法）；class存储类的实例方法和实例变量（-开头的）；

#### 这里应用一张别人的图来说明class、metaclass、superclass几者之间的关系就一目了然

![metaclass](/AllenChiangBlog/public/upload/2014-04-23-01.gif)

1. 类的实例对象的isa指向类，类的isa指向类的metaclass，类的metaclass指向基类的metaclass
2. 类的superclass指向父类，类的metaclass的superclass指向该类的superclassmetaclass
3. 基类的superclass指向nil，基类的metaclass指向基类自己

#### 说完了isa和superclass的关系，我们再说一下objc_class的其他属性

1. name：一个c字符串，指向类的名称。我们可以在运行期，通过这个名称查找到该类（通过：id objc_getClass(const char *aClassName)）或者该类的metaclass（id objc_getMetaClass(const char *aClassName)）
2. version：类的版本信息，默认初始化为0.我们可以在运行期对其进行修改（class_setVersion）或后去(class_getVersion)
3. info：标示类的类型，比如是metaclass还是class
4. instance_size：该类的实例变量大小
5. ivars：实例变量
6. methodLists：如果上面info说明当前是class，这里存储实例方法，如果是metaclass，这里存储类额静态方法
7. cache：指向objc_cache的指针，用于对方法的缓存
8. protocols：指向objc_protocol_list的指针，存储该类要遵守的协议
9. 