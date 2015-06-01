---
layout: post
title: "View Programming Guide for iOS"
date: 2014-12-26 21:41:10 +0800
comments: true
categories: 
---

###Views

因为view对象是app和用户交互的主要方式，view负责很多工作，比如：

* subview的管理和布局

	* view 定义了自身的 resizing 行为
	* view 管理自己的 subviews
	* 在需要的时候 view 可以重置子控件的大小和位置
	* view 可以把一个点的位置从自身坐标系成其他 view 或 window 的坐标系下的坐标。
* 绘制和动画
	* view 在一个矩形区域内绘制自身的内容
	* view 中的某些属性可以通过动画的形式从一个值变到另一个值。
* 事件处理
	* view 可以接收到触摸事件
	* view 是响应者链中的一节

###Create and Configuring View Objects

可以通过代码和 Interface Builder 两种方式得到试图对象。然后可以通过集成到 view hierarchies(体系)在项目使用。（意思应该是通过 view 组合成项目中需要的自定义view)

####使用IB
在 IB 中通过拖拽控件的方式集成 views 是最简单的方式。


