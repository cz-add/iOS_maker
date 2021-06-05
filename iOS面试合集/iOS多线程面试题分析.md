## 一、多线程的选择方案
| 技术方案 | 简介 | 语言 | 线程生命周期 | 使用评率 |
| --- | --- | --- | --- | --- |
| pthread | 一套通用的多线程API适用于Unix/Linux/Windows等系统跨平台/可移植使用难度大 | C |程序员管理 | 几乎不用 
| NSThread | 使用更加面向对象简单易用，可直接操作线程对象 | OC | 程序员管理 | 偶尔使用 |
| GCD | 旨在替代NSThread等线程技术充分利用设备的多核 | C | 自动管理 | 经常使用 |
| NSOperation | 基于GCD（底层是GCD）比GCD多了一些更简单实用的功能使用更加面向对象 | OC | 自动管理 | 经常使用 |

注意：如果使用NSThread的`performSelector:withObject:afterDelay:`时需要添加到当前线程的`runloop`中，因为在内部会创建一个`NSTimer`

## 二、GCD和NSOperation的比较

*   `GCD`和`NSOperation`的关系如下：

    *   `GCD`是面向底层的C语言的API
    *   `NSOperation`是用`GCD`封装构建的，是`GCD`的高级抽象
*   `GCD`和`NSOperation`的对比如下：

    1.  `GCD`执行效率更高，而且由于队列中执行的是由`block`构成的任务，这是一个轻量级的数据结构——写起来更加方便
    2.  `GCD`只支持`FIFO`的队列，而`NSOpration`可以设置最大并发数、设置优先级、添加依赖关系等调整执行顺序
    3.  `NSOpration`甚至可以跨队列设置依赖关系，但是`GCD`只能通过设置串行队列，或者在队列内添加`barrier`任务才能控制执行顺序，较为复杂
    4.  `NSOperation`支持`KVO`（面向对象）可以检测operation是否正在执行、是否结束、是否取消

> *   实际项目中，很多时候只会用到异步操作，不会有特别复杂的线程关系管理，所以苹果推崇的是优化完善、运行快速的GCD
> *   如果考虑异步操作之间的事务性、顺序性、依赖关系，比如多线程并发下载，GCD需要写更多的代码来实现，而NSOperation已经内建了这些支持
> *   不管是GCD还是NSOperation，我们接触的都是任务和队列，都没有直接接触到线程，事实上线程管理也的确不需要我们操心，系统对于线程的创建、调度管理和释放都做得很好；而NSThread需要我们自己去管理线程的生命周期，还要考虑线程同步、加锁问题，造成一些性能上的开销

## 三、多线程的应用场景

*   异步执行
    *   将耗时操作放在子线程中，使其不阻塞主线程
*   刷新UI
    *   异步网络请求，请求完毕`dispatch_get_main_queue()`回到主线程刷新UI
    *   同一页面多个网络请求使用`dispatch_group`统一调度刷新UI
*   `dispatch_once`
    *   在`单例`中使用，一个类仅有一个实例且提供一个全局访问点
    *   在`method-Swizzling`使用保证方法只交换一次
*   `dispatch_after`将任务延迟加入队列
*   `栅栏函数`可用作同步锁
*   `dispatch_semaphore_t`
    *   用作锁保证线程安全
    *   控制`GCD`的最大并发数
*   `dispatch_source定时器`替代误差较大的`NSTimer`
*   `AFNetworking`、`SDWebImage`等知名三方库中的`NSOperation`使用
*   ...

## 四、线程池的原理

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/141134_55e30d88_9027123.png "多线程分享1.png")



*   若`线程池大小`小于`核心线程池大小`时
    *   创建线程执行任务
*   若`线程池大小`大于等于`核心线程池大小`时
    1.  先判断线程池工作队列是否已满
    2.  若没满就将任务push进队列
    3.  若已满时，且`maximumPoolSize>corePoolSize`，将创建新的线程来执行任务
    4.  反之则交给`饱和策略`去处理

| 参数名 | 代表意义 |
| --- | --- |
| corePoolSize | 线程池的基本大小（核心线程池大小） |
| maximumPool | 线程池的最大大小 |
| keepAliveTime | 线程池中超过corePoolSize树木的空闲线程的最大存活时间 |
| unit | keepAliveTime参数的时间单位 |
| workQueue | 任务阻塞队列 |
| threadFactory | 新建线程的工厂 |
| handler | 当提交的任务数超过maxmumPoolSize与workQueue之和时，任务会交给RejectedExecutionHandler来处理 |

饱和策略有如下四个：

*   `AbortPolicy`直接抛出RejectedExecutionExeception异常来阻止系统正常运行
*   `CallerRunsPolicy`将任务回退到调用者
*   `DisOldestPolicy`丢掉等待最久的任务
*   `DisCardPolicy`直接丢弃任务

## 五、栅栏函数异同以及注意点

栅栏函数两个API的**异同**：

*   `dispatch_barrier_async`：可以控制队列中任务的执行顺序
*   `dispatch_barrier_sync`：不仅阻塞了队列的执行，也阻塞了线程的执行

栅栏函数**注意点**：

1.  尽量使用自定义的并发队列：
    *   使用`全局队列`起不到栅栏函数的作用
    *   使用`全局队列`时由于对全局队列造成堵塞，可能致使系统其他调用全局队列的地方也堵塞从而导致崩溃（并不是只有你在使用这个队列）
2.  栅栏函数只能控制同一并发队列：打个比方，平时在使用`AFNetworking`做网络请求时为什么不能用栅栏函数起到同步锁堵塞的效果，因为`AFNetworking`内部有自己的队列




## 六、栅栏函数的读写锁

多读单写功能指的是：可以多个读者同时读取数据，而在读的时候，不能写入数据；在写的过程中不能有其他写者去写。即读者之间是并发的，写者与其他写者、读者之间是互斥的

```
- (id)readDataForKey:(NSString*)key {
    __block id result;
    dispatch_sync(_concurrentQueue, ^{
        result = [self valueForKey:key];
    });
    return result;
}

- (void)writeData:(id)data forKey:(NSString*)key {
    dispatch_barrier_async(_concurrentQueue, ^{
        [self setValue:data forKey:key];
    });
}
```

*   读：`并发同步`获取到值后返回给读者
    *   若使用`并发异步`则会先返回空的`result 0x0`，再通过getter方法获取到值
*   写：写的那个时间段，不能有任何读者+其他写者
    *   `dispatch_barrier_async`满足：等队列中前面的读写任务都执行完了再来执行当前任务

## 七、GCD的并发量

不同于`NSOperation`中可以通过`maxConcurrentOperationCount`去控制并发数，GCD需要通过信号量才能达到效果

```
dispatch_semaphore_t sem = dispatch_semaphore_create(1);
dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);

for (int i = 0; i < 10; i++) {
    dispatch_async(queue, ^{
        NSLog(@"当前%d----线程%@", i, [NSThread currentThread]);
        // 打印任务结束后信号量解锁
        dispatch_semaphore_signal(sem);
    });
    // 由于异步执行，打印任务会较慢，所以这里信号量加锁
    dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
}

--------------------输出结果：-------------------
当前1----线程<NSThread: 0x600001448d40>{number = 3, name = (null)}
当前0----线程<NSThread: 0x60000140c240>{number = 6, name = (null)}
当前2----线程<NSThread: 0x600001448d40>{number = 3, name = (null)}
当前3----线程<NSThread: 0x60000140c240>{number = 6, name = (null)}
当前4----线程<NSThread: 0x60000140c240>{number = 6, name = (null)}
当前5----线程<NSThread: 0x600001448d40>{number = 3, name = (null)}
当前6----线程<NSThread: 0x600001448d40>{number = 3, name = (null)}
当前7----线程<NSThread: 0x60000140c240>{number = 6, name = (null)}
当前8----线程<NSThread: 0x600001448d40>{number = 3, name = (null)}
当前9----线程<NSThread: 0x60000140c240>{number = 6, name = (null)}
--------------------输出结果：-------------------
```

> 在面试中更多会考验开发人员对于指定场景的多线程知识，接下来就来看看一些综合运用

## 八、综合运用一

#### 1.下列代码会报错吗？

```
int a = 0;
while (a < 5) {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        a++; 
    });
}
```

*   编译会报错`Variable is not assignable (missing __block type specifier)`
    *   这块属于block的知识
*   捕获外界变量并进行修改需要加`__block int a = 0;`
    *   这块内容在接下来的`block`会讲到

#### 2.下列代码的输出

```
__block int a = 0;
while (a < 5) {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        a++;
    });
}
NSLog(@"%d", a);
```

*   会输出`0`吗？
    *   不会，尽管是并发异步执行，但是有`while`在，不满足条件就不会跳出循环
*   会输出`1~4`吗？
    *   不会（原因请往下看）
*   会输出`5`吗？
    *   有可能（原因请往下看）
*   会输出`6~∞`吗？
    *   极有可能

分析：

*   刚进入while循环时，`a=0`，然后进行`a++`
*   由于是`异步并发`会开辟子线程并有可能超车完成
    *   当`线程2`在`a=0`执行`a++`时，`线程3`有可能已经完成了`a++`使`a=1`
    *   由于是操作同一片内存空间，`线程3`修改了`a`导致`线程2`中`a`的值也发生了变化
    *   慢一拍的`线程2`对已经是`a=1`进行`a++`操作
*   同理还有`线程4`、`线程5`、`线程n`的存在
    *   可以这么理解，线程2、3、4、5、6同时在`a=0`时操作`a`
    *   线程2、3、4、5按顺序完成了操作，此时`a=4`
    *   然后`线程6`开始操作了，但是它还没执行完就跳到了下一次循环了开辟了`线程7`开始`a++`
    *   当`线程6`执行结束修改`a=5`之后来到`while条件判断`就会跳出循环
    *   然而`I/O`输出比较耗时，此时线程7又刚好完成了再打印，就会输出`大于5`
*   也有那么种理想情况，`异步并发`都比较听话，刚好在`a=5`时没有子线程
    *   此时就会输出`5`

> 如果还没有明白可以在while循环中添加打印代码

```
__block int a = 0;
while (a < 5) {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        NSLog(@"%d————%@", a, [NSThread currentThread]);
        a++;
    });
}
NSLog(@"此时的%d", a);
```

![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/141148_6034fd03_9027123.png "多线程分享2.png")


> 打印信息证明while外面的打印已经执行，但是子线程还是有可能在对a进行操作的

#### 3.怎么解决线程不安全？

可能有的小伙伴说这种需求不存在，但是我们只管解决便是了

此时我们应该能想到一下几种解决方案：

*   同步函数替换异步函数
*   使用栅栏函数
*   使用信号量

1.  同步函数替换异步函数

*   结果：能满足需求
*   效果：不是很好——能使用异步函数去使唤子线程为什么不用呢（虽然会消耗内存，但是效率高）

2.  使用栅栏函数

*   结果：能满足需求
*   效果：一般
    *   首先`栅栏函数`和`全局队列`搭配使用会无效，需要更换队列类型；
    *   其次`dispatch_barrier_sync`会阻塞线程，影响性能
    *   而`dispatch_barrier_async`不能满足需求，它只能控制前面的任务执行完毕再执行栅栏任务（控制任务执行）可是异步栅栏执行也是在子线程中，当`a=4`时会先继续下一次循环添加任务到队列中，再来异步执行栅栏任务（不能控制任务的添加）

```
__block int a = 0;
dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);
while (a < 5) {
    dispatch_async(queue, ^{
        a++;
    });
    dispatch_barrier_async(queue, ^{});
}

NSLog(@"此时的%d", a);
sleep(1);
NSLog(@"此时的%d", a);

--------------------输出结果：-------------------
此时的5
此时的17
--------------------输出结果：-------------------
```

3.  使用信号量

*   结果：能满足需求
*   效果：很好、简洁效率高

```
__block int a = 0;
dispatch_semaphore_t sem = dispatch_semaphore_create(0);
while (a < 5) {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        a++;
        dispatch_semaphore_signal(sem);
    });
    dispatch_semaphore_wait(sem, DISPATCH_TIME_FOREVER);
}

NSLog(@"此时的%d", a);
sleep(1);
NSLog(@"此时的%d", a);

--------------------输出结果：-------------------
此时的5
此时的5
--------------------输出结果：-------------------
```

## 九、综合运用二

#### 1.输出内容

```
dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);
NSMutableArray *marr = @[].mutableCopy;
for (int i = 0; i < 1000; i++) {
    dispatch_async(queue, ^{
        [marr addObject:@(i)];
    });
}
NSLog(@"%lu", marr.count);
```

> *   你：输出一个小于1000的数，因为for循环中是异步操作
> *   面试官：回去等消息吧
> *   然后你回去之后试了下大吃一惊——程序崩了

这是为什么呢？

其实跟`综合运用一`是一样的道理——for循环异步时无数条线程访问数组，造成了线程不安全

#### 2.怎么解决线程不安全？

*   使用串行队列

```
dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_SERIAL);
NSMutableArray *marr = @[].mutableCopy;
for (int i = 0; i < 1000; i++) {
    dispatch_async(queue, ^{
        [marr addObject:@(i)];
    });
}
NSLog(@"%lu", marr.count);

--------------------输出结果：-------------------
998
--------------------输出结果：-------------------
```

*   使用互斥锁

```
dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);
NSMutableArray *marr = @[].mutableCopy;
for (int i = 0; i < 1000; i++) {
    dispatch_async(queue, ^{
        @synchronized (self) {
            [marr addObject:@(i)];
        }
    });
}
NSLog(@"%lu", marr.count);

--------------------输出结果：-------------------
997
--------------------输出结果：-------------------
```

*   使用栅栏函数

```
dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);
NSMutableArray *marr = @[].mutableCopy;
for (int i = 0; i < 1000; i++) {
    dispatch_async(queue, ^{
        [marr addObject:@(i)];
    });
    dispatch_barrier_async(queue, ^{});
}
NSLog(@"%lu", marr.count);
```

#### 3.分析思路

单路千万条，跳跳通罗马——当然除了这三种还有其他办法

*   使用串行队列
    *   虽然效率低，但总归能解决线程安全问题
    *   虽然`串行异步`是任务一个接一个执行，但那是队列中的任务才满足执行规律
    *   要想得到打印结果`1000`，可以在队列中执行
    *   总的来说，能满足需求但不是很有效
*   使用互斥锁
    *   `@synchronized`是个好东西，简单易用还有效，但也没有满足我们的需求
    *   在for循环外使用队列内同步/异步都不能得到`100`
    *   要么先sleep一秒——这样不可控的代码是不可取的的
    *   且在iOS的锁家族中`@synchronized`效率很低
*   使用栅栏函数
    *   栅栏函数可以有效的控制任务的执行
    *   且与`综合运用一`不同，本题中是for循环
    *   至于怎么得到打印结果`1000`，只需要在同一队列中打印即可（栅栏函数的注意点）

```
dispatch_queue_t queue = dispatch_queue_create("Felix", DISPATCH_QUEUE_CONCURRENT);
NSMutableArray *marr = @[].mutableCopy;
for (int i = 0; i < 1000; i++) {
    dispatch_async(queue, ^{
        [marr addObject:@(i)];
    });
    dispatch_barrier_async(queue, ^{});
}
dispatch_async(queue, ^{
    NSLog(@"%lu", marr.count);
});
```

## 写在后面

多线程在日常开发中占有不少份量，同时面试中也是必问模块。但只有基础知识是一成不变的，综合运用题稍有改动就是另外一种类型的知识考量了，而且也有多种解决方案

