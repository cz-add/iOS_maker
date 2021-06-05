## 基础部分

### 1.为什么说OC是一门动态的语言?

*   动态和静态是相对的,OC通过`runtime`运行时机制可以做到纯静态语言做不到的事情:例如动态地增加、删除、替换`ivar`或者方法等
*   Objective-C 使用的是“消息结构”并非“函数调用”:使用消息结构的的语言,其运行时所应执行的代码由运行期决定;而使用函数调用的语言,则由编译器决定

### 2.讲一下MVC和MVVM，MVP？

*   MVC
    *   M:业务数据, V:视图,负责展示 C:控制器,负责协调M、V
    *   C作为M和V之间的连接, 负责响应视图事件,界面的跳转,view的声明周期,获取业务数据, 然后将处理后的数据输出到界面上做相应展示, 在数据有更新时, C需要及时提交相应更新到界面展示。View和Model之间没有直接的联系,[AppleMVC规范](https://developer.apple.com/library/content/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html#//apple_ref/doc/uid/TP40010810-CH14-SW14),理想的模型图如下:
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151251_76bd4ffd_9027123.png "基础1.png")
      *   在实际开发中,Model往往只有轻量级的数据,甚至.m中没有任何实现,View和Controller成对出现,导致Controller中,越来越多的属性、协议、网络数据处理等,View和Controller紧紧耦合在一起:
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151309_48c511f0_9027123.png "基础2.png")
*   MVP
    *   M:业务数据, V:视图, P:协调器,业务处理层
    *   P:业务逻辑的处理者,作为M、V的桥梁。当获取到数据后进行相应处理, 处理完成后会通知绑定的View数据有更新,View收到更新通知后从P获取格式化好的数据进行页面渲染。相比较MVC,MVP把业务逻辑和View的展示分离开：
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151329_7afd8987_9027123.png "基础3.png")
*   MVVM
    *   M:业务数据, V:视图, VM:视图模型,展示逻辑
    *   相比较MVC,View/ViewController不直接引用Model,而是通过ViewModel,ViewModel负责用户交互逻辑，视图显示逻辑，发起网络请求等
    *   听说MVVM和RAC更配~
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151432_24b7d06a_9027123.png "基础4.png")
关于设计模式,这篇[杂谈: MVC/MVP/MVVM](http://www.cocoachina.com/ios/20170313/18870.html)和[iOS 架构模式--解密 MVC，MVP，MVVM以及VIPER架构](http://www.cocoachina.com/ios/20160108/14916.html)有详细介绍




### 3.为什么代理要用weak？代理的delegate和dataSource有什么区别？block和代理的区别?

*   避免循环引用,`weak`表示该对象并不持有该`delegate`对象,`delegate`对象的销毁由外部控制;如果用`strong`则该对象强引用`delegate`，外界不能销毁`delegate`对象，会导致循环引用`(Retain Cycles)`
*   `delegate`是委托的意思,在`OC`中表示一个类委托另一个类实现某个方法。当一个对象接受到某个事件或者通知的时候，会向它的`delegate`对象查询它是否能够响应这个事件或者通知，如果可以这个对象就会给它的`delegate`对象发送一个消息（执行一个方法调用）。 `datasource`字面是数据源，一般和`delegate`伴生，这时数据源处理的数据就是`delegate`中发送委托的类中的数据，并通过`datasource`发送给接受委托的类。 `Instead of being delegated control of the user interface, a data source is delegated control of data.`官网上这句话解释的比较好，我们可以发现，`delegate`控制的是UI，是上层的东西；而`datasource`控制的是数据。他们本质都是回调，只是回调的对象不同。[官网原文](https://developer.apple.com/library/ios/#documentation/General/Conceptual/DevPedia-CocoaCore/Delegation.html)
*   `delegate`和`block`都可以实现回调传值。`block`写法简练,可以直接访问上下文,代码阅读性好,适合与状态无关的操作,更加面向结果,使用过程中需要注意避免造成循环引用。`delegate`更像一个生产流水线，每个回调方法是生产线上的一个处理步骤，一个回调的变动可能会引起另一个回调的变动,其更加面向过程,当有多个相关方法时建议使用`delegate`。

### 4.属性的实质是什么？包括哪几个部分？属性默认的关键字都有哪些？@dynamic关键字和@synthesize关键字是用来做什么的？

*   属性`@"property" = ivar + getter + setter`
*   原子性: `nonatomic`和`atomic`
*   读写特性: `readwrite`和`readonly`
*   内存管理特性: `assign`:修饰简单数据类型。 `weak`:弱引用,多数用来修饰`delegate`和`outlet`。 `copy`:拷贝,多用于修饰`NSString`、`block`等,其作为属性修饰符时是将`_property`先`release (_property release)`，然后拷贝参数内容`(_property copy)`,创建一块新的内存地址，最后`_property = property`。strong:强引用,其作为属性修饰符时是将`_property`先`release (_property release)`，然后将参数`retain(_property retain)`,最后`_property = property`。`retain`:MRC下特有,等同于`strong`。`unsafe_unretained`:和`weak` 一样，唯一的区别便是，对象即使被销毁，指针也不会自动置空， 此时指针指向的是一个无用的野地址。如果使用此指针，程序会抛出 `BAD_ACCESS` 的异常。
*   `@synthesize` 让编译器自动生成`getter/setter`方法。当有自定义的存取方法时，会覆盖该方法
*   `@dynamic`告诉编译器，不自动生成`getter/setter`方法。由自己实现存取方法或存取方法在运行时动态创建绑定

### 5.属性的默认关键字是什么？

ARC下:

*   基本数据类型默认关键字是 `atomic,readwrite,assign`
*   其他类型默认关键字是 `atomic,readwrite,strong`

MRC下:

*   基本数据类型默认关键字是 `atomic,readwrite,assign`
*   其他类型默认关键字是 `atomic,readwrite,retain`

### 6.NSString为什么要用copy关键字，如果用strong会有什么问题？（注意：这里没有说用strong就一定不行。使用copy和strong是看情况而定的）

```
@interface Person : NSObject

@property(nonatomic ,copy)NSString *cpName;
@property(nonatomic ,strong)NSString *stName;

@end

@implementation Person
@end

main(){
    NSMutableString *string = [NSMutableString stringWithFormat:@"name"];

    Person *person = [[Person alloc]init];
    person.cpName = string;
    person.stName = string;

    [string appendString:@"name"];
    NSLog(@"cpName:%@ stName:%@",person.cpName,person.stName);
}
```

打印结果如下:

```
cpName:name stName:namename
```

至于为什么要用copy, 大概是为了防止mutableString被无意中修改。

### 7.如何令自己所写的对象具有拷贝功能?

*   若想让自己所写的对象具有拷贝功能，则需实现 `NSCopying` 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 `NSCopying` 与 `NSMutableCopying` 协议。
*   要注意浅拷贝/深拷贝的不同。

### 8.可变集合类 和 不可变集合类的 copy 和 mutablecopy有什么区别？如果是集合是内容复制的话，集合里面的元素也是内容复制么？

```
//不可变字符串
NSString *string = @"string";
NSString *cpString1 = [string copy];            //指针拷贝
NSString *mcpString1 = [string mutableCopy];    //内容拷贝,生成可变对象
NSMutableString *mstr = [string mutableCopy];   //同上
NSLog(@"%p %p %p %p",string,cpString1,mcpString1,mstr);
打印结果如下:
0x100001070 0x100001070 0x10051bbc0 0x100600e10

//可变字符串
NSMutableString *mstr1 = [[NSMutableString alloc]initWithString:@"abcde"];
NSMutableString *mstr2 = [mstr1 copy];          //内容拷贝,生成不可变字符串
NSString *cpstring2 = [mstr1 copy];             //同上
NSMutableString *mstr3 = [mstr1 mutableCopy];   //内容拷贝
NSLog(@"%p %p %p %p",mstr1,mstr2, cpstring2,mstr3);
打印结果如下:
0x102063110 0x656463626155 0x656463626155 0x102063160
```

`NS* NSMutable*`等集合类的`copy、mutableCopy`同上述一样,需要注意的是,集合里面的元素并没有内容拷贝!若集合层级很多,且需要完全内容拷贝,可以利用`NSKeyedArchiver`实现。

### 9.为什么IBOutlet修饰的UIView也适用weak关键字？

*   ~~防止`(Retain Cycles)`~~ 更新:感谢各位指出错误!

*   在storyboard或者xib中创建的UIView,本身会被它的`superView`强引用,以`UILable`为例子:

    `UIViewController -> UIView -> subView -> UILable`

    此时控件拖线会默认为`weak`属性,因为`UIlable`已经被`UIView`**拥有**,当UIViewController释放的时候,UIView释放,UILable才可以释放,所以正常情况下UILable和UIView的生命周期是一样的。设置成`strong`也没什么大问题, 但是当UILable从其父视图UIView上`remove`掉,UIViewController对其还有一个`strong`强引用,UILable无法释放,这时就比较尴尬了...

### 10.nonatomic和atomic的区别？atomic是绝对的线程安全么？为什么？如果不是，那应该如何实现？

*   `nonatomic`:表示非原子性，不安全，但是效率高。
*   `atomic`：表示原子性，安全，但是效率较低。
*   `atomic`：通过锁定机制来确保其原子性,但只是**读/写安全**,不能绝对保证线程的安全，当多线程同时访问的时候，会造成线程不安全。可以使用线程锁来保证线程的安全。

### 11.UICollectionView自定义layout如何实现？

*   `UICollectionViewLayout`是为向`UICollectionView`提供布局信息的类，包括`cell`的布局信息等。
*   `UICollectionView`的自定义布局可以分为三种方式：
*   1.  初始化时传入的UICollectionViewLayout对象，通过设置UICollectionViewLayout对象属性的值可以设置item的基本布局，包括大小，间距等。
    2.  实现UICollectionViewLayoutDelegate协议对应的方法，返回布局需要的值。
    3.  继承UICollectionViewLayout类实现自定义的MyCollectionViewLayout,重写相关方法返回自定义的布局。

### 12.用StoryBoard开发界面有什么弊端？如何避免？

*   难以维护,难以定位问题。
*   ...

### 13.进程和线程的区别？同步异步的区别？并行和并发的区别？

*   进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位
*   进程中所包含的一个或多个执行单元称为线程`thread`
*   同步：多个任务情况下，一个任务A执行结束，才可以执行另一个任务B。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151454_f262f223_9027123.png "基础5.png")
*   异步：多个任务情况下，一个任务A正在执行，同时可以执行另一个任务B。任务B不用等待任务A结束才执行。异步虽然具有开启新线程的能力，但是并不一定开启新线程,跟任务所指定的队列类型有关。**同步和异步的主要区别在于会不会阻塞当前线程**。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151508_4df1e3b6_9027123.png "基础6.png")

*   并行：指两个或多个事件在同一时刻发生。多核CUP同时开启多条线程供多个任务同时执行，互不干扰。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151520_584838b6_9027123.png "基础7.png")

*   并发：指两个或多个事件在同一时间间隔内发生。可以在某条线程和其他线程之间反复多次进行上下文切换，看上去就好像一个CPU能够并且执行多个线程一样。其实是伪异步。如下并发图，在同一线程，任务A先执行了20%，然后A停止，任务B重新开始接管线程开始执行。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151535_aaee3dd8_9027123.png "基础8.png")

*   并发的关键是你有处理多个任务的能力，不一定要同时。 并行的关键是你有**同时**处理多个任务的能力。

### 14.线程间通信？

*   在1个进程中，线程往往不是孤立存在的，多个线程之间需要经常进行通信。可以利用pthread,NSthread,GCD,NSOperation进行相关操作。如:
*   `performSelector`函数

```
- (void)performSelectorOnMainThread:(SEL)aSelector withObject:(id)arg waitUntilDone:(BOOL)wait;
- (void)performSelector:(SEL)aSelector onThread:(NSThread *)thr withObject:(id)arg waitUntilDone:(BOOL)wait;
```

*   GCD

```
dispatch_async(otherQueue, ^{
    // dosth
});

```

*   NSOperation

```
[otherQueue addOperationWithBlock:^{
    // dosth
}];
```

### 15.GCD的一些常用的函数？（group，barrier，信号量，线程同步）

*   `group` : 当所有任务都执行完成之后，才执行`dispatch_group_notify`中的任务 dosthNotify。

```

- (void)gcdGroup{
    dispatch_group_t group =  dispatch_group_create();

    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // dosth1;
    });
    dispatch_group_async(group, dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        // dosth2;
    });
    dispatch_group_notify(group, dispatch_get_main_queue(), ^{
        // dosthNotify;
    });
}
```

*   `barrier` : 栅栏方法,当栅栏前一组操作执行完之后，才能开始执行后一组方法

```
- (void)barrier {
    dispatch_queue_t queue = dispatch_queue_create("net.bujige.testQueue", DISPATCH_QUEUE_CONCURRENT);

    dispatch_async(queue, ^{
        // dosth1;
    });
    dispatch_async(queue, ^{
        // dosth2;
    });
    dispatch_barrier_async(queue, ^{
        // doBarrier;
    });
    dispatch_async(queue, ^{
        // dosth4;
    });
    dispatch_async(queue, ^{
        // dosth5;
    });
}
```

*   信号量 : `dispatch_semaphore`。
*   主要作用:
    *   1.保持线程同步，将异步执行任务转换为同步执行任务

    ```
      dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
      dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
      dispatch_async(queue, ^{
          // dosth1
          // 使信号量+1并返回
          dispatch_semaphore_signal(semaphore);
      });
      // 若信号的信号量为0，则会阻塞当前线程，直到信号量大于0或者经过输入的时间值；若信号量大于0，则会使信号量减1并返回，程序继续住下执行
      dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
      // dosth2 ,只有当dosth1执行完,信号量+1之后,才会执行这里
    ```

    *   2.保证线程安全，为线程加锁:信号总量设为 1 时也可以当作锁来用

### 16.如何使用队列来避免资源抢夺？

*   将需要访问同一块资源的任务添加到一个非异步并行队列执行
*   使用GCD的group,barrier,semaphore函数等
*   线程加锁,如 : @synchronized 关键字加锁、NSLock 对象锁、NSConditionLock条件锁、NSRecursiveLock递归锁、pthread_mutex 互斥锁等。

### 17.数据持久化的几个方案（fmdb用没用过）

*   plist文件
*   preference偏好设置
*   NSKeyedArchiver
*   SQLite 3
*   CoreData
*   fmdb
*   realm

### 18.说一下AppDelegate的几个方法？从后台到前台调用了哪些方法？第一次启动调用了哪些方法？从前台到后台调用了哪些方法？

```
// 当应用程序启动时（不包括已在后台的情况下转到前台），调用此回调。launchOptions是启动参数，假如用户通过点击push通知启动的应用，这个参数里会存储一些push通知的信息
– (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions NS_AVAILABLE_IOS(3_0);
– (void)applicationDidBecomeActive:(UIApplication *)application;

//应用即将从前台状态转入后台
- (void)applicationWillResignActive:(UIApplication *)application;
– (void)applicationDidEnterBackground:(UIApplication *)application NS_AVAILABLE_IOS(4_0);

//从后台到前台调用了:
– (void)applicationWillEnterForeground:(UIApplication *)application NS_AVAILABLE_IOS(4_0);
– (void)applicationDidBecomeActive:(UIApplication *)application;
```

App启动过程如下:
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/151548_47ae4455_9027123.png "基础9.png")


### 19.NSCache优于NSDictionary的几点？

*   NSCache线程安全,在多线程操作时,不需要手动加锁
*   NSCache按照LRU规则,会对超出限制的数据进行自动清除,并且在系统发出低内存通知时，会自动删减缓存
*   NSCache的Key只是对对象的strong引用，对象不需要实现NSCopying协议，NSCache也不会像NSDictionary一样拷贝键。

### 20.知不知道Designated Initializer(指定初始化)？使用它的时候有什么需要注意的问题？

*   在OC中,对象的生成分为两步:内存分配,初始化实例变量

```
NSObject *object = [[NSObject alloc] init];
```

类方法`+ alloc`，其根据要创建的实例对象对应的类来分配足够的内存空间。除了分配内存空间，其实`+ alloc`方法还做了其他事情，包括将对象的引用计数记为1，将对象的isa指针指向对应的运行时类对象，以及将对象的成员变量置为对应的0值(0、nil、NULL)。但是`+ alloc`方法返回的对象还是不可用的，在之后完成初始化方法的调用后，对象的创建工作才算完成。初始化方法会设置对象的成员变量为一个正确的合理的值，以及获取一些其他额外的资源。

#### 1.Designated Initializer 指定初始化方法

所有对象都是要初始化的，而且很多情况下，对象在初始化时是需要接收额外的参数，这就可能会提供多个初始化方法。

```
- (instancetype)initWithTimeIntervalSinceReferenceDate:(NSTimeInterval)ti NS_DESIGNATED_INITIALIZER;
- (instancetype)initWithTimeIntervalSinceNow:(NSTimeInterval)secs;
- (instancetype)initWithTimeIntervalSince1970:(NSTimeInterval)secs;
- (instancetype)initWithTimeInterval:(NSTimeInterval)secsToBeAdded sinceDate:(NSDate *)date;
```

根据规范，通常选择一个接收参数最多的初始化方法作为指定初始化方法，真正的数据分配和其他相关初始化操作在这个方法中完成。而其他的初始化方法则作为便捷初始化方法去调用这个指定初始化方法。这样当实现改变时，只要修改指定初始化方法就可以了。便捷初始化方法接收的参数更少，它会在内部调用指定初始化方法时，直接设置未接收参数的默认值。便捷初始化方法也可以不直接调用指定初始化方法，它可以调用其他便捷初始化方法，但不管调用几层，最终是要调用到指定初始化方法的，因为真正的实现操作是在指定初始化方法中完成的。所有初始化方法统一以- init开始。如上例代码所示，`- initWithTimeIntervalSinceReferenceDate`方法是一个指定初始化方法，而其他初始化方法最终是要调用它的。

#### 2.子类实现指定初始化方法

当子类继承父类后实现了新的指定初始化方法，此时如果调用父类中的指定初始化方法则无法调用到子类新实现的初始化逻辑，所以子类同时还要重写父类的指定初始化方法，将其变为一个便捷初始化方法，最终去调用子类自己的指定初始化方法。而为了保证父类初始化逻辑的执行，在子类指定初始化方法中，首先要通过关键字super调用父类的指定初始化方法。 详见[正确编写Designated Initializer的几个原则](http://blog.jobbole.com/65762/)

#### 3.initWithCoder

如果父类也实现了协议，首先要调用父类的- initWithCoder:方法，如果父类没有实现，则调用父类的指定初始化方法。

#### 4.NS_DESIGNATED_INITIALIZER

当在接口中指定初始化方法的后面加上该宏，编译器就会检查我们实现的初始化调用链是否符合规则，并提示相应的警告。

```
- (instancetype)init NS_DESIGNATED_INITIALIZER;复制代码
```

### 21.实现description方法能取到什么效果？

*   当使用log打印该对象时,可以详细的知道该对象的信息,方便代码调试。

### 22.objc使用什么机制管理对象内存？

*   1.MRC(MannulReference Counting)
*   2.Xcode4.2+引入了ARC(Automatic Reference Counting)
*   3.在ObjC中内存的管理是依赖对象引用计数器来进行的：在ObjC中每个对象内部都有一个与之对应的整数（retainCount），叫“引用计数器”，当一个对象在创建之后它的引用计数器为1，当调用这个对象的alloc、retain、new、copy、mutableCopy方法之后该对象retainCount+1（ObjC中调用一个对象的方法就是给这个对象发送一个消息），当调用这个对象的release方法之后它的引用计数器减1，如果一个对象的引用计数器为0，则系统会自动调用这个对象的dealloc方法来销毁这个对象。

*   4.内存管理遵存这一规则:
    *   1.凡是用alloc,retain,new(或使用new开头)，copy(或使用copy开头的方法)，mutableCopy（或使用mutableCopy开头的方法）“创建”对的对象都必须使用release或者autoRelease方法“释放”。
    *   2.谁（在哪里）创建谁释放（哪个类创建，哪个类释放；谁写alloc，谁写release）
*   5.autorelease : 自动释放。 当某对象调用autorelease方法后，在其所在的NSAutoreleasePool废弃时，都将调用其release方法
*   6.autoreleasePool : 自动释放池。
推荐👇：

如果你想一起进阶，不妨添加一下交流群[642363427](https://jq.qq.com/?_wv=1027&k=NZv0AWXz)

