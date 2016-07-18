---

layout: post
title: "Swift API Design Guidelines"
date: 2016-07-08 15:00:34 +0800
comments: true
categories: Swift

---

### Swift API Desgin Guideline

#### 原因

每个语言都有它独特的性格，让人看一眼代码就能分辨出来的。
		
```
	DispatchQueue.main.async {
		self.listDocumentsViewController?.present(signedOutController, animated: true)
	}
```

<!--more-->
		
类似上面这种代码给人的感受是 Swift 希望在整个语言层面都能达到的，但是 Cocoa 的 guideline 是基于 Objective-C 的语言风格来的。
	
#### 基本原则
	
[原则](https://swift.org/documentation/api-design-guidelines/)：
	
1. Clarity at the point of use
	
	让 API 在使用的时候很清晰准确是第一位的，函数和属性声明一次，但是要到处使用。设计 API 就是要在使用的时候感觉简洁明了；使用 API 的时候都要有上下文环境。
	
2. Clarity is more important than brevity
	
	不要把代码整洁作为优先级第一的目标，代码的简洁更多是 Swift 3 静态强类型语言的自然结果。
	
	![](/images/swift3/clarity1.png)
	![](/images/swift3/clarity2.png)
	
	判断接口里的单词是否冗余，取决于这个单词是否帮助理解了。比如下面的的 `addChild(_:at)` 接口， child 这个单词表明了是要把view 加到层级接口的子级中，而第二个参数的 point 可以通过参数的 CGPoint 类型看出来肯定是个位置，所有 atPoint 就删去了 point。
	
	```
	mainView.addChild(sideBar, atPoint: origin)
	mainView.addChild(sideBar, at: origin)
	
	```
	
	
3. Write a documentation comment
	
	Swift 3 注释支持 markdown 的变种。可以写出结构更丰富的注释。[文档摸我](https://developer.apple.com/library/mac/documentation/Xcode/Reference/xcode_markup_formatting_ref/)
	
	![](/images/swift3/document.png)

#### 命名
	
1. 保留足够的单词避免歧义
	
	```
	extension List {
		public mutating func remove(at position: Index) -> Element
	}
	employees.remove(at: x)
	```
	
2. 删去不需要的单词
	
	```
	override func numberOfSectionsInTableView(tableView: UITableView) -> Int
	override func numberOfSections(in tableView: UITableView) -> Int
		
	func viewForZoomingInScrollView(scrollView: UIScrollView) -> UIView?
	func viewForZooming(in scrollView: UIScrollView) -> UIView?
	
	```
	
3. 参数属性关联类型的命名根据角色来，而不是类型

	```
	var string = "Hello"
	protocol ViewController {
		associatedtype ViewType : View
	}
	class ProductionLine {
		func restock(from widgetFactory: WidgetFactory)
	}
	```
	上面的代码中命名就不是建议的按照角色来命名的，推荐的命名如下：
		
	```
	var greeting = "Hello"
	protocol ViewController {
		associatedtype ContentView : View
	}
	class ProductionLine {
		func restock(from supplier: WidgetFactory)
	}
	```
	如果属性的角色和protocol类型一样，可以加个 Type 关联下。
	
	```
	protocol Sequence {
		associatedtype IteratorType : Iterator
	}
	```
	
4. 为弱类型补充额外的信息
	
	```
	func addObserver(_ observer: NSObject, forKeyPath path: String)
	grid.addObserver(self, forKeyPath: graphics) // clear
	```

#### 调用的流畅
	
1. 调用要尽量符合英语语法
	
	```
	x.insert(y, at: z)          “x, insert y at z”
	x.subViews(havingColor: y)  “x's subviews having color y”
	x.capitalizingNouns()       “x, capitalizing nouns”
	```
	
2. 工厂方法要以 make 开始
	
	```
	x.makeIterator()
	```
	
3. 初始化方法和工厂方法的短语应该不包括第一个参数
	
	```
	let foreground = Color(red: 32, green: 64, blue: 128)
	let newPart = factory.makeWidget(gears: 42, spindles: 14)
	```
	反例：
	
	```
	let foreground = Color(havingRGBValuesRed: 32, green: 64, andBlue: 128)
	let newPart = factory.makeWidget(havingGearCount: 42, andSpindleCount: 14)
	```
	
4. 命名函数要根据 side-effects
	
	* 没有副作用的读起来应该是名词短语，比如 `x.distance(to: y)`, `i.successor()`
	* 有副作用的应该是祈使句动词短语,比如 `print(x)`,`x.sort()`,`x.append(y)`
	* Mutating/nonmutating 方法要配对一致
		
		* 用动词描述的操作，应该用动词的祈使语气添加 ing 或 ed 后缀表示 mutating 方法 
		![](/images/swift3/mutating1.png)
	
		* 用名次描述的操作，应该添加 form 前缀来表示 mutating 方法 
		![](/images/swift3/mutating2.png)
	
5. 返回 boolean 的方法和属性，应该读起来像断言一样。比如：`x.isEmpty`
6. 描述性质的协议应该起名次的名字。比如 `Collection`
7. 描述能力的协议应该用后缀 able,ible 或者 ing,比如 `Equatable`,`ProgressReporting `
8. 其他的类型，属性，变量，常量都应该是名词
	

### The Grand Renaming

将这套 API Guidelines 改进到更大范围的 API 中
	
* Swift 标准库
* 所有 Cocoa 和 Cocoa Touch 框架
* Core Graphics 和 GCD 框架

一个API，两个名字，在 Objective-C 代码中通过 NS\_SWIFT\_NAME 声明一个别名给 Swift 使用，在 Swift 代码中通过 @objc 声明 Obejctive-C 函数名。

![](/images/swift3/rename.png)

在 Objective-C 中有很多 NSString 的常量，用来表示一种弱类型, 通过 NS\_EXTENSIOBLE\_STRING\_ENUM 自动处理成强类型。 

![](/images/swift3/wraptype.png)

\#selector,\#keyPath 来替换原先的 String

 C API，比如 Core Graphics 中的 C API 都不是 Swift 风格的，也是同样的方法通过 NS\_SWIFT\_NAME  进行了大量的重命名
 
```
let ctx = UIGraphicsGetCurrentContext()

let rectangle = CGRect(x: 0, y: 0, width: 512, height: 512)
CGContextSetFillColorWithColor(ctx, UIColor.redColor().CGColor)
CGContextSetStrokeColorWithColor(ctx, UIColor.blackColor().CGColor)
CGContextSetLineWidth(ctx, 10)
CGContextAddRect(ctx, rectangle)
CGContextDrawPath(ctx, .FillStroke)

UIGraphicsEndImageContext()
```

```
if let ctx = UIGraphicsGetCurrentContext() {
    let rectangle = CGRect(x: 0, y: 0, width: 512, height: 512)
    ctx.setFillColor(UIColor.red().cgColor)
    ctx.setStrokeColor(UIColor.black().cgColor)
    ctx.setLineWidth(10)
    ctx.addRect(rectangle)
    ctx.drawPath(using: .fillStroke)

    UIGraphicsEndImageContext()
}
```
 
### 总结

1. 所有的参数说明都默认显示了
2. 很多API删去不必要的单词
3. Enum的值和属性都统一用小写驼峰命名
4. Core Graphics 和 GCD 的 C API 都更新命名了