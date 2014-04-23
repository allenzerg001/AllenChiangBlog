---
layout: post
title: -layoutSubviews什么时候被调用
---

原文：http://blog.logichigh.com/2011/03/16/when-does-layoutsubviews-get-called/

### So the results is below:
1. init does not cause layoutSubviews to be called (duh)
2. addSubview causes layoutSubviews to be called on the view being added, the view it’s being added to (target view), and all the subviews of the target view
3. setFrame intelligently calls layoutSubviews on the view having it’s frame set only if the size parameter of the frame is different
4. scrolling a UIScrollView causes layoutSubviews to be called on the scrollView, and it’s superview
5. rotating a device only calls layoutSubview on the parent view (the responding viewControllers primary view)
6. removeFromSuperview – layoutSubviews is called on superview only (not show in table)

### 翻译如下

1. init不会引起layoutSubviews的调用；
2. addSubview会引起父view以及该父view所有子view的layoutSubviews方法的调用；
3. setFrame会在frame发生变化的时候引起layoutSubview的调用；
4. 滚动一个UIScrollview会引起该UIScrollview以及其父view的layoutSubviews的调用；
5. 滚动设备会引起当前viewController的view的layoutSubviews调用；
6. removeFromSuperview会调用父view的layoutSubviews；