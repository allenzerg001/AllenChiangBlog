---
layout: post
title: Auto Layout - iOS.6.by.tutorials笔记
---
因为接下来的其实是原作者Matthijs Hollemans对于Auto Layout的进阶篇内容,所以建议先阅读一下作者另外两篇关于Auto Layout基础的
在著名的raywenderlish上面有，链接如下：

[Beginning Auto Layout in iOS 6: Part 1/2](http://www.raywenderlich.com/20881/beginning-auto-layout-part-1-of-2)

[Beginning Auto Layout in iOS 6: Part 2/2](http://www.raywenderlich.com/20881/beginning-auto-layout-part-2-of-2)


## 魔法程式

	A = B*m + c

	view1.attribute = view2.attribute * multiplier + constant
	
同时中间的等于号你也可以使用>=或者<=来代替

## 通过代码添加约束

通过NSLayoutConstraint的类方法来创建一个约束：

	+ (instancetype)constraintWithItem:(id)view1 
	 attribute:(NSLayoutAttribute)attr1 
	 relatedBy:(NSLayoutRelation)relation 
	    toItem:(id)view2 
	 attribute:(NSLayoutAttribute)attr2 
	multiplier:(CGFloat)multiplier 
	  constant:(CGFloat)c
	  
上面￼constraint语法格式的含义，参考上面提到的魔法程式

![image](/public/upload/2015-01-06-constraint.png)

* 禁止添加Autoresizing mask

	translateAutoresizingMaskIntoConstraints = NO;
	
* 只有当你添加一条约束的时候，Auto Layout才会生效

	[view addConstraint:constraint];

* 设置控件自身的宽度和高度约束

	button1.width = nil * 1.0f + 200.0f

简单来看就是	

	button1.width = 200.0f

* 约束的优先级

NSLayoutConstraint有一个属性名称：priority

	@property UILayoutPriority priority

	enum {
		UILayoutPriorityRequired = 1000,
		UILayoutPriorityDefaultHigh = 750,
		UILayoutPriorityDefaultLow = 250,
		UILayoutPriorityFittingSizeLevel = 50,
	};
	typedef float UILayoutPriority;	
	

可以看到这其实是一个float的数字，在iOS上应该介于0~1000之间。数字越大的约束将优先于数字小的生效。

部分系统控件默认带有保持自身固有尺寸的约束，比如button（个人感觉这里跟Android的wrapcontent有点像）。这个默认约束的优先级是250，也就是priority=250。所以如果你添加的控件自身尺寸大小的约束要大于250才能生效；另外系统也提供了一个方法来降低这个默认约束的优先级：

	setContentHuggingPriority:forAxis:

* 调试约束的私有api: _autolayoutTrace


## 添加约束的规则

1. 如果是一个view和他的superview的约束，添加到他的superview；
2. 是同一个superview下的两个view的约束，添加到这个superview；
3. 只是针对一个view的约束，添加给他自己。比如width=height之类的

## VFL = Visual Format Language

NSLayoutConstraint同样支持使用vfl来创建一个约束，目的是比上面的更简洁

	+ (NSArray *)constraintsWithVisualFormat:(NSString*)format 
	options:(NSLayoutFormatOptions)opts 
	metrics:(NSDictionary *)metrics 
	  views:(NSDictionary *)views

一些相对经典的vfl语句

`|-[deleteButton]-(>=8)-[cancelButton]-[nextButton]-|`

`|-5-[deleteButton]-(>=8)-[cancelButton]-30-[nextButton]-|`

`|-[deleteButton]-(>=8)-[cancelButton(120)]-[nextButton(30)]-|`

`|-[deleteButton(==nextButton@700)]-(>=8)- [cancelButton(==nextButton@700)]-[nextButton]-|`


* 其中的deleteButton/cancelButton/nextButton等元素名称需要通过NSDictionary传入，你可以使用系统提供的宏来创建这个NSDictionary:

`#define NSDictionaryOfVariableBindings(...) _NSDictionaryOfVariableBindings(@"" # __VA_ARGS__, __VA_ARGS__, nil)`

* 含有metric的vfl语句

`V:[nextButton]-distance-|`

其中的distance需要通过metrics dictionary传入

	constraints = [NSLayoutConstraint constraintsWithVisualFormat:@"V:[nextButton]-distance-|" 	options:0 
	metrics:@{ @"distance": @50 }	  views:viewsDictionary];`
## 动态布局
前面都是静态的自动布局，一旦确定了约束就确定了控件的位置，无法改变。
但是实际情况中，我们的控件往往是需要响应用户的事件，并展现出如位置、大小等的变化。以前我们大都通过setFrame等方式来实现，那伴随约束的情况下又如何实现呢？
为了便于对约束的管理和维护，我们应该把所有的约束统一的写在重载了UIViewController的updateViewConstraints方法，或者UIView的updateConstraints方法中。

	- (void)updateConstraints
	- (void)updateViewConstraints

 每当布局展现之前，系统会自动调用上述方法。
 
对于UIView你也可以通过手动调用- setNeedsUpdateConstraints方法，来告诉系统布局约束需要被更新，系统会在将来自动调用上面- updateConstraints方法。

但是对于UIViewController好像就没有手动触发的方法了，可能要直接调用- updateViewConstraints方法了。

## 动画约束

之前做动画可能都是通过设置frame的绝对位置来实现，但是在使用了Auto Layout以后该怎么实现动画效果呢？

有如下两种方法：

1.去除旧的约束，添加新的约束

我们通过删除旧的约束，添加新的约束来实现变化，至于普通的动画么，我们也可以使用简单的方法：

	[UIView animateWithDuration:0.3f animations:^{
		[self.view layoutIfNeeded];
	}

2.改变已有动画的常量值

还记得上面的方程式么？A = B*m + c

我们可以通过改变c值的大小来实现动画的效果

* -(void)setNeedsLayout和-(void)layoutIfNeeded方法的区别
* 类似的还有如：
	* -(void)displayIfNeeded / -(void)setNeedsDisplay
	* -(void)updateConstraintsIfNeeded / setNeedsUpdateConstraints
	
区别在于：xxxIfNeeded方法是立即执行，而setNeedsxxx则是延迟到UIKit的下一个空闲时间片，参考RunLoop的概念。

## 自定义view的固有尺寸

通过重写UIView的-(CGSize)intrinsicContentSize方法返回内部尺寸大小。则Auto Layout会使用这个尺寸大小来计算布局。

