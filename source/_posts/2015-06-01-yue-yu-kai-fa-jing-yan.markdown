---
layout: post
title: "越狱开发经验小结"
date: 2015-06-01 23:45:32 +0800
comments: true
categories: 越狱开发
---


### 越狱程序开发

iOS 和 Mac OS X 公用同样的系统内核（Darwin Unix），提供基本一致的底层系统API，甚至连应用程序的文件结构都基本类似。抛开上层 GUI 来看，iOS 和 Mac OS X 可以看作同一个系统，越狱获取了设备的管理员权限，拥有了读写所有文件的能力，Mac OS X 上的命令行工具都可以安装使用。

<!--more-->

越狱开发相比于上线 App 开发的区别主要就是存在多种不同形式的 “App”（tweak,command-line tool,preference_bundle…）也可以做一些上线 App 不能做的事情，比如以 root 权限运行，调用私有 API，Hook 其他的 APP。

#### Method Swizzling(hook)

Method Swizzling 是利用 Runtime 特性把一个方法的实现与另一个方法的实现进行替换。只 hook 自己 App 中的方法，并不会影响 App 上线。

每个类里都有一个 Dispatch Table ，将方法的名字（SEL）跟方法的实现（IMP，指向 C 函数的指针）一一对应。Swizzle 一个方法其实就是在程序运行时在 Dispatch Table 里做点改动，让这个方法的名字（SEL）对应到另个 IMP 。

```
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        
        SEL originalSelector = @selector(paymentWithProduct:);
        SEL swizzledSelector = @selector(swizzling_paymentWithProduct:);
        
        //        Class class = [self class];
        //        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        //        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
        
        // When swizzling a class method, use the following:
        Class class = object_getClass((id)self);
        Method originalMethod = class_getClassMethod(class, originalSelector);
        Method swizzledMethod = class_getClassMethod(class, swizzledSelector);
        
        BOOL didAddMethod = class_addMethod(class,
                                            originalSelector,
                                            method_getImplementation(swizzledMethod),
                                            method_getTypeEncoding(swizzledMethod));
        
        if (didAddMethod) {
            class_replaceMethod(class,
                                swizzledSelector,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

#pragma mark- swizzling 
+ (instancetype)swizzling_paymentWithProduct:(SKProduct *)product
{
    id res = [self swizzling_paymentWithProduct:product];
    
    NSLog(@"swizzling payment");
    
    return res;
}


```

#### tweak

tweak 是越狱 iOS 中的重要组成部分，它的作用相当于插件，主要是通过 hook 住宿主应用的方法来提供一些额外的功能，比如通话录音，短信拦截…由于 Method Swizzling 只能作用于本进程的运行时环境，所以 Tweak 想要 hook 宿主应用就需要通过 dylib 的形式被于宿主应用 load 成为宿主应用的一部分，

MobileSubstrate（Cydia Substrate）是绝大部分 tweak 工作的基础，是一个提供了 Runtime 的 hook 其他方法的 framework。

MobileSubstrate 主要由三个部分组成，MobileHooker,MobileLoader 和 Safe mode。

* MobileHooker

	MobileHooker 的作用是 Hook 函数，主要包含以下两个函数:

	```
	void MSHookMessageEx(Class class, SEL selector, IMP replacement, IMP *result);

	void MSHookFunction(void* function, void* relpacement, void** p_original);
	```

	MSHookMessageEx 作用于 Objective-C函数， MSHookFunction 作用于 C 和 C++ 函数，在进程执行到原始的 function 的时候，转而执行 replacement ,并保存原始 function 的指令和返回地址，可以选择是否执行原始的 function 。

* MobileLoader
	
	MobileLoader 会在 App 运行的时候通过 DYLD\_INSERT\_LIBRARIES 环境变量运行起来，加载 /Library/MobileSubstrate/DynamicLibraries/	 路径下的所有合适的动态库。
	
	```
	// The attribute forces this function to be called on load.
	__attribute__((constructor))
	static void initialize() {
	  NSLog(@"MyExt: Loaded");
	  MSHookFunction(CFShow, replaced_CFShow, &original_CFShow);
	}
	```
	
	通过 \_\_attribute\__\((constructor)) 将函数在 dylib 加载的时候执行， tweak 就是通过这种方式 hook 宿主应用的方法。
		
* Safe mode
	由于 tweak 可以 Hook 任意的方法，如果 tweak 的对象是 SpringBoard 时，由于 tweak 的奔溃导致 SpringBoard 奔溃时，MobileLoader 会 catch 住这个 crash ,进入 safe mode,在 safe mode 下不会加载任何 tweak。
	
#### daemon

iOS 源于 OSX ，和所有类 UNIX 操作系统一样有 daemon （守护进程），越狱之后放开了苹果的限制之后，可以实现自己的 daemon。iOS 上的 daemon 由一个可执行文件和一个 plist 文件组成。 iOS 的根进程时 launchd ，它会在开机的时候检测 /System/Library/LaunchDaemons 和 /Library/LaunchDaemons 路径下的所有符合规定的 plist 文件。这个 plist 文件记录了 daemon 的基本信息。

```
{
  "Program" => "/usr/libexec/adDaemon"
  "RunAtLoad" => 1
  "KeepAlive" => 1
  "Label" => "cn.domob.adDaemon"
}
```

#### THEOS

越狱的开发工具和 普通 APP 的开发不太一样，没法完全依赖 Xcode 。THEOS 就是一个功能简化版的越狱开发 Xcode。

THEOS 用内建的 Logos 语法对 MobileSubstrate 进行了封装。也提供了快速创建 tweak 和 daemon 的模板。

```
#import <SpringBoard/SpringBoard.h>

%hook SpringBoard

-(void)applicationDidFinishLaunching:(id)application {
	%orig;
	  
	UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Welcome"
		          									message:@"Hello Tweak"
				          						   delegate:nil
						          		  cancelButtonTitle:@"确定"
								          otherButtonTitles:nil];
	[alert show];
	[alert release];
}

%end

```

以上代码是 hook SpringBoard 的 applicationDidFinishLaunching 方法后弹出一个 alertView。


* 目前情况

	tweak 是以 dylib 的形式存在，受限于 tweak 的对象，如果 tweak 的对象是普通的 app ，那么权限很低，依赖性太强。如果 tweak 的是 springboard 就相对灵活，但是较为危险，导致 safe mode。
	
	daemon 以后台守护进程存在，没找到好的办法在 daemon 的进程中直接 hook 目标 app ，可以通过 cycript 间接的达到目的（代码不够灵活）。
	
	
###工具集

####静态分析工具

* dumpdecrypted 
	
	在 AppStore 上的 app 经过了苹果的一层加密，在可执行文件外包了一层，使用 dumdecrypted 工具进行砸壳之后，才能获取到原始的可执行文件。
	
* class-dump
	
	class-dump 利用 runtime 特性将 Mach-O 格式的二进制文件的头文件信息 dump 出来。
	
* idaq/Hopper
	
	反汇编器，可以用来查看二进制文件的汇编代码。
	
####动态分析工具

* cycript

	这个工具可以轻松的注入目标 app ，获取对象在内存中的地址，通过对象调用方法，获取数据。
	
* lldb + debugserver

	lldb 是运行在 OSX 中的调试器，和 debugserver 配合可以对 iOS 设备上的 app 进行 debug。

### 逆向目标 app 的主要思路

#### 演示

* 准备工作
	* 砸壳
	* class-dump
* 定位功能
	* cycript	 

#### 总结

1. 从界面上定位关心的功能，根据 view 层级定位到相应的类名。
2. 从相应类的头文件中寻找突破点，使用 cycript 快速验证。
3. 必要时使用 Hopper/idaq 看实现方法。



相关链接：

[method-swizzling](http://nshipster.com/method-swizzling/)

[cycript-manual](http://www.cycript.org/manual/)

[iphonedevwiki](http://iphonedevwiki.net/index.php/Main_Page)




	