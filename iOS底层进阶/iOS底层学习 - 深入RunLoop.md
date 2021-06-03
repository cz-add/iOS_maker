RunLoop这个名词对于iOS开发来说应该是一个听腻了的词汇，而且只知其一不知其二，本篇章就来再深入复习一下RunLoop

## RunLoop简介

## 什么是RunLoop

一般来讲，一个线程一次只能执行一个任务，执行完成后线程就会退出。如果我们需要一个机制，让线程能随时处理事件但并不退出，这种模型通常被称作 Event Loop。 Event Loop 在很多系统和框架里都有实现，比如 Node.js 的事件处理，比如 Windows 程序的消息循环，再比如 OSX/iOS 里的 `RunLoop` 。

实现这种模型的关键点在于：管理事件/消息，让线程在没有处理消息时休眠以避免资源占用、在有消息到来时立刻被唤醒。

## RunLoop作用

1.  保持程序持续运行：程序一启动就会开一个主线程，主线程一开起来就会跑一个主线程对应的RunLoop,RunLoop保证主线程不会被销毁，也就保证了程序的持续运行
2.  处理App中的各种事件（比如：触摸事件，定时器事件，Selector事件等）
3.  节省CPU资源，提高程序性能：程序运行起来时，当什么操作都没有做的时候，RunLoop就告诉CPU，现在没有事情做，我要去休息，这时CPU就会将其资源释放出来去做其他的事情，当有事情做的时候RunLoop就会立马起来去做事情

我们先通过API内一张图片来简单看一下RunLoop内部运行原理

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134828_8655d4f4_9027123.png "runloop1.png")

## 为什么使用RunLoop

了解了RunLoop的作用，那么在苹果系统中，为什么使用RunLoop呢？主要有一下几点

1.  使程序一直运行，并接受用户输入
2.  决定程序在何时处理应该处理哪些Event
3.  **调用解耦（Message Queue）** : 比如一次滑屏事件，可能会触发多条消息，所以必须有一个类似Message Queue的模块去处理来解耦，形成一个队列来依次处理，这样用户的调用方和处理方实现了完全解耦
4.  节省CPU的时间和效率

## RunLoop底层原理

## RunLoop代码层级

```
NSRunloop
CFRunloop
系统层
```

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134842_656fe1d1_9027123.png "runloop2.png")

## RunLoop入口

在OC代码中，Runloop是由系统默认开启的，就再main函数中，会开启主线程和RunLoop。如果没有Runloop，那么main函数执行完毕后，程序就退出了，这说明在UIApplicationMain函数中，开启了一个和 **主线程相关的RunLoop** ，导致 **UIApplicationMain** 不会返回，一直在运行中，也就保证了程序的持续运行。

```
int main(int argc, char * argv[]) {
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```

接着我们查看源码,发现 `CFRunLoopRun` 的底层实现结构也非常简单，就是一个 `do...while` 循环，我们可以把RunLoop看成一个死循环。如果没有RunLoop，UIApplicationMain函数执行完毕之后将直接返回，也就没有程序持续运行一说了。

```
void CFRunLoopRun(void) {    /* DOES CALLOUT */
    int32_t result;
    do {
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}
```

## RunLoop与线程的关系

首先，iOS 开发中能遇到两个线程对象: `pthread_t` 和 `NSThread` 。过去苹果有份文档标明了 `NSThread` 只是 `pthread_t` 的封装，但那份文档已经失效了，现在它们也有可能都是直接包装自最底层的 `mach thread` 。苹果并没有提供这两个对象相互转换的接口，但不管怎么样，可以肯定的是 `pthread_t` 和 `NSThread` 是一一对应的。比如，你可以通过 `pthread_main_thread_np()` 或 `[NSThread mainThread]` 来获取主线程；也可以通过 `pthread_self()` 或 `[NSThread currentThread]` 来获取当前线程。 `CFRunLoop` 是基于 `pthread` 来管理的。

苹果不允许直接创建 RunLoop，它只提供了两个自动获取的函数： `CFRunLoopGetMain()` 和 `CFRunLoopGetCurrent()` 。 这两个函数内部的逻辑大概是下面这样:

```
:white_check_mark:// 获得当前线程的RunLoop对象，内部调用_CFRunLoopGet0函数
CFRunLoopRef CFRunLoopGetCurrent(void) {
    CHECK_FOR_FORK();
    CFRunLoopRef rl = (CFRunLoopRef)_CFGetTSD(__CFTSDKeyRunLoop);
    if (rl) return rl;
    return _CFRunLoopGet0(pthread_self());
}

:white_check_mark:// 查看_CFRunLoopGet0方法
CF_EXPORT CFRunLoopRef _CFRunLoopGet0(pthread_t t) {
    :white_check_mark:// 如果为空则t设置为主线程
    if (pthread_equal(t, kNilPthreadT)) {
    t = pthread_main_thread_np();
    }
    __CFLock(&loopsLock);
    :white_check_mark:// 如果不存在runloop，则创建
    if (!__CFRunLoops) {
        __CFUnlock(&loopsLock);
    CFMutableDictionaryRef dict = CFDictionaryCreateMutable(kCFAllocatorSystemDefault, 0, NULL, &kCFTypeDictionaryValueCallBacks);
    :white_check_mark:// 根据传入的主线程获取主线程对应的RunLoop
    CFRunLoopRef mainLoop = __CFRunLoopCreate(pthread_main_thread_np());
    :white_check_mark:// 保存主线程 将主线程-key和RunLoop-Value保存到字典中
    CFDictionarySetValue(dict, pthreadPointer(pthread_main_thread_np()), mainLoop);
    if (!OSAtomicCompareAndSwapPtrBarrier(NULL, dict, (void * volatile *)&__CFRunLoops)) {
        CFRelease(dict);
    }
    CFRelease(mainLoop);
        __CFLock(&loopsLock);
    }

    :white_check_mark:// 从字典里面拿，将线程作为key从字典里获取一个loop
    CFRunLoopRef loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));
    __CFUnlock(&loopsLock);

    :white_check_mark:// 如果loop为空，则创建一个新的loop，所以runloop会在第一次获取的时候创建
    if (!loop) {  
    CFRunLoopRef newLoop = __CFRunLoopCreate(t);
        __CFLock(&loopsLock);
    loop = (CFRunLoopRef)CFDictionaryGetValue(__CFRunLoops, pthreadPointer(t));

    :white_check_mark:// 创建好之后，以线程为key runloop为value，一对一存储在字典中，下次获取的时候，则直接返回字典内的runloop
    if (!loop) { 
        CFDictionarySetValue(__CFRunLoops, pthreadPointer(t), newLoop);
        loop = newLoop;
    }
        // do not release run loops inside the loopsLock, because CFRunLoopDeallocate may end up taking it
        __CFUnlock(&loopsLock);
    :white_check_mark://线程结束是销毁loop
    CFRelease(newLoop);
    }
    :white_check_mark:// 如果传入线程和当前线程相同
    if (pthread_equal(t, pthread_self())) {
        :white_check_mark:// 注册一个回调，当线程销毁时，顺便也销毁对应的RunLoop
        _CFSetTSD(__CFTSDKeyRunLoop, (void *)loop, NULL);
        if (0 == _CFGetTSD(__CFTSDKeyRunLoopCntr)) {
            _CFSetTSD(__CFTSDKeyRunLoopCntr, (void *)(PTHREAD_DESTRUCTOR_ITERATIONS-1), (void (*)(void *))__CFFinalizeRunLoop);
        }
    }
    return loop;
}
```

通过源码分析可以看出，线程和 `RunLoop` 之间是一一对应的，其关系是保存在一个 `Dictionary` 字典里。所以我们创建子线程 `RunLoop` 时，只需在子线程中获取当前线程的 `RunLoop` 对象即可 `[NSRunLoop currentRunLoop]` ;。如果不获取，那子线程就不会创建与之相关联的RunLoop，并且只能在一个线程的内部获取其RunLoop。

当通过调用 `[NSRunLoop currentRunLoop]` ;方法获取RunLoop时，会先看一下字典里有没有子线程对应的RunLoop，如果有则直接返回RunLoop，如果没有则会创建一个，并将与之对应的子线程存入字典中。当线程结束时，RunLoop会被销毁。

总结一下Runloop与线程的关系

1.  每条线程都有唯一的一个与之对应的RunLoop对象
2.  RunLoop保存在一个全局的Dictionary里，线程作为key,RunLoop作为value
3.  主线程的RunLoop已经自动创建好了，子线程的RunLoop需要主动创建
4.  RunLoop在第一次获取时创建，在线程结束时销毁

## RunLoop底层结构

### `CFRunLoopRef`

通过源码我们找到 `__CFRunLoop` 结构体

```
typedef struct __CFRunLoop * CFRunLoopRef;

struct __CFRunLoop
{
    // CoreFoundation 中的 runtime 基础信息
    CFRuntimeBase _base;
    // 针对获取 mode 列表操作的锁
    pthread_mutex_t _lock; /* locked for accessing mode list */
    // 唤醒端口
    __CFPort _wakeUpPort;  // used for CFRunLoopWakeUp
    // 是否使用过
    Boolean _unused;
    // runloop 运行会重置的一个数据结构
    volatile _per_run_data *_perRunData; // reset for runs of the run loop
    // runloop 所对应线程
    pthread_t _pthread;
    uint32_t _winthread;
    // 存放 common mode 的集合
    CFMutableSetRef _commonModes;
    // 存放 common mode item 的集合
    CFMutableSetRef _commonModeItems;
    // runloop 当前所在 mode
    CFRunLoopModeRef _currentMode;
    // 存放 mode 的集合
    CFMutableSetRef _modes;

    // runloop 内部 block 链表表头指针
    struct _block_item *_blocks_head;
    // runloop 内部 block 链表表尾指针
    struct _block_item *_blocks_tail;
    // 运行时间点
    CFAbsoluteTime _runTime;
    // 休眠时间点
    CFAbsoluteTime _sleepTime;
    CFTypeRef _counterpart;
};

// 每次 RunLoop 运行后会重置
typedef struct _per_run_data
{
    uint32_t a;
    uint32_t b;
    uint32_t stopped;   // runloop 是否停止
    uint32_t ignoreWakeUps; // runloop 是否已唤醒
} _per_run_data;

// 链表节点
struct _block_item
{
    // 指向下一个 _block_item
    struct _block_item *_next;
    // 要么是 string 类型，要么是集合类型，也就是说一个 block 可能对应单个或多个 mode
    CFTypeRef _mode; // CFString or CFSet
    // 存放的真正要执行的 block
    void (^_block)(void);
};

};
```

通过查看RunLoop的底层结构，我们发现了RunLoop也是一个结构体对象，其中有几个主要的变量：

```
CFRunLoopModeRef _currentMode
CFMutableSetRef _modes
```

通过上述变量，我们可以知道:

*   RunLoop可以有多个mode对象
*   Runloop在同一时间只能且必须在某一种特定的Mode下面Run，更换Mode时，必须要停止当前的Loop，然后重启新的Loop，重启的意思是退出当前的while循环，然后重新设置一个新的while

### `CFRunLoopModeRef`

CFRunLoopModeRef 其实是指向 `__CFRunLoopMode` 结构体的指针， `__CFRunLoopMode` 结构体源码如下

```
typedef struct __CFRunLoopMode *CFRunLoopModeRef;

struct __CFRunLoopMode
{
    // CoreFoundation 中的 runtime 基础信息
    CFRuntimeBase _base;
    // 互斥锁，加锁前需要 runloop 先加锁
    pthread_mutex_t _lock; /* must have the run loop locked before locking this */
    // mode 的名称
    CFStringRef _name;
    // mode 是否停止
    Boolean _stopped;
    char _padding[3];
    // source0
    CFMutableSetRef _sources0;
    // source1
    CFMutableSetRef _sources1;
    // observers
    CFMutableArrayRef _observers;
    // timers
    CFMutableArrayRef _timers;
    CFMutableDictionaryRef _portToV1SourceMap;
    // port 的集合
    __CFPortSet _portSet;
    // observer 的 mask
    CFIndex _observerMask;
    // 如果定义了 GCD 定时器
#if USE_DISPATCH_SOURCE_FOR_TIMERS
    // GCD 定时器
    dispatch_source_t _timerSource;
    // 队列
    dispatch_queue_t _queue;
    // 当 GCD 定时器触发时设置为 true
    Boolean _timerFired; // set to true by the source when a timer has fired
    Boolean _dispatchTimerArmed;
#endif
// 如果使用 MK_TIMER
#if USE_MK_TIMER_TOO
    // MK_TIMER 的 port
    mach_port_t _timerPort;
    Boolean _mkTimerArmed;
#endif
#if DEPLOYMENT_TARGET_WINDOWS
    DWORD _msgQMask;
    void (*_msgPump)(void);
#endif
    // 定时器软临界点
    uint64_t _timerSoftDeadline; /* TSR */
    // 定时器硬临界点
    uint64_t _timerHardDeadline; /* TSR */
};

```

我们发现，一个 `CFRunLoopModeRef` 也包含很多变量，主要有 `_sources0` , `_sources0` 两个集合和 `_observers` , `_timers` 两个数组。

这说明一个mode可以包含多种items模式

### `CFRunLoopSourceRef`

`CFRunLoopSourceRef` 是事件源（输入源）。通过源码可以发现，其分为 `source0` 和 `source1` 两个。

*   `source0` ：处理App内部事件，App自己负责管理（触发），如 `UIEvent` ， `CFSocket` 等；
*   `source1` ：由Runloop和内核管理， `mach port` 驱动，如CFMachPort（轻量级的进程间通信的方式，NSPort就是对它的封装，还有Runloop的睡眠和唤醒就是通过它来做的）， `CFMessagePort` ；

```
typedef struct __CFRunLoopSource * CFRunLoopSourceRef;

struct __CFRunLoopSource
{
    // CoreFoundation 中的 runtime 基础信息
    CFRuntimeBase _base;
    uint32_t _bits;
    // 互斥锁
    pthread_mutex_t _lock;
    // source 的优先级，值为小，优先级越高
    CFIndex _order; /* immutable */
    // runloop 集合
    CFMutableBagRef _runLoops;
    // 一个联合体，说明 source 要么为 source0，要么为 source1
    union {
        CFRunLoopSourceContext version0;  /* immutable, except invalidation */
        CFRunLoopSourceContext1 version1; /* immutable, except invalidation */
    } _context;
};

typedef struct {
    CFIndex version;
    // source 的信息
    void *  info;
    const void *(*retain)(const void *info);
    void    (*release)(const void *info);
    CFStringRef (*copyDescription)(const void *info);
    // 判断 source 相等的函数
    Boolean (*equal)(const void *info1, const void *info2);
    CFHashCode  (*hash)(const void *info);
    void    (*schedule)(void *info, CFRunLoopRef rl, CFStringRef mode);
    void    (*cancel)(void *info, CFRunLoopRef rl, CFStringRef mode);
    // source 要执行的任务块
    void    (*perform)(void *info);
} CFRunLoopSourceContext;

```

### `CFRunLoopObserverRef`

`CFRunLoopObserverRef` 是观察者，每个Observer都包含了一个回调(函数指针)，当RunLoop的状态发生变化时，观察者就能通过回调接受到这个变化。主要是用来向外界报告Runloop当前的状态的更改。

```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry = (1UL << 0),// 即将进入Loop
    kCFRunLoopBeforeTimers = (1UL << 1),// 即将处理Timer
    kCFRunLoopBeforeSources = (1UL << 2),// 即将处理Source
    kCFRunLoopBeforeWaiting = (1UL << 5),// 即将进入休眠
    kCFRunLoopAfterWaiting = (1UL << 6),// 刚从休眠中唤醒
    kCFRunLoopExit = (1UL << 7),// 即将退出Loop
    kCFRunLoopAllActivities = 0x0FFFFFFFU // 所有事件
};
```

```
typedef struct __CFRunLoopObserver * CFRunLoopObserverRef;

struct __CFRunLoopObserver
{
    // CoreFoundation 中的 runtime 基础信息
    CFRuntimeBase _base;
    // 互斥锁
    pthread_mutex_t _lock;
    // observer 对应的 runloop
    CFRunLoopRef _runLoop;
    // observer 观察了多少个 runloop
    CFIndex _rlCount;
    CFOptionFlags _activities;          /* immutable */
    // observer 优先级
    CFIndex _order;                     /* immutable */
    // observer 回调函数
    CFRunLoopObserverCallBack _callout; /* immutable */
    // observer 上下文
    CFRunLoopObserverContext _context;  /* immutable, except invalidation */
};

typedef struct {
    CFIndex	version;
    void *	info;
    const void *(*retain)(const void *info);
    void	(*release)(const void *info);
    CFStringRef	(*copyDescription)(const void *info);
} CFRunLoopObserverContext;

typedef void (*CFRunLoopObserverCallBack)(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info);

```

### `CFRunLoopTimerRef`

`CFRunLoopTimerRef` 是基于时间的触发器，它和NSTimer是toll-free bridged的，可以混用。其包含一个时间长度和一个回调(函数指针)。

当其加入到RunLoop时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒以执行那个回调。

总结一下关于RunLoop的结构

1.  `RunLoop` 本质也是一个结构体对象
2.  `RunloopMode` 是指的一个事件循环必须在某种模式下跑，系统会预定义几个模式。一个Runloop有多个Mode；
3.  `CFRunloopSource` ， `CFRunloopTimer` ， `CFRunloopObserver` 这些元素是在Mode里面的，Mode与这些元素的对应关系也是1对多的。但是必须至少有一个 `Source` 或者 `Timer` ，因为如果Mode为空，RunLoop运行到空模式不会进行空转，就会立刻退出。
4.  `CFRunloopSource` 分为 `source0` (处理用户事件)和 `source1` (处理内核事件)
5.  `CFRunloopObserver` 是监听和通知Runloop状态
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134854_ad216c69_9027123.png "runloop3.png")


## RunLoop的Mode

RunLoop 有五种运行模式，其中常见的有1.2两种。

```
1\. kCFRunLoopDefaultMode：App的默认Mode，通常主线程是在这个Mode下运行
2\. UITrackingRunLoopMode：界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响
3\. UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用，会切换到kCFRunLoopDefaultMode
4\. GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到
5\. kCFRunLoopCommonModes: 这是一个占位用的Mode，作为标记kCFRunLoopDefaultMode和UITrackingRunLoopMode用，并不是一种真正的Mode 
```

### Mode间的切换

我们平时在开发中一定遇到过，当我们使用NSTimer每一段时间执行一些事情时滑动UIScrollView，NSTimer就会暂停，当我们停止滑动以后，NSTimer又会重新恢复的情况，这是由于 `RunloopMode` 必须在同一个模式下跑。

主线程的 RunLoop 里有两个预置的 Mode： `kCFRunLoopDefaultMode` 和 `UITrackingRunLoopMode` 。这两个 Mode 都已经被标记为”Common”属性。 `DefaultMode` 是 App 平时所处的状态， `TrackingRunLoopMode` 是追踪 ScrollView 滑动时的状态。当你创建一个 Timer 并加到 DefaultMode 时，Timer 会得到重复回调，但此时滑动一个TableView时，RunLoop 会将 mode 切换为 `TrackingRunLoopMode` ，这时 Timer 就不会被回调，并且也不会影响到滑动操作。

有时你需要一个 Timer，在两个 Mode 中都能得到回调，一种办法就是将这个 Timer 分别加入这两个 Mode。还有一种方式，就是将 Timer 加入到顶层的 RunLoop 的 `“commonModeItems”` 中 `。”commonModeItems”` 被 RunLoop 自动更新到所有具有”Common”属性的 Mode 里去。

一个 Mode 可以将自己标记为”Common”属性（通过将其 ModeName 添加到 RunLoop 的 “commonModes” 中）。每当 RunLoop 的内容发生变化时，RunLoop 都会自动将 _commonModeItems 里的 Source/Observer/Timer 同步到具有 “Common” 标记的所有Mode里。

## RunLoop启动逻辑

我们知道在main函数启动时，会有Runloop的用DefaultMode默认启动和使用指定Mode进行启动，相关的源码如下，可以发现，其核心逻辑都是调用了 `CFRunLoopRunSpecific` 函数

```
/// 用DefaultMode启动
void CFRunLoopRun(void) {   /* DOES CALLOUT */
    int32_t result;
    do {
        result = CFRunLoopRunSpecific(CFRunLoopGetCurrent(), kCFRunLoopDefaultMode, 1.0e10, false);
        CHECK_FOR_FORK();
    } while (kCFRunLoopRunStopped != result && kCFRunLoopRunFinished != result);
}
/// 用指定的Mode启动，并允许设置RunLoop的超时时间
SInt32 CFRunLoopRunInMode(CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled) {     /* DOES CALLOUT */
    CHECK_FOR_FORK();
    return CFRunLoopRunSpecific(CFRunLoopGetCurrent(), modeName, seconds, returnAfterSourceHandled);
}
```

### `CFRunLoopRunSpecific`

接着我们查看 `CFRunLoopRunSpecific` 函数，根据其代码，主要总结为以下几个步骤：

*   从 runloop 中查找给定的 mode
*   将查找到的 mode 赋值到 runloop 的 _curentMode，也就是说在这 runloop 完成了 mode 的切换
*   调用核心函数 __CFRunLoopRun
*   如果注册了 observer，则通知runloop的开启，运行，结束等状态

```
SInt32 CFRunLoopRunSpecific(CFRunLoopRef rl, CFStringRef modeName, CFTimeInterval seconds, Boolean returnAfterSourceHandled)
{ /* DOES CALLOUT */
    CHECK_FOR_FORK();
    // 如果 runloop 正在回收中，直接返回 kCFRunLoopRunFinished ，表示 runloop 已经完成
    if (__CFRunLoopIsDeallocating(rl))
        return kCFRunLoopRunFinished;
    // 对 runloop 加锁
    __CFRunLoopLock(rl);
    :white_check_mark:// 从 runloop 中查找给定的 mode
    CFRunLoopModeRef currentMode = __CFRunLoopFindMode(rl, modeName, false);
    :white_check_mark:// 如果找不到 mode，且当前 runloop 的 currentMode 也为空，进入 if 逻辑
    // __CFRunLoopModeIsEmpty 函数结果为空的话，说明 runloop 已经处理完所有任务
    if (NULL == currentMode || __CFRunLoopModeIsEmpty(rl, currentMode, rl->_currentMode))
    {
        Boolean did = false;
        // 如果 currentMode 不为空
        if (currentMode)
            // 对 currentMode 解锁
            __CFRunLoopModeUnlock(currentMode);
        // 对 runloop 解锁
        __CFRunLoopUnlock(rl);
        return did ? kCFRunLoopRunHandledSource : kCFRunLoopRunFinished;
    }
    // 暂时取出 runloop 的 per_run_data
    volatile _per_run_data *previousPerRun = __CFRunLoopPushPerRunData(rl);
    :white_check_mark:// 取出 runloop 的当前 mode
    CFRunLoopModeRef previousMode = rl->_currentMode;
    :white_check_mark:// 将查找到的 mode 赋值到 runloop 的 _curentMode，也就是说在这 runloop 完成了 mode 的切换
    rl->_currentMode = currentMode;
    :white_check_mark:// 初始化返回结果 result
    int32_t result = kCFRunLoopRunFinished;

    :white_check_mark:// 如果注册了 observer 监听 kCFRunLoopEntry 状态(即将进入 loop)，则通知 observer
    if (currentMode->_observerMask & kCFRunLoopEntry)
        __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopEntry);
    :white_check_mark::white_check_mark::white_check_mark::white_check_mark:// runloop 核心函数 __CFRunLoopRun
    result = __CFRunLoopRun(rl, currentMode, seconds, returnAfterSourceHandled, previousMode);
    :white_check_mark:// 如果注册了 observer 监听 kCFRunLoopExit 状态(即将推出 loop)，则通知 observer
    if (currentMode->_observerMask & kCFRunLoopExit)
        __CFRunLoopDoObservers(rl, currentMode, kCFRunLoopExit);
    // 对 currentMode 解锁
    __CFRunLoopModeUnlock(currentMode);
    // 还原原来的 previousPerRun
    __CFRunLoopPopPerRunData(rl, previousPerRun);
    // 还原原来的 mode
    rl->_currentMode = previousMode;
    // 对 runloop 解锁
    __CFRunLoopUnlock(rl);
    return result;
}
```

### `CFRunLoopRun`

`CFRunLoopRun` 是RunLoop的核心函数，一次运行循环就是一次 `CFRunLoopRun` 的运行。其5个参数分别代表的意义如下：

```
CFRunLoopRef rl
CFRunLoopModeRef rlm
CFTimeInterval seconds
Boolean stopAfterHandle
CFRunLoopModeRef previousMode
```

```
static int32_t __CFRunLoopRun(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFTimeInterval seconds, Boolean stopAfterHandle, CFRunLoopModeRef previousMode)
```

由于 `CFRunLoopRun` 的函数过长，逻辑比较复杂，所以我们精简了代码，只讲解其中的一些的核心逻辑，主要有以下几个步骤：

1.  使用 `dispatch_source_t` 创建一个定时器来处理超时相关的逻辑，如果没设置会默认一个特别大的数字
2.  启动 `do...while` 循环开始处理事件
3.  通知 `Observers` : RunLoop 即将触发 `Timer` 回调。
4.  通知 `Observers` : RunLoop 即将触发 `Source0` (非port) 回调。
5.  执行被加入的block
6.  如果有 `Source1` (基于port) 处于 ready 状态，直接处理这个 Source1 然后跳转去处理消息。
7.  通知 `Observers` : RunLoop 的线程即将进入休眠(sleep)。
8.  调用 `mach_msg` 等待接受 `mach_port` 的消息。线程将进入休眠, 直到被下面某一个事件唤醒。
    *   基于 port 的 Source 事件；
    *   Timer 时间到；
    *   RunLoop 自身的超时时间到了
    *   被其他什么调用者手动唤醒
9.  通知 Observers: RunLoop 的线程刚刚被唤醒了。
10.  收到消息，处理消息。


    *   如果一个 Timer 到时间了，触发这个Timer的回调。
    *   如果有dispatch到main_queue的block，执行block。
    *   如果一个 Source1 (基于port) 发出事件了，处理这个事件
11.  执行加入到Loop的block
12.  根据当前 RunLoop 的状态来判断是否需要走下一个 loop。当被外部强制停止或 loop 超时时，就不继续下一个 loop 了，否则继续走下一个 loop 。

```
static int32_t __CFRunLoopRun(CFRunLoopRef rl, CFRunLoopModeRef rlm, CFTimeInterval seconds, Boolean stopAfterHandle, CFRunLoopModeRef previousMode)
{
     // 声明一个空的 GCD 定时器
    dispatch_source_t timeout_timer = NULL;
    // 初始化一个 「超时上下文」 结构体指针对象
    struct __timeout_context *timeout_context = (struct __timeout_context *)malloc(sizeof(*timeout_context));
    ...

    int32_t retVal = 0;
    do
    {
        // 通知 Observers 即将处理 Timers
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeTimers);

        // 通知 Observers 即将处理 Sources
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeSources);

        // 处理 Blocks
        __CFRunLoopDoBlocks(rl, rlm);

        // 处理 Source0
        if (__CFRunLoopDoSources0(rl, rlm, stopAfterHandle))
        {
            // 处理 Blocks
            __CFRunLoopDoBlocks(rl, rlm);
        }

        Boolean poll = sourceHandledThisLoop || (0ULL == timeout_context->termTSR);

        // 判断有无 Source1
        if (__CFRunLoopServiceMachPort(dispatchPort, &msg, sizeof(msg_buffer), &livePort, 0, &voucherState, NULL))
        {
            // 如果有 Source1，就跳转到 handle_msg
            goto handle_msg;
        }

        didDispatchPortLastTime = false;

        // 通知 Observers 即将休眠
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopBeforeWaiting);

        __CFRunLoopSetSleeping(rl);

        CFAbsoluteTime sleepStart = poll ? 0.0 : CFAbsoluteTimeGetCurrent();

        do
        {
            if (kCFUseCollectableAllocator)
            {
                // objc_clear_stack(0);
                // <rdar://problem/16393959>
                memset(msg_buffer, 0, sizeof(msg_buffer));
            }
            msg = (mach_msg_header_t *)msg_buffer;

            // 等待别的消息来唤醒当前线程
            __CFRunLoopServiceMachPort(waitSet, &msg, sizeof(msg_buffer), &livePort, poll ? 0 : TIMEOUT_INFINITY, &voucherState, &voucherCopy);

            if (modeQueuePort != MACH_PORT_NULL && livePort == modeQueuePort)
            {
                // Drain the internal queue. If one of the callout blocks sets the timerFired flag, break out and service the timer.
                while (_dispatch_runloop_root_queue_perform_4CF(rlm->_queue))
                    ;
                if (rlm->_timerFired)
                {
                    // Leave livePort as the queue port, and service timers below
                    rlm->_timerFired = false;
                    break;
                }
                else
                {
                    if (msg && msg != (mach_msg_header_t *)msg_buffer)
                        free(msg);
                }
            }
            else
            {
                // Go ahead and leave the inner loop.
                break;
            }
        } while (1);

        // user callouts now OK again
        __CFRunLoopUnsetSleeping(rl);

        // 通知 Observers 结束休眠
        __CFRunLoopDoObservers(rl, rlm, kCFRunLoopAfterWaiting);

    handle_msg:
        if (被 timer 唤醒)
        {
            // 处理 timers
            __CFRunLoopDoTimers(rl, rlm, mach_absolute_time());
        }

        else if (被 GCD 唤醒)
        {
            // 处理 GCD 主队列
            __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__(msg);
        }
        else
        {
            // 被 Source1 唤醒
            __CFRunLoopDoSource1(rl, rlm, rls, msg, msg->msgh_size, &reply) || sourceHandledThisLoop;
        }

        // 处理 Blocks
        __CFRunLoopDoBlocks(rl, rlm);

        if (sourceHandledThisLoop && stopAfterHandle)
        {
            retVal = kCFRunLoopRunHandledSource;
        }
        else if (timeout_context->termTSR < mach_absolute_time())
        {
            retVal = kCFRunLoopRunTimedOut;
        }
        else if (__CFRunLoopIsStopped(rl))
        {
            __CFRunLoopUnsetStopped(rl);
            retVal = kCFRunLoopRunStopped;
        }
        else if (rlm->_stopped)
        {
            rlm->_stopped = false;
            retVal = kCFRunLoopRunStopped;
        }
        else if (__CFRunLoopModeIsEmpty(rl, rlm, previousMode))
        {
            retVal = kCFRunLoopRunFinished;
        }

        voucher_mach_msg_revert(voucherState);
        os_release(voucherCopy);

    } while (0 == retVal);

    return retVal;
}

```
![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134909_7cba2eae_9027123.png "runloop4.png")

## RunLoop的退出

1.  主线程销毁RunLoop退出
2.  Mode中有一些Timer 、Source、 Observer，这些保证Mode不为空时保证RunLoop没有空转并且是在运行的，当Mode中为空的时候，RunLoop会立刻退出
3.  我们在启动RunLoop的时候可以设置什么时候停止

```
[NSRunLoop currentRunLoop]runUntilDate:<#(nonnull NSDate *)#>
[NSRunLoop currentRunLoop]runMode:<#(nonnull NSString *)#> beforeDate:<#(nonnull NSDate *)#>
```

## RunLoop的应用

## RunLoop在系统中的应用

### AutoreleasePool

App启动后，苹果在主线程 RunLoop 里注册了两个 Observer，其回调都是 `_wrapRunLoopWithAutoreleasePoolHandler()` 。

第一个 `Observer` 监视的事件是 `Entry(即将进入Loop)` ，其回调内会调用 `_objc_autoreleasePoolPush()` 创建自动释放池。其 order 是-2147483647，优先级最高，保证创建释放池发生在其他所有回调之前。

第二个 `Observer` 监视了两个事件： `BeforeWaiting(准备进入休眠)` 时调用 `_objc_autoreleasePoolPop()` 和 `_objc_autoreleasePoolPush()` 释放旧的池并创建新池；Exit(即将退出Loop) 时调用 `_objc_autoreleasePoolPop()` 来释放自动释放池。这个 Observer 的 order 是 2147483647，优先级最低，保证其释放池子发生在其他所有回调之后。

在主线程执行的代码，通常是写在诸如事件回调、Timer回调内的。这些回调会被 RunLoop 创建好的 `AutoreleasePool` 环绕着，所以不会出现内存泄漏，开发者也不必显示创建 Pool 了。

### 事件响应

苹果注册了一个 Source1 (基于 mach port 的) 用来接收系统事件，其回调函数为 `__IOHIDEventSystemClientQueueCallback()。`

当一个硬件事件(触摸/锁屏/摇晃等)发生后，首先由 `IOKit.framework` 生成一个 `IOHIDEvent` 事件并由 `SpringBoard` 接收。SpringBoard 只接收按键(锁屏/静音等)，触摸，加速，接近传感器等几种 Event，随后用 mach port 转发给需要的App进程。随后苹果注册的那个 Source1 就会触发回调，并调用 `_UIApplicationHandleEventQueue()` 进行应用内部的分发。

`_UIApplicationHandleEventQueue()` 会把 `IOHIDEvent` 处理并包装成 `UIEvent` 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的。

### 手势识别

当上面的 `_UIApplicationHandleEventQueue()` 识别了一个手势时，其首先会调用 `Cancel` 将当前的 `touchesBegin/Move/End` 系列回调打断。随后系统将对应的 `UIGestureRecognizer` 标记为待处理。

苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，这个Observer的回调函数是 `_UIGestureRecognizerUpdateObserver()` ，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。

当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。

### 界面更新

当在操作 UI 时，比如改变了 Frame、更新了 UIView/CALayer 的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理，并被提交到一个全局的容器去。

苹果注册了一个 Observer 监听 BeforeWaiting(即将进入休眠) 和 Exit (即将退出Loop) 事件，回调去执行一个很长的函数： `_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()` 。这个函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

这个函数内部的调用栈大概是这样的：

```
_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()
    QuartzCore:CA::Transaction::observer_callback:
        CA::Transaction::commit();
            CA::Context::commit_transaction();
                CA::Layer::layout_and_display_if_needed();
                    CA::Layer::layout_if_needed();
                        [CALayer layoutSublayers];
                            [UIView layoutSubviews];
                    CA::Layer::display_if_needed();
                        [CALayer display];
                            [UIView drawRect];
```

### 定时器

NSTimer 其实就是 CFRunLoopTimerRef，他们之间是 toll-free bridged 的。一个 NSTimer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件。例如 10:00, 10:10, 10:20 这几个时间点。RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer。Timer 有个属性叫做 Tolerance (宽容度)，标示了当时间点到后，容许有多少最大误差。

如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。

`CADisplayLink` 是一个和屏幕刷新率一致的定时器（但实际实现原理更复杂，和 NSTimer 并不一样，其内部实际是操作了一个 Source）。如果在两次屏幕刷新之间执行了一个长任务，那其中就会有一帧被跳过去（和 NSTimer 相似），造成界面卡顿的感觉。在快速滑动TableView时，即使一帧的卡顿也会让用户有所察觉。Facebook 开源的 AsyncDisplayLink 就是为了解决界面卡顿的问题，其内部也用到了 RunLoop.

### PerformSelecter

当调用 NSObject 的 performSelecter:afterDelay: 后，实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中。所以如果当前线程没有 RunLoop，则这个方法会失效。

当调用 performSelector:onThread: 时，实际上其会创建一个 Timer 加到对应的线程去，同样的，如果对应线程没有 RunLoop 该方法也会失效。

### 关于GCD

实际上 RunLoop 底层也会用到 GCD 的东西，NSTimer 是用了 XNU 内核的 `mk_timer` ，我也仔细调试了一下，发现 NSTimer 确实是由 `mk_timer` 驱动，而非 GCD 驱动的）。但同时 GCD 提供的某些接口也用到了 RunLoop， 例如 `dispatch_async()。`

当调用 `dispatch_async(dispatch_get_main_queue(), block)` 时，libDispatch 会向主线程的 RunLoop 发送消息，RunLoop会被唤醒，并从消息中取得这个 block，并在回调 `__CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__()` 里执行这个 block。但这个逻辑 **仅限于** dispatch 到主线程，dispatch 到其他线程仍然是由 libDispatch 处理的。

### 关于网络请求

iOS 中，关于网络请求的接口自下至上有如下几层:

```
CFSocket
CFNetwork       ->ASIHttpRequest
NSURLConnection ->AFNetworking
NSURLSession    ->AFNetworking2, Alamofire
```

*   CFSocket 是最底层的接口，只负责 socket 通信。
*   CFNetwork 是基于 CFSocket 等接口的上层封装，ASIHttpRequest 工作于这一层。
*   NSURLConnection 是基于 CFNetwork 的更高层的封装，提供面向对象的接口，AFNetworking 工作于这一层。
*   NSURLSession 是 iOS7 中新增的接口，表面上是和 NSURLConnection 并列的，但底层仍然用到了 NSURLConnection 的部分功能 (比如 com.apple.NSURLConnectionLoader 线程)，AFNetworking2 和 Alamofire 工作于这一层。

下面主要介绍下 `NSURLConnection` 的工作过程。

通常使用 `NSURLConnection` 时，你会传入一个 `Delegate` ，当调用了 `[connection start]` 后，这个 Delegate 就会不停收到事件回调。实际上，start 这个函数的内部会会获取 `CurrentRunLoop` ，然后在其中的 `DefaultMode` 添加了4个 `Source0` (即需要手动触发的Source)。

`CFMultiplexerSource` 是负责各种 Delegate 回调的，

`CFHTTPCookieStorage` 是处理各种 Cookie 的。

当开始网络传输时，我们可以看到 NSURLConnection 创建了两个新线程： `com.apple.NSURLConnectionLoader` 和 `com.apple.CFSocket.private` 。其中 `CFSocket` 线程是处理底层 socket 连接的。 `NSURLConnectionLoader` 这个线程内部会使用 RunLoop 来接收底层 socket 的事件，并通过之前添加的 `Source0` 通知到上层的 Delegate。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0528/134922_89ad3562_9027123.png "runloop5.png")


## RunLoop在实际开发中的应用

### AFNetworking

AFURLConnectionOperation 这个类是基于 NSURLConnection 构建的，其希望能在后台线程接收 Delegate 回调。为此 AFNetworking 单独创建了一个线程，并在这个线程中启动了一个 RunLoop：

```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];
        NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
        [runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runLoop run];
    }
}

+ (NSThread *)networkRequestThread {
    static NSThread *_networkRequestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```

RunLoop 启动前内部必须要有至少一个 `Timer/Observer/Source` ，所以 AFNetworking 在 [runLoop run] 之前先创建了一个新的 NSMachPort 添加进去了。通常情况下，调用者需要持有这个 `NSMachPort (mach_port)` 并在外部线程通过这个 port 发送消息到 loop 内；但此处添加 port 只是为了让 RunLoop 不至于退出，并没有用于实际的发送消息。

```
- (void)start {
    [self.lock lock];
    if ([self isCancelled]) {
        [self performSelector:@selector(cancelConnection) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    } else if ([self isReady]) {
        self.state = AFOperationExecutingState;
        [self performSelector:@selector(operationDidStart) onThread:[[self class] networkRequestThread] withObject:nil waitUntilDone:NO modes:[self.runLoopModes allObjects]];
    }
    [self.lock unlock];
}
```

当需要这个后台线程执行任务时，AFNetworking 通过调用 [NSObject performSelector:onThread:..] 将这个任务扔到了后台线程的 RunLoop 中。

### TableView延迟加载图片

将setImage放到NSDefaultRunLoopMode去做，也就是在滑动的时候并不会去调用这个方法，而是会等到滑动完毕切换到NSDefaultRunLoopMode下面才会调用。

```
UIImage *downLoadImage = ...;  
[self.avatarImageView performSelector:@selector(setImage:)  
                        withObject:downloadImage  
                        afterDelay:0  
                        inModes:@[NSDefaultRunLoopMode]];
```

### Crash的兼容处理

```
EXC_BAD_ACCESS
```

### 检测卡顿

当App发生主线程卡顿时，我们可以通过RunLoop来监听到相对应的堆栈信息，然后进行优化处理。

*   要想监听 RunLoop，你就首先需要创建一个 `CFRunLoopObserverContext` 观察者

```
CFRunLoopObserverContext context = {0,(__bridge void*)self,NULL,NULL};
runLoopObserver = CFRunLoopObserverCreate(kCFAllocatorDefault,kCFRunLoopAllActivities,YES,0,&runLoopObserverCallBack,&context);
```

*   将创建好的观察者 runLoopObserver 添加到主线程 RunLoop 的 common 模式下观察。然后，创建一个持续的子线程专门用来监控主线程的 RunLoop 状态。
*   一旦发现进入睡眠前的 kCFRunLoopBeforeSources 状态，或者唤醒后的状态 kCFRunLoopAfterWaiting，在设置的时间阈值内一直没有变化，即可判定为卡顿。接下来，我们就可以 dump 出堆栈的信息，从而进一步分析出具体是哪个方法的执行时间过长。

```
//创建子线程监控
dispatch_async(dispatch_get_global_queue(0, 0), ^{
    //子线程开启一个持续的 loop 用来进行监控
    while (YES) {
        long semaphoreWait = dispatch_semaphore_wait(dispatchSemaphore, dispatch_time(DISPATCH_TIME_NOW, 3 * NSEC_PER_SEC));
        if (semaphoreWait != 0) {
            if (!runLoopObserver) {
                timeoutCount = 0;
                dispatchSemaphore = 0;
                runLoopActivity = 0;
                return;
            }
            //BeforeSources 和 AfterWaiting 这两个状态能够检测到是否卡顿
            if (runLoopActivity == kCFRunLoopBeforeSources || runLoopActivity == kCFRunLoopAfterWaiting) {
                //将堆栈信息上报服务器的代码放到这里
            } //end activity
        }// end semaphore wait
        timeoutCount = 0;
    }// end while
});
```
