---
layout: post
title: "为已有的viewController绑定xib"
date: 2014-12-25 22:14:49 +0800
comments: true
categories: 
---

最近想把以前手写布局的项目改为使用autolayout，需要为已有的viewcontroller绑定xib，最方便的做法就是新建一个viewcontroller勾选上xib。再相关代码移过去。不过我尝试使用直接新建一个empty xib文件，绑定上已有的viewcontroller类的方法。

<!--more-->

首先新建一个空的xib文件，将xib文件名和viewcontroller类名保持一致。

![](/images/bindingxib/empty-xib.png)

然后尝试下拖入一个viewcontroller，并且将 file owner 的 custome class改为viewcontroller类名。

![](/images/bindingxib/add-viewcontroller.png)

勇敢的运行，结果华丽丽的出错了。=_=!
错误信息是：
>
Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: '-[UIViewController _loadViewFromNibNamed:bundle:] loaded the "DealDetailInfoController" nib but the view outlet was not set.'

是因为没有绑定 view ,连线，再试~
又错了。。。
>
A view can only be associated with at most one view controller at a time!

soga。是因为view已经绑定拖进来的viewcontroller。于是乎，将这个viewcontroller控件删掉，重新拖进来一个UIView控件。

![emptyxib](/images/bindingxib/addview.png)

再连接 file owner 的 view 。

![emptyxib](/images/bindingxib/setview.png)

运行，Bingo! OK 了。