---
layout: post
title: "iOS objc_msgSend 函数兼容64位"
date: 2014-12-04 23:14:27 +0800
comments: true
categories: 
---

32位设备上直接调用 objc_msgSend 函数没有问题。但是在64位设备上就无情的出问题了，搜索文档发现要要对函数类型进行转换<!--more-->，如下：

<!--more-->

```objective-c
int (*idAction)(id, SEL, id) = (int (*)(id, SEL, id)) objc_msgSend;
int (*doubleAction)(id, SEL, double) = (int (*)(id, SEL, double)) objc_msgSend;
int (*floatAction)(id, SEL, float) = (int (*)(id, SEL, float)) objc_msgSend;
int (*intAction)(id, SEL, int) = (int (*)(id, SEL, int)) objc_msgSend;
int (*longlongAction)(id, SEL,  long long) = (int (*)(id, SEL, long long)) objc_msgSend;
```

[官方文档](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaTouch64BitGuide/ConvertingYourAppto64-Bit/ConvertingYourAppto64-Bit.html)
