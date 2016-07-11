---
layout: post
title: "iOS 多线程技术总结-1"
date: 2015-01-08 10:58:36 +0800
comments: true
categories: iOS Concurrency
---

多线程其实是为了实现并发执行，而且线程是并发执行多个代码路径的多种技术之中比较轻量级的一种（对应较重的实现是多进程）。<!--more-->

iOS & OS X 实现多线程的方法的以下几种方式：

- POSIX 线程模型，iOS & OS X 提供了一套底层的 C 语言的 POSIX 线程 API 来创建管理线程，除非考虑到跨平台，一般极少直接使用 POSIX 线程 API。

-  NSObject 提供的以 performSelector 为前缀的一系列方法。这一系列方法简单易用，但只提供几个选择：指定执行的方法（但传入方法的参数数量有限制）；指定是在当前线程，还是在主线程，还是在后台线程执行；指定是否需要阻塞当前线程等待结果。

- NSTread 是 iOS 和 OS X 都提供的线程对象，是线程的一个轻量级实现。NSThread 适合执行一些轻量级的简单任务。因为使用 NSThead 仍然需要手动管理线程生命周期，进行线程间同步。线程状态，依赖性，线程间同步等线程相关的主题 NSThread 都没有涉及。比如，涉及到线程间同步仍然需要配合使用 NSLock，NSCondition 或者 @synchronized。所以，遇到复杂任务时，轻量级的 NSThread 并不合适。

- NSOperation 做的事情比 NSThread 更多。继承 NSOperation 的子类获得一些线程相关的特性，可以安全地管理线程生命周期。比如，以线程安全的方式建立状态，取消线程。配合 NSOperationQueue，可以控制线程间的优先级和依赖性。

- GCD(Grand Central Dispatch) 提供了线程的高级抽象，GCD 将线程管理的操作集成进系统中，不需要程序员手动管理。将需要并发的代码通过 block 加入到合适的 dispatch queue 中，由 GCD 处理线程管理、调度等工作。

苹果官方推荐使用 GCD 或者 NSOperation 结合 NSOperationQueue 的方式实现并发。

## Operation Queues 

Operation Queues 是以 Operation 为中心。Operation 是 NSOperation 类的实例，是经过封装的 工作单元。NSOperation 是抽象基类，必须被子类继承才能使用。Foundation framework 中有两种已经实现好的子类为：NSInvocationOperation,NSBlockOperation。

Operation 支持以下特性：
Operation 对象之间支持基于图的依赖关系，这些依赖关系能保证 Operation 对象在所有依赖的 Operation 执行之后再执行。
支持可选的完成 block 在 Operation 的 main task 完成之后执行。
支持 KVO 监听 Operation 状态的改变。
支持优先级操作，影响 Operation 的相对执行顺序。
支持取消语法，允许停止一个正在执行的 Operation。

Operations 被设计用来帮助提高应用的并发级别，同时也是一个很好的将应用的行为包含在各个分离的块中方法。可以将一个或多个 Operation 对象提交到队列中，让一个或多个独立的线程异步执行相应的工作，而不是在应用程序的主线程上执行一堆的代码。

虽然通常执行 Operations 是通过将其添加到 Operation Queue 中，但是手动调用 start 方法也可以让 Operation 执行，这样做是否能够并发取决于在子线程中调用 start 方法，可以通过 isConcurrent 方法来判断。通常情况下对于调用 start 方法的线程来说 Operation 是同步的。

想实现对于调用线程来说并发的 Operation 需要一些额外的代码来异步的开始 Operation。例如，开辟额外的线程，调用异步的方法等。绝大多数情况是不需要这么做的，只要我们将同步的 Operation 加入到 Operation Queue 中，队列会自动为 Operation 创建一个线程，因此同步的 Operation 加入到队列中之后表现的和异步一样。

### NSInvocationOperation

NSInvocationOperation 是 NSOperation 的子类，当运行的时候，会调用指定对象的指定selector，使用这个类可以省去大量实现自定义 NSOperation 子类的代码，特别是对已有项目修改时已经有了可以执行具体任务的方法的情况。也可以实现根据情况来调用某个方法，比如可以使用 NSInvocationOperation 根据用户输入来动态执行一个 selector。

```
- (NSOperation * )taskWithData:(id)data {
	NSInvocationOperation* theOp = [[NSInvocationOperation alloc]initWithTarget:self selector:@selector(myTaskMethod:) object:data];
	
return theOp;
}

// This is the method that does the actual work of the task.
- (void)myTaskMethod:(id)data {
// Perform the task.
}
```

### NSBlockOperation

NSBlockOperation 是 NSOperation 的子类，是对一个或多个 block 对象的封装，这个类为已经使用 Operation Queue 但不想使用 dispatch queue 的 block 提供了面向对象的封装。也可以使用 block operation 实现 dispatch queue 不支持的依赖，KVO 等特性。

创建 block operation 一般使用最少一个 block 来初始化，初始化后可以添加其他的 block。当 NSBlockOperation 对象执行的时候，对象会把所有的 block 添加到默认优先级、并发的 dispatch queue 中。NSBlockOperation 对象会在所有 block 执行结束后返回，当最后一个 block 结束之后，block operation 对象会将自己标记位 finished。因此可以通过 block operation 来追踪一组 block 的执行。就像使用一个线程合并来自多个线程的返回值一样，区别在于 block operation 在一个子线程中执行，应用的其他线程不会因为 block operation 阻塞。
//sample
NSBlockOperation* theOp = [NSBlockOperation blockOperationWithBlock: ^{
     NSLog(@"Beginning operation.\n");
     // Do some work.
}];

在创建 block operation 对象之后可以通过 addExecutionBlock: 方法来添加更多的 block。如果需要顺序的执行 blocks ，需要直接使用 dispatch queue。

### 自定义 Operation

如果 NSBlockOperation 和 NSInvocationOperation 不满足需求，就需要直接实现 NSOperation 的子类添加所需要的行为。NSOperation 提供了一些基础的功能，处理了大部分依赖和 KVO 的工作。

定义一个同步的 operation 子类比实现一个异步的 operation 简单很多，对于同步的 operation 来说需要做的就是实现 main task 以及在适当的处理取消事件，其他工作父类已经处理了。对于异步的 operation 需要重写很多父类的方法。

#### 实现 main task

官方建议自定义的 Operation 类最少需要实现两个方法：init 和 main 。
init 方法用来初始化 operation 对象，并将需要的参数传给 Operation
main 方法用来执行自定义的操作。

```
//.h
@interface MyNonConcurrentOperation : NSOperation
@property id (strong) myData;
-(id)initWithData:(id)data;
@end

//.m
@implementation MyNonConcurrentOperation
- (id)initWithData:(id)data {
     if (self = [super init])
          myData = data;
     return self;
}
-(void)main {
     @try {
     // Do some work on myData and report the results.
     }@catch(...) {
     // Do not rethrow exceptions.
     }
}
@end
```

但是实际上 init 函数也不一定要实现，可以通过属性设置 operation 所需的数据，这样调用父类的 init 的函数即可。

总结一下 Operation 的关键特性：

- 所有必需的数据都要在开始之前传递给 operation（通过init 方法/设置属性等），这样能避免运行过程中和其他对象交互，避免了加锁。

- 操作完成后结果保存在自身 ivar中，让使用者在 completionBlock中根据 operation 的对象取出结果，这样也是避免加锁。

- main 方法中定期检查 isCancelled 属性，以便在接收到取消事件的时候能够退出。这点很重要。可以在循环中检查，在任何相对容易停止 operation 的地方都可以检查。

不管是自定义的 Operation 还是系统提供的两个 Operation 初始化之后，加入 operation queue 之前都可以对 operation 对象进行相关设置：
依赖，通过 addDependency: 函数为 operation 添加依赖的 operation，可以依次添加依赖成为联系执行的队列，也可以添加成复杂的图依赖，但是注意不要形成环。
优先级，setQueuePriority: 这只会所在的 queue 起作用，且优先满足依赖的情况下确定相对顺序。setThreadPriority: 必须在添加到队列之前调用，且只对 operation 中 main 方法中的代码起作用。
完成的 block，setCompletionBlock: 设置完成的 block 。可以在此block 中根据 operation 属性获取结果。

#### 添加到 operation queue 

```
NSOperationQueue* aQueue = [[NSOperationQueue alloc] init];
[aQueue addOperation:anOp]; // Add a single operation
[aQueue addOperations:anArrayOfOps waitUntilFinished:NO]; // Add multiple operations
[aQueue addOperationWithBlock:^{
     /* Do something. */
}];
```