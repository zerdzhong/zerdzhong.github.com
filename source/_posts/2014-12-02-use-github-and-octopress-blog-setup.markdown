---
layout: post
title: "使用github和octopress搭建博客"
date: 2014-12-02 17:15:04 +0800
comments: false
categories:
---
这块地方的第一篇博客就记录下这个博客站点诞生的过程吧，使用Github和Octopress搭建博客，已经不是新鲜事了，主要是利用Github pages的功能作为博客的服务端，使用Octopress作为博客管理工具。

<!--more-->

###搭建步骤

* Ruby
	
	我的Mac上原来装过Ruby,但版本过低，使用rvm安装了最新的稳定版。在终端如下命令来安装rvm:
	
	```python
	curl -L https://get.rvm.io | bash -s stable --ruby  
	```

	执行以下命令查看最新版本:
	
	```python
	rvm list known 
	```	
	知道版本号之后，就可以用 rvm install 2.1（版本号） 安装ruby了。
	
	
* Octopress
   
   1.下载Octopress
   
   如果已经安装了git的话，可以直接运行下面的命令：

   ```python
	  git clone git://github.com/imathis/octopress.git octopress  
   ```

   如果没有安装的话，也可以直接去github的网站上直接下载zip包。
   
   2.安装依赖
   
   使用bundle安转项目依赖，如果没有安装过bundler，可以使用下面的命令安装：

	```python
	sudo gem install bundler
	```
	
	使用下面的命令安装依赖：
	
	```python
	bundle install
	```	
 
   

3.配置Github

4.部署Octopress到Github


