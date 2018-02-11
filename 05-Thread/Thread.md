# OC 高级编程 多线程和内存管理

# 自动引用计数
## 什么是自动引用计数
自动引用计数是指内存管理对引用采取自动计数的技术。
在OC中采用ARC技术，让编译器来进行内存管理。在新一代Apple LLVM编译器中设置ARC为有效状态，就无需再次键入retain或release代码，这在降低程序崩溃、内存泄漏等风险的同事，很大程序上减少了开发程序的工作量。

## 内存管理/引用计数
* 内存管理的思考方式
    * 自己生产的对象，自己所持有
    * 非自己生成的对象，自己也能持有
    * 不再需要自己持有的对象时释放
    * 非自己持有的对象无法释放

生成并持有对象 alloc/new/copy/mutableCopy 等方法
持有对象 retain 方法
释放对象 release 方法
废弃对象 dealloc 方法

* 自己生成的对象，自己所持有
是有以下名称开头的方法名意味着自己生成的对象只有自己持有：
    * alloc
    * new
    * copy
    * mutableCopy

* 非自己生成的对象，自己也能持有
通过retain方法，非自己生成的对象跟用alloc/new/copy/mutableCopy方法生成并持有的对象一样，成为了自己所持有的。
* 不再需要自己持有的对象时释放
自己持有的对象，一旦不在需要，持有者有义务释放该对象。释放使用release方法。

autorelease方法，可以使取得的对象存在，但自己不持有对象。是对象在超出指定的生产范围时能够自己并正确的释放。
* 无法释放非自己持有的对象
对于用alloc/new/copy/mutableCopy方法生成并持有的对象，或是用retain方法持有的对象，由于持有者是自己，所以在不需要该对象时需要将其释放。而由此以为所得到的对象绝对不能释放。倘若在应用程序中释放了非自己所持有的对象就会造成崩溃。例如自己生成并持有对象后，在释放完不在需要的对象之后再次释放。


## alloc/retain/release/dealloc实现


## autorelease
说道OC内存管理，就不能不提autorelease。
autorelease就是自动释放。
autorelease的具体使用方法如下：
1. 生成并持有NSAutoreleasePool对象
2. 调用已分配对象的autorelalease实例对象
3. 废弃NSAutoreleasePool对象

在Cocoa框架中，相当于程序主循环的NSRunLoop或者其他程序可运行的地方，对NSAutoreleasePool对象进行生成、持有和废弃处理。因此，应用程序不一定非得使用NSAutoreleasePool对象来进行开发工作。

尽管如此，但在大量产生autorelease的对象时，只要不废弃NSAutoreleasePool对象，那么生成的对象就不能被释放，因此有时会产生内存不足的现象。

另外， Cocoa框架中也有很多类方法用于返回autorelease的对象。

## autorelease实现

* autorelease NSAutoreleasePool对象
提问：如果autorelease NSAutoreleasePool对象会如何？
回答：发生异常
通常在使用OC，也就是Foundation框架是，无论调用哪一个对象的autoreloease实例方法，实现上是调用NSObject类的autorelease实例方法。但是对于NSAutoreleasePool类，autorelease实例方法已被该类重载，因此运行时就会出错。

## ARC规则
设置ARC有效的编译方法如下：
    * 使用clang（LLVM编译器）3以上
    * 指定编译器属性为：-fobjc-arc

* 所有权修饰符
OC编程中为了处理对象，可将变量类型定义为id类型或各种对象类型
所谓对象类型就是指向NSObject这样的OC类的指针。id类型用于隐藏对象类型的类名部分，相当于C语言中常用的void *。

ARC有效时，id类型和对象类型同C语言其他类型不同，其类型上必须附加所有权修饰符。
所有修饰符一共4中：
    * __strong 修饰符
    * __weak 修饰符
    * __unsafe_unretained 修饰符
    * __autoreleasing 修饰符

* __strong 修饰符
__strong修饰符是id类型和对象类型默认的所有权修饰符。
* __weak 修饰符
循环引用容易发生内存泄漏。所谓内存泄漏就是应当废弃的对象在超出其生产周期后继承存在。循环引用使得对象不能被再次废弃。

怎么样才能避免循环引用呢？
__weak修饰符与 __strong修饰符相反，提供弱引用。弱引用不能持有对象实例。

带__weak修饰符的变量不持有对象，所以在超出其变量作用域时，对象即被释放。

__weak修饰符还有另一有点。在持有某对象的弱引用时，则此弱引用将自动失效且处于nil被复制的状态。

* __unsafe_unretained修饰符
__unsafe_unretained修饰符正如其名unsafe所示，是不安全的所有权修饰符。
尽管ARC式的内存管理是编译器的工作，但附有__unsafe_unretained修饰符的变量不属于编译器的内存管理对象。


* __autoreleasing 修饰符
ARC有效时autorelease会如何呢？
指定 @autoreleasepool 块来替代 NSAutoreleasePool 类对象生成、持有以及废弃。
另外，ARC有效时，要通过附有 __autoreleasing 修饰符的变量等价于在ARC无效时调用对象的autorelease方法，即对象呗注册到autoreleasepool。

* 规则
在ARC有效的情况下编译源代码，必须遵守一定的规则。
    * 不能使用retain/release/retainCount/autorelease
    * 不能使用NSAllocateObject/NSDeallocateObject
    * 必须遵守内存管理的方法命名规则
    * 不要显示调用dealloc
    * 使用@autoreleasepool块替代NSAutoreleasePool
    * 不能使用区域NSZone
    * 对象型变量不能作为C语言结构体的成员
    * 显示转换id和void*


## 属性

## 数组





# Blocks

# Blocks模式
## block语法
^返回值类型 参数列表 表达式

## Block类型变量
Block类型变量可作为以下用途使用:
    * 自动变量
    * 函数参数
    * 静态变量
    * 静态全局变量
    * 全局变量
## Blocks的实现
* Block的实质
Block是带有自动变量值的匿名函数，但Block究竟是什么呢？
由Block语法转换的__main_block_func_0函数的指针被赋值成员变量FuncPtr中。



# Grand Central Dispatch GCD

# GCD概要
## 什么是GCD
GCD是异步执行任务的技术之一。一般将应用程序中记述的线程管理用的代码在系统级中实现。开发者只需要定义想执行的任务并追加到适当的Dispatch Queue中，GCD就能生成必要的线程并继续执行任务。由于线程管理是作为系统的一部分来实现的，因此可统一管理，也可执行任务，这样就比以前更有效率。
```Objective-C
dispatch_async(queue, ^{
    /**
     * 长时间处理
     */
    
    dispatch_async(dispatch_get_main_queue(), ^{
        /*
         * 只在主线程可以执行的处理
         * 例如用户界面更新
         */
    });
});
```


多线程的程序可以在某个线程和其他消除之间反复多次进行上下文切换，因此看上去就像1个CPU核能够并列执行多个线程一样。而且在具有多个CPU核的情况下，就不是看上去像了，而是真的提供了多个CPU核执行多个线程的技术。

但是，多线程编程实际上是一种已发生各种问题的编程技术。比如多个线程更新相同的资源会导致数据的不一致（数据竞争）、停止等待事件的线程会导致多个线程相互持续等待（死锁）、使用太多线程会消耗大量内存等。

尽管极易发生各种问题，也应该使用多线程编程。是因为多线程编程可以保证应用程序的响应性能。

应用程序在启动时，通过最先执行的线程，即主线程来描绘用户界面、处理触摸屏幕的时间等。如果在该主线程中进行长时间的处理，就会妨碍主线程的执行（阻塞）。在应用程序中，会妨碍主线程中被称为RunLoop的主循环的执行，从而导致不能更新用户界面、应用程序的画面长时间停滞等问题。

这就是长时间的处理不在主线程中执行而在其他线程中执行的原因。

# GCD的API
## Dispatch Queue
开发者要做的只是定义想执行的任务并追加到适当的Dispatch Queue中。

Dispatch Queue 是什么呢？是执行处理的等待队列。Dispatch Queue按照追加的顺序 FIFO 执行处理。
另外在执行处理时存在两种 Dispatch Queue. 一种是等待现在执行中处理的 Serial Dispatch Queue，另一种是不等待现在执行中处理的Concurrent Dispatch Queue。

并行处理：就是使用多个线程同时执行多个处理。

## dispatch_queue_create
可以自己创建
## Main Dispatch Queue/Global Dispatch Queue
获取系统标准提供的Dispatch Queue
实际上不用特意生成Dispatch Queue系统也会给我们提供几个。那就是Main Dispatch Queue和Global Dispatch Queue。

Main Dispatch Queue正如其名称含有Main一样，实在主线程中执行的Dispatch Queue。因为主线程只有一个，所以Main Dispatch Queue自然就是Serial Dispatch Queue。

另一个Global Dispatch Queue是所有应用程序能够使用的Concurrent Dispatch Queue。没有必要通过dispatch_queue_create函数逐个生成Concurrent Dispatch Queue。只要获取Global Dispatch Queue使用即可。
另外，Global Dispatch Queue有4个优先级，分别是高优先级 High、默认优先级 Default、低优先级 Low、和后台优先级 Background。

## dispatch_set_target_queue
dispatch_queue_create函数生成的Dispatch Queue不过是Serial Dispatch Queue还是Concurrent Dispatch Queue，都是用与默认优先级Global Dispatch Queue相同优先级的线程。而变更生成的Dispatch Queue的优先级要使用Dispatch_set_target_queue函数。

## dispatch_after
dispatch_after函数并不是在指定时间后执行处理，而只是在指定时间追加处理到Dispatch Queue。
因为Main Dispatch Queue在主线程的RunLoop中执行，所以在比如每隔1/60秒执行的RunLoop中，Block最快在3秒后执行，最慢在3秒+1/60秒后执行，并且在Main Dispatch Queue有大量处理追加或主线程的处理本省有延迟时，这个时间会更长。


## Dispatch Group
在追加到Disaptch Queue中的多个处理全部结束后想执行结束处理，这种情况会经常出现。只使用一个Serial Dispatch Queue时，只要将想执行的处理全部追加到该Serial Dispatch Queue中并在最后追加结束处理，即可实现。但是在使用Concurrent Dispatch Queue时或同时使用多个Dispatch Queue时，源代码就会变得颇为复杂。

这种情况下，使用Dispatch Group。
```Objective-C
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group, queue, ^(NSLog(@"blk0");));
dispatch_group_async(group, queue, ^(NSLog(@"blk1");));
dispatch_group_async(group, queue, ^(NSLog(@"blk2");));

dispatch_group_notify(group, dispatch_get_main(), ^(NSLog(@"done");));
dispatch_release(group);
```


## dispatch_barrier_async
虽然利用Dispatch Group和dispatch_set_target_queue函数可以避免数据竞争的问题，但是会比较复杂，GCD提供了更好的解决办法
dispatch_barrier_async函数
首先dispatch_queue_create函数生成Concurrent Dispatch Queue，在dispatch_async中追加读取处理。


## dispatch_sync
dispatch_async函数的async意味着非同步，就是将指定的Block非同步地追加到指定的Dispatch Queue中。dispatch_async函数不做任何等待。
既然有async，当然也就有sync。

## dispatch_apply
dispatch_apply函数式dispatch_sync函数和Dispatch Group的关联API。该函数按指定的次数将指定的Block追加到指定的Dispatch Queue中，并等待全部执行结束。

## dispatch_suspend/dispatch_resume
dispatch_suspend函数挂起指定的Dispatch Queue
dispatch_resume函数恢复指定的Dispatch Queue

## Dispatch Semaphore


## dispatch_once
dispatch_once 函数式保证在应用程序执行中只执行一次。

## Dispatch I/O

