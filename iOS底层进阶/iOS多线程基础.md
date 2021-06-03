#### 什么是进程？
*   进程是指在系统中正在运行的一个应用程序，每个进程之间是相互独立的，系统会给每个进程分配属于自己的内存空间

#### 什么是线程？

*   一个进程想要执行任务，必须得有线程，每个进程至少要有一个线程，进程的所有任务都在线程中执行，线程中的任务执行是串行的，同一时间内一个线程只能执行1个任务。

#### 什么是多线程？

*   多线程就是指一个进程中可以开启多条线程，可以同时执行不同的任务。多线程可以提高程序的执行效率。

#### 多线程在iOS中的使用

*   我们启动一个iOSAPP的时候，系统默认会开启一个线程，我们称为主线程或者UI线程，它主要的作用就是刷新页面处理和用户交互，但是如果我们开发中如果将一些比较耗时的操作放在主线程中，这种容易使我们的应用非常卡顿，影响用户体验。所以平时我们开发人员在开发的过程中，会将一些耗时的操作放在子线程中执行，例如网络请求、加载大图片等操作。
*   开发中我们常用的三种开始多线程的方法:
    (1) **NSThread**:苹果中NS开头的类肯定是面向对象的，使用简单，可直接操作对象。
    (2)**GCD**:这个是非常强大的，苹果推荐使用，充分利用设备多核，平时开发中使用也最多，理解使用步骤，操作起来也是很简单的。之后会详细介绍使用的方式。
    (3)**NSOperation**:这个是基于GCD，更加面向对象了，比GCD更加简单了。

#### 基本原理我们都了解了，下面直接上代码，学习如何使用NSThread开启线程。

*   创建线程，然后启动线程。只需2步就OK了

*   NSThread其他方式开始线程

    ```
    - (void)createThread3
     {
         [self performSelectorInBackground:@selector(run:) withObject:@"jack"];
     }
     - (void)createThread2
     {
          [NSThread detachNewThreadSelector:@selector(run:) toTarget:self  withObject:@"rose"];
     }
    ```

*   线程的状态一般就2种，就绪状态（等待执行），执行状态（正在执行）。

    ```
         [NSThread sleepForTimeInterval:2]; // 让线程睡眠2秒（阻塞2秒）
         [NSThread sleepUntilDate:[NSDate distantFuture]];//distantFuture 遥远的未来，如果写上这句，那么程序执行到这里就直接睡死了，下面的代码不会在执行了。
         [NSThread exit]; // 直接退出线程

    ```

#### 线程的安全

> 在多线程的运作之下，会出现多个任务共享同一资源，这样会造成数据错乱的问题。例如多个窗口买票，三个窗口同一时间都卖同一个座位的票，票只有一张到底怎么分配，所以这样会出现数据错乱的问题，为了解决这个问题，我们使用互斥锁去解决，同一时间只许一个人操作数据，将数据加锁，当前这个人没有解锁之前，别人无法访问资源。同时我们在使用锁的时候注意下，不能出现死锁的现象，死锁就是连个进程访问同一资源而陷入了相互等待。苹果给我们提供了一个关键字@synchronized处理加锁问题。代码如下
> ```
> @synchronized(self) {
> // 先取出总数
> NSInteger count = self.ticketCount;
> if (count > 0) {
> self.ticketCount = count - 1;
> NSLog(@"%@卖了一张票，还剩下%zd张", [NSThread currentThread].name, self.ticketCount);
> } else {
> NSLog(@“票已经卖完了”);
> break;
> }
> }

#### 线程间的通信

> 平时我们在子线中处理一些耗时的操作，处理完之后，我们拿到数据展示到页面上。处理UI只能在主线程中操作。所以如何从子线程中切回到主线程中呢？以下三种方式都可以回到主线程（MainThread）需要注意下。
> ```
> // 回到主线程，显示图片方式1
> [self.imageView performSelector:@selector(setImage:) onThread:[NSThread mainThread] withObject:image waitUntilDone:NO];
> / 回到主线程，显示图片方式2
> [self.imageView performSelectorOnMainThread:@selector(setImage:) withObject:image waitUntilDone:NO];
> / 回到主线程，显示图片方式3
> [self performSelectorOnMainThread:@selector(showImage:) withObject:image waitUntilDone:YES];

## 推荐文章


**[iOS多线程面试题分析](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/iOS%E9%9D%A2%E8%AF%95%E5%90%88%E9%9B%86/iOS%E5%A4%9A%E7%BA%BF%E7%A8%8B%E9%9D%A2%E8%AF%95%E9%A2%98%E5%88%86%E6%9E%90.md)**

**[iOS 多线程 线程间的状态](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/iOS%E5%BA%95%E5%B1%82%E8%BF%9B%E9%98%B6/iOS%20%E5%A4%9A%E7%BA%BF%E7%A8%8B%20%E7%BA%BF%E7%A8%8B%E9%97%B4%E7%9A%84%E7%8A%B6%E6%80%81.md)**

## 多线程视频资料

**[多线程底层解析](https://www.bilibili.com/video/BV1oi4y1j7sP)**


