#面试题一
**runtime中，SEL、Method 和 IMP有什么区别，使用场景？**

它们之间的关系可以这么解释：一个类（Class）持有一个分发表，在运行期分发消息，表中的每一个实体代表一个方法（Method），它的名字叫做选择子（SEL），对应着一种方法实现（IMP）。

##**具体的分析如下**

>**SEL定义**：typedef struct objc_selector *SEL，代表方法的名称。仅以名字来识别。翻译成中文叫做选择子或者选择器，选择子代表方法在 Runtime 期间的标识符。为 SEL 类型，虽然 SEL 是 objc_selector 结构体指针，但实际上它只是一个 C 字符串。在类加载的时候，编译器会生成与方法相对应的选择子，并注册到 Objective-C 的 Runtime 运行系统。不论两个类是否存在依存关系，只要他们拥有相同的方法名，那么他们的SEL都是相同的。比如，有n个viewcontroller页面，每个页面都有一个viewdidload,每个页面的载入，肯定都是不尽相同的。但是我们可以通过打印，观察发现，这些viewdidload的SEL都是同一个
```
SEL sel = @selector(methodName); 
// 方法名字 NSLog(@"address = %p",sel);
// log输出为 address = 0x1df807e29 因此类方法定义时，尽量不要用相同的名字，就算是变量类型不同也不行。否则会引起重复，例如：

-(void)setWidth:(int)width; -(void)setWidth:(double)width;
  ```
>**IMP定义**：typedef id (*IMP)(id, SEL, ...)，代表函数指针，即函数执行的入口。该函数使用标准的 C 调用。第一个参数指向 self（它代表当前类实例的地址，如果是类则指向的是它的元类），作为消息的接受者；第二个参数代表方法的选择子；... 代表可选参数，前面的 id 代表返回值。

>**Method定义**：typedef struct objc_method *Method，Method 对开发者来说是一种不透明的类型，被隐藏在我们平时书写的类或对象的方法背后。它是一个 objc_method 结构体指针，我们可以看到该结构体中包含一个SEL和IMP，实际上相当于在SEL和IMP之间作了一个映射。有了SEL，我们便可以找到对应的IMP，从而调用方法的实现代码。 objc_method 的定义为：
  ```
/// Method
struct objc_method {
    SEL method_name; 
    char *method_types;
    IMP method_imp;
 };  
```
方法名 method_name 类型为 SEL，前面提到过相同名字的方法即使在不同类中定义，它们的方法选择器也相同。

方法类型 method_types 是个 char 指针，其实存储着方法的参数类型和返回值类型，即是 Type Encoding 编码。

method_imp 指向方法的实现，本质上是一个函数的指针，就是前面讲到的 Implementation。

#面试题二
**举例说明 Swift 中 map、filtter、reduce的作用？**

**map: **方法作用是把数组[T]通过闭包函数把每一个数组中的元素变成U类型的值，最后组成数组[U]。定义如下： func map(transform: (T) -> U) -> [U]
```
// 将示例数组，每个数字都加10，获得一个新的数组： let numberArray = [1,2,3,4,5] var result = numberArray.map({$0 + 10}) print(result) 
// [11, 12, 13, 14, 15] 
// 在数字后拼接字符串，返回新的数组 let numberArray = [1,2,3,4,5] let resultArray = numberArray.map({"($0)只"}) print(resultArray) 
// 输出结果：["1只", "2只", "3只", "4只", "5只"]
```
>**拓展**：flatMap 更加强大，可以传入N个处理方法，将处理后得到数据，组合到同一个数组中
```
let numberArray = [1,2,3,4,5]
resultArray = numberArray.flatMap({["\($0)个","\($0 )只"]})
print(resultArray)
//输出结果：
["1个", "1只", "2个", "2只", "3个", "3只", "4个", "4只", "5个", "5只"]
```
filter就是筛选的功能，参数是一个用来判断是否筛除的筛选闭包，根据闭包函数返回的Bool值来过滤值。为True则加入到结果数组中。
定义如下：
```
 func filter(includeElement: (T) -> Bool) -> [T]

// 找出数组中大于2的数 let numberArray = [1,2,3,4,5] let filteredArray = numberArray.filter({$0 > 2}) print(filteredArray) // 输出结果： [3, 4, 5]

reduce的作用给定一个类型为U的初始值，把数组[T]中每一个元素传入到combine的闭包函数里面，通过计算得到最终类型为U的结果值。定义如下： func reduce(initial: U, combine: (U, T) -> U) -> U

// 求和 let numberArray = [1, 2, 3] let sum1 = numberArray.reduce(10) { (x, y) -> Int in print("x = (x) y = (y)") return x + y } let sum2 = numberArray.reduce(10) { $0 + $1 }

print("sum=(sum1)") // 16 print("sum1=(sum2)") // 16
```
#面试题三
**UIView与CALayer之间的关系是怎样的？**
UIView是专门处理事件传递与视图响应的，而CALayer 是负责UI视图的显示工作的,二者的关系用到了6大设计原则中的 单一职责原则 ，也就是二者区分工作的原因：`单一职责原则`

#面试题四
**常见的内存泄漏有哪些情况？如何排查和避免？**
**内存泄漏原理**：在百度上的解释就是“程序中已动态分配的堆内存由于某种原因程序未释放或无法释放，造成系统内存的浪费，导致程序运行速度减慢甚至系统崩溃等严重后果”。

##常见的内存泄漏情况：

`情况一`：**对象之间的循环引用问题** 循环引用的实质：多个对象相互之间有强引用，不能施放让系统回收。解决办法：使用 weak 打破对象之间的相互强引用
`情况二`：**block的循环引用** block在copy时都会对block内部用到的对象进行强引用的。解决办法使用：使用`__weak`打破循环的方法只在 ARC 下才有效，在 MRC 下应该使用`__block`
```
__weak typeof(self) weakSelf = self; self.myBlock = ^() { 
// 除了下面的还有 调用 self的一些属性等等 [weakSelf doSomething] 
};
```
`情况三`： delegate 的循环引用 delegate是委托模式.委托模式是将一件属于委托者做的事情，交给另外一个被委托者来处理，在这里我们可能会出现委托者和被委托人之间的相互强引用问题；

**解决办法：**
 >在声明 delegate 属性的时候 用weak 进行弱引用 或者 通过中间对象(代理对象)的方式来解决(效率更加高的中间对象NSProxy：不需要进行发送消息和再动态解析，直接进行消息转发)
```
@property(nonatomic, weak) id delegate;
```
`情况四`：CADisplayLink、NSTimer会对target产生强引用，如果target又对它们产生强引用，那么就会引发循环引用；
**解决办法：**
 >NSTimer 有一个block的方法，我们可以利用block的弱指针来解决__weak typeof(self) weakSelf = self; 传 weakSelf 进去

`情况五`：通知的循环引用 iOS9 以后，一般的通知，都不再需要手动移除观察者，系统会自动在dealloc 的时候调用 [[NSNotificationCenter defaultCenter] removeObserver: self]。iOS9 以前的需要手动进行移除。
**原因是**：iOS9 以前观察者注册时，通知中心并不会对观察者对象做 retain 操作，而是进行了 unsafe_unretained 引用，所以在观察者被回收的时候，如果不对通知进行手动移除，那么指针指向被回收的内存区域就会成为野指针，这时再发送通知，便会造成程序崩溃。
从 iOS9 开始通知中心会对观察者进行 weak 弱引用，这时即使不对通知进行手动移除，指针也会在观察者被回收后自动置空，这时再发送通知，向空指针发送消息是不会有问题的。

**建议最好加上移除通知的操作：**
```
(void)dealloc { [[NSNotificationCenter defaultCenter] removeObserver:self.observer name:@"name" object:nil]; }
```
`情况六`：**WKWebView 造成的内存泄漏**\ 总的来说，WKWebView 不管是性能还是功能，都要比 UIWebView 强大很多，本身也不存在内存泄漏问题，但是，如果开发者使用不当，还是会造成内存泄漏。
**请看下面这段代码：**
```
@property (nonatomic, strong) WKWebView *wkWebView;

(void)webviewMemoryLeak { WKWebViewConfiguration *config = [[WKWebViewConfiguration alloc] init];
 config.userContentController = [[WKUserContentController alloc] init]; 
[config.userContentController addScriptMessageHandler:self name:@"WKWebViewHandler"]; 
_wkWebView = [[WKWebView alloc] initWithFrame:self.view.bounds configuration:config]; 
_wkWebView.backgroundColor = [UIColor whiteColor]; [self.view addSubview:_wkWebView]; 
NSURLRequest *requset = [NSURLRequest requestWithURL:[NSURL URLWithString:@"[https://www.baidu.com](https://www.baidu.com/)"]]; 
[_wkWebView loadRequest:requset];
 } 
```
这样看起来没有问题，但是其实 “addScriptMessageHandler” 这个操作，导致了 wkWebView 对 self 进行了强引用，然后 “addSubview”这个操作，也让 self 对 wkWebView 进行了强引用，这就造成了循环引用。
**解决方法就是在合适的机会里对 “MessageHandler” 进行移除操作：**
```
(void)viewDidDisappear:(BOOL)animated { 
[super viewDidDisappear:animated];
 [_wkWebView.configuration.userContentController removeScriptMessageHandlerForName:@"WKWebViewHandler"];
 }
```


##内存泄漏的查询

**第一种查询方式：**Analyze 静态分析 （command ＋ shift ＋ b）也就是编译，
**主要分析以下四种问题：**
`逻辑错误`：访问空指针或未初始化的变量等；
`内存管理错误`：如内存泄漏等；
`声明错误`：从未使用过的变量；
`Api调用错误`：未包含使用的库和框架。
**第二种查询方式：**Instruments中的Leak动态分析内存泄漏，product－>profile －>leaks 打开工具主窗口
**第三种：**Facebook早已开源了一款检测内存问题的三方库FBRetainCycleDetector

##面试题五
**简单的描述一下 SDWebImage的缓存策略？**
首先，SDWebImage 的图片缓存采用的是 Memory(内存) 和 Disk(硬盘) 双重 Cache 机制，SDImageCache 中有一个叫做 memCache 的属性，它是一个 NSCache 对象，用于实现我们对图片的 Memory Cache，其实就是接受系统的内存警告通知，然后清除掉自身的图片缓存。
Disk Cache，也就是文件缓存，SDWebImage 会将图片存放到 NSCachesDirectory 目录中，然后为每一个缓存文件生成一个 md5 文件名, 存放到文件中。 
**整体机制如下：**

`Memory(内存)中查找：`SDImageCache 类的 queryDiskCacheForKey 方法，查询图片缓存，queryDiskCacheForKey 方法内部， 先会查询 Memory Cache ，如果查找到就直接返回，反之进入下面的硬盘查找。

`Disk(硬盘) 中查找：`如果 Memory Cache 查找不到， 就会查询 Disk Cache，查询 Disk Cache 的时候有一个小插曲，就是如果 Disk Cache 查询成功，还会把得到的图片再次设置到 Memory Cache 中。
 这样做可以最大化那些高频率展现图片的效率。
如果找不到就进入下面的网络下载。

`网路下载：`请求网络使用的是 imageDownloader 属性，这个示例专门负责下载图片数据。
 如果下载失败， 会把失败的图片地址写入 failedURLs 集合，为什么要有这个 failedURLs 呢， 因为 SDWebImage 默认会有一个对上次加载失败的图片拒绝再次加载的机制。
 也就是说，一张图片在本次会话加载失败了，如果再次加载就会直接拒绝，SDWebImage 这样做可能是为了提高性能。
如果下载图片成功了，接下来就会使用 [self.imageCache storeImage] 方法将它写入缓存 ，同时也会写入硬盘，并且调用 completedBlock 告诉前端显示图片。

`Disk(硬盘)缓存清理策略：`SDWebImage 会在每次 APP 结束的时候执行清理任务。 清理缓存的规则分两步进行。 第一步先清除掉过期的缓存文件。 如果清除掉过期的缓存之后，空间还不够。 那么就继续按文件时间从早到晚排序，先清除最早的缓存文件，直到剩余空间达到要求。

##面试题六
**用递归算法计算 1 到 n 的和 **
如下，递归主要是有终止的条件
```
/// 用递归方式获取 1~n 的相加之和 
///  - Parameter n: 数字几 
///  - Returns: 返回和 func getSum(_ n: Int) -> Int { return (n == 1) ? 1 : n + getSum(n - 1); }
```
##面试题七
**简述 MVC、MVP、MVVM 模式**
这三种模式均为MV*模式，M为模型层，V为视图层，都是希望能更好的对模型、视图与逻辑层的解耦。

*  ` MVC模型中，C为（controller）。`
主要处理逻辑为：View触发事件，controller响应并处理逻辑，调用Model，Model处理完成后将数据发送给View，View更新。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/143604_2659ff5b_9027123.png "十面埋伏1.png")


*   `MVP模型中`，P为Presenter，并以Presenter为核心，负责从model获取数据，并填充到View中。
该模型使得Model和View不再有联系，且View被称为“被动视图”，暴露出setter接口。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/143630_83c7a4aa_9027123.png "十面埋伏2.png")


*   `MVVM模型中`，VM为ViewModel，同样是以VM为核心，但是不同于MVP，MVVM采用了数据双向绑定的方案，替代了繁琐复杂的DOM操作。
该模型中，View与VM保持同步，View绑定到VM的属性上，如果VM数据发生变化，通过数据绑定的方式，View会自动更新视图；
VM同样也暴露出Model中的数据。
![输入图片说明](https://images.gitee.com/uploads/images/2021/0531/143643_470f1225_9027123.png "十面埋伏3.png")

##面试题八
**以下代码运行结果如何？**
```
(void)viewDidLoad { 
[super viewDidLoad]; 
NSLog(@"1"); 
dispatch_sync(dispatch_get_main_queue(), ^{ NSLog(@"2"); }); 
NSLog(@"3"); 
} 
```
只打印一个 1，原因是 viewDidLoad 和 dispatch_sync(dispatch_get_main_queue() 之间存在队列等待， viewDidLoad 方法是在串行队列优先执行完，而GCD的闭包要等到 viewDidLoad 执行完才能执行完，而 NSLog(@"3"); 要执行要先等GCD的闭包 执行完，相互等待，死锁

##面试题九
给定一个数组，用选择排序或者冒泡排序实现一个方法给数组排序

###冒泡排序
 将前后每两个数进行比较，较大的数往后排，一轮下来最大的数就排到最后去了。
然后再进行第二轮比较，第二大的数也排到倒数第二了，以此类推，里面一层循环在某次扫描中没有执行交换，则说明此时数组已经全部有序列，无需在扫描了。
因此，增加一个标记，每次发生交换，就标记，如果某次循环没有标记，则说明已经完成排序。
```
NSMutableArray *array = [NSMutableArray arrayWithArray:@[@3, @5, @1, @9]];
 BOOL isSort = true; for (int i = 0; i < array.count - 1; i++) { 
isSort = false; for (int j = 0; j < array.count - i- 1; j++) { 
if (array[j] > array[j + 1]) { 
[array exchangeObjectAtIndex:j withObjectAtIndex:j + 1]; 
isSort = YES;
 } 
} if (!isSort) { 
break;
 } 
}
```
###选择排序
 简单选择排序的基本思想：(从小到大) 第1趟，在待排序记录r[1]~r[n]中选出最小的记录，将它与r[1]交换； 第2趟，在待排序记录r[2]~r[n]中选出最小的记录，将它与r[2]交换； 以此类推，第i趟在待排序记录r[i]~r[n]中选出最小的记录，将它与r[i]交换，使有序序列不断增长直到全部排序完毕。
```
NSMutableArray *arr = [NSMutableArray arrayWithObjects:@1,@100,@4,@3,nil]; 
for (int i = 0; i< arr.count; i++) { 
for (int j = i + 1; j < arr.count; 
j++) { 
if (arr[i] > arr[j]) { 
[arr exchangeObjectAtIndex:i withObjectAtIndex:j]; 
} 
}
 } 
NSLog(@"%@",arr);
```
##面试题十
给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标

**方法一**：暴力法，遍历每个元素 x，并查找是否存在一个值与 target−x 相等的目标元素。
获取所有的可能
```
/// 给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标 
/// - Parameters: 
/// - nums: 数组 
/// - target: 目标值 
/// - Returns: 数组下标的元祖 func twoSum( nums: [Int], target: Int) -> [Any] { 
guard nums.count >= 2 else {
 return [0] 
} var indexArray: [Any] = [] for i in 0..<nums.count { let j = i + 1 for j in j..<nums.count { if target - nums[i] == nums[j] { 
indexArray.append((i,j)) 
} 
} 
} return indexArray 
} 
测试：print(twoSum([1,3,5,7], 8))，结果是：[(0, 3), (1, 2)]
```
**方法二**：hashmap 一次迭代 在进行迭代并将元素插入到表中的同时，我们还会回过头来检查表中是否已经存在当前元素所对应的目标元素。
如果它存在，那我们已经找到了对应解，并立即将其返回。
下面是求多个下标组
```
func twoSum( nums: [Int], target: Int) -> [Any] { 
guard nums.count >= 2 else { 
return [0] 
} var tempHash: [Int : Int] = [:] var result : [Int] = [] var indexArray: [Any] = [] for (i, value) in nums.enumerated() { 
if let index = tempHash[target - value]{
 result.append(index) result.append(i) indexArray.append((index,i)) 
} tempHash[value] = i 
} return indexArray 
}
```
