---
layout: post
title: 适配3d-touch
---

## 适配3d-touch

3d-touch含有3种feature功能，压力感应（Press Sensitivity）、Peek和Pop手势、快捷方式（Quick Actions）

需要xcode7 GM和iOS9以上版本才能支持开发该功能

官方文档说明模拟器还不支持3d-touch的功能，不过github真是牛人比较多，也有解决办法模拟器调试quick action功能。

[https://github.com/DeskConnect/SBShortcutMenuSimulator](https://github.com/DeskConnect/SBShortcutMenuSimulator)

### 3d-touch & Force-touch

你之前可能在iWatch上听说过类似的技术叫Force-touch，那跟现在的3d-touch一样的，还是有什么区别呢？

相对Force-touch来说，3d-touch有更灵敏的感应度，更快的响应速度。

另外Force-touch只有轻点和轻按，3d-touch多了重按。

![](/public/upload/2015-09-21-touch-compare.png)

### 快捷方式

![](/public/upload/2015-09-21-actions_gallery_3_large_2x.jpg)



申明Quick Action有两种方式：静态和动态

静态是在plist文件中申明，动态则是在代码中注册，系统支持两者同时存在。

-系统限制每个app最多显示4个快捷图标，包括静态和动态

### 静态

在app的plist文件中增加如下申明：

	<key>UIApplicationShortcutItems</key>
	<array>
	<dict>

		<key>UIApplicationShortcutItemIconType</key>
		<string>UIApplicationShortcutIconTypeSearch</string>

		<key>UIApplicationShortcutItemSubtitle</key>
		<string>shortcutSubtitle1</string>

		<key>UIApplicationShortcutItemTitle</key>
		<string>shortcutTitle1</string>

		<key>UIApplicationShortcutItemType</key>
		<string>First</string>
 
    	<key>UIApplicationShortcutItemUserInfo</key>
    	<dict>
        <key>firstShorcutKey1</key>
        <string>firstShortcutKeyValue1</string>
	   </dict>

	</dict>
	</array>



`UIApplicationShortcutItemType`：其实是快捷方式的id

`UIApplicationShortcutItemTitle`：标题

`UIApplicationShortcutItemSubtitle`：副标题（什么时候会显示暂时我也不知道，因为还没有真机可以模拟）

`UIApplicationShortcutItemIconType`：图标

系统自带7种图标样式分别是：Compose,Play,Pause,Add,Location,Search,Share，在plist配置的时候应该就是前面加UIApplicationShortcutIconType了，比如上面UIApplicationShortcutIconTypeSearch

`UIApplicationShortcutItemIconFile`： 自定义图标，上面IconType定义将被忽略。图标格式35x35像素单色。

`UIApplicationShortcutItemUserInfo`：额外信息

### 动态

`UIApplication`对象多了一个支持快捷方式的数组

	@property(nonatomic, copy) NSArray <UIApplicationShortcutItem *> *shortcutItems

如果需要增加快捷方式，可以赋值给`shortcutItems`属性。

如果静态和动态方式同时使用的话，赋值的时候也不会覆盖静态方式申明的快捷方式，不用担心。

`UIApplicationShortcutItem`对象init方法


	(instancetype)initWithType:(NSString *)type 
          localizedTitle:(NSString *)localizedTitle
       localizedSubtitle:(NSString *)localizedSubtitle
                    icon:(UIApplicationShortcutIcon *)icon
                userInfo:(NSDictionary *)userInfo`
                

各个字段跟上面静态方式申明的时候意义一样。

其中定义快捷方式图标`UIApplicationShortcutIcon`，有3种init方法：

`

	// 根据系统预置图标类型
	-(instancetype)iconWithType:(UIApplicationShortcutIconType)type

	// 根据app bundle里面的图标名称自定义
	-(instancetype)iconWithTemplateImageName:(NSString *)templateImageName

	// 根据通讯录联系人名称
	-(instancetype)iconWithContact:(CNContact *)contact`



## 处理Quick Action

### 冷启动

顾名思义即app在没有后台进程情况下启动

demo中的做法比较值得推荐，在不打断原有应用启动顺序的情况下处理快捷方式。



-其实这里有一个疑问，

	application:performActionForShortcutItem:completionHandler:

在UIApplicationDelegate的生命周期中处于哪个环节？

### 热启动

	application:performActionForShortcutItem:completionHandler:
	
UIAppDelegate类的上述方法中处理快捷方式，打开对应的页面

处理成功或者失败调用block：`completionHandler(YES)`

#### 参考文档

[Add iOS 9’s Quick Actions shortcut support in 15 minutes right now !](http://www.stringcode.co.uk/add-ios-9s-quick-actions-shortcut-support-in-15-minutes-right-now/?utm_campaign=iOS%2BDev%2BWeekly&utm_medium=email&utm_source=iOS_Dev_Weekly_Issue_216)

[3d-touch官方文档](https://developer.apple.com/ios/3d-touch/)

[15分钟搞定iOS9 Quick Actions](http://mp.weixin.qq.com/s?__biz=MjM5NDMzMTcxMg==&mid=212175593&idx=1&sn=887118aaa63d4d364ccf16be9e807a72#rd)