![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/160451_dc2ece63_9027123.jpeg "对1.jpg")

#Runtime相关面试问题

##1.Runtime是什么？

见名知意，其概念无非就是“因为 Objective-C 是一门动态语言，所以它需要一个运行时系统……这就是 Runtime 系统”云云。对博主这种菜鸟而言，Runtime 在实际开发中，其实就是一组C语言的函数。胡适说：“多研究些问题，少谈些主义”，云山雾罩的概念听多了总是容易头晕，接下来我们直接上runtime思维导图帮助大家理清思路：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/160505_e89f6ca6_9027123.png "对2.png")

##2.objc在向一个对象发送消息时，发生了什么？

objc在向一个对象发送消息时，runtime会根据对象的isa指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，如果一直到根类还没找到，转向拦截调用，走消息转发机制，一旦找到 ，就去执行它的实现IMP 。


##3.objc中向一个nil对象发送消息将会发生什么？

如果向一个nil对象发送消息，首先在寻找对象的isa指针时就是0地址返回了，所以不会出现任何错误。也不会崩溃。

##4.objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？

在objc编译时，[obj foo] 会被转意为：objc_msgSend(obj, @selector(foo));。

##5.什么时候会报unrecognized selector的异常？

objc在向一个对象发送消息时，runtime库会根据对象的isa指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，如果，在最顶层的父类中依然找不到相应的方法时，会进入消息转发阶段，如果消息三次转发流程仍未实现，则程序在运行时会挂掉并抛出异常unrecognized selector sent to XXX 。

##6.能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？

不能向编译后得到的类中增加实例变量；

能向运行时创建的类中添加实例变量；

**1.**因为编译后的类已经注册在 runtime 中,类结构体中的 objc_ivar_list 实例变量的链表和 instance_size 实例变量的内存大小已经确定，同时runtime会调用 class_setvarlayout 或 class_setWeaklvarLayout 来处理strong weak 引用.所以不能向存在的类中添加实例变量。                
  **2.**运行时创建的类是可以添加实例变量，调用class_addIvar函数. 但是的在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前,原因同上.

##7.给类添加一个属性后，在类结构体里哪些元素会发生变化？

instance_size ：实例的内存大小；objc_ivar_list *ivars:属性列表

##8.一个objc对象的isa的指针指向什么？有什么作用？

指向他的类对象,从而可以找到对象上的方法Root class (class)其实就是NSObject，NSObject是没有超类的，所以Root class(class)的superclass指向nil。

每个Class都有一个isa指针指向唯一的Meta classRoot class(meta)的superclass指向Root class(class)，也就是NSObject，形成一个回路。每个Meta class的isa指针都指向Root class (meta)。

##9.runtime如何通过selector找到对应的IMP地址？

每一个类对象中都一个方法列表,方法列表中记录着方法的名称,方法实现,以及参数类型,其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现.

##10._objc_msgForward函数是做什么的，直接调用它将会发生什么？

_objc_msgForward是 IMP 类型，用于消息转发的：当向一个对象发送一条消息，但它并没有实现的时候，_objc_msgForward会尝试做消息转发。

##11.runtime如何实现weak变量的自动置nil？知道SideTable吗？

runtime 对注册的类会进行布局，对于 weak 修饰的对象会放入一个 hash 表中。

 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

**更细一点的回答：**

**1.初始化时：**                                                                      runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。                  

**2.添加引用时：**                                                                  objc_initWeak函数会调用objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。                                  
  **3.释放时,调用clearDeallocating函数。**

clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。                                                  SideTable结构体是负责管理类的引用计数表和weak表，

##12.使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？

无论在MRC下还是ARC下均不需要，被关联的对象在生命周期内要比对象本身释放的晚很多，它们会在被 NSObject -dealloc 调用的object_dispose()方法中释放。                                                                  ** 详解： **                                                               

**1.**调用 -release ：                                                           

          引用计数变为零对象正在被销毁，生命周期即将结束. 不能再有新的 __weak 弱引用，否则将指向 nil.调用 [self dealloc]                          **2. **父类调用 -dealloc 继承关系中最直接继承的父类再调用 -dealloc 如果是 MRC 代码 则会手动释放实例变量们（iVars）继承关系中每一层的父类 都再调用 -dealloc                                                                **3. **NSObject 调 -dealloc 只做一件事：调用 Objective-C runtime 中object_dispose() 方法          

**4.**调用 object_dispose()为 C++ 的实例变量们（iVars）调用 destructors为 ARC 状态下的 实例变量们（iVars） 调用 -release 解除所有使用 runtime Associate方法关联的对象 解除所有 __weak 引用 调用 free()

 ##13.什么是method swizzling（俗称黑魔法)

      简单说就是进行方法交换在Objective-C中调用一个方法，其实是向一个对象发送消息，查找消息的唯一依据是selector的名字。 

      利用Objective-C的动态特性，可以实现在运行时偷换selector对应的方法实现，达到给方法挂钩的目的。

      每个类都有一个方法列表，存放着方法的名字和方法实现的映射关系，selector的本质其实就是方法名，IMP有点类似函数指针，指向具体的Method实现，通过selector就可以找到对应的IMP。                              换方法的几种实现方式利用 method_exchangeImplementations 交换两个方法的实现利用 class_replaceMethod 替换方法的实现利用 method_setImplementation 来直接设置某个方法的IMP



#多线程相关面试问题

多线程是一个比较轻量级的方法来实现单个应用程序内多个代码执行路径， 从技术角度来看,一个线程就是一个需要管理执行代码的内核级和应用级数据结 构组合。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/160520_9849a506_9027123.png "对3.png")

##1.iOS系统为我们提供的几种多线程技术各自的特点是咋样的？

GCD、NSOperation、NSThread。

**GCD**：简单的线程间同步，包括子线程分派、多读单写等场景的解决。

**NSOperation**：第三方框架AF，SD都会涉及到NSOperation，他可以对任务的状态进行控制，可以添加依赖，删除依赖。

**NSThread**：实现常驻线程。



#RunLoop相关面试问题

我相信大多数开发者一样，迷惑于runloop，最初只了解可以通过runloop一些监听事件的通知来做一些事情，优化性能。关于runloop源码的基础知识，可以参考下面的思维导图：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/160534_f5222e7e_9027123.png "对4.png")

**RunLoop运行流程**

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/160547_0df3c7f1_9027123.jpeg "对5.jpg")

没有事情的时候，Runloop处于休眠状态。当外部source将其唤醒后，它会依次处理接收到的timer/source，然后再次进入休眠。

##1.Runloop和线程是什么关系？

每条线程都有唯一的一个与之对应的RunLoop对象；

主线程的RunLoop已经自动创建，子线程的RunLoop需要主动创建；

RunLoop在第一次获取时创建，在线程结束时销毁

##2.Runloop的mode作用是什么？

指定事件在运行循环中的优先级的，

线程的运行需要不同的模式，去响应各种不同的事件，去处理不同情境模式。(比如可以优化tableview的时候可以设置UITrackingRunLoopMode下不进行一些操作，比如设置图片等。)

##3.以+scheduledTimerWithTimeInterval:的方式触发的timer，在滑动页面上的列表时，timer会暂停回调， 为什么？

滑动scrollView时，主线程的RunLoop会切换到UITrackingRunLoopMode这个Mode，执行的也是UITrackingRunLoopMode下的任务（Mode中的item），而timer是添加在NSDefaultRunLoopMode下的，所以timer任务并不会执行，只有当UITrackingRunLoopMode的任务执行完毕，runloop切换到NSDefaultRunLoopMode后，才会继续执行timer。

##4.如何解决在滑动页面上的列表时，timer会暂停回调？

将Timer放到NSRunLoopCommonModes中执行即可

##5.NSTImer使用时需要注意什么？

注意timer添加到runloop时应该设置为什么mode

注意timer在不需要时，一定要调用invalidate方法使定时器失效，否则得不到释放



#设计模式相关面试问题

设计模式（Design pattern）是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。
使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性。 毫无疑问，设计模式于己于他人于系统都是多赢的；模式使代码编制真正工程化；设计模式是软件工程的基石脉络，如同大厦的结构一样。

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/160559_755896bd_9027123.png "对6.png")



#架构/框架相关面试问题

“100个读者就有100个哈姆雷特”一样，对于架构的理解不同的软件工程师有不同的看法。架构设计往往是一个权衡的过程，每一个架构设计者都要考虑到各个因素，比如团队成员的技术水平、具体的业务场景、项目的成长阶段和开发周期。下图是小编的一些架构理念，仅供参考：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/164619_b45a3df0_9027123.png "对7.png")

##1.RAC中使用时线程问题？或者RAC的缺点？
##2.RAC中实现多个信号全部执行结束再执行与 多个信号任意一个结束就响应的处理方式？
##3.路由跳转的实现方式 ？



#算法相关面试问题

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/164632_f46933d3_9027123.png "对8.png")

##1.对以下一组数据进行降序排序（冒泡排序）。“24，17，85，13，9，54，76，45，5，63”
```
int main(int argc, char *argv[]) {
int array[10] = {24, 17, 85, 13, 9, 54, 76, 45, 5, 63};
int num = sizeof(array)/sizeof(int);
for(int i = 0; i < num-1; i++) {
for(int j = 0; j < num - 1 - i; j++) {
if(array[j] < array[j+1]) {
int tmp = array[j];
array[j] = array[j+1];
array[j+1] = tmp;
}
}
}
for(int i = 0; i < num; i++) {
printf("%d", array[i]);
if(i == num-1) {
printf("\n");
}
else {
printf(" ");
}
}
}
```

 ##2.对以下一组数据进行升序排序（选择排序）。“86, 37, 56, 29, 92, 73, 15, 63, 30, 8”
```
void sort(int a[],int n)
{
    int i, j, index;
    for(i = 0; i < n - 1; i++) {
        index = i;
        for(j = i + 1; j < n; j++) {
            if(a[index] > a[j]) {
                index = j;
            }
        }
        if(index != i) {
            int temp = a[i];
            a[i] = a[index];
            a[index] = temp;
        }
    }
}
int main(int argc, const char * argv[]) {
    int numArr[10] = {86, 37, 56, 29, 92, 73, 15, 63, 30, 8};
    sort(numArr, 10);
    for (int i = 0; i < 10; i++) {
        printf("%d, ", numArr[i]);
    }
    printf("\n");
    return 0;
}
```


##3.实现二分查找算法（编程语言不限）

```
int bsearchWithoutRecursion(int array[],int low,int high,int target) {
while(low <= high) {
int mid = (low + high) / 2;
if(array[mid] > target)
high = mid - 1;
else if(array[mid] < target)
low = mid + 1;
else  //findthetarget
return mid;
}
//the array does not contain the target
return -1;
}
----------------------------------------
递归实现
int binary_search(const int arr[],int low,int high,int key)
{
int mid=low + (high - low) / 2;
if(low > high)
return -1;
else{
if(arr[mid] == key)
return mid;
else if(arr[mid] > key)
return binary_search(arr, low, mid-1, key);
else
return binary_search(arr, mid+1, high, key);
}
}
```


##4.如何实现链表翻转（链表逆序）？


```
#include <stdio.h>
#include <stdlib.h>
typedef struct NODE {
    struct NODE *next;
    int num;
}node;
node *createLinkList(int length) {
    if (length <= 0) {
        return NULL;
    }
    node *head,*p,*q;
    int number = 1;
    head = (node *)malloc(sizeof(node));
    head->num = 1;
    head->next = head;
    p = q = head;
    while (++number <= length) {
        p = (node *)malloc(sizeof(node));
        p->num = number;
        p->next = NULL;
        q->next = p;
        q = p;
    }
    return head;
}
void printLinkList(node *head) {
    if (head == NULL) {
        return;
    }
    node *p = head;
    while (p) {
        printf("%d ", p->num);
        p = p -> next;
    }
    printf("\n");
}
node *reverseFunc1(node *head) {
    if (head == NULL) {
        return head;
    }
    node *p,*q;
    p = head;
    q = NULL;
    while (p) {
        node *pNext = p -> next;
        p -> next = q;
        q = p;
        p = pNext;
    }
    return q;
}
int main(int argc, const char * argv[]) {
    node *head = createLinkList(7);
    if (head) {
        printLinkList(head);
        node *reHead = reverseFunc1(head);
        printLinkList(reHead);
        free(reHead);
    }
    free(head);
    return 0;
}
```

#第三方库相关面试问题

![输入图片说明](https://images.gitee.com/uploads/images/2021/0601/164644_17f37afe_9027123.png "对9.png")



来源：公众号   iOS进阶宝典

[查看原文](https://mp.weixin.qq.com/s?__biz=MzI3MjU1MjE2NQ==&mid=2247485958&idx=1&sn=43c47c38d18e7a0178f9e10b1d0ad3ee&chksm=eb31938cdc461a9a864b32da55679af2f6a455ebcb2f8b69f305b60ac48bf9966411268e697a&token=1405749238&lang=zh_CN#rd)
