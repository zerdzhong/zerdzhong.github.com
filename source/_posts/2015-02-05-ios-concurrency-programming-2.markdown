---
layout: post
title: "iOS 多线程技术总结-2"
date: 2015-02-05 09:29:47 +0800
comments: true
categories: iOS Concurrency
---

上一篇总结了iOS中提供的一些实现并发的技术，这一篇主要总结一下GCD。<!--more-->

GCD(Grand Central Dispatch) 是为了提高 OS X 和 iOS 系统在多核处理器上运行并行代码的能力而开发的一系列相关技术，是苹果最推荐使用的实现并行的技术，NSOperationQueue 就是在 GCD 的基础上实现的，基本原理相似只不过是将 block 添加到 dispatch queue 中和 operation 添加到 operation queue 中的区别，dispatch queue 是严格的先进先出结构，无法直接使用优先级或调整 block 的执行顺序，如果有此需求最好使用 Operation Queue 而不是重新发明轮子。

GCD 提供一套纯 C 的 API，需要和 block 配合使用。block 是 GCD 的执行单元，dispatch queue 是组织 block 的，dispatch queue 就是队列，不是线程。将 block 添加到队列中并不会让 block 运行，只是把 block 加到线程末尾。Dispatch queue 来调度 block 的执行。

## Dispatch Queue

Dispatch queue 是执行任务的有力工具，dispatch queue 能够异步或者同步的执行任意多个block。在 GCD 中有三中 dispatch queue :

1.串行队列（Serial dispatch queue）
     串行分发队列也称为私有分发队列（Private dispatch queue）,队列中的任务按照加入队列的顺序依次执行，且同一时间只执行一个任务，常被用于实现同步锁，由于串行队列中正在执行的任务实在单独的线程中，所以串行对列相对于其他线程或队列是并发的。创建一个串行分发队列：

dispatch_queue_t serialQueue = dispatch_queue_create("com.example.MyQueue", NULL);

2.并发队列（Concurrent dispatch queue）
     并行分发队列也称为全局分发队列（Global dispatch queue）,队列中的任务也按照加入的顺序依次执行，但是同时可以执行多个任务。正在执行的任务在不同的线程上执行，这些线程由队列管理。创建一个并发队列：

```
//1.由系统提供的四种全局分发队列中获取
dispatch_queue_t aQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);


//2.手动创建
dispatch_queue_t bQueue = dispatch_queue_create("com.example.MyQueue",DISPATCH_QUEUE_CONCURRENT );

```

3.主队列（Main dispatch queue）
主分发队列是一个全局唯一的特殊的串行分发队列。队列中的任务会被在应用的主线程中执行。主分发队列可以用于执行 UI 相关的操作。取得主分发队列的方法：
dispatch_queue_t mainQueue = dispatch_get_main_queue();

使用 dispatch queue 来替代线程实现程序并发的好处在于，不需要写管理线程的代码，关注在真正需要并发的任务实现上，系统处理了所有创建线程，管理线程的工作（比应用中手动管理更有效率，通常比手动开线程更快的执行）。

dispatch queue 的一些关键特性如下：
dispatch queue 相对其他队列来说都是并发的，串行的任务只在所在的串行队列中被限制。
系统决定同时执行的任务数。假如有100个任务分别加入100个队列中，也不一定会同时执行所有的任务。
系统会将队列的优先级作为选择哪个任务下一个执行的考虑因素。

## GCD 任务执行方式

GCD 中有两种任务执行方式：
     异步执行, dispatch_async，意味将任务放入队列之后，主线程不会等待 block 的返回结果，而是立即继续执行下去。
     阻塞执行, dispatch_sync，意味将任务放入队列之后，主线程被阻塞，需要等待 block 的执行结果返回，才能继续执行下去。