---

title: CocoaPods 使用介绍
date: 2016-07-19 19:43:53
comments: true
categories: CocoaPods

---

CocoaPods 是 iOS 开发中常用的包管理工具，本文总结了 CocoaPods 的使用步骤，以及搭建一个私有 CocoaPods repo 的过程。

<!--more-->

### 安装

  因为被墙，需要对ruby源做一个替换操作，移除默认源添加淘宝的ruby源，否则的话安装很可能会失败。
   
``` shell
gem sources -r https://rubygems.org/
gem sources -a https://ruby.taobao.org/

#确认一下:
gem sources -l

#校验成功后执行安装操作：
sudo gem install cocoapods（默认最新版本）
sudo gem install cocoapods -v 0.35 (安装指定版本的cocoaPods)

#安装完成后 通过
pod --version 校验安装OK并查看安装的版本号
```

### 使用

常见操作是 使用第三方开源的代码：
想安装AFNetworking的最新版本，那么执行下搜索：

```	
pod search AFNetworking  (搜索AFNetworking的库的版本)
```

如果是第一次执行，会首先把Cocoapods master repo clone 下来,也可以手动执行 pod setup.查看的方式是 cd ~/.cocoapods/repos）

使用文本编辑器创建一个Podfile文件

```	
vi Podfile
```

编辑内容
		
```
pod 'AFNetworking','~>2.5.0'
#使用版本的说明：
#> 0.1 Any version higher than 0.1.
#>= 0.1 Version 0.1 and any higher version.
#< 0.1 Any version lower than 0.1.
#<= 0.1 Version 0.1 and any lower version.
#~> 0.1.2 Version 0.1.2 and the versions up to 0.2, not including 0.2. 
```

保存退出Podfile文件，之后执行 

```
Pod install
```

安装完成后会有提示，例如：

```	
[!] Please close any current Xcode sessions and use `CocoaPodsDemo.xcworkspace` for this project from now on.

```

注意提示使用**工程名.xcworkspace**文件


### 创建私有的 spec repo

pod的库管理可以看~/.cocoapods/repos 文件夹,
这个文件夹包含了所有能用到的被管理的库 默认只有 cocoapods 的 spec。

内部的代码不能共享，所以不能够push到 cocoapods 的 sepc 中，只能创建自己的私有的 spec 仓库。

1. 首先创建 spec 的 git repo。如:`git@github.com:zerdzhong/SpecName.git`
	
2. 将这个 spec 的 git repo 添加到 cocoapods 中。
	
	```
	pod repo add SpecName git@github.com:zerdzhong/SpecName.git
	```
	可以在下面的路径内查看。这个 repo 里面其实只是存放了每个库提交的podspec文件。

	```
	cd ~/.cocoapods/repos/SpecName
	pod repo lint . # 验证一下
	```
	对团队而言，每个人的电脑都需要配置这个私有库的过程（既第二步)执行完这一步原有的~/.cocoapods/repos 文件夹下就增加了一个叫做 SpecName 的文件夹,这样才能将创建的 pod 提交到这个 spec 中。


私有仓库有了，下面创建自己的pod。

### 创建Pod

pod 最主要的文件就是 .podsepc  文件，这个文件定义了这个 pod 中的源码的位置，依赖项，等等。可以通过 pod 提供的工具直接创建模板如下：

* pod spec create NAME
	
	通过终端找到自己要封装成库的工程，在 工程名.workspace 目录层级执行则创建了 工程名.podspec 文件，该文件默认给你自动生成了不少东西
	改一下podspec文件 注意所有的注释都要删除

	
* pod lib create NAME

	需要根据提示选择，选完后会生成一个工程+pod
这种情况的话 类文件放到Pod文件夹的Assets文件夹下，图片放到Pod文件夹的Assets文件夹下
	
	然后通过git add / git commit/ git remote add origin [url]/git push origin master 完成对远程git仓库的提交


写完podspec后,需要简单验证下

```
pod lib lint --allow-warnings(检测你的pod是否OK  如果有error是不可以的)
```
	
提交

```	
pod repo push SpecName x.podspec --allow-warnings
```
	

### 使用
和使用开源的 pod 一样，在 Podfile 中写明对于 Pod 库的依赖。

```
pod install --no-repo-update 安装的时候不再更新这个库
pod update 
```
	
### 常见问题
* 自己创建的库的互相引用问题，只要引用的版本是一样的就不会有问题。
* 库的版本管理，每一次的修改对应的版本的修改，修改完后需要打tag 当然直接指向分支的修改也是可以的，但是不便于查找某一个版本的问题
* 库升级以后的使用这个库的Podfile文件需不需要修改？看怎么写得，如果是指定写死的版本可能需要注意，如果使用了~>则看版本号的情况
* 如果是测试阶段，那么引用的路径需要指向本地的路径。测试完成后 要记得修改回来
* 针对图片资源和js资源的处理
* 库之间的依赖处理
* podfile上要写明source 地址
* 只想用库的一部分
	
### 相关链接

* [cocoapods 介绍](https://www.objc.io/issues/6-build-tools/cocoapods-under-the-hood/)

* [官方guide](https://guides.cocoapods.org/)

* [创建Pod](https://guides.cocoapods.org/syntax/podspec.html)
