1\. 你使用过Objective-C的运行时编程（Runtime Programming）么？如果使用过，你用它做了什么？你还能记得你所使用的相关的头文件或者某些方法的名称吗？

Objecitve-C的重要特性是Runtime（运行时）,在#import <objc/runtime.h> 下能看到相关的方法，用过objc_getClass()和class_copyMethodList()获取过私有API;使用

```
objective-cMethod method1 = class_getInstanceMethod(cls, sel1);Method method2 = class_getInstanceMethod(cls, sel2);method_exchangeImplementations(method1, method2);

```

代码交换两个方法，在写unit test时使用到。

2\. 你实现过多线程的Core Data么？NSPersistentStoreCoordinator，NSManagedObjectContext和NSManagedObject中的哪些需要在线程中创建或者传递？你是用什么样的策略来实现的？

3\. Core开头的系列的内容。是否使用过CoreAnimation和CoreGraphics。UI框架和CA，CG框架的联系是什么？分别用CA和CG做过些什么动画或者图像上的内容。（有需要的话还可以涉及Quartz的一些内容）

UI框架的底层有CoreAnimation，CoreAnimation的底层有CoreGraphics。

UIKit |
------------ |
Core Animation |
Core Graphics |
Graphics Hardware|

4\. 是否使用过CoreText或者CoreImage等？如果使用过，请谈谈你使用CoreText或者CoreImage的体验。

CoreText可以解决复杂文字内容排版问题。CoreImage可以处理图片，为其添加各种效果。体验是很强大，挺复杂的。

5\. NSNotification和KVO的区别和用法是什么？什么时候应该使用通知，什么时候应该使用KVO，它们的实现上有什么区别吗？如果用protocol和delegate（或者delegate的Array）来实现类似的功能可能吗？如果可能，会有什么潜在的问题？如果不能，为什么？（虽然protocol和delegate这种东西面试已经面烂了…）

NSNotification是通知模式在iOS的实现，KVO的全称是键值观察(Key-value observing),其是基于KVC（key-value coding）的，KVC是一个通过属性名访问属性变量的机制。例如将Module层的变化，通知到多个Controller对象时，可以使用NSNotification；如果是只需要观察某个对象的某个属性，可以使用KVO。
对于委托模式，在设计模式中是对象适配器模式，其是delegate是指向某个对象的，这是一对一的关系，而在通知模式中，往往是一对多的关系。委托模式，从技术上可以现在改变delegate指向的对象，但不建议这样做，会让人迷惑，如果一个delegate对象不断改变，指向不同的对象。

6\. 你用过NSOperationQueue么？如果用过或者了解的话，你为什么要使用NSOperationQueue，实现了什么？请描述它和GCD的区别和类似的地方（提示：可以从两者的实现机制和适用范围来描述）。

使用NSOperationQueue用来管理子类化的NSOperation对象，控制其线程并发数目。GCD和NSOperation都可以实现对线程的管理，区别是 NSOperation和NSOperationQueue是多线程的面向对象抽象。项目中使用NSOperation的优点是NSOperation是对线程的高度抽象，在项目中使用它，会使项目的程序结构更好，子类化NSOperation的设计思路，是具有面向对象的优点（复用、封装），使得实现是多线程支持，而接口简单，建议在复杂项目中使用。
项目中使用GCD的优点是GCD本身非常简单、易用，对于不复杂的多线程操作，会节省代码量，而Block参数的使用，会是代码更为易读，建议在简单项目中使用。

7\. 既然提到GCD，那么问一下在使用GCD以及block时要注意些什么？它们两是一回事儿么？block在ARC中和传统的MRC中的行为和用法有没有什么区别，需要注意些什么？如何避免循环引用？

使用block是要注意，若将block做函数参数时，需要把它放到最后，GCD是Grand Central Dispatch，是一个对线程开源类库，而Block是闭包，是能够读取其他函数内部变量的函数。

8\. 您是否做过异步的网络处理和通讯方面的工作？如果有，能具体介绍一些实现策略么？

使用NSOperation发送异步网络请求，使用NSOperationQueue管理线程数目及优先级，底层是用NSURLConnetion

9\. 对于Objective-C，你认为它最大的优点和最大的不足是什么？对于不足之处，现在有没有可用的方法绕过这些不足来实现需求。如果可以的话，你有没有考虑或者实践过重新实现OC的一些功能，如果有，具体会如何做？

最大的优点是它的运行时特性，不足是没有命名空间，对于命名冲突，可以使用长命名法或特殊前缀解决，如果是引入的第三方库之间的命名冲突，可以使用link命令及flag解决冲突。

10\. 你实现过一个框架或者库以供别人使用么？如果有，请谈一谈构建框架或者库时候的经验；如果没有，请设想和设计框架的public的API，并指出大概需要如何做、需要注意一些什么方面，来使别人容易地使用你的框架。

抽象和封装，方便使用。首先是对问题有充分的了解，比如构建一个文件解压压缩框架，从使用者的角度出发，只需关注发送给框架一个解压请求，框架完成复杂文件的解压操作，并且在适当的时候通知给是哦难过者，如解压完成、解压出错等。在框架内部去构建对象的关系，通过抽象让其更为健壮、便于更改。其次是API的说明文档。

## 推荐文章

[关于 iOS 性能优化方面的面试题](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/iOS%E9%9D%A2%E8%AF%95%E5%90%88%E9%9B%86/%E5%85%B3%E4%BA%8E%20iOS%20%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%E6%96%B9%E9%9D%A2%E7%9A%84%E9%9D%A2%E8%AF%95%E9%A2%98.md)

[iOS求职之OC面试题](https://gitee.com/cresta-df/i-os-engineers-secret/blob/master/iOS%E9%9D%A2%E8%AF%95%E5%90%88%E9%9B%86/iOS%E6%B1%82%E8%81%8C%E4%B9%8BOC%E9%9D%A2%E8%AF%95%E9%A2%98.md)


